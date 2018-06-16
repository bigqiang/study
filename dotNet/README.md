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
默认状态下，枚举类型保存值的存储类型是System.Int32（C# int）。不过这可以随时修改。C#的枚举类型对任意核心系统类型（byte，short，int，或long）。例如，想设置EmpType存储类型是 byte 而不是 int ， 可以这样写：
```C#
// This time, EmpType maps to an underlying byte.
enum EmpType : byte
{
    Manager = 10,
    Grunt = 1,
    Contractor = 100,
    VicePresident = 9
}
```
修改枚举的基础类型有助于在低内存设备上构建.NET应用。当然每个值必须在限定范围内，比如下代码就会报编译器错误：
```C#
// Compile-time error! 999 is too big for a byte!
enum EmpType : byte
{
    Manager = 10,
    Grunt = 1,
    Contractor = 100,
    VicePresident = 999
}
```

#### 枚举变量的声明
一旦确定了枚举的范围和存储类型后，就可用它代替数字。枚举是用户定义的数据类型，可以用于函数的返回值、方法的参数、局部变量等。假定你有一个名为 AskForBonus() 的方法，用 EmpType 做唯一参数，根据输出的参数值打印出对应的支付请求：
```C#
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("**** Fun with Enums *****");
        // Make an EmpType variable.
        EmpType emp = EmpType.Contractor;
        AskForBonus(emp);
        Console.ReadLine();
    }
    // Enums as parameters.
    static void AskForBonus(EmpType e)
    {
        switch (e)
        {
            case EmpType.Manager:
                Console.WriteLine("How about stock options instead?");
                break;
            case EmpType.Grunt:
                Console.WriteLine("You have got to be kidding...");
                break;
            case EmpType.Contractor:
                Console.WriteLine("You already get enough cash...");
                break;
            case EmpType.VicePresident:
                Console.WriteLine("VERY GOOD, Sir!");
                break;
        }
    }
}
```
注意：枚举类型是名值对确定的一套数据，因此给枚举变量设置一个未定义的值是非法。

#### System.Enum 类型
.NET的枚举类型是由 System.Enum 类获得功能。该类定义了许多方法，允许查询和转换一个给定的枚举。比如方法 `Enum.GetUnderlyingType()` ，就如名字暗示的，它会返回存储枚举类型值的数据类型。如前面的声明的 EmpType 类型是 System.Byte，代码示例：
```C#
static void Main(string[] args)
{
    Console.WriteLine("**** Fun with Enums *****");
    // Make a contractor type.
    EmpType emp = EmpType.Contractor;
    AskForBonus(emp);
    // Print storage for the enum.
    Console.WriteLine("EmpType uses a {0} for storage", Enum.GetUnderlyingType(emp.GetType()));
    Console.ReadLine();
}
```
如上所示，用了`GetType()`方法来获取元数据，该方法对.NET基类库中所有类型都通用。还有一个办法使用有C#的typeof运算符。这样做的好处是不必见到变量的元数据实体。
```C#
// This time use typeof to extract a Type.
Console.WriteLine("EmpType uses a {0} for storage", Enum.GetUnderlyingType(typeof(EmpType)));
```

#### 动态查找枚举的名值对
除 `Enum.GetUnderlyingType()` 方法，所有C#枚举还支持 `ToString()`。
```C#
static void Main(string[] args)
{
    Console.WriteLine("**** Fun with Enums *****");
    EmpType emp = EmpType.Contractor;
    AskForBonus(emp);
    // Prints out "emp is a Contractor".
    Console.WriteLine("emp is a {0}.", emp.ToString());
    Console.ReadLine();
}
```

