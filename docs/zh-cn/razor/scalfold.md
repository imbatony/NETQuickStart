# ASP.NET Core 中已搭建基架的 Razor 页面

## “创建”、“删除”、“详细信息”和“编辑”页面

检查 Pages/Movies/Index.cshtml.cs 页面模型：

```csharp
// Unused usings removed.
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;
using RazorPagesMovie.Models;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace RazorPagesMovie.Pages.Movies
{
    #region snippet1
    public class IndexModel : PageModel
    {
        private readonly RazorPagesMovie.Data.RazorPagesMovieContext _context;

        public IndexModel(RazorPagesMovie.Data.RazorPagesMovieContext context)
        {
            _context = context;
        }
        #endregion
        public IList<Movie> Movie { get;set; }

        public async Task OnGetAsync()
        {
            Movie = await _context.Movie.ToListAsync();
        }
    }
}
```

Razor 页面派生自 PageModel。 按照约定，PageModel 派生的类称为 <PageName>Model。 此构造函数使用依赖关系注入将 RazorPagesMovieContext 添加到页面：


```csharp
public class IndexModel : PageModel
{
    private readonly RazorPagesMovie.Data.RazorPagesMovieContext _context;

    public IndexModel(RazorPagesMovie.Data.RazorPagesMovieContext context)
    {
        _context = context;
    }
```

当返回类型是 IActionResult 或 Task<IActionResult> 时，必须提供返回语句。 例如，Pages/Movies/Create.cshtml.cs OnPostAsync 方法：


```csharp
 public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }

        _context.Movie.Add(Movie);
        await _context.SaveChangesAsync();

        return RedirectToPage("./Index");
    }
}
```

检查 Pages/Movies/Index.cshtml Razor 页面：

```html
@page
@model RazorPagesMovie.Pages.Movies.IndexModel

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-page="Create">Create New</a>
</p>
<table class="table">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.Movie[0].Title)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Movie[0].ReleaseDate)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Movie[0].Genre)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Movie[0].Price)
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
@foreach (var item in Model.Movie) {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.Title)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.ReleaseDate)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Genre)
            </td>
            <td>
                @Html.DisplayFor(modelItem => item.Price)
            </td>
            <td>
                <a asp-page="./Edit" asp-route-id="@item.ID">Edit</a> |
                <a asp-page="./Details" asp-route-id="@item.ID">Details</a> |
                <a asp-page="./Delete" asp-route-id="@item.ID">Delete</a>
            </td>
        </tr>
}
    </tbody>
</table>
```

Razor 可以从 HTML 转换为 C# 或 Razor 特定标记。 当 `@` 符号后跟 [Razor 保留关键字](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/razor?view=aspnetcore-5.0#razor-reserved-keywords)时，它会转换为 Razor 特定标记，否则会转换为 C#。

### @page 指令

