## go基础

    https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/07.3.md

    https://github.com/hyper0x/go_command_tutorial


### 关键字

Go 的源文件以 .go 为后缀名。

文件名默认以小写组成，如果文件名由多个部分组成，则以下划线`_` 进行分割。文件名不能包含空格或其他特殊字符。

Go 语言区分大小写。

标识符以字符(任何UTF-8编码的字符 或 `_`) 开头，然后紧跟0个或多个字符或Unicode 数字。

`_` 本身就是一个特殊的标识符，被称为空白标识符。

即任何赋值给这个标识符的值都将会被丢弃，该字符可以作为赋值的占位符。


Go 使用到的25个 关键字或保留字

    break       default     func    interface       select
    case        defer       go      map     struct
    chan        else        goto        package     switch  
    const       fallthrough     if      range       type
    continue    for     import      return      var


Go 为了简化在编译过程第一步中的代码解析，刻意减少关键字，比如：while 。

Go 语言还有 36个预定义标识符，作为基本类型或内置函数使用。

    append      bool        byte    cap     close   complex
    complex64       complex128      uint64      copy    false   float32
    float64     imag    int     int8    int16   uint32
    int32       int64       iota        len     make        new
    nil     panic       uint64      print       println     real
    recover     string      true        uint    uint8   uinptr



### 包的概念


包是结构化代码的一种方式：每个程序都由包（通常简称为 pkg）的概念组成，可以使用自身的包或者从其它包中导入内容。


Go 源码必须在源文件中非注释的第一行指明这个文件属于哪个包。

命名包 ，语法：

    package 包名

    
`注意：所有的包名都应该使用小写字母。`

main包为特殊的包， package main表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 main 的包。

标准库

在 Go 的安装文件里包含了一些可以直接使用的包，即标准库。
在 Linux 下，标准库在 Go 根目录下的子目录 pkg\linux_amd64 中,即一般情况下，标准包会存放在 $GOROOT/pkg/$GOOS_$GOARCH/ 目录下。

Go 的标准库包含了大量的包（如：fmt 和 os），但是你也可以创建自己的包。

如果想要构建一个程序，则包和包内的文件都必须以正确的顺序进行编译。

包的依赖关系决定了其构建顺序。

属于同一个包的源文件必须全部被一起编译，一个包即是编译时的一个单元。
因此根据惯例，每个目录都只包含一个包。

如果对一个包进行更改或重新编译，所有引用了这个包的客户端程序都必须全部重新编译。

Go 中的包模型采用了显式依赖关系的机制来达到快速编译的目的，编译器会从后缀名为 .o 的对象文件（需要且只需要这个文件）中提取传递依赖类型的信息。

如果 A.go 依赖 B.go，而 B.go 又依赖 C.go ：

    编译 C.go, B.go, 然后是 A.go.
    为了编译 A.go, 编译器读取的是 B.o 而不是 C.o.


使用包， 语法：

    import “包名”

    // 或引入多个包
    import (
       "包名"
       "包名"
        // ...
     )

    // 或者使用包的别名

    import fm "fmt"     // 即 fm 为包fmt 的别名。


可以使用GOPATH环境变量，或是使用相对路径来import你自己的package。

引入包的规则：
1. 在import中，你可以使用相对路径，如 ./或 ../ 来引用你的package

2. 如果没有使用相对路径，那么，go会去找$GOPATH/src/目录。


包的可见性规则：

    以标识符的首字母大小写为区分。大写字母为导出，即可见；小写为不可见。

当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；

标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 private ）。





### 关于分号

和C 一样，Go的正式语法使用分号来终止语句。
`和 C不同的是，这些分号由词法分析器在扫描源代码过程中使用简单的规则自动插入分号，因此输入源代码多数时候就不需要分号。`

规则是这样的：

    如果在一个新行前方的最后一个标记是一个标识符（包括像int和float64这样的单词）、一个基本的如数值这样的文字、或以下标记中的一个时，会自动插入分号

