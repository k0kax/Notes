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

普拉特语法分析器的**核心**---设置parseExpression方法
```go
// 解析表达式 普卡特语法解析器核心 1+2*3
func (p *Parser) parseExpression(precedence int) ast.Expression {
	// 第一步：找当前token对应的「前缀解析函数」
	prefix := p.prefixParseFns[p.curToken.Type]
	//当前token：1，注册前缀解析函数prefix也就是parseIntegerLiteral()
	// 第二步：如果没有对应的前缀解析函数，报错并返回nil
	if prefix == nil {
		p.noPrefixParseFnError(p.curToken.Type)
		return nil
	}

	// 第三步：执行前缀解析函数，得到表达式的「左部分」
	leftExp := prefix()
	// 执行parseIntegerLiteral()，leftExp = &ast.IntegerLiteral{Value: 1}
	// 此时 token 状态：curToken=INT(1)，peekToken=PLUS(+)

	// 第四步：循环解析「中缀表达式」（核心循环，处理优先级）左结合
	for !p.peekTokenIs(token.SEMICOLON) && precedence < p.peekPrecedence() {
		// 循环条件（两个都满足才继续）：
		// 1. !p.peekTokenIs(token.SEMICOLON)：下一个token不是分号（说明表达式还没结束）；
		// 2. precedence < p.peekPrecedence()：当前表达式的优先级 < 下一个token的优先级（保证先解析高优先级的）；

		//下一个token为+，进入循环

		// 找到下一个token对应的中缀解析函数
		infix := p.infixParseFns[p.peekToken.Type]
		// p.peekToken.Type = PLUS(+) → 拿到 parseInfixExpression 函数
		// infix = parseInfixExpression

		// 如果没有中缀解析函数，说明表达式结束，返回当前的左部分
		if infix == nil {
			return leftExp
		}
		// infix不为空跳过

		// 移动token
		p.nextToken()
		// 执行后 token 状态：curToken=PLUS(+)，peekToken=INT(2)

		// 执行中缀解析函数（核心递归），更新左部分为「完整的中缀表达式」
		leftExp = infix(leftExp)
		// 传入 leftExp=1，执行 parseInfixExpression(1)
	}

	// 第五步：返回最终解析好的表达式
	return leftExp //调用该解析函数
}
```
这个方法非常精妙，涉及到左对齐，递归调用，以及为啥用for循环而不用if，将在下篇中举例介绍

测试函数
分别测试中缀运算符和
```go
//parser_test.go
// 测试中缀运算符
func TestParsingInfixExpressions(t *testing.T) {
	infixTests := []struct {
		input      string
		leftValue  int64
		operator   string
		rightValue int64
	}{
		{"5 + 5;", 5, "+", 5},
		{"5 - 5;", 5, "-", 5},
		{"5 * 5;", 5, "*", 5},
		{"5 / 5;", 5, "/", 5},
		{"5 > 5;", 5, ">", 5},
		{"5 < 5;", 5, "<", 5},
		{"5 == 5;", 5, "==", 5},
		{"5 != 5;", 5, "!=", 5},
	}

	for _, tt := range infixTests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)

		if len(program.Statements) != 1 {
			t.Fatalf("program.Statements does not contain %d statements. got=%d\n", 1, len(program.Statements))
		}

		stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
		if !ok {
			t.Fatalf("program.Statements[0] is not ast.ExpressionStatement. got=%T", program.Statements[0])
		}

		exp, ok := stmt.Expression.(*ast.InfixExpression)
		if !ok {
		 	t.Fatalf("exp is not ast.InfixExpression. got=%T", stmt.Expression)
		 }

		 if !testIntergerLiteral(t, exp.Left, tt.leftValue) {
		 	return
		 }

		 if exp.Operator != tt.operator {
		 	t.Fatalf("exp.Operator is not '%s'. got=%s", tt.operator, exp.Operator)
		 }

		 if !testIntergerLiteral(t, exp.Right, tt.rightValue) {
		 	return
		 }
	}
}
```
