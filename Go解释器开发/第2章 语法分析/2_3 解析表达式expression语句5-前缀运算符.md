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
扩展语法分析器，添加更加详细的错误信息解析函数
```go
//parser.go

// 将格式化错误信息添加到语法分析器的errors字段
func (p *Parser) noPrefixParseFnError(t token.TokenType) {
    msg := fmt.Sprintf("no prefix parse function for %s found", t)
    p.errors = append(p.errors, msg)
}

// 解析表达式
func (p *Parser) parseExpression(precedence int) ast.Expression {
    prefix := p.prefixParseFns[p.curToken.Type] //只检查前缀位置是否有与p.curToken.Type关联的解析函数
    if prefix == nil {
        p.noPrefixParseFnError(p.curToken.Type)
        return nil
    }
    letExp := prefix() //有
    return letExp      //调用该解析函数
}
```
添加解析函数
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
    p.registerPrefix(token.INT, p.parseIntegerLiteral)         //注册integer整形相关的解析函数（parseIntegerLiteral）
    p.registerPrefix(token.BANG, p.parsePrefixExpression)      //注册！非的解析函数
    p.registerPrefix(token.MINUS, p.parsePrefixExpression)     //注册-负号的解析函数
    return p
}

// 解析前缀表达式 先解析左（操作符），接着解析右
func (p *Parser) parsePrefixExpression() ast.Expression {
    expression := &ast.PrefixExpression{
        Token:    p.curToken,
        Operator: p.curToken.Literal,
    }

    p.nextToken()
    expression.Right = p.parseExpression(PREFIX)

    return expression
}
```
编写测试函数
```go
func TestParsingPrefixExpressions(t *testing.T) {
    prefixTests := []struct {
        input        string
        operator     string
        integerValue int64
    }{
        {"!5;", "!", 5},
        {"-15;", "-", 15},
    }

    for _, tt := range prefixTests {
        l := lexer.New(tt.input)
        p := New(l)
        program := p.ParseProgram()
        checkParserErrors(t, p)

        if len(program.Statements) != 1 {
            t.Fatalf("program.Statements does not contain %d statements.got=%d\n", 1, len(program.Statements))
        }

        stmt, ok := program.Statements[0].(*ast.ExpressionStatement)
        if !ok {
            t.Fatalf("stmt is not ast.ExpressionStatement. got=%T",
                program.Statements[0])
        }
        
        exp, ok := stmt.Expression.(*ast.PrefixExpression)

        if !ok {
            t.Fatalf("stmt is not ast.PrefixExpression. got=%T", stmt.Expression)
        }

        if exp.Operator != tt.operator {
            t.Fatalf("exp.Operator is not '%s'. got=%s", tt.operator, exp.Operator)
        }

        if !testIntergerLiteral(t, exp.Right, tt.integerValue) {
            return
        }
    }
}

func testIntergerLiteral(t *testing.T, il ast.Expression, value int64) bool {
    integ, ok := il.(*ast.IntegerLiteral)
    if !ok {
        t.Errorf("il not *ast.IntegerLiteral. got=%d", il)
        return false
    }

    if integ.Value != value {
        t.Errorf("integ.Value not %d. got=%d", value, integ.Value)
        return false
    }

    if integ.TokenLiteral() != fmt.Sprintf("%d", value) {
        t.Errorf("integ.TokenLiteral not %d. got=%s", value, integ.TokenLiteral())
        return false
    }

    return true
}
```
测试结果：
```go
Running tool: C:\Program Files\Go\bin\go.exe test -timeout 30s -run ^TestParsingPrefixExpressions$ monkey_Interpreter/parser

=== RUN   TestParsingPrefixExpressions
--- PASS: TestParsingPrefixExpressions (0.00s)
PASS
ok      monkey_Interpreter/parser       0.163s
```