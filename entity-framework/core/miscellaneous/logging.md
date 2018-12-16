---
title: 日志记录-EF Core
author: rowanmiller
ms.date: 10/27/2016
ms.assetid: f6e35c6d-45b7-4258-be1d-87c1bb67438d
uid: core/miscellaneous/logging
ms.openlocfilehash: 65501b5ac03ae544c51b7fc1a07fa9eea849f1e3
ms.sourcegitcommit: 5e11125c9b838ce356d673ef5504aec477321724
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50022140"
---
# <a name="logging"></a>日志记录

> [!TIP]  
> 可在 GitHub 上查看此文章的[示例](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Miscellaneous/Logging)。

## <a name="aspnet-core-applications"></a>ASP.NET Core 应用程序

只要使用 `AddDbContext` 或 `AddDbContextPool`，EF Core 就会自动与 ASP.NET Core 的日志记录机制集成。 因此，在使用 ASP.NET Core 时，应按 [ASP.NET Core 文档](https://docs.microsoft.com/aspnet/core/fundamentals/logging?tabs=aspnetcore2x)中的说明配置日志记录。

## <a name="other-applications"></a>其他应用程序

EF Core 日志记录当前需要一个本身配置有一个或多个 ILoggerProvider 的 ILoggerFactory。 以下包中随附有常见提供程序：

* [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/)： 简单的控制台记录器。
* [Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices/)： 支持 Azure 应用服务诊断日志和日志流功能。
* [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug/)： 使用 System.Diagnostics.Debug.WriteLine() 调试器监视的日志。
* [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog/)： 记录到 Windows 事件日志。
* [Microsoft.Extensions.Logging.EventSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventSource/)： 支持 EventSource/EventListener。
* [Microsoft.Extensions.Logging.TraceSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.TraceSource/)： 使用 System.Diagnostics.TraceSource.TraceEvent() 的跟踪侦听器的日志。

安装相应的程序包之后, 该应用程序应创建 LoggerFactory 的单一实例/全局实例。 例如，使用控制台记录器：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Logging/Logging/BloggingContext.cs#DefineLoggerFactory)]

然后，应该在 `DbContextOptionsBuilder` 上向 EF Core 注册此单一实例/全局实例。 例如：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Logging/Logging/BloggingContext.cs#RegisterLoggerFactory)]

> [!WARNING]
> 应用程序不应为每个上下文实例创建新的 ILoggerFactory 实例，这一点非常重要。 这样做会导致内存泄漏和性能下降。

## <a name="filtering-what-is-logged"></a>筛选记录的内容

筛选所记录内容的最简单方法是在注册 ILoggerProvider 时对其进行配置。 例如：

[!code-csharp[Main](../../../samples/core/Miscellaneous/Logging/Logging/BloggingContextWithFiltering.cs#DefineLoggerFactory)]

在此示例中，筛选日志以仅返回以下位置的消息：
 * 在 Microsoft.EntityFrameworkCore.Database.Command 类别中
 * 在信息级别

对于 EF Core 记录器类别在中定义`DbLoggerCategory`类，以便可以方便地查找类别，但这些解析为简单的字符串。

有关基础日志记录基础结构的更多详细信息，请参阅 [ASP.NET Core 日志记录文档](https://docs.microsoft.com/aspnet/core/fundamentals/logging?tabs=aspnetcore2x)。