无论任何时候，你都不应该将一个控制结构（(if、for、switch或select）的左大括号放在下一行。

如果这样做，将会在大括号的前方插入一个分号，这可能导致出现不想要的结果。



### 常量（const）、变量（var）和类型（type）

类型可以是基本类型，如：int、float、bool、string；

结构化的（复合的），如：struct、array、slice、map、channel；

只描述类型的行为的，如：interface。


`常量 const`

常量使用关键字 const 定义，用于存储不会改变的数据。

存储在常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。
    
    常量的定义格式：  const identifier [type] = value
    省略类型时，由编译器推导类型。


常量的值必须是能够在编译时就能够确定的。
任何自定义函数不能对常量赋值，因为在编译期间自定义函数均属于未知。


数字型的常量是没有大小和符号的，并且可以使用任何精度而不会导致溢出。

不过需要注意的是，当常量赋值给一个精度过小的数字型变量时，可能会因为无法正确表达常量所代表的数值而导致溢出，这会在编译期间就引发错误。

常量也允许使用并行赋值的形式：
    
    const Pi = 3.14159

    const beef, two, c = "eat", 2, "veg"

    const (
	   Monday, Tuesday, Wednesday = 1, 2, 3
	   Thursday, Friday, Saturday = 4, 5, 6
    )

    const (
    	Unknown = 0
    	Female = 1
    	Male = 2
    )


`变量`

    声明变量的一般形式是使用 var 关键字：var identifier type。

或批量声明：
        
    var (
    	a int
    	b bool
    	str string
    )

    // 等同下面
    var a int
    var b bool
    var str string

`Go 和许多编程语言不同，它在声明变量时将变量的类型放在变量的名称之后。`


    当一个变量被声明之后，系统自动赋予它该类型的零值：

    int 为 0，
    float 为 0.0，
    bool 为 false，
    string 为空字符串，
    指针为 nil。

    记住，所有的内存在 Go 中都是经过初始化的。


变量可以重复声明：
```
for i := 0; i < 5; i++ {
	var v int
	fmt.Printf("%d ", v)
	v = 5
}
// 输出 0 0 0 0 0
```

变量的命名规则遵循骆驼命名法，即首个单词小写，每个新单词的首字母大写，例如：numShips 和 startDate。

一个变量（常量、类型或函数）在程序中都有一定的作用范围，称之为作用域。

在函数体内声明的变量称之为局部变量，它们的作用域只在函数体内，参数和返回值变量也是局部变量。

一般情况下，局部变量的作用域可以通过代码块（用大括号括起来的部分）判断。

一个变量在函数体外声明，则被认为是全局变量，可以在整个包甚至外部包（被导出后）使用。


变量可以编译期间就被赋值，赋值给变量使用运算符等号 =，当然你也可以在运行时对变量进行赋值操作。

Go 编译器在编译时可以根据变量的值来自动推断其类型。
需要特殊数据类型时，必须指定变量类型。


可以使用下面的这些形式来声明及初始化变量：

    // 缺省类型声明，编译期间推导类型
    var a = 15
    var b = false
    var str = "Go says hello to the world!"

变量的类型也可以在运行时实现自动推断，主要用于声明包级别的全局变量。 
如：
    
    var (
        	HOME = os.Getenv("HOME")
        	USER = os.Getenv("USER")
        	GOROOT = os.Getenv("GOROOT")
    )

在函数体内声明局部变量时，应使用简短声明语法  :=， 可以省略var关键字。

`简短形式，使用 := 赋值操作符`

    
    a, b, c := 5, 7, "abc"

右边的这些值以相同的顺序赋值给左边的变量，所以 a 的值是 5， b 的值是 7，c 的值是 "abc"。

这被称为 `并行` 或 同时 `赋值`。

交换两个变量的值，则可以简单地使用 a, b = b, a。

空白标识符 _ 也被用于抛弃值，如值 5 在：_, b = 5, 7 中被抛弃。


`值类型和引用类型`

像 int、float、bool 和 string 这些基本类型都属于值类型，使用这些类型的变量直接指向存在内存中的值

在 Go 语言中，指针属于引用类型，其它的引用类型还包括 slices，maps和 channel。

被引用的变量会存储在堆中，以便进行垃圾回收，且比栈拥有更大的内存空间。



### 基本类型和运算符

Go 是强类型语言，因此不会进行隐式转换，任何不同类型之间的转换都必须显式说明。

`布尔类型 bool`

布尔型的值只可以是常量 true 或者 false。
一个简单的例子：

    var b bool = true


Go 对于值之间的比较有非常严格的限制，只有两个类型相同的值才可以进行比较。


`数字类型`

整型 int 和浮点型 float

Go 语言支持整型和浮点型数字，并且原生支持复数，其中位的运算采用补码。

Go 也有基于架构的类型，例如：int、uint 和 uintptr。

+ int 和 uint 在 32 位操作系统上，它们均使用 32 位（4 个字节），在 64 位操作系统上，它们均使用 64 位（8 个字节）。

+ uintptr 的长度被设定为足够存放一个指针即可。


Go 语言中没有 double 类型。

与操作系统架构无关的类型都有固定的大小，并在类型的名称中就可以看出来：

整数：

    int8（-128 -> 127）
    int16（-32768 -> 32767）
    int32（-2,147,483,648 -> 2,147,483,647）
    int64（-9,223,372,036,854,775,808 -> 9,223,372,036,854,775,807）

无符号整数：

    uint8（0 -> 255）
    uint16（0 -> 65,535）
    uint32（0 -> 4,294,967,295）
    uint64（0 -> 18,446,744,073,709,551,615）


浮点型（IEEE-754 标准）：

    float32（+- 1e-45 -> +- 3.4 * 1e38）
    float64（+- 5 * 1e-324 -> 107 * 1e308）
    
    float32 精确到小数点后 7 位，float64 精确到小数点后 15 位。

int 型是计算最快的一种类型。

整型的零值为 0，浮点型的零值为 0.0。

可以通过增加前缀 0 来表示 8 进制数（如：077），增加前缀 0x 来表示 16 进制数（如：0xFF），以及使用 e 来表示 10 的连乘（如： 1e3 = 1000）。


Go 中不允许不同类型之间的混合使用，严格限制。

格式化说明符

    在格式化字符串里，%d 用于格式化整数（%x 和 %X 用于格式化 16 进制表示的数字），
    %g 用于格式化浮点型（%f 输出浮点数，%e 输出科学计数表示法），
    %0d 用于规定输出定长的整数，其中开头的数字 0 是必须的。

    %n.mg 用于表示数字 n 并精确到小数点后 m 位，除了使用 g 之外，还可以使用 e 或者 f，
    例如：使用格式化字符串 %5.2e 来输出 3.4 的结果为 3.40e+00。


数字值转换

例子展示如何安全地从 int 型转换为 int8：
```
func Uint8FromInt(n int) (uint8, error) {
	if 0 <= n && n <= math.MaxUint8 { // conversion is safe
		return uint8(n), nil
	}
	return 0, fmt.Errorf("%d is out of the uint8 range", n)
}
```

不过如果你实际存的数字超出你要转换到的类型的取值范围的话，则会引发 panic。


复数

    complex64 (32 位实数和虚数)
    complex128 (64 位实数和虚数)

复数使用 re+imI 来表示，其中 re 代表实数部分，im 代表虚数部分，I 代表根号负 1。
```
var c1 complex64 = 5 + 10i
fmt.Printf("The value is: %v", c1)
// 输出： 5 + 10i
```
函数 `real(c)` 和 `imag(c)` 可以分别获得相应的实数和虚数部分。

位运算

位运算只能用于整数类型的变量，且需当它们拥有等长位模式时。

`%b` 是用于表示位的格式化标识符。

在通讯中使用位左移表示标识的用例
```
type BitFlag int
const (
	Active BitFlag = 1 << iota // 1 << 0 == 1
	Send // 1 << 1 == 2
	Receive // 1 << 2 == 4
)

flag := Active | Send // == 3
```


运算符与优先级

下表列出了所有运算符以及它们的优先级，由上至下代表优先级由高到低:

    优先级 	运算符
     7 		^ !
     6 		* / % << >> & &^
     5 		+ - | ^
     4 		== != < <= >= >
     3 		<-
     2 		&&
     1 		||



类型别名

在 type TZ int 中，TZ 就是 int 类型的新名称


字符类型

严格来说，这并不是 Go 语言的一个类型，字符只是整数的特殊用例。

byte 类型是 uint8 的别名，对于只占用 1 个字节的传统 ASCII 编码的字符来说，完全没有问题。

字符使用单引号括起来。

Go支持 Unicode（UTF-8），因此字符同样称为 Unicode 代码点或者 runes，并在内存中使用 int 来表示。
在文档中，一般使用格式 U+hhhh 来表示，其中 h 表示一个 16 进制数。

rune 也是 Go 当中的一个类型，并且是 int32 的别名。

格式化说明符 %c 用于表示字符；
当和字符配合使用时，%v 或 %d 会输出用于表示该字符的整数；%U 输出格式为 U+hhhh 的字符串

包 unicode 包含了一些针对测试字符的非常有用的函数

    判断是否为字母：unicode.IsLetter(ch)
    判断是否为数字：unicode.IsDigit(ch)
    判断是否为空白符号：unicode.IsSpace(ch)


### 字符串

字符串是 UTF-8 字符的一个序列

字符串是一种值类型，且值不可变，即创建某个文本后你无法再次修改这个文本的内容；

更深入地讲，字符串是字节的定长数组。

ASCII 编码的字符占用 1 个字节，既每个索引都指向不同的字符，而非 ASCII 编码的字符（占有 2 到 4 个字节）不能单纯地使用索引来判断是否为同一个字符

Go 支持以下 2 种形式的字面值：

+ 解释字符串

    该类字符串使用双引号括起来，其中的相关的转义字符将被替换

        \n：换行符
        \r：回车符
        \t：tab 键
        \u 或 \U：Unicode 字符
        \\：反斜杠自身

+ 非解释字符串

    该类字符串使用反引号括起来，支持换行
    
         `This is a raw string \n` 中的 `\n\` 会被原样输出。


string 类型的零值为长度为零的字符串，即空字符串 ""。

字符串的内容（纯字节）可以通过标准索引法来获取，在中括号 [] 内写入索引，索引从 0 开始计数。

字符串拼接符 +


字符串关联包： `strings 和 strconv 包`

前缀和后缀

    strings.HasPrefix(s, prefix string) bool
    strings.HasSuffix(s, suffix string) bool

字符串包含关系

    strings.Contains(s, substr string) bool

判断子字符串或字符在父字符串中出现的位置

    // -1 表示字符串 s 不包含字符串 str
    strings.Index(s, str string) int
    

    // 最后出现
    strings.LastIndex(s, str string) int

    // ch 是非 ASCII 编码的字符，建议使用以下函数来对字符进行定位
    strings.IndexRune(s string, r rune) int

字符串替换

    // 如果 n = -1 则替换所有字符串 old 为字符串 new
    strings.Replace(str, old, new, n) string

统计字符串出现次数
    
    strings.Count(s, str string) int

重复字符串

    strings.Repeat(s, count int) string

修改字符串大小写

    strings.ToLower(s) string
    strings.ToUpper(s) string

修剪字符串

    // 剔除字符串开头和结尾的空白符号
    strings.TrimSpace(s) 

    // 将开头和结尾的 cut 去除掉
    strings.Trim(s, "cut") 

    TrimLeft 或者 TrimRight 来实现

分割字符串

    // 用于自定义分割符号来对指定字符串进行分割，同样返回 slice
    strings.Split(s, sep) 

    // 将会利用 1 个或多个空白符号来作为动态长度的分隔符将字符串分割成若干小块，并返回一个 slice
    strings.Fields(s) 

拼接 slice 到字符串

    strings.Join(sl []string, sep string) string

从字符串中读取内容

    函数 strings.NewReader(str) 用于生成一个 Reader 并读取字符串中的内容，然后返回指向该 Reader 的指针，从其它类型读取内容的函数还有

    Read() 从 []byte 中读取内容
    ReadByte() 和 ReadRune() 从字符串中读取下一个 byte 或者 rune


字符串与其它类型的转换

与字符串相关的类型转换都是通过 strconv 包实现的。

任何类型 T 转换为字符串总是成功的。

从数字类型转换到字符串，Go 提供了以下函数

从数字类型转换到字符串，Go 提供了以下函数:

    // 返回数字 i 所表示的字符串类型的十进制数
    strconv.Itoa(i int) string 

    // 将 64 位浮点型的数字转换为字符串
    strconv.FormatFloat(f float64, fmt byte, prec int, bitSize int) string

将字符串转换为其它类型 tp 并不总是可能的，可能会在运行时抛出错误 。

从字符串类型转换为数字类型，Go 提供了以下函数

    strconv.Atoi(s string) (i int, err error) 将字符串转换为 int 型。

    trconv.ParseFloat(s string, bitSize int) (f float64, err error) 将字符串转换为 float64 型。



### 时间和日期

time 包为我们提供了一个数据类型 time.Time（作为值使用）以及显示和测量时间和日期的功能函数。


当前时间可以使用 ` time.Now()` 获取

`t.Day()`、`t.Minute()` 等等来获取时间的一部分

Duration 类型表示两个连续时刻所相差的纳秒数，类型为 int64.

Location 类型映射某个时区的时间，UTC 表示通用协调世界时间。




### 指针

Go 语言为程序员提供了控制数据结构的指针的能力。

`但不能进行指针运算`

使用`*` 声明指针，指针变量名和 星号必须保持空格，避免为乘积

    var intP *int


Go 语言的取地址符是 &，放到一个变量前使用就会返回相应变量的内存地址。





### 控制结构

Go 提供了下面这些条件结构和分支结构：

    if-else 结构
    switch 结构
    select 结构，用于 channel 的选择

    使用迭代或循环结构来重复执行一次或多次某段代码

    for (range) 结构

    一些如 break 和 continue 这样的关键字可以用于中途改变循环的状态。

Go 完全省略了 if、switch 和 for 结构中条件语句两侧的括号。

#### if

`关键字 if 和 else 之后的左大括号 { 必须和关键字在同一行，如果你使用了 else-if 结构，则前段代码块的右大括号 } 必须和 else-if 关键字在同一行。这两条规则都是被编译器强制规定的。`

```
// 条件语句没有括号，且做大括号和条件语句同行

if condition {
	// do something	
} else {
	// do something	
}

```

判断一个字符串是否为空

    if str == "" { ... }
    if len(str) == 0 {...}

判断运行 Go 程序的操作系统类型
```
if runtime.GOOS == "windows"	 {
 	.	..
 } else { // Unix-like
 	.	..
 }
```

if 可以包含一个初始化语句
```
if initialization; condition {
	// do something
}

val := 10
if val > max {
	// do something
}

// 简短方式 := 声明的变量的作用域只存在于 if 结构中
if val := 10; val > max {
	// do something
}
```

#### switch 

Go 语言中的 switch 结构使用上一旦成功地匹配到某个分支，在执行完相应代码后就会退出整个 switch 代码块，也就是说您不需要特别使用 break 语句来表示结束。

`case 默认break`。

case 的条件可以接受任意形式的表达式，而不仅仅局限字符串或整数。

任何支持进行相等判断的类型都可以作为测试表达式的条件，包括 int、string、指针等。

测试多个可能符合条件的值，使用逗号分割它们，例如：case val1, val2, val3。

如果在执行完每个分支的代码后，还希望继续执行后续分支的代码，可以使用 fallthrough 关键字来达到目的。

```
switch i {
	case 0: fallthrough
	case 1:
		f() // 当 i == 0 时函数也会被调用
}

```

可选的 default 分支可以出现在任何顺序，但最好将它放在最后。



#### for 

基于计数器的迭代

```
for 初始化语句; 条件语句; 修饰语句 {}
```

基于条件判断的迭代

```
// 类似其它语言中的 while 循环
for 条件语句 {}
```

无限循环

```
for { } 

// ;; 会在使用 gofmt 时被移除
for ;; { }
```

for-range 结构

一般形式为：
```
for ix, val := range coll { }
```
它可以迭代任何一个集合（包括数组和 map）。
语法上很类似其它语言中 foreach 语句，但您依旧可以获得每次迭代所对应的索引。

val 始终为集合中对应索引的值拷贝，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值
如果 val 为指针，则会产生指针的拷贝，依旧可以修改集合中的原值。

```
    str := "Go is a beautiful language!"
	for pos, char := range str {
		fmt.Printf("Character on position %d is: %c \n", pos, char)
	}
	fmt.Println()
	
	str2 := "Chinese: 日本語"

	for pos, char := range str2 {
    	fmt.Printf("character %c starts at byte position %d\n", char, pos)
	}
	
```

for 可以使用break 与 continue 跳出循环。


#### 标签与 goto

for、switch 或 select 语句都可以配合标签（label）形式的标识符使用，即某一行第一个以冒号（:）结尾的单词（gofmt 会将后续代码自动移至下一行）。

标签的名称是大小写敏感的，为了提升可读性，一般建议使用全部大写字母。

定义但未使用标签会导致编译错误：label … defined and not used

注意标签和 goto 语句之间不能出现定义新变量的语句，否则会导致编译失败。






### 函数


Go是编译型语言，所以函数编写的顺序是无关紧要的；


Go 里面有三种类型的函数：

    普通的带有名字的函数
    匿名函数或者lambda函数
    方法（Methods ）


除了main()、init()函数外，其它所有类型的函数都可以有参数与返回值。

函数参数、返回值以及它们的类型被统称为函数签名。

在 Go 里面函数重载是不被允许的。这将导致一个编译错误。




`init 函数`

定义在包中的init 函数会在每个包完成初始化后自动执行，并且执行优先级比 main 函数高。


如果需要申明一个在外部定义的函数，你只需要给出函数名与函数签名，不需要给出函数体：

    func flushICache(begin, end uintptr) // implemented externally


函数也可以以申明的方式被使用，作为一个函数类型

    type binOp func(int, int) int



函数值（functions value）之间可以相互比较：如果它们引用的是相同的函数或者都是 nil 的话，则认为它们是相同的函数。

函数不能在其它函数里面声明（不能嵌套），不过我们可以通过使用匿名函数来破除这个限制。

目前 Go 没有泛型（generic）的概念，也就是说它不支持那种支持多种类型的函数。

没有参数的函数通常被称为 niladic 函数（niladic function），就像 main.main()。



#### 按值传递（call by value） 按引用传递（call by reference）

Go 默认使用按值传递来传递参数，也就是传递参数的副本。

如果你希望函数可以直接修改参数的值，而不是对参数的副本进行操作，你需要将参数的地址（变量名前面添加&符号，比如 &variable）传递给函数，这就是按引用传递。
此时传递给函数的是一个指针。如果传递给函数的是一个指针，指针的值（一个地址）会被复制，但指针的值所指向的地址上的值不会被复制；我们可以通过这个指针的值来修改这个值所指向的地址上的值。


在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显式的指出指针）。

#### 命名的返回值

有些函数只是完成一个任务，并没有返回值。

但是绝大部分的函数还是带有返回值的。

命名返回值作为结果形参（result parameters）被初始化为相应类型的零值，当需要返回的时候，我们只需要一条简单的不带参数的return语句。

```
func getX2AndX3_2(input int) (x2 int, x3 int) {
    x2 = 2 * input
    x3 = 3 * input
    
    // return x2, x3
    // 正常返回 x2, x3    
    return
}
```


#### 传递变长参数

函数的最后一个参数是采用 ...type 的形式，那么这个函数就可以处理一个变长的参数，这个长度可以为 0，这样的函数称为变参函数。

```
func myFunc(a, b, arg ...int) {}
```
这个函数接受一个类似某个类型的 slice 的参数

示例函数和调用
```
func Greeting(prefix string, who ...string)
Greeting("hello:", "Joe", "Anna", "Eileen")
```
在 Greeting 函数中，变量 who 的值为 []string{"Joe", "Anna", "Eileen"}。


如果变长参数的类型并不是都相同的呢？使用 5 个参数来进行传递并不是很明智的选择，有 2 种方案可以解决这个问题
+ 使用结构
    
        定义一个结构类型，用以存储所有可能的参数

+ 使用空接口
        
     如果一个变长参数的类型没有被指定，则可以使用默认的空接口 interface{}，这样就可以接受任何类型的参数


#### defer 和追踪

关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 return 语句之后）一刻才执行某个语句或函数。

关键字 defer 的用法类似于面向对象编程语言 Java 和 C# 的 finally 语句块，它一般用于释放某些已分配的资源。

使用 defer 的语句同样可以接受参数。

当有多个 defer 行为被注册时，它们会以逆序执行（类似栈，即后进先出）

```
func f() {
	for i := 0; i < 5; i++ {
		defer fmt.Printf("%d ", i)
	}
}

```
上面的代码将会输出：4 3 2 1 0。

关键字 defer 允许我们进行一些函数执行完成后的收尾工作，例如:
1. 关闭文件流
2. 解锁一个加锁的资源
3. 打印最终报告
4. 关闭数据库链接


使用 defer 语句来记录函数的参数与返回值:

```

package main

import (
	"io"
	"log"
)

func func1(s string) (n int, err error) {
	defer func() {
		log.Printf("func1(%q) = %d, %v", s, n, err)
	}()
	return 7, io.EOF
}

func main() {
	func1("Go")
}

```


#### 内置函数

Go 语言拥有一些不需要进行导入操作就可以使用的内置函数。

| 名称 |	说明 |
|:----:|:----:|
| close	|    用于管道通信 |
| len、cap	|   len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map）|
|new、make	|   new 和 make 均是用于分配内存：new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针。它也可以被用于基本类型：v := new(int)。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作 new() 是一个函数，不要忘记它的括号|
|copy、append |	  用于复制和连接切片|
|panic、recover	|  两者均用于错误处理机制|
|print、println 	|     底层打印函数，在部署环境中建议使用 fmt 包|
|complex、real imag |	   用于创建和操作复数|



#### 递归函数

当一个函数在其函数体内调用自身，则称之为递归。


Go 语言中也可以使用相互调用的递归函数：多个函数之间相互调用形成闭环。因为 Go 语言编译器的特殊性，这些函数的声明顺序可以是任意的。

```
package main

import (
	"fmt"
)

func main() {
	fmt.Printf("%d is even: is %t\n", 16, even(16)) // 16 is even: is true
	fmt.Printf("%d is odd: is %t\n", 17, odd(17))
	// 17 is odd: is true
	fmt.Printf("%d is odd: is %t\n", 18, odd(18))
	// 18 is odd: is false
}

func even(nr int) bool {
	if nr == 0 {
		return true
	}
	return odd(RevSign(nr) - 1)
}

func odd(nr int) bool {
	if nr == 0 {
		return false
	}
	return even(RevSign(nr) - 1)
}

func RevSign(nr int) int {
	if nr < 0 {
		return -nr
	}
	return nr
}
```

#### 将函数作为参数

函数可以作为其它函数的参数进行传递，然后在其它函数内调用执行，一般称之为回调。


#### 闭包

当我们不希望给函数起名字的时候，可以使用匿名函数，例如：`func(x, y int) int { return x + y }`。


defer 语句和匿名函数

关键字 defer 经常配合匿名函数使用，它可以用于改变函数的命名返回值。

匿名函数同样被称之为闭包（函数式语言的术语）：它们被允许调用定义在其它环境下的变量。


一个返回值为另一个函数的函数可以被称之为工厂函数，这在您需要创建一系列相似的函数的时候非常有用：书写一个工厂函数而不是针对每种情况都书写一个函数。

下面的函数演示了如何动态返回追加后缀的函数：

```
func MakeAddSuffix(suffix string) func(string) string {
	return func(name string) string {
		if !strings.HasSuffix(name, suffix) {
			return name + suffix
		}
		return name
	}
}

addBmp := MakeAddSuffix(“.bmp”)
addJpeg := MakeAddSuffix(“.jpeg”)

addBmp("file") // returns: file.bmp
addJpeg("file") // returns: file.jpeg
```

#### 多返回值函数的错误

Go 语言的函数经常使用两个返回值来表示执行是否成功：
返回某个值以及 true 表示成功；返回零值（或 nil）和 false 表示失败。

忽略了相关的错误检查:
```
anInt, _ = strconv.Atoi(origStr)
```

错误检查：
```
anInt, err = strconv.Atoi(origStr)

if err != nil {
    fmt.Printf(" error\n")
}
```



### 数组与切片

容器, 它是可以包含大量条目（item）的数据结构, 例如数组、切片和 map。

数组是具有相同 唯一类型 的一组已编号且长度固定的数据项序列。

数组关键词有： 长度， 类型，序列,如 ` [5]int{1,2,3,4,5} `。

数组长度必须是一个常量表达式，并且必须是一个非负整数。数组长度也是数组类型的一部分。

[5]int和[10]int是属于不同类型的。

数组元素可以通过 索引（位置）来读取（或者修改），索引从 0 开始，第一个元素索引为 0，第二个索引为 1，以此类推。

声明的格式是

    var identifier [len]type


比如：
```
var arr1 [5]int

// 每个元素是一个整形值，当声明数组时所有的元素都会被自动初始化为默认值 0
// arr1 的长度是 5，索引范围从 0 到 len(arr1)-1。
```


由于索引的存在，遍历数组的方法自然就是使用 for 结构:
```
for i:=0; i < len(arr1); i++ {
		arr1[i] = i * 2
	}
```

或使用 for-range 的生成方式：
```
for i,_:= range arr1 {
    // ...
}
```

Go 语言中的数组是一种 值类型（不像 C/C++ 中是指向首元素的指针），所以可以通过 new() 来创建：` var arr1 = new([5]int)`。

那么这种方式和 `var arr2 [5]int` 的区别是什么呢？
arr1 的类型是 *[5]int，而 arr2的类型是 [5]int。

这样的结果就是当把一个数组赋值给另一个时，需要在做一次数组内存的拷贝操作。例如：
```
// arr1 解指针赋值给arr2
arr2 := *arr1
arr2[2] = 100

```

#### 数组常量 

数组常量赋值：

形式1：

```
var arrAge = [5]int{18, 20, 15, 22, 16}
```

形式2：`编译时确定长度`

```
var arrLazy = [...]int{5, 6, 7, 8, 22}

// 此时 ... 可以忽略，即
var arrLazy = []int{5, 6, 7, 8, 22}

```

形式3 ： `key: value`
```
var arrKeyValue = [5]string{3: "Chris", 4: "Ron"}

// 只有索引 3 和 4 被赋予实际的值，其他元素都被设置为空的字符串，所以输出结果为
```


#### 多维数组

数组通常是一维的，但是可以用来组装成多维数组。
例如：
```
[3][5]int，[2][2][2]float64。
```

内部数组总是长度相同的。Go 语言的多维数组是矩形式的。


#### 将数组传递给函数

把一个大数组传递给函数会消耗很多内存。有两种方法可以避免这种现象：

    传递数组的指针
    使用数组的切片

#### 切片


切片（slice）是对数组一个连续片段的引用（该数组我们称之为相关数组，通常是匿名的），所以切片是一个引用类型。

必须先有数组，才有数组引用类型 -- 切片。

切片是可索引的，并且可以由 len() 函数获取长度。

和数组不同的是，切片的长度可以在运行时修改，最小为 0 , 最大为相关数组的长度：`切片是一个 长度可变的数组`。


切片提供了计算容量的函数 cap() 可以测量切片最长可以达到多少：
它等于切片的长度 + 数组除切片之外的长度。
如果 s 是一个切片，cap(s) 就是从 s[0] 到数组末尾的数组长度。

切片的长度永远不会超过它的容量，所以对于 切片 s 来说该不等式永远成立：0 <= len(s) <= cap(s)。

`优点` 因为切片是引用，所以它们不需要使用额外的内存并且比使用数组更有效率，所以在 Go 代码中 切片比数组更常用。


    声明切片的格式是：
    var identifier []type（不需要说明长度）。

    切片的初始化格式是：
    var slice1 []type = arr1[start:end]。

    表示 slice1 是由数组 arr1 从 start 索引到 end-1 索引之间的元素构成的子集

    start 和 end 都为数组的下标。
    start 为开区间
    end 为闭区间
    长度为： end - start 

一个切片在未初始化之前默认为 nil，长度为 0。

默认对等写法：

    // slice1 就等于完整的 arr1 数组
    var slice1 []type = arr1[:] 
    // 等价 arr1[0:len(arr1)]
    // 等价 slice1 = &arr1
    
    // 一个由数字 1、2、3 组成的切片可以这么生成：
    s := [3]int{1,2,3}[:] 
    // 等价  s := []int{1,2,3}


    arr1[2:] 
    arr1[2:len(arr1)]
    //  相同，都包含了数组从第三个到最后的所有元素。

    arr1[:3] 
    arr1[0:3] 
    // 相同，包含了从第一个到第三个元素（不包括第三个）。

    
    
对于每一个切片（包括 string），以下状态总是成立的：

    s == s[:i] + s[i:] // i是一个整数且: 0 <= i <= len(s)
    len(s) < cap(s)


切片在内存中的组织方式实际上是一个有 3 个域的结构体：指向相关数组的指针，切片 长度以及切片容量。


绝对不要用指针指向 slice。切片本身已经是一个引用类型，所以它本身就是一个指针!!



#### 用 make() 创建一个切片

当切片不依赖已声明的数组时，使用make关键字创建切片。

```
var slice1 []type = make([]type, len)。

// 简写为下面， len 是数组的长度并且也是 slice 的初始长度。
slice1 := make([]type, len)


```

make方法原型：
```
// 其中 cap 是可选参数,表示容量。
func make([]T, len, cap)
```
下面两种方法可以生成相同的切片
    
    make([]int, 50, 100)
    new([100]int)[0:50]

#### new() 和 make() 的区别

二者都在堆上分配内存，但是它们的行为不同，适用于不同的类型。

+ new(T) 为每个新的类型T分配一片内存，初始化为 0 并且返回类型为*T的内存地址：这种方法 返回一个指向类型为 T，值为 0 的地址的指针，它适用于值类型如数组和结构体；它相当于 &T{}。

+ make(T) 返回一个类型为 T 的初始值，它只适用于3种内建的引用类型：切片、map 和 channel 。

new 函数分配内存，make 函数初始化。


#### 多维 切片

和数组一样，切片通常也是一维的，但是也可以由一维组合成高维。

通过分片的分片（或者切片的数组），长度可以任意动态变化，所以 Go 语言的多维切片可以任意切分。

而且，内层的切片必须单独分配（通过 make 函数）。


#### bytes 包

类型 []byte 的切片十分常见，Go 语言有一个 bytes 包专门用来解决这种类型的操作方法。

#### for-range 结构

这种构建方法可以应用与数组和切片:

```
for ix, value := range slice1 {
	...
}
```
第一个返回值 ix 是数组或者切片的索引，第二个是在该索引位置的值；


多维切片下的 for-range：

```
for row := range screen {
    //  screen[row]
	for column := range screen[row] {
		screen[row][column] = 1
	}
}

```


#### 切片重组（reslice）

改变切片长度的过程称之为切片重组 reslicing，做法如下：slice1 = slice1[0:end]，其中 end 是新的末尾索引（即长度）。


切片可以反复扩展直到占据整个相关数组。

```
package main
import "fmt"

func main() {
	slice1 := make([]int, 0, 10)
	
	// load the slice, cap(slice1) is 10:
	for i := 0; i < cap(slice1); i++ {
		slice1 = slice1[0:i+1]
		slice1[i] = i
		fmt.Printf("The length of slice is %d\n", len(slice1))
	}

}
```

#### 切片的复制与追加
如果想增加切片的容量，我们必须创建一个新的更大的切片并把原分片的内容都拷贝过来。

下面的代码描述了从拷贝切片的 copy 函数和向切片追加新元素的 append 函数。

```
package main
import "fmt"

func main() {
	sl_from := []int{1, 2, 3}
	sl_to := make([]int, 10)

	n := copy(sl_to, sl_from)
	fmt.Println(sl_to)
	fmt.Printf("Copied %d elements\n", n) // n == 3

	sl3 := []int{1, 2, 3}
	sl3 = append(sl3, 4, 5, 6)
	fmt.Println(sl3)
}
```
func copy(dst, src []T) int copy 方法将类型为 T 的切片从源地址 src 拷贝到目标地址 dst，覆盖 dst 的相关元素，并且返回拷贝的元素个数。

如果你还想继续使用 src，在拷贝结束后执行 src = dst。


func append(s[]T, x ...T) []T 其中 append 方法将 0 个或多个具有相同类型 s 的元素追加到切片后面并且返回新的切片；

追加的元素必须和原切片的元素同类型。

如果 s 的容量不足以存储新增元素，append 会分配新的切片来保证已有切片元素和新增元素的存储。

因此，返回的切片可能已经指向一个不同的相关数组了。

append 方法总是返回成功，除非系统内存耗尽了。
    

#### 字符串、数组和切片的应用

从字符串生成字节切片

假设 s 是一个字符串（本质上是一个字节数组），
那么就可以直接通过 c := []byte(s) 来获取一个字节的切片 c。

另外，您还可以通过 copy 函数来达到相同的目的：copy(dst []byte, src string)。

同样的，还可以使用 for-range 来获得每个元素

可以将字符串转换为元素类型为 rune 的切片：r := []rune(s)。

字符串和切片的内存结构

在内存中，一个字符串实际上是一个双字结构，即一个指向实际数据的指针和记录字符串长度的整数。

可以依旧把字符串看做是一个值类型，也就是一个字符数组。

Go 语言中的字符串是不可变的，也就是说 str[index] 这样的表达式是不可以被放在等号左侧的。

必须先将字符串转换成字节数组，然后再通过修改数组中的元素值来达到修改字符串的目的，最后将字节数组转换回字符串格式。

```
s := "hello"
c := []byte(s)
c[0] = ’c’
s2 := string(c) // s2 == "cello"
```

#### append 函数常见操作
    
    将切片 b 的元素追加到切片 a 之后：a = append(a, b...)

    复制切片 a 的元素到新的切片 b 上：
    
    b = make([]T, len(a))
    copy(b, a)
    删除位于索引 i 的元素：a = append(a[:i], a[i+1:]...)
    
    切除切片 a 中从索引 i 至 j 位置的元素：a = append(a[:i], a[j:]...)
    
    为切片 a 扩展 j 个元素长度：a = append(a, make([]T, j)...)
    
    在索引 i 的位置插入元素 x：a = append(a[:i], append([]T{x}, a[i:]...)...)
    
    在索引 i 的位置插入长度为 j 的新切片：a = append(a[:i], append(make([]T, j), a[i:]...)...)
    
    在索引 i 的位置插入切片 b 的所有元素：a = append(a[:i], append(b, a[i:]...)...)
    
    取出位于切片 a 最末尾的元素 x：x, a = a[len(a)-1], a[:len(a)-1]
    
    将元素 x 追加到切片 a：a = append(a, x)



#### 切片和垃圾回收

切片的底层指向一个数组，该数组的实际容量可能要大于切片所定义的容量。

只有在没有任何切片指向的时候，底层的数组内层才会被释放，这种特性有时会导致程序占用多余的内存。


### Map

map 是一种特殊的数据结构：一种元素对（pair）的无序集合，pair 的一个元素是 key，对应的另一个元素是 value。

ap 这种数据结构在其他编程语言中也称为字典（Python）、hash 和 HashTable 等。


map 是引用类型，可以使用如下声明：

    var map1 map[keytype]valuetype
    var map1 map[string]int


在声明的时候不需要知道 map 的长度，map 是可以动态增长的。

未初始化的 map 的值是 nil。

value 可以是任意类型的；

令 v := map1[key1] 可以将 key1 对应的值赋值为 v；
如果 map 中没有 key1 存在，那么 v 将被赋值为 map1 的值类型的空值。

声明和初始化
```
package main
import "fmt"

func main() {
	var mapLit map[string]int
	//var mapCreated map[string]float32
	var mapAssigned map[string]int

	mapLit = map[string]int{"one": 1, "two": 2}
	
   // 等价： mapCreated := map[string]float32{}
	mapCreated := make(map[string]float32)
	
	mapAssigned = mapLit

	mapCreated["key1"] = 4.5
	mapCreated["key2"] = 3.14159
	mapAssigned["two"] = 3

	fmt.Printf("Map literal at \"one\" is: %d\n", mapLit["one"])
	fmt.Printf("Map created at \"key2\" is: %f\n", mapCreated["key2"])
	fmt.Printf("Map assigned at \"two\" is: %d\n", mapLit["two"])
	fmt.Printf("Map literal at \"ten\" is: %d\n", mapLit["ten"])
}
```


map 容量

map 可以根据新增的 key-value 对动态的伸缩，因此它不存在固定长度或者最大限制。

但是你也可以选择标明 map 的初始容量 capacity：

    make(map[keytype]valuetype, cap)。

当 map 增长到容量上限的时候，如果再增加新的 key-value 对，map 的大小会自动加 1。

所以出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明。

#### 切片作为 map 的值

既然一个 key 只能对应一个 value，而 value 又是一个原始类型，那么如果一个 key 要对应多个值怎么办？

```
mp1 := make(map[int][]int)
mp2 := make(map[int]*[]int)
```

判断某个 key 是否存在而不关心它对应的值到底是多少

    // 如果key1存在则ok == true，否在ok为false
    _, ok := map1[key1] 

从 map1 中删除 key1
    
    // 如果 key1 不存在，该操作不会产生错误。
    delete(map1, key1)

#### for-range 的配套用法

可以使用 for 循环构造 map：

```
for key, value := range map1 {
	...
}

```

如果只想获取 key，你可以这么使用：
```
for key := range map1 {
	fmt.Printf("key is: %d\n", key)
}
```


#### map 类型的切片

获取一个 map 类型的切片，我们必须使用两次 make() 函数，第一次分配切片，第二次分配 切片中每个 map 元素。

```
package main
import "fmt"

func main() {
	// Version A:
	items := make([]map[int]int, 5)
	for i:= range items {
		items[i] = make(map[int]int, 1)
		items[i][1] = 2
	}
	fmt.Printf("Version A: Value of items: %v\n", items)

	// Version B: NOT GOOD!
	items2 := make([]map[int]int, 5)
	for _, item := range items2 {
		item = make(map[int]int, 1) // item is only a copy of the slice element.
		item[1] = 2 // This 'item' will be lost on the next iteration.
	}
	fmt.Printf("Version B: Value of items: %v\n", items2)
}
```

#### map 的排序

map 默认是无序的，不管是按照 key 还是按照 value 默认都不排序。

如果你想为 map 排序，需要将 key（或者 value）拷贝到一个切片，再对切片排序，然后可以使用切片的 for-range 方法打印出所有的 key 和 value。

```
// the telephone alphabet:
package main
import (
	"fmt"
	"sort"
)

var (
	barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
							"delta": 87, "echo": 56, "foxtrot": 12,
							"golf": 34, "hotel": 16, "indio": 87,
							"juliet": 65, "kili": 43, "lima": 98}
)

