---
title: ASP.NET Core SignalR JavaScript 客户端
author: tdykstra
description: ASP.NET Core SignalR JavaScript 客户端的概述。
monikerRange: '>= aspnetcore-2.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 08/14/2018
uid: signalr/javascript-client
ms.openlocfilehash: 10958c414aa4a285c8a2810bb99e278f719c5b7f
ms.sourcegitcommit: ce6b6792c650708e92cdea051a5d166c0708c7c0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "46483044"
---
# <a name="aspnet-core-signalr-javascript-client"></a><span data-ttu-id="0cd2d-103">ASP.NET Core SignalR JavaScript 客户端</span><span class="sxs-lookup"><span data-stu-id="0cd2d-103">ASP.NET Core SignalR JavaScript client</span></span>

<span data-ttu-id="0cd2d-104">作者：[Rachel Appel](http://twitter.com/rachelappel)</span><span class="sxs-lookup"><span data-stu-id="0cd2d-104">By [Rachel Appel](http://twitter.com/rachelappel)</span></span>

<span data-ttu-id="0cd2d-105">ASP.NET Core SignalR JavaScript 客户端库，开发人员可以调用服务器端中心代码。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-105">The ASP.NET Core SignalR JavaScript client library enables developers to call server-side hub code.</span></span>

<span data-ttu-id="0cd2d-106">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/live/aspnetcore/signalr/javascript-client/sample)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）</span><span class="sxs-lookup"><span data-stu-id="0cd2d-106">[View or download sample code](https://github.com/aspnet/Docs/tree/live/aspnetcore/signalr/javascript-client/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

## <a name="install-the-signalr-client-package"></a><span data-ttu-id="0cd2d-107">安装 SignalR 客户端包</span><span class="sxs-lookup"><span data-stu-id="0cd2d-107">Install the SignalR client package</span></span>

<span data-ttu-id="0cd2d-108">作为提供 SignalR JavaScript 客户端库[npm](https://www.npmjs.com/)包。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-108">The SignalR JavaScript client library is delivered as an [npm](https://www.npmjs.com/) package.</span></span> <span data-ttu-id="0cd2d-109">如果使用 Visual Studio，运行`npm install`从**程序包管理器控制台**时的根文件夹中。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-109">If you're using Visual Studio, run `npm install` from the **Package Manager Console** while in the root folder.</span></span> <span data-ttu-id="0cd2d-110">对于 Visual Studio Code 中，运行中的命令**集成终端**。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-110">For Visual Studio Code, run the command from the **Integrated Terminal**.</span></span>

  ```console
  npm init -y
  npm install @aspnet/signalr
  ```

<span data-ttu-id="0cd2d-111">npm 安装中的包内容 *node_modules\\@aspnet\signalr\dist\browser* 文件夹。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-111">npm installs the package contents in the *node_modules\\@aspnet\signalr\dist\browser* folder.</span></span> <span data-ttu-id="0cd2d-112">创建一个名为的新文件夹*signalr*下*wwwroot\\lib*文件夹。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-112">Create a new folder named *signalr* under the *wwwroot\\lib* folder.</span></span> <span data-ttu-id="0cd2d-113">复制 *signalr.js* 的文件 *wwwroot\lib\signalr* 文件夹。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-113">Copy the *signalr.js* file to the *wwwroot\lib\signalr* folder.</span></span>

## <a name="use-the-signalr-javascript-client"></a><span data-ttu-id="0cd2d-114">使用 SignalR JavaScript 客户端</span><span class="sxs-lookup"><span data-stu-id="0cd2d-114">Use the SignalR JavaScript client</span></span>

<span data-ttu-id="0cd2d-115">引用中的 SignalR JavaScript 客户端`<script>`元素。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-115">Reference the SignalR JavaScript client in the `<script>` element.</span></span>

```html
<script src="~/lib/signalr/signalr.js"></script>
```

## <a name="connect-to-a-hub"></a><span data-ttu-id="0cd2d-116">连接到中心</span><span class="sxs-lookup"><span data-stu-id="0cd2d-116">Connect to a hub</span></span>

<span data-ttu-id="0cd2d-117">下面的代码创建并启动连接。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-117">The following code creates and starts a connection.</span></span> <span data-ttu-id="0cd2d-118">在中心的名称是不区分大小写。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-118">The hub's name is case insensitive.</span></span>

[!code-javascript[Call hub methods](javascript-client/sample/wwwroot/js/chat.js?range=9-12,28)]

### <a name="cross-origin-connections"></a><span data-ttu-id="0cd2d-119">跨域的连接</span><span class="sxs-lookup"><span data-stu-id="0cd2d-119">Cross-origin connections</span></span>

<span data-ttu-id="0cd2d-120">通常情况下，浏览器请求的页面所在的域从加载的连接。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-120">Typically, browsers load connections from the same domain as the requested page.</span></span> <span data-ttu-id="0cd2d-121">但是，有一些情况下需要与另一个域的连接时。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-121">However, there are occasions when a connection to another domain is required.</span></span>

<span data-ttu-id="0cd2d-122">若要防止恶意站点读取另一个站点中的敏感数据[跨域连接](xref:security/cors)默认处于禁用状态。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-122">To prevent a malicious site from reading sensitive data from another site, [cross-origin connections](xref:security/cors) are disabled by default.</span></span> <span data-ttu-id="0cd2d-123">若要允许跨域请求，请启用它在`Startup`类。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-123">To allow a cross-origin request, enable it in the `Startup` class.</span></span>

[!code-csharp[Cross-origin connections](javascript-client/sample/Startup.cs?highlight=29-35,56)]

## <a name="call-hub-methods-from-client"></a><span data-ttu-id="0cd2d-124">从客户端调用集线器方法</span><span class="sxs-lookup"><span data-stu-id="0cd2d-124">Call hub methods from client</span></span>

<span data-ttu-id="0cd2d-125">JavaScript 客户端上中心通过调用公共方法[调用](/javascript/api/%40aspnet/signalr/hubconnection#invoke)方法[HubConnection](/javascript/api/%40aspnet/signalr/hubconnection)。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-125">JavaScript clients call public methods on hubs via the [invoke](/javascript/api/%40aspnet/signalr/hubconnection#invoke) method of the [HubConnection](/javascript/api/%40aspnet/signalr/hubconnection).</span></span> <span data-ttu-id="0cd2d-126">`invoke`方法接受两个参数：</span><span class="sxs-lookup"><span data-stu-id="0cd2d-126">The `invoke` method accepts two arguments:</span></span>

* <span data-ttu-id="0cd2d-127">集线器方法的名称。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-127">The name of the hub method.</span></span> <span data-ttu-id="0cd2d-128">在以下示例中，中心上的方法名称是`SendMessage`。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-128">In the following example, the method name on the hub is `SendMessage`.</span></span>
* <span data-ttu-id="0cd2d-129">在集线器方法中定义的任何参数。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-129">Any arguments defined in the hub method.</span></span> <span data-ttu-id="0cd2d-130">在以下示例中，参数名称是`message`。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-130">In the following example, the argument name is `message`.</span></span>

  [!code-javascript[Call hub methods](javascript-client/sample/wwwroot/js/chat.js?range=24)]

## <a name="call-client-methods-from-hub"></a><span data-ttu-id="0cd2d-131">从集线器调用客户端方法</span><span class="sxs-lookup"><span data-stu-id="0cd2d-131">Call client methods from hub</span></span>

<span data-ttu-id="0cd2d-132">若要从中心接收消息，定义方法使用[上](/javascript/api/%40aspnet/signalr/hubconnection#on)方法的`HubConnection`。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-132">To receive messages from the hub, define a method using the [on](/javascript/api/%40aspnet/signalr/hubconnection#on) method of the `HubConnection`.</span></span>

* <span data-ttu-id="0cd2d-133">JavaScript 客户端方法的名称。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-133">The name of the JavaScript client method.</span></span> <span data-ttu-id="0cd2d-134">在以下示例中，方法名称是`ReceiveMessage`。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-134">In the following example, the method name is `ReceiveMessage`.</span></span>
* <span data-ttu-id="0cd2d-135">该中心将传递给方法的参数。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-135">Arguments the hub passes to the method.</span></span> <span data-ttu-id="0cd2d-136">在以下示例中，参数值是`message`。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-136">In the following example, the argument value is `message`.</span></span>

[!code-javascript[Receive calls from hub](javascript-client/sample/wwwroot/js/chat.js?range=14-19)]

<span data-ttu-id="0cd2d-137">在前面的代码`connection.on`运行时服务器端代码将调用它使用[SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync)方法。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-137">The preceding code in `connection.on` runs when server-side code calls it using the [SendAsync](/dotnet/api/microsoft.aspnetcore.signalr.clientproxyextensions.sendasync) method.</span></span>

[!code-csharp[Call client-side](javascript-client/sample/hubs/chathub.cs?range=8-11)]

<span data-ttu-id="0cd2d-138">SignalR 确定要进行匹配的方法名称来调用的客户端方法和参数中定义`SendAsync`和`connection.on`。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-138">SignalR determines which client method to call by matching the method name and arguments defined in `SendAsync` and `connection.on`.</span></span>

> [!NOTE]
> <span data-ttu-id="0cd2d-139">最佳做法是，调用[启动](/javascript/api/%40aspnet/signalr/hubconnection#start)方法`HubConnection`后`on`。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-139">As a best practice, call the [start](/javascript/api/%40aspnet/signalr/hubconnection#start) method on the `HubConnection` after `on`.</span></span> <span data-ttu-id="0cd2d-140">这样做可确保您的处理程序注册之前接收任何消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-140">Doing so ensures your handlers are registered before any messages are received.</span></span>

## <a name="error-handling-and-logging"></a><span data-ttu-id="0cd2d-141">错误处理和日志记录</span><span class="sxs-lookup"><span data-stu-id="0cd2d-141">Error handling and logging</span></span>

<span data-ttu-id="0cd2d-142">链`catch`方法的末尾`start`方法以处理客户端错误。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-142">Chain a `catch` method to the end of the `start` method to handle client-side errors.</span></span> <span data-ttu-id="0cd2d-143">使用`console.error`向浏览器的控制台输出错误。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-143">Use `console.error` to output errors to the browser's console.</span></span>

[!code-javascript[Error handling](javascript-client/sample/wwwroot/js/chat.js?range=28)]

<span data-ttu-id="0cd2d-144">通过传递要进行连接时记录的记录器和事件类型的安装程序客户端的日志跟踪。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-144">Setup client-side log tracing by passing a logger and type of event to log when the connection is made.</span></span> <span data-ttu-id="0cd2d-145">使用指定的日志级别和更高版本记录的消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-145">Messages are logged with the specified log level and higher.</span></span> <span data-ttu-id="0cd2d-146">可用日志级别为按如下所示：</span><span class="sxs-lookup"><span data-stu-id="0cd2d-146">Available log levels are as follows:</span></span>

* <span data-ttu-id="0cd2d-147">`signalR.LogLevel.Error` &ndash; 错误消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-147">`signalR.LogLevel.Error` &ndash; Error messages.</span></span> <span data-ttu-id="0cd2d-148">日志`Error`仅消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-148">Logs `Error` messages only.</span></span>
* <span data-ttu-id="0cd2d-149">`signalR.LogLevel.Warning` &ndash; 有关潜在错误的警告消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-149">`signalR.LogLevel.Warning` &ndash; Warning messages about potential errors.</span></span> <span data-ttu-id="0cd2d-150">日志`Warning`，和`Error`消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-150">Logs `Warning`, and `Error` messages.</span></span>
* <span data-ttu-id="0cd2d-151">`signalR.LogLevel.Information` &ndash; 不包含错误的状态消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-151">`signalR.LogLevel.Information` &ndash; Status messages without errors.</span></span> <span data-ttu-id="0cd2d-152">日志`Information`， `Warning`，和`Error`消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-152">Logs `Information`, `Warning`, and `Error` messages.</span></span>
* <span data-ttu-id="0cd2d-153">`signalR.LogLevel.Trace` &ndash; 跟踪消息。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-153">`signalR.LogLevel.Trace` &ndash; Trace messages.</span></span> <span data-ttu-id="0cd2d-154">记录所有内容，包括数据中心和客户端之间传输。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-154">Logs everything, including data transported between hub and client.</span></span>

<span data-ttu-id="0cd2d-155">使用[configureLogging](/javascript/api/%40aspnet/signalr/hubconnectionbuilder#configurelogging)方法[HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder)若要配置日志级别。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-155">Use the [configureLogging](/javascript/api/%40aspnet/signalr/hubconnectionbuilder#configurelogging) method on [HubConnectionBuilder](/javascript/api/%40aspnet/signalr/hubconnectionbuilder) to configure the log level.</span></span> <span data-ttu-id="0cd2d-156">消息会记录到浏览器控制台。</span><span class="sxs-lookup"><span data-stu-id="0cd2d-156">Messages are logged to the browser console.</span></span>

[!code-javascript[Logging levels](javascript-client/sample/wwwroot/js/chat.js?range=9-12)]

## <a name="additional-resources"></a><span data-ttu-id="0cd2d-157">其他资源</span><span class="sxs-lookup"><span data-stu-id="0cd2d-157">Additional resources</span></span>

* [<span data-ttu-id="0cd2d-158">JavaScript API 参考</span><span class="sxs-lookup"><span data-stu-id="0cd2d-158">JavaScript API reference</span></span>](/javascript/api/?view=signalr-js-latest)
* [<span data-ttu-id="0cd2d-159">中心</span><span class="sxs-lookup"><span data-stu-id="0cd2d-159">Hubs</span></span>](xref:signalr/hubs)
* [<span data-ttu-id="0cd2d-160">.NET 客户端</span><span class="sxs-lookup"><span data-stu-id="0cd2d-160">.NET client</span></span>](xref:signalr/dotnet-client)
* [<span data-ttu-id="0cd2d-161">发布到 Azure</span><span class="sxs-lookup"><span data-stu-id="0cd2d-161">Publish to Azure</span></span>](xref:signalr/publish-to-azure-web-app)
* [<span data-ttu-id="0cd2d-162">启用 ASP.NET Core 中的跨源请求 (CORS)</span><span class="sxs-lookup"><span data-stu-id="0cd2d-162">Enable Cross-Origin Requests (CORS) in ASP.NET Core</span></span>](xref:security/cors)
