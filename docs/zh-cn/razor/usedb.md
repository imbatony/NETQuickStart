# 使用数据库

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50)

`RazorPagesMovieContext` 对象处理连接到数据库并将 `Movie` 对象映射到数据库记录的任务。 在 Startup.cs 的 `ConfigureServices` 方法中向[依赖关系注入](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-5.0)容器注册数据库上下文：



```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages();

    services.AddDbContext<RazorPagesMovieContext>(options =>
      options.UseSqlServer(Configuration.GetConnectionString("RazorPagesMovieContext")));
}
```

ASP.NET Core [配置](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0)系统会读取 `ConnectionString` 键。 进行本地开发时，配置从 appsettings.json 文件获取连接字符串。

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "RazorPagesMovieContext": "Server=(localdb)\\mssqllocaldb;Database=RazorPagesMovieContext-bc;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

将应用部署到测试或生产服务器时，可以使用环境变量将连接字符串设置为测试或生产数据库服务器。 有关详细信息，请参阅[配置](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/configuration/?view=aspnetcore-5.0)。

## SQL Server Express LocalDB

LocalDB 是轻型版的 SQL Server Express 数据库引擎，以程序开发为目标。 LocalDB 作为按需启动并在用户模式下运行的轻量级数据库没有复杂的配置。 默认情况下，LocalDB 数据库在 `C:\Users\<user>\` 目录下创建 `*.mdf` 文件。



1. 从“视图”菜单中，打开“SQL Server 对象资源管理器”(SSOX) 。

   ![“视图”菜单](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql/_static/5/ssox.png?view=aspnetcore-5.0)

2. 右键单击 `Movie` 表，然后选择“视图设计器”：

   ![Movie 表上打开的上下文菜单](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql/_static/5/design.png?view=aspnetcore-5.0)

   ![设计器中打开的 Movie 表](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql/_static/dv.png?view=aspnetcore-5.0)

   请注意 `ID` 旁边的密钥图标。 默认情况下，EF 为该主键创建一个名为 `ID` 的属性。

3. 右键单击 `Movie` 表，然后选择“查看数据”：

   ![显示表数据的打开的 Movie 表](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql/_static/vd22.png?view=aspnetcore-5.0)

##  设定数据库种子

使用以下代码在 Models 文件夹中创建一个名为 `SeedData` 的新类：

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using RazorPagesMovie.Data;
using System;
using System.Linq;

namespace RazorPagesMovie.Models
{
    public static class SeedData
    {
        public static void Initialize(IServiceProvider serviceProvider)
        {
            using (var context = new RazorPagesMovieContext(
                serviceProvider.GetRequiredService<
                    DbContextOptions<RazorPagesMovieContext>>()))
            {
                // Look for any movies.
                if (context.Movie.Any())
                {
                    return;   // DB has been seeded
                }

                context.Movie.AddRange(
                    new Movie
                    {
                        Title = "When Harry Met Sally",
                        ReleaseDate = DateTime.Parse("1989-2-12"),
                        Genre = "Romantic Comedy",
                        Price = 7.99M
                    },

                    new Movie
                    {
                        Title = "Ghostbusters ",
                        ReleaseDate = DateTime.Parse("1984-3-13"),
                        Genre = "Comedy",
                        Price = 8.99M
                    },

                    new Movie
                    {
                        Title = "Ghostbusters 2",
                        ReleaseDate = DateTime.Parse("1986-2-23"),
                        Genre = "Comedy",
                        Price = 9.99M
                    },

                    new Movie
                    {
                        Title = "Rio Bravo",
                        ReleaseDate = DateTime.Parse("1959-4-15"),
                        Genre = "Western",
                        Price = 3.99M
                    }
                );
                context.SaveChanges();
            }
        }
    }
}
```

如果数据库中有任何电影，则会返回种子初始值设定项，并且不会添加任何电影。

```csharp
if (context.Movie.Any())
{
    return;
}
```

### 添加种子初始值设定项

将 Program.cs 的内容替换为以下代码：

C#复制

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using RazorPagesMovie.Models;
using System;

namespace RazorPagesMovie
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();

            using (var scope = host.Services.CreateScope())
            {
                var services = scope.ServiceProvider;

                try
                {
                    SeedData.Initialize(services);
                }
                catch (Exception ex)
                {
                    var logger = services.GetRequiredService<ILogger<Program>>();
                    logger.LogError(ex, "An error occurred seeding the DB.");
                }
            }

            host.Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}
```

前面的代码修改了 `Main` 方法，以便执行以下操作：

- 从依赖注入容器中获取数据库上下文实例。
- 调用 `seedData.Initialize` 方法，并向其传递数据库上下文实例。
- Seed 方法完成时释放上下文。 [using 语句](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/using-statement)将确保释放上下文。

未运行 `Update-Database` 时出现以下异常：

> ```bash
> SqlException: Cannot open database "RazorPagesMovieContext-" requested by the login. The login failed.` `Login failed for user 'user name'.
> ```

### 测试应用

1. 删除数据库中的所有记录。 使用浏览器中的删除链接，也可以从 [SSOX](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/new-field?view=aspnetcore-5.0#ssox) 执行此操作

2. 通过调用 `Startup` 类中的方法强制应用初始化，使种子方法能够正常运行。 若要强制进行初始化，必须先停止 IIS Express，然后再重新启动它。 使用以下任一方法停止并重启 IIS：

   1. 右键单击通知区域中的 IIS Express 系统任务栏图标，然后选择“退出”或“停止站点”：

      ![IIS Express 系统任务栏图标](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/first-mvc-app/working-with-sql/_static/iisexicon.png?view=aspnetcore-5.0)

      ![上下文菜单](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql/_static/stopiis.png?view=aspnetcore-5.0)

   2. 如果应用是在非调试模式下运行的，请按 F5 以在调试模式下运行。

   3. 如果应用处于调试模式下，请停止调试程序并按 F5。

应用将显示设定为种子的数据：

![在浏览器中打开的显示电影数据的电影应用程序](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/razor-pages/sql/_static/5/m55.png?view=aspnetcore-5.0)