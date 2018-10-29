---
title: ASP.NET Core MVC 和 EF Core - 并发 - 第 8 个教程（共 10 个）
author: rick-anderson
description: 本教程介绍如何处理多个用户同时更新同一实体时出现的冲突。
ms.author: tdykstra
ms.custom: mvc
ms.date: 10/24/2018
uid: data/ef-mvc/concurrency
ms.openlocfilehash: 0ae566a76a2ef656843452ed537b8fdfbddaed22
ms.sourcegitcommit: 4d74644f11e0dac52b4510048490ae731c691496
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/25/2018
ms.locfileid: "50090896"
---
# <a name="aspnet-core-mvc-with-ef-core---concurrency---8-of-10"></a><span data-ttu-id="0eccc-103">ASP.NET Core MVC 和 EF Core - 并发 - 第 8 个教程（共 10 个）</span><span class="sxs-lookup"><span data-stu-id="0eccc-103">ASP.NET Core MVC with EF Core - Concurrency - 8 of 10</span></span>

[!INCLUDE [RP better than MVC](~/includes/RP-EF/rp-over-mvc-21.md)]

::: moniker range="= aspnetcore-2.0"

<span data-ttu-id="0eccc-104">作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="0eccc-104">By [Tom Dykstra](https://github.com/tdykstra) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="0eccc-105">Contoso 大学示例 web 应用程序演示如何使用 Entity Framework Core 和 Visual Studio 创建 ASP.NET Core MVC web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="0eccc-105">The Contoso University sample web application demonstrates how to create ASP.NET Core MVC web applications using Entity Framework Core and Visual Studio.</span></span> <span data-ttu-id="0eccc-106">若要了解教程系列，请参阅[本系列中的第一个教程](intro.md)。</span><span class="sxs-lookup"><span data-stu-id="0eccc-106">For information about the tutorial series, see [the first tutorial in the series](intro.md).</span></span>

<span data-ttu-id="0eccc-107">在之前的教程中，你学习了如何更新数据。</span><span class="sxs-lookup"><span data-stu-id="0eccc-107">In earlier tutorials, you learned how to update data.</span></span> <span data-ttu-id="0eccc-108">本教程介绍如何处理多个用户同时更新同一实体时出现的冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-108">This tutorial shows how to handle conflicts when multiple users update the same entity at the same time.</span></span>

<span data-ttu-id="0eccc-109">你将创建可处理 Department 实体的 Web 页面并处理并发错误。</span><span class="sxs-lookup"><span data-stu-id="0eccc-109">You'll create web pages that work with the Department entity and handle concurrency errors.</span></span> <span data-ttu-id="0eccc-110">下图显示了“编辑”和“删除”页面，包括发生并发冲突时显示的一些消息。</span><span class="sxs-lookup"><span data-stu-id="0eccc-110">The following illustrations show the Edit and Delete pages, including some messages that are displayed if a concurrency conflict occurs.</span></span>

![“院系编辑”页](concurrency/_static/edit-error.png)

![“院系删除”页](concurrency/_static/delete-error.png)

## <a name="concurrency-conflicts"></a><span data-ttu-id="0eccc-113">并发冲突</span><span class="sxs-lookup"><span data-stu-id="0eccc-113">Concurrency conflicts</span></span>

<span data-ttu-id="0eccc-114">当某用户显示实体数据以对其进行编辑，而另一用户在上一用户的更改写入数据库之前更新同一实体的数据时，会发生并发冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-114">A concurrency conflict occurs when one user displays an entity's data in order to edit it, and then another user updates the same entity's data before the first user's change is written to the database.</span></span> <span data-ttu-id="0eccc-115">如果不启用此类冲突的检测，则最后更新数据库的人员将覆盖其他用户的更改。</span><span class="sxs-lookup"><span data-stu-id="0eccc-115">If you don't enable the detection of such conflicts, whoever updates the database last overwrites the other user's changes.</span></span> <span data-ttu-id="0eccc-116">在许多应用程序中，此风险是可接受的：如果用户很少或更新很少，或者一些更改被覆盖并不重要，则并发编程可能弊大于利。</span><span class="sxs-lookup"><span data-stu-id="0eccc-116">In many applications, this risk is acceptable: if there are few users, or few updates, or if isn't really critical if some changes are overwritten, the cost of programming for concurrency might outweigh the benefit.</span></span> <span data-ttu-id="0eccc-117">在此情况下，不必配置应用程序来处理并发冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-117">In that case, you don't have to configure the application to handle concurrency conflicts.</span></span>

### <a name="pessimistic-concurrency-locking"></a><span data-ttu-id="0eccc-118">悲观并发（锁定）</span><span class="sxs-lookup"><span data-stu-id="0eccc-118">Pessimistic concurrency (locking)</span></span>

<span data-ttu-id="0eccc-119">如果应用程序确实需要防止并发情况下出现意外数据丢失，一种方法是使用数据库锁定。</span><span class="sxs-lookup"><span data-stu-id="0eccc-119">If your application does need to prevent accidental data loss in concurrency scenarios, one way to do that is to use database locks.</span></span> <span data-ttu-id="0eccc-120">这称为悲观并发。</span><span class="sxs-lookup"><span data-stu-id="0eccc-120">This is called pessimistic concurrency.</span></span> <span data-ttu-id="0eccc-121">例如，在从数据库读取一行内容之前，请求锁定为只读或更新访问。</span><span class="sxs-lookup"><span data-stu-id="0eccc-121">For example, before you read a row from a database, you request a lock for read-only or for update access.</span></span> <span data-ttu-id="0eccc-122">如果将一行锁定为更新访问，则其他用户无法将该行锁定为只读或更新访问，因为他们得到的是正在更改的数据的副本。</span><span class="sxs-lookup"><span data-stu-id="0eccc-122">If you lock a row for update access, no other users are allowed to lock the row either for read-only or update access, because they would get a copy of data that's in the process of being changed.</span></span> <span data-ttu-id="0eccc-123">如果将一行锁定为只读访问，则其他人也可将其锁定为只读访问，但不能进行更新。</span><span class="sxs-lookup"><span data-stu-id="0eccc-123">If you lock a row for read-only access, others can also lock it for read-only access but not for update.</span></span>

<span data-ttu-id="0eccc-124">管理锁定有缺点。</span><span class="sxs-lookup"><span data-stu-id="0eccc-124">Managing locks has disadvantages.</span></span> <span data-ttu-id="0eccc-125">编程可能很复杂。</span><span class="sxs-lookup"><span data-stu-id="0eccc-125">It can be complex to program.</span></span> <span data-ttu-id="0eccc-126">它需要大量的数据库管理资源，且随着应用程序用户数量的增加，可能会导致性能问题。</span><span class="sxs-lookup"><span data-stu-id="0eccc-126">It requires significant database management resources, and it can cause performance problems as the number of users of an application increases.</span></span> <span data-ttu-id="0eccc-127">由于这些原因，并不是所有的数据库管理系统都支持悲观并发。</span><span class="sxs-lookup"><span data-stu-id="0eccc-127">For these reasons, not all database management systems support pessimistic concurrency.</span></span> <span data-ttu-id="0eccc-128">Entity Framework Core 未提供对它的内置支持，并且本教程不展示其实现方式。</span><span class="sxs-lookup"><span data-stu-id="0eccc-128">Entity Framework Core provides no built-in support for it, and this tutorial doesn't show you how to implement it.</span></span>

### <a name="optimistic-concurrency"></a><span data-ttu-id="0eccc-129">开放式并发</span><span class="sxs-lookup"><span data-stu-id="0eccc-129">Optimistic Concurrency</span></span>

<span data-ttu-id="0eccc-130">悲观并发的替代方法是乐观并发。</span><span class="sxs-lookup"><span data-stu-id="0eccc-130">The alternative to pessimistic concurrency is optimistic concurrency.</span></span> <span data-ttu-id="0eccc-131">悲观并发是指允许发生并发冲突，并在并发冲突发生时作出正确反应。</span><span class="sxs-lookup"><span data-stu-id="0eccc-131">Optimistic concurrency means allowing concurrency conflicts to happen, and then reacting appropriately if they do.</span></span> <span data-ttu-id="0eccc-132">例如，Jane 访问“院系编辑”页面，并将英语系的预算从 350,000.00 美元更改为 0.00 美元。</span><span class="sxs-lookup"><span data-stu-id="0eccc-132">For example, Jane visits the Department Edit page and changes the Budget amount for the English department from $350,000.00 to $0.00.</span></span>

![将预算更改为零](concurrency/_static/change-budget.png)

<span data-ttu-id="0eccc-134">在 Jane 单击“保存”之前，John 访问了相同页面，并将开始日期字段从 2007/1/9 更改为 2013/1/9。</span><span class="sxs-lookup"><span data-stu-id="0eccc-134">Before Jane clicks **Save**, John visits the same page and changes the Start Date field from 9/1/2007 to 9/1/2013.</span></span>

![将开始日期更改为 2013](concurrency/_static/change-date.png)

<span data-ttu-id="0eccc-136">Jane 先单击“保存”，并在浏览器返回索引页时看到她的更改。</span><span class="sxs-lookup"><span data-stu-id="0eccc-136">Jane clicks **Save** first and sees her change when the browser returns to the Index page.</span></span>

![预算已更改为零](concurrency/_static/budget-zero.png)

<span data-ttu-id="0eccc-138">然后 John 单击“编辑”页面上的“保存”，但页面上预算仍显示为 350,000.00 美元。</span><span class="sxs-lookup"><span data-stu-id="0eccc-138">Then John clicks **Save** on an Edit page that still shows a budget of $350,000.00.</span></span> <span data-ttu-id="0eccc-139">接下来的情况取决于并发冲突的处理方式。</span><span class="sxs-lookup"><span data-stu-id="0eccc-139">What happens next is determined by how you handle concurrency conflicts.</span></span>

<span data-ttu-id="0eccc-140">其中一些选项包括：</span><span class="sxs-lookup"><span data-stu-id="0eccc-140">Some of the options include the following:</span></span>

* <span data-ttu-id="0eccc-141">可以跟踪用户已修改的属性，并仅更新数据库中相应的列。</span><span class="sxs-lookup"><span data-stu-id="0eccc-141">You can keep track of which property a user has modified and update only the corresponding columns in the database.</span></span>

     <span data-ttu-id="0eccc-142">在示例方案中，不会有数据丢失，因为是由两个用户更新不同的属性。</span><span class="sxs-lookup"><span data-stu-id="0eccc-142">In the example scenario, no data would be lost, because different properties were updated by the two users.</span></span> <span data-ttu-id="0eccc-143">下次有人浏览英语系时，将看到 Jane 和 John 两个人的更改 - 开始日期为 9/1/2013，预算为零美元。</span><span class="sxs-lookup"><span data-stu-id="0eccc-143">The next time someone browses the English department, they will see both Jane's and John's changes -- a start date of 9/1/2013 and a budget of zero dollars.</span></span> <span data-ttu-id="0eccc-144">这种更新方法可减少可能导致数据丢失的冲突次数，但是如果对实体的同一属性进行竞争性更改，则数据难免会丢失。</span><span class="sxs-lookup"><span data-stu-id="0eccc-144">This method of updating can reduce the number of conflicts that could result in data loss, but it can't avoid data loss if competing changes are made to the same property of an entity.</span></span> <span data-ttu-id="0eccc-145">Entity Framework 是否以这种方式工作取决于更新代码的实现方式。</span><span class="sxs-lookup"><span data-stu-id="0eccc-145">Whether the Entity Framework works this way depends on how you implement your update code.</span></span> <span data-ttu-id="0eccc-146">通常不适合在 Web 应用程序中使用，因为它要求保持大量的状态，以便跟踪实体的所有原始属性值以及新值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-146">It's often not practical in a web application, because it can require that you maintain large amounts of state in order to keep track of all original property values for an entity as well as new values.</span></span> <span data-ttu-id="0eccc-147">维护大量的状态可能会影响应用程序的性能，因为它需要服务器资源或必须包含在网页本身（例如隐藏字段）或 Cookie 中。</span><span class="sxs-lookup"><span data-stu-id="0eccc-147">Maintaining large amounts of state can affect application performance because it either requires server resources or must be included in the web page itself (for example, in hidden fields) or in a cookie.</span></span>

* <span data-ttu-id="0eccc-148">可让 John 的更改覆盖 Jane 的更改。</span><span class="sxs-lookup"><span data-stu-id="0eccc-148">You can let John's change overwrite Jane's change.</span></span>

     <span data-ttu-id="0eccc-149">下次有人浏览英语系时，将看到 2013/1/9 和还原的值 350,000.00 美元。</span><span class="sxs-lookup"><span data-stu-id="0eccc-149">The next time someone browses the English department, they will see 9/1/2013 and the restored $350,000.00 value.</span></span> <span data-ttu-id="0eccc-150">这称为“客户端优先”或“最后一个优先”。</span><span class="sxs-lookup"><span data-stu-id="0eccc-150">This is called a *Client Wins* or *Last in Wins* scenario.</span></span> <span data-ttu-id="0eccc-151">（客户端的所有值优先于数据存储的值。）正如本部分的介绍所述，如果不为并发处理编写任何代码，则自动执行此操作。</span><span class="sxs-lookup"><span data-stu-id="0eccc-151">(All values from the client take precedence over what's in the data store.) As noted in the introduction to this section, if you don't do any coding for concurrency handling, this will happen automatically.</span></span>

* <span data-ttu-id="0eccc-152">可以阻止在数据库中更新 John 的更改。</span><span class="sxs-lookup"><span data-stu-id="0eccc-152">You can prevent John's change from being updated in the database.</span></span>

     <span data-ttu-id="0eccc-153">通常，将显示一条错误消息，向他显示数据的当前状态，并允许他重新应用其更改（若他仍想更改）。</span><span class="sxs-lookup"><span data-stu-id="0eccc-153">Typically, you would display an error message, show him the current state of the data, and allow him to reapply his changes if he still wants to make them.</span></span> <span data-ttu-id="0eccc-154">这称为“存储优先”方案。</span><span class="sxs-lookup"><span data-stu-id="0eccc-154">This is called a *Store Wins* scenario.</span></span> <span data-ttu-id="0eccc-155">（数据存储值优先于客户端提交的值。）本教程将执行“存储优先”方案。</span><span class="sxs-lookup"><span data-stu-id="0eccc-155">(The data-store values take precedence over the values submitted by the client.) You'll implement the Store Wins scenario in this tutorial.</span></span> <span data-ttu-id="0eccc-156">此方法可确保用户在未收到具体发生内容的警报时，不会覆盖任何更改。</span><span class="sxs-lookup"><span data-stu-id="0eccc-156">This method ensures that no changes are overwritten without a user being alerted to what's happening.</span></span>

### <a name="detecting-concurrency-conflicts"></a><span data-ttu-id="0eccc-157">检测并发冲突</span><span class="sxs-lookup"><span data-stu-id="0eccc-157">Detecting concurrency conflicts</span></span>

<span data-ttu-id="0eccc-158">你可以通过处理 Entity Framework 引发的 `DbConcurrencyException` 异常来解决冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-158">You can resolve conflicts by handling `DbConcurrencyException` exceptions that the Entity Framework throws.</span></span> <span data-ttu-id="0eccc-159">为了知道何时引发这些异常，Entity Framework 必须能够检测到冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-159">In order to know when to throw these exceptions, the Entity Framework must be able to detect conflicts.</span></span> <span data-ttu-id="0eccc-160">因此，你必须正确配置数据库和数据模型。</span><span class="sxs-lookup"><span data-stu-id="0eccc-160">Therefore, you must configure the database and the data model appropriately.</span></span> <span data-ttu-id="0eccc-161">启用冲突检测的某些选项包括：</span><span class="sxs-lookup"><span data-stu-id="0eccc-161">Some options for enabling conflict detection include the following:</span></span>

* <span data-ttu-id="0eccc-162">数据库表中包含一个可用于确定某行更改时间的跟踪列。</span><span class="sxs-lookup"><span data-stu-id="0eccc-162">In the database table, include a tracking column that can be used to determine when a row has been changed.</span></span> <span data-ttu-id="0eccc-163">然后可配置 Entity Framework，将该列包含在 SQL Update 或 Delete 命令的 Where 子句中。</span><span class="sxs-lookup"><span data-stu-id="0eccc-163">You can then configure the Entity Framework to include that column in the Where clause of SQL Update or Delete commands.</span></span>

     <span data-ttu-id="0eccc-164">跟踪列的数据类型通常是 `rowversion`。</span><span class="sxs-lookup"><span data-stu-id="0eccc-164">The data type of the tracking column is typically `rowversion`.</span></span> <span data-ttu-id="0eccc-165">`rowversion` 值是一个序列号，该编号随着每次行的更新递增。</span><span class="sxs-lookup"><span data-stu-id="0eccc-165">The `rowversion` value is a sequential number that's incremented each time the row is updated.</span></span> <span data-ttu-id="0eccc-166">在 Update 或 Delete 命令中，Where 子句包含跟踪列的原始值（原始行版本）。</span><span class="sxs-lookup"><span data-stu-id="0eccc-166">In an Update or Delete command, the Where clause includes the original value of the tracking column (the original row version) .</span></span> <span data-ttu-id="0eccc-167">如果正在更新的行已被其他用户更改，则 `rowversion` 列中的值与原始值不同，这导致 Update 或 Delete 语句由于 Where 子句而找不到要更新的行。</span><span class="sxs-lookup"><span data-stu-id="0eccc-167">If the row being updated has been changed by another user, the value in the `rowversion` column is different than the original value, so the Update or Delete statement can't find the row to update because of the Where clause.</span></span> <span data-ttu-id="0eccc-168">当 Entity Framework 发现 Update 或 Delete 命令没有更新行（即受影响的行数为零）时，便将其解释为并发冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-168">When the Entity Framework finds that no rows have been updated by the Update or Delete command (that is, when the number of affected rows is zero), it interprets that as a concurrency conflict.</span></span>

* <span data-ttu-id="0eccc-169">配置 Entity Framework，在 Update 或 Delete 命令的 Where 子句中包含表中每个列的原始值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-169">Configure the Entity Framework to include the original values of every column in the table in the Where clause of Update and Delete commands.</span></span>

     <span data-ttu-id="0eccc-170">与第一个选项一样，如果行自第一次读取后发生更改，Where 子句将不返回要更新的行，Entity Framework 会将其解释为并发冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-170">As in the first option, if anything in the row has changed since the row was first read, the Where clause won't return a row to update, which the Entity Framework interprets as a concurrency conflict.</span></span> <span data-ttu-id="0eccc-171">对于包含许多列的数据库表，此方法可能导致非常多的 Where 子句，并且可能需要维持大量的状态。</span><span class="sxs-lookup"><span data-stu-id="0eccc-171">For database tables that have many columns, this approach can result in very large Where clauses, and can require that you maintain large amounts of state.</span></span> <span data-ttu-id="0eccc-172">如前所述，维持大量的状态会影响应用程序的性能。</span><span class="sxs-lookup"><span data-stu-id="0eccc-172">As noted earlier, maintaining large amounts of state can affect application performance.</span></span> <span data-ttu-id="0eccc-173">因此通常不建议使用此方法，并且它也不是本教程中使用的方法。</span><span class="sxs-lookup"><span data-stu-id="0eccc-173">Therefore this approach is generally not recommended, and it isn't the method used in this tutorial.</span></span>

     <span data-ttu-id="0eccc-174">如果确实想要实现此并发方法，必须通过向要跟踪并发的实体添加 `ConcurrencyCheck` 属性来标记它们的所有非主键属性。</span><span class="sxs-lookup"><span data-stu-id="0eccc-174">If you do want to implement this approach to concurrency, you have to mark all non-primary-key properties in the entity you want to track concurrency for by adding the `ConcurrencyCheck` attribute to them.</span></span> <span data-ttu-id="0eccc-175">所作更改使 Entity Framework 能够将所有列包含在 Update 和 Delete 语句的 SQL Where 子句中。</span><span class="sxs-lookup"><span data-stu-id="0eccc-175">That change enables the Entity Framework to include all columns in the SQL Where clause of Update and Delete statements.</span></span>

<span data-ttu-id="0eccc-176">在本教程的其余部分，将向 Department 实体添加 `rowversion` 跟踪属性，创建控制器和视图，并进行测试以验证是否一切正常工作。</span><span class="sxs-lookup"><span data-stu-id="0eccc-176">In the remainder of this tutorial you'll add a `rowversion` tracking property to the Department entity, create a controller and views, and test to verify that everything works correctly.</span></span>

## <a name="add-a-tracking-property-to-the-department-entity"></a><span data-ttu-id="0eccc-177">向 Department 实体添加跟踪属性</span><span class="sxs-lookup"><span data-stu-id="0eccc-177">Add a tracking property to the Department entity</span></span>

<span data-ttu-id="0eccc-178">在 Models/Department.cs 中，添加名为 RowVersion 的跟踪属性：</span><span class="sxs-lookup"><span data-stu-id="0eccc-178">In *Models/Department.cs*, add a tracking property named RowVersion:</span></span>

[!code-csharp[](intro/samples/cu/Models/Department.cs?name=snippet_Final&highlight=26,27)]

<span data-ttu-id="0eccc-179">`Timestamp` 属性指定此列将包含在发送到数据库的 Update 和 Delete 命令的 Where 子句中。</span><span class="sxs-lookup"><span data-stu-id="0eccc-179">The `Timestamp` attribute specifies that this column will be included in the Where clause of Update and Delete commands sent to the database.</span></span> <span data-ttu-id="0eccc-180">该属性称为 `Timestamp`，因为 SQL Server 的之前版本在 SQL `rowversion` 类型将其替换之前使用 SQL `timestamp` 数据类型。</span><span class="sxs-lookup"><span data-stu-id="0eccc-180">The attribute is called `Timestamp` because previous versions of SQL Server used a SQL `timestamp` data type before the SQL `rowversion` replaced it.</span></span> <span data-ttu-id="0eccc-181">用于 `rowversion` 的 .NET 类型为字节数组。</span><span class="sxs-lookup"><span data-stu-id="0eccc-181">The .NET type for `rowversion` is a byte array.</span></span>

<span data-ttu-id="0eccc-182">如果更愿意使用 Fluent API，可使用 `IsConcurrencyToken` 方法（路径为 Data/SchoolContext.cs）指定跟踪属性，如下例所示：</span><span class="sxs-lookup"><span data-stu-id="0eccc-182">If you prefer to use the fluent API, you can use the `IsConcurrencyToken` method (in *Data/SchoolContext.cs*) to specify the tracking property, as shown in the following example:</span></span>

```csharp
modelBuilder.Entity<Department>()
    .Property(p => p.RowVersion).IsConcurrencyToken();
```

<span data-ttu-id="0eccc-183">通过添加属性，更改了数据库模型，因此需要再执行一次迁移。</span><span class="sxs-lookup"><span data-stu-id="0eccc-183">By adding a property you changed the database model, so you need to do another migration.</span></span>

<span data-ttu-id="0eccc-184">保存更改并生成项目，然后在命令窗口中输入以下命令：</span><span class="sxs-lookup"><span data-stu-id="0eccc-184">Save your changes and build the project, and then enter the following commands in the command window:</span></span>

```console
dotnet ef migrations add RowVersion
```

```console
dotnet ef database update
```

## <a name="create-a-departments-controller-and-views"></a><span data-ttu-id="0eccc-185">创建“院系”控制器和视图</span><span class="sxs-lookup"><span data-stu-id="0eccc-185">Create a Departments controller and views</span></span>

<span data-ttu-id="0eccc-186">像之前在学生、课程和讲师教程中操作的那样，为“院系”控制器和视图创建基架。</span><span class="sxs-lookup"><span data-stu-id="0eccc-186">Scaffold a Departments controller and views as you did earlier for Students, Courses, and Instructors.</span></span>

![“为院系”创建基架](concurrency/_static/add-departments-controller.png)

<span data-ttu-id="0eccc-188">在 DepartmentsController.cs 文件中，将出现的 4 次“FirstMidName”更改为“FullName”，以便院系管理员下拉列表将包含讲师的全名，而不仅仅是姓氏。</span><span class="sxs-lookup"><span data-stu-id="0eccc-188">In the *DepartmentsController.cs* file, change all four occurrences of "FirstMidName" to "FullName" so that the department administrator drop-down lists will contain the full name of the instructor rather than just the last name.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_Dropdown)]

