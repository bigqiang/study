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









































