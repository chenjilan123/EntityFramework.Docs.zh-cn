---
title: 使用实体状态的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: acb27f46-3f3a-4179-874a-d6bea5d7120c
ms.openlocfilehash: ef0e8d5a5a9d66adab7046088c49d8cd472edc8a
ms.sourcegitcommit: e5f9ca4aa41e64141fa63a1e5fcf4d4775d67d24
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/05/2018
ms.locfileid: "52899647"
---
# <a name="working-with-entity-states"></a>使用实体状态
本主题将介绍如何添加和将实体附加到上下文和实体框架如何处理这些在 SaveChanges 期间。
实体框架负责的跟踪的实体时它们已连接到一个上下文，但在断开连接或 N 层方案中，您可以让 EF 知道你的实体的何种状态的状态应为。
本主题所介绍的方法同样适用于查询使用 Code First 和 EF 设计器创建的模型。  

## <a name="entity-states-and-savechanges"></a>实体状态和 SaveChanges

实体可以是一个五种状态中定义的 EntityState 枚举。 这些状态包括：  

- 添加了： 实体正在由上下文跟踪，但在数据库中尚不存在  
- 未更改： 实体正在由上下文跟踪和在数据库中存在并从数据库中的值未更改其属性值  
- 修改： 实体正在由上下文跟踪和在数据库中存在并已修改某些或所有其属性值  
- 已删除： 实体正在由上下文跟踪和在数据库中存在但已标记为删除从数据库下次调用 SaveChanges 时  
- 已分离： 实体未被跟踪的上下文  

SaveChanges 将执行不同操作的实体中的不同状态：  

- 未更改的实体不涉及通过 SaveChanges。 更新会向数据库中未更改状态的实体。  
- 添加实体插入到数据库，然后会保持不变时 SaveChanges 返回。  
- 已修改的实体在数据库中更新，然后会保持不变时 SaveChanges 返回。  
- 已删除的实体从数据库中删除，然后从上下文分离。  

以下示例显示实体或实体关系图状态可以更改的方法。  

## <a name="adding-a-new-entity-to-the-context"></a>将新实体添加到上下文  

新的实体可以通过在 DbSet 上调用 Add 方法添加到上下文。
这将实体的已添加状态，这意味着，它将插入到数据库调用 SaveChanges 的下一个时间。
例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = new Blog { Name = "ADO.NET Blog" };
    context.Blogs.Add(blog);
    context.SaveChanges();
}
```  

若要将新实体添加到上下文的另一种方法是将其状态更改为 Added。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    var blog = new Blog { Name = "ADO.NET Blog" };
    context.Entry(blog).State = EntityState.Added;
    context.SaveChanges();
}
```  

最后，可以挂接到已被跟踪的另一个实体的上下文中添加新实体。
这可能是通过将新实体添加到另一个实体的集合导航属性或通过设置另一个实体以指向新的实体的引用导航属性。 例如：  

``` csharp
using (var context = new BloggingContext())
{
    // Add a new User by setting a reference from a tracked Blog
    var blog = context.Blogs.Find(1);
    blog.Owner = new User { UserName = "johndoe1987" };

    // Add a new Post by adding to the collection of a tracked Blog
    blog.Posts.Add(new Post { Name = "How to Add Entities" });

    context.SaveChanges();
}
```  

请注意，对于所有这些示例，如果要添加的实体具有对不是其他实体的引用尚未然后跟踪这些新实体也将添加到上下文，并将插入到该数据库下次调用 SaveChanges。  

## <a name="attaching-an-existing-entity-to-the-context"></a>将现有实体附加到上下文  

如果你有已经知道的实体存在于数据库，但这当前未跟踪上下文则可指示要跟踪对 DbSet 使用附加方法的实体的上下文。 实体将处于 Unchanged 状态的上下文中。 例如：  

``` csharp
var existingBlog = new Blog { BlogId = 1, Name = "ADO.NET Blog" };

using (var context = new BloggingContext())
{
    context.Blogs.Attach(existingBlog);

    // Do some more work...  

    context.SaveChanges();
}
```  

请注意是否 SaveChanges 调用而无需执行附加的任何的实体其他操作将对数据库进行任何更改。 这是因为实体处于 Unchanged 状态。  

若要将现有实体附加到上下文的另一种方法是将其状态更改为 Unchanged。 例如：  

``` csharp
var existingBlog = new Blog { BlogId = 1, Name = "ADO.NET Blog" };

using (var context = new BloggingContext())
{
    context.Entry(existingBlog).State = EntityState.Unchanged;

    // Do some more work...  

    context.SaveChanges();
}
```  

请注意，为这两个示例如果要附加的实体具有对其他实体未被跟踪的引用然后这些新的实体将还附加到上下文中未更改状态。  

## <a name="attaching-an-existing-but-modified-entity-to-the-context"></a>附加现有的修改后的实体的上下文  

如果你有已经知道的实体存在在数据库中，但到哪个可能进行了更改则可指示要将附加该实体并将其状态设置为已修改的上下文。
例如：  

``` csharp
var existingBlog = new Blog { BlogId = 1, Name = "ADO.NET Blog" };

using (var context = new BloggingContext())
{
    context.Entry(existingBlog).State = EntityState.Modified;

    // Do some more work...  

    context.SaveChanges();
}
```  

时将状态更改为已修改的实体的所有属性将都标记为已修改，并调用 SaveChanges 时的所有属性值将都发送到数据库。  

请注意是否要附加的实体具有对其他实体未被跟踪的引用，这些新的实体将附加到上下文中未更改状态，它们将不被指定为已修改。
如果有多个需要标记为已修改的实体应在单独的每个实体设置状态。  

## <a name="changing-the-state-of-a-tracked-entity"></a>更改跟踪的实体的状态  

你可以通过在其条目上设置的状态属性已被跟踪实体的状态。 例如：  

``` csharp
var existingBlog = new Blog { BlogId = 1, Name = "ADO.NET Blog" };

using (var context = new BloggingContext())
{
    context.Blogs.Attach(existingBlog);
    context.Entry(existingBlog).State = EntityState.Unchanged;

    // Do some more work...  

    context.SaveChanges();
}
```  

请注意，已跟踪的实体调用添加或附加还可用来将实体状态更改。 例如，调用附加的实体，当前处于已添加状态将更改其状态为 Unchanged。  

## <a name="insert-or-update-pattern"></a>插入或更新模式  

某些应用程序的常见模式是将实体添加为新的 （导致数据库插入） 或附加与现有实体并将其标记为修改 （导致数据库更新） 根据为主键的值。
例如，使用数据库生成整数主键时，通常会将具有零个密钥作为新的实体和具有非零值密钥作为现有的实体。
此模式可以通过设置基于主键值的检查的实体状态来实现。 例如：  

``` csharp
public void InsertOrUpdate(Blog blog)
{
    using (var context = new BloggingContext())
    {
        context.Entry(blog).State = blog.BlogId == 0 ?
                                   EntityState.Added :
                                   EntityState.Modified;

        context.SaveChanges();
    }
}
```  

请注意时将状态更改为已修改的实体的所有属性将都标记为已修改的所有属性值将在调用 SaveChanges 时被都发送到数据库。  