## <a name="update-the-departments-index-view"></a><span data-ttu-id="0eccc-189">更新“院系索引”视图</span><span class="sxs-lookup"><span data-stu-id="0eccc-189">Update the Departments Index view</span></span>

<span data-ttu-id="0eccc-190">基架引擎在索引视图中创建 RowVersion 列，但不应显示该字段。</span><span class="sxs-lookup"><span data-stu-id="0eccc-190">The scaffolding engine created a RowVersion column in the Index view, but that field shouldn't be displayed.</span></span>

<span data-ttu-id="0eccc-191">将 Views/Departments/Index.cshtml 中的代码替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="0eccc-191">Replace the code in *Views/Departments/Index.cshtml* with the following code.</span></span>

[!code-html[](intro/samples/cu/Views/Departments/Index.cshtml?highlight=4,7,44)]

<span data-ttu-id="0eccc-192">这会将标题更改为“院系”，删除 RowVersion 列，并显示全名（而非管理员的名字）。</span><span class="sxs-lookup"><span data-stu-id="0eccc-192">This changes the heading to "Departments", deletes the RowVersion column, and shows full name instead of first name for the administrator.</span></span>

## <a name="update-the-edit-methods-in-the-departments-controller"></a><span data-ttu-id="0eccc-193">更新“院系”控制器中的编辑方法</span><span class="sxs-lookup"><span data-stu-id="0eccc-193">Update the Edit methods in the Departments controller</span></span>

