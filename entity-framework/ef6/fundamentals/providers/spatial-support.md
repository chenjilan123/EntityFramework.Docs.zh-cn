---
title: 针对空间类型-EF6 的提供程序支持
author: divega
ms.date: 10/23/2016
ms.assetid: 1097cb00-15f5-453d-90ed-bff9403d23e3
ms.openlocfilehash: 9c00e82c663daec219fe649a8d889afcc81564f7
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022270"
---
# <a name="provider-support-for-spatial-types"></a>针对空间类型的提供程序支持
实体框架支持使用通过 DbGeography 或 DbGeometry 类的空间数据。 这些类要依赖于实体框架提供程序提供特定于数据库的功能。 并非所有提供程序支持空间数据，这样做的那些可能具有如的空间类型程序集安装的其他先决条件。 下面提供了有关针对空间类型的提供程序支持的详细信息。  

可以在两个演练中，一个用于 Code First，另一个用于 Database First 或 Model First 找到如何在应用程序中使用的空间类型的其他信息：  

- [在代码中首先空间数据类型。](~/ef6/modeling/code-first/data-types/spatial.md)  
- [在 EF 设计器中的空间数据类型](~/ef6/modeling/designer/data-types/spatial.md)  

## <a name="ef-releases-that-support-spatial-types"></a>支持空间类型的 EF 版本  

在 EF5 中引入了针对空间类型的支持。 但是，在 EF5 空间类型才支持面向和.NET 4.5 上运行应用程序时。  

从 EF6 空间类型支持面向.NET 4 和.NET 4.5 的应用程序开始。  

## <a name="ef-providers-that-support-spatial-types"></a>支持空间类型的 EF 提供程序  

### <a name="ef5"></a>EF5  

EF5 我们已经注意到，支持空间类型与实体框架提供程序：  

- Microsoft SQL Server 提供程序  
    - 此提供程序附带的 EF5。  
    - 此提供程序取决于安装可能需要一些其他低级别库 — 有关详细信息，请参阅下文。  
- [适用于 Oracle 的 Devart dotConnect](http://www.devart.com/dotconnect/oracle/)  
    - 这是从 Devart 的第三方提供程序。  

如果你知道 EF5 支持的提供程序的空间类型然后请联系我们，我们将很高兴地将其添加到此列表。  

### <a name="ef6"></a>EF6  

Ef6，我们已经注意到，支持空间类型与实体框架提供程序：  

- Microsoft SQL Server 提供程序  
    - 此提供程序附带的 EF6。  
    - 此提供程序取决于安装可能需要一些其他低级别库 — 有关详细信息，请参阅下文。  
- [适用于 Oracle 的 Devart dotConnect](http://www.devart.com/dotconnect/oracle/)  
    - 这是从 Devart 的第三方提供程序。  

如果你知道的 EF6 支持的提供程序的空间类型然后请联系我们，我们将很高兴地将其添加到此列表。  

## <a name="prerequisites-for-spatial-types-with-microsoft-sql-server"></a>使用 Microsoft SQL Server 空间类型的先决条件  

SQL Server 空间支持取决于 SqlGeography 和 SqlGeometry 的低级别，特定于 SQL Server 的类型。 这些类型位于程序集 Microsoft.SqlServer.Types.dll，并作为 EF 的一部分或作为.NET Framework 的一部分不提供此程序集。  

安装 Visual Studio 时它通常还会安装 SQL Server 的版本，这将包括的 Microsoft.SqlServer.Types.dll 安装。  

如果想要使用的空间类型，在计算机上未安装 SQL Server 或从 SQL Server 安装中排除空间类型，您将需要手动安装它们。 可以使用安装类型`SQLSysClrTypes.msi`，这是 Microsoft SQL Server 功能包的一部分。 空间类型是 SQL Server 特定于版本，因此建议[搜索"SQL Server 功能包"](https://www.microsoft.com/search/result.aspx?q=sql+server+feature+pack)在 Microsoft 下载中心，然后选择并下载到您将使用的 SQL Server 的版本相对应的选项。
