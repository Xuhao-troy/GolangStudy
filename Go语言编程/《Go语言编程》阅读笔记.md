# 《Go语言编程》阅读笔记

[TOC]


# 第一章 初识Go语言



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




# 第二章 顺序编程



## 2.1 流程控制

Go语言支持以下几种流程控制语句：

- 条件语句， 对应关键字为 if、else和 else if；

- 选择语句， 对应关键字为switch、case和select；

- 循环语句， 对应关键字为for和range；

- 跳转语句， 对应的关键字为goto。

  ​

### 2.1.1 条件语句

1. 样例：

   ```go
   if a < 0 {
       return -1
   } else if a == 0 {
       return 0
   } else {
       return 1
   }
   ```

   ​

2. 条件语句的注意点：

   - 不需要括号将条件包含起来；

   - 无论语句体中有几条语句，花括号必须存在；

   - 左花括号必须与if或else在同一行；

     ​

### 2.1.2 选择语句

1. 样例：

   ```go
   switch i {
      case 0: fmt.Printf("0")
      case 1: fmt.Printf("1")
      case 2,3: fmt.Printf("2,3")
      default: fmt.Printf("Default")
   } //运行结果： i=0时，输出0； i=1时，输出1.
   //switch后面的表达式甚至不是必须的。
   switch {
       case 0 <= Num && Num <= 3:
       fmt.Printf("0-3")
       case 4 <= Num && Num <= 6:
       fmt.Printf("4-6")
       case 7 <= Num && Num <= 9:
       fmt.Printf("7-9")
   }
   ```

   ​

2. switch结构注意点：

   - 左花括号必须与switch处于同一行

   - 条件表达式不限于常量或者整数

   - 单个case中，可出现多个结果选项

   - 与C语言规则不同，Go语言不需要break来明确退出一个case，而不会顺序到下一个case

   - 只有在case中明确添加fallthrough关键字，才会继续执行紧跟的下一个case

   - 可不设定switch之后的条件表达式，此时整个switch结构与多个if...else...逻辑作用相同。

     ​

### 2.1.3 循环语句

1. 样例

   ```go
   sum := 0
   for i:=0; i<10; i++{
       sum += i
   }
   //Go中没有while语句，可用for实现
   s := 0
   for {
       sum ++
       if sum > 100{
           break
       }
   }
   ```

   ​

2. 注意点：

   - 左花括号必须与for处于同一行

   - Go在for循环条件语句中，不支持以逗号为间隔的多个赋值语句，必须使用平行赋值的方式来初始化多个变量。

   - Go语言的for循环同样支持continue和break来控制循环。但是它提供了一种**更高级的break**， 可**选择中断哪些循环**， 如下例：

     ```go
     JLoop:
     for j := 0; j<5; j++{
         for i := 0; i<10; i++{
             if i>5 {
                 break JLoop
             }
             fmt.Println(i, j)
         }
     }
     	Println("End!") 
     //输出：
     /*	0 0
     	1 0
     	2 0
     	3 0
     	4 0
     	5 0	*/
     ```

     本例中，break语句终止的是JLoop标签处的外层循环。

     ​

### 2.1.4 跳转语句

1. goto语句的语义是跳转到**本函数内**的某个标签。样例：

   ```go
   	for j := 0; j < 5; j++ {
   		for i := 0; i < 10; i++ {
   			if i > 5 {
   				goto JLoop
   			}
   			fmt.Println(i, j)
   		}
   	}
   	JLoop:
   	fmt.Println("End!")//与上文中输出相同
   ```

   ​

## 2.2 函数

函数构成代码执行的逻辑结构。



### 2.2.1 函数定义

1. 样例：（简单的加法函数）

   ```go
   package mymath
   import (
   	"errors"
   )
   func Add(a , b int) (int, err error){
       if a<0 || b<0 { //假设这个函数只支持两个非负数字的加法
           err = errors.New("Should be non-negative numbers!")
           return
       }
       return a + b, nil //支持多返回值
   }
   ```

