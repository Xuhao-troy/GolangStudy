# 《Go程序设计语言》阅读笔记

[TOC]

# 第一章 入门

## 1.1 hello, world
1. helloworld 代码如下：
```go
package main
impaort "fmt"
func main() {
	fmt.Println("hello, world")
}
```
2. Go是**编译型语言**。 Go的工具链将**程序的源文件**转变为**机器相关的原生二进制指令**。比如：
 * go run helloworld.go ：将helloworld源文件进行编译、链接，然后运行生成的可执行文件。
 * go build helloworld.go: 将源文件编译成可复用的程序。该命令生成了一个名为helloworld的二进制程序，并可随时通过命令`$./helloworld`执行。

3. Go不需要在语句或声明后面使用分号结尾，除非有多个语句或声明出现在同一行。
4. Go对于代码的格式化要求非常严格。goimports工具可以按需管理导入声明的插入和移除。

##1.2 命令行参数
1. 程序的输入通常来自一个外部源：文件、网络连接、其他程序的输出、键盘、命令行参数等。
2. os包提供一些函数和变量，以与平台无关的方式和操作系统打交道。命令行参数以os包中的Args名字的变量供系统访问，在os包外面，使用os.Args这个名字。
3. 变量os.Args是一个字符串slice。可简单理解为一个动态容量的顺序数组s，可通过s[i]来访问单个元素，通过s[m:n]来访问一段连续子区间，数组长度用len(s)表示。
4. 在Go中，所有的索引使用**半开区间**，即**包含第一个索引，不包含最后一个索引**。表达式`s[m:n]`表示从第m个到第n-1个元素的slice。
5. 如果m或n缺失，默认分别为0或len(s)。
6. 分析一个例子：UNIX echo命令的实现，将命令行参数输出到一行。代码如下
	```go
	//echo1 输出其命令行参数
	package main
	import(
		"fmt"
		"os"
	)
	
	func main(){
		var s, sep string
		for i:=1; i<len(os.Args); i++{
			s+=sep + os.Args[i]
			sep = " "
		}
		fmt.Println(s)
	}
	```
7. 递增语句`i++`是对i进行叫1操作。不同于c语言，`j=i++`是不合法的。并且只支持后缀，所以`++i`也是不合法的。
8. 循环的索引变量i在for循环开始处声明。`:=`符号用于**短变量声明**，声明一个或多个变量，并根据初始化的值给予合适的类型。
9. Go中没有while语句。可以用for实现传统的while循环。
	```
	for condition {
		//...
	}
	```
10. 另一种形式的for循环在字符串或slice数据上迭代。以下是改进版echo程序：
	```go
	import(
		"fmt"
		"os"
	)
	
	func main(){
		s, sep := "", ""
		for _, arg := range os.Args[1:] {
			s+=sep + os.Args[i]
			sep = " "
		}
		fmt.Println(s)
	}
	```
说明： 每一次迭代，range产生一对值：索引和该索引处的元素值。
11. 变量的两种声明方式：隐式声明`s := ""`和显式声明`var s string`。使用显式初始化来说明初始化变量的重要性，使用隐式的初始化来表明初始化变量不重要。
12. 在上述两种方案中，每次循环，字符串s有了新的内容。+=语句通过追加旧的字符串、空格字符和下一个参数，生成了一个新字符串，然后赋给s。旧的内容不再需要使用，会被例行垃圾回收。
13. 如果有大量的数据需要处理，这样做的代价会很大。一个简单高效的实现方法是使用strings包中的Join函数：
```go
	func main(){
	fmt.Println(strings.Join(os.Args[1:], " "))
	}
```

14. 练习
* 修改echo程序输出os.Args[0]，即命令的名字。
* 修改echo程序，输出参数的索引和值，每行一个。

