# Go语言编程——第四章 并发编程

[TOC]



## 4.1 并发基础

1. 并发意味着程序在运行时有多个执行上下文，对应着多个调用栈。

2. 每个进程在运行时，都有自己的调用栈和堆，有完整的上下文。而操作系统在调度进程是，会保存被调度进程的上下文环境。等该进程获得时间片后，再恢复该进程的上下文到系统中。

3. 并发的价值在于以下几种场景：
   - 一方面需要响应灵敏的图形用户界面，一方面程序还要执行大量的运算或IO密集操作。我们需要让界面响应和运算同时执行。
   - 当Web服务器面对大量用户请求时，需要有更多的"Web服务器工作单元"来分别响应用户。
   - 事务处于分布式环境上，相同的工作单元在不同的计算机上处理着被分片的数据。

4. 并发的优势在于：
   - 能更客观的表现问题模型；
   - 可充分利用CPU核心的优势，提高程序执行效率；
   - 充分利用CPU与其他硬件设备固有的异步性。

5. 实现并发的几种模型：
   - **多进程**：操作系统层面的并发模式。同时也是开销最大的模式。

     ​		应用：Web服务器会用进程A负责网络端口的监听和链接管理，进程B负责事务和运算。

     ​		优势：简单，进程间互不影响。

     ​		劣势：系统开销大（所有进程都由内核管理）。

   - **多线程**：系统层面的并发模式。

     ​		应用：使用最多最有效的一种模式。几乎所有工具链都是用这种模式。

     ​		优势：比多线程开销小得多。

     ​		劣势：开销依旧大。高并发模式下，效率受影响。

   - **基于回调的非阻塞/异步IO**：诞生于多线程模式的危机。通过事件驱动的方式使用异步IO，使服务器持续运转，且尽可能少用线程。

     ​		应用：Node.js。

     ​		优势：开销小。

     ​		劣势：编程比多线程更复杂。由于把线程做分割，对问题本身的反应不够自然。

   - **协程**：本质上是一种用户态线程。不需要操作系统来进行抢占式调度，且在真正实现中寄存于线程中。

     ​		优势：开销极小。编程简单，结构清晰。

     ​		劣势：需要语言支持。

6. ​ 以前的并发思路是：线程+共享内存， 即“共享内存系统”。
   一种新的思路：线程间状态共享的各种操作都被封装在线程之间传递的消息中，即“消息传递系统”。

   个人理解：“消息传递系统”的改进点在于提供消息传递的接口，比如channel。但其本质仍是“锁+共享内存”，只不过Go语言对其添加了一层封装。

   ​

## 4.2 协程

1. “执行体“是个抽象的概念。在操作系统层面有多个概念相对应，比如进程(process)、进程内的线程(thread)以及进程内的协程(coroutine，也叫轻量级线程)。

2. 与传统的系统级线程和进程比（最多不超过1万个），协程的最大优势在于”轻量级“（轻松创建上百万个）。

3. Go语言在语言级别支持轻量级线程，叫做goroutine。由Go运行时管理。

   ​



## 4.3 goroutine

1. 使用方式。代码如下：

   ```go
   func Add(x, y int){
       z := x + y
       fmt.Println(z)
   }
   //并发执行：
   go Add(1, 1)
   ```

   ​

2. 在函数前加`go`关键字， 这次调用就会在新的goroutine中并发执行。当被调用的函数返回时，这个goroutine**直接结束**。

3. 如果作为协程运行的函数具有返回值的话，**这个返回值会被丢弃**。故协程函数**不能**放在赋值表达式的**右侧**。

   ```go
   func main() {
   	var c int
   	c = go add(1, 2)//错误，编译不通过
   	go add(1, 2) //正确

   }

   func add(a, b int) int {
   	return a + b
   }
   ```

4. 一个简单的例子：在main函数中调用10次Add()函数，并发执行。代码如下：

   ```go
   package main

   import "fmt"

   func add(x, y int) {
   	z := x + y
   	fmt.Println(z)
   }
   func main() {
   	for i := 0; i < 10; i++ {
   		go add(i, i)
   	}
   }
   ```

   奇怪的是，**程序输出为空**。

   原因在于， Go程序从初始化main package 并执行main()函数开始， 到main()函数返回，程序退出为止， 程序并不等待其他goroutine结束。解决此问题需要用到多个goroutine的通信。

   ​

## 4.4 并发通信

