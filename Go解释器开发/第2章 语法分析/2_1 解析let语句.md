> 该篇涉及大量接口相关，注意查看相关文章[[../../Go基础/接口Interface|接口Interface]]

语法分析器parser是将前文生成的词法单元进一步转化为抽象语法树AST
### 一、ast抽象语法树设计

观察let的相关代码
```shell
示例如下：
let five = 5;  
let ten =10;  
let add = fn(x,y){  
	return x+y;  
};  
let result = add(five,ten);  
```
可知，let的功能就是将某个值，绑定到某个给定的名称上。这个名称就是个标识符，这个值就是个表达式。综合可以抽象为：
```
let <标识符indent>=<表达式expression>
```
five、ten、add都是标识符indent，5、10和函数字面量fn(x,y)都是表达式。
对let进行语法分析，也就是生成一个属于它的AST

实现let语句的语法树，需要两个不同的节点：表达式expression（纯expression）和语句statement（expression包含它）。表达式会产生值，语句不会。
```
原作者的话：
表达式会产生值，语句不会。let x = 5不会产生值，而5会产生值（产生的值就是5）。return 5; 不会产生值，但是add(5，5)会产生值。

表达式或语句到底是什么，哪些会产生值，哪些不会，这由编程语言决定。在某些语言中，函数字面量（例如fn(x, y){return x + y;}）是表达式，可以用在任何允许其他表达式使用地方。在另外一些编程语言中，函数字面量只能在程序顶层作为函数声明语句的一部分。还有某些语言具有if表达式，其中的条件语句部分是表达式并产生值。因此这些区别完全取决于语言设计师的选择。Monkey中的大多数是表达式，包括函数字面量。
```
故而初始定义为:
```go 
// ast/ast.go
package ast
type Node interface {//节点接口
	TokenLiteral() string//token字面方法，返回关联的字面量，用于调试
}
type Statement interface {//语句接口
	Node
	statementNode()//占位方法，并不必须
}
type Expression interface {//表达式接口
	Node
	expressionNode()//占位方法，并不必须
}
```
它包含了三个接口，Node节点(ast每个节点都要实现，不然连不到一起)，statement语句接口，expression表达式接口
#### AST根节点program
程序结构体Program，也就是整个AST的根节点，包含Statement语句接口的切片Statements
```go
// ast/ast.go
type Program struct {
	Statements []Statement //接口类型的切片
}
//获取字面量
func (p *Program) TokenLiteral() string {
	if len(p.Statements) > 0 {
		return p.Statements[0].TokenLiteral()//返回接口第一个token的字面量
	} else {
		return ""
	}
}
```
#### let语句的抽象语法树AST
此处的AST抽象语法树采取如下结构
![](https://raw.githubusercontent.com/k0kax/PicGo/main/images20251130150722302.png)
包括词法单元Token、标识符名称Name、表达式express（可能是值，也可能是方法公式之类的）
首先第一个字段是变量名Name，它对应标识符结构体Ident，还需要一个指向等号右侧的Value（产生值的表达式），对应表达式expression。
``` 
原作者的话：
为了持有绑定的标识符，let x = 5;中的x是Identifier结构类型，该类型实现了Expression接口。但是let语句中的标识符不会产生值，那么为什么要作为表达式来使用呢？答案是为了保持简单。Monkey程序其他地方的标识符会产生值，例如let x =valueProducingIdentifier;。为了减少AST中各种类型节点的数量，在变量绑定中使用Identifier表示名称，之后在表示表达式中的标识符的时候，可以复用Identifier节点。
```
因此LetStatement需要设计为：
```go 
//ast.go
type LetStatement struct {
	Token token.Token // token.LET词法单元
	Name  *Identifier //保存绑定的标识符Ident名称 字面量类型
	Value Expression  //产生值的表达式expression 表达式接口
}
```
还需要实现它的两个接口，语法节点statementNode()和token字面量TokenLiteral()
```go
//ast.go
func (ls *LetStatement) statementNode() {}
func (ls *LetStatement) TokenLiteral() string { return ls.Token.Literal }
```
为了绑定标识符x, `let x=5` 变量x为Ident标识符结构体
```go
//ast.go
type Identifier struct {
	Token token.Token // token.IDENT词法单元
	Value string //字面量的值
}
```
还需要对齐它的表达式接口expressionNode()及其字面量接口TokenLiteral()
```go
//ast.go
// 表达式节点
func (i *Identifier) expressionNode() {}

// 词法单元字面量
func (i *Identifier) TokenLiteral() string { return i.Token.Literal }
```
总代码ast/ast.go
```go
// ast/ast.go
package ast

//Abstrcat Syntax Tree 抽象语法树

//语法分析器将文本或者词法单元形式的源码作为输入，产生一个表示该源码的数据结构。
import "monkey_Interpreter/token"


// 接口1
// 用于返回字面量

type Node interface {
	TokenLiteral() string
}

// 接口2
// 语句
type Statement interface {
	Node
	statementNode() //语句节点
}

// 接口3
// 表达式
type Expression interface {
	Node
	expressionNode() //表达式节点
}

// 程序 根节点
type Program struct {
	Statements []Statement //接口类型的切片
}

// Token字面量
func (p *Program) TokenLiteral() string {
	if len(p.Statements) > 0 {
		return p.Statements[0].TokenLiteral()
	} else {
		return ""
	}
}

// 定义所需字段
type LetStatement struct {
	Token token.Token // token.LET词法单元
	Name  *Identifier //保存绑定的标识符名称
	Value Expression  //产生值的表达式
}

func (ls *LetStatement) statementNode()       {}                          //语句节点
func (ls *LetStatement) TokenLiteral() string { return ls.Token.Literal } //词法单元字面值

// 标识符
type Identifier struct {
	Token token.Token //token.IDENT词法单元
	Value string      //字面量值
}

// 表达式节点
func (i *Identifier) expressionNode() {}

// 词法单元字面量
func (i *Identifier) TokenLiteral() string { return i.Token.Literal }
```
### 二、语法分析器设计
##### 2.1语法分析器的结构
包括词法单元指针lexer，当前词法单元curToken，下一个词法单元peekToken，此处和[[1_1词法分析器]]的position/readPosition 类似
```go parser.go
// 语法分析器结构
type Parser struct {
	l *lexer.Lexer //指向词法分析器实例的指针

	curToken  token.Token //当前词法单元
	peekToken token.Token //当前词法单元的下一位
}
```
##### 2.2实例化语法分析器
需要先带入词法单元
然后设置curToken和peekToken，使得词法分析器不断执行
```go parser.go
// 初始化（实例化）语法分析器
func New(l *lexer.Lexer) *Parser {
	p := &Parser{
			l: l,
			errors:[]string,
		} //语法分析器实例

	//读取两个词法单元，以设置curToken和peekToken
	p.nextToken()
	p.nextToken()
	return p
}
```
### 2.3 让词法单元动起来
``` go
// 获取下一个词法单元 前移curToken和peekToken
func (p *Parser) nextToken() {
	p.curToken = p.peekToken
	p.peekToken = p.l.NextToken()//递归
}
```
### 2.4 解析

##### 解析程序
```go
func (p *Parser) ParseProgram() *ast.Program {

	program := &ast.Program{}              
	program.Statements = []ast.Statement{} //接口切片集

	for p.curToken.Type != token.EOF {
		stmt := p.parseStatement()
		if stmt != nil {
			program.Statements = append(program.Statements, stmt)
		}
		p.nextToken() //下移
	}
	return program
}
```
##### 解析总语句
以解析`let  x = 2;`为例 
注意由skipWhiteSpace()跳过空白
第一轮
```go
//parser.go
type Parser struct {
	l *lexer.Lexer //指向词法分析器实例的指针 

	curToken  token.Token //当前词法单元 let
	peekToken token.Token //当前词法单元的下一位 x
	errors []string//语法处理时的错误集合
}
```
注意
```go
type Token struct {  
Type TokenType //LET
// 字面量  
Literal string  //let
}
```
读到`LET`
对当前的词法单元的类型进行判断，如果是LET就进入专属的解析程序，返回一个let语句节点接口
```go
// 解析语句
func (p *Parser) parseStatement() ast.Statement {
	switch p.curToken.Type {
	case token.LET:
		return p.parseLetStatement()
	default:
		return nil
	}
}
```
##### 解析let语句parseLetStatement()
具体如下：
```go
// 解析let语句 以为例let x=5;
//此时：curtoken=let peektoken=x
func (p *Parser) parseLetStatement() *ast.LetStatement {
    stmt := &ast.LetStatement{Token: p.curToken}
    //运行后：stmt.token=let curtoken=let peektoken=x
    
    //1.检测标识符
    //检测下一个token(也就是peektoken)不是标识符indent，不是则退出（此处检测到为x是标识符，不退）,是则peektoken、curtoken后移一位
    if !p.expectPeek(token.IDENT) {//执行expectPeek(),检测到peektoken.type=IDENT,不执行{}内容，peektoken、curtoken都后移一位
        return nil
    }
    //运行后：stmt.token=let curtoken=x peektoken = "="

    //2.
    //将当前词法单元作为标识符的 Token 字段，并将其字面值literal作为标识符indent的值value赋给 stmt.Name
    stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	//运行后：stmt.token=let curtoken=x peektoken = "=" stmt.Name=&{Token: IDENT("x"), Value: "x"}
  
	

    //3.检测等号=
    //检查下一个token（peektoken）,,是则继续，不是则退出，peektoken、curtoken后移一位
    if !p.expectPeek(token.ASSIGN) {
        return nil
    }
	//运行后：stmt.token=let curtoken="=" peektoken = "5"  stmt.Name=&{Token: IDENT("x"), Value: "x"}
  
	//4.TODO: 跳过对表达式的处理parseExpression()
	//运行后：stmt.token=let curtoken="5" peektoken = ";"  stmt.Name=&{Token: IDENT("x"), Value: "x"} tmt.Value = &IntegerLiteral {Token: INT ("5"), Value: 5}
	
    //5.检测分号（;）     处理语句末尾的分号（;）
    //检测当前token（curtoken）是否是分号（;） 
    if !p.curTokenIs(token.SEMICOLON) {//是，则不需要移动
        p.nextToken() //不是，则peektoken、curtoken后移一位，直接解析下一句
    }
	//运行后：stmt.token=let curtoken=";" peektoken = ""  stmt.Name=&{Token: IDENT("x"), Value: "x"} stmt.
    
    //6.直接返回stmt
    return stmt 
    //LetStatement {Token: LET ("let"), Name: Identifier ("x"), Value: IntegerLiteral (5)}
}
```
##### 辅助：断言函数
```go
// 当前token判断
func (p *Parser) curTokenIs(t token.TokenType) bool {
	return p.curToken.Type == t
}

// 下一个token判断
func (p *Parser) peekTokenIs(t token.TokenType) bool {
	return p.peekToken.Type == t
}

// 用于判断下一个词法单元的类型是否与给定的类型匹配，并移动到下一个词法单元
func (p *Parser) expectPeek(t token.TokenType) bool {
	if p.peekTokenIs(t) {
		p.nextToken()
		return true
	} else {
		p.peekErrors(t)
		return false
	}
}
```
##### 辅助：错误检测处理函数
```go
// 错误检测
func (p *Parser) Errors() []string {
    return p.errors
}

func (p *Parser) peekErrors(t token.TokenType) {
    msg := fmt.Sprintf("expected next token to be “%s”,got=%s instead", t, p.peekToken.Type)
    p.errors = append(p.errors, msg)
}
```
#### 2.4总代码parse/parse.go
```go
package parser

  

//语法分析器

import (

    "fmt"

    "monkey_Interpreter/ast"

    "monkey_Interpreter/lexer"

    "monkey_Interpreter/token"

)

  

// 语法分析器结构

type Parser struct {

    l *lexer.Lexer //指向词法分析器实例的指针

  

    curToken  token.Token //当前词法单元

    peekToken token.Token //当前词法单元的下一位

  

    errors []string //错误集合

}

  

// 实例化语法分析器

func New(l *lexer.Lexer) *Parser {

    p := &Parser{l: l,

        errors: []string{},

    } //语法分析器实例

  

    //读取两个词法单元，以设置curToken和peekToken

    p.nextToken()

    p.nextToken()

    return p

}

  

// 获取下一个词法单元 前移curToken和peekToken

func (p *Parser) nextToken() {

    p.curToken = p.peekToken

    p.peekToken = p.l.NextToken() //递归

}

  

// 入口点

// 解析程序AST的根节点

func (p *Parser) ParseProgram() *ast.Program {

  

    program := &ast.Program{}              //声明构造ast根节点program

    program.Statements = []ast.Statement{} //语句接口切片集

  

    for p.curToken.Type != token.EOF { //碰到词法法单元Token EOF文件结尾 表示已将遍历完终止

        stmt := p.parseStatement() //解析具体语法

        if stmt != nil {

            program.Statements = append(program.Statements, stmt) //不断解析语句，并且存到statements切片中

        }

        p.nextToken() //下移

    }

  

    return program

}

  

// 解析语句

func (p *Parser) parseStatement() ast.Statement {

    switch p.curToken.Type {

    case token.LET:

        return p.parseLetStatement()

    default:

        return nil

    }

}

  

// 解析let语句 以为例let x=5;

// 此时：curtoken=let peektoken=x

func (p *Parser) parseLetStatement() *ast.LetStatement {

    //1.初始化 LetStatement 节点：把当前 curToken（就是 LET 类型的 "let"）绑定到节点

    stmt := &ast.LetStatement{Token: p.curToken}

    //运行后：stmt.token=let curtoken=let peektoken=x

  

    //2.检测标识符

    //检测下一个token(也就是peektoken)不是标识符indent，不是则退出（此处检测到为x是标识符，不退）,是则peektoken、curtoken后移一位

    if !p.expectPeek(token.IDENT) { //执行expectPeek(),检测到peektoken.type=IDENT,不执行{}内容，peektoken、curtoken都后移一位

        return nil

    }

    //运行后：stmt.token=let curtoken=x peektoken = "="

  

    //3.

    //将当前词法单元作为标识符的 Token 字段，并将其字面值literal作为标识符indent的值value赋给 stmt.Name

    stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}

    //运行后：stmt.token=let curtoken=x peektoken = "=" stmt.Name=&{Token: IDENT("x"), Value: "x"}

  

    //4.检测等号=

    //检查下一个token（peektoken）,,是则继续，不是则退出，peektoken、curtoken后移一位

    if !p.expectPeek(token.ASSIGN) {

        return nil

    }

    //运行后：stmt.token=let curtoken="=" peektoken = "5"  stmt.Name=&{Token: IDENT("x"), Value: "x"}

  

    //5.TODO: 跳过对表达式的处理parseExpression()

    //运行后：stmt.token=let curtoken="5" peektoken = ";"  stmt.Name=&{Token: IDENT("x"), Value: "x"} tmt.Value = &IntegerLiteral {Token: INT ("5"), Value: 5}

  

    //6.检测分号（;）     处理语句末尾的分号（;）

    //检测当前token（curtoken）是否是分号（;）

    if !p.curTokenIs(token.SEMICOLON) { //是，则不需要移动

        p.nextToken() //不是，则peektoken、curtoken后移一位，直接解析下一句

    }

    //运行后：stmt.token=let curtoken=";" peektoken = ""  stmt.Name=&{Token: IDENT("x"), Value: "x"} stmt.

  

    //7.直接返回stmt

    return stmt

    //LetStatement {Token: LET ("let"), Name: Identifier ("x"), Value: IntegerLiteral (5)}

  

}

  

// 辅助断言函数

// 当前token判断

func (p *Parser) curTokenIs(t token.TokenType) bool {

    return p.curToken.Type == t

}

  

// 下一个token判断

func (p *Parser) peekTokenIs(t token.TokenType) bool {

    return p.peekToken.Type == t

}

  

// 用于判断下一个词法单元的类型是否与给定的类型匹配，并移动到下一个词法单元

func (p *Parser) expectPeek(t token.TokenType) bool {

    if p.peekTokenIs(t) {

        p.nextToken() //后移一位 curtoken变成peektoken,peektoken变成peektoken的下一位

        return true

    } else {

        p.peekErrors(t)

        return false

    }

}

  

// 错误检测

func (p *Parser) Errors() []string {

    return p.errors

}

  

func (p *Parser) peekErrors(t token.TokenType) {
    msg := fmt.Sprintf("expected next token to be “%s”,got=%s instead", t, p.peekToken.Type)
    p.errors = append(p.errors, msg)
}

```
### 三、测试函数

parser/parser_test.go

```go
package parser

import (
	"monkey_Interpreter/ast"
	"monkey_Interpreter/lexer"
	"testing"
)

func TestLetStatements(t *testing.T) {
	//输入
	input := `let x = 5;
			let y = 10;
			let foobar = 838383;`
	l := lexer.New(input) //词词法分析器实例
	p := New(l)           //语法解析器

	program := p.ParseProgram() //解析程序，并将返回的抽象语法树（AST）存储在变量program中

	//非空检查
	if program == nil {
		t.Fatalf("ParseProgram() returned nil")
	}

	//程序语法长度检查
	if len(program.Statements) != 3 {
		t.Fatalf("program.Statements does not contain 3 statements.got=%d",
			len(program.Statements))
	}

	tests := []struct {
		expectedIdentifier string //预期的标识符
	}{
		{"x"},
		{"y"},
		{"foobar"},
	}

	//遍历
	//i是循环索引，表示当前迭代的索引位置，从0开始递增。tt 是一个临时变量，它代表 tests 切片中的每个元素。
	for i, tt := range tests {
		stmt := program.Statements[i]
		if !testLetStatement(t, stmt, tt.expectedIdentifier) {
			return
		}
	}
}

// 测试let语句的断言 期望字面量名称
func testLetStatement(t *testing.T, s ast.Statement, name string) bool {
	if s.TokenLiteral() != "let" { //字面量检查
		t.Errorf("s.TokenLiteral not 'let'. got=%q", s.TokenLiteral())//报错
		return false
	}

	//将s转换为*ast.LetStatement类型
	letStmt, ok := s.(*ast.LetStatement) //断言
	if !ok {
		t.Errorf("s not *ast.LetStatement. got=%T", s)
		return false
	}
	//字面值
	if letStmt.Name.Value != name {
		t.Errorf("letStmt.Name.Value not '%s'. got=%s", name, letStmt.Name.Value)
		return false
	}
	//字面值
	if letStmt.Name.TokenLiteral() != name {
		t.Errorf("letStmt.Name.TokenLiteral() not '%s'. got=%s",
			name, letStmt.Name.TokenLiteral())
		return false
	}
	return true
}

```
测试结果
![](https://cdn.jsdelivr.net/gh/k0kax/PicGo@main/image/202311142217388.png)