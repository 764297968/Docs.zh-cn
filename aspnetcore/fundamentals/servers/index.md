---
title: ASP.NET Core 中的 Web 服务器实现
author: guardrex
description: 发现适用于 ASP.NET Core 的 Web 服务器 Kestrel 和 HTTP.sys。 了解如何选择服务器以及何时使用反向代理服务器。
ms.author: tdykstra
ms.custom: mvc
ms.date: 09/21/2018
uid: fundamentals/servers/index
ms.openlocfilehash: 06d4bf09b07fc70a10b3e260e78c29fe189486c5
ms.sourcegitcommit: edb9d2d78c9a4d68b397e74ae2aff088b325a143
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/09/2018
ms.locfileid: "51505721"
---
# <a name="web-server-implementations-in-aspnet-core"></a>ASP.NET Core 中的 Web 服务器实现

作者：[Tom Dykstra](https://github.com/tdykstra)、 [Steve Smith](https://ardalis.com/)、[Stephen Halter](https://twitter.com/halter73) 和 [Chris Ross](https://github.com/Tratcher)

ASP.NET Core 应用与进程内 HTTP 服务器实现一起运行。 服务器实现侦听 HTTP 请求，并将它们作为由 <xref:Microsoft.AspNetCore.Http.HttpContext> 组成的[请求功能](xref:fundamentals/request-features)的集合呈现给应用。

ASP.NET Core 交付以下服务器实现：

::: moniker range=">= aspnetcore-2.2"

* [Kestrel](xref:fundamentals/servers/kestrel) 是适用于 ASP.NET Core 的默认跨平台 HTTP 服务器。
* `IISHttpServer` 与 Windows 上的[进程内托管模型](xref:fundamentals/servers/aspnet-core-module#in-process-hosting-model)和 [ASP.NET Core 模块](xref:fundamentals/servers/aspnet-core-module)一起使用。
* [HTTP.sys](xref:fundamentals/servers/httpsys) 是仅适用于 Windows 的 HTTP 服务器，它基于 [ 核心驱动程序和 HTTP 服务器 API](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx)。 （HTTP.sys 在 ASP.NET 1.x 中被命名为 [WebListener](xref:fundamentals/servers/weblistener)。）

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* [Kestrel](xref:fundamentals/servers/kestrel) 是适用于 ASP.NET Core 的默认跨平台 HTTP 服务器。
* [HTTP.sys](xref:fundamentals/servers/httpsys) 是仅适用于 Windows 的 HTTP 服务器，它基于 [ 核心驱动程序和 HTTP 服务器 API](https://msdn.microsoft.com/library/windows/desktop/aa364510.aspx)。 （HTTP.sys 在 ASP.NET 1.x 中被命名为 [WebListener](xref:fundamentals/servers/weblistener)。）

::: moniker-end

## <a name="kestrel"></a>Kestrel

Kestrel 是 ASP.NET Core 项目模板中包括的默认 Web 服务器。

::: moniker range=">= aspnetcore-2.0"

Kestrel 可以单独使用，也可以与反向代理服务器（如 IIS、Nginx 或 Apache）一起使用。 反向代理服务器接收到来自 Internet 的 HTTP 请求，并在进行一些初步处理后将这些请求转发到 Kestrel。

![Kestrel 直接与 Internet 通信，不使用反向代理服务器](kestrel/_static/kestrel-to-internet2.png)

![Kestrel 通过反向代理服务器（如 IIS、Nginx 或 Apache）间接与 Internet 进行通信](kestrel/_static/kestrel-to-internet.png)

使用或不使用反向代理服务器进行配置对 ASP.NET Core 2.0 或更高版本的应用来说都是有效且受支持的托管配置。 有关详细信息，请参阅[何时结合使用 Kestrel 和反向代理](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

如果应用仅接受来自内部网络的请求，则可单独使用 Kestrel。

![Kestrel 直接与内部网络进行通信](kestrel/_static/kestrel-to-internal.png)

如果将应用公开到 Internet，则必须将 IIS、Nginx 或 Apache 用作反向代理服务器。 反向代理服务器接收到来自 Internet 的 HTTP 请求，并在进行一些初步处理后将其转发到 Kestrel，如下图所示：

![Kestrel 通过反向代理服务器（如 IIS、Nginx 或 Apache）间接与 Internet 进行通信](kestrel/_static/kestrel-to-internet.png)

为公共边缘服务器部署（直接公开到 Internet）使用反向代理的最重要的原因是出于安全考虑。 Kestrel 的 1.x 版本不包含防御来自 Internet 的攻击的重要安全功能。 这包括但不限于相应的超时、请求大小限制和并发连接限制。

有关详细信息，请参阅[何时结合使用 Kestrel 和反向代理](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)。

::: moniker-end

在没有 Kestrel 或[自定义服务器实现](#custom-servers)的情况下，不能使用 IIS、Nginx 和 Apache。 ASP.NET Core 设计为在其自己的进程中运行，以实现跨平台统一操作。 IIS、Nginx 和 Apache 规定自己的启动过程和环境。 若要直接使用这些服务器技术，ASP.NET Core 必须满足每个服务器的需求。 使用 Kestrel 等 Web 服务器实现时，ASP.NET Core 可以控制托管在不同服务器技术上的启动过程和环境。

### <a name="iis-with-kestrel"></a>IIS 与 Kestrel

::: moniker range=">= aspnetcore-2.2"

使用 [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) 或 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 时，ASP.NET Core 应用在与 IIS 工作进程（进程内托管模型）相同的进程中运行或者在与 IIS 工作进程分离的进程中运行（进程外托管模型）。

[ASP.NET Core 模块](xref:fundamentals/servers/aspnet-core-module)是一个本机 IIS 模块，用于处理进程内 IIS Http 服务器或进程外 Kestrel 服务器之间的本机 IIS 请求。 有关更多信息，请参见<xref:fundamentals/servers/aspnet-core-module>。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

将 [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) 或 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 用作 ASP.NET Core 的反向代理时，ASP.NET Core 应用在独立于 IIS 工作进程的某个进程中运行。 在 IIS 进程中，[ASP.NET Core 模块](xref:fundamentals/servers/aspnet-core-module)协调反向代理关系。 ASP.NET Core 模块的主要功能是启动 ASP.NET Core 应用，在其出现故障时重启应用，并向应用转发 HTTP 流量。 有关更多信息，请参见<xref:fundamentals/servers/aspnet-core-module>。

::: moniker-end

### <a name="nginx-with-kestrel"></a>Nginx 与 Kestrel

若要了解如何在 Linux 上使用 Nginx 作为 Kestrel 的反向代理服务器，请参阅[在 Linux 上使用 Nginx 进行托管](xref:host-and-deploy/linux-nginx)。

### <a name="apache-with-kestrel"></a>Apache 与 Kestrel

若要了解如何在 Linux 上使用 Apache 作为 Kestrel 的反向代理服务器，请参阅[在 Linux 上使用 Apache 进行托管](xref:host-and-deploy/linux-apache)。

## <a name="httpsys"></a>HTTP.sys

::: moniker range=">= aspnetcore-2.0"

如果 ASP.NET Core 应用在 Windows 上运行，则 HTTP.sys 是 Kestrel 的替代选项。 为了获得最佳性能，通常建议使用 Kestrel。 在应用向 Internet 公开且所需功能受 HTTP.sys（而不是 Kestrel）支持的方案中，可以使用 HTTP.sys。 有关 HTTP.sys 的信息，请参阅 [HTTP.sys](xref:fundamentals/servers/httpsys)。

![HTTP.sys 直接与 Internet 进行通信](httpsys/_static/httpsys-to-internet.png)

对于仅向内部网络公开的应用，HTTP.sys 同样适用。

![HTTP.sys 直接与内部网络进行通信](httpsys/_static/httpsys-to-internal.png)

::: moniker-end

::: moniker range="< aspnetcore-2.0"

HTTP.sys 在 ASP.NET Core 1.x 中被命名为 [WebListener](xref:fundamentals/servers/weblistener)。 对于 IIS 不可用于托管应用的方案，如果 ASP.NET Core 应用在 Windows 上运行，则 WebListener 可用作替代选项。

![Weblistener 直接与 Internet 进行通信](weblistener/_static/weblistener-to-internet.png)

如果所需功能受 WebListener（而不是 Kestrel）支持，则对于仅向内部网络公开的应用而言，还可以使用 WebListener 来替代 Kestrel。 有关 WebListener 的信息，请参阅 [WebListener](xref:fundamentals/servers/weblistener)。

![Weblistener 直接与内部网络进行通信](weblistener/_static/weblistener-to-internal.png)

::: moniker-end

## <a name="aspnet-core-server-infrastructure"></a>ASP.NET Core 服务器基础结构

`Startup.Configure` 方法中提供的 [IApplicationBuilder](/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder) 公开了类型 [IFeatureCollection](/dotnet/api/microsoft.aspnetcore.http.features.ifeaturecollection) 的 [ServerFeatures](/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder.serverfeatures) 属性。 Kestrel 和 HTTP.sys（在 ASP.NET Core 1.x 中为 WebListener）各自仅公开单个功能，即 [IServerAddressesFeature](/dotnet/api/microsoft.aspnetcore.hosting.server.features.iserveraddressesfeature)，但是不同的服务器实现可能公开其他功能。

`IServerAddressesFeature` 可用于查找服务器实现在运行时绑定的端口。

## <a name="custom-servers"></a>自定义服务器

如果内置服务器无法满足应用需求，可以创建一个自定义服务器实现。 [.NET 的开放 Web 接口 (OWIN) 指南](xref:fundamentals/owin) 演示了如何编写基于 [Nowin](https://github.com/Bobris/Nowin) 的 [IServer](/dotnet/api/microsoft.aspnetcore.hosting.server.iserver) 实现。 只有应用使用的功能接口需要实现，但至少必须支持 [IHttpRequestFeature](/dotnet/api/microsoft.aspnetcore.http.features.ihttprequestfeature) 和 [IHttpResponseFeature](/dotnet/api/microsoft.aspnetcore.http.features.ihttpresponsefeature)。

## <a name="server-startup"></a>服务器启动

如果使用 [Visual Studio](https://www.visualstudio.com/vs/)、[Visual Studio for Mac](https://www.visualstudio.com/vs/mac/) 或 [Visual Studio Code](https://code.visualstudio.com/)，则服务器将在集成开发环境 (IDE) 启动应用时启动。 在 Windows 上的 Visual Studio 中，可使用启动配置文件通过 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview)/[ASP.NET Core 模块](xref:fundamentals/servers/aspnet-core-module)或控制台来启动应用和服务器。 在 Visual Studio Code 中，应用和服务器由 [Omnisharp](https://github.com/OmniSharp/omnisharp-vscode) 启动，这会激活 CoreCLR 调试程序。 使用 Visual Studio for Mac 时，应用和服务器由 [Mono Soft-Mode 调试程序](http://www.mono-project.com/docs/advanced/runtime/docs/soft-debugger/)启动。

从项目文件夹中的命令提示符启动应用时，[dotnet run](/dotnet/core/tools/dotnet-run) 会启动该应用和服务器（仅 Kestrel 和 HTTP.sys）。 可通过 `-c|--configuration` 选项指定此配置，该选项设置为 `Debug`（默认值）或 `Release`。 如果启动配置文件位于 launchSettings.json 文件中，请使用 `--launch-profile <NAME>` 选项设置启动配置文件（例如 `Development` 或 `Production`）。 有关详细信息，请参阅 [dotnet run](/dotnet/core/tools/dotnet-run) 和 [.NET Core 分发打包](/dotnet/core/build/distribution-packaging)主题。

## <a name="http2-support"></a>HTTP/2 支持

以下部署方案中的 ASP.NET Core 支持 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

::: moniker range=">= aspnetcore-2.2"

* [Kestrel](xref:fundamentals/servers/kestrel#http2-support)
  * 操作系统
    * Windows Server 2016/Windows 10 或更高版本&dagger;
    * 具有 OpenSSL 1.0.2 或更高版本的 Linux（例如，Ubuntu 16.04 或更高版本）
    * macOS 的未来版本将支持 HTTP/2。
  * 目标框架：.NET Core 2.2 或更高版本
* [HTTP.sys](xref:fundamentals/servers/httpsys#http2-support)
  * Windows Server 2016/Windows 10 或更高版本
  * 目标框架：不适用于 HTTP.sys 部署。
* [IIS（进程内）](xref:host-and-deploy/iis/index#http2-support)
  * Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
  * 目标框架：.NET Core 2.2 或更高版本
* [IIS（进程外）](xref:host-and-deploy/iis/index#http2-support)
  * Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
  * 面向公众的边缘服务器连接使用 HTTP/2，但与 Kestrel 的反向代理连接使用 HTTP/1.1。
  * 目标框架：不适用于 IIS 进程外部署。

&dagger;Kestrel 在 Windows Server 2012 R2 和 Windows 8.1 上对 HTTP/2 的支持有限。 支持受限是因为可在这些操作系统上使用的受支持 TLS 密码套件列表有限。 可能需要使用椭圆曲线数字签名算法 (ECDSA) 生成的证书来保护 TLS 连接。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* [HTTP.sys](xref:fundamentals/servers/httpsys#http2-support)
  * Windows Server 2016/Windows 10 或更高版本
  * 目标框架：不适用于 HTTP.sys 部署。
* [IIS（进程外）](xref:host-and-deploy/iis/index#http2-support)
  * Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
  * 面向公众的边缘服务器连接使用 HTTP/2，但与 Kestrel 的反向代理连接使用 HTTP/1.1。
  * 目标框架：不适用于 IIS 进程外部署。

::: moniker-end

HTTP/2 连接必须使用[应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 和 TLS 1.2 或更高版本。 有关详细信息，请参阅与服务器部署方案相关的主题。

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/servers/kestrel>
* <xref:fundamentals/servers/aspnet-core-module>
* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/azure-apps/index>
* <xref:host-and-deploy/linux-nginx>
* <xref:host-and-deploy/linux-apache>
* <xref:fundamentals/servers/httpsys>（对于 ASP.NET Core 1.x，请参阅 <xref:fundamentals/servers/weblistener>）