func main() {
	fmt.Println("unsorted:")
	for k, v := range barVal {
		fmt.Printf("Key: %v, Value: %v / ", k, v)
	}
	keys := make([]string, len(barVal))
	i := 0
	for k, _ := range barVal {
		keys[i] = k
	i++
	}
	sort.Strings(keys)
	fmt.Println()
	fmt.Println("sorted:")
	for _, k := range keys {
		fmt.Printf("Key: %v, Value: %v / ", k, barVal[k])
	}
}
```



### 包

像 fmt、os 等这样具有常用功能的内置包在 Go 语言中有 150 个以上，它们被称为标准库。

#### regexp 包

如果是简单模式，使用 Match 方法便可：
```
ok, _ := regexp.Match(pat, []byte(searchIn))
```

也可以使用 MatchString:
```
ok, _ := regexp.MatchString(pat, searchIn)
```

更多方法中，必须先将正则通过 Compile 方法返回一个 Regexp 对象。
```
    pat := "[0-9]+.[0-9]+" //正则
    re, _ := regexp.Compile(pat)
	//将匹配到的部分替换为"##.#"
	str := re.ReplaceAllString(searchIn, "##.#")
	fmt.Println(str)
```




#### 锁和 sync 包

在一些复杂的程序中，通常通过不同线程执行不同应用来实现程序的并发。

 map 类型是不存在锁的机制来实现这种效果(出于对性能的考虑)，所以 map 类型是非线程安全的。当并行访问一个共享的 map 类型的数据，map 数据将会出错。


在 Go 语言中这种锁的机制是通过 sync 包中 Mutex 来实现的。

sync.Mutex 是一个互斥锁，它的作用是守护在临界区入口来确保同一时间只能有一个线程进入临界区。


####  精密计算和 big 包

当对超出 int64 或者 uint64 类型这样的大数进行计算时，如果对精度没有要求，float32 或者 float64 可以胜任，但如果对精度有严格要求的时候，我们不能使用浮点数，在内存中它们只能被近似的表示。


#### 自定义包和可见性


导入外部安装包


如果你要在你的应用中使用一个或多个外部包，首先你必须使用 go install在你的本地机器上安装它们。

假设你想使用 ` http://codesite.ext/author/goExample/goex` 这种托管在 Google Code、GitHub 和 Launchpad 等代码网站上的包。

