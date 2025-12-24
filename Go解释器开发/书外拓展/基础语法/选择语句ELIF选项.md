实际上，使用elif关键词，不用else if 作为if语句的中间选项，是为了偷懒，不想重写部分词法单元。但问题不大后期会改掉。新的语法结构如下：
```go
if(<条件1>)
<结果1>
elif(<条件2>)
<结果2>
elif(<条件3>)
<结果3>
...
else
<可替代的结果>
```
当然可以仅仅只有if。好，让我们开始吧。
## 词法解析
要实现elif多分支的条件，无非是多加个关键词，改改token即可
```go
//token.go
// 声明一些词法常量
const (
	// 特殊类型
	ILLEGAL = "ILLEGAL" // 未知字符
	EOF     = "EOF"     // 文件结尾

	// 标识符+字面量
	IDENT = "IDENT" // add, foobar, x, y
	INT   = "INT"   // 1343456

	// 运算符
	ASSIGN   = "="
	PLUS     = "+"
	MINUS    = "-"
	BANG     = "!"
	ASTERISK = "*"
	SLASH    = "/"

	// 分隔符
	COMMA     = ","
	SEMICOLON = ";"

	LT     = "<"
	GT     = ">"
	LPAREN = "("
	RPAREN = ")"
	LBRACE = "{"
	RBRACE = "}"

	// 关键字
	FUNCTION = "FUNCTION"
	LET      = "LET"
	//*********
	IF     = "IF"
	ELIF   = "ELIF"
	ELSE   = "ELSE"
	RETURN = "RETURN"
	TRUE   = "TRUE"
	FALSE  = "FALSE"

	//比较运算符
	EQ     = "=="
	NOT_EQ = "!="

	//字符串
	STRING = "STRING"

	//ToDo
	//逻辑运算符
	AND = "&&" //和
	OR  = "||" //或
)

//！-/*5；
//			5 < 10 > 5;

// 关键词表 哈希表
var keywords = map[string]TokenType{
	"fn":  FUNCTION,
	"let": LET,
	//*********
	"if":     IF,
	"else":   ELSE,
	"elif":   ELIF,
	"return": RETURN,

	"true":  TRUE,
	"false": FALSE,
}
```
这个比较简单，连词法分析器都不用改，只加个关键词即可
测试代码
```go
//parser_test.go
func TestNextToken(t *testing.T) {
	input := `let five = 5;
let ten = 10;

let add = fn(x, y) {
  x + y;
};

