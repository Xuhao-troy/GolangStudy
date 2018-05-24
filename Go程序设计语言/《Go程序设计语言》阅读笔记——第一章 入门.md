# Golang学习——第一章 入门

[TOC]



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