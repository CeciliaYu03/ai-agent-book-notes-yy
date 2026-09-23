# Task2 : Tools 与 MCP

## 实验：4-2	perception-tools	

### 安装

####  1. 在仓库根目录使用统一的第 4 章环境  

![alt text](<截屏2026-09-23 16.24.03.png>)

####  2. 切换目录  

![alt text](<截屏2026-09-23 16.26.10.png>)

### 配置


### 使用

#### 1.python cli.py --help # 查看总帮助与所有子命令
![alt text](<截屏2026-09-23 19.12.19.png>)

#### 2.python cli.py list # 列出所有执行工具
![alt text](<截屏2026-09-23 19.13.51.png>)

#### 3.python cli.py demo # 端到端离线演示（推荐先看这个；无需 API key 即可运行）

>  (agentbook) lvdousha@192 execution-tools % python cli.py demo  
演示工作区：/var/folders/zy/dcdry7216jq52lgwhvrvksv80000gn/T/exec_tools_demo_oo3f54wa  （离线路径，无需 API key；如已配置 key，审批/总结将走真实 LLM）
================================================================  
> 1. file_write：写入词频统计脚本（自动语法校验）  
================================================================  
结果：success=True, verification=passed  
写入：/private/var/folders/zy/dcdry7216jq52lgwhvrvksv80000gn/T/exec_tools_demo_oo3f54wa/wordcount.py  
================================================================  
> 2. file_write：写入含语法错误的代码（linter 应拦截）  
================================================================  
结果：success=False  
校验反馈：Syntax validation failed: Syntax error at line 1: invalid syntax
================================================================  
> 3. file_write：生成样本数据文件  
================================================================  
结果：success=True，写入 64 字节
================================================================  
> 4. code_interpreter：运行统计逻辑（Python 沙盒）  
================================================================  
结果：success=True, returncode=0
stdout:  
  apple: 4  
  banana: 3  
  cherry: 2  
================================================================  
> 5. virtual_terminal：用 shell 校验数据文件  
================================================================  
结果：success=True, returncode=0  
stdout:  
        10 /var/folders/zy/dcdry7216jq52lgwhvrvksv80000gn/T/exec_tools_demo_oo3f54wa/data.txt  
  --- 词数统计完成 ---  
================================================================  
> 6. code_interpreter：长输出自动截断并落盘  
================================================================  
上下文中保留的输出行数：102（原始 1000 行）  
完整输出落盘文件：/var/folders/zy/dcdry7216jq52lgwhvrvksv80000gn/T/code_interpreter_output_x8b_8maq.txt  
上下文中输出的尾部片段：  
  line 998: xxxxxxxxxxxxxxxxxxxx  
  line 999: xxxxxxxxxxxxxxxxxxxx  
  [如需完整输出，请使用 read_file 工具读取 /var/folders/zy/dcdry7216jq52lgwhvrvksv80000gn/T/code_interpreter_output_x8b_8maq.txt]  
================================================================  
> 7. virtual_terminal：危险命令触发审批
================================================================  
结果：success=True  
（审批通过：已配置 API key，真实 LLM 判定该命令针对不存在路径、无副作用而放行。）  
================================================================  
演示完成  
================================================================  
覆盖的安全机制：自动 linter 校验、危险命令审批、长输出截断持久化。  
演示产物位于：/var/folders/zy/dcdry7216jq52lgwhvrvksv80000gn/T/exec_tools_demo_oo3f54wa  


#### 4.  # 单独调用某个工具

