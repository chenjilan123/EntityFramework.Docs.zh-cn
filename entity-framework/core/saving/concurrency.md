---
title: 处理并发冲突 - EF Core
author: rowanmiller
ms.date: 03/03/2018
uid: core/saving/concurrency
ms.openlocfilehash: e050b17bfa31a4785161c700bc0355e83162b405
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993107"
---
# <a name="handling-concurrency-conflicts"></a>处理并发冲突

> [!NOTE]
> 此页面记录了并发在 EF Core 中的工作原理以及如何处理应用程序中的并发冲突。 有关如何配置模型中的并发令牌的详细信息，请参阅[并发令牌](xref:core/modeling/concurrency)。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Concurrency/)。

_数据库并发_指多个进程或用户同时访问或更改数据库中的相同数据的情况。 _并发控制_指的是用于在发生并发更改时确保数据一致性的特定机制。

EF Core 实现_了乐观并发控制_，这意味着它将允许多个进程或用户独立进行更改，而不会产生同步或锁定的开销。 在理想情况下，这些更改将不会相互干扰，因此都能够成功。 在最坏的情况下，两个或更多进程将尝试进行冲突更改，而其中只有一个进程会成功。

## <a name="how-concurrency-control-works-in-ef-core"></a>并发控制在 EF Core 中的工作原理

配置为并发令牌的属性用于实现乐观并发控制：每当在 `SaveChanges` 期间执行更新或删除操作时，会将数据库上的并发令牌值与通过 EF Core 读取的原始值进行比较。

- 如果这些值匹配，则可以完成该操作。
- 如果这些值不匹配，EF Core 会假设另一个用户已执行冲突操作，并中止当前事务。

另一个用户已执行与当前操作冲突的操作的情况被称为_并发冲突_。

数据库提供程序负责实现并发令牌值的比较。

在关系数据库上，EF Core 会对任何 `UPDATE` 或 `DELETE` 语句的 `WHERE` 子句中的并发令牌值进行检查。 执行这些语句后，EF Core 会读取受影响的行数。

如果未影响任何行，将检测到并发冲突，并且 EF Core 会引发 `DbUpdateConcurrencyException`。

例如，我们可能希望将 `Person` 的 `LastName` 配置为并发令牌。 这样，对 Person 的任何更新操作都将在 `WHERE` 子句中包括并发检查：

``` sql
UPDATE [Person] SET [FirstName] = @p1
WHERE [PersonId] = @p0 AND [LastName] = @p2;
```

## <a name="resolving-concurrency-conflicts"></a>解决并发冲突

继续前面的示例，如果一个用户尝试保存对 `Person` 所做的某些更改，但另一个用户已更改 `LastName`，则将引发异常。

此时，应用程序可能只需通知用户由于发生冲突更改而导致更新未成功，然后继续操作。 但可能需要提示用户确保此记录仍表示同一实际用户并重试该操作。

此过程是_解决并发冲突_的一个示例。

解决并发冲突涉及将当前 `DbContext` 中挂起的更改与数据库中的值进行合并。 要合并的值因应用程序而异并可由用户输入指示。

**有三组值可用于帮助解决并发冲突：**

* “当前值”是应用程序尝试写入数据库的值。

* “原始值”是在进行任何编辑之前最初从数据库中检索的值。

* “数据库值”是当前存储在数据库中的值。

处理并发冲突的常规方法是：

1. 在 `SaveChanges` 期间捕获 `DbUpdateConcurrencyException`。
2. 使用 `DbUpdateConcurrencyException.Entries` 为受影响的实体准备一组新更改。
3. 刷新并发令牌的原始值以反映数据库中的当前值。
4. 重试该过程，直到不发生任何冲突。

在下面的示例中，将 `Person.FirstName` 和 `Person.LastName` 设置为并发令牌。 在包括应用程序特定逻辑以选择要保存的值的位置处有一条 `// TODO:` 注释。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Concurrency/Sample.cs?name=ConcurrencyHandlingCode&highlight=34-35)]
