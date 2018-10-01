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



### 2 继承和多态

### 3 结构化异常的处理

### 4 与接口同行

