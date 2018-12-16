---
title: EF Core 2.2 中的新变化 - EF Core
author: divega
ms.date: 11/14/2018
ms.assetid: 998C04F3-676A-4FCF-8450-CFB0457B4198
uid: core/what-is-new/ef-core-2.2
ms.openlocfilehash: 79b4efc3aee23e19a9ea1deb6373b9984b77f886
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688742"
---
# <a name="new-features-in-ef-core-22"></a>EF Core 2.2 中的新增功能

## <a name="spatial-data-support"></a>空间数据支持

空间数据可用于表示对象的物理位置和形状。
许多数据库都可以本机存储、索引和查询空间数据。 常见方案包括，查询给定距离内的对象，以及测试多边形是否包含给定位置。
EF Core 2.2 现在支持使用 [NetTopologySuite](https://github.com/NetTopologySuite/NetTopologySuite) (NTS) 库中的类型，处理各种数据库中的空间数据。

空间数据支持是作为一系列提供程序专用扩展包进行实现。
每个包都为 NTS 类型和方法以及数据库中相应的空间类型和函数提供映射。
此类提供程序扩展现在可用于 [SQL Server](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer.NetTopologySuite/)、[SQLite](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Sqlite.NetTopologySuite/) 和 [PostgreSQL](https://www.nuget.org/packages/Npgsql.EntityFrameworkCore.PostgreSQL.NetTopologySuite/)（来自 [Npgsql 项目](http://www.npgsql.org/)）。
空间类型可以直接与 [EF Core 内存中提供程序](https://docs.microsoft.com/en-us/ef/core/providers/in-memory/)一起使用，无需使用额外扩展。

安装提供程序扩展后，便能向实体添加受支持类型的属性。 例如:

``` csharp
using NetTopologySuite.Geometries;

namespace MyApp
{
  public class Friend
  {
    [Key]
    public string Name { get; set; }
  
    [Required]
    public Point Location { get; set; }
  }
}
``` 

然后，可以暂留包含空间数据的实体：

``` csharp
using (var context = new MyDbContext())
{
    context.Add(
        new Friend
        {
            Name = "Bill",
            Location = new Point(-122.34877, 47.6233355) {SRID = 4326 }
        });
    context.SaveChanges();
}
```
还可以根据空间数据和操作执行数据库查询：

``` csharp
  var nearestFriends =
      (from f in context.Friends
      orderby f.Location.Distance(myLocation) descending
      select f).Take(5).ToList();
```

若要详细了解此功能，请参阅[空间类型文档](xref:core/modeling/spatial)。 

## <a name="collections-of-owned-entities"></a>从属实体集合

EF Core 2.0 增加了在一对一关联中对所有权进行建模的功能。
EF Core 2.2 将此功能扩展为，将所有权表达为一对多关联。
所有权有助于约束实体的使用方式。

例如，从属实体：
- 只能出现在其他实体类型的导航属性中。 
- 自动加载，并且只能被 DbContext 和所有者跟踪。

在关系数据库中，从属集合映射到所有者的各个表，就像是常规的一对多关联一样。
不过，在面向文档的数据库中，我们计划将（从属集合或引用中的）从属实体与所有者嵌套在同一个文档中。

若要使用此功能，可以调用新增的 OwnsMany() API：

``` csharp
modelBuilder.Entity<Customer>().OwnsMany(c => c.Addresses);
```

有关详细信息，请参阅[更新后的从属实体文档](xref:core/modeling/owned-entities#collections-of-owned-types)。

## <a name="query-tags"></a>查询标记

此功能简化了将代码中的 LINQ 查询与日志中捕获的已生成 SQL 查询相关联的过程。

若要利用查询标记，请使用新增的 TagWith() 方法对 LINQ 查询进行批注。
使用上一示例中的空间查询：

``` csharp
  var nearestFriends =
      (from f in context.Friends.TagWith(@"This is my spatial query!")
      orderby f.Location.Distance(myLocation) descending
      select f).Take(5).ToList();
```

此 LINQ 查询将生成以下 SQL 输出：

``` sql
-- This is my spatial query!

SELECT TOP(@__p_1) [f].[Name], [f].[Location]
FROM [Friends] AS [f]
ORDER BY [f].[Location].STDistance(@__myLocation_0) DESC
```

有关详细信息，请参阅[查询标记文档](xref:core/querying/tags)。 