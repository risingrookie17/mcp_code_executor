# MCP 代码执行器

[![smithery badge](https://smithery.ai/badge/@risingrookie17/mcp_code_executor)](https://smithery.ai/server/@risingrookie17/mcp_code_executor)

MCP 代码执行器是一个 MCP 服务器，允许 LLM 在指定的 Python 环境中执行 Python 代码。这使 LLM 能够运行访问环境中定义的库和依赖的代码。它还支持增量代码生成，以处理可能超出 token 限制的大型代码块。

<a href="https://glama.ai/mcp/servers/45ix8xode3"><img width="380" height="200" src="https://glama.ai/mcp/servers/45ix8xode3/badge" alt="Code Executor MCP server" /></a>

## 功能特性

- 执行 LLM 提示中的 Python 代码
- 支持增量代码生成以克服 token 限制
- 在指定环境中运行代码（Conda、virtualenv 或 UV virtualenv）
- 按需安装依赖
- 检查包是否已安装
- 运行时动态配置环境
- 可配置的代码存储目录

## 环境要求

- 已安装 Node.js
- 以下任一环境：
  - 已创建 Conda 环境的 Conda
  - Python virtualenv
  - UV virtualenv

## 安装步骤

1. 克隆仓库：

```bash
git clone https://github.com/risingrookie17/mcp_code_executor.git
```

2. 进入项目目录：

```bash
cd mcp_code_executor
```

3. 安装 Node.js 依赖：

```bash
npm install
```

4. 构建项目：

```bash
npm run build
```

## 配置

在 MCP 服务器配置文件中添加以下配置：

### 使用 Node.js

```json
{
  "mcpServers": {
    "mcp-code-executor": {
      "command": "node",
      "args": [
        "/path/to/mcp_code_executor/build/index.js"
      ],
      "env": {
        "CODE_STORAGE_DIR": "/path/to/code/storage",
        "ENV_TYPE": "conda",
        "CONDA_ENV_NAME": "your-conda-env"
      }
    }
  }
}
```

### 使用 Docker

```json
{
  "mcpServers": {
    "mcp-code-executor": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "mcp-code-executor"
      ]
    }
  }
}
```

> **注意：** Dockerfile 仅在 venv-uv 环境类型下测试过。其他环境类型可能需要额外配置。

### 环境变量

#### 必需变量
- `CODE_STORAGE_DIR`：生成代码的存储目录

#### 环境类型（选择一种）

- **Conda 环境：**
  - `ENV_TYPE`：设置为 `conda`
  - `CONDA_ENV_NAME`：要使用的 Conda 环境名称

- **标准 Virtualenv：**
  - `ENV_TYPE`：设置为 `venv`
  - `VENV_PATH`：virtualenv 目录路径

- **UV Virtualenv：**
  - `ENV_TYPE`：设置为 `venv-uv`
  - `UV_VENV_PATH`：UV virtualenv 目录路径

## 可用工具

MCP 代码执行器向 LLM 提供以下工具：

### 1. `execute_code`
在配置的环境中执行 Python 代码。适用于短代码片段。
```json
{
  "name": "execute_code",
  "arguments": {
    "code": "import numpy as np\nprint(np.random.rand(3,3))",
    "filename": "matrix_gen"
  }
}
```

### 2. `install_dependencies`
在环境中安装 Python 包。
```json
{
  "name": "install_dependencies",
  "arguments": {
    "packages": ["numpy", "pandas", "matplotlib"]
  }
}
```

### 3. `check_installed_packages`
检查包是否已安装在环境中。
```json
{
  "name": "check_installed_packages",
  "arguments": {
    "packages": ["numpy", "pandas", "non_existent_package"]
  }
}
```

### 4. `configure_environment`
动态更改环境配置。
```json
{
  "name": "configure_environment",
  "arguments": {
    "type": "conda",
    "conda_name": "new_env_name"
  }
}
```

### 5. `get_environment_config`
获取当前环境配置。
```json
{
  "name": "get_environment_config",
  "arguments": {}
}
```

### 6. `initialize_code_file`
创建包含初始内容的新 Python 文件。对于可能超出 token 限制的较长代码，请使用此作为第一步。
```json
{
  "name": "initialize_code_file",
  "arguments": {
    "content": "def main():\n    print('Hello, world!')\n\nif __name__ == '__main__':\n    main()",
    "filename": "my_script"
  }
}
```

### 7. `append_to_code_file`
将内容追加到现有 Python 代码文件。使用此工具可向使用 initialize_code_file 创建的文件添加更多代码。
```json
{
  "name": "append_to_code_file",
  "arguments": {
    "file_path": "/path/to/code/storage/my_script_abc123.py",
    "content": "\ndef another_function():\n    print('This was appended to the file')\n"
  }
}
```

### 8. `execute_code_file`
执行现有的 Python 文件。在使用 initialize_code_file 和 append_to_code_file 构建代码后，使用此作为最后一步。
```json
{
  "name": "execute_code_file",
  "arguments": {
    "file_path": "/path/to/code/storage/my_script_abc123.py"
  }
}
```

### 9. `read_code_file`
读取现有 Python 代码文件的内容。在追加更多内容或执行文件之前，使用此工具验证文件的当前状态。
```json
{
  "name": "read_code_file",
  "arguments": {
    "file_path": "/path/to/code/storage/my_script_abc123.py"
  }
}
```

## 使用方法

配置完成后，MCP 代码执行器将允许 LLM 通过在指定的 `CODE_STORAGE_DIR` 中生成文件并在配置的环境中运行来执行 Python 代码。

LLM 可以通过在提示中引用此 MCP 服务器来生成和执行代码。

### 处理大型代码块

对于可能超出 LLM token 限制的大型代码块，请使用增量代码生成方法：

1. **初始化文件** - 使用 `initialize_code_file` 创建基本结构
2. **添加更多代码** - 在后续调用中使用 `append_to_code_file`
3. **验证文件内容** - 如需要使用 `read_code_file`
4. **执行完整代码** - 使用 `execute_code_file`

这种方法允许 LLM 编写复杂的多部分代码，而不会遇到 token 限制问题。

## 向后兼容性

此包与早期版本保持向后兼容性。仅指定 Conda 环境的老用户无需更改配置即可继续工作。

## 贡献

欢迎贡献！请提交 Issue 或 Pull Request。

## 许可证

本项目基于 MIT 许可证授权。
