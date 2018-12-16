---
title: EF4、 EF5 和 EF6 的性能注意事项
author: divega
ms.date: 10/23/2016
ms.assetid: d6d5a465-6434-45fa-855d-5eb48c61a2ea
ms.openlocfilehash: c87c1412cb23abf232663d7e4f44eef5f7818ea2
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022384"
---
# <a name="performance-considerations-for-ef-4-5-and-6"></a>有关 EF 4、 5 和 6 的性能注意事项
由 David Obando、 Eric Dettinger 等

发布时间： 2012 年 4 月

上次更新时间： 2014 年 5 月

------------------------------------------------------------------------

## <a name="1-introduction"></a>1.介绍

对象关系映射框架都提供面向对象的应用程序中的数据访问的抽象的简便方法。 对于.NET 应用程序，Microsoft 建议的 O/RM 是实体框架。 不过，使用任何抽象层性能可能会是问题。

此白皮书旨在开发应用程序使用 Entity Framework，开发人员以了解可能会影响性能，实体框架内部算法并提供提示以进行调查时显示的性能注意事项和提高性能，使用实体框架的应用程序中。 有多个对性能的良好主题已在 web 上，并且我们也尝试了指向这些资源，在可能的情况。

性能是一个棘手的主题。 此白皮书旨在作为资源以帮助使性能相关的使用实体框架的应用程序的决策。 我们包含了一些测试指标来演示性能，但这些指标不应为绝对指示器，您将看到在应用程序中的性能。

实用的角度而言，本文档假定 Entity Framework 4 运行在.NET 4.0 和 Entity Framework 5 和 6.NET 4.5 下运行。 Entity Framework 5 的性能改进的许多驻留在.NET 4.5 附带的核心组件。

实体框架 6 是带外版本，并不依赖于随.NET 实体框架组件。 实体框架 6，适用于.NET 4.0 和.NET 4.5 中，并可提供向那些尚未升级从.NET 4.0，但希望将其应用程序中的最新实体框架位大性能优势。 当本文档提到 Entity Framework 6 时，它是指在撰写本文时可用的最新版本： 版本 6.1.0。

## <a name="2-cold-vs-warm-query-execution"></a>2.冷 vs。热查询执行

第一次针对给定的模型，进行任何查询的实体框架做了大量工作在后台加载和验证模型。 我们经常引用此第一个查询为"冷"查询。  针对已加载模型的进一步查询名为"热"的查询，并且快得多。

让时间的花费时使用实体框架中，执行查询的高级视图，请参阅操作 Entity Framework 6 中的改进。

**第一次查询执行 – 冷查询**

| 代码用户写入                                                                                     | 操作                    | EF4 性能影响                                                                                                                                                                                                                                                                                                                                                                                                        | EF5 性能影响                                                                                                                                                                                                                                                                                                                                                                                                                                                    | EF6 对性能的影响                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|:-----------------------------------------------------------------------------------------------------|:--------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `using(var db = new MyContext())` <br/> `{`                                                          | 上下文创建          | 中等                                                                                                                                                                                                                                                                                                                                                                                                                        | 中等                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `  var q1 = ` <br/> `    from c in db.Customers` <br/> `    where c.Id == id1` <br/> `    select c;` | 查询表达式创建 | 低                                                                                                                                                                                                                                                                                                                                                                                                                           | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `  var c1 = q1.First();`                                                                             | LINQ 查询执行      | 元数据加载： 高但缓存 <br/> -查看生成： 可能会非常高但缓存 <br/> 参数评估： 中型 <br/> -查询转换： 中型 <br/> -具体化器生成： 中等但缓存 <br/> -数据库执行查询： 可能会很高 <br/> + Connection.Open <br/> + Command.ExecuteReader <br/> + DataReader.Read <br/> 对象具体化： 中型 <br/> -标识查找： 中型 | 元数据加载： 高但缓存 <br/> -查看生成： 可能会非常高但缓存 <br/> 参数评估： 低 <br/> -查询转换： 中等但缓存 <br/> -具体化器生成： 中等但缓存 <br/> -数据库执行查询： 可能会很高 （更好的查询在某些情况下） <br/> + Connection.Open <br/> + Command.ExecuteReader <br/> + DataReader.Read <br/> 对象具体化： 中型 <br/> -标识查找： 中型 | 元数据加载： 高但缓存 <br/> -查看生成： 中等但缓存 <br/> 参数评估： 低 <br/> -查询转换： 中等但缓存 <br/> -具体化器生成： 中等但缓存 <br/> -数据库执行查询： 可能会很高 （更好的查询在某些情况下） <br/> + Connection.Open <br/> + Command.ExecuteReader <br/> + DataReader.Read <br/> 对象具体化： 中等 （比 EF5 速度要快） <br/> -标识查找： 中型 |
| `}`                                                                                                  | Connection.Close          | 低                                                                                                                                                                                                                                                                                                                                                                                                                           | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |


**第二个查询执行 – 暖查询**

| 代码用户写入                                                                                     | 操作                    | EF4 性能影响                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | EF5 性能影响                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | EF6 对性能的影响                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|:-----------------------------------------------------------------------------------------------------|:--------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `using(var db = new MyContext())` <br/> `{`                                                          | 上下文创建          | 中等                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | 中等                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `  var q1 = ` <br/> `    from c in db.Customers` <br/> `    where c.Id == id1` <br/> `    select c;` | 查询表达式创建 | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `  var c1 = q1.First();`                                                                             | LINQ 查询执行      | 元数据~~加载~~查找：~~高但缓存~~低 <br/> -查看~~代~~查找：~~可能会非常高但缓存~~低 <br/> 参数评估： 中型 <br/> -查询~~翻译~~查找： 中型 <br/> -具体化~~代~~查找：~~中等但缓存~~低 <br/> -数据库执行查询： 可能会很高 <br/> + Connection.Open <br/> + Command.ExecuteReader <br/> + DataReader.Read <br/> 对象具体化： 中型 <br/> -标识查找： 中型 | 元数据~~加载~~查找：~~高但缓存~~低 <br/> -查看~~代~~查找：~~可能会非常高但缓存~~低 <br/> 参数评估： 低 <br/> -查询~~翻译~~查找：~~中等但缓存~~低 <br/> -具体化~~代~~查找：~~中等但缓存~~低 <br/> -数据库执行查询： 可能会很高 （更好的查询在某些情况下） <br/> + Connection.Open <br/> + Command.ExecuteReader <br/> + DataReader.Read <br/> 对象具体化： 中型 <br/> -标识查找： 中型 | 元数据~~加载~~查找：~~高但缓存~~低 <br/> -查看~~代~~查找：~~中等但缓存~~低 <br/> 参数评估： 低 <br/> -查询~~翻译~~查找：~~中等但缓存~~低 <br/> -具体化~~代~~查找：~~中等但缓存~~低 <br/> -数据库执行查询： 可能会很高 （更好的查询在某些情况下） <br/> + Connection.Open <br/> + Command.ExecuteReader <br/> + DataReader.Read <br/> 对象具体化： 中等 （比 EF5 速度要快） <br/> -标识查找： 中型 |
| `}`                                                                                                  | Connection.Close          | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | 低                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |


有几种方法，以减少冷和热查询的性能开销，我们将查看这些下一节中。 具体而言，我们将介绍减少模型加载冷查询中通过使用预生成的视图，这将有助于缓解性能问题的视图生成过程中发生的成本。 对于感兴趣的查询，我们将介绍查询计划缓存、 没有跟踪查询和不同的查询执行选项。

### <a name="21-what-is-view-generation"></a>2.1 视图生成是什么？

若要了解哪些视图生成是，我们必须先了解什么是"映射视图"。 映射视图是可执行表示形式中每个实体集和关联的映射指定的转换。 在内部，这些映射视图将采用 CQTs （规范查询目录树） 的形状。 有两种类型的映射视图：

-   查询视图： 这些表示从数据库架构转到概念模型所需的转换。
-   更新的视图： 这些表示从概念模型转到数据库架构所需的转换。

请记住，概念模型可能不同于数据库架构以各种方式。 例如，可能使用一个单个表来存储两个不同的实体类型的数据。 继承和重要的映射的映射视图复杂程度方面扮演角色。

计算与指定的映射基于这些视图的过程是我们所说的视图生成。 视图生成可以或者发生动态加载模型时，或在生成时，通过使用"预生成的视图";后一种进行序列化到 C 的 Entity SQL 语句的形式\#或 VB 文件。

视图生成时，它们也进行了验证。 从性能角度看，大多数的成本视图生成的是实际的验证视图这可确保实体之间的连接有意义，并且具有正确的基数为所有受支持的操作。

针对实体集的查询执行时，查询结合相应的查询视图中，此组合的结果是通过和运行计划编译器创建的查询的后备存储可以理解的表示形式。 对于 SQL Server，此编译的最终结果将 T-SQL SELECT 语句。 首次执行实体集的更新，更新视图是通过将其转换为目标数据库的 DML 语句类似的过程运行。

### <a name="22-factors-that-affect-view-generation-performance"></a>2.2 因素会影响视图生成性能

视图生成步骤的性能不仅取决于您的模型的大小，但还上如何相互关联的模型是。 如果通过继承链或关联连接两个实体，即是连接。 同样如果两个表连接通过外键，则说明连接。 已连接的实体和架构中的表数增加时，视图生成的成本增加。

虽然我们使用一些优化此进行改进，我们使用来生成和验证视图的算法是在最坏的情况下，指数。 似乎对性能产生负面影响的最大因素是：

-   模型引用的实体数和这些实体之间的关联量大小。
-   模型的复杂性，特别是涉及大量的类型的继承。
-   使用独立关联，而不外键关联。

小型的简单模型的成本可能很小，可不会受到使用预生成的视图。 模型大小和复杂性增加，有几种方式可以降低的成本视图生成和验证。

### <a name="23-using-pre-generated-views-to-decrease-model-load-time"></a>2.3 使用 Pre-Generated 视图以减少模型加载时间

有关如何使用 Entity Framework 6 的预生成的视图的详细信息，请访问[Pre-Generated 映射视图](~/ef6/fundamentals/performance/pre-generated-views.md)

#### <a name="231-pre-generated-views-using-the-entity-framework-power-tools-community-edition"></a>2.3.1 预生成视图使用 Entity Framework Power Tools 社区版

可以使用[Entity Framework 6 Power 工具 Community Edition](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EntityFramework6PowerToolsCommunityEdition)通过右键单击模型类文件并使用实体框架菜单选择"生成视图"中生成的 EDMX 和 Code First 模型视图。 Entity Framework Power Tools Community 版本仅适用于 DbContext 派生上下文。

#### <a name="232-how-to-use-pre-generated-views-with-a-model-created-by-edmgen"></a>2.3.2 如何 EDMGen 所创建的模型使用预生成的视图

EDMGen 是一个实用工具，附带了.NET 和工作原理与 Entity Framework 4 和 5，但不能与 Entity Framework 6。 EDMGen 可以从命令行生成的模型文件、 对象层和视图。 一个输出将为你的选择，VB 或 C 语言中的视图文件\#。 这是包含每个实体集的实体 SQL 代码段的代码文件。 若要启用预生成的视图，只应在项目中包含该文件。

如果您手动对模型的架构文件进行编辑，将需要重新生成视图文件。 您可以执行此操作通过运行使用 EDMGen **/mode:ViewGeneration**标志。

#### <a name="233-how-to-use-pre-generated-views-with-an-edmx-file"></a>2.3.3 如何 Pre-Generated 视图使用 EDMX 文件

此外可以使用 EDMGen 生成的 EDMX 文件视图-上面引用的 MSDN 主题介绍如何添加预生成事件，若要这样做的但这是复杂，有某些情况下，根本不可能。 它是使用 T4 模板在 edmx 文件中您的模型时生成视图通常更为方便。

