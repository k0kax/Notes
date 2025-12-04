自下而上的运算符优先级分析（普拉特解析法）

## 专业术语
前缀运算符
后缀运算符
中缀运算符
运算符优先级


## 构建表达式expression的AST
```go
//ast.go
type ExpressionStatement struct {
    Token      token.Token //该表达式的第一个词法单元
    Expression Expression  //保存表达式
}

func (es *ExpressionStatement) statementNode()       {}
func (es *ExpressionStatement) TokenLiteral() string { return es.Token.Literal }
```
表达式语句的结构体主要由两个字段组成Token和Expression(保存表达式)




