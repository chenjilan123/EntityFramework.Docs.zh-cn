---
title: Load 方法的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 03c5a069-b7b4-455f-a16f-ee3b96cc4e28
ms.openlocfilehash: bcea8ab2477f44281cd5de824457a72a84ccc766
ms.sourcegitcommit: 4a795285004612ac03ab26532ac09ca333cb4c8f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50123812"
---
# <a name="the-load-method"></a>加载方法
有几种方案，你可能想要实体从数据库加载到上下文中而无需立即执行与这些实体的任何内容。 很好的例子加载实体，可用于数据绑定，如中所述[本地数据](~/ef6/querying/local-data.md)。 一种常用的方式来执行此操作是编写 LINQ 查询，然后对它，只是为了立即放弃创建的列表调用 ToList。 Load 扩展方法工作方式相似 ToList，不同之处在于它完全避免了列表的创建。  

本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

下面是使用负载的两个示例。 第一个执行从 Windows 窗体数据绑定应用程序负载可查询的实体绑定到的本地集合之前，如中所述[本地数据](~/ef6/querying/local-data.md):  

``` csharp
protected override void OnLoad(EventArgs e)
{
    base.OnLoad(e);

    _context = new ProductContext();

    _context.Categories.Load();
    categoryBindingSource.DataSource = _context.Categories.Local.ToBindingList();
}
```  

使用负载加载经过筛选的相关实体集合中所述的第二个示例所示[加载相关实体](~/ef6/querying/related-data.md):  

``` csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.Find(1);

    // Load the posts with the 'entity-framework' tag related to a given blog
    context.Entry(blog)
        .Collection(b => b.Posts)
        .Query()
        .Where(p => p.Tags.Contains("entity-framework"))
        .Load();
}
```  