ADO.NET 团队博客中发布了一文章，介绍如何为视图生成使用 T4 模板 ( \<http://blogs.msdn.com/b/adonet/archive/2008/06/20/how-to-use-a-t4-template-for-view-generation.aspx>)。 此文章包括可以下载并添加到项目的模板。 模板已编写实体框架的第一个版本，因此它们不保证适用于实体框架的最新版本。 但是，您可以下载一组较新的视图生成模板的 Entity Framework 4 和 5from Visual Studio 库：

-   VB.NET: \<http://visualstudiogallery.msdn.microsoft.com/118b44f2-1b91-4de2-a584-7a680418941d>
-   C\#: \<http://visualstudiogallery.msdn.microsoft.com/ae7730ce-ddab-470f-8456-1b313cd2c44d>

如果您使用的 Entity Framework 6，可以获取视图生成 T4 模板从 Visual Studio 库\<http://visualstudiogallery.msdn.microsoft.com/18a7db90-6705-4d19-9dd1-0a6c23d0751f>。

### <a name="24-reducing-the-cost-of-view-generation"></a>2.4 降低的成本视图生成

使用预生成的视图移到设计时的视图生成从加载的模型 （运行时间） 的成本。 尽管这改进了在运行时的启动性能，你将在开发时仍然遇到视图生成的痛苦。 有几个其他技巧可帮助降低成本的同时在编译时和运行的时的视图生成。

#### <a name="241-using-foreign-key-associations-to-reduce-view-generation-cost"></a>2.4.1 使用外键关联以减少视图生成成本

我们已看到许多情况下，极大地切换到外键关联从独立关联模型中的关联改进了视图生成中所用的时间。

为了演示这一改进，我们使用 EDMGen 生成 Navision 模型的两个版本。 *注意： seeappendix Cfor Navision 模型的说明。* 对于此练习，因其极大量的实体和它们之间的关系而有趣的是 Navision 模型。

与外键关联生成的此非常大的模型的一个版本，其他生成使用独立关联。 我们然后超时所用的时间来生成每个模型的视图。 实体 Framework5 测试用于从类 EntityViewGenerator GenerateViews() 方法生成视图，而 Entity Framework 6 测试使用了从类 StorageMappingItemCollection GenerateViews() 方法。 这由于代码重新构建 Entity Framework 6 基本代码中发生的。

使用 Entity Framework 5，具有外键的模型的视图生成需要在实验室计算机 65 分钟。 未知的时间长度则可能需要花费生成使用独立关联的模型的视图。 我们保留超过一个月前我们的实验室安装每月更新中重新启动计算机正在运行的测试。

使用 Entity Framework 6，具有外键的模型的视图生成需要在同一台实验室计算机 28 秒。 使用独立关联的模型的视图生成需要 58 秒。 对其视图生成代码完成到 Entity Framework 6 的改进意味着多个项目不需要预生成的视图来获取更快的启动时间。

它是重要的预生成 Entity Framework 4 和 5 中的视图，可完成使用 EDMGen 或 Entity Framework Power Tools 的备注。 通过 Entity Framework Power Tools 或如中所述以编程方式可以在 Entity Framework 6 图完成生成[Pre-Generated 映射视图](~/ef6/fundamentals/performance/pre-generated-views.md)。

##### <a name="2411-how-to-use-foreign-keys-instead-of-independent-associations"></a>2.4.1.1 如何使用外键而不是独立关联

默认情况下，在 Visual Studio 中使用 EDMGen 或在实体设计器，获取 Fk，它只需单个复选框或命令行标志的 Fk 和 IAs 之间进行切换。

如果您有一个大型的 Code First 模型，使用独立关联必须视图生成相同的作用。 虽然一些开发人员会认为它污染它们的对象模型，通过包括你依赖的对象的类上的外键属性，可以避免这种影响。 您可以找到这一主题的详细信息\<http://blog.oneunicorn.com/2011/12/11/whats-the-deal-with-mapping-foreign-keys-using-the-entity-framework/>。

| 使用时      | 操作步骤                                                                                                                                                                                                                                                                                                                              |
|:----------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 实体设计器 | 添加后两个实体之间的关联，请确保具有引用约束。 引用约束告诉实体框架使用而不是独立关联的外键。 有关更多详细信息，请访问\<http://blogs.msdn.com/b/efdesign/archive/2009/03/16/foreign-keys-in-the-entity-framework.aspx>。 |
| EDMGen          | 当使用 EDMGen 从数据库生成你的文件，将考虑外键，并将其添加到这种情况下的模型中。 有关由 EDMGen 公开的不同选项的详细信息，请访问[http://msdn.microsoft.com/library/bb387165.aspx](https://msdn.microsoft.com/library/bb387165.aspx)。                           |
| Code First      | 请参阅的"关系约定"部分[Code First 约定](~/ef6/modeling/code-first/conventions/built-in.md)主题，了解如何使用 Code First 时包括依赖对象上的外键属性的信息。                                                                                              |

#### <a name="242-moving-your-model-to-a-separate-assembly"></a>2.4.2 将您的模型移动到单独的程序集

时直接在应用程序的项目中包含您的模型并生成通过预生成事件或 T4 模板的视图，视图生成和验证会发生时重新生成项目时，即使该模型未更改。 如果将模型移动到单独的程序集并从应用程序的项目中引用它，可以无需重新生成包含模型的项目对应用程序进行其他更改。

*注意：*  移动您的模型来分隔时程序集，请务必将该模型的连接字符串复制到客户端项目的应用程序配置文件。

#### <a name="243-disable-validation-of-an-edmx-based-model"></a>2.4.3 禁用了基于 edmx 的模型的验证

EDMX 模型是在编译时验证，即使模型保持不变。 如果已验证了您的模型，可以禁止在编译时验证显示在属性窗口中的"验证"生成属性设置为 false。 当您更改映射或模型时，可以重新暂时启用验证，以验证所做的更改。

请注意，性能改进了实体框架设计器的 Entity Framework 6 的"验证"生成成本是远低于在以前版本的设计器。

## <a name="3-caching-in-the-entity-framework"></a>在实体框架中的 3 个缓存

实体框架具有以下形式的缓存内置：

1.  对象缓存-内置于一个 ObjectContext 实例 ObjectStateManager 跟踪已使用该实例在检索对象的内存中。 这也称为是第一级缓存。
2.  查询计划缓存的不止一次执行查询时重复使用生成的存储命令。
3.  元数据缓存-在同一个模型的不同连接之间共享模型的元数据。

除了 EF 提供了默认情况下，一种特殊的 ADO.NET 数据提供程序名为换行提供程序还可用于扩展实体框架使用从数据库中检索的结果缓存的缓存也称为第二级缓存。

### <a name="31-object-caching"></a>3.1 对象缓存

默认情况下时查询的结果中返回实体之前只是 EF 将具体化，ObjectContext 会检查是否具有相同键的实体具有已被加载到其 ObjectStateManager。 如果已存在具有相同键的实体 EF 将其包括在查询的结果。 尽管 EF 仍将发出对数据库查询，此行为可以跳过大多数具体化实体多次的成本。

#### <a name="311-getting-entities-from-the-object-cache-using-dbcontext-find"></a>3.1.1 从使用查找 DbContext 对象缓存中获取实体

与常规查询，不同 DbSet (第一次 EF 4.1 中包含的 Api) 中的查找方法将搜索在内存中执行之前甚至发出对数据库查询。 请务必注意，两个不同的 ObjectContext 实例将具有两个不同的 ObjectStateManager 实例，这意味着它们具有单独的对象缓存。

查找使用的主键值来尝试查找由上下文跟踪的实体。 如果该实体不在上下文中然后执行并针对数据库评估查询和如果上下文中或在数据库中找不到该实体，则返回 null。 请注意，查找也会返回已添加到上下文，但尚未保存到数据库的实体。

没有需要时使用查找执行性能考虑事项。 默认情况下此方法的调用以便能够检测到仍在等待提交到数据库的更改都将触发对象缓存的验证。 此过程可能会非常昂贵，如果有大量的对象缓存中或要添加到对象缓存中，一个大型的对象图中的对象，但它也可禁用。 在某些情况下，您可能会觉察到通过调用方法时禁用自动检测的到差异一个数量级的更改。 尚未第二个数量级，所以我们认为该对象实际上是时与在缓存中对象具有要检索从数据库时。 下面是与所使用的以毫秒为单位的负载为 5000 实体表示我们 microbenchmarks 一些测量结果的示例图表：

![.NET 4.5 对数刻度](~/ef6/media/net45logscale.png ".NET 4.5 的对数刻度")

禁用自动检测更改的查找示例：

``` csharp
    context.Configuration.AutoDetectChangesEnabled = false;
    var product = context.Products.Find(productId);
    context.Configuration.AutoDetectChangesEnabled = true;
    ...
```

您拥有使用 Find 方法时要考虑的是：

1.  如果对象不在缓存中查找的优点起作用，但语法仍比通过键查询更简单。
2.  如果自动检测更改已启用的 Find 方法成本可能会增加了一个数量级，或更多具体取决于您的模型和对象缓存中的实体的量的复杂性。

此外，请记住，仅查找返回要查找的实体，它不会自动加载其关联的实体如果它们尚不在对象缓存中。 如果需要检索关联的实体，可以通过预先加载与键使用查询。 有关详细信息请参阅**8.1 延迟加载 vs。预先加载**。

#### <a name="312-performance-issues-when-the-object-cache-has-many-entities"></a>3.1.2 性能问题时对象缓存中有多个实体

对象缓存有助于提高实体框架的整体响应能力。 但是，当对象缓存了极大量的实体加载它可能会影响某些操作，如添加、 删除、 查找、 入口、 SaveChanges 和的详细信息。 具体而言，将极大对象缓存产生负面影响触发调用 DetectChanges 的操作。 DetectChanges 对象状态管理器和直接由对象关系图的大小决定其性能将与同步对象图。 有关 DetectChanges 的详细信息，请参阅[跟踪 POCO 实体中的更改](https://msdn.microsoft.com/library/dd456848.aspx)。

在使用 Entity Framework 6，开发人员就能够直接在 DbSet，而不是对集合进行迭代和调用添加一次每个实例上调用 AddRange 和 RemoveRange。 使用 range 方法的优点是 DetectChanges 的成本仅为整个实体而不是每个添加的实体每一次的集的一次付费。

### <a name="32-query-plan-caching"></a>3.2 查询计划缓存

第一次执行查询时，它将经历内部计划编译器来将概念的查询转换为存储命令 (例如，T-SQL 针对 SQL Server 运行时执行)。  如果启用了查询计划缓存，查询是在下次执行存储命令检索直接从查询计划缓存的执行，从而绕过计划编译器。

查询计划缓存在同一 AppDomain 中的 ObjectContext 实例之间共享。 无需保存到一个 ObjectContext 实例，以便从查询计划缓存中受益。

#### <a name="321-some-notes-about-query-plan-caching"></a>3.2.1 有关查询计划缓存的一些注意事项

-   查询计划缓存共享的所有查询类型： 实体 SQL、 LINQ to Entities 和 CompiledQuery 对象。
-   默认情况下，查询计划缓存可用于 Entity SQL 查询是否执行通过 EntityCommand 或 ObjectQuery。 它还默认情况下启用 linq to Entities 查询在实体框架在.NET 4.5 和 Entity Framework 6
    -   可以通过 （在 EntityCommand 或 ObjectQuery） EnablePlanCaching 属性设置为 false 禁用查询计划缓存。 例如：
``` csharp
                    var query = from customer in context.Customer
                                where customer.CustomerId == id
                                select new
                                {
                                    customer.CustomerId,
                                    customer.Name
                                };
                    ObjectQuery oQuery = query as ObjectQuery;
                    oQuery.EnablePlanCaching = false;
```
-   对于参数化查询，更改参数的值将仍命中缓存的查询。 但是，更改参数的方面 （例如，大小、 精度或规模） 都将遇到缓存中的不同项。
-   使用实体 SQL，查询字符串时键的一部分。 在所有更改查询将导致不同的缓存条目，即使查询在功能上等效。 这包括更改大小写或空格。
-   当使用 LINQ，查询的处理来生成键的一部分。 LINQ 表达式更改因此将生成不同的密钥。
-   可以应用其他技术限制;有关更多详细信息，请参阅 Autocompiled 查询。

#### <a name="322-cache-eviction-algorithm"></a>3.2.2 缓存逐出算法

了解内部算法可将帮助你找出时到启用或禁用查询计划缓存。 清理算法如下所示：

1.  后缓存中包含了固定数量的条目 (800)，我们开始计时器定期 （一次每分钟） 扫描缓存。
2.  缓存扫描期间条目已从 LFRU （最频繁-最近使用过的） 上的缓存的基础。 确定哪些项弹出时，此算法将考虑命中的次数和年龄。
3.  在每个缓存扫描结束时，缓存中同样包含 800 条目。

确定要逐出的项时，将同等对待所有缓存条目。 这意味着 CompiledQuery 存储命令具有相同的机会逐出作为 Entity SQL 查询的存储命令。

请注意，缓存逐出计时器触发事件时在缓存中，有 800 实体但缓存仅扫频 60 秒后此计时器已启动。 这意味着，最多 60 秒缓存可能会增长得非常大。

#### <a name="323-test-metrics-demonstrating-query-plan-caching-performance"></a>3.2.3 测试演示查询计划缓存性能指标

若要演示的查询计划缓存在应用程序的性能效果，我们执行测试，我们执行多个针对 Navision 模型的 Entity SQL 查询。 请参阅的附录 Navision 模型和执行的查询的类型的说明。 在此测试中，我们首先循环访问查询的列表并执行每个一次将其添加到缓存中，（如果缓存已启用）。 此步骤是 untimed。 接下来，我们睡眠状态超过 60 秒，以允许缓存扫描结合使用以执行; 的主线程最后，我们循环访问列表第 2 个时间来执行缓存的查询。 此外，他 SQL Server 计划高速缓存刷新之前每个集的查询执行，以便我们获得准确的时间反映由查询计划缓存的优势。

##### <a name="3231-test-results"></a>3.2.3.1 测试结果

| 测试                                                                   | EF5 没有缓存 | EF5 缓存 | EF6 不缓存 | EF6 缓存 |
|:-----------------------------------------------------------------------|:-------------|:-----------|:-------------|:-----------|
| 枚举所有 18723 查询                                          | 124          | 125.4      | 达到了 1.243        | 125.3      |
| 避免扫描 （只是第一次 800 查询，无论复杂性如何）  | 41.7         | 5.5        | 40.5         | 5.4        |
| 只需 AggregatingSubtotals 查询 （178 总计-可避免扫描） | 39.5         | 4.5        | 38.1         | 4.6        |

*所有时间以秒为单位。*

道德的时执行还有的不同查询 （例如，动态创建的查询），缓存不起作用并生成刷新的缓存可以保留受益最充分地利用计划缓存中实际使用的查询。

AggregatingSubtotals 查询是最复杂的与我们测试的查询。 按预期运行，更复杂的查询是，你将看到从查询计划缓存的益处越多。

由于 CompiledQuery 实际上是具有其计划缓存的 LINQ 查询，而不是等效的 Entity SQL 查询 CompiledQuery 比较应具有类似的结果。 事实上，如果应用具有大量动态 Entity SQL 查询，填充查询缓存将实际上也会导致 CompiledQueries"反编译"时从缓存中刷新。 在此方案中，可能通过禁用动态查询，以确定优先级 CompiledQueries 的缓存提高性能。 更棒的是，当然，会重写应用程序以使用参数化的查询，而不是动态查询。

### <a name="33-using-compiledquery-to-improve-performance-with-linq-queries"></a>3.3 使用 CompiledQuery 提高使用 LINQ 查询的性能

我们测试表明，使用 CompiledQuery 可以通过自带的 7%权益 autocompiled LINQ 查询;这意味着，您将 7%更少的时间从 Entity Framework 堆栈; 执行代码它并不意味着你的应用程序将在 7%更快。 通常情况下，编写和 EF 5.0 中的 CompiledQuery 对象进行维护的成本可能不值得时带来的益处相的麻烦。 您的实际效果可能会有所不同，因此执行此选项，如果你的项目需要额外的推送。 请注意，CompiledQueries 仅与 ObjectContext 派生模型兼容，并与 DbContext 派生模型不兼容。

有关创建和调用 CompiledQuery 的详细信息，请参阅[编译的查询 (LINQ to Entities)](https://msdn.microsoft.com/library/bb896297.aspx)。

有两个您需要执行使用 CompiledQuery，即要求使用静态实例和这些问题所拥有的可组合性时的注意事项。 以下是这些两个注意事项的详细说明。

#### <a name="331-use-static-compiledquery-instances"></a>3.3.1 使用静态 CompiledQuery 实例

编译 LINQ 查询是一个耗时的过程，因为我们不希望每次我们需要从数据库提取数据时执行此操作。 CompiledQuery 实例可以进行一次编译和运行多次，但必须要小心操作和采购以重复使用相同的 CompiledQuery 实例而不是反复地编译每次。 使用静态成员来存储 CompiledQuery 实例将成为必需的;否则不会看到任何权益。

例如，假设页面包含用于处理显示所选类别产品的以下方法正文：

``` csharp
    // Warning: this is the wrong way of using CompiledQuery
    using (NorthwindEntities context = new NorthwindEntities())
    {
        string selectedCategory = this.categoriesList.SelectedValue;

        var productsForCategory = CompiledQuery.Compile<NorthwindEntities, string, IQueryable<Product>>(
            (NorthwindEntities nwnd, string category) =>
                nwnd.Products.Where(p => p.Category.CategoryName == category)
        );

        this.productsGrid.DataSource = productsForCategory.Invoke(context, selectedCategory).ToList();
        this.productsGrid.DataBind();
    }

    this.productsGrid.Visible = true;
```

在这种情况下，将动态创建新的 CompiledQuery 实例，每次调用的方法。 而不是通过从查询计划缓存中检索存储命令来查看性能优势，CompiledQuery 会通过计划编译器每次创建一个新实例。 事实上，您将为污染查询计划缓存的新的 CompiledQuery 项，每次调用的方法。

相反，你想要创建的已编译的查询，一个静态实例，以便每次调用该方法时所调用同一个已编译的查询。 一种方法这是通过将 CompiledQuery 实例添加为对象上下文的成员。  然后可以操作小更简洁的帮助器方法通过访问 CompiledQuery:

``` csharp
    public partial class NorthwindEntities : ObjectContext
    {
        private static readonly Func<NorthwindEntities, string, IEnumerable<Product>> productsForCategoryCQ = CompiledQuery.Compile(
            (NorthwindEntities context, string categoryName) =>
                context.Products.Where(p => p.Category.CategoryName == categoryName)
            );

        public IEnumerable<Product> GetProductsForCategory(string categoryName)
        {
            return productsForCategoryCQ.Invoke(this, categoryName).ToList();
        }
```

此帮助器方法会调用，如下所示：

``` csharp
    this.productsGrid.DataSource = context.GetProductsForCategory(selectedCategory);
```

#### <a name="332-composing-over-a-compiledquery"></a>3.3.2 撰写 CompiledQuery 转移

用于任何 LINQ 查询组合的功能是非常有用;若要执行此操作，您只需调用方法后 IQueryable 如*skip （)* 或*count （)*。 但是，所以本质上来说返回一个新的 IQueryable 对象。 虽然没有执行任何操作，使您从技术上讲不再通过 CompiledQuery 撰写，但这样做将导致生成新的 IQueryable 对象，需要再次传递通过计划编译器。

某些组件将使用组合 IQueryable 对象，若要启用高级的功能。 例如，ASP。NET 的 GridView 可以进行数据绑定到 IQueryable 对象通过 SelectMethod 属性。 GridView 然后将组合此 IQueryable 对象，以允许排序和分页通过数据模型。 正如您所看到的 GridView 使用 CompiledQuery 将不会命中已编译的查询，但会生成一个新的 autocompiled 查询。

客户顾问团队讨论了此问题在其"潜在性能问题与编译 LINQ 查询重新编译"博客文章： <http://blogs.msdn.com/b/appfabriccat/archive/2010/08/06/potential-performance-issues-with-compiled-linq-query-re-compiles.aspx>。

当向查询添加渐进式筛选器时，其中可能会遇到此类的一个位置。 例如，假设你有客户页的可选筛选器 （例如，国家/地区和 OrdersCount） 的多个下拉列表。 你可以组合这些筛选器的 CompiledQuery IQueryable 结果，但这样做将导致每次执行该计划编译器通过将新查询。

``` csharp
    using (NorthwindEntities context = new NorthwindEntities())
    {
        IQueryable<Customer> myCustomers = context.InvokeCustomersForEmployee();

        if (this.orderCountFilterList.SelectedItem.Value != defaultFilterText)
        {
            int orderCount = int.Parse(orderCountFilterList.SelectedValue);
            myCustomers = myCustomers.Where(c => c.Orders.Count > orderCount);
        }

        if (this.countryFilterList.SelectedItem.Value != defaultFilterText)
        {
            myCustomers = myCustomers.Where(c => c.Address.Country == countryFilterList.SelectedValue);
        }

        this.customersGrid.DataSource = myCustomers;
        this.customersGrid.DataBind();
    }
```

 若要避免此重新编译，可以重写 CompiledQuery 要考虑的可能的筛选器：

``` csharp
    private static readonly Func<NorthwindEntities, int, int?, string, IQueryable<Customer>> customersForEmployeeWithFiltersCQ = CompiledQuery.Compile(
        (NorthwindEntities context, int empId, int? countFilter, string countryFilter) =>
            context.Customers.Where(c => c.Orders.Any(o => o.EmployeeID == empId))
            .Where(c => countFilter.HasValue == false || c.Orders.Count > countFilter)
            .Where(c => countryFilter == null || c.Address.Country == countryFilter)
        );
```

这会在 UI 中，如调用：

``` csharp
    using (NorthwindEntities context = new NorthwindEntities())
    {
        int? countFilter = (this.orderCountFilterList.SelectedIndex == 0) ?
            (int?)null :
            int.Parse(this.orderCountFilterList.SelectedValue);

        string countryFilter = (this.countryFilterList.SelectedIndex == 0) ?
            null :
            this.countryFilterList.SelectedValue;

        IQueryable<Customer> myCustomers = context.InvokeCustomersForEmployeeWithFilters(
                countFilter, countryFilter);

        this.customersGrid.DataSource = myCustomers;
        this.customersGrid.DataBind();
    }
```

 此处的影响是生成的存储命令始终具有带 null 检查，筛选器，但这些应是要优化的数据库服务器相当简单：

``` SQL
...
WHERE ((0 = (CASE WHEN (@p__linq__1 IS NOT NULL) THEN cast(1 as bit) WHEN (@p__linq__1 IS NULL) THEN cast(0 as bit) END)) OR ([Project3].[C2] > @p__linq__2)) AND (@p__linq__3 IS NULL OR [Project3].[Country] = @p__linq__4)
```

### <a name="34-metadata-caching"></a>3.4 元数据缓存

实体框架还支持元数据缓存。 这实质上跨同一个模型的不同连接缓存的类型信息和类型到数据库的映射信息。 元数据缓存是每个 AppDomain 唯一的。

#### <a name="341-metadata-caching-algorithm"></a>3.4.1 元数据缓存算法

1.  每个 EntityConnection 存储在 ItemCollection 模型的元数据信息。
    -   请注意，有不同的 ItemCollection 对象模型的不同部分。 例如，StoreItemCollections 包含数据库模型中; 有关的信息ObjectItemCollection 包含数据模型中; 有关的信息EdmItemCollection 包含有关概念模型的信息。

2.  如果两个连接使用相同的连接字符串，它们将共享同一个 ItemCollection 实例。
3.  在功能上等效，但文本上不同的连接字符串可能会导致不同的元数据缓存。 我们不要对进行词汇切分的连接字符串，因此，只需更改标记的顺序应导致共享元数据。 但是，在功能上看起来相同的两个连接字符串可能无法计算为相同后词汇切分。
4.  ItemCollection 定期检查的使用。 如果确定，未被最近访问工作区，它会将标记为在下一步的缓存扫描的清理。
5.  只创建一个 EntityConnection 将导致要创建 （尽管直到打开连接时，将不会初始化在其中的项集合） 的元数据缓存。 此工作区将保留内存中，直到缓存算法确定它不是"使用中"。

客户顾问团队编写了描述保存到 ItemCollection 的引用以使用大型模型时避免"弃用"的博客文章： \<http://blogs.msdn.com/b/appfabriccat/archive/2010/10/22/metadataworkspace-reference-in-wcf-services.aspx>。

#### <a name="342-the-relationship-between-metadata-caching-and-query-plan-caching"></a>3.4.2 元数据缓存和缓存查询计划之间的关系

查询计划缓存实例位于 MetadataWorkspace ItemCollection 的存储类型。 这意味着缓存的存储命令将用于针对任何上下文中使用给定的 MetadataWorkspace 实例化的查询。 这还意味着，如果您有两个连接字符串稍有不同之后将拆分为标记, 不匹配，将具有不同的查询计划缓存实例。

### <a name="35-results-caching"></a>3.5 结果缓存

与结果缓存 （也称为"第二层缓存"），在本地缓存中保留查询的结果。 发出查询时，您首先查看结果是否之前针对存储查询本地可用。 虽然实体框架不直接支持缓存的结果，就可以使用包装提供程序添加第二层缓存。 第二层缓存使用的示例包装提供程序是 Alachisoft 的[实体框架第二个级别缓存的基础 NCache](http://www.alachisoft.com/ncache/entity-framework.html)。

此实现的第二级缓存是一种采用 LINQ 表达式计算后的位置 （和 funcletized） 注入的功能和计算或从第一级缓存中检索查询执行计划。 第二级缓存将然后仅将结果存储在原始数据库以便具体化管道仍之后执行。

#### <a name="351-additional-references-for-results-caching-with-the-wrapping-provider"></a>3.5.1 使用包装提供程序缓存的结果的其他参考

-   Julie Lerman 撰写了包括如何更新示例包装提供程序以使用 Windows Server AppFabric caching"第二级缓存在实体框架和 Windows Azure"MSDN 文章： [https://msdn.microsoft.com/magazine/hh394143.aspx](https://msdn.microsoft.com/magazine/hh394143.aspx)
-   如果您正在使用 Entity Framework 5，团队博客中发布了一文章，介绍如何使用 Entity Framework 5 的缓存提供程序运行的工作： \<http://blogs.msdn.com/b/adonet/archive/2010/09/13/ef-caching-with-jarek-kowalski-s-provider.aspx>。 它还包括帮助自动执行添加到你的项目第二级缓存的 T4 模板。

## <a name="4-autocompiled-queries"></a>4 个 Autocompiled 查询

在使用实体框架对数据库发出查询时，它必须都将通过一系列的步骤之前实际上具体化结果;此类的一个步骤是将查询编译。 实体 SQL 查询是已知可具有良好的性能，因为它们会自动缓存，因此第二个或第三个时间执行同一查询，它可以跳过计划编译器，改为使用缓存的计划。

实体框架 5 引入了自动缓存 LINQ to Entities 查询以及。 在过去版本的 Entity Framework 创建 CompiledQuery 加快应用的性能是常见的做法，因为这将使 LINQ to Entities 查询可缓存。 由于缓存而不使用的 compiledquery 都现在自动完成，我们调用此功能"autocompiled 查询"。 有关查询计划缓存和其机制的详细信息，请参阅查询计划缓存。

实体框架检测时查询所需重新编译，并且这样做，即使它已编译之前调用该查询时。 导致要重新编译的查询的常见情况是：

-   更改 mergeoption，然后在查询相关联。 将不使用缓存的查询、 改为将再次运行计划编译器和新创建的计划缓存。
-   更改 ContextOptions.UseCSharpNullComparisonBehavior 的值。 获取更改 mergeoption，然后与相同的效果。

其他条件可以防止您的查询使用缓存。 常见示例包括：

-   使用 IEnumerable&lt;T&gt;。包含&lt;&gt;（T 值）。
-   使用生成常量具有查询的函数。
-   使用非映射对象的属性。
-   将查询链接到另一个需要在下次重新编译的查询。

### <a name="41-using-ienumerablelttgtcontainslttgtt-value"></a>4.1 使用 IEnumerable&lt;T&gt;。包含&lt;T&gt;（T 值）

实体框架不会缓存查询调用 IEnumerable&lt;T&gt;。包含&lt;T&gt;（T 值） 对内存中集合，因为集合的值被视为易失性。 下面的示例查询将不缓存，因此始终将由计划编译器处理：

``` csharp
int[] ids = new int[10000];
...
using (var context = new MyContext())
{
    var query = context.MyEntities
                    .Where(entity => ids.Contains(entity.Id));

    var results = query.ToList();
    ...
}
```

请注意，它包含对 IEnumerable 的大小执行速度快或如何慢确定您的查询进行编译。 使用在上面的示例所示的大型集合时，性能可能会显著降低。

实体框架 6 包含优化方式 IEnumerable&lt;T&gt;。包含&lt;T&gt;执行查询时，工作原理 （T 值）。 生成的 SQL 代码是生成要快得多且更具可读性，并在大多数情况下它也会执行更快地在服务器中。

### <a name="42-using-functions-that-produce-queries-with-constants"></a>4.2 使用生成常量具有查询的函数

Skip （）、 take （）、 contains （） 和 DefautIfEmpty() LINQ 运算符不会生成包含参数的 SQL 查询，但改为将作为常量传递给它们的值。 正因为如此，会是相同的最终污染查询的查询计划缓存中，EF 堆栈上和在数据库服务器上，并执行操作不获取重复利用除非在后续查询执行过程中使用相同的常量。 例如：

``` csharp
var id = 10;
...
using (var context = new MyContext())
{
    var query = context.MyEntities.Select(entity => entity.Id).Contains(id);

    var results = query.ToList();
    ...
}
```

在此示例中，每次执行此查询时使用的查询 id 的值将编译到新的计划。

中的 Skip 和 Take 执行分页时使用特定注意。 在 EF6 中这些方法具有有效地使缓存的查询计划可重复使用，因为 EF 可以捕获变量传递给这些方法，并将它们转换成 SQLparameters lambda 重载。 这也有助于使缓存更整洁，因为每个查询使用 Skip 和 Take 的不同常量否则会收到其自己的查询计划缓存条目。

请考虑下面的代码，这并非最佳，但仅用来将此类查询：

``` csharp
var customers = context.Customers.OrderBy(c => c.LastName);
for (var i = 0; i < count; ++i)
{
    var currentCustomer = customers.Skip(i).FirstOrDefault();
    ProcessCustomer(currentCustomer);
}
```

这一相同代码的速度更快版本将涉及使用 lambda 调用跳过：

``` csharp
var customers = context.Customers.OrderBy(c => c.LastName);
for (var i = 0; i \< count; ++i)
{
    var currentCustomer = customers.Skip(() => i).FirstOrDefault();
    ProcessCustomer(currentCustomer);
}
```

第二个代码片段可能运行更快达 11%，因为每次运行查询时，这可以节省 CPU 时间，可避免污染查询缓存使用相同的查询计划。 此外，由于要跳过该参数是在闭包中的代码可能也类似以下形式现在：

``` csharp
var i = 0;
var skippyCustomers = context.Customers.OrderBy(c => c.LastName).Skip(() => i);
for (; i < count; ++i)
{
    var currentCustomer = skippyCustomers.FirstOrDefault();
    ProcessCustomer(currentCustomer);
}
```

### <a name="43-using-the-properties-of-a-non-mapped-object"></a>4.3 使用非映射对象的属性

当查询使用非映射对象类型的属性，因为将不获取缓存参数则执行查询。 例如：

``` csharp
using (var context = new MyContext())
{
    var myObject = new NonMappedType();

    var query = from entity in context.MyEntities
                where entity.Name.StartsWith(myObject.MyProperty)
                select entity;

   var results = query.ToList();
    ...
}
```

在此示例中，假定类 NonMappedType 不是实体模型的一部分。 此查询可以轻易改为使用非映射类型并改为使用本地变量作为查询的参数：

``` csharp
using (var context = new MyContext())
{
    var myObject = new NonMappedType();
    var myValue = myObject.MyProperty;
    var query = from entity in context.MyEntities
                where entity.Name.StartsWith(myValue)
                select entity;

    var results = query.ToList();
    ...
}
```

在这种情况下，查询将能够获取缓存，并将从查询计划缓存中受益。

### <a name="44-linking-to-queries-that-require-recompiling"></a>4.4 链接到要求重新编译的查询

与上述示例相同，如果您依赖于需要进行重新编译的查询的第二个查询整个第二个查询将还重新编译。 下面是示例演示了此方案中：

``` csharp
int[] ids = new int[10000];
...
using (var context = new MyContext())
{
    var firstQuery = from entity in context.MyEntities
                        where ids.Contains(entity.Id)
                        select entity;

    var secondQuery = from entity in context.MyEntities
                        where firstQuery.Any(otherEntity => otherEntity.Id == entity.Id)
                        select entity;

    var results = secondQuery.ToList();
    ...
}
```

该示例为泛型，但它演示了如何将链接到 firstQuery 导致 secondQuery 无法获取缓存。 如果 firstQuery 不需要重新编译的查询，然后 secondQuery 将已缓存。

## <a name="5-notracking-queries"></a>5 NoTracking 查询

### <a name="51-disabling-change-tracking-to-reduce-state-management-overhead"></a>5.1 禁用更改跟踪来减少管理开销状态

如果您在只读的情况下，并且想要避免加载到 ObjectStateManager 对象的开销，您可以发出"不跟踪"的查询。  可以在查询级别禁用更改跟踪。

但请注意，通过禁用更改跟踪您会有效地关闭对象缓存。 在查询实体时，我们不能通过从 ObjectStateManager 中拉取之前具体化查询结果中跳过具体化。 如果重复查询上相同的上下文相同的实体，您实际上可能会看到从启用更改跟踪中受益的性能。

在查询时使用 ObjectContext，ObjectQuery 和 ObjectSet 实例将记住 MergeOption 设置，并对其组成的查询将继承父查询有效 MergeOption 后。 在使用 DbContext，可以通过对 DbSet 调用 AsNoTracking() 修饰符禁用跟踪。

#### <a name="511-disabling-change-tracking-for-a-query-when-using-dbcontext"></a>5.1.1 禁用更改跟踪的查询使用 DbContext 时

可以通过链接到查询中的 AsNoTracking() 方法的调用来切换到 NoTracking 的查询模式。 与不同的 ObjectQuery，DbContext API 中的 DbSet 和 DbQuery 类不为 mergeoption，然后包含可变属性。

``` csharp
    var productsForCategory = from p in context.Products.AsNoTracking()
                                where p.Category.CategoryName == selectedCategory
                                select p;


```

#### <a name="512-disabling-change-tracking-at-the-query-level-using-objectcontext"></a>5.1.2 禁用更改跟踪在查询级别使用 ObjectContext

``` csharp
    var productsForCategory = from p in context.Products
                                where p.Category.CategoryName == selectedCategory
                                select p;

    ((ObjectQuery)productsForCategory).MergeOption = MergeOption.NoTracking;
```

#### <a name="513-disabling-change-tracking-for-an-entire-entity-set-using-objectcontext"></a>5.1.3 禁用更改跟踪的整个实体集使用 ObjectContext

``` csharp
    context.Products.MergeOption = MergeOption.NoTracking;

    var productsForCategory = from p in context.Products
                                where p.Category.CategoryName == selectedCategory
                                select p;
```

### <a name="52test-metrics-demonstrating-the-performance-benefit-of-notracking-queries"></a>5.2 测试演示 NoTracking 查询的性能优势的度量值

在此测试我们将介绍但代价是通过比较跟踪对 Navision 模型 NoTracking 查询填充 ObjectStateManager。 请参阅的附录 Navision 模型和执行的查询的类型的说明。 在此测试中，我们循环访问查询的列表，并执行每个一次。 我们已使用默认合并选项的"仅附加"运行测试、 一次使用 NoTracking 查询和一次的两种变体。 我们运行 3 次的每个变体，并需要在运行的平均值。 测试之间我们清除 SQL Server 上的查询缓存，并将 tempdb 收缩通过运行以下命令：

1.  DBCC DROPCLEANBUFFERS
2.  DBCC FREEPROCCACHE
3.  DBCC SHRINKDATABASE (tempdb，0)

测试结果，中间值的 3 个运行：

|                        | 不跟踪-工作集 | 无跟踪-时间 | 仅 – 追加工作集 | 追加专用 – 时间 |
|:-----------------------|:--------------------------|:-------------------|:--------------------------|:-------------------|
| **实体框架 5** | 460361728                 | 1163536 ms         | 596545536                 | 1273042 ms         |
| **Entity Framework 6** | 647127040                 | 190228 ms          | 832798720                 | 195521 ms          |

实体框架 5 将具有较小的内存需求量运行结束时比 Entity Framework 6。 使用 Entity Framework 6 的其他内存是额外的内存结构和启用新功能和更好的性能的代码的结果。

使用 ObjectStateManager 时，也是有占用的内存中有明显差异。 当跟踪我们从数据库具体化的所有实体，实体框架 5 30%的增加其内存占用。 执行此操作时，实体框架 6 通过 28%增加其内存占用。

根据时间，Entity Framework 6 优于 Entity Framework 5，在此测试中由大边距。 实体框架 6 完成大约 16%的时间由 Entity Framework 5 中的测试。 此外，Entity Framework 5 需要 9%的时间才能完成时正在使用 ObjectStateManager。 在比较中，Entity Framework 6 使用多的时间来使用 ObjectStateManager 时的 3%。

## <a name="6-query-execution-options"></a>6 个查询执行选项

实体框架提供几种不同方式查询。 我们将看一看以下选项，比较各自的优缺点，并检查其性能特征：

-   LINQ to Entities。
-   不是跟踪，LINQ 到实体。
-   ObjectQuery 上的 SQL 的实体。
-   实体 EntityCommand 上的 SQL。
-   ExecuteStoreQuery。
-   SqlQuery。
-   CompiledQuery。

### <a name="61-linq-to-entities-queries"></a>6.1 LINQ to Entities 查询

``` csharp
var q = context.Products.Where(p => p.Category.CategoryName == "Beverages");
```

**专业人员**

-   适用于 CUD 操作。
-   完全实例化的对象。
-   最简单的方法编写使用内置于编程语言的语法。
-   良好的性能。

**缺点**

-   某些技术限制，如：
    -   OUTER JOIN 查询使用 DefaultIfEmpty 模式会导致更复杂查询的速度比在实体 SQL 的简单 OUTER JOIN 语句。
    -   仍不能使用 LIKE 与常规模式匹配。

### <a name="62-no-tracking-linq-to-entities-queries"></a>6.2 无跟踪 LINQ to Entities 查询

当上下文派生 ObjectContext:

``` csharp
context.Products.MergeOption = MergeOption.NoTracking;
var q = context.Products.Where(p => p.Category.CategoryName == "Beverages");
```

当上下文派生 DbContext:

``` csharp
var q = context.Products.AsNoTracking()
                        .Where(p => p.Category.CategoryName == "Beverages");
```

**专业人员**

-   通过定期的 LINQ 查询的改进了的性能。
-   完全实例化的对象。
-   最简单的方法编写使用内置于编程语言的语法。

**缺点**

-   不适用于 CUD 操作。
-   某些技术限制，如：
    -   OUTER JOIN 查询使用 DefaultIfEmpty 模式会导致更复杂查询的速度比在实体 SQL 的简单 OUTER JOIN 语句。
    -   仍不能使用 LIKE 与常规模式匹配。

请注意，即使未指定 NoTracking 不会跟踪项目标量属性的查询。 例如：

``` csharp
var q = context.Products.Where(p => p.Category.CategoryName == "Beverages").Select(p => new { p.ProductName });
```

此特定查询未显式指定正在 NoTracking，但由于它不具体化不跟踪具有已知的对象状态管理器然后实例化的结果的类型。

### <a name="63-entity-sql-over-an-objectquery"></a>6.3 实体 ObjectQuery 上的 SQL

``` csharp
ObjectQuery<Product> products = context.Products.Where("it.Category.CategoryName = 'Beverages'");
```

**专业人员**

-   适用于 CUD 操作。
-   完全实例化的对象。
-   支持查询计划缓存。

**缺点**

-   涉及到文本的查询字符串，它们是更容易出现用户错误比内置于语言的查询构造。

### <a name="64-entity-sql-over-an-entity-command"></a>6.4 实体的实体命令上的 SQL

``` csharp
EntityCommand cmd = eConn.CreateCommand();
cmd.CommandText = "Select p From NorthwindEntities.Products As p Where p.Category.CategoryName = 'Beverages'";

using (EntityDataReader reader = cmd.ExecuteReader(CommandBehavior.SequentialAccess))
{
    while (reader.Read())
    {
        // manually 'materialize' the product
    }
}
```

**专业人员**

-   支持查询计划缓存在.NET 4.0 （计划缓存的.NET 4.5 中的所有其他查询类型支持）。

**缺点**

-   涉及到文本的查询字符串，它们是更容易出现用户错误比内置于语言的查询构造。
-   不适用于 CUD 操作。
-   结果自动未具体化，并且必须从数据读取器读取。

### <a name="65-sqlquery-and-executestorequery"></a>6.5 SqlQuery 和 ExecuteStoreQuery

在数据库上的 SqlQuery:

``` csharp
// use this to obtain entities and not track them
var q1 = context.Database.SqlQuery<Product>("select * from products");
```

DbSet 的 SqlQuery:

``` csharp
// use this to obtain entities and have them tracked
var q2 = context.Products.SqlQuery("select * from products");
```

ExecyteStoreQuery:

``` csharp
var beverages = context.ExecuteStoreQuery<Product>(
@"     SELECT        P.ProductID, P.ProductName, P.SupplierID, P.CategoryID, P.QuantityPerUnit, P.UnitPrice, P.UnitsInStock, P.UnitsOnOrder, P.ReorderLevel, P.Discontinued, P.DiscontinuedDate
       FROM            Products AS P INNER JOIN Categories AS C ON P.CategoryID = C.CategoryID
       WHERE        (C.CategoryName = 'Beverages')"
);
```

**专业人员**

-   通常最快的性能由于计划编译器，则跳过。
-   完全实例化的对象。
-   适用于 CUD 操作中使用的 DbSet。

**缺点**

-   查询是文本且容易出错。
-   通过使用存储区语义而不概念语义情况下，查询绑定到特定的后端。
-   当存在继承时，精心设计的查询所需的请求的类型的映射条件。

### <a name="66-compiledquery"></a>6.6 CompiledQuery

``` csharp
private static readonly Func<NorthwindEntities, string, IQueryable<Product>> productsForCategoryCQ = CompiledQuery.Compile(
    (NorthwindEntities context, string categoryName) =>
        context.Products.Where(p => p.Category.CategoryName == categoryName)
        );
…
var q = context.InvokeProductsForCategoryCQ("Beverages");
```

**专业人员**

-   通过定期的 LINQ 查询提供最多 7%的性能改善。
-   完全实例化的对象。
-   适用于 CUD 操作。

**缺点**

-   增加复杂性和编程的开销。
-   在撰写基于已编译查询时，性能改进会丢失。
-   某些 LINQ 查询不能被编写为 CompiledQuery-例如，匿名类型的投影。

### <a name="67-performance-comparison-of-different-query-options"></a>6.7 性能比较的不同的查询选项

已不定时上下文创建的简单 microbenchmarks 已放置到测试。 查询的非缓存在受控环境中的实体集 5000 倍，我们测量。 这些数字是要执行但出现警告： 它们不反映实际数字生成由应用程序，而它们是非常准确多少不同的查询选项进行比较时没有性能差异的度量值同类对象，不包括创建新的上下文的成本。

| EF  | 测试                                 | 时间 （毫秒） | 内存   |
|:----|:-------------------------------------|:----------|:---------|
| EF5 | ObjectContext ESQL                   | 2414      | 38801408 |
| EF5 | ObjectContext Linq 查询             | 2692      | 38277120 |
| EF5 | DbContext Linq 查询无跟踪     | 2818      | 41840640 |
| EF5 | DbContext Linq 查询                 | 2930      | 41771008 |
| EF5 | ObjectContext Linq 查询无跟踪 | 3013      | 38412288 |
|     |                                      |           |          |
| EF6 | ObjectContext ESQL                   | 2059      | 46039040 |
| EF6 | ObjectContext Linq 查询             | 3074      | 45248512 |
| EF6 | DbContext Linq 查询无跟踪     | 3125      | 47575040 |
| EF6 | DbContext Linq 查询                 | 3420      | 47652864 |
| EF6 | ObjectContext Linq 查询无跟踪 | 3593      | 45260800 |

![EF5 微基准，5000 热迭代](~/ef6/media/ef5micro5000warm.png)

![EF6 微基准，5000 热迭代](~/ef6/media/ef6micro5000warm.png)

Microbenchmarks 都对代码中的微小变化非常敏感。 在这种情况下，Entity Framework 5 的成本和 Entity Framework 6 之间的差异是由于的加法[拦截](~/ef6/fundamentals/logging-and-interception.md)并[事务改进](~/ef6/saving/transactions.md)。 这些 microbenchmarks 数字，但是，是放大的构想嵌入的实体框架的作用非常小片段。 从 Entity Framework 5 升级到 Entity Framework 6 时，实际方案的热查询看不到性能回归。

若要比较不同的查询选项的实际性能，我们创建了 5 个单独的测试变体，其中我们使用不同的查询选项来选择所有产品类别名称是"饮料"。 每次迭代包括创建上下文的成本和将返回所有实体具体化的成本。 10 次迭代执行已超时的 1000年次迭代的总和之前运行 untimed。 显示的结果是从每个测试的 5 个运行的中间值运行。 有关详细信息，请参阅附录 B，其中包括测试的代码。

| EF  | 测试                                        | 时间 （毫秒） | 内存   |
|:----|:--------------------------------------------|:----------|:---------|
| EF5 | ObjectContext 实体命令                | 621       | 39350272 |
| EF5 | DbContext 上数据库的 Sql 查询             | 825       | 37519360 |
| EF5 | ObjectContext 存储查询                   | 878       | 39460864 |
| EF5 | ObjectContext Linq 查询无跟踪        | 969       | 38293504 |
| EF5 | 使用对象查询的 ObjectContext 实体 Sql | 1089      | 38981632 |
| EF5 | ObjectContext 已编译的查询                | 1099      | 38682624 |
| EF5 | ObjectContext Linq 查询                    | 1152      | 38178816 |
| EF5 | DbContext Linq 查询无跟踪            | 1208      | 41803776 |
| EF5 | DbSet 的 DbContext Sql 查询                | 1414      | 37982208 |
| EF5 | DbContext Linq 查询                        | 1574      | 41738240 |
|     |                                             |           |          |
| EF6 | ObjectContext 实体命令                | 480       | 47247360 |
| EF6 | ObjectContext 存储查询                   | 493       | 46739456 |
| EF6 | DbContext 上数据库的 Sql 查询             | 614       | 41607168 |
| EF6 | ObjectContext Linq 查询无跟踪        | 684       | 46333952 |
| EF6 | 使用对象查询的 ObjectContext 实体 Sql | 767       | 48865280 |
| EF6 | ObjectContext 已编译的查询                | 788       | 48467968 |
| EF6 | DbContext Linq 查询无跟踪            | 878       | 47554560 |
| EF6 | ObjectContext Linq 查询                    | 953       | 47632384 |
| EF6 | DbSet 的 DbContext Sql 查询                | 1023      | 41992192 |
| EF6 | DbContext Linq 查询                        | 1290      | 47529984 |


![EF5 热查询 1000年次迭代](~/ef6/media/ef5warmquery1000.png)

![EF6 热查询 1000年次迭代](~/ef6/media/ef6warmquery1000.png)

> [!NOTE]
> 出于完整性的考虑，我们将包括 EntityCommand 执行 Entity SQL 查询的一种变体。 但是，对于这种查询未具体化结果，因为比较并不一定是苹果苹果。 测试包括具体化尝试使较公平地进行比较的近似值。

在端到端此种情况下，Entity Framework 6 优于 Entity Framework 5 上的堆栈，包括得更亮 DbContext 初始化和更快的 MetadataCollection 多个部分由性能改进引起&lt;T&gt;查找。

## <a name="7-design-time-performance-considerations"></a>7 设计时间性能注意事项

### <a name="71-inheritance-strategies"></a>7.1 继承策略

使用实体框架时的另一个性能注意事项是你使用的继承策略。 实体框架支持 3 种基本类型的继承和它们二者的组合：

-   每个层次结构 (TPH) – 其中每个继承设置映射到表中以指示要表示的行中的层次结构中的特定类型的鉴别器列的表。
-   表每个类型 (TPT) – 其中每个类型具有其自己的表中该数据库。子表只定义父表不包含的列。
-   表每个类 (TPC) – 其中每个类型具有自己的完整表中该数据库。子表定义所有字段，包括那些在父类型中定义。

如果您的模型使用 TPT 继承，生成的查询将比使用其他继承策略，这可能导致存储区上的执行时间较长上生成更复杂。  它通常需要花费更长，以通过 TPT 模型生成查询并具体化的对象。

请参阅"性能注意事项时在实体框架中使用 （每种类型的表） TPT 继承"MSDN 博客文章： \<http://blogs.msdn.com/b/adonet/archive/2010/08/17/performance-considerations-when-using-tpt-table-per-type-inheritance-in-the-entity-framework.aspx>。

#### <a name="711-avoiding-tpt-in-model-first-or-code-first-applications"></a>7.1.1 Model First 或 Code First 应用程序中避免 TPT

通过具有 TPT 架构的现有数据库创建模型，你没有在很多选项。 但在创建时使用 Model First 或 Code First 的应用程序，应避免性能问题的 TPT 继承。

当在实体设计器向导中使用模型优先时，会获得 TPT 任何继承模型中。 如果你想要切换到使用模型优先的 TPH 继承策略，可以使用"Entity Designer Database Generation Power Pack"可从 Visual Studio 库 ( \<http://visualstudiogallery.msdn.microsoft.com/df3541c3-d833-4b65-b942-989e7ec74c87/>)。

在使用 Code First 配置继承模型的映射，EF 将使用 TPH 默认情况下，因此继承层次结构中的所有实体将都映射到同一个表。 请参阅 MSDN 杂志 》 中的"代码第一个中实体 Framework4.1"项目的"映射的 Fluent API"一节 ( [http://msdn.microsoft.com/magazine/hh126815.aspx](https://msdn.microsoft.com/magazine/hh126815.aspx)) 的更多详细信息。

### <a name="72-upgrading-from-ef4-to-improve-model-generation-time"></a>7.2 升级从 EF4 以提高模型生成时间

安装 Visual Studio 2010 SP1 时，用于生成存储层 (SSDL) 的算法的特定于 SQL Server 的改进是模型的在 Entity Framework 5 和 6，并作为对 Entity Framework 4 的更新可用。 以下测试结果时生成非常大的模型，在这种情况下 Navision 模型演示改进。 更多详细信息，请参阅附录 C。

该模型包含 1005年实体集和 4227 关联集。

| 配置                              | 使用时间的细目                                                                                                                                               |
|:-------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Visual Studio 2010 中，实体框架 4     | SSDL 生成： 2 小时 27 分钟 <br/> 映射生成： 1 秒 <br/> CSDL 生成： 1 秒 <br/> ObjectLayer 生成： 1 秒 <br/> 视图生成： 2 小时 14 分钟 |
| Visual Studio 2010 SP1 中，实体框架 4 | SSDL 生成： 1 秒 <br/> 映射生成： 1 秒 <br/> CSDL 生成： 1 秒 <br/> ObjectLayer 生成： 1 秒 <br/> 视图生成： 1 小时 53 分钟   |
| Visual Studio 2013 中，实体框架 5     | SSDL 生成： 1 秒 <br/> 映射生成： 1 秒 <br/> CSDL 生成： 1 秒 <br/> ObjectLayer 生成： 1 秒 <br/> 视图生成： 65 分钟    |
| Visual Studio 2013 中，实体框架 6     | SSDL 生成： 1 秒 <br/> 映射生成： 1 秒 <br/> CSDL 生成： 1 秒 <br/> ObjectLayer 生成： 1 秒 <br/> 视图生成： 28 秒。   |


值得注意是，生成 SSDL 时, 负载几乎完全所用的 SQL 服务器上，等待客户端开发计算机时，结果才会返回从服务器空闲。 Dba 应特别感谢此项改进。 它也是值得注意成本模型生成的实质上是整个现在采用视图生成中的位置。

### <a name="73-splitting-large-models-with-database-first-and-model-first"></a>7.3 首先拆分与数据库的大型模型和模型优先

随着模型大小的增加，设计器图面变得混乱且难以使用。 我们通常认为具有 300 多个实体太大而无法有效地使用设计器的模型。 下面的博客文章介绍了有关拆分大型模型的多个选项： \<http://blogs.msdn.com/b/adonet/archive/2008/11/25/working-with-large-models-in-entity-framework-part-2.aspx>。

开机自检编写实体框架的第一个版本，但步骤仍然适用。

### <a name="74-performance-considerations-with-the-entity-data-source-control"></a>7.4 实体数据源控件的性能注意事项

我们已经看到在多线程的性能和压力测试的情况下，使用 EntityDataSource 控件的 web 应用程序的性能会降低显著。 根本原因是 EntityDataSource Web 应用程序，以发现要用作实体的类型引用的程序集上反复调用 MetadataWorkspace.LoadFromAssembly。

解决方案是设置的 EntityDataSource ContextTypeName 为派生的 ObjectContext 类的类型名称。 这将关闭扫描的实体类型的所有引用程序集的机制。

设置 ContextTypeName 字段还可以防止其中 EntityDataSource.NET 4.0 中的时，会引发出现 ReflectionTypeLoadException 它不能从通过反射程序集加载的类型的功能问题。 在.NET 4.5 中已修复此问题。

### <a name="75-poco-entities-and-change-tracking-proxies"></a>7.5 POCO 实体和更改跟踪代理

实体框架，可自定义数据类与数据模型一起使用而无需对数据类本身进行任何修改。 这意味着可以将“纯旧式”CLR 对象 (POCO)（例如，现有的域对象）与数据模型一起使用。 这些 POCO 数据类 （也称为持久性未知对象），其映射到实体数据模型中定义的支持的大多数相同的查询、 插入、 更新，和删除实体类型由 Entity Data Model 工具生成的行为。

实体框架还可以创建派生自 POCO 类型，在你想要启用功能，例如延迟加载和自动更改跟踪 POCO 实体时使用的代理类。 POCO 类必须满足某些要求允许实体框架才能使用代理，如下所述： [http://msdn.microsoft.com/library/dd468057.aspx](https://msdn.microsoft.com/library/dd468057.aspx)。

机会跟踪代理的任何实体的属性使实体框架知道你的实体的实际状态的时间已发生更改时，其值每次将通知对象状态管理器。 这是通过将通知事件添加到您的属性的 setter 方法的正文，并让对象状态管理器处理此类事件。 请注意，创建代理实体将通常要比创建非代理 POCO 实体由于添加了一组事件 Entity Framework 创建的昂贵得多。

如果 POCO 实体不具有更改跟踪代理，通过比较针对以前保存的状态的副本实体的内容来发现更改。 在您的上下文中包含多个实体或实体具有极大量的属性，即使它们没有任何更改上次比较以来此深入比较将成为一个漫长的过程。

总之： 需支付费用的性能下降时创建的更改跟踪代理，但是更改跟踪可帮助您加快完成此更改检测过程，当你的实体具有许多属性或您的模型中有多个实体时。 对于较少的属性，其中的实体不会增长过多的实体，具有更改跟踪代理不可能的很大的收益。

## <a name="8-loading-related-entities"></a>8 加载相关实体

### <a name="81-lazy-loading-vs-eager-loading"></a>8.1 延迟加载 vs。预先加载

实体框架提供了几种方式将加载到目标实体相关的实体。 例如，当查询产品时，有多种相关的订单将被加载到对象状态管理器。 从性能角度看，最大问题来加载相关的实体时，请考虑将能是否使用延迟加载或预先加载。

在使用预先加载时，你的目标实体集以及加载相关的实体。 在查询中使用一个 Include 语句以指示哪些相关的实体，你想要导入。

在使用延迟加载，您的初始查询只会使目标实体集中。 但是，只要访问导航属性，对要加载相关的实体的存储区发出另一个查询。

加载实体后，任何进一步查询的实体将加载它直接从对象状态管理器中，无论您使用延迟加载或预先加载。

### <a name="82-how-to-choose-between-lazy-loading-and-eager-loading"></a>8.2 如何延迟加载和预先加载之间进行选择

重要的一点是了解延迟加载和预先加载之间的差异，以便您可以为应用程序做出正确选择。 这将帮助你评估对与单个请求，可能包含大型有效负载的数据库的多个请求之间的权衡。 它可能需要在其他部分中使用你的应用程序的某些部分中预先加载和延迟加载。

作为在后台发生的情况的示例，假设您想要查询的客户居住在英国和其订单计数。

**使用预先加载**

``` csharp
using (NorthwindEntities context = new NorthwindEntities())
{
    var ukCustomers = context.Customers.Include(c => c.Orders).Where(c => c.Address.Country == "UK");
    var chosenCustomer = AskUserToPickCustomer(ukCustomers);
    Console.WriteLine("Customer Id: {0} has {1} orders", customer.CustomerID, customer.Orders.Count);
}
```

**使用延迟加载**

``` csharp
using (NorthwindEntities context = new NorthwindEntities())
{
    context.ContextOptions.LazyLoadingEnabled = true;

    //Notice that the Include method call is missing in the query
    var ukCustomers = context.Customers.Where(c => c.Address.Country == "UK");

    var chosenCustomer = AskUserToPickCustomer(ukCustomers);
    Console.WriteLine("Customer Id: {0} has {1} orders", customer.CustomerID, customer.Orders.Count);
}
```

在使用预先加载时，将发布单个查询返回所有客户的所有订单。 存储命令如下所示：

``` SQL
SELECT
[Project1].[C1] AS [C1],
[Project1].[CustomerID] AS [CustomerID],
[Project1].[CompanyName] AS [CompanyName],
[Project1].[ContactName] AS [ContactName],
[Project1].[ContactTitle] AS [ContactTitle],
[Project1].[Address] AS [Address],
[Project1].[City] AS [City],
[Project1].[Region] AS [Region],
[Project1].[PostalCode] AS [PostalCode],
[Project1].[Country] AS [Country],
[Project1].[Phone] AS [Phone],
[Project1].[Fax] AS [Fax],
[Project1].[C2] AS [C2],
[Project1].[OrderID] AS [OrderID],
[Project1].[CustomerID1] AS [CustomerID1],
[Project1].[EmployeeID] AS [EmployeeID],
[Project1].[OrderDate] AS [OrderDate],
[Project1].[RequiredDate] AS [RequiredDate],
[Project1].[ShippedDate] AS [ShippedDate],
[Project1].[ShipVia] AS [ShipVia],
[Project1].[Freight] AS [Freight],
[Project1].[ShipName] AS [ShipName],
[Project1].[ShipAddress] AS [ShipAddress],
[Project1].[ShipCity] AS [ShipCity],
[Project1].[ShipRegion] AS [ShipRegion],
[Project1].[ShipPostalCode] AS [ShipPostalCode],
[Project1].[ShipCountry] AS [ShipCountry]
FROM ( SELECT
      [Extent1].[CustomerID] AS [CustomerID],
       [Extent1].[CompanyName] AS [CompanyName],
       [Extent1].[ContactName] AS [ContactName],
       [Extent1].[ContactTitle] AS [ContactTitle],
       [Extent1].[Address] AS [Address],
       [Extent1].[City] AS [City],
       [Extent1].[Region] AS [Region],
       [Extent1].[PostalCode] AS [PostalCode],
       [Extent1].[Country] AS [Country],
       [Extent1].[Phone] AS [Phone],
       [Extent1].[Fax] AS [Fax],
      1 AS [C1],
       [Extent2].[OrderID] AS [OrderID],
       [Extent2].[CustomerID] AS [CustomerID1],
       [Extent2].[EmployeeID] AS [EmployeeID],
       [Extent2].[OrderDate] AS [OrderDate],
       [Extent2].[RequiredDate] AS [RequiredDate],
       [Extent2].[ShippedDate] AS [ShippedDate],
       [Extent2].[ShipVia] AS [ShipVia],
       [Extent2].[Freight] AS [Freight],
       [Extent2].[ShipName] AS [ShipName],
       [Extent2].[ShipAddress] AS [ShipAddress],
       [Extent2].[ShipCity] AS [ShipCity],
       [Extent2].[ShipRegion] AS [ShipRegion],
       [Extent2].[ShipPostalCode] AS [ShipPostalCode],
       [Extent2].[ShipCountry] AS [ShipCountry],
      CASE WHEN ([Extent2].[OrderID] IS NULL) THEN CAST(NULL AS int) ELSE 1 END AS [C2]
      FROM  [dbo].[Customers] AS [Extent1]
      LEFT OUTER JOIN [dbo].[Orders] AS [Extent2] ON [Extent1].[CustomerID] = [Extent2].[CustomerID]
      WHERE N'UK' = [Extent1].[Country]
)  AS [Project1]
ORDER BY [Project1].[CustomerID] ASC, [Project1].[C2] ASC
```

在使用延迟加载，您将最初发出以下查询：

``` SQL
SELECT
[Extent1].[CustomerID] AS [CustomerID],
[Extent1].[CompanyName] AS [CompanyName],
[Extent1].[ContactName] AS [ContactName],
[Extent1].[ContactTitle] AS [ContactTitle],
[Extent1].[Address] AS [Address],
[Extent1].[City] AS [City],
[Extent1].[Region] AS [Region],
[Extent1].[PostalCode] AS [PostalCode],
[Extent1].[Country] AS [Country],
[Extent1].[Phone] AS [Phone],
[Extent1].[Fax] AS [Fax]
FROM [dbo].[Customers] AS [Extent1]
WHERE N'UK' = [Extent1].[Country]
```

和访问客户的订单导航属性每次对应用商店发出另一个查询如下所示：

``` SQL
exec sp_executesql N'SELECT
[Extent1].[OrderID] AS [OrderID],
[Extent1].[CustomerID] AS [CustomerID],
[Extent1].[EmployeeID] AS [EmployeeID],
[Extent1].[OrderDate] AS [OrderDate],
[Extent1].[RequiredDate] AS [RequiredDate],
[Extent1].[ShippedDate] AS [ShippedDate],
[Extent1].[ShipVia] AS [ShipVia],
[Extent1].[Freight] AS [Freight],
[Extent1].[ShipName] AS [ShipName],
[Extent1].[ShipAddress] AS [ShipAddress],
[Extent1].[ShipCity] AS [ShipCity],
[Extent1].[ShipRegion] AS [ShipRegion],
[Extent1].[ShipPostalCode] AS [ShipPostalCode],
[Extent1].[ShipCountry] AS [ShipCountry]
FROM [dbo].[Orders] AS [Extent1]
WHERE [Extent1].[CustomerID] = @EntityKeyValue1',N'@EntityKeyValue1 nchar(5)',@EntityKeyValue1=N'AROUT'
```

有关详细信息，请参阅[加载相关对象](https://msdn.microsoft.com/library/bb896272.aspx)。

#### <a name="821-lazy-loading-versus-eager-loading-cheat-sheet"></a>8.2.1 与预先加载备忘单延迟加载

就没有通用于选择预先加载与延迟加载。 第一次尝试了解这两种策略，这样您可以为也明智的决策; 之间的差异此外，请考虑到任何以下情况下是否适合你的代码：

| 方案                                                                    | 我们建议                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|:----------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 若要从提取实体访问多个导航属性需要吗？ | **不**-这两个选项可能会执行操作。 但是，如果将您的查询的有效负载不是太大，则可能会遇到性能优势，使用预先加载，因为它将需要更少的网络往返行程具体化对象。 <br/> <br/> **是**-如果需要从实体来访问多个导航属性，将执行操作，通过使用多个语句在查询中包括与预先加载。 更多实体包含，越大你的查询将返回的有效负载。 一旦在查询中包括三个或多个实体，请考虑切换为延迟加载。 |
| 您是否知道将完全在运行时间需要哪些数据？                   | **不**的延迟加载会更好。 否则，可能最终会查询数据，则不需。 <br/> <br/> **是**-预先加载可能是最好的方法; 它将帮助更快地加载整个集。 如果您的查询需要提取非常大量的数据，并且这变得太慢，然后尝试加载改为 Lazy。                                                                                                                                                                                                                                                       |
| 姑且不论您的数据库执行你的代码？ （增加了的网络延迟）  | **不**-时的网络延迟不是问题，使用延迟加载可以简化你的代码。 请记住你的应用程序拓扑可能会更改，因此不需要数据库邻近理所当然的。 <br/> <br/> **是**-网络问题时，仅可以决定更好地适合您的方案。 通常将预先加载是更好，因为它需要往返次数较少。                                                                                                                                                                                                      |


#### <a name="822-performance-concerns-with-multiple-includes"></a>8.2.2 使用多个包括性能问题

如果我们听到涉及服务器响应时间问题的性能问题，问题的原因通常是具有多个包含语句的查询。 虽然在查询中包括相关的实体很强大，务必了解在后台发生的事情。

它在其中可通过我们内部计划编译器生成的存储命令需要具有多个包含语句的查询相对较长时间。 大多数此时间花费在尝试优化生成的查询上。 将生成的存储命令将包含 Outer Join 或 Union 的每个包含，具体取决于您的映射。 此类查询将从您的数据库中单个有效负载，将 acerbate 带宽问题，尤其是当存在很多 （例如，在使用多个级别的 Include 遍历时的有效负载中的冗余引入大已连接的关系图中的一个多方向关联）。

你可以检查您的查询，来访问基础 TSQL 查询使用 ToTraceString 和 SQL Server Management Studio，若要查看的有效负载大小中执行存储命令，从而返回过大负载的情况。 在这种情况下，您可以尝试减少直接在查询中的 Include 语句数引入所需的数据。 或者，您可能能够将查询分解为较小序列的子查询，例如：

**之前的重大查询：**

``` csharp
using (NorthwindEntities context = new NorthwindEntities())
{
    var customers = from c in context.Customers.Include(c => c.Orders)
                    where c.LastName.StartsWith(lastNameParameter)
                    select c;

    foreach (Customer customer in customers)
    {
        ...
    }
}
```

**之后的重大查询：**

``` csharp
using (NorthwindEntities context = new NorthwindEntities())
{
    var orders = from o in context.Orders
                 where o.Customer.LastName.StartsWith(lastNameParameter)
                 select o;

    orders.Load();

    var customers = from c in context.Customers
                    where c.LastName.StartsWith(lastNameParameter)
                    select c;

    foreach (Customer customer in customers)
    {
        ...
    }
}
```

这将仅适用于跟踪的查询，因为我们将使用的上下文具有以自动执行标识解析和关联链接地址信息的功能。

与延迟加载后，其不利的一面将更多的较小的负载的查询。 您还可以使用投影的个别属性的显式从每个实体中，选择所需的数据，但您将不会加载实体在这种情况下，和不支持更新。

#### <a name="823-workaround-to-get-lazy-loading-of-properties"></a>8.2.3 解决方法，以获取延迟加载的属性

实体框架当前不支持延迟加载的标量或复杂属性。 但是，在必须包含一个大型对象，如 BLOB 的表的情况下，您可以使用表拆分将大型属性分成单独的实体。 例如，假设有一个包括 varbinary photo 列的产品表。 如果不经常需要访问此属性在查询中的，可以使用表拆分，以使在仅部分通常需要的实体。 在明确需要时，将仅加载表示产品照片的实体。

演示如何启用表拆分的良好资源是方便 Gil Fink 的 《 表拆分中 Entity Framework 》 博客文章： \<http://blogs.microsoft.co.il/blogs/gilf/archive/2009/10/13/table-splitting-in-entity-framework.aspx>。

## <a name="9-other-considerations"></a>9 其他注意事项

### <a name="91-server-garbage-collection"></a>9.1 服务器垃圾回收

某些用户可能会遇到限制的并行度，垃圾回收器未正确配置时收到意外的资源争用。 每次在多线程方案中，使用 EF 或任何应用程序中的类似于服务器端的系统，请确保启用服务器垃圾回收。 通过在应用程序配置文件中的简单设置完成此操作：

``` xml
<?xmlversion="1.0" encoding="utf-8" ?>
<configuration>
        <runtime>
               <gcServer enabled="true" />
        </runtime>
</configuration>
```

这应减少线程争用并提高吞吐量最多 30 %cpu 饱和方案中。 概括地说，您始终应测试您的应用程序的行为方式使用经典的垃圾回收 （，更好地适用于 UI 和客户端端方案） 以及服务器垃圾回收。

### <a name="92-autodetectchanges"></a>9.2 AutoDetectChanges

前面曾提到，实体框架可能会显示性能问题时对象缓存中有多个实体。 某些操作，如添加、 删除、 查找、 条目和 SaveChanges，触发调用 DetectChanges，这可能会占用大量 CPU 基于对象缓存已成为多大。 这样做的原因是对象缓存和对象状态管理器尝试将保持为同步尽可能执行到上下文，以便生成的数据保证可正确了很多情况下每个操作。

它通常是将实体框架自动脏值检测启用你的应用程序的整个生存期内保留一个好办法。 如果你的方案受 CPU 使用率过高造成负面影响，并且在配置文件指示罪魁祸首是调用 DetectChanges，请考虑暂时关闭 AutoDetectChanges，在你的代码的敏感部分中：

``` csharp
try
{
    context.Configuration.AutoDetectChangesEnabled = false;
    var product = context.Products.Find(productId);
    ...
}
finally
{
    context.Configuration.AutoDetectChangesEnabled = true;
}
```

在关闭 AutoDetectChanges 之前, 是最好的了解，这可能会导致实体框架可以在其无法跟踪的更改的实体上所发生的某些信息。 如果处理不正确，这可能导致你的应用程序上的数据不一致。 关闭 AutoDetectChanges 的详细信息，请阅读\<\ http://blog.oneunicorn.com/2012/03/12/secrets-of-detectchanges-part-3-switching-off-automatic-detectchanges/>。

### <a name="93-context-per-request"></a>9.3 每个请求上下文

实体框架的上下文是为了用作生存期较短实例以便提供最佳性能体验。 上下文都将是短生存期和丢弃的项，并且这种情况下已实现非常轻型的软件和 reutilize 只要有可能的元数据。 在 web 方案中是一定要记住这一点并不具有对多个单个请求的持续时间的上下文。 同样，在非 web 方案中，上下文应放弃根据您对不同级别的缓存在实体框架的理解。 通常情况下，一个应避免必须在整个生命周期内的应用程序，以及每个线程的上下文和静态上下文的上下文实例。

### <a name="94-database-null-semantics"></a>9.4 数据库 null 语义

默认情况下的实体框架将生成具有 C 的 SQL 代码\#null 比较语义。 请考虑下面的示例查询：

``` csharp
            int? categoryId = 7;
            int? supplierId = 8;
            decimal? unitPrice = 0;
            short? unitsInStock = 100;
            short? unitsOnOrder = 20;
            short? reorderLevel = null;

            var q = from p incontext.Products
                    wherep.Category.CategoryName == "Beverages"
                          || (p.CategoryID == categoryId
                                || p.SupplierID == supplierId
                                || p.UnitPrice == unitPrice
                                || p.UnitsInStock == unitsInStock
                                || p.UnitsOnOrder == unitsOnOrder
                                || p.ReorderLevel == reorderLevel)
                    select p;

            var r = q.ToList();
```

在此示例中，我们比较数对可以为 null 的实体，例如供应商和单价属性可以为 null 的变量。 如果参数值的列值，与相同或在参数和列的值均为 null，将要求生成此查询的 SQL。 这将隐藏数据库服务器处理 null 值的方式，并将提供一致的 C\#跨不同的数据库供应商，则为 null 的体验。 但是，生成的代码有点令人费解，可能无法执行时，也比较量在 where 语句的查询数增长到很多。

若要处理这种情况的一种方法是使用数据库 null 语义。 请注意，这可能会行为的行为不同到 C\# null 语义，因为现在实体框架将生成更简单的 SQL 数据库引擎处理 null 值的方式公开。 数据库 null 语义可以是激活每个上下文只需单个配置一行针对上下文配置：

``` csharp
                context.Configuration.UseDatabaseNullSemantics = true;
```

小型到中等大小的查询将不会显示可察觉的性能改进时使用数据库 null 语义，但不同之处将变得更明显上具有大量的潜在 null 比较查询。

在上面的示例查询中，性能差异是在受控环境中运行 microbenchmark 小于 2%。

### <a name="95-async"></a>9.5 异步

异步操作时运行在.NET 4.5 或更高版本的 entity Framework 6 引入了支持。 大多数情况下，具有 IO 的应用程序相关的争用将受益于最多使用异步查询和保存操作。 如果你的应用程序不会受到 IO 争用，使用 async 将在最佳情况下，以同步方式运行和返回结果，在相同的时间量为同步调用，或在最坏情况下，只需推迟到一个异步任务的执行并添加额外 tim完成你的方案的 e。

有关如何异步编程工作，可帮助您决定是否异步会提高应用程序的性能所访问的信息[http://msdn.microsoft.com/library/hh191443.aspx](https://msdn.microsoft.com/library/hh191443.aspx)。 有关使用实体框架的异步操作的详细信息，请参阅[异步查询和保存](~/ef6/fundamentals/async.md
)。

### <a name="96-ngen"></a>9.6 NGEN

Entity Framework 6 不能在默认安装.NET framework 中。 在这种情况下，实体框架程序集不是默认情况下，这意味着所有实体框架代码都受到相同的 JIT'ing 成本与任何其他 MSIL 程序集的 NGEN 'd。 这可能会降低 F5 体验，同时开发和生产环境中的应用程序的冷启动。 为了降低 JIT'ing 的 CPU 和内存成本最好是的 NGEN 映像作为相应的实体框架。 有关如何改进使用 NGEN Entity Framework 6 的启动性能的详细信息，请参阅[使用 NGen 提高启动性能](~/ef6/fundamentals/performance/ngen.md)。

### <a name="97-code-first-versus-edmx"></a>9.7 代码优先与 EDMX

有关面向对象编程和通过概念模型 （对象）、 存储架构 （数据库） 和之间的映射的内存中表示的关系数据库之间的阻抗不匹配问题的实体框架原因两个。 此元数据的实体数据模型或 EDM 为调用短。 此 EDM 中，从实体框架将派生自视图往返数据到数据库的内存中对象和备份。

当 Entity Framework 用于 EDMX 文件的正式指定概念模型、 存储架构和映射，则模型加载阶段仅具有来验证 EDM 是正确 （例如，请确保没有映射是缺失），然后生成的视图，然后验证视图，并已准备好使用此元数据。 仅执行然后可以查询或将新的数据保存到数据存储区。

代码第一种方法就其本质而言，是一个复杂的实体数据模型生成器。 Entity Framework 有以生成所提供的代码; 从 EDM它会分析模型、 应用约定和配置通过 Fluent API 模型中涉及的类。 生成 EDM 后，实体框架实质上是行为的行为相同方式与将具有 EDMX 文件已存在于项目的方式。 因此，从 Code First 生成模型将添加额外的复杂性，转换为实体框架相比具有 EDMX 时速度较慢的启动时间。 成本是模型的完全依赖于的大小和复杂性正在生成。

在选择使用 Code First 和 EDMX，务必知道引入的 Code First 的灵活性，增加的第一次生成模型的成本。 如果你的应用程序可承受的此第一次加载成本则通常 Code First 将转的首选的方法。

## <a name="10-investigating-performance"></a>10 调查性能

### <a name="101-using-the-visual-studio-profiler"></a>10.1 使用 Visual Studio Profiler

如果在使用实体框架的性能问题，可以使用类似于内置到 Visual Studio 探查器以查看你的应用程序上花费其时间。 这是我们用于生成饼图"探讨 ADO.NET 实体框架的第 1 部分的性能"博客文章中的工具 ( \<http://blogs.msdn.com/b/adonet/archive/2008/02/04/exploring-the-performance-of-the-ado-net-entity-framework-part-1.aspx>) ，可显示实体框架何处消耗在冷和热查询过程及其时间。

由数据和建模客户顾问团队编写的 《 使用 Visual Studio 2010 Profiler 分析 Entity Framework 》 博客文章显示了如何在使用探查器来调查性能问题的一个实际示例。  \<http://blogs.msdn.com/b/dmcat/archive/2010/04/30/profiling-entity-framework-using-the-visual-studio-2010-profiler.aspx>. 此文章针对 windows 应用程序的编写。 如果你需要分析 web 应用程序的 Windows 性能记录器 (WPR) 和 Windows Performance Analyzer (WPA) 工具可能效果更佳从 Visual Studio 的工作。 WPR 和 WPA 是 Windows 性能工具包附带 Windows 评估和部署工具包的一部分 ( [http://www.microsoft.com/download/details.aspx?id=39982](https://www.microsoft.com/download/details.aspx?id=39982))。

### <a name="102-applicationdatabase-profiling"></a>10.2 应用程序/数据库分析

内置到 Visual Studio 探查器等工具告诉你的应用程序上花费时间。  另一种类型的探查器是可用的执行动态分析运行的应用程序，在生产或根据需要，预生产中，并查找常见缺陷和反模式的数据库访问权限。

两个地区销售的探查器是实体框架 Profiler ( \<http://efprof.com>) ORMProfiler 和 ( \<http://ormprofiler.com>)。

如果应用程序是使用 Code First 的 MVC 应用程序，可以使用 StackExchange 的 MiniProfiler。 Scott Hanselman 在他的博客中介绍了此工具： \<http://www.hanselman.com/blog/NuGetPackageOfTheWeek9ASPNETMiniProfilerFromStackExchangeRocksYourWorld.aspx>。

有关详细信息分析应用程序的数据库活动，请参阅标题为 Julie Lerman 的 MSDN 杂志 》 文章[分析实体框架中的数据库活动](https://msdn.microsoft.com/magazine/gg490349.aspx)。

### <a name="103-database-logger"></a>10.3 数据库记录器

如果您使用的 Entity Framework 6 也可考虑使用内置日志记录功能。 通过简单的单行配置其活动记录，可指示上下文的数据库属性：

``` csharp
    using (var context = newQueryComparison.DbC.NorthwindEntities())
    {
        context.Database.Log = Console.WriteLine;
        var q = context.Products.Where(p => p.Category.CategoryName == "Beverages");
        q.ToList();
    }
```

在此示例中的数据库活动将记录到控制台中，但日志属性可以配置为调用任何操作&lt;字符串&gt;委托。

如果你想要启用数据库日志记录，而无需重新编译，并且您使用的 Entity Framework 6.1 或更高版本，您可以这样做通过你的应用程序的 web.config 或 app.config 文件中添加侦听器。

``` xml
  <interceptors>
    <interceptor type="System.Data.Entity.Infrastructure.Interception.DatabaseLogger, EntityFramework">
      <parameters>
        <parameter value="C:\Path\To\My\LogOutput.txt"/>
      </parameters>
    </interceptor>
  </interceptors>
```

有关如何添加日志记录，无需重新编译转到详细信息\<http://blog.oneunicorn.com/2014/02/09/ef-6-1-turning-on-logging-without-recompiling/>。

## <a name="11-appendix"></a>11 附录

### <a name="111-a-test-environment"></a>11.1 A.测试环境

此环境使用具有客户端应用程序从一台计算机上数据库的计算机 2 的安装程序。 机位于同一机架中，因此网络延迟是相对较低，但比单计算机环境更真实。

#### <a name="1111-app-server"></a>11.1.1 应用服务器

##### <a name="11111-software-environment"></a>11.1.1.1 软件环境

-   实体框架 4 软件环境
    -   OS 名称： Windows Server 2008 R2 Enterprise SP1。
    -   Visual Studio 2010 – Ultimate。
    -   （仅适用于某些比较） 的 visual Studio 2010 SP1。
-   Entity Framework 5 和 6 软件环境
    -   OS 名称： Windows 8.1 企业版
    -   Visual Studio 2013 – Ultimate。

##### <a name="11112-hardware-environment"></a>11.1.1.2 硬件环境

-   双处理器: @ 2.27 GHz intel （) 至强® CPU L5520 W3530、 2261 Mhz8 g h z、 4 个核心，84 逻辑处理器。
-   2412 GB RamRAM。
-   136 GB SCSI250GB SATA 7200 rpm 3 GB/s 驱动器将拆分为 4 个分区。

#### <a name="1112-db-server"></a>11.1.2 DB 服务器

##### <a name="11121-software-environment"></a>11.1.2.1 软件环境

-   OS 名称： Windows Server 2008 R28.1 Enterprise SP1。
-   SQL Server 2008 R22012。

##### <a name="11122-hardware-environment"></a>11.1.2.2 硬件环境

-   单处理器： 2.27 GHz intel （) 至强® CPU L5520、 2261 MhzES-1620 0 @ 3.60 g h z、 4 个核心、 8 逻辑处理器。
-   824 GB RamRAM。
-   465 GB ATA500GB SATA 7200 rpm 6 GB/s 驱动器将拆分为 4 个分区。

### <a name="112-b-query-performance-comparison-tests"></a>11.2 B.查询性能比较测试

Northwind 模型用于执行这些测试。 它是使用实体框架设计器从数据库生成的。 然后，使用以下代码来比较查询执行选项的性能：

``` csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.Common;
using System.Data.Entity.Infrastructure;
using System.Data.EntityClient;
using System.Data.Objects;
using System.Linq;

namespace QueryComparison
{
    public partial class NorthwindEntities : ObjectContext
    {
        private static readonly Func<NorthwindEntities, string, IQueryable<Product>> productsForCategoryCQ = CompiledQuery.Compile(
            (NorthwindEntities context, string categoryName) =>
                context.Products.Where(p => p.Category.CategoryName == categoryName)
                );

        public IQueryable<Product> InvokeProductsForCategoryCQ(string categoryName)
        {
            return productsForCategoryCQ(this, categoryName);
        }
    }

    public class QueryTypePerfComparison
    {
        private static string entityConnectionStr = @"metadata=res://*/Northwind.csdl|res://*/Northwind.ssdl|res://*/Northwind.msl;provider=System.Data.SqlClient;provider connection string='data source=.;initial catalog=Northwind;integrated security=True;multipleactiveresultsets=True;App=EntityFramework'";

        public void LINQIncludingContextCreation()
        {
            using (NorthwindEntities context = new NorthwindEntities())
            {                 
                var q = context.Products.Where(p => p.Category.CategoryName == "Beverages");
                q.ToList();
            }
        }

        public void LINQNoTracking()
        {
            using (NorthwindEntities context = new NorthwindEntities())
            {
                context.Products.MergeOption = MergeOption.NoTracking;

                var q = context.Products.Where(p => p.Category.CategoryName == "Beverages");
                q.ToList();
            }
        }

        public void CompiledQuery()
        {
            using (NorthwindEntities context = new NorthwindEntities())
            {
                var q = context.InvokeProductsForCategoryCQ("Beverages");
                q.ToList();
            }
        }

        public void ObjectQuery()
        {
            using (NorthwindEntities context = new NorthwindEntities())
            {
                ObjectQuery<Product> products = context.Products.Where("it.Category.CategoryName = 'Beverages'");
                products.ToList();
            }
        }

        public void EntityCommand()
        {
            using (EntityConnection eConn = new EntityConnection(entityConnectionStr))
            {
                eConn.Open();
                EntityCommand cmd = eConn.CreateCommand();
                cmd.CommandText = "Select p From NorthwindEntities.Products As p Where p.Category.CategoryName = 'Beverages'";

                using (EntityDataReader reader = cmd.ExecuteReader(CommandBehavior.SequentialAccess))
                {
                    List<Product> productsList = new List<Product>();
                    while (reader.Read())
                    {
                        DbDataRecord record = (DbDataRecord)reader.GetValue(0);

                        // 'materialize' the product by accessing each field and value. Because we are materializing products, we won't have any nested data readers or records.
                        int fieldCount = record.FieldCount;

                        // Treat all products as Product, even if they are the subtype DiscontinuedProduct.
                        Product product = new Product();  

                        product.ProductID = record.GetInt32(0);
                        product.ProductName = record.GetString(1);
                        product.SupplierID = record.GetInt32(2);
                        product.CategoryID = record.GetInt32(3);
                        product.QuantityPerUnit = record.GetString(4);
                        product.UnitPrice = record.GetDecimal(5);
                        product.UnitsInStock = record.GetInt16(6);
                        product.UnitsOnOrder = record.GetInt16(7);
                        product.ReorderLevel = record.GetInt16(8);
                        product.Discontinued = record.GetBoolean(9);

                        productsList.Add(product);
                    }
                }
            }
        }

        public void ExecuteStoreQuery()
        {
            using (NorthwindEntities context = new NorthwindEntities())
            {
                ObjectResult<Product> beverages = context.ExecuteStoreQuery<Product>(
@"    SELECT        P.ProductID, P.ProductName, P.SupplierID, P.CategoryID, P.QuantityPerUnit, P.UnitPrice, P.UnitsInStock, P.UnitsOnOrder, P.ReorderLevel, P.Discontinued
    FROM            Products AS P INNER JOIN Categories AS C ON P.CategoryID = C.CategoryID
    WHERE        (C.CategoryName = 'Beverages')"
);
                beverages.ToList();
            }
        }

        public void ExecuteStoreQueryDbContext()
        {
            using (var context = new QueryComparison.DbC.NorthwindEntities())
            {
                var beverages = context.Database.SqlQuery\<QueryComparison.DbC.Product>(
@"    SELECT        P.ProductID, P.ProductName, P.SupplierID, P.CategoryID, P.QuantityPerUnit, P.UnitPrice, P.UnitsInStock, P.UnitsOnOrder, P.ReorderLevel, P.Discontinued
    FROM            Products AS P INNER JOIN Categories AS C ON P.CategoryID = C.CategoryID
    WHERE        (C.CategoryName = 'Beverages')"
);
                beverages.ToList();
            }
        }

        public void ExecuteStoreQueryDbSet()
        {
            using (var context = new QueryComparison.DbC.NorthwindEntities())
            {
                var beverages = context.Products.SqlQuery(
@"    SELECT        P.ProductID, P.ProductName, P.SupplierID, P.CategoryID, P.QuantityPerUnit, P.UnitPrice, P.UnitsInStock, P.UnitsOnOrder, P.ReorderLevel, P.Discontinued
    FROM            Products AS P INNER JOIN Categories AS C ON P.CategoryID = C.CategoryID
    WHERE        (C.CategoryName = 'Beverages')"
);
                beverages.ToList();
            }
        }

        public void LINQIncludingContextCreationDbContext()
        {
            using (var context = new QueryComparison.DbC.NorthwindEntities())
            {                 
                var q = context.Products.Where(p => p.Category.CategoryName == "Beverages");
                q.ToList();
            }
        }

        public void LINQNoTrackingDbContext()
        {
            using (var context = new QueryComparison.DbC.NorthwindEntities())
            {
                var q = context.Products.AsNoTracking().Where(p => p.Category.CategoryName == "Beverages");
                q.ToList();
            }
        }
    }
}
```

### <a name="113-c-navision-model"></a>11.3 C.Navision 模型

Navision 数据库是用于演示 Microsoft Dynamics – 导航的大型数据库 生成概念模型包含 1005年实体集和 4227 关联集。 在测试中使用的模型是"平面"– 没有继承已添加到它。

#### <a name="1131-queries-used-for-navision-tests"></a>11.3.1 查询用于 Navision 测试

使用 Navision 模型使用的查询列表中包含 Entity SQL 查询的 3 个的类别：

##### <a name="11311-lookup"></a>11.3.1.1 查找

简单的查找查询不包含聚合

-   计数： 16232
-   示例:

``` xml
  <Query complexity="Lookup">
    <CommandText>Select value distinct top(4) e.Idle_Time From NavisionFKContext.Session as e</CommandText>
  </Query>
```

##### <a name="11312singleaggregating"></a>11.3.1.2 SingleAggregating

具有多个聚合，但没有小计 （单个查询） 的正常 BI 查询

-   计数： 2313年
-   示例:

``` xml
  <Query complexity="SingleAggregating">
    <CommandText>NavisionFK.MDF_SessionLogin_Time_Max()</CommandText>
  </Query>
```

其中 MDF\_SessionLogin\_时间\_max （） 作为模型中定义：

``` xml
  <Function Name="MDF_SessionLogin_Time_Max" ReturnType="Collection(DateTime)">
    <DefiningExpression>SELECT VALUE Edm.Min(E.Login_Time) FROM NavisionFKContext.Session as E</DefiningExpression>
  </Function>
```

##### <a name="11313aggregatingsubtotals"></a>11.3.1.3 AggregatingSubtotals

具有聚合和小计 （通过所有的并集） 的 BI 查询

-   计数： 178
-   示例:

``` xml
  <Query complexity="AggregatingSubtotals">
    <CommandText>
using NavisionFK;
function AmountConsumed(entities Collection([CRONUS_International_Ltd__Zone])) as
(
    Edm.Sum(select value N.Block_Movement FROM entities as E, E.CRONUS_International_Ltd__Bin as N)
)
function AmountConsumed(P1 Edm.Int32) as
(
    AmountConsumed(select value e from NavisionFKContext.CRONUS_International_Ltd__Zone as e where e.Zone_Ranking = P1)
)
----------------------------------------------------------------------------------------------------------------------
(
    select top(10) Zone_Ranking, Cross_Dock_Bin_Zone, AmountConsumed(GroupPartition(E))
    from NavisionFKContext.CRONUS_International_Ltd__Zone as E
    where AmountConsumed(E.Zone_Ranking) > @MinAmountConsumed
    group by E.Zone_Ranking, E.Cross_Dock_Bin_Zone
)
union all
(
    select top(10) Zone_Ranking, Cast(null as Edm.Byte) as P2, AmountConsumed(GroupPartition(E))
    from NavisionFKContext.CRONUS_International_Ltd__Zone as E
    where AmountConsumed(E.Zone_Ranking) > @MinAmountConsumed
    group by E.Zone_Ranking
)
union all
{
    Row(Cast(null as Edm.Int32) as P1, Cast(null as Edm.Byte) as P2, AmountConsumed(select value E
                                                                         from NavisionFKContext.CRONUS_International_Ltd__Zone as E
                                                                         where AmountConsumed(E.Zone_Ranking) > @MinAmountConsumed))
}</CommandText>
    <Parameters>
      <Parameter Name="MinAmountConsumed" DbType="Int32" Value="10000" />
    </Parameters>
  </Query>
```
