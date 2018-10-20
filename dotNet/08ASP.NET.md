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

App_Start 文件夹下的文件：

文件 | Meaning in Life
-|-
BundleConfig.cs | Creates the file bundles for JavaScript and CSS files. Additional bundles can (and should) be created in this class.
FilterConfig.cs | 注册 action 过滤器 (如认证或授权) 在全局域
IdentityConfig.cs | 容纳用于 ASP.NET Identity 的支持类.
RouteConfig.cs | 配置路由表位置的类
Startup.Auth.cs | ASP.NET Identity 配置的入口

##### BundleConfig 文件
该类文件用于对CSS及JavaScript文件设置打包和压缩。默认情况在使用 `ScriptBundle` 时，这些文件会在生产环境被打包压缩，而debug环境不处理。这是通过 Web.config 或该类文件自身进行控制。要关闭打包压缩功能，可到最上层 Web.config 文件的 system.web 部分，改如下内容：
```XML
<system.web>
	<compilation debug="true" targetFramework="4.7" />
</system.web>
```
或者在 `BundleConfig.cs`文件中的 `RegisterBundles` 方法里添加 `BundleTable.EnableOptimizations = false`：
```C#
public static void RegisterBundles(BundleCollection bundles)
{
	bundles.Add(new ScriptBundle("~/bundles/jquery").Include("~/Scripts/jquery-{version}.js"));
	// Code removed for clarity.
	BundleTable.EnableOptimizations = false;
}
```

###### Bundling
打包是把多个文件合并成一个文件的过程。这样做的主要原因可以加速网站访问。浏览器从单一服务器下载多个文件时存在并发数量限制。如果站点页面存在大量小文件，缓慢的加载速度会降低用户体验。可能通过多个内容分发网络（CDN）加速文件下载。另一方法就对文件打包。


###### Minification
和打包一样，压缩也是为了减少网页加载时间。多数现代框架交付两个版本的 CSS和JavaScript文件。比如Bootstrap，bootstrap.css用于开发，bootstrap.min.css用于生产。


##### FilterConfig 文件
过滤器是自定义的类文件，提供拦截操作和请求的机制。他们在操作/控制器实施拦截，或者全局拦截。MVC有四种类型过滤器，如表格所示：

Filter | Type Meaning in Life
-|-
Authorization | 实现 `IAuthorizationFilter` ，并且在任意其他过滤器前运行。两个例子是`Authorize`和`AllowAnonymous`。例如，`AccountController`类中用`[Authorize]`属性注释，要认证的用户通过 Identity 验证，并且 `Login`操作用`[AllowAnonymous]`属性标记以许可任意用户。
Action | 实现 `IActionFilter`，允许用`OnActionExecuting`及`OnActionExecuted`拦截action的执行。
Result | 实现 `IResultFilter`，用 `OnResultExecuting`及`OnResultExecuted`拦截action的结果。
Exception | 实现 `IExceptionFilter`，如果ASP.NET管线执行期间抛出一个未处理的异常时执行。默认情况下`HandleError`过滤器被配置成全局生效。该过滤器显示的错误视图页面位于 `Shared\\Error\\Error.cshtml`。


##### IdentityConfig 文件
`Identity.config.cs`和`Startup.Auth.cs`两个文件都用于ASP.NET Identity。

##### RouteConfig 文件
以前URL定义是根据项目物理文件夹结构，而这已经被 `HttpModules` 和 `HttpHandlers`改变。该类文件允许你定义URL以更好适合用户需求。

#### Content 文件夹
该文件夹用于保存站点的CSS文件、图片以及其他直接被渲染的内容。默认的文件 Site.css 容纳站点初始的样式。

##### Bootstrap
ASP.NET MVC 自带Bootstrap。

##### Fonts 文件夹
Bootstrap要求它的字体文件要放在该文件夹

##### Scripts 文件夹


#### NuGet 包的更新
有很多文件和包是由核心的MVC项目模板构成，还有很多是来自开源框架。开源项目更新速度比微软在VS模板中发布更新的速度要快。刚创建好项目可能包就过期了。
还好，用 NuGet GUI 来更新这些包就简单了。右击项目，选择“管理NuGet程序包”，根据提示更新即可。

---

### Routing
路由是 MVC 将应用中的URL请求与控制器和操作匹配的方式。MVC用链接的模式来决定控制器与操作方法的执行。

