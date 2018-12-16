---
title: EF Core 工具参考 (.NET CLI)-EF Core
author: bricelam
ms.author: bricelam
ms.date: 09/20/2018
uid: core/miscellaneous/cli/dotnet
ms.openlocfilehash: 959785c7b10ca668f3691106f62076d538978c03
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688662"
---
# <a name="entity-framework-core-tools-reference---net-cli"></a>Entity Framework Core 工具引用的.NET CLI

Entity Framework Core 的命令行接口 (CLI) 工具执行设计时开发任务。 例如，创建方法[迁移](/aspnet/core/data/ef-mvc/migrations?view=aspnetcore-2.0#introduction-to-migrations)、 应用迁移，并为基于现有数据库的模型生成代码。 这些命令是对跨平台扩展[dotnet](/dotnet/core/tools)命令，它是一部分的[.NET Core SDK](https://www.microsoft.com/net/core)。 这些工具适用于.NET Core 项目。

如果使用 Visual Studio，我们建议[程序包管理器控制台工具](powershell.md)改为：
* 它们可自动处理与当前项目中所选**程序包管理器控制台**而无需手动切换目录。
* 它们会自动打开由命令在命令完成后生成的文件。

## <a name="installing-the-tools"></a>安装工具

安装过程取决于项目类型和版本：

* ASP.NET Core 2.1 和更高版本
* EF Core 2.x
* EF Core 1.x

### <a name="aspnet-core-21"></a>ASP.NET Core 2.1 +

* 安装最新[.NET Core SDK](https://www.microsoft.com/net/download/core)。 即使有 Visual Studio 2017 的最新版本，也必须安装 SDK。

  这是因为适用于 ASP.NET Core 2.1 + 所需的所有`Microsoft.EntityFrameworkCore.Design`包包含在[Microsoft.AspNetCore.App 元包](/aspnet/core/fundamentals/metapackage-app)。

### <a name="ef-core-2x-not-aspnet-core"></a>EF Core 2.x (而不是 ASP.NET Core)

`dotnet ef`命令包含在.NET Core SDK，但若要启用这些命令必须安装`Microsoft.EntityFrameworkCore.Design`包。

* 安装最新[.NET Core SDK](https://www.microsoft.com/net/download/core)。 即使有 Visual Studio 2017 的最新版本，也必须安装 SDK。

* 安装最新稳定`Microsoft.EntityFrameworkCore.Design`包。

  ``` Console
  dotnet add package Microsoft.EntityFrameworkCore.Design
  ```

### <a name="ef-core-1x"></a>EF Core 1.x

* 安装.NET Core SDK 版本 2.1.200。 更高版本不兼容使用 EF Core 1.0 和 1.1 的 CLI 工具。

* 通过修改配置为使用 2.1.200 SDK 版本的应用程序及其[global.json](/dotnet/core/tools/global-json)文件。 此文件通常包含在解决方案目录 （一个项目上方）。

* 编辑项目文件并添加`Microsoft.EntityFrameworkCore.Tools.DotNet`作为`DotNetCliToolReference`项。 指定的最新的 1.x 版本，例如： 1.1.6。 请参阅本部分的结尾处的项目文件示例。

* 安装的最新的 1.x 版本`Microsoft.EntityFrameworkCore.Design`包，例如：

  ```console
  dotnet add package Microsoft.EntityFrameworkCore.Design -v 1.1.6
  ```

  与添加这两个包引用，项目文件如下所示：

  ``` xml
  <Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
      <OutputType>Exe</OutputType>
      <TargetFramework>netcoreapp1.1</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
      <PackageReference Include="Microsoft.EntityFrameworkCore.Design"
                        Version="1.1.6"
                         PrivateAssets="All" />
    </ItemGroup>
    <ItemGroup>
       <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet"
                              Version="1.1.6" />
    </ItemGroup>
  </Project>
  ```

  使用的包引用`PrivateAssets="All"`不公开到引用此项目的项目。 此限制是在开发过程中通常仅使用的包尤其有用。

### <a name="verify-installation"></a>验证安装

运行以下命令以验证正确安装 EF Core CLI 工具：

  ``` Console
  dotnet restore
  dotnet ef
  ```

该命令的输出标识的版本中使用的工具：

```console

                     _/\__
               ---==/    \\
         ___  ___   |.    \|\
        | __|| __|  |  )   \\\
        | _| | _|   \_/ |  //|\\
        |___||_|       /   \\\/\\

Entity Framework Core .NET Command-line Tools 2.1.3-rtm-32065

<Usage documentation follows, not shown.>
```

## <a name="using-the-tools"></a>使用的工具

工具使用之前，可能需要创建一个启动项目或将环境设置。

### <a name="target-project-and-startup-project"></a>目标项目和启动项目

命令是指*项目*和一个*启动项目*。

* *项目*也称为*目标项目*因为它是命令添加或删除文件的位置。 默认情况下，当前目录中的项目为目标项目。 可以通过使用作为目标项目中指定一个不同的项目<nobr> `--project` </nobr>选项。

* *启动项目*是指的工具生成和运行。 这些工具必须在设计时，获取有关该项目，如数据库连接字符串和模型的配置信息执行应用程序代码。 默认情况下，当前目录中的项目为启动项目。 您可以通过使用不同的项目指定为启动项目<nobr> `--startup-project` </nobr>选项。

启动项目和目标项目通常是在同一个项目。 它们的单独的项目的典型情况是当：

* EF Core 上下文和实体类是在.NET Core 类库中。
* .NET Core 控制台应用程序或 web 应用程序引用的类库。

此外，还可以向[迁移代码置于独立于 EF Core 上下文类库](xref:core/managing-schemas/migrations/projects)。

### <a name="other-target-frameworks"></a>其他目标框架

CLI 工具适用于.NET Core 项目和.NET Framework 项目。 EF Core 模型具有.NET Standard 类库中的应用可能不具有.NET Core 或.NET Framework 项目。 例如，这是 Xamarin 和通用 Windows 平台应用，则返回 true。 在这种情况下，可以创建.NET Core 控制台应用项目，其唯一用途是作为启动项目的工具。 项目可以是虚拟项目不包含实际代码&mdash;只需为工具提供一个目标。

为什么是虚拟的项目所需？ 如前文所述，这些工具必须在设计时执行应用程序代码。 若要做到这一点，他们需要使用.NET Core 运行时。 EF Core 模型在面向.NET Core 或.NET Framework 的项目时，EF Core 工具借用运行时从该项目。 如果在 EF Core 模型处于.NET Standard 类库，他们无法做到这一点。 .NET Standard 并不实际的.NET 实现;它是一种规范的一组.NET 实现必须支持的 Api。 因此.NET Standard 是不够的 EF Core 工具来执行应用程序代码。 创建要用作启动项目的虚拟项目提供了工具可以在其中加载.NET Standard 类库的具体目标平台。

### <a name="aspnet-core-environment"></a>ASP.NET Core 环境

若要指定用于 ASP.NET Core 项目的环境，请设置**ASPNETCORE_ENVIRONMENT**之前运行命令的环境变量。

## <a name="common-options"></a>常用选项

|                   | 选项                            | 描述                                                                                                                                                                                                                                                   |
|:------------------|:----------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                   | `--json`                          | 显示 JSON 输出。                                                                                                                                                                                                                                             |
| <nobr>`-c`</nobr> | `--context <DBCONTEXT>`           | `DbContext`类使用。 唯一或完全限定的命名空间的类名称。  如果省略此选项，则 EF Core 会发现上下文类。 如果有多个上下文类，则需要此选项。                                            |
| `-p`              | `--project <PROJECT>`             | 目标项目的项目文件夹的相对路径。  默认值为当前文件夹。                                                                                                                                                              |
| `-s`              | `--startup-project <PROJECT>`     | 为启动项目的项目文件夹的相对路径。 默认值为当前文件夹。                                                                                                                                                              |
|                   | `--framework <FRAMEWORK>`         | [目标框架名字对象](/dotnet/standard/frameworks#supported-target-framework-versions)有关[目标框架](/dotnet/standard/frameworks)。  使用项目文件指定多个目标框架，并且想要选择其中之一时。 |
|                   | `--configuration <CONFIGURATION>` | 生成配置，例如：`Debug`或`Release`。                                                                                                                                                                                                   |
|                   | `--runtime <IDENTIFIER>`          | 若要还原的包的目标运行时标识符。 有关运行时标识符 (RID) 的列表，请参阅 [RID 目录](/dotnet/core/rid-catalog)。                                                                                                      |
| `-h`              | `--help`                          | 显示帮助信息。                                                                                                                                                                                                                                        |
| `-v`              | `--verbose`                       | 显示详细输出。                                                                                                                                                                                                                                          |
|                   | `--no-color`                      | 不为着色输出。                                                                                                                                                                                                                                        |
|                   | `--prefix-output`                 | 输出级别使用的前缀。                                                                                                                                                                                                                                     |

## <a name="dotnet-ef-database-drop"></a>dotnet ef 数据库拖放

删除数据库。

选项:

|                   | 选项                   | 描述                                              |
|:------------------|:-------------------------|:---------------------------------------------------------|
| <nobr>`-f`</nobr> | <nobr>`--force`</nobr>   | 不确认。                                           |
|                   | <nobr>`--dry-run`</nobr> | 显示哪个数据库会被丢弃，但没有删除它。 |

## <a name="dotnet-ef-database-update"></a>dotnet ef 数据库更新

更新数据库，到最后一个迁移或指定的迁移。

参数：

| 参数      | 描述                                                                                                                                                                                                                                                     |
|:--------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `<MIGRATION>` | 目标迁移。 可能会被标识迁移，按名称或 id。 数字 0 是一种特殊情况，意味着*第一次迁移之前*并导致所有迁移操作，从而还原。 如果指定无需迁移，则此命令的最后一个默认迁移。 |

下面的示例将数据库更新为指定的迁移。 第一个示例使用迁移名称和另一个使用迁移 ID:

```console
dotnet ef database update InitialCreate
dotnet ef database update 20180904195021_InitialCreate
```

## <a name="dotnet-ef-dbcontext-info"></a>dotnet ef dbcontext 信息

获取有关的信息`DbContext`类型。

## <a name="dotnet-ef-dbcontext-list"></a>dotnet ef dbcontext 列表

列出可用`DbContext`类型。

## <a name="dotnet-ef-dbcontext-scaffold"></a>dotnet ef dbcontext 基架

为生成代码`DbContext`和数据库的实体类型。 为了使此命令来生成实体类型，数据库表必须具有主键。

参数：

| 参数       | 描述                                                                                                                                                                                                             |
|:---------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `<CONNECTION>` | 数据库的连接字符串。 对于 ASP.NET Core 2.x 项目，值可以是*名称 =\<的连接字符串名称 >*。 在这种情况下名称来自于为项目设置的配置源。 |
| `<PROVIDER>`   | 要使用的提供程序。 通常这是 NuGet 包的名称为例： `Microsoft.EntityFrameworkCore.SqlServer`。                                                                                           |

选项:

|                 | 选项                                   | 描述                                                                                                                                                                    |
|:----------------|:-----------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <nobr>-d</nobr> | `--data-annotations`                     | 特性用于将模型配置 （如果可能）。 如果省略此选项，则使用仅 fluent API。                                                                |
| `-c`            | `--context <NAME>`                       | 名称`DbContext`类生成。                                                                                                                                 |
|                 | `--context-dir <PATH>`                   | 要放置的目录`DbContext`中的类文件。 路径是相对于项目目录。 命名空间派生自文件夹名称。                                 |
| `-f`            | `--force`                                | 覆盖现有文件。                                                                                                                                                      |
| `-o`            | `--output-dir <PATH>`                    | 要将实体类文件放入的目录。 路径是相对于项目目录。                                                                                       |
|                 | <nobr>`--schema <SCHEMA_NAME>...`</nobr> | 要生成的实体类型的表架构。 若要指定多个架构，请重复`--schema`为每个。 如果省略此选项，则包括所有架构。          |
| `-t`            | `--table <TABLE_NAME>`...                | 要生成的实体类型的表。 若要指定多个表，请重复`-t`或`--table`为每个。 如果省略此选项，则包括所有表。                |
|                 | `--use-database-names`                   | 在数据库中显示的完全相同，请使用表和列的名称。 如果省略此选项，则会更改数据库名称，使其更紧密地符合 C# 名称样式约定。 |

下面的示例搭建基架以所有架构和表，并将新文件放入*模型*文件夹。

```console
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models
```

下面的示例搭建基架以选定的表，并具有指定名称的单独文件夹中创建的上下文：

```console
dotnet ef dbcontext scaffold "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models -t Blog -t Post --context-dir Context -c BlogContext
```

## <a name="dotnet-ef-migrations-add"></a>添加 dotnet ef 迁移

添加新的迁移。

参数：

| 参数 | 描述                |
|:---------|:---------------------------|
| `<NAME>` | 迁移的名称。 |

选项:

|                   | 选项                             | 描述                                                                                                      |
|:------------------|:-----------------------------------|:-----------------------------------------------------------------------------------------------------------------|
| <nobr>`-o`</nobr> | <nobr>`--output-dir <PATH>`</nobr> | 目录 （及其子命名空间） 来使用。 路径是相对于项目目录。 默认值为"迁移"。 |

## <a name="dotnet-ef-migrations-list"></a>dotnet ef 迁移列表

列出了可用的迁移。

## <a name="dotnet-ef-migrations-remove"></a>dotnet ef 迁移删除

删除 （请回滚迁移已完成的代码更改） 的最后一个迁移。

选项:

|                   | 选项    | 描述                                                                     |
|:------------------|:----------|:--------------------------------------------------------------------------------|
| <nobr>`-f`</nobr> | `--force` | 还原迁移 （请回滚已应用到数据库的更改）。 |

## <a name="dotnet-ef-migrations-script"></a>dotnet ef 迁移脚本

从迁移中生成的 SQL 脚本。

参数：

| 参数 | 描述                                                                                                                                                   |
|:---------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `<FROM>` | 开始迁移。 可能会被标识迁移，按名称或 id。 数字 0 是一种特殊情况，意味着*第一次迁移之前*。 默认值为 0。 |
| `<TO>`   | 结束的迁移。 默认的最后一个迁移。                                                                                                         |

选项:

|                   | 选项            | 描述                                                        |
|:------------------|:------------------|:-------------------------------------------------------------------|
| <nobr>`-o`</nobr> | `--output <FILE>` | 要写入到脚本的文件。                                   |
| `-i`              | `--idempotent`    | 生成脚本，可以在任何迁移的数据库上使用。 |

以下示例创建一个脚本，以便 InitialCreate 迁移：

```console
dotnet ef migrations script 0 InitialCreate
```

下面的示例创建 InitialCreate 迁移后迁移的所有脚本。

```console
dotnet ef migrations script 20180904195021_InitialCreate
```

## <a name="additional-resources"></a>其他资源

* [迁移](xref:core/managing-schemas/migrations/index)
* [反向工程](xref:core/managing-schemas/scaffolding)
