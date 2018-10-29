---
title: ASP.NET Core MVC 和 EF Core - 更新相关数据 - 第 7 个教程（共 10 个）
author: rick-anderson
description: 本教程将通过更新外键字段和导航属性来更新相关数据。
ms.author: tdykstra
ms.custom: mvc
ms.date: 10/24/2018
uid: data/ef-mvc/update-related-data
ms.openlocfilehash: 37985c945f2e4b15cfcefb0c126c3209e0bdeac4
ms.sourcegitcommit: 4d74644f11e0dac52b4510048490ae731c691496
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50090727"
---
# <a name="aspnet-core-mvc-with-ef-core---update-related-data---7-of-10"></a><span data-ttu-id="4ff34-103">ASP.NET Core MVC 和 EF Core - 更新相关数据 - 第 7 个教程（共 10 个）</span><span class="sxs-lookup"><span data-stu-id="4ff34-103">ASP.NET Core MVC with EF Core - Update Related Data - 7 of 10</span></span>

[!INCLUDE [RP better than MVC](~/includes/RP-EF/rp-over-mvc-21.md)]

::: moniker range="= aspnetcore-2.0"

<span data-ttu-id="4ff34-104">作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="4ff34-104">By [Tom Dykstra](https://github.com/tdykstra) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="4ff34-105">Contoso 大学示例 web 应用程序演示如何使用 Entity Framework Core 和 Visual Studio 创建 ASP.NET Core MVC web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="4ff34-105">The Contoso University sample web application demonstrates how to create ASP.NET Core MVC web applications using Entity Framework Core and Visual Studio.</span></span> <span data-ttu-id="4ff34-106">若要了解教程系列，请参阅[本系列中的第一个教程](intro.md)。</span><span class="sxs-lookup"><span data-stu-id="4ff34-106">For information about the tutorial series, see [the first tutorial in the series](intro.md).</span></span>

<span data-ttu-id="4ff34-107">上一个教程显示出了相关数据，本教程将通过更新外键字段和导航属性来更新相关数据。</span><span class="sxs-lookup"><span data-stu-id="4ff34-107">In the previous tutorial you displayed related data; in this tutorial you'll update related data by updating foreign key fields and navigation properties.</span></span>

<span data-ttu-id="4ff34-108">下图是一些将会用到的页面。</span><span class="sxs-lookup"><span data-stu-id="4ff34-108">The following illustrations show some of the pages that you'll work with.</span></span>

![“课程编辑”页面](update-related-data/_static/course-edit.png)

![“讲师编辑”页面](update-related-data/_static/instructor-edit-courses.png)

## <a name="customize-the-create-and-edit-pages-for-courses"></a><span data-ttu-id="4ff34-111">自定义课程的创建和编辑页面</span><span class="sxs-lookup"><span data-stu-id="4ff34-111">Customize the Create and Edit Pages for Courses</span></span>

<span data-ttu-id="4ff34-112">创建新的课程实体时，新实体必须与现有院系有关系。</span><span class="sxs-lookup"><span data-stu-id="4ff34-112">When a new course entity is created, it must have a relationship to an existing department.</span></span> <span data-ttu-id="4ff34-113">为此，基架代码需包括控制器方法、创建视图和编辑视图，且视图中应包括用于选择院系的下拉列表。</span><span class="sxs-lookup"><span data-stu-id="4ff34-113">To facilitate this, the scaffolded code includes controller methods and Create and Edit views that include a drop-down list for selecting the department.</span></span> <span data-ttu-id="4ff34-114">下拉列表设置了 `Course.DepartmentID` 外键属性，而这正是 Entity Framework 使用适当的 Department 实体加载 `Department` 导航属性所需要的。</span><span class="sxs-lookup"><span data-stu-id="4ff34-114">The drop-down list sets the `Course.DepartmentID` foreign key property, and that's all the Entity Framework needs in order to load the `Department` navigation property with the appropriate Department entity.</span></span> <span data-ttu-id="4ff34-115">将用到基架代码，但需对其稍作更改，以便添加错误处理和对下拉列表进行排序。</span><span class="sxs-lookup"><span data-stu-id="4ff34-115">You'll use the scaffolded code, but change it slightly to add error handling and sort the drop-down list.</span></span>

<span data-ttu-id="4ff34-116">在 CoursesController.cs 中，删除四种 Create 和 Edit 方法，并将其替换为以下代码：</span><span class="sxs-lookup"><span data-stu-id="4ff34-116">In *CoursesController.cs*, delete the four Create and Edit methods and replace them with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreateGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_CreatePost)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditGet)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_EditPost)]

<span data-ttu-id="4ff34-117">在 `Edit` HttpPost 方法之后，新建一个方法来为下拉列表加载院系信息。</span><span class="sxs-lookup"><span data-stu-id="4ff34-117">After the `Edit` HttpPost method, create a new method that loads department info for the drop-down list.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?name=snippet_Departments)]

<span data-ttu-id="4ff34-118">`PopulateDepartmentsDropDownList` 方法获取按名称排序的所有院系的列表，为下拉列表创建 `SelectList` 集合，并将该集合传递给 `ViewBag` 中的视图。</span><span class="sxs-lookup"><span data-stu-id="4ff34-118">The `PopulateDepartmentsDropDownList` method gets a list of all departments sorted by name, creates a `SelectList` collection for a drop-down list, and passes the collection to the view in `ViewBag`.</span></span> <span data-ttu-id="4ff34-119">该方法可以使用可选的 `selectedDepartment` 参数，而调用的代码可以通过该参数来指定呈现下拉列表时被选择的项。</span><span class="sxs-lookup"><span data-stu-id="4ff34-119">The method accepts the optional `selectedDepartment` parameter that allows the calling code to specify the item that will be selected when the drop-down list is rendered.</span></span> <span data-ttu-id="4ff34-120">视图将 DepartmentID 名称传递给 `<select>` 标记帮助器，该帮助器就知道在 `ViewBag` 对象中查找名为 DepartmentID 的 `SelectList`。</span><span class="sxs-lookup"><span data-stu-id="4ff34-120">The view will pass the name "DepartmentID" to the `<select>` tag helper, and the helper then knows to look in the `ViewBag` object for a `SelectList` named "DepartmentID".</span></span>