##### python cli.py code --language python --code "print(2 ** 10)"
```json
(agentbook) lvdousha@192 execution-tools % python cli.py code --language python --code "print(2 ** 10)"
{
  "success": true,
  "status": "success",
  "language": "python",
  "stdout": "1024\n",
  "stderr": "",
  "stdout_file": null,
  "stderr_file": null,
  "returncode": 0,
  "error": null,
  "compile_output": null,
  "phase": null,
  "execution_time": 0.0267031192779541,
  "sandbox": {
    "kind": "local-process",
    "degraded": true
  },
  "verification": "passed"
}
```
##### python cli.py shell "python3 --version"
```
(agentbook) lvdousha@192 execution-tools % python cli.py shell "python3 --version"
{
  "success": true,
  "returncode": 0,
  "stdout": "Python 3.12.14\n",
  "stderr": "",
  "stdout_file": null,
  "stderr_file": null
}
```
##### python cli.py write --path notes.txt --content "hello" --overwrite
```
(agentbook) lvdousha@192 execution-tools % python cli.py write --path notes.txt --content "hello" --overwrite
{
  "success": true,
  "path": "/Users/lvdousha/ai-agent-book/chapter4/execution-tools/notes.txt",
  "bytes_written": 5,
  "verification": "passed"
}
```
![alt text](<截屏2026-09-23 19.47.56.png>)

##### python cli.py edit --path notes.txt --search hello --replace world
```
(agentbook) lvdousha@192 execution-tools % python cli.py edit --path notes.txt --search hello --replace world
{
  "success": true,
  "path": "/Users/lvdousha/ai-agent-book/chapter4/execution-tools/notes.txt",
  "diff_preview": "Line 1:\n  - hello\n  + world",
  "verification": "passed"
}
```

![alt text](<截屏2026-09-23 19.49.17.png>)

#### 运行 MCP 服务器
![alt text](<截屏2026-09-23 20.02.30.png>)

#### 测试单个工具

##### # Test file operations
python test_file_tools.py
```
(agentbook) lvdousha@192 execution-tools % python test_file_tools.py
=== File Tools Tests ===

Testing file write...
✓ File write successful: {'success': True, 'path': '/Users/lvdousha/ai-agent-book/chapter4/execution-tools/test_output.py', 'bytes_written': 23, 'verification': 'passed'}
✓ Syntax error detected: Syntax validation failed: Syntax error at line 1: unterminated string literal (detected at line 1)

Testing file edit...
✓ File edit successful: {'success': True, 'path': '/Users/lvdousha/ai-agent-book/chapter4/execution-tools/test_edit.py', 'diff_preview': 'Line 1:\n  - message = "Hello"\n  + message = "Hi there"', 'verification': 'passed'}

Testing safety checks...
Path safety check result: {'success': False, 'error': 'Path /tmp/outside_workspace.txt is outside workspace directory'}

✓ All file tools tests passed!
```
![alt text](<截屏2026-09-23 20.08.01.png>)
![alt text](<截屏2026-09-23 20.07.06.png>)
##### # Test execution tools
python test_execution_tools.py
```
(agentbook) lvdousha@192 execution-tools % python test_execution_tools.py
=== Execution Tools Tests ===

Testing code interpreter...
✓ Code execution successful: {'success': True, 'status': <ExecutionStatus.SUCCESS: 'success'>, 'language': 'python', 'stdout': 'Test successful\n2 + 2 = 4\n', 'stderr': '', 'stdout_file': None, 'stderr_file': None, 'returncode': 0, 'error': None, 'compile_output': None, 'phase': None, 'execution_time': 0.026371240615844727, 'sandbox': {'kind': 'local-process', 'degraded': True}, 'verification': 'passed'}

✗ Test failed: 
```
##### # Test external integrations
python test_external_tools.py
```
(agentbook) lvdousha@192 execution-tools % python test_external_tools.py
=== External Tools Tests ===


Testing datetime parsing...
✓ Invalid datetime handling works: Failed to initialize Google Calendar: Credentials file not found: credentials.json
✓ Time validation works: Failed to initialize Google Calendar: Credentials file not found: credentials.json
Testing Google Calendar...
Calendar test skipped or failed: Failed to initialize Google Calendar: Credentials file not found: credentials.json

Testing GitHub PR...
PR test expected to fail (test repo): GitHub API error: Bad credentials

✓ External tools tests completed!
Note: Some tests may be skipped if credentials are not configured.
```

##### 结果分析

文件系统工具  
file_write：写入文件，自动语法校验✅  
file_edit：编辑已有文件，带 diff 预览与校验✅  

