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
在构建`*ast.IntegerLitera`时，需将其`*ast.IntegerLitera.Token.Literal`的字符串转化为int64