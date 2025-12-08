中缀表达式将实现如下：
```go
5+5
5-5
5*5
5/5
5>5
5<5
5==5
5!=5
```
可以抽象为：`<表达式><中缀运算符><表达式>`

构建AST
```go
//ast.go
type InfixExpression struct {
	Token token.Token // 运算符词法单元，如+
	Left Expression
	Operator string
	Right Expression
}

func (ie *InfixExpression) expressionNode() {}
func (ie *InfixExpression) TokenLiteral() string { return ie.Token.Literal }
func (ie *InfixExpression) String() string {
	var out bytes.Buffer
	
	out.WriteString("(")
	out.WriteString(ie.Left.String())
	out.WriteString(" " + ie.Operator + " ")
	out.WriteString(ie.Right.String())
	out.WriteString(")")
	return out.String()
}
```
设置token的优先级映射表及辅助方法
```go
//parser.go
var precedences = map[token.TokenType]int{
	token.EQ: EQUALS,
	token.NOT_EQ: EQUALS,
	token.LT: LESSGREATER,
	token.GT: LESSGREATER,
	token.PLUS: SUM,
	token.MINUS: SUM,
	token.SLASH: PRODUCT,
	token.ASTERISK: PRODUCT,
}
//获取下一个token的优先级（peekToken）
func (p *Parser) peekPrecedence() int {
	if p, ok := precedences[p.peekToken.Type]; ok {
		return p
	}
	return LOWEST
}
//获取当前token的优先级
func (p *Parser) curPrecedence() int {
	if p, ok := precedences[p.curToken.Type]; ok {
		return p
	}
	return LOWEST
}
```

设置中缀解析函数
```go

```