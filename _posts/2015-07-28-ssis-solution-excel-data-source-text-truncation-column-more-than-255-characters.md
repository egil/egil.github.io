---
title: 'SSIS: Solution for Excel data source text truncation of column with more than 255 characters'
layout: post
---
When using SQL Server Integration Service (SSIS) Excel data source to import text from a column in a spreadsheet, the data source will automatically *try* to determine the maximum length of text in the text column. It does so by sampling the 9 first rows in the spreadsheet and measuring their length. If it does not find any column longer than 255 characters, it will set the columns type as DT_WSTR with a length of 255, otherwise it will use DT_NTEXT (see [Excel Source on TechNet](https://technet.microsoft.com/en-us/library/ms141683.aspx) for more details). 

<!--break-->

The obvious problem is that if any row after 9th row is longer than 255 characters, the import will break with the error message: **"Text was truncated or one or more characters had no match in the target code page."**

The suggested solution on TechNet is to change registry key to increase the number of rows the Excel data source will sample, and other suggestions online include changing the spreadsheet to have a larger row in the top 8 rows. However, neither solution is stable if you cannot change spreadsheet and if the spreadsheet changes each time you execute your SSIS package.

My solution is based on using the Script Component instead of the Excel data source. It allows you to define the column type and their size/length, and if you have choose your types and length right, the solution should be stable. The only other requirement is that there is a header row in the sheet you are trying to import from.

## Solution
To help guide us through the solution, we will use this very simple Excel spreadsheet as our example:

![](/assets/ssis-excel-import-sample-sheet.png)

1. Add an *Excel Connection Manager* to the package as you normally would. Point it to your source file, choose an Excel version, and check the *"First row has column names"* checkbox.  
   **Note:** This particular solution assums the first row in the Excel spreadsheet is a header row. This solution can be modified to work without, but not without changes to the script code. 

2. Add a Script Component as a Source and open the Script Component editor (Script Transformation Editor)

  1. Add your Excel Connection Manager in the *"Connection Managers"* tab. Make sure to name it `DataSourceConnection`, we will use that name in the script code later.
  ![](/assets/ssis-excel-import-script-component-connection-manager.png)  
  2. In the *"Inputs and Outputs"* tab, first rename the default *"Output 0"* to the *exact name* of the sheet in your Excel spreadsheet you want to import data from.
  3. Then continue adding the columns you want to export to the output, also naming each column with the *exact name* it has in the spreadsheet and pick the desired type.  
  **Note:** The names of the columns and sheet has to be the same since we use them to automatically generate the OleDB select statement that will read the data from the spread sheet.  
  ![](/assets/ssis-excel-import-script-component-input-output.png)
  4. Go to the Script tab and click the *"Edit Script..."* button. This will open a new instance of Visual Studio.

3. In the Visual Studio script edit, make the following changes to the script file:

Add the following using statements to the top of the file:

~~~csharp
using System.Data.OleDb;
using System.Text;
~~~

At the top of the `ScriptMain` class, add these three lines:

~~~csharp
private OleDbConnection connection;
private OleDbCommand command;
private OleDbDataReader reader;
~~~

Replace the `PreExecute()` and `PostExecute()` methods with the following new versions:

~~~csharp
public override void PreExecute()
{
  base.PreExecute();

  connection = new OleDbConnection(Connections.DataSourceConnection.ConnectionString);
  connection.Open();
  command = new OleDbCommand();
  command.Connection = connection;

  // Build select query by iterating over the columns in first output collection
  var query = new StringBuilder("SELECT ");
  var outputCollection = ComponentMetaData.OutputCollection[0];

  for (int i = 0; i < outputCollection.OutputColumnCollection.Count; i++)
  {
    query.AppendFormat(" [{0}]", outputCollection.OutputColumnCollection[i].Name);
    if (i < outputCollection.OutputColumnCollection.Count - 1) query.Append(",");
  }

  query.AppendFormat(" FROM [{0}]", outputCollection.Name);

  command.CommandText = query.ToString();
  reader = command.ExecuteReader();
}

public override void PostExecute()
{
  base.PostExecute();

  command = null;
  reader.Close();
  connection.Close();
}
~~~

Finally you need to do a little coding yourself in the `CreateNewOutputRows` method. Here, you parse each column in each row and add it to the output buffer. The following is the code used in the example:

~~~csharp
public override void CreateNewOutputRows()
{
  while (reader.Read())
  {
    // Replace with own import code matching the name
    // of the import buffer defined in the Script Transformation Editor
    // and column names
    NumbersandTextBuffer.AddRow();

    NumbersandTextBuffer.TimesUsed = (double)reader["Times Used"]
    NumbersandTextBuffer.TextShown = (string)reader["Text Shown"];
  }                     
}
~~~

Once you are done tweaking the `CreateNewOutputRows` method, close the Visual Studio script editor instance and you are done. The complete `ScriptMain` example file is posted below for reference. 

Hope this helps!

~~~csharp
using System;
using System.Data;
using Microsoft.SqlServer.Dts.Pipeline.Wrapper;
using Microsoft.SqlServer.Dts.Runtime.Wrapper;
using System.Data.OleDb;
using System.Text;

[Microsoft.SqlServer.Dts.Pipeline.SSISScriptComponentEntryPointAttribute]
public class ScriptMain : UserComponent
{
  private OleDbConnection connection;
  private OleDbCommand command;
  private OleDbDataReader reader;

  public override void PreExecute()
  {
    base.PreExecute();
 
    connection = new OleDbConnection(Connections.DataSourceConnection.ConnectionString);
    connection.Open();
    command = new OleDbCommand();
    command.Connection = connection;

    // Build select query by iterating over the columns in first output collection
    var query = new StringBuilder("SELECT ");
    var outputCollection = ComponentMetaData.OutputCollection[0];

    for (int i = 0; i < outputCollection.OutputColumnCollection.Count; i++)
    {
      query.AppendFormat(" [{0}]", outputCollection.OutputColumnCollection[i].Name);
      if (i < outputCollection.OutputColumnCollection.Count - 1) query.Append(",");
    }

    query.AppendFormat(" FROM [{0}]", outputCollection.Name);

    command.CommandText = query.ToString();
    reader = command.ExecuteReader();
  }

  public override void PostExecute()
  {
    base.PostExecute();

    command = null;
    reader.Close();
    connection.Close();
  }

  public override void CreateNewOutputRows()
  {
    while (reader.Read())
    {
      // Replace with own import code matching the name
      // of the import buffer defined in the Script Transformation Editor
      // and column names
      NumbersandTextBuffer.AddRow();

      NumbersandTextBuffer.TimesUsed = (double)reader["Times Used"]
      NumbersandTextBuffer.TextShown = (string)reader["Text Shown"];
    }                     
  }
}
~~~
