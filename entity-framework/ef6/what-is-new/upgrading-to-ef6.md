---
title: 升级到 Entity Framework 6
author: divega
ms.date: 10/23/2016
ms.assetid: 29958ae5-85d3-4585-9ba6-550b8ec9393a
ms.openlocfilehash: 711f1940080de27bd23cb8f641a5c7f2711dd65b
ms.sourcegitcommit: a6082a2caee62029f101eb1000656966195cd6ee
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/10/2018
ms.locfileid: "53182002"
---
# <a name="upgrading-to-entity-framework-6"></a>升级到 Entity Framework 6

在以前版本的 EF 代码已附带的.NET Framework 核心库 (主要 System.Data.Entity.dll 中) 和 NuGet 包中提供的带外 (OOB) 库 (主要 EntityFramework.dll) 之间拆分。 EF6 从核心库中获取代码，并合并到 OOB 库。 这是必要的以允许 EF 进行开放源代码并使其成为能够从.NET Framework 的不同速度发展。 此结果是应用程序将需要重新生成对已移动的类型。

这应该是直接使用 DbContext 产品附带 EF 4.1 及更高版本的应用程序。 需要执行一些操作是必需的应用程序使用 ObjectContext 的但它仍不难做到。

以下是你需要如何升级到 EF6 的现有应用程序的内容的清单。

## <a name="1-install-the-ef6-nuget-package"></a>1.安装 EF6 NuGet 包

需要升级到新的 Entity Framework 6 运行时。

1. 右键单击项目并选择**管理 NuGet 包...**  
2. 下**联机**选项卡上选择**EntityFramework**单击**安装**  
   > [!NOTE]
   > 如果以前版本的 EntityFramework NuGet 包已安装这会将其升级到 EF6。

或者，您可以从包管理器控制台运行以下命令：

``` powershell
Install-Package EntityFramework
```

## <a name="2-ensure-that-assembly-references-to-systemdataentitydll-are-removed"></a>2.请确保删除对 System.Data.Entity.dll 的程序集引用

安装 EF6 NuGet 包应自动删除为您对 System.Data.Entity 从你的项目的任何引用。

## <a name="3-swap-any-ef-designer-edmx-models-to-use-ef-6x-code-generation"></a>3.交换任何 EF 设计器 (EDMX) 模型以使用 EF 6.x 代码生成

如果有使用 EF 设计器创建的任何模型，您需要更新代码生成模板，以生成 EF6 兼容的代码。

> [!NOTE]
> 目前只有 EF 6.x DbContext 生成器模板可用于 Visual Studio 2012 和 2013年。

1. 删除现有的代码生成模板。 这些文件通常将叫做 **\<edmx_file_name\>.tt**并 **\<edmx_file_name\>。Context.tt**和嵌套在你在解决方案资源管理器的 edmx 文件下。 你可以选择模板中，解决方案资源管理器并按**Del**键以将其删除。  
   > [!NOTE]
   > 在网站项目模板将不会嵌套在您的 edmx 文件下, 但与其一起列出在解决方案资源管理器。  

   > [!NOTE]
   > 在 VB.NET 项目中将需要启用显示所有文件若要能够看到嵌套的模板文件。
2. 添加适当的 EF 6.x 代码生成模板。 打开您的模型在 EF 设计器中，右键单击设计图面，然后选择**添加代码生成项...**
    - 如果使用 DbContext API （推荐），然后**EF 6.x DbContext Generator**均可下**数据**选项卡。  
      > [!NOTE]
      > 如果使用 Visual Studio 2012，你需要安装 EF 6 工具具有此模板。 请参阅[获取实体框架](~/ef6/fundamentals/install.md)有关详细信息。  

    - 如果您使用 ObjectContext API，则将需要选择**联机**选项卡并搜索**EF 6.x EntityObject 生成器**。  
3. 如果任何自定义应用于代码生成模板，需要重新将其应用到已更新的模板。

## <a name="4-update-namespaces-for-any-core-ef-types-being-used"></a>4.更新适用于正在使用任何核心 EF 类型命名空间

未更改 DbContext 和 Code First 类型的命名空间。 这意味着，对于许多应用程序使用 EF 4.1 或更高版本将不需要进行任何更改。

类型，如 ObjectContext 以前在 System.Data.Entity.dll 中都已移到新的命名空间。 这意味着可能需要更新您*使用*或*导入*指令以生成针对 EF6。

命名空间的更改的一般规则是 System.Data.* 中的任何类型被移动到 System.Data.Entity.Core.*。 换而言之，只需插入**Entity.Core。** 之后 System.Data。 例如：

- System.Data.EntityException = > System.Data。**Entity.Core**。EntityException  
- System.Data.Objects.ObjectContext = > System.Data。**Entity.Core**。Objects.ObjectContext  
- System.Data.Objects.DataClasses.RelationshipManager = > System.Data。**Entity.Core**。Objects.DataClasses.RelationshipManager  

这些类型是在*Core*命名空间因为它们不能直接提供大多数基于 DbContext 的应用程序。 属于 System.Data.Entity.dll 某些类型仍然可用于通常和直接基于 DbContext 的应用程序，因此不已移动到*Core*命名空间。 这些是：

- System.Data.EntityState = > System.Data。**实体**。EntityState  
- System.Data.Objects.DataClasses.EdmFunctionAttribute = > System.Data。**Entity.DbFunctionAttribute**  
  > [!NOTE]
  > 此类已重命名;具有旧名称的类仍存在，且工作原理，但它现在标记为已过时。  
- System.Data.Objects.EntityFunctions = > System.Data。**Entity.DbFunctions**  
  > [!NOTE]
  > 此类已重命名;具有旧名称的类仍存在，并且工作原理，但它现已标记为已过时。）  
- 空间类 （例如，DbGeography，DbGeometry） 已从 System.Data.Spatial = > System.Data。**实体**。空间

> [!NOTE]
> System.Data 命名空间中的某些类型是在 System.Data.dll 中这不是 EF 程序集。 未将这些类型，因此它们的命名空间保持不变。
