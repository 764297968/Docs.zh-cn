---
title: ASP.NET Core 中的 URL 重写中间件
author: guardrex
description: 了解如何在 ASP.NET Core 应用程序中使用 URL 重写中间件进行 URL 重写和重定向。
ms.author: riande
ms.date: 08/17/2017
uid: fundamentals/url-rewriting
ms.openlocfilehash: d9f33f34f75fe7bf534146c5a426335e74635018
ms.sourcegitcommit: 4bdf7703aed86ebd56b9b4bae9ad5700002af32d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326064"
---
# <a name="url-rewriting-middleware-in-aspnet-core"></a><span data-ttu-id="4f9f2-103">ASP.NET Core 中的 URL 重写中间件</span><span class="sxs-lookup"><span data-stu-id="4f9f2-103">URL Rewriting Middleware in ASP.NET Core</span></span>

<span data-ttu-id="4f9f2-104">作者：[Luke Latham](https://github.com/guardrex) 和 [Mikael Mengistu](https://github.com/mikaelm12)</span><span class="sxs-lookup"><span data-stu-id="4f9f2-104">By [Luke Latham](https://github.com/guardrex) and [Mikael Mengistu](https://github.com/mikaelm12)</span></span>

<span data-ttu-id="4f9f2-105">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/url-rewriting/sample/)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）</span><span class="sxs-lookup"><span data-stu-id="4f9f2-105">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/url-rewriting/sample/) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

<span data-ttu-id="4f9f2-106">URL 重写是根据一个或多个预定义规则修改请求 URL 的行为。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-106">URL rewriting is the act of modifying request URLs based on one or more predefined rules.</span></span> <span data-ttu-id="4f9f2-107">URL 重写会在资源位置和地址之间创建一个抽象，使位置和地址不紧密相连。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-107">URL rewriting creates an abstraction between resource locations and their addresses so that the locations and addresses are not tightly linked.</span></span> <span data-ttu-id="4f9f2-108">在以下几种方案中，URL 重写很有价值：</span><span class="sxs-lookup"><span data-stu-id="4f9f2-108">There are several scenarios where URL rewriting is valuable:</span></span>

* <span data-ttu-id="4f9f2-109">暂时或永久移动或替换服务器资源，同时维护这些资源的稳定定位符。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-109">Moving or replacing server resources temporarily or permanently while maintaining stable locators for those resources.</span></span>
* <span data-ttu-id="4f9f2-110">拆分在不同应用或同一应用的不同区域中处理的请求。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-110">Splitting request processing across different apps or across areas of one app.</span></span>
* <span data-ttu-id="4f9f2-111">删除、添加或重新组织传入请求上的 URL 段。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-111">Removing, adding, or reorganizing URL segments on incoming requests.</span></span>
* <span data-ttu-id="4f9f2-112">优化搜索引擎优化 (SEO) 的公共 URL。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-112">Optimizing public URLs for Search Engine Optimization (SEO).</span></span>
* <span data-ttu-id="4f9f2-113">允许使用友好的公共 URL 帮助用户预测他们将通过链接找到的内容。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-113">Permitting the use of friendly public URLs to help people predict the content they will find by following a link.</span></span>
* <span data-ttu-id="4f9f2-114">将不安全请求重定向到安全终结点。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-114">Redirecting insecure requests to secure endpoints.</span></span>
* <span data-ttu-id="4f9f2-115">阻止映像盗链。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-115">Preventing image hotlinking.</span></span>

<span data-ttu-id="4f9f2-116">可通过多种方式定义更改 URL 的规则，包括正则表达式、Apache mod_rewrite 模块规则、IIS 重写模块规则以及使用自定义规则逻辑。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-116">You can define rules for changing the URL in several ways, including Regex, Apache mod_rewrite module rules, IIS Rewrite Module rules, and using custom rule logic.</span></span> <span data-ttu-id="4f9f2-117">本文档介绍 URL 重写并说明如何在 ASP.NET Core 应用中使用 URL 重写中间件。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-117">This document introduces URL rewriting with instructions on how to use URL Rewriting Middleware in ASP.NET Core apps.</span></span>

> [!NOTE]
> <span data-ttu-id="4f9f2-118">URL 重写可能会降低应用的性能。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-118">URL rewriting can reduce the performance of an app.</span></span> <span data-ttu-id="4f9f2-119">如果可行，应限制规则的数量和复杂度。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-119">Where feasible, you should limit the number and complexity of rules.</span></span>

## <a name="url-redirect-and-url-rewrite"></a><span data-ttu-id="4f9f2-120">URL 重定向和 URL 重写</span><span class="sxs-lookup"><span data-stu-id="4f9f2-120">URL redirect and URL rewrite</span></span>

<span data-ttu-id="4f9f2-121">URL 重定向和 URL 重写之间的用词差异乍一看可能很细微，但这对于向客户端提供资源具有重要意义。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-121">The difference in wording between *URL redirect* and *URL rewrite* may seem subtle at first but has important implications for providing resources to clients.</span></span> <span data-ttu-id="4f9f2-122">ASP.NET Core 的 URL 重写中间件能够满足两者的需求。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-122">ASP.NET Core's URL Rewriting Middleware is capable of meeting the need for both.</span></span>

<span data-ttu-id="4f9f2-123">URL 重定向是客户端操作，指示客户端访问另一个地址的资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-123">A *URL redirect* is a client-side operation, where the client is instructed to access a resource at another address.</span></span> <span data-ttu-id="4f9f2-124">这需要往返服务器。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-124">This requires a round-trip to the server.</span></span> <span data-ttu-id="4f9f2-125">客户端对资源发出新请求时，返回客户端的重定向 URL 会出现在浏览器地址栏。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-125">The redirect URL returned to the client appears in the browser's address bar when the client makes a new request for the resource.</span></span> 

<span data-ttu-id="4f9f2-126">如果将 `/resource` 重定向到 `/different-resource`，客户端会请求`/resource`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-126">If `/resource` is *redirected* to `/different-resource`, the client requests `/resource`.</span></span> <span data-ttu-id="4f9f2-127">服务器通过指示重定向是临时还是永久的状态代码作出响应，表示客户端应该获取 `/different-resource` 处的资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-127">The server responds that the client should obtain the resource at `/different-resource` with a status code indicating that the redirect is either temporary or permanent.</span></span> <span data-ttu-id="4f9f2-128">客户端在重定向 URL 处对资源执行新的请求。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-128">The client executes a new request for the resource at the redirect URL.</span></span>

![WebAPI 服务终结点已暂时从服务器上的版本 1 (v1) 更改为版本 2 (v2)。](url-rewriting/_static/url_redirect.png)

<span data-ttu-id="4f9f2-134">将请求重定向到不同的 URL 时，可指示重定向是永久的还是临时的。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-134">When redirecting requests to a different URL, you indicate whether the redirect is permanent or temporary.</span></span> <span data-ttu-id="4f9f2-135">如果资源有一个新的永久性 URL，并且你希望指示客户端所有将来的资源请求都使用新 URL，则使用“301 (永久移动)”状态代码。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-135">The 301 (Moved Permanently) status code is used where the resource has a new, permanent URL and you wish to instruct the client that all future requests for the resource should use the new URL.</span></span> <span data-ttu-id="4f9f2-136">收到 301 状态代码时，客户端可能会缓存响应。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-136">*The client may cache the response when a 301 status code is received.*</span></span> <span data-ttu-id="4f9f2-137">如果重定向是临时的或一般会更改的，则使用“302 (已找到)”状态代码，以使客户端将来不应存储和重用重定向 URL。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-137">The 302 (Found) status code is used where the redirection is temporary or generally subject to change, such that the client shouldn't store and reuse the redirect URL in the future.</span></span> <span data-ttu-id="4f9f2-138">有关详细信息，请参阅 [RFC 2616：状态代码定义](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html)。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-138">For more information, see [RFC 2616: Status Code Definitions](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).</span></span>

<span data-ttu-id="4f9f2-139">URL 重写是服务器端操作，提供来自不同资源地址的资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-139">A *URL rewrite* is a server-side operation to provide a resource from a different resource address.</span></span> <span data-ttu-id="4f9f2-140">重写 URL 不需要往返服务器。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-140">Rewriting a URL doesn't require a round-trip to the server.</span></span> <span data-ttu-id="4f9f2-141">重写的 URL 不会返回客户端，也不会出现在浏览器地址栏。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-141">The rewritten URL isn't returned to the client and won't appear in the browser's address bar.</span></span> <span data-ttu-id="4f9f2-142">`/resource` 重写到 `/different-resource` 时，客户端会请求 `/resource`，并且服务器会在内部提取 `/different-resource` 处的资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-142">When `/resource` is *rewritten* to `/different-resource`, the client requests `/resource`, and the server *internally* fetches the resource at `/different-resource`.</span></span> <span data-ttu-id="4f9f2-143">尽管客户端可能能够检索已重写 URL 处的资源，但是，客户端发出请求并收到响应时，并不会知道已重写 URL 处存在的资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-143">Although the client might be able to retrieve the resource at the rewritten URL, the client won't be informed that the resource exists at the rewritten URL when it makes its request and receives the response.</span></span>

