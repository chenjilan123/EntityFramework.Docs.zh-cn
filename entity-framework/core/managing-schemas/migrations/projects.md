---
title: 多个项目的 EF Core 的迁移
author: bricelam
ms.author: bricelam
ms.date: 10/30/2017
uid: core/managing-schemas/migrations/projects
ms.openlocfilehash: 30a6afad1488e74ce2585be3d780186311379a97
ms.sourcegitcommit: ad1bdea58ed35d0f19791044efe9f72f94189c18
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/28/2018
ms.locfileid: "47447139"
---
<a name="using-a-separate-project"></a>使用一个独立的项目
========================
你可能希望将迁移保存在一个独立的程序集当中，而不是和`DbContext`放在同一个程序集中。 你也可以通过这个策略维护多个迁移集合（针对同一个DbContext）。比如，一份迁移用于开发环境升级，另一份用于发布环境的升级。

为此...

1. 创建一个新的类库。

2. 为该类库添加对 DbContext 所在程序集的引用。

3. 将迁移和模型快照文件移动到这个新类库。
   > [!TIP]
   > 如果没有现存的迁移，必须先在 DbContext 所在项目中生成一个迁移，再将其移到上述独立类库项目中。 这非常重要，因为如果迁移程序集不包含现有的迁移，将找不到 DbContext Add-migration 命令。

4. 配置迁移程序集：

   ``` csharp
   options.UseSqlServer(
       connectionString,
       x => x.MigrationsAssembly("MyApp.Migrations"));
   ```

5. 为迁移程序集添加对启动程序集的引用。
   * 如果此操作导致循环依赖，则需修改类库的输出路径：

     ``` xml
     <PropertyGroup>
       <OutputPath>..\MyStartupProject\bin\$(Configuration)\</OutputPath>
     </PropertyGroup>
     ```

如果您的操作全部正确，您应能够向项目添加新迁移。

``` powershell
Add-Migration NewMigration -Project MyApp.Migrations
```
``` Console
dotnet ef migrations add NewMigration --project MyApp.Migrations
```