<span data-ttu-id="0eccc-194">在 HttpGet `Edit` 方法和 `Details` 方法中，添加 `AsNoTracking`。</span><span class="sxs-lookup"><span data-stu-id="0eccc-194">In both the HttpGet `Edit` method and the `Details` method, add `AsNoTracking`.</span></span> <span data-ttu-id="0eccc-195">在 HttpGet `Edit` 方法中，为管理员添加预先加载。</span><span class="sxs-lookup"><span data-stu-id="0eccc-195">In the HttpGet `Edit` method, add eager loading for the Administrator.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EagerLoading&highlight=2,3)]

<span data-ttu-id="0eccc-196">将 HttpPost `Edit` 方法的现有代码替换为以下代码：</span><span class="sxs-lookup"><span data-stu-id="0eccc-196">Replace the existing code for the HttpPost `Edit` method with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_EditPost)]

<span data-ttu-id="0eccc-197">代码先尝试读取要更新的院系。</span><span class="sxs-lookup"><span data-stu-id="0eccc-197">The code begins by trying to read the department to be updated.</span></span> <span data-ttu-id="0eccc-198">如果 `SingleOrDefaultAsync` 方法返回 NULL，则该院系已被另一用户删除。</span><span class="sxs-lookup"><span data-stu-id="0eccc-198">If the `SingleOrDefaultAsync` method returns null, the department was deleted by another user.</span></span> <span data-ttu-id="0eccc-199">此情况下，代码将使用已发布的表单值创建院系实体，以便“编辑”页重新显示错误消息。</span><span class="sxs-lookup"><span data-stu-id="0eccc-199">In that case the code uses the posted form values to create a department entity so that the Edit page can be redisplayed with an error message.</span></span> <span data-ttu-id="0eccc-200">或者，如果仅显示错误消息而未重新显示院系字段，则不必重新创建 Department 实体。</span><span class="sxs-lookup"><span data-stu-id="0eccc-200">As an alternative, you wouldn't have to re-create the department entity if you display only an error message without redisplaying the department fields.</span></span>

