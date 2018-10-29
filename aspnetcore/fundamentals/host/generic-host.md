---
title: .NET 通用主机
author: guardrex
description: 了解有关 .NET 中负责应用启动和生存期管理的通用主机。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/16/2018
uid: fundamentals/host/generic-host
ms.openlocfilehash: 0924e2764958911dc1711d5427f6dd58e8873739
ms.sourcegitcommit: f5d403004f3550e8c46585fdbb16c49e75f495f3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/20/2018
ms.locfileid: "49477600"
---
# <a name="net-generic-host"></a><span data-ttu-id="e9d10-103">.NET 通用主机</span><span class="sxs-lookup"><span data-stu-id="e9d10-103">.NET Generic Host</span></span>

<span data-ttu-id="e9d10-104">作者：[Luke Latham](https://github.com/guardrex)</span><span class="sxs-lookup"><span data-stu-id="e9d10-104">By [Luke Latham](https://github.com/guardrex)</span></span>

<span data-ttu-id="e9d10-105">.NET Core 应用配置和启动“主机”。</span><span class="sxs-lookup"><span data-stu-id="e9d10-105">.NET Core apps configure and launch a *host*.</span></span> <span data-ttu-id="e9d10-106">主机负责应用程序启动和生存期管理。</span><span class="sxs-lookup"><span data-stu-id="e9d10-106">The host is responsible for app startup and lifetime management.</span></span> <span data-ttu-id="e9d10-107">本主题介绍 ASP.NET Core 通用主机 ([HostBuilder](/dotnet/api/microsoft.extensions.hosting.hostbuilder))，该主机对于托管不处理 HTTP 请求的应用非常有用。</span><span class="sxs-lookup"><span data-stu-id="e9d10-107">This topic covers the ASP.NET Core Generic Host ([HostBuilder](/dotnet/api/microsoft.extensions.hosting.hostbuilder)), which is useful for hosting apps that don't process HTTP requests.</span></span> <span data-ttu-id="e9d10-108">有关 Web 主机 ([WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder)) 的介绍，请参阅 <xref:fundamentals/host/web-host>。</span><span class="sxs-lookup"><span data-stu-id="e9d10-108">For coverage of the Web Host ([WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder)), see <xref:fundamentals/host/web-host>.</span></span>

<span data-ttu-id="e9d10-109">通用主机的目标是将 HTTP 管道从 Web 主机 API 中分离出来，从而启用更多的主机方案。</span><span class="sxs-lookup"><span data-stu-id="e9d10-109">The goal of the Generic Host is to decouple the HTTP pipeline from the Web Host API to enable a wider array of host scenarios.</span></span> <span data-ttu-id="e9d10-110">基于通用主机的消息、后台任务和其他非 HTTP 工作负载可从横切功能（如配置、依赖关系注入 [DI] 和日志记录）中受益。</span><span class="sxs-lookup"><span data-stu-id="e9d10-110">Messaging, background tasks, and other non-HTTP workloads based on the Generic Host benefit from cross-cutting capabilities, such as configuration, dependency injection (DI), and logging.</span></span>

<span data-ttu-id="e9d10-111">通用主机是 ASP.NET Core 2.1 中的新增功能，不适用于 Web 承载方案。</span><span class="sxs-lookup"><span data-stu-id="e9d10-111">The Generic Host is new in ASP.NET Core 2.1 and isn't suitable for web hosting scenarios.</span></span> <span data-ttu-id="e9d10-112">对于 Web 承载方案，请使用 [Web 主机](xref:fundamentals/host/web-host)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-112">For web hosting scenarios, use the [Web Host](xref:fundamentals/host/web-host).</span></span> <span data-ttu-id="e9d10-113">通用主机正处于开发阶段，用于在未来版本中替换 Web 主机，并在 HTTP 和非 HTTP 方案中充当主要的主机 API。</span><span class="sxs-lookup"><span data-stu-id="e9d10-113">The Generic Host is under development to replace the Web Host in a future release and act as the primary host API in both HTTP and non-HTTP scenarios.</span></span>

<span data-ttu-id="e9d10-114">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）</span><span class="sxs-lookup"><span data-stu-id="e9d10-114">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

<span data-ttu-id="e9d10-115">在 [Visual Studio Code](https://code.visualstudio.com/) 中运行示例应用时，请使用外部或集成终端。</span><span class="sxs-lookup"><span data-stu-id="e9d10-115">When running the sample app in [Visual Studio Code](https://code.visualstudio.com/), use an *external or integrated terminal*.</span></span> <span data-ttu-id="e9d10-116">请勿在 `internalConsole` 中运行示例。</span><span class="sxs-lookup"><span data-stu-id="e9d10-116">Don't run the sample in an `internalConsole`.</span></span>

<span data-ttu-id="e9d10-117">在 Visual Studio Code 中设置控制台：</span><span class="sxs-lookup"><span data-stu-id="e9d10-117">To set the console in Visual Studio Code:</span></span>

1. <span data-ttu-id="e9d10-118">打开 .vscode/launch.json 文件。</span><span class="sxs-lookup"><span data-stu-id="e9d10-118">Open the *.vscode/launch.json* file.</span></span>
1. <span data-ttu-id="e9d10-119">在 .NET Core 启动（控制台）配置中，找到控制台条目。</span><span class="sxs-lookup"><span data-stu-id="e9d10-119">In the **.NET Core Launch (console)** configuration, locate the **console** entry.</span></span> <span data-ttu-id="e9d10-120">将值设置为 `externalTerminal` 或 `integratedTerminal`。</span><span class="sxs-lookup"><span data-stu-id="e9d10-120">Set the value to either `externalTerminal` or `integratedTerminal`.</span></span>

## <a name="introduction"></a><span data-ttu-id="e9d10-121">介绍</span><span class="sxs-lookup"><span data-stu-id="e9d10-121">Introduction</span></span>

<span data-ttu-id="e9d10-122">通用主机库位于 [Microsoft.Extensions.Hosting 命名空间](/dotnet/api/microsoft.extensions.hosting)中，由 [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting/) 包提供。</span><span class="sxs-lookup"><span data-stu-id="e9d10-122">The Generic Host library is available in the [Microsoft.Extensions.Hosting namespace](/dotnet/api/microsoft.extensions.hosting) and provided by the [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting/) package.</span></span> <span data-ttu-id="e9d10-123">[Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)（ASP.NET Core 2.1 或更高版本）中包括 `Microsoft.Extensions.Hosting` 包。</span><span class="sxs-lookup"><span data-stu-id="e9d10-123">The `Microsoft.Extensions.Hosting` package is included in the [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) (ASP.NET Core 2.1 or later).</span></span>

<span data-ttu-id="e9d10-124">[IHostedService](/dotnet/api/microsoft.extensions.hosting.ihostedservice) 是执行代码的入口点。</span><span class="sxs-lookup"><span data-stu-id="e9d10-124">[IHostedService](/dotnet/api/microsoft.extensions.hosting.ihostedservice) is the entry point to code execution.</span></span> <span data-ttu-id="e9d10-125">每个 `IHostedService` 实现都按照 [ConfigureServices 中服务注册](#configureservices)的顺序执行。</span><span class="sxs-lookup"><span data-stu-id="e9d10-125">Each `IHostedService` implementation is executed in the order of [service registration in ConfigureServices](#configureservices).</span></span> <span data-ttu-id="e9d10-126">主机启动时，每个 `IHostedService` 上都会调用 [StartAsync](/dotnet/api/microsoft.extensions.hosting.ihostedservice.startasync)主机正常关闭时，以反向注册顺序调用 [StopAsync](/dotnet/api/microsoft.extensions.hosting.ihostedservice.stopasync)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-126">[StartAsync](/dotnet/api/microsoft.extensions.hosting.ihostedservice.startasync) is called on each `IHostedService` when the host starts, and [StopAsync](/dotnet/api/microsoft.extensions.hosting.ihostedservice.stopasync) is called in reverse registration order when the host shuts down gracefully.</span></span>

## <a name="set-up-a-host"></a><span data-ttu-id="e9d10-127">设置主机</span><span class="sxs-lookup"><span data-stu-id="e9d10-127">Set up a host</span></span>

<span data-ttu-id="e9d10-128">[IHostBuilder](/dotnet/api/microsoft.extensions.hosting.ihostbuilder) 是供库和应用初始化、生成和运行主机的主要组件：</span><span class="sxs-lookup"><span data-stu-id="e9d10-128">[IHostBuilder](/dotnet/api/microsoft.extensions.hosting.ihostbuilder) is the main component that libraries and apps use to initialize, build, and run the host:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_HostBuilder)]

## <a name="default-services"></a><span data-ttu-id="e9d10-129">默认服务</span><span class="sxs-lookup"><span data-stu-id="e9d10-129">Default services</span></span>

<span data-ttu-id="e9d10-130">在主机初始化期间注册以下服务：</span><span class="sxs-lookup"><span data-stu-id="e9d10-130">The following services are registered during host initialization:</span></span>

* <span data-ttu-id="e9d10-131">[环境](xref:fundamentals/environments) (<xref:Microsoft.Extensions.Hosting.IHostingEnvironment>)</span><span class="sxs-lookup"><span data-stu-id="e9d10-131">[Environment](xref:fundamentals/environments) (<xref:Microsoft.Extensions.Hosting.IHostingEnvironment>)</span></span>
* <xref:Microsoft.Extensions.Hosting.HostBuilderContext>
* <span data-ttu-id="e9d10-132">[配置](xref:fundamentals/configuration/index) (<xref:Microsoft.Extensions.Configuration.IConfiguration>)</span><span class="sxs-lookup"><span data-stu-id="e9d10-132">[Configuration](xref:fundamentals/configuration/index) (<xref:Microsoft.Extensions.Configuration.IConfiguration>)</span></span>
* <span data-ttu-id="e9d10-133"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime> (<xref:Microsoft.Extensions.Hosting.Internal.ApplicationLifetime>)</span><span class="sxs-lookup"><span data-stu-id="e9d10-133"><xref:Microsoft.Extensions.Hosting.IApplicationLifetime> (<xref:Microsoft.Extensions.Hosting.Internal.ApplicationLifetime>)</span></span>
* <span data-ttu-id="e9d10-134"><xref:Microsoft.Extensions.Hosting.IHostLifetime> (<xref:Microsoft.Extensions.Hosting.Internal.ConsoleLifetime>)</span><span class="sxs-lookup"><span data-stu-id="e9d10-134"><xref:Microsoft.Extensions.Hosting.IHostLifetime> (<xref:Microsoft.Extensions.Hosting.Internal.ConsoleLifetime>)</span></span>
* <xref:Microsoft.Extensions.Hosting.IHost>
* <span data-ttu-id="e9d10-135">[选项](xref:fundamentals/configuration/options) (<xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions*>)</span><span class="sxs-lookup"><span data-stu-id="e9d10-135">[Options](xref:fundamentals/configuration/options) (<xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions*>)</span></span>
* <span data-ttu-id="e9d10-136">[日志记录](xref:fundamentals/logging/index) (<xref:Microsoft.Extensions.DependencyInjection.LoggingServiceCollectionExtensions.AddLogging*>)</span><span class="sxs-lookup"><span data-stu-id="e9d10-136">[Logging](xref:fundamentals/logging/index) (<xref:Microsoft.Extensions.DependencyInjection.LoggingServiceCollectionExtensions.AddLogging*>)</span></span>

## <a name="host-configuration"></a><span data-ttu-id="e9d10-137">主机配置</span><span class="sxs-lookup"><span data-stu-id="e9d10-137">Host configuration</span></span>

<span data-ttu-id="e9d10-138">[HostBuilder](/dotnet/api/microsoft.extensions.hosting.hostbuilder) 依赖于以下方法来设置主机配置值：</span><span class="sxs-lookup"><span data-stu-id="e9d10-138">[HostBuilder](/dotnet/api/microsoft.extensions.hosting.hostbuilder) relies on the following approaches to set the host configuration values:</span></span>

* <span data-ttu-id="e9d10-139">配置生成器</span><span class="sxs-lookup"><span data-stu-id="e9d10-139">Configuration builder</span></span>
* <span data-ttu-id="e9d10-140">扩展方法配置</span><span class="sxs-lookup"><span data-stu-id="e9d10-140">Extension method configuration</span></span>

### <a name="configuration-builder"></a><span data-ttu-id="e9d10-141">配置生成器</span><span class="sxs-lookup"><span data-stu-id="e9d10-141">Configuration builder</span></span>

<span data-ttu-id="e9d10-142">通过在 [IHostBuilder](/dotnet/api/microsoft.extensions.hosting.ihostbuilder) 实现上调用 [ConfigureHostConfiguration](/dotnet/api/microsoft.extensions.hosting.ihostbuilder.configurehostconfiguration) 来创建主机生成器配置。</span><span class="sxs-lookup"><span data-stu-id="e9d10-142">Host builder configuration is created by calling [ConfigureHostConfiguration](/dotnet/api/microsoft.extensions.hosting.ihostbuilder.configurehostconfiguration) on the [IHostBuilder](/dotnet/api/microsoft.extensions.hosting.ihostbuilder) implementation.</span></span> <span data-ttu-id="e9d10-143">`ConfigureHostConfiguration` 使用 [IConfigurationBuilder](/dotnet/api/microsoft.extensions.configuration.iconfigurationbuilder) 为主机创建 [IConfiguration](/dotnet/api/microsoft.extensions.configuration.iconfiguration)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-143">`ConfigureHostConfiguration` uses an [IConfigurationBuilder](/dotnet/api/microsoft.extensions.configuration.iconfigurationbuilder) to create an [IConfiguration](/dotnet/api/microsoft.extensions.configuration.iconfiguration) for the host.</span></span> <span data-ttu-id="e9d10-144">配置生成器初始化 [IHostingEnvironment](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment)，以在应用程序的生成过程中使用。</span><span class="sxs-lookup"><span data-stu-id="e9d10-144">The configuration builder initializes the [IHostingEnvironment](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment) for use in the app's build process.</span></span>

<span data-ttu-id="e9d10-145">默认情况下，不添加环境变量配置。</span><span class="sxs-lookup"><span data-stu-id="e9d10-145">Environment variable configuration isn't added by default.</span></span> <span data-ttu-id="e9d10-146">对主机生成器调用 [AddEnvironmentVariables](/dotnet/api/microsoft.extensions.configuration.environmentvariablesextensions.addenvironmentvariables) 可以通过环境变量配置主机。</span><span class="sxs-lookup"><span data-stu-id="e9d10-146">Call [AddEnvironmentVariables](/dotnet/api/microsoft.extensions.configuration.environmentvariablesextensions.addenvironmentvariables) on the host builder to configure the host from environment variables.</span></span> <span data-ttu-id="e9d10-147">`AddEnvironmentVariables` 接受用户定义的前缀（可选）。</span><span class="sxs-lookup"><span data-stu-id="e9d10-147">`AddEnvironmentVariables` accepts an optional user-defined prefix.</span></span> <span data-ttu-id="e9d10-148">示例应用使用前缀 `PREFIX_`。</span><span class="sxs-lookup"><span data-stu-id="e9d10-148">The sample app uses a prefix of `PREFIX_`.</span></span> <span data-ttu-id="e9d10-149">当系统读取环境变量时，便会删除前缀。</span><span class="sxs-lookup"><span data-stu-id="e9d10-149">The prefix is removed when the environment variables are read.</span></span> <span data-ttu-id="e9d10-150">配置示例应用的主机后，`PREFIX_ENVIRONMENT` 的环境变量值就变成 `environment` 密钥的主机配置值。</span><span class="sxs-lookup"><span data-stu-id="e9d10-150">When the sample app's host is configured, the environment variable value for `PREFIX_ENVIRONMENT` becomes the host configuration value for the `environment` key.</span></span>

<span data-ttu-id="e9d10-151">在开发过程中，如果使用 [Visual Studio](https://www.visualstudio.com/) 或通过 `dotnet run` 运行应用，可能会在 Properties/launchSettings.json 文件中设置环境变量。</span><span class="sxs-lookup"><span data-stu-id="e9d10-151">During development when using [Visual Studio](https://www.visualstudio.com/) or running an app with `dotnet run`, environment variables may be set in the *Properties/launchSettings.json* file.</span></span> <span data-ttu-id="e9d10-152">若在开发过程中使用 [Visual Studio Code](https://code.visualstudio.com/)，可能会在 .vscode/launch.json 文件中设置环境变量。</span><span class="sxs-lookup"><span data-stu-id="e9d10-152">In [Visual Studio Code](https://code.visualstudio.com/), environment variables may be set in the *.vscode/launch.json* file during development.</span></span> <span data-ttu-id="e9d10-153">有关更多信息，请参见<xref:fundamentals/environments>。</span><span class="sxs-lookup"><span data-stu-id="e9d10-153">For more information, see <xref:fundamentals/environments>.</span></span>

<span data-ttu-id="e9d10-154">可多次调用 `ConfigureHostConfiguration`，并得到累计结果。</span><span class="sxs-lookup"><span data-stu-id="e9d10-154">`ConfigureHostConfiguration` can be called multiple times with additive results.</span></span> <span data-ttu-id="e9d10-155">主机使用任何一个选项设置上一个值。</span><span class="sxs-lookup"><span data-stu-id="e9d10-155">The host uses whichever option sets a value last.</span></span>

<span data-ttu-id="e9d10-156">hostsettings.json：</span><span class="sxs-lookup"><span data-stu-id="e9d10-156">*hostsettings.json*:</span></span>

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/hostsettings.json)]