2. 注意点：

   - 若参数列表中若干相邻参数类型相同， 比如a和b，则可以只保留一个类型声明。

   - 若返回值列表中多个返回值类型相同，也可用相同的方式合并。

   - 如果函数只有一个返回值， 可直接写成：

     ```go
     func Add(a, b int) int {
         //...
     }
     ```

### 2.2.2 函数调用

1. 函数调用需先导入该函数所在包，再直接调用：

   ```go
   import "mymath" //假设Add被放在一个叫mymath的包中
   // ...
   c := mymath.Add(1,2)
   ```

2. Go语言中函数名都首字母大小写体现了该**函数的可见性**。 **小写**函数**仅包内可见**。**大写**函数**跨包可见**。这一规则也适用于**类型**和**变量**的可见性。

   ​

### 2.2.3 不定参数

1. 不定参数是指函数传入懂得参数个数为不定数量。

2. 为达到这一目的， 首先需要将函数定义为接收不定参数类型：

   ```go
   func myfunc1(args ...int){
       for _,  arg := range args{
           fmt.Println(arg)
       }
   }
   //函数调用
   myfunc1(1,2,3,4)

   //与传slice的区别
   func myfunc2(args []int){
       for _,  arg := range args{
           fmt.Println(arg)
       }
   }
   //函数调用
   args := []int{1,2,3,4}
   myfunc2(args)
   ```

3. 形如`...type`格式的类型**只能**作为**函数的参数类型**存在， 并且**必须**是**最后一个参数**。

4. 这种写法是一个**语法糖**(syntactic sugar)， 对语言的功能并没有影响，但是更方便程序员使用。

5. 从实现机理来说， 类型`...type`本质上是一个数组切片，也就是`[]type`。

6. **不定参数的传递**。

   ```go
   func myfunc(args ...int){
       //按原样传递
       myfunc3(args...)
       //传递片段， 实际上任意的int slice都可以传进去
       myfunc3(args[1:]...)
   }
   ```

   ​

7. 任意类型的不定参数

   以上例子中将不定参数类型约束为int。 如果希望传任意类型， 可指定类型为`interface{}`。

   例子1：Go语言标准库中fmt.Printf()函数原型。

   ```go
   func Printf(format string, args ...interface){
       //...
   }
   ```

   例子2：编写自己的Printf()函数

   ```go
   func myPrintf(args ...interface{}) {
   	for _, arg := range args {
   		switch arg.(type) {
   		case int:
   			fmt.Println(arg, "is an int value.")
   		case string:
   			fmt.Println(arg, "is a string value.")
   		case int64:
   			fmt.Println(arg, "is an int64 value.")
   		default:
   			fmt.Println(arg, "is an unknown type.")
   		}
   	}
   }
   //函数调用
   myPrintf(int(1), int64(234), string("hello"), float32(1.234))
   //结果：
   /*1 is an int value.
   234 is an int64 value.
   hello is a string value.
   1.234 is an unknown type.*/
   ```

   ​

### 2.2.4 匿名函数

1. 匿名函数：不需要定义函数名的一种函数实现方式。

2. 在Go中，函数可以像普通变量一样被传递或使用， 这与C语言的回调函数类似。不同的是， Go支持随时在代码中定义匿名函数。

3. 匿名函数组成：不带函数名的函数声明+函数体。

4. 匿名函数可以直接赋值给一个变量或是直接执行。样例：

   ```go
   f := func(x, y int) int {
   		return x + y
   	}(1, 2)		//花括号后直接跟参数列表，表示函数调用。
   fmt.Println(f) 	//结果： 3
   ```

5. Go的匿名函数是一个闭包。

   ​

### 2.2.5 闭包

1. 和变量的声明不同的是，Go语言中**不能在函数里声明另一个函数**。

2. 因此，在Go的源程序中， **函数声明**都出现在**最外层**。

3. “声明”就是把一种类型的变量和一个名字联系起来。

4. Go里有函数类型的变量。 因此， 虽然不能在一个函数中直接声明另一个函数， 但可以**在一个函数中可以声明一个函数类型的变量**， **此时的函数称为闭包**(closure)。

