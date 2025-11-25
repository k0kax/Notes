## 一、都有哪些基础变量
### 1.整形 int
| int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 有符号 | 有 | 有 | 有 | 有 | 无符号 | 无 | 无 | 无 | 无 |
| 32/64 | 8位 | 16 | 32 | 64 | 32/64 | 8 | 16 | 32 | 64 |


### 2.浮点数 float
| float32 | 32位 | 6-7位小数   |
| ------- | --- | -------- |
| float64 | 64  | 15-16位小数 |


### 3.布尔型 bool
true或者false

### 4.字符rune
rune<font style="color:rgba(0, 0, 0, 0.85);"> 是 </font>int32的别名，专门表示Unicode 码点（单个字符，包括中文、特殊符号等）。

rune 的主要目的是解决ASCII 编码和多字节字符（如中文、日文、韩文、 emoji 等）之间的矛盾。

Go 语言的string类型在底层是一个不可变的字节序列，并且默认采用UTF-8编码。UTF-8 是一种变长编码，它为不同的字符分配不同长度的字节：

+ <font style="color:rgb(0, 0, 0);">英文字母、数字等 ASCII 字符，只占</font><font style="color:rgb(0, 0, 0);"> </font>**<font style="color:rgb(0, 0, 0) !important;">1 个字节</font>**<font style="color:rgb(0, 0, 0);">。</font>
+ <font style="color:rgb(0, 0, 0);">大部分常见的非 ASCII 字符（如中文），占</font><font style="color:rgb(0, 0, 0);"> </font>**<font style="color:rgb(0, 0, 0) !important;">3 个字节</font>**<font style="color:rgb(0, 0, 0);">。</font>
+ <font style="color:rgb(0, 0, 0);">一些特殊字符或 emoji，可能占 </font>**<font style="color:rgb(0, 0, 0) !important;">4 个字节</font>**<font style="color:rgb(0, 0, 0);">。</font>

#### 使用场景
正确遍历字符串中的字符

```go
package main

import "fmt"

func main() {
    s := "Hello, 世界!"
    
    fmt.Println("使用 for range 遍历:")
    for index, char := range s {
        // char 的类型是 rune
        fmt.Printf("索引: %d, 字符: %c, rune 值: %U\n", index, char, char)
    }
}


运行结果：
使用 for range 遍历:
索引: 0, 字符: H, rune 值: U+0048
索引: 1, 字符: e, rune 值: U+0065
索引: 2, 字符: l, rune 值: U+006C
索引: 3, 字符: l, rune 值: U+006C
索引: 4, 字符: o, rune 值: U+006F
索引: 5, 字符: ,, rune 值: U+002C
索引: 6, 字符:  , rune 值: U+0020
索引: 7, 字符: 世, rune 值: U+4E16
索引: 10, 字符: 界, rune 值: U+754C
索引: 13, 字符: !, rune 值: U+0021
```

使用场景：将字符串转换为rune切片

```go
package main

import "fmt"

func main() {
    s := "Hello, 世界!"
    
    // 将字符串转换为 rune 切片
    runes := []rune(s)
    
    // 现在可以通过索引访问字符了
    fmt.Println("字符总数:", len(runes)) // 输出：9
    fmt.Println("第 8 个字符:", string(runes[7])) // 输出：世
    fmt.Println("第 9 个字符:", string(runes[8])) // 输出：界
    
    // 修改 rune 切片中的字符
    runes[0] = 'h'
    runes[7] = '国'
    
    // 将 rune 切片转换回字符串
    modifiedS := string(runes)
    fmt.Println("修改后的字符串:", modifiedS) // 输出：hello, 国界!
}
```

### 5.字符串 string
### 6.比特byte
在Go builtin包中，byte的定义如下

```go
type byte = uint8
```

二者等价，故可以相互赋值

byte主要用来存储：

+ Ascii字符（0~127），注意超出的范围不行，和rune要区分
+ 二进制数字（网络字节流）

