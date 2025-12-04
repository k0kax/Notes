自下而上的运算符优先级分析（普拉特解析法）

## 专业术语
前缀运算符
后缀运算符
中缀运算符
运算符优先级


## 构建表达式expression的AST

```go
//ast.go
type ExpressionStatement struct {
    Token      token.Token //该表达式的第一个词法单元
    Expression Expression  //保存表达式
}

func (es *ExpressionStatement) statementNode()       {}
func (es *ExpressionStatement) TokenLiteral() string { return es.Token.Literal }
```
表达式语句ExpressionStatement的结构体主要由两个字段组成Token和Expression(保存表达式)。实现节点接口Node的TokenLiteral()方法和语句接口Statement的statementNode()方法，故可以加入到program的Statements切片。

为所有AST添加String()方法，返回完整的字符串，用于调试和比较
```go
//ast.go
type Node interface {
    TokenLiteral() string //token字面量
    String() string       //token的string形式，用于调试
}
```

再根节点program添加String()方法
```go
// ast.go
// 主要用于调试时打印ast节点
func (p *Program) String() string {
    var out bytes.Buffer //创建缓冲区

    for _, s := range p.Statements {
        out.WriteString(s.String()) //将每条语句的String()方法返回值写入缓冲区
    }

    return out.String() //以字符串形式返回
}
```
为Let、Return、表达式express的AST添加string()方法。还有字面量。此处涉及到字节流的使用
``` go
// ast.go
func (ls *LetStatement) String() string {
    var out bytes.Buffer
    
    out.WriteString(ls.TokenLiteral() + " ")
    out.WriteString(ls.Name.String())
    out.WriteString("=")

    if ls.Value != nil {
        out.WriteString(ls.Value.String())
    }

    out.WriteString(";")
    return out.String()
}

func (rs *ReturnStatement) String() string {
    var out bytes.Buffer

    out.WriteString(rs.TokenLiteral() + " ")

    if rs.ReturnVaule != nil {
        out.WriteString(rs.ReturnVaule.String())
    }
    
    out.WriteString(";")
    return out.String()

}

func (es *ExpressionStatement) String() string {
    if es.Expression != nil {
        return es.Expression.String()
    }
    return ""
}

func (i *Identifier) String() string { return i.Value }
```
测试AST
```go
//ast_test.go
package ast

import (
    "monkey_Interpreter/token"
    "testing"
)


// 测试：let myVar = anotherVar
func TestString(t *testing.T) {
    program := &Program{
        Statements: []Statement{
            &LetStatement{
                Token: token.Token{Type: token.LET, Literal: "let"},
                Name: &Identifier{
                    Token: token.Token{Type: token.IDENT, Literal: "myVar"},
                    Value: "myVar",
                },
                Value: &Identifier{
                    Token: token.Token{Type: token.IDENT, Literal: "anotherVar"},
                    Value: "anotherVar",
                },
            },
        },
    }

    if program.String() != "let myVar=anotherVar;" {
        t.Errorf("program.String() wrong.got=%q", program.String())
    }
}
```
