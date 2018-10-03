## 3 C#面向对象编程
本章通过演示很多短小独立的主题对C#进行正常剖析，这些主题力求难度适宜。要学的第一件事是构建应用对象的方法，以及检查可执行程序入口点“Main()”的结构。其次剖析包含在System.String和System.Text.StringBuilder类中 C# 的基本数据类型（以及在System命名空间中的等效类型）。
了解这些基础.NET数据类型后，就可以研究许多数据类型转换的技巧，包括缩小操作(narrowing operation)、放大操作(widening operation)和 `checked` 和 `unchecked` 关键词用法。
本章也学习 C# 中`var`关键词的作用，它允许隐式地定义一个局部变量。本书后面可以见到，隐式类型非常有用。



### 1 理解封装
要从学习C#面向对象的能力开始。首先学习构建定义完整的类类型的过程，要支持构造器。理解类定义及对象分配的基础知识后，本章最后会剖析封装的作用。
在这过程中，会学习定义类属性的方法，以及理解`static`关键词的详细用法，还有对象初始化语法、只读字段、常量数据以及partial类。

#### C#类的类型介绍
类是用户定义的类型，由字段数据（常称作变量），以及操作这些数据的成员（构造器、属性、方法、事件等）。这些字段数据表示类实例（也称作对象）的状态。象C#这样面向对象语言的威力在于，对数据和相关的功能分组成一个统一的类定义，让软件能模拟真实世界的对应实体。
为了让球滚动，创建一个新的C#控制台应用程序项目(C# Console Application project)，命名 SimpleClassExample。 
然后，插入一个新类文件(Car.cs)到该项目中，方法使用 项目->添加类 菜单，从结果对话框中选择 类 图标，点击“添加”按钮。
接着在该类文件中定义两个数据类型分别表示车的昵称及当前速度：
```C#
class Car
{
	// The 'state' of the Car.
	public string petName;
	public int currSpeed;
}
```
>注：类中的字段数据几乎很少定义成 public。为保护数据状态的完整性，定义成 private 更优。

再定义方法 SpeedUp() 和 PrintState() :
```C#
class Car
{
	// The 'state' of the Car.
	public string petName;
	public int currSpeed;
	// The functionality of the Car.
	// Using the expression-bodied member syntax introduced in C# 6
	public void PrintState()
		=> Console.WriteLine("{0} is going {1} MPH.",
	petName, currSpeed);
	public void SpeedUp(int delta)
		=> currSpeed += delta;
}
```
PrintState() 是个诊断函数，只是把Car对象当前状态输出到命令窗口。 SpeedUp() 用于增加Cars对象的速度。现在更新 Main() 方法：
```C#
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Class Types*****\n");

	// Allocate and configure a Car object.
	Car myCar = new Car();
	myCar.petName = "Henry";
	myCar.currSpeed = 10;
	// Speed up the car a few times and print out the
	// new state.
	for (int i = 0; i <= 10; i++)
	{
		myCar.SpeedUp(5);
		myCar.PrintState();
	}
	Console.ReadLine();
}
```
运行程序后，显示如下结果：
```
***** Fun with Class Types *****
Henry is going 15 MPH.
Henry is going 20 MPH.
Henry is going 25 MPH.
Henry is going 30 MPH.
Henry is going 35 MPH.
Henry is going 40 MPH.
Henry is going 45 MPH.
Henry is going 50 MPH.
Henry is going 55 MPH.
Henry is going 60 MPH.
Henry is going 65 MPH.
```

#### 用 new 关键词分配对象
如前代码所示，对象必须使用 new 关键词分配到内存。如果不使用它，想在后续代码声明中使用该类变量，会出现编译错误。错误示例：
```C#
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Class Types*****\n");
	// Compiler error! Forgot to use 'new' to create object!
	Car myCar;
	myCar.petName = "Fred";
}
```
正确示例：
```C#
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Class Types*****\n");
	Car myCar = new Car();
	myCar.petName = "Fred";
}
```
如果想在想在独立一行代码上定义声明类实例也可这样做：
```C#
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Class Types*****\n");
	Car myCar;	//先声明
	myCar = new Car();	//再指派指向内存有效对象的引用
	myCar.petName = "Fred";
}
```

#### 构造函数
它可以在对象创建之初进行初始化。它是类的特殊方法，在用 new 创建对象时被间接调用。因此不同于“常规”方法，构造函数不返回值（甚至不是void）。