System.Enum 也定义另一个静态方法 GetValues()。该方法返回了一个 `System.Array` 的实例。数组中的每个元素项与指定枚举的一个成员对应。它能打印任意枚举类型传参的名值对：
```C#
// This method will print out the details of any enum.
static void EvaluateEnum(System.Enum e)
{
    Console.WriteLine("=> Information about {0}", e.GetType().Name);
    Console.WriteLine("Underlying storage type: {0}", Enum.GetUnderlyingType(e.GetType()));
    // Get all name-value pairs for incoming parameter.
    Array enumData = Enum.GetValues(e.GetType());
    Console.WriteLine("This enum has {0} members.", enumData.Length);
    // Now show the string name and associated value, using the D format
    // flag (see Chapter 3).
    for(int i = 0; i < enumData.Length; i++)
    {
        Console.WriteLine("Name: {0}, Value: {0:D}", enumData.GetValue(i));
    }
    Console.WriteLine();
}
```

### 理解结构类型（也叫值类型）
结构是用户定义类型（如枚举）；结构不是简单的名值对集合，它包含任意数量的数据字段以及操作这些字段的成员方法。
```C#
struct Point
{
    // Fields of the structure.
    public int X;
    public int Y;
    // Add 1 to the (X, Y) position.
    public void Increment()
    {
        X++; Y++;
    }
    // Subtract 1 from the (X, Y) position.
    public void Decrement()
    {
        X--; Y--;
    }
    // Display the current position.
    public void Display()
    {
        Console.WriteLine("X = {0}, Y = {1}", X, Y);
    }
}
static void Main(string[] args)
{
    Console.WriteLine("***** A First Look at Structures *****\n");
    // Create an initial Point.
    Point myPoint;
    myPoint.X = 349;
    myPoint.Y = 76;
    myPoint.Display();
    // Adjust the X and Y values.
    myPoint.Increment();
    myPoint.Display();
    Console.ReadLine();
}
```
输出结果：
```
***** A First Look at Structures *****
X = 349, Y = 76
X = 350, Y = 77
```
注意：在类和结构中定义public数据常被看作坏风格。

#### 结构变量的创建
创建好结构变量，在调用其成员方法前要先对每个public字段数据赋值，否则会有编译器错误：
```C#
// Error! Did not assign Y value.
Point p1;
p1.X = 10;
p1.Display();
// OK! Both fields assigned before use.
Point p2;
p2.X = 10;
p2.Y = 10;
p2.Display();
```
也可以用 new 关键词创建结构变量，它会调用结构的默认构造器。按定义，默认构造器没有任何参数。调用结构的默认构造器的好处是会给每个字段数据自动设置成默认值。
```C#
// Set all fields to default values
// using the default constructor.
Point p1 = new Point();
// Prints X=0,Y=0.
p1.Display();
```
还可以自定义一个构造器。如下：
```C#
struct Point
{
    // Fields of the structure.
    public int X;
    public int Y;
    // A custom constructor.
    public Point(int XPos, int YPos)
    {
        X = XPos;
        Y = YPos;
    }
    ...
}

...
// Call custom constructor.
Point p2 = new Point(50, 60);
// Prints X=50,Y=60.
p2.Display();
```

### 值类型与引用类型区别
与数组、字串或枚举不同，结构在.NET库中没有一样的命名表示（即，没有System.Structure类），仅隐式地派生自 System.ValueType。System.ValueType的角色保证了所派生的类型都是在堆栈中分配，而不是在可被垃圾收集的堆中。栈中的分配的数据创建和销毁都很快，生命周期由定义的域决定；堆分配的数据，由垃圾收集器监控，生命周期由许多因素决定。
功能上，System.ValueType 的唯一目的就是重载由 基于值与基于引用语义的System.Object定义的虚拟方法。ValueType的基类是System.Object。

