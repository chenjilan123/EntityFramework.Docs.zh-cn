---
title: 支持的 .NET 实现 - EF Core
author: rowanmiller
ms.date: 08/30/2017
uid: core/platforms/index
ms.openlocfilehash: 8fc25f4a35794162c92fd292990c24e977d1bf1b
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022250"
---
# <a name="net-implementations-supported-by-ef-core"></a>EF Core 支持的 .NET 实现

我们希望 EF Core 可用于任何可编写 .NET 代码 的平台，我们仍在为实现这一目标而努力。 尽管自动测试证明 EF Core 支持 .NET Core 和 .NET Framework ，同时许多应用程序已成功使用 EF Core，但 Mono、Xamarin 和 UWP 仍存在一些问题。

## <a name="overview"></a>概述

下表提供了每个 .NET 实现的指南：

| .NET 实现                                                                                                  | 状态                                                             | EF Core 1.x 要求                                                                                | EF Core 2.x 要求 <sup>(1)</sup>                                                                 |
|:---------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------|
| **.NET Core**（[ASP.NET Core](../get-started/aspnetcore/index.md)、[控制台](../get-started/netcore/index.md) 等） | 完全支持且推荐使用                                    | [.NET Core SDK 1.x](https://www.microsoft.com/net/core/)                                                | [.NET Core SDK 2.x](https://www.microsoft.com/net/core/)                                                |
| .NET Framework（WinForms、WPF、ASP.NET、[控制台](../get-started/full-dotnet/index.md)等）                    | 完全支持且推荐使用。 EF6 也适用 <sup>(2)</sup> | .NET Framework 4.5.1                                                                                    | .NET Framework 4.6.1                                                                                    |
| **Mono 和 Xamarin**                                                                                                   | 正在进行 <sup>(3)</sup>                                         | Mono 4.6 <br/> Xamarin.iOS 10 <br/> Xamarin.Mac 3 <br/> Xamarin.Android 7                               | Mono 5.4 <br/> Xamarin.iOS 10.14 <br/> Xamarin.Mac 3.8 <br/> Xamarin.Android 7.5                        |
| [通用 Windows 平台](../get-started/uwp/index.md)                                                        | 推荐 EF Core 2.0.1 <sup>(4)</sup>                           | [.NET Core UWP 5.x 包](https://www.nuget.org/packages/Microsoft.NETCore.UniversalWindowsPlatform/) | [.NET Core UWP 6.x 包](https://www.nuget.org/packages/Microsoft.NETCore.UniversalWindowsPlatform/) |

<sup>(1)</sup> EF Core 2.0 面向 .NET 实现，因此需要支持 [.NET Standard 2.0](https://docs.microsoft.com/dotnet/standard/net-standard) 的 .NET 实现。

<sup>(2)</sup> 请参阅[比较 EF Core 和 EF6](../../efcore-and-ef6/index.md) 以选择合适的技术。

<sup>(3)</sup> Xamarin 存在一些问题和已知限制，这些问题和限制可能会阻止部分使用 EF Core 2.0 开发的应用程序正常运行。 查看[未解决问题](https://github.com/aspnet/entityframeworkCore/issues?q=is%3Aopen+is%3Aissue+label%3Aarea-xamarin)列表，了解解决方法。

<sup>(4)</sup> 请参阅本文的[通用 Windows 平台](#universal-windows-platform)部分。

## <a name="universal-windows-platform"></a>通用 Windows 平台

早期的 EF Core 和 .NET UWP 版本存在许多兼容性问题，尤其是用于使用 .NET Native 工具链编译的应用程序时。 新的 .NET UWP 版本增加了对 .NET Standard 2.0 的支持，且包含了 .NET Native 2.0，修复了之前报告的大多数兼容性问题。 我们使用 UWP 对 EF Core 2.0.1 进行了更彻底的测试，但测试不是自动执行的。

在 UWP 上使用 EF Core 时：

* 若要优化查询性能，请避免在 LINQ 查询中使用匿名类型。 将 UWP 应用程序部署到应用商店要求使用 .NET Native 编译应用程序。 使用匿名类型的查询在 .NET Native 上性能较差。

* 若要优化 `SaveChanges()` 性能，请使用 [ChangeTrackingStrategy.ChangingAndChangedNotifications](/dotnet/api/microsoft.entityframeworkcore.changetrackingstrategy)，并在你的实体类型中实施 [INotifyPropertyChanged](https://msdn.microsoft.com/library/system.componentmodel.inotifypropertychanged.aspx)、[INotifyPropertyChanging](https://msdn.microsoft.com/library/system.componentmodel.inotifypropertychanging.aspx) 和 [INotifyCollectionChanged](https://msdn.microsoft.com/library/system.collections.specialized.inotifycollectionchanged.aspx)。

## <a name="report-issues"></a>报告问题

对于未按预期工作的任意组合，我们建议在 [EF Core 问题跟踪程序](https://github.com/aspnet/entityframeworkcore/issues/new)中创建新问题。 对于特定于 Xamarin 的问题，请使用 [Xamarin.Android](https://github.com/xamarin/xamarin-android/issues/new) 或 [Xamarin.iOS](https://github.com/xamarin/xamarin-macios/issues/new) 问题跟踪程序。
