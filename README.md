# 三术 (Sanshu) — Fork 二开记录

基于 [yuaotian/sanshu](https://github.com/yuaotian/sanshu) 的二次开发分支。

## 配置 MCP 客户端

### 配置环境变量

将可执行文件所在目录添加到系统 PATH 环境变量中：

<details>
<summary>🪟 Windows</summary>

**方法一：图形界面**

1. `Win + R` → 输入 `sysdm.cpl` → 回车
2. 「高级」→「环境变量」
3. 找到 `Path` →「编辑」→「新建」→ 添加可执行文件目录（如 `C:\Program Files\sanshu\`）
4. 确定保存，**重启终端**生效

**方法二：PowerShell**

```powershell
# 用户级（无需管理员权限）
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$userPath;C:\Program Files\sanshu\", "User")

# 系统级（需管理员权限）
$machinePath = [Environment]::GetEnvironmentVariable("Path", "Machine")
[Environment]::SetEnvironmentVariable("Path", "$machinePath;C:\Program Files\sanshu\", "Machine")
```

> ⚠️ 配置后需**重启终端**或**重新登录**生效

</details>

<details>
<summary>🐧 Linux</summary>

```bash
# Bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc

# Zsh
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```

</details>

<details>
<summary>🍎 macOS</summary>

```bash
# Zsh（默认 Shell）
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc

# Bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bash_profile && source ~/.bash_profile
```

</details>

### 添加 MCP 服务器

在 MCP 客户端配置文件中添加：

```json
{
  "mcpServers": {
    "sanshu": {
      "command": "三术"
    }
  }
}
```

> [!IMPORTANT]
> 环境变量配置是 MCP 客户端正常工作的前提条件。
>
> - ✅ **已配置 PATH**：可直接使用 `"command": "三术"` 或 `"command": "sanshu"`
> - ❌ **未配置 PATH**：需使用完整路径，如 `"command": "C:\\Program Files\\sanshu\\三术.exe"`
>
> 某些客户端可能无法识别中文命令名 `三术`，可改用 `sanshu`。

### 配置提示词

在三术设置页「参考提示词」标签中点击复制按钮获取提示词，粘贴到对应 IDE 配置位置：

<details>
<summary>Cursor</summary>

设置 → `Rules, Skills, Subagents` → 添加 User Rule 或 Project Rule

</details>

<details>
<summary>VS Code Copilot</summary>

推荐[系统级自定义指令](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)：`~/.copilot/instructions/` 下创建 `.instructions.md` 文件

也可用项目级：`.github/copilot-instructions.md`

</details>

<details>
<summary>Claude Desktop / 其他</summary>

配置为系统提示词或项目级默认提示词

</details>

> 提示词会根据启用的 MCP 工具自动调整内容。

## 二开内容

- 本地提示词增强，移除外部服务依赖
- 结构化响应（用户输入/选项/图片/文件/条件偏好分区传递）
- 图片 Badge 索引序号 + 文件名、截图自动唯一命名、拖拽自动聚焦
- `prefill` 弹窗预填充
- Markdown 渲染对齐 Naive UI + 代码块复制修复
- Fork 品牌重塑、UI 标准化、本地字体、置顶常驻、Toast/Modal 层级修复
- 关闭按钮双色反馈（黄色警告 → 红色确认）
- `zhi` 子代理约束 + 抗上下文压缩 + 强制 `project_root_path`
- 执行偏好独立 Content::text、连续取消关闭保护、统一弹窗协议
- WebView2 多实例黑屏修复、Context7 重定向处理
- 自定义提示词独立存储 + 条件性开关 + MCP 工具联动
- 自动更新（CDN 检查 + 标题栏按钮 + 下载进度 + auto-tag 发布）
