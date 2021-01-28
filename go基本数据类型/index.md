# Go基本数据类型


# 基本数据类型

## 整型

|  类型  |             描述              |
| :----: | :---------------------------: |
| uint8  |   无符号 8 位整型(0 到 255)   |
| uint16 |  无符号 16 位整型(0 到 255)   |
| uint32 |  无符号 32 位整型(0 到 255)   |
| uint64 |  无符号 64 位整型(0 到 255)   |
|  int8  | 有符号 8 位整型(-128 到 127)  |
| int16  | 有符号 16 位整型(-128 到 127) |
| int32  | 有符号 32 位整型(-128 到 127) |
| int64  | 有符号 32 位整型(-128 到 127) |

## 特殊整型

|  类型   |                         描述                         |
| :-----: | :--------------------------------------------------: |
|  uint   | 32 位操作系统上就是 uint32，64 位操作系统就是 uint64 |
|   int   |  32 位操作系统上就是 int32，64 位操作系统就是 int64  |
| uintptr |             无符号整型，用于存放一个指针             |

## 浮点型

|  类型   |                  描述                  |
| :-----: | :------------------------------------: |
| float32 | 最大范围约为 3.4e38，可以使用常量定义  |
| float64 | 最大范围约为 1.8e308，可以使用常量定义 |

## 布尔类型

| 类型  | 描述 |
| :---: | :--: |
| true  |  真  |
| false |  假  |

## 字符串转义字符

| 转义符 |                含义                |
| :----: | :--------------------------------: |
|   \r   |         回车符（返回行首）         |
|   \n   | 换行符（直接跳到下一行的同列位置） |
|   \t   |               制表符               |
|   \'   |               单引号               |
|   \"   |               双引号               |
|   \\   |               反斜杠               |

## 多行字符串

Go 语言中要定义多行字符串时，必须使用<font color=yellow>\`\`</font>

```Go
sl :=`第一行
第二行
第三行
`
```

## 字符串的常用操作

|                方法                 |       介绍       |
| :---------------------------------: | :--------------: |
|              len(str)               |      求长度      |
|                  +                  |    拼接字符串    |
|            strings.Split            |       分割       |
|          strings.contains           |   判断是否包含   |
| strings.HasPrefix,strings.HasSuffix |    前后缀判断    |
| strings.Index(),strings.LastIndex() | 字符串出现的位置 |
| strings.Join(a[]string,sep string)  |    join 操作     |

## byte 和 rune 类型

Go 语言的字符有以下俩种:

- <font color=yellow>uint8</font> 类型，或者叫 <font color=yellow>byte</font>类型，代表了 ASCII 码的一个字符。
- <font color=yellow>rune</font>类型，代表一个 UTF-8 字符。
  当需要处理中文、日文或者其他复合字符时，则需要用到<font color=yellow>rune</font>类型类型实际是一个 <font color=yellow>int32</font>。

## 修改字符串

要修改字符串，需要先将其转换成<font color=yellow>[]rune</font>或 byte<font color=yellow>[]</font>，完成后再转化成<font color=yellow>string</font>。无论那种转换，都会重新分配内存，并复制字节数组。

```Go
func changeString() {
    s1 := "big"
    //强制类型转换
    byteS1 := []byte(s1)
    byteS1[0] = 'p'
    fmt.Println(string(byteS1))

    s2 := "白萝卜"
    //强制类型转换
    runeS2 := []rune(s2)
    runeS2[0] = "胡"
    fmt.Println(string(runeS2))
}
```

## 类型转换

Go 语言只有强制类型转换，没有隐式类型转换。（该语法只能在两个类型之间支持相互转换的时候使用），语法如下

```Go
T(表达式)
```

其中，T 表示要转换的类型。表达式包括变量、函数返回值等。

```Go
func sqrtDemo() {
     a,b := 3,4
     var c int
     c = int(math.Sqrt(float64(a*a + b*b)))
     fmt.Println(c)
}
```