<span data-ttu-id="4ff34-121">HttpGet `Create` 方法调用 `PopulateDepartmentsDropDownList` 方法，但不会设置选定项，因为对于新课程而言，其院系尚未建立：</span><span class="sxs-lookup"><span data-stu-id="4ff34-121">The HttpGet `Create` method calls the `PopulateDepartmentsDropDownList` method without setting the selected item, because for a new course the department isn't established yet:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=3&name=snippet_CreateGet)]

<span data-ttu-id="4ff34-122">HttpGet `Edit` 方法根据正在编辑的课程已分配到的院系 ID 设置选定项：</span><span class="sxs-lookup"><span data-stu-id="4ff34-122">The HttpGet `Edit` method sets the selected item, based on the ID of the department that's already assigned to the course being edited:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=15&name=snippet_EditGet)]

<span data-ttu-id="4ff34-123">`Create` 和 `Edit` 这二者的 HttpPost 方法还包括一段代码，用于在错误后重新显示页面时设置选定项。</span><span class="sxs-lookup"><span data-stu-id="4ff34-123">The HttpPost methods for both `Create` and `Edit` also include code that sets the selected item when they redisplay the page after an error.</span></span> <span data-ttu-id="4ff34-124">这样可以确保当页面重新显示出现错误消息时，选择的任何院系都将保持选中状态。</span><span class="sxs-lookup"><span data-stu-id="4ff34-124">This ensures that when the page is redisplayed to show the error message, whatever department was selected stays selected.</span></span>

### <a name="add-asnotracking-to-details-and-delete-methods"></a><span data-ttu-id="4ff34-125">将 .AsNoTracking 添加到 Details 和 Delete 方法</span><span class="sxs-lookup"><span data-stu-id="4ff34-125">Add .AsNoTracking to Details and Delete methods</span></span>

<span data-ttu-id="4ff34-126">为优化“课程详细信息”和“删除”页面的性能，请在 `Details` 和 HttpGet `Delete` 方法中添加 `AsNoTracking` 调用。</span><span class="sxs-lookup"><span data-stu-id="4ff34-126">To optimize performance of the Course Details and Delete pages, add `AsNoTracking` calls in the `Details` and HttpGet `Delete` methods.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_Details)]

[!code-csharp[](intro/samples/cu/Controllers/CoursesController.cs?highlight=10&name=snippet_DeleteGet)]

### <a name="modify-the-course-views"></a><span data-ttu-id="4ff34-127">修改课程视图</span><span class="sxs-lookup"><span data-stu-id="4ff34-127">Modify the Course views</span></span>

<span data-ttu-id="4ff34-128">在 Views/Courses/Create.cshtml 中，向“院系”下拉列表添加一个“选择院系”选项，将标题从 DepartmentID 更改为 Department，并添加一条验证消息。</span><span class="sxs-lookup"><span data-stu-id="4ff34-128">In *Views/Courses/Create.cshtml*, add a "Select Department" option to the **Department** drop-down list, change the caption from **DepartmentID** to **Department**, and add a validation message.</span></span>

[!code-html[](intro/samples/cu/Views/Courses/Create.cshtml?highlight=2-6&range=29-34)]

<span data-ttu-id="4ff34-129">在 Views/Courses/Edit.cshtml 中，对“院系”字段进行与 Create.cshtml 中相同的更改。</span><span class="sxs-lookup"><span data-stu-id="4ff34-129">In *Views/Courses/Edit.cshtml*, make the same change for the Department field that you just did in *Create.cshtml*.</span></span>

<span data-ttu-id="4ff34-130">另外，在 Views/Courses/Edit.cshtml 中，在“标题”字段之前添加一个课程编号字段。</span><span class="sxs-lookup"><span data-stu-id="4ff34-130">Also in *Views/Courses/Edit.cshtml*, add a course number field before the **Title** field.</span></span> <span data-ttu-id="4ff34-131">课程编号是主键，因此只会显示，无法更改。</span><span class="sxs-lookup"><span data-stu-id="4ff34-131">Because the course number is the primary key, it's displayed, but it can't be changed.</span></span>

[!code-html[](intro/samples/cu/Views/Courses/Edit.cshtml?range=15-18)]

<span data-ttu-id="4ff34-132">“编辑”视图中已有一个隐藏的课程编号字段（`<input type="hidden">`。</span><span class="sxs-lookup"><span data-stu-id="4ff34-132">There's already a hidden field (`<input type="hidden">`) for the course number in the Edit view.</span></span> <span data-ttu-id="4ff34-133">添加 `<label>` 标记帮助器后仍然需要该隐藏字段，因为添加标记帮助器后，用户在“编辑”页面上单击“保存”时，已发布数据中并不会包含课程编号。</span><span class="sxs-lookup"><span data-stu-id="4ff34-133">Adding a `<label>` tag helper doesn't eliminate the need for the hidden field because it doesn't cause the course number to be included in the posted data when the user clicks **Save** on the **Edit** page.</span></span>

<span data-ttu-id="4ff34-134">在 Views/Courses/Delete.cshtml 中，在顶部添加一个课程编号字段，并将院系 ID 更改为院系名称。</span><span class="sxs-lookup"><span data-stu-id="4ff34-134">In *Views/Courses/Delete.cshtml*, add a course number field at the top and change department ID to department name.</span></span>

[!code-html[](intro/samples/cu/Views/Courses/Delete.cshtml?highlight=14-19,36)]

