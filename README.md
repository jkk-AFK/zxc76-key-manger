## 概述与架构

这是一个基于命令行的 Python 管理系统，用于管理用户、激活码（卡密）和配置文件。整个应用的核心逻辑都封装在 `admin_system.py` 的 `AdminSystem` 类中。

- **数据存储**: 系统使用 JSON 文件作为简单的数据库：
  - `users.json`: 存储用户信息，包括用户名、密码和所属身份组。
  - `keys.json`: 存储激活码。每个激活码都有一个关联的身份组。
  - `groups.json`: 定义了所有可用的身份组（例如 "admin", "betauser", "user"）。
- **配置管理**: `.cfg` 配置文件存储在 `configs/` 目录中。
- **日志**: 操作日志记录在 `admin_system.log` 文件中。
- **核心依赖**: `rich` 库用于在终端中创建格式化的输出（如表格和颜色）。

## 关键工作流

### 运行系统

直接在终端中执行主脚本即可启动应用。系统会首先要求管理员登录。

```bash
python admin_system.py
```

### 激活码（卡密）生命周期

这是一个核心业务流程：

1.  **生成**: 管理员可以为特定的身份组（如 `user` 或 `betauser`）生成一个或多个16位的激活码。这些码被添加到 `keys.json` 中，状态为“未使用”。
2.  **激活**: 用户使用其用户名和激活码调用 `activate_key` 功能。
3.  **效果**: 系统验证激活码：
    - 将用户的身份组更新为与该激活码关联的身份组。
    - **激活码被立即从 `keys.json` 中删除**，确保其一次性使用。

示例: `activate_key(username="testuser", key="SOMEKEY123456789")` 会将 `testuser` 的身份组更改为 `SOMEKEY...` 所关联的组，然后删除这个 key。

## 代码约定与模式

- **数据处理**: 所有对 `users.json`, `keys.json`, 和 `groups.json` 的读写操作都通过 `_load_data` 和 `_save_data` 方法进行。在添加或修改任何数据后，请务必调用 `_save_data` 以确保更改被持久化。
- **返回格式**: 大多数方法返回一个元组 `(bool, str)`，其中 `bool` 表示操作是否成功，`str` 是给用户的反馈信息。
- **用户交互**:
  - 使用 `rich.console.Console` 和 `rich.table.Table` 来显示格式化的数据，而不是简单的 `print()`。
  - 使用 `getpass.getpass()` 来安全地输入密码。
- **身份组管理**: "admin" 是最高权限组。某些操作（如设置其他管理员）需要检查操作者是否为 `admin`。

## 外部依赖

- **rich**: 用于增强终端输出。安装命令如下：
  ```bash
  python -m pip install rich
  ```
在进行任何涉及终端输出的修改时，请优先使用 `rich` 库的功能。