5. 例子：

   ```go
   add := func(base int) func(int) int {
   		return func(i int) int {
   			return base + i
   		}
   	}

   	add5 := add(5)
   	fmt.Println("add5(10)=", add5(10))
   //结果： add5(10)= 15
   ```

   add可认为是一个闭包作坊， 根据入参返回（生产）一个闭包。 这样add5就是使用5作为add的参数得到的一个闭包。

6. 闭包的声明在另一个函数的内部，形成嵌套。 和块嵌套一样， 内层的变量可覆盖同名的外层变量， 而外层变量可直接在内层使用。 比如， add的base参数在return返回的闭包的外层，所以它的值5在add返回并赋值给add5后依旧存在（原因是go的编译器会把闭包使用的外围变量分配到堆上， 而堆上的变量是不会随着函数返回自动消失的，只会被垃圾回收。）

## 2.3 错误处理

### 2.3.1 error接口

1. Go语言中引入了一个关于错误处理的标准模式，即error接口，定义如下：

   ```go
   type error interface{
       Error() string
   }
   ```

2. 对于绝大多数函数，如果返回i错误，应将error作为多种返回值中的最后一个。

   ```go
   func Foo(param int)(n int, err error){
       //...
   }
   ```

   ​

3. 调用含有错误类型输出变量的函数时按照以下方式：

   ```go
   n, err := Foo(1)
   if err != nil{
       //错误处理
   } else {
       //使用返回值n
   }
   ```

### 2.3.2 defer

1. 例子：

   ```go
   func CopyFile(dst, src string)(w int64, err error){
       srcFile, err := os.Open(src)
       if err != nil{
           return 
       }
       
       defer srcFile.Close()
       
       dstFile, err := os.Create(dst)
       if err != nil{
           return
       }
       
       defer dstFile.Close()
       
       return io.Copy(dstFile, srcFile)
   }
   ```

   即使Copy()函数抛出异常， Go仍能保证dstFile和srcFile先后被正常关闭。

2. 如果觉得一句话做不完清理的工作，可在defer后加一个匿名函数：

   ```go
   defer func(){
       //做复杂的清理工作
   }
   ```

   ​

3. 当一个函数中存在多个defer语句， 语句调用是按照先入后出的原则。即最后一个defer语句将最先被执行。

### 2.3.3  panic() 和 recover()

1. Go语言中引入了两个内置函数panic()和recover()来报告和处理运行时的错误和程序中的错误场景：

   ```go
   func panic(interface{})
   func recover() interface{}
   ```

2. 当一个函数执行过程中调用panic()函数时， 正常的函数执行流程将立即终止， 但函数中之前使用defer关键字延迟执行的语句将正常展开执行。 之后该函数将返回到调用函数， 并导致逐层向上执行panic流程， 直至所属的goroutine中所有正在执行的函数被终止。 错误信息将被报告，包括在调用panic()函数时传入的参数，这个过程称为错误处理流程。

3. recover()函数用于终止错误处理流程。 一般情况下， recover()应该在一个使用defer关键字的函数中执行以有效截取错误处理流程。

4. 如果没有在发生异常的goroutine中明确调用recover()， 会导致该goroutine所属的进程打印异常信息后直接退出。






# 第三章 面向对象编程

Go语言的优雅之处在于， 对面向对象编程的支持是语言类型系统中的天然组成部分。整个类型系统通过接口串联，浑然一体。

## 3.1 类型系统

1. 类型系统是指一个语言的类型体系结构。
2. 一个典型的类型系统通常包括以下内容：
   - 基础类型： 如int、float、bool等；
   - 复合类型： 如数组、结构体、指针等；
   - 可以指向任意对象的类型（Any类型）；
   - 值语义和引用语义；
   - 面向对象， 即所有具备面向对象特征的类型；
   - 接口。

### 3.1.1 为类型添加方法

1. 在Go语言中， 可以给任意类型（包括内置类型，但不包括指针类型）添加相应的方法，例如：

   ```go
   type Integer int
   func (a Integer) Less (b Integer) bool {
       return a < b
   }
   ```


1. Go语言只在修改对象时，才必须使用指针，例如：

   ```go
   func (a *Integer) Add (b Integer){
       *a += b
   }
   ```

   原因是Go和C一样，类型都是基于值传递的。要想修改变量的值，只能传递参数。

