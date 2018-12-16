---
title: Entity Framework Core 路线图
author: divega
ms.date: 02/20/2018
ms.assetid: 834C9729-7F6E-4355-917D-DE3EE9FE149E
uid: core/what-is-new/roadmap
ms.openlocfilehash: a12d628a28515f0c6710bfa59bc6dcdf41fcb58b
ms.sourcegitcommit: b3c2b34d5f006ee3b41d6668f16fe7dcad1b4317
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2018
ms.locfileid: "51688584"
---
# <a name="entity-framework-core-roadmap"></a>Entity Framework Core 路线图

> [!IMPORTANT]
> 请注意，将来版本的功能集和计划会发生更改，尽管我们会尽力使此页面保持最新状态，但它可能不会始终反映我们的最新计划。

### <a name="ef-core-22"></a>EF Core 2.2

此版本将包含多个 Bug 修复和少量新功能。 此版本计划于 2018 年底发布。 若要详细了解此版本，请参阅 [EF Core 2.2 中的新变化](xref:core/what-is-new/ef-core-2.2)。 

### <a name="ef-core-30"></a>EF Core 3.0

我们计划发布与 .NET Core 3.0 和 ASP.NET 3.0 对接的主要 EF Core 版本，但尚未完成它的[发布计划流程](#release-planning-process)。

若要查看暂时分配给 3.0 的工作项，请[在我们的问题跟踪程序中使用此查询](https://github.com/aspnet/EntityFrameworkCore/issues?q=is%3Aopen+is%3Aissue+milestone%3A3.0.0+sort%3Areactions-%2B1-desc)。

## <a name="schedule"></a>计划

EF Core 的计划与 [.NET Core 计划](https://github.com/dotnet/core/blob/master/roadmap.md)以及 [ASP.NET Core 计划](https://github.com/aspnet/Home/wiki/Roadmap)同步。

## <a name="backlog"></a>积压工作 (backlog)

在我们的问题跟踪程序中，[积压工作 (backlog) 里程碑](https://github.com/aspnet/EntityFrameworkCore/issues?q=is%3Aopen+is%3Aissue+milestone%3ABacklog+sort%3Areactions-%2B1-desc)包含我们希望今后能解决的问题，或我们认为社区中的某人可以解决的问题。
客户可随时提交对这些问题的评论和投票。
建议着手解决其中任意问题的参与者先开始讨论如何处理这些问题。

我们从来没有保证过，将努力在特定版本的 EF Core 中提供任何给定功能。
与所有软件项目一样，优先级、发布计划和可用资源可能会随时发生变化。
不过，如果我们打算在特定时间范围内解决某个问题，我们将会把问题分配到发布里程碑，而不是积压工作 (backlog) 里程碑。
在[发布计划流程](#release-planning-process)中，我们会定期在积压工作 (backlog) 里程碑和发布里程碑之间移动问题。

如果我们不打算解决某个问题，可能会关闭它。
不过，对于之前关闭的问题，如果我们获得了它的新信息，也可以重新考虑解决它。

## <a name="release-planning-process"></a>版本规划过程

我们常常会被问到如何选择将添加到特定版本的特定功能。
积压工作 (backlog) 绝不会自动转换成发布计划。
如果 EF6 中有某项功能，也并不意味着需要在 EF Core 中实现此功能。

很难详细说明我们遵循的整个发布计划流程。
大多数都是在讨论具体功能、机会和优先级，而且流程本身也会随着每次发布而发生演变。
不过，我们可以总结在确定接下来工作内容时尝试回答的常见问题：

1. **我们认为有多少开发人员会使用该功能？该功能会使他们的应用程序/体验有多大的改善？** 为了回答这个问题，我们收集众多来源（其中包括对问题的评论和投票）的反馈。

2. **在尚未实现此功能的情况下，用户可用的变通方法是什么？** 例如，许多开发人员可以映射联接表，以解决缺少本机多对多支持的问题。 显然，并非所有开发人员都希望这样做，但很多开发人员都可以这样做，而这也作为决策的一个考虑因素。

3. **实现此功能是否有助于 EF Core 的体系结构发展，从而帮助我们实现其他功能？** 我们往往偏爱充当其他功能的构建基块的功能。 例如，属性包实体可有助于提供多对多支持，而且实体构造函数也启用了延迟加载支持。 

4. **功能是可扩展性点吗？** 我们往往偏爱扩展点（而不是常规功能），因为它们能让开发人员挂钩自己的行为，并补偿缺少的任何功能。 

5. **该功能与其他产品结合使用时的增效作用如何？** 我们往往偏爱能够实现或显著改善结合使用 EF Core 与其他产品（如 .NET Core、最新版 Visual Studio、Microsoft Azure 等）的体验的功能。

6. **从事功能开发工作的人员的技能如何？如何最充分地利用这些资源？** EF 团队的每个成员以及我们的社区参与者都拥有各个领域的不同程度经验，我们需要相应地进行计划。 即便我们希望“全员就位”开发特定功能（如 GroupBy 转换或多对多支持），这也是不切实际的。

如前所述，发布计划流程在每次发布时都会发生演变。
今后，我们将尝试为社区成员提供更多机会，让他们能够对发布计划提供意见。
例如，我们希望查看功能和发布计划本身的设计草稿能够变得更简单。