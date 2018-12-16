---
title: 查询数据 - EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: 7c65ec3e-46c8-48f8-8232-9e31f96c277b
uid: core/querying/index
ms.openlocfilehash: 51aaa5de11d3fe38b4fba82db8dcb5658088cc27
ms.sourcegitcommit: dadee5905ada9ecdbae28363a682950383ce3e10
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/27/2018
ms.locfileid: "42993529"
---
# <a name="querying-data"></a>查询数据

Entity Framework Core 使用语言集成查询 (LINQ) 来查询数据库中的数据。 通过 LINQ 可使用 C#（或你选择的其他 .NET 语言）基于派生上下文和实体类编写强类型查询。 LINQ 查询的一种表示形式会传递给数据库提供程序，进而转换为特定于数据库的查询语言（例如，适用于关系数据库的 SQL）。 有关如何处理查询的更多详细信息，请参阅[查询的工作原理](overview.md)。
