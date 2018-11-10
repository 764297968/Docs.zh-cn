<span data-ttu-id="0846b-101">下表详细说明了 ASP.NET Core 代码生成器参数：</span><span class="sxs-lookup"><span data-stu-id="0846b-101">The following table details the ASP.NET Core code generator parameters:</span></span>

| <span data-ttu-id="0846b-102">参数</span><span class="sxs-lookup"><span data-stu-id="0846b-102">Parameter</span></span>               | <span data-ttu-id="0846b-103">描述</span><span class="sxs-lookup"><span data-stu-id="0846b-103">Description</span></span>|
| ----------------- | ------------ |
| <span data-ttu-id="0846b-104">-m</span><span class="sxs-lookup"><span data-stu-id="0846b-104">-m</span></span>  | <span data-ttu-id="0846b-105">模型的名称。</span><span class="sxs-lookup"><span data-stu-id="0846b-105">The name of the model.</span></span> |
| <span data-ttu-id="0846b-106">-dc</span><span class="sxs-lookup"><span data-stu-id="0846b-106">-dc</span></span>  | <span data-ttu-id="0846b-107">数据上下文。</span><span class="sxs-lookup"><span data-stu-id="0846b-107">The data context.</span></span> |
| <span data-ttu-id="0846b-108">-udl</span><span class="sxs-lookup"><span data-stu-id="0846b-108">-udl</span></span> | <span data-ttu-id="0846b-109">使用默认布局。</span><span class="sxs-lookup"><span data-stu-id="0846b-109">Use the default layout.</span></span> |
| <span data-ttu-id="0846b-110">-outDir</span><span class="sxs-lookup"><span data-stu-id="0846b-110">-outDir</span></span> | <span data-ttu-id="0846b-111">用于创建视图的相对输出文件夹路径。</span><span class="sxs-lookup"><span data-stu-id="0846b-111">The relative output folder path to create the views.</span></span> |
| <span data-ttu-id="0846b-112">--referenceScriptLibraries</span><span class="sxs-lookup"><span data-stu-id="0846b-112">--referenceScriptLibraries</span></span> | <span data-ttu-id="0846b-113">向“编辑”和“创建”页面添加 `_ValidationScriptsPartial`</span><span class="sxs-lookup"><span data-stu-id="0846b-113">Adds `_ValidationScriptsPartial` to Edit and Create pages</span></span> |

<span data-ttu-id="0846b-114">使用 `h` 开关获取 `aspnet-codegenerator razorpage` 命令方面的帮助：</span><span class="sxs-lookup"><span data-stu-id="0846b-114">Use the `h` switch to get help on the `aspnet-codegenerator razorpage` command:</span></span>

```console
dotnet aspnet-codegenerator razorpage -h
```

<a name="test"></a>

### <a name="test-the-app"></a><span data-ttu-id="0846b-115">测试应用</span><span class="sxs-lookup"><span data-stu-id="0846b-115">Test the app</span></span>

* <span data-ttu-id="0846b-116">运行应用并将 `/Movies` 追加到浏览器中的 URL (`http://localhost:port/Movies`)。</span><span class="sxs-lookup"><span data-stu-id="0846b-116">Run the app and append `/Movies` to the URL in the browser (`http://localhost:port/Movies`).</span></span>
* <span data-ttu-id="0846b-117">测试“创建”链接。</span><span class="sxs-lookup"><span data-stu-id="0846b-117">Test the **Create** link.</span></span>

  ![创建页面](../../tutorials/razor-pages/model/_static/conan.png)

<a name="scaffold"></a>

* <span data-ttu-id="0846b-119">测试“编辑”、“详细信息”和“删除”链接。</span><span class="sxs-lookup"><span data-stu-id="0846b-119">Test the **Edit**, **Details**, and **Delete** links.</span></span>

<span data-ttu-id="0846b-120">如果收到类似如下的错误，则验证是否已运行迁移并更新数据库：</span><span class="sxs-lookup"><span data-stu-id="0846b-120">If you get the error similar to the following, verify you have run migrations and updated the database:</span></span>

`An unhandled exception occurred while processing the request. 'no such table: Movie'.`
