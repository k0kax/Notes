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
token
### 测试函数