<span data-ttu-id="0eccc-201">该视图将原始 `RowVersion` 值存储在隐藏字段中，且此方法在 `rowVersion` 参数中接收该值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-201">The view stores the original `RowVersion` value in a hidden field, and this method receives that value in the `rowVersion` parameter.</span></span> <span data-ttu-id="0eccc-202">在调用 `SaveChanges` 之前，必须将该原始 `RowVersion` 属性值置于实体的 `OriginalValues` 集合中。</span><span class="sxs-lookup"><span data-stu-id="0eccc-202">Before you call `SaveChanges`, you have to put that original `RowVersion` property value in the `OriginalValues` collection for the entity.</span></span>

```csharp
_context.Entry(departmentToUpdate).Property("RowVersion").OriginalValue = rowVersion;
```

<span data-ttu-id="0eccc-203">然后，当 Entity Framework 创建 SQL UPDATE 命令时，该命令将包含一个 WHERE 子句，用于查找具有原始 `RowVersion` 值的行。</span><span class="sxs-lookup"><span data-stu-id="0eccc-203">Then when the Entity Framework creates a SQL UPDATE command, that command will include a WHERE clause that looks for a row that has the original `RowVersion` value.</span></span> <span data-ttu-id="0eccc-204">如果没有行受到 UPDATE 命令影响（没有行具有原始 `RowVersion` 值），则 Entity Framework 会引发 `DbUpdateConcurrencyException` 异常。</span><span class="sxs-lookup"><span data-stu-id="0eccc-204">If no rows are affected by the UPDATE command (no rows have the original `RowVersion` value),  the Entity Framework throws a `DbUpdateConcurrencyException` exception.</span></span>

