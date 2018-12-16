---
title: 实体框架提供程序 - EF6
author: divega
ms.date: 06/27/2018
ms.assetid: 7BFB7763-CD6C-4520-93A2-7B265F5FA586
ms.openlocfilehash: 0cfe6c30d6533b0207f78a031d342847a7f68632
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022218"
---
# <a name="entity-framework-6-providers"></a>实体框架 6 提供程序
> [!NOTE]
> **仅限 EF6 及更高版本** - 此页面中讨论的功能、API 等已引入实体框架 6。 如果使用的是早期版本，则部分或全部信息不适用。

实体框架现在正在开源许可证下开发，EF6 及更高版本不会作为 .NET Framework 的一部分提供。 这样做有许多好处，但也需要针对 EF6 程序集重建 EF 提供程序。 这就意味着 EF5 及以下版本的 EF 提供程序在重建之前不能用于 EF6。

## <a name="which-providers-are-available-for-ef6"></a>哪些提供程序可用于 EF6？

已知的针对 EF6 重建的提供程序包括：

*   **Microsoft SQL Server 提供程序**
    *   构建于[实体框架开源代码库](http://github.com/aspnet/EntityFramework6)
    *   作为 [EntityFramework NuGet 包](http://nuget.org/packages/EntityFramework)的一部分提供
*   **Microsoft SQL Server Compact Edition 提供程序**
    *   构建于[实体框架开源代码库](http://github.com/aspnet/EntityFramework6)
    *   在 [EntityFramework.SqlServerCompact NuGet 包](http://nuget.org/packages/EntityFramework.SqlServerCompact)中提供
*   [**Devart dotConnect 数据提供程序**](http://www.devart.com/dotconnect/)
    *   来自 [Devart](http://www.devart.com/) 的提供各种数据库的第三方提供程序，包括 Oracle、MySQL、PostgreSQL、SQLite、Salesforce、DB2 和 SQL Server
*   [**CData Software 提供程序**](http://www.cdata.com/ado/)
    *   来自 [CData Software](http://www.cdata.com/ado/) 的提供各种数据存储的第三方提供程序，包括 Salesforce、Azure 表存储、MySql 等等
*   **Firebird 提供程序**
    *   作为 [NuGet 包](http://www.nuget.org/packages/FirebirdSql.Data.FirebirdClient/)提供
*   **Visual Fox Pro 提供程序**
    *   作为 [NuGet 包](https://www.nuget.org/packages/VFPEntityFrameworkProvider2/) 提供
*   **MySQL**
    *   [MySQL Connector/Net](http://dev.mysql.com/downloads/connector/net/)
*   **PostgreSQL**
    *   Npgsql 作为 [NuGet 包](http://www.nuget.org/packages/Npgsql.EF6/)提供
*   **Oracle**
    *   ODP.NET 作为 [NuGet 包](https://www.nuget.org/packages/Oracle.ManagedDataAccess.EntityFramework/)提供

请注意，此列表并不表示给定提供程序的功能级别或支持情况，只表示已构建了 EF6。

## <a name="registering-ef-providers"></a>注册 EF 提供程序

从实体框架 6 开始，可以使用基于代码的配置或在应用程序的配置文件中注册 EF 提供程序。

### <a name="config-file-registration"></a>配置文件注册

在 app.config 或 web.config 中注册 EF 提供程序的格式如下：


``` xml
    <entityFramework>
       <providers>
         <provider invariantName="My.Invariant.Name" type="MyProvider.MyProviderServices, MyAssembly" />
       </providers>
    </entityFramework>
```

请注意，如果从 NuGet 安装 EF 提供程序，则通常 NuGet 包会自动将此注册添加到配置文件中。 如果向非应用启动项目安装 NuGet 包，则可能需要将该注册复制到启动项目的配置文件中。

此注册中的“invariantName”是用于标识 ADO.NET 提供程序的相同固定名称。 它是 DbProviderFactories 注册中的“invariant”属性，是连接字符串注册中的“providerName”属性。 要使用的固定名称也应包含在该提供程序的文档中。 固定名称的示例：SQL Server 为“System.Data.SqlClient”，SQL Server Compact 为“System.Data.SqlServerCe.4.0”。

此注册中的“类型”是派生自“System.Data.Entity.Core.Common.DbProviderServices”的提供程序类型的程序集限定名称。 例如，用于 SQL Compact 的字符串是“System.Data.Entity.SqlServerCompact.SqlCeProviderServices, EntityFramework.SqlServerCompact”。 此处要使用的类型应包含在该提供程序的文档中。

### <a name="code-based-registration"></a>基于代码的注册

从实体框架 6 开始，可在代码中指定整个应用程序的 EF 配置。 有关完整的详细信息，请参阅[实体框架基于代码的配置](https://msdn.microsoft.com/data/jj680699)。 使用基于代码的配置注册 EF 提供程序的常规方法是，创建一个派生自 System.Data.Entity.DbConfiguration 的新类，并将其放置在与 DbContext 类相同的程序集中。 然后，DbConfiguration 类应在其构造函数中注册该提供程序。 例如，要注册 SQL Compact 提供程序，DbConfiguration 类如下所示：

``` csharp
    public class MyConfiguration : DbConfiguration
    {
        public MyConfiguration()
        {
            SetProviderServices(
                SqlCeProviderServices.ProviderInvariantName,
                SqlCeProviderServices.Instance);
        }
    }
```

在此代码中，“SqlCeProviderServices.ProviderInvariantName”为 SQL Server Compact 提供程序的固定名称字符串（“System.Data.SqlServerCe.4.0”）提供了便利，SqlCeProviderServices.Instance 会返回 SQL Compact EF 提供程序的单一实例。

## <a name="what-if-the-provider-i-need-isnt-available"></a>如果未提供我需要的提供程序怎么办？

如果该提供程序适用于以前版本的 EF，则建议联系提供程序的所有者并要求他们创建 EF6 版本。 应包含对 [EF6 提供程序模型的文档](~/ef6/fundamentals/providers/provider-model.md)的引用。

## <a name="can-i-write-a-provider-myself"></a>可以自己编写提供程序吗？

当然可以自己创建 EF 提供程序，但不应把这项任务考虑的太简单。 可以从上面关于 EF6 提供程序模型的链接开始着手。 你可能还会发现，从 [EF 开源代码库](https://github.com/aspnet/EntityFramework6)中包含的 SQL Server 和 SQL CE 提供程序的代码着手或用作参考会很有用。

请注意，从 EF6 开始，EF 提供程序与基础 ADO.NET 提供程序的耦合紧密程度降低。 这使得编写 EF 提供程序更轻松，且无需编写或包装 ADO.NET 类。
