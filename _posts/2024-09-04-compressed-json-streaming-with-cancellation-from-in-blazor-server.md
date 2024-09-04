---
title: 'Blazor Server: Stream large amount of data to JavaScript with cancellation'
layout: post
---

In a recent Blazor Server (.NET 8) project I needed to send a large amount of data to Plotly.js (+ 50 MB of data). Blazor already have [support for streaming data from .NET to JavaScript](https://learn.microsoft.com/en-us/aspnet/core/blazor/javascript-interoperability/call-javascript-from-dotnet?view=aspnetcore-8.0#stream-from-net-to-javascript), however, I wanted to also add support for compression and cancellation of an ongoing stream of data, in case the user changes their mind and wants to view a different set of data, before the current set of data has been sent.

<!--break-->

To eanble compression and cancellation, the following is needed:

- Custom JavaScript that can handle decompression and cancellation.
- A new .NET type that handles the serialization and compression of data, and cancellation from the server.

The `StreamingDataJsInterop` is the main component on the server side of things. It will allow one active stream of data to be sent at the time. Sending new data cancelled any active data streaming before sending the new data. If you want multiple streams of data concurrently, just instantiate as many `StreamingDataJsInterop` as you need.

```csharp
using System.IO.Compression;
using System.Text.Json;
using System.Text.Json.Serialization;
using Microsoft.Extensions.Logging.Abstractions;
using Microsoft.JSInterop;

namespace Microsoft.JSInterop.Streaming;

public sealed partial class StreamingDataJsInterop : IDisposable
{
    private readonly IJSRuntime jsRuntime;
    private readonly ILogger logger;
    private readonly CompressionLevel compressionLevel;
    private readonly JsonSerializerOptions jsonSerializerOptions;
    private readonly object sendLock = new();
    private CancellationTokenSource cancellationTokenSource;
    private bool streamingActive;
    private Task? currentSendTask;

    public StreamingDataJsInterop(
        IJSRuntime jsRuntime,
        ILogger? logger = null,
        CompressionLevel compressionLevel = CompressionLevel.Optimal,
        JsonSerializerOptions? jsonSerializerOptions = null)
    {
        this.jsRuntime = jsRuntime;
        this.logger = logger ?? NullLogger.Instance;
        this.compressionLevel = compressionLevel;
        this.jsonSerializerOptions = jsonSerializerOptions ?? JsonSerializerOptions.Default;
        cancellationTokenSource = new();
        RegisterAbortJsCallback();
    }

    public void Dispose()
    {
        cancellationTokenSource.Cancel();
        cancellationTokenSource.Dispose();
    }

    /// <summary>
    /// Invokes the specified JavaScript function asynchronously.
    /// </summary>
    /// <remarks>
    /// If a previous call is still actively streaming data to the JavaScript, that
    /// call is cancelled and this call is then started.
    /// </remarks>
    /// <param name="identifier">An identifier for the function to invoke. For example, the value <c>"someScope.someFunction"</c> will invoke the function <c>window.someScope.someFunction</c>.</param>
    /// <param name="data">The data that should be compressed and serialized to JSON before sending to the client.</param>
    /// <param name="args">JSON-serializable arguments.</param>
    /// <returns>A <see cref="Task"/> that represents the asynchronous invocation operation.</returns>
    [SuppressMessage("Performance", "CA1849:Call async methods when in an async method", Justification = "Cannot await in lock.")]
    public Task InvokeStreamingVoidAsync<TStreamData>(string identifier, TStreamData data, params object?[]? args)
    {
        // Only allow one streaming invocation to be active at
        // the same time. If there is one that is active, cancel the
        // previous before proceeding.
        lock (sendLock)
        {
            if (currentSendTask is { IsCompleted: false })
            {
                LogCancellingActiveStream();
                cancellationTokenSource.Cancel();
                cancellationTokenSource.Dispose();
                cancellationTokenSource = new();
                RegisterAbortJsCallback();
            }

            // We have to use Task.Run, otherwise
            // the task is started on Blazor dispatcher and will
            // block it until the task completes.
            currentSendTask = Task.Run(
                () => StartStreaming(identifier, data, cancellationTokenSource.Token, args),
                cancellationTokenSource.Token);

            return currentSendTask;
        }
    }

    [SuppressMessage("Reliability", "CA2012:Use ValueTasks correctly", Justification = "Unable to await in cancellation token registration callback.")]
    private void RegisterAbortJsCallback()
    {
        cancellationTokenSource.Token.Register(
            static (state, token) =>
            {
                var @this = (StreamingDataJsInterop)state!;
                if (@this.streamingActive)
                {
                    @this.LogSendingAbortSignal();
                    _ = @this.jsRuntime.InvokeVoidAsync("abortDotNetJsonStream", token.GetHashCode());
                }
            },
            this);
    }

    private async Task StartStreaming<TStreamData>(string identifier, TStreamData data, CancellationToken cancellationToken, params object?[]? args)
    {
        try
        {
            LogComressingData(identifier);
            var stream = await CreateCompressedJsonStream(data, cancellationToken);

            // Setting streamingActive signals that an abort signal
            // should be sent to JavaScript from the cancellation token
            // registration.
            // Having the boolean toggle avoids unnecessary abort calls
            // to JavaScrip if streaming was cancelled during compression.
            streamingActive = true;

            LogStartStreaming(identifier, stream.Length);
            var streamRef = new DotNetGZippedJsonStreamReference
            {
                Ref = new DotNetStreamReference(stream),
                StreamId = cancellationToken.GetHashCode(),
            };

            await jsRuntime.InvokeVoidAsync(
                identifier,
                cancellationToken,
                args is { Length: > 0 } ? [streamRef, .. args] : [streamRef]);

            LogFinishedStreaming(identifier);
        }
        catch (OperationCanceledException ex)
        {
            LogCancelledStreaming(ex, identifier);
        }
        finally
        {
            streamingActive = false;
        }
    }

    private async Task<MemoryStream> CreateCompressedJsonStream<TData>(TData data, CancellationToken cancellationToken)
    {
        // Stream is closed by DotNetStreamReference/JSInterop when its done sending or streaming gets cancelled.
        // Thus, zipStream should leave it open.
        var resultStream = new MemoryStream();

        await using (var zipStream = new GZipStream(resultStream, compressionLevel, leaveOpen: true))
        {
            await JsonSerializer.SerializeAsync(zipStream, data, jsonSerializerOptions, cancellationToken: cancellationToken);
        }

        resultStream.Position = 0;
        return resultStream;
    }

    private sealed class DotNetGZippedJsonStreamReference
    {
        [JsonPropertyName(name: "__dotNetGZippedJsonStream")]
        public required int StreamId { get; init; }

        [JsonPropertyName(name: "ref")]
        public required DotNetStreamReference Ref { get; init; }
    }

    [LoggerMessage(
        Level = LogLevel.Debug,
        Message = "Compressing data for {Identifier}.")]
    private partial void LogComressingData(string identifier);

    [LoggerMessage(
        Level = LogLevel.Debug,
        Message = "Start streaming {Size} data to {Identifier}.")]
    private partial void LogStartStreaming(string identifier, long size);

    [LoggerMessage(
    Level = LogLevel.Debug,
    Message = "Finished streaming data to {Identifier}.")]
    private partial void LogFinishedStreaming(string identifier);

    [LoggerMessage(
        Level = LogLevel.Debug,
        Message = "Cancelling active stream.")]
    private partial void LogCancellingActiveStream();

    [LoggerMessage(
        Level = LogLevel.Debug,
        Message = "Sending abort signal to JavaScript.")]
    private partial void LogSendingAbortSignal();

    [LoggerMessage(
        Level = LogLevel.Debug,
        Message = "Cancelled streaming data to {Identifier}.")]
    private partial void LogCancelledStreaming(OperationCanceledException exception, string identifier);
}
```

On the other end of the SignalR connection we need a bit of JavaScript that can unpack the compressed data before it is passed to the JavaScript method, e.g., `gzip-json-stream-interop.js`: 

```js
// gzip-json-stream-interop.js
const dotNetGZippedJsonStream = "__dotNetGZippedJsonStream";
const abortControllers = {};

DotNet.attachReviver((key, value) => {
  if (value && typeof value === "object" && value.hasOwnProperty(dotNetGZippedJsonStream)) {
    const streamId = value[dotNetGZippedJsonStream];
    const ref = value["ref"];
    return new DotNetGZippedJsonStream(ref, streamId, abortControllers);
  }

  // Unrecognized - let another reviver handle it
  return value;
});

function abortDotNetJsonStream(streamId) {
  if (abortControllers[streamId]) {
    abortControllers[streamId].abort();
  }
}

class DotNetGZippedJsonStream {
  constructor(_streamRef, _streamId, _abortControllers) {
    this._streamRef = _streamRef;
    this._streamId = _streamId;
    this._abortControllers = _abortControllers;
  }

  async getData() {
    const controller = new AbortController();
    this._abortControllers[this._streamId] = controller;
    try {
      const stream = await this._streamRef.stream();
      const decompressedStream = stream.pipeThrough(
        new DecompressionStream('gzip'),
        { signal: controller.signal });
      const json = await new Response(decompressedStream).json();
      return json;
    } catch (err) {
      if (controller.signal.aborted)
        throw new Error('GZipped JSON stream was aborted from .NET');
      else
        throw err;
    }
    finally {
      delete this._abortControllers[this._streamId];
    }
  }
}
```


Hope this help!
