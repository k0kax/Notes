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

vscode的调试分为两个模式---启动模式laungh与attach附加模式。启动模式，就是通过调试器启动程序并调试，多用于后端开发；附加模式，多是在程序已经运行了，再启动调试，多用于网页开发。

## launch.json的配置属性

### 以下属性是所有启动配置的**必填项**：


- **type**（调试器类型）—— 此启动配置要使用的调试器类型。每个已安装的调试扩展都会定义一种类型：例如，内置 Node 调试器对应 `node`，PHP 和 Go 扩展则分别对应 `php` 和 `go`。
- **request**（请求类型）—— 此启动配置的请求类型。目前支持 `launch`（启动）和 `attach`（附加）两种类型。
- **name**（名称）—— 显示在 “调试启动配置” 下拉菜单中的易读名称。调试界面的显示名。

### 以下是所有启动配置均支持的**可选属性**：

- **presentation**（展示设置）—— 通过 `presentation` 对象中的 `order`（排序）、`group`（分组）和 `hidden`（隐藏）属性，你可以在 “调试配置” 下拉菜单和 “调试快速选择” 面板中对配置项及复合配置项进行排序、分组和隐藏。
- **preLaunchTask**（启动前任务）—— 若要在调试会话开始前执行某个任务，可将此属性设为 `tasks.json`（工作区的 `.vscode` 文件夹中）中指定的任务标签。也可将其设为 `${defaultBuildTask}`，以调用默认的构建任务。必须有该文件。
- **postDebugTask**（调试后任务）—— 若要在调试会话结束时执行某个任务，可将此属性设为 `tasks.json`（工作区的 `.vscode` 文件夹中）中指定的任务名称。
- **internalConsoleOptions**（内置控制台选项）—— 此属性控制调试会话期间 “调试控制台” 面板的可见性。
	- `openOnStart`（默认）：调试启动时自动打开控制`neverOpen`：永远不自动打开
	- `openOnFirstSessionStart`：仅第一次调试时打开
- **debugServer**（调试服务器）—— 仅适用于调试扩展开发者：此属性允许你连接到指定端口，而非启动调试适配器。
- **serverReadyAction**（服务器就绪操作）—— 若希望被调试程序向调试控制台或集成终端输出特定消息时，自动在浏览器中打开某个 URL，可配置此属性。
### 许多调试器还支持以下部分属性：

- **program**（程序路径）—— 启动调试器时要运行的可执行文件或目标文件。
	- 假设文件在目录的xx文件夹下的yy文件，则为`${workspaceFolder}/xx/yy`

- **args**（参数）—— 传递给待调试程序的参数。仅在laungh模式下有效：
	先看一个核心逻辑：**终端里怎么传参，args 就怎么拆分成数组**。

| 终端运行命令（示例）                                                   | 对应的 args 配置                                                                                        | 说明                       |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- | ------------------------ |
| `node app.js --port 3000 --env dev`                          | `"args": ["--port", "3000", "--env", "dev"]`                                                       | 空格分隔的参数，每个拆成数组元素         |
| `go run main.go -name "张三" -age 20`                          | `"args": ["-name", "张三", "-age", "20"]`                                                            | 带中文 / 空格的参数，直接写（无需额外加引号） |
| `python script.py --input ./data.txt --output ./result.json` | `"args": ["--input", "${workspaceFolder}/data.txt", "--output", "${workspaceFolder}/result.json"]` |                          |
	
- **env**（环境变量）—— 需设置的环境变量（值设为 `null` 可 “取消定义” 某个变量）。
- **envFile**（环境变量文件路径）—— 包含环境变量的 dotenv 文件路径。
- **cwd**（当前工作目录）—— 用于查找依赖项和其他文件的当前工作目录。
- **port**（端口）—— 附加到正在运行的进程时使用的端口。
- **stopOnEntry**（入口处中断）—— 程序启动后立即中断执行。
- **console**（控制台类型）—— 要使用的控制台类型，例如 `internalConsole`（内置控制台）、`integratedTerminal`（集成终端）或 `externalTerminal`（外部终端）。

### mode属性字段
该字段并非通用，不同的语言不一样。
- **本质**：补充 `request`（launch/attach）的细节，定义 “调试器以何种方式介入程序执行”；
- **适用范围**：不是所有调试器都支持（比如 PHP、Chrome 调试器无此属性），主要集中在**编译型 / 解释型后端语言**（Node.js、Go、Python 等）；
- **核心作用**：决定调试器是 “逐文件执行”“自动附加子进程” 还是 “仅调试主进程”。

在go语言中
Go 调试器的 `mode` 属性是**必填项**（区别于 Node.js 的可选），取值完全不同，核心是指定 “调试的代码范围”：

|`mode` 取值|核心作用|适用场景|
|---|---|---|
|`debug`|调试单个可执行文件 / 包（最常用）|调试单个 `main.go` 或编译后的二进制文件|
|`test`|调试 Go 测试用例（`*_test.go`）|单元测试 / 集成测试调试（比如 `go test`）|
|`exec`|调试已编译的 Go 二进制文件|调试打包后的可执行程序（非源码调试）|
|`remote`|远程调试（附加到远程 Go 进程）|调试服务器上运行的 Go 程序|
