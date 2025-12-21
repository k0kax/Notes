调试文件编写
通过左边栏虫子图标的调试按钮进行设置

可以通过编写调试文件launch.json进行精细化的调试。
```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",//版本
    "configurations": [//配置信息
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "test",
            "program": "${workspaceFolder}/evaluator/evaluator_test.go",
            "env": {},
            "args": ["-test.run", "TestLetStatements"],
            "stopOnEntry": false // 不需要启动就暂停，手动触发断点
        }
    ]
}
```
主要的配置信息在configurations里面

vscode的调试分为两个模式---启动模式laungh与附加模式。
