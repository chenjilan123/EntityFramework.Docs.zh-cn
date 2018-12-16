---
title: 查询类型的 EF Core
author: anpete
ms.date: 02/26/2018
ms.assetid: 9F4450C5-1A3F-4BB6-AC19-9FAC64292AAD
uid: core/modeling/query-types
ms.openlocfilehash: cb391343e6f24092ae0874003c0ef2935dd4e03f
ms.sourcegitcommit: 8dd71a57a01c439431164c163a0722877d0e5cd8
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/07/2018
ms.locfileid: "53028175"
---
# <a name="query-types"></a>查询类型
> [!NOTE]
> 此功能是 EF Core 2.1 中的新增功能

EF Core 模型可以包含实体类型，除了_查询类型_，这可以用于执行针对未映射到实体类型的数据的数据库查询。

## <a name="compare-query-types-to-entity-types"></a>比较查询类型与实体类型

查询类型它们与实体类型中的一样：

- 可以添加到模型或者在`OnModelCreating`或通过"设置"属性上派生_DbContext_。
- 支持许多相同的映射功能，如继承映射和导航属性。 上关系存储，他们可以配置的目标数据库对象和列通过 fluent API 方法或数据注释。

但是，它们是不同于实体中的类型：

- 不需要定义的键。
- 永远不会跟踪的更改上_DbContext_并因此永远不会插入、 更新或删除数据库。
- 永远不会由约定发现。
- 仅支持一部分导航映射功能-具体而言：
  - 它们可能永远不会作为关系的主体端。
  - 它们仅可包含指向的实体的引用导航属性。
  - 实体不能包含查询类型的导航属性。
- 得到解决_ModelBuilder_使用`Query`方法而不是`Entity`方法。
- 在映射_DbContext_类型的属性通过`DbQuery<T>`而非 `DbSet<T>`
- 映射到使用的数据库对象`ToView`方法，而非`ToTable`。
- 可以映射到_定义查询_-定义查询是可查询类型的数据源在模型中声明的第二次查询。

## <a name="usage-scenarios"></a>使用方案

下面是一些查询类型主要使用方案：

- 作为返回类型的即席`FromSql()`查询。
- 映射到数据库视图。
- 映射到不具有定义的主键的表。
- 映射到模型中定义的查询。

## <a name="mapping-to-database-objects"></a>映射到数据库对象

查询类型映射到的数据库对象通过实现`ToView`fluent API。 此方法中指定的数据库对象是从 EF Core 的角度来看，_视图_，这意味着它将被视为只读查询源和不能作为目标的更新、 插入或删除操作。 但是，这并不意味着该数据库对象是实际需要的数据库视图-它或者可以是以只读方式将被视为一个数据库表。 相反，实体类型，EF Core 假定数据库对象中指定`ToTable`方法可以视为_表_、 意味着它可以用作查询源，但是还针对由更新、 删除和插入操作。 事实上，可以指定数据库视图中的名称`ToTable`和所有内容应该能够正常工作，只要视图被配置为可在数据库上更新。

## <a name="example"></a>示例

下面的示例演示如何使用查询类型来查询数据库视图。

> [!TIP]
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/QueryTypes)。

首先，我们定义一个简单的博客和文章模型：

[!code-csharp[Main](../../../samples/core/QueryTypes/Program.cs#Entities)]

接下来，我们定义一个简单的数据库视图，这样就可以查询与每个博客帖子数：

[!code-csharp[Main](../../../samples/core/QueryTypes/Program.cs#View)]

接下来，我们定义一个类来保存数据库视图的结果：

[!code-csharp[Main](../../../samples/core/QueryTypes/Program.cs#QueryType)]

接下来，我们配置中的查询类型_OnModelCreating_使用`modelBuilder.Query<T>`API。
我们使用标准的 fluent 配置 Api 来配置查询类型的映射：

[!code-csharp[Main](../../../samples/core/QueryTypes/Program.cs#Configuration)]

最后，我们可以采用标准方式来查询数据库视图：

[!code-csharp[Main](../../../samples/core/QueryTypes/Program.cs#Query)]

> [!TIP]
> 请注意我们还定义了上下文级别查询属性 (DbQuery) 作为针对此类型的查询的根。
