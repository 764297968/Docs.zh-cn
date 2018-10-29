---
title: SignalR HubContext
author: tdykstra
description: 了解如何使用 ASP.NET Core SignalR HubContext 服务用于向外部的客户端从一个中心发送通知。
monikerRange: '>= aspnetcore-2.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 06/13/2018
uid: signalr/hubcontext
ms.openlocfilehash: 8be888e1f7b16d65ebbaa24b618e84fca029d80b
ms.sourcegitcommit: 375e9a67f5e1f7b0faaa056b4b46294cc70f55b7
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/29/2018
ms.locfileid: "50207948"
---
# <a name="send-messages-from-outside-a-hub"></a><span data-ttu-id="34d0a-103">从向外发送邮件中心</span><span class="sxs-lookup"><span data-stu-id="34d0a-103">Send messages from outside a hub</span></span>

<span data-ttu-id="34d0a-104">通过[Mikael Mengistu](https://twitter.com/MikaelM_12)</span><span class="sxs-lookup"><span data-stu-id="34d0a-104">By [Mikael Mengistu](https://twitter.com/MikaelM_12)</span></span>

<span data-ttu-id="34d0a-105">SignalR 中心是用于将消息发送到客户端连接到 SignalR 服务器的核心抽象。</span><span class="sxs-lookup"><span data-stu-id="34d0a-105">The SignalR hub is the core abstraction for sending messages to clients connected to the SignalR server.</span></span> <span data-ttu-id="34d0a-106">还有可能将消息从你的应用使用中的其他位置发送`IHubContext`服务。</span><span class="sxs-lookup"><span data-stu-id="34d0a-106">It's also possible to send messages from other places in your app using the `IHubContext` service.</span></span> <span data-ttu-id="34d0a-107">此文章介绍了如何访问 SignalR`IHubContext`将通知发送到外部的客户端从一个中心。</span><span class="sxs-lookup"><span data-stu-id="34d0a-107">This article explains how to access a SignalR `IHubContext` to send notifications to clients from outside a hub.</span></span>

<span data-ttu-id="34d0a-108">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/signalr/hubcontext/sample/) [（如何下载）](xref:index#how-to-download-a-sample)</span><span class="sxs-lookup"><span data-stu-id="34d0a-108">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/signalr/hubcontext/sample/) [(how to download)](xref:index#how-to-download-a-sample)</span></span>

## <a name="get-an-instance-of-ihubcontext"></a><span data-ttu-id="34d0a-109">获取 IHubContext 的实例</span><span class="sxs-lookup"><span data-stu-id="34d0a-109">Get an instance of IHubContext</span></span>

<span data-ttu-id="34d0a-110">在 ASP.NET Core SignalR，您可以访问的实例`IHubContext`通过依赖关系注入。</span><span class="sxs-lookup"><span data-stu-id="34d0a-110">In ASP.NET Core SignalR, you can access an instance of `IHubContext` via dependency injection.</span></span> <span data-ttu-id="34d0a-111">您可以注入的实例`IHubContext`到控制器、 中间件或其他 DI 服务。</span><span class="sxs-lookup"><span data-stu-id="34d0a-111">You can inject an instance of `IHubContext` into a controller, middleware, or other DI service.</span></span> <span data-ttu-id="34d0a-112">使用的实例将消息发送到客户端。</span><span class="sxs-lookup"><span data-stu-id="34d0a-112">Use the instance to send messages to clients.</span></span>

> [!NOTE]
> <span data-ttu-id="34d0a-113">这不同于 ASP.NET 4.x SignalR GlobalHost 用于提供对访问`IHubContext`。</span><span class="sxs-lookup"><span data-stu-id="34d0a-113">This differs from ASP.NET 4.x SignalR which used GlobalHost to provide access to the `IHubContext`.</span></span> <span data-ttu-id="34d0a-114">ASP.NET Core具有的依赖关系注入框架，无需此全局单一实例。</span><span class="sxs-lookup"><span data-stu-id="34d0a-114">ASP.NET Core has a dependency injection framework that removes the need for this global singleton.</span></span>

### <a name="inject-an-instance-of-ihubcontext-in-a-controller"></a><span data-ttu-id="34d0a-115">注入控制器中的 IHubContext 实例</span><span class="sxs-lookup"><span data-stu-id="34d0a-115">Inject an instance of IHubContext in a controller</span></span>

<span data-ttu-id="34d0a-116">您可以注入的实例`IHubContext`插入控制器将其添加到您的构造函数：</span><span class="sxs-lookup"><span data-stu-id="34d0a-116">You can inject an instance of `IHubContext` into a controller by adding it to your constructor:</span></span>

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=12-19,57)]

<span data-ttu-id="34d0a-117">现在，有权访问的实例`IHubContext`，好像您在中心本身中，可以调用集线器方法。</span><span class="sxs-lookup"><span data-stu-id="34d0a-117">Now, with access to an instance of `IHubContext`, you can call hub methods as if you were in the hub itself.</span></span>

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=21-25)]

### <a name="get-an-instance-of-ihubcontext-in-middleware"></a><span data-ttu-id="34d0a-118">获取中间件中的 IHubContext 实例</span><span class="sxs-lookup"><span data-stu-id="34d0a-118">Get an instance of IHubContext in middleware</span></span>

<span data-ttu-id="34d0a-119">访问`IHubContext`中间件管道中如下所示：</span><span class="sxs-lookup"><span data-stu-id="34d0a-119">Access the `IHubContext` within the middleware pipeline like so:</span></span>

```csharp
app.Use(next => async (context) =>
{
    var hubContext = context.RequestServices
                            .GetRequiredService<IHubContext<MyHub>>();
    //...
});
```

> [!NOTE]
> <span data-ttu-id="34d0a-120">外部的从调用集线器方法时`Hub`类中，没有与调用相关联的任何调用方。</span><span class="sxs-lookup"><span data-stu-id="34d0a-120">When hub methods are called from outside of the `Hub` class, there's no caller associated with the invocation.</span></span> <span data-ttu-id="34d0a-121">因此，没有不能访问`ConnectionId`， `Caller`，和`Others`属性。</span><span class="sxs-lookup"><span data-stu-id="34d0a-121">Therefore, there's no access to the `ConnectionId`, `Caller`, and `Others` properties.</span></span>

## <a name="related-resources"></a><span data-ttu-id="34d0a-122">相关资源</span><span class="sxs-lookup"><span data-stu-id="34d0a-122">Related resources</span></span>

* [<span data-ttu-id="34d0a-123">入门</span><span class="sxs-lookup"><span data-stu-id="34d0a-123">Get started</span></span>](xref:tutorials/signalr)
* [<span data-ttu-id="34d0a-124">中心</span><span class="sxs-lookup"><span data-stu-id="34d0a-124">Hubs</span></span>](xref:signalr/hubs)
* [<span data-ttu-id="34d0a-125">发布到 Azure</span><span class="sxs-lookup"><span data-stu-id="34d0a-125">Publish to Azure</span></span>](xref:signalr/publish-to-azure-web-app)