代码如下：
```go
package main
import(
	"fmt"
	"os"
)

func main(){
	var s, sep string
	for i:=1; i<len(os.Args); i++{
		s+=sep + os.Args[i]
		sep = " "
	}
	fmt.Println(s)
	fmt.Println(os.Args[1:])
	fmt.Println(os.Args[0])
	for i ,arg :=range os.Args[1:]{
		fmt.Printf("%d %s\n",i,arg  )
	}

}
```
执行结果：
```go
2018/03/30 09:18:49 debugger.go:482: continuing
141 13 105 17 208
[141 13 105 17 208]
F:\myGo\src\hello\debug
0 141
1 13

```

## 1.3 找出重复行
1. dup程序输出标准输入中出现次数大于1的行，前面是次数。代码如下
```go
package main
import(
	"bufio"
	"fmt"
	"os"
)

func main(){
	counts := make(map[string]int)
	input := bufio.NewScanner(os.Stdin)
	for input.Scan(){
		counts[input.Text()]++
	}
	//note: ignore possible errors in input.Err()
	for line, n := range counts {
		if n > 1 {
			fmt.Printf("%d\t%s\n", n, line)
		}
	}
}
```
2. Printf函数有超过10个转义字符，称为verb。

| verb | 描述 |
|-|-|
| %d | 十进制整数 |
| %x, %o, %b | 十六进制、八进制、二进制整数 |
| %f, %g, %e | 浮点数： 如3.141593， 3.1415926e+00 |
| %t | 布尔型： true或false |
| %c | 字符(Unicode码点) |
| %s | 字符串 |
| %q | 带引号字符串("abc")或者字符(如'c') |
| %v | 内置格式的任何值 |
| %T | 任何值的类型 |
| %% | 百分号本身(无操作数) |







# 第二章 程序结构

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








# 第三章 基本数据

1. Go的数据类型分为四大类： 基础类型、聚合类型、引用类型和接口类型。
2. 基础类型：包括数字 (number)、字符串 (string) 和 布尔型 (boolean) 。
3. 聚合类型： 包括数组 (array) 和 结构体 (struct) 。
4. 引用类型： 包含多种不同类型， 如指针 (pointer)， slice, map, 函数和通道 (channel) 。共同点在于间接指向程序变量或状态，操作所引用数据的效果会遍及该数据的所有引用。、
5. 接口类型： 见第七章

## 3.1 整数

1. Go的整数类型有: (u)int8、 (u)int16、 (u)int32、 (u)int64、 uintptr
2. rune 类型是 int32 类型的同义词， 常用于指明一个值是Unicode码点。两者可互换使用。
3. byte 类型是 uint8 类型的同义词， 强调一个值是**原始数据**，而非量值。
4. 无符号整数uintptr 大小并不明确， 但**足以完整存放指针**。
5. int 、uint 和 uintptr 都有别于其大小明确的 相似类型。 比如， 尽管int 默认大小是32位， 但是int 与 int32 仍是**不同类型**，两者**必须显式转换**。
6. **有符号整数**以**补码**表示，保留最高位作为符号位， n位数字的取值范围是 $-2^{n-1} \thicksim 2^{n-1}-1$。
7. **无符号整数**由全部位构成非负值， 范围为$ 0 \thicksim 2^n-1$。

## 3.2 浮点数

1.  Go具有两种大小的浮点数float32 和 float64 
2. math包给出了浮点值的极限。 常量math.MaxFloat32 是float32 的最大值， 约为3.4e38。 而math.MaxFloat64 约为1.8e308。 

## 3.3 复数

1. Go具备两种大小的复数 complex64 和 complex128， 二者分别由float32 和 foat64构成。
2. 内置的**complex函数**根据**给定的实部和虚部创建复数**，而内置的**real函数**和**imag函数**则分别提取**复数的实部和虚部**。
```go
   var x complex128 = complex(1, 2) // 1+2i
   var y complex128 = complex(3, 4) // 3+4i
   fmt.Println(x*y) 			    //"(-5+10i)"
   fmt.Println(real(x*y)) 			//"-5"
   fmt.Println(imag(x*y)) 			//"10"
```
3. 源码中，在浮点数或十进制整数后添加字母i，则变成虚数。