#### 值类型、引用类型以及赋值操作符
```C#
// Assigning two intrinsic value types results in
// two independent variables on the stack.
static void ValueTypeAssignment()
{
    Console.WriteLine("Assigning value types\n");
    Point p1 = new Point(10, 10);
    Point p2 = p1;
    // Print both points.
    p1.Display();
    p2.Display();
    // Change p1.X and print again. p2.X is not changed.
    p1.X = 100;
    Console.WriteLine("\n=> Changed p1.X\n");
    p1.Display();
    p2.Display();
}
```
这里将p1赋值给p2，二者是值类型。则二者分别独立的。因此更其中一个结构的字段数据不会影响别一个：
```
Assigning value types
X = 10, Y = 10
X = 10, Y = 10
=> Changed p1.X
X = 100, Y = 10
X = 10, Y = 10
```
与之形成强烈对照的是引用类型（就是所有的类实例），该引用变量指向的是内存。举例：
```C#
// Classes are always reference types.
class PointRef
{
    // Same members as the Point structure...
    // Be sure to change your constructor name to PointRef!
    public PointRef(int XPos, int YPos)
    {
        X = XPos;
        Y = YPos;
    }
}
static void ReferenceTypeAssignment()
{
    Console.WriteLine("Assigning reference types\n");
    PointRef p1 = new PointRef(10, 10);
    PointRef p2 = p1;
    // Print both point refs.
    p1.Display();
    p2.Display();
    // Change p1.X and print again.
    p1.X = 100;
    Console.WriteLine("\n=> Changed p1.X\n");
    p1.Display();
    p2.Display();
}
```
本例中两个引用指向受控堆中同一对象，因此是用p1的引用更改的X值，p2.X会报出相同值。输出结果如下：
```
Assigning reference types
X = 10, Y = 10
X = 10, Y = 10
=> Changed p1.X
X = 100, Y = 10
X = 100, Y = 10
```

#### 包含引用类型的值类型
以下一个类使用了自定义构造器，由一个字串信息维护。还用到一个结构。
```C#
class ShapeInfo
{
    public string InfoString;
    public ShapeInfo(string info)
    {
        InfoString = info;
    }
}
struct Rectangle
{
    // The Rectangle structure contains a reference type member.
    public ShapeInfo RectInfo;
    public int RectTop, RectLeft, RectBottom, RectRight;
    public Rectangle(string info, int top, int left, int bottom, int right)
    {
        RectInfo = new ShapeInfo(info);
        RectTop = top; RectBottom = bottom;
        RectLeft = left; RectRight = right;
    }
    public void Display()
    {
        Console.WriteLine("String = {0}, Top = {1}, Bottom = {2}, " + "Left = {3}, Right = {4}", RectInfo.infoString, RectTop, RectBottom, RectLeft, RectRight);
    }
}
```
本例中，值类型中有一个引用类型。如果将一个 Rectangele变量赋值给另一个会怎样？引用类型对象的状态会完全复制吗？可以在 Main() 中调用看看：
```C#
static void ValueTypeContainingRefType()
{
    // Create the first Rectangle.
    Console.WriteLine("-> Creating r1");
    Rectangle r1 = new Rectangle("First Rect", 10, 10, 50, 50);
    // Now assign a new Rectangle to r1.
    Console.WriteLine("-> Assigning r2 to r1");
    Rectangle r2 = r1;
    // Change some values of r2.
    Console.WriteLine("-> Changing values of r2");
    r2.RectInfo.InfoString = "This is new info!";
    r2.RectBottom = 4444;
    // Print values of both rectangles.
    r1.Display();
    r2.Display();
}
```
输出结果：
```
-> Creating r1
-> Assigning r2 to r1
-> Changing values of r2
String = This is new info!, Top = 10, Bottom = 50, Left = 10, Right = 50
String = This is new info!, Top = 10, Bottom = 4444, Left = 10, Right = 50
```
如你所见，r1/r2显示相同值。默认情况下，值类型包含引用类型时，赋值会得到引用的副本。本例中，有两个独立结构，每个结构都包含了指向内存中相同的对象。如果想执行深拷贝，要内部引用状态完全复制给另一个新对象，唯一的办法是实现 ICloneable 接口。

