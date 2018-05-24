# Go调用分析工具的安装与使用

[TOC]

## 一、 准备工具

- Go 1.8+

- [Graphviz](http://www.graphviz.org/download)

- [go-callvis](https://github.com/TrueFurby/go-callvis)

  ​

## 二、 安装步骤

### 2.1 Graphviz安装

​	Graphviz （英文：Graph Visualization Software的缩写）是一个由AT&T实验室启动的开源工具包，用于绘制DOT语言脚本描述的图形。

**安装步骤如下**：

1. 下载并解压。

2. 配置环境变量：将解压后的bin目录添加到Path环境变量中。

3. 验证安装是否成功：命令行输入`dot -version`



### 2.2 go-callvis安装

​	go-callvis 是一个开发工具，其目的是通过使用来自函数调用关系图的数据及其与包和类型的关系来对程序进行可视概览。

​	有以下可选特性：

- 关注程序中的特定包

- 按包区分组函数和按类型区分方法

- 将包限制到自定义路径前缀

- 忽略包含路径前缀的包

- 省略来自/到std包的调用


**安装步骤如下**：
```go
go get -u github.com/TrueFurby/go-callvis
cd $GOPATH/src/github.com/TrueFurby/go-callvis
go install
```



## 三、使用说明

### 3.1 命令格式 

 `go-callvis [OPTIONS] <main pkg> | dot -Tpng -o output.png`

### 3.2 字段含义

1. OPTIONS：特性选项。 选择如下

   ```
   -focus string
         Focus package with import path or name. (default: main)
   -limit string
         Limit package paths to prefix. (separate multiple by comma)
   -group string
         Grouping functions by [pkg, type]. (separate multiple by comma)
   -ignore string
         Ignore package paths with prefix. (separate multiple by comma)
   -nostd
         Omit calls from/to std packages.
   -minlen uint
         Minimum edge length (for wider output). (default: 2)
   -nodesep float
         Minimum space between two adjacent nodes in the same rank (for taller output). (default: 0.35)
   ```

2. main pkg: 待分析的包(在$GOPATH/src下的路径)

3. dot: 用dot布局

4. -Tpng: 生成png图片格式

5. sample.dot ：脚本文件名

6. -o output.png: 生成并输出指定名称的图片

### 3.3 使用样例

1. 命令:`go-callvis -group pkg -limit github.com/TrueFurby/go-callvis github.com/TrueFurby/go-callvis | dot -Tpng -o callvis_group.png`

2. 结果： 

   ![调用关系图](callvis_group.png)

   ​