<span data-ttu-id="4ff34-135">在 Views/Courses/Details.cshtml 中，进行对 Delete.cshtml 所作相同的更改。</span><span class="sxs-lookup"><span data-stu-id="4ff34-135">In *Views/Courses/Details.cshtml*, make the same change that you just did for *Delete.cshtml*.</span></span>

### <a name="test-the-course-pages"></a><span data-ttu-id="4ff34-136">测试“课程”页</span><span class="sxs-lookup"><span data-stu-id="4ff34-136">Test the Course pages</span></span>

<span data-ttu-id="4ff34-137">运行应用，选择“课程”选项卡，单击“新建”，然后输入新课程的数据：</span><span class="sxs-lookup"><span data-stu-id="4ff34-137">Run the app, select the **Courses** tab, click **Create New**, and enter data for a new course:</span></span>

![课程创建页面](update-related-data/_static/course-create.png)

<span data-ttu-id="4ff34-139">单击 **“创建”**。</span><span class="sxs-lookup"><span data-stu-id="4ff34-139">Click **Create**.</span></span> <span data-ttu-id="4ff34-140">课程索引页面随即显示，并且新课程已添加在列表中。</span><span class="sxs-lookup"><span data-stu-id="4ff34-140">The Courses Index page is displayed with the new course added to the list.</span></span> <span data-ttu-id="4ff34-141">索引页列表中的院系名称来自导航属性，表明已正确建立关系。</span><span class="sxs-lookup"><span data-stu-id="4ff34-141">The department name in the Index page list comes from the navigation property, showing that the relationship was established correctly.</span></span>

<span data-ttu-id="4ff34-142">在课程索引页中的课程上，单击“编辑”。</span><span class="sxs-lookup"><span data-stu-id="4ff34-142">Click **Edit** on a course in the Courses Index page.</span></span>

![“课程编辑”页面](update-related-data/_static/course-edit.png)

<span data-ttu-id="4ff34-144">更改页面上的数据，然后单击“保存”。</span><span class="sxs-lookup"><span data-stu-id="4ff34-144">Change data on the page and click **Save**.</span></span> <span data-ttu-id="4ff34-145">含有更新后的课程数据的“课程索引”页面随即显示。</span><span class="sxs-lookup"><span data-stu-id="4ff34-145">The Courses Index page is displayed with the updated course data.</span></span>

## <a name="add-an-edit-page-for-instructors"></a><span data-ttu-id="4ff34-146">添加讲师的编辑页面</span><span class="sxs-lookup"><span data-stu-id="4ff34-146">Add an Edit Page for Instructors</span></span>

<span data-ttu-id="4ff34-147">编辑讲师记录时，有时希望能更新讲师的办公室分配。</span><span class="sxs-lookup"><span data-stu-id="4ff34-147">When you edit an instructor record, you want to be able to update the instructor's office assignment.</span></span> <span data-ttu-id="4ff34-148">Instructor 实体和 OfficeAssignment 实体之间存在一对零或一的关系，这意味着代码必须处理一下情况：</span><span class="sxs-lookup"><span data-stu-id="4ff34-148">The Instructor entity has a one-to-zero-or-one relationship with the OfficeAssignment entity, which means your code has to handle the following situations:</span></span>

* <span data-ttu-id="4ff34-149">如果用户清除了办公室分配，并且办公室分配最初具有一个值，则删除 OfficeAssignment 实体。</span><span class="sxs-lookup"><span data-stu-id="4ff34-149">If the user clears the office assignment and it originally had a value, delete the OfficeAssignment entity.</span></span>

* <span data-ttu-id="4ff34-150">如果用户输入了办公室分配值，并且该值最初为空，则创建一个新的 OfficeAssignment 实体。</span><span class="sxs-lookup"><span data-stu-id="4ff34-150">If the user enters an office assignment value and it originally was empty, create a new OfficeAssignment entity.</span></span>

* <span data-ttu-id="4ff34-151">如果用户更改了办公室分配的值，则更改现有 OfficeAssignment 实体中的值。</span><span class="sxs-lookup"><span data-stu-id="4ff34-151">If the user changes the value of an office assignment, change the value in an existing OfficeAssignment entity.</span></span>

### <a name="update-the-instructors-controller"></a><span data-ttu-id="4ff34-152">更新讲师控制器</span><span class="sxs-lookup"><span data-stu-id="4ff34-152">Update the Instructors controller</span></span>

<span data-ttu-id="4ff34-153">在 InstructorsController.cs 中，更改 HttpGet `Edit` 方法中的代码，使其加载 Instructor 实体的 `OfficeAssignment` 导航属性并调用 `AsNoTracking`：</span><span class="sxs-lookup"><span data-stu-id="4ff34-153">In *InstructorsController.cs*, change the code in the HttpGet `Edit` method so that it loads the Instructor entity's `OfficeAssignment` navigation property and calls `AsNoTracking`:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=9,10&name=snippet_EditGetOA)]

<span data-ttu-id="4ff34-154">将 HttpPost `Edit` 方法更新为以下代码，以便处理办公室分配更新：</span><span class="sxs-lookup"><span data-stu-id="4ff34-154">Replace the HttpPost `Edit` method with the following code to handle office assignment updates:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_EditPostOA)]

<span data-ttu-id="4ff34-155">该代码执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="4ff34-155">The code does the following:</span></span>

-  <span data-ttu-id="4ff34-156">将方法名称更改为 `EditPost`，因为现在的签名与 HttpGet `Edit` 方法相同（`ActionName` 特性指定仍然使用 `/Edit/` URL）。</span><span class="sxs-lookup"><span data-stu-id="4ff34-156">Changes the method name to `EditPost` because the signature is now the same as the HttpGet `Edit` method (the `ActionName` attribute specifies that the `/Edit/` URL is still used).</span></span>

