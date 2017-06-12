## Go 并发

### 并发 concurrency

	goroutine 是由 Go 运行时环境管理的轻量级线程。

	语法：
	go function()


goroutine 奉行通过通信来共享内存，而不是共享内存来通信。

即通过channel 变量实现共享内存。

channel 是有类型的管道，可以用 channel 操作符 <- 对其发送或者接收值。

默认情况下，在另一端准备好之前，发送和接收都会阻塞。

这使得 goroutine 可以在没有明确的锁或竞态变量的情况下进行同步。

如果channel是无缓冲的，接收方会一直阻塞直到有数据到来。
发送方会一直阻塞直到接收方将数据取出。


如果channel带有缓冲区，发送方会一直阻塞直到数据被拷贝到缓冲区；
如果缓冲区已满，则发送方只能在接收方取走数据后才能从阻塞状态恢复。


```

ch := make(chan int)
ch <- v    // 将 v 送入 channel ch。
v := <-ch  // 从 ch 接收，并且赋值给 v。

```

使用 make 创建channel 类型，"箭头" 代表数据流的方向。


简单例子：
```
package main
import (
    "fmt"
    "time"
)

func main() {
    go Go()
    // 直接推出，没有等待 Go() 执行
    // time.Sleep( 2 * time.Second ), 等待两秒，Go() 可以执行。
    // 当Go()执行时间过长时，等待不是最好当实现。
}

func Go() {
    fmt.Println(" do .... ");
}

```
Channel 创建格式：
```
c := make (chan T)

T: 代表底层类型，int，string ...
```



发送者可以 close 一个 channel 来表示再没有值会被发送了。
接收者可以通过赋值语句的第二参数来测试 channel 是否被关闭：当没有值可以接收并且 channel 已经被关闭：

```
v, ok := <-ch

```
之后 ok 会被设置为 false。



循环 `for i := range c` 会不断从 channel 接收值，直到它被关闭。

只有发送者才能关闭 channel，而不是接收者。向一个已经关闭的 channel 发送数据会引起 panic。






Channel

+ Channel 是goroutine 沟通但桥梁，大都是阻塞同步的
+ 通过make 创建， close 关闭
+ Channel 是引用类型
+ 可以使用 for range 来迭代不断操作channel
+ 可以设置缓存大小， 在未被填满前不会发生阻塞，0大小表示阻塞
+ 有缓冲默认死异步的，无缓存默认是阻塞的。

Select

+ 可以处理一个或多个 channel的发送与接收
+ 同时有多个可用的channel 时按随机顺序处理
+ 可用空的select 来阻塞main 函数
+ 可以设置超时

```
package main
import (
    "fmt"
    "time"
)

func main() {
    // 创建 channel
    c := make( chan bool)
    go func() {
        fmt.Println("do ....")
        // 执行完，写的操作
        c <- true
    } ()
    // 读的操作，阻塞，等goroutine执行
    <- c
}

```

```
package main
import (
    "fmt"
    "time"
)

func main() {
    // 创建 channel
    c := make( chan bool)
    go func() {
        fmt.Println("do ....")
        // 执行完，写的操作
        c <- true
        // 注释掉会导致for range 进入死循环
        close(c)
    } ()
    // 读的操作，阻塞，等close执行,再接收
   for v := range c {
       fmt.Println(c)
   }
}

```


select 语句使得一个 goroutine 在多个通讯操作上等待。

select 会阻塞，直到条件分支中的某个可以继续执行，这时就会执行那个条件分支。
当多个都准备好的时候，会随机选择一个，并不保证按顺序选择那个。

当 select 中的其他条件分支都没有准备好的时候，default 分支会被执行。

```
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```




### sync.Mutex


如何保证在每个时刻，只有一个 goroutine 能访问一个共享的变量从而避免冲突？

通常使用 _互斥锁_(mutex)_来提供这个限制。

Go 标准库中提供了 sync.Mutex 类型及其两个方法：

```
Lock
Unlock
```


在代码前调用 Lock 方法，在代码后调用 Unlock 方法来保证一段代码的互斥执行。

也可以用 defer 语句来保证互斥锁一定会被解锁。





