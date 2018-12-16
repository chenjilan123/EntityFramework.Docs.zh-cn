---
title: 跟踪与非跟踪查询 - EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: e17e060c-929f-4180-8883-40c438fbcc01
uid: core/querying/tracking
ms.openlocfilehash: 985adc795f379199a3bacc985843f32f3168cf64
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993350"
---
# <a name="tracking-vs-no-tracking-queries"></a>跟踪与非跟踪查询

跟踪行为决定了 Entity Framework Core 是否将有关实体实例的信息保留在其更改跟踪器中。 如果已跟踪某个实体，则该实体中检测到的任何更改都会在 `SaveChanges()` 期间永久保存到数据库。 Entity Framework Core 还会修正从跟踪查询中获取的实体与先前已加载到 DbContext 实例中的实体两者之间的导航属性。

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。

## <a name="tracking-queries"></a>跟踪查询

返回实体类型的查询是默认会被跟踪的。 这表示可以更改这些实体实例，然后通过 `SaveChanges()` 持久化这些更改。

在以下示例中，将检测到对博客评分所做的更改，并在 `SaveChanges()` 期间将这些更改持久化到数据库中。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs)] -->
``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.SingleOrDefault(b => b.BlogId == 1);
    blog.Rating = 5;
    context.SaveChanges();
}
```

## <a name="no-tracking-queries"></a>非跟踪查询

在只读方案中使用结果时，非跟踪查询十分有用。 可以更快速地执行非跟踪查询，因为无需设置更改跟踪信息。

可以将单个查询替换为非跟踪查询：

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs?highlight=4)] -->
``` csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .AsNoTracking()
        .ToList();
}
```

还可以在上下文实例级别更改默认跟踪行为：

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs?highlight=3)] -->
``` csharp
using (var context = new BloggingContext())
{
    context.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;

    var blogs = context.Blogs.ToList();
}
```

> [!NOTE]  
> 非跟踪查询仍在查询过程中执行标识解析。 如果结果集多次包含相同的实体，则每次会在结果集中返回实体类的相同实例。 但是，弱引用用于跟踪已返回的实体。 如果具有相同标识的上一个结果超出范围，并遇到垃圾回收，则可能会获得新的实体实例。 有关详细信息，请参阅[查询的工作原理](overview.md)。

## <a name="tracking-and-projections"></a>跟踪和投影

即使查询的结果类型不是实体类型，但如果结果中包含实体类型，则默认情况下也会跟踪这些实体类型。 在以下返回匿名类型的查询中，结果集中的 `Blog` 实例会被跟踪。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs?highlight=7)] -->
``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Select(b =>
            new
            {
                Blog = b,
                Posts = b.Posts.Count()
            });
}
```

如果结果集不包含任何实体类型，则不会执行跟踪。 在以下返回匿名类型（具有实体中的某些值，但没有实际实体类型的实例）的查询中，不会执行跟踪。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs)] -->
``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Select(b =>
            new
            {
                Id = b.BlogId,
                Url = b.Url
            });
}
```
