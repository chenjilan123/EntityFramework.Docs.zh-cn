---
title: 查询的工作原理 - EF Core
author: rowanmiller
ms.date: 09/26/2018
ms.assetid: de2e34cd-659b-4cab-b5ed-7a979c6bf120
uid: core/querying/overview
ms.openlocfilehash: 23d26f9c0ac17fc0df744f5339946947ea366911
ms.sourcegitcommit: 15022dd06d919c29b1189c82611ea32f9fdc6617
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2018
ms.locfileid: "47415726"
---
# <a name="how-queries-work"></a>查询的工作原理

Entity Framework Core 使用语言集成查询 (LINQ) 来查询数据库中的数据。 通过 LINQ 可使用 C#（或你选择的其他 .NET 语言）基于派生上下文和实体类编写强类型查询。

## <a name="the-life-of-a-query"></a>查询的生命周期

下面是每个查询所经历的过程的高级概述。

1. LINQ 查询由 Entity Framework Core 处理，用于生成已准备好由数据库提供程序处理的表示形式
   1. 结果会被缓存，以便每次执行查询时无需重复进行此处理
2. 结果会传递到数据库提供程序
   1. 数据库提供程序会识别出查询的哪些部分可以在数据库中求值
   2. 查询的这些部分会转换为特定数据库的查询语言（例如，关系数据库的 SQL）
   3. 一个或多个查询会发送到数据库并返回结果集（返回的是数据库中的值，而不是实体实例中的）
3. 对于结果集中的每一项
   1. 如果这是跟踪查询，EF 会检查数据是否表示上下文实例内更改跟踪器中的现有实体
      * 如果是，则会返回现有实体
      * 如果不是，则会创建新实体、设置更改跟踪并返回该新实体
   2. 如果这是非跟踪查询，EF 会检查数据是否表示此查询结果集中的现有实体
      * 如果是，则会返回现有实体 <sup>(1)</sup>
      * 如果不是，则会创建新实体并返回该新实体

<sup>(1)</sup> 非跟踪查询使用弱引用跟踪已返回的实体。 如果具有相同标识的上一个结果超出范围，并遇到垃圾回收机制执行，则可能会获得新的实体实例。

## <a name="when-queries-are-executed"></a>执行查询时

调用 LINQ 运算符时，只会构建查询在内存中的表示形式。 只有在使用结果时，查询才会发送到数据库。

导致查询发送到数据库的最常见操作如下：
* 在 `for` 循环中循环访问结果
* 使用 `ToList`、`ToArray`、`Single`、`Count` 等操作
* 将查询结果数据绑定到 UI

> [!WARNING]  
> 始终验证用户输入：虽然 EF Core通过在查询中使用参数和转义文字来防止 SQL 注入攻击，但它不会验证输入。 根据应用程序的要求，在将 LINQ 查询中使用的来自不受信任的源的值分配给实体属性或传递给其他 EF Core API 之前，应执行相应的验证。 这包括用于动态构造查询的所有用户输入。 即使在使用 LINQ 时，如果接受用于生成表达式的用户输入，也应确保只能构造符合预期的表达式。