<span data-ttu-id="e9d10-157">示例 `HostBuilder` 配置使用 `ConfigureHostConfiguration`：</span><span class="sxs-lookup"><span data-stu-id="e9d10-157">Example `HostBuilder` configuration using `ConfigureHostConfiguration`:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureHostConfiguration)]

> [!NOTE]
> <span data-ttu-id="e9d10-158">[AddConfiguration](/dotnet/api/microsoft.extensions.configuration.chainedbuilderextensions.addconfiguration) 扩展方法当前不能分析由 [GetSection](/dotnet/api/microsoft.extensions.configuration.iconfiguration.getsection) 返回的配置部分（例如 `.AddConfiguration(Configuration.GetSection("section"))`）。</span><span class="sxs-lookup"><span data-stu-id="e9d10-158">The [AddConfiguration](/dotnet/api/microsoft.extensions.configuration.chainedbuilderextensions.addconfiguration) extension method isn't currently capable of parsing a configuration section returned by [GetSection](/dotnet/api/microsoft.extensions.configuration.iconfiguration.getsection) (for example, `.AddConfiguration(Configuration.GetSection("section"))`.</span></span> <span data-ttu-id="e9d10-159">`GetSection` 方法将配置键筛选到所请求的部分，但将节名称保留在键上（例如 `section:environment`）。</span><span class="sxs-lookup"><span data-stu-id="e9d10-159">The `GetSection` method filters the configuration keys to the section requested but leaves the section name on the keys (for example, `section:environment`).</span></span> <span data-ttu-id="e9d10-160">`AddConfiguration` 方法需要键来匹配 `HostBuilder` 键（例如 `environment`）。</span><span class="sxs-lookup"><span data-stu-id="e9d10-160">The `AddConfiguration` method expects the keys to match the `HostBuilder` keys (for example, `environment`).</span></span> <span data-ttu-id="e9d10-161">键上存在的节名称阻止节的值配置主机。</span><span class="sxs-lookup"><span data-stu-id="e9d10-161">The presence of the section name on the keys prevents the section's values from configuring the host.</span></span> <span data-ttu-id="e9d10-162">将在即将发布的版本中解决此问题。</span><span class="sxs-lookup"><span data-stu-id="e9d10-162">This issue will be addressed in an upcoming release.</span></span> <span data-ttu-id="e9d10-163">有关详细信息和解决方法，请参阅[将配置节传入到 WebHostBuilder.UseConfiguration 使用完整的键](https://github.com/aspnet/Hosting/issues/839)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-163">For more information and workarounds, see [Passing configuration section into WebHostBuilder.UseConfiguration uses full keys](https://github.com/aspnet/Hosting/issues/839).</span></span>

### <a name="extension-method-configuration"></a><span data-ttu-id="e9d10-164">扩展方法配置</span><span class="sxs-lookup"><span data-stu-id="e9d10-164">Extension method configuration</span></span>

<span data-ttu-id="e9d10-165">在 `IHostBuilder` 实现上调用扩展方法，用于配置内容根和环境。</span><span class="sxs-lookup"><span data-stu-id="e9d10-165">Extension methods are called on the `IHostBuilder` implementation to configure the content root and the environment.</span></span>

#### <a name="application-key-name"></a><span data-ttu-id="e9d10-166">应用程序键（名称）</span><span class="sxs-lookup"><span data-stu-id="e9d10-166">Application Key (Name)</span></span>

<span data-ttu-id="e9d10-167">[IHostingEnvironment.ApplicationName](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.applicationname) 属性是在主机构造期间通过主机配置设定的。</span><span class="sxs-lookup"><span data-stu-id="e9d10-167">The [IHostingEnvironment.ApplicationName](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment.applicationname) property is set from host configuration during host construction.</span></span> <span data-ttu-id="e9d10-168">要显式设置值，请使用 [HostDefaults.ApplicationKey](/dotnet/api/microsoft.extensions.hosting.hostdefaults.applicationkey)：</span><span class="sxs-lookup"><span data-stu-id="e9d10-168">To set the value explicitly, use the [HostDefaults.ApplicationKey](/dotnet/api/microsoft.extensions.hosting.hostdefaults.applicationkey):</span></span>

<span data-ttu-id="e9d10-169">**密钥**：applicationName</span><span class="sxs-lookup"><span data-stu-id="e9d10-169">**Key**: applicationName</span></span>  
<span data-ttu-id="e9d10-170">**类型**：string</span><span class="sxs-lookup"><span data-stu-id="e9d10-170">**Type**: *string*</span></span>  
<span data-ttu-id="e9d10-171">**默认**：包含应用入口点的程序集的名称。</span><span class="sxs-lookup"><span data-stu-id="e9d10-171">**Default**: The name of the assembly containing the app's entry point.</span></span>  
<span data-ttu-id="e9d10-172">**设置使用**：`HostBuilderContext.HostingEnvironment.ApplicationName`</span><span class="sxs-lookup"><span data-stu-id="e9d10-172">**Set using**: `HostBuilderContext.HostingEnvironment.ApplicationName`</span></span>  
<span data-ttu-id="e9d10-173">**环境变量**：`<PREFIX_>APPLICATIONNAME`（`<PREFIX_>` 是[用户定义的可选前缀](#configuration-builder)）</span><span class="sxs-lookup"><span data-stu-id="e9d10-173">**Environment variable**: `<PREFIX_>APPLICATIONNAME` (`<PREFIX_>` is [optional and user-defined](#configuration-builder))</span></span>

```csharp
var host = new HostBuilder()
    .ConfigureAppConfiguration((hostContext, configApp) =>
    {
        hostContext.HostingEnvironment.ApplicationName = "CustomApplicationName";
    })
```

#### <a name="content-root"></a><span data-ttu-id="e9d10-174">内容根</span><span class="sxs-lookup"><span data-stu-id="e9d10-174">Content Root</span></span>

<span data-ttu-id="e9d10-175">此设置确定主机从哪里开始搜索内容文件。</span><span class="sxs-lookup"><span data-stu-id="e9d10-175">This setting determines where the host begins searching for content files.</span></span>

<span data-ttu-id="e9d10-176">**键**：contentRoot</span><span class="sxs-lookup"><span data-stu-id="e9d10-176">**Key**: contentRoot</span></span>  
<span data-ttu-id="e9d10-177">**类型**：string</span><span class="sxs-lookup"><span data-stu-id="e9d10-177">**Type**: *string*</span></span>  
<span data-ttu-id="e9d10-178">**默认值**：默认为应用程序集所在的文件夹。</span><span class="sxs-lookup"><span data-stu-id="e9d10-178">**Default**: Defaults to the folder where the app assembly resides.</span></span>  
<span data-ttu-id="e9d10-179">**设置使用**：`UseContentRoot`</span><span class="sxs-lookup"><span data-stu-id="e9d10-179">**Set using**: `UseContentRoot`</span></span>  
<span data-ttu-id="e9d10-180">**环境变量**：`<PREFIX_>CONTENTROOT`（`<PREFIX_>` 是[用户定义的可选前缀](#configuration-builder)）</span><span class="sxs-lookup"><span data-stu-id="e9d10-180">**Environment variable**: `<PREFIX_>CONTENTROOT` (`<PREFIX_>` is [optional and user-defined](#configuration-builder))</span></span>

<span data-ttu-id="e9d10-181">如果路径不存在，主机将无法启动。</span><span class="sxs-lookup"><span data-stu-id="e9d10-181">If the path doesn't exist, the host fails to start.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseContentRoot)]

#### <a name="environment"></a><span data-ttu-id="e9d10-182">环境</span><span class="sxs-lookup"><span data-stu-id="e9d10-182">Environment</span></span>

<span data-ttu-id="e9d10-183">设置应用的[环境](xref:fundamentals/environments)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-183">Sets the app's [environment](xref:fundamentals/environments).</span></span>

<span data-ttu-id="e9d10-184">**键**：环境</span><span class="sxs-lookup"><span data-stu-id="e9d10-184">**Key**: environment</span></span>  
<span data-ttu-id="e9d10-185">**类型**：string</span><span class="sxs-lookup"><span data-stu-id="e9d10-185">**Type**: *string*</span></span>  
<span data-ttu-id="e9d10-186">**默认值**：生产</span><span class="sxs-lookup"><span data-stu-id="e9d10-186">**Default**: Production</span></span>  
<span data-ttu-id="e9d10-187">**设置使用**：`UseEnvironment`</span><span class="sxs-lookup"><span data-stu-id="e9d10-187">**Set using**: `UseEnvironment`</span></span>  
<span data-ttu-id="e9d10-188">**环境变量**：`<PREFIX_>ENVIRONMENT`（`<PREFIX_>` 是[用户定义的可选前缀](#configuration-builder)）</span><span class="sxs-lookup"><span data-stu-id="e9d10-188">**Environment variable**: `<PREFIX_>ENVIRONMENT` (`<PREFIX_>` is [optional and user-defined](#configuration-builder))</span></span>

<span data-ttu-id="e9d10-189">环境可以设置为任何值。</span><span class="sxs-lookup"><span data-stu-id="e9d10-189">The environment can be set to any value.</span></span> <span data-ttu-id="e9d10-190">框架定义的值包括 `Development``Staging` 和 `Production`。</span><span class="sxs-lookup"><span data-stu-id="e9d10-190">Framework-defined values include `Development`, `Staging`, and `Production`.</span></span> <span data-ttu-id="e9d10-191">值不区分大小写。</span><span class="sxs-lookup"><span data-stu-id="e9d10-191">Values aren't case sensitive.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseEnvironment)]

## <a name="configureappconfiguration"></a><span data-ttu-id="e9d10-192">ConfigureAppConfiguration</span><span class="sxs-lookup"><span data-stu-id="e9d10-192">ConfigureAppConfiguration</span></span>

<span data-ttu-id="e9d10-193">通过在 [IHostBuilder](/dotnet/api/microsoft.extensions.hosting.ihostbuilder) 实现上调用 [ConfigureAppConfiguration](/dotnet/api/microsoft.extensions.hosting.ihostbuilder.configureappconfiguration) 来创建应用生成器配置。</span><span class="sxs-lookup"><span data-stu-id="e9d10-193">App builder configuration is created by calling [ConfigureAppConfiguration](/dotnet/api/microsoft.extensions.hosting.ihostbuilder.configureappconfiguration) on the [IHostBuilder](/dotnet/api/microsoft.extensions.hosting.ihostbuilder) implementation.</span></span> <span data-ttu-id="e9d10-194">`ConfigureAppConfiguration` 使用 [IConfigurationBuilder](/dotnet/api/microsoft.extensions.configuration.iconfigurationbuilder) 为应用创建 [IConfiguration](/dotnet/api/microsoft.extensions.configuration.iconfiguration)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-194">`ConfigureAppConfiguration` uses an [IConfigurationBuilder](/dotnet/api/microsoft.extensions.configuration.iconfigurationbuilder) to create an [IConfiguration](/dotnet/api/microsoft.extensions.configuration.iconfiguration) for the app.</span></span> <span data-ttu-id="e9d10-195">可多次调用 `ConfigureAppConfiguration`，并得到累计结果。</span><span class="sxs-lookup"><span data-stu-id="e9d10-195">`ConfigureAppConfiguration` can be called multiple times with additive results.</span></span> <span data-ttu-id="e9d10-196">应用会使用最后设置值的选项。</span><span class="sxs-lookup"><span data-stu-id="e9d10-196">The app uses whichever option sets a value last.</span></span> <span data-ttu-id="e9d10-197">用于进行后续操作和 [Services](/dotnet/api/microsoft.extensions.hosting.ihost.services) 中的 [HostBuilderContext.Configuration](/dotnet/api/microsoft.extensions.hosting.hostbuildercontext.configuration) 中可以使用由 `ConfigureAppConfiguration` 创建的配置。</span><span class="sxs-lookup"><span data-stu-id="e9d10-197">The configuration created by `ConfigureAppConfiguration` is available at [HostBuilderContext.Configuration](/dotnet/api/microsoft.extensions.hosting.hostbuildercontext.configuration) for subsequent operations and in [Services](/dotnet/api/microsoft.extensions.hosting.ihost.services).</span></span>

<span data-ttu-id="e9d10-198">示例应用配置使用 `ConfigureAppConfiguration`：</span><span class="sxs-lookup"><span data-stu-id="e9d10-198">Example app configuration using `ConfigureAppConfiguration`:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureAppConfiguration)]

<span data-ttu-id="e9d10-199">appsettings.json：</span><span class="sxs-lookup"><span data-stu-id="e9d10-199">*appsettings.json*:</span></span>

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/appsettings.json)]

