# Golang学习——第二章 程序结构
[TOC]





## 2.1 名称

1. Go中**函数、变量、常量、类型、语句标签和包**的命名遵顼一个规则： 名称的开头是一个**字母**或**下划线**。
2. Go的25个关键字：

| break | default | func | interface | select |
| - |
| case | defer | go | map | struct |
| chan | else | goto | package | switch |
| const | fallthrough | if | range | type |
| continue | for | import | return | var |

3. Go内置的预声明常量、类型和函数：

* 常量： true false iota nil
* 类型： (u)int (u)int8 (u)int16 (u)int32 (u)int64 
uintptr float32 float64 complex128 complex64
bool type rune string error 
* 函数： make len cap new append copy close delete complex real imag panic recover

4. 如果一个**实体**在**函数中**声明，则只在函数**局部**有效。若声明在**函数外**，则**对包中所有源文件可见**。
5. **实体首字母大小写决定了其是否跨包可见**。 **大写字母开头的实体是导出的，对包外是可见和可访问的**，比如fmt包中的Printf函数。
6.  **包名**本身**总是以小写字母组成**。
7.  Go的编程风格倾向于使用**短名称**。通常，**名称的作用域越大，就使用越长且更有意义的名称**。

## 2.2 声明
1. 有4种主要的声明：**变量(var)、 常量(const)、 类型(type)** 和**函数(func)**。本章讨论变量和类型，常量放在第三章，函数在第五章。
2. Go程序存储在一个或多个以`.go`为后缀的文件中。
3. 每个文件以package声明开头，表明文件属于那个包。
4. package后面是import声明，然后是**包级别**的类型、变量、常量、函数声明，不区分顺序。
5. 包级别的实体名称不仅对包含其声明的源文件可见，而且对同个包内所有源文件都可见。
6. 局部声明则仅仅是在声明所在函数内可见，甚至是函数内一小块区域可见。

## 2.3 变量
1. var声明的通用形式：`var name type = expression`

2. 类型和表达式部分可以省略一个，但不能都省略。 如果**类型省略， 变量类型将由初始化表达式决定**。 如果**表达式省略， 初始值对应于类型的零值**——数字是0、布尔值是false、字符串是""、接口和引用类型（slice、指针、map、通道、函数）是nil。 对于数组或结构体这样的复合类型，零值是其所有元素或成员的零值。 

3. 零值机制保证所有变量都是良好定义的， Go中不存在未初始化变量。

4. 可以声明一个变量列表，并使用对应的表达式列表分别初始化。

   ```go
   var i, j, k int //int, int, int
   var b, f, s = true, 2.3, "four" // bool, float64, string
   ```

5. 包级别的初始化在main开始前进行， 局部变量初始化和声明一样在函数执行期间进行。

6. 变量也可以通过调用返回多个值的函数进行初始化：

   ​	`var f,err = os.Open(name) //os.Open返回一个文件和一个错误`

### 2.3.1 短变量声明 

1. 短变量声明标准形式：`name:= expression`

2. `name`的类型由`expression`的类型决定。

3. 局部变量的声明和初始化中主要使用短声明。 

4. var声明主要为跟初始化表达式类型不一致的局部变量保留的（如`var boiling float64 = 100`），或者用于后面才对变量赋值以及变量初始值不重要的情况。

5. 与var声明一样，多个变量可以用短声明的方式声明和初始化：

   `i, j := 0, 1`

   **注意：**要与`=`赋值相区分。`i, j = j, i //交换i和j的值。`

6. 短变量声明**不需要声明所有在左边的变量**。如果有变量**在同一个词法块中声明**，那么对于这些变量，短声明的行为等同于**赋值**。例如：

   ```go
      in, err := os.Open(infile)//声明了in和err
      out, err := os.Create(outfile)//声明out, 对err进行赋值
   ```

7. 短变量声明最少声明一个新变量，否则编译无法通过。

   ```go
      f, err := os.Open(infile)
      f, err := os.Create(outfile)//编译错误:没有新的变量。
   ```
### 2.3.2 指针 

1. 指针的值是变量的地址。

   ```go
   x := 1
   p := &x			// p是整型指针(*int)，指向x
   fmt.Println(*p)  // "1"
   *p = 2			// 等于 x=2
   fmt.Println(x)   // 结果 "2"
   ```

   ​

2. 指针类型的零值是nil。测试`p != nil`，结果是true 说明p指向一个变量。

3. 指针是可比较的，两个指针当且仅当指向同一变量或者两者都为nil时才相等。

   ```go
   var y, z int
   fmt.Println(&y == &y, &y == &z, &y == nil) //"true false false"
   ```

4. 函数返回局部变量的地址是非常安全的（这在c++中是错误的，因为c++中函数调用返回后会释放局部变量）。如以下代码中：

   ```go
   var p = f()
   func f() *int {
       v:= 1
       return &v
   }
   //每次调用f都会返回一个不同的值：
   fmt.Println(f() == f()) //"false"
   ```

   通过f产生的局部变量即使在调用返回后依然存在。

### 2.3.3 new函数

1. 另一种创建变量的方式是使用内置的new函数。

2. 表达式`new(T)`创建一个未命名的T类型变量，初始化为零值，**并返回其地址**。

   ```go
   p := new(int)	//*int类型的p，指向未命名的int变量
   fmt.Println(*p) //输出"0"
   *p = 2		   //把未命名的int设置为2
   fmt.Println(*p) //输出"2"
   ```

3. 每次调用new返回一个具有唯一地址的不同变量。

   ```go
   p := new(int)
   q := new(int)
   fmt.Println(p == q) //"false"
   ```

### 2.3.4 变量的生命周期

1. **生命周期**是指**在程序执行过程中，变量存在的时间段**。

2. 编译器可以选择使用堆或栈上的空间来分配，并且这一选择与使用var或new关键字来声明变量无关。

   ```
   var global *int 
   func f(){
       var x int
       x = 1
       global = &x
   }
   func g(){
       y := new(int)
       *y = 1
   }
   ```

   在上述代码中，尽管变量`x`和`*y`都是局部变量，但是`x`一定使用了**堆空间**，而`*y`可以分配在**栈**上。 因为`x`在f函数返回后仍能从`global`变量访问，称作`x`从f中**逃逸**。而`*y`在g函数返回后变得不可访问，可回收，所以没有从g中逃逸。

3. 长生命周期对象中保持短生命周期对象不必要的指针，特别是在全局变量中，会阻止垃圾回收器回收短生命周期的对象空间。

### 2.4 赋值

1. 不管是隐式还是显式赋值， 如果左边的（变量）和右边的（值）类型相同，就是**合法的**。 即赋值只有在**值对于变量类型**是**可赋值的**时才合法。

2. 可赋值性根据类型不同有不同的规则。对于已讨论的类型，规则为：类型必须精准匹配，nil可以被赋值给任何接口变量或引用类型。

3. 两个值使用`==`和`!=`进行比较时，第一个操作数相对于第二个操作数的类型必须是可赋值的，或者可以反过来赋值。

### 2.5 类型声明
1. 类型声明定义一个新的命名类型，他和某个已有类型使用同样的**底层类型**。

2. 类型声明通用格式为`type name underlying-type`, 通常出现在包级别。

### 2.6 包和文件
1. Go语言中，**包的作用**和其他语言中的**库或模块作用类似**，用于支持**模块化**、**封装**、**编译隔离**和**重用**。
2. 每个包给包中的声明提供了**独立的命名空间**。例如`image.Decode`和`utf16.Decode`使用相同标识符，但是关联了不同的函数。因此在包外调用函数时，需要**明确修饰标识符**。

