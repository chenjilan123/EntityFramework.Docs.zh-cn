---
title: 查询标记 - EF Core
author: divega
ms.date: 11/14/2018
ms.assetid: 73C7A627-C8E9-452D-9CD5-AFCC8FEFE395
uid: core/querying/tags
ms.openlocfilehash: 3a4d516cab5836c659e42d825c4f1bf89355d671
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688741"
---
# <a name="query-tags"></a>查询标记
> [!NOTE]
> 此为 EF Core 2.2 中的新增功能。

此功能有助于将代码中的 LINQ 查询与日志中捕获的已生成 SQL 查询相关联。
使用新增的 `TagWith()` 方法对 LINQ 查询进行批注： 

``` csharp
  var nearestFriends =
      (from f in context.Friends.TagWith("This is my spatial query!")
      orderby f.Location.Distance(myLocation) descending
      select f).Take(5).ToList();
```

此 LINQ 查询转换为以下 SQL 语句：

``` sql
-- This is my spatial query!

SELECT TOP(@__p_1) [f].[Name], [f].[Location]
FROM [Friends] AS [f]
ORDER BY [f].[Location].STDistance(@__myLocation_0) DESC
```

可以对同一个查询多次调用 `TagWith()`。
查询标记具有累积性。
例如，给定方法如下：

``` csharp
IQueryable<Friend> GetNearestFriends(Point myLocation) =>
    from f in context.Friends.TagWith("GetNearestFriends")
    orderby f.Location.Distance(myLocation) descending
    select f;

IQueryable<T> Limit<T>(IQueryable<T> source, int limit) =>
    source.TagWith("Limit").Take(limit);
```

以下查询：   

``` csharp
var results = Limit(GetNearestFriends(myLocation), 25).ToList();
```

转换为：

``` sql
-- GetNearestFriends

-- Limit

SELECT TOP(@__p_1) [f].[Name], [f].[Location]
FROM [Friends] AS [f]
ORDER BY [f].[Location].STDistance(@__myLocation_0) DESC
```

也可以使用多线串作为查询标记。
例如:

``` csharp
var results = Limit(GetNearestFriends(myLocation), 25).TagWith(
@"This is a multi-line
string").ToList();
```

生成以下 SQL：

``` sql
-- GetNearestFriends

-- Limit

-- This is a multi-line
-- string

SELECT TOP(@__p_1) [f].[Name], [f].[Location]
FROM [Friends] AS [f]
ORDER BY [f].[Location].STDistance(@__myLocation_0) DESC
```

## <a name="known-limitations"></a>已知限制
**查询标记不可参数化：** EF Core 始终将 LINQ 查询中的查询标记视为，已生成 SQL 中包含的字符串文本。
禁止使用将查询标记用作参数的已编译查询。