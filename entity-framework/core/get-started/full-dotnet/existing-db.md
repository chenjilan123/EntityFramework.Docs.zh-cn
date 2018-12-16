---
title: .NET Framework 入门 - 现有数据库 - EF Core
author: rowanmiller
ms.date: 08/06/2018
ms.assetid: a29a3d97-b2d8-4d33-9475-40ac67b3b2c6
uid: core/get-started/full-dotnet/existing-db
ms.openlocfilehash: 1b90c491a3b2025da750a3266ff45d9d92bb1d0d
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688610"
---
# <a name="getting-started-with-ef-core-on-net-framework-with-an-existing-database"></a>通过现有数据库在 .NET Framework 上开始使用 EF Core

在本教程中，你将构建一个控制台应用程序，它使用 Entity Framework 对 Microsoft SQL Server 数据库执行基本数据访问。 通过对现有数据库进行反向工程，创建 Entity Framework 模型。

[在 GitHub 上查看此文章的示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/GetStarted/FullNet/ConsoleApp.ExistingDb)。

## <a name="prerequisites"></a>系统必备

* [Visual Studio 2017 版本 15.7 或更高版本](https://www.visualstudio.com/downloads/)

## <a name="create-blogging-database"></a>创建博客数据库

本教程使用 LocalDb 实例上的博客数据库作为现有数据库。 如果已在其他教程中创建了博客数据库，请跳过这些步骤。

* 打开 Visual Studio

* “工具”->“连接到数据库...”

* 选择“Microsoft SQL Server”，然后单击“继续”

* 输入“(localdb)\mssqllocaldb”作为服务器名称

* 输入“master”作为数据库名称，然后单击“确定”

* Master 数据库现在显示在“服务器资源管理器”的“数据连接”中

* 右键单击“服务器资源管理器”中的数据库，然后选择“新建查询”

* 将下列脚本复制到查询编辑器中

* 右键单击查询编辑器，然后选择“执行”

[!code-sql[Main](../_shared/create-blogging-database-script.sql)]

## <a name="create-a-new-project"></a>创建新项目

* 打开 Visual Studio 2017

* “文件”>“新建”>“项目...”

* 从左侧菜单中选择“已安装”>“Visual C#”->“Windows Desktop”

* 选择“控制台应用(.NET Framework)”项目模板

* 确保项目面向 **.NET Framework 4.6.1** 或更高版本

* 将项目命名为 ConsoleApp.ExistingDb，并单击“确定”

## <a name="install-entity-framework"></a>安装 Entity Framework

要使用 EF Core，请为要作为目标对象的数据库提供程序安装程序包。 本教程使用 SQL Server。 有关可用提供程序的列表，请参阅[数据库提供程序](../../providers/index.md)。

* “工具”>“NuGet 包管理器”>“包管理器控制台”

* 运行 `Install-Package Microsoft.EntityFrameworkCore.SqlServer`

下一步，要使用一些 Entity Framework Tools 对该数据库进行反向工程。 因此，请同时安装该工具包。

* 运行 `Install-Package Microsoft.EntityFrameworkCore.Tools`

## <a name="reverse-engineer-the-model"></a>对模型实施反向工程

现在是时候基于现有数据库创建 EF 模型了。

* “工具”–>“NuGet 包管理器”–>“包管理器控制台”

* 运行以下命令以从现有数据库创建模型

  ``` powershell
  Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer
  ```

> [!TIP]  
> 可以通过将 `-Tables` 参数添加到上述命令来指定要为其生成实体的表。 例如 `-Tables Blog,Post`。

反向工程过程基于现有数据库的架构创建实体类（`Blog` 和 `Post`）和派生上下文（`BloggingContext`）。

实体类是简单的 C# 对象，代表要查询和保存的数据。 以下是 `Blog` 和 `Post` 实体类：

 [!code-csharp[Main](../../../../samples/core/GetStarted/FullNet/ConsoleApp.ExistingDb/Blog.cs)]

[!code-csharp[Main](../../../../samples/core/GetStarted/FullNet/ConsoleApp.ExistingDb/Post.cs)]

> [!TIP]  
> 若要启用延迟加载，可以创建导航属性 `virtual`（Blog.Post 和 Post.Blog）。

上下文表示与数据库的会话。 其中有可用于查询和保存实体类实例的方法。

[!code-csharp[Main](../../../../samples/core/GetStarted/FullNet/ConsoleApp.ExistingDb/BloggingContext.cs)]

## <a name="use-the-model"></a>使用模型

现在可以使用模型执行数据访问。

* 打开 Program.cs

* 将此文件的内容替换为以下代码

  [!code-csharp[Main](../../../../samples/core/GetStarted/FullNet/ConsoleApp.ExistingDb/Program.cs)] 

* “调试”>“开始执行(不调试)”

  可以看到，把一个博客保存到数据库后，所有博客的详细信息都将显示在控制台中。

  ![图像](_static/output-existing-db.png)

## <a name="next-steps"></a>后续步骤

有关如何构建上下文和实体类的详细信息，请参阅以下文章：
* [反向工程](xref:core/managing-schemas/scaffolding)
* [Entity Framework Core 工具参考 - .NET CLI](xref:core/miscellaneous/cli/dotnet#dotnet-ef-dbcontext-scaffold)
* [Entity Framework Core 工具参考 - 程序包管理器控制台](xref:core/miscellaneous/cli/powershell#scaffold-dbcontext)