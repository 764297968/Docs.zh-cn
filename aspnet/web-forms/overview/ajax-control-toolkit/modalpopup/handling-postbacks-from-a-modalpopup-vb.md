---
uid: web-forms/overview/ajax-control-toolkit/modalpopup/handling-postbacks-from-a-modalpopup-vb
title: "处理从 ModalPopup (VB) 的回发 |Microsoft 文档"
author: wenz
description: "AJAX 控件工具包中的 ModalPopup 控制提供一种简单的方法，以创建模式的弹出项，使用客户端的方式。 必须格外小心时 pos..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: f70ac2b3-900f-40fa-858f-ab057904506b
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/modalpopup/handling-postbacks-from-a-modalpopup-vb
msc.type: authoredcontent
ms.openlocfilehash: 7915d555fef363f41aa51bd77f0183c97c2f3769
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2017
---
<a name="handling-postbacks-from-a-modalpopup-vb"></a><span data-ttu-id="8d6a9-104">处理回发从 ModalPopup (VB)</span><span class="sxs-lookup"><span data-stu-id="8d6a9-104">Handling Postbacks from a ModalPopup (VB)</span></span>
====================
<span data-ttu-id="8d6a9-105">通过[Christian Wenz](https://github.com/wenz)</span><span class="sxs-lookup"><span data-stu-id="8d6a9-105">by [Christian Wenz](https://github.com/wenz)</span></span>

<span data-ttu-id="8d6a9-106">[下载代码](http://download.microsoft.com/download/2/4/0/24052038-f942-4336-905b-b60ae56f0dd5/ModalPopup3.vb.zip)或[下载 PDF](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/modalpopup3VB.pdf)</span><span class="sxs-lookup"><span data-stu-id="8d6a9-106">[Download Code](http://download.microsoft.com/download/2/4/0/24052038-f942-4336-905b-b60ae56f0dd5/ModalPopup3.vb.zip) or [Download PDF](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/modalpopup3VB.pdf)</span></span>

> <span data-ttu-id="8d6a9-107">AJAX 控件工具包中的 ModalPopup 控制提供一种简单的方法，以创建模式的弹出项，使用客户端的方式。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-107">The ModalPopup control in the AJAX Control Toolkit offers a simple way to create a modal popup using client-side means.</span></span> <span data-ttu-id="8d6a9-108">从弹出窗口中创建回发时，必须采用格外小心。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-108">Special care must be taken when a postback is created from within the popup.</span></span>


## <a name="overview"></a><span data-ttu-id="8d6a9-109">概述</span><span class="sxs-lookup"><span data-stu-id="8d6a9-109">Overview</span></span>

<span data-ttu-id="8d6a9-110">AJAX 控件工具包中的 ModalPopup 控制提供一种简单的方法，以创建模式的弹出项，使用客户端的方式。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-110">The ModalPopup control in the AJAX Control Toolkit offers a simple way to create a modal popup using client-side means.</span></span> <span data-ttu-id="8d6a9-111">从弹出窗口中创建回发时，必须采用格外小心。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-111">Special care must be taken when a postback is created from within the popup.</span></span>

## <a name="steps"></a><span data-ttu-id="8d6a9-112">步骤</span><span class="sxs-lookup"><span data-stu-id="8d6a9-112">Steps</span></span>

<span data-ttu-id="8d6a9-113">为了激活 ASP.NET AJAX 和控件工具包中的功能`ScriptManager`必须在页面上任意位置放置控件 (但内`<form>`元素):</span><span class="sxs-lookup"><span data-stu-id="8d6a9-113">In order to activate the functionality of ASP.NET AJAX and the Control Toolkit, the `ScriptManager` control must be put anywhere on the page (but within the `<form>` element):</span></span>

[!code-aspx[Main](handling-postbacks-from-a-modalpopup-vb/samples/sample1.aspx)]

<span data-ttu-id="8d6a9-114">接下来，添加一个面板，它可作为模式弹出窗口。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-114">Next, add a panel which serves as the modal popup.</span></span> <span data-ttu-id="8d6a9-115">存在，用户可以输入一个名称和电子邮件地址。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-115">There, the user can enter a name and an email address.</span></span> <span data-ttu-id="8d6a9-116">一个按钮用于关闭弹出窗口，并保存信息。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-116">A button is used to close the popup and save the information.</span></span> <span data-ttu-id="8d6a9-117">请注意，`OnClick`特性设置，以便当单击此按钮的回发时发生：</span><span class="sxs-lookup"><span data-stu-id="8d6a9-117">Note that the `OnClick` attribute is set so that a postback occurs when this button is clicked:</span></span>

[!code-aspx[Main](handling-postbacks-from-a-modalpopup-vb/samples/sample2.aspx)]

<span data-ttu-id="8d6a9-118">页本身包含的完全相同的信息的两个标签： 名称和电子邮件地址。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-118">The page itself consists of two labels for exactly the same information: name and email address.</span></span> <span data-ttu-id="8d6a9-119">一个按钮用于触发模式弹出窗口：</span><span class="sxs-lookup"><span data-stu-id="8d6a9-119">A button is used to trigger the modal popup:</span></span>

[!code-aspx[Main](handling-postbacks-from-a-modalpopup-vb/samples/sample3.aspx)]

<span data-ttu-id="8d6a9-120">若要显示的弹出窗口，添加`ModalPopupExtender`控件。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-120">In order to make the popup appear, add the `ModalPopupExtender` control.</span></span> <span data-ttu-id="8d6a9-121">设置`PopupControlID`属性面板的 ID 和`TargetControlID`到按钮的 ID:</span><span class="sxs-lookup"><span data-stu-id="8d6a9-121">Set the `PopupControlID` attribute to the panel's ID and `TargetControlID` to the button's ID:</span></span>

[!code-aspx[Main](handling-postbacks-from-a-modalpopup-vb/samples/sample4.aspx)]

<span data-ttu-id="8d6a9-122">现在只要`Save`单击模式弹出窗口中的按钮时，服务器端`SaveData()`执行方法。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-122">Now whenever the `Save` button within the modal popup is clicked, the server-side `SaveData()` method is executed.</span></span> <span data-ttu-id="8d6a9-123">存在，无法将输入的数据保存在数据存储中。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-123">There, you could save the entered data in a data store.</span></span> <span data-ttu-id="8d6a9-124">为简单起见，新的数据只需标签中的输出：</span><span class="sxs-lookup"><span data-stu-id="8d6a9-124">For the sake of simplicity, the new data is just output in the label:</span></span>

[!code-vb[Main](handling-postbacks-from-a-modalpopup-vb/samples/sample5.vb)]

<span data-ttu-id="8d6a9-125">此外，应使用当前名称和电子邮件填充模式的弹出窗口中将文本框控件。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-125">Also, the textbox controls within the modal popup should be filled with the current name and email.</span></span> <span data-ttu-id="8d6a9-126">但是这是仅有必要无回发发生时。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-126">However this is only necessary when no postback occurs.</span></span> <span data-ttu-id="8d6a9-127">如果回发，则 ASP.NET viewstate 功能将自动填充使用适当的值的文本框。</span><span class="sxs-lookup"><span data-stu-id="8d6a9-127">If there is a postback, the ASP.NET viewstate feature will automatically fill the textboxes with the appropriate values.</span></span>

[!code-vb[Main](handling-postbacks-from-a-modalpopup-vb/samples/sample6.vb)]


<span data-ttu-id="8d6a9-128">[![模式弹出导致回发](handling-postbacks-from-a-modalpopup-vb/_static/image2.png)](handling-postbacks-from-a-modalpopup-vb/_static/image1.png)</span><span class="sxs-lookup"><span data-stu-id="8d6a9-128">[![The modal popup causes a postback](handling-postbacks-from-a-modalpopup-vb/_static/image2.png)](handling-postbacks-from-a-modalpopup-vb/_static/image1.png)</span></span>

<span data-ttu-id="8d6a9-129">模式弹出导致回发 ([单击以查看实际尺寸的图像](handling-postbacks-from-a-modalpopup-vb/_static/image3.png))</span><span class="sxs-lookup"><span data-stu-id="8d6a9-129">The modal popup causes a postback ([Click to view full-size image](handling-postbacks-from-a-modalpopup-vb/_static/image3.png))</span></span>

>[!div class="step-by-step"]
<span data-ttu-id="8d6a9-130">[上一页](using-modalpopup-with-a-repeater-control-vb.md)
[下一页](positioning-a-modalpopup-vb.md)</span><span class="sxs-lookup"><span data-stu-id="8d6a9-130">[Previous](using-modalpopup-with-a-repeater-control-vb.md)
[Next](positioning-a-modalpopup-vb.md)</span></span>