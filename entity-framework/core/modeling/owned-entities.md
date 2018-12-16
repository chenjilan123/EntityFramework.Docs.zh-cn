---
title: 拥有 EF Core 的实体类型-
author: AndriySvyryd
ms.author: ansvyryd
ms.date: 02/26/2018
ms.assetid: 2B0BADCE-E23E-4B28-B8EE-537883E16DF3
uid: core/modeling/owned-entities
ms.openlocfilehash: 58da3b6b951b3fa4aa04ec75f5759555c1f0cde5
ms.sourcegitcommit: 39080d38e1adea90db741257e60dc0e7ed08aa82
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/03/2018
ms.locfileid: "50980023"
---
# <a name="owned-entity-types"></a>固有的实体类型

>[!NOTE]
> 此功能是在 EF Core 2.0 中的新增功能。

EF Core 让你可以只显示对其他实体类型的导航属性的模型实体类型。 这些被称为_固有实体类型_。 包含固有的实体类型的实体是其_所有者_。

## <a name="explicit-configuration"></a>显式配置

拥有永远不会包含类型由 EF Core 模型中按照约定的实体。 你可以使用`OwnsOne`中的方法`OnModelCreating`或批注与类型`OwnedAttribute`（新在 EF Core 2.1） 配置为拥有类型的类型。

在此示例中，`StreetAddress`是没有标识属性的类型。 它用作 Order 类型的属性来指定特定订单的发货地址。

我们可以使用`OwnedAttribute`会将其视为拥有实体时从另一个实体类型引用：

[!code-csharp[StreetAddress](../../../samples/core/Modeling/OwnedEntities/StreetAddress.cs?name=StreetAddress)]

[!code-csharp[Order](../../../samples/core/Modeling/OwnedEntities/Order.cs?name=Order)]

还有可能要使用`OwnsOne`中的方法`OnModelCreating`以指定`ShippingAddress`属性是所拥有的实体`Order`实体类型，并根据需要配置其他方面。

[!code-csharp[OwnsOne](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOne)]

如果`ShippingAddress`属性为私有`Order`类型，则可以使用的字符串版本`OwnsOne`方法：

[!code-csharp[OwnsOneString](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneString)]

请参阅[完整的示例项目](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Modeling/OwnedEntities)有关更多上下文。 

## <a name="implicit-keys"></a>隐式键

固有类型配置了`OwnsOne`或通过引用导航发现始终具有一对一的关系的所有者，因此他们不需要其自己的密钥值，如外键的值是唯一的。 在上一示例中，`StreetAddress`类型不需要定义键属性。  

若要了解如何 EF Core 可跟踪这些对象，最好先认为，作为创建主键[阴影属性](xref:core/modeling/shadow-properties)对于固有类型。 固有类型的实例键的值将为所有者实例的键的值相同。

## <a name="collections-of-owned-types"></a>固有类型的集合

>[!NOTE]
> 此功能是 EF Core 2.2 中的新增功能。

若要配置的固有类型的集合`OwnsMany`应在`OnModelCreating`。 但是主要密钥将不会配置按照约定，因此它需要显式指定。 它是通常会为这些类型的合并的所有者和其他唯一属性，也可以在卷影状态中的外键的实体使用复杂的密钥：

[!code-csharp[OwnsMany](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsMany)]

## <a name="mapping-owned-types-with-table-splitting"></a>映射固有类型与表拆分

使用关系时数据库，按约定引用所有类型映射到与所有者相同的表。 这要求将在两个表拆分： 某些列将用于存储数据的所有者，并且某些列将用于存储数据的固有实体。 这是一项称为表拆分的常见功能。

> [!TIP]
> 拥有表拆分与存储在一起的类型可以是使用类似于复杂类型的用法在 EF6 中。

按照约定，EF Core 将遵循模式的固有的实体类型的属性命名的数据库列_Navigation_OwnedEntityProperty_。 因此`StreetAddress`属性将显示在具有名称 ShippingAddress_Street 和 ShippingAddress_City 的订单表。

您可以将附加`HasColumnName`方法来重命名这些列：

[!code-csharp[ColumnNames](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=ColumnNames)]