![WebAPI 服务终结点已从服务器上的版本 1 (v1) 更改为版本 2 (v2)。](url-rewriting/_static/url_rewrite.png)

## <a name="url-rewriting-sample-app"></a><span data-ttu-id="4f9f2-148">URL 重写示例应用</span><span class="sxs-lookup"><span data-stu-id="4f9f2-148">URL rewriting sample app</span></span>

<span data-ttu-id="4f9f2-149">可使用 [URL 重写示例应用](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/url-rewriting/sample/)了解 URL 重写中间件的功能。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-149">You can explore the features of the URL Rewriting Middleware with the [URL rewriting sample app](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/url-rewriting/sample/).</span></span> <span data-ttu-id="4f9f2-150">此应用使用重写和重定向规则，并显示重写或重定向 URL。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-150">The app applies rewrite and redirect rules and shows the rewritten or redirected URL.</span></span>

## <a name="when-to-use-url-rewriting-middleware"></a><span data-ttu-id="4f9f2-151">何时使用 URL 重写中间件</span><span class="sxs-lookup"><span data-stu-id="4f9f2-151">When to use URL Rewriting Middleware</span></span>

<span data-ttu-id="4f9f2-152">无法结合使用 [URL 重写模块](https://www.iis.net/downloads/microsoft/url-rewrite)和 Windows Server 上的 IIS、Apache Server 上的 [Apache mod_rewrite 模块](https://httpd.apache.org/docs/2.4/rewrite/)、[Nginx 上的 URL 重写](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)，或者应用托管在 [HTTP.sys 服务器](xref:fundamentals/servers/httpsys)（以前称为 [WebListener](xref:fundamentals/servers/weblistener)）上时，请使用 URL 重写中间件。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-152">Use URL Rewriting Middleware when you are unable to use the [URL Rewrite module](https://www.iis.net/downloads/microsoft/url-rewrite) with IIS on Windows Server, the [Apache mod_rewrite module](https://httpd.apache.org/docs/2.4/rewrite/) on Apache Server, [URL rewriting on Nginx](https://www.nginx.com/blog/creating-nginx-rewrite-rules/), or your app is hosted on [HTTP.sys server](xref:fundamentals/servers/httpsys) (formerly called [WebListener](xref:fundamentals/servers/weblistener)).</span></span> <span data-ttu-id="4f9f2-153">在 IIS、Apache 或 Nginx 中使用基于服务器的 URL 重写技术的主要原因是，中间件不支持这些模块的全部功能，而且中间件的性能可能与模块的性能不匹配。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-153">The main reasons to use the server-based URL rewriting technologies in IIS, Apache, or Nginx are that the middleware doesn't support the full features of these modules and the performance of the middleware probably won't match that of the modules.</span></span> <span data-ttu-id="4f9f2-154">但是，服务器模块的某些功能不适用于 ASP.NET Core 项目，例如 IIS 重写模块的 `IsFile` 和 `IsDirectory` 约束。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-154">However, there are some features of the server modules that don't work with ASP.NET Core projects, such as the `IsFile` and `IsDirectory` constraints of the IIS Rewrite module.</span></span> <span data-ttu-id="4f9f2-155">在这些情况下，请改为使用中间件。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-155">In these scenarios, use the middleware instead.</span></span>

## <a name="package"></a><span data-ttu-id="4f9f2-156">Package</span><span class="sxs-lookup"><span data-stu-id="4f9f2-156">Package</span></span>

<span data-ttu-id="4f9f2-157">要将中间件包含在项目中，请添加对 [`Microsoft.AspNetCore.Rewrite`](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite/) 包的引用。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-157">To include the middleware in your project, add a reference to the [`Microsoft.AspNetCore.Rewrite`](https://www.nuget.org/packages/Microsoft.AspNetCore.Rewrite/) package.</span></span> <span data-ttu-id="4f9f2-158">此功能适用于面向 ASP.NET Core 1.1 或更高版本的应用。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-158">This feature is available for apps that target ASP.NET Core 1.1 or later.</span></span>

## <a name="extension-and-options"></a><span data-ttu-id="4f9f2-159">扩展和选项</span><span class="sxs-lookup"><span data-stu-id="4f9f2-159">Extension and options</span></span>

<span data-ttu-id="4f9f2-160">通过使用扩展方法为每条规则创建 [RewriteOptions](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptions) 类的实例，建立 URL 重写和重写定向规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-160">Establish your URL rewrite and redirect rules by creating an instance of the [RewriteOptions](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptions) class with extension methods for each of your rules.</span></span> <span data-ttu-id="4f9f2-161">按所需的处理顺序链接多个规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-161">Chain multiple rules in the order that you would like them processed.</span></span> <span data-ttu-id="4f9f2-162">使用 `app.UseRewriter(options);` 将 `RewriteOptions` 添加到请求管道时，它会被传递到 URL 重写中间件。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-162">The `RewriteOptions` are passed into the URL Rewriting Middleware as it's added to the request pipeline with `app.UseRewriter(options);`.</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](url-rewriting/sample/Startup.cs?name=snippet1)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    var options = new RewriteOptions()
        .AddRedirect("redirect-rule/(.*)", "redirected/$1")
        .AddRewrite(@"^rewrite-rule/(\d+)/(\d+)", "rewritten?var1=$1&var2=$2", 
            skipRemainingRules: true)
        .AddApacheModRewrite(env.ContentRootFileProvider, "ApacheModRewrite.txt")
        .AddIISUrlRewrite(env.ContentRootFileProvider, "IISUrlRewrite.xml")
        .Add(RedirectXMLRequests)
        .Add(new RedirectImageRequests(".png", "/png-images"))
        .Add(new RedirectImageRequests(".jpg", "/jpg-images"));

    app.UseRewriter(options);
}
```

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

### <a name="redirect-non-www-to-www"></a><span data-ttu-id="4f9f2-163">将非 www 重定向到 www</span><span class="sxs-lookup"><span data-stu-id="4f9f2-163">Redirect non-www to www</span></span>

<span data-ttu-id="4f9f2-164">三个选项允许应用将非 `www` 重新定向到 `www`：</span><span class="sxs-lookup"><span data-stu-id="4f9f2-164">Three options permit the app to redirect non-`www` requests to `www`:</span></span>

* <span data-ttu-id="4f9f2-165">[AddRedirectToWwwPermanent(RewriteOptions)](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptionsextensions.addredirecttowwwpermanent) &ndash; 如果请求是非 `www`，则将请求永久重定向到 `www` 子域。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-165">[AddRedirectToWwwPermanent(RewriteOptions)](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptionsextensions.addredirecttowwwpermanent) &ndash; Permanently redirect the request to the `www` subdomain if the request is non-`www`.</span></span> <span data-ttu-id="4f9f2-166">使用 [Status308PermanentRedirect](/dotnet/api/microsoft.aspnetcore.http.statuscodes.status308permanentredirect) 状态代码进行重定向。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-166">Redirects with a [Status308PermanentRedirect](/dotnet/api/microsoft.aspnetcore.http.statuscodes.status308permanentredirect) status code.</span></span>
* <span data-ttu-id="4f9f2-167">[ AddRedirectToWww（RewriteOptions）](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptionsextensions.addredirecttowww) &ndash; 如果传入请求为非 `www`，则将请求重定向到 `www` 子域。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-167">[AddRedirectToWww(RewriteOptions)](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptionsextensions.addredirecttowww) &ndash; Redirect the request to the `www` subdomain if the incoming request is non-`www`.</span></span> <span data-ttu-id="4f9f2-168">使用 [Status307TemporaryRedirect](/dotnet/api/microsoft.aspnetcore.http.statuscodes.status307temporaryredirect) 状态代码进行重定向。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-168">Redirects with a [Status307TemporaryRedirect](/dotnet/api/microsoft.aspnetcore.http.statuscodes.status307temporaryredirect) status code.</span></span>
* <span data-ttu-id="4f9f2-169">[ AddRedirectToWww（RewriteOptions，Int32）](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptionsextensions.addredirecttowww) &ndash; 如果传入请求为非 `www`，则将请求重定向到 `www` 子域。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-169">[AddRedirectToWww(RewriteOptions, Int32)](/dotnet/api/microsoft.aspnetcore.rewrite.rewriteoptionsextensions.addredirecttowww) &ndash; Redirect the request to the `www` subdomain if the incoming request is non-`www`.</span></span> <span data-ttu-id="4f9f2-170">允许提供响应状态代码。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-170">Allows you to provide the status code for the response.</span></span> <span data-ttu-id="4f9f2-171">使用 [StatusCodes](/dotnet/api/microsoft.aspnetcore.http.statuscodes) 类的字段进行 `AddRedirectToWww` 的分配。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-171">Use the fields of the [StatusCodes](/dotnet/api/microsoft.aspnetcore.http.statuscodes) class for assignments to `AddRedirectToWww`.</span></span>

::: moniker-end

### <a name="url-redirect"></a><span data-ttu-id="4f9f2-172">URL 重定向</span><span class="sxs-lookup"><span data-stu-id="4f9f2-172">URL redirect</span></span>

<span data-ttu-id="4f9f2-173">使用 `AddRedirect` 将请求重定向。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-173">Use `AddRedirect` to redirect requests.</span></span> <span data-ttu-id="4f9f2-174">第一个参数包含用于匹配传入 URL 路径的正则表达式。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-174">The first parameter contains your regex for matching on the path of the incoming URL.</span></span> <span data-ttu-id="4f9f2-175">第二个参数是替换字符串。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-175">The second parameter is the replacement string.</span></span> <span data-ttu-id="4f9f2-176">第三个参数（如有）指定状态代码。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-176">The third parameter, if present, specifies the status code.</span></span> <span data-ttu-id="4f9f2-177">如不指定状态代码，则默认为“302 (已找到)”，指示资源暂时移动或替换。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-177">If you don't specify the status code, it defaults to 302 (Found), which indicates that the resource is temporarily moved or replaced.</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](url-rewriting/sample/Startup.cs?name=snippet1&highlight=9)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirect("redirect-rule/(.*)", "redirected/$1");

    app.UseRewriter(options);
}
```

