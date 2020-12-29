在本节中，添加了用于管理数据库中的电影的类。

应用的模型类使用 Entity Framework Core (EF Core) 来处理数据库。 EF Core 是一种对象关系映射器 (ORM)，可简化数据访问。 首先要编写模型类，然后 EF Core 将创建数据库。
模型类称为 POCO 类（源自“简单传统 CLR 对象” ），因为它们与 EF Core 没有任何依赖关系。 它们定义数据库中存储的数据属性。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)

## 添加数据模型

1. 在“解决方案资源管理器”中，右键单击“RazorPagesMovie”项目 >“添加” > “新建文件夹”。 将文件夹命名为“Models”。
2. 右键单击“Models”文件夹。 选择“添加” > “类” 。 将类命名“Movie”。
3. 向 Movie 类添加以下属性：

``` csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace RazorPagesMovie.Models
{
    public class Movie
    {
        public int ID { get; set; }
        public string Title { get; set; }

        [DataType(DataType.Date)]
        public DateTime ReleaseDate { get; set; }
        public string Genre { get; set; }
        public decimal Price { get; set; }
    }
}
```

Movie 类包含：
- 数据库需要 ID 字段以获取主键。
- [DataType(DataType.Date)]：[DataType] 属性指定数据的类型 (Date)。 通过此特性：
    - 用户无需在日期字段中输入时间信息。
    - 仅显示日期，而非时间信息。

## 搭建“电影”模型的基架
在此部分，将搭建“电影”模型的基架。 确切地说，基架工具将生成页面，用于对“电影”模型执行创建、读取、更新和删除 (CRUD) 操作。

1. 创建“Pages/Movies”文件夹：
    1. 右键单击“Pages”文件夹 >“添加”>“新建文件夹”。
    2. 将文件夹命名为“Movies”。

2. 右键单击“Pages/Movies”文件夹 >“添加”>“新搭建基架的项目”。

![https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/5/sca.png?view=aspnetcore-5.0](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/5/sca.png?view=aspnetcore-5.0)

3. 在“添加基架”对话框中，依次选择“使用实体框架的 Razor 页面 (CRUD)”>“添加”

![https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/add_scaffold.png?view=aspnetcore-5.0](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/add_scaffold.png?view=aspnetcore-5.0)

4. 完成“添加使用实体框架的 Razor 页面 (CRUD)”对话框：
    1. 在“模型类”下拉列表中，选择“Movie (RazorPagesMovie.Models)” 。
    2. 在“数据上下文类”行中，选择 +（加号） 。
    3. 在“添加数据上下文”对话框中，将生成类名称 RazorPagesMovie.Data.RazorPagesMovieContext。
    4. 选择“添加” 。

![https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/3/arp.png?view=aspnetcore-5.0](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/3/arp.png?view=aspnetcore-5.0)

appsettings.json 文件通过用于连接到本地数据的连接字符串进行更新。


### 创建的文件

在搭建基架时，会创建并更新以下文件：
- Pages/Movies：“创建”、“删除”、“详细信息”和 Index。
- Data/RazorPagesMovieContext.cs

### 更新文件
Startup.cs

### 使用 EF 的迁移功能创建初始数据库架构

Entity Framework Core 中的迁移功能提供了一种方法来执行以下操作：
- 创建初始数据库架构。
- 以增量的方式更新数据库架构，使其与应用程序的数据模型保持同步。 保存数据库中的现有数据。


在此部分中，程序包管理器控制台 (PMC) 窗口用于：
- 添加初始迁移。
- 使用初始迁移来更新数据库。

1. 从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台” 

![https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/5/pmc.png?view=aspnetcore-5.0](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/5/pmc.png?view=aspnetcore-5.0)

2. 在 PMC 中，输入以下命令：

``` bash
Add-Migration InitialCreate
Update-Database
```

前面的命令生成以下警告：“No type was specified for the decimal column 'Price' on entity type 'Movie'. This will cause values to be silently truncated if they do not fit in the default precision and scale. Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'.”

忽略警告，因为它将在后面的步骤中得到解决。

`migrations` 命令生成用于创建初始数据库架构的代码。 该架构基于在 `DbContext` 中指定的模型。 `InitialCreate` 参数用于为迁移命名。 可以使用任何名称，但是按照惯例，会选择可说明迁移的名称。<br>
`update` 命令在尚未应用的迁移中运行 `Up` 方法。 在这种情况下，`update` 在用于创建数据库的 Migrations/<time-stamp>_InitialCreate.cs 文件中运行 Up 方法。

## 检查通过依赖关系注入注册的上下文


ASP.NET Core 通过`依赖关系注入`进行生成。 在应用程序启动过程中，通过依赖注入注册相关服务（例如 EF Core 数据库上下文）。 需要这些服务（如 Razor 页面）的组件通过构造函数参数提供相应服务。 本教程的后续部分介绍了用于获取数据库上下文实例的构造函数代码。
基架工具自动创建数据库上下文并将其注册到依赖关系注入容器。
检查 `Startup.ConfigureServices` 方法。 基架添加了突出显示的行：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages();

    services.AddDbContext<RazorPagesMovieContext>(options =>
      options.UseSqlServer(Configuration.GetConnectionString("RazorPagesMovieContext")));
}

```


RazorPagesMovieContext 为 Movie 模型协调 EF Core 功能，例如“创建”、“读取”、“更新”和“删除”。  
数据上下文 (RazorPagesMovieContext) 派生自 `Microsoft.EntityFrameworkCore.DbContext`。 数据上下文指定数据模型中包含哪些实体

```csharp
using Microsoft.EntityFrameworkCore;

namespace RazorPagesMovie.Data
{
    public class RazorPagesMovieContext : DbContext
    {
        public RazorPagesMovieContext (
            DbContextOptions<RazorPagesMovieContext> options)
            : base(options)
        {
        }

        public DbSet<RazorPagesMovie.Models.Movie> Movie { get; set; }
    }
}
```

前面的代码为实体集创建 DbSet<Movie> 属性。 在实体框架术语中，实体集通常与数据表相对应。 实体对应表中的行。  
通过调用 DbContextOptions 对象中的一个方法将连接字符串名称传递到上下文。 进行本地开发时，配置系统在 appsettings.json 文件中读取连接字符串。


## 测试应用

1. 运行应用并将 /Movies 追加到浏览器中的 URL (http://localhost:port/movies)
2. 测试“创建”链接。  

![https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/conan5.png?view=aspnetcore-5.0](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/model/_static/conan5.png?view=aspnetcore-5.0)

3. 测试“编辑”、“详细信息”和“删除”链接。