---
layout: post
title: 'C#/.NET: Safe way to convert UTC time to local time when databinding'
created: 1193646229
redirect_from:
  - /2007/10/29/safe-way-to-convert-utc-time-to-local-time-when-databinding
  - /2011/09/27/safe-way-convert-utc-time-local-time-when-databinding
---
After reading [Scott Mitchell's article Using Coordinated Universal Time (UTC) to Store Date/Time Values](http://aspnet.4guysfromrolla.com/articles/081507-1.aspx), I decided to do as he suggest and use UTC for a web application I'm building at work. All in all it is fairly easy to deal with UTC in the .NET Framwork. There is a [DateTime.UtcNow](http://msdn2.microsoft.com/en-us/library/system.datetime.utcnow.aspx) you can use instead of the regular DateTime.Now property, and there is a [DateTime.ToLocalTime](http://msdn2.microsoft.com/en-us/library/system.datetime.tolocaltime.aspx) method that will convert your UTC DateTime value to the local server time zone, even taking daylight savings time in to consideration. When converting the other way, the [DateTime.ToUniversalTime](http://msdn2.microsoft.com/en-us/library/system.datetime.touniversaltime.aspx) method is provided.

<!--break-->

With MsSQL's getutcdate() function (instead of getdate()), just about everything is smooth sailing. The only problem I have encountered is when binding a to a ASP.NET server control. Scott suggests doing like this:

```csharp
((DateTime) Eval("DateUpdated")).ToLocalTime()
```

This only works when DateUpdated is not null, which is not always the case in my application. To get around this, I created the following simple class and stuck it in my App_Code folder.

```csharp
public static class Extensions
{
    /// <summary>
    /// If object is a DateTime, it will convert
    /// the DateTime to local time.
    /// </summary>
    /// <param name="obj"></param>
    /// <returns></returns>
    public static object ToLocalTime(this object obj)
    {
        if (obj is DateTime)
            return ((DateTime)obj).ToLocalTime();
        else
            return obj;
    }
}
```

The method `ToLocalTime(this object obj)` uses the extension method feature in .NET 3.0, that allows you to extend the functionality of other class. Here I extend upon Object, adding a `.ToLocalTime()` method to it. This allows me to use it in ASP.NET server controls like this:

```csharp
Eval("DateUpdated").ToLocalTime()
```

This works because `Eval()` returns a object, which is also the reason why I did not extend a more specific class.
