---
layout: post
title: "SQL Server: (re)import CSV data the easy way"
date: 2014-04-12 19:31
comments: true
categories: [Databases, MS SQL]
keywords: "ms sql, sql server, databases, csv, import"
---
Importing data into SQL Server is not hard using the import data feature but I really don't appreciate the fact that there is no way to save that import profile/rules for later reuse. Solution: use BULK INSERT from a regular reusable sql script.

[BULK INSERT (Transact-SQL)](http://technet.microsoft.com/en-us/library/ms188365.aspx) is a really nice feature that allows to easily create a reusable data import script capable of import one or many CSV files. I'm going to build a simple script using this nice feature.
<!--more-->

For this example I have a simple CSV file containing a set of data from a fictional marketing contacts database.

{% codeblock Example CSV source %}
Id;Name;EmailAddress;MobileNumber
1;John;john@gmail.com;123456789
2;Mary;mary@gmail.com;123456789
...
{% endcodeblock  %}

In order to use BULK INSERT I need the target table structure to have the same structure as the source data. Not always this would be the case so I'm going to create a temporary table to hold my source data that will be retrieved from the BULK INSERT.

**Creating the temporary table:**

{% codeblock lang:sql Temporary table to hold CSV data %}
CREATE TABLE #tempSource_Table_Name (
    Id int,
    [Name] nvarchar(500),
    EmailAddress nvarchar(500) null,
    MobileNumber nvarchar(500) null
);
{% endcodeblock  %}

This temporary table columns definitions should reflect the source structure and columns types.

Now that I have a temporary table that can receive the source data from the CSV.

**Building the BULK INSERT script:**

{% codeblock lang:sql BULK INSERT %}
BULK INSERT #tempSource_Table_Name
FROM 'C:\csv_file_full_path\source_table_data.csv'
WITH
(
    FIRSTROW = 2,
    FIELDTERMINATOR = ';',
    ROWTERMINATOR = '\n',
    CODEPAGE = 'ACP'
);
{% endcodeblock  %}

I'm just applying a short set of the [available arguments](http://technet.microsoft.com/en-us/library/ms188365.aspx). Based on your data source apply the necessary arguments.

This is a very powerful tool. Just for the example, is possible to have a source data file from a UNC share, take control over encoding, etc.

Now that I have a temporary table containing the imported data I can insert it to my target table applying some extra import rules and tweaks.

**Building the insert script for the target table:**

{% codeblock lang:sql Insert data to target table  %}
INSERT INTO [dbo].[Target_Table_Name]
   ([Name]
   ,[MobileNumber]
   ,[EmailAddress]
   ,[Password]
   ,[LastKnownIPAddress]
   ,[CreatedOn]
   ,[ChangedOn]
   ,[VerifiedOn]
   ,[Blocked]
   ,[BlockEmailMarketing])
SELECT 
    [Name]
    ,[MobileNumber]
    ,(
        CASE
            WHEN [EmailAddress] IS NULL 
                THEN CONCAT(CONVERT(varchar, [Id]), '@example.com') 
            ELSE [EmailAddress] 
        END
    )
    ,NULL
    ,NULL
    ,(SELECT GETDATE())
    ,NULL
    ,NULL
    ,0
    ,0          
FROM #tempSource_Table_Name
-- apply any addional filtering/sorting as needed
-- WHERE [Id] IS NOT NULL
-- ORDER BY EmailAddress
{% endcodeblock  %}

This example shows that is quite easy to apply some data transformations to the source data before inserting it on my target table. All the power of Transact-SQL is available to make a serious import script using this strategy.

**Also I want to add to my import script some extra features:**

- perform all my operations wrapped within a transaction;
- sanitize data before import (useful I want to test the import multiple times);
- demonstrate that is easy to import multiple files in one run.

**Here the full script:**

{% codeblock lang:sql Complete script with comments  %}
/* 
    set target database
*/

USE [Target_Database_Name]
GO

/* 
    wrap everything in a transaction 
*/

BEGIN TRAN
GO

/* 
    (optional) sanitize target table(s)
*/

-- remove all rows
DELETE FROM [Target_Table_Name];
GO

-- if your the target table contains a identity key, reset its value
DBCC CHECKIDENT ([Target_Table_Name], reseed, 0)
GO

-- extra sanitization operations like disabling triggers, FK's, etc

/*
    creat a temp table, for each CSV that you wish to import, to hold CSV data

    example CSV:

    Id;Name;EmailAddress;MobileNumber
    1;John;john@gmail.com;123456789
    2;Mary;mary@gmail.com;123456789

    table structure and column types must match with the CSV
*/

CREATE TABLE #tempSource_Table_Name (
    Id int,
    [Name] nvarchar(500),
    EmailAddress nvarchar(500) null,
    MobileNumber nvarchar(500) null
);
GO

/*
    import CSV data to the temp table
*/

BULK INSERT #tempSource_Table_Name
FROM 'C:\csv_file_full_path\source_table_data.csv'
WITH
(
    FIRSTROW = 2,
    FIELDTERMINATOR = ';',
    ROWTERMINATOR = '\n',
    CODEPAGE = 'ACP'
);
GO

/*
    insert data to the target table using temp table as the source
    apply as many rules as needed to transform imput data
*/

INSERT INTO [dbo].[Target_Table_Name]
   ([Name]
   ,[MobileNumber]
   ,[EmailAddress]
   ,[Password]
   ,[LastKnownIPAddress]
   ,[CreatedOn]
   ,[ChangedOn]
   ,[VerifiedOn]
   ,[Blocked]
   ,[BlockEmailMarketing])
SELECT 
    [Name]
    ,[MobileNumber]
    ,(
        CASE
            WHEN [EmailAddress] IS NULL 
                THEN CONCAT(CONVERT(varchar, [Id]), '@example.com') 
            ELSE [EmailAddress] 
        END
    )
    ,NULL
    ,NULL
    ,(SELECT GETDATE())
    ,NULL
    ,NULL
    ,0
    ,0          
FROM #tempSource_Table_Name
-- apply any addional filtering/sorting as needed
-- WHERE [Id] IS NOT NULL
-- ORDER BY EmailAddress

GO

/*
    drop temp table(s)
*/

DROP TABLE #tempSource_Table_Name;
GO

/* 
    extra operations
*/

-- enable triggers, FK's, etc

/*
    all good, time to commit transaction
*/

COMMIT TRAN
GO
{% endcodeblock  %}
