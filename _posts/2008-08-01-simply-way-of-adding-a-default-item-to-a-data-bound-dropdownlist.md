---
layout: post
title: Simply way of adding a default item to a data bound DropDownList
created: 1217559949
redirect_from:
  - /2008/07/31/simply-way-of-adding-a-default-item-to-a-data-bound-dropdownlist/
  - /2008/07/31/simply-way-adding-default-item-data-bound-dropdownlist/
---
I have always found it too difficult to add a *"Choose xxxxx"* item to the top of a `DropDownList`, when the `DropDownList` binds to a `ObjectDataSource`, `SqlDataSource` or similar. I almost always ended up just feeding the `DropDownList` its content from the code behind, even though it felt like something that should be possible directly. The control do have a `AppendDataBoundItems` property, but most of the time you just end up with duplicated items.

<!--break-->

Today I discovered the simplest and most elegant solution I’ve seen to date. One line of code in the code behind file is still required, but the elegance of `DataSource` controls is kept. In short, once the `DropDownList` has bound to its data source, add the *"Choose xxxxx"* item to the items collection of the `DropDownList`. The trick is to use the Insert method instead of the `Add` method, since the Insert method allows you to specify where the new item should be inserted, and not just added to the end of the item list.

In this example I use the `OnDataBound` event to insert the new item:

```
<asp:DropDownList ID="ddlCustomer" runat="server"
    DataSourceID="odsCustomer" DataTextField="DisplayName"
    DataValueField="CustomerId" OnDataBound="ddlCustomer_DataBound">
</asp:DropDownList>
```

The code behind is just one line of code (excluding the method definition):

```
protected void ddlCustomer_DataBound(object sender, EventArgs e)
{
    ddlCustomer.Items.Insert(0, new ListItem("Choose a Customer", "0"));
}
```
