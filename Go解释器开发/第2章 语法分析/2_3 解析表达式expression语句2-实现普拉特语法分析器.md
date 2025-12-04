普拉特语法分析器的主要思想是==将解析函数与词法单元类型相关联==。当遇到对应的词法单元，就会调用对应的解析函数来解析对应的表达式，返回AST节点。
每个词法单元类型最多绑定两个解析函数，主要取决于它的位置，是前缀还是中缀。

设置解析函数
```go
//parser.go
type (
	prefixParseFn func() ast.Expression               //前缀解析函数，左侧为空
    infixParseFn  func(ast.Expression) ast.Expression //中缀解析函数，接受的参数时中缀表达式左边的内容
)
```

在Parser结构体添加对应映射，即不同的词法单元对应不同的注册方法
```go
//parser.go
type Parser struct {

    l *lexer.Lexer //指向词法分析器实例的指针
    curToken  token.Token //当前词法单元
    peekToken token.Token //当前词法单元的下一位
    
    errors []string //错误集合

    //添加两个解析函数的映射
    prefixParseFns map[token.TokenType]prefixParseFn
    infixParseFns  map[token.TokenType]infixParseFn
}
```
还需要向映射中添加内容的方法
```go 
//parser.go

// 辅助函数
// 向前缀、中缀解析函数映射表添加数据
func (p *Parser) registerPrefix(tokenType token.TokenType, fn prefixParseFn) {
    p.prefixParseFns[tokenType] = fn
}

func (p *Parser) registerInfix(tokenType token.TokenType, fn infixParseFn) {
    p.infixParseFns[tokenType] = fn
}
```
