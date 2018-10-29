---
title: ASP.NET Core 中的 WebSocket 支持
author: rick-anderson
description: 了解如何在 ASP.NET Core 中开始使用 WebSocket。
monikerRange: '>= aspnetcore-1.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 06/28/2018
uid: fundamentals/websockets
ms.openlocfilehash: b1e2180ed8dc93e2474ecca371d386830b7f3a9f
ms.sourcegitcommit: 6e6002de467cd135a69e5518d4ba9422d693132a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2018
ms.locfileid: "49348450"
---
# <a name="websockets-support-in-aspnet-core"></a><span data-ttu-id="1ca53-103">ASP.NET Core 中的 WebSocket 支持</span><span class="sxs-lookup"><span data-stu-id="1ca53-103">WebSockets support in ASP.NET Core</span></span>

<span data-ttu-id="1ca53-104">作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Andrew Stanton-Nurse](https://github.com/anurse)</span><span class="sxs-lookup"><span data-stu-id="1ca53-104">By [Tom Dykstra](https://github.com/tdykstra) and [Andrew Stanton-Nurse](https://github.com/anurse)</span></span>

<span data-ttu-id="1ca53-105">本文介绍 ASP.NET Core 中 WebSocket 的入门方法。</span><span class="sxs-lookup"><span data-stu-id="1ca53-105">This article explains how to get started with WebSockets in ASP.NET Core.</span></span> <span data-ttu-id="1ca53-106">[WebSocket](https://wikipedia.org/wiki/WebSocket) ([RFC 6455](https://tools.ietf.org/html/rfc6455)) 是一个协议，支持通过 TCP 连接建立持久的双向信道。</span><span class="sxs-lookup"><span data-stu-id="1ca53-106">[WebSocket](https://wikipedia.org/wiki/WebSocket) ([RFC 6455](https://tools.ietf.org/html/rfc6455)) is a protocol that enables two-way persistent communication channels over TCP connections.</span></span> <span data-ttu-id="1ca53-107">它用于从快速实时通信中获益的应用，如聊天、仪表板和游戏应用。</span><span class="sxs-lookup"><span data-stu-id="1ca53-107">It's used in apps that benefit from fast, real-time communication, such as chat, dashboard, and game apps.</span></span>

<span data-ttu-id="1ca53-108">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/websockets/samples)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）。</span><span class="sxs-lookup"><span data-stu-id="1ca53-108">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/websockets/samples) ([how to download](xref:tutorials/index#how-to-download-a-sample)).</span></span> <span data-ttu-id="1ca53-109">有关详细信息，请参阅[后续步骤](#next-steps)部分。</span><span class="sxs-lookup"><span data-stu-id="1ca53-109">See the [Next steps](#next-steps) section for more information.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="1ca53-110">系统必备</span><span class="sxs-lookup"><span data-stu-id="1ca53-110">Prerequisites</span></span>

* <span data-ttu-id="1ca53-111">ASP.NET Core 1.1 或更高版本</span><span class="sxs-lookup"><span data-stu-id="1ca53-111">ASP.NET Core 1.1 or later</span></span>
* <span data-ttu-id="1ca53-112">支持 ASP.NET Core 的任何操作系统：</span><span class="sxs-lookup"><span data-stu-id="1ca53-112">Any OS that supports ASP.NET Core:</span></span>
  
  * <span data-ttu-id="1ca53-113">Windows 7/Windows Server 2008 或更高版本</span><span class="sxs-lookup"><span data-stu-id="1ca53-113">Windows 7 / Windows Server 2008 or later</span></span>
  * <span data-ttu-id="1ca53-114">Linux</span><span class="sxs-lookup"><span data-stu-id="1ca53-114">Linux</span></span>
  * <span data-ttu-id="1ca53-115">macOS</span><span class="sxs-lookup"><span data-stu-id="1ca53-115">macOS</span></span>
  
* <span data-ttu-id="1ca53-116">如果应用在安装了 IIS 的 Windows 上运行：</span><span class="sxs-lookup"><span data-stu-id="1ca53-116">If the app runs on Windows with IIS:</span></span>

  * <span data-ttu-id="1ca53-117">Windows 8 / Windows Server 2012 及更高版本</span><span class="sxs-lookup"><span data-stu-id="1ca53-117">Windows 8 / Windows Server 2012 or later</span></span>
  * <span data-ttu-id="1ca53-118">IIS 8 / IIS 8 Express</span><span class="sxs-lookup"><span data-stu-id="1ca53-118">IIS 8 / IIS 8 Express</span></span>
  * <span data-ttu-id="1ca53-119">必须启用 WebSocket（请参阅 [IIS/IIS Express 支持](#iisiis-express-support)部分。）。</span><span class="sxs-lookup"><span data-stu-id="1ca53-119">WebSockets must be enabled (See the [IIS/IIS Express support](#iisiis-express-support) section.).</span></span>
  
* <span data-ttu-id="1ca53-120">如果应用在 [HTTP.sys](xref:fundamentals/servers/httpsys) 上运行：</span><span class="sxs-lookup"><span data-stu-id="1ca53-120">If the app runs on [HTTP.sys](xref:fundamentals/servers/httpsys):</span></span>

  * <span data-ttu-id="1ca53-121">Windows 8 / Windows Server 2012 及更高版本</span><span class="sxs-lookup"><span data-stu-id="1ca53-121">Windows 8 / Windows Server 2012 or later</span></span>

* <span data-ttu-id="1ca53-122">有关支持的浏览器，请参阅 https://caniuse.com/#feat=websockets。</span><span class="sxs-lookup"><span data-stu-id="1ca53-122">For supported browsers, see https://caniuse.com/#feat=websockets.</span></span>

## <a name="when-to-use-websockets"></a><span data-ttu-id="1ca53-123">何时使用 WebSocket</span><span class="sxs-lookup"><span data-stu-id="1ca53-123">When to use WebSockets</span></span>

<span data-ttu-id="1ca53-124">通过 WebSocket 可直接使用套接字连接。</span><span class="sxs-lookup"><span data-stu-id="1ca53-124">Use WebSockets to work directly with a socket connection.</span></span> <span data-ttu-id="1ca53-125">例如，使用 WebSocket 可以让实时游戏达到最佳性能。</span><span class="sxs-lookup"><span data-stu-id="1ca53-125">For example, use WebSockets for the best possible performance with a real-time game.</span></span>

<span data-ttu-id="1ca53-126">[ASP.NET Core SignalR](xref:signalr/introduction) 是一个库，可用于简化向应用添加实时 Web 功能。</span><span class="sxs-lookup"><span data-stu-id="1ca53-126">[ASP.NET Core SignalR](xref:signalr/introduction) is a library that simplifies adding real-time web functionality to apps.</span></span> <span data-ttu-id="1ca53-127">它会尽可能地使用 WebSocket。</span><span class="sxs-lookup"><span data-stu-id="1ca53-127">It uses WebSockets whenever possible.</span></span>

## <a name="how-to-use-websockets"></a><span data-ttu-id="1ca53-128">如何使用 Websocket</span><span class="sxs-lookup"><span data-stu-id="1ca53-128">How to use WebSockets</span></span>

* <span data-ttu-id="1ca53-129">安装 [Microsoft.AspNetCore.WebSockets](https://www.nuget.org/packages/Microsoft.AspNetCore.WebSockets/) 包。</span><span class="sxs-lookup"><span data-stu-id="1ca53-129">Install the [Microsoft.AspNetCore.WebSockets](https://www.nuget.org/packages/Microsoft.AspNetCore.WebSockets/) package.</span></span>
* <span data-ttu-id="1ca53-130">配置中间件。</span><span class="sxs-lookup"><span data-stu-id="1ca53-130">Configure the middleware.</span></span>
* <span data-ttu-id="1ca53-131">接受 WebSocket 请求。</span><span class="sxs-lookup"><span data-stu-id="1ca53-131">Accept WebSocket requests.</span></span>
* <span data-ttu-id="1ca53-132">发送和接收消息。</span><span class="sxs-lookup"><span data-stu-id="1ca53-132">Send and receive messages.</span></span>

### <a name="configure-the-middleware"></a><span data-ttu-id="1ca53-133">配置中间件</span><span class="sxs-lookup"><span data-stu-id="1ca53-133">Configure the middleware</span></span>

<span data-ttu-id="1ca53-134">在 `Startup` 类的 `Configure` 方法中添加 WebSocket 中间件：</span><span class="sxs-lookup"><span data-stu-id="1ca53-134">Add the WebSockets middleware in the `Configure` method of the `Startup` class:</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSockets)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=UseWebSockets)]

::: moniker-end

<span data-ttu-id="1ca53-135">可配置以下设置：</span><span class="sxs-lookup"><span data-stu-id="1ca53-135">The following settings can be configured:</span></span>

* <span data-ttu-id="1ca53-136">`KeepAliveInterval` - 向客户端发送“ping”帧的频率，以确保代理保持连接处于打开状态。</span><span class="sxs-lookup"><span data-stu-id="1ca53-136">`KeepAliveInterval` - How frequently to send "ping" frames to the client to ensure proxies keep the connection open.</span></span>
* <span data-ttu-id="1ca53-137">`ReceiveBufferSize` - 用于接收数据的缓冲区的大小。</span><span class="sxs-lookup"><span data-stu-id="1ca53-137">`ReceiveBufferSize` - The size of the buffer used to receive data.</span></span> <span data-ttu-id="1ca53-138">高级用户可能需要对其进行更改，以便根据数据大小调整性能。</span><span class="sxs-lookup"><span data-stu-id="1ca53-138">Advanced users may need to change this for performance tuning based on the size of the data.</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptions)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptions)]

