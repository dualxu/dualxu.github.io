---
layout:       post
title:        "Access数据库UPDATE语句的语法错误问题"
subtitle:     "关键字命名的字段更新引起异常的解决方案"
date:         2018-06-30 13:45:53
author:       "Devdog"
header-img:   "img/in-post/post-bg/post-bg-random30.jpg"
header-mask:  0.3
catalog:      windows
multilingual: false
tags:
    - SQL
    - Access
---



## UPDATE语句的语法错误问题 ##

昨天项目上由于时间迫近，想到一个临时解决方案。使用Access数据库保存和获取本地离线图书信息，晚上碰到一个奇怪的问题百思不得其解。当用update语句时，发生“UPDATE 语句的语法错误”的异常，但看到半夜也没发现语法有啥不对。

<pre><code>
strSQL = string.Format("UPDATE t_bookinfo_tjxh SET count = count + 1"
                                        + " WHERE isbn = '" + listv.isbn + "'"
                                        );
</code></pre>


## UPDATE语法 ##

还是让我们来看看Access中SQL语句update的语法吧。
微软上的说明参见这里[UPDATE Statement (Microsoft Access SQL)](https://docs.microsoft.com/en-us/previous-versions/office/developer/office-2007/bb221186(v=office.12))。

>Syntax
UPDATE table SET newvalue WHERE criteria;

The UPDATE statement has these parts:

- table	The name of the table containing the data you want to modify.
- newvalue	An expression that determines the value to be inserted into a particular field in the updated records.
- criteria	An expression that determines which records will be updated. Only records that satisfy the expression are updated.

举一个栗子:
<pre><code>
UPDATE Orders
  
SET OrderAmount = OrderAmount * 1.1,
  
Freight = Freight * 1.03
  
WHERE ShipCountry = 'UK';
</code></pre>

我的update语句是不是没毛病？


## 解决方案 ##

一通百度/Google之后，原来我定义表格式时将关键字count定义为了字段，这个字段在update时将会发生语法错误的异常，解决办法是将关键字用[]扩起来。

Access和MS SQL一样，对关键字使用"["和"]"来进行转义，如果我们在数据库字段定义中使用了关键字，则需要使用转义符来进行转义，比如[user].[password]等等。不过，强烈建议在数据库设计时尽量避免使用关键字作为字段名。

修改如下:
<pre><code>
strSQL = string.Format("UPDATE t_bookinfo_tjxh SET [count] = [count] + 1"
                                        + " WHERE isbn = '" + listv.isbn + "'"
                                        );
</code></pre>




## 关键字 ##

最后还是附上这些关键字吧(以后再也不用Access了)！

- A
	-     ADD
	-     ALL
	-     Alphanumeric
	-     ALTER
	-     AND
	-     ANY
	-     Application
	-     AS
	-     ASC
	-     Assistant
	-     AT
	-     AUTOINCREMENT
	-     Avg
- B
	-     BETWEEN
	-     BINARY
	-     BIT
	-     BOOLEAN
	-     BY
	-     BYTE
- C
	-     CHAR, CHARACTER
	-     COLUMN
	-     CompactDatabase
	-     CONSTRAINT
	-     Container
	-     Count
	-     COUNTER
	-     CREATE
	-     CreateDatabase
	-     CreateField
	-     CreateGroup
	-     CreateIndex
	-     CreateObject
	-     CreateProperty
	-     CreateRelation
	-     CreateTableDef
	-     CreateUser
	-     CreateWorkspace
	-     CURRENCY
	-     CurrentUser
- D
	-     DATABASE
	-     DATE
	-     DATETIME
	-     DELETE
	-     DESC
	-     Description
	-     DISALLOW
	-     DISTINCT
	-     DISTINCTROW
	-     Document
	-     DOUBLE
	-     DROP
- E
	-     Echo
	-     Else
	-     End
	-     Eqv
	-     Error
	-     EXISTS
	-     Exit
- F
	-     FALSE
	-     Field, Fields
	-     FillCache
	-     FLOAT, FLOAT4, FLOAT8
	-     FOREIGN
	-     Form, Forms
	-     FROM
	-     Full
	-     FUNCTION
- G
	-     GENERAL
	-     GetObject
	-     GetOption
	-     GotoPage
	-     GROUP
	-     GROUP BY
	-     GUID
- H
	-     HAVING
- I
	-     Idle
	-     IEEEDOUBLE, IEEESINGLE
	-     If
	-     IGNORE
	-     Imp
	-     IN
	-     INDEX
	-     Index, Indexes
	-     INNER
	-     INSERT
	-     InsertText
	-     INT, INTEGER, INTEGER1, INTEGER2, INTEGER4
	-     INTO
	-     IS
- J
	-     JOIN
- K
	-     KEY
- L
	-     LastModified
	-     LEFT
	-     Level
	-     Like
	-     LOGICAL, LOGICAL1
	-     LONG, LONGBINARY, LONGTEXT
               
- M
	-     Macro
	-     Match
	-     Max, Min, Mod
	-     MEMO
	-     Module
	-     MONEY
	-     Move
- N
	-     NAME
	-     NewPassword
	-     NO
	-     Not
	-     Note
	-     NULL
	-     NUMBER, NUMERIC
- O
	-     Object
	-     OLEOBJECT
	-     OFF
	-     ON
	-     OpenRecordset
	-     OPTION
	-     OR
	-     ORDER
	-     Orientation
	-     Outer
	-     OWNERACCESS
- P
	-     Parameter
	-     PARAMETERS
	-     Partial
	-     Password
	-     PERCENT
	-     PIVOT
	-     PRIMARY
	-     PROCEDURE
	-     Property
- Q
	-     Queries
	-     Query
	-     Quit
- R
	-     REAL
	-     Recalc
	-     Recordset
	-     REFERENCES
	-     Refresh
	-     RefreshLink
	-     RegisterDatabase
	-     Relation
	-     Repaint
	-     RepairDatabase
	-     Report
	-     Reports
	-     Requery
	-     RIGHT
- S
	-     SCREEN
	-     SECTION
	-     SELECT
	-     SET
	-     SetFocus
	-     SetOption
	-     SHORT
	-     SINGLE
	-     Size
	-     SMALLINT
	-     SOME
	-     SQL
	-     StDev, StDevP
	-     STRING
	-     Sum
- T
	-     TABLE
	-     TableDef, TableDefs
	-     TableID
	-     TEXT
	-     TIME, TIMESTAMP
	-     TOP
	-     TRANSFORM
	-     TRUE
	-     Type
- U
	-     UNION
	-     UNIQUE
	-     UPDATE
	-     USER
- V
	-     VALUE
	-     VALUES
	-     Var, VarP
	-     VARBINARY, VARCHAR
- W
	-     WHERE
	-     WITH
	-     Workspace
- X
	-     Xor
- Y
	-     Year
	-     YES
	-     YESNO