::: moniker-end

<span data-ttu-id="4f9f2-178">在启用了开发人员工具的浏览器中，向路径为 `/redirect-rule/1234/5678` 的示例应用发出请求。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-178">In a browser with developer tools enabled, make a request to the sample app with the path `/redirect-rule/1234/5678`.</span></span> <span data-ttu-id="4f9f2-179">正则表达式匹配 `redirect-rule/(.*)` 上的请求路径，且该路径会被 `/redirected/1234/5678` 替代。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-179">The regex matches the request path on `redirect-rule/(.*)`, and the path is replaced with `/redirected/1234/5678`.</span></span> <span data-ttu-id="4f9f2-180">重定向 URL 以“302 (已找到)”状态代码发回客户端。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-180">The redirect URL is sent back to the client with a 302 (Found) status code.</span></span> <span data-ttu-id="4f9f2-181">浏览器会在浏览器地址栏中出现的重定向 URL 上发出新请求。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-181">The browser makes a new request at the redirect URL, which appears in the browser's address bar.</span></span> <span data-ttu-id="4f9f2-182">由于示例应用中的任何规则均不匹配重定向 URL，因此第二个请求会收到来自应用的“200 (正常)”响应，且响应正文会显示此重定向 URL。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-182">Since no rules in the sample app match on the redirect URL, the second request receives a 200 (OK) response from the app and the body of the response shows the redirect URL.</span></span> <span data-ttu-id="4f9f2-183">重定向 URL 时，系统将向服务器进行一次往返。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-183">A roundtrip is made to the server when a URL is *redirected*.</span></span>

> [!WARNING]
> <span data-ttu-id="4f9f2-184">建立重定向规则时务必小心。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-184">Be cautious when establishing your redirect rules.</span></span> <span data-ttu-id="4f9f2-185">系统会根据应用的每个请求（包括重定向后的请求）对重定向规则进行评估。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-185">Your redirect rules are evaluated on each request to the app, including after a redirect.</span></span> <span data-ttu-id="4f9f2-186">很容易便会意外创建无限重定向循环。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-186">It's easy to accidently create a loop of infinite redirects.</span></span>

<span data-ttu-id="4f9f2-187">原始请求：`/redirect-rule/1234/5678`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-187">Original Request: `/redirect-rule/1234/5678`</span></span>

![开发人员工具正跟踪请求和响应的浏览器窗口](url-rewriting/_static/add_redirect.png)

<span data-ttu-id="4f9f2-189">括号内的表达式部分称为“捕获组”。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-189">The part of the expression contained within parentheses is called a *capture group*.</span></span> <span data-ttu-id="4f9f2-190">表达式的点 (`.`) 表示匹配任何字符。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-190">The dot (`.`) of the expression means *match any character*.</span></span> <span data-ttu-id="4f9f2-191">星号 (`*`) 表示零次或多次匹配前面的字符。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-191">The asterisk (`*`) indicates *match the preceding character zero or more times*.</span></span> <span data-ttu-id="4f9f2-192">因此，URL 的最后两个路径段 `1234/5678` 由捕获组 `(.*)` 捕获。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-192">Therefore, the last two path segments of the URL, `1234/5678`, are captured by capture group `(.*)`.</span></span> <span data-ttu-id="4f9f2-193">在请求 URL 中提供的位于 `redirect-rule/` 之后的任何值均由此单个捕获组捕获。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-193">Any value you provide in the request URL after `redirect-rule/` is captured by this single capture group.</span></span>

<span data-ttu-id="4f9f2-194">在替换字符串中，将捕获组注入带有美元符号 (`$`)、后跟捕获序列号的字符串中。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-194">In the replacement string, captured groups are injected into the string with the dollar sign (`$`) followed by the sequence number of the capture.</span></span> <span data-ttu-id="4f9f2-195">获取的第一个捕获组值为 `$1`，第二个为 `$2`，并且正则表达式中的其他捕获组值将依次继续排列。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-195">The first capture group value is obtained with `$1`, the second with `$2`, and they continue in sequence for the capture groups in your regex.</span></span> <span data-ttu-id="4f9f2-196">示例应用的重定向规则正则表达式中只有一个捕获组，因此替换字符串中只有一个注入组，即 `$1`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-196">There's only one captured group in the redirect rule regex in the sample app, so there's only one injected group in the replacement string, which is `$1`.</span></span> <span data-ttu-id="4f9f2-197">如果应用此规则，URL 将变为 `/redirected/1234/5678`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-197">When the rule is applied, the URL becomes `/redirected/1234/5678`.</span></span>

### <a name="url-redirect-to-a-secure-endpoint"></a><span data-ttu-id="4f9f2-198">URL 重定向到安全的终结点</span><span class="sxs-lookup"><span data-stu-id="4f9f2-198">URL redirect to a secure endpoint</span></span>

<span data-ttu-id="4f9f2-199">使用 `AddRedirectToHttps` 将 HTTP 请求重定向到采用 HTTPS (`https://`) 的相同主机和路径。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-199">Use `AddRedirectToHttps` to redirect HTTP requests to the same host and path using HTTPS (`https://`).</span></span> <span data-ttu-id="4f9f2-200">如不提供状态代码，则中间件默认为“302(已找到)”。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-200">If the status code isn't supplied, the middleware defaults to 302 (Found).</span></span> <span data-ttu-id="4f9f2-201">如不提供端口，则中间件默认为 `null`，这表示协议更改为 `https://` 且客户端访问端口 443 上的资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-201">If the port isn't supplied, the middleware defaults to `null`, which means the protocol changes to `https://` and the client accesses the resource on port 443.</span></span> <span data-ttu-id="4f9f2-202">该示例演示如何将状态代码设置为“301(永久移动)”并将端口更改为 5001。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-202">The example shows how to set the status code to 301 (Moved Permanently) and change the port to 5001.</span></span>

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttps(301, 5001);

    app.UseRewriter(options);
}
```

<span data-ttu-id="4f9f2-203">使用 `AddRedirectToHttpsPermanent` 将不安全的请求重定向到采用安全 HTTPS 协议（端口 443 上的 `https://`）的相同主机和路径。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-203">Use `AddRedirectToHttpsPermanent` to redirect insecure requests to the same host and path with secure HTTPS protocol (`https://` on port 443).</span></span> <span data-ttu-id="4f9f2-204">中间件将状态代码设置为“301 (永久移动)”。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-204">The middleware sets the status code to 301 (Moved Permanently).</span></span>

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRedirectToHttpsPermanent();

    app.UseRewriter(options);
}
```

> [!NOTE]
> <span data-ttu-id="4f9f2-205">当重定向到 HTTPS 并且不需要其他重定向规则时，建议使用 HTTPS 重定向中间件。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-205">When redirecting to HTTPS without the requirement for additional redirect rules, we recommend using HTTPS Redirection Middleware.</span></span> <span data-ttu-id="4f9f2-206">有关详细信息，请参阅[强制使用 HTTPS](xref:security/enforcing-ssl#require-https)主题。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-206">For more information, see the [Enforce HTTPS](xref:security/enforcing-ssl#require-https) topic.</span></span>

<span data-ttu-id="4f9f2-207">示例应用能够演示如何使用 `AddRedirectToHttps` 或 `AddRedirectToHttpsPermanent`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-207">The sample app is capable of demonstrating how to use `AddRedirectToHttps` or `AddRedirectToHttpsPermanent`.</span></span> <span data-ttu-id="4f9f2-208">将扩展方法添加到 `RewriteOptions`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-208">Add the extension method to the `RewriteOptions`.</span></span> <span data-ttu-id="4f9f2-209">在任何 URL 上向应用发出不安全的请求。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-209">Make an insecure request to the app at any URL.</span></span> <span data-ttu-id="4f9f2-210">消除自签名证书不受信任的浏览器安全警告，或创建例外以信任证书。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-210">Dismiss the browser security warning that the self-signed certificate is untrusted or create an exception to trust the certificate.</span></span>

<span data-ttu-id="4f9f2-211">使用 `AddRedirectToHttps(301, 5001)` 的原始请求：`http://localhost:5000/secure`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-211">Original Request using `AddRedirectToHttps(301, 5001)`: `http://localhost:5000/secure`</span></span>