##### 默认构造函数的作用
任何C#类都提供了一个默认构造方法。根据定义，默认构造方法没有任何参数。在指派对象到内存后，默认构造方法要确保所有字段数据都设置了合适的默认值。
对默认的构造方法不满意，可以重定义它以符合自己的需求。举例说明：
```C#
class Car
{
	// The 'state' of the Car.
	public string petName;
	public int currSpeed;
	// A custom default constructor.
	public Car()
	{
		petName = "Chuck";
		currSpeed = 10;
	}
	...
}
```
本例中强制所有 Car 对象的昵称为 Chuck，并且速度为 10mph。示例：
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Class Types*****\n");
	// Invoking the default constructor.
	Car chuck = new Car();
	// Prints "Chuck is going 10 MPH."
	chuck.PrintState();
	...
}

##### 自定义的构造函数
通常，类会定义额外的构造方法以超越默认的限制。以下提供三种构造方法：
```C#
class Car
{
	// The 'state' of the Car.
	public string petName;
	public int currSpeed;
	// A custom default constructor.
	public Car()
	{
		petName = "Chuck";
		currSpeed = 10;
	}

	// Here, currSpeed will receive the
	// default value of an int (zero).
	public Car(string pn)
	{
		petName = pn;
	}

	// Let caller set the full state of the Car.
	public Car(string pn, int cs)
	{
		petName = pn;
		currSpeed = cs;
	}
	...
}
```
记住，一个构造方法区别（C#编译器角度看）于其他构造方法的要点在于构造方法参数的数量及类型。示例：
```C#
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Class Types*****\n");
	// Make a Car called Chuck going 10 MPH.
	Car chuck = new Car();
	chuck.PrintState();
	// Make a Car called Mary going 0 MPH.
	Car mary = new Car("Mary");
	mary.PrintState();
	// Make a Car called Daisy going 75 MPH.
	Car daisy = new Car("Daisy", 75);
	daisy.PrintState();
	...
}
```
可以使用 Expression-Bodied Members（新）风格，前面的构造方法可以写成如下形式：
```C#
// Here, currSpeed will receive the
// default value of an int (zero).
public Car(string pn) => petName = pn;
```
由于 表达式主体成员（Expression-Bodied Members）方式的设计用于单行代码的方法，所以第二个构造方法并非是个有效的备选方案。

#### 默认构造函数深入
所有类都提供一个默认的构造方法。因此再引入一个新类到当前项目中，命名为 Motorcycle，如下：
```C#
class Motorcycle
{
	public void PopAWheely()
	{
		Console.WriteLine("Yeeeeeee Haaaaaeewww!");
	}
}
```
通过默认构造方法，再创建一个Motorcycle类型的实例：
```C#
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Class Types*****\n");
	Motorcycle mc = new Motorcycle();
	mc.PopAWheely();
	...
}
```
一旦你定义了带参数的自定义构造方法，默认的构造方法被悄悄从类中移出，不再可用。
如果未定义自定义构造方法，C#编译器会允许对象的字段数据设置成正确的默认值。但是在你定义了一个唯一的构造方法时，编译器会假定你已经要掌控这些事。
所以，如果想让默认构造方法来创建类实例，你必须显示地对默认构造方法重新定义。示例：
```C#
class Motorcycle
{
	public int driverIntensity;
	public void PopAWheely()
	{
		for (int i = 0; i <= driverIntensity; i++)
		{
			Console.WriteLine("Yeeeeeee Haaaaaeewww!");
		}
	}
	// Put back the default constructor, which will
	// set all data members to default values.
	public Motorcycle() {}
	// Our custom constructor.
	public Motorcycle(int intensity)
	{
		driverIntensity = intensity;
	}
}
```
>注：VS IDE 提供一个 `ctor` 代码片段，键入 `ctor` 再调皮 Tab 键两次，IDE会自动替你定义一个自定义的默认构造方法。可以在其中加入自定义的参数和实现逻辑。

