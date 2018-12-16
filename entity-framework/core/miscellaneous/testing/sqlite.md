---
title: 使用 SQLite 的 EF Core测试
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 7a2b75e2-1875-4487-9877-feff0651b5a6
uid: core/miscellaneous/testing/sqlite
ms.openlocfilehash: bc9d6768a90ce17160c4126d2a68fddaa30d63de
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42996863"
---
# <a name="testing-with-sqlite"></a>使用 SQLite 进行测试

SQLite 具有内存中模式，借此可使用 SQLite 针对关系数据库编写测试，而不会产生实际数据库操作的开销。

> [!TIP]  
> 您可以查看此文章[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Testing)GitHub 上

## <a name="example-testing-scenario"></a>示例测试方案

请考虑以下允许应用程序代码执行一些与博客相关的操作的服务。 该服务在内部使用连接到 SQL Server 数据库的 `DbContext`。 交换此上下文可有效连接到内存中 SQLite 数据库，这样我们就可以为该服务编写高效的测试，而无需修改代码或执行大量的工作来创建上下文的测试副本。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/BusinessLogic/BlogService.cs)]

## <a name="get-your-context-ready"></a>准备您的上下文

### <a name="avoid-configuring-two-database-providers"></a>避免配置两个数据库提供程序

在测试中，从外部配置上下文以使用 InMemory 提供程序。 如果要通过在上下文中替代 `OnConfiguring` 来配置数据库提供程序，则需要添加一些条件代码，从而确保只有在尚未配置数据库提供程序的情况下才进行配置。

> [!TIP]  
> 如果使用的是 ASP.NET Core，则不需要此代码，因为数据库提供程序是在上下文之外（在 Startup.cs 中）配置的。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/BusinessLogic/BloggingContext.cs#OnConfiguring)]

### <a name="add-a-constructor-for-testing"></a>添加用于测试的构造函数

若要启用针对不同的数据库的测试的最简单方法是修改上下文，以公开一个构造函数接受`DbContextOptions<TContext>`。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/BusinessLogic/BloggingContext.cs#Constructors)]

> [!TIP]  
> `DbContextOptions<TContext>` 告知上下文其所有的设置，例如要连接的数据库。 这与在上下文中运行 OnConfiguring 方法所生成的对象相同。

## <a name="writing-tests"></a>编写测试

使用此提供程序进行测试的关键是，可告知上下文使用 SQLite 并控制内存中数据库的范围。 通过打开和关闭连接来控制数据库的范围。 数据库的范围限定为连接打开的持续时间。 通常，每个测试方法都需要一个干净的数据库。

[!code-csharp[Main](../../../../samples/core/Miscellaneous/Testing/TestProject/SQLite/BlogServiceTests.cs)]
