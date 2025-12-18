删除Let和Return的todo
```go
//parser.go
// 解析let语句 以为例let x=5;
// 此时：curtoken=let peektoken=x
func (p *Parser) parseLetStatement() *ast.LetStatement {
	//1.初始化 LetStatement 节点：把当前 curToken（就是 LET 类型的 "let"）绑定到节点
	stmt := &ast.LetStatement{Token: p.curToken}
	//运行后：stmt.token=let curtoken=let peektoken=x

	//2.检测标识符
	//检测下一个token(也就是peektoken)不是标识符indent，不是则退出（此处检测到为x是标识符，不退）,是则peektoken、curtoken后移一位
	if !p.expectPeek(token.IDENT) { //执行expectPeek(),检测到peektoken.type=IDENT,不执行{}内容，peektoken、curtoken都后移一位
		return nil
	}
	//运行后：stmt.token=let curtoken=x peektoken = "="

	//3.
	//将当前词法单元作为标识符的 Token 字段，并将其字面值literal作为标识符indent的值value赋给 stmt.Name
	stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	//运行后：stmt.token=let curtoken=x peektoken = "=" stmt.Name=&{Token: IDENT("x"), Value: "x"}

	//4.检测等号=
	//检查下一个token（peektoken）,,是则继续，不是则退出，peektoken、curtoken后移一位
	if !p.expectPeek(token.ASSIGN) {
		return nil
	}
	//运行后：stmt.token=let curtoken="=" peektoken = "5"  stmt.Name=&{Token: IDENT("x"), Value: "x"}

	//5.TODO: 跳过对表达式的处理parseExpression()
	//运行后：stmt.token=let curtoken="5" peektoken = ";"  stmt.Name=&{Token: IDENT("x"), Value: "x"} tmt.Value = &IntegerLiteral {Token: INT ("5"), Value: 5}
	p.nextToken()
	
    stmt.Value = p.parseExpression(LOWEST)

	//6.检测分号（;）     处理语句末尾的分号（;）
	//检测当前token（curtoken）是否是分号（;）
	if !p.curTokenIs(token.SEMICOLON) { //是，则不需要移动
		p.nextToken() //不是，则peektoken、curtoken后移一位，直接解析下一句
	}
	//运行后：stmt.token=let curtoken=";" peektoken = ""  stmt.Name=&{Token: IDENT("x"), Value: "x"} stmt.

	//7.直接返回stmt
	return stmt
	//LetStatement {Token: LET ("let"), Name: Identifier ("x"), Value: IntegerLiteral (5)}

}

func (p *Parser) parseReturnStatement() *ast.ReturnStatement {
	stmt := &ast.ReturnStatement{Token: p.curToken}

	p.nextToken()

	//TODO:跳过对表达式的处理，直接遇到分号
	stmt.ReturnValue = p.parseExpression(LOWEST)

	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}

	return stmt
}
```

重写TestLetStatement测试函数
```go
//parser_test.go
func TestLetStatements(t *testing.T) {
	tests := []struct {
		input              string
		expectedIdentifier string
		expectedValue      interface{}
	}{
		{"let x = 5;", "x", 5},
		{"let y = true;", "y", true},
		{"let foobar = y;", "foobar", "y"},
	}

	for _, tt := range tests {
		l := lexer.New(tt.input)
		p := New(l)
		program := p.ParseProgram()
		checkParserErrors(t, p)

		if len(program.Statements) != 1 {
			t.Fatalf("program.Statements does not contain 1 statements. got=%d", len(program.Statements))
		}

		stmt := program.Statements[0]
		if !testLetStatement(t, stmt, tt.expectedIdentifier) {
			return
		}

		val := stmt.(*ast.LetStatement).Value
		if !testLiteralExpression(t, val, tt.expectedValue) {
			return
		}
	}
}
```