<span data-ttu-id="e9d10-200">appsettings.Development.json：</span><span class="sxs-lookup"><span data-stu-id="e9d10-200">*appsettings.Development.json*:</span></span>

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/appsettings.Development.json)]

<span data-ttu-id="e9d10-201">appsettings.Production.json：</span><span class="sxs-lookup"><span data-stu-id="e9d10-201">*appsettings.Production.json*:</span></span>

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/appsettings.Production.json)]

> [!NOTE]
> <span data-ttu-id="e9d10-202">[AddConfiguration](/dotnet/api/microsoft.extensions.configuration.chainedbuilderextensions.addconfiguration) 扩展方法当前不能分析由 [GetSection](/dotnet/api/microsoft.extensions.configuration.iconfiguration.getsection) 返回的配置部分（例如 `.AddConfiguration(Configuration.GetSection("section"))`）。</span><span class="sxs-lookup"><span data-stu-id="e9d10-202">The [AddConfiguration](/dotnet/api/microsoft.extensions.configuration.chainedbuilderextensions.addconfiguration) extension method isn't currently capable of parsing a configuration section returned by [GetSection](/dotnet/api/microsoft.extensions.configuration.iconfiguration.getsection) (for example, `.AddConfiguration(Configuration.GetSection("section"))`.</span></span> <span data-ttu-id="e9d10-203">`GetSection` 方法将配置键筛选到所请求的部分，但将节名称保留在键上（例如 `section:Logging:LogLevel:Default`）。</span><span class="sxs-lookup"><span data-stu-id="e9d10-203">The `GetSection` method filters the configuration keys to the section requested but leaves the section name on the keys (for example, `section:Logging:LogLevel:Default`).</span></span> <span data-ttu-id="e9d10-204">`AddConfiguration` 方法预期得到配置键（例如 `Logging:LogLevel:Default`）的完全匹配项。</span><span class="sxs-lookup"><span data-stu-id="e9d10-204">The `AddConfiguration` method expects an exact match to configuration keys (for example, `Logging:LogLevel:Default`).</span></span> <span data-ttu-id="e9d10-205">键上存在的节名称阻止节的值配置应用。</span><span class="sxs-lookup"><span data-stu-id="e9d10-205">The presence of the section name on the keys prevents the section's values from configuring the app.</span></span> <span data-ttu-id="e9d10-206">将在即将发布的版本中解决此问题。</span><span class="sxs-lookup"><span data-stu-id="e9d10-206">This issue will be addressed in an upcoming release.</span></span> <span data-ttu-id="e9d10-207">有关详细信息和解决方法，请参阅[将配置节传入到 WebHostBuilder.UseConfiguration 使用完整的键](https://github.com/aspnet/Hosting/issues/839)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-207">For more information and workarounds, see [Passing configuration section into WebHostBuilder.UseConfiguration uses full keys](https://github.com/aspnet/Hosting/issues/839).</span></span>

<span data-ttu-id="e9d10-208">要将设置文件移动到输出目录，请在项目文件中将设置文件指定为 [MSBuild 项目项](/visualstudio/msbuild/common-msbuild-project-items)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-208">To move settings files to the output directory, specify the settings files as [MSBuild project items](/visualstudio/msbuild/common-msbuild-project-items) in the project file.</span></span> <span data-ttu-id="e9d10-209">示例应用移动具有以下 **&lt;Content:&gt;** 项的 JSON 应用设置文件和 *hostsettings.json*：</span><span class="sxs-lookup"><span data-stu-id="e9d10-209">The sample app moves its JSON app settings files and *hostsettings.json* with the following **&lt;Content:&gt;** item:</span></span>

```xml
<ItemGroup>
  <Content Include="**\*.json" CopyToOutputDirectory="PreserveNewest" />
</ItemGroup>
```

## <a name="configureservices"></a><span data-ttu-id="e9d10-210">ConfigureServices</span><span class="sxs-lookup"><span data-stu-id="e9d10-210">ConfigureServices</span></span>

<span data-ttu-id="e9d10-211">[ConfigureServices](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.configureservices) 将服务添加到应用的[依赖关系注入](xref:fundamentals/dependency-injection)容器。</span><span class="sxs-lookup"><span data-stu-id="e9d10-211">[ConfigureServices](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.configureservices) adds services to the app's [dependency injection](xref:fundamentals/dependency-injection) container.</span></span> <span data-ttu-id="e9d10-212">可多次调用 `ConfigureServices`，并得到累计结果。</span><span class="sxs-lookup"><span data-stu-id="e9d10-212">`ConfigureServices` can be called multiple times with additive results.</span></span>

<span data-ttu-id="e9d10-213">托管服务是一个类，具有实现 [IHostedService](/dotnet/api/microsoft.extensions.hosting.ihostedservice) 接口的后台任务逻辑。</span><span class="sxs-lookup"><span data-stu-id="e9d10-213">A hosted service is a class with background task logic that implements the [IHostedService](/dotnet/api/microsoft.extensions.hosting.ihostedservice) interface.</span></span> <span data-ttu-id="e9d10-214">有关更多信息，请参见<xref:fundamentals/host/hosted-services>。</span><span class="sxs-lookup"><span data-stu-id="e9d10-214">For more information, see <xref:fundamentals/host/hosted-services>.</span></span>

<span data-ttu-id="e9d10-215">[示例应用](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/)使用 `AddHostedService` 扩展方法向应用添加生存期事件 `LifetimeEventsHostedService` 和定时后台任务 `TimedHostedService` 服务：</span><span class="sxs-lookup"><span data-stu-id="e9d10-215">The [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/) uses the `AddHostedService` extension method to add a service for lifetime events, `LifetimeEventsHostedService`, and a timed background task, `TimedHostedService`, to the app:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureServices)]

