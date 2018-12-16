---
title: 异步保存 - EF Core
author: rowanmiller
ms.date: 01/24/2017
ms.assetid: b64a606e-ecd9-4807-829a-b6ec05ade33f
uid: core/saving/async
ms.openlocfilehash: 6f482a77300ff2930953686751a579b022bf6f77
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42997277"
---
# <a name="asynchronous-saving"></a>异步保存

当所做的更改被写入数据库时，异步保存可避免阻塞线程。 这有助于避免胖客户端应用程序的 UI 被冻结。 异步操作还可以增加 Web 应用程序的吞吐量，可以释放线程以在数据库操作完成前去处理其他请求。 有关详细信息，请参阅[使用 C# 异步编程](https://docs.microsoft.com/dotnet/csharp/async)。

> [!WARNING]  
> EF Core 不支持在同一上下文实例上运行多个并行操作。 应始终等待操作完成，然后再开始下一个操作。 这通常是通过在每个异步操作上使用 `await` 关键字完成的。

Entity Framework Core 提供了 `DbContext.SaveChangesAsync()` 作为 `DbContext.SaveChanges()` 的异步替代方法。

[!code-csharp[Main](../../../samples/core/Saving/Saving/Async/Sample.cs#Sample)]
