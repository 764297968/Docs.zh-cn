---
title: 将新字段添加到 ASP.NET Core MVC 应用
author: rick-anderson
description: 了解如何使用 Entity Framework Code First 迁移将新字段添加到模型，并将此更改迁移到数据库。
ms.author: riande
ms.custom: mvc
ms.date: 10/24/2018
uid: tutorials/first-mvc-app/new-field
ms.openlocfilehash: e58d5af90b997c66cb749ab8f1b2f8049b8f7303
ms.sourcegitcommit: 4d74644f11e0dac52b4510048490ae731c691496
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50089676"
---
# <a name="add-a-new-field-to-an-aspnet-core-mvc-app"></a><span data-ttu-id="7ac92-103">将新字段添加到 ASP.NET Core MVC 应用</span><span class="sxs-lookup"><span data-stu-id="7ac92-103">Add a new field to an ASP.NET Core MVC app</span></span>

<span data-ttu-id="7ac92-104">作者：[Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="7ac92-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="7ac92-105">在本部分中，将使用 [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First 迁移将新字段添加到模型，并将此更改迁移到数据库。</span><span class="sxs-lookup"><span data-stu-id="7ac92-105">In this section you'll use [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations to add a new field to the model and migrate that change to the database.</span></span>

<span data-ttu-id="7ac92-106">使用 EF Code First 自动创建数据库时，Code First 会向数据库添加表格，以帮助跟踪数据库的架构是否与从其中生成它的模型类同步。</span><span class="sxs-lookup"><span data-stu-id="7ac92-106">When you use EF Code First to automatically create a database, Code First adds a table to the database to help track whether the schema of the database is in sync with the model classes it was generated from.</span></span> <span data-ttu-id="7ac92-107">如果它们不同步，EF 则会引发异常。</span><span class="sxs-lookup"><span data-stu-id="7ac92-107">If they aren't in sync, EF throws an exception.</span></span> <span data-ttu-id="7ac92-108">这使查找不一致的数据库/代码问题变得更加轻松。</span><span class="sxs-lookup"><span data-stu-id="7ac92-108">This makes it easier to find inconsistent database/code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="7ac92-109">向电影模型添加分级属性</span><span class="sxs-lookup"><span data-stu-id="7ac92-109">Adding a Rating Property to the Movie Model</span></span>

<span data-ttu-id="7ac92-110">打开 Models/Movie.cs 文件，并添加 `Rating` 属性：</span><span class="sxs-lookup"><span data-stu-id="7ac92-110">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie21/Models/MovieDateRating.cs?highlight=13&name=snippet)]

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Models/MovieDateRating.cs?highlight=11&range=7-18)]

::: moniker-end

<span data-ttu-id="7ac92-111">生成应用 (Ctrl+Shift+B)。</span><span class="sxs-lookup"><span data-stu-id="7ac92-111">Build the app (Ctrl+Shift+B).</span></span>

<span data-ttu-id="7ac92-112">因为已经添加新字段到 `Movie` 类，所以还需要更新绑定允许名单，将此新属性纳入其中。</span><span class="sxs-lookup"><span data-stu-id="7ac92-112">Because you've added a new field to the `Movie` class, you also need to update the binding white list so this new property will be included.</span></span> <span data-ttu-id="7ac92-113">在 MoviesController.cs 中，更新 `Create` 和 `Edit` 操作方法的 `[Bind]` 属性，以包括 `Rating` 属性：</span><span class="sxs-lookup"><span data-stu-id="7ac92-113">In *MoviesController.cs*, update the `[Bind]` attribute for both the `Create` and `Edit` action methods to include the `Rating` property:</span></span>

```csharp
[Bind("ID,Title,ReleaseDate,Genre,Price,Rating")]
   ```

<span data-ttu-id="7ac92-114">还需要更新视图模板，以便在浏览器视图中显示、创建和编辑新的 `Rating` 属性。</span><span class="sxs-lookup"><span data-stu-id="7ac92-114">You also need to update the view templates in order to display, create and edit the new `Rating` property in the browser view.</span></span>

<span data-ttu-id="7ac92-115">编辑 /Views/Movies/Index.cshtml 文件并添加 `Rating` 字段：</span><span class="sxs-lookup"><span data-stu-id="7ac92-115">Edit the */Views/Movies/Index.cshtml* file and add a `Rating` field:</span></span>

