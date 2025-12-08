标识符是最简单的表达式类型，可以是程序员自定义的、用来指代变量、函数、常量、数组 / 哈希键等实体的 “名字”，本质是一串符合语法规则的字符序列，是程序中 “东西” 的 “代号”。具体如下：
```go
foobar;
add(foobar, barfoo);
foobar + barfoo;
if (foobar) {
// [...]
}
```
扩展语法解析器，让其可以解析多种语句
```go
//parser.go
// 解析语句

func (p *Parser) parseStatement() ast.Statement {
    switch p.curToken.Type {
    case token.LET:
        return p.parseLetStatement()
    case token.RETURN:
        return p.parseReturnStatement()
    default:
        return p.parseExpressionStatement()//默认解析表达式语句
    }
}
```
编写parseExpressionStatement()方法
```go
// 解析表达式语句expression

func (p *Parser) parseExpressionStatement() *ast.ExpressionStatement {

    stmt := &ast.ExpressionStatement{Token: p.curToken} //构建AST
    stmt.Expression = p.parseExpression(LOWEST)
    if p.peekTokenIs(token.SEMICOLON) { //检查分号（有没有分号都可以运行）
        p.nextToken()
    }
    return stmt
}
```
添加优先级precedence
```go
//parser.go
const (
	_ int = iota
	LOWEST
	EQUALS // ==相等
	LESSGREATER // > or <大于或小于
	SUM // + 加减
	PRODUCT // *乘除
	PREFIX // -X or !X负数或非x
	CALL // myFunction(X)函数
)
```
具体的解析语句，此处并没有完全实现
此版本只检查前缀位置是否有相关联的解析函数
```go
//parser.go
// 解析表达式

func (p *Parser) parseExpression(precedence int) ast.Expression {//根据precedence优先级的高低，确定执行，此间并没有完全实现
    prefix := p.prefixParseFns[p.curToken.Type] //只检查前缀位置是否有与p.curToken.Type关联的解析函数 取出来解析函数
    if prefix == nil {
        return nil
    }
    letExp := prefix() //有
    return letExp      //调用该解析函数
}
```

在初始化函数New()里添加那两个映射，注册标识符的解析函数
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
    //前缀解析解析函数
    p.prefixParseFns = make(map[token.TokenType]prefixParseFn) //初始化映射
    p.registerPrefix(token.IDENT, p.parseIdentifier)           //注册ident标识符相关的解析函数（parseIdentifier）
    return p

}
```
具体的标识符的解析函数
返回标识符indent的结构体：
```go
//parser.go
// 解析函数：解析标识符indent

func (p *Parser) parseIdentifier() ast.Expression {
    return &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
}
```

测试函数：
```go
// parser_test.go
// 测试 标识符indent 表达式 foobar;
func TestIdentifierExpression(t *testing.T) {
    input := "foobar"

    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p) //检查错误

    if len(program.Statements) != 1 {
        t.Fatalf("program has not enough statements.got=%d",
            len(program.Statements))
    }

    //检查program.Statements中唯一的语句是否为*ast.ExpressionStatement
    stmt, ok := program.Statements[0].(*ast.ExpressionStatement)

    if !ok {
        t.Fatalf("program.Statements[0] is not ast.ExpressionStatement.got=%T",
            program.Statements[0])
    }

  

    //检查*ast.ExpressionStatement.Expression是否为*ast.Identifier
    ident, ok := stmt.Expression.(*ast.Identifier)
    if !ok {
        t.Fatalf("exp not *ast.Identifier.got=%T", stmt.Expression)

    }

    //检查标识符是否具有正确的"foobar"值
    if ident.Value != "foobar" {
        t.Errorf("ident.Value not %s.got=%s", "foobar", ident.Value)
    }

    if ident.TokenLiteral() != "foobar" {
        t.Errorf("ident.TokenLiteral not %s.got=%s", "foobar", ident.TokenLiteral())
    }
}
```
测试结果：
```go
Running tool: C:\Program Files\Go\bin\go.exe test -timeout 30s -run ^TestIdentifierExpression$ monkey_Interpreter/parser

=== RUN   TestIdentifierExpression
--- PASS: TestIdentifierExpression (0.00s)
PASS
ok      monkey_Interpreter/parser       0.130s
```