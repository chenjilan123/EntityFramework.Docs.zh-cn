---
title: 配置 DbContext 的 EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: d7a22b5a-4c5b-4e3b-9897-4d7320fcd13f
uid: core/miscellaneous/configuring-dbcontext
ms.openlocfilehash: 393349c05ffaf42c6d2520e73abce23def6becc0
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42995933"
---
# <a name="configuring-a-dbcontext"></a>配置 DbContext

这篇文章演示用于配置基本模式`DbContext`通过`DbContextOptions`连接到使用特定的 EF Core 提供程序和可选行为的数据库。

## <a name="design-time-dbcontext-configuration"></a>设计时 DbContext 配置

EF Core 设计时工具如[迁移](xref:core/managing-schemas/migrations/index)需要能够发现和创建的工作实例`DbContext`以收集有关应用程序的实体类型以及它们如何映射到数据库架构的详细信息的类型。 此过程可以为自动，只要该工具可以轻松地创建`DbContext`，会将其配置同样到它如何将配置在运行时的方式。

尽管提供了必要的配置信息到任何模式`DbContext`可在运行时，需要使用的工具`DbContext`在设计时仅适用于有限数量的模式。 这些内容中更详细地介绍[设计时上下文创建](xref:core/miscellaneous/cli/dbcontext-creation)部分。

## <a name="configuring-dbcontextoptions"></a>配置 DbContextOptions

`DbContext` 必须具有的实例`DbContextOptions`才能执行任何工作。 `DbContextOptions`实例执行的配置信息如：

- 数据库提供程序，若要使用，通常情况下调用的方法，如选择`UseSqlServer`或 `UseSqlite`
- 任何必要的连接字符串或标识符的数据库实例中，通常作为参数传递到上述提供程序选择方法
- 任何提供程序级别的可选行为选择器，通常还链接到提供程序选择方法调用中
- 任何常规 EF Core 行为选择器，通常链接之后或之前提供程序选择器方法

下面的示例将配置`DbContextOptions`若要使用 SQL Server 提供程序，在连接包含`connectionString`变量、 提供程序级别的命令超时，以及可使在中执行的所有查询 EF Core 行为选择器`DbContext`[否跟踪](xref:core/querying/tracking#no-tracking-queries)默认情况下：

``` csharp
optionsBuilder
    .UseSqlServer(connectionString, providerOptions=>providerOptions.CommandTimeout(60))
    .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
```

> [!NOTE]  
> 提供程序选择器方法和上面提到的其他行为选择器方法是扩展方法上`DbContextOptions`或特定于提供程序的选项类。 若要有权访问这些扩展方法，可能需要具有一个命名空间 (通常`Microsoft.EntityFrameworkCore`) 中的作用域以及在项目中包含其他包依赖项。

`DbContextOptions`可以提供给`DbContext`通过重写`OnConfiguring`方法或构造函数参数通过从外部。

如果将使用它们，`OnConfiguring`最后应用，并且可以覆盖选项提供给构造函数参数。

### <a name="constructor-argument"></a>构造函数参数

使用构造函数的上下文代码：

``` csharp
public class BloggingContext : DbContext
{
    public BloggingContext(DbContextOptions<BloggingContext> options)
        : base(options)
    { }

    public DbSet<Blog> Blogs { get; set; }
}
```

> [!TIP]  
> DbContext 基构造函数还接受非泛型版本的`DbContextOptions`，但不是建议使用多个上下文类型的应用程序使用的非泛型版本。

从构造函数自变量进行初始化的应用程序代码：

``` csharp
var optionsBuilder = new DbContextOptionsBuilder<BloggingContext>();
optionsBuilder.UseSqlite("Data Source=blog.db");

using (var context = new BloggingContext(optionsBuilder.Options))
{
  // do stuff
}
```

### <a name="onconfiguring"></a>OnConfiguring

上下文代码`OnConfiguring`:

``` csharp
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlite("Data Source=blog.db");
    }
}
```

应用程序代码来初始化`DbContext`，它使用`OnConfiguring`:

``` csharp
using (var context = new BloggingContext())
{
  // do stuff
}
```

> [!TIP]
> 此方法不会将自身添加到测试，除非测试以完整的数据库为目标。

### <a name="using-dbcontext-with-dependency-injection"></a>使用依赖关系注入使用 DbContext

EF Core 支持使用`DbContext`与依赖关系注入容器。 DbContext 类型可以通过使用添加到服务容器`AddDbContext<TContext>`方法。

`AddDbContext<TContext>` 将这两个 DbContext 类型， `TContext`，并相应`DbContextOptions<TContext>`可用于从服务容器的注入。

请参阅[多个读取](#more-reading)以下依赖关系注入的其他信息。

添加`Dbcontext`依赖关系注入到：

``` csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<BloggingContext>(options => options.UseSqlite("Data Source=blog.db"));
}
```

这要求将添加[构造函数参数](#constructor-argument)到 DbContext 类型接受`DbContextOptions<TContext>`。

上下文代码：

``` csharp
public class BloggingContext : DbContext
{
    public BloggingContext(DbContextOptions<BloggingContext> options)
      :base(options)
    { }

    public DbSet<Blog> Blogs { get; set; }
}
```

（在 ASP.NET Core) 的应用程序代码：

``` csharp
public class MyController
{
    private readonly BloggingContext _context;

    public MyController(BloggingContext context)
    {
      _context = context;
    }

    ...
}
```

（服务提供商处直接使用，不太常见） 的应用程序代码：

``` csharp
using (var context = serviceProvider.GetService<BloggingContext>())
{
  // do stuff
}

var options = serviceProvider.GetService<DbContextOptions<BloggingContext>>();
```

## <a name="more-reading"></a>详细阅读

* 读取[首先使用 ASP.NET Core](../get-started/aspnetcore/index.md)有关 ASP.NET Core 中使用 EF 的详细信息。
* 读取[依赖关系注入](https://docs.microsoft.com/aspnet/core/fundamentals/dependency-injection)若要了解有关使用 DI 的详细信息。
* 读取[测试](testing/index.md)有关详细信息。
