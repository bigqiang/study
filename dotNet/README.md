# .NET学习笔记
内容多源于《Pro C# 7》学习及相关的文档。

.NET的哲学

## 一、基本
### 1 字串
#### 字串插入
```c#
int age = 4;
string name = "Soren";
// Using curly bracket syntax.
string greeting = string.Format("Hello {0} you are {1} years old.", name, age);
// Using string interpolation
string greeting2 = $"Hello {name} you are {age} years old.";
string greeting = string.Format("Hello {0} you are {1} years old.", name.ToUpper(), age);
string greeting2 = $"Hello {name.ToUpper()} you are {age} years old.";
string greeting = string.Format("\tHello {0} you are {1} years old.", name.ToUpper(), age);
string greeting2 = $"\tHello {name.ToUpper()} you are {age} years old.";
```

### 2 switch/case
```C#
//This is new the pattern matching switch statement
// 新特性，可对类型检查
switch (choice)
{
    case int i:
        Console.WriteLine("Your choice is an integer {0}.",i);
        break;
    case string s:
        Console.WriteLine("Your choice is a string. {0}", s);
        break;
    case decimal d:
        Console.WriteLine("Your choice is a decimal. {0}", d);
        break;
    default:
        Console.WriteLine("Your choice is something else");
        break;
}
```

```C#
//除对类型检查，还对值进行匹配
static void ExecutePatternMatchingSwitchWithWhen()
{
    Console.WriteLine("1 [C#], 2 [VB]");
    Console.Write("Please pick your language preference: ");
    object langChoice = Console.ReadLine();
    var choice = int.TryParse(langChoice.ToString(),
    out int c) ? c : langChoice;
    switch (choice)
    {
        case int i when i == 2:
        case string s when s.Equals("VB",
            StringComparison.OrdinalIgnoreCase):
            Console.WriteLine("VB: OOP, multithreading, and more!");
            break;
        case int i when i == 1:
        case string s when s.Equals("C#", StringComparison.OrdinalIgnoreCase):
            Console.WriteLine("Good choice, C# is a fine language.");
            break;
        default:
            Console.WriteLine("Well...good luck with that!");
            break;
    }
    Console.WriteLine();
}
```
### 3 数组
```C#
// 创建索引号为0、1、2的三个int整数元素的数组
int[] myInts = new int[3];

// 创建100个string项的数组，索引从 0 - 99
string[] booksOnDotNet = new string[100];
```

声明一组但没有显式地给每个索引项填充数据，则该项会以该数据类型的默认值填充。如 bool数组默认值是 false, int数组默认值0。

#### C#数组的初始化
```C#
// 使用 new 关键词
string[] stringArray = new string[] { "one", "two", "three" };
Console.WriteLine("stringArray has {0} elements", stringArray.Length);
// 也可以不使用 new 关键词
bool[] boolArray = { false, false, true };
Console.WriteLine("boolArray has {0} elements", boolArray.Length);
// 用 new 关键词，并设置大小
int[] intArray = new int[4] { 20, 22, 23, 0 };
Console.WriteLine("intArray has {0} elements", intArray.Length);
Console.WriteLine();
}
```

#### 局部数组变量的隐式类型(var)
```C#
// a is really int[].
var a = new[] { 1, 10, 100, 1000 };
Console.WriteLine("a is a: {0}", a.ToString());
// b is really double[].
var b = new[] { 1, 1.5, 2, 2.5 };
Console.WriteLine("b is a: {0}", b.ToString());
// c is really string[].
var c = new[] { "hello", null, "world" };
Console.WriteLine("c is a: {0}", c.ToString());
```

与预期不同，隐式类型的局部数组元素默认不是 `System.Object`；因此，下面代码会生成编译时错误：
```C#
// Error! Mixed types!
var d = new[] { 1, "one", 2, "two", false };
```

#### 对象类型数组的定义
`System.Object`在.NET类型系统中是所有类型的终极基类（包括基础的数据类型）。因此，如果定义数组的数据类型是 `System.Object`，则其子项可以是任何东西，看以下代码：
```C#
// An array of objects can be anything at all.
object[] myObjects = new object[4];
myObjects[0] = 10;
myObjects[1] = false;
myObjects[2] = new DateTime(1969, 3, 24);
myObjects[3] = "Form & Void";
foreach (object obj in myObjects)
{
    // Print the type and value for each item in
    array.
    Console.WriteLine("Type: {0}, Value: {1}",
    obj.GetType(), obj);
}
```
以上代码会返回以下结果：
```
Type: System.Int32, Value: 10
Type: System.Boolean, Value: False
Type: System.DateTime, Value: 3/24/1969 12:00:00 AM
Type: System.String, Value: Form & Void
```

