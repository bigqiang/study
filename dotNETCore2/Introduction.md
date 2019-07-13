# 介绍

## .NET Core 的世界
1. .NET Core 开源
2. .NET Core 使用现代模式
3. .NET Core 支持多平台开发
4. ASP.NET Core 可在Windows 或 Linux 上运行

## C# 的世界
C#6中，编译器的源码完全重写了。
C#7中，又对函数编程后台新增很多特性，如局部函数、元组、模式匹配。

## C# 7 新特性
C#6扩展包含 `static using`，expression-bodied 方法以及属性，自动完成属性初始化器，字典初始化器，异常筛选器， 以及await in catch。具体C#7有哪些改变呢？

### 数字分隔符
使用 `_` 分隔声明变量的数字，可增加代码易读性。编译器会移除 `_`。示例：
```C#
//In C# 6
long n1 = 0x1234567890ABCDEF;

//In C# 7
long n2 = 0x1234_5678_90AB_CDEF;

//In C# 7.2,也可使用分隔符
long n2 = 0x_1234_5678_90AB_CDEF;
Digit separators are covered in Chapter 2, “Core C#.”

```

### 二进制字面量
C#7为二进制提供了一个新字面量。二进制仅能有值0或1。现在数字分隔符变得特别重要：
```C#
//In C# 7
uint binary1 = 0b1111_0000_1010_0101_1111_0000_1010_0101;
```

### Expression-Bodied 成员
C#6允许 Expression-Bodied 方法和属性。C# 7中，expression-bodie用于构造器、析构器、局部函数、属性访问器等。以下是C#6/7间区别：
```C#
//In C# 6
private string _firstName;
public string FirstName
{
	get { return _firstName; }
	set { Set(ref _firstName, value); }
}
//In C# 7
private string _firstName;
public string FirstName
{
	get => _firstName;
	set => Set(ref _firstName, value);
}
```

### out 变量
C# 7以前，`out`变量的类型必须在变量使用前声明。 C# 7中可以省一行代码，因为变量可以在使用时声明：
```C#
//In C# 6
string n = "42";
int result;
if (string.TryParse(n, out result)
{
	Console.WriteLine($"Converting to a number was successful: {result}");
}
//In C# 7
string n = "42";
if (string.TryParse(n, out var result)
{
	Console.WriteLine($"Converting to a number was successful: {result}");
}
```

### Non-Trailing Named Arguments
C# supports named arguments that are required with optional arguments but can support readability in any
cases. With C# 7.2, non-trailing named arguments are supported. Argument names can be added to any
argument with C# 7.2:
In C# 7.0
if (Enum.TryParse(weekdayRecommendation.Entity, ignoreCase: true,
result: out DayOfWeek weekday ))
{
reservation.Weekday = weekday;
}
lvii
INTRODUCTION
In C# 7.2
if (Enum.TryParse(weekdayRecommendation.Entity, ignoreCase: true,
out DayOfWeek weekday))
{
reservation.Weekday = weekday;
}


### Readonly Struct

### In Parameters

### Private Protected

### Target-Typed Default

### Local Functions

### Tuples


### Inferred Tuple Names

### Deconstructors


### Pattern Matching


### Throw Expressions

### Async Main


### Reference Semantics





## ASP.NET Core 新特性

## 通用Windows平台新特性

## 需要怎么写和跑C#代码

## 本树封面

## 约定

## 源码

## GitHub

## 勘误