## <a name="configurelogging"></a><span data-ttu-id="e9d10-216">ConfigureLogging</span><span class="sxs-lookup"><span data-stu-id="e9d10-216">ConfigureLogging</span></span>

<span data-ttu-id="e9d10-217">[ConfigureLogging](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.configurelogging) 添加一个委托，用于配置提供的 [ILoggingBuilder](/dotnet/api/microsoft.extensions.logging.iloggingbuilder)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-217">[ConfigureLogging](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.configurelogging) adds a delegate for configuring the provided [ILoggingBuilder](/dotnet/api/microsoft.extensions.logging.iloggingbuilder).</span></span> <span data-ttu-id="e9d10-218">可以利用相加结果多次调用 `ConfigureLogging`。</span><span class="sxs-lookup"><span data-stu-id="e9d10-218">`ConfigureLogging` may be called multiple times with additive results.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureLogging)]

### <a name="useconsolelifetime"></a><span data-ttu-id="e9d10-219">UseConsoleLifetime</span><span class="sxs-lookup"><span data-stu-id="e9d10-219">UseConsoleLifetime</span></span>

<span data-ttu-id="e9d10-220">[UseConsoleLifetime](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.useconsolelifetime) 侦听 `Ctrl+C`/SIGINT 或 SIGTERM 并调用 [StopApplication](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.stopapplication) 来启动关闭进程。</span><span class="sxs-lookup"><span data-stu-id="e9d10-220">[UseConsoleLifetime](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.useconsolelifetime) listens for `Ctrl+C`/SIGINT or SIGTERM and calls [StopApplication](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.stopapplication) to start the shutdown process.</span></span> <span data-ttu-id="e9d10-221">`UseConsoleLifetime` 解除阻止 [ RunAsync](#runasync) 和 [WaitForShutdownAsync](#waitforshutdownasync) 等扩展。</span><span class="sxs-lookup"><span data-stu-id="e9d10-221">`UseConsoleLifetime` unblocks extensions such as [RunAsync](#runasync) and [WaitForShutdownAsync](#waitforshutdownasync).</span></span> <span data-ttu-id="e9d10-222">[ConsoleLifetime](/dotnet/api/microsoft.extensions.hosting.internal.consolelifetime) 预注册为默认生存期实现。</span><span class="sxs-lookup"><span data-stu-id="e9d10-222">[ConsoleLifetime](/dotnet/api/microsoft.extensions.hosting.internal.consolelifetime) is pre-registered as the default lifetime implementation.</span></span> <span data-ttu-id="e9d10-223">使用注册的最后一个生存期。</span><span class="sxs-lookup"><span data-stu-id="e9d10-223">The last lifetime registered is used.</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseConsoleLifetime)]

## <a name="container-configuration"></a><span data-ttu-id="e9d10-224">容器配置</span><span class="sxs-lookup"><span data-stu-id="e9d10-224">Container configuration</span></span>

<span data-ttu-id="e9d10-225">为支持插入其他容器中，主机可以接受 [IServiceProviderFactory](/dotnet/api/microsoft.extensions.dependencyinjection.iserviceproviderfactory-1)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-225">To support plugging in other containers, the host can accept an [IServiceProviderFactory](/dotnet/api/microsoft.extensions.dependencyinjection.iserviceproviderfactory-1).</span></span> <span data-ttu-id="e9d10-226">提供工厂不属于 DI 容器注册，而是用于创建具体 DI 容器的主机内部函数。</span><span class="sxs-lookup"><span data-stu-id="e9d10-226">Providing a factory isn't part of the DI container registration but is instead a host intrinsic used to create the concrete DI container.</span></span> <span data-ttu-id="e9d10-227">[UseServiceProviderFactory(IServiceProviderFactory&lt;TContainerBuilder&gt;)](/dotnet/api/microsoft.extensions.hosting.hostbuilder.useserviceproviderfactory) 重写用于创建应用的服务提供程序的默认工厂。</span><span class="sxs-lookup"><span data-stu-id="e9d10-227">[UseServiceProviderFactory(IServiceProviderFactory&lt;TContainerBuilder&gt;)](/dotnet/api/microsoft.extensions.hosting.hostbuilder.useserviceproviderfactory) overrides the default factory used to create the app's service provider.</span></span>

<span data-ttu-id="e9d10-228">[ConfigureContainer](/dotnet/api/microsoft.extensions.hosting.hostbuilder.configurecontainer) 方法托管自定义容器配置。</span><span class="sxs-lookup"><span data-stu-id="e9d10-228">Custom container configuration is managed by the [ConfigureContainer](/dotnet/api/microsoft.extensions.hosting.hostbuilder.configurecontainer) method.</span></span> <span data-ttu-id="e9d10-229">`ConfigureContainer` 提供在基础主机 API 的基础之上配置容器的强类型体验。</span><span class="sxs-lookup"><span data-stu-id="e9d10-229">`ConfigureContainer` provides a strongly-typed experience for configuring the container on top of the underlying host API.</span></span> <span data-ttu-id="e9d10-230">可以利用相加结果多次调用 `ConfigureContainer`。</span><span class="sxs-lookup"><span data-stu-id="e9d10-230">`ConfigureContainer` can be called multiple times with additive results.</span></span>

<span data-ttu-id="e9d10-231">为应用创建服务容器：</span><span class="sxs-lookup"><span data-stu-id="e9d10-231">Create a service container for the app:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainer.cs)]