`@page` Razor 指令将文件转换为一个 MVC 操作，这意味着它可以处理请求。 `@page` 必须是页面上的第一个 Razor 指令。 `@page` 和 `@model` 是转换为 Razor 特定标记的示例。 有关详细信息，请参阅 [Razor 语法](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/razor?view=aspnetcore-5.0#razor-syntax)。



### @model 指令

```html
@page
@model RazorPagesMovie.Pages.Movies.IndexModel
```

`@model` 指令指定传递到 Razor 页面的模型类型。 在前面的示例中，`@model` 行使 `PageModel` 派生的类可用于 Razor 页面。 在页面上的 `@Html.DisplayNameFor` 和 `@Html.DisplayFor` [HTML 帮助程序](https://docs.microsoft.com/zh-cn/aspnet/mvc/overview/older-versions-1/views/creating-custom-html-helpers-cs#understanding-html-helpers)中使用该模型。



检查以下 HTML 帮助程序中使用的 Lambda 表达式：

```html
@Html.DisplayNameFor(model => model.Movie[0].Title)
```

[DisplayNameExtensions.DisplayNameFor](https://docs.microsoft.com/zh-cn/dotnet/api/system.web.mvc.html.displaynameextensions.displaynamefor) HTML 帮助程序检查 Lambda 表达式中引用的 `Title` 属性来确定显示名称。 检查 Lambda 表达式（而非求值）。 这意味着当 `model`、`model.Movie` 或 `model.Movie[0]` 为 `null` 或为空时，不会存在任何访问冲突。 对 Lambda 表达式求值时（例如，使用 `@Html.DisplayFor(modelItem => item.Title)`），将求得该模型的属性值。

###  布局页

选择菜单链接（“Razor PagesMovie”、“主页”和“隐私”） 。 每页显示相同的菜单布局。 菜单布局是在 Pages/Shared/_Layout.cshtml 文件中实现。

打开并检查 Pages/Shared/_Layout.cshtml 文件。

[布局](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/layout?view=aspnetcore-5.0)模板允许 HTML 容器具有如下布局：

- 在一个位置指定。
- 应用于站点中的多个页面。

查找 `@RenderBody()` 行。 `RenderBody` 是显示全部页面专用视图的占位符，已包装在布局页中。 例如，选择“隐私”链接后，Pages/Privacy.cshtml 视图在 `RenderBody` 方法中呈现。

### ViewData 和布局

考虑来自 Pages/Movies/Index.cshtml 文件的以下标记：

```html
@model RazorPagesMovie.Pages.Movies.IndexModel

@{
    ViewData["Title"] = "Index";
}
```

前面突出显示的标记是 Razor 转换为 C# 的一个示例。 `{` 和 `}` 字符括住 C# 代码块。

`PageModel` 基类包含 `ViewData` 字典属性，可用于将数据传递到某个视图。 可以使用键值*_模式将对象添加到 `ViewData` 字典。 在前面的示例中，`Title` 属性被添加到 `ViewData` 字典。

`Title` 属性用于 _Pages/Shared/_Layout.cshtml* 文件。 以下标记显示 _Layout.cshtml 文件的前几行。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - RazorPagesMovie</title>

    @*Markup removed for brevity.*@
```

行 `@*Markup removed for brevity.*@` 是 Razor 注释。 与 HTML 注释 `<!-- -->` 不同，Razor 注释不会发送到客户端。 有关详细信息，请参阅 [MDN Web 文档：HTML 入门](https://developer.mozilla.org/docs/Learn/HTML/Introduction_to_HTML/Getting_started#HTML_comments)。

### 更新布局

1. 更改 Pages/Shared/_Layout.cshtml 文件中的 `<title>` 元素以显示 Movie 而不是 RazorPagesMovie 。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Movie</title>
```

2. 在 Pages/Shared/_Layout.cshtml 文件中，查找以下定位点元素。

```html
<a class="navbar-brand" asp-area="" asp-page="/Index">RazorPagesMovie</a>
```

3. 将前面的元素替换为以下标记：

```html
<a class="navbar-brand" asp-page="/Movies/Index">RpMovie</a>
```

前面的定位点元素是一个[标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-5.0)。 此处它是[定位点标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/tag-helpers/built-in/anchor-tag-helper?view=aspnetcore-5.0)。 `asp-page="/Movies/Index"` 标记帮助程序属性和值可以创建指向 `/Movies/Index` Razor 页面的链接。 `asp-area` 属性值为空，因此在链接中未使用区域。 有关详细信息，请参阅[区域](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/areas?view=aspnetcore-5.0)。

4. 保存所做的更改，并通过选择“RpMovie”链接测试应用。 如果遇到任何问题，请参阅 GitHub 中的 [_Layout.cshtml](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Shared/_Layout.cshtml) 文件。

5. 测试“主页”、“RpMovie”、“创建”、“编辑”和“删除”链接 。 每个页面都设置有标题，可以在浏览器选项卡中看到标题。将某个页面加入书签时，标题用于该书签。

在 Pages/_ViewStart.cshtml 文件中设置 `Layout` 属性：

```csharp
@{
    Layout = "_Layout";
}
```

前面的标记针对 Pages 文件夹下的所有 Razor 文件将布局文件设置为 Pages/Shared/_Layout.cshtml。 请参阅[布局](https://docs.microsoft.com/zh-cn/aspnet/core/razor-pages/?view=aspnetcore-5.0#layout)了解详细信息。

### “创建”页面模型

检查 *Pages/Movies/Create.cshtml.cs* 页面模型：



```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using RazorPagesMovie.Models;
using System;
using System.Threading.Tasks;

namespace RazorPagesMovie.Pages.Movies
{
    public class CreateModel : PageModel
    {
        private readonly RazorPagesMovie.Data.RazorPagesMovieContext _context;

        public CreateModel(RazorPagesMovie.Data.RazorPagesMovieContext context)
        {
            _context = context;
        }

        public IActionResult OnGet()
        {
            return Page();
        }

        [BindProperty]
        public Movie Movie { get; set; }

        public async Task<IActionResult> OnPostAsync()
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            _context.Movie.Add(Movie);
            await _context.SaveChangesAsync();

            return RedirectToPage("./Index");
        }
    }
}
```

`OnGet` 方法初始化页面所需的任何状态。 “创建”页没有任何要初始化的状态，因此返回 `Page`。 在本教程的后面部分中，将介绍 `OnGet` 初始化状态的示例。 `Page` 方法创建用于呈现 Create.cshtml 页的 `PageResult` 对象。

`Movie` 属性使用 [[BindProperty\]](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.bindpropertyattribute) 特性来选择加入[模型绑定](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/models/model-binding?view=aspnetcore-5.0)。 当“创建”表单发布表单值时，ASP.NET Core 运行时将发布的值绑定到 `Movie` 模型。

当页面发布表单数据时，运行 `OnPostAsync` 方法：

```csharp
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }

    _context.Movie.Add(Movie);
    await _context.SaveChangesAsync();

    return RedirectToPage("./Index");
}
```

如果不存在任何模型错误，将重新显示表单，以及发布的任何表单数据。 在发布表单前，可以在客户端捕获到大部分模型错误。 模型错误的一个示例是，发布的日期字段值无法转换为日期。 本教程后面讨论了客户端验证和模型验证。

如果没有模型错误：

- 将保存数据。
- 浏览器将重定向到 Index 页。

### “创建 Razor”页面

检查 Pages/Movies/Create.cshtml Razor 页面文件：

```html
@model RazorPagesMovie.Pages.Movies.CreateModel

@{
    ViewData["Title"] = "Create";
}

<h1>Create</h1>

<h4>Movie</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form method="post">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Movie.Title" class="control-label"></label>
                <input asp-for="Movie.Title" class="form-control" />
                <span asp-validation-for="Movie.Title" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Movie.ReleaseDate" class="control-label"></label>
                <input asp-for="Movie.ReleaseDate" class="form-control" />
                <span asp-validation-for="Movie.ReleaseDate" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Movie.Genre" class="control-label"></label>
                <input asp-for="Movie.Genre" class="form-control" />
                <span asp-validation-for="Movie.Genre" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Movie.Price" class="control-label"></label>
                <input asp-for="Movie.Price" class="form-control" />
                <span asp-validation-for="Movie.Price" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Create" class="btn btn-primary" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-page="Index">Back to List</a>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
```

Visual Studio 以用于标记帮助程序的特殊加粗字体显示以下标记：

- `<form method="post">`
- `<div asp-validation-summary="ModelOnly" class="text-danger"></div>`
- `<label asp-for="Movie.Title" class="control-label"></label>`
- `<input asp-for="Movie.Title" class="form-control" />`
- `<span asp-validation-for="Movie.Title" class="text-danger"></span>`

![https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/page/_static/th3.png?view=aspnetcore-5.0](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/page/_static/th3.png?view=aspnetcore-5.0)

基架引擎在模型中为每个字段（ID 除外）创建 Razor 标记，如下所示：

```html
<div asp-validation-summary="ModelOnly" class="text-danger"></div>
<div class="form-group">
    <label asp-for="Movie.Title" class="control-label"></label>
    <input asp-for="Movie.Title" class="form-control" />
    <span asp-validation-for="Movie.Title" class="text-danger"></span>
</div>
```

[验证标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/working-with-forms?view=aspnetcore-5.0#the-validation-tag-helpers)（`<div asp-validation-summary` 和 `<span asp-validation-for`）显示验证错误。 本系列后面的部分将更详细地讨论有关验证的信息。

[标签标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/working-with-forms?view=aspnetcore-5.0#the-label-tag-helper) (`<label asp-for="Movie.Title" class="control-label"></label>`) 生成标签描述和 `Title` 属性的 `[for]` 特性。

[输入标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/working-with-forms?view=aspnetcore-5.0) (`<input asp-for="Movie.Title" class="form-control">`) 使用 [DataAnnotations](https://docs.microsoft.com/zh-cn/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 属性并在客户端生成 jQuery 验证所需的 HTML 属性。

有关标记帮助程序（如 `<form method="post">`）的详细信息，请参阅 [ASP.NET Core 中的标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-5.0)。