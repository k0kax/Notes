## 数组
在 Go 中，**数组** 是一个具有编号且**长度固定**的元素序列。
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

## 切片

切片Slice是另一种灵活的数组。长度可变，可增可减
- slice的声明需要内建函数 make
- len()返回slice长度
- append(slice,'s')增加元素
- copy(s,c)将s复制给c
- slice[low:high]取切片low~high的值
```go
package main

import "fmt"

func main() {

	
		var 切片名 []类型

    s := make([]string, 3)//短声明string切片
    fmt.Println("emp:", s)

    s[0] = "a"
    s[1] = "b"
    s[2] = "c"
    fmt.Println("set:", s)
    fmt.Println("get:", s[2])

    fmt.Println("len:", len(s))

    s = append(s, "d")
    s = append(s, "e", "f")
    fmt.Println("apd:", s)

    c := make([]string, len(s))
    copy(c, s)
    fmt.Println("cpy:", c)

    l := s[2:5]
    fmt.Println("sl1:", l)

    l = s[:5]
    fmt.Println("sl2:", l)

    l = s[2:]
    fmt.Println("sl3:", l)

    t := []string{"g", "h", "i"}
    fmt.Println("dcl:", t)

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