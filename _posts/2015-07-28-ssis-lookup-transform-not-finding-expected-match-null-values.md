---
title: 'SSIS: Lookup Transform not finding expected match with NULL values'
layout: post
---
If you use the Lookup Transform to perform lookups with columns that can be NULL, the Lookup Transform will fail to find expected matches because in a database world, [NULL does not equal NULL](http://stackoverflow.com/questions/1843451/why-does-null-null-evaluate-to-false-in-sql-server#answer-1843460).

So how do we make the Lookup Transform match as expected when source column and lookup column can be NULL? We use [ISNULL](https://msdn.microsoft.com/en-us/library/ms184325.aspx) function, partial caching and a custom query in the Lookup Transform. 

Let us look at a quick example. This is our lookup table definition, which allows NULLs in the LastName column, that we want to get the Id of a person from, based on their first and last name:

```SQL
CREATE TABLE Person (
  Id int IDENTITY(1,1) NOT NULL,
  FirstName varchar(50) NOT NULL,
  LastName varchar(50) NULL
)
```

In SSIS, we have a input source with a `FirstName` column and a `LastName`, where the `LastName` column might sometimes be NULL. 
To find the `Id` of each name, we add a Lookup Transform to the Data Flow Task, and configure it to use *Partial cache* in the *General* tab. In the *Connection* tab we point to our `Person` table as normal, and in the *Columns* tab we map our input's `FirstName` column and `LastName` column to the lookup's `FirstName` and `LastName` columns.

Until now, every step has been the same as with *Full caching*. However, now we continue to the *Advanced* tab and check the **Modify the SQL statement* checkbox and change the SQL like so:

```SQL
select * from (select * from [Person]) [refTable]
where [refTable].[FirstName] = ? and ISNULL([refTable].[LastName], '') = ISNULL(?, '')
```

With the help of the `ISNULL` function, we rewrite any NULL values to an empty string, which is comparable, but only if the LastName is actually NULL.

Last step is to click the *Parameters* button and assign the parameters.

Hope this helps!