---
title: 托管和部署 ASP.NET Core
author: rick-anderson
description: 了解如何设置托管环境和部署 ASP.NET Core 应用。
ms.author: riande
ms.custom: mvc
ms.date: 11/26/2018
uid: host-and-deploy/index
ms.openlocfilehash: f70b05df6bf710e2ab54a1eaafb71b4f9b306cbe
ms.sourcegitcommit: e9b99854b0a8021dafabee0db5e1338067f250a9
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/28/2018
ms.locfileid: "52450614"
---
# <a name="host-and-deploy-aspnet-core"></a>托管和部署 ASP.NET Core

一般而言，向托管环境部署 ASP.NET Core 应用需执行以下操作：

* 将应用发布到托管服务器上的文件夹。
* 设置进程管理器，该管理器在收到请求时启动应用，并在应用发生故障或服务器重启后重新启动应用。
* 如果需要配置反向代理，设置将请求转发到应用的反向代理。

## <a name="publish-to-a-folder"></a>发布到文件夹

[dotnet 发布](/dotnet/articles/core/tools/dotnet-publish) CLI 命令编译应用代码并复制所需的文件，以便在“发布”文件夹中运行应用。 从 Visual Studio 进行部署时，在文件被复制到部署目标前，系统会自动执行 [dotnet publish](/dotnet/core/tools/dotnet-publish) 步骤。

### <a name="folder-contents"></a>文件夹内容

“发布”文件夹包含应用的“.exe”和“.dll”文件、应用依赖项以及 .NET 运行时（根据需要）。

.NET Core 应用可以作为“独立”应用或“依赖于框架”的应用进行发布。 如果应用是独立应用，则包含 .NET 运行时的“.dll”文件会包括在“发布”文件夹内。 如果应用依赖于框架，.NET 运行时文件就不包含在内，因为应用包含对服务器上安装的 .NET 版本的引用。 默认部署模型是依赖于框架的模型。 有关详细信息，请参阅 [.NET Core 应用程序部署](/dotnet/articles/core/deploying/index)。

除了“.exe”和“.dll”文件以外，ASP.NET Core 应用的“发布”文件夹通常包含配置文件、静态资产和 MVC 视图。 有关更多信息，请参见<xref:host-and-deploy/directory-structure>。

## <a name="set-up-a-process-manager"></a>设置进程管理器

ASP.NET Core 应用是一个控制台应用，在服务器启动时必须启动该应用，并且在出现故障后必须重新启动它。 若要自动启动和重新启动，需要使用进程管理器。 用于 ASP.NET Core 的最常见进程管理器是：

* Linux
  * [Nginx](xref:host-and-deploy/linux-nginx)
  * [Apache](xref:host-and-deploy/linux-apache)
* Windows
  * [IIS](xref:host-and-deploy/iis/index)
  * [Windows 服务](xref:host-and-deploy/windows-service)

## <a name="set-up-a-reverse-proxy"></a>设置反向代理

::: moniker range=">= aspnetcore-2.0"

如果该应用使用的是 [Kestrel](xref:fundamentals/servers/kestrel) Web 服务器，则可以将 [Nginx](xref:host-and-deploy/linux-nginx)、[Apache](xref:host-and-deploy/linux-apache) 或 [IIS](xref:host-and-deploy/iis/index) 用作反向代理服务器。 反向代理服务器接收到来自 Internet 的 HTTP 请求，并在进行一些初步处理后将这些请求转发到 Kestrel。

使用或不使用反向代理服务器进行配置对 ASP.NET Core 2.0 或更高版本的应用来说都是有效且受支持的托管配置。 有关详细信息，请参阅[何时结合使用 Kestrel 和反向代理](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

如果该应用使用的是 [Kestrel](xref:fundamentals/servers/kestrel) Web 服务器并将被公开到 Internet，则需将 [Nginx](xref:host-and-deploy/linux-nginx)、[Apache](xref:host-and-deploy/linux-apache) 或 [IIS](xref:host-and-deploy/iis/index) 用作反向代理服务器。 反向代理服务器接收到来自 Internet 的 HTTP 请求，并在进行一些初步处理后将这些请求转发到 Kestrel。 使用反向代理的主要原因是出于安全考虑。 有关详细信息，请参阅[何时结合使用 Kestrel 和反向代理](xref:fundamentals/servers/kestrel?tabs=aspnetcore1x#when-to-use-kestrel-with-a-reverse-proxy)。

::: moniker-end

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

对于托管在代理服务器和负载均衡器后方的应用，可能需要附加配置。 如不提供附加配置，应用可能无法访问方案 (HTTP/HTTPS) 和发出请求的远程 IP 地址。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

## <a name="use-visual-studio-and-msbuild-to-automate-deployments"></a>使用 Visual Studio 和 MSBuild 自动执行部署

除将输出从 [dotnet publish](/dotnet/core/tools/dotnet-publish) 复制到服务器外，部署通常还需要其他任务。 例如，可能需要从“发布”文件夹获取或排除额外文件。 Visual Studio 使用 MSBuild 进行 Web 部署，用户可以自定义 MSBuild 以在部署期间执行其他任务。 有关详细信息，请参阅 <xref:host-and-deploy/visual-studio-publish-profiles> 和[使用 MSBuild 和 Team Foundation Build](http://msbuildbook.com/) 一书。

通过[发布 Web 功能](xref:tutorials/publish-to-azure-webapp-using-vs)或[内置 Git 支持](xref:host-and-deploy/azure-apps/azure-continuous-deployment)，可以将应用从 Visual Studio 直接部署到 Azure 应用服务。 Azure DevOps Services 支持[对 Azure 应用服务进行持续部署](/azure/devops/pipelines/targets/webapp)。

## <a name="publish-to-azure"></a>发布到 Azure

有关如何使用 Visual Studio 将应用发布到 Azure 的说明，请参阅 <xref:tutorials/publish-to-azure-webapp-using-vs>。 还可以从[命令行](/azure/app-service/app-service-web-get-started-dotnet)将该应用发布到 Azure。

## <a name="host-in-a-web-farm"></a>在 Web 场中托管

有关如何配置在 Web 场环境中托管 ASP.NET Core 应用（例如，部署应用的多个实例以实现可伸缩性）的信息，请参阅 <xref:host-and-deploy/web-farm>。

## <a name="additional-resources"></a>其他资源

* <xref:host-and-deploy/docker/index>
* <xref:test/troubleshoot>
