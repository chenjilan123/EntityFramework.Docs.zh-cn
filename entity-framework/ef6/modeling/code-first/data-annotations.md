---
title: 第一个数据注释-EF6 的代码
author: divega
ms.date: 10/23/2016
ms.assetid: 80abefbd-23c9-4fce-9cd3-520e5df9856e
ms.openlocfilehash: 8d85ef85f56a23d9b3b526554417dc9dd360e139
ms.sourcegitcommit: 39080d38e1adea90db741257e60dc0e7ed08aa82
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/03/2018
ms.locfileid: "50980036"
---
# <a name="code-first-data-annotations"></a>Code First 数据注释
> [!NOTE]
> **EF4.1 及更高版本仅**的功能，Api，Entity Framework 4.1 中引入了此页所述的等。 如果您使用的是早期版本，则不适用的部分或所有这些信息。

此页上的内容是从作者： Julie Lerman 最初编写一篇文章 (\<http://thedatafarm.com>)。

实体框架 Code First 允许您使用您自己的域类来表示 EF 依赖于执行查询，该模型更改跟踪和更新功能。 代码首先将利用一种编程模式称为惯例优先于配置。 代码首先将假定您的类遵循的约定的实体框架，以及在这种情况下，将自动解决如何执行它的作业。 但是，如果您的类不遵循这些约定，必须将配置添加到您的类以提供必要的信息，EF 的功能。

代码首先提供了两种方法将这些配置添加到您的类。 一种使用简单的属性名为 DataAnnotations，并且第二个使用 Code First Fluent API，后者为您提供一种在代码中以强制方式，描述配置方法。

本文将重点介绍使用 DataAnnotations （System.ComponentModel.DataAnnotations 命名空间中） 来配置你的类，突出显示的最常见所需的配置。 DataAnnotations 还能够理解的大量的.NET 应用程序，如 ASP.NET MVC 允许这些应用程序可以利用相同的注释进行客户端验证。


## <a name="the-model"></a>模型

我将演示代码使用类的一个简单的对第一个 DataAnnotations： 博客和文章。

``` csharp
    public class Blog
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string BloggerName { get; set;}
        public virtual ICollection<Post> Posts { get; set; }
    }

    public class Post
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public DateTime DateCreated { get; set; }
        public string Content { get; set; }
        public int BlogId { get; set; }
        public ICollection<Comment> Comments { get; set; }
    }
```

因为它们是在博客和文章类方便地遵循代码的第一个约定并需要启用 EF 兼容性的调整。 但是，您可以使用批注的类和数据库映射到的有关向 EF 提供的详细信息。

 

## <a name="key"></a>键

Entity Framework 依赖于具有一个密钥值，用于跟踪的实体的每个实体。 一种约定的 Code First 是隐式键属性;第一次代码将查找名为"Id"或类名称和"Id"，如"BlogId"的组合的属性。 此属性将映射到数据库中的主键列。

博客和文章类都遵循此约定。 如果他们不乐意这样？ 如果博客使用名称*PrimaryTrackingKey*相反，或甚至*foo*？ 如果代码第一次未找到匹配此约定属性它将引发异常，由于实体框架的需求，您必须具有键属性。 关键批注用于指定哪些属性是要使用的 entitykey。

``` csharp
    public class Blog
    {
        [Key]
        public int PrimaryTrackingKey { get; set; }
        public string Title { get; set; }
        public string BloggerName { get; set;}
        public virtual ICollection<Post> Posts { get; set; }
    }
```

如果您是使用 code first 是数据库生成功能、 博客表具有名为 PrimaryTrackingKey，默认情况下也定义为标识主键列。

![博客具有主键的表](~/ef6/media/jj591583-figure01.png)

### <a name="composite-keys"></a>组合键

实体框架支持的复合键-主键由多个属性组成。 例如，你可以其主键为 PassportNumber 和 IssuingCountry 的组合的 Passport 类。

``` csharp
    public class Passport
    {
        [Key]
        public int PassportNumber { get; set; }
        [Key]
        public string IssuingCountry { get; set; }
        public DateTime Issued { get; set; }
        public DateTime Expires { get; set; }
    }
```

尝试在 EF 模型中使用上述的类将导致`InvalidOperationException`:

*无法确定复合主键排序类型 Passport。使用 ColumnAttribute 或 HasKey 方法指定为复合主键的订单。*

若要使用的复合键，实体框架要求定义键属性的顺序。 可以使用列批注以指定的顺序来执行此操作。

>[!NOTE]
> 将顺序值是相对 （而非基于索引） 以便可以使用任何值。 例如，100 到 200 个将代替 1 和 2 可接受。

``` csharp
    public class Passport
    {
        [Key]
        [Column(Order=1)]
        public int PassportNumber { get; set; }
        [Key]
        [Column(Order = 2)]
        public string IssuingCountry { get; set; }
        public DateTime Issued { get; set; }
        public DateTime Expires { get; set; }
    }
```

如果你有具有复合外键的实体，则必须指定相同的列排序用于相应的主键属性。

仅的相对顺序中的外键属性必须是相同的确切的值分配给**顺序**不需要匹配。 例如，在以下类中，3 和 4 可用来代替 1 和 2。

``` csharp
    public class PassportStamp
    {
        [Key]
        public int StampId { get; set; }
        public DateTime Stamped { get; set; }
        public string StampingCountry { get; set; }

        [ForeignKey("Passport")]
        [Column(Order = 1)]
        public int PassportNumber { get; set; }

        [ForeignKey("Passport")]
        [Column(Order = 2)]
        public string IssuingCountry { get; set; }

        public Passport Passport { get; set; }
    }
```

## <a name="required"></a>必需

需要的批注告知 EF 特定的属性是必需。

添加所需的 Title 属性将强制 EF （和 MVC） 以确保该属性中包含的数据。

``` csharp
    [Required]
    public string Title { get; set; }
```

任何其他不使用代码或标记在应用程序中的更改，MVC 应用程序将执行客户端验证，甚至动态生成使用的属性和批注名称的消息。

![创建具有标题页是所需的错误](~/ef6/media/jj591583-figure02.png)

所需的属性也会影响生成的数据库，从而映射的属性不可为 null。 请注意，标题字段已更改为"not null"。

>[!NOTE]
> 在某些情况下它不可能要为不可为 null，即使该属性是必需的数据库中的列。 例如，当对多个类型使用 TPH 继承策略数据存储在单个表。 如果派生的类型包括必需的属性列不能成为不可为 null 因为层次结构中的不是所有类型都将都具有此属性。

 

![Blogs 表](~/ef6/media/jj591583-figure03.png)

 

## <a name="maxlength-and-minlength"></a>MaxLength 和 MinLength

MaxLength 和 MinLength 属性，可以指定其他属性验证，像您那样使用所需。

下面是具有长度要求 BloggerName。 该示例还演示如何组合属性。

``` csharp
    [MaxLength(10),MinLength(5)]
    public string BloggerName { get; set; }
```

MaxLength 批注会影响数据库的属性的长度设置为 10。

![Blogs 表 BloggerName 列上显示的最大长度](~/ef6/media/jj591583-figure04.png)

客户端的批注 MVC 和 EF 4.1 服务器端批注将都遵循此验证，再次动态生成一条错误消息:"字段 BloggerName 具有最大长度为"10"的字符串或数组类型必须是"。该消息是有点长。 很多批注，可以使用 ErrorMessage 属性指定一条错误消息。

``` csharp
    [MaxLength(10, ErrorMessage="BloggerName must be 10 characters or less"),MinLength(5)]
    public string BloggerName { get; set; }
```

此外可以在所需的批注中指定错误消息。

![创建具有自定义错误消息页](~/ef6/media/jj591583-figure05.png)

 

## <a name="notmapped"></a>NotMapped

第一个代码约定决定了是受支持的数据类型的每个属性表示为数据库中。 但这并非总是这种情况在您的应用程序。 例如，可能博客类，用于创建基于字段的标题和 BloggerName 代码中有属性。 该属性可以动态创建实体并且不需要存储。 可以将标记不会映射到此 BlogCode 属性如 NotMapped 批注与数据库的任何属性。

``` csharp
    [NotMapped]
    public string BlogCode
    {
        get
        {
            return Title.Substring(0, 1) + ":" + BloggerName.Substring(0, 1);
        }
    }
```

 

## <a name="complextype"></a>ComplexType

不常见对域实体跨一组类的描述，然后分层这些类来描述完整实体。 例如，您可以添加到您的模型名 BlogDetails 的类。

``` csharp
    public class BlogDetails
    {
        public DateTime? DateCreated { get; set; }

        [MaxLength(250)]
        public string Description { get; set; }
    }
```

请注意 BlogDetails 不具有任何类型的键属性。 在域驱动设计中，BlogDetails 称为值对象。 实体框架将值对象称为复杂类型。  不能自行跟踪复杂类型。

但是作为博客类，它将作为博客对象的一部分进行跟踪的 BlogDetails 中的属性。 为了使代码首先将这种情况，你必须将 BlogDetails 类标记为复杂类型。

``` csharp
    [ComplexType]
    public class BlogDetails
    {
        public DateTime? DateCreated { get; set; }

        [MaxLength(250)]
        public string Description { get; set; }
    }
```

现在可以在博客类来表示该博客 BlogDetails 中添加一个属性。

``` csharp
        public BlogDetails BlogDetail { get; set; }
```

在数据库中，博客表将包含的所有博客包括其 BlogDetail 属性中包含的属性的属性。 默认情况下，每个前面带有复杂类型，BlogDetail 的名称。

![具有复杂类型的网络日志表](~/ef6/media/jj591583-figure06.png)

另一个注意事项是，虽然 DateCreated 属性被定义为不可为 null 的日期时间在类中，相关数据库字段是可以为 null。 如果你想要影响的数据库架构，则必须使用所需的批注。

 

## <a name="concurrencycheck"></a>ConcurrencyCheck

ConcurrencyCheck 批注可以标记一个或多个要在进行并发检查数据库中，当用户编辑或删除的实体的属性。 如果您一直使用 EF 设计器，这会将与将属性的 ConcurrencyMode 设置为 Fixed。

让我们了解 ConcurrencyCheck 通过将其添加到 BloggerName 属性的工作原理。

``` csharp
    [ConcurrencyCheck, MaxLength(10, ErrorMessage="BloggerName must be 10 characters or less"),MinLength(5)]
    public string BloggerName { get; set; }
```

当调用 SaveChanges 时，由于 ConcurrencyCheck 批注 BloggerName 字段中，将更新中使用该属性的原始值。 该命令将尝试查找正确的行通过筛选不仅上的密钥值，还在 BloggerName 的原始值。  以下是 UPDATE 命令发送到数据库，您可以看到该命令将更新包含 PrimaryTrackingKey 的行的关键部分为 1，该博客从数据库中检索时的原始值的"Julie"BloggerName。

``` SQL
    where (([PrimaryTrackingKey] = @4) and ([BloggerName] = @5))
    @4=1,@5=N'Julie'
```

如果有人已更改该博客的博主名称，请在此期间，此更新将失败，则会将需要处理 DbUpdateConcurrencyException。

 

## <a name="timestamp"></a>时间戳

它是更常见的是进行并发检查中使用行版本或时间戳字段。 但是，而不是使用 ConcurrencyCheck 批注，您可以使用更具体的时间戳批注的属性类型为字节数组。 代码首先将相同的处理时间戳属性为 ConcurrencyCheck 属性，但它还将确保代码首先生成的数据库字段是不可为 null。 仅可以在给定类中有一个时间戳属性。

将以下属性添加到博客类：

``` csharp
    [Timestamp]
    public Byte[] TimeStamp { get; set; }
```

在代码中首次在数据库表中创建不可以为 null 的时间戳列的结果。

![使用时间戳列的 Blogs 表](~/ef6/media/jj591583-figure07.png)

 

## <a name="table-and-column"></a>表和列

如果您让代码优先在创建数据库，可能想要更改的表和列创建的名称。 您还可以使用 Code First 与现有数据库。 但它并不总是类和你的域中的属性的名称与表和数据库中的列的名称相匹配的情况。

我的类名为博客，并按照约定，代码首先假定这会映射到名为博客的表。 如果不是这种情况可以使用表属性中指定的表的名称。 此处，批注指定表名 InternalBlogs。

``` csharp
    [Table("InternalBlogs")]
    public class Blog
```

列批注是详细擅长中指定的映射列的特性。 您可以规定名称、 数据类型或甚至一列的表中显示的顺序。 下面是列属性的一个示例。

``` csharp
    [Column("BlogDescription", TypeName="ntext")]
    public String Description {get;set;}
```

不要混淆列的数据类型 DataAnnotation TypeName 属性。 数据类型是 UI 使用一条注释，并忽略由 Code First。

下面是表后已重新生成。 表名已更改为 InternalBlogs，从复杂类型说明列现在是 BlogDescription。 因为在批注中指定了名称，但代码首先将不使用的复杂类型的名称开头的列名称的约定。

![Blogs 表和列重命名](~/ef6/media/jj591583-figure08.png)

 

## <a name="databasegenerated"></a>DatabaseGenerated

重要的数据库功能是能够具有计算属性。 如果您打算映射代码优先类到包含的表的计算列，您不希望实体框架可以尝试更新这些列。 但您希望 EF 从数据库中返回这些值后已插入或更新数据。 DatabaseGenerated 批注可用于计算枚举以及在类中标记这些属性。 其他枚举都是无和标识。

``` csharp
    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    public DateTime DateCreated { get; set; }
```

可以使用代码首先生成数据库时生成字节或时间戳列上的数据库，否则您才应使用此时指向现有数据库，因为代码首先将无法确定的公式计算所得的列。

默认情况下，读取上面是一个整数键属性将成为在数据库中的标识键。 这将是与将 DatabaseGenerated 设为 DatabaseGeneratedOption.Identity 相同。 如果不希望其成为标识键，您可以将值设置为 DatabaseGeneratedOption.None。

 

## <a name="index"></a>索引

> [!NOTE]
> **EF6.1 及更高版本仅**-Entity Framework 6.1 中引入了索引属性。 如果您使用的是早期版本不适用于此部分中的信息。

可以使用的一个或多个列创建索引**IndexAttribute**。 将特性添加到一个或多个属性将导致 EF 在数据库中创建相应的索引时自动创建该数据库，或创建相应的基架**CreateIndex**调用，如果使用 Code First 迁移。

例如，下面的代码将导致在创建索引**评级**的列**帖子**数据库表中的。

``` csharp
    public class Post
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }
        [Index]
        public int Rating { get; set; }
        public int BlogId { get; set; }
    }
```

默认情况下，将名为索引**IX\_&lt;属性名称&gt;** (IX\_评级在上面的示例)。 但还可以指定索引的名称。 下面的示例指定该索引，应将命名为**PostRatingIndex**。

``` csharp
    [Index("PostRatingIndex")]
    public int Rating { get; set; }
```

默认情况下，索引是非唯一的但你可以使用**IsUnique**名为参数来指定应该唯一索引。 下面的示例上引入了唯一索引**用户**的登录名。

``` csharp
    public class User
    {
        public int UserId { get; set; }

        [Index(IsUnique = true)]
        [StringLength(200)]
        public string Username { get; set; }

        public string DisplayName { get; set; }
    }
```

### <a name="multiple-column-indexes"></a>多列索引

跨多个列的索引指定为给定的表在多个索引批注中使用相同的名称。 创建多列索引时，需要在索引中指定列的顺序。 例如，以下代码将创建在多列索引**评级**并**BlogId**调用**IX\_BlogAndRating**。 **BlogId**是在索引中的第一列和**评级**是第二个。

``` csharp
    public class Post
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }
        [Index("IX_BlogIdAndRating", 2)]
        public int Rating { get; set; }
        [Index("IX_BlogIdAndRating", 1)]
        public int BlogId { get; set; }
    }
```

 

## <a name="relationship-attributes-inverseproperty-and-foreignkey"></a>关系属性： InverseProperty 和 ForeignKey

> [!NOTE]
> 此页提供了有关设置关系中使用数据注释在 Code First 模型的信息。 有关在 EF 和如何访问和处理数据使用关系的关系的常规信息，请参阅[关系和导航属性](~/ef6/fundamentals/relationships.md)。 *

第一个代码约定将负责在模型中最常见的关系，但有某些情况下，应帮助在位置。

更改造成的问题及其与文章的博客类中的键属性的名称。 

在生成数据库时，代码将首先看到在开机自检类的 BlogId 属性，并识别它，通过它与类名以及"Id"，匹配作为博客类的外键的约定。 但在博客类中没有任何 BlogId 属性。 此问题的解决方案是在 Post 中创建导航属性并使用外 DataAnnotation 帮助代码首先了解如何生成两个类之间的关系，使用 Post.BlogId 属性，以及如何指定中的约束数据库。

``` csharp
    public class Post
    {
            public int Id { get; set; }
            public string Title { get; set; }
            public DateTime DateCreated { get; set; }
            public string Content { get; set; }
            public int BlogId { get; set; }
            [ForeignKey("BlogId")]
            public Blog Blog { get; set; }
            public ICollection<Comment> Comments { get; set; }
    }
```

在数据库中的约束演示 InternalBlogs.PrimaryTrackingKey 和 Posts.BlogId 之间的关系。 

![InternalBlogs.PrimaryTrackingKey 与 Posts.BlogId 之间的关系](~/ef6/media/jj591583-figure09.png)

有多个类之间的关系时，使用 InverseProperty。

在发布类中，你可能想要跟踪谁编写了一篇博客文章的以及谁对其进行编辑。 以下是 Post 类的两个新的导航属性。

``` csharp
    public Person CreatedBy { get; set; }
    public Person UpdatedBy { get; set; }
```

此外需要引用这些属性的 Person 类中添加。 Person 类具有导航属性返回到 Post，一个用于所有的人，一个用于更新由该作者的文章的所有文章。

``` csharp
    public class Person
    {
            public int Id { get; set; }
            public string Name { get; set; }
            public List<Post> PostsWritten { get; set; }
            public List<Post> PostsUpdated { get; set; }
    }
```

代码首先不能在其自己的两个类中的属性匹配。 文章的数据库表的 CreatedBy 人员应有一个外键和一个用于 UpdatedBy 人但代码首先将创建四个将外键属性： Person\_Id、 人\_Id1、 CreatedBy\_Id 和UpdatedBy\_id。

![Posts 表包含的额外外键](~/ef6/media/jj591583-figure10.png)

若要解决这些问题，可以使用 InverseProperty 批注指定属性的对齐方式。

``` csharp
    [InverseProperty("CreatedBy")]
    public List<Post> PostsWritten { get; set; }

    [InverseProperty("UpdatedBy")]
    public List<Post> PostsUpdated { get; set; }
```

亲自 PostsWritten 属性就知道，这是指 Post 类型，因为它将生成与 Post.CreatedBy 之间的关系。 同样，PostsUpdated 将连接到 Post.UpdatedBy。 且代码首先会创建额外外键。

![Posts 表，而无需额外外键](~/ef6/media/jj591583-figure11.png)

 

## <a name="summary"></a>总结

DataAnnotations 不仅允许您描述客户端和服务器端验证在您的代码的第一个类，但它们还使你可以增强和甚至更正代码首先将使有关基于其约定类的假设。 使用 DataAnnotations，可不只让数据库架构生成，但还可以将您的代码的第一类映射到预先存在的数据库。

尽管它们非常灵活，请记住，DataAnnotations 提供仅大多数通常所需的配置更改可以使您的代码的第一个类。 若要配置您的类的一些边缘情况，您应查找的备用配置机制，Code First Fluent API。