#### 链接的模式
路由项是由URL模式构成，而URL模式包含了占位符变量以及字面量，依序占位，被称作路由表。每一项都定义了不同匹配的URL模式。
占位符可能是自定义变量或来自一个预定义列表。打开 RouteConfig.cs 文件看看：
```C#
public class RouteConfig
{
	public static void RegisterRoutes(RouteCollection routes)
	{
		routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
		routes.MapRoute(
			name: "Default",
			url: "{controller}/{action}/{id}",
			defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
		);
	}
}
```
第一行告诉路由引擎忽略 `.axd` 扩展名的请求，该扩展名文件表示一个 `HttpHandler`。这里的`IgnoreRoute()`方法把请求返回给web server，这里是IIS。 模式`{*pathinfo}`处理参数中的可变数，可以把这个匹配模式用于所有URL，包括 `HttpHandler`。
`MapRoute()` 方法在路由表中添加了一个新项，它指明了名称、URL模式以及URL模式中涉及变量的默认值。`{controller}`和`{action}`分别指向控制器和一个操作方法。而当前占位符`{id}`是自定义的，会转换成一个名为id的参数供action方法使用。
当URL请求时，先检查路由表。如果匹配，则相应代码执行。比如路由`Inventory/Delete/5`，会在控制器`InventoryController`中调用`Delete()`方法，并传值5给`id`参数。
要牢记，路由处理是串行按序进行。处理会在第一次匹配成功即停止；至于路由表后面是否有一个更好的匹配根本不重要。
要注意，路由不包含服务器（域名）地址。

#### 给Contact和About页创建路由
打开RouteConfig.cs ，在IgnoreRoutes调用之后，默认路由调用之前插入以下代码：
```C#
routes.MapRoute("Contact", "Contact", new {controller = "Home", action = "Contact" });
```
它仅包含一个字面量值`Contact`，映射到 `Home/Contact`。浏览器运行该URL访问可生效，不过URL变成`Home/Contact/Foo`执行结果相同。
如果改成:
```C#
routes.MapRoute("Contact", "Contact/{*pathinfo}", new {controller = "Home", action = "Contact" });
```
`{*pathinfo}`模式表示允许任意数量的额外URL参数。

#### 路由重定向
路由项可双向使用，即可以匹配输入的请求，也可以构建本站的链接。例如，在`View/Shared`文件夹的 `_Layout.cshtml` 文件中的一行：
```
@Html.ActionLink("Contact", "Contact", "Home")
```
HTML助手工具`ActionLink()`创建了一个超链接，并指明该链接的文本内容是`Contact`，操作方法是 `Home`控制器的`Contact`。它会生成以下链接：
```html
<a href="/Contact">Contact</a>
```
如果没有添加 Contact 路由，路由引擎会创建如下链接：
```html
<a href="/Home/Contact">Contact</a>
```

> 注：本内容涉及了新的知识点，如 “@”、“Html”对象及“_Layout.cshtml”文件。

---

### 控制器和操作

如前所述，浏览器的每次请求都会映射到一个指定控制器类的操作方法中。控制器是继承自两个抽象类之一：`Controller`和`AsyncController`。注意，也可通过实现`IController`来创建一个控制器。action方法在控制器类中是一个public方法。

#### Action Results
Action 常返回一个 `ActionResult`（或 `Task<ActionResult>`用于异步操作）。`ActionResult`派生出多种类型，下面列表列出几种常用类型：

Action Result | Meaning in Life
-|-
ViewResult PartialViewResult | 返回一个网页视图（或 partial view）
RedirectResult RedirectToRouteResult | 重定向另一个 action
JsonResult | 返回一个序列化的JSON对象给客户端
FileResult | 返回一个二进制文件内容给客户端
ContentResult | 返回一个用户定义的内容类型给客户端
HttpStatusCodeResult | 返回一个指定的HTTP状态码

#### 避免二次提交(Double Posts)
用户点击提交按钮容易出现提交表单两次的问题。MVC采用了一种极大缓解该问题的模式。在说该模式之前，先学习两个在MVC应用中的HTTP谓词:HttpGet 和 HttpPost。

在 Web Form中，这两个差别不大。HTTP协议定义，HttpGet是向服务器请求数据，HttpPost是向一个特定源提交数据去处理。
MVC中，任何不带 HTTP属性（比如HttpPost属性）值的action都视作 HttpGet操作。要指定HttpPost操作，必须对 action 用 `[HttpPost]`属性来修饰。

> 注意：MVC不用 HTTP DELETE 或 PUT 谓词，只使用 GET和POST，与浏览器功能保持一致。ASP.NET Web API 使用所有四个谓词。

Post-Redirect-Get (PRG) 模式可以阻止重复提交。它重定向每个成功 Post 的action到一个Get的action。因此，用户再点击提交按钮时，它仅刷新Get操作的页面，不会再次提交原先的Post操作。