1. 例子：

   ```go
   package main

   import "fmt"
   import "sync"
   import "runtime"

   var counter int = 0

   func count(lock *sync.Mutex) {
   	lock.Lock()
   	counter++
   	fmt.Println(counter)
   	lock.Unlock()
   }
   func main() {
   	lock := &sync.Mutex{}
   	for i := 0; i < 10; i++ {
   		go count(lock)
   	}
   	for {
   		lock.Lock()
   		c := counter
   		lock.Unlock()
   		fmt.Println(c)
   		runtime.Gosched()
   		if c >= 10 {
   			break
   		}
   	}
   }
   ```
   上述例子中10个协程共享了全局变量counter。由于10个协程是并发执行的，因此引入锁的机制（即lock变量）。每次对counter的操作，都要先将锁锁上。操作完成后，再将锁打开。

2. 以上例子是用Go中加锁+共享内存的方式来实现并发。本质上与其他语言一致，是比较繁琐的方案。

3. Go提供了另一种更优的通信模型，即以**消息机制**而非共享内存作为通信方式。

4. Go语言提供的消息通信机制被称为`channel`。

   ​

## 4.5 channel

1. `channel`是Go语言在**语言级别**提供的goroutine间的通信方式。

2. `channel`是**进程内**的通信方式。 若要**跨进程通信**，建议使用**Socket或HTTP等通信协议**。

3. 用`channel`方式重写上述例子：

   ```go
   package main

   import "fmt"

   func count(ch chan int, i int) {
   	fmt.Println("Counting")
   	ch <- i //向channel中写数据。在channel被读取前，该操作是阻塞的。
   	
   }
   func main() {
   	chs := make([]chan int, 10)
   	for i := 0; i < 10; i++ {
   		chs[i] = make(chan int)
   		go count(chs[i], i)
   	}
   	for _, ch := range chs {
   		j := <-ch //从channel中读数据。在channel被写入前，该操作是阻塞的。
   		fmt.Println(j)
   	}
   }
   ```




### 4.5.1 基本语法

1. channel的声明：（一般用于全局变量）

   ```go
   var chanName chan ElementType
   var m map[string] chan bool //声明一个map， 元素是bool型channel。
   ```

2. channel的定义：（适用于局部变量）

   ```go
   ch := make(chan int)//双向channel,支持读取/写入
   ```

3. 写入：

   ```go
   ch <- value
   ```

4. 读取：

   ```
   value := <-ch
   ```




### 4.5.2 select

1. Go语言在语言级别支持select关键字，用于处理异步IO问题。

2. select用法与switch语句相类似。但是select最大的限制是，每个case语句里，必须是一个IO语句。大致结构如下：

   ```go
   select {
       case <-chan1:
       // 如果chan1成功读到数据，则进行该case处理语句
       case chan2 <- 1:
       // 如果成功向chan2写入数据，则进行该case处理语句
       default:
       // 如果上面都没有成功，则进入default处理流程
   }
   ```

   ​

3. select和switch的主要区别如下：

   - select后面不带判断条件，而是直接查看case语句。
   - 每个case语句都必须是面向channel的操作。

4. 样例：实现随机向channel中写入0或1。

   ```go
   package main

   import (
   	"fmt"
   )

   func main() {
   	ch := make(chan int, 1)
   	for index := 0; index < 10; index++ {
   		select {
   		case ch <- 0:
   		case ch <- 1:
   		}
   		i := <-ch
   		fmt.Println("Value received:", i)
   	}
   }
   ```




### 4.5.3 缓冲机制

1. 创建带缓冲的channel:

   ```go
   c := make(chan int, 1024)// 创建一个大小为1024的int类型channel
   ```

   即使没有读取方，写入方也可以一直往channel里写入，在缓冲区被填满前都不会阻塞。

2. 可用`range`关键字实现更简便的循环读取（先入后出）：

   ```go
   ch = make(chan int, 5)
   for i := 0; i < 5; i++ {
   	ch <- i
   }
   for i := range ch {
   	fmt.Println("Received：", i)
   }
   //结果如下：
   /*
   Received： 0
   Received： 1
   Received： 2
   Received： 3
   Received： 4
   fatal error: all goroutines are asleep - deadlock!*/
   ```

   错误原因在于range在读第6次时锁死。

   ​

### 4.5.4 超时机制

1. 在并发编程的通信过程中，最需要处理的就是超时问题。主要用于解决死锁问题。

2. 超时机制本身存在问题，比如两台机器运行速度差别很大，会出现结果不一致的现象。

