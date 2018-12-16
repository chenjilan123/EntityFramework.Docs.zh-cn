---
title: Entity Framework 6 提供程序模型的 EF6
author: divega
ms.date: 06/27/2018
ms.assetid: 066832F0-D51B-4655-8BE7-C983C557E0E4
ms.openlocfilehash: d07a8689fe968bb1512095a59a61abc7ac346a31
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022319"
---
# <a name="the-entity-framework-6-provider-model"></a>Entity Framework 6 提供程序模型

实体框架提供程序模型允许实体框架与不同类型的数据库服务器一起使用。 例如，一个提供程序可以插入，若要允许另一个提供程序可以插入到，以允许使用针对 Microsoft SQL Server Compact Edition EF 时对 Microsoft SQL Server，使用 EF。 我们已经注意到 EF6 的提供程序，可[实体框架提供程序](~/ef6/fundamentals/providers/index.md)页。

某些更改而需要向 EF 与提供程序以允许 EF 根据开放源许可发布的交互的方式。 这些更改需要重新生成的 EF 提供者针对与提供程序注册的新机制的 EF6 程序集。

## <a name="rebuilding"></a>重新生成

与 EF6 现在发送以前是.NET Framework 的一部分的核心代码，以带外 (OOB) 程序集。 可以上找到有关如何构建针对 EF6 应用程序的详细信息[更新应用程序的 EF6](~/ef6/what-is-new/upgrading-to-ef6.md)页。 提供程序还需要使用这些说明重新生成。

## <a name="provider-types-overview"></a>提供程序类型概述

EF 提供程序实际上是特定于提供程序的服务定义的这些服务将从扩展 （适用于基类） 或 （对于接口） 实现的 CLR 类型的集合。 两个这些服务是基本和 EF 才能完全正常的必要条件。 其他人是可选的只需在特定的功能是必需的和/或这些服务的默认实现不适合作为目标的特定数据库服务器。

## <a name="fundamental-provider-types"></a>基本提供程序类型

### <a name="dbproviderfactory"></a>DbProviderFactory