![开发人员工具正跟踪请求和响应的浏览器窗口](url-rewriting/_static/add_redirect_to_https.png)

<span data-ttu-id="4f9f2-213">使用 `AddRedirectToHttpsPermanent` 的原始请求：`http://localhost:5000/secure`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-213">Original Request using `AddRedirectToHttpsPermanent`: `http://localhost:5000/secure`</span></span>

![开发人员工具正跟踪请求和响应的浏览器窗口](url-rewriting/_static/add_redirect_to_https_permanent.png)

### <a name="url-rewrite"></a><span data-ttu-id="4f9f2-215">URL 重写</span><span class="sxs-lookup"><span data-stu-id="4f9f2-215">URL rewrite</span></span>

<span data-ttu-id="4f9f2-216">使用 `AddRewrite` 创建重写 URL 的规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-216">Use `AddRewrite` to create a rule for rewriting URLs.</span></span> <span data-ttu-id="4f9f2-217">第一个参数包含用于匹配传入 URL 路径的正则表达式。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-217">The first parameter contains your regex for matching on the incoming URL path.</span></span> <span data-ttu-id="4f9f2-218">第二个参数是替换字符串。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-218">The second parameter is the replacement string.</span></span> <span data-ttu-id="4f9f2-219">第三个参数 `skipRemainingRules: {true|false}` 指示如果当前规则适用，中间件是否要跳过其他重写规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-219">The third parameter, `skipRemainingRules: {true|false}`, indicates to the middleware whether or not to skip additional rewrite rules if the current rule is applied.</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](url-rewriting/sample/Startup.cs?name=snippet1&highlight=10-11)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .AddRewrite(@"^rewrite-rule/(\d+)/(\d+)", "rewritten?var1=$1&var2=$2", 
            skipRemainingRules: true);

    app.UseRewriter(options);
}
```

::: moniker-end

<span data-ttu-id="4f9f2-220">原始请求：`/rewrite-rule/1234/5678`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-220">Original Request: `/rewrite-rule/1234/5678`</span></span>

![开发人员工具正跟踪请求和响应的浏览器窗口](url-rewriting/_static/add_rewrite.png)

<span data-ttu-id="4f9f2-222">在正则表达式中首先注意到的是表达式开头的脱字号 (`^`)。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-222">The first thing you notice in the regex is the carat (`^`) at the beginning of the expression.</span></span> <span data-ttu-id="4f9f2-223">这意味着匹配从 URL 路径的开头处开始。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-223">This means that matching starts at the beginning of the URL path.</span></span>

<span data-ttu-id="4f9f2-224">在上述采用重定向规则 `redirect-rule/(.*)` 的示例中，正则表达式的开头没有任何脱字号；因此，成功匹配的路径中的任何字符都可以位于 `redirect-rule/` 之前。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-224">In the earlier example with the redirect rule, `redirect-rule/(.*)`, there's no carat at the start of the regex; therefore, any characters may precede `redirect-rule/` in the path for a successful match.</span></span>

| <span data-ttu-id="4f9f2-225">路径</span><span class="sxs-lookup"><span data-stu-id="4f9f2-225">Path</span></span>                               | <span data-ttu-id="4f9f2-226">匹配</span><span class="sxs-lookup"><span data-stu-id="4f9f2-226">Match</span></span> |
| ---------------------------------- | :---: |
| `/redirect-rule/1234/5678`         | <span data-ttu-id="4f9f2-227">是</span><span class="sxs-lookup"><span data-stu-id="4f9f2-227">Yes</span></span>   |
| `/my-cool-redirect-rule/1234/5678` | <span data-ttu-id="4f9f2-228">是</span><span class="sxs-lookup"><span data-stu-id="4f9f2-228">Yes</span></span>   |
| `/anotherredirect-rule/1234/5678`  | <span data-ttu-id="4f9f2-229">是</span><span class="sxs-lookup"><span data-stu-id="4f9f2-229">Yes</span></span>   |

<span data-ttu-id="4f9f2-230">重写规则 `^rewrite-rule/(\d+)/(\d+)` 只能与以 `rewrite-rule/` 开头的路径匹配。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-230">The rewrite rule, `^rewrite-rule/(\d+)/(\d+)`, only matches paths if they start with `rewrite-rule/`.</span></span> <span data-ttu-id="4f9f2-231">请注意以下重写规则和以上重定向规则之间的匹配差异。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-231">Notice the difference in matching between the rewrite rule below and the redirect rule above.</span></span>

| <span data-ttu-id="4f9f2-232">路径</span><span class="sxs-lookup"><span data-stu-id="4f9f2-232">Path</span></span>                              | <span data-ttu-id="4f9f2-233">匹配</span><span class="sxs-lookup"><span data-stu-id="4f9f2-233">Match</span></span> |
| --------------------------------- | :---: |
| `/rewrite-rule/1234/5678`         | <span data-ttu-id="4f9f2-234">是</span><span class="sxs-lookup"><span data-stu-id="4f9f2-234">Yes</span></span>   |
| `/my-cool-rewrite-rule/1234/5678` | <span data-ttu-id="4f9f2-235">否</span><span class="sxs-lookup"><span data-stu-id="4f9f2-235">No</span></span>    |
| `/anotherrewrite-rule/1234/5678`  | <span data-ttu-id="4f9f2-236">否</span><span class="sxs-lookup"><span data-stu-id="4f9f2-236">No</span></span>    |

<span data-ttu-id="4f9f2-237">在表达式的 `^rewrite-rule/` 部分之后，有两个捕获组 `(\d+)/(\d+)`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-237">Following the `^rewrite-rule/` portion of the expression, there are two capture groups, `(\d+)/(\d+)`.</span></span> <span data-ttu-id="4f9f2-238">`\d` 表示与数字匹配。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-238">The `\d` signifies *match a digit (number)*.</span></span> <span data-ttu-id="4f9f2-239">加号 (`+`) 表示与前面的一个或多个字符匹配。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-239">The plus sign (`+`) means *match one or more of the preceding character*.</span></span> <span data-ttu-id="4f9f2-240">因此，URL 必须包含数字加正斜杠加另一个数字的形式。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-240">Therefore, the URL must contain a number followed by a forward-slash followed by another number.</span></span> <span data-ttu-id="4f9f2-241">这些捕获组以 `$1` 和 `$2` 的形式注入重写 URL 中。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-241">These capture groups are injected into the rewritten URL as `$1` and `$2`.</span></span> <span data-ttu-id="4f9f2-242">重写规则替换字符串将捕获组放入查询字符串中。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-242">The rewrite rule replacement string places the captured groups into the querystring.</span></span> <span data-ttu-id="4f9f2-243">重写 `/rewrite-rule/1234/5678` 的请求路径，获取 `/rewritten?var1=1234&var2=5678` 处的资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-243">The requested path of `/rewrite-rule/1234/5678` is rewritten to obtain the resource at `/rewritten?var1=1234&var2=5678`.</span></span> <span data-ttu-id="4f9f2-244">如果原始请求中存在查询字符串，则重写 URL 时会保留此字符串。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-244">If a querystring is present on the original request, it's preserved when the URL is rewritten.</span></span>

<span data-ttu-id="4f9f2-245">无需往返服务器来获取资源。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-245">There's no roundtrip to the server to obtain the resource.</span></span> <span data-ttu-id="4f9f2-246">如果资源存在，系统会提取资源并以“200 (正常)”状态代码返回给客户端。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-246">If the resource exists, it's fetched and returned to the client with a 200 (OK) status code.</span></span> <span data-ttu-id="4f9f2-247">因为客户端不会被重定向，所以浏览器地址栏中的 URL 不会发生更改。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-247">Because the client isn't redirected, the URL in the browser address bar doesn't change.</span></span> <span data-ttu-id="4f9f2-248">对客户端来说，从未发生过 URL 重写操作。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-248">As far as the client is concerned, the URL rewrite operation never occurred.</span></span>

> [!NOTE]
> <span data-ttu-id="4f9f2-249">尽可能使用 `skipRemainingRules: true`，因为匹配规则代价高昂且会减少应用响应时间。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-249">Use `skipRemainingRules: true` whenever possible, because matching rules is an expensive process and reduces app response time.</span></span> <span data-ttu-id="4f9f2-250">对于最快的应用响应：</span><span class="sxs-lookup"><span data-stu-id="4f9f2-250">For the fastest app response:</span></span>
> * <span data-ttu-id="4f9f2-251">按照从最频繁匹配的规则到最不频繁匹配的规则排列重写规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-251">Order your rewrite rules from the most frequently matched rule to the least frequently matched rule.</span></span>
> * <span data-ttu-id="4f9f2-252">如果出现匹配项且无需处理任何其他规则，则跳过剩余规则的处理。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-252">Skip the processing of the remaining rules when a match occurs and no additional rule processing is required.</span></span>

### <a name="apache-modrewrite"></a><span data-ttu-id="4f9f2-253">Apache mod_rewrite</span><span class="sxs-lookup"><span data-stu-id="4f9f2-253">Apache mod_rewrite</span></span>

<span data-ttu-id="4f9f2-254">使用 `AddApacheModRewrite` 应用 Apache mod_rewrite 规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-254">Apply Apache mod_rewrite rules with `AddApacheModRewrite`.</span></span> <span data-ttu-id="4f9f2-255">请确保将规则文件与应用一起部署。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-255">Make sure that the rules file is deployed with the app.</span></span> <span data-ttu-id="4f9f2-256">有关 mod_rewrite 规则的详细信息和示例，请参阅 [Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/)。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-256">For more information and examples of mod_rewrite rules, see [Apache mod_rewrite](https://httpd.apache.org/docs/2.4/rewrite/).</span></span>

::: moniker range=">= aspnetcore-2.0"

<span data-ttu-id="4f9f2-257">`StreamReader` 用于读取 ApacheModRewrite.txt 规则文件中的规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-257">A `StreamReader` is used to read the rules from the *ApacheModRewrite.txt* rules file.</span></span>

[!code-csharp[](url-rewriting/sample/Startup.cs?name=snippet1&highlight=3-4,12)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

<span data-ttu-id="4f9f2-258">第一个参数采用 `IFileProvider`，后者可通过[依赖关系注入](dependency-injection.md)提供。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-258">The first parameter takes an `IFileProvider`, which is provided via [Dependency Injection](dependency-injection.md).</span></span> <span data-ttu-id="4f9f2-259">注入 `IHostingEnvironment` 以提供 `ContentRootFileProvider`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-259">The `IHostingEnvironment` is injected to provide the `ContentRootFileProvider`.</span></span> <span data-ttu-id="4f9f2-260">第二个参数是规则文件（即示例应用中的 ApacheModRewrite.txt）的路径。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-260">The second parameter is the path to your rules file, which is *ApacheModRewrite.txt* in the sample app.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    var options = new RewriteOptions()
        .AddApacheModRewrite(env.ContentRootFileProvider, "ApacheModRewrite.txt");

    app.UseRewriter(options);
}
```

