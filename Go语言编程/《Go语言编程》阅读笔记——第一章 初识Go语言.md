# Go语言编程——第一章 初识Go语言

[TOC]



## 1. 语言特性

Go语言具有以下最主要的特性：

- 自动垃圾回收
- 更丰富的内置类型
- 函数多返回值
- 错误处理
- 匿名函数和闭包
- 类型和接口
- 并发编程
- 反射
- 语言交互性

### 1.1 自动垃圾回收

1. 手动管理内存的问题：内存泄漏、悬空指针。

2. 内存检查工具: Rational Purity、 Compuware BoundsChecker和英特尔的Parallel Inspector等。

3. 设计方法角度衍生了类似**内存引用计数**之类的方法（“智能指针”）。

4. 目前为止， 内存泄漏的最佳解决方案是在语言层面引入自动垃圾回收机制(Garbage Collection)。

5. 垃圾回收： 所有的**内存分配动作**都会在**运行时记录**， 同时**任何对该内存的使用也会被记录**。然后**垃圾回收器**会**对所有已分配的内存**进行**跟踪监测**， 一旦发现有些内存已不再被任何人使用，就**阶段性地回收**这些**没人用的内存**。

   ​

### 1.2 更丰富的内置类型

1. Go语言内置了字典类型(map) ，这对于其他静态类型语言通常用库方式支持。

2. Go新增了一个数据类型： 数组切片(slice)， 可认为slice是一种可动态增长的数组。与C++的vector相似。

   ​

### 1.3 函数多返回值

1. 目前主流语言中，除python外基本都不支持函数的多返回值功能。

2. Go提供多返回值功能。由于返回值都有名字，因此各返回值可在不同位置进行赋值，提供很大的灵活性。

3. 并非所有返回值都要赋值，没有明确赋值的返回值将保持默认的**空值**。

4. 如果对某些返回值不关心，可用**下划线**作为占位符来忽略。

   ​

### 1.4 错误处理

1. Go引入3个关键字用于标准的错误处理流程： defer 、panic 和 recover 。

2. Go的错误处理机制相较于C++/Java的异常捕获机制，大量减少代码量， 无需层层嵌套的try-catch语句。

   ​

### 1.5 匿名函数和闭包

1. Go中所有函数也是值类型， 可作为参数传递。

2. Go支持常规的匿名函数和闭包。 如下定义了名为f的匿名函数：

   ```
   f := func(x, y int) int {
       return x + y
   }
   ```

   ​

### 1.6 类型和接口

1. Go的类型沿用了struct关键字， 并且定义非常接近C语言中结构(struct) 。

2. Go不支持继承与重载，只支持最基本的类型组合功能。

3. C++中实现一个接口之前必须先定义该接口，并将类型和接口紧密绑定， 即接口的修改会影响到所有实现该接口的类型。 而Go中， 类型实现时没有声明与接口的关系， 但接口和类型可直接转换。甚至接口定义都不用在类型定义之前。

   ​

### 1.7 并发编程

1. Go语言引入了goroutine概念，使得并发编程更简单。

2. 通过在函数调用前使用关键字go， 可使该函数以goroutine方式执行。

3. goroutine 是一种比线程更轻盈、更省资源的协程。

4. 当一个协程阻塞时， 调度器会自动把其他协程安排到另外的线程中执行， 实现程序无需等待、并行化运行。

5. Go语言实现了CSP(通信顺序进程，Communicating Sequential Process) 模型来作为goroutine间的推荐通信方式。

6. CSP模型中，一个并发系统有多个并行运行的顺序进程组成，每个进程不能对其他进程的变量赋值。进程之间只能通过一对通信原语实现协作。

7. Go语言中用channel(通道)这一概念来实现CSP模型。通过channel可方便地进行跨goroutine的通信。

8. 由于一个进程内创建的所有goroutine运行在同一内存地址空间中， 因此如果不同的goroutine不得不去访问共享的内存变量， 访问前应先获取相应的读写锁。 Go标准库中的sync包提供了完备的读写锁功能。

   ```Go
   package main
   import "fmt"
   func sum(values [] int, resultChan chan int) {
   	sum := 0
   	for _, value := range values {
   		sum += value
   	}
   	resultChan <- sum // 将计算结果发送到channel中
   }
   func main() {
   	values := [] int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
   	resultChan := make(chan int, 2)
   	go sum(values[:len(values)/2], resultChan)
   	go sum(values[len(values)/2:], resultChan)
   	sum1, sum2 := <-resultChan, <-resultChan // 接收结果
   	fmt.Println("Result:", sum1, sum2, sum1 + sum2)// "15 40 55" or "40 15 55"
   }
   ```

   以上例子演示goroutine和channel的使用方式。这是一个并行计算的例子，由于两个goroutine进行并行的累加计算，待两个计算过程都结束后打印计算结果。

   ​

### 1.8 反射

通过反射，你可以获取对象类型的详细信息， 并可动态操作对象。



### 1.9 语言交互性

1. Go代码中， 可以按照Cgo的特定语法混合编写C语言代码， 然后Cgo工具可以将这些混合的C代码提取并生成对于C代码的调用包装代码。

   ```go
   package main
   /*
   #include <stdio.h>
   */
   import "C"
   import "unsafe"

   func main() {
   	cstr := C.CString("Hello, world")
   	C.puts(cstr)
   	C.free(unsafe.Pointer(cstr))
   }
   ```

   ​

