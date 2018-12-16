---
title: ASP.NET Core入门 - 新数据库 - EF Core
author: rick-anderson
ms.author: riande
ms.date: 08/03/2018
ms.assetid: e153627f-f132-4c11-b13c-6c9a607addce
uid: core/get-started/aspnetcore/new-db
ms.openlocfilehash: 2248c60045a914c902f1c958a86c69b283abd722
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022231"
---
# <a name="getting-started-with-ef-core-on-aspnet-core-with-a-new-database"></a>使用新数据库在 ASP.NET Core 上开始使用 EF Core

在本教程中，你将使用 Entity Framework Core 构建执行基本数据访问的 ASP.NET Core MVC 应用程序。 该教程使用迁移基于此数据模型创建数据库。

你可在 Windows 上使用 Visual Studio 2017，或在 Windows、macOS 或 Linux 上使用 .NET Core CLI 来学习本教程。

在 GitHub 上查看此文章的示例：
* [Visual Studio 2017 with SQL Server](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb)（配有 SQL Server 的 Visual Studio 2017）
* [.NET Core CLI with SQLite](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb.Sqlite)（配有 SQLite 的 .NET Core CLI）。

## <a name="prerequisites"></a>系统必备

安装以下软件：

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* 具有以下工作负载的 [Visual Studio 2017 15.7 版或更高版本](https://www.visualstudio.com/downloads/)：
  * “ASP.NET 和 Web 开发”（位于“Web 和云”下）
  * “.NET Core 跨平台开发”（位于“其他工具集”下）
* [.NET Core 2.1 SDK](https://www.microsoft.com/net/download/core).

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

* [.NET Core 2.1 SDK](https://www.microsoft.com/net/download/core).

---

## <a name="create-a-new-project"></a>创建新项目

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* 打开 Visual Studio 2017
* “文件”>“新建”>“项目”
* 从左菜单中选择“已安装”>“Visual C#”>“.NET Core”。
* 选择“ASP.NET Core Web 应用程序”。
* 输入“EFGetStarted.AspNetCore.NewDb”作为名称，然后单击“确定”。
* 在“新建 ASP.NET Core Web 应用程序”对话框中：
  * 确保在下拉列表中选择“.NET Core”和“ASP.NET Core 2.1”
  * 选择“Web 应用程序(模型视图控制器)”项目模板
  * 确保将“身份验证”设置为“无身份验证”
  * 单击“确定” 

警告：如果你使用“个人用户帐户”而不是“无身份验证”，则会将 Entity Framework Core 模型添加到 `Models\IdentityModel.cs` 中的项目。 使用在本教程中学习的技巧，可以选择添加第二个模型，或者扩展此现有模型以包含实体类。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

* 运行以下命令以创建 MVC 项目：

   ```cli
   dotnet new mvc -n EFGetStarted.AspNetCore.NewDb
   ```
* 更改为项目目录。 你输入的下一个命令需要针对新项目运行。

   ```cli
   cd EFGetStarted.AspNetCore.NewDb
   ```
---

## <a name="install-entity-framework-core"></a>安装 Entity Framework Core

要安装 EF Core，请为要作为目标对象的 EF Core 数据库提供程序安装程序包。 有关可用提供程序的列表，请参阅[数据库提供程序](../../providers/index.md)。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

对于本教程，无需安装提供程序包，因为本教程使用 SQL Server。 SQL Server 提供程序包包含在 [Microsoft.AspnetCore.App 元包](https://docs.microsoft.com/aspnet/core/fundamentals/metapackage-app?view=aspnetcore-2.1)中。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

本教程使用 SQLite 的原因是，它可在 .NET Core 支持的所有平台上运行。

* 运行以下命令安装 SQLite 提供程序：

   ```cli
   dotnet add package Microsoft.EntityFrameworkCore.Sqlite
   ```

---

## <a name="create-the-model"></a>创建模型

定义构成模型的上下文类和实体类。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* 右键单击“Models”文件夹，然后选择“添加”>“类”。
* 输入“Model.cs”作为名称，然后单击“确定”。
* 将此文件的内容替换为以下代码：

  [!code-csharp[Main](../../../../samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb/Models/Model.cs)]

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

* 在 Models 文件夹中，使用以下代码创建 Model.cs：

  [!code-csharp[Main](../../../../samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb.Sqlite/Models/Model.cs)]

---

生产应用通常会将每个类放在单独的文件中。 为简单起见，本教程将这些类放在一个文件中。

## <a name="register-the-context-with-dependency-injection"></a>使用依赖注入注册上下文

服务（例如 `BloggingContext`）在应用程序启动期间通过[依赖关系注入](http://docs.asp.net/en/latest/fundamentals/dependency-injection.html)进行注册。 需要这些服务的组件（如 MVC 控制器）可以通过向构造函数或属性添加相关参数来获得对应服务。

若要使 `BloggingContext` 对 MVC 控制器可用，请将其注册为服务。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在 Startup.cs 中，添加以下 `using` 语句：

  [!code-csharp[Main](../../../../samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb/Startup.cs#AddedUsings)]

* 将以下突出显示的代码添加到 `ConfigureServices` 方法：

  [!code-csharp[Main](../../../../samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb/Startup.cs?name=ConfigureServices&highlight=12-14)]

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

* 在 Startup.cs 中，添加以下 `using` 语句：

  [!code-csharp[Main](../../../../samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb.Sqlite/Startup.cs#AddedUsings)]

* 将以下突出显示的代码添加到 `ConfigureServices` 方法：

  [!code-csharp[Main](../../../../samples/core/GetStarted/AspNetCore/EFGetStarted.AspNetCore.NewDb.Sqlite/Startup.cs?name=ConfigureServices&highlight=12-14)]

---

生产应用通常会将连接字符串放在配置文件或环境变量中。 为简单起见，本教程在代码中定义它。 有关详细信息，请参阅[连接字符串 ](../../miscellaneous/connection-strings.md)。

## <a name="create-the-database"></a>创建数据库

以下步骤使用[迁移](xref:core/managing-schemas/migrations/index)创建数据库。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* “工具”>“NuGet 包管理器”>“包管理器控制台”
* 运行以下命令：

  ```powershell
  Add-Migration InitialCreate
  Update-Database
  ```

  如果收到错误，指出 `The term 'add-migration' is not recognized as the name of a cmdlet`，请关闭并重新打开 Visual Studio。

  `Add-Migration` 命令为迁移搭建基架，以便为模型创建一组初始表。 `Update-Database` 命令创建数据库并向其应用新的迁移。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

* 运行以下命令：

  ```cli
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```

  `migrations` 命令为迁移搭建基架，以便为模型创建一组初始表。 `database update` 命令创建数据库并向其应用新的迁移。

---

## <a name="create-a-controller"></a>创建控制器

生成 `Blog` 实体控制器和视图的基架。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在“解决方案资源管理器”中，右键单击“控制器”文件夹，然后选择“添加”>“控制器”。
* 选择“视图使用 Entity Framework 的 MVC 控制器”，然后单击“添加”。
* 将“模型类”设置为“Blog”，将“数据上下文类”设置为“BloggingContext”。
* 单击 **添加**。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

* 运行以下命令：

  ```cli
  dotnet tool install -g dotnet-aspnet-codegenerator
  dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
  dotnet restore
  dotnet aspnet-codegenerator controller -name BlogsController -m Blog -dc BloggingContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  `tool install` 和 `add package` 命令安装可以为控制器和视图搭建基架的工具。 `restore` 命令可确保下载所有的项目包，`aspnet-codegenerator` 命令可执行基架搭建。
---

基架引擎创建以下文件：

* 控制器 (Controllers/BlogsController.cs)
* “创建”、“删除”、“详细信息”、“编辑”和“索引”页面的 Razor 视图 (Views/Movies/*.cshtml)

## <a name="run-the-application"></a>运行此应用程序

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* 调试 > 开始执行(不调试)

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```cli
dotnet run
```
---

* 导航到 `/Blogs`

* 使用“新建”链接创建一些博客条目。

  ![创建页面](_static/create.png)

* 测试“详细信息”、“编辑”和“删除”链接。

  ![索引页](_static/index-new-db.png)

## <a name="additional-resources"></a>其他资源

* [教程：使用 SQLite 通过新数据库在 .NET Core 上开始使用 EF Core](xref:core/get-started/netcore/new-db-sqlite)
* [教程：在 ASP.NET Core 中开始使用 Razor Pages](https://docs.microsoft.com/aspnet/core/tutorials/razor-pages/razor-pages-start)
* [教程：ASP.NET Core 中的 Razor Pages 和 Entity Framework Core](https://docs.microsoft.com/aspnet/core/data/ef-rp/intro)