-  <span data-ttu-id="4ff34-157">使用 `OfficeAssignment` 导航属性的预先加载从数据库获取当前的 Instructor 实体。</span><span class="sxs-lookup"><span data-stu-id="4ff34-157">Gets the current Instructor entity from the database using eager loading for the `OfficeAssignment` navigation property.</span></span> <span data-ttu-id="4ff34-158">此操作与在 HttpGet `Edit` 方法中进行的操作相同。</span><span class="sxs-lookup"><span data-stu-id="4ff34-158">This is the same as what you did in the HttpGet `Edit` method.</span></span>

-  <span data-ttu-id="4ff34-159">将检索出的 Instructor 实体更新为模型绑定器中的值。</span><span class="sxs-lookup"><span data-stu-id="4ff34-159">Updates the retrieved Instructor entity with values from the model binder.</span></span> <span data-ttu-id="4ff34-160">通过 `TryUpdateModel` 重载可以将想包括的属性列入到允许列表。</span><span class="sxs-lookup"><span data-stu-id="4ff34-160">The `TryUpdateModel` overload enables you to whitelist the properties you want to include.</span></span> <span data-ttu-id="4ff34-161">这样可以防止[第二个教程](crud.md)中所述的过度发布。</span><span class="sxs-lookup"><span data-stu-id="4ff34-161">This prevents over-posting, as explained in the [second tutorial](crud.md).</span></span>

    <!-- Snippets don't play well with <ul> [!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?range=241-244)] -->

    ```csharp
    if (await TryUpdateModelAsync<Instructor>(
        instructorToUpdate,
        "",
        i => i.FirstMidName, i => i.LastName, i => i.HireDate, i => i.OfficeAssignment))
    ```

-   <span data-ttu-id="4ff34-162">如果办公室位置为空，请将 Instructor.OfficeAssignment 属性设置为 NULL，以便删除 OfficeAssignment 表中的相关行。</span><span class="sxs-lookup"><span data-stu-id="4ff34-162">If the office location is blank, sets the Instructor.OfficeAssignment property to null so that the related row in the OfficeAssignment table will be deleted.</span></span>

    <!-- Snippets don't play well with <ul>  "intro/samples/cu/Controllers/InstructorsController.cs"} -->

    ```csharp
    if (String.IsNullOrWhiteSpace(instructorToUpdate.OfficeAssignment?.Location))
    {
        instructorToUpdate.OfficeAssignment = null;
    }
    ```

- <span data-ttu-id="4ff34-163">将更改保存到数据库。</span><span class="sxs-lookup"><span data-stu-id="4ff34-163">Saves the changes to the database.</span></span>

### <a name="update-the-instructor-edit-view"></a><span data-ttu-id="4ff34-164">更新讲师编辑视图</span><span class="sxs-lookup"><span data-stu-id="4ff34-164">Update the Instructor Edit view</span></span>

<span data-ttu-id="4ff34-165">在 Views/Instructors/Edit.cshtml 中，在“保存”按钮之前的末尾处，添加一个用于编辑办公室位置的新字段：</span><span class="sxs-lookup"><span data-stu-id="4ff34-165">In *Views/Instructors/Edit.cshtml*, add a new field for editing the office location, at the end before the **Save** button:</span></span>

[!code-html[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=30-34)]

<span data-ttu-id="4ff34-166">运行应用，选择“讲师”选项卡，然后单击讲师页面上的“编辑”。</span><span class="sxs-lookup"><span data-stu-id="4ff34-166">Run the app, select the **Instructors** tab, and then click **Edit** on an instructor.</span></span> <span data-ttu-id="4ff34-167">更改“办公室位置”，然后单击“保存”。</span><span class="sxs-lookup"><span data-stu-id="4ff34-167">Change the **Office Location** and click **Save**.</span></span>

![“讲师编辑”页面](update-related-data/_static/instructor-edit-office.png)

## <a name="add-course-assignments-to-the-instructor-edit-page"></a><span data-ttu-id="4ff34-169">向“讲师编辑”页添加课程分配</span><span class="sxs-lookup"><span data-stu-id="4ff34-169">Add Course assignments to the Instructor Edit page</span></span>

<span data-ttu-id="4ff34-170">讲师可能教授任意数量的课程。</span><span class="sxs-lookup"><span data-stu-id="4ff34-170">Instructors may teach any number of courses.</span></span> <span data-ttu-id="4ff34-171">现在可以通过使用一组复选框来更改课程分配，从而增强讲师编辑页面的性能，如以下屏幕截图所示：</span><span class="sxs-lookup"><span data-stu-id="4ff34-171">Now you'll enhance the Instructor Edit page by adding the ability to change course assignments using a group of check boxes, as shown in the following screen shot:</span></span>

![带课程信息的讲师“编辑”页](update-related-data/_static/instructor-edit-courses.png)

<span data-ttu-id="4ff34-173">Course 和 Instructor 实体之间是多对多的关系。</span><span class="sxs-lookup"><span data-stu-id="4ff34-173">The relationship between the Course and Instructor entities is many-to-many.</span></span> <span data-ttu-id="4ff34-174">若要添加和删除关系，可以向 CourseAssignments 联接实体集添加实体和从中删除实体。</span><span class="sxs-lookup"><span data-stu-id="4ff34-174">To add and remove relationships, you add and remove entities to and from the CourseAssignments join entity set.</span></span>

<span data-ttu-id="4ff34-175">用于更改讲师所对应的课程的 UI 是一组复选框。</span><span class="sxs-lookup"><span data-stu-id="4ff34-175">The UI that enables you to change which courses an instructor is assigned to is a group of check boxes.</span></span> <span data-ttu-id="4ff34-176">该复选框中会显示数据库中的所有课程，选中讲师当前对应的课程即可。</span><span class="sxs-lookup"><span data-stu-id="4ff34-176">A check box for every course in the database is displayed, and the ones that the instructor is currently assigned to are selected.</span></span> <span data-ttu-id="4ff34-177">用户可以通过选择或清除复选框来更改课程分配。</span><span class="sxs-lookup"><span data-stu-id="4ff34-177">The user can select or clear check boxes to change course assignments.</span></span> <span data-ttu-id="4ff34-178">如果课程的数量过大，建议使用其他方法在视图中呈现数据，但创建或删除关系的方法与操作联接实体的方法相同。</span><span class="sxs-lookup"><span data-stu-id="4ff34-178">If the number of courses were much greater, you would probably want to use a different method of presenting the data in the view, but you'd use the same method of manipulating a join entity to create or delete relationships.</span></span>

