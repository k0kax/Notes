整数字面量也是表达式，其产生的值就是整数本身。

构建AST
```go
// ast.go
// -------------------------------------------int字面量-----------------------------------------
type IntegerLiteral struct {
    Token token.Token
    Value int64
}

func (il *IntegerLiteral) expressionNode()      {}
func (il *IntegerLiteral) TokenLiteral() string { return il.Token.Literal }
func (il *IntegerLiteral) String() string       { return il.Token.Literal }
```
在构建`*ast.IntegerLitera`时，需将其`*ast.IntegerLitera.Token.Literal`的字符串转化为int64，此处使用` strconv.ParseInt()`方法，具体参考[[01基础变量--声明、赋值、打印、转换（strconv）]]
```go
// parser.go
// 解析函数：解析整形Int
func (p *Parser) parseIntegerLiteral() ast.Expression {
    lit := &ast.IntegerLiteral{Token: p.curToken}
    value, err := strconv.ParseInt(p.curToken.Literal, 0, 64)
    
    if err != nil {
        msg := fmt.Sprintf("could not parse %q as integer", p.curToken.Literal)
        p.errors = append(p.errors, msg)
        return nil
    }

    lit.Value = value
    return lit
}
```
在初始化里添加New()相关的注册函数
```go 
// parser.go
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
    p.registerPrefix(token.INT, p.parseIntegerLiteral)         //注册integer整形相关的解析函数（parseIntegerLiteral）
    
    return p
}
```
测试代码
```go
//parser_test.go
// 测试 解析整形int字面值literal
func TestIntergerLiteralExpression(t *testing.T) {
    input := "5;"

    l := lexer.New(input)
    p := New(l)
    program := p.ParseProgram()
    checkParserErrors(t, p)

    if len(program.Statements) != 1 {
        t.Fatalf("program has not enough statements.got=%d", len(program.Statements))
    }

    stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
    if !ok {
        t.Fatalf("program.Statements[0] is not ast.ExpressionStatement.got=%T", program.Statements[0])
    }

    literal, ok := stmt.Expression.(*ast.IntegerLiteral)
    if !ok {
        t.Fatalf("exp not *ast.IntegerLiteral.got=%T", stmt.Expression)
    }

    if literal.Value != 5 {
        t.Fatalf("literal.Value not %d.got=%d", 5, literal.Value)
    }

    if literal.TokenLiteral() != "5" {
        t.Errorf("literal.TokenLiteral not %s.got=%s", "s", literal.TokenLiteral())
    }
    
}
```
测试结果：
```go
Running tool: C:\Program Files\Go\bin\go.exe test -timeout 30s -run ^TestIntergerLiteralExpression$ monkey_Interpreter/parser

=== RUN   TestIntergerLiteralExpression
--- PASS: TestIntergerLiteralExpression (0.00s)
PASS
ok      monkey_Interpreter/parser       0.162s
```