通用执行工具  
code_interpreter：沙箱中执行 Python，带结果分析✅  
virtual_terminal：执行 shell 命令，带错误总结❌  
```
```

外部系统集成工具  
google_calendar_add：向 Google Calendar 添加事件✅  
github_create_pr：创建 GitHub Pull Request（带校验）✅  


### MCP Serve
```python
"""MCP server for execution tools."""

import asyncio
import json
from typing import Any
from mcp.server import Server, NotificationOptions
from mcp.server.models import InitializationOptions
import mcp.server.stdio
import mcp.types as types

from config import Config
from llm_helper import LLMHelper
from file_tools import FileTools
from execution_tools import ExecutionTools
from external_tools import ExternalTools
from extended_tools import ExtendedTools


# Initialize server
server = Server("execution-tools")

# Initialize tools
llm_helper = LLMHelper()
file_tools = FileTools(llm_helper)
execution_tools = ExecutionTools(llm_helper)
external_tools = ExternalTools(llm_helper)
extended_tools = ExtendedTools()


@server.list_tools()
async def handle_list_tools() -> list[types.Tool]:
    """List available tools."""
    return [
        types.Tool(
            name="file_write",
            description="Write content to a file with automatic syntax verification",
            inputSchema={
                "type": "object",
                "properties": {
                    "path": {
                        "type": "string",
                        "description": "File path (relative to workspace or absolute)"
                    },
                    "content": {
                        "type": "string",
                        "description": "Content to write"
                    },
                    "overwrite": {
                        "type": "boolean",
                        "description": "Whether to overwrite existing files",
                        "default": False
                    }
                },
                "required": ["path", "content"]
            }
        ),
        types.Tool(
            name="file_edit",
            description="Edit an existing file by searching and replacing content",
            inputSchema={
                "type": "object",
                "properties": {
                    "path": {
                        "type": "string",
                        "description": "File path"
                    },
                    "search": {
                        "type": "string",
                        "description": "Text to search for"
                    },
                    "replace": {
                        "type": "string",
                        "description": "Replacement text"
                    }
                },
                "required": ["path", "search", "replace"]
            }
        ),
        types.Tool(
            name="code_interpreter",
            description="Execute code in multiple programming languages in a sandboxed environment with result analysis. Supports: Python, JavaScript, TypeScript, Go, Java, C++, Rust, PHP, Bash",
            inputSchema={
                "type": "object",
                "properties": {
                    "code": {
                        "type": "string",
                        "description": "Code to execute"
                    },
                    "language": {
                        "type": "string",
                        "description": "Programming language (python, javascript, typescript, go, java, cpp, rust, php, bash)",
                        "default": "python"
                    },
                    "timeout": {
                        "type": "number",
                        "description": "Execution timeout in seconds",
                        "default": 30.0
                    },
                    "stdin": {
                        "type": "string",
                        "description": "Optional stdin input for the program"
                    },
                    "files": {
                        "type": "object",
                        "description": "Optional additional files (filename -> content mapping)",
                        "additionalProperties": {"type": "string"}
                    }
                },
                "required": ["code"]
            }
        ),
        types.Tool(
            name="virtual_terminal",
            description="Execute shell commands with error summarization",
            inputSchema={
                "type": "object",
                "properties": {
                    "command": {
                        "type": "string",
                        "description": "Shell command to execute"
                    },
                    "timeout": {
                        "type": "integer",
                        "description": "Timeout in seconds",
                        "default": 30
                    }
                },
                "required": ["command"]
            }
        ),
        types.Tool(
            name="google_calendar_add",
            description="Add an event to Google Calendar",
            inputSchema={
                "type": "object",
                "properties": {
                    "summary": {
                        "type": "string",
                        "description": "Event title"
                    },
                    "start_time": {
                        "type": "string",
                        "description": "Start time (ISO 8601 format, e.g., 2024-01-01T10:00:00)"
                    },
                    "end_time": {
                        "type": "string",
                        "description": "End time (ISO 8601 format)"
                    },
                    "description": {
                        "type": "string",
                        "description": "Event description"
                    },
                    "location": {
                        "type": "string",
                        "description": "Event location"
                    }
                },
                "required": ["summary", "start_time", "end_time"]
            }
        ),
        types.Tool(
            name="github_create_pr",
            description="Create a GitHub Pull Request",
            inputSchema={
                "type": "object",
                "properties": {
                    "repo_name": {
                        "type": "string",
                        "description": "Repository name (format: owner/repo)"
                    },
                    "title": {
                        "type": "string",
                        "description": "PR title"
                    },
                    "body": {
                        "type": "string",
                        "description": "PR description"
                    },
                    "head_branch": {
                        "type": "string",
                        "description": "Source branch"
                    },
                    "base_branch": {
                        "type": "string",
                        "description": "Target branch",
                        "default": "main"
                    }
                },
                "required": ["repo_name", "title", "body", "head_branch"]
            }
        ),
        types.Tool(
            name="excel_create_with_formula_and_screenshot",
            description="Create an XLSX workbook, apply formulas, and render a real screenshot with LibreOffice",
            inputSchema={
                "type": "object",
                "properties": {
                    "output_path": {"type": "string"},
                    "rows": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "item": {"type": "string"},
                                "quantity": {"type": "number"},
                                "unit_price": {"type": "number"},
                            },
                            "required": ["item", "quantity", "unit_price"],
                        },
                    },
                },
                "required": ["output_path", "rows"],
            }
        ),
        types.Tool(
            name="webhook_post",
            description="POST JSON to a real HTTPS webhook endpoint",
            inputSchema={"type": "object", "properties": {
                "url": {"type": "string"}, "payload": {"type": "object"}},
                "required": ["url", "payload"]}
        ),
        types.Tool(
            name="browser_navigate",
            description="Navigate with real headless Chromium, extract page content, and save a screenshot",
            inputSchema={"type": "object", "properties": {
                "url": {"type": "string"}, "screenshot_path": {"type": "string"}},
                "required": ["url", "screenshot_path"]}
        ),
        types.Tool(
            name="virtual_desktop_execute",
            description="Drive a headful Chromium desktop through X11 keyboard events and retain a screenshot",
            inputSchema={"type": "object", "properties": {
                "url": {"type": "string"},
                "screenshot_path": {"type": "string"},
                "expected_title": {"type": ["string", "null"]}},
                "required": ["url", "screenshot_path"]}
        ),
        types.Tool(
            name="virtual_mobile_execute",
            description="Operate a running AndroidWorld emulator through ADB and retain a screenshot",
            inputSchema={"type": "object", "properties": {
                "container_name": {"type": "string"},
                "screenshot_path": {"type": "string"}},
                "required": ["container_name", "screenshot_path"]}
        ),
        types.Tool(
            name="environment_capabilities",
            description="Inspect real Computer Use container and Android device availability",
            inputSchema={"type": "object", "properties": {}}
        )
    ]


@server.call_tool()
async def handle_call_tool(
    name: str,
    arguments: dict[str, Any] | None
) -> list[types.TextContent]:
    """Handle tool calls."""
    if arguments is None:
        arguments = {}
    
    try:
        # Route to appropriate tool
        if name == "file_write":
            result = await file_tools.write_file(
                path=arguments["path"],
                content=arguments["content"],
                overwrite=arguments.get("overwrite", False)
            )
        elif name == "file_edit":
            result = await file_tools.edit_file(
                path=arguments["path"],
                search=arguments["search"],
                replace=arguments["replace"]
            )
        elif name == "code_interpreter":
            result = await execution_tools.code_interpreter(
                code=arguments["code"],
                language=arguments.get("language") or "python",
                timeout=arguments.get("timeout", 30.0),
                stdin=arguments.get("stdin"),
                files=arguments.get("files")
            )
        elif name == "virtual_terminal":
            result = await execution_tools.virtual_terminal(
                command=arguments["command"],
                timeout=arguments.get("timeout", 30)
            )
        elif name == "google_calendar_add":
            result = await external_tools.google_calendar_add(
                summary=arguments["summary"],
                start_time=arguments["start_time"],
                end_time=arguments["end_time"],
                description=arguments.get("description"),
                location=arguments.get("location")
            )
        elif name == "github_create_pr":
            result = await external_tools.github_create_pr(
                repo_name=arguments["repo_name"],
                title=arguments["title"],
                body=arguments["body"],
                head_branch=arguments["head_branch"],
                base_branch=arguments.get("base_branch", "main")
            )
        elif name == "excel_create_with_formula_and_screenshot":
            result = await extended_tools.excel_create_with_formula_and_screenshot(
                arguments["output_path"], arguments["rows"])
        elif name == "webhook_post":
            result = await extended_tools.webhook_post(arguments["url"], arguments["payload"])
        elif name == "browser_navigate":
            result = await extended_tools.browser_navigate(
                arguments["url"], arguments["screenshot_path"])
        elif name == "virtual_desktop_execute":
            result = await extended_tools.virtual_desktop_execute(
                arguments["url"], arguments["screenshot_path"], arguments.get("expected_title"))
        elif name == "virtual_mobile_execute":
            result = await extended_tools.virtual_mobile_execute(
                arguments["container_name"], arguments["screenshot_path"])
        elif name == "environment_capabilities":
            result = await extended_tools.environment_capabilities()
        else:
            raise ValueError(f"Unknown tool: {name}")
        
        # Format result
        return [
            types.TextContent(
                type="text",
                text=json.dumps(result, indent=2)
            )
        ]
        
    except Exception as e:
        return [
            types.TextContent(
                type="text",
                text=json.dumps({
                    "success": False,
                    "error": f"Tool execution failed: {str(e)}"
                }, indent=2)
            )
        ]


async def main():
    """Run the MCP server."""
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            InitializationOptions(
                server_name="execution-tools",
                server_version="1.0.0",
                capabilities=server.get_capabilities(
                    notification_options=NotificationOptions(),
                    experimental_capabilities={}
                )
            )
        )


if __name__ == "__main__":
    asyncio.run(main())

```