### <a name="update-the-instructors-controller"></a><span data-ttu-id="4ff34-179">更新讲师控制器</span><span class="sxs-lookup"><span data-stu-id="4ff34-179">Update the Instructors controller</span></span>

<span data-ttu-id="4ff34-180">若要为复选框列表的视图提供数据，将使用视图模型类。</span><span class="sxs-lookup"><span data-stu-id="4ff34-180">To provide data to the view for the list of check boxes, you'll use a view model class.</span></span>

<span data-ttu-id="4ff34-181">在 SchoolViewModels 文件夹中创建 AssignedCourseData.cs，并将现有代码替换为以下代码：</span><span class="sxs-lookup"><span data-stu-id="4ff34-181">Create *AssignedCourseData.cs* in the *SchoolViewModels* folder and replace the existing code with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/AssignedCourseData.cs)]

<span data-ttu-id="4ff34-182">在 InstructorsController.cs 中，将 HttpGet `Edit` 方法替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="4ff34-182">In *InstructorsController.cs*, replace the HttpGet `Edit` method with the following code.</span></span> <span data-ttu-id="4ff34-183">突出显示所作更改。</span><span class="sxs-lookup"><span data-stu-id="4ff34-183">The changes are highlighted.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=10,17,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36&name=snippet_EditGetCourses)]

<span data-ttu-id="4ff34-184">该代码为 `Courses` 导航属性添加了预先加载，并调用新的 `PopulateAssignedCourseData` 方法使用 `AssignedCourseData` 视图模型类为复选框数组提供信息。</span><span class="sxs-lookup"><span data-stu-id="4ff34-184">The code adds eager loading for the `Courses` navigation property and calls the new `PopulateAssignedCourseData` method to provide information for the check box array using the `AssignedCourseData` view model class.</span></span>

<span data-ttu-id="4ff34-185">`PopulateAssignedCourseData` 方法中的代码会读取所有 Course 实体，以便使用视图模型类加载课程列表。</span><span class="sxs-lookup"><span data-stu-id="4ff34-185">The code in the `PopulateAssignedCourseData` method reads through all Course entities in order to load a list of courses using the view model class.</span></span> <span data-ttu-id="4ff34-186">对每门课程而言，该代码都会检查讲师的 `Courses` 导航属性中是否存在该课程。</span><span class="sxs-lookup"><span data-stu-id="4ff34-186">For each course, the code checks whether the course exists in the instructor's `Courses` navigation property.</span></span> <span data-ttu-id="4ff34-187">为高效检查某门课程是否被分配给了讲师，可将分配给该讲师的课程放置于 `HashSet` 集合中。</span><span class="sxs-lookup"><span data-stu-id="4ff34-187">To create efficient lookup when checking whether a course is assigned to the instructor, the courses assigned to the instructor are put into a `HashSet` collection.</span></span> <span data-ttu-id="4ff34-188">对于讲师分配到的课程，`Assigned` 属性则设置为 true。</span><span class="sxs-lookup"><span data-stu-id="4ff34-188">The `Assigned` property  is set to true for courses the instructor is assigned to.</span></span> <span data-ttu-id="4ff34-189">视图将使用此属性来确定应将哪些复选框显示为选中状态。</span><span class="sxs-lookup"><span data-stu-id="4ff34-189">The view will use this property to determine which check boxes must be displayed as selected.</span></span> <span data-ttu-id="4ff34-190">最后，该列表会被传递给 `ViewData` 中的视图。</span><span class="sxs-lookup"><span data-stu-id="4ff34-190">Finally, the list is passed to the view in `ViewData`.</span></span>

<span data-ttu-id="4ff34-191">接下来，添加用户单击“保存”时执行的代码。</span><span class="sxs-lookup"><span data-stu-id="4ff34-191">Next, add the code that's executed when the user clicks **Save**.</span></span> <span data-ttu-id="4ff34-192">将 `EditPost` 方法替换为以下代码，并添加一个新方法，用于更新 Instructor 实体的 `Courses` 导航属性。</span><span class="sxs-lookup"><span data-stu-id="4ff34-192">Replace the `EditPost` method with the following code, and add a new method that updates the `Courses` navigation property of the Instructor entity.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=1,3,12,13,25,39-40&name=snippet_EditPostCourses)]

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=1-31)]

<span data-ttu-id="4ff34-193">现在的方法签名与 HttpGet `Edit` 方法不同，因此方法名称将从 `EditPost` 变回 `Edit`。</span><span class="sxs-lookup"><span data-stu-id="4ff34-193">The method signature is now different from the HttpGet `Edit` method, so the method name changes from `EditPost` back to `Edit`.</span></span>