```go
func main() {
    // 存储 ASCII 字符
    var c byte = 'A' // 'A' 的 ASCII 码是 65
    fmt.Println(c)  // 输出：65
    fmt.Printf("类型: %T, 值: %c\n", c, c) // 输出：类型: uint8, 值: A

    // 存储二进制数据
    data := []byte{0x48, 0x65, 0x6C, 0x6C, 0x6F} // "Hello" 的 ASCII 码
    fmt.Println(string(data)) // 输出：Hello

    // byte 和 uint8 相互转换
    var b byte = 255
    var u uint8 = b
    fmt.Println(u) // 输出：255
}
```

## 二、如何声明这些变量
### 1.关键字声明 var
#### 完整声明（声明类型加赋值）
```go
var 变量名 类型 = 初始值

var age int = 25       // 明确类型为 int，赋值 25
var name string = "Go" // 明确类型为 string，赋值 "Go"
```

#### 类型推断（未明确类型，自动推导）
```go
var 变量名 = 初始值

var count = 100  // 推断为 int 类型
var pi = 3.1415  // 推断为 float64 类型
var isTrue = true// 推断为 bool 类型
fmt.Printf("count: %T, pi: %T, isTrue: %T\n", count, pi, isTrue)
// 输出：count: int, pi: float64, isTrue: bool
```

_可通过%T打印类型_

#### 仅声明（不赋值，用零值）
int float为0 0.0

bool为false

string为""空字符串

```go
var 变量名 类型

var x int       // 零值为 0
var s string    // 零值为空字符串
var m map[string]int // 零值为 nil
fmt.Println(x, s, m) // 输出：0  map[]
```

#### 多变量声明
```go
// 形式1：同一类型多变量
var 变量1, 变量2, 变量3 类型 = 初始值1, 初始值2, 初始值3

// 形式2：不同类型多变量（用括号包裹）
var (
    变量1 类型1 = 初始值1
    变量2 类型2 = 初始值2
    变量3 类型3  // 仅声明，用零值
)

// 形式1：同一类型批量赋值
var a, b, c int = 1, 2, 3

// 形式2：不同类型批量声明
var (
    username string = "admin"
    password string = "123456"
    loginTime int64 // 零值为 0
)
fmt.Println(a, b, c)         // 输出：1 2 3
fmt.Println(username, password, loginTime) // 输出：admin 123456 0
```

### 2.短变量声明
使用 **:=**  声明，仅能在函数内部声明，会自动推断类型，支持多变量声明 + 赋值。

```go
package main
import "fmt"

func main() {
    // 单变量短声明
    score := 95 // 推断为 int 类型
    fmt.Println(score)

    // 多变量短声明
    name, age := "Alice", 28 // name: string，age: int
    fmt.Println(name, age)

    // 重新赋值（需包含新变量）
    age, isStudent := 29, true // age 重新赋值，isStudent 是新变量
    fmt.Println(age, isStudent)
}
```

## 三、打印这些变量
通常使用fmt.Printf来格式化打印

| **<font style="color:rgb(0, 0, 0) !important;">变量类型</font>**    | **<font style="color:rgb(0, 0, 0) !important;">占位符</font>** | **<font style="color:rgb(0, 0, 0) !important;">说明</font>**                 |
| :-------------------------------------------------------------- | :---------------------------------------------------------- | :------------------------------------------------------------------------- |
| int系列                                                           | %d                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">十进制整数</font>           |
| uint系列                                                          | %d或%v                                                       | <font style="color:rgba(0, 0, 0, 0.85) !important;">无符号整数</font>           |
| float32/float64                                                 | %f                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">浮点数（默认保留 6 位小数）</font> |
|                                                                 | %.2f                                                        | <font style="color:rgba(0, 0, 0, 0.85) !important;">保留 2 位小数</font>        |
|                                                                 | %e                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">科学计数法</font>           |
| bool                                                            | %t                                                          | 布尔值true或false                                                              |
| string                                                          | %s                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">字符串</font>             |
| byte                                                            | %c                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">字符（ASCII 码对应）</font>   |
|                                                                 | %d                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">字节的十进制值</font>         |
| rune                                                            | %c                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">Unicode 字符</font>      |
|                                                                 | %U                                                          | Unicode 码点                                                                 |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">任意类型</font> | %v                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">变量的默认格式</font>         |
|                                                                 | %T                                                          | <font style="color:rgba(0, 0, 0, 0.85) !important;">变量的类型</font>           |