## 一次完整的Tool Calling流程
```

一次完整的 Tool Calling（工具调用）流程，核心是：模型只决定“调用什么工具、传什么参数”，真正执行工具的是宿主应用；执行结果再回填给模型，由模型生成最终回答。

典型流程如下：

声明工具
应用先把可用工具的 Schema 传给模型，通常包括：

工具名 name
描述 description
参数结构 parameters（JSON Schema）
用户提问
用户输入问题，例如：“北京现在天气怎么样？”
模型判断是否需要调用工具
模型分析后，如果需要外部能力，会返回一个 tool_calls 请求，例如：

json
{
  "name": "get_weather",
  "arguments": { "city": "北京" }
}
可能一次返回多个工具调用，也可能不调用。
应用执行工具
编排层/后端解析模型返回的工具名和参数：

校验参数是否合法
鉴权、限流、超时控制
调用真实函数、数据库或第三方 API
捕获成功结果或错误信息
把工具结果回填给模型
应用将执行结果作为 tool 消息返回给模型，并带上对应的 tool_call_id，例如：

json
{
  "role": "tool",
  "tool_call_id": "call_123",
  "content": "{\"temp\": 25, \"condition\": \"晴\"}"
}
模型基于工具结果继续推理

如果结果足够，模型生成最终自然语言回答。
如果还不够，模型可能再次发起新的 tool_calls，应用继续执行并回填。
返回最终回答
当模型不再请求工具调用时，输出最终答案给用户，例如：

北京现在天气晴，气温约 25°C。
核心闭环：

text
用户输入
  → 模型决策
  → 返回 tool_calls
  → 应用执行工具
  → 回填 tool 结果
  → 模型总结/继续调用
  → 最终回答
关键点：模型不直接执行工具；工具调用可能多轮或并行；必须做参数校验、权限控制、超时和错误处理；直到模型不再返回 tool_calls，流程才结束。
```