::: moniker-end

### <a name="accept-websocket-requests"></a><span data-ttu-id="1ca53-139">接受 WebSocket 请求</span><span class="sxs-lookup"><span data-stu-id="1ca53-139">Accept WebSocket requests</span></span>

<span data-ttu-id="1ca53-140">在请求生命周期后期（例如在 `Configure` 方法或 MVC 操作的后期），检查它是否是 WebSocket 请求并接受 WebSocket 请求。</span><span class="sxs-lookup"><span data-stu-id="1ca53-140">Somewhere later in the request life cycle (later in the `Configure` method or in an MVC action, for example) check if it's a WebSocket request and accept the WebSocket request.</span></span>

<span data-ttu-id="1ca53-141">以下示例来自 `Configure` 方法的后期：</span><span class="sxs-lookup"><span data-stu-id="1ca53-141">The following example is from later in the `Configure` method:</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=AcceptWebSocket&highlight=7)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=AcceptWebSocket&highlight=7)]

::: moniker-end

<span data-ttu-id="1ca53-142">WebSocket 请求可以来自任何 URL，但此示例代码只接受 `/ws` 的请求。</span><span class="sxs-lookup"><span data-stu-id="1ca53-142">A WebSocket request could come in on any URL, but this sample code only accepts requests for `/ws`.</span></span>

