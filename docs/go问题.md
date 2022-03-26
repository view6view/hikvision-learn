# GO语言中=和:=的区别

> 错误的做法

```go
//声明变量a
var a int
//声明变量a并给变量a赋值
a := 1
//错误提示
no new variables on left side of :=
//说明：重复声明变量a
```

> 声明不赋值的初始化值

- 整型和浮点型变量的默认值为 0，如var a int，默认a=0
- 字符串变量的默认值为空字符串
- 布尔型变量默认为 bool
- 切片、函数、指针变量的默认为 nil

> 使用编译器推导类型

```go
var a=10 //默认a为整型
```

### 特殊例子

> 正确

```go
var conn net.Conn
var err error
conn, err = net.Dial("tcp", "127.0.0.1:8080")
conn, err = net.Dial("tcp", "127.0.0.1:8080")
```

> 正确（特殊）

```go
//虽然err重复声明了，但是conn和conn2没有重复声明，只要有一个新声明，不会报错
conn, err := net.Dial("tcp", "127.0.0.1:8080")
conn2, err := net.Dial("tcp", "127.0.0.1:8080")
```

> 错误

```go
//重复声明了
conn, err := net.Dial("tcp", "127.0.0.1:8080")
conn, err := net.Dial("tcp", "127.0.0.1:8080")
```

# go语言指针符号的*和& 

## 先看一段代码

先放一段代码，人工运行一下，看看自己能做对几题？

```go
package main

import "fmt"

func main() {
    var a int = 1 
    var b *int = &a
    var c **int = &b
    var x int = *b
    fmt.Println("a = ",a)
    fmt.Println("&a = ",&a)
    fmt.Println("*&a = ",*&a)
    fmt.Println("b = ",b)
    fmt.Println("&b = ",&b)
    fmt.Println("*&b = ",*&b)
    fmt.Println("*b = ",*b)
    fmt.Println("c = ",c)
    fmt.Println("*c = ",*c)
    fmt.Println("&c = ",&c)
    fmt.Println("*&c = ",*&c)
    fmt.Println("**c = ",**c)
    fmt.Println("***&*&*&*&c = ",***&*&*&*&*&c)
    fmt.Println("x = ",x)
}
```

## 解释

### 理论

`&`符号的意思是对变量取地址，如：变量`a`的地址是`&a`
`*`符号的意思是对指针取值，如:`*&a`，就是`a`变量所在地址的值，当然也就是`a`的值了

### 简单的解释

`*`和 `&` 可以互相抵消,同时注意，`*&`可以抵消掉，但`&*`是不可以抵消的
`a`和`*&a`是一样的，都是a的值，值为1 (因为`*&`互相抵消掉了)
同理，`a`和`*&*&*&*&a`是一样的，都是1 (因为4个`*&`互相抵消掉了)

### 展开

因为有
`var b *int = &a`
所以
`a`和`*&a`和`*b`是一样的，都是a的值，值为1 (把`b`当做`&a`看)

### 再次展开

因为有
`var c **int = &b`
所以
`**c`和`**&b`是一样的，把*&约去后
会发现`**c`和`*b`是一样的 (从这里也不难看出，`*c`和`b`也是一样的) 又因为上面得到的`*&a`和`*b`是一样的 所以`**c`和`*&a`是一样的，再次把*&约去后`**c`和`a`是一样的，都是1

**不信你试试？**

## 公布结果

运行的结果内的地址值（0xc200开头的）可能会因不同机器运行而不同，你懂的

```
$ go run main.go 
a     =     1
&a     =     0xc200000018
*&a     =     1
b     =     0xc200000018
&b     =     0xc200000020
*&b     =     0xc200000018
*b     =     1
c     =     0xc200000020
*c     =     0xc200000018
&c     =     0xc200000028
*&c     =     0xc200000020
**c     =     1
***&*&*&*&c     =     1
x     =     1
```

## 两个符号抵消顺序

`*&`可以在任何时间抵消掉，但`&*`不可以被抵消的，因为顺序不对

```go
fmt.Println("*&a\t=\t",*&a)  //成功抵消掉，打印出1，即a的值
fmt.Println("&*a\t=\t",&*a)  //无法抵消，会报错
```

# 命名规范及大小写的访问权限

1、golang的命名推荐使用驼峰命名法，必须以一个字母（Unicode字母）或下划线开头，后面可以跟任意数量的字母、数字或下划线。

2、golang中根据首字母的大小写来确定可以访问的权限。无论是方法名、常量、变量名还是结构体的名称，如果首字母大写，则可以被其他的包访问；如果首字母小写，则只能在本包中使用

  可以简单的理解成，首字母大写是公有的，首字母小写是私有的

3、结构体中属性名的大写

如果属性名小写则在数据解析（如json解析,或将结构体作为请求或访问参数）时无法解析

```go
type User struct { 
    name string
    age  int
}
```

```go
func main() {
     user:=User{"Tom",18}
     if userJSON,err:=json.Marshal(user);err==nil{
   　　  fmt.Println(string(userJSON))   //数据无法解析
    }
}
```

如上面的例子，如果结构体中的字段名为小写，则无法数据解析。所以一般建议结构体中的字段大写
