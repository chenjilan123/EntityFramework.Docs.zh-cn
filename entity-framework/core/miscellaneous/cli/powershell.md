---
title: EF Core 工具参考 （程序包管理器控制台）-EF Core
author: bricelam
ms.author: bricelam
ms.date: 09/18/2018
uid: core/miscellaneous/cli/powershell
ms.openlocfilehash: 468698d1bbd17d4ad10b1b1601bfbc315a01c1ff
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688701"
---
# <a name="entity-framework-core-tools-reference---package-manager-console-in-visual-studio"></a>Entity Framework Core 工具引用-Visual Studio 中的包管理器控制台

Entity Framework Core 的包管理器控制台 (PMC) 工具执行设计时开发任务。 例如，创建方法[迁移](/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-2.0#introduction-to-migrations)、 应用迁移，并为基于现有数据库的模型生成代码。 在 Visual Studio 中使用的内部运行命令[程序包管理器控制台](/nuget/tools/package-manager-console)。 这些工具同时适用于 .NET Framework 和 .NET Core 项目。

如果不使用 Visual Studio，我们建议[EF Core 命令行工具](dotnet.md)相反。 CLI 工具是跨平台，且在命令提示符下运行。

## <a name="installing-the-tools"></a>安装工具

ASP.NET Core 2.1 + 和早期版本或其他项目类型之间存在差异的安装和更新工具的过程。

### <a name="aspnet-core-version-21-and-later"></a>ASP.NET Core 2.1 和更高版本

这些工具会自动包含在 ASP.NET Core 2.1 + 项目，因为`Microsoft.EntityFrameworkCore.Tools`包包含在[Microsoft.AspNetCore.App 元包](/aspnet/core/fundamentals/metapackage-app)。

因此，无需执行任何操作来安装工具，但可以为：
* 在新项目中使用的工具之前还原包。
* 安装包更新到较新版本的工具。

若要确保获得最新版本的工具，我们建议您以下步骤：

* 编辑你 *.csproj*文件，并添加一行以指定的最新版本[Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools/)包。 例如， *.csproj*文件可能包含`ItemGroup`如下所示：

  ```xml
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.App" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="2.1.3" />
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.1.1" />
  </ItemGroup>
  ```

更新的工具时你收到一条消息，如下例所示：

> EF Core 工具版本"2.1.1-rtm-30846"为早于的运行时"2.1.3-rtm-32065"。 更新适用于最新功能和 bug 修补程序的工具。

若要更新的工具：
* 安装最新的.NET Core SDK。
* Visual Studio 更新到最新版本。
* 编辑 *.csproj*文件以使其包括到最新工具包，包引用，如前面所示。

### <a name="other-versions-and-project-types"></a>其他版本和项目类型

通过在运行以下命令安装程序包管理器控制台工具**程序包管理器控制台**:

``` powershell
Install-Package Microsoft.EntityFrameworkCore.Tools
```

通过在运行以下命令更新的工具**程序包管理器控制台**。

``` powershell
Update-Package Microsoft.EntityFrameworkCore.Tools
```

### <a name="verify-the-installation"></a>验证安装

验证通过运行以下命令安装这些工具：

``` powershell
Get-Help about_EntityFrameworkCore
```

输出如下所示 （它不会让您正在使用的工具版本）：

```console

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

TOPIC
    about_EntityFrameworkCore

SHORT DESCRIPTION
    Provides information about the Entity Framework Core Package Manager Console Tools.

<A list of available commands follows, omitted here.>
```

## <a name="using-the-tools"></a>使用的工具

之前使用的工具：
* 了解目标和启动项目之间的差异。
* 了解如何使用.NET Standard 类库的工具。
* 对于 ASP.NET Core 项目设置的环境。

### <a name="target-and-startup-project"></a>目标和启动项目

命令是指*项目*和一个*启动项目*。

* *项目*也称为*目标项目*因为它是命令添加或删除文件的位置。 默认情况下**默认项目**中所选**程序包管理器控制台**是目标项目。 可以通过使用作为目标项目中指定一个不同的项目<nobr> `--project` </nobr>选项。

* *启动项目*是指的工具生成和运行。 这些工具必须在设计时，获取有关该项目，如数据库连接字符串和模型的配置信息执行应用程序代码。 默认情况下**启动项目**中**解决方案资源管理器**是启动项目。 您可以通过使用不同的项目指定为启动项目<nobr> `--startup-project` </nobr>选项。

启动项目和目标项目通常是在同一个项目。 它们的单独的项目的典型情况是当：

* EF Core 上下文和实体类是在.NET Core 类库中。
* .NET Core 控制台应用程序或 web 应用程序引用的类库。

此外，还可以向[迁移代码置于独立于 EF Core 上下文类库](xref:core/managing-schemas/migrations/projects)。

### <a name="other-target-frameworks"></a>其他目标框架

包管理器控制台工具适用于.NET Core 或.NET Framework 项目。 EF Core 模型具有.NET Standard 类库中的应用可能不具有.NET Core 或.NET Framework 项目。 例如，这是 Xamarin 和通用 Windows 平台应用，则返回 true。 在这种情况下，可以创建其唯一用途是作为启动项目的工具的.NET Core 或.NET Framework 控制台应用项目。 项目可以是虚拟项目不包含实际代码&mdash;只需为工具提供一个目标。

为什么是虚拟的项目所需？ 如前文所述，这些工具必须在设计时执行应用程序代码。 若要做到这一点，他们需要使用.NET Core 或.NET Framework 运行时。 EF Core 模型在面向.NET Core 或.NET Framework 的项目时，EF Core 工具借用运行时从该项目。 如果在 EF Core 模型处于.NET Standard 类库，他们无法做到这一点。 .NET Standard 并不实际的.NET 实现;它是一种规范的一组.NET 实现必须支持的 Api。 因此.NET Standard 是不够的 EF Core 工具来执行应用程序代码。 创建要用作启动项目的虚拟项目提供了工具可以在其中加载.NET Standard 类库的具体目标平台。

### <a name="aspnet-core-environment"></a>ASP.NET Core 环境

若要指定用于 ASP.NET Core 项目的环境，请设置**env:ASPNETCORE_ENVIRONMENT**之前运行命令。

## <a name="common-parameters"></a>通用参数

下表显示了所共有的所有 EF Core 命令参数：

| 参数                 | 描述                                                                                                                                                                                                          |
|:--------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 上下文\<字符串 >        | `DbContext`类使用。 唯一或完全限定的命名空间的类名称。  如果省略此参数，EF Core 将查找上下文类。 如果有多个上下文类，此参数是必需的。 |
| -项目\<字符串 >        | 目标项目中。 如果省略此参数，则**默认项目**有关**程序包管理器控制台**用作目标项目。                                                                             |
| -StartupProject\<字符串 > | 启动项目。 如果省略此参数，则**启动项目**中**解决方案属性**用作目标项目。                                                                                 |
| -Verbose                  | 显示详细输出。                                                                                                                                                                                                 |

若要显示有关命令的帮助信息，请使用 PowerShell 的`Get-Help`命令。

> [!TIP]
> 上下文、 项目和 StartupProject 参数支持选项卡扩展。

## <a name="add-migration"></a>添加迁移

添加新的迁移。

参数：

| 参数                         | 描述                                                                                                             |
|:----------------------------------|:------------------------------------------------------------------------------------------------------------------------|
| <nobr>-Name\<字符串 ><nobr>       | 迁移的名称。 这是位置参数，是必需的。                                              |
| <nobr>-OutputDir\<字符串 ></nobr> | 目录 （及其子命名空间） 来使用。 路径是相对于目标项目目录。 默认值为"迁移"。 |

## <a name="drop-database"></a>删除数据库

删除数据库。

参数：

| 参数 | 描述                                              |
|:----------|:---------------------------------------------------------|
| -WhatIf   | 显示哪个数据库会被丢弃，但没有删除它。 |

## <a name="get-dbcontext"></a>Get-DbContext

列出可用`DbContext`类型。

## <a name="remove-migration"></a>Remove-Migration

删除 （请回滚迁移已完成的代码更改） 的最后一个迁移。

参数：

| 参数 | 描述                                                                     |
|:----------|:--------------------------------------------------------------------------------|
| -Force    | 还原迁移 （请回滚已应用到数据库的更改）。 |

## <a name="scaffold-dbcontext"></a>DbContext 基架

为生成代码`DbContext`和数据库的实体类型。 为了使`Scaffold-DbContext`若要生成的实体类型，数据库表必须具有主键。

参数：

| 参数                          | 描述                                                                                                                                                                                                                                                             |
|:-----------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>-连接\<字符串 ></nobr> | 数据库的连接字符串。 对于 ASP.NET Core 2.x 项目，值可以是*名称 =\<的连接字符串名称 >*。 在这种情况下名称来自于为项目设置的配置源。 这是位置参数，是必需的。 |
| <nobr>提供程序\<字符串 ></nobr>   | 要使用的提供程序。 通常这是 NuGet 包的名称为例： `Microsoft.EntityFrameworkCore.SqlServer`。 这是位置参数，是必需的。                                                                                           |
| -OutputDir\<字符串 >               | 要将文件放入的目录。 路径是相对于项目目录。                                                                                                                                                                                             |
| -ContextDir\<字符串 >              | 要放置的目录`DbContext`文件中。 路径是相对于项目目录。                                                                                                                                                                              |
| 上下文\<字符串 >                 | 名称`DbContext`类生成。                                                                                                                                                                                                                          |
| -架构\<String [] >               | 要生成的实体类型的表架构。 如果省略此参数，则包括所有架构。                                                                                                                                                             |
| -表\<String [] >                | 要生成的实体类型的表。 如果省略此参数，则包括所有表。                                                                                                                                                                         |
| -DataAnnotations                   | 特性用于将模型配置 （如果可能）。 如果省略此参数，则使用仅 fluent API。                                                                                                                                                      |
| -UseDatabaseNames                  | 在数据库中显示的完全相同，请使用表和列的名称。 如果省略此参数，则会更改数据库名称，使其更紧密地符合 C# 名称样式约定。                                                                                       |
| -Force                             | 覆盖现有文件。                                                                                                                                                                                                                                               |

示例:

```powershell
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
```

搭建基架以选定的表，并具有指定名称的单独文件夹中创建上下文的示例：

```powershell
Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Tables "Blog","Post" -ContextDir Context -Context BlogContext
```

## <a name="script-migration"></a>Script-Migration

生成应用的所有更改从一个所选迁移到另一个所选的迁移的 SQL 脚本。

参数：

| 参数                | 描述                                                                                                                                                                                                                |
|:-------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| *-从*\<字符串 >        | 开始迁移。 可能会被标识迁移，按名称或 id。 数字 0 是一种特殊情况，意味着*第一次迁移之前*。 默认值为 0。                                                              |
| *-To* \<字符串 >          | 结束的迁移。 默认的最后一个迁移。                                                                                                                                                                      |
| <nobr>幂等</nobr> | 生成脚本，可以在任何迁移的数据库上使用。                                                                                                                                                         |
| -输出\<字符串 >        | 要将结果写入的文件。 如果省略此参数，创建应用程序的运行时文件时，例如使用相同的文件夹中生成的名称创建该文件： */obj/Debug/netcoreapp2.1/ghbkztfz.sql/*。 |

> [!TIP]
> 收件人、、 和输出参数支持选项卡扩展。

以下示例创建一个脚本，以便使用迁移名称 InitialCreate 迁移。

```powershell
Script-Migration -To InitialCreate
```

下面的示例创建一个脚本，以便所有迁移 InitialCreate 迁移之后，使用迁移 id。

```powershell
Script-Migration -From 20180904195021_InitialCreate
```

## <a name="update-database"></a>更新数据库

更新数据库，到最后一个迁移或指定的迁移。

| 参数                           | 描述                                                                                                                                                                                                                                                     |
|:------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>*-迁移*\<字符串 ></nobr> | 目标迁移。 可能会被标识迁移，按名称或 id。 数字 0 是一种特殊情况，意味着*第一次迁移之前*并导致所有迁移操作，从而还原。 如果指定无需迁移，则此命令的最后一个默认迁移。 |

> [!TIP]
> 迁移参数支持选项卡扩展。

下面的示例将恢复所有迁移。

```powershell
Update-Database -Migration 0
```

下面的示例将数据库更新为指定的迁移。 第一个示例使用迁移名称和另一个使用迁移 ID:

```powershell
Update-Database -Migration InitialCreate
Update-Database -Migration 20180904195021_InitialCreate
```

## <a name="additional-resources"></a>其他资源

* [迁移](xref:core/managing-schemas/migrations/index)
* [反向工程](xref:core/managing-schemas/scaffolding)