### <a name="send-and-receive-messages"></a><span data-ttu-id="1ca53-143">发送和接收消息</span><span class="sxs-lookup"><span data-stu-id="1ca53-143">Send and receive messages</span></span>

<span data-ttu-id="1ca53-144">`AcceptWebSocketAsync` 方法将 TCP 连接升级到 WebSocket 连接，并提供 [WebSocket](/dotnet/core/api/system.net.websockets.websocket) 对象。</span><span class="sxs-lookup"><span data-stu-id="1ca53-144">The `AcceptWebSocketAsync` method upgrades the TCP connection to a WebSocket connection and provides a [WebSocket](/dotnet/core/api/system.net.websockets.websocket) object.</span></span> <span data-ttu-id="1ca53-145">使用 `WebSocket` 对象发送和接收消息。</span><span class="sxs-lookup"><span data-stu-id="1ca53-145">Use the `WebSocket` object to send and receive messages.</span></span>

<span data-ttu-id="1ca53-146">之前显示的接受 WebSocket 请求的代码将 `WebSocket` 对象传递给 `Echo` 方法。</span><span class="sxs-lookup"><span data-stu-id="1ca53-146">The code shown earlier that accepts the WebSocket request passes the `WebSocket` object to an `Echo` method.</span></span> <span data-ttu-id="1ca53-147">代码接收消息并立即发回相同的消息。</span><span class="sxs-lookup"><span data-stu-id="1ca53-147">The code receives a message and immediately sends back the same message.</span></span> <span data-ttu-id="1ca53-148">循环发送和接收消息，直到客户端关闭连接：</span><span class="sxs-lookup"><span data-stu-id="1ca53-148">Messages are sent and received in a loop until the client closes the connection:</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=Echo)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

[!code-csharp[](websockets/samples/1.x/WebSocketsSample/Startup.cs?name=Echo)]

::: moniker-end

<span data-ttu-id="1ca53-149">如果在开始循环之前接受 WebSocket 连接，中间件管道会结束。</span><span class="sxs-lookup"><span data-stu-id="1ca53-149">When accepting the WebSocket connection before beginning the loop, the middleware pipeline ends.</span></span> <span data-ttu-id="1ca53-150">关闭套接字后，管道展开。</span><span class="sxs-lookup"><span data-stu-id="1ca53-150">Upon closing the socket, the pipeline unwinds.</span></span> <span data-ttu-id="1ca53-151">即接受 WebSocket 时，请求停止在管道中推进。</span><span class="sxs-lookup"><span data-stu-id="1ca53-151">That is, the request stops moving forward in the pipeline when the WebSocket is accepted.</span></span> <span data-ttu-id="1ca53-152">循环结束且套接字关闭时，请求继续回到管道。</span><span class="sxs-lookup"><span data-stu-id="1ca53-152">When the loop is finished and the socket is closed, the request proceeds back up the pipeline.</span></span>