可以通过如下命令安装

    go install codesite.ext/author/goExample/goex

将一个名为 codesite.ext/author/goExample/goex 的 map 安装在 $GOROOT/src/ 目录下。

通过以下方式，一次性安装，并导入到你的代码中：

    import goex "codesite.ext/author/goExample/goex"



### 结构（struct）与方法（method）

Go 通过类型别名（alias types）和结构体的形式支持用户自定义类型，或者叫定制类型。


结构体是复合类型（composite types），当需要定义一个类型，它由一系列属性组成，每个属性都有自己的类型和值的时候，就应该使用结构体，它把数据聚集在一起。


组成结构体类型的那些数据称为 字段（fields）。每个字段都有一个类型和一个名字。


不过因为 Go 语言中没有类的概念，因此在 Go 中结构体有着更为重要的地位。


#### 结构体定义
结构体定义的一般方式如下：

```
type identifier struct {
    field1 type1
    field2 type2
    ...
}

// type T struct {a, b int} 也是合法的语法，它更适用于简单的结构体。
```

初始化

使用 new 函数给一个新的结构体变量分配内存，它返回指向已分配内存的指针

使用点号符可以获取结构体字段的值：structname.fieldname。

在 Go 语言中这叫 `选择器（selector）`。

无论变量是一个结构体类型还是一个结构体类型指针，都使用同样的 选择器符（selector-notation） 来引用结构体的字段：
```
type myStruct struct { i int }

var v myStruct    // v是结构体类型变量
var p *myStruct   // p是指向一个结构体类型变量的指针
v.i
p.i

```
初始化一个结构体实例（一个结构体字面量：struct-literal）的更简短和惯用的方式如下：
```
ms := &struct1{10, 15.5, "Chris"}
// 此时ms的类型是 *struct1

var ms struct1
ms = struct1{10, 15.5, "Chris"}

    
    
```

