本教程演示如何生成和修改 Blazor 应用。 您将学习如何：

- 创建待办事项列表 Blazor 应用项目
- 修改 Razor 组件
- 在组件中使用事件处理和数据绑定
- 在 Blazor 应用中使用路由

在本教程结束时，你将拥有一个正常运行的待办事项列表应用。

## 必备条件

[.NET 5.0 SDK 或更高版本](https://dotnet.microsoft.com/download/dotnet/5.0)

## 创建待办事项列表 Blazor 应用

1. 在命令行界面中创建名为 `TodoList` 的新 Blazor 应用：

   ```bash
   dotnet new blazorserver -o TodoList
   ```

   上述命令将创建一个名为 `TodoList` 的文件夹来保存应用。 `TodoList` 文件夹是项目的根文件夹。 使用以下命令将目录切换到 `TodoList` 文件夹：

   ```bash
   cd TodoList
   ```

2. 使用以下命令向 `Pages` 文件夹中的应用添加一个新的 `Todo` Razor 组件：

   ```dotnetcli
   dotnet new razorcomponent -n Todo -o Pages
   ```

    重要

   Razor 组件文件名要求首字母大写。 打开 `Pages` 文件夹，确认 `Todo` 组件文件名以大写字母 `T` 开头。 文件名应为 `Todo.razor`。

3. 在 `Pages/Todo.razor` 中为组件提供初始标记：

   ```html
   @page "/todo"
   
   <h3>Todo</h3>
   ```

   保存 `Pages/Todo.razor` 文件。

4. 将 `Todo` 组件添加到导航栏。

   `NavMenu` 组件 (`Shared/NavMenu.razor`) 用于应用的布局。 布局是可以避免应用中出现重复内容的组件。

   通过在 `Shared/NavMenu.razor` 文件中的现有列表项下添加以下列表项标记，为 `Todo` 组件添加一个 `<NavLink>` 元素：

   ```html
   <li class="nav-item px-3">
       <NavLink class="nav-link" href="todo">
           <span class="oi oi-list-rich" aria-hidden="true"></span> Todo
       </NavLink>
   </li>
   ```

   保存 `Shared/NavMenu.razor` 文件。

5. 从 `TodoList` 文件夹，在命令行界面中执行 [`dotnet watch run`](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/dotnet-watch) 命令，以生成并运行应用。 访问新的“待办事项”页面 (`https://localhost:5001/todo`)，确认指向 `Todo` 组件的边栏导航链接有效。

6. 向项目的根目录添加 `TodoItem.cs` 文件（`TodoList` 文件夹），以保存一个用于表示待办项的类。 为 `TodoItem` 类使用以下 C# 代码：

   ```csharp
   public class TodoItem
   {
       public string Title { get; set; }
       public bool IsDone { get; set; }
   }
   ```

7. 返回到 `Todo` 组件 (`Pages/Todo.razor`)：

   - 在 `@code` 块中为待办项添加一个字段。 `Todo` 组件使用此字段来维护待办项列表的状态。
   - 添加无序列表标记和 `foreach` 循环，以将每个待办项呈现为列表项 (`<li>`)。

   ```html
   @page "/todo"
   
   <h3>Todo</h3>
   
   <ul>
       @foreach (var todo in todos)
       {
           <li>@todo.Title</li>
       }
   </ul>
   
   @code {
       private IList<TodoItem> todos = new List<TodoItem>();
   }
   ```

8. 该应用需要 UI 元素来将待办项添加到列表。 在未排序列表 (`<ul>...</ul>`) 下方添加一个文本输入 (`<input>`) 和一个按钮 (`<button>`)：

   ```html
   @page "/todo"
   
   <h3>Todo</h3>
   
   <ul>
       @foreach (var todo in todos)
       {
           <li>@todo.Title</li>
       }
   </ul>
   
   <input placeholder="Something todo" />
   <button>Add todo</button>
   
   @code {
       private IList<TodoItem> todos = new List<TodoItem>();
   }
   ```

9. 保存 `TodoItem.cs` 文件和更新的 `Pages/Todo.razor` 文件。 在命令行界面中，保存文件时，将自动重新生成应用。 浏览器会暂时断开与该应用的连接，并在重新建立连接后重新加载页面。

10. 选择“`Add todo`”按钮时没有任何反应，因为没有事件处理程序连接到该按钮。

11. 向 `Todo` 组件添加 `AddTodo` 方法，并使用 `@onclick` 属性注册该方法以选择按钮。 选择按钮时，会调用 `AddTodo` C# 方法：

    ```html
    <input placeholder="Something todo" />
    <button @onclick="AddTodo">Add todo</button>
    
    @code {
        private IList<TodoItem> todos = new List<TodoItem>();
    
        private void AddTodo()
        {
            // Todo: Add the todo
        }
    }
    ```

12. 要获得新待办项标题，请在 `@code` 块顶部添加 `newTodo` 字符串字段，并使用 `<input>` 元素中的 `bind` 属性将其绑定到文本输入的值：

    

    ```html
    @code {
        private IList<TodoItem> todos = new List<TodoItem>();
        private string newTodo;
    
        ...
    }
    ```

    

    ```html
    <input placeholder="Something todo" @bind="newTodo" />
    ```

13. 更新 `AddTodo` 方法，将具有指定标题的 `TodoItem` 添加到列表。 通过将 `newTodo` 设置为空字符串来清除文本输入的值：

    

    ```html
    @page "/todo"
    
    <h3>Todo</h3>
    
    <ul>
        @foreach (var todo in todos)
        {
            <li>@todo.Title</li>
        }
    </ul>
    
    <input placeholder="Something todo" @bind="newTodo" />
    <button @onclick="AddTodo">Add todo</button>
    
    @code {
        private IList<TodoItem> todos = new List<TodoItem>();
        private string newTodo;
    
        private void AddTodo()
        {
            if (!string.IsNullOrWhiteSpace(newTodo))
            {
                todos.Add(new TodoItem { Title = newTodo });
                newTodo = string.Empty;
            }
        }
    }
    ```

14. 保存 `Pages/ToDo.razor` 文件。 应用程序会在命令行界面中自动重新生成。 浏览器重新连接到应用后，页面会在浏览器中重新加载。

15. 每个待办项的标题文本都可以编辑，复选框可以帮助用户跟踪已完成的项。 为每个待办项添加一个复选框输入，并将它的值绑定到 `IsDone` 属性。 将 `@todo.Title` 更改为绑定到 `@todo.Title` 的 `<input>` 元素：

    

    ```html
    <ul>
        @foreach (var todo in todos)
        {
            <li>
                <input type="checkbox" @bind="todo.IsDone" />
                <input @bind="todo.Title" />
            </li>
        }
    </ul>
    ```

16. 若要验证这些值是否已绑定，请更新 `<h3>` 标头以显示尚未完成的待办项计数（`IsDone` 是 `false`）。

    

    ```html
    <h3>Todo (@todos.Count(todo => !todo.IsDone))</h3>
    ```

17. 已完成的 `Todo` 组件 (`Pages/Todo.razor`)：

    

    ```html
    @page "/todo"
    
    <h3>Todo (@todos.Count(todo => !todo.IsDone))</h3>
    
    <ul>
        @foreach (var todo in todos)
        {
            <li>
                <input type="checkbox" @bind="todo.IsDone" />
                <input @bind="todo.Title" />
            </li>
        }
    </ul>
    
    <input placeholder="Something todo" @bind="newTodo" />
    <button @onclick="AddTodo">Add todo</button>
    
    @code {
        private IList<TodoItem> todos = new List<TodoItem>();
        private string newTodo;
    
        private void AddTodo()
        {
            if (!string.IsNullOrWhiteSpace(newTodo))
            {
                todos.Add(new TodoItem { Title = newTodo });
                newTodo = string.Empty;
            }
        }
    }
    ```

18. 保存 `Pages/ToDo.razor` 文件。 应用程序会在命令行界面中自动重新生成。 浏览器重新连接到应用后，页面会在浏览器中重新加载。

19. 添加待办项以测试新代码。

20. 完成后，请在命令行界面中关闭应用。 许多命令行界面接受键盘命令 Ctrl+c 来停止应用。

## 后续步骤

在本教程中，你将了解：

- 创建待办事项列表 Blazor 应用项目
- 修改 Razor 组件
- 在组件中使用事件处理和数据绑定
- 在 Blazor 应用中使用路由