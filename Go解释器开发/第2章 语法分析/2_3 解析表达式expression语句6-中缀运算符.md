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
//parser.go
// 中缀解析函数
func (p *Parser) parseInfixExpression(left ast.Expression) ast.Expression {
	// 1. 初始化中缀表达式节点（+）
	expression := &ast.InfixExpression{
		Token:    p.curToken,         //+
		Operator: p.curToken.Literal, //“+”
		Left:     left,               //1
	}

	precedence := p.curPrecedence()                  //记录当前词法单元+的优先级 SUM
	p.nextToken()                                    //curToken=INT(2)，peekToken=ASTERISK(*)
	expression.Right = p.parseExpression(precedence) //递归调用 parseExpression(SUM) 解析右边的 2*3（重点！）

	return expression
}
```
后面会举例介绍具体工作流程

注册解析函数
```go
//parser.go
// 实例化语法分析器
func New(l *lexer.Lexer) *Parser {
	p := &Parser{l: l,
		errors: []string{},
	} //语法分析器实例

	//读取两个词法单元，以设置curToken和peekToken
	p.nextToken()
	p.nextToken()

	//关联解析函数
	//前缀解析函数
	p.prefixParseFns = make(map[token.TokenType]prefixParseFn) //初始化映射
	p.registerPrefix(token.IDENT, p.parseIdentifier)           //注册ident标识符相关的解析函数（parseIdentifier）
	p.registerPrefix(token.INT, p.parseIntegerLiteral)         //注册integer整形相关的解析函数（parseIntegerLiteral）
	p.registerPrefix(token.BANG, p.parsePrefixExpression)      //注册！非的解析函数
	p.registerPrefix(token.MINUS, p.parsePrefixExpression)     //注册-负号的解析函数

	//中缀解析函数
	p.infixParseFns = make(map[token.TokenType]infixParseFn)
	p.registerInfix(token.PLUS, p.parseInfixExpression)     //注册加号+的解析函数
	p.registerInfix(token.MINUS, p.parseInfixExpression)    //注册-的解析函数
	p.registerInfix(token.SLASH, p.parseInfixExpression)    //注册/的解析函数
	p.registerInfix(token.ASTERISK, p.parseInfixExpression) //注册*的解析函数
	p.registerInfix(token.EQ, p.parseInfixExpression)       //注册==的解析函数
	p.registerInfix(token.NOT_EQ, p.parseInfixExpression)   //注册!=的解析函数
	p.registerInfix(token.LT, p.parseInfixExpression)       //注册<的解析函数
	p.registerInfix(token.GT, p.parseInfixExpression)       //注册>的解析函数
	
	return p
}
```

设置parseExpression方法