<span data-ttu-id="0eccc-205">该异常的 catch 块中的代码获取受影响的 Department 实体，该实体具有来自异常对象上的 `Entries` 属性的更新值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-205">The code in the catch block for that exception gets the affected Department entity that has the updated values from the `Entries` property on the exception object.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=164)]

<span data-ttu-id="0eccc-206">`Entries` 集合将仅包含一个 `EntityEntry` 对象。</span><span class="sxs-lookup"><span data-stu-id="0eccc-206">The `Entries` collection will have just one `EntityEntry` object.</span></span>  <span data-ttu-id="0eccc-207">该对象可用于获取用户输入的新值和当前的数据库值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-207">You can use that object to get the new values entered by the user and the current database values.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=165-166)]

<span data-ttu-id="0eccc-208">对于所含的数据库值与用户在“编辑”页上输入的值不同的每一列，该代码均为其添加一个自定义错误消息（为简洁起见，此处仅显示一个字段）。</span><span class="sxs-lookup"><span data-stu-id="0eccc-208">The code adds a custom error message for each column that has database values different from what the user entered on the Edit page (only one field is shown here for brevity).</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=174-178)]

<span data-ttu-id="0eccc-209">最后，该代码将 `departmentToUpdate` 的 `RowVersion` 值设置为从数据库中检索到的新值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-209">Finally, the code sets the `RowVersion` value of the `departmentToUpdate` to the new value retrieved from the database.</span></span> <span data-ttu-id="0eccc-210">重新显示“编辑”页时，这个新的 `RowVersion` 值将存储在隐藏字段中，当用户下次单击“保存”时，将只捕获自“编辑”页重新显示起发生的并发错误。</span><span class="sxs-lookup"><span data-stu-id="0eccc-210">This new `RowVersion` value will be stored in the hidden field when the Edit page is redisplayed, and the next time the user clicks **Save**, only concurrency errors that happen since the redisplay of the Edit page will be caught.</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?range=199-200)]