## 3.4 布尔值

1. bool值取值： true 或 false
2. 布尔值**无法隐式转换**成数值（如0或1），反之亦然。
3. 假如转换操作经常用到，就需要专门写个函数：
```go
func btoi (b bool) int {
    if b {
        return 1
    }
    return 0
}
```
## 3.5 字符串

1. 字符串是**不可变**的字节序列。

2. 内置的 len函数 返回字符串的字节数（并非文字符号的数目），下标访问操作 s[i] 则取得第i个字符， 其中$0 \leq i < len(s)$ 。

3. 字符串的第 i 个字节不一定是第 i 个字符。因为非ASCII字符的UTF-8码点需要两个或多个字节。

4. 子串生成操作`s[i:j]`产生新的字符串，取自原字符串的字节， 下标从i（含边界值）开始， 直到 j （不含边界值）结束。 结果的大小为 $j-i$ 个字节。

   **注意：**若下标越界， 或者 j 的值小于 i 的值， 则触发宕机异常。 若j等于i，则子串为空字符串。

```go
   fmt.Printf("%v", s[1:1]=="") //"true"
```

5. 加号运算符连接两字符串而生成一个新的字符串：
```go
   s := "hello, world"
   fmt.Println("goodbye" + s[5:]) //"goodbye, world"
```

## 3.6 常量
1. 常量是一种表达式， 保证在编译阶段就计算出表达式的值。

2.  常量声明可以同时指定类型和值。如果没有显式指定类型，则类型根据右边的表达式推断。

3. 若同时声明一组变量， 除了第一项外， 其他项在等号右侧的表达式均可忽略，会自动都用前一项的表达式及类型。如：

```go
   const (
       a = 1
       b
       c = 2
       d
   )
   fmt.Println(a, b, c, d) //"1 1 2 2"
```

### 3.6.1 常量生成器 iota
1. 常量声明可以使用常量生成器iota来创建一系列相关值。 

2. iota从0开始取值， 逐项加1。

   ```go
   type weekday = iota
   const (
       Sunday Weekday = iota //"0"
       Monday				//"1"
       Tuesday				//"2"
       Wednesday			//"3"
       Thursday			//"4"
       Friday				//"5"
       Saturday			//"6"
   )
   ```

   ​

3. 考虑更复杂的情况

   ```go
   const (
       A = m << (n * iota) //"m"
       B				  //"m*2^n"	
       C				  //"m*2^(2n)"
   )
   ```

   ​

### 3.6.2 无类型常量

1. Go语言中， 许多常量并不从属某一具体类型，称为无类型常量。

2. 无类型常量的优势在于： 值比基本类型的数字**精度更高**，且**算术精度**高于原生的机器精度（可认为精度至少达到256位）。

3. 从属类型待定的常量共有6种， 分为**无类型布尔、无类型整数、无类型文字符号、无类型浮点数、无类型复数和无类型字符串**。

4. 浮点型常量 `math.Pi`可用于任何需要浮点值或复数的地方

   ```Go
   var x float32 = math.Pi
   var y float64 = math.Pi
   var z complex128 = math.Pi
   ```

   如果`math.Pi`一开始就是确定类型的，则会导致精度下降。并且在上述表达式中需要转换类型。

5. 只有常量才可以无类型。若将无类型常量声明为变量，或将其赋值给类型明确的变量，则常量会被隐式转换成该变量的类型。

   ```
   var f float64 = 3 + 0i //无类型复数 -> float64
   f = 2				 // 无类型整数 -> float64
   f = 1e123			 // 无类型浮点数 -> float64
   ```

   ​
# 第四章 复合数据类型

