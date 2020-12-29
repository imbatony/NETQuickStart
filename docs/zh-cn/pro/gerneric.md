### 一、概述

泛型我们一定都用过，最常见的 List<T> 集合。.NET2.0 开始支持泛型，创建的目的就是为了**`不同类型创建相同的方法或类，也包括接口，委托的泛型`**。比如常见的 ORM 映射，一个方法通过传入不同的类，返回不同的类实例，再调用时才确定参数类型。

我们知道想要一个类相同名称的方法，如果仅参数类型不同，那么要重载。重载会有很多冗余的代码。在. NET1.0 时代也可以不用重载，那就是参数类型直接用 Object 类型，那么任何类型都能传进去了，但是会有装箱拆箱操作，影响性能。

``` csharp
public static void Show(string sValue)
{
    Console.WriteLine(sValue);
}

public static void Show(int iValue)
{
    Console.WriteLine(iValue);
}

public static void Show(object oValue)
{
    Console.WriteLine(oValue);
}
```

### 二、泛型的好处

值类型和引用类型的装箱拆箱消耗。**`值类型分配在线程栈上，引用类型分配在堆上，只把指针放在栈上`**。如图所示，如果把 int 类型 1 装箱，就要把 1 拷贝到堆中，就会有内存的交换。以前的 ArrayList 就是类型不安全的，需要频繁的进行装拆箱操作，Add 元素的时候全部装箱 object，取的时候要拆箱，性能损失比较大。

