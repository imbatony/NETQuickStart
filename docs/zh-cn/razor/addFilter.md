# 页面筛选器

Razor 页面筛选器 [IPageFilter](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter?view=aspnetcore-2.0) 和 [IAsyncPageFilter](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter?view=aspnetcore-2.0) 允许 Razor Pages 在运行 Razor 页面处理程序的前后运行代码。 Razor 页面筛选器与 [ASP.NET Core MVC 操作筛选器](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/filters?view=aspnetcore-5.0#action-filters)类似，但它们不能应用于单个页面处理程序方法。

Razor 页面筛选器：

- 在选择处理程序方法后但在模型绑定发生前运行代码。
- 在模型绑定完成后，执行处理程序方法前运行代码。
- 在执行处理程序方法后运行代码。
- 可在页面或全局范围内实现。
- 无法应用于特定的页面处理程序方法。
- 可以用[依赖项注入](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-5.0) (DI) 填充构造函数依赖项。 有关详细信息，请参阅 [ServiceFilterAttribute](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/filters?view=aspnetcore-5.0#servicefilterattribute) 和 [TypeFilterAttribute](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/filters?view=aspnetcore-5.0#typefilterattribute)。

虽然页构造函数和中间件允许在处理程序方法执行之前执行自定义代码，但只有 Razor 页面筛选器允许访问 [HttpContext](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.httpcontext#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_HttpContext) 和页面。 中间件可以访问 `HttpContext`，但不能访问“页面上下文”。 筛选器具有 [FilterContext](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.filtercontext) 派生参数，该参数提供对 `HttpContext` 的访问权限。 下面是页面筛选器的示例：[实现筛选器属性](https://docs.microsoft.com/zh-cn/aspnet/core/razor-pages/filter?view=aspnetcore-5.0#ifa)，该属性将标头添加到响应中，而使用构造函数或中间件则无法做到这点。 对页面上下文的访问（包括对页面实例及其模型的访问）仅在执行筛选器、处理程序或 Razor 页面的正文时适用。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/filter/3.1sample)（[如何下载](https://docs.microsoft.com/zh-cn/aspnet/core/introduction-to-aspnet-core?view=aspnetcore-5.0#how-to-download-a-sample)）

Razor 页面筛选器提供的以下方法可在全局或页面级应用：

- 同步方法：
  - [OnPageHandlerSelected](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerselected?view=aspnetcore-2.0)：在选择处理程序方法后，但在模型绑定发生之前调用。
  - [OnPageHandlerExecuting](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuting?view=aspnetcore-2.0)：在模型绑定完成后，执行处理程序方法之前调用。
  - [OnPageHandlerExecuted](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.ipagefilter.onpagehandlerexecuted?view=aspnetcore-2.0)：在执行处理器方法后，生成操作结果前调用。
- 异步方法：
  - [OnPageHandlerSelectionAsync](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerselectionasync?view=aspnetcore-2.0)：在选择处理程序方法后，但在模型绑定发生前，进行异步调用。
  - [OnPageHandlerExecutionAsync](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncpagefilter.onpagehandlerexecutionasync?view=aspnetcore-2.0)：在调用处理程序方法前，但在模型绑定结束后，进行异步调用。

筛选器接口的同步和异步版本任意实现一个，而不是同时实现 。 该框架会先查看筛选器是否实现了异步接口，如果是，则调用该接口。 如果不是，则调用同步接口的方法。 如果两个接口都已实现，则只会调用异步方法。 对页面中的替代应用相同的规则，同步替代或异步替代只能任选其一实现，不可二者皆选。

## 全局实现 Razor 页面筛选器

以下代码实现了 `IAsyncPageFilter`：

```csharp
public class SampleAsyncPageFilter : IAsyncPageFilter
{
    private readonly IConfiguration _config;

    public SampleAsyncPageFilter(IConfiguration config)
    {
        _config = config;
    }

    public Task OnPageHandlerSelectionAsync(PageHandlerSelectedContext context)
    {
        var key = _config["UserAgentID"];
        context.HttpContext.Request.Headers.TryGetValue("user-agent",
                                                        out StringValues value);
        ProcessUserAgent.Write(context.ActionDescriptor.DisplayName,
                               "SampleAsyncPageFilter.OnPageHandlerSelectionAsync",
                               value, key.ToString());

        return Task.CompletedTask;
    }

    public async Task OnPageHandlerExecutionAsync(PageHandlerExecutingContext context,
                                                  PageHandlerExecutionDelegate next)
    {
        // Do post work.
        await next.Invoke();
    }
}
```

在前面的代码中，`ProcessUserAgent.Write` 是用户提供的与用户代理字符串一起使用的代码。

以下代码启用 `Startup` 类中的 `SampleAsyncPageFilter`：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages()
        .AddMvcOptions(options =>
        {
            options.Filters.Add(new SampleAsyncPageFilter(Configuration));
        });
}
```

以下代码调用 [AddFolderApplicationModelConvention](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.pageconventioncollection.addfolderapplicationmodelconvention)，将 `SampleAsyncPageFilter` 仅应用于 /Movies 中的页面：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages(options =>
    {
        options.Conventions.AddFolderApplicationModelConvention(
            "/Movies",
            model => model.Filters.Add(new SampleAsyncPageFilter(Configuration)));
    });
}
```

以下代码实现同步的 `IPageFilter`：

```csharp
public class SamplePageFilter : IPageFilter
{
    private readonly IConfiguration _config;

    public SamplePageFilter(IConfiguration config)
    {
        _config = config;
    }

    public void OnPageHandlerSelected(PageHandlerSelectedContext context)
    {
        var key = _config["UserAgentID"];
        context.HttpContext.Request.Headers.TryGetValue("user-agent", out StringValues value);
        ProcessUserAgent.Write(context.ActionDescriptor.DisplayName,
                               "SamplePageFilter.OnPageHandlerSelected",
                               value, key.ToString());
    }

    public void OnPageHandlerExecuting(PageHandlerExecutingContext context)
    {
        Debug.WriteLine("Global sync OnPageHandlerExecuting called.");
    }

    public void OnPageHandlerExecuted(PageHandlerExecutedContext context)
    {
        Debug.WriteLine("Global sync OnPageHandlerExecuted called.");
    }
}
```

以下代码启用 `SamplePageFilter`：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages()
        .AddMvcOptions(options =>
        {
            options.Filters.Add(new SamplePageFilter(Configuration));
        });
}
```

## 通过重写筛选器方法实现 Razor 页面筛选器

以下代码替代异步 Razor 页面筛选器：

```csharp
public class IndexModel : PageModel
{
    private readonly IConfiguration _config;

    public IndexModel(IConfiguration config)
    {
        _config = config;
    }

    public override Task OnPageHandlerSelectionAsync(PageHandlerSelectedContext context)
    {
        Debug.WriteLine("/IndexModel OnPageHandlerSelectionAsync");
        return Task.CompletedTask;
    }

    public async override Task OnPageHandlerExecutionAsync(PageHandlerExecutingContext context, 
                                                           PageHandlerExecutionDelegate next)
    {
        var key = _config["UserAgentID"];
        context.HttpContext.Request.Headers.TryGetValue("user-agent", out StringValues value);
        ProcessUserAgent.Write(context.ActionDescriptor.DisplayName,
                               "/IndexModel-OnPageHandlerExecutionAsync",
                                value, key.ToString());

        await next.Invoke();
    }
}
```



## 实现筛选器属性

基于属性的内置筛选器 [OnResultExecutionAsync](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.mvc.filters.iasyncresultfilter.onresultexecutionasync) 可以进行子类化。 以下筛选器向响应添加标头：

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc.Filters;

namespace PageFilter.Filters
{
    public class AddHeaderAttribute  : ResultFilterAttribute
    {
        private readonly string _name;
        private readonly string _value;

        public AddHeaderAttribute (string name, string value)
        {
            _name = name;
            _value = value;
        }

        public override void OnResultExecuting(ResultExecutingContext context)
        {
            context.HttpContext.Response.Headers.Add(_name, new string[] { _value });
        }
    }
}
```

以下代码应用 `AddHeader` 属性：

```csharp
using Microsoft.AspNetCore.Mvc.RazorPages;
using PageFilter.Filters;

namespace PageFilter.Movies
{
    [AddHeader("Author", "Rick")]
    public class TestModel : PageModel
    {
        public void OnGet()
        {

        }
    }
}
```

使用浏览器开发人员工具等工具来检查标头。 在响应标头下，将显示 `author: Rick`。

有关重写顺序的说明，请参阅[重写默认顺序](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/filters?view=aspnetcore-5.0#overriding-the-default-order)。

有关将筛选器管道与筛选器短路的说明，请参阅[取消和设置短路](https://docs.microsoft.com/zh-cn/aspnet/core/mvc/controllers/filters?view=aspnetcore-5.0#cancellation-and-short-circuiting)。



## 授权筛选器属性

[Authorize](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute?view=aspnetcore-2.0) 属性可应用于 `PageModel`：

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace PageFilter.Pages
{
    [Authorize]
    public class ModelWithAuthFilterModel : PageModel
    {
        public IActionResult OnGet() => Page();
    }
}
```