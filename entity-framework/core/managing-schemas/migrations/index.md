---
title: 迁移 - EF Core
author: bricelam
ms.author: bricelam
ms.date: 10/05/2018
uid: core/managing-schemas/migrations/index
ms.openlocfilehash: 5ae06a4342a556936dc44c5bf6622814eaad4733
ms.sourcegitcommit: 7a7da65404c9338e1e3df42576a13be536a6f95f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2018
ms.locfileid: "48834741"
---
<a name="migrations"></a>迁移
==========

开发期间，数据模型将发生更改并与数据库不同步。 可以删除该数据库，让 EF 创建一个新的数据库来匹配该模型，但此过程会导致数据丢失。 EF Core 中的迁移功能能够以递增方式更新数据库架构，使其与应用程序的数据模型保持同步，同时保留数据库中的现有数据。

迁移包括命令行工具和 API，可协助我们执行以下任务：

* [创建迁移](#create-a-migration)。 生成用于更新数据库的代码，使数据库与一系列模型更改同步。
* [更新数据库](#update-the-database)。 应用挂起的迁移更新数据库架构。
* [自定义迁移代码](#customize-migration-code)。 有时，需要修改或补充生成的代码。
* [删除迁移](#remove-a-migration)。 删除生成的代码。
* [还原迁移](#revert-a-migration)。 撤消数据库更改。
* [生成 SQL 脚本](#generate-sql-scripts)。 可能需要一个脚本来更新生产数据库，或者对迁移代码进行故障排除。
* [在运行时应用迁移](#apply-migrations-at-runtime)。 如果在设计期间更新和运行脚本不是最佳选项时，可在运行时调用 `Migrate()` 方法。

<a name="install-the-tools"></a>安装工具
-----------------

安装[命令提示符工具](xref:core/miscellaneous/cli/index)：
* 对于 Visual Studio，建议使用 [包管理器控制台工具](xref:core/miscellaneous/cli/powershell)。
* 对于其他开发环境，请选择 [.NET Core CLI 工具](xref:core/miscellaneous/cli/dotnet)。

<a name="create-a-migration"></a>创建迁移
------------------

[定义初始模型](xref:core/modeling/index)后，即应创建数据库。 若要添加初始迁移，请运行以下命令。

``` powershell
Add-Migration InitialCreate
```
``` Console
dotnet ef migrations add InitialCreate
```

向**Migrations**目录下的项目添加以下三个文件：

* **00000000000000_InitialCreate.cs**--主迁移文件。 包含应用迁移所需的操作（在 `Up()` 中）和还原迁移所需的操作（在 `Down()` 中）。
* **00000000000000_InitialCreate.Designer.cs**--迁移元数据文件。 包含 EF 所用的信息。
* **MyContextModelSnapshot.cs**--当前模型的快照。 用于确定添加下一迁移时的更改内容。

文件名中的时间戳有助于保证文件按时间顺序排列，以便你查看更改情况。

> [!TIP]
> 可以自由移动“Migrations”目录下的迁移文件并更改其命名空间。 新建的迁移和上个迁移同级。

<a name="update-the-database"></a>更新数据库
-------------------

接下来，将迁移应用到数据库以创建架构。

``` powershell
Update-Database
```
``` Console
dotnet ef database update
```

<a name="customize-migration-code"></a>自定义迁移代码
------------------------

更改 EF Core 模型后，数据库架构可能不同步。为使其保持最新，请再添加一个迁移。 迁移名称的用途与版本控制系统中的提交消息类似。 例如，如果更改对于评审是一个新的实体类，可以选择一个名称，如 AddProductReviews。

``` powershell
Add-Migration AddProductReviews
```
``` Console
dotnet ef migrations add AddProductReviews
```

搭建迁移基架（为其生成代码）后，检查代码的准确性，并添加、删除或修改正确应用代码所需的任何操作。

例如，迁移可能包含以下操作：

``` csharp
migrationBuilder.DropColumn(
    name: "FirstName",
    table: "Customer");

migrationBuilder.DropColumn(
    name: "LastName",
    table: "Customer");

migrationBuilder.AddColumn<string>(
    name: "Name",
    table: "Customer",
    nullable: true);
```

虽然这些操作可使数据库架构兼容，但是它们不会保留现有客户姓名。 如下所示，我们可以重写以改善这一情况。

``` csharp
migrationBuilder.AddColumn<string>(
    name: "Name",
    table: "Customer",
    nullable: true);

migrationBuilder.Sql(
@"
    UPDATE Customer
    SET Name = FirstName + ' ' + LastName;
");

migrationBuilder.DropColumn(
    name: "FirstName",
    table: "Customer");

migrationBuilder.DropColumn(
    name: "LastName",
    table: "Customer");
```

> [!TIP]
> 当某个操作可能会导致数据丢失（例如删除某列），搭建迁移基架过程将对此发出警告。 如果看到此警告，务必检查迁移代码的准确性。

使用相应命令将迁移应用到数据库。

``` powershell
Update-Database
```
``` Console
dotnet ef database update
```

### <a name="empty-migrations"></a>空迁移

有时模型未变更，直接添加迁移也很有用处。 在这种情况下，添加新迁移会创建一个带空类的代码文件。 可以自定义此迁移，执行与 EF Core 模型不直接相关的操作。 可能需要通过此方式管理的一些事项包括：

* 全文搜索
* 函数
* 存储过程
* 触发器
* 视图

<a name="remove-a-migration"></a>删除迁移
------------------
有时，你可能在添加迁移后意识到需要在应用迁移前对 EF Core 模型作出其他更改。 要删除上个迁移，请使用如下命令。

``` powershell
Remove-Migration
```
``` Console
dotnet ef migrations remove
```

删除迁移后，可对模型作出其他更改，然后再次添加迁移。

<a name="revert-a-migration"></a>还原迁移
------------------
如果已对数据库应用一个迁移（或多个迁移），但需将其复原，则可使用同一命令来应用迁移，并指定回退时的目标迁移名称。

``` powershell
Update-Database LastGoodMigration
```
``` Console
dotnet ef database update LastGoodMigration
```

<a name="generate-sql-scripts"></a>生成 SQL 脚本
--------------------
调试迁移或将其部署到生产数据库时，生成一个 SQL 脚本很有帮助。 之后可进一步检查该脚本的准确性，并对其作出调整以满足生产数据库的需求。 该脚本还可与部署技术结合使用。 基本命令如下。

``` powershell
Script-Migration
```
``` Console
dotnet ef migrations script
```

此命令有几个选项。

from 迁移应是运行该脚本前应用到数据库的最后一个迁移。 如果未应用任何迁移，请指定 `0`（默认值）。

to 迁移是运行该脚本后应用到数据库的最后一个迁移。 它默认为项目中的最后一个迁移。

可以选择生成 idempotent 脚本。 此脚本仅会应用尚未应用到数据库的迁移。 如果不确知应用到数据库的最后一个迁移或需要部署到多个可能分别处于不同迁移的数据库，此脚本非常有用。

<a name="apply-migrations-at-runtime"></a>在运行时应用迁移
---------------------------
启动或首次运行期间，一些应用可能需要在运行时应用迁移。 为此，请使用 `Migrate()` 方法。

此方法构建于 `IMigrator` 服务之上，该服务可用于更多高级方案。 请使用 `DbContext.GetService<IMigrator>()` 进行访问。

``` csharp
myDbContext.Database.Migrate();
```

> [!WARNING]
> * 此方法并不适合所有人。 尽管此方法非常适合具有本地数据库的应用，但是大多数应用程序需要更可靠的部署策略，例如生成 SQL 脚本。
> * 请勿在 `Migrate()` 前调用 `EnsureCreated()`。 `EnsureCreated()` 会绕过迁移创建架构，这会导致 `Migrate()` 失败。

<a name="next-steps"></a>后续步骤
----------

有关更多信息，请参见<xref:core/miscellaneous/cli/index>。