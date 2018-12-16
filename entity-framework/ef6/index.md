---
title: 概述 - EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 8ae74d63-6bad-4686-b325-bbf9d68f3743
uid: ef6/index
ms.openlocfilehash: 1efadf4484a13d5df2a2f11aad3d0e8f9ceff543
ms.sourcegitcommit: 8b42045cd21f80f425a92f5e4e9dd4972a31720b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2018
ms.locfileid: "49315628"
---
# <a name="entity-framework-6"></a>Entity Framework 6
实体框架 6 (EF6) 是经试验和测试的关系映射器 (O/RM)，适用于 .NET 的对象，其功能和稳定性经过了多年的开发和调试。

作为 O/RM，EF6 降低了关系方面和面向对象的方面之间的阻抗不匹配，使开发人员能够使用表示应用程序域的强类型 .NET 对象来编写应用程序，该应用程序可与存储在关系数据库中的数据交互，同时使开发人员无需再编写大部分的数据访问“管道”代码。

EF6 可实现许多热门 O/RM 功能：
- 不依赖于任何 EF 类型的 [POCO](~/ef6/resources/glossary.md#poco) 实体类的映射
- 自动更改跟踪
- 标识解析和工作单元
- 预先、延迟和显式加载
- 使用 LINQ（语言集成查询）转换强类型查询
- 丰富的映射功能，可支持：
  - 一对一、一对多和多对多关系
  - 继承（每个层次结构一张表、每个类型一张表和每个具体类一张表）
  - 复杂类型
  - 存储过程
- 通过可视化设计器创建实体模型。
- 通过编写代码创建实体模型的“Code First”体验。
- 既可从现有数据库生成模型，然后手动编辑，也可从头开始创建模型，然后用于生成新的数据库。
- 与 .NET Framework 应用程序模型（包括 ASP.NET）集成，并通过数据绑定与 WPF 和 WinForms 集成。
- 基于 ADO.NET 的数据库连接和可用于连接到 SQL Server、Oracle、MySQL、SQLite、PostgreSQL、DB2 等的众多提供程序。

## <a name="should-i-use-ef6-or-ef-core"></a>应使用 EF6 还是 EF Core？

EF Core 是更现代、可扩展的轻量级实体框架版本，与 EF6 的功能和优点非常相似。
EF Core 则完全进行了重写，包含许多 EF6 没有的新功能，但还是缺少 EF6 中最高级的一些映射功能。
如果功能集与需求匹配，请考虑在新应用程序中使用 EF Core。
[比较 EF Core 和 EF6](xref:efcore-and-ef6/index)中更详细地讨论了此选项。

## <a name="get-startedef6get-startedmd"></a>[入门](~/ef6/get-started.md)

将 EntityFramework NuGet 包添加到项目或安装适用于 Visual Studio 的 Entity Framework Tools。 然后观看视频、阅读教程和高级文档，以充分利用 EF6。

## <a name="past-entity-framework-versions"></a>过去的实体框架版本

本文档针对的是最新版本的实体框架 6 ，但其中大部分内容也适用于过去的版本。
请查看[新增功能](~/ef6/what-is-new/index.md)和[过去的版本](~/ef6/what-is-new/past-releases.md)，了解 EF 版本和其中引入的功能的完整列表。
