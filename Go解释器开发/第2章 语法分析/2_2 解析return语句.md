好吧，有了前面的经验我们的第一步是确定return有那些形式
```go
return 5;
return 10;
return add(15);
```
可以抽象为`return <表达式>;`

### 构建return语句的AST
```go 
//ast.go
// return相关
type ReturnStatement struct {
    Token       token.Token //return词法单元
    ReturnVaule Expression  //返回值
}

func (rs *ReturnStatement) statementNode()       {}
func (rs *ReturnStatement) TokenLiteral() string { return rs.Token.Literal }
```
### 添加解析器
添加解析词法单元
```go
//parser.go
func (p *Parser) parseStatement() ast.Statement {
	switch p.curToken.Type {
	case token.LET:
		return p.parseLetStatement()
	case token.RETURN:
		return p.parseReturnStatement()
	default:
	return nil
	}
}
```
设置解析函数
```go
//parser.go

```