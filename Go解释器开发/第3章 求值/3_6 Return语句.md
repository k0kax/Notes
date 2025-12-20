好的，我们先回顾一下return语句：`return <表达式>`，我们要的是return返回的值

构造return的对象
```go
//object.go
const (
	INTEGER_OBJ      = "INTEGER"      //整数
	BOOLEAN_OBJ      = "BOOLEAN"      //布尔
	NULL_OBJ         = "NULL"         //空值
	RETURN_VALUE_OBJ = "RETURN_VALUE" //返回值
)

//用来存放返回值
type ReturnValue struct {
	Value Object
}

func (rv *ReturnValue) Type() ObjectType { return RETURN_VALUE_OBJ }
func (rv *ReturnValue) Inspect() string  { return rv.Value.Inspect() }
```

给Eval添加return的求值选项
```go
//evaluator.go
func Eval(node ast.Node) object.Object {
	switch node := node.(type) {

	//语句
	case *ast.Program:
		return evalProgram(node.Statements)

	//表达式语句
	case *ast.ExpressionStatement:
		return Eval(node.Expression)

	//整形字面量
	case *ast.IntegerLiteral:
		return &object.Integer{Value: node.Value}

	//布尔型字面量
	case *ast.Boolean:
		return nativeBoolToBooleanObject(node.Value)

	//前缀表达式
	case *ast.PrefixExpression:
		right := Eval(node.Right)
		return evalPrefixExpression(node.Operator, right)

	//中缀表达式
	case *ast.InfixExpression:
		left := Eval(node.Left)
		right := Eval(node.Right)
		return evalInfixExpression(node.Operator, left, right)

	//块
	case *ast.BlockStatement:
		return evalBlockStatement(node)

	//if语句
	case *ast.IfExpression:
		return evalIfExpression(node)

	//return语句
	case *ast.ReturnStatement:
		val := Eval(node.ReturnValue)
		return &object.ReturnValue{Value: val}
	}

	return nil
}
```

具体的求值方法
```go
//evaluator.go
// 语句求值
func evalStatements(stmts []ast.Statement) object.Object {
	var result object.Object

	for _, statement := range stmts {
		result = Eval(statement)

		if returnValue, ok := result.(*object.ReturnValue); ok {
			return returnValue.Value
		}
	}

	return result
}
```

但是会遇到一些问题，如果遇到嵌套的额return，就执行不到底，例如：
```go
if (10 > 1) {
	if (10 > 1) {
		return 10;
	}
	return 1;
}
```
只会返回1，原因是if语句的结果是一个嵌套的块（也是一个if语句），它没有执行到底。而如果没有嵌套快，就能正常运行。要让嵌套的块也能正常运行，就不能复用 evalStatements函数。故将其改为 evalProgram。
```go
//evaluator.go
// 语句求值
func evalProgram(stmts []ast.Statement) object.Object {
	var result object.Object

	for _, statement := range stmts {
		result = Eval(statement)

		if returnValue, ok := result.(*object.ReturnValue); ok {
			return returnValue.Value
		}
	}

	return result
}
```

块由专门的函数去处理
```go
//evaluator.go
// block求值的辅助函数
func evalBlockStatement(block *ast.BlockStatement) object.Object {
	var result object.Object

	for _, statement := range block.Statements {
		result = Eval(statement)

		if result != nil && result.Type() == object.RETURN_VALUE_OBJ {
			return result
		}
	}

	return result
}
```
它和上面的函数很像，但是并没有取解包取值，仅仅是检查了结果类型。

新的Eval函数如下：
```go
//evaluator.go
func Eval(node ast.Node) object.Object {
	switch node := node.(type) {

	//语句
	case *ast.Program:
		return evalProgram(node.Statements)

	//表达式语句
	case *ast.ExpressionStatement:
		return Eval(node.Expression)

	//整形字面量
	case *ast.IntegerLiteral:
		return &object.Integer{Value: node.Value}

	//布尔型字面量
	case *ast.Boolean:
		return nativeBoolToBooleanObject(node.Value)

	//前缀表达式
	case *ast.PrefixExpression:
		right := Eval(node.Right)
		return evalPrefixExpression(node.Operator, right)

	//中缀表达式
	case *ast.InfixExpression:
		left := Eval(node.Left)
		right := Eval(node.Right)
		return evalInfixExpression(node.Operator, left, right)

	//块
	case *ast.BlockStatement:
		return evalBlockStatement(node)

	//if语句
	case *ast.IfExpression:
		return evalIfExpression(node)

	//return语句
	case *ast.ReturnStatement:
		val := Eval(node.ReturnValue)
		return &object.ReturnValue{Value: val}
	}

	return nil
}
```