### 3.1.2 值语义和引用语义

1. C语言中，通过函数传递一个数组的时候基于引用语义，而在结构体中定义数组变量时基于值语义。

   而Go语言中， 数组和基本类型没有区别，是很纯粹的值类型。 例如：

   ```go
   	var a = [3]int{1,2,3}
   	var b = a
   	b[1] ++
   	fmt.Println(a,b)
   	//运行结果： [1 2 3] [1 3 3]
   ```

2. Go中有4个类型看起来是引用类型：

   - 数组切片

   - map

   - channel

   - 接口

     本质上是因为以上类型都是含有指针成员的结构体。

   ```go
   //slice:
   type slice struct {
       first *T
       len int
       cap int
   }
   //map:
   type Map_K_V struct {
       // ...
   }
   type map[K]V struct{
       imp1 *Map_K_V
   }
   //channel和map类似，本质是一个指针。
   //接口：
   type interface struct{
       data *void
       itab *Itab
   }
   ```

## 3.2 初始化

1. 定义一个Rect类型后， 创建并初始化对象实例的方法：

   ```go
   rect1 := new(Rect)
   rect2 := &Rect{}
   rect3 := &Rect{0,0,100,200}
   rect4 := &Rect{width: 100, height: 200}
   ```

2. Go语言中，对象的创建通常交由一个全局的创建函数来完成，以`NewXXX`命名，表示该结构体的“构造函数”：

   ```go
   func NewRect (x, y, width, height float64)*Rect{
       return &Rect{x, y, width, height}
   }
   ```

## 3.3 匿名组合

1. 确切来说， Go也提供了继承，只不过采用了**组合**的文法， 成为**匿名组合**：

   ```go
   type Base struct{
       Name string
   }

   func (base *Base) Foo(){
       //...
   }

   func (base *Base) Bar(){
       //...
   }

   type Far struct {
       Base
       //...
   }

   func (far *Far) Bar(){
       far.Base.Bar()
       //...
   }
   ```

   上述代码定义了一个Base类（实现了Foo()和Bar()两个成员方法），然后定义了一个Far类， 该类从Base类“继承”并改写了Bar()方法（该方法实现时先调用了基类的Bar()方法）

   在“派生类”Far没有改写“基类”Base的成员方法时， 相应的方法就被“继承”。 即调用far.Foo()和far.Base.Foo()效果一致。


1. 在Go语言中，还可以以指针方式从一个类型“派生”：

   ```go
   type Foo struct{
       *Base
       ...
   } 
   //与1中实例不同的是，此写法在Foo创建实例的时候，需要外部提供一个Base类实例的指针。
   ```

2. 接口组合中的名字冲突问题：

   ```go
   type X struct {
       Name string
   }
   type Y struct {
       X
       Name string
   }
   ```

   组合的类型和被组合的类型都包含一个Name成员，但是不会产生问题。因为所有Y类型的Name成员的访问都只会访问到最外层的那个Name变量， X.Name 变量相当于被隐藏起来了。

3. 以下情况会出问题：

   ```go
   type Logger struct {
       Level int
   }
   type Y struct {
       *Logger
       Name string
       *log.Logger
   }
   ```

   出问题的原因在于，匿名组合类型相当于以其类型名称（去掉包名部分）作为成员变量的名字。按此规则， Y类型中存在两个名为Logger的成员，会导致编译错误。



## 3.4 可见性

1. Go中无private、protected、public这样的关键字。
2. 要使符号（变量、类型、函数）在包外可见，需要变量名首字母大写。否则，仅包内可见。
3. Go语言中符号的可访问性是包一级的，而非类型一级的。

## 3.5 接口

接口是Go语言整个类型系统中的基石。



### 3.5.1 其他语言的接口

