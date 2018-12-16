---
title: 自定义迁移历史记录表的 EF Core
author: bricelam
ms.author: bricelam
ms.date: 11/07/2017
ms.openlocfilehash: 1a253972a8f4e410421ec8a77c079e588d368819
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45488811"
---
<a name="custom-migrations-history-table"></a>自定义迁移历史记录表
===============================
默认情况下，EF Core通过名为`__EFMigrationsHistory`的表跟踪记录哪些迁移已应用于数据库中。 由于各种原因，你可能想要自定义此表，以更好地满足你的需求。

> [!IMPORTANT]
> 如果你在应用迁移*之后*才自定义了迁移历史记录表，那么您应负责更新数据库中的现有表。

<a name="schema-and-table-name"></a>架构和表名称
----------------------
您可以通过`OnConfiguring()`方法(或 ASP.NET Core 的`ConfigureServices()`方法)中的`MigrationsHistoryTable()`方法来更改架构和表名。 此处是一个使用 SQL Server EF Core提供程序的示例。

``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder options)
    => options.UseSqlServer(
        connectionString,
        x => x.MigrationsHistoryTable("__MyMigrationsHistory", "mySchema"));
```

<a name="other-changes"></a>其他更改
-------------
若要配置该表的其他方面，只要重写并替换数据库提供程序中`IHistoryRepository`服务的实现。 下面是在 SQL Server 中修改 MigrationId 列名称为 *Id* 的示例。

``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder options)
    => options
        .UseSqlServer(connectionString)
        .ReplaceService<IHistoryRepository, MyHistoryRepository>();
```

> [!WARNING]
> `SqlServerHistoryRepository` 是在一个内部命名空间中，在未来版本可能会更改。

``` csharp
class MyHistoryRepository : SqlServerHistoryRepository
{
    public MyHistoryRepository(HistoryRepositoryDependencies dependencies)
        : base(dependencies)
    {
    }

    protected override void ConfigureTable(EntityTypeBuilder<HistoryRow> history)
    {
        base.ConfigureTable(history);

        history.Property(h => h.MigrationId).HasColumnName("Id");
    }
}
```
