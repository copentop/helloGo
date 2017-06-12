## 错误处理与测试


Go 没有像 Java 和 .NET 那样的 try/catch 异常机制：不能执行抛异常操作。

但是有一套 defer-panic-and-recover 机制。

 Go 检查和报告错误条件的惯有方式:

+ 产生错误的函数会返回两个变量，一个值和一个错误码；如果后者是 nil 就是成功，非 nil 就是发生了错误。

+ 为了防止发生错误时正在执行的函数（如果有必要的话甚至会是整个程序）被中止，在调用函数后必须检查错误。



```
if value, err := pack1.Func1(param1); err != nil {
	fmt.Printf(“Error %s in pack1.Func1 with parameter %v”, err.Error(), param1)
	return    // or: return err
}
// Process(value)

```


Go 有一个预先定义的 error 接口类型

```
type error interface {
	Error() string
}
```

任何时候当你需要一个新的错误类型，都可以用 errors（必须先 import）包的 errors.New 函数接收合适的错误信息来创建。

像下面这样：

```
err := errors.New("math - square root of negative number")
```


错误类型以 “Error” 结尾，错误变量以 “err” 或 “Err” 开头。

syscall 是低阶外部包，用来提供系统基本调用的原始接口。
它们返回整数的错误码；类型 syscall.Errno 实现了 Error 接口。

os 包也提供了一套像 os.EINAL 这样的标准错误，它们基于 syscall 错误:

```
var (
	EPERM		Error = Errno(syscall.EPERM)
	ENOENT		Error = Errno(syscall.ENOENT)
	ESRCH		Error = Errno(syscall.ESRCH)
	EINTR		Error = Errno(syscall.EINTR)
	EIO			Error = Errno(syscall.EIO)
	...
)
```


`用 fmt 创建错误对象`

通常你想要返回包含错误参数的更有信息量的字符串，例如：可以用 fmt.Errorf() 来实现.



### 运行时异常和 panic

当发生像数组下标越界或类型断言失败这样的运行错误时，Go 运行时会触发运行时 panic，伴随着程序的崩溃抛出一个 runtime.Error 接口类型的值。

这个错误值有个 RuntimeError() 方法用于区别普通错误。


panic 可以直接从代码初始化：当错误条件（我们所测试的代码）很严苛且不可恢复，程序不能继续运行时，可以使用 panic 函数产生一个中止程序的运行时错误。

panic 接收一个做任意类型的参数，通常是字符串，在程序死亡时被打印出来。


```
package main

import "fmt"

func main() {
	fmt.Println("Starting the program")
	panic("A severe error occurred: stopping the program!")
	fmt.Println("Ending the program")
}

```

在多层嵌套的函数调用中调用 panic，可以马上中止当前函数的执行，所有的 defer 语句都会保证执行并把控制权交还给接收到 panic 的函数调用者。

这样向上冒泡直到最顶层，并执行（每层的） defer，在栈顶处程序崩溃，并在命令行中用传给 panic 的值报告错误情况：这个终止过程就是 panicking。




### 从 panic 中恢复（Recover）

这个（recover）内建函数被用于从 panic 或 错误场景中恢复：让程序可以从 panicking 重新获得控制权，停止终止过程进而恢复正常执行。



recover 只能在 defer 修饰的函数中使用：

用于取得 panic 调用中传递过来的错误值，如果是正常执行，调用 recover 会返回 nil，且没有其它效果。

```
func protect(g func()) {
	defer func() {
		log.Println("done")
		// Println executes normally even if there is a panic
		if err := recover(); err != nil {
		log.Printf("run time panic: %v", err)
		}
	}()
	log.Println("start")
	g() //   possible runtime-error
}

```

panic，defer 和 recover 怎么结合使用的完整例子。

```
// panic_recover.go
package main

import (
	"fmt"
)

func badCall() {
	panic("bad end")
}

func test() {
	defer func() {
		if e := recover(); e != nil {
			fmt.Printf("Panicing %s\r\n", e)
		}
	}()
	badCall()
	fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt
}

func main() {
	fmt.Printf("Calling test\r\n")
	test()
	fmt.Printf("Test completed\r\n")
}
```
输出

	Calling test
	Panicing bad end
	Test completed



defer-panic-recover 在某种意义上也是一种像 if，for 这样的控制流机制。



### 自定义包中的错误处理和 panicking

所有自定义包实现者应该遵守的最佳实践:

+ 在包内部，总是应该从 panic 中 recover：不允许显式的超出包范围的 panic()

+ 向包的调用者返回错误值（而不是 panic）。




每当函数返回时，我们应该检查是否有错误发生：但是这会导致重复乏味的代码。

结合 defer/panic/recover 机制和闭包可以得到一个我们马上要讨论的更加优雅的模式。


这个模式只有当所有的函数都是同一种签名时可用，这样就有相当大的限制。

例子：

假设所有的函数都有这样的签名:
```
func f(a type1, b type2)
// 给这个类型一个名字
fType1 = func f(a type1, b type2)
```

使用了两个帮助函数
```
// check：这是用来检查是否有错误和 panic 发生的函数
func check(err error) { if err != nil { panic(err) } }

// errorhandler：这是一个包装函数。

func errorHandler(fn fType1) fType1 {
	return func(a type1, b type2) {
		defer func() {
			if err, ok := recover().(error); ok {
				log.Printf(“run time panic: %v”, err)
			}
		}()
		fn(a, b)
	}
}

```

通过这种机制，所有的错误都会被 recover，并且调用函数后的错误检查代码也被简化为调用 check(err) 即可。
在这种模式下，不同的错误处理必须对应不同的函数类型；它们（错误处理）可能被隐藏在错误处理包内部。


### 启动外部命令和程序

os 包有一个 StartProcess 函数可以调用或启动外部系统命令和二进制可执行文件:
它的第一个参数是要运行的进程，
第二个参数用来传递选项或参数，
第三个参数是含有系统环境基本信息的结构体。



exec 包中也有同样功能的更简单的结构体和函数:
主要是 exec.Command(name string, arg ...string) 和 Run()。

首先需要用系统命令或可执行文件的名字创建一个 Command 对象，然后用这个对象作为接收者调用 Run()。

```
// exec.go
package main
import (
	"fmt"
    "os/exec"
	"os"
)

func main() {
	// 1) os.StartProcess //
	/*********************/
	/* Linux: */
	env := os.Environ()
	procAttr := &os.ProcAttr{
			Env: env,
			Files: []*os.File{
				os.Stdin,
				os.Stdout,
				os.Stderr,
			},
		}
	// 1st example: list files
	pid, err := os.StartProcess("/bin/ls", []string{"ls", "-l"}, procAttr)  
	if err != nil {
		fmt.Printf("Error %v starting process!", err)  //
		os.Exit(1)
	}
	fmt.Printf("The process id is %v", pid)

```




