#### 多维数组使用
除一维数组外，C#支持两种多维数组。一种称为`矩形数组`，这是一种简单的多维数组，第一行长度相同。声明及填充方法如下：
```C#
// A rectangular MD array.
int[,] myMatrix;
myMatrix = new int[3,4];
// Populate (3 * 4) array.
for(int i = 0; i < 3; i++)
    for(int j = 0; j < 4; j++)
        myMatrix[i, j] = i * j;
// Print (3 * 4) array.
for(int i = 0; i < 3; i++)
{
    for(int j = 0; j < 4; j++)
    Console.Write(myMatrix[i, j] + "\t");
    Console.WriteLine();
}
```
第二种多维数组类型称之为` jagged array`（锯齿数组）。顾名思义，该类型数组包含多个内部数组，且每个数组的上限不尽相同，如：
```C#
// A jagged MD array (i.e., an array of arrays).
// Here we have an array of 5 different arrays.
int[][] myJagArray = new int[5][];
// Create the jagged array.
for (int i = 0; i < myJagArray.Length; i++)
    myJagArray[i] = new int[i + 7];
// Print each row (remember, each element is defaulted to zero!).
for(int i = 0; i < 5; i++)
{
    for(int j = 0; j < myJagArray[i].Length; j++)
        Console.Write(myJagArray[i][j] + " ");
    Console.WriteLine();
}
```

#### 数组做参数或返回值
示例：
```C#
static void PrintArray(int[] myInts)
{
    for(int i = 0; i < myInts.Length; i++)
        Console.WriteLine("Item {0} is {1}", i,    myInts[i]);
}
static string[] GetStringArray()
{
    string[] theStrings = {"Hello", "from","GetStringArray"};
    return theStrings;
}
```

#### 基类 System.Array
每个创建的数组都有很多来自`System.Array`类的功能。如下表提供一些成员方式：
Clear()        静态方法，将数组中一定范围的元素设置空值（数值型0 对象引用null，布尔型false）
CopyTo()    用于复制源数组元素到目标数组中
Length    属性。返回数组的项数
Rank    属性。返回当前数组的维数
Reverse()    静态方法。对一维数组的内容反转
Sort()    静态方法。对一维数组基础类型数据排序，也可对自定义类型排序。


### 方法和参数修饰符
#### 返回值及expression-bodied 成员
`static int Add(int x, int y) => x + y;
这是一种单行的简化语法，常称作语法糖，意味着生成的IL并没有什么不同。该语法常用作只读成员属性。
C# 7扩展了该能力，包括单行构造器、finalizer、get和set属性访问器和索引器。

#### 方法参数修饰符
给函数传参默认是值传递。简言之，如果你未对参数加上修饰符，传递函数的就是参数值的拷贝。
参数修饰符：
（None） 没有修饰符，会以值传递
out 输出参数必须由调用的方法指派，因此传递的引用。如果调用的方法不指派输出参数，则会编译错误
ref 由调用方法初始指派的值，该值可以被调用方法随意修饰（也是引用传值）。如调用的方法不指派 ref 参数，不会有编译错误
params 该参数修饰符允许你使用可变数量的参数以单个逻辑参数发送。一个方法仅有一个 params 修饰符，它必须是方法最后一个参数。实际上，你可能不常用到它。

##### out修饰符
以下Add方法使用out返回两个整数之和。注意该方法return实际返回的是void：
```C#
// out参数必须由调用方法指定
static void Add(int x, int y, out int ans)
{
    ans = x + y;
}

