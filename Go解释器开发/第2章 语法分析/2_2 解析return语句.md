好吧，有了前面的经验我们的第一步是确定return有那些形式
```go
return 5;
return 10;
return add(15);
```
可以抽象为`return <表达式>;`
### 添加return的token词法单元
```go
//token.go
const(
//......

// 关键字
    FUNCTION = "FUNCTION"
    LET      = "LET"
    //*********
    IF     = "IF"
    ELSE   = "ELSE"
    RETURN = "RETURN"
    
//......
)
```
### 添加return语句的AST
```go 
//ast.go
// return相关
type ReturnStatement struct {
    Token       token.Token //return词法单元
    ReturnVaule Expression  //返回值
}

func (rs *ReturnStatement) statementNode()       {}
func (rs *ReturnStatement) TokenLiteral() string { return rs.Token.Literal }
```
### 添加解析器
添加解析词法单元
```go
//parser.go
func (p *Parser) parseStatement() ast.Statement {
	switch p.curToken.Type {
	case token.LET:
		return p.parseLetStatement()
	case token.RETURN:
		return p.parseReturnStatement()
	default:
	return nil
	}
}
```
设置解析函数parseReturnStatement()
```go
//parser.go
func (p *Parser) parseReturnStatement() *ast.ReturnStatement {

    stmt := &ast.ReturnStatement{Token: p.curToken}
    p.nextToken()
    //TODO:跳过对表达式的处理，直接遇到分号

    for !p.curTokenIs(token.SEMICOLON) {
        p.nextToken()
    }

    return stmt
}
```
相关代码：
ast.go:
```go
// ast/ast.go

package ast
//Abstrcat Syntax Tree 抽象语法树
//普拉特语法分析器，自上而下
//语法分析器将文本或者词法单元形式的源码作为输入，产生一个表示该源码的数据结构。
import "monkey_Interpreter/token"

  

//三个接口
// 接口1
// 用于返回字面量
type Node interface {
    TokenLiteral() string
}

// 接口2
// 语句
type Statement interface {
    Node            //继承node接口
    statementNode() //语句节点
}

// 接口3
// 表达式
type Expression interface {
    Node             //继承node接口
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
    Name  *Identifier //保存绑定的标识符名称
    Value Expression  //产生值的表达式

}

  

//对齐前面两个接口
func (ls *LetStatement) statementNode()       {}                          //语句节点
func (ls *LetStatement) TokenLiteral() string { return ls.Token.Literal } //词法单元字面值

// 标识符
type Identifier struct {
    Token token.Token //token.IDENT词法单元
    Value string      //字面量值
}

  

// 语句节点
func (i *Identifier) expressionNode() {}

// 词法单元字面量
func (i *Identifier) TokenLiteral() string { return i.Token.Literal }

// return相关
type ReturnStatement struct {
    Token       token.Token //return词法单元
    ReturnVaule Expression  //返回值
}

  

func (rs *ReturnStatement) statementNode()       {}
func (rs *ReturnStatement) TokenLiteral() string { return rs.Token.Literal }
```
token.go
```go
package token

  

// 词法单元类型
type TokenType string

// 词法单元
type Token struct {

    Type TokenType

    // 字面量

    Literal string

}

  

// 声明一些词法常量

const (

    // 特殊类型

    ILLEGAL = "ILLEGAL" // 未知字符

    EOF     = "EOF"     // 文件结尾

  

    // 标识符+字面量

    IDENT = "IDENT" // add, foobar, x, y

    INT   = "INT"   // 1343456

  

    // 运算符

    ASSIGN   = "="

    PLUS     = "+"

    MINUS    = "-"

    BANG     = "!"

    ASTERISK = "*"

    SLASH    = "/"

  

    // 分隔符

    COMMA     = ","

    SEMICOLON = ";"

  

    LT     = "<"

    GT     = ">"

    LPAREN = "("

    RPAREN = ")"

    LBRACE = "{"

    RBRACE = "}"

  

    // 关键字

    FUNCTION = "FUNCTION"

    LET      = "LET"

    //*********

    IF     = "IF"

    ELSE   = "ELSE"

    RETURN = "RETURN"

  

    //比较运算符

    EQ     = "=="

    NOT_EQ = "!="

)

  

//！-/*5；

//          5 < 10 > 5;

  

// 关键词表 哈希表

var keywords = map[string]TokenType{

    "fn":  FUNCTION,

    "let": LET,

    //*********

    "if":     IF,

    "else":   ELSE,

    "return": RETURN,

}

  

// 通过关键词表判断给定的标识符是否是关键词

func LookupIdent(ident string) TokenType {

    if tok, ok := keywords[ident]; ok {

        return tok

    }

    return IDENT

}
```
parser.go
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

    case token.RETURN:

        return p.parseReturnStatement()

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

  

func (p *Parser) parseReturnStatement() *ast.ReturnStatement {

    stmt := &ast.ReturnStatement{Token: p.curToken}

  

    p.nextToken()

  

    //TODO:跳过对表达式的处理，直接遇到分号

    for !p.curTokenIs(token.SEMICOLON) {

        p.nextToken()

    }

  

    return stmt

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
### 测试函数
parser_test.go
```go
package parser

import (
    "monkey_Interpreter/ast"
    "monkey_Interpreter/lexer"
    "testing"
)

// 测试解析错误
func checkParserErrors(t *testing.T, p *Parser) {
    errors := p.Errors()
    if len(errors) == 0 {
        return
    }
    t.Errorf("parser has %d errors", len(errors))
    for _, msg := range errors {
        t.Errorf("parser error: %q", msg)
    }
    t.FailNow()
}

  
func TestReturnStatements(t *testing.T) {
    input := `
return 5;
return 10;
return 993 322;
`
    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()

    checkParserErrors(t, p)
    if len(program.Statements) != 3 {
        t.Fatalf("program.Statements does not contain 3 statements. got=%d",
            len(program.Statements))
    }

    for _, stmt := range program.Statements {
        returnStmt, ok := stmt.(*ast.ReturnStatement)
        if !ok {
            t.Errorf("stmt not *ast.ReturnStatement. got=%T", stmt)
            continue
        }
        if returnStmt.TokenLiteral() != "return" {
            t.Errorf("returnStmt.TokenLiteral not 'return', got %q",
                returnStmt.TokenLiteral())
        }
    }
}
```
测试结果：
```go
Running tool: C:\Program Files\Go\bin\go.exe test -timeout 30s -run ^TestReturnStatements$ monkey_Interpreter/parser

=== RUN   TestReturnStatements
--- PASS: TestReturnStatements (0.00s)
PASS
ok      monkey_Interpreter/parser       0.125s
```