<span data-ttu-id="e9d10-232">提供服务容器工厂：</span><span class="sxs-lookup"><span data-stu-id="e9d10-232">Provide a service container factory:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainerFactory.cs)]

<span data-ttu-id="e9d10-233">使用该工厂并为应用配置自定义服务容器：</span><span class="sxs-lookup"><span data-stu-id="e9d10-233">Use the factory and configure the custom service container for the app:</span></span>

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ContainerConfiguration)]

## <a name="extensibility"></a><span data-ttu-id="e9d10-234">扩展性</span><span class="sxs-lookup"><span data-stu-id="e9d10-234">Extensibility</span></span>

<span data-ttu-id="e9d10-235">在 `IHostBuilder` 上使用扩展方法实现主机扩展性。</span><span class="sxs-lookup"><span data-stu-id="e9d10-235">Host extensibility is performed with extension methods on `IHostBuilder`.</span></span> <span data-ttu-id="e9d10-236">以下示例介绍扩展方法如何使用 <xref:fundamentals/host/hosted-services> 中所示的 [TimedHostedService](xref:fundamentals/host/hosted-services#timed-background-tasks) 示例来扩展 `IHostBuilder` 实现。</span><span class="sxs-lookup"><span data-stu-id="e9d10-236">The following example shows how an extension method extends an `IHostBuilder` implementation with the [TimedHostedService](xref:fundamentals/host/hosted-services#timed-background-tasks) example demonstrated in <xref:fundamentals/host/hosted-services>.</span></span>

```csharp
var host = new HostBuilder()
    .UseHostedService<TimedHostedService>()
    .Build();

await host.StartAsync();
```

<span data-ttu-id="e9d10-237">应用建立 `UseHostedService` 扩展方法，以注册在 `T` 中传递的托管服务：</span><span class="sxs-lookup"><span data-stu-id="e9d10-237">An app establishes the `UseHostedService` extension method to register the hosted service passed in `T`:</span></span>

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public static class Extensions
{
    public static IHostBuilder UseHostedService<T>(this IHostBuilder hostBuilder)
        where T : class, IHostedService, IDisposable
    {
        return hostBuilder.ConfigureServices(services =>
            services.AddHostedService<T>());
    }
}
```

## <a name="manage-the-host"></a><span data-ttu-id="e9d10-238">管理主机</span><span class="sxs-lookup"><span data-stu-id="e9d10-238">Manage the host</span></span>

<span data-ttu-id="e9d10-239">[IHost](/dotnet/api/microsoft.extensions.hosting.ihost) 实现负责启动和停止服务容器中注册的 `IHostedService` 实现。</span><span class="sxs-lookup"><span data-stu-id="e9d10-239">The [IHost](/dotnet/api/microsoft.extensions.hosting.ihost) implementation is responsible for starting and stopping the `IHostedService` implementations that are registered in the service container.</span></span>

### <a name="run"></a><span data-ttu-id="e9d10-240">运行</span><span class="sxs-lookup"><span data-stu-id="e9d10-240">Run</span></span>

<span data-ttu-id="e9d10-241">[Run](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.run) 运行应用并阻止调用线程，直到关闭主机：</span><span class="sxs-lookup"><span data-stu-id="e9d10-241">[Run](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.run) runs the app and blocks the calling thread until the host is shut down:</span></span>

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        host.Run();
    }
}
```

### <a name="runasync"></a><span data-ttu-id="e9d10-242">RunAsync</span><span class="sxs-lookup"><span data-stu-id="e9d10-242">RunAsync</span></span>

<span data-ttu-id="e9d10-243">[RunAsync](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.runasync) 运行应用并返回在触发取消令牌或关闭时完成的 `Task`：</span><span class="sxs-lookup"><span data-stu-id="e9d10-243">[RunAsync](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.runasync) runs the app and returns a `Task` that completes when the cancellation token or shutdown is triggered:</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        await host.RunAsync();
    }
}
```

### <a name="runconsoleasync"></a><span data-ttu-id="e9d10-244">RunConsoleAsync</span><span class="sxs-lookup"><span data-stu-id="e9d10-244">RunConsoleAsync</span></span>

<span data-ttu-id="e9d10-245">[RunConsoleAsync](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.runconsoleasync) 启用控制台、生成和启动主机，以及等待 `Ctrl+C`/SIGINT 或 SIGTERM 关闭。</span><span class="sxs-lookup"><span data-stu-id="e9d10-245">[RunConsoleAsync](/dotnet/api/microsoft.extensions.hosting.hostinghostbuilderextensions.runconsoleasync) enables console support, builds and starts the host, and waits for `Ctrl+C`/SIGINT or SIGTERM to shut down.</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var hostBuilder = new HostBuilder();

        await hostBuilder.RunConsoleAsync();
    }
}
```