1. 复合数据类型是由基本数据类型以各种方式组合而构成的。
2. 本章介绍四种复合数据类型：**数组、slice、map和结构体**。
3. **数组和结构体**的长度都是**固定的**，而**slice和map**都是**动态数据结构**，长度在元素添加到结构中时可动态增长。

## 4.1 数组

1. 由于数组长度固定，所以在Go中很少直接使用。

2. 数组的初始化

   ```Go
   var q [3]int = [3]int{1, 2, 3}
   var r [3]int = [3]int{1, 2}
   fmt.Println(r[2]) //"0"
   p := []int{1,2,3}
   q := [...]int{1,2,3}
   fmt.Printf("%T %T\n", p, q) // "[]int [3]int"
   ```

   数组初始化时，若省略号出现在数组长度的位置， 那么数组的长度由初始化数组的元素个数决定。

3. 数组长度是数据类型的一部分， [3]int 和 [4]int 是两种不同的数组类型。

4. 数组的初始化也可以不按照索引递增的方式进行：

   ```go
   	q := [...]string{0 :"$", 1: "@", 3: "%", 2: "#"}
   	fmt.Printf("%s %d\n", q, len(q)) //[$ @ # %] 4
   ```

5. 当作为函数的参数时，别的语言中数组是隐式地使用**引用传递**， 而Go把数组和其他类型都看成**值传递**。

6. 当然，也可以显式地传递一个数组的指针给函数。比如将一个数组 [32]byte 的元素清零：

   ```go
   func zero(ptr *[32]byte){
       for i := range ptr {
           ptr[i] = 0
       }
   }
   ```

   ​                                                                                                                                                                                                                                                                                                                                             

## 4.2 slice

1. slice 表示一个拥有相同类型元素的可变长度的序列。

2. slice 通常写成 `[]T`， 其中元素类型都是T ; 看上去象没有长度的数组类型。

3. slice有三个属性： **指针**、**长度**和**容量**。
    * **指针**指向数组的**第一个可以从slice中访问的元素**，不一定是数组的第一个元素。
    * **长度**是指slice中的**元素个数**， **不能超过slice的容量。**
    * **容量**的大小通常是从slice的**起始元素**到**底层数组的最后一个元素**间的元素个数。

4. Go的内置函数 len 和 cap 用来返回slice的长度和容量。

    ```go
    months := [...]string{1: "January", /* ... */, 12: "December"}
    Q2 := months[4:7]  //len=3, cap=9
    summer := months[6:9] //len=3, cap=7
    ```

5. 如果slice的引用超过cap(s)，则会导致程序宕机； 如果超过len(s)，则最终slice会比原slice长。

    ```go
    fmt.Println(summer[:20]) 	//宕机：超过被引用对象的边界
    endlessSummer := summer[:5]	//在slice容量范围内扩展了slice
    fmt.Println(endlessSummer)	//"[June July August September October]"
    ```

6. ​ slice包含了指向数组元素的指针，所以将slice传递给函数时，可在函数内部修改底层数组的元素。

    ```go
    //就地反转一个整型slice中的元素
    func reverse(s []int) {
        for i, j := 0, len(s)-1; i<j; i, j = i+1, j-1{
            s[i], s[j] = s[j], s[i]
        }
    }
    a := [...]int{0,1,2,3}
    reverse(a[:])
    fmt.Println(a) //"[3,2,1,0]"
    ```

7. 将一个slice**循环左移n个元素**的简单方法是**连续调用三次reverse 函数**。 第一次反转前n个元素， 第二次反转剩下的元素，最后对整个slice再做一个反转。 **循环右移的步骤是3-1-2**。

8. 初始化slice和初始化数组的区别在于， slice没有指定长度。

9. 和数组不同的是，slice无法做比较，因此不能用`==`来测试两个slice是否拥有相同元素。唯一允许的比较是与nil比较。

