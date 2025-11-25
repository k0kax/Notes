## 一、数组
在 Go 中，**数组** 是一个具有编号且**长度固定**的元素序列。
代码如下：
```go
package main

import "fmt"

func main() {
	//数组声明 （仅声明）
	var a [5]int
    fmt.Println("emp:", a)
	
	//数组赋值
    a[4] = 100
    fmt.Println("set:", a)//访问数组
    fmt.Println("get:", a[4])//访问数组元素

    fmt.Println("len:", len(a))//数组长度
    
	//声明+赋值
    b := [5]int{1, 2, 3, 4, 5}
    fmt.Println("dcl:", b)

	
    //使用 ... 让编译器自动计算长度
    arr := [...]int{7, 8, 9, 10}
    fmt.Println(arr) // 输出: [7 8 9 10]

	//二维数组
    var twoD [2][3]int
    
    for i := 0; i < 2; i++ {
        for j := 0; j < 3; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD)
    
	twoD = [2][3]int{
        {1, 2, 3},
        {1, 2, 3},
    }
    fmt.Println("2d: ", twoD)
}
```
## 二、切片

切片Slice是另一种灵活的数组，是一种基于数组的动态试图。它本身不存储数据，而是通过指针引用底层数组的一部分连续元素。数组是切片的 “底层容器”，切片是数组的 “动态窗口”。
- slice的声明需要内建函数 `make([]T, len, cap)`或`s := arr[start:end]`
- len()返回slice长度，cap()返回slice容量
- append(slice,'s')增加元素
- copy(s,c)将s复制给c
- slice[low:high]取切片low~high的值，low和high都可为0
代码如下：
```go
package main

import "fmt"

func main() {

	
	var 切片名 []类型      //声明切片

    s := make([]string, 3)//短声明string切片，长度为3
    fmt.Println("emp:", s)

    s[0] = "a"
    s[1] = "b"
    s[2] = "c"
    fmt.Println("set:", s)
    fmt.Println("get:", s[2])//取切片的值

    fmt.Println("len:", len(s))//切片长度
	fmt.Println("cap:", cap(s))//切片容量
    s = append(s, "d")//新增切片
    s = append(s, "e", "f")
    fmt.Println("apd:", s)

    c := make([]string, len(s))
    copy(c, s)//切片复制
    fmt.Println("cpy:", c)

    l := s[2:5]//切片切取
    fmt.Println("sl1:", l)

    l = s[:5]//切取s[0]~s[5](不包含5)
    fmt.Println("sl2:", l)

    l = s[2:]//切取s[2](包含2)之后的元素
    fmt.Println("sl3:", l)

    t := []string{"g", "h", "i"}
    fmt.Println("dcl:", t)

	//多维切片
    twoD := make([][]int, 3)
    for i := 0; i < 3; i++ {
        innerLen := i + 1
        twoD[i] = make([]int, innerLen)
        for j := 0; j < innerLen; j++ {
            twoD[i][j] = i + j
        }
    }
    fmt.Println("2d: ", twoD)
}
```
PS：动态扩容，当`len == cap`时会触发扩容机制，创建一个新的更大的数组（通常是原容量的 2 倍），将原数组的元素复制到新数组，然后切片的指针指向新数组。
```go

```