#### 关键定 this 的作用
该关键字能访问当前类实例。它的一个可能用法在于解析域（scope）的模糊性，这种模糊性在引入的参数与类中数据字段名称一致时就会产生。你当然可以采用一个不会导致这种二义性的命名规则。为说明该问题，在 Motorcycle 类中引入一个新的 string 字段名为 name，以表示发动机的名称。接着引入一个方法 setDriverName()：
```C#
class Motorcycle
{
	public int driverIntensity;
	// New members to represent the name of the driver.
	public string name;
	public void SetDriverName(string name)
	{
		name = name;
	}
	...
}
```
尽管代码编译正常，VS会显示警告信息说：you have assigned a variable back to itself!
为便于说明，修改 Main() 调用 SetDriverName 方法，然后打印 name 字段的值。你可能惊讶地发现 name 的值是空的：
```C#
// Make a Motorcycle with a rider named Tiny?
Motorcycle c = new Motorcycle(5);
c.SetDriverName("Tiny");
c.PopAWheely();
Console.WriteLine("Rider name is {0}", c.name); //Prints an empty name value!
```
问题在于SetDriverName的实现上，编译器假定当前方法域内name指向的是当前变量，而不是类域内的name字段。可用this来解决这种模糊性：
```C#
public void SetDriverName(string name)
{
	this.name = name;
}
```
要明白，如果类访问的是自己的数据字段或成员不存在二义模糊性问题时，可以不用this关键字的，它已经隐含了。例如，把字段 name 改成 driverName， 使用this就变成成可选了，不存在二义性了：
```C#
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	public void SetDriverName(string name)
	{
		// These two statements are functionally the same.
		driverName = name;
		this.driverName = name;
	}
	...
}
```

#### 链式构造方法调用使用 this
this 的另一种用法是使用一种名构造方法链的技巧来设计一个类。该设计模式在定义了多个构造方法的类中很管用。
假如构造方法要经常验证输入参数以保证各种业务规则，在类中的构造方法集合中找出冗余验证逻辑会很普通。修改Motorcycle:
```C#
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	public Motorcycle() { }
	// Redundant constructor logic!
	public Motorcycle(int intensity)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
	}
	public Motorcycle(int intensity, string name)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
		driverName = name;
	}
	...
}
```
代码中每个构造方法都确保 intensity 级别不大于 10。在两个构造器中存在冗余的代码语句。如果规则变化（比如，不大于5），就要在多处位置修改代码。
解决当前问题的办法是在类中定义一个方法来验证输入参数。这样每个构造方法只要调用该方法。该操作允许你代码隔离，以处理冗余：
```C#
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	// Constructors.
	public Motorcycle() { }
	public Motorcycle(int intensity)
	{
		SetIntensity(intensity);
	}
	public Motorcycle(int intensity, string name)
	{
		SetIntensity(intensity);
		driverName = name;
	 }
	public void SetIntensity(int intensity)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
	}
	...
}
```
一种更清洁环保的办法是设计一个构造方法，它把多数参数作为“主构造方法”(master constructor)，并且让他们必须执行验证逻辑。其他构造方法使用 this 关键字把输入参数转给主构造方法，提供必要的额外参数。这样，在整个类中，只需要维护一个构造方法，其他构造方法基本是空的。示例：
```C#
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	// Constructor chaining.
	public Motorcycle() {}
	public Motorcycle(int intensity)
	: this(intensity, "") {}
	public Motorcycle(string name)
	: this(0, name) {}
	// This is the 'master' constructor that does
	all the real work.
	public Motorcycle(int intensity, string name)
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
		driverName = name;
	}
	...
}
```
该技巧简化类定义，更好维护。同时，简化编程任务。

#### 构造方法流
一旦构造方法传参给主构造方法，被调用器最先调用的构造方法会完成执行余下代码。示例说明这一点：
```C#
class Motorcycle
{
	public int driverIntensity;
	public string driverName;
	// Constructor chaining.
	public Motorcycle()
	{
		Console.WriteLine("In default ctor");
	}
	public Motorcycle(int intensity)
	: this(intensity, "")
	{
		Console.WriteLine("In ctor taking an int");
	}
	public Motorcycle(string name)
	: this(0, name)
	{
		Console.WriteLine("In ctor taking a string");
	}
	// This is the 'master' constructor that does all the real work.
	public Motorcycle(int intensity, string name)
	{
		Console.WriteLine("In master ctor ");
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
		driverName = name;
	}
	...
}
```