### <a name="start-and-stopasync"></a><span data-ttu-id="e9d10-246">Start 和 StopAsync</span><span class="sxs-lookup"><span data-stu-id="e9d10-246">Start and StopAsync</span></span>

<span data-ttu-id="e9d10-247">[Start](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.start) 同步启动主机。</span><span class="sxs-lookup"><span data-stu-id="e9d10-247">[Start](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.start) starts the host synchronously.</span></span>

<span data-ttu-id="e9d10-248">[StopAsync(TimeSpan)](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.stopasync) 尝试在提供的超时时间内停止主机。</span><span class="sxs-lookup"><span data-stu-id="e9d10-248">[StopAsync(TimeSpan)](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.stopasync) attempts to stop the host within the provided timeout.</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            await host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

### <a name="startasync-and-stopasync"></a><span data-ttu-id="e9d10-249">StartAsync 和 StopAsync</span><span class="sxs-lookup"><span data-stu-id="e9d10-249">StartAsync and StopAsync</span></span>

<span data-ttu-id="e9d10-250">[StartAsync](/dotnet/api/microsoft.extensions.hosting.ihost.startasync) 启动应用。</span><span class="sxs-lookup"><span data-stu-id="e9d10-250">[StartAsync](/dotnet/api/microsoft.extensions.hosting.ihost.startasync) starts the app.</span></span>