混合字面量语法（composite literal syntax）&struct1{a, b, c} 是一种简写，底层仍然会调用 new ()，这里值的顺序必须按照字段顺序来写。


    表达式 new(Type) 和 &Type{} 是等价的。


时间间隔（开始和结束时间以秒为单位）是使用结构体的一个典型例子：
```
type Interval struct {
    start int
    end   int
}

// 初始化方式
intr := Interval{0, 3}            (A)
intr := Interval{end:5, start:1}  (B)
intr := Interval{end:5}           (C)
```

结构体类型和字段的命名遵循可见性规则，一个导出的结构体类型中有些字段是导出的，另一些不是，这是可能的。


结构体的内存布局

Go 语言中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。


递归结构体

结构体类型可以通过引用自身来定义。

这在定义链表或二叉树的元素（通常叫节点）时特别有用，此时节点包含指向临近节点的链接（地址）。


结构体转换

Go 中的类型转换遵循严格的规则。
当为结构体定义了一个 alias 类型时，此结构体类型和它的 alias 类型都有相同的底层类型。

#### 结构体工厂

Go 语言不支持面向对象编程语言中那样的构造子方法。

为了方便通常会为类型定义一个工厂，按惯例，工厂的名字以 new 或 New 开头。

```
type File struct {
    fd      int     // 文件描述符
    name    string  // 文件名
}

func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}

// 调用

f := NewFile(10, "./test.txt")
```