<span data-ttu-id="4ff34-194">视图没有 Course 实体的集合，因此模型绑定器无法自动更新 `CourseAssignments` 导航属性。</span><span class="sxs-lookup"><span data-stu-id="4ff34-194">Since the view doesn't have a collection of Course entities, the model binder can't automatically update the `CourseAssignments` navigation property.</span></span> <span data-ttu-id="4ff34-195">可在新的 `UpdateInstructorCourses` 方法中更新 `CourseAssignments` 导航属性，而不必使用模型绑定器。</span><span class="sxs-lookup"><span data-stu-id="4ff34-195">Instead of using the model binder to update the `CourseAssignments` navigation property, you do that in the new `UpdateInstructorCourses` method.</span></span> <span data-ttu-id="4ff34-196">为此，需要从模型绑定中排除 `CourseAssignments` 属性。</span><span class="sxs-lookup"><span data-stu-id="4ff34-196">Therefore you need to exclude the `CourseAssignments` property from model binding.</span></span> <span data-ttu-id="4ff34-197">此操作无需对调用 `TryUpdateModel` 的代码进行任何更改，因为使用的是允许列表重载，并且 `CourseAssignments` 不包括在该列表中。</span><span class="sxs-lookup"><span data-stu-id="4ff34-197">This doesn't require any change to the code that calls `TryUpdateModel` because you're using the whitelisting overload and `CourseAssignments` isn't in the include list.</span></span>

<span data-ttu-id="4ff34-198">如果未选中任何复选框，则 `UpdateInstructorCourses` 中的代码将使用空集合初始化 `CourseAssignments` 导航属性，并返回以下内容：</span><span class="sxs-lookup"><span data-stu-id="4ff34-198">If no check boxes were selected, the code in `UpdateInstructorCourses` initializes the `CourseAssignments` navigation property with an empty collection and returns:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_UpdateCourses&highlight=3-7)]

<span data-ttu-id="4ff34-199">之后，代码会循环访问数据库中的所有课程，并逐一检查当前分配给讲师的课程和视图中处于选中状态的课程。</span><span class="sxs-lookup"><span data-stu-id="4ff34-199">The code then loops through all courses in the database and checks each course against the ones currently assigned to the instructor versus the ones that were selected in the view.</span></span> <span data-ttu-id="4ff34-200">为便于高效查找，后两个集合存储在 `HashSet` 对象中。</span><span class="sxs-lookup"><span data-stu-id="4ff34-200">To facilitate efficient lookups, the latter two collections are stored in `HashSet` objects.</span></span>

<span data-ttu-id="4ff34-201">如果某课程的复选框处于选中状态，但该课程不在 `Instructor.CourseAssignments` 导航属性中，则会将该课程添加到导航属性中的集合中。</span><span class="sxs-lookup"><span data-stu-id="4ff34-201">If the check box for a course was selected but the course isn't in the `Instructor.CourseAssignments` navigation property, the course is added to the collection in the navigation property.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=14-20&name=snippet_UpdateCourses)]

<span data-ttu-id="4ff34-202">如果某课程的复选框未处于选中状态，但该课程存在 `Instructor.CourseAssignments` 导航属性中，则会从导航属性中删除该课程。</span><span class="sxs-lookup"><span data-stu-id="4ff34-202">If the check box for a course wasn't selected, but the course is in the `Instructor.CourseAssignments` navigation property, the course is removed from the navigation property.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=21-29&name=snippet_UpdateCourses)]

### <a name="update-the-instructor-views"></a><span data-ttu-id="4ff34-203">更新讲师视图</span><span class="sxs-lookup"><span data-stu-id="4ff34-203">Update the Instructor views</span></span>

<span data-ttu-id="4ff34-204">在 Views/Instructors/Edit.cshtml 中，通过在“办公室”字段的 `div` 元素之后和“保存”按钮的 `div` 元素之前添加以下代码，以便添加带有一系列复选框的“课程”字段。</span><span class="sxs-lookup"><span data-stu-id="4ff34-204">In *Views/Instructors/Edit.cshtml*, add a **Courses** field with an array of check boxes by adding the following code immediately after the `div` elements for the **Office** field and before the `div` element for the **Save** button.</span></span>