[!code-HTML[](start-mvc/sample/MvcMovie/Views/Movies/IndexGenreRating.cshtml?highlight=17,39&range=24-64)]

<span data-ttu-id="7ac92-116">使用 `Rating` 字段更新 /Views/Movies/Create.cshtml。</span><span class="sxs-lookup"><span data-stu-id="7ac92-116">Update the */Views/Movies/Create.cshtml* with a `Rating` field.</span></span> <span data-ttu-id="7ac92-117">可以复制/粘贴之前的“窗体组”，并让 intelliSense 帮助更新字段。</span><span class="sxs-lookup"><span data-stu-id="7ac92-117">You can copy/paste the previous "form group" and let intelliSense help you update the fields.</span></span> <span data-ttu-id="7ac92-118">IntelliSense 适用于[标记帮助程序](xref:mvc/views/tag-helpers/intro)。</span><span class="sxs-lookup"><span data-stu-id="7ac92-118">IntelliSense works with [Tag Helpers](xref:mvc/views/tag-helpers/intro).</span></span> <span data-ttu-id="7ac92-119">请注意：在 Visual Studio 2017 的 RTM 版本中，需要安装 Razor intelliSense 的 [Razor 语言服务](https://marketplace.visualstudio.com/items?itemName=ms-madsk.RazorLanguageServices)。</span><span class="sxs-lookup"><span data-stu-id="7ac92-119">Note: In the RTM verison of Visual Studio 2017 you need to install the [Razor Language Services](https://marketplace.visualstudio.com/items?itemName=ms-madsk.RazorLanguageServices) for Razor intelliSense.</span></span> <span data-ttu-id="7ac92-120">此问题将在下一版本中修复。</span><span class="sxs-lookup"><span data-stu-id="7ac92-120">This will be fixed in the next release.</span></span>

![开发人员已在视图的第二个标签元素中键入字母 R 用作 asp-for 的特性值。](new-field/_static/cr.png)

<span data-ttu-id="7ac92-124">在将 DB 更新为包括新字段之前，应用不会正常工作。</span><span class="sxs-lookup"><span data-stu-id="7ac92-124">The app won't work until we update the DB to include the new field.</span></span> <span data-ttu-id="7ac92-125">如果现在运行，会出现以下 `SqlException`：</span><span class="sxs-lookup"><span data-stu-id="7ac92-125">If you run it now, you'll get the following `SqlException`:</span></span>

`SqlException: Invalid column name 'Rating'.`

<span data-ttu-id="7ac92-126">看到此错误是因为更新的 Movie 模型类与现有数据库的 Movie 表架构不同。</span><span class="sxs-lookup"><span data-stu-id="7ac92-126">You're seeing this error because the updated Movie model class is different than the schema of the Movie table of the existing database.</span></span> <span data-ttu-id="7ac92-127">（数据库表中没有“Rating”列。）</span><span class="sxs-lookup"><span data-stu-id="7ac92-127">(There's no Rating column in the database table.)</span></span>

<span data-ttu-id="7ac92-128">可通过几种方法解决此错误：</span><span class="sxs-lookup"><span data-stu-id="7ac92-128">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="7ac92-129">让 Entity Framework 自动丢弃，并基于新的模型类架构重新创建数据库。</span><span class="sxs-lookup"><span data-stu-id="7ac92-129">Have the Entity Framework automatically drop and re-create the database based on the new model class schema.</span></span> <span data-ttu-id="7ac92-130">在测试数据库上进行开发时，此方法在开发周期早期很方便；通过它可以一起快速改进模型和数据库架构。</span><span class="sxs-lookup"><span data-stu-id="7ac92-130">This approach is very convenient early in the development cycle when you are doing active development on a test database; it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="7ac92-131">但其缺点是会丢失数据库中的现有数据 - 因此请勿对生产数据库使用此方法！</span><span class="sxs-lookup"><span data-stu-id="7ac92-131">The downside, though, is that you lose existing data in the database — so you don't want to use this approach on a production database!</span></span> <span data-ttu-id="7ac92-132">使用初始值设定项，以使用测试数据自动设定数据库种子，这通常是开发应用程序的有效方式。</span><span class="sxs-lookup"><span data-stu-id="7ac92-132">Using an initializer to automatically seed a database with test data is often a productive way to develop an application.</span></span>