在 Go 语言中常常像上面这样在工厂方法里使用初始化来简便的实现构造函数。

如果 File 是一个结构体类型，那么表达式 new(File) 和 &File{} 是等价的。


如果想知道结构体类型T的一个实例占用了多少内存，可以使用：`size := unsafe.Sizeof(T{})`。


如何强制使用工厂方法

通过应用可见性规则就可以禁止使用 new 函数，强制用户使用工厂方法，从而使类型变成私有的，就像在面向对象语言中那样。

```
type matrix struct {
    ...
}

func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}

// 在其他包里使用工厂方法
package main
import "matrix"
...
wrong := new(matrix.matrix)     // 编译失败（matrix 是私有的）
right := matrix.NewMatrix(...)  // 实例化 matrix 的唯一方式
```


#### map 和 struct vs new() 和 make()


使用 make() 的创建三种类型:

    slices  /  maps / channels

但不能make结构体。

下面的例子说明了在映射上使用 new 和 make 的区别以及可能发生的错误

```
package main

type Foo map[string]string
type Bar struct {
    thingOne string
    thingTwo int
}

func main() {
    // OK
    y := new(Bar)
    (*y).thingOne = "hello"
    (*y).thingTwo = 1

    // NOT OK
    z := make(Bar) // 编译错误：cannot make type Bar
    (*z).thingOne = "hello"
    (*z).thingTwo = 1

    // OK
    x := make(Foo)
    x["x"] = "goodbye"
    x["y"] = "world"

    // NOT OK
    u := new(Foo)
    (*u)["x"] = "goodbye" // 运行时错误!! panic: assignment to entry in nil map
    (*u)["y"] = "world"
    
}
```