3. Go没有提供直接的超时处理机制，但可利用select机制。select只要其中一个case完成，程序就会继续往下执行，而不会考虑其他case的情况。例子如下：

   ```go
   package main

   import (
   	"fmt"
   	"time"
   )

   func main() {
   	ch := make(chan int,1)
   	//ch <-1                     
   	timeout := make(chan bool, 1)
   	go func() {
   		time.Sleep(1e9) //等待1秒
   		timeout <- true
   	}()

   	select {
   	case i:=<-ch:
   		{
   			fmt.Println("ch:", i)
   		}
   	case <-timeout:
   		fmt.Println("Timeout!")
   	}
   }
   //结果是等待1秒，然后输出"Timeout！"。
   //入股把代码中注释那行还原，则结果为"ch:1"。
   ```

   ​

### 4.5.5 channel的传递

1. 在Go语言中`channel`是一个原生类型，因此channel本身在定义后也可以通过channel来传递。

   ​

### 4.5.6 单向channel

1. channel本身必然是同时支持读写的。

2. 所谓的单向channel概念，其实是对channel的一种使用限制。

3. 单项channel变量的声明：

   ```go
   var ch1 chan int 		 	//ch1是一个正常的channel，不是单向的
   var ch2 chan<- float64	 	//ch2是单向channel，只用于写float64数据
   var ch3 <-chan int			//ch3是单向channel，只用于读取int数据
   ```

4. 单项channel变量的定义：

   ```go
   ch4 := make(chan<- float64,1) 	//ch4是单向channel,只用于写float64数据
   ch5 := make(<-chan int, 1)  	//ch5是单向channel，只用于读int数据
   ```

5. 单向channel与双向channel的转换：

   ```go
   ch6 := make(chan int) 			//ch6是双向channel
   ch7 := chan<- int(ch6)			//ch7是单向写入channel
   ch8 := <-chan int(ch6)			//ch8是单向读取channel
   ```

6. 做单向channel限制的原因在于，设计角度而言，所有代码应该遵循”最小权限原则“，避免没必要的使用泛滥，导致程序失控。

7. 单向channel类似与C++中的const指针，为了起到一种使用限制的作用。

8. 单向channel的用法实例：

   ```go
   func Parse(ch <-chan int) {
   	for value := range ch {
   		fmt.Println("Parsing value", value)
   	}
   }
   ```




### 4.5.7 关闭channel

1. 语法：

   ```go
   close(ch)
   ```

2. 判断一个channel是否已被关闭。

   ```go
   x, ok :=<-ch//第二个参数是bool返回值，如果是false则表示ch已关闭。
   ```



## 4.6 多核并行化

1. 目前Go语言中，虽然从运行状态看多个goroutine在并行运行，但实际上这些goroutine都运行在同一个CPU核心上。在一个goroutine得到时间片后，其他协程都会处于等待状态。因此， goroutine的整体效率并不真正高于单线程程序。

2. 要想控制使用的CPU核心数，可通过以下方法：

   - 设定环境变量`GOMAXPROCS`。

   - 在代码中启动goroutine前先调用以下语句：

     ````go
     runtime.GOMAXPROCS(16)//设置16个CPU核心
     ````

3. 应该设置多少个CPU核心，可通过runtime包的`NUMCPU()`函数来获取核心数。

   ​

## 4.7 出让时间片

1. Go语言中，可以控制每个goroutine相似何时主动出让时间片给其他goroutine，可通过`runtime.Gosched()`实现。

2. 要想精细控制goroutine的行为，就必须深入了解Go的runtime包所提供的具体功能。

   ​

## 4.8 同步

### 4.8.1 同步锁

1. Go中的sync包提供了两种锁：

   - sync.Mutex: 最简单暴力的一种锁类型。一个协程获得Mutex后，其他协程只能等它释放该Mutex。

   - sync.RWMutex：相对友好， **单写多读**。

     在读锁占用的情况下，不能写，但可以读，即多个协程同时获得读锁（调用`RLock()`方法）。

     而写锁（调用`Lock()`方法）会阻止任何其他协程（无论读写）进来。整个锁由该协程独占。

2.  对于两种锁，任何一个`Lock()`和`RLock()`都需要保证调用相应的`Unlock()`和`RUnlock()`。

3. 锁的典型使用模式如下：

   ```go
   var l sync.Mutex
   func foo(){
       l.Lock()
       defer l.Unlock()
       //...
   }
   ```

   ​

### 4.8.2 全局唯一性操作

1. 对于全局只需运行一次的代码（比如全局初始化操作），Go语言提供了一个`Sync.Once`类型来保证全局的唯一性操作。具体代码如下：

   ```go
   package main

   import (
   	"fmt"
   	"sync"
   	"time"
   )

   var once sync.Once

   func setup() {
   	fmt.Println("hello, world")
   }
   func doprint() {
   	once.Do(setup)
   }
   func main() {
   	go doprint()
   	go doprint()
   	time.Sleep(time.Second)
   }
   //输出结果："hello, world"
   ```

   ​

   ​