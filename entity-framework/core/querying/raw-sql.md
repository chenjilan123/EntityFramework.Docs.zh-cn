---
title: 原生 SQL 查询 - EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 70aae9b5-8743-4557-9c5d-239f688bf418
uid: core/querying/raw-sql
ms.openlocfilehash: 0ad9731840c5f72064f2f66932b9867a0144f437
ms.sourcegitcommit: 2da6f9b05e1ce3a46491e5cc68f17758bdeb6b02
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/06/2018
ms.locfileid: "53006864"
---
# <a name="raw-sql-queries"></a>原生 SQL 查询

通过 Entity Framework Core 可以在使用关系数据库时下降到原始 SQL 查询。 这在无法使用 LINQ 表达要执行的查询，或因使用 LINQ 查询而导致低效的 SQL 查询时非常有用。 原始 SQL 查询可返回实体类型，或者，从 EF Core 2.1 开始，可返回模型中的[查询类型](xref:core/modeling/query-types)。

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。

## <a name="limitations"></a>限制

使用原生 SQL 查询时需注意以下几个限制：

* SQL 查询必须返回实体或查询类型的所有属性的数据。

* 结果集中的列名必须与属性映射到的列名称匹配。 请注意，这与 EF6 不同，EF6 中忽略了原始 SQL 查询时的属性/列映射关系，只需结果集列名与属性名相匹配即可。

* SQL 查询不能包含关联数据。 但是，在许多情况下你可以在查询后面紧跟着使用 `Include` 方法以返回关联数据（请参阅[包含关联数据](#including-related-data)）。

* 传递到此方法的 `SELECT` 语句通常应可进行编辑：如果 EF Core 需要在服务器端计算更多查询运算符（例如，转换 `FromSql` 后跟的 LINQ 运算符），则提供的 SQL 将被视为子查询。 这意味着传递的 SQL 不应包含子查询上无效的任何字符或选项，如：
  * 结尾分号
  * 在 SQL Server 上，结尾处的查询级提示（例如，`OPTION (HASH JOIN)`）
  * 在 SQL Server 上， `SELECT` 子句中不带 `TOP 100 PERCENT` 的 `ORDER BY` 子句

* 除 `SELECT` 以外的其他 SQL 语句自动识别为不可编写。 因此，存储过程的完整结果将始终返回到客户端，且在内存中计算 `FromSql` 后应用的任何 LINQ 运算符。

## <a name="basic-raw-sql-queries"></a>基本原生 SQL 查询

可以使用 *FromSql* 扩展方法基于原生 SQL 查询开始 LINQ 查询。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
``` csharp
var blogs = context.Blogs
    .FromSql("SELECT * FROM dbo.Blogs")
    .ToList();
```

原生 SQL 查询可用于执行存储过程。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
``` csharp
var blogs = context.Blogs
    .FromSql("EXECUTE dbo.GetMostPopularBlogs")
    .ToList();
```

## <a name="passing-parameters"></a>传递参数

正如接受 SQL 的任何 API 一样，务必参数化任何用户输入以抵御 SQL 注入攻击。 可以将参数占位符包含在 SQL 查询字符串中，然后提供参数值作为其他参数。 提供的任何参数值将自动转换为 `DbParameter`。

下面的示例将一个参数传递到存储过程。 尽管这看上去可能像 `String.Format` 语法，但提供的值包装在参数中，且生成的参数名称插入在指定 `{0}` 占位符的位置。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
``` csharp
var user = "johndoe";

var blogs = context.Blogs
    .FromSql("EXECUTE dbo.GetMostPopularBlogsForUser {0}", user)
    .ToList();
```

这是相同的查询，但使用 EF Core 2.0 及更高版本支持的字符串内插语法：

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
``` csharp
var user = "johndoe";

var blogs = context.Blogs
    .FromSql($"EXECUTE dbo.GetMostPopularBlogsForUser {user}")
    .ToList();
```

还可以构造 DbParameter 并将其作为参数值提供。 这样可以在 SQL 查询字符串中使用命名参数

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
``` csharp
var user = new SqlParameter("user", "johndoe");

var blogs = context.Blogs
    .FromSql("EXECUTE dbo.GetMostPopularBlogsForUser @user", user)
    .ToList();
```

## <a name="composing-with-linq"></a>使用 LINQ 编写

如果发送到数据库中的 SQL 查询是可组合的，则可以在原始 SQL 查询后面紧跟着使用 LINQ 运算符。 以 `SELECT` 关键字开始的 SQL 查询一般是可组合的。

下面的示例使用原始 SQL 从一个表值函数 (Table-Valued Function，TVF) 中进行查询，然后结合使用 LINQ 在其上进行筛选和排序。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
``` csharp
var searchTerm = ".NET";

var blogs = context.Blogs
    .FromSql($"SELECT * FROM dbo.SearchBlogs({searchTerm})")
    .Where(b => b.Rating > 3)
    .OrderByDescending(b => b.Rating)
    .ToList();
```

### <a name="including-related-data"></a>包含关联数据

结合 LINQ 运算符可将关联数据包含在查询中。

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
``` csharp
var searchTerm = ".NET";

var blogs = context.Blogs
    .FromSql($"SELECT * FROM dbo.SearchBlogs({searchTerm})")
    .Include(b => b.Posts)
    .ToList();
```

> [!WARNING]  
> **始终为原始 SQL 查询使用参数化：** 接受原始 SQL 字符串的 API （如 `FromSql` 和 `ExecuteSqlCommand`）有比较方便的参数化传值机制。 除了验证用户输入，还应始终为原始 SQL 查询或命令中使用的任何值启用参数化传值机制。 如果使用字符串拼接来动态生成查询字符串中的任何部分，则你应负责验证所有输入以抵御 SQL 注入攻击。
