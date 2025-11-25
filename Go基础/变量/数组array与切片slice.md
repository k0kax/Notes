## 数组
在 Go 中，**数组** 是一个具有编号且长度固定的元素序列。
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
