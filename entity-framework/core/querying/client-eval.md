---
title: 客户端与服务器评估 - EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 8b6697cc-7067-4dc2-8007-85d80503d123
uid: core/querying/client-eval
ms.openlocfilehash: 47e22be274d02b5221c638d07151d9607aa7e24f
ms.sourcegitcommit: 0d36e8ff0892b7f034b765b15e041f375f88579a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/09/2018
ms.locfileid: "44250798"
---
# <a name="client-vs-server-evaluation"></a>客户端与服务器评估

Entity Framework Core 支持部分查询在客户端上求值，而将其他部分推送到数据库执行。 由数据库提供程序确定查询的哪些部分会在数据库中求值。

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。

## <a name="client-evaluation"></a>客户端求值

在下面的示例中，一个辅助方法用于标准化从 SQL Server 数据库中返回的博客的 URL。 由于 SQL Server 提供程序不了解此方法的实现方式，因此不可以将其转换为 SQL。 除了在客户端上是通过执行该方法来返回 `URL`，查询的其余部分都是在数据库中执行的。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs?highlight=6)] -->
``` csharp
var blogs = context.Blogs
    .OrderByDescending(blog => blog.Rating)
    .Select(blog => new
    {
        Id = blog.BlogId,
        Url = StandardizeUrl(blog.Url)
    })
    .ToList();
```

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs)] -->
``` csharp
public static string StandardizeUrl(string url)
{
    url = url.ToLower();

    if (!url.StartsWith("http://"))
    {
        url = string.Concat("http://", url);
    }

    return url;
}
```

## <a name="client-evaluation-performance-issues"></a>客户端求值性能问题

虽然客户端求值非常有用，但在某些情况下可能会导致性能不佳。 请考虑以下查询，该查询现在在筛选器中使用辅助方法。 由于无法在数据库中执行此操作，因此所有数据将被拉入内存中，然后会在客户端上应用筛选器。 根据数据量以及筛选出的数据量，这可能会导致性能低下。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs)] -->
``` csharp
var blogs = context.Blogs
    .Where(blog => StandardizeUrl(blog.Url).Contains("dotnet"))
    .ToList();
```

## <a name="client-evaluation-logging"></a>客户端求值日志记录

默认情况下，当执行客户端求值时，EF Core 将记录警告。 有关查看日志记录输出的详细信息，请参阅[日志记录](../miscellaneous/logging.md)。 

## <a name="optional-behavior-throw-an-exception-for-client-evaluation"></a>可选行为：客户端求值引发异常

我们可以将执行客户端求值时记录警告的默认行为改为引发异常或不执行任何操作。 这是在上下文设置选项时进行设置的（通常在 `DbContext.OnConfiguring` 中进行设置，如果使用的是 ASP.NET Core，则在 `Startup.cs` 中进行设置）。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/ThrowOnClientEval/BloggingContext.cs?highlight=5)] -->
``` csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=EFQuerying;Trusted_Connection=True;")
        .ConfigureWarnings(warnings => warnings.Throw(RelationalEventId.QueryClientEvaluationWarning));
}
```
