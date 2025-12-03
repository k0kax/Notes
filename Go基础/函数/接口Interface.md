go语言的接口是一种抽象类型，是系列方法的集合。接口做的事就是定义一个协议（规则），只要一台机器具有洗衣和甩干的功能，就称之为洗衣机。不关心属性（数据），只关心行为（方法）。

**核心：只声明方法，不实现方法**
# 一、接口
### 2、为啥用接口？
如下代码：
``` go
type Cat struct{}

func (c Cat) Say() string { return "喵喵喵" }

type Dog struct{}

func (d Dog) Say() string { return "汪汪汪" }

func main() {
    c := Cat{}
    fmt.Println("猫:", c.Say())
    d := Dog{}
    fmt.Println("狗:", d.Say())
}
```
上面的代码，猫狗都会叫，在main里面就重复了，如果还有其他动物，就更加冗杂。相同的叫的行为可以统一为一个会叫的动物来处理，这就是接口。
### 2、接口的定义
接口是一个或多个方法签名的集合。任何类型的方法集中只要拥有该接口对应的全部方法签名。就表示它 "实现" 了该接口，无须在该类型上显式声明实现了哪个接口。
所谓对应方法，是指有相同名称、参数列表 (不包括参数名) 以及返回值。  当然，该类型还可以有其他方法。
- 接口只有方法声明，没有实现，没有数据字段。
- 接口可以匿名嵌入其他接口，或嵌入到结构中。
- 对象赋值给接口时，会发生拷贝，而接口内部存储的是指向这个复制品的指针，既无法修改复制品的状态，也无法获取指针。
- 只有当接口存储的类型和对象都为nil时，接口才等于nil。
- 接口调用不会做receiver的自动转换。
- 接口同样支持匿名字段方法。
- 接口也可实现类似OOP中的多态。
- 空接口可以作为任何类型数据的容器。
- 一个类型可实现多个接口。
- 接口命名习惯以 er 结尾。
接口的定义格式如下：
``` go
type 接口类型名 interface{
        方法名1( 参数列表1 ) 返回值列表1
        方法名2( 参数列表2 ) 返回值列表2
        …
    }
    
//1.接口名：使用type将接口定义为自定义的类型名。Go语言的接口在命名时，一般会在单词后面添加er，如有写操作的接口叫Writer，有字符串功能的接口叫Stringer等。接口名最好要能突出该接口的类型含义。
//2.方法名：当方法名首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包（package）之外的代码访问。
//3.参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略。
//例如：

type writer interface{
    Write([]byte) error
}
```
### 3、实现接口的条件
一个对象只要全部实现了接口中的方法，那它就实现了接口。接口就是一个需要被实现得到方法列表
``` go
// Sayer 接口
type Sayer interface {
    say()
}

type dog struct {}
type cat struct {}

// dog实现了Sayer接口
func (d dog) say() {
    fmt.Println("汪汪汪")
}

// cat实现了Sayer接口
func (c cat) say() {
    fmt.Println("喵喵喵")
}
```
### 4、接口类型变量
实现接口咋用？
``` go
func main() {
    var x Sayer // 声明一个Sayer类型的变量x
    a := cat{}  // 实例化一个cat
    b := dog{}  // 实例化一个dog
    x = a       // 可以把cat实例直接赋值给x
    x.say()     // 喵喵喵
    x = b       // 可以把dog实例直接赋值给x
    x.say()     // 汪汪汪
}
```
如上，Sayer类型的变量x能够存储dog和cat类型的变量a和b。
### 5、值接收者和指针接受者实现接口的区别
有一个Mover接口和一个dog结构体
``` go
type Mover interface {
    move()
}

type dog struct {}
```
#### 值接收者实现窗口
``` go
func (d dog) move() {
    fmt.Println("狗会动")
}

func main() {
    var x Mover
    var wangcai = dog{} // 旺财是dog类型
    x = wangcai         // x可以接收dog类型
    var fugui = &dog{}  // 富贵是*dog类型
    x = fugui           // x可以接收*dog类型
    x.move()
}
```
不管是dog结构体还是结构体指针`*dog`类型的变量都可以赋值给该接口变量。因为Go语言中有对指针类型变量求值的语法糖，dog指针fugui内部会自动求值`*fugui`。
#### 指针接受者实现接口
```go
func (d *dog) move() {
    fmt.Println("狗会动")
}
func main() {
    var x Mover
    var wangcai = dog{} // 旺财是dog类型
    x = wangcai         // x不可以接收dog类型
    var fugui = &dog{}  // 富贵是*dog类型
    x = fugui           // x可以接收*dog类型
}
```
此时实现Mover接口的是`*dog`类型，所以不能给x传入dog类型的wangcai，此时x只能存储`*dog`类型的值。
总的来说，值接收者可以用值和指针，指针接收者只能用指针
## 二、类型与接口的关系
### 1.一个类型实现多个接口
同一个类型可以同时实现多个接口，且接口彼此独立。狗能叫，也能动
``` go
// Sayer 接口
type Sayer interface {
    say()
}

// Mover 接口
type Mover interface {
    move()
}

type dog struct {
    name string
}

// 实现Sayer接口
func (d dog) say() {
    fmt.Printf("%s会叫汪汪汪\n", d.name)
}

// 实现Mover接口
func (d dog) move() {
    fmt.Printf("%s会动\n", d.name)
}

func main() {
    var x Sayer
    var y Mover

    var a = dog{name: "旺财"}
    x = a
    y = a
    x.say()
    y.move()
}
```
### 2.多个类型实现同一接口
不同类型可以实现同一接口，狗和车都可以动如下
``` go
// Mover 接口
type Mover interface {
    move()
}

type dog struct {
    name string
}

type car struct {
    brand string
}

// dog类型实现Mover接口
func (d dog) move() {
    fmt.Printf("%s会跑\n", d.name)
}

// car类型实现Mover接口
func (c car) move() {
    fmt.Printf("%s速度70迈\n", c.brand)
}

func main() {
    var x Mover
    var a = dog{name: "旺财"}
    var b = car{brand: "保时捷"}
    x = a
    x.move()
    x = b
    x.move()
}
```
并且一个接口的方法，不一定需要由一个类型完全实现，接口的方法可以通过在类型中嵌入其他类型或者结构体来实现。
``` go
// WashingMachine 洗衣机
type WashingMachine interface {
    wash()
    dry()
}

// 甩干器
type dryer struct{}

// 实现WashingMachine接口的dry()方法
func (d dryer) dry() {
    fmt.Println("甩一甩")
}

// 海尔洗衣机
type haier struct {
    dryer //嵌入甩干器
}

// 实现WashingMachine接口的wash()方法
func (h haier) wash() {
    fmt.Println("洗刷刷")
}
```
### 3.接口嵌套
接口与接口间可以通过嵌套创造出新的接口。嵌套得到的接口的使用与普通接口一样
```go
// Sayer 接口
type Sayer interface {
    say()
}

// Mover 接口
type Mover interface {
    move()
}

// 接口嵌套
type animal interface {
    Sayer
    Mover
}

type cat struct {
    name string
}

func (c cat) say() {
    fmt.Println("喵喵喵")
}

func (c cat) move() {
    fmt.Println("猫会动")
}

func main() {
    var x animal
    x = cat{name: "花花"}
    x.move()
    x.say()
}
```
## 三、空接口
### 1.定义
空接口是指没有定义任何方法的接口。因此任何类型都实现了空接口。
空接口类型的变量可以存储任意类型的变量。
``` go
func main() {
    // 定义一个空接口x
    var x interface{}
    s := "pprof.cn"
    x = s
    fmt.Printf("type:%T value:%v\n", x, x)
    i := 100
    x = i
    fmt.Printf("type:%T value:%v\n", x, x)
    b := true
    x = b
    fmt.Printf("type:%T value:%v\n", x, x)
}
```
### 2.应用
#### 作为函数的参数
使用空接口实现可以接收任意类型的函数参数。
``` go
// 空接口作为函数参数
func show(a interface{}) {
    fmt.Printf("type:%T value:%v\n", a, a)
}
```
#### 作为map的值
使用空接口实现可以保存任意值的字典。
``` go
// 空接口作为map值
    var studentInfo = make(map[string]interface{})
    studentInfo["name"] = "李白"
    studentInfo["age"] = 18
    studentInfo["married"] = false
    fmt.Println(studentInfo)
```