1. Go语言的接口与其他语言(C++、Java、C#等)提供的接口概念**不同**。
2. 除Go之外的其他语言中， 接口主要作为不同组件之间的契约存在。 对契约的实现是强制的。
3. 这种接口称为**侵入式接口**。“侵入式”的主要表现在于实现类需要明确自己实现了某个接口。
4. 侵入式接口的问题：
   - 提供哪些接口好？
   - 如果两个类实现了相同的接口，应该把接口放到哪个包好？

### 3.5.2 非入侵式接口、

1. Go语言中， 一个类只需实现了接口要求的所有函数，就认为这个类实现了该接口，例如：

   ```go
   //类：
   type File struct {
       // ...
   }
   func (f *File) Read(buf []byte) (n int, err error)
   func (f *File) Write(buf []byte) (n int, err error)
   func (f *File) Seek(off int64, whence int) (pos int64, err error)
   func (f *File) Close() error

   //接口：
   type IFile interface{
       Read(buf []byte) (n int, err error)
       Write(buf []byte) (n int, err error)
       Seek(off int64, whence int)(pos int64, err error)
       Close() error
   }
   type IReader interface{
       Read(buf []byte) (n int, err error)
   }
   type IWriter interface{
       Write(buf []byte) (n int, err error)
   }
   type ICloser interface{
       Close() error
   }
   ```

   尽管File类并没有从这些接口继承， 甚至不知道这些接口的存在， 但其确实实现了这些接口，可以进行赋值：

   ```go
   var file1 IFile = new(File)
   var file2 IReader = new(File)
   var file3 IWriter = new(File)
   var file4 ICloser = new(File)
   ```

2. Go语言的非入侵式接口影响深远：

   - Go的标准库不再需要绘制类库的继承树图。
   - 实现类的时候，只需关心自己应该提供哪些方法，不用再纠结接口需要拆得多细才合理。接口由使用方按需定义，而不用事先规划。
   - 不用为实现一个接口而导入一个包，因为多引用一个外部的包，就意味着更多耦合。

### 3.5.3 接口赋值

1. 接口赋值在Go语言中分成两种情况：
   - 将对象实例赋值给接口；
   - 将一个接口赋值给另一个接口。


1. 将类型的对象实例赋值给接口，要求该对象实例实现了接口要求的所有方法，例如：

   ```go
   //定义类型
   type Integer int
   func (a Integer) Less(b Integer) bool{
       return a<b
   }
   func (a *Integer) Add(b Integer){
       *a +=b
   }
   //定义接口
   type LessAdder interface{
       Less(b integer) bool
       Add (b Integer)
   }

   // 赋值
   var a Integer = 1 
   var b LessAdder = &a
   ```

2. 另一种情形：将一个接口赋值给另一个接口。 只要两个接口拥有相同的方法列表（顺序不管），就可认为两者等同， 可相互赋值。

### 3.5.4 接口查询

1. 代码:

   ```go
   var file1 Writer =...
   if file5, ok := file1.(two.IStream); ok{
       //...
   }
   ```

   这个if语句检查file1接口指向的对象实例是否实现了two.IStream接口。若实现了，则执行特定的代码。

### 3.5.5  类型查询

1. 在Go语言中， 可直接询问接口指向的对象实例的类型，例如：

   ```go
   var v1 interface{} = ...
   switch v := v1.(type){
       case int: 		//当前v的类型是int
       case string: 	//当前v的类型是string
   }
   ```

2. 由于语言中的类型很多， 所以类型查询并不经常使用。 通常作为一个补充，与接口查询配合使用：

   ```go
   type Stringer interface {
       String() string
   }
   func Println(args ...interface()){
       for _,arg := range args{
           switch v:= v1.(type) {
               case int: 		//...
               case string : 	//...
               default:		
               if v, ok := arg.(Stringer);  ok { //v的类型是Stringer
                   val := v.String()
                   //...
               } else {
                   // ...
               }
           }
       }
   }
   ```

3. 利用反射也可进行类型查询，详情可参阅reflect.TypeOf()方法的相关文档。

### 3.5.6 接口组合

1. Go不仅支持类型组合，也支持接口组合。 

2. 以来自io包中的一个接口io.ReadWriter为例：

   ```go
   //ReadWriter接口将基本的Read和writer方法组合起来
   type ReadWriter interface{
       Reader
       Writer
   }
   //完全等同于以下写法：
   type ReadWriter interface{
       Read(p []byte) (n int, err error)
       Write(p []byte)(n int, err error)
   }
   ```

   因为两种写法的表意完全相同：ReadWriter接口同时能做Reader和Writer接口的所有事情。

3. 可以认为接口组合是类型匿名组合的一个特定场景，只不过接口只包含方法，而不包含任何成员变量。

### 3.5.7 Any类型

1. 由于Go语言中任何对象实例都满足空接口interface()， 所以interface()看起来像是可以指向任何对象的Any类型。例子如下：

   ```go
   var v1 interface{} = 1 		//将int类型赋值给interface{}
   var v2 interface{} = "abc"	//将string类型赋值给interface{}
   var v3 interface{} = &v2	//将*interface{}类型赋值给interface{}
   var v4 interface{} = struct{X int}{1}
   var v5 interface{} = &struct{X int}{1}
   ```

2. 当函数可以接收任意的对象实例时， 会将其声明为interface{}。

3. 最典型的例子是标准库fmt中PrintXXX系列的函数，例如：

   ```go
   func Printf(fmt string, args ...interface{})
   func Println(args ...interface{})
   ...
   ```





# 第四章 并发编程



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

     ```go
     runtime.GOMAXPROCS(16)//设置16个CPU核心
     ```

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

2. 对于两种锁，任何一个`Lock()`和`RLock()`都需要保证调用相应的`Unlock()`和`RUnlock()`。

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

#第五章 网络编程

Go语言标准库里提供的net包，支持基于IP层、TCP/UDP层及更高层面（如HTTP、FTP、SMTP）的网络操作，其中用于IP层的称为RawSocket。



## 5.1 Socket编程

1. 其它语言中使用Socket编程时，会按照以下步骤展开：

   建立Socket -> 绑定Socket -> 监听 -> 接受连接 -> 接收/发送消息 。

2. Go标准库对此过程进行了抽象和封装。无论使用什么协议建立什么形式的连接， 都只需调用net.Dial()即可。

### 5.1.1 Dial()函数

1. 函数原型：

   ````go
   func Dial(net, addr string)(Conn, error)
   //输入：
   // net参数： 网络协议的名称
   // addr参数： IP地址或域名， 端口号以":"的形式跟随在地址或域名的后面，端口号可选。
   //返回值：
   //若连接成功， 则返回连接对象； 否则，返回error。
   ````

2. 几种常见协议的调用方式：

   - TCP链接：

     `conn, err := net.Dial("tcp", "192.168.0.10:2100")`

   - UDP链接：

     `conn, err := net.Dial("udp", "192.168.0.12:975")`

   - ICMP链接（使用协议名称）：

     `conn, err := net.Dial("ip4:icmp", "www.baidu.com")`

   - ICMP链接（使用版本号）：

     `conn, err := net.Dial("ip4:1", "10.0.0.3")`

3. 目前，Dial()函数支持以下几种网络协议：

   tcp、tcp4（仅限IPv4）、tcp6（仅限IPv6）、

   udp、 udp4（仅限IPv4）、 udp6（仅限IPv6）、 

   ip、ip4（仅限IPv4）和ip6（仅限IPv6）。


### 5.1.2 TCP示例程序
1. 下面建立TCP链接来实现初步的HTTP协议：通过向网络主机发送HTTP Head请求，读取网络主机返回的信息。代码如下：

   ```go
   package main

   import (
   	"bytes"
   	"fmt"
   	"io"
   	"net"
   	"os"
   )

   func main() {

   	conn, err := net.Dial("tcp", "www.baidu.com:80") //建立连接
   	checkError(err)

   	_, err = conn.Write([]byte("HEAD / HTTP/1.1\r\n\r\n")) //发送HTTP HEAD请求
   	checkError(err)

   	result, err := readFully(conn) //读取网站返回的信息
   	checkError(err)

   	fmt.Println(string(result)) //显示信息
   	os.Exit(0)
   }

   func checkError(err error) {
   	if err != nil {
   		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
   		os.Exit(1)
   	}
   }

   func readFully(conn net.Conn) ([]byte, error) {
   	defer conn.Close()
   	result := bytes.NewBuffer(nil)
   	var buf [512]byte
   	for {
   		n, err := conn.Read(buf[0:])
   		result.Write(buf[0:n])
   		if err != nil {
   			if err == io.EOF {
   				break
   			}
   			return nil, err
   		}
   	}
   	return result.Bytes(), nil
   }

   ```

   执行结果如下：

   ```go
   HTTP/1.1 302 Moved Temporarily
   Server: bfe/1.0.8.18
   Date: Fri, 13 Apr 2018 05:54:53 GMT
   Content-Type: text/html
   Content-Length: 17931
   Connection: Keep-Alive
   ETag: "54d9748e-460b"
   ```

### 5.1.3 更丰富的网络通信
1. 实际上， Dial()函数是对DialTCP()、DialUDP()、DialIP()和DialUnix()的封装。也可直接使用这些函数：

   ```go
   func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err error)
   func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err error)
   func DialIP(netProto string, laddr, raddr *IPAddr) (*IPConn, error)
   func DialUnix(net string, laddr, raddr *UnixAddr) (c *UnixConn, err error)
   ```

2. 之前（5.1.2中）的代码可改写为：

   ```go
   package main

   import (
   	"fmt"
   	"io/ioutil"
   	"net"
   	"os"
   )

   func main() {

   	tcpAddr, err := net.ResolveTCPAddr("tcp4", "www.baidu.com:80") //用于解析地址和端口号
   	checkError(err)

   	conn, err := net.DialTCP("tcp", nil, tcpAddr)//用于建立链接
   	checkError(err)

   	_, err = conn.Write([]byte("HEAD / HTTP/1.1\r\n\r\n"))
   	checkError(err)

   	result, err := ioutil.ReadAll(conn)
   	checkError(err)

   	fmt.Println(string(result))
   	os.Exit(0)
   }

   func checkError(err error) {
   	if err != nil {
   		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
   		os.Exit(1)
   	}
   }

   ```

   结果与5.1.2中样例相同。

3. net包中还包含了一系列工具函数：

   ```go
   func net.ParseIP() //验证IP地址有效性
   func IPv4Mask(a, b, c, d byte) IPMask //创建子网掩码
   func (ip IP) DefaultMask() IPMask //获取默认子网掩码
   //根据域名查找IP的代码如下：
   func ResolveIPAddr (net, addr string)(*IPAddr, error)
   func LookupHost(name string) (cname string, addrs []string, err error)
   ```

   ​

## 5.2 HTTP编程

Go标准库中提供了net/http包， 涵盖了HTTP客户端和服务端的具体实现。

### 5.2.1 HTTP客户端

1. 基本方法

   ```go
   func (c *Client) Get(url string) (r *Response, err error)
   func (c *Client) Post(url string, bodyType string, body io.Reader) (r *Response, err error)
   func (c *Client) PostForm(url string, data url.Values) (r *Response, err error)
   func (c *Client) Head(url string) (r *Response, err error)
   func (c *Client) Do(req *Request) (resp *Response, err error)
   ```

2. `http.Get()` ： 从指定资源获取数据

   以下样例获取网页的内容并存储在文件中：

   ```go
   package main

   import (
   	"fmt"
   	"net/http"
   	"os"
   )

   func main() {
   	resp, err := http.Get("http://example.com/") //获取网页数据
   	checkError(err)
   	defer resp.Body.Close()

   	f, err := os.Create("./example.txt") //创建文件
   	checkError(err)
   	defer f.Close()

   	buf := make([]byte, 1024)
   	for {
   		n, _ := resp.Body.Read(buf)
   		if 0 == n {
   			break
   		}
   		f.WriteString(string(buf[:n])) //将数据写入文件
   	}
   }

   func checkError(err error) {
   	if err != nil {
   		fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
   		os.Exit(1)
   	}
   }
   ```

3. `http.Post()` : 向指定资源发送数据。

   ```go
   func (c *Client) Post(
   url string, 		//网站的URL
   bodyType string, 	//将要POST数据的资源类型(MIMEType)
   body io.Reader 		//数据的比特流([]byte形式)
   ) (r *Response, err error)
   ```


4. `http.PostForm()`：实现表单提交。

5. `http.Head()`：只请求目标URL的头部信息， 不要HTTP BODY。

6. `(*http.Client).Do()`:适用于发起的HTTP请求需要更多的定制信息，希望设定一些自定义的Http Header字段。

### 5.2.2 HTTP服务端

本节介绍HTTP服务端技术，包括如何处理HTTP请求和HTTPS请求

#### 1. 处理HTTP请求

1. `http.ListenAndServe()`方法可以在指定的地址进行监听，开启一个HTTP。函数原型：

   ```go
   func ListenAndServe(
   addr string, //监听地址
   handler Handler //服务器处理程序。通常为空，表示服务器调用http.DefaultServeMux
   ) error
   ```

2. 服务器编写的业务逻辑处理程序`http.Handle()`或`http.handleFunc()`默认注入`http.DefaultServeMux`中。

   ```go
   http.Handle("/foo", fooHandler)
   http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
   fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
   })
   log.Fatal(http.ListenAndServe(":8080", nil))
   ```




## 5.3 RPC编程

1. RPC(Remote Procedure Call，远程过程调用)是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络细节的应用程序通信协议。

2. RPC协议构建于TCP或UDP，或者是HTTP之上。

3. RPC采用客户端——服务器的工作模式。

### 5.3.1 Go中的RPC支持与处理
1. 标准库提供的`net/rpc`包实现了RCP协议需要的相关细节。
2. 一个RPC服务器可以注册多个不同类型的对象， 但不允许注册同一类型懂得多个对象。
3. 一个对象中只有满足如下条件的方法，才能被RPC服务端设置为可供远程访问：
- 必须是在对象外部可公开调用的方法（首字母大写）；

- 必须有两个参数，且参数的类型必须是包外可访问的类型或是Go内建支持的类型；

- 第二个参数必须是指针；

- 方法必须返回一个error类型的值。

  方法原型即：

  ```go
  func (t *T)MethodName (
  	argType T1,    //由RPC客户端传入的参数
  	replyType *T2  //要返回给RPC客户端
  ) 
  error
  ```

  以上代码中，类型T、T1和T2默认会使用Go内置的`encoding/gob`包进行编码解码。
4. RPC服务端可通过调用`rpc.ServeConn`来处理单个连接请求。 多数情况下，通过TCP/HTTP在某个网络地址上进行监听来创建该服务。

5. RPC客户端可通过`net/rpc`包提供的`rpc.Dial()`和`rpc.DialHTTP()`方法来与指定RPC服务器建立连接。

6. 建立连接后， Go的`net/rpc`包允许客户端使用**同步**或**异步**的方式接收RPC服务端的处理结果。

   调用`Call()`方法：同步处理。。客户端程序按序执行，只有等接受完RPC服务端的处理结果才能继续执行后面的程序。

   调用`Go()`方法：异步处理。客户端程序无需等待服务端结果即可执行后面的程序。

### 5.3.2 Gob简介

1. Gob是Go的一个序列化数据结构的编码解码工具，在Go标准库中内置encoding/gob包以供使用。

2. 数据结构使用Gob进行序列化以后，可以进行网络传输。

3. 与JSON和XML不同，Gob是二进制编码的数据流。

### 5.3.3 RPC接口
1. Go的`net/rpc`很灵活，它在数据传输前后实现了编码解码器的接口定义。

2. 开发者可自定义数据的传输方式，以及C/S之间的交互行为。

3. RPC提供的编码解码器接口如下：

```go
//接口ClientCodec定义了 RPC 客户端如何在一个 RPC 会话中发送请求和读取响应。
type ClientCodec interface {
	WriteRequest(*Request, interface{}) error 	 //客户端将请求写入RPC连接中
	ReadResponseHeader(*Response) error			//读取服务器响应信息
	ReadResponseBody(interface{}) error
	Close() error
}

//接口ServerCodec定义了 RPC 服务端如何在一个 RPC 会话中接收请求并发送响应。
type ServerCodec interface {
	ReadRequestHeader(*Request) error			//从RPC连接中读取请求信息
	ReadRequestBody(interface{}) error
	WriteResponse(*Response, interface{}) error	 //向RPC客户端发送响应
	Close() error
}
```

4. 通过以上接口， 用户可自定义数据传输前后的编码/解码格式，而不局限于Gob。