<span data-ttu-id="0eccc-211">`ModelState` 具有旧的 `RowVersion` 值，因此需使用 `ModelState.Remove` 语句。</span><span class="sxs-lookup"><span data-stu-id="0eccc-211">The `ModelState.Remove` statement is required because `ModelState` has the old `RowVersion` value.</span></span> <span data-ttu-id="0eccc-212">在此视图中，当两者都存在时，字段的 `ModelState` 值优于模型属性值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-212">In the view, the `ModelState` value for a field takes precedence over the model property values when both are present.</span></span>

## <a name="update-the-department-edit-view"></a><span data-ttu-id="0eccc-213">更新“院系编辑”视图</span><span class="sxs-lookup"><span data-stu-id="0eccc-213">Update the Department Edit view</span></span>

<span data-ttu-id="0eccc-214">在 Views/Departments/Edit.cshtml 中，进行以下更改：</span><span class="sxs-lookup"><span data-stu-id="0eccc-214">In *Views/Departments/Edit.cshtml*, make the following changes:</span></span>

* <span data-ttu-id="0eccc-215">添加隐藏字段以保存 `RowVersion` 属性值，紧跟在 `DepartmentID` 属性的隐藏字段后面。</span><span class="sxs-lookup"><span data-stu-id="0eccc-215">Add a hidden field to save the `RowVersion` property value, immediately following the hidden field for the `DepartmentID` property.</span></span>

* <span data-ttu-id="0eccc-216">向下拉列表添加“选择管理员”选项。</span><span class="sxs-lookup"><span data-stu-id="0eccc-216">Add a "Select Administrator" option to the drop-down list.</span></span>

[!code-html[](intro/samples/cu/Views/Departments/Edit.cshtml?highlight=16,34-36)]

## <a name="test-concurrency-conflicts-in-the-edit-page"></a><span data-ttu-id="0eccc-217">测试“编辑”页中的并发冲突</span><span class="sxs-lookup"><span data-stu-id="0eccc-217">Test concurrency conflicts in the Edit page</span></span>

<span data-ttu-id="0eccc-218">运行应用并转到“院系索引”页。</span><span class="sxs-lookup"><span data-stu-id="0eccc-218">Run the app and go to the Departments Index page.</span></span> <span data-ttu-id="0eccc-219">右键单击英语系的“编辑”超链接，并选择“在新选项卡中打开”，然后单击英语系的“编辑”超链接。</span><span class="sxs-lookup"><span data-stu-id="0eccc-219">Right-click the **Edit** hyperlink for the English department and select **Open in new tab**, then click the **Edit** hyperlink for the English department.</span></span> <span data-ttu-id="0eccc-220">现在，两个浏览器选项卡显示相同的信息。</span><span class="sxs-lookup"><span data-stu-id="0eccc-220">The two browser tabs now display the same information.</span></span>

<span data-ttu-id="0eccc-221">在第一个浏览器选项卡中更改一个字段，然后单击“保存”。</span><span class="sxs-lookup"><span data-stu-id="0eccc-221">Change a field in the first browser tab and click **Save**.</span></span>

![更改后的“院系编辑”页 1](concurrency/_static/edit-after-change-1.png)

<span data-ttu-id="0eccc-223">浏览器显示具有更改值的索引页。</span><span class="sxs-lookup"><span data-stu-id="0eccc-223">The browser shows the Index page with the changed value.</span></span>

<span data-ttu-id="0eccc-224">在第二个浏览器选项卡中更改一个字段。</span><span class="sxs-lookup"><span data-stu-id="0eccc-224">Change a field in the second browser tab.</span></span>

![更改后的“院系编辑”页 2](concurrency/_static/edit-after-change-2.png)

<span data-ttu-id="0eccc-226">单击“保存” 。</span><span class="sxs-lookup"><span data-stu-id="0eccc-226">Click **Save**.</span></span> <span data-ttu-id="0eccc-227">看见一条错误消息：</span><span class="sxs-lookup"><span data-stu-id="0eccc-227">You see an error message:</span></span>

![“院系编辑”页错误消息](concurrency/_static/edit-error.png)

<span data-ttu-id="0eccc-229">再次单击“保存”。</span><span class="sxs-lookup"><span data-stu-id="0eccc-229">Click **Save** again.</span></span> <span data-ttu-id="0eccc-230">保存在第二个浏览器选项卡中输入的值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-230">The value you entered in the second browser tab is saved.</span></span> <span data-ttu-id="0eccc-231">在索引页中出现时，可以看到已保存的值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-231">You see the saved values when the Index page appears.</span></span>

## <a name="update-the-delete-page"></a><span data-ttu-id="0eccc-232">更新“删除”页</span><span class="sxs-lookup"><span data-stu-id="0eccc-232">Update the Delete page</span></span>

<span data-ttu-id="0eccc-233">对于“删除”页，Entity Framework 以类似方式检测其他人编辑院系所引起的并发冲突。</span><span class="sxs-lookup"><span data-stu-id="0eccc-233">For the Delete page, the Entity Framework detects concurrency conflicts caused by someone else editing the department in a similar manner.</span></span> <span data-ttu-id="0eccc-234">当 HttpGet `Delete` 方法显示确认视图时，该视图包含隐藏字段中的原始 `RowVersion` 值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-234">When the HttpGet `Delete` method displays the confirmation view, the view includes the original `RowVersion` value in a hidden field.</span></span> <span data-ttu-id="0eccc-235">然后，该值可用于在用户确认删除时进行调用的 HttpPost `Delete` 方法。</span><span class="sxs-lookup"><span data-stu-id="0eccc-235">That value is then available to the HttpPost `Delete` method that's called when the user confirms the deletion.</span></span> <span data-ttu-id="0eccc-236">当 Entity Framework 创建 SQL DELETE 命令时，它包含具有原始 `RowVersion` 值的 WHERE 子句。</span><span class="sxs-lookup"><span data-stu-id="0eccc-236">When the Entity Framework creates the SQL DELETE command, it includes a WHERE clause with the original `RowVersion` value.</span></span> <span data-ttu-id="0eccc-237">如果该命令不影响任何行（即在显示“删除”确认页面后更改了行），则引发并发异常，调用 HttpGet `Delete` 方法，并将错误标志设置为 true，以重新显示具有一条错误消息的确认页面。</span><span class="sxs-lookup"><span data-stu-id="0eccc-237">If the command results in zero rows affected (meaning the row was changed after the Delete confirmation page was displayed), a concurrency exception is thrown, and the HttpGet `Delete` method is called with an error flag set to true in order to redisplay the confirmation page with an error message.</span></span> <span data-ttu-id="0eccc-238">任意行均未受影响的原因也可能是，该行已被其他用户删除，因此在此情况下不显示错误消息。</span><span class="sxs-lookup"><span data-stu-id="0eccc-238">It's also possible that zero rows were affected because the row was deleted by another user, so in that case no error message is displayed.</span></span>