let result = add(five, ten);
!-/*5;
5 < 10 > 5;

if (5 < 10) {
	return true;
} elif(5==10) {
	return false;
}
10 == 10;
10 != 9;
"foobar"
"foo bar"
&&
||
`

	tests := []struct {
		expectedType    token.TokenType
		expectedLiteral string
	}{
		{token.LET, "let"},
		{token.IDENT, "five"},
		{token.ASSIGN, "="},
		{token.INT, "5"},
		{token.SEMICOLON, ";"},
		{token.LET, "let"},
		{token.IDENT, "ten"},
		{token.ASSIGN, "="},
		{token.INT, "10"},
		{token.SEMICOLON, ";"},
		{token.LET, "let"},
		{token.IDENT, "add"},
		{token.ASSIGN, "="},
		{token.FUNCTION, "fn"},
		{token.LPAREN, "("},
		{token.IDENT, "x"},
		{token.COMMA, ","},
		{token.IDENT, "y"},
		{token.RPAREN, ")"},
		{token.LBRACE, "{"},
		{token.IDENT, "x"},
		{token.PLUS, "+"},
		{token.IDENT, "y"},
		{token.SEMICOLON, ";"},
		{token.RBRACE, "}"},
		{token.SEMICOLON, ";"},
		{token.LET, "let"},
		{token.IDENT, "result"},
		{token.ASSIGN, "="},
		{token.IDENT, "add"},
		{token.LPAREN, "("},
		{token.IDENT, "five"},
		{token.COMMA, ","},
		{token.IDENT, "ten"},
		{token.RPAREN, ")"},
		{token.SEMICOLON, ";"},
		{token.BANG, "!"},
		{token.MINUS, "-"},
		{token.SLASH, "/"},
		{token.ASTERISK, "*"},
		{token.INT, "5"},
		{token.SEMICOLON, ";"},
		{token.INT, "5"},
		{token.LT, "<"},
		{token.INT, "10"},
		{token.GT, ">"},
		{token.INT, "5"},
		{token.SEMICOLON, ";"},
		{token.IF, "if"},
		{token.LPAREN, "("},
		{token.INT, "5"},
		{token.LT, "<"},
		{token.INT, "10"},
		{token.RPAREN, ")"},
		{token.LBRACE, "{"},
		{token.RETURN, "return"},
		{token.TRUE, "true"},
		{token.SEMICOLON, ";"},
		{token.RBRACE, "}"},
		{token.ELIF, "elif"},
		{token.LPAREN, "("},
		{token.INT, "5"},
		{token.EQ, "=="},
		{token.INT, "10"},
		{token.RPAREN, ")"},
		{token.LBRACE, "{"},
		{token.RETURN, "return"},
		{token.FALSE, "false"},
		{token.SEMICOLON, ";"},
		{token.RBRACE, "}"},
		{token.INT, "10"},
		{token.EQ, "=="},
		{token.INT, "10"},
		{token.SEMICOLON, ";"},
		{token.INT, "10"},
		{token.NOT_EQ, "!="},
		{token.INT, "9"},
		{token.SEMICOLON, ";"},
		{token.STRING, "foobar"},
		{token.STRING, "foo bar"},
		{token.AND, "&&"},
		{token.OR, "||"},
		{token.EOF, ""},
	}

	// 创建词法分析器实例
	l := New(input) //lexer

	//i：表示当前tests中的位置。
	//tt：表示当前迭代的expectedType（词法单元类型）和expectedLiteral（字面量）字段。
	for i, tt := range tests {

		// 调用词法分析器的NextToken方法获取下一个词法单元
		tok := l.NextToken() //token

		// 检查实际的词法单元类型是否与预期的类型一致
		if tok.Type != tt.expectedType {
			t.Fatalf("tests[%d] - tokentype wrong. expected=%q, got==%q", i, tt.expectedType, tok.Type)
		}
		// 检查实际的词法单元字面量是否与预期的字面量一致
		if tok.Literal != tt.expectedLiteral {
			t.Fatalf("tests[%d] - literal wrong. expected=%q, got==%q", i, tt.expectedLiteral, tok.Literal)
		}
	}

}

```
## 语法解析
现在到语法解析了，新加个elif关键词，就需要为它新建ast，如下所示：
```go
//ast.go
// 中间选项elif
type ElIfExpression struct {
	Token       token.Token
	Condition   Expression//条件
	Consequence *BlockStatement//结果
}

func (ef *ElIfExpression) expressionNode()      {}
func (ef *ElIfExpression) TokenLiteral() string { return ef.Token.Literal }
func (ef *ElIfExpression) String() string {
	var out bytes.Buffer

	//写入首选项的条件和结果
	out.WriteString("elif")
	out.WriteString(" ")
	out.WriteString(ef.Condition.String())
	out.WriteString(" ")
	out.WriteString(ef.Consequence.String())

	return out.String()
}
```
同时，if的AST就需要改动，加入elif了
```go
//ast.go
// ---------------------------------------------------if-else--------------------------------------------
type IfExpression struct {
	Token           token.Token
	Condition       Expression //首选项
	Consequence     *BlockStatement
	Alternatives    []*ElIfExpression //中间选项
	LastAlternative *BlockStatement   //最后选项
}

func (ie *IfExpression) expressionNode()      {}
func (ie *IfExpression) TokenLiteral() string { return ie.Token.Literal }
func (ie *IfExpression) String() string {
	var out bytes.Buffer

	//写入首选项的条件和结果
	out.WriteString("if")
	out.WriteString(" ")
	out.WriteString(ie.Condition.String())
	out.WriteString(" ")
	out.WriteString(ie.Consequence.String())

	//写入中间选项的条件和结果
	for _, alternative := range ie.Alternatives {
		alternative.String()
	}

	//最后选项
	if ie.LastAlternative != nil {
		out.WriteString(" else ")
		out.WriteString(ie.LastAlternative.String())
	}

	return out.String()
}
```
接下里需要修改if的语法解析函数，此处采用嵌套的方法，单独处理elif语句，再用if调用它
```go
// 解析if语句
func (p *Parser) parseIfExpression() ast.Expression {
	expression := &ast.IfExpression{Token: p.curToken}

	//if部分 首选
	//识别条件
	if !p.expectPeek(token.LPAREN) { //下一个token是左括号(，移动token，继续，进入条件识别；不是则直接退出
		return nil
	}

	p.nextToken()                                    //token移动开始识别条件
	expression.Condition = p.parseExpression(LOWEST) //低权限识别条件，并写入条件中

	if !p.expectPeek(token.RPAREN) { //下一个token是右括号)，移动token，继续，表明条件结束
		return nil
	}

	//识别结果
	if !p.expectPeek(token.LBRACE) { //下一个token是左大括号{，移动token，继续，开始进行结果识别；不是则直接退出
		return nil
	}

	//写入首选结果
	expression.Consequence = p.parseBlockStatement()

	//中间选项
	for p.peekTokenIs(token.ELIF) {
		p.nextToken()
		elif := p.parseElIfExpression()
		expression.Alternatives = append(expression.Alternatives, elif)
	}

	//else部分 最后选项
	if p.peekTokenIs(token.ELSE) { //识别下一个token是else,进入else的判断；不是则直接退出
		p.nextToken() //后移token 匹配到真正的

		if !p.expectPeek(token.LBRACE) { //识别下一个token是左大括号{
			return nil
		}

		expression.LastAlternative = p.parseBlockStatement() //写入可替换结果
	}

	return expression
}

// 解析ELIF 和解析IF差不多
func (p *Parser) parseElIfExpression() *ast.ElIfExpression {
	expression := &ast.ElIfExpression{Token: p.curToken}
	if !p.expectPeek(token.LPAREN) { //下一个token是左括号(，移动token，继续，进入条件识别；不是则直接退出
		return nil
	}

	p.nextToken()                                    //token移动开始识别条件
	expression.Condition = p.parseExpression(LOWEST) //低权限识别条件，并写入条件中

	if !p.expectPeek(token.RPAREN) { //下一个token是右括号)，移动token，继续，表明条件结束
		return nil
	}

	//识别结果
	if !p.expectPeek(token.LBRACE) { //下一个token是左大括号{，移动token，继续，开始进行结果识别；不是则直接退出
		return nil
	}

	//写入首选结果
	expression.Consequence = p.parseBlockStatement()

	return expression
}
```
## 求值
