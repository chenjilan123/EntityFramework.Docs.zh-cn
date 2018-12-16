---
title: 反向工程-EF Core
author: bricelam
ms.author: bricelam
ms.date: 11/13/2018
ms.assetid: 6263EF7D-4989-42E6-BDEE-45DA770342FB
uid: core/managing-schemas/scaffolding
ms.openlocfilehash: ef729c0c26d5a1f57099f339eb51cda7e83289df
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688675"
---
# <a name="reverse-engineering"></a>反向工程

反向工程是实体类型类和一个基于数据库架构的 DbContext 类的基架的过程。 可以执行使用`Scaffold-DbContext`EF Core 包管理器控制台 (PMC) 工具的命令或`dotnet ef dbcontext scaffold`.NET 命令行接口 (CLI) 工具的命令。

## <a name="installing"></a>安装

反向工程之前你将需要安装[PMC 工具](xref:core/miscellaneous/cli/powershell)(仅限 Visual Studio) 或[CLI 工具](xref:core/miscellaneous/cli/dotnet)。 请参阅有关详细信息的链接。

您还需要安装相应[数据库提供程序](xref:core/providers/index)你想要实施反向工程的数据库架构。

## <a name="connection-string"></a>连接字符串

该命令的第一个参数是数据库的连接字符串。 工具将使用此连接字符串来读取数据库架构。

如何用引号括起来并转义的连接字符串取决于哪个 shell 您正在使用执行命令。 请参阅 shell 的文档了解详细信息。 例如，PowerShell 要求你进行转义`$`字符，但不是`\`。

``` powershell
Scaffold-DbContext 'Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook' Microsoft.EntityFrameworkCore.SqlServer
```

``` Console
dotnet ef dbcontext scaffold "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook" Microsoft.EntityFrameworkCore.SqlServer
```

### <a name="configuration-and-user-secrets"></a>配置和用户机密

如果您有一个 ASP.NET Core 项目，则可以使用`Name=<connection-string>`语法来从配置中读取连接字符串。

这适用于[机密管理器工具](https://docs.microsoft.com/aspnet/core/security/app-secrets#secret-manager)以便将自己的数据库密码与你的基本代码分开。

``` Console
dotnet user-secrets set ConnectionStrings.Chinook "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook"
dotnet ef dbcontext scaffold Name=Chinook Microsoft.EntityFrameworkCore.SqlServer
```

## <a name="provider-name"></a>提供程序名称

第二个参数是提供程序名称。 提供程序名称通常是与提供程序的 NuGet 包名称相同。

## <a name="specifying-tables"></a>指定表

数据库架构中的所有表都被反向工程到实体类型，默认情况下。 您可以限制哪些表是反向工程处理通过指定架构和表。

`-Schemas`在 PMC 中的参数和`--schema`CLI 中的选项可用于包含在架构中的每个表。

`-Tables` (PMC) 和`--table`(CLI) 可用于包括特定的表。

若要在 PMC 中包含多个表，使用一个数组。

``` powershell
Scaffold-DbContext ... -Tables Artist, Album
```

若要在 CLI 中包含多个表，请多次指定选项。

``` Console
dotnet ef dbcontext scaffold ... --table Artist --table Album
```

## <a name="preserving-names"></a>保留名称

表和列的名称被固定的以更好地匹配的类型和属性的.NET 命名约定默认情况下。 指定`-UseDatabaseNames`切换在 PMC 中或`--use-database-names`CLI 中的选项将禁用此行为保留尽可能多地原始数据库名称。 仍将修复无效的.NET 标识符和合成的名称，如导航属性仍将遵循.NET 命名约定。

## <a name="fluent-api-or-data-annotations"></a>Fluent API 或数据注释

实体类型是使用 Fluent API 默认配置。 指定`-DataAnnotations`(PMC) 或`--data-annotations`(CLI) 要改为使用数据注释在可能的情况。

例如，使用 Fluent API 将基架生成此。

``` csharp
entity.Property(e => e.Title)
    .IsRequired()
    .HasMaxLength(160);