试图 make() 一个结构体变量，会引发一个编译错误，这还不是太糟糕，但是 new() 一个映射并试图使用数据填充它，将会引发运行时错误！ 

因为 new(Foo) 返回的是一个指向 nil 的指针，它尚未被分配内存。所以在使用 map 时要特别谨慎。


#### 带标签的结构体

结构体中的字段除了有名字和类型外，还可以有一个可选的标签（tag）：它是一个附属于字段的字符串，可以是文档或其他的重要标记。

标签的内容不可以在一般的编程中使用，只有包 reflect 能获取它。

reflect包，它可以在运行时自省类型、属性和方法。

```
package main

import (
	"fmt"
	"reflect"
)

type TagType struct { // tags
	field1 bool   "An important answer"
	field2 string "The name of the thing"
	field3 int    "How much there are"
}

func main() {
	tt := TagType{true, "Barak Obama", 1}
	for i := 0; i < 3; i++ {
		refTag(tt, i)
	}
}

func refTag(tt TagType, ix int) {
	ttType := reflect.TypeOf(tt)
	ixField := ttType.Field(ix)
	fmt.Printf("%v\n", ixField.Tag)
}
```

#### 匿名字段和内嵌结构体

结构体可以包含一个或多个 匿名（或内嵌）字段，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字。