## <font style="color:rgb(0, 0, 0) !important;">四、如何相互转换</font>
### 1.转换规则
**<font style="color:rgb(0, 0, 0) !important;">兼容类型之间可以直接转换</font>**<font style="color:rgb(0, 0, 0);">：</font>

+ 整数类型之间（如 </font>`<font style="color:rgb(0, 0, 0);">int</font>`<font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">↔</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);">int64</font>`<font style="color:rgb(0, 0, 0);">、</font>`<font style="color:rgb(0, 0, 0);">uint</font>`<font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">↔</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);">uint8</font>`<font style="color:rgb(0, 0, 0);">）</font>
+ <font style="color:rgb(0, 0, 0);">浮点数类型之间（如 </font>`<font style="color:rgb(0, 0, 0);">float32</font>`<font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">↔</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);">float64</font>`<font style="color:rgb(0, 0, 0);">）</font>
+ <font style="color:rgb(0, 0, 0);">整数与浮点数之间（如 </font>`<font style="color:rgb(0, 0, 0);">int</font>`<font style="color:rgb(0, 0, 0);"> </font><font style="color:rgb(0, 0, 0);">↔</font><font style="color:rgb(0, 0, 0);"> </font>`<font style="color:rgb(0, 0, 0);">float64</font>`<font style="color:rgb(0, 0, 0);">）</font>
+ `<font style="color:rgb(0, 0, 0);">byte</font>`<font style="color:rgb(0, 0, 0);"> 与 </font>`<font style="color:rgb(0, 0, 0);">int</font>`<font style="color:rgb(0, 0, 0);"> 之间（</font>`<font style="color:rgb(0, 0, 0);">byte</font>`<font style="color:rgb(0, 0, 0);"> 是 </font>`<font style="color:rgb(0, 0, 0);">uint8</font>`<font style="color:rgb(0, 0, 0);"> 的别名）</font>
+ `<font style="color:rgb(0, 0, 0);">rune</font>`<font style="color:rgb(0, 0, 0);"> 与 </font>`<font style="color:rgb(0, 0, 0);">int</font>`<font style="color:rgb(0, 0, 0);"> 之间（</font>`<font style="color:rgb(0, 0, 0);">rune</font>`<font style="color:rgb(0, 0, 0);"> 是 </font>`<font style="color:rgb(0, 0, 0);">int32</font>`<font style="color:rgb(0, 0, 0);"> 的别名）</font>

**<font style="color:rgb(0, 0, 0) !important;">不兼容类型转换会编译错误</font>**<font style="color:rgb(0, 0, 0);">：</font>

+ <font style="color:rgb(0, 0, 0);">整数 / 浮点数与 </font>`<font style="color:rgb(0, 0, 0);">bool</font>`<font style="color:rgb(0, 0, 0);"> 之间不能直接转换</font>
+ `<font style="color:rgb(0, 0, 0);">string</font>`<font style="color:rgb(0, 0, 0);"> 与其他基础类型之间不能直接转换（需要通过 </font>`<font style="color:rgb(0, 0, 0);">strconv</font>`<font style="color:rgb(0, 0, 0);"> 包）</font>

**<font style="color:rgb(0, 0, 0) !important;">转换可能导致精度损失或溢出</font>**<font style="color:rgb(0, 0, 0);">：</font>

+ <font style="color:rgb(0, 0, 0);">从大范围类型转换到小范围类型（如 </font>`<font style="color:rgb(0, 0, 0);">int64</font>`<font style="color:rgb(0, 0, 0);"> → </font>`<font style="color:rgb(0, 0, 0);">int32</font>`<font style="color:rgb(0, 0, 0);">）可能会溢出</font>
+ <font style="color:rgb(0, 0, 0);">从浮点数转换到整数会截断小数部分（不是四舍五入）</font>

### 2.转换方式
**显示类型转换**

```go
package main