## <a name="iisiis-express-support"></a><span data-ttu-id="1ca53-153">IIS/IIS Express 支持</span><span class="sxs-lookup"><span data-stu-id="1ca53-153">IIS/IIS Express support</span></span>

<span data-ttu-id="1ca53-154">安装了 IIS/IIS Express 8 或更高版本的 Windows Server 2012 或更高版本以及 Windows 8 或更高版本支持 WebSocket 协议。</span><span class="sxs-lookup"><span data-stu-id="1ca53-154">Windows Server 2012 or later and Windows 8 or later with IIS/IIS Express 8 or later has support for the WebSocket protocol.</span></span>

> [!NOTE]
> <span data-ttu-id="1ca53-155">使用 IIS Express 时始终启用 WebSocket。</span><span class="sxs-lookup"><span data-stu-id="1ca53-155">WebSockets are always enabled when using IIS Express.</span></span>

### <a name="enabling-websockets-on-iis"></a><span data-ttu-id="1ca53-156">在 IIS 上启用 Websocket</span><span class="sxs-lookup"><span data-stu-id="1ca53-156">Enabling WebSockets on IIS</span></span>

<span data-ttu-id="1ca53-157">在 Windows Server 2012 或更高版本上启用对 WebSocket 协议的支持：</span><span class="sxs-lookup"><span data-stu-id="1ca53-157">To enable support for the WebSocket protocol on Windows Server 2012 or later:</span></span>

> [!NOTE]
> <span data-ttu-id="1ca53-158">使用 IIS Express 时无需执行这些步骤</span><span class="sxs-lookup"><span data-stu-id="1ca53-158">These steps are not required when using IIS Express</span></span>

1. <span data-ttu-id="1ca53-159">通过“管理”菜单或“服务器管理器”中的链接使用“添加角色和功能”向导。</span><span class="sxs-lookup"><span data-stu-id="1ca53-159">Use the **Add Roles and Features** wizard from the **Manage** menu or the link in **Server Manager**.</span></span>
1. <span data-ttu-id="1ca53-160">选择“基于角色或基于功能的安装”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-160">Select **Role-based or Feature-based Installation**.</span></span> <span data-ttu-id="1ca53-161">选择“下一步”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-161">Select **Next**.</span></span>
1. <span data-ttu-id="1ca53-162">选择适当的服务器（默认情况下选择本地服务器）。</span><span class="sxs-lookup"><span data-stu-id="1ca53-162">Select the appropriate server (the local server is selected by default).</span></span> <span data-ttu-id="1ca53-163">选择“下一步”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-163">Select **Next**.</span></span>
1. <span data-ttu-id="1ca53-164">在“角色”树中展开“Web 服务器 (IIS)”、然后依次展开“Web 服务器”和“应用程序开发”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-164">Expand **Web Server (IIS)** in the **Roles** tree, expand **Web Server**, and then expand **Application Development**.</span></span>
1. <span data-ttu-id="1ca53-165">选择“WebSocket 协议”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-165">Select **WebSocket Protocol**.</span></span> <span data-ttu-id="1ca53-166">选择“下一步”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-166">Select **Next**.</span></span>
1. <span data-ttu-id="1ca53-167">如果无需其他功能，请选择“下一步”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-167">If additional features aren't needed, select **Next**.</span></span>
1. <span data-ttu-id="1ca53-168">选择“安装” 。</span><span class="sxs-lookup"><span data-stu-id="1ca53-168">Select **Install**.</span></span>
1. <span data-ttu-id="1ca53-169">安装完成后，选择“关闭”以退出向导。</span><span class="sxs-lookup"><span data-stu-id="1ca53-169">When the installation completes, select **Close** to exit the wizard.</span></span>

<span data-ttu-id="1ca53-170">在 Windows 8 或更高版本上启用对 WebSocket 协议的支持：</span><span class="sxs-lookup"><span data-stu-id="1ca53-170">To enable support for the WebSocket protocol on Windows 8 or later:</span></span>

> [!NOTE]
> <span data-ttu-id="1ca53-171">使用 IIS Express 时无需执行这些步骤</span><span class="sxs-lookup"><span data-stu-id="1ca53-171">These steps are not required when using IIS Express</span></span>

