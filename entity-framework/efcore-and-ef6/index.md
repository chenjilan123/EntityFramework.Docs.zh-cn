---
title: 比较 Entity Framework 6 和 Entity Framework Core
description: 介绍如何在 Entity Framework 6 和 Entity Framework Core 之间进行选择。
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: a6b9cd22-6803-4c6c-a4d4-21147c0a81cb
uid: efcore-and-ef6/index
ms.openlocfilehash: d5fe9b388707f653fdeb2d6a5daa7215ced71c1d
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688714"
---
# <a name="compare-ef-core--ef6"></a>比较 EF Core 和 EF6

Entity Framework 是适用于.NET 的对象关系映射程序 (O/RM)。 本文比较了两个版本：Entity Framework 6 和 Entity Framework Core。

## <a name="entity-framework-6"></a>Entity Framework 6

Entity Framework 6 (EF6) 是一种久经验证的数据访问技术。 2008 年，它作为 .NET Framework 3.5 SP1 和 Visual Studio 2008 SP1 的一部分首次发布。 从 4.1 版开始，已作为 [EntityFramework](https://www.nuget.org/packages/EntityFramework/) NuGet 包发布。 EF6 在 .NET Framework 4.x 上运行，这意味着它仅在 Windows 上运行。 

EF6 产品仍受支持，并将继续提供 bug 修复和细微改进。

## <a name="entity-framework-core"></a>Entity Framework Core

Entity Framework Core (EF Core) 是在 2016 年首次发布的 EF6 的完全重写。 它附带于 Nuget 包中，是 [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/) 的主要组成部分。 EF Core 是一种跨平台产品，可以在 .NET Core 或 .NET Framework 上运行。

EF Core 旨在提供类似于 EF6 的开发人员体验。 大多数顶级 API 保持不变，因此，用过 EF6 的开发人员都会对 EF Core 感到很熟悉。

## <a name="feature-comparison"></a>功能比较

EF Core 提供了在 EF6 中不会实现的新功能（如[备选键](xref:core/modeling/alternate-keys)、[批量更新](xref:core/what-is-new/ef-core-1.0#relational-batching-of-statements)以及 [LINQ 查询中的混合客户端/数据库评估](xref:core/querying/client-eval)。 但由于它是一个新代码库，所以会缺少一些 EF6 中的功能。

下表比较了 EF Core 和 EF6 中可用的功能。 这只是大致比较，没有列出全部功能或解释不同 EF 版本中相同功能之间的差异。

EF Core 列指出了功能首次出现的产品版本。

### <a name="creating-a-model"></a>创建模型

| **功能**                                           | EF 6 | **EF Core**                           |
|:------------------------------------------------------|:---------|:--------------------------------------|
| 基本类映射                                   | 是      | 1.0                                   |
| 带有参数的构造函数                          |          | 2.1                                   |
| 属性值转换                            |          | 2.1                                   |
| 没有键的映射类型（查询类型）               |          | 2.1                                   |
| 约定                                           | 是      | 1.0                                   |
| 自定义约定                                    | 是      | 1.0（部分）                         |
| 数据注释                                      | 是      | 1.0                                   |
| Fluent API                                            | 是      | 1.0                                   |
| 继承：每个层次结构一张表 (TPH)                | 是      | 1.0                                   |
| 继承：每个类型一张表 (TPT)                     | 是      |                                       |
| 继承：每个具体类一张表 (TPC)           | 是      |                                       |
| 阴影状态属性                               |          | 1.0                                   |
| 备用键                                        |          | 1.0                                   |
| 多对多，无联接实体                      | 是      |                                       |
| 密钥生成：数据库                              | 是      | 1.0                                   |
| 密钥生成：客户端                                |          | 1.0                                   |
| 复杂/已拥有类型                                   | 是      | 2.0                                   |
| 空间数据                                          | 是      | 2.2                                   |
| 模型的图形可视化效果                      | 是      |                                       |
| 图形模型编辑器                                | 是      |                                       |
| 模型格式：代码                                    | 是      | 1.0                                   |
| 模型格式：EDMX (XML)                              | 是      |                                       |
| 从数据库创建模型：命令行              | 是      | 1.0                                   |
| 从数据库创建模型：VS 向导                 | 是      |                                       |
| 从数据库更新模型                            | 部分  |                                       |
| 全局查询筛选器                                  |          | 2.0                                   |
| 表拆分                                       | 是      | 2.0                                   |
| 实体拆分                                      | 是      |                                       |
| 数据库标量函数映射                      | 差     | 2.0                                   |
| 字段映射                                         |          | 1.1                                   |

### <a name="querying-data"></a>查询数据

| **功能**                                           | EF6  | **EF Core**                           |
|:------------------------------------------------------|:---------|:--------------------------------------|
| LINQ 查询                                          | 是      | 1.0（进行中，针对复杂查询） |
| 可读内容生成的 SQL                                | 差     | 1.0                                   |
| 混合客户端/服务器评估                        |          | 1.0                                   |
| GroupBy 转换                                   | 是      | 2.1                                   |
| 加载相关数据：预先加载                           | 是      | 1.0                                   |
| 加载相关数据：预先加载派生类型 |          | 2.1                                   |
| 加载相关数据：延迟加载                            | 是      | 2.1                                   |
| 加载相关数据：显式加载                        | 是      | 1.1                                   |
| 原始 SQL 查询：实体类型                         | 是      | 1.0                                   |
| 原始 SQL 查询：非实体类型（例如查询类型）       | 是      | 2.1                                   |
| 原始 SQL 查询：使用 LINQ 编写                  |          | 1.0                                   |
| 显式编译的查询                           | 差     | 2.0                                   |
| 基于文本的查询语言（实体 SQL）                | 是      |                                       |

### <a name="saving-data"></a>保存数据

| **功能**                                           | EF6  | EF Core                           |
|:------------------------------------------------------|:---------|:--------------------------------------|
| 更改跟踪：快照                             | 是      | 1.0                                   |
| 更改追踪：通知                         | 是      | 1.0                                   |
| 更改跟踪：代理                              | 是      |                                       |
| 访问跟踪的状态                               | 是      | 1.0                                   |
| 开放式并发                                | 是      | 1.0                                   |
| 事务                                          | 是      | 1.0                                   |
| 批处理语句                                |          | 1.0                                   |
| 存储过程映射                              | 是      |                                       |
| 断开连接低级别 API 图形                     | 差     | 1.0                                   |
| 断开连接端到端图形                         |          | 1.0（部分）                         |

### <a name="other-features"></a>其他功能

| **功能**                                           | EF6  | **EF Core**                           |
|:------------------------------------------------------|:---------|:--------------------------------------|
| 迁移                                            | 是      | 1.0                                   |
| 数据库创建/删除 API                       | 是      | 1.0                                   |
| 种子数据                                             | 是      | 2.1                                   |
| 连接复原                                 | 是      | 1.1                                   |
| 生命周期挂钩（事件、截取）                | 是      |                                       |
| 简单的日志记录 (Database.Log)                         | 是      |                                       |
| DbContext 池                                     |          | 2.0                                   |

### <a name="database-providers"></a>数据库提供程序

| **功能**                                           | EF6  | **EF Core**                           |
|:------------------------------------------------------|:---------|:--------------------------------------|
| SQL Server                                            | 是      | 1.0                                   |
| MySQL                                                 | 是      | 1.0                                   |
| postgresql                                            | 是      | 1.0                                   |
| Oracle                                                | 是      | 1.0 <sup>(1)</sup>                    |
| SQLite                                                | 是      | 1.0                                   |
| SQL Server Compact                                    | 是      | 1.0 <sup>(2)</sup>                    |
| DB2                                                   | 是      | 1.0                                   |
| Firebird                                              | 是      | 2.0                                   |
| Jet (Microsoft Access)                                |          | 2.0 <sup>(2)</sup>                    |
| 内存中（用于测试）                               |          | 1.0                                   |

<sup>1</sup> 目前提供适用于 Oracle 的付费提供程序。 适用于 Oracle 的免费官方提供程序目前正在开发中。

<sup>2</sup> SQL Server Compact 和 Jet 提供程序仅适用于 .NET Framework（而不适用于 .NET Core）。

### <a name="net-implementations"></a>.NET 实现

| **功能**                                           | EF6  | **EF Core**                           |
|:------------------------------------------------------|:---------|:--------------------------------------|
| .NET framework（控制台、WinForms、WPF、ASP.NET）      | 是      | 1.0                                   |
| .NET Core（控制台、ASP.NET Core）                     |          | 1.0                                   |
| Mono 和 Xamarin                                        |          | 1.0（进行中）                     |
| UWP                                                   |          | 1.0（进行中）                     |

## <a name="guidance-for-new-applications"></a>针对新应用程序的选择指南

如果以下两个条件都成立，请考虑为新应用程序使用 EF Core：
* 应用需要.NET Core 的功能。 有关详细信息，请参阅[在 .NET Core 和 .NET Framework 之间进行选择以用于服务器应用](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server)。
* EF Core 支持应用需要的所有功能。 如果缺少所需的功能，请检查 [EF Core 路线图](xref:core/what-is-new/roadmap)，了解是否有计划在将来为其提供支持。 

如果以下两个条件都成立，请考虑使用 EF6：
* 应用将在 Windows 和 .NET Framework 4.0 或更高版本上运行。
* EF6 支持应用需要的所有功能。

## <a name="guidance-for-existing-ef6-applications"></a>针对现有 EF6 应用程序的选择指南

由于 EF Core 有一些根本性变化，我们不建议将 EF6 应用程序迁移至 EF Core，除非有令人信服的理由进行此项更改。 如果想迁移到 EF Core 以使用新功能，请确保你了解其各项限制。 有关详细信息，请参阅[从 EF6 到 EF Core 的迁移](porting/index.md)。 从 EF6 到 EF Core 更像是迁移而不是升级。 

## <a name="next-steps"></a>后续步骤

有关详细信息，请参阅文档：
* <xref:core/index>
* <xref:ef6/index>