<span data-ttu-id="e9d10-251">[StopAsync](/dotnet/api/microsoft.extensions.hosting.ihost.stopasync) 停止应用。</span><span class="sxs-lookup"><span data-stu-id="e9d10-251">[StopAsync](/dotnet/api/microsoft.extensions.hosting.ihost.stopasync) stops the app.</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.StopAsync();
        }
    }
}
```

### <a name="waitforshutdown"></a><span data-ttu-id="e9d10-252">WaitForShutdown</span><span class="sxs-lookup"><span data-stu-id="e9d10-252">WaitForShutdown</span></span>

<span data-ttu-id="e9d10-253">通过 [IHostLifetime](/dotnet/api/microsoft.extensions.hosting.ihostlifetime)，如 [ConsoleLifetime](/dotnet/api/microsoft.extensions.hosting.internal.consolelifetime)（侦听 `Ctrl+C`/SIGINT 或 SIGTERM）来触发 [WaitForShutdown](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.waitforshutdown)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-253">[WaitForShutdown](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.waitforshutdown) is triggered via the [IHostLifetime](/dotnet/api/microsoft.extensions.hosting.ihostlifetime), such as [ConsoleLifetime](/dotnet/api/microsoft.extensions.hosting.internal.consolelifetime) (listens for `Ctrl+C`/SIGINT or SIGTERM).</span></span> <span data-ttu-id="e9d10-254">`WaitForShutdown` 调用 [StopAsync](/dotnet/api/microsoft.extensions.hosting.ihost.stopasync)。</span><span class="sxs-lookup"><span data-stu-id="e9d10-254">`WaitForShutdown` calls [StopAsync](/dotnet/api/microsoft.extensions.hosting.ihost.stopasync).</span></span>

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            host.WaitForShutdown();
        }
    }
}
```

### <a name="waitforshutdownasync"></a><span data-ttu-id="e9d10-255">WaitForShutdownAsync</span><span class="sxs-lookup"><span data-stu-id="e9d10-255">WaitForShutdownAsync</span></span>

<span data-ttu-id="e9d10-256">[WaitForShutdownAsync](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.waitforshutdownasync) 返回在通过给定的定牌和调用 [StopAsync](/dotnet/api/microsoft.extensions.hosting.ihost.stopasync) 来触发关闭时完成的 `Task`。</span><span class="sxs-lookup"><span data-stu-id="e9d10-256">[WaitForShutdownAsync](/dotnet/api/microsoft.extensions.hosting.hostingabstractionshostextensions.waitforshutdownasync) returns a `Task` that completes when shutdown is triggered via the given token and calls [StopAsync](/dotnet/api/microsoft.extensions.hosting.ihost.stopasync).</span></span>

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.WaitForShutdownAsync();
        }

    }
}
```

### <a name="external-control"></a><span data-ttu-id="e9d10-257">外部控件</span><span class="sxs-lookup"><span data-stu-id="e9d10-257">External control</span></span>

<span data-ttu-id="e9d10-258">使用可从外部调用的方法，能够实现主机的外部控件：</span><span class="sxs-lookup"><span data-stu-id="e9d10-258">External control of the host can be achieved using methods that can be called externally:</span></span>

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

<span data-ttu-id="e9d10-259">在 [StartAsync](/dotnet/api/microsoft.extensions.hosting.ihost.startasync) 开始时调用 [IHostLifetime.WaitForStartAsync](/dotnet/api/microsoft.extensions.hosting.ihostlifetime.waitforstartasync)，在继续之前，会一直等待该操作完成。</span><span class="sxs-lookup"><span data-stu-id="e9d10-259">[IHostLifetime.WaitForStartAsync](/dotnet/api/microsoft.extensions.hosting.ihostlifetime.waitforstartasync) is called at the start of [StartAsync](/dotnet/api/microsoft.extensions.hosting.ihost.startasync), which waits until it's complete before continuing.</span></span> <span data-ttu-id="e9d10-260">它可用于延迟启动，直到外部事件发出信号。</span><span class="sxs-lookup"><span data-stu-id="e9d10-260">This can be used to delay startup until signaled by an external event.</span></span>

## <a name="ihostingenvironment-interface"></a><span data-ttu-id="e9d10-261">IHostingEnvironment 接口</span><span class="sxs-lookup"><span data-stu-id="e9d10-261">IHostingEnvironment interface</span></span>

<span data-ttu-id="e9d10-262">[IHostingEnvironment](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment) 提供有关应用托管环境的信息。</span><span class="sxs-lookup"><span data-stu-id="e9d10-262">[IHostingEnvironment](/dotnet/api/microsoft.extensions.hosting.ihostingenvironment) provides information about the app's hosting environment.</span></span> <span data-ttu-id="e9d10-263">使用[构造函数注入](xref:fundamentals/dependency-injection)获取 `IHostingEnvironment` 以使用其属性和扩展方法：</span><span class="sxs-lookup"><span data-stu-id="e9d10-263">Use [constructor injection](xref:fundamentals/dependency-injection) to obtain the `IHostingEnvironment` in order to use its properties and extension methods:</span></span>

```csharp
public class MyClass
{
    private readonly IHostingEnvironment _env;