EF 依赖于具有某一类型派生自[System.Data.Common.DbProviderFactory](https://msdn.microsoft.com/library/system.data.common.dbproviderfactory.aspx)执行的所有低级数据库访问。 DbProviderFactory 不是实际的 EF 的一部分，但却为 ADO.NET 提供程序提供的入口点的.NET Framework 中的类可用于通过 EF，其他 O/Rm 或直接由应用程序获取实例的连接、 命令、 参数和提供程序中的其他 ADO.NET 抽象无关的方式。 有关 DbProviderFactory 的详细信息中找到[ADO.NET 的 MSDN 文档](https://msdn.microsoft.com/library/a6cd7c08.aspx)。

### <a name="dbproviderservices"></a>DbProviderServices

EF 依赖于具有某一类型派生自 DbProviderServices 提供所需的 EF 基于 ADO.NET 提供程序已提供的功能的其他功能。 在较旧版本的 EF DbProviderServices 类是.NET Framework 的一部分，未找到 System.Data.Common 命名空间中。 此类从 EF6 现在是 EntityFramework.dll 的一部分并 System.Data.Entity.Core.Common 命名空间中。

可以上找到有关 DbProviderServices 实现的基本功能的更多详细信息[MSDN](https://msdn.microsoft.com/library/ee789835.aspx)。 但请注意，截至撰写本文时此信息不会更新 ef6 尽管的大多数概念仍然有效。 SQL Server 和 SQL Server Compact 的 DbProviderServices 实现到还签入[开源 o d e b](https://github.com/aspnet/EntityFramework6/)并且可作为有用参考信息的其他实现。

在较旧版本的 EF 中直接从 ADO.NET 提供程序中获取了要使用的 DbProviderServices 实现。 这是通过强制转换为 IServiceProvider，DbProviderFactory 和调用 GetService 方法。 这与 DbProviderFactory 紧密耦合的 EF 提供程序。 这种耦合阻止 EF 移出.NET Framework，因此为 EF6 这种紧密耦合已被删除，直接在应用程序的配置文件中或在基于代码的现已注册的 DbProviderServices 实现配置更多详细信息中所述_注册 DbProviderServices_下面一节。

## <a name="additional-services"></a>其他服务

除了上面所述的基本服务还有其他许多服务使用的 EF 始终或有时提供程序特定。 DbProviderServices 实现可以提供这些服务的默认提供程序特定于实现。 应用程序还可以重写这些服务的实现或 DbProviderServices 类型不提供默认值时提供的实现。 这在更多详细信息中所述_解析其他服务_下面一节。

下面列出了提供程序可能需要向提供程序的其他服务类型。 可以在 API 文档中找到有关每个服务类型的更多详细信息。

### <a name="idbexecutionstrategy"></a>IDbExecutionStrategy

这是一项可选服务，允许提供程序时对数据库执行查询和命令实现重试或其他行为。 如果未不提供任何实现，然后 EF 将只需执行命令并引发任何异常传播。 SQL Server 的此服务用于提供的重试策略，这是针对基于云的数据库服务器，例如 SQL Azure 运行时特别有用。

### <a name="idbconnectionfactory"></a>IDbConnectionFactory

这是一项可选服务，允许一个提供程序来创建 DbConnection 对象按照约定，如果给定的数据库名称。 请注意，虽然 DbProviderServices 实现即可解决此服务，它 EF 4.1 以来一直存在，也可以显式设置配置文件中或在代码中。 提供程序仅将有机会若要解决此服务，如果它注册为默认提供程序 (请参阅_的默认提供程序_下面)，如果默认连接工厂尚未设置其他位置。

### <a name="dbspatialservices"></a>DbSpatialServices

这是一项可选服务，允许一个提供程序来添加对 geography 和 geometry 空间类型的支持。 为了使应用程序使用的空间类型使用 EF，必须提供此服务的实现。 DbSptialServices 要求的两种方式。 首先，提供程序特定的空间服务请求使用 DbProviderInfo 对象 (其中包含固定条件名称和清单标记) 作为键。 其次，DbSpatialServices 可以要求不含键。 这用于解决"全局空间提供程序"创建独立的 DbGeography 或 DbGeometry 类型时使用。

### <a name="migrationsqlgenerator"></a>MigrationSqlGenerator

这是一项可选服务，用于生成使用 SQL 创建和修改数据库架构中由 Code First 的 EF 迁移。 实现所需支持迁移。 如果提供了一个实现然后它将还使用时使用数据库初始值设定项或 Database.Create 方法创建数据库。

### <a name="funcdbconnection-string-historycontextfactory"></a>Func < DbConnection，HistoryContextFactory 字符串 >

这是一项可选服务，允许一个提供程序来配置映射到 HistoryContext`__MigrationHistory`使用 EF 迁移的表。 HistoryContext 是代码的第一个 DbContext，并且可以使用正常的 fluent API 来更改像表和列映射规范的名称进行配置。 此服务为所有提供程序返回的 EF 的默认实现可能适用于给定的数据库服务器，如果该提供程序支持所有的默认值表和列映射。 这种情况下不需要提供此服务的实现提供程序。

### <a name="idbproviderfactoryresolver"></a>IDbProviderFactoryResolver

这是一项可选服务用于从给定的 DbConnection 对象获取正确的 DbProviderFactory。 此服务为所有提供程序返回的 EF 的默认实现需要适用于所有提供程序。 但是，.NET 4 上运行时，DbProviderFactory 不可公开访问从一个 if 其 DbConnections。 因此，EF 使用某些启发搜索已注册的提供程序查找匹配项。 很可能某些提供商对于这些试探法将失败，而且该提供程序应在这种情况下提供的新实现。

## <a name="registering-dbproviderservices"></a>注册 DbProviderServices

在应用程序的配置文件 （app.config 或 web.config） 或使用基于代码的配置中，可以注册要使用的 DbProviderServices 实现。 在任一情况下注册作为键使用提供程序的"固定名称"。 这允许多个提供程序注册和单个应用程序中使用。 用于 ADO.NET 提供程序注册和连接字符串的固定名称相同的 EF 注册使用的固定名称。 例如，对于 SQL Server 的固定名称"System.Data.SqlClient"使用。

### <a name="config-file-registration"></a>配置文件注册

要使用的 DbProviderServices 类型注册为应用程序配置文件的 entityFramework 部分的提供程序列表中的提供程序元素。 例如：

``` xml
<entityFramework>
  <providers>
    <provider invariantName="My.Invariant.Name" type="MyProvider.MyProviderServices, MyAssembly" />
  </providers>
</entityFramework>
```

_类型_字符串必须是要使用的 DbProviderServices 实现的程序集限定类型名称。

### <a name="code-based-registration"></a>基于代码的注册

此外可使用代码注册从 EF6 提供程序。 这允许要使用而无需对应用程序的配置文件的任何更改的 EF 提供程序。 若要使用基于代码的配置应用程序应创建一个 DbConfiguration 类，如中所述[基于代码的配置文档](https://msdn.com/data/jj680699)。 然后，DbConfiguration 类的构造函数应调用 SetProviderServices 注册 EF 提供程序。 例如：

``` csharp
public class MyConfiguration : DbConfiguration
{
    public MyConfiguration()
    {
        SetProviderServices("My.New.Provider", new MyProviderServices());
    }
}
```

## <a name="resolving-additional-services"></a>解析其他服务

如中所述_提供程序类型概述_部分 DbProviderServices 类还可用于解决其他服务。 这可能是因为 DbProviderServices 实现 IDbDependencyResolver 并且每个已注册的 DbProviderServices 类型添加为"默认解析程序"。 中更详细地介绍 IDbDpendencyResolver 机制[依赖项解析](~/ef6/fundamentals/configuring/dependency-resolution.md)。 但是，不需要了解此规范，若要解决提供程序中的其他服务中的所有概念。

若要解决其他服务的提供程序的最常见方法是调用 DbProviderServices.AddDependencyResolver DbProviderServices 类的构造函数中每个服务。 例如，SqlProviderServices （SQL Server 的 EF 提供程序） 具有类似于此的初始化代码：

``` csharp
private SqlProviderServices()
{
    AddDependencyResolver(new SingletonDependencyResolver<IDbConnectionFactory>(
        new SqlConnectionFactory()));

    AddDependencyResolver(new ExecutionStrategyResolver<DefaultSqlExecutionStrategy>(
        "System.data.SqlClient", null, () => new DefaultSqlExecutionStrategy()));

    AddDependencyResolver(new SingletonDependencyResolver<Func<MigrationSqlGenerator>>(
        () => new SqlServerMigrationSqlGenerator(), "System.data.SqlClient"));

    AddDependencyResolver(new SingletonDependencyResolver<DbSpatialServices>(
        SqlSpatialServices.Instance,
        k =>
        {
            var asSpatialKey = k as DbProviderInfo;
            return asSpatialKey == null
                || asSpatialKey.ProviderInvariantName == ProviderInvariantName;
        }));
}
```

此构造函数使用以下帮助器类：

*   SingletonDependencyResolver： 提供用于解析单一实例服务的简单方法，即为其同一个实例返回每次调用 GetService 的服务。 暂时性服务通常注册为一个单一实例工厂，它将用于按需创建暂时性的实例。
*   ExecutionStrategyResolver： 冲突解决程序特定于返回 IExecutionStrategy 实现。

而不是使用 DbProviderServices.AddDependencyResolver 还有可能重写 DbProviderServices.GetService 并直接解决其他服务。 当 EF 需要定义通过某一类型，然后在某些情况下，给定密钥的服务时，将调用此方法。 如果它可以或返回为 null，以选择退出的返回该服务并允许另一个类来解决此问题，该方法应返回该服务。 例如，若要解决的默认连接工厂 GetService 中的代码可能如下所示：

``` csharp
public override object GetService(Type type, object key)
{
    if (type == typeof(IDbConnectionFactory))
    {
        return new SqlConnectionFactory();
    }
    return null;
}
```

### <a name="registration-order"></a>注册顺序

在应用程序的配置文件中注册的多个 DbProviderServices 实现后，它们将作为辅助冲突解决程序中所列的顺序添加。 由于冲突解决程序总是会添加到辅助冲突解决程序链顶部这意味着，在列表末尾的提供程序将有机会解决先于其他的依赖项。 （这可能看起来稍有违反语感一开始，但它是有意义，如果您假设每个提供程序的列表，并堆叠在现有提供程序一起）。

这种顺序通常并不重要因为大多数提供程序服务是特定于提供程序和由提供程序固定名称键控。 但是，为服务，不由进行键控提供程序固定名称或某些其他特定于提供程序的密钥将解析该服务基于此排序。 例如，如果未显式设置以不同方式某处 else，然后使用默认连接工厂将来自链中的最顶层的提供程序。

## <a name="additional-config-file-registrations"></a>其他配置文件注册

就可以显式注册的某些其他提供程序服务直接在应用程序的配置文件中上面所述。 完成此操作后的配置文件中的注册，将使用而不是任何返回的 DbProviderServices 实现的 GetService 方法。

### <a name="registering-the-default-connection-factory"></a>注册默认连接工厂

从 EF5 EntityFramework NuGet 包自动注册的 SQL Express 连接工厂或 LocalDb 连接工厂的配置文件中。

例如：

``` xml
<entityFramework>
  <defaultConnectionFactory type="System.Data.Entity.Infrastructure.SqlConnectionFactory, EntityFramework" >
</entityFramework>
```

_类型_是默认连接工厂必须实现 IDbConnectionFactory 的程序集限定类型名称。

建议提供程序 NuGet 包安装时这种方式设置的默认连接工厂。 请参阅_提供程序的 NuGet 包_下面。

## <a name="additional-ef6-provider-changes"></a>其他 EF6 提供程序更改

### <a name="spatial-provider-changes"></a>空间提供程序更改

从 DbSpatialDataReader 派生类中支持空间类型的提供程序现在必须实现一些其他方法：

*   `public abstract bool IsGeographyColumn(int ordinal)`
*   `public abstract bool IsGeometryColumn(int ordinal)`

也有新的建议的默认实现委托给同步方法，并因此不会执行以异步方式重写的现有方法的异步版本：

*   `public virtual Task<DbGeography> GetGeographyAsync(int ordinal, CancellationToken cancellationToken)`
*   `public virtual Task<DbGeometry> GetGeometryAsync(int ordinal, CancellationToken cancellationToken)`

### <a name="native-support-for-enumerablecontains"></a>对 Enumerable.Contains 的本机支持

EF6 引入了新的表达式类型 DbInExpression，已添加到在 LINQ 查询中使用 Enumerable.Contains 围绕解决性能问题。 DbProviderManifest 类有一个新的虚拟方法，SupportsInExpression，EF 来确定处理新的表达式类型的提供的调用。 与现有提供程序实现的兼容性，方法返回 false。 若要从这一改进中获益，EF6 提供程序可以添加代码以处理 DbInExpression 并重写 SupportsInExpression 返回 true。 可以通过调用 DbExpressionBuilder.In 方法创建的 DbInExpression 实例。 DbInExpression 实例组成 DbExpression，通常表示表列和 DbConstantExpression 要测试其匹配项的列表。

## <a name="nuget-packages-for-providers"></a>提供程序的 NuGet 包

若要使 EF6 提供程序可用的一种方法是作为 NuGet 包发布。 使用 NuGet 包具有以下优点：

*   它是易于使用的 NuGet 提供程序注册添加到应用程序的配置文件
*   其他更改可以对配置文件来设置默认连接工厂，以便按照约定进行的连接将使用已注册的提供程序
*   添加绑定重定向，以便 EF6 提供程序应继续工作，即使新的 EF 程序包发布的 NuGet 句柄

此示例是 EntityFramework.SqlServerCompact 包随附于[开源 o d e b](http://github.com/aspnet/entityframework6)。 此包提供用于创建 EF 提供程序 NuGet 包的合适的模板。

### <a name="powershell-commands"></a>PowerShell 命令

安装 EntityFramework NuGet 包时它将注册的 PowerShell 模块，包含对于提供程序的包非常有用的两个命令：

*   添加 EFProvider 将新实体添加目标项目的配置文件中的提供程序，并确保它在已注册的提供程序的列表的末尾处。
*   添加 EFDefaultConnectionFactory 添加，或更新目标项目的配置文件中的 defaultConnectionFactory 注册。

这两个命令负责将 entityFramework 部分添加到配置文件并添加提供程序集合，如有必要。

其目的是，从 install.ps1 NuGet 脚本调用这些命令。 例如，SQL Compact 提供程序的 install.ps1 看起来类似于此：

``` powershell
param($installPath, $toolsPath, $package, $project)
Add-EFDefaultConnectionFactory $project 'System.Data.Entity.Infrastructure.SqlCeConnectionFactory, EntityFramework' -ConstructorArguments 'System.Data.SqlServerCe.4.0'
Add-EFProvider $project 'System.Data.SqlServerCe.4.0' 'System.Data.Entity.SqlServerCompact.SqlCeProviderServices, EntityFramework.SqlServerCompact'</pre>
```

可以通过包管理器控制台窗口中使用 get-help 获取有关这些命令的详细信息。

## <a name="wrapping-providers"></a>包装提供程序

换行提供程序是包装现有的提供程序以使用其他功能，如分析或跟踪功能来扩展它的 EF 和/或 ADO.NET 提供程序。 包装提供程序可以注册以正常方式，但通常会更方便地通过截获与提供程序相关的服务的分辨率设置在运行时包装提供程序。 DbConfiguration 类上的静态事件 OnLockingConfiguration 可用来执行此操作。

OnLockingConfiguration 称为 EF 已确定将来获取应用程序域的所有 EF 配置后，但用于锁定之前。 在应用启动时 （EF 使用之前） 应用程序应注册此事件的事件处理程序。 （我们正在考虑添加对配置文件中注册此处理程序的支持，但尚不支持，此）。事件处理程序应请 ReplaceService 以包装所需的每个服务调用。  

例如，来包装 IDbConnectionFactory 和 DbProviderService，应注册一个处理程序如下所示：

``` csharp
DbConfiguration.OnLockingConfiguration +=
    (_, a) =>
    {
        a.ReplaceService<DbProviderServices>(
            (s, k) => new MyWrappedProviderServices(s));

        a.ReplaceService<IDbConnectionFactory>(
            (s, k) => new MyWrappedConnectionFactory(s));
    };
```

已解决，现在应该用于将服务解析的关键字以及包装的服务将传递给处理程序。 然后，该处理程序可以包装此服务，并返回的服务替换为已包装的版本。

## <a name="resolving-a-dbproviderfactory-with-ef"></a>解决使用 EF DbProviderFactory

DbProviderFactory 是所需的 EF 中所述的基本提供程序类型之一_提供程序类型概述_上面一节。 如前面所述，它不是 EF 类型和注册通常不 EF 配置的一部分，而是改为在 machine.config 文件和/或应用程序配置文件正常的 ADO.NET 提供程序注册。

尽管此 EF 仍使用其普通依赖项解析机制查找 DbProviderFactory 时使用。 默认冲突解决程序在配置文件中使用普通 ADO.NET 注册并因此这通常是透明的。 由于正常的依赖项解析使用机制，但这意味着 IDbDependencyResolver 可用于解决 DbProviderFactory，即使尚未执行普通 ADO.NET 注册。

在这种方式中解决 DbProviderFactory 具有几个含义：

*   使用基于代码的配置的应用程序可以在注册相应的 DbProviderFactory 其 DbConfiguration 类中添加调用。 这是特别有用的应用程序不希望 （或无法） 进行根本使用的任何基于文件的配置。
*   该服务可以包装或替换中所述，使用 ReplaceService_包装提供程序_上面部分
*   理论上讲，DbProviderServices 实现无法解析 DbProviderFactory。

关于执行其中的任何内容需要注意的重要一点是，它们只会影响由 EF 的 DbProviderFactory 的查找。 其他非 EF 代码仍可能要以正常方式注册的 ADO.NET 提供程序，如果找不到注册可能会失败。 出于此原因是通常更适合普通的 ADO.NET 方法中注册 DbProviderFactory。

### <a name="related-services"></a>相关的服务

如果 EF 用于解析 DbProviderFactory，则它还应解析的 IProviderInvariantName 和 IDbProviderFactoryResolver 服务。

IProviderInvariantName 是一种服务，用于确定给定类型的 DbProviderFactory 的提供程序固定名称。 此服务的默认实现使用 ADO.NET 提供程序注册。 这意味着，如果因为 DbProviderFactory 正在解析由 EF，ADO.NET 提供程序未注册以正常方式，然后它将还需要解析此服务。 请注意使用 DbConfiguration.SetProviderFactory 方法时，会自动添加此服务的冲突解决程序。

如中所述_提供程序类型概述_上面部分中，IDbProviderFactoryResolver 用于从给定的 DbConnection 对象获取正确的 DbProviderFactory。 此服务在.NET 4 运行时的默认实现使用 ADO.NET 提供程序注册。 这意味着，如果因为 DbProviderFactory 正在解析由 EF，ADO.NET 提供程序未注册以正常方式，然后它将还需要解析此服务。