#### 模型绑定（Model Binding）
模型绑定会从HTML表单（包括表单和查询字串等）获取所有的名值对，然后通过从名值对匹配属性名并赋值的方式尝试重建特定类型。如果一或多个值没被赋值（例如数据类型转换问题或验证错误），模型绑定引擎会设置`ModelState.IsValid`属性值为`false`。如果匹配就是`true`。
除`IsValid`属性外，`ModelState`是一个`ModelStateDictionary`，包含了每个失败属性的错误信息，与模型级别的错误信息相同。如果想新增一个特定属性错误，可编码；
```C#
ModelState.AddModelError("Name","Name is required");
```
想为整个模型新增错误，要用`string.Empty`作属性名：
```C#
ModelState.AddModelError(string.Empty, $"Unable to create record: {ex.Message}");
```

#### 防篡改token(AntiForgery Token)
`AntiForgeryToken`属性是一种特殊的表单值。该表单值用于验证MVC模式下服务器端`HttpPost`请求，只要 atcion 方法有 `[ValidateAntiForgeryToken]` 属性存在。任何表单都应该加一个`AntiForgeryToken`，并且任何 `HttpPost`的action都应验证它。

#### Bind 属性
`HttpPost`方法中的 `Bind`属性允许你设置黑白名单以阻止提交过量（over-posting）攻击。白名单中的字段会通过模型绑定赋值，黑名单的字段则不会。

---

### Razor视图引擎
Razor视图并没有扩展`System.Web.Page`（没有使用`Page`指令），只新增了可测试性。
Razor是服务器端把模板标记解析成C#的语法。Razor视图引擎也支持Web Form视图引擎支持的一切，因此使用Razor不用放弃什么。
Razor也支持方法创建，以用于代码封装表单中的HTML助手方法、Razor函数及Razor委托。视图中创建的这些文件位于 App_Code 文件夹，创建他们要用 `static`关键词

> 注：Web Form 视图引擎在MVC5中仍然支持。

#### Razor语法
Razor引擎中添加代码用“@”符号，不需要添加关闭“@”的符号，不象Web Form，一定要有开（<%）有闭（%>）。

##### 语句，代码块，以及标记
单行语句用“@”符号做前缀，不用关闭符号：
```
@ShowInventory(item)
```

对于语句块以“@”打开，以花括号封闭：
```
@foreach (var item in Model)
{
}
//or
@{
ShowInventory(item);
}
```

代码块可以混合标记与代码。以标记标签开始的行会被解析成HTML，以代码开始的行被解析成代码：
```
@foreach (var item in Model)
{
	int x = 0;
	<tr></tr>
}
```
这个“@”符号等效于`Response.Write()`，并且默认情况下HTML会编码所有值。如果想输出未编码的数据（例如，可能存在unsafe的数据），就必须使用 `@Html.Raw(username)` 语法。Razor和标记可以一起使用：
```
<h1>Hello, @username</h1>
```

“@:”和“<text>”标签表示应该渲染成标记一部分的文本，例如：
```
@:Straight Text
<div>Value:@i</div>
<text>
Lines without HTML tag
</text>
```

“@*”和“*@”表示一个多行注释：
```
@* This is a comment that could go across multiple
lines *@
```

Razor也是相当智能的，碰到email地址，它不会解析成代码：
```
someone@foo.com @* email address *@
someone@(foo).com @* foo is interpreted as Razor
code and the variable foo is replaced by its value*@
```

如果需要对“@”转义，可以写两个“@”，如下：
```
@@foo
```

#### 内置的HTML助手工具方法(HTML helper)
一些常用的内置HTML助手工具方法：

HTML | Helper Meaning in Life
-|-
ActionLink | 创建一个锚标记
TextBox[For] TextArea[For] CheckBox[For] RadioButton[For] DropDownList[For] ListBox[For] Hidden[For] Password[For] Label[For] | 每个都创建一个同名HTML表单控件用于模型的属性。`For`版本是强类型
Display[For] | 根据模板把属性渲染成只读。如果没找到自定义的模板，就使用默认模板（根据数据类型）。 `For`版本是强类型
Editor[For] | 根据模板把属性渲染成一个表单控件。如果没有自定义模板，则使用默认模板（根据数据类型）。`For`版本是强类型
DisplayForModel EditotForModel | 为整个模型渲染显示或编辑器标记。如果存在自定义模板则使用，否则使用默认模板。
DisplayName[For] | 如果存在赋值，则获取显示名称，否则显示属性名。`For`版本是强类型
ValidationSummary ValidationMessageFor | 在客户端或服务端验证失败时创建一个验证信息总结。
@Html.BeginForm() | 创建一个HTML表单标签，包含了在封闭在花括号内所有东西
@Html.AntiForgeryToken() |  与`ValidateAntiForgeryToken` action 方法属性协同