<a id="notepad"></a>
> [!NOTE]
> <span data-ttu-id="4ff34-205">将代码粘贴到 Visual Studio 中时，换行符会发生更改，从而导致代码中断。</span><span class="sxs-lookup"><span data-stu-id="4ff34-205">When you paste the code in Visual Studio, line breaks will be changed in a way that breaks the code.</span></span>  <span data-ttu-id="4ff34-206">按 Ctrl+Z 一次可撤消自动格式设置。</span><span class="sxs-lookup"><span data-stu-id="4ff34-206">Press Ctrl+Z one time to undo the automatic formatting.</span></span>  <span data-ttu-id="4ff34-207">这样可以修复换行符，使其看起来如此处所示。</span><span class="sxs-lookup"><span data-stu-id="4ff34-207">This will fix the line breaks so that they look like what you see here.</span></span> <span data-ttu-id="4ff34-208">缩进不一定要完美，但 `@</tr><tr>`、`@:<td>`、`@:</td>` 和 `@:</tr>` 行必须各成一行（如下所示），否则会出现运行时错误。</span><span class="sxs-lookup"><span data-stu-id="4ff34-208">The indentation doesn't have to be perfect, but the `@</tr><tr>`, `@:<td>`, `@:</td>`, and `@:</tr>` lines must each be on a single line as shown or you'll get a runtime error.</span></span> <span data-ttu-id="4ff34-209">选中新的代码块后，按 Tab 三次，使新代码与现有代码对齐。</span><span class="sxs-lookup"><span data-stu-id="4ff34-209">With the block of new code selected, press Tab three times to line up the new code with the existing code.</span></span> <span data-ttu-id="4ff34-210">可在[此处](https://developercommunity.visualstudio.com/content/problem/147795/razor-editor-malforms-pasted-markup-and-creates-in.html)查看此问题的状态。</span><span class="sxs-lookup"><span data-stu-id="4ff34-210">You can check the status of this problem [here](https://developercommunity.visualstudio.com/content/problem/147795/razor-editor-malforms-pasted-markup-and-creates-in.html).</span></span>

[!code-html[](intro/samples/cu/Views/Instructors/Edit.cshtml?range=35-61)]

<span data-ttu-id="4ff34-211">此代码将创建一个具有三列的 HTML 表。</span><span class="sxs-lookup"><span data-stu-id="4ff34-211">This code creates an HTML table that has three columns.</span></span> <span data-ttu-id="4ff34-212">每个列中都有一个复选框，随后是一段由课程编号和标题组成的描述文字。</span><span class="sxs-lookup"><span data-stu-id="4ff34-212">In each column is a check box followed by a caption that consists of the course number and title.</span></span> <span data-ttu-id="4ff34-213">所有复选框都具有相同的名称，即 selectedCourses，以告知模型绑定器将它们视为一组。</span><span class="sxs-lookup"><span data-stu-id="4ff34-213">The check boxes all have the same name ("selectedCourses"), which informs the model binder that they're to be treated as a group.</span></span> <span data-ttu-id="4ff34-214">每个复选框的值特性被设置为 `CourseID` 的值。</span><span class="sxs-lookup"><span data-stu-id="4ff34-214">The value attribute of each check box is set to the value of `CourseID`.</span></span> <span data-ttu-id="4ff34-215">发布页面时，模型绑定器会向控制器传递一个数组，其中只包括所选复选框的 `CourseID` 值。</span><span class="sxs-lookup"><span data-stu-id="4ff34-215">When the page is posted, the model binder passes an array to the controller that consists of the `CourseID` values for only the check boxes which are selected.</span></span>

<span data-ttu-id="4ff34-216">这些复选框最开始呈现时，对于分配给讲师的课程的复选框，其特性处于选中状态。</span><span class="sxs-lookup"><span data-stu-id="4ff34-216">When the check boxes are initially rendered, those that are for courses assigned to the instructor have checked attributes, which selects them (displays them checked).</span></span>

<span data-ttu-id="4ff34-217">运行应用，选择“讲师”选项卡，然后单击讲师页面上的“编辑”以查看“编辑”页面。</span><span class="sxs-lookup"><span data-stu-id="4ff34-217">Run the app, select the **Instructors** tab, and click **Edit** on an instructor to see the **Edit** page.</span></span>

![带课程信息的讲师“编辑”页](update-related-data/_static/instructor-edit-courses.png)

<span data-ttu-id="4ff34-219">更改某些课程分配并单击“保存”。</span><span class="sxs-lookup"><span data-stu-id="4ff34-219">Change some course assignments and click Save.</span></span> <span data-ttu-id="4ff34-220">所作更改将反映在索引页上。</span><span class="sxs-lookup"><span data-stu-id="4ff34-220">The changes you make are reflected on the Index page.</span></span>

> [!NOTE]
> <span data-ttu-id="4ff34-221">此处所使用的编辑讲师课程数据的方法适用于数量有限的课程。</span><span class="sxs-lookup"><span data-stu-id="4ff34-221">The approach taken here to edit instructor course data works well when there's a limited number of courses.</span></span> <span data-ttu-id="4ff34-222">若是远大于此的集合，则需要使用不同的 UI 和不同的更新方法。</span><span class="sxs-lookup"><span data-stu-id="4ff34-222">For collections that are much larger, a different UI and a different updating method would be required.</span></span>

## <a name="update-the-delete-page"></a><span data-ttu-id="4ff34-223">更新“删除”页</span><span class="sxs-lookup"><span data-stu-id="4ff34-223">Update the Delete page</span></span>

<span data-ttu-id="4ff34-224">在 InstructorsController.cs 中，删除 `DeleteConfirmed` 方法，并在其位置插入以下代码。</span><span class="sxs-lookup"><span data-stu-id="4ff34-224">In *InstructorsController.cs*, delete the `DeleteConfirmed` method and insert the following code in its place.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?highlight=5-7,9-12&name=snippet_DeleteConfirmed)]

<span data-ttu-id="4ff34-225">此代码会更改以下内容：</span><span class="sxs-lookup"><span data-stu-id="4ff34-225">This code makes the following changes:</span></span>

* <span data-ttu-id="4ff34-226">对 `CourseAssignments` 导航属性执行预先加载。</span><span class="sxs-lookup"><span data-stu-id="4ff34-226">Does eager loading for the `CourseAssignments` navigation property.</span></span>  <span data-ttu-id="4ff34-227">必须包括此内容，否则 EF 不知道相关的 `CourseAssignment` 实体，也不会删除它们。</span><span class="sxs-lookup"><span data-stu-id="4ff34-227">You have to include this or EF won't know about related `CourseAssignment` entities and won't delete them.</span></span>  <span data-ttu-id="4ff34-228">为避免在此处阅读它们，可以在数据库中配置级联删除。</span><span class="sxs-lookup"><span data-stu-id="4ff34-228">To avoid needing to read them here you could configure cascade delete in the database.</span></span>

* <span data-ttu-id="4ff34-229">如果要删除的讲师被指派为任何系的管理员，则需从这些系中删除该讲师分配。</span><span class="sxs-lookup"><span data-stu-id="4ff34-229">If the instructor to be deleted is assigned as administrator of any departments, removes the instructor assignment from those departments.</span></span>

## <a name="add-office-location-and-courses-to-the-create-page"></a><span data-ttu-id="4ff34-230">向创建页添加办公室位置和课程</span><span class="sxs-lookup"><span data-stu-id="4ff34-230">Add office location and courses to the Create page</span></span>

<span data-ttu-id="4ff34-231">在 InstructorsController.cs 中，删除 HttpGet 和 HttpPost `Create` 方法，然后在其位置添加以下代码：</span><span class="sxs-lookup"><span data-stu-id="4ff34-231">In *InstructorsController.cs*, delete the HttpGet and HttpPost `Create` methods, and then add the following code in their place:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/InstructorsController.cs?name=snippet_Create&highlight=3-5,12,14-22,29)]

