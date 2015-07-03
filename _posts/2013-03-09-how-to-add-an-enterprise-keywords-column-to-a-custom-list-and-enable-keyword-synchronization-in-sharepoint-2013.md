---
layout: post
title: How to add an Enterprise Keywords column to a custom list and enable Keyword
  synchronization in SharePoint 2013
created: 1362846664
redirect_from:
  - /2013/03/09/how-add-enterprise-keywords-column-custom-list-and-enable-keyword-synchronization
---
This is a quick step-by-step guide to add an Enterprise Keywords column to a custom list and enable Keyword synchronization with Visual Studio 2012 in a SharePoint project. The SharePoint tools for Visual Studio 2012 have become a lot better in the latest edition, but a little extra work is required to get the Enterprise keywords column wired up correctly.

<!--break-->

**Step 1:** Create an empty SharePoint 2013 project.

**Step 2:** Add *Content Type* and add these three columns to it manually through the *Elements.xml* file or through the graphical interface:

```xml
<FieldRef ID="{23f27201-bee3-471e-b2e7-b64fd8b7ca38}" 
	DisplayName="$Resources:osrvcore,field_KeywordsFieldName" 
	Required="FALSE" 
	Description="$Resources:osrvcore,field_KeywordsFieldDesc" 
	Hidden="FALSE" 
	Name="TaxKeyword" 
	Sealed="TRUE" 
	Sortable="FALSE" />
<FieldRef ID="{1390a86a-23da-45f0-8efe-ef36edadfb39}"
	DisplayName="TaxKeywordTaxHTField" 
	Required="FALSE" 
	Hidden="TRUE" 
	Name="TaxKeywordTaxHTField" />
<FieldRef ID="{f3b0adf9-c1a2-4b02-920d-943fba4b3611}" 
	DisplayName="Taxonomy Catch All Column" 
	Required="FALSE" 
	Hidden="TRUE" 
	Name="TaxCatchAll" 
	Sealed="TRUE" 
	Sortable="FALSE" />
```

**Step 3:** Add a *List* to the project; add the custom content type you created previously content. Alternatively, if you already have an existing list, add these fields definitions inside the `<Fields>` tag in lists *Schema.xml* file:

```xml
<Field Type='TaxonomyFieldTypeMulti'
  DisplayName='Enterprise Keywords'
  StaticName='TaxKeyword'
  Name='TaxKeyword'
  ID='{23f27201-bee3-471e-b2e7-b64fd8b7ca38}'
  List="Lists/TaxonomyHiddenList"
  WebId='~sitecollection'
  ShowInViewForms='TRUE'
  DefaultListField='TRUE'
  Required='FALSE'
  Hidden='FALSE'
  ShowField="Term$Resources:core,Language;"
  Mult='TRUE'
  Sortable='FALSE'
  Group="$Resources:osrvcore,field_KeywordsGroupName"
  Description="$Resources:osrvcore,field_KeywordsFieldDesc"
  AllowDeletion='TRUE'>
  <Customization>
    <ArrayOfProperty>
      <Property>
        <Name>SspId</Name>
        <Value
          xmlns:q1='http://www.w3.org/2001/XMLSchema' p4:type='q1:string'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>00000000-0000-0000-0000-000000000000</Value>
      </Property>
      <Property>
        <Name>GroupId</Name>
      </Property>
      <Property>
        <Name>TermSetId</Name>
        <Value
          xmlns:q2='http://www.w3.org/2001/XMLSchema' p4:type='q2:string'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>00000000-0000-0000-0000-000000000000</Value>
      </Property>
      <Property>
        <Name>AnchorId</Name>
        <Value
          xmlns:q3='http://www.w3.org/2001/XMLSchema' p4:type='q3:string'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>00000000-0000-0000-0000-000000000000</Value>
      </Property>
      <Property>
        <Name>UserCreated</Name>
        <Value
          xmlns:q4='http://www.w3.org/2001/XMLSchema' p4:type='q4:boolean'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>false</Value>
      </Property>
      <Property>
        <Name>Open</Name>
        <Value
          xmlns:q5='http://www.w3.org/2001/XMLSchema' p4:type='q5:boolean'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>true</Value>
      </Property>
      <Property>
        <Name>TextField</Name>
        <Value
          xmlns:q6='http://www.w3.org/2001/XMLSchema' p4:type='q6:string'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>{1390a86a-23da-45f0-8efe-ef36edadfb39}</Value>
      </Property>
      <Property>
        <Name>IsPathRendered</Name>
        <Value
          xmlns:q7='http://www.w3.org/2001/XMLSchema' p4:type='q7:boolean'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>false</Value>
      </Property>
      <Property>
        <Name>IsKeyword</Name>
        <Value
          xmlns:q8='http://www.w3.org/2001/XMLSchema' p4:type='q8:boolean'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>true</Value>
      </Property>
      <Property>
        <Name>TargetTemplate</Name>
      </Property>
      <Property>
        <Name>CreateValuesInEditForm</Name>
        <Value
          xmlns:q9='http://www.w3.org/2001/XMLSchema' p4:type='q9:boolean'
          xmlns:p4='http://www.w3.org/2001/XMLSchema-instance'>false</Value>
      </Property>
      <Property>
        <Name>FilterAssemblyStrongName</Name>
      </Property>
      <Property>
        <Name>FilterClassName</Name>
      </Property>
      <Property>
        <Name>FilterMethodName</Name>
      </Property>
      <Property>
        <Name>FilterJavascriptProperty</Name>
      </Property>
    </ArrayOfProperty>
  </Customization>
</Field>
<Field Type="Note"
  DisplayName="TaxKeywordTaxHTField"
  StaticName="TaxKeywordTaxHTField"
  Name="TaxKeywordTaxHTField"
  ID="{1390a86a-23da-45f0-8efe-ef36edadfb39}"
  ShowInViewForms="FALSE"
  Required="FALSE"
  Hidden="TRUE"
  CanToggleHidden="TRUE" />

<Field Type="LookupMulti"
  DisplayName="Taxonomy Catch All Column"
  StaticName="TaxCatchAll"
  Name="TaxCatchAll"
  ID="{f3b0adf9-c1a2-4b02-920d-943fba4b3611}"
  ShowInViewForms="FALSE"
  Required="FALSE"
  Hidden="TRUE"
  ShowField="CatchAllData"
  Mult="TRUE"
  List="Lists/TaxonomyHiddenList"
  WebId="~sitecollection"
  Sortable="FALSE"
  AllowDeletion="TRUE" 
  Sealed="TRUE" />
```

