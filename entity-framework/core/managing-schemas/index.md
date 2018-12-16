---
title: 管理数据库架构 - EF Core
author: bricelam
ms.date: 10/30/2017
ms.openlocfilehash: c1ebe33b5575cab76a54721ef86ecbcb7ff8b98b
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42994380"
---
# <a name="managing-database-schemas"></a>管理数据库架构
EF Core 提供两种主要方法来保持 EF Core 模型和数据库架构同步。至于我们应该选用哪个方法，请确定你是希望以 EF Core 模型为准还是以数据库为准。

如果希望以 EF Core 模型为准，请使用[迁移][1]。 对 EF Core 模型进行更改时，此方法会以增量方式将相应架构更改应用到数据库，以使数据库保持与 EF Core 模型兼容。

如果希望以数据库架构为准，请使用[反向工程][2]。 使用此方法，可通过将数据库架构反向工程到 EF Core 模型来生成相应的 DbContext 和实体类型。

> [!NOTE]
> [创建和删除 API][3] 也可从 EF Core 模型创建数据库架构。 但是，它们主要适用于允许删除数据库的场景，比如测试、原型制作等。


  [1]: migrations/index.md
  [2]: scaffolding.md
  [3]: ensure-created.md