::: moniker-end

<span data-ttu-id="4f9f2-261">示例应用将请求从 `/apache-mod-rules-redirect/(.\*)` 重定向到 `/redirected?id=$1`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-261">The sample app redirects requests from `/apache-mod-rules-redirect/(.\*)` to `/redirected?id=$1`.</span></span> <span data-ttu-id="4f9f2-262">响应状态代码为“302 (已找到)”。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-262">The response status code is 302 (Found).</span></span>

[!code[](url-rewriting/sample/ApacheModRewrite.txt)]

<span data-ttu-id="4f9f2-263">原始请求：`/apache-mod-rules-redirect/1234`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-263">Original Request: `/apache-mod-rules-redirect/1234`</span></span>

![开发人员工具正跟踪请求和响应的浏览器窗口](url-rewriting/_static/add_apache_mod_redirect.png)

<span data-ttu-id="4f9f2-265">中间件支持下列 Apache mod_rewrite 服务器变量：</span><span class="sxs-lookup"><span data-stu-id="4f9f2-265">The middleware supports the following Apache mod_rewrite server variables:</span></span>

* <span data-ttu-id="4f9f2-266">CONN_REMOTE_ADDR</span><span class="sxs-lookup"><span data-stu-id="4f9f2-266">CONN_REMOTE_ADDR</span></span>
* <span data-ttu-id="4f9f2-267">HTTP_ACCEPT</span><span class="sxs-lookup"><span data-stu-id="4f9f2-267">HTTP_ACCEPT</span></span>
* <span data-ttu-id="4f9f2-268">HTTP_CONNECTION</span><span class="sxs-lookup"><span data-stu-id="4f9f2-268">HTTP_CONNECTION</span></span>
* <span data-ttu-id="4f9f2-269">HTTP_COOKIE</span><span class="sxs-lookup"><span data-stu-id="4f9f2-269">HTTP_COOKIE</span></span>
* <span data-ttu-id="4f9f2-270">HTTP_FORWARDED</span><span class="sxs-lookup"><span data-stu-id="4f9f2-270">HTTP_FORWARDED</span></span>
* <span data-ttu-id="4f9f2-271">HTTP_HOST</span><span class="sxs-lookup"><span data-stu-id="4f9f2-271">HTTP_HOST</span></span>
* <span data-ttu-id="4f9f2-272">HTTP_REFERER</span><span class="sxs-lookup"><span data-stu-id="4f9f2-272">HTTP_REFERER</span></span>
* <span data-ttu-id="4f9f2-273">HTTP_USER_AGENT</span><span class="sxs-lookup"><span data-stu-id="4f9f2-273">HTTP_USER_AGENT</span></span>
* <span data-ttu-id="4f9f2-274">HTTPS</span><span class="sxs-lookup"><span data-stu-id="4f9f2-274">HTTPS</span></span>
* <span data-ttu-id="4f9f2-275">IPV6</span><span class="sxs-lookup"><span data-stu-id="4f9f2-275">IPV6</span></span>
* <span data-ttu-id="4f9f2-276">QUERY_STRING</span><span class="sxs-lookup"><span data-stu-id="4f9f2-276">QUERY_STRING</span></span>
* <span data-ttu-id="4f9f2-277">REMOTE_ADDR</span><span class="sxs-lookup"><span data-stu-id="4f9f2-277">REMOTE_ADDR</span></span>
* <span data-ttu-id="4f9f2-278">REMOTE_PORT</span><span class="sxs-lookup"><span data-stu-id="4f9f2-278">REMOTE_PORT</span></span>
* <span data-ttu-id="4f9f2-279">REQUEST_FILENAME</span><span class="sxs-lookup"><span data-stu-id="4f9f2-279">REQUEST_FILENAME</span></span>
* <span data-ttu-id="4f9f2-280">REQUEST_METHOD</span><span class="sxs-lookup"><span data-stu-id="4f9f2-280">REQUEST_METHOD</span></span>
* <span data-ttu-id="4f9f2-281">REQUEST_SCHEME</span><span class="sxs-lookup"><span data-stu-id="4f9f2-281">REQUEST_SCHEME</span></span>
* <span data-ttu-id="4f9f2-282">REQUEST_URI</span><span class="sxs-lookup"><span data-stu-id="4f9f2-282">REQUEST_URI</span></span>
* <span data-ttu-id="4f9f2-283">SCRIPT_FILENAME</span><span class="sxs-lookup"><span data-stu-id="4f9f2-283">SCRIPT_FILENAME</span></span>
* <span data-ttu-id="4f9f2-284">SERVER_ADDR</span><span class="sxs-lookup"><span data-stu-id="4f9f2-284">SERVER_ADDR</span></span>
* <span data-ttu-id="4f9f2-285">SERVER_PORT</span><span class="sxs-lookup"><span data-stu-id="4f9f2-285">SERVER_PORT</span></span>
* <span data-ttu-id="4f9f2-286">SERVER_PROTOCOL</span><span class="sxs-lookup"><span data-stu-id="4f9f2-286">SERVER_PROTOCOL</span></span>
* <span data-ttu-id="4f9f2-287">TIME</span><span class="sxs-lookup"><span data-stu-id="4f9f2-287">TIME</span></span>
* <span data-ttu-id="4f9f2-288">TIME_DAY</span><span class="sxs-lookup"><span data-stu-id="4f9f2-288">TIME_DAY</span></span>
* <span data-ttu-id="4f9f2-289">TIME_HOUR</span><span class="sxs-lookup"><span data-stu-id="4f9f2-289">TIME_HOUR</span></span>
* <span data-ttu-id="4f9f2-290">TIME_MIN</span><span class="sxs-lookup"><span data-stu-id="4f9f2-290">TIME_MIN</span></span>
* <span data-ttu-id="4f9f2-291">TIME_MON</span><span class="sxs-lookup"><span data-stu-id="4f9f2-291">TIME_MON</span></span>
* <span data-ttu-id="4f9f2-292">TIME_SEC</span><span class="sxs-lookup"><span data-stu-id="4f9f2-292">TIME_SEC</span></span>
* <span data-ttu-id="4f9f2-293">TIME_WDAY</span><span class="sxs-lookup"><span data-stu-id="4f9f2-293">TIME_WDAY</span></span>
* <span data-ttu-id="4f9f2-294">TIME_YEAR</span><span class="sxs-lookup"><span data-stu-id="4f9f2-294">TIME_YEAR</span></span>

