---
title: 异步查询 - EF Core
author: rowanmiller
ms.date: 01/24/2017
ms.assetid: b6429b14-cba0-4af4-878f-b829777c89cb
uid: core/querying/async
ms.openlocfilehash: de00e25279e29355a4eb3e55597a8578ceccecb6
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993560"
---
# <a name="asynchronous-queries"></a>异步查询

当在数据库中执行查询时，异步查询可避免阻止线程。 这有助于避免冻结富客户端应用程序的 UI。 异步操作还可以增加 Web 应用程序的吞吐量，可以在数据库操作完成时释放线程去处理其他请求。 有关详细信息，请参阅[使用 C# 异步编程](https://docs.microsoft.com/dotnet/csharp/async)。

> [!WARNING]  
> EF Core 不支持在同一上下文实例上运行多个并行操作。 应始终等待操作完成，然后再开始下一个操作。 这通常是通过在每个异步操作上使用 `await` 关键字完成的。

Entity Framework Core 提供了一组异步扩展方法，可用作执行查询并返回结果的 LINQ 方法的替代方法。 示例包括 `ToListAsync()`、`ToArrayAsync()`、`SingleAsync()` 等。对于部分 LINQ 运算符（如 `Where(...)`、`OrderBy(...)` 等），没有对应的异步版本，因为这些方法仅用于构建 LINQ 表达式树，而未将查询发送到数据库中执行。

> [!IMPORTANT]  
> EF Core 异步扩展方法在 `Microsoft.EntityFrameworkCore` 命名空间中定义 。 必须导入此命名空间才能使这些方法可用。

[!code-csharp[Main](../../../samples/core/Querying/Querying/Async/Sample.cs#Sample)]