1. <span data-ttu-id="1ca53-172">导航到“控制面板” > “程序” > “程序和功能” > “打开或关闭 Windows 功能”（位于屏幕左侧）。</span><span class="sxs-lookup"><span data-stu-id="1ca53-172">Navigate to **Control Panel** > **Programs** > **Programs and Features** > **Turn Windows features on or off** (left side of the screen).</span></span>
1. <span data-ttu-id="1ca53-173">打开以下节点：“Internet Information Services” > “万维网服务” > “应用程序开发功能”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-173">Open the following nodes: **Internet Information Services** > **World Wide Web Services** > **Application Development Features**.</span></span>
1. <span data-ttu-id="1ca53-174">选择“WebSocket 协议”功能。</span><span class="sxs-lookup"><span data-stu-id="1ca53-174">Select the **WebSocket Protocol** feature.</span></span> <span data-ttu-id="1ca53-175">选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-175">Select **OK**.</span></span>

### <a name="disable-websocket-when-using-socketio-on-nodejs"></a><span data-ttu-id="1ca53-176">在 Node.js 上使用 socket.io 时禁用 WebSocket</span><span class="sxs-lookup"><span data-stu-id="1ca53-176">Disable WebSocket when using socket.io on Node.js</span></span>

<span data-ttu-id="1ca53-177">如果在 [Node.js](https://nodejs.org/) 的 [socket.io](https://socket.io/) 中使用 WebSocket 支持，请使用 web.config 或 applicationHost.config 中的 `webSocket` 元素禁用默认的 IIS WebSocket 模块。如果不执行此步骤，IIS WebSocket 模块将尝试处理 WebSocket 通信而不是 Node.js 和应用。</span><span class="sxs-lookup"><span data-stu-id="1ca53-177">If using the WebSocket support in [socket.io](https://socket.io/) on [Node.js](https://nodejs.org/), disable the default IIS WebSocket module using the `webSocket` element in *web.config* or *applicationHost.config*. If this step isn't performed, the IIS WebSocket module attempts to handle the WebSocket communication rather than Node.js and the app.</span></span>

```xml
<system.webServer>
  <webSocket enabled="false" />
</system.webServer>
```

## <a name="next-steps"></a><span data-ttu-id="1ca53-178">后续步骤</span><span class="sxs-lookup"><span data-stu-id="1ca53-178">Next steps</span></span>

<span data-ttu-id="1ca53-179">本文附带的[示例应用](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/websockets/samples)是一个 echo 应用。</span><span class="sxs-lookup"><span data-stu-id="1ca53-179">The [sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/websockets/samples) that accompanies this article is an echo app.</span></span> <span data-ttu-id="1ca53-180">它有一个可建立 WebSocket 连接的网页，且服务器将其收到的消息重新发回到客户端。</span><span class="sxs-lookup"><span data-stu-id="1ca53-180">It has a web page that makes WebSocket connections, and the server resends any messages it receives back to the client.</span></span> <span data-ttu-id="1ca53-181">从命令提示符运行该应用（它未设置为在安装了 IIS Express 的 Visual Studio 中运行）并导航到 http://localhost:5000。</span><span class="sxs-lookup"><span data-stu-id="1ca53-181">Run the app from a command prompt (it's not set up to run from Visual Studio with IIS Express) and navigate to http://localhost:5000.</span></span> <span data-ttu-id="1ca53-182">该网页的左上方显示连接状态：</span><span class="sxs-lookup"><span data-stu-id="1ca53-182">The web page shows the connection status in the upper left:</span></span>

![网页的初始状态](websockets/_static/start.png)

<span data-ttu-id="1ca53-184">选择“连接”，向显示的 URL 发送 WebSocket 请求。</span><span class="sxs-lookup"><span data-stu-id="1ca53-184">Select **Connect** to send a WebSocket request to the URL shown.</span></span> <span data-ttu-id="1ca53-185">输入测试消息并选择“发送”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-185">Enter a test message and select **Send**.</span></span> <span data-ttu-id="1ca53-186">完成后，请选择“关闭套接字”。</span><span class="sxs-lookup"><span data-stu-id="1ca53-186">When done, select **Close Socket**.</span></span> <span data-ttu-id="1ca53-187">“通信日志”部分会报告每一个发生的“打开”、“发送”和“关闭”操作。</span><span class="sxs-lookup"><span data-stu-id="1ca53-187">The **Communication Log** section reports each open, send, and close action as it happens.</span></span>

![网页的初始状态](websockets/_static/end.png)