At this point, we are usually done. The columns were added to the list definition and the columns does indeed show up in the list. However, they keywords added to items in the list does not show up. What is missing is the event receiver that takes care of the keyword synchronization. We will add that now.

**Step 4:** Add an *Event Receiver* to the project. Make sure to select the list you just created as the event source when going through the wizard, and any events in the events list (we will replace the code generated, so it does not matter which).

Aside: I like to add event receivers beneath the list they are associated with so it is clear that what list the event receiver belongs to, but anywhere inside the project is fine.

**Step 5:** Delete the event receiver's code file (.cs or .vb) so you are only left with the *Elements.xml* file.

**Step 6:** Open the event receiver's *Elements.xml* file and replace whatever `<Receiver>` tags where generated with these:

```xml
<Receiver>
	<Name>TaxonomyItemSynchronousAddedEventReceiver</Name>
	<Synchronization>Synchronous</Synchronization>
	<Type>ItemAdding</Type>
	<SequenceNumber>10000</SequenceNumber>
	<Assembly>Microsoft.SharePoint.Taxonomy, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c</Assembly>
	<Class>Microsoft.SharePoint.Taxonomy.TaxonomyItemEventReceiver</Class>
	<Data></Data>
</Receiver>
<Receiver>
	<Name>TaxonomyItemUpdatingEventReceiver</Name>
	<Synchronization>Synchronous</Synchronization>
	<Type>ItemUpdating</Type>
	<SequenceNumber>10000</SequenceNumber>
	<Assembly>Microsoft.SharePoint.Taxonomy, Version=15.0.0.0, Culture=neutral, PublicKeyToken=71e9bce111e9429c</Assembly>
	<Class>Microsoft.SharePoint.Taxonomy.TaxonomyItemEventReceiver</Class>
	<Data></Data>
</Receiver>
```

The important thing to make sure is that the value of the `ListTemplateId` attribute on the `<Receivers ListTemplateId="xxxxx"> ... </Receivers>` tag is the same as the value of the ` Type="xxxxx"` attribute on the `< ListTemplate ... />` tag. Otherwise the event receivers will not attach to the right list.

That is it. If you need a guide for SharePoint 2010, please take a look at this [blog post over at metaengine.com](http://www.metaengine.com/2012/03/SharePoint-sandbox-solution---Document-Library-with-Enterprise-Keywords). It helped me get this working in SharePoint 2013.

Hope this helps.