static void Main(string[] args)
{
    Console.WriteLine("***** Fun with Methods *****");
    ...
    // 不用给用做输出的局部变量指派初值, 仅作为输出参数用于一次性使用
    // C# 7 允许 out 参数在方法调用中声明
    Add(90, 90, out int ans);
    Console.WriteLine("90 + 90 = {0}", ans);
    Console.ReadLine();
}
```
C# 的 out 修饰符有一个有用目的：允许调用者从单一方法中获取多个输出。
```C#
// Returning multiple output parameters.
static void FillTheseValues(out int a, out string b, out bool c)
{
    a = 9;
    b = "Enjoy your string.";
    c = true;
}
static void Main(string[] args)
{
    Console.WriteLine("***** Fun with Methods *****");
    ...
    int i; string str; bool b;
    FillTheseValues(out i, out str, out b);
    Console.WriteLine("Int is: {0}", i);
    Console.WriteLine("String is: {0}", str);
    Console.WriteLine("Boolean is: {0}", b);
    Console.ReadLine();
}
```
记住，定义输出参数的方法一定要在退出该方法域前给该参数赋一个合法值。下面代码未遵从该原则，会导致一个编译错误：
```C#
static void ThisWontCompile(out int a)
{
    Console.WriteLine("Error! Forgot to assign output arg!");
}
```
如果不在意 out 参数值，可用一个下划线（discard）做占位符。比如，想决定一个字串是否是合法的日期格式，但不关心解析后的日期，可以这样写：
```C#
if (DateTime.TryParse(dateString, out _)
{
    //do something
}
```
#### ref 修饰符
out 和 ref 差别
 1. 在传给方法前，out参数不用初始化。在方法退出前，必须在方法内赋值。
 2. 在传值给方法前，ref 参数必须初始化。如果未赋值，则它与未赋值的局部变量相同。

```C#
// Reference parameters.
public static void SwapStrings(ref string s1, ref string s2)
{
    string tempStr = s1;
    s1 = s2;
    s2 = tempStr;
}
static void Main(string[] args)
{
    Console.WriteLine("***** Fun with Methods *****");
    ...
    string str1 = "Flip";
    string str2 = "Flop";
    Console.WriteLine("Before: {0}, {1} ", str1, str2);
    SwapStrings(ref str1, ref str2);
    Console.WriteLine("After: {0}, {1} ", str1, str2);
    Console.ReadLine();
}
```

##### ref修饰的局部变量及返回值
ref除修饰参数外，C#7还可在其他地方定义使用和返回引用型变量。
```C#
// Returning a reference.
public static ref string SampleRefReturn(string[]
strArray, int position)
{
    return ref strArray[position];
}
```
任何指定引用的变化返回都会修改这个数组，如下：
```C#
#region Ref locals and params
Console.WriteLine("=> Use Ref Return");
Console.WriteLine("Before: {0}, {1}, {2} ", stringArray[0], stringArray[1], stringArray[2]);
ref var refOutput = ref SampleRefReturn(stringArray, pos);
refOutput = "new";
Console.WriteLine("After: {0}, {1}, {2} ", stringArray[0], stringArray[1], stringArray[2]);
```
执行结果：
=> Use Ref Return
Before: one, two, three
After: one, new, three

有三点值得注意：
 1. 标准方法的结果不能赋值给一个ref的局部变量。该方法必须为一个ref修饰返回的方法。
 2. 在ref修饰的方法内的局部变量不能以 ref修饰的局部变量返回。以下代码不会运行：
 ```C#
 ThisWillNotWork(string[] array)
 {
    int foo = 5;
    return ref foo;
 }
 ```
 3. 该新特性不适用于异步方法

#### params修饰符
可支持数组参数的使用。该关键定允许你给方法传递数量可变类型一致的参数（或有继承关系的类类型）作为单个逻辑参数。
```C#
// Return average of "some number" of doubles.
static double CalculateAverage(params double[] values)
{
    Console.WriteLine("You sent me {0} doubles.", values.Length);
    double sum = 0;
    if(values.Length == 0)
        return sum;
    for (int i = 0; i < values.Length; i++)
        sum += values[i];
    return (sum / values.Length);
}
static void Main(string[] args)
{
    Console.WriteLine("***** Fun with Methods *****");
    ...
    // Pass in a comma-delimited list of doubles...
    double average;
    average = CalculateAverage(4.0, 3.2, 5.7, 64.22,87.2);
    Console.WriteLine("Average of data is: {0}", average);
    // ...or pass an array of doubles.
    double[] data = { 4.0, 3.2, 5.7 };
    average = CalculateAverage(data);
    Console.WriteLine("Average of data is: {0}", average);
    // Average of 0 is 0!
    Console.WriteLine("Average of data is: {0}", CalculateAverage());
    Console.ReadLine();
}
```

注意：为避免歧义，C#要求方法只能支持一个params参数，而且必须是参数列表中最后一个传参。

#### 定义可选参数
C#允许你创建的方法中可以选填参数。即在调用方法时允许省略你认为不必要的参数。
```C#
static void EnterLogData(string message, string owner = "Programmer")
{
    Console.Beep();
    Console.WriteLine("Error: {0}", message);
    Console.WriteLine("Owner of Error: {0}", owner);
}
static void Main(string[] args)
{
    Console.WriteLine("***** Fun with Methods *****");
    ...
    EnterLogData("Oh no! Grid can't find data");
    EnterLogData("Oh no! I can't find the payroll data", "CFO");
    Console.ReadLine();
}
```
注意：可选参数必须在编译时确定，绝不能在运行时解析确定。否则会报编译时错误。下面改写上面的EnterLogData方法说明。
```C#
// Error! The default value for an optional arg must be known
// at compile time!
static void EnterLogData(string message, string owner = "Programmer", DateTime timeStamp = DateTime.Now)
{
    Console.Beep();
    Console.WriteLine("Error: {0}", message);
    Console.WriteLine("Owner of Error: {0}", owner);
    Console.WriteLine("Time of Error: {0}", timeStamp);
}
```
这会编译失败，因为 DateTime类中的 Now属性在运行时才会解析，不会在编译时进行。

为避免歧义，可选参数总是占位于方法签名的尾部。否则会报编译器错误。

##### 使用命名参数调用方法
命名参数（named argument）允许以随意选择顺序的参数调用方法。示例：
```C#
static void DisplayFancyMessage(ConsoleColor textColor, ConsoleColor backgroundColor, string message)
{
    // Store old colors to restore after message is printed.
    ConsoleColor oldTextColor = Console.ForegroundColor;
    ConsoleColor oldbackgroundColor = Console.BackgroundColor;
    // Set new colors and print message.
    Console.ForegroundColor = textColor;
    Console.BackgroundColor = backgroundColor;
    Console.WriteLine(message);
    // Restore previous colors.
    Console.ForegroundColor = oldTextColor;
    Console.BackgroundColor = oldbackgroundColor;
}
static void Main(string[] args)
{
    Console.WriteLine("***** Fun with Methods *****");
    ...
    DisplayFancyMessage(message: "Wow! Very Fancy indeed!", textColor: ConsoleColor.DarkRed, backgroundColor: ConsoleColor.White);
    DisplayFancyMessage(backgroundColor: ConsoleColor.Green, message: "Testing...", textColor: ConsoleColor.DarkBlue);
    Console.ReadLine();
}
```
以上代码执行良好。如果要想使用命名参数调用方法，就必须在调用时先列出形参名称后面加上冒号，再跟上实参。
注意下面的两种写法，前者正确，后者错误：
```C#
// This is OK, as positional args are listed before named args.
DisplayFancyMessage(ConsoleColor.Blue, message: "Testing...", backgroundColor: ConsoleColor.White);
// This is an ERROR, as positional args are listed after named args.
DisplayFancyMessage(message: "Testing...", backgroundColor: ConsoleColor.White, ConsoleColor.Blue);
```
命名参数经常与可选参数联合使用：
```C#
static void DisplayFancyMessage(ConsoleColor textColor = ConsoleColor.Blue, ConsoleColor backgroundColor = ConsoleColor.White, string message = "Test Message")
{
    ...
}

//仅需要传参 message ，其他用默认
DisplayFancyMessage(message: "Hello!");
```
##### 方法的重载
简言之，定义了一组名称相同的方法，唯一的不同就是虚参的数量或类型，被称之重载的方法。

注意：泛型方法就使用了重载的概念。

#### 局部函数（新）
C# 7的新特性，在方法中创建方法。局部函数是在另一个函数中声明的函数。

注意：一直用“方法”，怎么突然用“函数”。两者有什么区别。学术上讲，二者不同仍有争议。实际上二者可互换使用。在本文中，二者等效。官方命名“局部函数”，我不想因本文一致性而改变它。

示例：
```C#
static int AddWrapper(int x, int y)
{
    //Do some validation here
    return Add();
    int Add()
    {
        return x + y;
    }
}
```

### enum类型
注意：不要把 enum 和 enumerator 混淆。enum 是名值对的自定义数据类型。enumerator 是.NET接口 IEnumerable 实现的类或结构。通常这个接口都是用象System.Array这样的集合类（collection class）来实现。

示例：
```C#
// A custom enumeration.
enum EmpType
{
    Manager, // = 0
    Grunt, // = 1
    Contractor, // = 2
    VicePresident // = 3
}
// Begin with 102.
enum EmpType
{
    Manager = 102,
    Grunt, // = 103
    Contractor, // = 104
    VicePresident // = 105
}
// 也可以不连续
enum EmpType
{
    Manager = 10,
    Grunt = 1,
    Contractor = 100,
    VicePresident = 9
}
```

#### Controlling the Underlying Storage for an enum
（P260）



