```

使用数据注释时将创建基架这。

``` csharp
[Required]
[StringLength(160)]
public string Title { get; set; }
```

## <a name="dbcontext-name"></a>DbContext 名称

已搭建基架的 DbContext 类名称将用作后缀的数据库的名称*上下文*默认情况下。 若要指定一个不同，使用`-Context`在 PMC 中和`--context`CLI 中。

## <a name="directories-and-namespaces"></a>目录和命名空间

实体类和 DbContext 类到项目的根目录的基架，并使用项目的默认命名空间。 可以指定的目录的类使用基架`-OutputDir`(PMC) 或`--output-dir`(CLI)。 命名空间将为根命名空间加上任何项目的根目录下的子目录的名称。

此外可以使用`-ContextDir`(PMC) 和`--context-dir`(CLI) 来创建到一个单独的目录从实体类型类的基架的 DbContext 类。

``` powershell
Scaffold-DbContext ... -ContextDir Data -OutputDir Models
```

``` Console
dotnet ef dbcontext scaffold ... --context-dir Data --output-dir Models
```

## <a name="how-it-works"></a>其工作原理

反向工程开始时读取数据库架构。 它将读取有关表、 列、 约束和索引的信息。

接下来，它使用的架构信息创建 EF Core 模型。 使用表来创建实体类型;使用列来创建属性;和外键用于创建关系。

最后，该模型用于生成代码。 相应的实体类型的类、 Fluent API 和数据批注已搭建基架以重新创建相同的模型从您的应用程序中。

## <a name="what-doesnt-work"></a>什么不起作用

并非所有有关模型的内容都可以表示使用数据库架构。 例如，有关的信息**继承层次结构**，**固有类型**，并**表拆分**数据库架构中不存在。 因此，这些构造将永远不能反向工程处理。

此外，**某些列类型**可能不支持通过 EF Core 提供程序。 这些列不包含在模型中。

EF Core，需要每个实体类型的键。 表，但是，无需指定为主键。 **表没有主键**是当前不实施反向工程。

您可以定义**并发标记**EF Core 模型以防止两个用户在同一时间更新同一实体中。 有些数据库具有一种特殊类型来表示这种情况下，我们可以反向设计此信息; 在这种类型的列 （例如，SQL Server 中的行版本）但是，其他并发令牌将不能反向工程处理。

## <a name="customizing-the-model"></a>自定义模型

EF Core 生成的代码是你的代码。 随意对其进行更改。 它将仅重新生成如果再次反向工程，相同的模型。 基架的代码表示*一个*模型，可用于访问数据库，但它当然不是*仅*可以使用的模型。

自定义实体类型类和 DbContext 类，以满足你的需求。 例如，你可以选择重命名类型和属性，引入了继承层次结构或拆分到多个实体的表。 此外可以从模型中删除非唯一索引、 未使用的序列和导航属性、 可选的标量属性和约束名称。

此外可以添加其他构造函数、 方法、 属性等。 在一个单独的文件中使用另一个分部类。 此方法适用于即使你想要再次反向工程的模型。

## <a name="updating-the-model"></a>更新模型

之后对数据库进行更改，可能需要更新您的 EF Core 模型以反映这些更改。 如果数据库更改很简单，它可能是最简单的方法手动对 EF Core 模型进行更改。 例如，重命名表或列、 删除列，或更新列的类型是无关紧要的更改以使在代码中。

更显著的更改，但是，不是为轻松进行手动。 一种常见的工作流是反向设计模型从再次使用的数据库`-Force`(PMC) 或`--force`(CLI) 使用已更新覆盖现有模型。

另一个经常请求的功能是能够从数据库更新模型同时还能保留自定义项，例如重命名、 类型层次结构，等等。使用问题[#831](https://github.com/aspnet/EntityFrameworkCore/issues/831)来跟踪此功能的进度。

> [!WARNING]
> 如果进行反向工程的模型从数据库再次，您对文件所做的任何更改将丢失。
