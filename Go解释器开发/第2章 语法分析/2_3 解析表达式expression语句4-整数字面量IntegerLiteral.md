整数字面量也是表达式，其产生的值就是整数本身。

### 构建AST
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