## <a name="sharing-the-same-net-type-among-multiple-owned-types"></a>共享多个固有类型之间的相同.NET 类型

固有的实体类型可以是相同的.NET 类型，作为另一个固有的实体类型，因此.NET 类型可能都不足以识别固有的类型。

在这些情况下，所有者从指向固有实体的属性将成为_定义导航_的固有的实体类型。 从 EF Core 的角度来看，定义导航是标识的旁边的.NET 类型的类型的一部分。   

例如，在下面的类`ShippingAddress`并`BillingAddress`均为相同的.NET 类型， `StreetAddress`:

[!code-csharp[OrderDetails](../../../samples/core/Modeling/OwnedEntities/OrderDetails.cs?name=OrderDetails)]

若要了解如何 EF Core 将区分跟踪这些对象的实例，可能会有用考虑定义导航已旁边的值的键的所有者的实例键的一部分，拥有类型的.NET 类型。

## <a name="nested-owned-types"></a>嵌套固有的类型

在此示例中`OrderDetails`拥有`BillingAddress`并`ShippingAddress`，它们均`StreetAddress`类型。 然后 `OrderDetails` 归 `DetailedOrder` 类型所有。

[!code-csharp[DetailedOrder](../../../samples/core/Modeling/OwnedEntities/DetailedOrder.cs?name=DetailedOrder)]

[!code-csharp[OrderStatus](../../../samples/core/Modeling/OwnedEntities/OrderStatus.cs?name=OrderStatus)]

嵌套固有类型，除了固有的类型可以引用常规实体，只要固有的实体的依赖端上，它可以是所有者或不同的实体。 此功能在 EF6 中设置除了复杂类型的固有的实体类型。

[!code-csharp[OrderDetails](../../../samples/core/Modeling/OwnedEntities/OrderDetails.cs?name=OrderDetails)]

它是链接到`OwnsOne`中配置此模型的 fluent 调用的方法：

[!code-csharp[OwnsOneNested](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneNested)]

还有可能使用相同的操作实现目的`OwnedAttribute`上都`OrderDetails`和`StreetAdress`。

## <a name="storing-owned-types-in-separate-tables"></a>存储拥有单独的表中的类型

此外固有的类型不同于 EF6 的复杂类型，它可以存储在单独的表所有者。 若要重写将固有的类型映射到与所有者相同的表的约定，您可以简单地调用`ToTable`并提供一个不同的表名称。 下面的示例将映射`OrderDetails`到从单独的表及其两个址`DetailedOrder`:

[!code-csharp[OwnsOneTable](../../../samples/core/Modeling/OwnedEntities/OwnedEntityContext.cs?name=OwnsOneTable)]

## <a name="querying-owned-types"></a>查询固有的类型

查询所有者时，固有类型将默认包含在内。 不需要使用`Include`方法，即使固有的类型存储在一个单独的表中。 根据之前所述的模型，以下查询将获取`Order`，`OrderDetails`和两个拥有`StreetAddresses`从数据库：

[!code-csharp[DetailedOrderQuery](../../../samples/core/Modeling/OwnedEntities/Program.cs?name=DetailedOrderQuery)]

## <a name="limitations"></a>限制

其中某些限制是基础如何固有实体类型的工作，但另一些限制，我们可能无法将其删除在将来的版本：

### <a name="by-design-restrictions"></a>按设计限制
- 无法创建`DbSet<T>`对于固有类型
- 不能调用`Entity<T>()`固有类型上使用 `ModelBuilder`

### <a name="current-shortcomings"></a>当前的不足之处
- 包含的继承层次结构拥有不支持实体类型
- 引用导航到所拥有的实体类型不能为 null，除非它们显式映射到单独的表中的所有者
- 固有的实体类型的实例无法共享的多个所有者 （这是一个已知的值对象不能使用固有的实体类型实现的方案）

### <a name="shortcomings-in-previous-versions"></a>在早期版本中的不足之处
- 在 EF Core 2.0 中，导航到拥有不能在派生的实体类型中声明实体类型，除非自有的实体被显式映射到一个单独的表所有者层次结构中。 EF Core 2.1 中已删除此限制
- En EF Core 2.0 和 2.1 仅引用导航，固有类型到受支持。 在 EF Core 2.2 中消除这一限制