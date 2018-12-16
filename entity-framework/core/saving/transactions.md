---
title: 事务 - EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: d3e6515b-8181-482c-a790-c4a6778748c1
uid: core/saving/transactions
ms.openlocfilehash: 7083a1228420416a1b60d9744ca2dad2339be53f
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993600"
---
# <a name="using-transactions"></a>使用事务

事务允许以原子方式处理多个数据库操作。 如果已提交事务，则所有操作都会成功应用到数据库。 如果已回滚事务，则所有操作都不会应用到数据库。

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Transactions/)。

## <a name="default-transaction-behavior"></a>默认事务行为

默认情况下，如果数据库提供程序支持事务，则会在单次调用 `SaveChanges()` 时将所有更改都将应用到事务中。 如果其中有任何更改失败，则会回滚事务且所有更改都不会应用到数据库。 这意味着，`SaveChanges()` 可保证要么完全成功，要么在出现错误时不修改数据库。

对于大多数应用程序，此默认行为已足够。 除非应用程序确有需求，否则不应手动控制事务。

## <a name="controlling-transactions"></a>控制事务

可以使用 `DbContext.Database` API 开始、提交和回滚事务。 以下示例显示了在单个事务中执行的两个 `SaveChanges()` 操作以及 一个LINQ 查询。

并非所有数据库提供程序都支持事务。 调用事务 API 时，某些提供程序可能会引发异常或不执行任何操作。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Transactions/ControllingTransaction/Sample.cs?name=Transaction&highlight=3,17,18,19)]

## <a name="cross-context-transaction-relational-databases-only"></a>跨上下文事务（仅限关系数据库）

您还可以跨多个上下文实例共享一个事务。 此功能仅在使用关系数据库提供程序时才可用，因为它需要使用特定于关系数据库的 `DbTransaction` 和 `DbConnection`。

若要共享事务，上下文必须共享 `DbConnection` 和 `DbTransaction`。

### <a name="allow-connection-to-be-externally-provided"></a>允许在外部提供连接

共享 `DbConnection` 需要在构造上下文时向其中传入连接的功能。

允许在外部提供 `DbConnection` 的最简单方式是，停止使用 `DbContext.OnConfiguring` 方法来配置上下文并在外部创建 `DbContextOptions`，然后将其传递到上下文构造函数。

> [!TIP]  
> `DbContextOptionsBuilder` 是在 `DbContext.OnConfiguring` 中用于配置上下文的 API，现在即将在外部使用它来创建 `DbContextOptions`。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Transactions/SharingTransaction/Sample.cs?name=Context&highlight=3,4,5)]

替代方法是继续使用 `DbContext.OnConfiguring`，但接受已保存并随后在 `DbContext.OnConfiguring` 中使用的 `DbConnection`。

``` csharp
public class BloggingContext : DbContext
{
    private DbConnection _connection;

    public BloggingContext(DbConnection connection)
    {
      _connection = connection;
    }

    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(_connection);
    }
}
```

### <a name="share-connection-and-transaction"></a>共享连接和事务

现在可以创建共享同一连接的多个上下文实例。 然后使用 `DbContext.Database.UseTransaction(DbTransaction)` API 在同一事务中登记两个上下文。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Transactions/SharingTransaction/Sample.cs?name=Transaction&highlight=1,2,3,7,16,23,24,25)]

## <a name="using-external-dbtransactions-relational-databases-only"></a>使用外部 DbTransactions（仅限关系数据库）

如果使用多个数据访问技术来访问关系数据库，则可能希望在这些不同技术所执行的操作之间共享事务。

以下示例显示了如何在同一事务中执行 ADO.NET SqlClient 操作和 Entity Framework Core 操作。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Transactions/ExternalDbTransaction/Sample.cs?name=Transaction&highlight=4,10,21,26,27,28)]

## <a name="using-systemtransactions"></a>使用 System.Transactions

> [!NOTE]  
> 此功能是 EF Core 2.1 中的新增功能。

如果需要跨较大作用域进行协调，则可以使用环境事务。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Transactions/AmbientTransaction/Sample.cs?name=Transaction&highlight=1,2,3,26,27,28)]

还可以在显式事务中登记。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Transactions/CommitableTransaction/Sample.cs?name=Transaction&highlight=1,15,28,29,30)]

### <a name="limitations-of-systemtransactions"></a>System.Transactions 的限制  

1. EF Core 依赖数据库提供程序以实现对 System.Transactions 的支持。 虽然支持在 .NET Framework 的 ADO.NET 提供程序之间十分常见，但最近才将 API 添加到 .NET Core，因此支持并未得到广泛应用。 如果提供程序未实现对 System.Transactions 的支持，则可能会完全忽略对这些 API 的调用。 SqlClient for .NET Core 从 2.1 及以上版本开始支持 System.Transactions。 尝试使用该功能时，SqlClient for .NET Core 2.0 将引发异常。 

   > [!IMPORTANT]  
   > 建议你测试在依赖提供程序以管理事务之前 API 与该提供程序的行为是否正确。 如果不正确，则建议你与数据库提供程序的维护人员联系。 

2. 自版本 2.1 起，.NET Core 中的 System.Transactions 实现不包括对分布式事务的支持，因此不能使用 `TransactionScope` 或 `CommittableTransaction` 来跨多个资源管理器协调事务。 
