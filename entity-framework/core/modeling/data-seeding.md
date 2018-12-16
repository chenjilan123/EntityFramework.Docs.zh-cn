---
title: 数据种子设定的 EF Core
author: AndriySvyryd
ms.author: ansvyryd
ms.date: 11/02/2018
ms.assetid: 3154BF3C-1749-4C60-8D51-AE86773AA116
uid: core/modeling/data-seeding
ms.openlocfilehash: 8f28dfea12461572ade8fbf3910ebd216dafb389
ms.sourcegitcommit: fa863883f1193d2118c2f9cee90808baa5e3e73e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2018
ms.locfileid: "52857424"
---
# <a name="data-seeding"></a>数据种子设定

数据种子设定是用一组初始的数据填充数据库的过程。

有几种方法可以在 EF Core 中完成此：
* 模型种子数据
* 手动迁移自定义项
* 自定义初始化逻辑

## <a name="model-seed-data"></a>模型种子数据

> [!NOTE]
> 此功能是在 EF Core 2.1 的新增功能。

不同于在 EF6 中，EF Core 中设定数据种子会与作为模型配置的一部分的实体类型相关联。 然后 EF Core [迁移](xref:core/managing-schemas/migrations/index)什么插入、 更新或删除操作需要时将数据库升级到新版本的模型的应用可以自动计算。

> [!NOTE]
> 确定应执行什么操作以将种子数据添加到所需的状态时，迁移只考虑模型更改。 因此对数据执行迁移之外的任何更改可能会丢失，或导致错误。

例如，这会将配置的种子数据`Blog`在`OnModelCreating`:

[!code-csharp[BlogSeed](../../../samples/core/Modeling/DataSeeding/DataSeedingContext.cs?name=BlogSeed)]

若要添加实体具有关系的外键值的需要指定：

[!code-csharp[PostSeed](../../../samples/core/Modeling/DataSeeding/DataSeedingContext.cs?name=PostSeed)]

如果实体类型具有卷影状态中的任何属性的匿名类可用于提供的值：

[!code-csharp[AnonymousPostSeed](../../../samples/core/Modeling/DataSeeding/DataSeedingContext.cs?name=AnonymousPostSeed)]

固有的实体类型可以以类似的方式进行种子设定：

[!code-csharp[OwnedTypeSeed](../../../samples/core/Modeling/DataSeeding/DataSeedingContext.cs?name=OwnedTypeSeed)]

请参阅[完整的示例项目](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Modeling/DataSeeding)有关更多上下文。

将数据添加到模型中后,[迁移](xref:core/managing-schemas/migrations/index)应该用于将应用所做的更改。

> [!TIP]
> 如果你需要将迁移应用自动部署的一部分可以[创建一个 SQL 脚本](xref:core/managing-schemas/migrations/index#generate-sql-scripts)，可以预览之前执行。

或者，可以使用`context.Database.EnsureCreated()`若要创建新的数据库包含种子数据，例如对于测试数据库，或者在使用内存中提供程序或任何非关系数据库。 请注意，如果数据库已存在，`EnsureCreated()`既不会更新数据库中的架构和种子数据。 对于关系数据库不应调用`EnsureCreated()`如果你打算使用迁移。

此类型的种子数据由迁移，该脚本以更新已在数据库中的数据需要为其生成无需连接到数据库。 这具有一些限制：
* 需要指定即使它通常由数据库生成的主键值。 它将用于检测之间迁移的数据更改。
* 以前如果以任何方式更改主键植入的数据将被删除。

因此，此功能是最适用于具有不希望变化迁移之外，而不依赖任何其他数据库，例如 ZIP 代码中的静态数据。

如果你的方案包括以下任一建议使用自定义初始化逻辑中的最后一节所述：
* 临时数据以供测试
* 取决于数据库状态的数据
* 需要由数据库，包括备用键用作标识的实体生成的密钥值的数据
* 需要自定义转换的数据 (未由[值转换](xref:core/modeling/value-conversions))，如某些密码哈希
* 需要对外部 API 调用，例如 ASP.NET Core 标识的角色和用户创建的数据

## <a name="manual-migration-customization"></a>手动迁移自定义项

当添加迁移所做的更改与指定的数据`HasData`转换为调用`InsertData()`， `UpdateData()`，和`DeleteData()`。 解决的一些的限制的一种方法`HasData`是手动添加这些调用或[自定义操作](xref:core/managing-schemas/migrations/operations)到迁移相反。

[!code-csharp[CustomInsert](../../../samples/core/Modeling/DataSeeding/Migrations/20181102235626_Initial.cs?name=CustomInsert)]

## <a name="custom-initialization-logic"></a>自定义初始化逻辑

将使用一种简单、 功能强大的方法来执行数据种子设定[ `DbContext.SaveChanges()` ](xref:core/saving/index)逻辑开始执行之前在主应用程序。

[!code-csharp[Main](../../../samples/core/Modeling/DataSeeding/Program.cs?name=CustomSeeding)]

> [!WARNING]
> 因为当多个实例都运行，并且还需要有权修改数据库架构的应用，这可能会导致并发问题，种子设定代码不应正常的应用程序执行的一部分。

具体取决于你的部署的约束可以以不同的方式执行初始化代码：
* 初始化应用程序在本地运行
* 正在初始化应用程序与主应用程序，调用初始化例程，然后禁用或删除初始化应用的部署。

通常可通过使用自动[发布配置文件](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/visual-studio-publish-profiles)。
