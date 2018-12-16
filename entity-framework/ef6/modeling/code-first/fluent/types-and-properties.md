---
title: Fluent API 的配置和映射属性和类型的 EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 648ed274-c501-4630-88e0-d728ab5c4057
ms.openlocfilehash: 7371cc99142ccf8fc6bea237d7d58d1e67fcecec
ms.sourcegitcommit: 75f8a179ac9a70ad390fc7ab2a6c5e714e701b8b
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/27/2018
ms.locfileid: "52339798"
---
# <a name="fluent-api---configuring-and-mapping-properties-and-types"></a>Fluent API-配置和映射属性和类型
使用 Entity Framework Code First 时的默认行为是将您的 POCO 类映射到表使用一系列融入 EF 的约定。 有时，但是，您无法或不希望遵循这些约定，并且需要将实体映射到以外的约定的规定。  

有两种主要方式，你可以配置以外的约定，即使用 EF[批注](~/ef6/modeling/code-first/data-annotations.md)或 EFs fluent API。 批注仅涵盖 fluent API 功能的子集，因此不能使用批注来实现的映射方案。 本文旨在演示如何使用 fluent API 配置属性。  

Code first fluent API 最常访问通过重写[OnModelCreating](https://msdn.microsoft.com/library/system.data.entity.dbcontext.onmodelcreating.aspx)方法在派生[DbContext](https://msdn.microsoft.com/library/system.data.entity.dbcontext.aspx)。 以下示例旨在演示如何执行各种任务使用 fluent api，并允许您将代码复制和自定义它以满足您的模型，如果你想要看到它们可用作与该模型的则本文结尾处提供。  

## <a name="model-wide-settings"></a>模型范围内的设置  

### <a name="default-schema-ef6-onwards"></a>默认架构 (EF6 及更高版本)  

从 EF6 您可以使用 HasDefaultSchema 方法上 DbModelBuilder 来指定要使用的所有表、 存储的过程等的数据库架构。此默认设置将被重写的任何对象的显式配置为不同的架构。  

``` csharp
modelBuilder.HasDefaultSchema("sales");
```  

### <a name="custom-conventions-ef6-onwards"></a>自定义约定 (EF6 及更高版本)  

从 EF6，您可以创建您自己的约定来补充包括在 Code First 的开始。 有关更多详细信息，请参阅[自定义代码优先约定](~/ef6/modeling/code-first/conventions/custom.md)。  

## <a name="property-mapping"></a>属性映射  

[属性](https://msdn.microsoft.com/library/system.data.entity.infrastructure.dbentityentry.property.aspx)方法用于配置每个属性属于实体或复杂类型的属性。 属性方法用于获取给定属性的配置对象。 是特定于要配置; 的类型上的配置对象选项例如，IsUnicode 是仅适用于字符串属性。  

### <a name="configuring-a-primary-key"></a>配置为主键  

为主键的实体框架约定是：  

1. 类定义其名称为"ID"或"Id"的属性  
2. 或类名称后跟"ID"或"Id"  

若要显式设置的属性是主键，可以使用 HasKey 方法。 在以下示例中，HasKey 方法用于在 OfficeAssignment 类型配置 InstructorID 为主键。  

``` csharp
modelBuilder.Entity<OfficeAssignment>().HasKey(t => t.InstructorID);
```  

### <a name="configuring-a-composite-primary-key"></a>配置组合主键  

下面的示例配置为部门类型的复合主键的 DepartmentID 和名称属性。  

``` csharp
modelBuilder.Entity<Department>().HasKey(t => new { t.DepartmentID, t.Name });
```  

### <a name="switching-off-identity-for-numeric-primary-keys"></a>对于数值主键关闭标识切换  

下面的示例将 DepartmentID 属性设置为 System.ComponentModel.DataAnnotations.DatabaseGeneratedOption.None 以指示将由数据库生成值。  

``` csharp
modelBuilder.Entity<Department>().Property(t => t.DepartmentID)
    .HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);
```  

### <a name="specifying-the-maximum-length-on-a-property"></a>在属性上指定的最大长度  

在以下示例中，Name 属性应为不超过 50 个字符。 如果将值设置超过 50 个字符，则会收到[DbEntityValidationException](https://msdn.microsoft.com/library/system.data.entity.validation.dbentityvalidationexception.aspx)异常。 如果 Code First 从此模型创建数据库它还将名称列的最大长度为 50 个字符。  

``` csharp
modelBuilder.Entity<Department>().Property(t => t.Name).HasMaxLength(50);
```  

### <a name="configuring-the-property-to-be-required"></a>配置要为所需的属性  

在以下示例中，该名称属性是必需的。 如果不指定名称，则会 DbEntityValidationException 异常。 如果第一个代码从此模型创建数据库然后用于存储此属性的列通常是不可以为 null。  

> [!NOTE]
> 在某些情况下它不可能要为不可为 null，即使该属性是必需的数据库中的列。 例如，当对多个类型使用 TPH 继承策略数据存储在单个表。 如果派生的类型包括必需的属性列不能成为不可为 null 因为层次结构中的不是所有类型都将都具有此属性。  

``` csharp
modelBuilder.Entity<Department>().Property(t => t.Name).IsRequired();
```  

### <a name="configuring-an-index-on-one-or-more-properties"></a>上一个或多个属性配置索引  

> [!NOTE]
> **EF6.1 及更高版本仅**-Entity Framework 6.1 中引入了索引属性。 如果您使用的是早期版本不适用于此部分中的信息。  

创建索引并不本机支持通过 Fluent API，但可以使用支持的**IndexAttribute**通过 Fluent API。 索引属性由包括然后转换为更高版本中管道的数据库中的索引对模型的模型注释处理。 您可以手动添加这些相同使用 Fluent API 的批注。  

若要执行此操作的最简单方法是创建的实例**IndexAttribute** ，其中包含用于新索引的所有设置。 然后可以创建的实例**IndexAnnotation**即会将转换的 EF 特定类型**IndexAttribute**到可以存储在 EF 模型的模型批注的设置。 这些然后传递给**HasColumnAnnotation** Fluent api 中，指定名称的方法**索引**的批注。  

``` csharp
modelBuilder
    .Entity<Department>()
    .Property(t => t.Name)
    .HasColumnAnnotation("Index", new IndexAnnotation(new IndexAttribute()));
```  

有关中可用的设置的完整列表**IndexAttribute**，请参阅*索引*一部分[Code First 数据批注](~/ef6/modeling/code-first/data-annotations.md)。 这包括自定义索引名称、 创建唯一索引，以及创建多列索引。  

可以通过传递数组的单个属性上指定多个索引批注**IndexAttribute**的构造函数**IndexAnnotation**。  

``` csharp
modelBuilder
    .Entity<Department>()
    .Property(t => t.Name)
    .HasColumnAnnotation(
        "Index",  
        new IndexAnnotation(new[]
            {
                new IndexAttribute("Index1"),
                new IndexAttribute("Index2") { IsUnique = true }
            })));
```  

### <a name="specifying-not-to-map-a-clr-property-to-a-column-in-the-database"></a>指定不将映射到数据库中的列的 CLR 属性  

下面的示例演示如何指定 CLR 类型上的属性未映射到数据库中的列。  

``` csharp
modelBuilder.Entity<Department>().Ignore(t => t.Budget);
```  

### <a name="mapping-a-clr-property-to-a-specific-column-in-the-database"></a>映射到数据库中的特定列的 CLR 属性  

下面的示例将名称 CLR 属性映射到 DepartmentName 数据库列。  

``` csharp
modelBuilder.Entity<Department>()
    .Property(t => t.Name)
    .HasColumnName("DepartmentName");
```  

### <a name="renaming-a-foreign-key-that-is-not-defined-in-the-model"></a>重命名模型中未定义的外键  

如果您选择不在 CLR 类型上定义外键，但想要在数据库中指定它应具有什么名称，请执行以下操作：  

``` csharp
modelBuilder.Entity<Course>()
    .HasRequired(c => c.Department)
    .WithMany(t => t.Courses)
    .Map(m => m.MapKey("ChangedDepartmentID"));
```  

### <a name="configuring-whether-a-string-property-supports-unicode-content"></a>配置是否为字符串属性支持 Unicode 内容  

默认情况下字符串是 Unicode (SQL Server 中的 nvarchar)。 IsUnicode 方法可用于指定的 varchar 类型应为字符串。  

``` csharp
modelBuilder.Entity<Department>()
    .Property(t => t.Name)
    .IsUnicode(false);
```  

### <a name="configuring-the-data-type-of-a-database-column"></a>配置数据库列的数据类型  

[HasColumnType](https://msdn.microsoft.com/library/system.data.entity.modelconfiguration.configuration.stringcolumnconfiguration.hascolumntype.aspx)方法使映射到相同的基本类型的不同表示形式。 使用此方法不会启用你执行任何转换的数据在运行时。 请注意 IsUnicode 是 varchar、 设置列的首选的方式，这是因为它与数据库无关。  

``` csharp
modelBuilder.Entity<Department>()   
    .Property(p => p.Name)   
    .HasColumnType("varchar");
```  

### <a name="configuring-properties-on-a-complex-type"></a>复杂类型上配置属性  

有两种方法来配置上的复杂类型的标量属性。  

可以对 ComplexTypeConfiguration 调用属性。  

``` csharp
modelBuilder.ComplexType<Details>()
    .Property(t => t.Location)
    .HasMaxLength(20);
```  

此外可以使用点表示法访问复杂类型的属性。  

``` csharp
modelBuilder.Entity<OnsiteCourse>()
    .Property(t => t.Details.Location)
    .HasMaxLength(20);
```  

### <a name="configuring-a-property-to-be-used-as-an-optimistic-concurrency-token"></a>配置要用作开放式并发标记的属性  

若要指定实体中的某一属性表示并发标记，可以使用 ConcurrencyCheck 属性或 IsConcurrencyToken 方法。  

``` csharp
modelBuilder.Entity<OfficeAssignment>()
    .Property(t => t.Timestamp)
    .IsConcurrencyToken();
```  

IsRowVersion 方法还可用于将属性配置为在数据库中的行版本。 属性设置为行版本会自动将其配置为进行开放式并发标记。  

``` csharp
modelBuilder.Entity<OfficeAssignment>()
    .Property(t => t.Timestamp)
    .IsRowVersion();
```  

## <a name="type-mapping"></a>类型映射  

### <a name="specifying-that-a-class-is-a-complex-type"></a>指定类一种复杂类型  

按照约定，指定没有主键的类型被视为一种复杂类型。 有某些情况下，其中第一个代码将不会检测复杂类型 （例如，如果您具有一个名为 ID 属性，但您并不意味着它是主键）。 在这种情况下，将使用 fluent API 来显式指定类型是复杂类型。  

``` csharp
modelBuilder.ComplexType<Details>();
```  

### <a name="specifying-not-to-map-a-clr-entity-type-to-a-table-in-the-database"></a>指定不将 CLR 实体类型映射到数据库中的表  

下面的示例演示如何从映射到数据库中的表中排除的 CLR 类型。  

``` csharp
modelBuilder.Ignore<OnlineCourse>();
```  

### <a name="mapping-an-entity-type-to-a-specific-table-in-the-database"></a>一个实体类型映射到数据库中的特定表  

部门的所有属性将都映射到一个名为 t_ 部门表中的列。  

``` csharp
modelBuilder.Entity<Department>()  
    .ToTable("t_Department");
```  

此外可以指定架构名称如下：  

``` csharp
modelBuilder.Entity<Department>()  
    .ToTable("t_Department", "school");
```  

### <a name="mapping-the-table-per-hierarchy-tph-inheritance"></a>映射的每个层次结构一张表 (TPH) 继承  

TPH 映射方案中，在继承层次结构中的所有类型都映射到单个表。 鉴别器列用于标识每行的类型。 当使用 Code First 创建模型，TPH 是参与继承层次结构的类型的默认策略。 默认情况下，鉴别器列添加到具有名称"Discriminator"表和层次结构中的每种类型的 CLR 类型名称用于鉴别器值。 可以通过使用 fluent API 来修改默认行为。  

``` csharp
modelBuilder.Entity<Course>()  
    .Map<Course>(m => m.Requires("Type").HasValue("Course"))  
    .Map<OnsiteCourse>(m => m.Requires("Type").HasValue("OnsiteCourse"));
```  

### <a name="mapping-the-table-per-type-tpt-inheritance"></a>映射的每种类型一个表 (TPT) 继承  

TPT 映射方案中，在所有类型都映射到单独的表。 仅属于某个基类型或派生类型的属性存储在映射到该类型的一个表中。 用于派生类型的映射的表还存储将派生表与基础表的外键。  

``` csharp
modelBuilder.Entity<Course>().ToTable("Course");  
modelBuilder.Entity<OnsiteCourse>().ToTable("OnsiteCourse");
```  

### <a name="mapping-the-table-per-concrete-class-tpc-inheritance"></a>映射表每个具体类 (TPC) 继承  

在 TPC 映射方案中，层次结构中的所有非抽象类型映射到单独的表。 将映射到派生类的表格具有映射到数据库中的基类的表没有关系。 类，包括继承的属性的所有属性都映射到相应的表的列。  

调用 MapInheritedProperties 方法来配置每个派生的类型。 MapInheritedProperties 将重新映射到的表的派生类中的新列从基类继承的所有属性。  

> [!NOTE]
> 请注意，因为不共享参与 TPC 继承层次结构的表有主键将重复的实体键时如果具有生成的数据库值与相同的标识种子将映射到子类的表中插入。 若要解决此问题，您可以指定每个表的不同的初始种子值，也可以关闭主键属性上的标识。 使用 Code First 时，标识是整数键属性的默认值。  

``` csharp
modelBuilder.Entity<Course>()
    .Property(c => c.CourseID)
    .HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);

modelBuilder.Entity<OnsiteCourse>().Map(m =>
{
    m.MapInheritedProperties();
    m.ToTable("OnsiteCourse");
});

modelBuilder.Entity<OnlineCourse>().Map(m =>
{
    m.MapInheritedProperties();
    m.ToTable("OnlineCourse");
});
```  

### <a name="mapping-properties-of-an-entity-type-to-multiple-tables-in-the-database-entity-splitting"></a>实体类型的属性映射到数据库 （实体拆分） 中的多个表  

实体拆分允许要分布在多个表的实体类型的属性。 在以下示例中，在部门实体拆分为两个表： 部门和 DepartmentDetails。 实体拆分使用 Map 方法对多个调用映射到特定的表属性的子集。  

``` csharp
modelBuilder.Entity<Department>()
    .Map(m =>
    {
        m.Properties(t => new { t.DepartmentID, t.Name });
        m.ToTable("Department");
    })
    .Map(m =>
    {
        m.Properties(t => new { t.DepartmentID, t.Administrator, t.StartDate, t.Budget });
        m.ToTable("DepartmentDetails");
    });
```  

### <a name="mapping-multiple-entity-types-to-one-table-in-the-database-table-splitting"></a>多个实体类型映射到数据库 （表拆分） 中的一个表  

下面的示例将共享到一个表的主键的两个实体类型映射。  

``` csharp
modelBuilder.Entity<OfficeAssignment>()
    .HasKey(t => t.InstructorID);

modelBuilder.Entity<Instructor>()
    .HasRequired(t => t.OfficeAssignment)
    .WithRequiredPrincipal(t => t.Instructor);

modelBuilder.Entity<Instructor>().ToTable("Instructor");

modelBuilder.Entity<OfficeAssignment>().ToTable("Instructor");
```  

### <a name="mapping-an-entity-type-to-insertupdatedelete-stored-procedures-ef6-onwards"></a>一个实体类型映射到 Insert/Update/Delete 存储过程 (EF6 及更高版本)  

与 EF6 开始，可以将映射实体将存储的过程用于插入更新和删除。 有关更多详细信息，请参阅[代码第一个 Insert/Update/Delete 存储过程](~/ef6/modeling/code-first/fluent/cud-stored-procedures.md)。  

## <a name="model-used-in-samples"></a>示例中所用的模型  

下面的代码优先模型用于在此页上的示例。  

``` csharp
using System.Data.Entity;
using System.Data.Entity.ModelConfiguration.Conventions;
// add a reference to System.ComponentModel.DataAnnotations DLL
using System.ComponentModel.DataAnnotations;
using System.Collections.Generic;
using System;

public class SchoolEntities : DbContext
{
    public DbSet<Course> Courses { get; set; }
    public DbSet<Department> Departments { get; set; }
    public DbSet<Instructor> Instructors { get; set; }
    public DbSet<OfficeAssignment> OfficeAssignments { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Configure Code First to ignore PluralizingTableName convention
        // If you keep this convention then the generated tables will have pluralized names.
        modelBuilder.Conventions.Remove<PluralizingTableNameConvention>();
    }
}

public class Department
{
    public Department()
    {
        this.Courses = new HashSet<Course>();
    }
    // Primary key
    public int DepartmentID { get; set; }
    public string Name { get; set; }
    public decimal Budget { get; set; }
    public System.DateTime StartDate { get; set; }
    public int? Administrator { get; set; }

    // Navigation property
    public virtual ICollection<Course> Courses { get; private set; }
}

public class Course
{
    public Course()
    {
        this.Instructors = new HashSet<Instructor>();
    }
    // Primary key
    public int CourseID { get; set; }

    public string Title { get; set; }
    public int Credits { get; set; }

    // Foreign key
    public int DepartmentID { get; set; }

    // Navigation properties
    public virtual Department Department { get; set; }
    public virtual ICollection<Instructor> Instructors { get; private set; }
}

public partial class OnlineCourse : Course
{
    public string URL { get; set; }
}

public partial class OnsiteCourse : Course
{
    public OnsiteCourse()
    {
        Details = new Details();
    }

    public Details Details { get; set; }
}

public class Details
{
    public System.DateTime Time { get; set; }
    public string Location { get; set; }
    public string Days { get; set; }
}

public class Instructor
{
    public Instructor()
    {
        this.Courses = new List<Course>();
    }

    // Primary key
    public int InstructorID { get; set; }
    public string LastName { get; set; }
    public string FirstName { get; set; }
    public System.DateTime HireDate { get; set; }

    // Navigation properties
    public virtual ICollection<Course> Courses { get; private set; }
}

public class OfficeAssignment
{
    // Specifying InstructorID as a primary
    [Key()]
    public Int32 InstructorID { get; set; }

    public string Location { get; set; }

    // When Entity Framework sees Timestamp attribute
    // it configures ConcurrencyCheck and DatabaseGeneratedPattern=Computed.
    [Timestamp]
    public Byte[] Timestamp { get; set; }

    // Navigation property
    public virtual Instructor Instructor { get; set; }
}
```  
