新的repl加入了解析语法的部分
```go
// repl/repl.go
package repl

import (
	"bufio"
	"fmt"
	"io"
	"monkey_Interpreter/lexer"
	"monkey_Interpreter/parser"
)

const PROMPT = ">>" //表示 REPL 提示符

// 函数接受输入流 in 和输出流 out 作为参数
func Start(in io.Reader, out io.Writer) {

	//在函数内部，创建了一个 bufio.Scanner 对象 scanner，用于从输入流中读取用户输入
	scanner := bufio.NewScanner(in)

	for {
		//使用 fmt.Fprintf 函数将提示符输出到输出流 out
		fmt.Fprintf(out, PROMPT)

		//调用 scanner.Scan() 方法来等待用户输入，并返回一个布尔值表示是否成功读取到输入
		scanned := scanner.Scan()

		if !scanned {
			return
		}

		//如果成功读取到输入，将用户输入的文本保存在变量 line 中
		line := scanner.Text()

		//创建了一个 lexer.Lexer 对象 l，并使用用户输入的文本作为输入来初始化该对象
		l := lexer.New(line)

		//语法解析
		p := parser.New(l)
		program := p.ParseProgram()

		if len(p.Errors()) != 0 {
			printParserErrors(out, p.Errors())
			continue
		}

		io.WriteString(out, program.String())
		io.WriteString(out, "\n")

	}
}

// 写入错误
func printParserErrors(out io.Writer, errors []string) {
	for _, msg := range errors {
		io.WriteString(out, "\t"+msg+"\n")
	}
}
```

主程序
```go
//main.go
package main

import (
	"fmt"
	"monkey_Interpreter/repl"
	"os"
	"os/user"
)

func main() {
	//在 main 函数内部，首先使用 os/user 包中的 user.Current() 函数获取当前用户的信息
	user, err := user.Current()
	if err != nil {
		panic(err)
	}

	fmt.Printf("Hello %s! This is the Monkey programmimg language!\n", user.Username)
	fmt.Printf("Feel freee to type in commamds\n")
	repl.Start(os.Stdin, os.Stdout)
}

```

运行结果：
```go
PS E:\Codes\Go\monkey_Interpreter> go run .\main.go
Hello ! This is the Monkey programmimg language!
Feel freee to type in commamds
>>let x=1*3*3+12+2*6
let x=((((1 * 3) * 3) + 12) + (2 * 6));
>>let x=add(1,2,7*7);
let x=add(1,2,(7 * 7));
>>true == true
(true == true)
>>
```

在错误提示中加入猴子脸
```go
// repl/repl.go
package repl

import (
	"bufio"
	"fmt"
	"io"
	"monkey_Interpreter/lexer"
	"monkey_Interpreter/parser"
)

const PROMPT = ">>" //表示 REPL 提示符
const MONKEY_FACE = `            __,__
   .--.  .-"     "-.  .--.
  / .. \/  .-. .-.  \/ .. \
 | |  '|  /   Y   \  |'  | |
 | \   \  \ 0 | 0 /  /   / |
  \ '- ,\.-"""""""-./, -' /
   ''-' /_   ^ ^   _\ '-''
       |  \._   _./  |
       \   \ '~' /   /
        '._ '-=-' _.'
           '-----'
`

// 函数接受输入流 in 和输出流 out 作为参数
func Start(in io.Reader, out io.Writer) {

	//在函数内部，创建了一个 bufio.Scanner 对象 scanner，用于从输入流中读取用户输入
	scanner := bufio.NewScanner(in)

	for {
		//使用 fmt.Fprintf 函数将提示符输出到输出流 out
		fmt.Fprintf(out, PROMPT)

		//调用 scanner.Scan() 方法来等待用户输入，并返回一个布尔值表示是否成功读取到输入
		scanned := scanner.Scan()

		if !scanned {
			return
		}

		//如果成功读取到输入，将用户输入的文本保存在变量 line 中
		line := scanner.Text()

		//创建了一个 lexer.Lexer 对象 l，并使用用户输入的文本作为输入来初始化该对象
		l := lexer.New(line)

		//语法解析
		p := parser.New(l)
		program := p.ParseProgram()

		if len(p.Errors()) != 0 {
			printParserErrors(out, p.Errors())
			continue
		}

		io.WriteString(out, program.String())
		io.WriteString(out, "\n")

	}
}

// 写入错误
func printParserErrors(out io.Writer, errors []string) {
	io.WriteString(out, MONKEY_FACE)
	io.WriteString(out, "Woops! We ran into some monkey business here!\n")
	io.WriteString(out, " parser errors:\n")
	for _, msg := range errors {
		io.WriteString(out, "\t"+msg+"\n")
	}
}
```