再改写 Main() 方法中 Motorcycle 对象：
```C#
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with class Types*****\n");
	// Make a Motorcycle.
	Motorcycle c = new Motorcycle(5);
	c.SetDriverName("Tiny");
	c.PopAWheely();
	Console.WriteLine("Rider name is {0}",
	c.driverName);
	Console.ReadLine();
}
```
执行结果：
```
***** Fun with class Types *****
In master ctor
In ctor taking an int
Yeeeeeee Haaaaaeewww!
Yeeeeeee Haaaaaeewww!
Yeeeeeee Haaaaaeewww!
Yeeeeeee Haaaaaeewww!
Yeeeeeee Haaaaaeewww!
Yeeeeeee Haaaaaeewww!
Rider name is Tiny
```
如你所见，构造方法逻辑流程如下:
- 根据提供的单整数参数给创建的对象调用构造方法
- 该构造方法把参数转给主构造方法，并提供额外的初始参数
- 主构造方法指派输入的数据给对象的字段数据
- 控制权返回给最初调用的构造方法执行所有余下的代码语句

#### 复习可选参数
可选参数允许你对输入的参数提供默认值。如果调用方法乐意，可以不要求指定值，用默认值代替。下面的 Motorcycle版本提供了一个使用单一构造方法定义完成多种构造对象的方式：
```C#
class Motorcycle
{
	// Single constructor using optional args.
	public Motorcycle(int intensity = 0, string name= "")
	{
		if (intensity > 10)
		{
			intensity = 10;
		}
		driverIntensity = intensity;
		driverName = name;
	}
	...
}
static void MakeSomeBikes()
{
	// driverName = "", driverIntensity = 0
	Motorcycle m1 = new Motorcycle();
	Console.WriteLine("Name= {0}, Intensity= {1}",
	m1.driverName, m1.driverIntensity);
	// driverName = "Tiny", driverIntensity = 0
	Motorcycle m2 = new Motorcycle(name:"Tiny");
	Console.WriteLine("Name= {0}, Intensity= {1}", m2.driverName, m2.driverIntensity);
	// driverName = "", driverIntensity = 7
	Motorcycle m3 = new Motorcycle(7);
	Console.WriteLine("Name= {0}, Intensity= {1}", m3.driverName, m3.driverIntensity);
}
```

#### static关键字的使用
static定义的成员必须以类的级别调用，不能以对象引用变量方式调用。为说明区别，以 System.Console 类为例说明：
```C#
// Compiler error! WriteLine() is not an object level method!
Console c = new Console();
c.WriteLine("I can't be printed...");
```
正确的写法应该是用类名做前缀：
```C#
// Correct! WriteLine() is a static method.
Console.WriteLine("Much better! Thanks...");
```
简言之，静态成员被视作普通项，完全无必要在调用该成员前先创建该类的实例，在 utility 类中相当常见。比如象 Console、Math、Environment。
要记住，静态成员提升指定项到类级别，不是对象级别。static 关键字用于如下类型：
- 类的数据
- 类的方法
- 类的属性
- 构造方法
- 整个类定义
- 与C#的using关键字联合使用

##### 定义静态字段数据
静态数据为所有对象共享，如果值被更改，则所有对象都得到新值。

##### 定义静态方法
>注：非静态成员的方法实现中对静态成员引用会有一个编译错误。另一个错误是在静态成员中使用 this 关键字，因为 this 隐含指向一个对象！

##### 定义静态构造方法
典型的构造方法用于设定一个对象创建时的实例级别的数据。但是如果要在构造方法中给静态数据赋值呢？你可能吃惊的发现，这个静态值会在每次创建新对象时都被重置！
一个设置静态字段的办法就是使用成员初始化语法。不管类创建多少个对象，可确保静态字段仅一次赋值。但是如果该值要在运行时才能获取到该怎么办呢？这就必须要用构造方法。
为解决这个问题引入了静态构造方法，它允许安全修改静态数据。示例：
```C#
class SavingsAccount
{
	public double currBalance;
	public static double currInterestRate;
	public SavingsAccount(double balance)
	{
		currBalance = balance;
	}
	// A static constructor!
	static SavingsAccount()
	{
		Console.WriteLine("In static ctor!");
		currInterestRate = 0.04;
	}
	...
}
```
简言之，静态构造方法是一个特殊的构造方法，它对在编译时仍未确定的静态值进行初始化是理想所在。注意CLR在第一次类实例化前只调用所有静态构造方法一次。以下关于静态构造方法的要点：
- 指定的类仅可以定义唯一一个静态构造方法。换言之，静态构造方法不能重载。
- 静态构造方法不用任何访问修饰符，并且不能接收任何参数
- 静态构造方法只执行一次，不管类创建了多少对象。
- 运行时创建一个类的实例时，或在通过调用器访问第一个静态成员前，先调用静态构造方法
- 静态构造方法在任何实例级别的构造方法执行前先执行。

