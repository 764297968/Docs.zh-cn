---
title: 将新字段添加到 ASP.NET Core 中的 Razor 页面
author: rick-anderson
description: 演示如何使用 Entity Framework Core 将新字段添加到 Razor 页面
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/24/2018
uid: tutorials/razor-pages/new-field
ms.openlocfilehash: f8be269887903797803257d8a21e002519102047
ms.sourcegitcommit: 4d74644f11e0dac52b4510048490ae731c691496
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50089508"
---
# <a name="add-a-new-field-to-a-razor-page-in-aspnet-core"></a><span data-ttu-id="61f9d-103">将新字段添加到 ASP.NET Core 中的 Razor 页面</span><span class="sxs-lookup"><span data-stu-id="61f9d-103">Add a new field to a Razor Page in ASP.NET Core</span></span>

<span data-ttu-id="61f9d-104">作者：[Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="61f9d-104">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="61f9d-105">本部分使用 [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First 迁移将新字段添加到模型，并将此更改迁移到数据库。</span><span class="sxs-lookup"><span data-stu-id="61f9d-105">In this section you use [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First Migrations to add a new field to the model and migrate that change to the database.</span></span>

<span data-ttu-id="61f9d-106">使用 EF Code First 自动创建数据库时，Code First 将：</span><span class="sxs-lookup"><span data-stu-id="61f9d-106">When using EF Code First to automatically create a database, Code First:</span></span>

* <span data-ttu-id="61f9d-107">向数据库添加表格，以跟踪数据库的架构是否与从生成它的模型类同步。</span><span class="sxs-lookup"><span data-stu-id="61f9d-107">Adds a table to the database to track whether the schema of the database is in sync with the model classes it was generated from.</span></span>
* <span data-ttu-id="61f9d-108">如果该模型类未与数据库同步，EF 将引发异常。</span><span class="sxs-lookup"><span data-stu-id="61f9d-108">If the model classes aren't in sync with the DB, EF throws an exception.</span></span> 

<span data-ttu-id="61f9d-109">通过自动验证同步的架构/模型可以更容易地发现不一致的数据库/代码问题。</span><span class="sxs-lookup"><span data-stu-id="61f9d-109">Automatic verification of schema/model in sync makes it easier to find inconsistent database/code issues.</span></span>

## <a name="adding-a-rating-property-to-the-movie-model"></a><span data-ttu-id="61f9d-110">向电影模型添加分级属性</span><span class="sxs-lookup"><span data-stu-id="61f9d-110">Adding a Rating Property to the Movie Model</span></span>

<span data-ttu-id="61f9d-111">打开 Models/Movie.cs 文件，并添加 `Rating` 属性：</span><span class="sxs-lookup"><span data-stu-id="61f9d-111">Open the *Models/Movie.cs* file and add a `Rating` property:</span></span>

::: moniker range="= aspnetcore-2.0"

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Models/MovieDateRating.cs?highlight=11&range=7-18)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie21/Models/MovieDateRating.cs?highlight=13&name=snippet)]

::: moniker-end

<span data-ttu-id="61f9d-112">生成应用 (Ctrl+Shift+B)。</span><span class="sxs-lookup"><span data-stu-id="61f9d-112">Build the app (Ctrl+Shift+B).</span></span>

<span data-ttu-id="61f9d-113">编辑 Pages/Movies/Index.cshtml，并添加 `Rating` 字段：</span><span class="sxs-lookup"><span data-stu-id="61f9d-113">Edit *Pages/Movies/Index.cshtml*, and add a `Rating` field:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml?highlight=40-42,61-63)]

<span data-ttu-id="61f9d-114">将 `Rating` 字段添加到“删除”和“详细信息”页面。</span><span class="sxs-lookup"><span data-stu-id="61f9d-114">Add the `Rating` field to the Delete and Details pages.</span></span>

<span data-ttu-id="61f9d-115">使用 `Rating` 字段更新 Create.cshtml。</span><span class="sxs-lookup"><span data-stu-id="61f9d-115">Update *Create.cshtml* with a `Rating` field.</span></span> <span data-ttu-id="61f9d-116">可以复制/粘贴之前的 `<div>` 元素，并让 intelliSense 帮助更新字段。</span><span class="sxs-lookup"><span data-stu-id="61f9d-116">You can copy/paste the previous `<div>` element and let intelliSense help you update the fields.</span></span> <span data-ttu-id="61f9d-117">IntelliSense 适用于[标记帮助程序](xref:mvc/views/tag-helpers/intro)。</span><span class="sxs-lookup"><span data-stu-id="61f9d-117">IntelliSense works with [Tag Helpers](xref:mvc/views/tag-helpers/intro).</span></span>

![开发人员已在视图的第二个标签元素中键入字母 R 用作 asp-for 的特性值。](new-field/_static/cr.png)

<span data-ttu-id="61f9d-121">下面的代码显示具有 `Rating` 字段的 Create.cshtml：</span><span class="sxs-lookup"><span data-stu-id="61f9d-121">The following code shows *Create.cshtml* with a `Rating` field:</span></span>

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Create.cshtml?highlight=36-40)]