### 3.类型断言
一个接口的值（简称接口值）是由一个具体类型和具体类型的值两部分组成的。这两部分分别称为接口的动态类型和动态值。如下：
``` go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```
分解如下：
![](https://raw.githubusercontent.com/k0kax/PicGo/main/images20251130162842434.png)

想要判断空接口中的值这个时候就可以使用类型断言，其语法格式为：`x.(T)`，其中：
```go
 x：表示类型为interface{}的变量
 T：表示断言x可能是的类型。
```
该语法返回两个参数，第一个参数是x转化为T类型后的变量，第二个值是一个布尔值，若为true则表示断言成功，为false则表示断言失败。
```go
func main() {
    var x interface{}
    x = "pprof.cn"
    v, ok := x.(string)
    if ok {
        fmt.Println(v)
    } else {
        fmt.Println("类型断言失败")
    }
}
```
上面的示例中如果要断言多次就需要写多个if判断，这个时候我们可以使用switch语句来实现：
``` go
func justifyType(x interface{}) {
    switch v := x.(type) {
    case string:
        fmt.Printf("x is a string，value is %v\n", v)
    case int:
        fmt.Printf("x is a int is %v\n", v)
    case bool:
        fmt.Printf("x is a bool is %v\n", v)
    default:
        fmt.Println("unsupport type！")
    }
}
```
因为空接口可以存储任意类型值的特点，所以空接口在Go语言中的使用十分广泛。

关于接口需要注意的是，只有当有两个或两个以上的具体类型必须以相同的方式进行处理时才需要定义接口。不要为了接口而写接口，那样只会增加不必要的抽象，导致不必要的运行时损耗。