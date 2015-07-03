---
layout: post
title: System.Diagnostics + ASP.Net Web Site – remember to set compilerOptions="/d:TRACE"
created: 1212549020
redirect_from:
  - /2008/06/03/systemdiagnostics-aspnet-website-remember-to-set-compileroptions-d-trace
  - /2008/06/03/systemdiagnostics-aspnet-web-site-remember-set-compileroptions-dtrace
---
Today I was looking at [Ukadc.Diagnostics](http://ukadcdiagnostics.codeplex.com/), an extension to the [System.Diagnostics namespace](http://msdn.microsoft.com/en-us/library/system.diagnostics.aspx), and decided to build a test website myself to get more familiar with it. After reading through [Ukadc.Diagnostics Reference Example](http://www.codeplex.com/UkadcDiagnostics/Wiki/View.aspx?title=QuickReferenceExample&referringTitle=Home) I thought it would be a homerun in the first try, but it turns out that without adding `compilerOptions="/d:TRACE"` to each compiler in the compilers section in the Web.config, the tracing code is never compiled into the assemblies at runtime. Needless to say, I was scratching my head since my sample console application worked as expected the first time.

<!--break-->

My compilers section ended up looking like this (it is located in the system.codedom section):

```
<compilers>
  <compiler language="c#;cs;csharp" extension=".cs"
            compilerOptions="/d:TRACE"
            warningLevel="4"
            type="Microsoft.CSharp.CSharpCodeProvider, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089">
    <providerOption name="CompilerVersion" value="v3.5"/>
    <providerOption name="WarnAsError" value="false"/>
  </compiler>
  <compiler language="vb;vbs;visualbasic;vbscript"
            compilerOptions="/d:Trace=true"
            extension=".vb"
            warningLevel="4"
            type="Microsoft.VisualBasic.VBCodeProvider, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089">
    <providerOption name="CompilerVersion" value="v3.5"/>
    <providerOption name="OptionInfer" value="true"/>
    <providerOption name="WarnAsError" value="false"/>
  </compiler>
</compilers>
```

Do note that this is not an issue with ASP.Net Web Applications, since they are precompiled. For more on the subject, [MSDN has a great article](http://msdn.microsoft.com/en-us/library/b0ectfxd.aspx) which certainly saved me in this instance.
