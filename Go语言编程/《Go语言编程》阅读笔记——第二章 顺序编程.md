

# Go语言编程——第二章 顺序编程

[TOC]

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