### <a name="update-the-delete-methods-in-the-departments-controller"></a><span data-ttu-id="0eccc-239">更新“院系”控制器中的删除方法</span><span class="sxs-lookup"><span data-stu-id="0eccc-239">Update the Delete methods in the Departments controller</span></span>

<span data-ttu-id="0eccc-240">在 DepartmentsController.cs 中，将 HttpGet `Delete` 方法替换为以下代码：</span><span class="sxs-lookup"><span data-stu-id="0eccc-240">In *DepartmentsController.cs*, replace the HttpGet `Delete` method with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeleteGet&highlight=1,10,14-17,21-29)]

<span data-ttu-id="0eccc-241">该方法接受可选参数，该参数指示是否在并发错误之后重新显示页面。</span><span class="sxs-lookup"><span data-stu-id="0eccc-241">The method accepts an optional parameter that indicates whether the page is being redisplayed after a concurrency error.</span></span> <span data-ttu-id="0eccc-242">如果此标志为 true 且指定的院系不复存在，则它已被其他用户删除。</span><span class="sxs-lookup"><span data-stu-id="0eccc-242">If this flag is true and the department specified no longer exists, it was deleted by another user.</span></span> <span data-ttu-id="0eccc-243">此情况下，代码将重定向到索引页。</span><span class="sxs-lookup"><span data-stu-id="0eccc-243">In that case, the code redirects to the Index page.</span></span>  <span data-ttu-id="0eccc-244">如果此标志为 true 且该院系确实存在，则它已被其他用户更改。</span><span class="sxs-lookup"><span data-stu-id="0eccc-244">If this flag is true and the Department does exist, it was changed by another user.</span></span> <span data-ttu-id="0eccc-245">此情况下，代码将使用 `ViewData` 向视图发送一条错误消息。</span><span class="sxs-lookup"><span data-stu-id="0eccc-245">In that case, the code sends an error message to the view using `ViewData`.</span></span>

<span data-ttu-id="0eccc-246">将 HttpPost `Delete` 方法（名为 `DeleteConfirmed`）中的代码替换为以下代码：</span><span class="sxs-lookup"><span data-stu-id="0eccc-246">Replace the code in the HttpPost `Delete` method (named `DeleteConfirmed`) with the following code:</span></span>

[!code-csharp[](intro/samples/cu/Controllers/DepartmentsController.cs?name=snippet_DeletePost&highlight=1,3,5-8,11-18)]

<span data-ttu-id="0eccc-247">在刚替换的基架代码中，此方法仅接受记录 ID：</span><span class="sxs-lookup"><span data-stu-id="0eccc-247">In the scaffolded code that you just replaced, this method accepted only a record ID:</span></span>

```csharp
public async Task<IActionResult> DeleteConfirmed(int id)
```

<span data-ttu-id="0eccc-248">已将此参数更改为由模型绑定器创建的 Department 实体实例。</span><span class="sxs-lookup"><span data-stu-id="0eccc-248">You've changed this parameter to a Department entity instance created by the model binder.</span></span> <span data-ttu-id="0eccc-249">这样，EF 就可访问除记录键之外的 RowVersion 属性值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-249">This gives EF access to the RowVersion property value in addition to the record key.</span></span>

```csharp
public async Task<IActionResult> Delete(Department department)
```

<span data-ttu-id="0eccc-250">你还将操作方法名称从 `DeleteConfirmed` 更改为了 `Delete`。</span><span class="sxs-lookup"><span data-stu-id="0eccc-250">You have also changed the action method name from `DeleteConfirmed` to `Delete`.</span></span> <span data-ttu-id="0eccc-251">基架代码使用 `DeleteConfirmed` 名称来为 HttpPost 方法提供唯一签名。</span><span class="sxs-lookup"><span data-stu-id="0eccc-251">The scaffolded code used the name `DeleteConfirmed` to give the HttpPost method a unique signature.</span></span> <span data-ttu-id="0eccc-252">（CLR 要求重载方法具有不同的方法参数。）既然签名是唯一的，就可继续使用 MVC 约定，并为 HttpPost 和 HttpGet 删除方法使用同一名称。</span><span class="sxs-lookup"><span data-stu-id="0eccc-252">(The CLR requires overloaded methods to have different method parameters.) Now that the signatures are unique, you can stick with the MVC convention and use the same name for the HttpPost and HttpGet delete methods.</span></span>

<span data-ttu-id="0eccc-253">如果已删除院系，则 `AnyAsync` 方法返回 false，应用程序仅返回到 Index 方法。</span><span class="sxs-lookup"><span data-stu-id="0eccc-253">If the department is already deleted, the `AnyAsync` method returns false and the application just goes back to the Index method.</span></span>

<span data-ttu-id="0eccc-254">如果捕获到并发错误，代码将重新显示“删除”确认页，并提供一个指示它应显示并发错误消息的标志。</span><span class="sxs-lookup"><span data-stu-id="0eccc-254">If a concurrency error is caught, the code redisplays the Delete confirmation page and provides a flag that indicates it should display a concurrency error message.</span></span>

### <a name="update-the-delete-view"></a><span data-ttu-id="0eccc-255">更新“删除”视图</span><span class="sxs-lookup"><span data-stu-id="0eccc-255">Update the Delete view</span></span>

