---
title: 安装 Entity Framework Core
author: divega
ms.date: 08/06/2017
ms.assetid: 608cc774-c570-4809-8a3e-cd2c8446b8b2
uid: core/get-started/install/index
ms.openlocfilehash: 58c79d477d590eea355a922b3e1233bbecb305cc
ms.sourcegitcommit: a6082a2caee62029f101eb1000656966195cd6ee
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/10/2018
ms.locfileid: "53181976"
---
# <a name="installing-entity-framework-core"></a>安装 Entity Framework Core

## <a name="prerequisites"></a>系统必备

* EF Core 是一个 [.NET Standard 2.0](/dotnet/standard/net-standard) 库。 因此，EF Core 需要支持运行 .NET Standard 2.0 的实现。 其他 .NET Standard 2.0 库也可引用 EF Core。 

* 例如，可使用 EF Core 开发面向 .NET Core 的应用。 生成 .NET Core 应用需要 [.NET Core SDK](https://dotnet.microsoft.com/download)。 还可选择使用 Visual Studio、Visual Studio for Mac 或 Visual Studio Code 等开发环境。 有关详细信息，请参阅 [.NET Core 入门](/dotnet/core/get-started)。

* 可在 Windows 上的 Visual Studio 中使用 EF Core 来开发面向 .NET Framework 4.6.1 或更高版本的应用程序。 建议使用最新版本的 Visual Studio。 如果想要使用较旧版本（例如 Visual Studio 2015），请确保[将 NuGet 客户端升级到版本 3.6.0](https://www.nuget.org/downloads)，以便使用 .NET Standard 2.0 库。

* EF Core 可以在 Xamarin 和 .NET Native 等其他 .NET 实现上运行。 但在实践中，这些实现具有运行时限制，可能会影响 EF Core 处理应用的效率。 有关详细信息，请参阅 [EF Core 支持的 .NET 实现](xref:core/platforms/index)。

* 最后，不同的数据库提供程序可能需要特定的数据库引擎版本、.NET 实现或操作系统。 请确保可用的 [EF Core 数据库提供程序](xref:core/providers/index)支持适用于应用程序的环境。

## <a name="get-the-entity-framework-core-runtime"></a>获取 Entity Framework Core 运行时

要将 EF Core 添加到应用程序，请安装适用于要使用的数据库提供程序的 NuGet 包。

如果要生成 ASP.NET Core 应用程序，不需要安装内存中和 SQL Server 提供程序。 这些提供程序随 EF Core 运行时一起包含在当前版本的 ASP.NET Core 中。  

要安装或更新 NuGet 包，可以使用 [.NET Core 命令行界面 (CLI)、Visual Studio 包管理器对话框或 Visual Studio 包管理器控制台。

### <a name="net-core-cli"></a>.NET Core CLI

* 在操作系统的命令行中使用以下 .NET Core CLI 命令来安装或更新 EF Core SQL Server 提供程序：

  ``` Console
  dotnet add package Microsoft.EntityFrameworkCore.SqlServer
  ```

* 可以使用 `-v` 修饰符在 `dotnet add package` 命令中指明特定的版本。 例如，若要安装 EF Core 2.2.0 包，请将 `-v 2.2.0` 追加到命令中。

有关详细信息，请参阅 [.NET 命令行接口 (CLI) 工具](/dotnet/core/tools/)。

### <a name="visual-studio-nuget-package-manager-dialog"></a>Visual Studio NuGet 包管理器对话框

* 从 Visual Studio 菜单中选择“项目”>“管理 NuGet 包”

* 单击“浏览”或“更新”选项卡

* 若要安装或更新 SQL Server 提供程序，请选择 `Microsoft.EntityFrameworkCore.SqlServer` 包并确认。

有关详细信息，请参阅 [NuGet 包管理器对话框](/nuget/tools/package-manager-ui)。

### <a name="visual-studio-nuget-package-manager-console"></a>Visual Studio NuGet 包管理器控制台

* 从 Visual Studio 菜单中选择“工具”>“NuGet 包管理器”>“包管理器控制台”

* 若要安装 SQL Server 提供程序，请在包管理器控制台中运行以下命令：

  ``` PowerShell  
  Install-Package Microsoft.EntityFrameworkCore.SqlServer
  ```
* 若要更新提供程序，使用 `Update-Package` 命令。

* 若要指定特定版本，可以使用 `-Version` 修饰符。 例如，若要安装 EF Core 2.2.0 包，请将 `-Version 2.2.0` 追加到命令中

有关详细信息，请参阅[包管理器控制台](/nuget/tools/package-manager-console)。

## <a name="get-the-entity-framework-core-tools"></a>获取 Entity Framework Core 工具

可以安装工具来执行项目中与 EF Core 相关的任务，例如创建和应用数据库迁移，或基于现有数据库创建 EF Core 模型。

提供了两个工具集：

* [.NET Core 命令行接口 (CLI) 工具](xref:core/miscellaneous/cli/dotnet)可用于 Windows、Linux 或 macOS。 这些命令以 `dotnet ef` 开头。 

* [包管理器控制台 (PMC) 工具](xref:core/miscellaneous/cli/powershell)在 Windows 上的 Visual Studio 中运行。 这些命令以动词开头，例如 `Add-Migration`、`Update-Database`。

虽然也可在包管理器控制台中使用 `dotnet ef` 命令，但在使用 Visual Studio 时建议使用包管理器控制台工具：

* 它们会自动使用在 Visual Studio 的 PMC 中选择的当前项目，无需手动切换目录。  

* 命令完成后，它们会自动在 Visual Studio 中打开命令所生成的文件。

<a name="cli"></a>

### <a name="get-the-net-core-cli-tools"></a>获取 .NET Core CLI 工具

.NET core CLI 工具需要前面的[系统必备](#prerequisites)中提到的 .NET Core SDK。

`dotnet ef` 命令包含在当前版本的 .NET Core SDK 中，但若要对特定命令启用该命令，必须安装 `Microsoft.EntityFrameworkCore.Design` 包：

 ``` Console    
dotnet add package Microsoft.EntityFrameworkCore.Design 
``` 

ASP.NET Core 应用自动随附此包。

> [!IMPORTANT]      
> 请务必使用与运行时包主版本匹配的工具包版本。

### <a name="get-the-package-manager-console-tools"></a>获取包管理器控制台工具

若要获取适用于 EF Core 的包管理器控制台工具，请安装 `Microsoft.EntityFrameworkCore.Tools` 包。 例如，在 Visual Studio 中：

``` PowerShell  
Install-Package Microsoft.EntityFrameworkCore.Tools
``` 

ASP.NET Core 应用自动随附此包。

## <a name="upgrading-to-the-latest-ef-core"></a>升级到最新版本的 EF Core

* 每次发布新版本的 EF Core 时，我们都还会在 EF Core 项目中发布新版本的提供程序，如 Microsoft.EntityFrameworkCore.SqlServer、Microsoft.EntityFrameworkCore.Sqlite 和 Microsoft.EntityFrameworkCore.InMemory。 只需升级到新版本的提供程序即可获取所有改进。 

* EF Core 以及 SQL Server 和内存中提供程序包含在当前版本的 ASP.NET Core 中。 若要将现有 ASP.NET Core 应用程序升级到较新版本的 EF Core，请务必升级 ASP.NET Core 的版本。

* 如需更新使用第三方数据库提供程序的应用程序，请始终检查与要使用的 EF Core 版本兼容的提供程序有无更新。 例如，早期版本的数据库提供程序不兼容 2.0 版 EF Core 运行时。

* 用于 EF Core 的第三方提供程序通常不随 EF Core 运行发布修补程序版本。 若要将使用第三方提供程序的应用程序升级到 EF Core 的修补程序版本，可能需要添加对单独的 EF Core 运行时组件（如 Microsoft.EntityFrameworkCore 和 Microsoft.EntityFrameworkCore.Relational）的直接引用。

* 若要将现有应用程序升级到最新版本的 EF Core，可能需要手动删除一些对旧版 EF Core 包的引用：

  * EF Core 2.0 及更高版本不再需要或支持 `Microsoft.EntityFrameworkCore.SqlServer.Design` 等数据库提供程序设计时包，但在升级其他包后，它们不会被自动删除。

  * 自版本 2.1 起的 .NET SDK 包含 .NET CLI 工具，这样就可以从项目文件中删除对相应包的引用：

    ```
    <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
    ```

* 面向 .NET Framework 的应用程序可能需要更改为使用 .NET Standard 2.0 库：

  * 编辑项目文件，并确保以下条目出现在初始属性组中：

    ``` xml
    <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>
    ```

  * 对于测试项目，还要确保存在以下条目：

    ``` xml
    <GenerateBindingRedirectsOutputType>true</GenerateBindingRedirectsOutputType>
    ```
