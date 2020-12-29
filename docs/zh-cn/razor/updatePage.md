# 更新生成的页面

构架的电影应用有个不错的开始，但是展示效果还不够理想。 ReleaseDate 应是两个词 (Release Date)。

![在 Chrome 中打开的电影应用程序](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql/_static/5/m55.png?view=aspnetcore-5.0)

## 更新生成的代码

打开 Models/Movie.cs 文件，并添加以下代码中突出显示的行：

```csharp
using System;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace RazorPagesMovie.Models
{
    public class Movie
    {
        public int ID { get; set; }
        public string Title { get; set; }

        [Display(Name = "Release Date")]
        [DataType(DataType.Date)]
        public DateTime ReleaseDate { get; set; }
        public string Genre { get; set; }

        [Column(TypeName = "decimal(18, 2)")]
        public decimal Price { get; set; }
    }
}
```

在前面的代码中：

- `[Column(TypeName = "decimal(18, 2)")]` 数据注释使 Entity Framework Core 可以将 `Price` 正确映射到数据库中的货币。 有关详细信息，请参阅[数据类型](https://docs.microsoft.com/zh-cn/ef/core/modeling/relational/data-types)。
- [[Display\]](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.metadata.displaymetadata) 属性指定字段的显示名称。 在前面的代码中即“Release Date”，而非“ReleaseDate”。
- [[DataType\]](https://docs.microsoft.com/zh-cn/dotnet/api/system.componentmodel.dataannotations.datatypeattribute) 属性指定数据的类型 (`Date`)。 此字段中存储的时间信息不显示。

[DataAnnotations](https://docs.microsoft.com/zh-cn/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) 未包括在下一个教程中。

浏览到 Pages/Movies，并将鼠标悬停在“编辑”链接上以查看目标 URL。

![鼠标悬停在“编辑”链接上的浏览器窗口，显示了 https://localhost:1234/Movies/Edit/5 的链接 URL](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/da1/edit7.png?view=aspnetcore-5.0)

“编辑”、“详细信息”和“删除”链接是在 Pages/Movies/Index.cshtml 文件中由[定位标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/tag-helpers/built-in/anchor-tag-helper?view=aspnetcore-5.0)生成的 。

```html
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

[标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-5.0)使服务器端代码可以在 Razor 文件中参与创建和呈现 HTML 元素。

在前面的代码中，[定位标记帮助程序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/views/tag-helpers/built-in/anchor-tag-helper?view=aspnetcore-5.0)从 Razor 页面（路由是相对的）、`asp-page` 和路由标识符 (`asp-route-id`) 动态生成 HTML `href` 特性值。 有关详细信息，请参阅[页面的 URL 生成](https://docs.microsoft.com/zh-cn/aspnet/core/razor-pages/?view=aspnetcore-5.0#url-generation-for-pages)。

在浏览器中使用“查看源”来检查生成的标记。 生成的 HTML 的一部分如下所示：

```html
<td>
  <a href="/Movies/Edit?id=1">Edit</a> |
  <a href="/Movies/Details?id=1">Details</a> |
  <a href="/Movies/Delete?id=1">Delete</a>
</td>
```

动态生成的链接通过查询字符串传递电影 ID。 例如，`https://localhost:5001/Movies/Details?id=1` 中的 `?id=1`。

### 添加路由模板

更新“编辑”、“详细信息”和“删除”Razor 页面以使用 `{id:int}` 路由模板。 将上述每个页面的页面指令从 `@page` 更改为 `@page "{id:int}"`。 运行应用，然后查看源。

生成的 HTML 会将 ID 添加到 URL 的路径部分：

```html
<td>
  <a href="/Movies/Edit/1">Edit</a> |
  <a href="/Movies/Details/1">Details</a> |
  <a href="/Movies/Delete/1">Delete</a>
</td>
```

如果对具有 `{id:int}` 路由模板的页面进行的请求中不包含整数，则将返回 HTTP 404（未找到）错误。 例如，`https://localhost:5001/Movies/Details` 将返回 404 错误。 若要使 ID 可选，请将 `?` 追加到路由约束：

CSHTML复制

```cshtml
@page "{id:int?}"
```

测试 `@page "{id:int?}"` 的行为：

1. 在 Pages/Movies/Details.cshtml 中将 page 指令设置为 `@page "{id:int?}"`。
2. 在 `public async Task<IActionResult> OnGetAsync(int? id)` 中（位于 Pages/Movies/Details.cshtml.cs 中）设置断点。
3. 导航到 `https://localhost:5001/Movies/Details/`。

使用 `@page "{id:int}"` 指令时，永远不会命中断点。 路由引擎返回 HTTP 404。 使用 `@page "{id:int?}"` 时，`OnGetAsync` 方法返回 `NotFound` (HTTP 404)：

```csharp
public async Task<IActionResult> OnGetAsync(int? id)
{
    if (id == null)
    {
        return NotFound();
    }

    Movie = await _context.Movie.FirstOrDefaultAsync(m => m.ID == id);

    if (Movie == null)
    {
        return NotFound();
    }
    return Page();
}
```

### 查看并发异常处理

查看 Pages/Movies/Edit.cshtml.cs 文件中的 `OnPostAsync` 方法：

```csharp
public async Task<IActionResult> OnPostAsync()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }

    _context.Attach(Movie).State = EntityState.Modified;

    try
    {
        await _context.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException)
    {
        if (!MovieExists(Movie.ID))
        {
            return NotFound();
        }
        else
        {
            throw;
        }
    }

    return RedirectToPage("./Index");
}

private bool MovieExists(int id)
{
    return _context.Movie.Any(e => e.ID == id);
}
```

当一个客户端删除电影并且另一个客户端对电影发布更改时，前面的代码会检测并发异常。

测试 `catch` 块：

1. 在 `catch (DbUpdateConcurrencyException)` 上设置断点。
2. 对电影选择“编辑”，进行更改，但不要输入“保存”。
3. 在其他浏览器窗口中，选择同一电影的“删除”链接，然后删除此电影。
4. 在之前的浏览器窗口中，将更改发布到电影。

生产代码可能要检测并发冲突。 有关详细信息，请参阅[处理并发冲突](https://docs.microsoft.com/zh-cn/aspnet/core/data/ef-rp/concurrency?view=aspnetcore-5.0)。

### 发布和绑定审阅

检查 Pages/Movies/Edit.cshtml.cs 文件：

```csharp
public class EditModel : PageModel
{
    private readonly RazorPagesMovie.Data.RazorPagesMovieContext _context;

    public EditModel(RazorPagesMovie.Data.RazorPagesMovieContext context)
    {
        _context = context;
    }

    [BindProperty]
    public Movie Movie { get; set; }

    public async Task<IActionResult> OnGetAsync(int? id)
    {
        if (id == null)
        {
            return NotFound();
        }

        Movie = await _context.Movie.FirstOrDefaultAsync(m => m.ID == id);

        if (Movie == null)
        {
            return NotFound();
        }
        return Page();
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }

        _context.Attach(Movie).State = EntityState.Modified;

        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException)
        {
            if (!MovieExists(Movie.ID))
            {
                return NotFound();
            }
            else
            {
                throw;
            }
        }

        return RedirectToPage("./Index");
    }

    private bool MovieExists(int id)
    {
        return _context.Movie.Any(e => e.ID == id);
    }
```

当对 Movies/Edit 页面（例如 `https://localhost:5001/Movies/Edit/3`）进行 HTTP GET 请求时：

- `OnGetAsync` 方法从数据库提取电影并返回 `Page` 方法。
- `Page` 方法呈现 Pages/Movies/Edit.cshtml Razor 页面。 Pages/Movies/Edit.cshtml 文件包含模型指令 `@model RazorPagesMovie.Pages.Movies.EditModel`，这使电影模型在页面上可用。
- “编辑”表单中会显示电影的值。

当发布 Movies/Edit 页面时：

- 此页面上的表单值将绑定到 `Movie` 属性。 `[BindProperty]` 特性会启用[模型绑定](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/models/model-binding?view=aspnetcore-5.0)。

  C#复制

  ```csharp
  [BindProperty]
  public Movie Movie { get; set; }
  ```

- 如果模型状态中存在错误（例如，`ReleaseDate` 无法被转换为日期），则会使用已提交的值重新显示表单。

- 如果没有模型错误，则电影已保存。

Index、“创建”和“删除”Razor页面中的 HTTP GET 方法遵循一个类似的模式。 “创建”Razor 页面中的 HTTP POST `OnPostAsync` 方法遵循的模式类似于“编辑”Razor 页面中的 `OnPostAsync` 方法所遵循的模式。