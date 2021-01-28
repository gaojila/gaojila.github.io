# 从零开始学习Golang(变量)


# 前言

不知道从什么时候开始决定要深入学习一门开发语言了，从 17 年开始工作的时候漫无目的的学习 python 到现在，python 平时也会写但是大概还停留在抄代码的阶段吧！18 年 9 月入职了新公司，受小伙伴影响对 Golang 开始感兴趣了，从 6 月到现在陆陆续续学习总是没有头绪，可能时间太过闲散，每次学习一次大概要过很久很久才会继续学习，知识点忘了，现在从头开始吧！都记录在博客上。

# 变量

## 变量声明

Go 语言中的变量必声明后才能使用，同一作用域内不支持重复声明。

## 标准声明

Go 语言到变量声明格式为：

```Go
var 变量名 变量类型
```

变量声明以关键字<font color=yellow>var</font>开头，变量类型放在变量后面，行尾不需要分号(这一点后 C 不同)

```Go
var name string
var age int
var ok bool
```

## 批量声明

Go 语言是支持批量声明的。

```Go
var {
    a string
    b int
    c bool
    d float64
}
```

## 变量的初始化

Go 语言在声明变量到时候，会自动对变量对应到内存区域进行初始化操作，每个变量都会被初始化为其类型的默认值，例如：整型和浮点型的默认值为<font color=yellow>0</font>。
字符串变量的默认值为<font color=yellow>空字符串</font>。布尔型变量的默认值为<font color=yellow>false</font>。切片、函数、指针变量到默认值为<font color=yellow>nil</font>。
我们也可以在声明变量时指定初始值。

```Go
var 变量名 类型=表达式
var name string="gaojila"
var age int=25
```

我们也可以一次性初始化多个变量。

```Go
var name,age="gaojila",25
```

## 类型推倒

有时候我们会将变量的类型省略，这个时候编译器会根据等号右边的值来推导变量的类型完成初始化。

```Go
var name="gaojila"
var age=25
```

## 短变量声明

在函数内部，可以使用更简略的<font color=yellow>:=</font> 方式声明并初始化变量。(只能在函数体内)

```Go
package main

import {
    "fmt"
}

var name = "miaomiao"
func main() {
    age := 25
    name := "gaojila"//局部变量name
    fmt.Println(name,age)
}
```

## 匿名变量

在使用多重赋值时，如果想要忽略某个值，可以使用匿名变量<font color=yellow>（anonymous variable）</font>。 匿名变量用一个下划线<font color=yellow>\_</font>表示。

```Go
x,_ := 1
_,y := 2
```

# 常量

相对于变量，常量是恒定不变到值，多用于程序运行期间不会改变的那些值。常量的声明和变量的声明非常的类似，只是把<font color=yellow>var</font>换成了<font color=yellow>const</font>,常量在定义的时候必须赋值。

```Go
const pi = 3.14159
const e = 2.71
```

多个常量也可以一起声明

```Go
const {
    pi = 3.14159
    e =2.71
}
```

const 同时声明多个常量时，如果省略了值则表示和上面一行的值相同。

```Go
const {
    a = 1
    b
    c
    d
}
```

## iota

<font color=yellow>iota</font>是 go 语言中的常量计数器，只能在常量表达式中使用。
<font color=yellow>iota</font>在 const 关键字出现的时候将被重置为 0。const 每新增一行常量声明将使<font color=yellow>iota</font>计数一次。

```Go
const {
    a = iota //0
    b        //1
    c        //2
    d        //3
}
```

## 常见 iota 示例：

使用<font color=yellow>\_</font>跳过某些值

```Go
const {
    a = iota //0
    b        //1
    _
    d         //3
}
```

声明中间插队

```Go
const {
    a = iota //0
    b = 100
    c = iota //2
    d        //3
}
```

定义数量级

```Go
const {
    _ = iota
    KB = 1 << (10 * iota)
    .
    .
    .
    .
    PB = 1 << (10 * iota)
}
```

多个<font color=yellow>iota</font>>定义在一行

```Go
const {
    a,b = iota+1,iota+2 //1,2
    c,d                 //2,3
    e,f                 //3,4
}
```