<span data-ttu-id="4ff34-232">此代码与 `Edit` 方法中所示内容类似，只是最开始未选择任何课程。</span><span class="sxs-lookup"><span data-stu-id="4ff34-232">This code is similar to what you saw for the `Edit` methods except that initially no courses are selected.</span></span> <span data-ttu-id="4ff34-233">HttpGet `Create` 方法调用 `PopulateAssignedCourseData` 方法不是因为可能有课程处于选中状态，而是为了在视图中为 `foreach` 循环提供空集合（否则该视图代码将引发空引用异常）。</span><span class="sxs-lookup"><span data-stu-id="4ff34-233">The HttpGet `Create` method calls the `PopulateAssignedCourseData` method not because there might be courses selected but in order to provide an empty collection for the `foreach` loop in the view (otherwise the view code would throw a null reference exception).</span></span>

<span data-ttu-id="4ff34-234">检查是否存在验证错误并向数据库添加新讲师之前，HttpPost `Create` 方法会将每个选定课程添加到 `CourseAssignments` 导航属性。</span><span class="sxs-lookup"><span data-stu-id="4ff34-234">The HttpPost `Create` method adds each selected course to the `CourseAssignments` navigation property before it checks for validation errors and adds the new instructor to the database.</span></span> <span data-ttu-id="4ff34-235">即使存在模型错误也会添加课程，因此出现模型错误（例如用户键入了无效的日期）并且页面重新显示并出现错误消息时，所作的任何课程选择都会自动还原。</span><span class="sxs-lookup"><span data-stu-id="4ff34-235">Courses are added even if there are model errors so that when there are model errors (for an example, the user keyed an invalid date), and the page is redisplayed with an error message, any course selections that were made are automatically restored.</span></span>

<span data-ttu-id="4ff34-236">请注意，为了能够向 `CourseAssignments` 导航属性添加课程，必须将该属性初始化为空集合：</span><span class="sxs-lookup"><span data-stu-id="4ff34-236">Notice that in order to be able to add courses to the `CourseAssignments` navigation property you have to initialize the property as an empty collection:</span></span>

```csharp
instructor.CourseAssignments = new List<CourseAssignment>();
```

<span data-ttu-id="4ff34-237">除了在控制器代码中进行此操作之外，还可以在“导师”模型中进行此操作，方法是将该属性更改为不存在集合时自动创建集合，如以下示例所示：</span><span class="sxs-lookup"><span data-stu-id="4ff34-237">As an alternative to doing this in controller code, you could do it in the Instructor model by changing the property getter to automatically create the collection if it doesn't exist, as shown in the following example:</span></span>

```csharp
private ICollection<CourseAssignment> _courseAssignments;
public ICollection<CourseAssignment> CourseAssignments
{
    get
    {
        return _courseAssignments ?? (_courseAssignments = new List<CourseAssignment>());
    }
    set
    {
        _courseAssignments = value;
    }
}
```

<span data-ttu-id="4ff34-238">如果通过这种方式修改 `CourseAssignments` 属性，则可以删除控制器中的显式属性初始化代码。</span><span class="sxs-lookup"><span data-stu-id="4ff34-238">If you modify the `CourseAssignments` property in this way, you can remove the explicit property initialization code in the controller.</span></span>

<span data-ttu-id="4ff34-239">在 Views/Instructor/Create.cshtml 中，添加一个办公室位置文本框和课程的复选框，然后按“提交”按钮。</span><span class="sxs-lookup"><span data-stu-id="4ff34-239">In *Views/Instructor/Create.cshtml*, add an office location text box and check boxes for courses before the Submit button.</span></span> <span data-ttu-id="4ff34-240">与“编辑”页面中一样，[如果粘贴代码时 Visual Studio 重新设置了其格式，则修复该格式](#notepad)。</span><span class="sxs-lookup"><span data-stu-id="4ff34-240">As in the case of the Edit page, [fix the formatting if Visual Studio reformats the code when you paste it](#notepad).</span></span>

[!code-html[](intro/samples/cu/Views/Instructors/Create.cshtml?range=29-61)]

<span data-ttu-id="4ff34-241">通过运行应用并创建讲师来进行测试。</span><span class="sxs-lookup"><span data-stu-id="4ff34-241">Test by running the app and creating an instructor.</span></span>

## <a name="handling-transactions"></a><span data-ttu-id="4ff34-242">处理事务</span><span class="sxs-lookup"><span data-stu-id="4ff34-242">Handling Transactions</span></span>

<span data-ttu-id="4ff34-243">如 [CRUD 教程](crud.md)中所述，Entity Framework 隐式实现事务。</span><span class="sxs-lookup"><span data-stu-id="4ff34-243">As explained in the [CRUD tutorial](crud.md), the Entity Framework implicitly implements transactions.</span></span> <span data-ttu-id="4ff34-244">如果需要更多控制操作（例如，如果想要在事务中包含在 Entity Framework 外部完成的操作），请参阅[事务](/ef/core/saving/transactions)。</span><span class="sxs-lookup"><span data-stu-id="4ff34-244">For scenarios where you need more control -- for example, if you want to include operations done outside of Entity Framework in a transaction -- see [Transactions](/ef/core/saving/transactions).</span></span>

## <a name="summary"></a><span data-ttu-id="4ff34-245">总结</span><span class="sxs-lookup"><span data-stu-id="4ff34-245">Summary</span></span>

<span data-ttu-id="4ff34-246">处理相关数据的介绍至此已告一段落。</span><span class="sxs-lookup"><span data-stu-id="4ff34-246">You have now completed the introduction to working with related data.</span></span> <span data-ttu-id="4ff34-247">下一个教程将介绍如何处理并发冲突。</span><span class="sxs-lookup"><span data-stu-id="4ff34-247">In the next tutorial you'll see how to handle concurrency conflicts.</span></span>

::: moniker-end

> [!div class="step-by-step"]
> <span data-ttu-id="4ff34-248">[上一页](read-related-data.md)
> [下一页](concurrency.md)</span><span class="sxs-lookup"><span data-stu-id="4ff34-248">[Previous](read-related-data.md)
[Next](concurrency.md)</span></span>
