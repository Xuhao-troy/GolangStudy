#Golang - import导入包的语法

## 1. 常用导入

在写Go代码时时常用到import命令来导入包文件，通常如下：
```
import(
	"fmt"
)
```
然后在后面的代码中通过如下方式调用：

`fmt.Println("hello world")`

　　以上调用方式中，由于`fmt`是Golang 的标准库，所以是去GOROOT路径下加载该模块。
　　当导入包非标准库时，需要制定其路径，分为**相对路径**和**绝对路径**两种。
* **相对路径** `import "./model"` //当前文件目录下的model目录（不建议）
* **绝对路径** `import "hello/model"` //加载GOPATH/src/hello/model模块。

## 2. 点操作
**代码示例**
`import (. "fmt")`

**含义**
包导入后，在调用该包的函数时，可以省略前缀的包名。
比如调用`fmt.Println("hello world")`可以省略为`Println("hello world")`

## 3. 别名操作
**代码示例**
`import (f "fmt")`

**含义**
别名操作调用包函数时，前缀变成重命名的前缀，即`f.Println("fmt")`

## 4. _操作
**代码示例**
`import _ "hello"`

**含义**
当用此方式导入一个包时，会在main函数中默认执行该包文件中所有的`init()`函数，而不是把整个包都导入，所以不能通过`hello.Print()`的方式来调用hello包中函数。

## 5. pb
**代码示例**
`fmt pb "hello/hello"`

**含义**
该方式导入的是hello.pb.go所在的包，并非是指$GOPATH/src/hello/hello目录。