##### 编辑模板的HTML助手工具方法
编辑器助手工具比创建其他相关控件做的工作多些。例如，`EditorFor()`HTML助手工具基于lambda表达式引用属性的数据类型创建了一个输入字段。例如：
```
@Html.EditorFor(model => model.Make, new { htmlAttributes = new { @class = "form-control" } })
```
会生成：
```html
<input name="Make" class="form-control text-box
single-line" id="Make" type="text" value="" data-
val-length-max="50" data-val-length="The field Make
must be a string with a maximum length of 50." data-
val="true">
```
HTML元素中的`name`和`id`来源于属性的`name`，`type`来源于属性的数据类型，`class`值来自于HTML助手工具和其他HTML助手工具添加属性的组合。

##### 调整由HTML助手工具渲染的HTML
HTML助手工具会自己渲染标记，如果想添加样式或其他属性，就需要多些额外操作。先要创建一个新的带HTML名值对的匿名对象。如果HTML属性名是C#的保留字，就必须用“@”符号转义。例如：
```
@Html.LabelFor(model => model.Color,
htmlAttributes: new { @class = "control-label col-md-2" })
```
这会把CSS类属性更新到label标记中。

#### 自定义的HTML助手工具
除内置的外，还可以创建自定义的HTML助手工具减少代码量。

##### Razor函数
Razor函数不返回标记，而是用于封装重用代码。


##### Razor 委托
它工作方式与C#委托类似。

---

### MVC 视图
MVC中的视图代表MVC站点的UI。它非常轻量，把服务器端处理传给控制器，客户端处理传给JavaScript。

#### 布局
在`Views/Shared`文件夹下有文件`_Layout.cshtml`，这是个全功能的HTML文件，有完整的`<head>`和`<body>`标签，混合了HTML标记和Razor的助手工具标记。有两个关键项需要牢记：body和section。body是视图和布局在渲染时要插入的地方，它通过下面的Razor代码来定位：
```
@RenderBody()
```
section是布局页面的区域，它在运行时内填充。他们可以必选或可选的，通过`RenderSection()`引入布局页面。下面代码创建了一个名`scripts`的section:
```
@RenderSection("scripts", required: false)
```

如果想创建一个名为`Header`的必填新section，就这样编码：
```
@RenderSection("Header",required: true)
```

为了渲染视图中的section，要使用 `@section` 的 Razor语句块。例如，下面的代码行会向渲染页中加入jQuery验证包：
```
@section Scripts {
	@Scripts.Render("~/bundles/jqueryval")
}
```

##### 布局页选择
MVC的视图是基于母版布局（master layout）给整个站点一个统一的界面外观。视图可以显式声明要被渲染的布局视图，或者使用默认的布局视图。

##### 使用指定布局页
除了使用默认布局页，还可对特定页专门指定视图，可通过在视图文件顶部加入一个`Layout`行。例如：
```
@{
	ViewBag.Title = "Index";
	Layout = "∼/Views/Shared/_LayoutNew.cshtml";
}
```

#### 局部视图(Partial View)
局部视图与普通视图一样，除了它不用作布局视图，它用于封装UI.局部视图与全视图的区别在于渲染的方式。全视图（由一个控制器的 `View()`方法返回）使用布局页，而局部视图由 `PartialView()`方法（或`Partial() HTML Helper` ）渲染，不用默认布局，而是使用用`Layout`的Razor语句指定的布局。示例：
```
public ActionResult Index()
{
	return PartialView(_repo.GetAll());
}
```

#### 向视图传参
action方法可以向视图返回数据。通过传递一个对象（或对象列表）给`View()`方法即可完成。传参给`View()`方法的数据也是该视图的模型。

##### 强类型视图和视图模型
MVC的视图常常使用的是强类型的数据。预期的数据类型在视图中用 `@model` 语句定义，例如：
```
@model IEnumerable<AutoLotDAL.Models.Inventory>
```
要在视图其他地方访问该数据，要使用`Model`关键词。注意`Model`中的M要大写，而`@model`行中的m要小写。示例：
```
@foreach (var item in Model)
{
	//Do something interesting here
}
```

##### ViewBag, ViewData, TempData
这三个对象是给视图传递微量数据的机制。示例：
```
@{
	ViewBag.Title = "Index";
}
```
`ViewBat.Title` 属性用于发送视图的title给布局页：
```
<title>@ViewBag.Title - My ASP.NET Application</title>
```

发送数据给视图的方式

数据传输对象 | Meaning in Life
-|-
TempData | 仅生存于当前请求和下次请求之间的短生存期的对象
ViewData | 存储名值对的字典类型数据。示例:`ViewData["Title"] = "Foo"`
ViewBag | 对`ViewData`字典对象的协态包装。示例：`ViewBag.Title = "Foo"`

现在应该明白数据如何传递到视图的方式了。

---

### Display数据注释
很多Entity Framework中用到的数据注释也用于MVC的视图引擎中渲染标记和数据校验。有些额外数所注释EF不使用，而MVC会用到，比如 `Display`数据注释。


P1744


### ASP.NET Web API





























