# 添加字段



在此部分中，[Entity Framework](https://docs.microsoft.com/zh-cn/ef/core/get-started/aspnetcore/new-db) Code First 迁移用于：

- 将新字段添加到模型。
- 将新字段架构更改迁移到数据库。

使用 EF Code First 自动创建数据库时，Code First 将：

- 向数据库添加 [`__EFMigrationsHistory`](https://docs.microsoft.com/zh-cn/ef/core/managing-schemas/migrations/history-table) 表格，以跟踪数据库的架构是否与从生成它的模型类同步。
- 如果该模型类未与数据库同步，EF 将引发异常。

自动验证架构与模型是否同步可以更容易地发现不一致的数据库代码问题。

## 向电影模型添加分级属性

1. 打开 Models/Movie.cs 文件，并添加 `Rating` 属性：

   ```csharp
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
       public string Rating { get; set; }
   }
   ```

2. 构建应用程序。

3. 编辑 Pages/Movies/Index.cshtml，并添加 `Rating` 字段：

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
               <th>
                   @Html.DisplayNameFor(model => model.Movie[0].Rating)
               </th>
               <th></th>
           </tr>
       </thead>
       <tbody>
           @foreach (var item in Model.Movie)
           {
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
                       @Html.DisplayFor(modelItem => item.Rating)
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

4. 更新以下页面：

   1. 将 `Rating` 字段添加到“删除”和“详细信息”页面。
   2. 使用 `Rating` 字段更新 [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Create.cshtml)。
   3. 将 `Rating` 字段添加到“编辑”页面。

在数据库更新为包括新字段之前，应用将不会正常工作。 在不更新数据库的情况下运行应用会引发 `SqlException`：

```
SqlException: Invalid column name 'Rating'.
```

`SqlException` 异常是由于更新的 Movie 模型类与数据库的 Movie 表架构不同导致的。 数据库表中没有 `Rating` 列。

可通过几种方法解决此错误：

1. 让 Entity Framework 自动丢弃并使用新的模型类架构重新创建数据库。 此方法在开发周期早期很方便，通过该方法可以一起快速改进模型和数据库架构。 此方法的缺点是会导致数据库中的现有数据丢失。 请勿对生产数据库使用此方法！ 在架构更改时丢弃数据库，并使用初始化表达式通过测试数据自动设定数据库种子，这通常是开发应用的有效方式。
2. 对现有数据库架构进行显式修改，使它与模型类相匹配。 此方法的优点是可以保留数据。 可以手动或通过创建数据库更改脚本进行此更改。
3. 使用 Code First 迁移更新数据库架构。

对于本教程，请使用 Code First 迁移。

更新 `SeedData` 类，使它提供新列的值。 示例更改如下所示，但对每个 `new Movie` 块做出此更改。

```csharp
context.Movie.AddRange(
    new Movie
    {
        Title = "When Harry Met Sally",
        ReleaseDate = DateTime.Parse("1989-2-12"),
        Genre = "Romantic Comedy",
        Price = 7.99M,
        Rating = "R"
    },
```

请参阅[已完成的 SeedData.cs 文件](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Models/SeedDataRating.cs)。

生成解决方案。

- [Visual Studio](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/new-field?view=aspnetcore-5.0&tabs=visual-studio#tabpanel_CeZOj-G++Q_visual-studio)
- [Visual Studio Code / Visual Studio for Mac](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/new-field?view=aspnetcore-5.0&tabs=visual-studio#tabpanel_CeZOj-G++Q_visual-studio-code+visual-studio-mac)



### 添加用于评级字段的迁移

1. 从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”。

2. 在 PMC 中，输入以下命令：

   PowerShell复制

   ```powershell
   Add-Migration Rating
   Update-Database
   ```

`Add-Migration` 命令会通知框架执行以下操作：

- 将 `Movie` 模型与 `Movie` 数据库架构进行比较。
- 创建代码以将数据库架构迁移到新模型。

名称“Rating”是任意的，用于对迁移文件进行命名。 为迁移文件使用有意义的名称是有帮助的。

`Update-Database` 命令指示框架将架构更改应用到数据库并保留现有数据。



如果删除数据库中的所有记录，初始化表达式会设定数据库种子，并将包括 `Rating` 字段。 可以使用浏览器中的删除链接，也可以从 [Sql Server 对象资源管理器](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql?view=aspnetcore-5.0#ssox) (SSOX) 执行此操作。

另一个方案是删除数据库，并使用迁移来重新创建该数据库。 删除 SSOX 中的数据库：

1. 在 SSOX 中选择数据库。

2. 右键单击数据库，并选择“删除”。

3. 检查“关闭现有连接”。

4. 选择“确定” 。

5. 在 [PMC](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/new-field?view=aspnetcore-5.0#pmc) 中更新数据库：

   PowerShell复制

   ```powershell
   Update-Database
   ```

运行应用，并验证是否可以创建/编辑/显示具有 `Rating` 字段的电影。 如果数据库未设定种子，则在 `SeedData.Initialize` 方法中设置断点。