![](https://user-gold-cdn.xitu.io/2019/10/26/16e07fbf6348d84e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

泛型的效率等同于硬编码的方式，就是和你很多功能相同的类效率差不多。泛型每个类型只实例化一次，下面泛型缓存会详细解读下。先简单介绍下 CLR 的运行原理（详细在 CLR 章节）以了解泛型的原理机制。

![](https://user-gold-cdn.xitu.io/2019/10/26/16e07fbf63ef2496?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

.NET 编译器和解释器两阶段，我们先经过编译器编译成 IL 中间语言（dll、exe），和 java 的字节码类似，然后经过 JIT 解释成机器码。这样做的好处就是我们只需要编译成 IL 后，在各个不同计算机系统上，只要有对应的 CLR（JIT）就行，这样就和平台无关。**`二次编译：为了一次编译，不同平台使用`**。泛型在第一个编译时会用一个占位符代替，在第二次运行时会编译成具体的类型。所以性能相当于硬编码的方式，每种类型最终都有自己的机器码。

List<T> 是在使用时定义类型，JIT 编译器解析时动态的生成，如定义 List<int>，在 JIT 运行时就声称 List<int > 类型，然后操作就不会出现装箱拆箱，而且只能添加指定的类型，这就**`类型安全`**。

### 三、泛型使用

#### 1、泛型方法

常见的泛型方法就是在方法后面带上**`<T>(T param)，“T”可以随便定义，只要不是关键保留字就行`**，默认约定俗成都用 T，此处就代表你定义了一个 T 类，然后后面参数就可以用这个 T 类型。（如果把鼠标光标放在参数类型 T 上，然后 F12 转到定义就会定位到前面这个 T。）**`这样就可以用一个方法，满足不同的参数类型，去做相同的事情`**。把参数的类型申明推迟到调用时，延迟声明。后面框架中也会有很多这种**`延迟思想`**，延迟以达到更好的扩展。

```csharp
public static void Show<T>(T tValue)
{
    Console.WriteLine(tValue);
}
CommonMethod.Show<int>(123);
```

#### 2、泛型类、泛型接口

创建方法类似，语法一样 <T>。用的最多的 List<T > 就是很典型的泛型类，**`用来满足不同的具体类型，完成相同的事情`**。

```csharp
public class GenericClass<T>
{
    public T _T;
}

public interface IGenericInterface<T>
{
    T GetT();
}
```

### 四、泛型的功能

#### 1、泛型中的默认值

既然用了泛型，那么在内部想要初始化怎么办呢？因为泛型进来的类型不一定是值类型或引用类型，所以初始化就不能简单直接赋 null。这个时候需要用到**`default`**关键字，用于将泛型类型初始化为 null 或其他值类型默认值（0，0001/1/1 0:00:00 日期等）；

#### 2、约束

泛型导致任何类型都可以进来，那么如何去使用这个类型 T，编写的时候我们是不知道 T 是什么，也不知道它能干什么。一个方法就是可以用反射，任何一个类型通过发射都能获取内部的结构属性方法调用。泛型约束提供更简便的方法。在声明泛型时在参数后面追加**`where`**关键字。约束可以同时指定多个，像这样**`where:T People,IWork,new()`**。同时约束传进来的类型 People 或其子类，并且继承了 IWork 接口，有无参数构造函数。

![](https://user-gold-cdn.xitu.io/2019/10/26/16e07fbf64385841?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

``` csharp
public static void Show<T>(T tValue) where T : People
{
    Console.WriteLine(tValue.Name);
}
```

#### 3、协变逆变

协变逆变就是对参数和返回值的类型进行转换。协变用一个派生更大的类去代替某个类型（小代替大），其实就是设计原则的里氏替换原则，比如狗继承自动物，那么任何用动物作为参数类型的地方，调用时都可以用狗代替。逆变就是反过来。

``` csharp
//协变
public void ShowName(Animal animal)
{
}

ShowName(dog);
```

泛型接口的协变逆变。如果泛型类型用了 out 关键字标注，泛型接口就是协变的。这也意味着返回类型只能是 T。如果用了 in 关键字标注，就是逆变，只能把泛型类型 T 用作方法的输入。这块很绕，实际使用非常少。

``` csharp
//一堆狗肯定是一堆动物啊，为啥就不能这么做呢？下面这句编译不通过
//前后两个类型是没有父子关系的
List<Animal> animalLst = new List<Dog>();
//下面这句就可以呢？
IEnumerable<Animal> animalLst2 = new List<Dog>();

//因为在接口中添加了out关键字
public interface IEnumerable<out T> : IEnumerable
{
    //
    // 摘要:
    //     Returns an enumerator that iterates through the collection.
    //
    // 返回结果:
    //     An enumerator that can be used to iterate through the collection.
    IEnumerator<T> GetEnumerator();
}

ICustomListIn<Dog> customLstIn = new CustomListIn<Animal>();

public interface ICustomListIn<in T>
{
    void Show(T t);
}

public class CustomListIn<T> : ICustomListIn<T>
{
    public void Show(T t)
    {
        Console.WriteLine(typeof(T).FullName);
    }
}

interface ISetData<in T>  //使用逆变
{
    void SetData(T data);
}

interface IGetData<out T>   //使用协变
{
    T GetData();
}

class MyTest<T> : ISetData<T>, IGetData<T>//继承两个泛型接口
{
    private T data;
    public void SetData(T data)
    {
        this.data = data;   //赋值
    }
    public T GetData()
    {
        return this.data;   //取数据
    }
}

MyTest<object> my = new MyTest<object>();
ISetData<string> set = my;
set.SetData("nihao");
```

其实协变逆变就是语法糖，为了让不是继承关系的类型也可以互相赋值编译通过。运行时实际右边是什么类型就是什么类型。（欺骗编译器，自己应该会很少写协变逆变的接口或委托）

#### 5、泛型委托

以 Action 为例，Action 是. NET Framework 内置的泛型委托，可以使用 Action 委托以参数形式传递方法，而不用显示声明自定义的委托。其实我们撸代码过程中不太需要自己定义委托，**`内置的Action和Func就够用，也便于统一`**。Action 无返回值委托，可以有 16 个参数，可以传入不同的类型。在委托事件一章会详细介绍。

![](https://user-gold-cdn.xitu.io/2019/10/26/16e07fbf64784c6f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 4、泛型缓存

**`泛型类的静态成员只能在类的一个实例中共享`**。运行时泛型类的实例已经指定了具体类型，每一个不同的泛型类实例共享静态成员，利用这个特点就可以做缓存。**`每一个不同的T缓存一个版本数据`**。如例子所示，当第一次指定不同的 T 时，会重新构造，再次有相同的类型时，就不会进入静态构造函数了。相当于为缓存了多个版本的静态成员。比如在各个数据库实体类需要有一些增删改查的 SQL 时，就可以利用用泛型特性，每一个数据库实体类都会缓存一份自己的增删改查 SQL。

``` csharp
public class GenericCache<T>
{
    static GenericCache()
    {
        Console.WriteLine("进入静态构造函数");
        _TypeTime = $"{typeof(T).FullName}_{DateTime.Now.ToString()}";
    }

    private static string _TypeTime = "";

    public static string GetCache()
    {
        return _TypeTime;
    }
}

Console.WriteLine("************************");
Console.WriteLine(GenericCache<int>.GetCache());
Thread.Sleep(1000);
Console.WriteLine(GenericCache<string>.GetCache());
Thread.Sleep(1000);
Console.WriteLine("认真比较打印出的静态成员值");
Console.WriteLine(GenericCache<int>.GetCache());
Thread.Sleep(1000);
Console.WriteLine(GenericCache<string>.GetCache());
Console.WriteLine("************************");

```

### 五、总结

通过泛型类可以创建独立于类型的类，泛型方法创建出独立于类型的方法。接口、结构、委托也可以用泛型的方式创建。**`建议如果我们需要设计和类型无关的对象时，可以使用泛型，把锅甩给调用方，由上端决定实例化具体什么类型。`**