#### 按值类型传参引用类型
引用类型或值类型都可以传参给方法，但二者有所不同。
```C#
class Person
{
    public string personName;
    public int personAge;
    // Constructors.
    public Person(string name, int age)
    {
        personName = name;
        personAge = age;
    }
    public Person(){}
    public void Display()
    {
        Console.WriteLine("Name: {0}, Age: {1}", personName, personAge);
    }
}
static void SendAPersonByValue(Person p)
{
    // Change the age of "p"?
    p.personAge = 99;
    // Will the caller see this reassignment?
    p = new Person("Nikki", 99);
}
```
上面调用Person对象是按值传递的（注意没有象out或ref这样的参数修饰符）。现在在Main()方法中测试一下：
```C#
static void Main(string[] args)
{
    // Passing ref-types by value.
    Console.WriteLine("***** Passing Person object by value *****");
    Person fred = new Person("Fred", 12);
    Console.WriteLine("\nBefore by value call, Person is:");
    fred.Display();
    SendAPersonByValue(fred);
    Console.WriteLine("\nAfter by value call, Person is:");
    fred.Display();
    Console.ReadLine();
}
```
返回如下结果：
```
***** Passing Person object by value *****
Before by value call, Person is:
Name: Fred, Age: 12
After by value call, Person is:
Name: Fred, Age: 99
```

#### 按引用类型传递引用类型
现改用 SendAPersonByReference() 方法，按引用类型传递。（注意用了ref 参数修饰符）
```C#
static void SendAPersonByReference(ref Person p)
{
    // Change some data of "p".
    p.personAge = 555;
    // "p" is now pointing to a new object on the heap!
    p = new Person("Nikki", 999);
}
static void Main(string[] args)
{
    // Passing ref-types by ref.
    Console.WriteLine("***** Passing Person object by reference *****");
    ...
    Person mel = new Person("Mel", 23);
    Console.WriteLine("Before by ref call, Person is:");
    mel.Display();
    SendAPersonByReference(ref mel);
    Console.WriteLine("After by ref call, Person is:");
    mel.Display();
    Console.ReadLine();
}
```
返回：
```
***** Passing Person object by reference *****
Before by ref call, Person is:
Name: Mel, Age: 23
After by ref call, Person is:
Name: Nikki, Age: 999
```
如你所见Mel更改了传递给它的变量。总结以上示例，牢记两条黄金规则：
1. 如果引用类型是通过引用传递的，则被调用者可能会更改对象状态数据的值以及它所引用的对象的值。
2. 如果引用类型是通过值传递的，则被调用者可能会更改对象的状态数据的值，但不会更改它所引用的对象的值。


### C# 可空类型(Nullable)
值类型不可能赋值 null。null常用于创建一个空对象引用。
```C#
static void Main(string[] args)
{
    // Compiler errors!
    // Value types cannot be set to null!
    bool myBool = null;
    int myInt = null;
    // OK! Strings are reference types.
    string myString = null;
}
```
C#支持可空数据类型的概念。简言之，null类型可表示基础类型的所有值，再加null值。因此声明可空的bool，可以赋值的集合{true,false,null}，在关系型数据库中很管用，库表中遇到未定义列的情况很常见。
定义可空变量类型，要在基础数据类型后加“?”后缀。该语法仅用于值类型时是合法的。如果想创建一个可空的引用类型（含string）,会遇到编译时错误。与非空型变量类似，局部可空变量在使用前必须赋初始值。
```C#
static void LocalNullableVariables()
{
    // Define some local nullable variables.
    int? nullableInt = 10;
    double? nullableDouble = 3.14;
    bool? nullableBool = null;
    char? nullableChar = 'a';
    int?[] arrayOfNullableInts = new int?[10];
    // Error! Strings are reference types!
    // string? s = "oops";
}
```

C# 的“?”后缀是创建泛型 System.Nullable<T>结构类型的快捷方法。

判断一个可空变量实际赋值是否是null，可用 HasValue 属性或`!=`操作符。

#### 使用 Nullable 类型










