### <a name="iis-url-rewrite-module-rules"></a><span data-ttu-id="4f9f2-295">IIS URL 重写模块规则</span><span class="sxs-lookup"><span data-stu-id="4f9f2-295">IIS URL Rewrite Module rules</span></span>

<span data-ttu-id="4f9f2-296">要使用适用于 IIS URL 重写模块的规则，请使用 `AddIISUrlRewrite`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-296">To use rules that apply to the IIS URL Rewrite Module, use `AddIISUrlRewrite`.</span></span> <span data-ttu-id="4f9f2-297">请确保将规则文件与应用一起部署。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-297">Make sure that the rules file is deployed with the app.</span></span> <span data-ttu-id="4f9f2-298">当 web.config 文件在 Windows Server IIS 上运行时，请勿指示中间件使用该文件。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-298">Don't direct the middleware to use your *web.config* file when running on Windows Server IIS.</span></span> <span data-ttu-id="4f9f2-299">使用 IIS 时，应将这些规则存储在 web.config 之外，避免与 IIS 重写模块发生冲突。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-299">With IIS, these rules should be stored outside of your *web.config* to avoid conflicts with the IIS Rewrite module.</span></span> <span data-ttu-id="4f9f2-300">有关 IIS URL 重写模块规则的详细信息和示例，请参阅 [Using Url Rewrite Module 2.0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20)（使用 URL 重写模块 2.0）和 [URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)（URL 重写模块配置引用）。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-300">For more information and examples of IIS URL Rewrite Module rules, see [Using Url Rewrite Module 2.0](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20) and [URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference).</span></span>

::: moniker range=">= aspnetcore-2.0"

<span data-ttu-id="4f9f2-301">`StreamReader` 用于读取 IISUrlRewrite.xml 规则文件中的规则。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-301">A `StreamReader` is used to read the rules from the *IISUrlRewrite.xml* rules file.</span></span>

