# 从零开始学Golang-切片


# 切片

## 切片的定义

切片的声明

```Go
var name []T
```

- name:变量名
- T:变量类型

## 切片的长度和容量

切片拥有自己的长度和容量，可以使用内置的 len()函数求长度，使用内置的 cap()函数求切片的容量。

## 基于数组定义切片

由于切片的底层就是一个数组，我们可以基于数组定义切片。

```Go
func main() {
    //基于数组定义切片
    a := [5]int{55,56,57,58,59}
    b := a[1:4]
    fmt.Println(b)
    fmt.Println("type of b:%T\n",b)
}

/* c := a[1:] [56,57,58,59]
   d := a[:4] [55,56,57]
   e := a[:]  [全都要] */
```

## 切片再切片

```Go
func main() {
	//切片再切片
	a := [...]string{"北京", "上海", "广州", "深圳", "成都", "重庆"}
	fmt.Printf("a:%v type:%T len:%d  cap:%d\n", a, a, len(a), cap(a))
	b := a[1:3]
	fmt.Printf("b:%v type:%T len:%d  cap:%d\n", b, b, len(b), cap(b))
	c := b[1:5]
	fmt.Printf("c:%v type:%T len:%d  cap:%d\n", c, c, len(c), cap(c))
}
```

## 使用<font color=red>make()</font>函数构造切片

语法：

```Go
make([]T,size,cap)
```

- T:切片的元素类型
- size:切片长度（元素数量）
- cap:切片的容量

## 切片的本质

切片的本质就是对底层数组的封装，它包含了三个信息：底层数组的指针、切片的长度（len）和切片的容量（cap）。

## 使用 <font color=red>append()</font> 向切片内添加元素

```Go
func main() {
	//append()添加元素和切片扩容
	var numSlice []int
	for i := 0; i < 10; i++ {
		numSlice = append(numSlice, i)
		fmt.Printf("%v  len:%d  cap:%d  ptr:%p\n", numSlice, len(numSlice), cap(numSlice), numSlice)
	}
}
```

## 切片的赋值和拷贝

切片的赋值实际都是指向了同一个内存地址，修改一个切片的值另一个也会跟着更改。
切片的拷贝使用<font color=yellow>copy()</font>函数
语法：

```Go
copy(destSlice, srcSlice []T)
```

- srcSlice: 数据来源切片
- destSlice: 目标切片

## 切片的遍历

一般情况下都使用<font color=yellow>for range</font>遍历

```Go
func main() {
	s := []int{1, 3, 5}

	for i := 0; i < len(s); i++ {
		fmt.Println(i, s[i])
	}

	for index, value := range s {
		fmt.Println(index, value)
	}
}
```

# 注意

- 切片是引用类型，切片赋值实际都是指向同一个内存地址

