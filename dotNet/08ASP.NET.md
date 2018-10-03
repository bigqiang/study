## 8 ASP.NET

本章介绍 ASP.MVC，Web Form 不做介绍。

---
### MVC模式
MVC模式（Model-View-Controller）大约在20世纪70年代就有了，起初创造出来是用于Smalltalk。现在该模式又复活了，广泛用于种语言。对.NET开发者而言，该模式实现的框架被称作 ASP.NET MVC，第一次在2007年引入。

#### Model
模型是应用程序的数据。该数据经常表示成 plain old CLR objects (POCOs)， , as used in the data access library
created in the previous two chapters 。模型是由一个或多个模型组成，根据要使用他们们的视图特别定制。对于模型和视图就象数据库表和数据库视图。
学术上，模型应该非常干净，不含验证或任何其他业务规则。从实际上看，模型是否包含验证逻辑或其他业务规则，完全取决于所用的语言和框架，以及特定应用的需求。例如，EF Core 容纳了许多数据注释，作为定制数据库表的机制和验证的手段，这些注释在Core MVC中起到了双重作用。

#### View
视图是应用程序的UI。视图接受命令，并把这些命令的执行结果渲染给用户。视图应尽可能的轻量，实际上并不处理工作。相反，他把所有工作都转给控制器。

#### Controller
控制器是操作的中枢。控制器负责两件事：一是接受用户的命令/请求（referred to as actions），然后正确适当地调度他们；二是把任何变化传给视图。
控制器（与模型和视图一样）应该轻量，能利用其他组件维护分散的关系。

#### 为什么用MVC？
ASP.NET MVC 发布于2007年，在Web Form已投产6年了。

#### 进入 MVC

约定胜于配置（Convention over Configuration）
这意味着明确的约定（如命名约定和目录结构）可头破血流应用大量的配置。

---

### ASP.NET MVC 应用程序模板
理论太多，该写代码了。用VS构建一个MVC应用 CarLotMVC

#### 项目根目录文件

File | Meaning in Life | Deployed?
-|-|-
favicon.ico | 在浏览器地址栏紧挨页面名称显示的图标。无此文件会导致性能问题，浏览器会不断寻找该文件。 | Yes
Global.asax / Global.asax.cs | 应用程序的入口位置 | Yes
packages.config | 当前项目中NuGet包使用的配置信息。 | No
ApplicationInsights.config | 用于配置 application insights (本书未包含)。 | Yes
Startup.cs | Startup 类，用于 Open Web Interface for .NET (OWIN)，在 ASP.NET Identity 中使用 | Yes (Compiled)
Web.config | 项目配置文件 | Yes


#### Global.asax.cs
该文件钩在ASP.NET管线的位置。默认项目模板仅使用 Application_Start 事件处理方法，如果愿意也会用到更多的事件。下表为Gblobal.asax.cs文件的常用事件：

Event | Meaning in Life
-|-
Application_Start | 应用程序的第一次请求触发
Application_End | 应用程序结束时触发
Application_Error | 聘一个未处理错误时触发
Session_Start | 一个会话的第一次请求触发
Session_End | 一个会结束（或超时）时触发
Application_BeginRequest | 在对服务器做请求时触发
Application_EndRequest | 在ASP.NET响应请求时，在执行HTTP管线链中最后一个事件时触发

#### Models 文件夹
存放模型类的位置。在大型应用程序中，应该使用一个数据访问工具库保存维护数据访问模型。该文件常用于特定视图的模型。

#### Controllers 文件夹
控制器文件所在位置。

#### Views 文件夹
视图存放位置。该文件夹下目录结构存在一个约定：每个控制器在 Views 文件夹下都有自己的文件夹。控制器在该文件夹下可以发现与控制器同名（去除Controller）文件夹。
在 Views 的根路径下存在 Web.confg 和 _ViewStart.cshtml 文件。前者用于明确该文件夹层级中的视图，并且定义基础页面类型（如 System.Web.Mvc.WebViewPage）, 在基于Razor的项目中，加入所有用于Razor的引用和using语句。后者在任何视图渲染给用户前先执行。该布局视图与Web Form的母版页类似。


##### Shared 文件夹
Views下的一个文件夹。可被所有视图访问。该文件夹包含两个文件：_Layout.cshtml、Error.cshtml。前者是 _ViewStart.cshtml 中的默认布局。后者是该应用的默认错误模板。

> 注：为什么_ViewStart.html(还有_Layout.cshtml)文件有一个前导下划线？Razor视图引擎最初是为 WebMatrix 创建的，它允许任意不用前缀下划线的文件去渲染，因此对于核心文件（象布局和配置）的名子都要以下划线开始。该命名约定也用于 partial views 。不过这并非是MVC在意的约定，因为 MVC没有与 WebMatrix相同的问题，不过下划线的规则却遗留下来。

#### ASP.NET 保留的文件夹

文件夹 | Meaning in Life
-|-
App_Code | 容纳动态编译的代码文件
App_GlobalResources | 存有用于整个应用程序的资源文件，常用于本地化。
App_LocalResources | 包含针对特定页面的资源。常用于本地化。
App_Data | 包含有应用程序使用的基于文件的数据
App_Browsers | 存放浏览器兼容性文件的位置
App_Themes | 保存该站点的主题

#### App_Start 文件夹
在Global.asax.cs类文件中，MVC 早期版本包含所有站点配置代码（如路由、安全）。随着配置增多，MVC项目组的开发者明智地把代码拆分成几个类。App_Start文件夹中的所有代码，都会自动编译到站点。

App_Start 文件夹下的文件

文件 | Meaning in Life
-|-
BundleConfig.cs | Creates the file bundles for JavaScript and CSS files. Additional bundles can (and should) be created in this class.
FilterConfig.cs | 注册 action 过滤器 (如认证或授权) 在全局域
IdentityConfig.cs | 容纳用于 ASP.NET Identity 的支持类.
RouteConfig.cs | 配置路由表位置的类
Startup.Auth.cs | ASP.NET Identity 配置的入口

P1708



































