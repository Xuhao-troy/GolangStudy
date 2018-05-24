# Go语言编程——第三章 面向对象编程

[TOC]

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



2. Go语言只在修改对象时，才必须使用指针，例如：

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

1.  定义一个Rect类型后， 创建并初始化对象实例的方法：

   ```go
   rect1 := new(Rect)
   rect2 := &Rect{}
   rect3 := &Rect{0,0,100,200}
   rect4 := &Rect{width: 100, height: 200}
   ```

2.  Go语言中，对象的创建通常交由一个全局的创建函数来完成，以`NewXXX`命名，表示该结构体的“构造函数”：

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



2. 在Go语言中，还可以以指针方式从一个类型“派生”：

   ```go
   type Foo struct{
       *Base
       ...
   } 
   //与1中实例不同的是，此写法在Foo创建实例的时候，需要外部提供一个Base类实例的指针。
   ```

3. 接口组合中的名字冲突问题：

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

4. 以下情况会出问题：

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

2.  Go语言的非入侵式接口影响深远：

   -  Go的标准库不再需要绘制类库的继承树图。
   - 实现类的时候，只需关心自己应该提供哪些方法，不用再纠结接口需要拆得多细才合理。接口由使用方按需定义，而不用事先规划。
   - 不用为实现一个接口而导入一个包，因为多引用一个外部的包，就意味着更多耦合。



### 3.5.3 接口赋值

1. 接口赋值在Go语言中分成两种情况：

   - 将对象实例赋值给接口；

   - 将一个接口赋值给另一个接口。



2. 将类型的对象实例赋值给接口，要求该对象实例实现了接口要求的所有方法，例如：

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

3. 另一种情形：将一个接口赋值给另一个接口。 只要两个接口拥有相同的方法列表（顺序不管），就可认为两者等同， 可相互赋值。



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