<span data-ttu-id="61f9d-122">将 `Rating` 字段添加到“编辑”页面。</span><span class="sxs-lookup"><span data-stu-id="61f9d-122">Add the `Rating` field to the Edit Page.</span></span>

<span data-ttu-id="61f9d-123">在 DB 更新为包括新字段之前，应用将不会正常工作。</span><span class="sxs-lookup"><span data-stu-id="61f9d-123">The app won't work until the DB is updated to include the new field.</span></span> <span data-ttu-id="61f9d-124">如果立即运行，应用会引发 `SqlException`：</span><span class="sxs-lookup"><span data-stu-id="61f9d-124">If run now, the app throws a `SqlException`:</span></span>

```
SqlException: Invalid column name 'Rating'.
```

<span data-ttu-id="61f9d-125">此错误是由于更新的 Movie 模型类与数据库的 Movie 表架构不同导致的。</span><span class="sxs-lookup"><span data-stu-id="61f9d-125">This error is caused by the updated Movie model class being different than the schema of the Movie table of the database.</span></span> <span data-ttu-id="61f9d-126">（数据库表中没有 `Rating` 列。）</span><span class="sxs-lookup"><span data-stu-id="61f9d-126">(There's no `Rating` column in the database table.)</span></span>

<span data-ttu-id="61f9d-127">可通过几种方法解决此错误：</span><span class="sxs-lookup"><span data-stu-id="61f9d-127">There are a few approaches to resolving the error:</span></span>

1. <span data-ttu-id="61f9d-128">让 Entity Framework 自动丢弃并使用新的模型类架构重新创建数据库。</span><span class="sxs-lookup"><span data-stu-id="61f9d-128">Have the Entity Framework automatically drop and re-create the database using  the new model class schema.</span></span> <span data-ttu-id="61f9d-129">此方法在开发周期早期很方便；通过它可以一起快速改进模型和数据库架构。</span><span class="sxs-lookup"><span data-stu-id="61f9d-129">This approach is convenient early in the development cycle; it allows you to quickly evolve the model and database schema together.</span></span> <span data-ttu-id="61f9d-130">此方法的缺点是会导致数据库中的现有数据丢失。</span><span class="sxs-lookup"><span data-stu-id="61f9d-130">The downside is that you lose existing data in the database.</span></span> <span data-ttu-id="61f9d-131">请勿对生产数据库使用此方法！</span><span class="sxs-lookup"><span data-stu-id="61f9d-131">You don't want to use this approach on a production database!</span></span> <span data-ttu-id="61f9d-132">当架构更改时丢弃数据库并使用初始值设定项以使用测试数据自动设定数据库种子，这通常是开发应用的有效方式。</span><span class="sxs-lookup"><span data-stu-id="61f9d-132">Dropping the DB on schema changes and using an initializer to automatically seed the database with test data is often a productive way to develop an app.</span></span>

2. <span data-ttu-id="61f9d-133">对现有数据库架构进行显式修改，使它与模型类相匹配。</span><span class="sxs-lookup"><span data-stu-id="61f9d-133">Explicitly modify the schema of the existing database so that it matches the model classes.</span></span> <span data-ttu-id="61f9d-134">此方法的优点是可以保留数据。</span><span class="sxs-lookup"><span data-stu-id="61f9d-134">The advantage of this approach is that you keep your data.</span></span> <span data-ttu-id="61f9d-135">可以手动或通过创建数据库更改脚本进行此更改。</span><span class="sxs-lookup"><span data-stu-id="61f9d-135">You can make this change either manually or by creating a database change script.</span></span>

3. <span data-ttu-id="61f9d-136">使用 Code First 迁移更新数据库架构。</span><span class="sxs-lookup"><span data-stu-id="61f9d-136">Use Code First Migrations to update the database schema.</span></span>

<span data-ttu-id="61f9d-137">对于本教程，请使用 Code First 迁移。</span><span class="sxs-lookup"><span data-stu-id="61f9d-137">For this tutorial, use Code First Migrations.</span></span>

<span data-ttu-id="61f9d-138">更新 `SeedData` 类，使它提供新列的值。</span><span class="sxs-lookup"><span data-stu-id="61f9d-138">Update the `SeedData` class so that it provides a value for the new column.</span></span> <span data-ttu-id="61f9d-139">示例更改如下所示，但可能需要对每个 `new Movie` 块做出此更改。</span><span class="sxs-lookup"><span data-stu-id="61f9d-139">A sample change is shown below, but you'll want to make this change for each `new Movie` block.</span></span>

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

::: moniker range="= aspnetcore-2.0"

<span data-ttu-id="61f9d-140">请参阅[已完成的 SeedData.cs 文件](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/Models/SeedDataRating.cs)。</span><span class="sxs-lookup"><span data-stu-id="61f9d-140">See the [completed SeedData.cs file](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie/Models/SeedDataRating.cs).</span></span>

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

<span data-ttu-id="61f9d-141">请参阅[已完成的 SeedData.cs 文件](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie21/Models/SeedDataRating.cs)。</span><span class="sxs-lookup"><span data-stu-id="61f9d-141">See the [completed SeedData.cs file](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie21/Models/SeedDataRating.cs).</span></span>

::: moniker-end

<span data-ttu-id="61f9d-142">生成解决方案。</span><span class="sxs-lookup"><span data-stu-id="61f9d-142">Build the solution.</span></span>

<a name="pmc"></a>

<span data-ttu-id="61f9d-143">从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”。</span><span class="sxs-lookup"><span data-stu-id="61f9d-143">From the **Tools** menu, select **NuGet Package Manager > Package Manager Console**.</span></span>
<span data-ttu-id="61f9d-144">在 PMC 中，输入以下命令：</span><span class="sxs-lookup"><span data-stu-id="61f9d-144">In the PMC, enter the following commands:</span></span>

```powershell
Add-Migration Rating
Update-Database
```

<span data-ttu-id="61f9d-145">`Add-Migration` 命令会通知框架执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="61f9d-145">The `Add-Migration` command tells the framework to:</span></span>

* <span data-ttu-id="61f9d-146">将 `Movie` 模型与 `Movie` DB 架构进行比较。</span><span class="sxs-lookup"><span data-stu-id="61f9d-146">Compare the `Movie` model with the `Movie` DB schema.</span></span>
* <span data-ttu-id="61f9d-147">创建代码以将 DB 架构迁移到新模型。</span><span class="sxs-lookup"><span data-stu-id="61f9d-147">Create code to migrate the DB schema to the new model.</span></span>

<span data-ttu-id="61f9d-148">名称“Rating”是任意的，用于对迁移文件进行命名。</span><span class="sxs-lookup"><span data-stu-id="61f9d-148">The name "Rating" is arbitrary and is used to name the migration file.</span></span> <span data-ttu-id="61f9d-149">为迁移文件使用有意义的名称是有帮助的。</span><span class="sxs-lookup"><span data-stu-id="61f9d-149">It's helpful to use a meaningful name for the migration file.</span></span>

<a name="ssox"></a>

<span data-ttu-id="61f9d-150">如果删除 DB 中的所有记录，种子初始值设定项会设定 DB 种子，并将包括 `Rating` 字段。</span><span class="sxs-lookup"><span data-stu-id="61f9d-150">If you delete all the records in the DB, the initializer will seed the DB and include the `Rating` field.</span></span> <span data-ttu-id="61f9d-151">可以使用浏览器中的删除链接，也可以从 [Sql Server 对象资源管理器](xref:tutorials/razor-pages/sql#ssox) (SSOX) 执行此操作。</span><span class="sxs-lookup"><span data-stu-id="61f9d-151">You can do this with the delete links in the browser or from [Sql Server Object Explorer](xref:tutorials/razor-pages/sql#ssox) (SSOX).</span></span> <span data-ttu-id="61f9d-152">从 SSOX 中删除数据库：</span><span class="sxs-lookup"><span data-stu-id="61f9d-152">To delete the database from SSOX:</span></span>

* <span data-ttu-id="61f9d-153">在 SSOX 中选择数据库。</span><span class="sxs-lookup"><span data-stu-id="61f9d-153">Select the database in SSOX.</span></span>
* <span data-ttu-id="61f9d-154">右键单击数据库，并选择“删除”。</span><span class="sxs-lookup"><span data-stu-id="61f9d-154">Right click on the database, and select *Delete*.</span></span>
* <span data-ttu-id="61f9d-155">检查“关闭现有连接”。</span><span class="sxs-lookup"><span data-stu-id="61f9d-155">Check **Close existing connections**.</span></span>
* <span data-ttu-id="61f9d-156">选择“确定”。</span><span class="sxs-lookup"><span data-stu-id="61f9d-156">Select **OK**.</span></span>
* <span data-ttu-id="61f9d-157">在 [PMC](xref:tutorials/razor-pages/new-field#pmc) 中更新数据库：</span><span class="sxs-lookup"><span data-stu-id="61f9d-157">In the [PMC](xref:tutorials/razor-pages/new-field#pmc), update the database:</span></span>

  ```powershell
  Update-Database
  ```

<span data-ttu-id="61f9d-158">运行应用，并验证是否可以创建/编辑/显示具有 `Rating` 字段的电影。</span><span class="sxs-lookup"><span data-stu-id="61f9d-158">Run the app and verify you can create/edit/display movies with a `Rating` field.</span></span> <span data-ttu-id="61f9d-159">如果数据库未设定种子，请先停止 IIS Express，然后再运行应用。</span><span class="sxs-lookup"><span data-stu-id="61f9d-159">If the database isn't seeded, stop IIS Express, and then run the app.</span></span>

> [!div class="step-by-step"]
> <span data-ttu-id="61f9d-160">[上一篇：添加搜索](xref:tutorials/razor-pages/search)
> [下一篇：添加验证](xref:tutorials/razor-pages/validation)</span><span class="sxs-lookup"><span data-stu-id="61f9d-160">[Previous: Adding Search](xref:tutorials/razor-pages/search)
[Next: Adding Validation](xref:tutorials/razor-pages/validation)</span></span>