[!code-csharp[](url-rewriting/sample/Startup.cs?name=snippet1&highlight=5-6,13)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

<span data-ttu-id="4f9f2-302">第一个参数采用 `IFileProvider`，而第二个参数是 XML 规则文件（即示例应用中的 IISUrlRewrite.xml）的路径。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-302">The first parameter takes an `IFileProvider`, while the second parameter is the path to your XML rules file, which is *IISUrlRewrite.xml* in the sample app.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    var options = new RewriteOptions()
        .AddIISUrlRewrite(env.ContentRootFileProvider, "IISUrlRewrite.xml");

    app.UseRewriter(options);
}
```

::: moniker-end

<span data-ttu-id="4f9f2-303">示例应用将请求从 `/iis-rules-rewrite/(.*)` 重写为 `/rewritten?id=$1`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-303">The sample app rewrites requests from `/iis-rules-rewrite/(.*)` to `/rewritten?id=$1`.</span></span> <span data-ttu-id="4f9f2-304">以“200 (正常)”状态代码作为响应发送到客户端。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-304">The response is sent to the client with a 200 (OK) status code.</span></span>

[!code-xml[](url-rewriting/sample/IISUrlRewrite.xml)]

<span data-ttu-id="4f9f2-305">原始请求：`/iis-rules-rewrite/1234`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-305">Original Request: `/iis-rules-rewrite/1234`</span></span>

![开发人员工具正跟踪请求和响应的浏览器窗口](url-rewriting/_static/add_iis_url_rewrite.png)

<span data-ttu-id="4f9f2-307">如果有配置了服务器级别规则（可对应用产生不利影响）的活动 IIS 重写模块，则可禁用应用的 IIS 重写模块。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-307">If you have an active IIS Rewrite Module with server-level rules configured that would impact your app in undesirable ways, you can disable the IIS Rewrite Module for an app.</span></span> <span data-ttu-id="4f9f2-308">有关详细信息，请参阅[禁用 IIS 模块](xref:host-and-deploy/iis/modules#disabling-iis-modules)。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-308">For more information, see [Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules).</span></span>

#### <a name="unsupported-features"></a><span data-ttu-id="4f9f2-309">不支持的功能</span><span class="sxs-lookup"><span data-stu-id="4f9f2-309">Unsupported features</span></span>

::: moniker range=">= aspnetcore-2.0"

<span data-ttu-id="4f9f2-310">与 ASP.NET Core 2.x 一同发布的中间件不支持以下 IIS URL 重写模块功能：</span><span class="sxs-lookup"><span data-stu-id="4f9f2-310">The middleware released with ASP.NET Core 2.x doesn't support the following IIS URL Rewrite Module features:</span></span>

* <span data-ttu-id="4f9f2-311">出站规则</span><span class="sxs-lookup"><span data-stu-id="4f9f2-311">Outbound Rules</span></span>
* <span data-ttu-id="4f9f2-312">自定义服务器变量</span><span class="sxs-lookup"><span data-stu-id="4f9f2-312">Custom Server Variables</span></span>
* <span data-ttu-id="4f9f2-313">通配符</span><span class="sxs-lookup"><span data-stu-id="4f9f2-313">Wildcards</span></span>
* <span data-ttu-id="4f9f2-314">LogRewrittenUrl</span><span class="sxs-lookup"><span data-stu-id="4f9f2-314">LogRewrittenUrl</span></span>

::: moniker-end

::: moniker range="< aspnetcore-2.0"

<span data-ttu-id="4f9f2-315">与 ASP.NET Core 1.x 一同发布的中间件不支持以下 IIS URL 重写模块功能：</span><span class="sxs-lookup"><span data-stu-id="4f9f2-315">The middleware released with ASP.NET Core 1.x doesn't support the following IIS URL Rewrite Module features:</span></span>

* <span data-ttu-id="4f9f2-316">全局规则</span><span class="sxs-lookup"><span data-stu-id="4f9f2-316">Global Rules</span></span>
* <span data-ttu-id="4f9f2-317">出站规则</span><span class="sxs-lookup"><span data-stu-id="4f9f2-317">Outbound Rules</span></span>
* <span data-ttu-id="4f9f2-318">重写映射</span><span class="sxs-lookup"><span data-stu-id="4f9f2-318">Rewrite Maps</span></span>
* <span data-ttu-id="4f9f2-319">CustomResponse 操作</span><span class="sxs-lookup"><span data-stu-id="4f9f2-319">CustomResponse Action</span></span>
* <span data-ttu-id="4f9f2-320">自定义服务器变量</span><span class="sxs-lookup"><span data-stu-id="4f9f2-320">Custom Server Variables</span></span>
* <span data-ttu-id="4f9f2-321">通配符</span><span class="sxs-lookup"><span data-stu-id="4f9f2-321">Wildcards</span></span>
* <span data-ttu-id="4f9f2-322">Action:CustomResponse</span><span class="sxs-lookup"><span data-stu-id="4f9f2-322">Action:CustomResponse</span></span>
* <span data-ttu-id="4f9f2-323">LogRewrittenUrl</span><span class="sxs-lookup"><span data-stu-id="4f9f2-323">LogRewrittenUrl</span></span>

::: moniker-end

#### <a name="supported-server-variables"></a><span data-ttu-id="4f9f2-324">受支持的服务器变量</span><span class="sxs-lookup"><span data-stu-id="4f9f2-324">Supported server variables</span></span>

<span data-ttu-id="4f9f2-325">中间件支持下列 IIS URL 重写模块服务器变量：</span><span class="sxs-lookup"><span data-stu-id="4f9f2-325">The middleware supports the following IIS URL Rewrite Module server variables:</span></span>

* <span data-ttu-id="4f9f2-326">CONTENT_LENGTH</span><span class="sxs-lookup"><span data-stu-id="4f9f2-326">CONTENT_LENGTH</span></span>
* <span data-ttu-id="4f9f2-327">CONTENT_TYPE</span><span class="sxs-lookup"><span data-stu-id="4f9f2-327">CONTENT_TYPE</span></span>
* <span data-ttu-id="4f9f2-328">HTTP_ACCEPT</span><span class="sxs-lookup"><span data-stu-id="4f9f2-328">HTTP_ACCEPT</span></span>
* <span data-ttu-id="4f9f2-329">HTTP_CONNECTION</span><span class="sxs-lookup"><span data-stu-id="4f9f2-329">HTTP_CONNECTION</span></span>
* <span data-ttu-id="4f9f2-330">HTTP_COOKIE</span><span class="sxs-lookup"><span data-stu-id="4f9f2-330">HTTP_COOKIE</span></span>
* <span data-ttu-id="4f9f2-331">HTTP_HOST</span><span class="sxs-lookup"><span data-stu-id="4f9f2-331">HTTP_HOST</span></span>
* <span data-ttu-id="4f9f2-332">HTTP_REFERER</span><span class="sxs-lookup"><span data-stu-id="4f9f2-332">HTTP_REFERER</span></span>
* <span data-ttu-id="4f9f2-333">HTTP_URL</span><span class="sxs-lookup"><span data-stu-id="4f9f2-333">HTTP_URL</span></span>
* <span data-ttu-id="4f9f2-334">HTTP_USER_AGENT</span><span class="sxs-lookup"><span data-stu-id="4f9f2-334">HTTP_USER_AGENT</span></span>
* <span data-ttu-id="4f9f2-335">HTTPS</span><span class="sxs-lookup"><span data-stu-id="4f9f2-335">HTTPS</span></span>
* <span data-ttu-id="4f9f2-336">LOCAL_ADDR</span><span class="sxs-lookup"><span data-stu-id="4f9f2-336">LOCAL_ADDR</span></span>
* <span data-ttu-id="4f9f2-337">QUERY_STRING</span><span class="sxs-lookup"><span data-stu-id="4f9f2-337">QUERY_STRING</span></span>
* <span data-ttu-id="4f9f2-338">REMOTE_ADDR</span><span class="sxs-lookup"><span data-stu-id="4f9f2-338">REMOTE_ADDR</span></span>
* <span data-ttu-id="4f9f2-339">REMOTE_PORT</span><span class="sxs-lookup"><span data-stu-id="4f9f2-339">REMOTE_PORT</span></span>
* <span data-ttu-id="4f9f2-340">REQUEST_FILENAME</span><span class="sxs-lookup"><span data-stu-id="4f9f2-340">REQUEST_FILENAME</span></span>
* <span data-ttu-id="4f9f2-341">REQUEST_URI</span><span class="sxs-lookup"><span data-stu-id="4f9f2-341">REQUEST_URI</span></span>

> [!NOTE]
> <span data-ttu-id="4f9f2-342">也可通过 `PhysicalFileProvider` 获取 `IFileProvider`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-342">You can also obtain an `IFileProvider` via a `PhysicalFileProvider`.</span></span> <span data-ttu-id="4f9f2-343">这种方法可为重写规则文件的位置提供更大的灵活性。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-343">This approach may provide greater flexibility for the location of your rewrite rules files.</span></span> <span data-ttu-id="4f9f2-344">请确保将重写规则文件部署到所提供路径的服务器中。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-344">Make sure that your rewrite rules files are deployed to the server at the path you provide.</span></span>
> ```csharp
> PhysicalFileProvider fileProvider = new PhysicalFileProvider(Directory.GetCurrentDirectory());
> ```

### <a name="method-based-rule"></a><span data-ttu-id="4f9f2-345">基于方法的规则</span><span class="sxs-lookup"><span data-stu-id="4f9f2-345">Method-based rule</span></span>

<span data-ttu-id="4f9f2-346">使用 `Add(Action<RewriteContext> applyRule)` 在方法中实现自己的规则逻辑。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-346">Use `Add(Action<RewriteContext> applyRule)` to implement your own rule logic in a method.</span></span> <span data-ttu-id="4f9f2-347">为了在方法中使用 `HttpContext`，`RewriteContext` 会将其公开。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-347">The `RewriteContext` exposes the `HttpContext` for use in your method.</span></span> <span data-ttu-id="4f9f2-348">`context.Result` 决定如何处理其他管道进程。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-348">The `context.Result` determines how additional pipeline processing is handled.</span></span>

| <span data-ttu-id="4f9f2-349">context.Result</span><span class="sxs-lookup"><span data-stu-id="4f9f2-349">context.Result</span></span>                       | <span data-ttu-id="4f9f2-350">操作</span><span class="sxs-lookup"><span data-stu-id="4f9f2-350">Action</span></span>                                                          |
| ------------------------------------ | --------------------------------------------------------------- |
| <span data-ttu-id="4f9f2-351">`RuleResult.ContinueRules`（默认值）</span><span class="sxs-lookup"><span data-stu-id="4f9f2-351">`RuleResult.ContinueRules` (default)</span></span> | <span data-ttu-id="4f9f2-352">继续应用规则</span><span class="sxs-lookup"><span data-stu-id="4f9f2-352">Continue applying rules</span></span>                                         |
| `RuleResult.EndResponse`             | <span data-ttu-id="4f9f2-353">停止应用规则并发送响应</span><span class="sxs-lookup"><span data-stu-id="4f9f2-353">Stop applying rules and send the response</span></span>                       |
| `RuleResult.SkipRemainingRules`      | <span data-ttu-id="4f9f2-354">停止应用规则并将上下文发送给下一个中间件</span><span class="sxs-lookup"><span data-stu-id="4f9f2-354">Stop applying rules and send the context to the next middleware</span></span> |

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](url-rewriting/sample/Startup.cs?name=snippet1&highlight=14)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .Add(RedirectXMLRequests);

    app.UseRewriter(options);
}
```

::: moniker-end

<span data-ttu-id="4f9f2-355">示例应用演示了如何对以 .xml 结尾的路径的请求进行重定向。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-355">The sample app demonstrates a method that redirects requests for paths that end with *.xml*.</span></span> <span data-ttu-id="4f9f2-356">如果为 `/file.xml` 发出请求，则它将重定向到 `/xmlfiles/file.xml`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-356">If you make a request for `/file.xml`, it's redirected to `/xmlfiles/file.xml`.</span></span> <span data-ttu-id="4f9f2-357">状态代码设置为“301 (永久移动)”。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-357">The status code is set to 301 (Moved Permanently).</span></span> <span data-ttu-id="4f9f2-358">对于重定向，必须明确设置响应的状态代码；否则会返回 “200 (正常)”状态代码，且在客户端上不会进行重定向。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-358">For a redirect, you must explicitly set the status code of the response; otherwise, a 200 (OK) status code is returned and the redirect won't occur on the client.</span></span>

[!code-csharp[](url-rewriting/sample/RewriteRules.cs?name=snippet1)]

<span data-ttu-id="4f9f2-359">原始请求：`/file.xml`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-359">Original Request: `/file.xml`</span></span>

![开发人员工具正跟踪 file.xml 的请求和响应的浏览器窗口](url-rewriting/_static/add_redirect_xml_requests.png)

### <a name="irule-based-rule"></a><span data-ttu-id="4f9f2-361">基于 IRule 的规则</span><span class="sxs-lookup"><span data-stu-id="4f9f2-361">IRule-based rule</span></span>

