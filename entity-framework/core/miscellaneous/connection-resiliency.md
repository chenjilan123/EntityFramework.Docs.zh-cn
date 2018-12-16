---
title: 连接复原的 EF Core
author: rowanmiller
ms.date: 11/15/2016
ms.assetid: e079d4af-c455-4a14-8e15-a8471516d748
uid: core/miscellaneous/connection-resiliency
ms.openlocfilehash: 729cf9b8c038ea2adba8c79c68d9f6fb1676fefa
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022179"
---
# <a name="connection-resiliency"></a>连接复原

连接复原将自动重试已失败的数据库命令。 通过提供“执行策略”，它封装检测故障，然后重试命令所需的逻辑，该功能可以应用于任何数据库。 EF Core 提供程序针对特定数据库的失败条件提供定制的执行策略，同时提供最佳的重试策略。

例如，SQL Server 提供程序包括专门针对 SQL Server （包括 SQL Azure） 的执行策略。 它知道可以重试的异常类型，并且具有合理的默认值的最大重试，重试次数等之间的延迟。

为上下文配置选项时将指定执行策略。 这通常位于派生上下文的 `OnConfiguring` 方法中，或位于 ASP.NET Core 应用程序的 `Startup.cs` 中。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#OnConfiguring)]

## <a name="custom-execution-strategy"></a>自定义执行策略

没有一种机制来注册您如果想要更改任何默认设置自己的自定义执行策略。

``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseMyProvider(
            "<connection string>",
            options => options.ExecutionStrategy(...));
}
```

## <a name="execution-strategies-and-transactions"></a>执行策略和事务

在出现故障时自动重试的执行策略需要能够回滚失败的重试块中的每个操作。 启用重试后，通过 EF Core 执行的每个操作都将成为其自身的可重试操作。 也就是说，如果出现暂时性故障，每个查询和对 `SaveChanges()` 的每次调用都将作为一个单元进行重试。

但是，如果你的代码使用 `BeginTransaction()` 启动事务，则你将定义自己的操作组（这些操作需要被视为一个单元），并且如果发生故障，将需要回滚事务内的所有内容。 如果尝试在使用执行策略时执行此操作，将收到如下所示的异常：

> InvalidOperationException： 配置的执行策略 SqlServerRetryingExecutionStrategy 不支持用户启动的事务。 使用由“DbContext.Database.CreateExecutionStrategy()”返回的执行策略执行事务（作为一个可回溯单元）中的所有操作。

解决方法是使用代表需要执行的所有内容的委托来手动调用执行策略。 如果发生暂时性故障，执行策略将再次调用委托。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#ManualTransaction)]

这种方法还可用于环境事务。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#AmbientTransaction)]

## <a name="transaction-commit-failure-and-the-idempotency-issue"></a>事务提交失败和幂等性问题

一般情况下，连接失败时当前事务会回滚。 但是，如果在连接断开时在事务正在将提交所生成的事务状态为未知。 请参阅此[博客文章](https://blogs.msdn.com/b/adonet/archive/2013/03/11/sql-database-connectivity-and-the-idempotency-issue.aspx)的更多详细信息。

默认情况下，执行策略将重试该操作，就像该事务已回滚一样。但如果未这样做，当新数据库状态不兼容时，则可能导致异常；或者当操作不依赖于特定状态时（例如使用自动生成的键值插入新行时），则可能导致数据损坏。

有几种方法来解决此问题。

### <a name="option-1---do-almost-nothing"></a>选项 1-执行操作 （几乎） 执行任何操作

事务提交期间连接失败的可能性很低，因此如果实际发生此情况，应用程序可能会失败。

但是，需要避免使用存储生成的密钥，以确保引发异常而不是添加重复的行。 请考虑使用客户端生成的 GUID 值或客户端值生成器。

### <a name="option-2---rebuild-application-state"></a>选项 2-重新生成应用程序状态

1. 放弃当前`DbContext`。
2. 创建一个新`DbContext`和从数据库中还原应用程序的状态。
3. 通知用户上次操作可能尚未成功完成。

### <a name="option-3---add-state-verification"></a>选项 3-添加状态验证

对于大多数更改数据库状态的操作就可以添加代码来检查它是否成功。 EF 提供了一个扩展方法来简化此过程- `IExecutionStrategy.ExecuteInTransaction`。

此方法将开始并提交事务，还将接受在事务提交期间出现暂时性错误时所调用的 `verifySucceeded`参数中的函数。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#Verification)]

> [!NOTE]
> 这里`SaveChanges`调用使用了`acceptAllChangesOnSuccess`设置为`false`为了避免更改的状态`Blog`到实体`Unchanged`如果`SaveChanges`成功。 这样就可以重试相同的操作，如果提交失败并回滚事务。

### <a name="option-4---manually-track-the-transaction"></a>选项 4-手动跟踪事务

如果需要使用存储生成的密钥或需要一种处理提交失败且不依赖于所执行操作的通用方法，则可以为每个事务分配一个可在提交失败时进行检查的 ID。

1. 将表添加到用于跟踪事务的状态的数据库。
2. 插入到表中每个事务的开始处的行。
3. 如果连接失败在提交期间，检查存在的数据库中的相应行。
4. 如果提交成功，删除对应的行，以避免对表的增长。

[!code-csharp[Main](../../../samples/core/Miscellaneous/ConnectionResiliency/Program.cs#Tracking)]

> [!NOTE]
> 请确保用于验证的上下文具有定义的执行策略，因为如果在事务提交期间连接失败，则连接可能在验证期间再次失败。
