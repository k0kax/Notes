## 数组
在 Go 中，**数组** 是一个具有编号且长度固定的元素序列
```go
package main

import "fmt"

func main() {
	//数组声明 （仅声明）
	var a [5]int
    fmt.Println("emp:", a)
	
	//数组赋值
    a[4] = 100
    fmt.Println("set:", a)
    fmt.Println("get:", a[4])

    fmt.Println("len:", len(a))//数组长度
    
	//声明+赋值
    b := [5]int{1, 2, 3, 4, 5}
    fmt.Println("dcl:", b)

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