2. <span data-ttu-id="7ac92-133">对现有数据库架构进行显式修改，使它与模型类相匹配。</span><span class="sxs-lookup"><span data-stu-id="7ac92-133">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="7ac92-134">此方法的优点是可以保留数据。</span><span class="sxs-lookup"><span data-stu-id="7ac92-134">The advantage of this approach is that you keep your data.</span></span> <span data-ttu-id="7ac92-135">可以手动或通过创建数据库更改脚本进行此更改。</span><span class="sxs-lookup"><span data-stu-id="7ac92-135">You can make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="7ac92-136">使用 Code First 迁移更新数据库架构。</span><span class="sxs-lookup"><span data-stu-id="7ac92-136">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="7ac92-137">本教程使用 Code First 迁移。</span><span class="sxs-lookup"><span data-stu-id="7ac92-137">For this tutorial, we'll use Code First Migrations.</span></span>

<span data-ttu-id="7ac92-138">更新 `SeedData` 类，使它提供新列的值。</span><span class="sxs-lookup"><span data-stu-id="7ac92-138">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="7ac92-139">示例更改如下所示，但可能需要对每个 `new Movie` 做出此更改。</span><span class="sxs-lookup"><span data-stu-id="7ac92-139">A sample change is shown below, but you'll want to make this change for each `new Movie`.</span></span>

[!code-csharp[](start-mvc/sample/MvcMovie/Models/SeedDataRating.cs?name=snippet1&highlight=6)]

<span data-ttu-id="7ac92-140">生成解决方案。</span><span class="sxs-lookup"><span data-stu-id="7ac92-140">Build the solution.</span></span>

<span data-ttu-id="7ac92-141">从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”。</span><span class="sxs-lookup"><span data-stu-id="7ac92-141">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>

  ![PMC 菜单](adding-model/_static/pmc.png)

<span data-ttu-id="7ac92-143">在 PMC 中，输入以下命令：</span><span class="sxs-lookup"><span data-stu-id="7ac92-143">In the PMC, enter the following commands:</span></span>

```powershell
Add-Migration Rating
Update-Database
```

<span data-ttu-id="7ac92-144">`Add-Migration` 命令会通知迁移框架使用当前 `Movie` DB 架构检查当前 `Movie` 模型，并创建必要的代码，将 DB 迁移到新模型。</span><span class="sxs-lookup"><span data-stu-id="7ac92-144">The `Add-Migration` command tells the migration framework to examine the current `Movie` model with the current `Movie` DB schema and create the necessary code to migrate the DB to the new model.</span></span> <span data-ttu-id="7ac92-145">名称“Rating”是任意的，用于对迁移文件进行命名。</span><span class="sxs-lookup"><span data-stu-id="7ac92-145">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="7ac92-146">为迁移文件使用有意义的名称是有帮助的。</span><span class="sxs-lookup"><span data-stu-id="7ac92-146">It's helpful to use a meaningful name for the migration file.</span></span>

<span data-ttu-id="7ac92-147">如果删除 DB 中的所有记录，初始化会设定 DB 种子，并将包括 `Rating` 字段。</span><span class="sxs-lookup"><span data-stu-id="7ac92-147">If you delete all the records in the DB, the initialize will seed the DB and include the `Rating` field.</span></span> <span data-ttu-id="7ac92-148">可以使用浏览器中的删除链接，也可从 SSOX 执行此操作。</span><span class="sxs-lookup"><span data-stu-id="7ac92-148">You can do this with the delete links in the browser or from SSOX.</span></span>

<span data-ttu-id="7ac92-149">运行应用，并验证是否可以创建/编辑/显示具有 `Rating` 字段的电影。</span><span class="sxs-lookup"><span data-stu-id="7ac92-149">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="7ac92-150">还应向 `Edit`、`Details` 和 `Delete` 视图模板添加 `Rating` 字段。</span><span class="sxs-lookup"><span data-stu-id="7ac92-150">You should also add the `Rating` field to the `Edit`, `Details`, and `Delete` view templates.</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="7ac92-151">[上一页](search.md)
> [下一页](validation.md)</span><span class="sxs-lookup"><span data-stu-id="7ac92-151">[Previous](search.md)
[Next](validation.md)</span></span>  
