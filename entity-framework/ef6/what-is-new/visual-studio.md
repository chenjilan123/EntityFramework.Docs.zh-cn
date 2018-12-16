---
title: Visual Studio 版本的 EF6
author: divega
ms.date: 07/05/2018
ms.assetid: 028FF890-4EDB-4F03-AE53-72F9C33EC92F
ms.openlocfilehash: 16bcdc6d0e7c5632d4f4c06ba285a7a666f24204
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022244"
---
# <a name="visual-studio-releases"></a>Visual Studio 版本

我们建议始终使用最新版本的 Visual Studio，因为它包含用于.NET、 NuGet 和实体框架的最新的工具。
事实上，各种示例和跨实体框架文档的演练假定您使用最新版本的 Visual Studio。

但是，若要使用较旧版本的 Visual Studio 使用不同版本的实体框架，只要您考虑到帐户一些差异，可以：

## <a name="visual-studio-2017-157-and-newer"></a>Visual Studio 2017 15.7年及更高版本

- 此版本的 Visual Studio 包括最新版本的实体框架工具和 EF 6.2 运行时，并且不需要额外的步骤。
请参阅[What's New](~/ef6/what-is-new/index.md)有关这些版本的详细信息。
- 将实体框架添加到新项目使用 EF 工具将自动添加 EF 6.2 NuGet 包。
可以手动安装或联机升级到可用任何 EF NuGet 包。
- 默认情况下，适用于此版本的 Visual Studio 的 SQL Server 实例是一个 MSSQLLocalDB 命名 LocalDB 实例。
连接字符串应使用的服务器部分是"(localdb)\\MSSQLLocalDB"。
请记住使用前缀为原义字符串`@`声或两个反斜杠"\\\\"在 C# 代码中指定的连接字符串时。  


## <a name="visual-studio-2015-to-visual-studio-2017-156"></a>Visual Studio 2015，转至 Visual Studio 2017 15.6 版

- 这些版本的 Visual Studio 包括实体框架工具和运行时 6.1.3。
请参阅[过去释放](~/ef6/what-is-new/past-releases.md#ef-613)有关这些版本的详细信息。
- 将实体框架添加到新项目使用 EF 工具将自动添加 EF 6.1.3 NuGet 包。
可以手动安装或联机升级到可用任何 EF NuGet 包。
- 默认情况下，适用于此版本的 Visual Studio 的 SQL Server 实例是一个 MSSQLLocalDB 命名 LocalDB 实例。
连接字符串应使用的服务器部分是"(localdb)\\MSSQLLocalDB"。
请记住使用前缀为原义字符串`@`声或两个反斜杠"\\\\"在 C# 代码中指定的连接字符串时。  


## <a name="visual-studio-2013"></a>Visual Studio 2013
- 此版本的 Visual Studio 包括和旧版本的实体框架工具和运行时。
我们建议您升级到 Entity Framework Tools 6.1.3，使用[安装程序](https://www.microsoft.com/download/details.aspx?id=40762)Microsoft Download Center 中可用。
请参阅[过去释放](~/ef6/what-is-new/past-releases.md#ef-613)有关这些版本的详细信息。
- 将实体框架添加到新项目使用升级后的 EF 工具将自动添加 EF 6.1.3 NuGet 包。
可以手动安装或联机升级到可用任何 EF NuGet 包。
- 默认情况下，适用于此版本的 Visual Studio 的 SQL Server 实例是一个 MSSQLLocalDB 命名 LocalDB 实例。
连接字符串应使用的服务器部分是"(localdb)\\MSSQLLocalDB"。
请记住使用前缀为原义字符串`@`声或两个反斜杠"\\\\"在 C# 代码中指定的连接字符串时。  

## <a name="visual-studio-2012"></a>Visual Studio 2012

- 此版本的 Visual Studio 包括和旧版本的实体框架工具和运行时。
我们建议您升级到 Entity Framework Tools 6.1.3，使用[安装程序](https://www.microsoft.com/download/details.aspx?id=40762)Microsoft Download Center 中可用。
请参阅[过去释放](~/ef6/what-is-new/past-releases.md#ef-613)有关这些版本的详细信息。
- 将实体框架添加到新项目使用升级后的 EF 工具将自动添加 EF 6.1.3 NuGet 包。
可以手动安装或联机升级到可用任何 EF NuGet 包。
- 默认情况下，可使用此版本的 Visual Studio 的 SQL Server 实例是名为 v11.0 的 LocalDB 实例。
连接字符串应使用的服务器部分是"(localdb)\\v11.0"。
请记住使用前缀为原义字符串`@`声或两个反斜杠"\\\\"在 C# 代码中指定的连接字符串时。  

## <a name="visual-studio-2010"></a>Visual Studio 2010

- 与此版本的 Visual Studio 提供的实体框架工具的版本与 Entity Framework 6 运行时不兼容，并且不能升级。
- 默认情况下，实体框架工具将向你的项目添加 Entity Framework 4.0。
若要创建使用任何较新版本的 EF 应用程序，首先需要安装[NuGet 包管理器扩展](https://marketplace.visualstudio.com/items?itemName=NuGetTeam.NuGetPackageManager)。
- 默认情况下，EF 工具的版本中的所有代码生成都基于 EntityObject 和 Entity Framework 4。
我们推荐你切换代码生成通过安装适用于的 DbContext 代码生成模板基于 DbContext 和 Entity Framework 5 [C#](https://marketplace.visualstudio.com/items?itemName=EntityFrameworkTeam.EF5xDbContextGeneratorforC)或[Visual Basic](https://marketplace.visualstudio.com/items?itemName=EntityFrameworkTeam.EF5xDbContextGeneratorforVBNET)。
- 一旦您已安装的 NuGet 包管理器扩展，您可以手动安装或联机升级到可用任何 EF NuGet 包并使用 EF6 Code First，它不需要使用设计器。
- 默认情况下，可使用此版本的 Visual Studio 的 SQL Server 实例是名为 SQLEXPRESS 的 SQL Server Express。
连接字符串应使用的服务器部分是"。\\SQLEXPRESS"。
请记住使用前缀为原义字符串`@`声或两个反斜杠"\\\\"在 C# 代码中指定的连接字符串时。