    public MyClass(IHostingEnvironment env)
    {
        _env = env;
    }

    public void DoSomething()
    {
        var environmentName = _env.EnvironmentName;
    }
}
```

<span data-ttu-id="e9d10-264">有关更多信息，请参见<xref:fundamentals/environments>。</span><span class="sxs-lookup"><span data-stu-id="e9d10-264">For more information, see <xref:fundamentals/environments>.</span></span>

## <a name="iapplicationlifetime-interface"></a><span data-ttu-id="e9d10-265">IApplicationLifetime 接口</span><span class="sxs-lookup"><span data-stu-id="e9d10-265">IApplicationLifetime interface</span></span>

<span data-ttu-id="e9d10-266">[IApplicationLifetime](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime) 允许启动后和关闭活动，包括正常关闭请求。</span><span class="sxs-lookup"><span data-stu-id="e9d10-266">[IApplicationLifetime](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime) allows for post-startup and shutdown activities, including graceful shutdown requests.</span></span> <span data-ttu-id="e9d10-267">接口上的三个属性是用于注册 `Action` 方法（用于定义启动和关闭事件）的取消标记。</span><span class="sxs-lookup"><span data-stu-id="e9d10-267">Three properties on the interface are cancellation tokens used to register `Action` methods that define startup and shutdown events.</span></span>

| <span data-ttu-id="e9d10-268">取消标记</span><span class="sxs-lookup"><span data-stu-id="e9d10-268">Cancellation Token</span></span> | <span data-ttu-id="e9d10-269">触发条件</span><span class="sxs-lookup"><span data-stu-id="e9d10-269">Triggered when&#8230;</span></span> |
| ------------------ | --------------------- |
| [<span data-ttu-id="e9d10-270">ApplicationStarted</span><span class="sxs-lookup"><span data-stu-id="e9d10-270">ApplicationStarted</span></span>](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstarted) | <span data-ttu-id="e9d10-271">主机已完全启动。</span><span class="sxs-lookup"><span data-stu-id="e9d10-271">The host has fully started.</span></span> |
| [<span data-ttu-id="e9d10-272">ApplicationStopped</span><span class="sxs-lookup"><span data-stu-id="e9d10-272">ApplicationStopped</span></span>](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopped) | <span data-ttu-id="e9d10-273">主机正在完成正常关闭。</span><span class="sxs-lookup"><span data-stu-id="e9d10-273">The host is completing a graceful shutdown.</span></span> <span data-ttu-id="e9d10-274">应处理所有请求。</span><span class="sxs-lookup"><span data-stu-id="e9d10-274">All requests should be processed.</span></span> <span data-ttu-id="e9d10-275">关闭受到阻止，直到完成此事件。</span><span class="sxs-lookup"><span data-stu-id="e9d10-275">Shutdown blocks until this event completes.</span></span> |
| [<span data-ttu-id="e9d10-276">ApplicationStopping</span><span class="sxs-lookup"><span data-stu-id="e9d10-276">ApplicationStopping</span></span>](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.applicationstopping) | <span data-ttu-id="e9d10-277">主机正在执行正常关闭。</span><span class="sxs-lookup"><span data-stu-id="e9d10-277">The host is performing a graceful shutdown.</span></span> <span data-ttu-id="e9d10-278">仍在处理请求。</span><span class="sxs-lookup"><span data-stu-id="e9d10-278">Requests may still be processing.</span></span> <span data-ttu-id="e9d10-279">关闭受到阻止，直到完成此事件。</span><span class="sxs-lookup"><span data-stu-id="e9d10-279">Shutdown blocks until this event completes.</span></span> |

<span data-ttu-id="e9d10-280">构造函数将 `IApplicationLifetime` 服务注入到任何类中。</span><span class="sxs-lookup"><span data-stu-id="e9d10-280">Constructor-inject the `IApplicationLifetime` service into any class.</span></span> <span data-ttu-id="e9d10-281">[示例应用](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/)将构造函数注入到 `LifetimeEventsHostedService` 类（一个 `IHostedService` 实现）中，用于注册事件。</span><span class="sxs-lookup"><span data-stu-id="e9d10-281">The [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/) uses constructor injection into a `LifetimeEventsHostedService` class (an `IHostedService` implementation) to register the events.</span></span>

<span data-ttu-id="e9d10-282">LifetimeEventsHostedService.cs：</span><span class="sxs-lookup"><span data-stu-id="e9d10-282">*LifetimeEventsHostedService.cs*:</span></span>

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/LifetimeEventsHostedService.cs?name=snippet1)]

<span data-ttu-id="e9d10-283">[StopApplication](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.stopapplication) 请求应用终止。</span><span class="sxs-lookup"><span data-stu-id="e9d10-283">[StopApplication](/dotnet/api/microsoft.extensions.hosting.iapplicationlifetime.stopapplication) requests termination of the app.</span></span> <span data-ttu-id="e9d10-284">以下类在调用类的 `Shutdown` 方法时使用 `StopApplication` 正常关闭应用：</span><span class="sxs-lookup"><span data-stu-id="e9d10-284">The following class uses `StopApplication` to gracefully shut down an app when the class's `Shutdown` method is called:</span></span>

```csharp
public class MyClass
{
    private readonly IApplicationLifetime _appLifetime;

    public MyClass(IApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public void Shutdown()
    {
        _appLifetime.StopApplication();
    }
}
```

## <a name="additional-resources"></a><span data-ttu-id="e9d10-285">其他资源</span><span class="sxs-lookup"><span data-stu-id="e9d10-285">Additional resources</span></span>

* <xref:fundamentals/host/hosted-services>
* [<span data-ttu-id="e9d10-286">GitHub 上托管的存储库示例</span><span class="sxs-lookup"><span data-stu-id="e9d10-286">Hosting repo samples on GitHub</span></span>](https://github.com/aspnet/Hosting/tree/release/2.1/samples)
