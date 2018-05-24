# Go语言编程——第五章 网络编程

[TOC]



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



