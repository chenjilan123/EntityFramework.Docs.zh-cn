---
title: UWP 入门 - 新数据库 - EF Core
author: rowanmiller
ms.date: 10/13/2018
ms.assetid: a0ae2f21-1eef-43c6-83ad-92275f9c0727
uid: core/get-started/uwp/getting-started
ms.openlocfilehash: 456967a0dc981053919064a19cc9c98bf7309865
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022371"
---
# <a name="getting-started-with-ef-core-on-universal-windows-platform-uwp-with-a-new-database"></a>通过新数据库在通用 Windows 平台 (UWP) 上开始使用 EF Core

在本教程中，将构建一个通用 Windows 平台 (UWP) 应用程序，该应用程序使用 Entity Framework Core 对本地 SQLite 数据库执行基本数据访问。

[在 GitHub 上查看此文章的示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/GetStarted/UWP)。

## <a name="prerequisites"></a>系统必备

* [Windows 10 Fall Creators Update（10.0；内部版本 16299）或更改版本](https://support.microsoft.com/help/4027667/windows-update-windows-10)。

* 具有通用 Windows 平台开发工作负载的 [Visual Studio 2017 15.7 版或更高版本](https://www.visualstudio.com/downloads/)。

* [.NET Core 2.1 SDK 或更高版本](https://www.microsoft.com/net/core)。

> [!IMPORTANT]
> 本教程使用 Entity Framework Core [迁移](xref:core/managing-schemas/migrations/index)命令创建和更新数据库的架构。
> 这些命令不会直接使用 UWP 项目。
> 为此，应用程序的数据模型位于共享库项目中，并且单独的 .NET Core 控制台应用程序用于运行这些命令。

## <a name="create-a-library-project-to-hold-the-data-model"></a>创建库项目以存放数据模型

* 打开 Visual Studio

* “文件”>“新建”>“项目”

* 从左侧菜单中选择“已安装”>“Visual C#”>“.NET Standard”。

* 选择“类库(.NET Standard)”模板。

* 将该项目命名为 Blogging.Model。

* 将该解决方案命名为 Blogging。

* 单击 **“确定”**。

## <a name="install-entity-framework-core-runtime-in-the-data-model-project"></a>在数据模型项目中安装 Entity Framework Core 运行时

要使用 EF Core，请为要作为目标对象的数据库提供程序安装程序包。 本教程使用 SQLite。 有关可用提供程序的列表，请参阅[数据库提供程序](../../providers/index.md)。

* “工具”>“NuGet 包管理器”>“包管理器控制台”。

* 确保在包管理器控制台中选择库项目 Blogging.Model 作为默认项目。

* 运行 `Install-Package Microsoft.EntityFrameworkCore.Sqlite`

## <a name="create-the-data-model"></a>创建数据模型

现在是时候定义构成模型的 DbContext 和实体类了。

* 删除 Class1.cs。

* 用以下代码创建 Model.cs：

  [!code-csharp[Main](../../../../samples/core/GetStarted/UWP/Blogging.Model/Model.cs)]

## <a name="create-a-new-console-project-to-run-migrations-commands"></a>创建新的控制台项目以运行迁移命令

* 在解决方案资源管理器中，右键单击该解决方案，然后选择“添加”>“新项目”。

* 从左菜单中选择“已安装”>“Visual C#”>“.NET Core”。

* 选择“控制台应用程序(.NET Core)”项目模板。

* 将该项目命名为 Blogging.Migrations.Startup ，然后单击“确定”。

* 将 Blogging.Migrations.Startup 项目中的项目引用添加到 Blogging.Model 项目。

## <a name="install-entity-framework-core-tools-in-the-migrations-startup-project"></a>在迁移启动项目中安装 Entity Framework Core 工具

若要在包管理器控制台中启用 EF Core 迁移命令，请在控制台应用程序中安装 EF Core 工具包。

* “工具”>“NuGet 包管理器”>“包管理器控制台”

* 运行 `Install-Package Microsoft.EntityFrameworkCore.Tools -ProjectName Blogging.Migrations.Startup`

## <a name="create-the-initial-migration"></a>创建初始迁移

 创建初始迁移，将控制台应用程序指定为启动项目。

* 运行 `Add-Migration InitialCreate -StartupProject Blogging.Migrations.Startup`

此命令为迁移搭建基架，该迁移为数据模型创建一组初始数据库表。

## <a name="create-the-uwp-project"></a>创建 UWP 项目

* 在解决方案资源管理器中，右键单击该解决方案，然后选择“添加”>“新项目”。

* 从左侧菜单中选择“已安装”>“Visual C#”>“Windows Universal”。

* 选择“空白应用(通用 Windows)”项目模板。

* 将项目命名为 Blogging.UWP ，然后单击“确定”

> [!IMPORTANT]
> 将目标和最低版本设置为至少“Windows 10 Fall Creators Update (10.0；内部版本 16299.0)”。
> 早期版本的 Windows 10 不支持 .NET Standard 2.0，但它是 Entity Framework Core 必需的。

## <a name="add-code-to-create-the-database-on-application-startup"></a>添加代码以在应用程序启动时创建数据库

由于希望在运行该应用的设备上创建数据库，因此，将添加代码，以便在应用程序启动时将任何待定迁移应用到本地数据库。 应用第一次运行时，该操作将创建本地数据库。

* 将 Blogging.UWP 项目中的项目引用添加到 Blogging.Model 项目。

* 打开 App.xaml.cs。

* 添加突出显示的代码，应用任何待定迁移。

  [!code-csharp[Main](../../../../samples/core/GetStarted/UWP/Blogging.UWP/App.xaml.cs?highlight=1-2,26-29)]

> [!TIP]  
> 如果对模型进行了更改，则请使用 `Add-Migration` 命令为新迁移搭建基架，以便将相应更改应用到数据库。 当应用程序启动时，任何待定迁移都将应用到每个设备上的本地数据库。
>
>EF Core 使用数据库中的 `__EFMigrationsHistory` 表来跟踪哪些迁移已应用到数据库。

## <a name="use-the-data-model"></a>使用数据模型

现在可以使用 EF Core 执行数据访问。

* 打开 MainPage.xaml.

* 添加下面突出显示的网页加载处理程序和 UI 内容

[!code-xml[Main](../../../../samples/core/GetStarted/UWP/Blogging.UWP/MainPage.xaml?highlight=9,11-23)]

现在，添加代码来连接 UI 和数据库

* 打开 MainPage.xaml.cs.

* 从以下列表中添加突出显示的代码：

[!code-csharp[Main](../../../../samples/core/GetStarted/UWP/Blogging.UWP/MainPage.xaml.cs?highlight=1,31-49)]

现在可以运行应用程序来查看其实际运行情况。

* 在解决方案资源管理器中，右键单击Blogging.UWP 项目，然后选择“部署”。

* 将 Blogging.UWP 设置为启动项目。

* “调试”>“开始执行(不调试)”

  将生成并运行测试应用。

* 输入 URL，然后单击“添加”按钮

  ![图像](_static/create.png)

  ![图像](_static/list.png)

  哇哦！ 现在便有了一个运行 Entity Framework Core 的简单 UWP 应用。

## <a name="next-steps"></a>后续步骤

有关结合使用 EF Core 和 UWP 时应了解的兼容性和性能信息，请参阅 [EF Core 支持的 .NET 实现](../../platforms/index.md#universal-windows-platform)。

查阅本文档中的其他文章，深入了解 Entity Framework Core 功能。
