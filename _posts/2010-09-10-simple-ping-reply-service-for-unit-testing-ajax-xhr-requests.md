---
layout: post
title: Simple Ping/Reply Service for Unit Testing AJAX/XHR requests
created: 1284089210
---
To make it easier for me to unit test a [OData JavaScript library](http://github.com/egil/ODataJS) I am working, I created a small HTTP handler that will reply back to a AJAX request with the headers, HTTP verb and query string it received. My initial thought was to do this through a WCF service, and while this is certainly possible, I found it much easier just to create a simple HTTP handler.

===
~~~csharp
using System.IO;
using System.Linq;
using System.Runtime.Serialization.Json;
using System.Text;
using System.Web;

namespace ODataJS.services
{
    public class AjaxPingHandler : IHttpHandler
    {
        public void ProcessRequest(HttpContext context)
        {
            var request = context.Request;
            var response = context.Response;

            // serialize PingReply object
            string jsonText = string.Empty;
            using (var ms = new MemoryStream())
            {
                var jsonSerializer = new DataContractJsonSerializer(typeof(PingReply));
                jsonSerializer.WriteObject(ms, new PingReply(request));
                jsonText = Encoding.Default.GetString(ms.ToArray());
            }

            // set content type to json
            response.ContentType = request.AcceptTypes.Contains("application/json") ? "application/json" : "text/javascript";

            // wrap in d object to avoid script injection
            jsonText = "{\"d\":{" + jsonText.Substring(1, jsonText.Length - 1) + "}";

            // output content
            response.Write(jsonText);
        }

        public bool IsReusable
        {
            // To enable pooling, return true here.
            // This keeps the handler in memory.
            get { return false; }
        }
    }
}
~~~
===[The code for the *AjaxPingHandler.cs* handler.]

As you can see, it uses the native .net [DataContractJsonSerializer](http://msdn.microsoft.com/en-us/library/system.runtime.serialization.json.datacontractjsonserializer.aspx) to serialize a custom class called PingReply. The ContentType return header is determined by the AcceptTypes request header. If the AcceptTypes array contains "application/json" the ContentType header is set to the same, otherwise we set the ContentType header to "text/javascript".

Just for good measure, the result data is wrapped in a dummy object `{ "d" : . }`, just like .net's JSON enabled services does it. It is not because I am terrible worried about script injection attacks when doing unit tests, but it makes the service behave more like other .net JSON services, which makes things consistent.

===
~~~csharp
using System.Linq;
using System.Runtime.Serialization;
using System.Web;

namespace ODataJS.services
{
    [DataContract]
    public class PingReply
    {
        [DataMember]
        public string HTTPVerb { get; set; }

        [DataMember]
        public JsonDictionary Headers { get; private set; }

        [DataMember]
        public JsonDictionary QueryString { get; private set; }

        public PingReply(HttpRequest request)
        {
            Headers = new JsonDictionary();
            QueryString = new JsonDictionary();
            foreach (var key in request.Headers.AllKeys.Where(x => x != null))
                Headers.Add(key, request.Headers[key]);

            foreach (var key in request.QueryString.AllKeys.Where(x => x != null))
                QueryString.Add(key, request.QueryString[key]);

            HTTPVerb = request.HttpMethod;
        }
    }
}
~~~
===[The *PingReply.cs* class.]

The only thing interesting about the PingReply class is that it uses the JsonDictionary class to return the headers and query string instead of the native .net Dictionary collection class. I decided to use the JsonDictionary class because it serializes to a proper JavaScript object, and not a array of one-element objects like the .net Dictionary. [Carlos Figueira](http://social.msdn.microsoft.com/profile/carlos%20figueira/) first published the JsonDictionary class over at the [msdn forums](http://social.msdn.microsoft.com/Forums/en-US/wcf/thread/8bef40bc-8466-4c6f-a717-15f3d6e61e3c).

===
~~~csharp
using System;
using System.Collections.Generic;
using System.Runtime.Serialization;

namespace ODataJS.services
{
    [Serializable]
    public class JsonDictionary : ISerializable
    {
        readonly Dictionary<string, object> _dict = new Dictionary<string, object>();
        public JsonDictionary() { }
        protected JsonDictionary(SerializationInfo info, StreamingContext context)
        {
            foreach (var entry in info)
            {
                _dict.Add(entry.Name, entry.Value);
            }
        }

        public void GetObjectData(SerializationInfo info, StreamingContext context)
        {
            foreach (string key in _dict.Keys)
            {
                info.AddValue(key, _dict[key], _dict[key] == null ? typeof(object) : _dict[key].GetType());
            }
        }

        public void Add(string key, object value)
        {
            _dict.Add(key, value);
        }
    }
}
~~~
===[The *JsonDictionary.cs* class.]

To make it all work, you simply need to add the following to your web.config file:

~~~
<configuration>
  <system.web>
    <httpHandlers>
      <add verb="*" path="AjaxPing.ashx"
        type="ODataJS.services.AjaxPingHandler"/>
    </httpHandlers>
  </system.web>
  <system.webServer>
    <handlers>
      <add verb="*" path="AjaxPing.ashx"
        name="AjaxUnittestService"
        type="ODataJS.services.AjaxPingHandler"/>
    </handlers>
  </system.webServer>
</configuration>
~~~

Then it is just a matter of querying AjaxPing.ashx and unit test against the reply from the service.

I hope this helps others who also want to produce better JavaScript code.