import "fmt"

func main() {
    var a int = 42
    
    // int -> int64
    b := int64(a)
    fmt.Printf("a: %d (%T), b: %d (%T)\n", a, a, b, b)

    // float64 -> int (会截断小数部分)
    var c float64 = 3.99
    d := int(c)
    fmt.Printf("c: %f (%T), d: %d (%T)\n", c, c, d, d)

    // byte -> rune
    var e byte = 'A'
    f := rune(e)
    fmt.Printf("e: %c (%T), f: %c (%T)\n", e, e, f, f)
}
```

**使用strconv包转换**

适用范围：<font style="background-color:#FBDE28;">字符串转其他基础类型</font>

strconv提供了一系列函数来完成这些转换，它们通常成对出现：

+ ParseXxx将字符串转换为Xxx类型。FormatXxx将 Xxx 类型转换为字符串。

注意:使用 strconv的转换函数时，**<font style="color:rgb(0, 0, 0) !important;background-color:#FBDE28;">必须检查返回的错误</font>**

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    // 1. 字符串 -> 整数
    strNum := "12345"
    intNum, err := strconv.Atoi(strNum) // Atoi = ASCII to integer
    if err != nil {
        fmt.Println("转换失败:", err)
    } else {
        fmt.Printf("strNum: %s -> intNum: %d (%T)\n", strNum, intNum, intNum)
    }

    // 更灵活的 ParseInt
    bigNumStr := "9876543210"
    bigIntNum, err := strconv.ParseInt(bigNumStr, 10, 64) // 基数10进制，64位
    if err != nil {
        fmt.Println("转换失败:", err)
    } else {
        fmt.Printf("bigNumStr: %s -> bigIntNum: %d (%T)\n", bigNumStr, bigIntNum, bigIntNum)
    }

    // 2. 整数 -> 字符串
    num := 9876
    str := strconv.Itoa(num) // Itoa = Integer to ASCII //此处err省略了了
    fmt.Printf("num: %d -> str: %s (%T)\n", num, str, str)

    // 3. 字符串 -> 浮点数
    floatStr := "3.14159"
    floatNum, err := strconv.ParseFloat(floatStr, 64)
    if err != nil {
        fmt.Println("转换失败:", err)
    } else {
        fmt.Printf("floatStr: %s -> floatNum: %f (%T)\n", floatStr, floatNum, floatNum)
    }

    // 4. 浮点数 -> 字符串
    pi := 3.1415926
    piStr := strconv.FormatFloat(pi, 'f', 4, 64) // 格式'f'，保留4位小数，64位
    fmt.Printf("pi: %f -> piStr: %s (%T)\n", pi, piStr, piStr)

    // 5. 字符串 -> 布尔值
    boolStr1 := "true"
    boolVal1, err := strconv.ParseBool(boolStr1)
    if err != nil {
        fmt.Println("转换失败:", err)
    } else {
        fmt.Printf("boolStr1: %s -> boolVal1: %t (%T)\n", boolStr1, boolVal1, boolVal1)
    }

    // 6. 布尔值 -> 字符串
    isReady := false
    isReadyStr := strconv.FormatBool(isReady)
    fmt.Printf("isReady: %t -> isReadyStr: %s (%T)\n", isReady, isReadyStr, isReadyStr)
}
```

