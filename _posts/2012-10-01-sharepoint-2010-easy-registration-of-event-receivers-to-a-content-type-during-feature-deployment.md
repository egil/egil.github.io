---
layout: post
title: 'SharePoint 2010: Easy registration of Event Receivers to a Content Type during
  Feature deployment'
created: 1349089564
redirect_from:
  - /2012/10/01/sharepoint-2010-easy-registration-event-receivers-content-type-during-feature-deployment/
---
When registering a custom event receiver to a content type during a feature deployment using the [SPEventReceiverDefinition class](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.speventreceiverdefinition.aspx), you need the full assembly name and the full class name. However, during development, it can be tedious to keep track of the assembly names, so to make things easier we can just grap it using the [typeof operator](http://msdn.microsoft.com/en-us/library/58918ffs(v=vs.110).aspx). This small reusable helper method makes this easy:

<!--break-->

RegisterContentTypeEventReceiver() helper method:

~~~csharp
private void RegisterContentTypeEventReceiver(SPContentType contentType, SPEventReceiverType type, Type target, int sequenceNumber) 
{
    SPEventReceiverDefinition addingEventReceiver = contentType.EventReceivers.Add();
    addingEventReceiver.Type = type;
    addingEventReceiver.Assembly = target.Assembly.FullName;
    addingEventReceiver.Class = target.FullName;
    addingEventReceiver.SequenceNumber = sequenceNumber;
    addingEventReceiver.Update();

    contentType.Update(true);
}
~~~

Example usage (added to the Features event receiver). Assumes a "Customers" content type and a CustomerEventReceiver class exists:

~~~csharp
public override void FeatureActivated(SPFeatureReceiverProperties properties)
{
    var web = (SPWeb)properties.Feature.Parent;    
	RegisterContentTypeEventReceiver(web.ContentTypes["Customers"], SPEventReceiverType.ItemUpdated, typeof(CustomerEventReceiver), 1000);
}
~~~

Hope this helps :)