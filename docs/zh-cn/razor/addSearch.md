# 添加搜索

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)（[如何下载](https://docs.microsoft.com/zh-cn/aspnet/core/introduction-to-aspnet-core?view=aspnetcore-5.0#how-to-download-a-sample)）。

在以下部分中，添加了按流派或名称搜索电影。

将以下突出显示的 using 语句和属性添加到 Pages/Movies/Index.cshtml.cs：

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.EntityFrameworkCore;
using RazorPagesMovie.Models;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace RazorPagesMovie.Pages.Movies
{

    public class IndexModel : PageModel
    {
        private readonly RazorPagesMovie.Data.RazorPagesMovieContext _context;

        public IndexModel(RazorPagesMovie.Data.RazorPagesMovieContext context)
        {
            _context = context;
        }

        public IList<Movie> Movie { get; set; }
        [BindProperty(SupportsGet = true)]
        public string SearchString { get; set; }
        public SelectList Genres { get; set; }
        [BindProperty(SupportsGet = true)]
        public string MovieGenre { get; set; }
```

在前面的代码中：

- `SearchString`：包含用户在搜索文本框中输入的文本。 `SearchString` 也有 [`[BindProperty\]`](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.bindpropertyattribute) 属性。 `[BindProperty]` 会绑定名称与属性相同的表单值和查询字符串。 在 HTTP GET 请求中进行绑定需要 `[BindProperty(SupportsGet = true)]`。
- `Genres`：包含流派列表。 `Genres` 使用户能够从列表中选择一种流派。 `SelectList` 需要 `using Microsoft.AspNetCore.Mvc.Rendering;`
- `MovieGenre`：包含用户选择的特定流派。 例如，“Western”。
- `Genres` 和 `MovieGenre` 会在本教程的后续部分中使用。

 警告

出于安全原因，必须选择绑定 `GET` 请求数据以对模型属性进行分页。 请在将用户输入映射到属性前对其进行验证。 当处理依赖查询字符串或路由值的方案时，选择加入 `GET` 绑定非常有用。

若要将属性绑定在 `GET` 请求上，请将 [](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.bindpropertyattribute) 特性的 `SupportsGet` 属性设置为 `true`：

```csharp
[BindProperty(SupportsGet = true)]
```

有关详细信息，请参阅 [ASP.NET Core Community Standup:Bind on GET discussion (YouTube)](https://www.youtube.com/watch?v=p7iHB9V-KVU&feature=youtu.be&t=54m27s)（绑定 GET 讨论）。

使用以下代码更新 Index 页面的 `OnGetAsync` 方法：

```csharp
public async Task OnGetAsync()
{
    var movies = from m in _context.Movie
                 select m;
    if (!string.IsNullOrEmpty(SearchString))
    {
        movies = movies.Where(s => s.Title.Contains(SearchString));
    }

    Movie = await movies.ToListAsync();
}
```

`OnGetAsync` 方法的第一行创建了 [LINQ](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/) 查询用于选择电影：

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

此时仅对查询进行了定义，它还 *不会* 针对数据库运行。

如果 `SearchString` 属性不为 null 或空，则电影查询会修改为根据搜索字符串进行筛选：

```csharp
if (!string.IsNullOrEmpty(SearchString))
{
    movies = movies.Where(s => s.Title.Contains(SearchString));
}
```

`s => s.Title.Contains()` 代码是 [Lambda 表达式](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)。 Lambda 在基于方法的 [LINQ](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/) 查询中用作标准查询运算符方法的参数，如 [Where](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) 方法或 `Contains`。 在对 LINQ 查询进行定义或通过调用方法（如 `Where`、`Contains` 或 `OrderBy`）进行修改后，此查询不会执行。 相反，会延迟执行查询。 表达式的计算会延迟，直到循环访问其实现的值或者调用 `ToListAsync` 方法为止。 有关详细信息，请参阅 [Query Execution](https://docs.microsoft.com/zh-cn/dotnet/framework/data/adonet/ef/language-reference/query-execution)（查询执行）。

 备注

[Contains](https://docs.microsoft.com/zh-cn/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) 方法在数据库中运行，而不是在 C# 代码中运行。 查询是否区分大小写取决于数据库和排序规则。 在 SQL Server 上，`Contains` 映射到 [SQL LIKE](https://docs.microsoft.com/zh-cn/sql/t-sql/language-elements/like-transact-sql)，这是不区分大小写的。 在 SQLite 中，由于使用了默认排序规则，因此需要区分大小写。

导航到电影页面，并向 URL 追加一个如 `?searchString=Ghost` 的查询字符串。 例如 `https://localhost:5001/Movies?searchString=Ghost`。 筛选的电影将显示出来。



![Index 视图](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/search/_static/ghost.png?view=aspnetcore-5.0)

如果向 Index 页面添加了以下路由模板，搜索字符串则可作为 URL 段传递。 例如 `https://localhost:5001/Movies/Ghost`。

```cshtml
@page "{searchString?}"
```

前面的路由约束允许按路由数据（URL 段）搜索标题，而不是按查询字符串值进行搜索。 `"{searchString?}"` 中的 `?` 表示这是可选路由参数。

![Index 视图，显示添加到 URL 的“ghost”一词，以及返回的两部电影（Ghostbusters 和 Ghostbusters 2）的电影列表](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/search/_static/g2.png?view=aspnetcore-5.0)

ASP.NET Core 运行时使用[模型绑定](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/models/model-binding?view=aspnetcore-5.0)，通过查询字符串 (`?searchString=Ghost`) 或路由数据 (`https://localhost:5001/Movies/Ghost`) 设置 `SearchString` 属性的值。 模型绑定 *不* 区分大小写。

但是，用户不能通过修改 URL 来搜索电影。 在此步骤中，会添加 UI 来筛选电影。 如果已添加路由约束 `"{searchString?}"`，请将它删除。

打开 _Pages/Movies/Index.cshtml* 文件，并添加以下代码中突出显示的标记：

```cshtml
@page
@model RazorPagesMovie.Pages.Movies.IndexModel

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-page="Create">Create New</a>
</p>

<form>
    <p>
        Title: <input type="text" asp-for="SearchString" />
        <input type="submit" value="Filter" />
    </p>
</form>

<table class="table">
    @*Markup removed for brevity.*@
```

HTML `<form>` 标记使用以下[标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-5.0)：

- [表单标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/working-with-forms?view=aspnetcore-5.0#the-form-tag-helper)。 提交表单时，筛选器字符串将通过查询字符串发送到 Pages/Movies/Index 页面。
- [输入标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/working-with-forms?view=aspnetcore-5.0#the-input-tag-helper)

保存更改并测试筛选器。

![显示标题筛选器文本框中键入了 ghost 一词的 Index 视图](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/search/_static/filter.png?view=aspnetcore-5.0)

## 按流派搜索

使用以下代码更新 Index 页面的 `OnGetAsync` 方法：

```csharp
public async Task OnGetAsync()
{
    // Use LINQ to get list of genres.
    IQueryable<string> genreQuery = from m in _context.Movie
                                    orderby m.Genre
                                    select m.Genre;

    var movies = from m in _context.Movie
                 select m;

    if (!string.IsNullOrEmpty(SearchString))
    {
        movies = movies.Where(s => s.Title.Contains(SearchString));
    }

    if (!string.IsNullOrEmpty(MovieGenre))
    {
        movies = movies.Where(x => x.Genre == MovieGenre);
    }
    Genres = new SelectList(await genreQuery.Distinct().ToListAsync());
    Movie = await movies.ToListAsync();
}
```

下面的代码是一种 LINQ 查询，可从数据库中检索所有流派。

```csharp
// Use LINQ to get list of genres.
IQueryable<string> genreQuery = from m in _context.Movie
                                orderby m.Genre
                                select m.Genre;
```

流派的 `SelectList` 是通过投影不包含重复值的流派创建的。

```csharp
Genres = new SelectList(await genreQuery.Distinct().ToListAsync());
```

### 将按流派搜索添加到 Razor 页面

1. 更新 Index.cshtml [`<form>` 元素] (https://developer.mozilla.org/docs/Web/HTML/Element/form) ，如以下标记中突出显示：

   CSHTML复制

   ```cshtml
   @page
   @model RazorPagesMovie.Pages.Movies.IndexModel
   
   @{
       ViewData["Title"] = "Index";
   }
   
   <h1>Index</h1>
   
   <p>
       <a asp-page="Create">Create New</a>
   </p>
   
   <form>
       <p>
           <select asp-for="MovieGenre" asp-items="Model.Genres">
               <option value="">All</option>
           </select>
           Title: <input type="text" asp-for="SearchString" />
           <input type="submit" value="Filter" />
       </p>
   </form>
   
   <table class="table">
       @*Markup removed for brevity.*@
   ```

2. 通过按流派或/和电影标题搜索来测试应用。