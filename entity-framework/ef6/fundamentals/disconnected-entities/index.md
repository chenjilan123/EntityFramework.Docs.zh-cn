---
title: 使用断开连接的实体 - EF6
author: divega
ms.date: 10/23/2016
ms.assetid: 12138003-a373-4817-b1b7-724130202f5f
ms.openlocfilehash: 11ca2a9a4161e02d32d98bf03dd4cf28545334b7
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022166"
---
# <a name="working-with-disconnected-entities"></a>使用断开连接的实体
在基于实体框架的应用程序中，上下文类负责检测应用于已跟踪实体的更改。 调用 SaveChanges 方法会将上下文跟踪的更改保存到数据库。 使用 n 层应用程序时，实体对象通常会在从上下文断开连接时发生变动，且必须决定如何跟踪更改并向上下文报告这些更改。 本主题讨论使用实体框架和断开连接的实体时不同的可用选项。   

## <a name="web-service-frameworks"></a>Web 服务框架

Web 服务技术通常支持可用于在各个断开连接的对象上保存更改的模式。 例如，通过 ASP.NET Web API 可编写控制器操作，这些操作可以包括对 EF 的调用，以保存对数据库上的对象所做的更改。 实际上，Visual Studio 中的 Web API 工具可以轻松地从实体框架 6 模型构建 Web API 控制器。 有关详细信息，请参阅[一起使用 Web API 和实体框架 6](https://docs.microsoft.com/aspnet/web-api/overview/data/using-web-api-with-entity-framework/)。   

以前，还有其他一些 Web 服务技术提供与实体框架的集成，例如 [WCF 数据服务](https://docs.microsoft.com/dotnet/framework/data/wcf/create-a-data-service-using-an-adonet-ef-data-wcf)和 [RIA 服务](https://docs.microsoft.com/previous-versions/dotnet/wcf-ria/ee707344(v=vs.91))。

## <a name="low-level-ef-apis"></a>低级别 EF API

如果不想使用现有的 n 层解决方案，或者想要自定义 Web API 服务中控制器操作内发生的内容，则可通过实体框架提供的 API 应用在断开连接的层上所做的更改。 有关详细信息，请参阅[添加、附加和实体状态](~/ef6/saving/change-tracking/entity-state.md)。  

## <a name="self-tracking-entities"></a>自跟踪实体  

在从 EF 上下文断开连接时，跟踪实体的任意图上的更改是一个难题。 可采用自跟踪实体代码生成模板来尝试解决。 此模板生成实体类作为实体本身的状态，该类包含可跟踪断开连接的层上所做更改的逻辑。 还会生成一组扩展方法，将这些更改应用于上下文。

此模板可与使用 EF 设计器创建的模型一起使用，但不能与 Code First 模型一起使用。 有关详细信息，请参阅[自跟踪实体](self-tracking-entities/index.md)。  

> [!IMPORTANT]
> 我们不再建议使用自跟踪实体模板。 它将仅继续用于支持现有应用程序。 如果应用程序需要使用断开连接的实体图，请考虑其他替代方案，例如[可跟踪实体](http://trackableentities.github.io/)，它与自跟踪实体类似，社区在更积极地开发这种技术，或使用低级别更改跟踪 API 编写自定义代码。
