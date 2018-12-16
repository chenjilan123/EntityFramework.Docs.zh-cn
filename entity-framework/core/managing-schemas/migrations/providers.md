---
title: 多个提供程序的 EF Core 的迁移
author: bricelam
ms.author: bricelam
ms.date: 11/08/2017
ms.openlocfilehash: 75c055221609679db3f00016b9cb44c6c8c6e473
ms.sourcegitcommit: 2b787009fd5be5627f1189ee396e708cd130e07b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/13/2018
ms.locfileid: "45488772"
---
<a name="migrations-with-multiple-providers"></a>多个提供程序的迁移
==================================
[EF Core 工具][ 1]仅为指定的提供程序创建迁移的基架。 但是，有时你可能想在 DbContext 中使用多个提供程序 （例如 Microsoft SQL Server 和 SQLite） 。 有两种方法来处理这种迁移情况。 你可以选择维护两份迁移——每个提供程序一份，或者合并成同一份也行。

<a name="two-migration-sets"></a>两份迁移
------------------
第一种方法，为每个模型变更生成两个迁移。

一种处理方式是将两份迁移分别放[在单独的程序集中][ 2]，然后在添加迁移时，手动切换数据库提供程序（及其对应的迁移程序集）。

另一种处理方式让工具的使用更简便点，我们可以创建一个新的类派生自你的 DbContext ，并重写指定其使用哪个数据库提供程序。 此类型仅用于在开发设计阶段添加或应用迁移。

``` csharp
class MySqliteDbContext : MyDbContext
{
    protected override void OnConfiguring(DbContextOptionsBuilder options)
        => options.UseSqlite("Data Source=my.db");
}
```

> [!NOTE]
> 由于每个迁移集使用其自己的 DbContext 类型，此方法不需要使用独立的迁移程序集。

在添加新的迁移时，指定上下文类型。

``` powershell
Add-Migration InitialCreate -Context MyDbContext -OutputDir Migrations\SqlServerMigrations
Add-Migration InitialCreate -Context MySqliteDbContext -OutputDir Migrations\SqliteMigrations
```
``` Console
dotnet ef migrations add InitialCreate --context MyDbContext --output-dir Migrations/SqlServerMigrations
dotnet ef migrations add InitialCreate --context MySqliteDbContext --output-dir Migrations/SqliteMigrations
```

> [!TIP]
> 不需要指定之后迁移的输出目录，因为它们会被创建在最后一个迁移的同级目录中。

<a name="one-migration-set"></a>一份迁移
-----------------
如果您不喜欢维护两份迁移，可以手动将它们合并成一份可同时应用于两个提供程序的迁移。

数据注释可以共存，因为提供程序将忽略任何不理解的数据注释。 例如，同时适用于 Microsoft SQL Server 和 SQLite 的主键列可能如下所示。

``` csharp
Id = table.Column<int>(nullable: false)
    .Annotation("SqlServer:ValueGenerationStrategy",
        SqlServerValueGenerationStrategy.IdentityColumn)
    .Annotation("Sqlite:Autoincrement", true),
```

如果操作只能应用于一个数据库提供程序上 （或者对于不同的数据库提供程序有非常大的差异），使用`ActiveProvider`属性来区分当前是哪个提供程序。

``` csharp
if (migrationBuilder.ActiveProvider == "Microsoft.EntityFrameworkCore.SqlServer")
{
    migrationBuilder.CreateSequence(
        name: "EntityFrameworkHiLoSequence");
}
```


  [1]: ../../miscellaneous/cli/index.md
  [2]: projects.md