匿名字段本身可以是一个结构体类型，即 结构体可以包含内嵌结构体。


可以粗略地将这个和面向对象语言中的继承概念相比较，随后将会看到它被用来模拟类似继承的行为。

Go 语言中的继承是通过内嵌或组合来实现的，所以可以说，在 Go 语言中，相比较于继承，组合更受青睐。

在一个结构体中对于每一种数据类型只能有一个匿名字段。
需要多个相同类型的匿名字段，必须以匿名结构体存在。

```
package main

import "fmt"

type innerS struct {
	in1 int
	in2 int
}

type outerS struct {
	b    int
	c    float32
	int  // anonymous field
	innerS //anonymous field
}

func main() {
	outer := new(outerS)
	outer.b = 6
	outer.c = 7.5
	
	// 匿名的
	outer.int = 60
	outer.in1 = 5
	outer.in2 = 10

	fmt.Printf("outer.b is: %d\n", outer.b)
	fmt.Printf("outer.c is: %f\n", outer.c)
	fmt.Printf("outer.int is: %d\n", outer.int)
	fmt.Printf("outer.in1 is: %d\n", outer.in1)
	fmt.Printf("outer.in2 is: %d\n", outer.in2)

	// 使用结构体字面量
	outer2 := outerS{6, 7.5, 60, innerS{5, 10}}
	fmt.Println("outer2 is:", outer2)
}
```

命名冲突

当两个字段拥有相同的名字（可能是继承来的名字）时该怎么办呢？
+ 外层名字会覆盖内层名字（但是两者的内存空间都保留），这提供了一种重载字段或方法的方式；

+ 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误（不使用没关系）。没有办法来解决这种问题引起的二义性，必须由程序员自己修正。

```
type B struct {a, b int}
type D struct {B; b float32}
var d D;

// 规则1：使用 d.b 是没问题的：它是 float32，而不是 B 的 b。如果想要内层的 b 可以通过 d.B.b 得到。


type A struct {a int}
type B struct {a, b int}

type C struct {A; B}
var c C;

// 规则 2：使用 c.a 是错误的，到底是 c.A.a 还是 c.B.a 呢？会导致编译器错误
```

#### 方法

在 Go 语言中，结构体就像是类的一种简化形式，那么面向对象程序员可能会问：类的方法在哪里呢？

在 Go 中有一个概念，它和方法有着同样的名字，并且大体上意思相同：
Go 方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。
因此方法是一种特殊类型的函数。

接收者类型可以是（几乎）任何类型，不仅仅是结构体类型：任何类型都可以有方法，甚至可以是函数类型，可以是 int、bool、string 或数组的别名类型。

但是接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却是具体实现；

如果这样做会引发一个编译错误：invalid receiver type…。

最后接收者不能是一个指针类型，但是它可以是任何其他允许类型的指针。


一个类型加上它的方法等价于面向对象中的一个类。

一个重要的区别是：在 Go 中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，它们可以存在在不同的源文件。

唯一的要求是：它们必须是同一个包的。

类型 T（或 *T）上的所有方法的集合叫做类型 T（或 *T）的方法集。

因为方法是函数，所以同样的，不允许方法重载，即对于一个类型只能有一个给定名称的方法。

但是如果基于接收者类型，是有重载的：具有同样名字的方法可以在 2 个或多个不同的接收者类型上存在，比如在同一个包里这么做是允许的。