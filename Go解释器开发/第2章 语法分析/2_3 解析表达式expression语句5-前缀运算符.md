monkey语言有两个前缀运算符：`!和-`。举例如下：
```go
-5;
!foobar;
5+-10;
```
可以抽象为`<前缀运算符><表达式>`
构造AST
```go
// ast.go
// ------------------------------------------------前缀运算符的AST-----------------------------------------
// <前缀运算符><表达式>
type PrefixExpression struct {
    Token    token.Token
    Operator string     //可能是- ！
    Right    Expression //运算符右边的表达式
}

func (pe *PrefixExpression) expressionNode()      {}
func (pe *PrefixExpression) TokenLiteral() string { return pe.Token.Literal }
func (pe *PrefixExpression) String() string {
    var out bytes.Buffer
    
    out.WriteString("(")
    out.WriteString(pe.Operator)
    out.WriteString(pe.Right.String())
    out.WriteString(")")

    return out.String()
}
```
扩展语法分析器
```go
//parser.go

```