##### 定义静态类
一个类被定义成static时，就不可再用 new 来创建对象，它的数据字段和成员都必须用 static 关键字，否则编译会报错。该用法常用于实用工具类(utility class)。
```C#
// Static classes can only contain static members!
static class TimeUtilClass
{
	public static void PrintTime() => Console.WriteLine(Now.ToShortTimeString());
	public static void PrintDate() => Console.WriteLine(Today.ToShortDateString());
}
static void Main(string[] args)
{
	Console.WriteLine("***** Fun with Static Classes*****\n");
	// This is just fine.
	TimeUtilClass.PrintDate();
	TimeUtilClass.PrintTime();

	// Compiler error! Can't create instance of static classes!
	TimeUtilClass u = new TimeUtilClass ();
	Console.ReadLine();
}
```

##### 通过 using 关键字导入静态成员
C#6 支持using关键字导入静态成员。想一下C#所定义的实用工具类：调用Console类的WriteLine()方法，调用DateTime类的Now属性，一定要先用 using 引入 System 命名空间。既然这些类的成员全为静态，那么就可以使用静态的using指令：
```C#
// Import the static members of Console and DateTime.
using static System.Console;
using static System.DateTime;
```
这样，代码就可以直接使用Console和DateTime类的静态成员了，不必再加类名前缀：
```C#
static class TimeUtilClass
{
	public static void PrintTime() => WriteLine(Now.ToShortTimeString());
	public static void PrintDate() => WriteLine(Today.ToShortDateString());
}
```

注意，过度使用静态导入会存在潜在混淆问题。第一，如果多个类定义了WriteLine()方法怎么办？编译器不明所以，也影响别人阅读。其次，除非开发者熟知.NET代码库，否则不一定知道WriteLine()是Console类的成员。基于这些原因，我会限制本文中静态using的使用。

---
#### 定义OOP之柱
面向对象三原则：
- 封装：隐藏对象的内部实现细节，保护数据完整性
- 继承：提高代码的重用性
- 多态：以相同类似方式处理相关联对象

#### C# 访问修饰符
C# 访问修饰符 | 适用于 | Meaning in Life
-|-|-
public | 类型或类型成员 | 无访问限制。对象可访问public成员，派生类一样。其他外部程序集也可访问 public 类型。
private | 类型成员或嵌套类型 | private 项仅能被定义该项的类（或结构）访问。
protected | 类型成员或嵌套类型 | protected 项仅能被定义它和其所有子类访问使用。因此，protected 项从外部通过C#点操作符访问。
internal | 类型成员或嵌套类型 | internal 项仅在当前程序集内可访问。因此，如果在.NET类库内定义了一套internal类型，则这些类型无法被其他程序集使用。
protected internal | 类型成员或嵌套类型 | 当 protected 和 internal 关键字组合修饰某项时，该项可在被定义的程序集和被定义的类内访问使用，也可以被派生类访问。

##### 默认访问修饰符
类型成员默认是隐式的 private，而类型是隐式的 internal。因此，下面代码中的类的定义自动设成internal，而类型的默认构造方法自动设成private：
```C#
// An internal class with a private default constructor.
class Radio
{
	Radio(){}
}
```
等效于：
```C#
// An internal class with a private default constructor.
internal class Radio
{
	private Radio(){}
}
```

##### 访问修饰符及嵌套类型
嵌套类型是一个在类或结构的域内直接声明的类型。举例说明，private 枚举类型 CarColor 嵌套在 public 类 SportsCar内：
```C#
public class SportsCar
{
	// OK! Nested types can be marked private.
	private enum CarColor
	{
		Red, Green, Blue
	}
}
```
这里，对嵌套类型上使用private访问修饰符是允许的。但是非嵌套类型（这里是 SportsCar）仅能使用 public或 internal修饰符。因此以下类定义是非法的：
```C#
// Error! Nonnested types cannot be marked private!
private class SportsCar
{}
```

---

#### 第一支柱：C#的封装服务
（P332）


### 2 继承和多态

### 3 结构化异常的处理

### 4 与接口同行


























