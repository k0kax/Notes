实际上，使用elif关键词，不用else if 作为if语句的中间选项，是为了偷懒，不想重写部分词法单元。但问题不大后期会改掉。好，让我们开始吧。
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

```

## 语法解析

## 求值