10. 内置函数make可以创建一个具有指定元素类型、长度和容量的slice。其中容量参数可以省略，此时，slice的长度和容量相等。

    ​

### 4.2.1 append函数

1. 内置append函数用于将元素追加到slice后面，对数组不可用。

2. 用法：

   ```
   var s []int
   for i:=1; i<5; i++{
       s = append(s, i)
   }
   fmt.Println(s) //"[1,2,3,4]"
   ```

   ​

3. **每次append调用**都必须检查slice**是否仍有足够容量**来**存储数据中的新元素**。如果不够， 则会进行底层数组的**容量扩展**。每次数组容量扩展时，通过**扩展一倍的容量**来**减少内存分配的次数**， 也可以保证追加一个元素所耗时间是固定时间。

4. 由于内置append函数策略复杂，不能确定一次append调用会不会导致新的内存分配，所以不能假设原始slice和append后的slice指向同一底层数组。 也不能假设旧slice上对元素的操作会不会影响新的slice元素。因此， **需要把append的调用结果再次赋值给传入append函数的slice**: `s = append(s, i)`

5. 函数append()的第二个参数其实是一个不定参数， 可添加若干个元素， 甚至直接将一个数组切片追加到另一个数组切片的末尾：

   ```
   mySlice2 := []int{8, 9, 10}
   mySlice = append(mySlice, mySlice2...)
   ```

   **注意：**在第二个参数mySlice2后面加了三个点，即一个省略号，相当于把mySlice2的所有元素打散后传入。 如果省略的话，因为mySlice中的元素类型为int，所以直接传递mySlice2是行不通的。

   ​

## 4.3 map

1. `map[K]V` 是一个拥有**键值对元素**的**无序**集合。

2. 集合中，**键的值是唯一的**，键对应的值可通过键来**获取**、**更新**或**移除**。

3. 无论map有多大， 以上值操作基本上通过**常量时间**的**键比较**就可以完成。

4. **键的类型K**必须是**可通过操作符==来进行比较**的数据类型，因此可检测某个键是否已经存在。

5. 值类型V没有任何限制。

6. map的声明：

   ```go
   var myMap map[string]int
   ```

7. map的创建
8.  利用内置函数delete根据键移除一个元素：

   ```go
   delete(ages, "alice") //移除元素ages["alice"]
   ```
```go
myMap = make(map[string]int)
ages :=make(map[string]int) // 通过内置make函数来创建map
ages["alice"] = 31
ages["bob"] = 32
//等价于
agess := map[string]int{
    "alice": 31,
    "bob" : 32,
}
```

8. 元素查找：Go语言中，map的查找功能设计精巧。

   ```go
   value, ok := myMap["1234"]
   if ok { //找到了
       //处理找到的value
   }
   ```

   判断是否成功找到特定的键，不需要检查取到的值是否是nil，只需查看第二个返回值ok。


## 4.4 结构体

1. 结构体是将零个或者多个任意类型的命名变量组合在一起的聚合数据类型。

   ```go
   type employee struct{
       ID			int
       Name		string
       Address		string
       DoB			time.Time
       Position	string
       Salary 		int
       ManagerID 	int
   }
   var dilbert Employee
   ```

   ​

2. 每个变量叫做**结构体的成员**。

3. 如果一个结构体的成员变量名称是首字母大写的，那么这个变量是可导出的。

4. 命名结构体类型S不可以定义一个拥有相同结构体类型S的成员变量。

5. 但是S中可以定义一个S的指针类型*S。这样就可以创建一些递归数据结构，比如链表和树。

6. 结构体类型的值的设置：

    ```go
    type Point struct{
        X, Y int
    }
    p := Point{1, 2}
    ```

7. 通过指定部分或全部成员变量的名称和值来初始化结构体变量。

    ```go
    p := Point{Y : 2}
    ```

8. 如果结构体的所有成员变量都可以比较，那么这个结构体就是可比较的。比较可以使用==或!=。