<span data-ttu-id="0eccc-256">在 Views/Departments/Delete.cshtml 中，将基架代码替换为添加 DepartmentID 和 RowVersion 属性的错误消息字段和隐藏字段的以下代码。</span><span class="sxs-lookup"><span data-stu-id="0eccc-256">In *Views/Departments/Delete.cshtml*, replace the scaffolded code with the following code that adds an error message field and hidden fields for the DepartmentID and RowVersion properties.</span></span> <span data-ttu-id="0eccc-257">突出显示所作更改。</span><span class="sxs-lookup"><span data-stu-id="0eccc-257">The changes are highlighted.</span></span>

[!code-html[](intro/samples/cu/Views/Departments/Delete.cshtml?highlight=9,38,44,45,48)]

<span data-ttu-id="0eccc-258">这将进行以下更改：</span><span class="sxs-lookup"><span data-stu-id="0eccc-258">This makes the following changes:</span></span>

* <span data-ttu-id="0eccc-259">在 `h2` 和 `h3` 标题之间添加错误消息。</span><span class="sxs-lookup"><span data-stu-id="0eccc-259">Adds an error message between the `h2` and `h3` headings.</span></span>

* <span data-ttu-id="0eccc-260">将“管理员”字段中的 FirstMidName 替换为 FullName。</span><span class="sxs-lookup"><span data-stu-id="0eccc-260">Replaces FirstMidName with FullName in the **Administrator** field.</span></span>

* <span data-ttu-id="0eccc-261">删除 RowVersion 字段。</span><span class="sxs-lookup"><span data-stu-id="0eccc-261">Removes the RowVersion field.</span></span>

* <span data-ttu-id="0eccc-262">添加 `RowVersion` 属性的隐藏字段。</span><span class="sxs-lookup"><span data-stu-id="0eccc-262">Adds a hidden field for the `RowVersion` property.</span></span>

<span data-ttu-id="0eccc-263">运行应用并转到“院系索引”页。</span><span class="sxs-lookup"><span data-stu-id="0eccc-263">Run the app and go to the Departments Index page.</span></span> <span data-ttu-id="0eccc-264">右键单击英语系的“删除”超链接，并选择“在新选项卡中打开”，然后在第一个选项卡中单击英语系的“编辑”超链接。</span><span class="sxs-lookup"><span data-stu-id="0eccc-264">Right-click the **Delete** hyperlink for the English department and select **Open in new tab**, then in the first tab click the **Edit** hyperlink for the English department.</span></span>

<span data-ttu-id="0eccc-265">在第一个窗口中，更改其中一个值，然后单击“保存”：</span><span class="sxs-lookup"><span data-stu-id="0eccc-265">In the first window, change one of the values, and click **Save**:</span></span>

![删除之前显示的更改后的“院系删除”页面](concurrency/_static/edit-after-change-for-delete.png)

<span data-ttu-id="0eccc-267">在第二个选项卡中，单击“删除”。</span><span class="sxs-lookup"><span data-stu-id="0eccc-267">In the second tab, click **Delete**.</span></span> <span data-ttu-id="0eccc-268">你将看到并发错误消息，且已使用数据库中的当前内容刷新了“院系”值。</span><span class="sxs-lookup"><span data-stu-id="0eccc-268">You see the concurrency error message, and the Department values are refreshed with what's currently in the database.</span></span>

![显示有并发错误的“院系删除”确认页](concurrency/_static/delete-error.png)

<span data-ttu-id="0eccc-270">如果再次单击“删除”，会重定向到已删除显示院系的索引页。</span><span class="sxs-lookup"><span data-stu-id="0eccc-270">If you click **Delete** again, you're redirected to the Index page, which shows that the department has been deleted.</span></span>

## <a name="update-details-and-create-views"></a><span data-ttu-id="0eccc-271">更新“详细信息”和“创建”视图</span><span class="sxs-lookup"><span data-stu-id="0eccc-271">Update Details and Create views</span></span>

<span data-ttu-id="0eccc-272">可选择性地清理“详细信息”和“创建”视图中的基架代码。</span><span class="sxs-lookup"><span data-stu-id="0eccc-272">You can optionally clean up scaffolded code in the Details and Create views.</span></span>

<span data-ttu-id="0eccc-273">替换 Views/Departments/Details.cshtml 中的代码，以删除 RowVersion 列并显示管理员的全名。</span><span class="sxs-lookup"><span data-stu-id="0eccc-273">Replace the code in *Views/Departments/Details.cshtml* to delete the RowVersion column and show the full name of the Administrator.</span></span>

[!code-html[](intro/samples/cu/Views/Departments/Details.cshtml?highlight=35)]

<span data-ttu-id="0eccc-274">替换 Views/Departments/Create.cshtml 中的代码，向下拉列表添加“选择”选项。</span><span class="sxs-lookup"><span data-stu-id="0eccc-274">Replace the code in *Views/Departments/Create.cshtml* to add a Select option to the drop-down list.</span></span>

[!code-html[](intro/samples/cu/Views/Departments/Create.cshtml?highlight=32-34)]

## <a name="summary"></a><span data-ttu-id="0eccc-275">总结</span><span class="sxs-lookup"><span data-stu-id="0eccc-275">Summary</span></span>

<span data-ttu-id="0eccc-276">处理并发冲突已介绍完毕。</span><span class="sxs-lookup"><span data-stu-id="0eccc-276">This completes the introduction to handling concurrency conflicts.</span></span> <span data-ttu-id="0eccc-277">要深入了解如何处理 EF Core 中的并发，请参阅[并发冲突](/ef/core/saving/concurrency)。</span><span class="sxs-lookup"><span data-stu-id="0eccc-277">For more information about how to handle concurrency in EF Core, see [Concurrency conflicts](/ef/core/saving/concurrency).</span></span> <span data-ttu-id="0eccc-278">下一个教程将介绍如何为 Instructor 和 Students 实体实现“每个层次结构一个表”继承。</span><span class="sxs-lookup"><span data-stu-id="0eccc-278">The next tutorial shows how to implement table-per-hierarchy inheritance for the Instructor and Student entities.</span></span>

::: moniker-end

> [!div class="step-by-step"]
> <span data-ttu-id="0eccc-279">[上一页](update-related-data.md)
> [下一页](inheritance.md)</span><span class="sxs-lookup"><span data-stu-id="0eccc-279">[Previous](update-related-data.md)
[Next](inheritance.md)</span></span>
