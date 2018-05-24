# Golang学习——第四章 复合数据类型
[TOC]

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