<span data-ttu-id="4f9f2-362">使用 `Add(IRule)` 在派生自 `IRule` 的类中实现自己的规则逻辑。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-362">Use `Add(IRule)` to implement your own rule logic in a class that derives from `IRule`.</span></span> <span data-ttu-id="4f9f2-363">通过 `IRule` 为使用基于方法的规则方式提供更大的灵活性。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-363">Using an `IRule` provides greater flexibility over using the method-based rule approach.</span></span> <span data-ttu-id="4f9f2-364">派生类可能包含构造函数，你可在其中传入 `ApplyRule` 方法的参数。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-364">Your derived class may include a constructor, where you can pass in parameters for the `ApplyRule` method.</span></span>

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](url-rewriting/sample/Startup.cs?name=snippet1&highlight=15-16)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    var options = new RewriteOptions()
        .Add(new RedirectImageRequests(".png", "/png-images"))
        .Add(new RedirectImageRequests(".jpg", "/jpg-images"));

    app.UseRewriter(options);
}
```

::: moniker-end

<span data-ttu-id="4f9f2-365">检查示例应用中 `extension` 和 `newPath` 的参数值是否符合多个条件。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-365">The values of the parameters in the sample app for the `extension` and the `newPath` are checked to meet several conditions.</span></span> <span data-ttu-id="4f9f2-366">`extension` 须包含一个值，并且该值必须是 .png、.jpg 或 .gif。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-366">The `extension` must contain a value, and the value must be *.png*, *.jpg*, or *.gif*.</span></span> <span data-ttu-id="4f9f2-367">如果 `newPath` 无效，则会引发 `ArgumentException`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-367">If the `newPath` isn't valid, an `ArgumentException` is thrown.</span></span> <span data-ttu-id="4f9f2-368">如果为 image.png 发出请求，则它将重定向到 `/png-images/image.png`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-368">If you make a request for *image.png*, it's redirected to `/png-images/image.png`.</span></span> <span data-ttu-id="4f9f2-369">如果为 image.jpg 发出请求，则它将重定向到 `/jpg-images/image.jpg`。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-369">If you make a request for *image.jpg*, it's redirected to `/jpg-images/image.jpg`.</span></span> <span data-ttu-id="4f9f2-370">状态代码设置为“301 (永久移动)”，`context.Result` 设置为停止处理规则并发送响应。</span><span class="sxs-lookup"><span data-stu-id="4f9f2-370">The status code is set to 301 (Moved Permanently), and the `context.Result` is set to stop processing rules and send the response.</span></span>

[!code-csharp[](url-rewriting/sample/RewriteRules.cs?name=snippet2)]

<span data-ttu-id="4f9f2-371">原始请求：`/image.png`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-371">Original Request: `/image.png`</span></span>

![开发人员工具正跟踪 image.png 的请求和响应的浏览器窗口](url-rewriting/_static/add_redirect_png_requests.png)

<span data-ttu-id="4f9f2-373">原始请求：`/image.jpg`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-373">Original Request: `/image.jpg`</span></span>

![开发人员工具正跟踪 image.jpg 的请求和响应的浏览器窗口](url-rewriting/_static/add_redirect_jpg_requests.png)

## <a name="regex-examples"></a><span data-ttu-id="4f9f2-375">正则表达式示例</span><span class="sxs-lookup"><span data-stu-id="4f9f2-375">Regex examples</span></span>

| <span data-ttu-id="4f9f2-376">目标</span><span class="sxs-lookup"><span data-stu-id="4f9f2-376">Goal</span></span> | <span data-ttu-id="4f9f2-377">正则表达式字符串和</span><span class="sxs-lookup"><span data-stu-id="4f9f2-377">Regex String &</span></span><br><span data-ttu-id="4f9f2-378">匹配示例</span><span class="sxs-lookup"><span data-stu-id="4f9f2-378">Match Example</span></span> | <span data-ttu-id="4f9f2-379">替换字符串和</span><span class="sxs-lookup"><span data-stu-id="4f9f2-379">Replacement String &</span></span><br><span data-ttu-id="4f9f2-380">输出示例</span><span class="sxs-lookup"><span data-stu-id="4f9f2-380">Output Example</span></span> |
| ---- | :-----------------------------: | :------------------------------------: |
| <span data-ttu-id="4f9f2-381">将路径重写为查询字符串</span><span class="sxs-lookup"><span data-stu-id="4f9f2-381">Rewrite path into querystring</span></span> | `^path/(.*)/(.*)`<br>`/path/abc/123` | `path?var1=$1&var2=$2`<br>`/path?var1=abc&var2=123` |
| <span data-ttu-id="4f9f2-382">去掉尾部反斜杠</span><span class="sxs-lookup"><span data-stu-id="4f9f2-382">Strip trailing slash</span></span> | `(.*)/$`<br>`/path/` | `$1`<br>`/path` |
| <span data-ttu-id="4f9f2-383">强制添加尾部反斜杠</span><span class="sxs-lookup"><span data-stu-id="4f9f2-383">Enforce trailing slash</span></span> | `(.*[^/])$`<br>`/path` | `$1/`<br>`/path/` |
| <span data-ttu-id="4f9f2-384">避免重写特定请求</span><span class="sxs-lookup"><span data-stu-id="4f9f2-384">Avoid rewriting specific requests</span></span> | <span data-ttu-id="4f9f2-385">`^(.*)(?<!\.axd)$` 或 `^(?!.*\.axd$)(.*)$`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-385">`^(.*)(?<!\.axd)$` or `^(?!.*\.axd$)(.*)$`</span></span><br><span data-ttu-id="4f9f2-386">正确：`/resource.htm`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-386">Yes: `/resource.htm`</span></span><br><span data-ttu-id="4f9f2-387">错误：`/resource.axd`</span><span class="sxs-lookup"><span data-stu-id="4f9f2-387">No: `/resource.axd`</span></span> | `rewritten/$1`<br>`/rewritten/resource.htm`<br>`/resource.axd` |
| <span data-ttu-id="4f9f2-388">重新排列 URL 段</span><span class="sxs-lookup"><span data-stu-id="4f9f2-388">Rearrange URL segments</span></span> | `path/(.*)/(.*)/(.*)`<br>`path/1/2/3` | `path/$3/$2/$1`<br>`path/3/2/1` |
| <span data-ttu-id="4f9f2-389">替换 URL 段</span><span class="sxs-lookup"><span data-stu-id="4f9f2-389">Replace a URL segment</span></span> | `^(.*)/segment2/(.*)`<br>`/segment1/segment2/segment3` | `$1/replaced/$2`<br>`/segment1/replaced/segment3` |

## <a name="additional-resources"></a><span data-ttu-id="4f9f2-390">其他资源</span><span class="sxs-lookup"><span data-stu-id="4f9f2-390">Additional resources</span></span>

* [<span data-ttu-id="4f9f2-391">应用程序启动</span><span class="sxs-lookup"><span data-stu-id="4f9f2-391">Application Startup</span></span>](startup.md)
* [<span data-ttu-id="4f9f2-392">中间件</span><span class="sxs-lookup"><span data-stu-id="4f9f2-392">Middleware</span></span>](xref:fundamentals/middleware/index)
* [<span data-ttu-id="4f9f2-393">.NET 中的正则表达式</span><span class="sxs-lookup"><span data-stu-id="4f9f2-393">Regular expressions in .NET</span></span>](/dotnet/articles/standard/base-types/regular-expressions)
* [<span data-ttu-id="4f9f2-394">正则表达式语言 - 快速参考</span><span class="sxs-lookup"><span data-stu-id="4f9f2-394">Regular expression language - quick reference</span></span>](/dotnet/articles/standard/base-types/quick-ref)
* [<span data-ttu-id="4f9f2-395">Apache mod_rewrite</span><span class="sxs-lookup"><span data-stu-id="4f9f2-395">Apache mod_rewrite</span></span>](https://httpd.apache.org/docs/2.4/rewrite/)
* [<span data-ttu-id="4f9f2-396">使用 URL 重写模块 2.0（适用于 IIS）</span><span class="sxs-lookup"><span data-stu-id="4f9f2-396">Using Url Rewrite Module 2.0 (for IIS)</span></span>](/iis/extensions/url-rewrite-module/using-url-rewrite-module-20)
* <span data-ttu-id="4f9f2-397">[URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)（URL 重写模块配置引用）</span><span class="sxs-lookup"><span data-stu-id="4f9f2-397">[URL Rewrite Module Configuration Reference](/iis/extensions/url-rewrite-module/url-rewrite-module-configuration-reference)</span></span>
* [<span data-ttu-id="4f9f2-398">IIS URL 重写模块论坛</span><span class="sxs-lookup"><span data-stu-id="4f9f2-398">IIS URL Rewrite Module Forum</span></span>](https://forums.iis.net/1152.aspx)
* <span data-ttu-id="4f9f2-399">[Keep a simple URL structure](https://support.google.com/webmasters/answer/76329?hl=en)（保持简单的 URL 结构）</span><span class="sxs-lookup"><span data-stu-id="4f9f2-399">[Keep a simple URL structure](https://support.google.com/webmasters/answer/76329?hl=en)</span></span>
* <span data-ttu-id="4f9f2-400">[10 URL Rewriting Tips and Tricks](http://ruslany.net/2009/04/10-url-rewriting-tips-and-tricks/)（10 个 URL 重写提示和技巧）</span><span class="sxs-lookup"><span data-stu-id="4f9f2-400">[10 URL Rewriting Tips and Tricks](http://ruslany.net/2009/04/10-url-rewriting-tips-and-tricks/)</span></span>
* <span data-ttu-id="4f9f2-401">[To slash or not to slash](https://webmasters.googleblog.com/2010/04/to-slash-or-not-to-slash.html)（用斜杠或不用斜杠）</span><span class="sxs-lookup"><span data-stu-id="4f9f2-401">[To slash or not to slash](https://webmasters.googleblog.com/2010/04/to-slash-or-not-to-slash.html)</span></span>
