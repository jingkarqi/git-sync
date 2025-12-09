---
tags:
  - API-配置
  - Claude-Code
  - DeepSeek
  - PowerShell
  - 环境变量
---
```powershell
# ==============================================================================
# Claude Code 临时环境变量设置 (Windows PowerShell)
# 这些设置仅在当前 PowerShell 会话中有效。
# ==============================================================================

Write-Host "正在为当前会话配置 Claude Code 环境变量..." -ForegroundColor Yellow

# ------------------------------------------------------------------------------
# 1. API 提供商配置 (最关键的部分)
#    用于连接到非官方 Anthropic 的、但兼容其 API 的第三方服务。
# ------------------------------------------------------------------------------

# [必需] 设置 API 请求的基础 URL
# 示例:
# - DeepSeek: "https://api.deepseek.com" (注意: 根据其文档，可能不需要 /anthropic 后缀)
# - 智谱AI:   "https://open.bigmodel.cn/api/anthropic/v1"
$env:ANTHROPIC_BASE_URL = "https://api.deepseek.com"

# [必需] 设置您的 API 密钥
$env:ANTHROPIC_API_KEY = "在此处粘贴你的API密钥"


# ------------------------------------------------------------------------------
# 2. 模型别名覆盖 (可选)
#    如果您想改变 `opus`, `sonnet` 等别名具体指向的模型ID，可以在这里设置。
#    如果不设置，Claude Code 将使用其内部默认值。
# ------------------------------------------------------------------------------

# 定义 'opus' 或 'opusplan' (计划模式) 别名所使用的模型
$env:ANTHROPIC_DEFAULT_OPUS_MODEL = "deepseek-chat"

# 定义 'sonnet' 或 'opusplan' (执行模式) 别名所使用的模型
$env:ANTHROPIC_DEFAULT_SONNET_MODEL = "deepseek-chat"

# 定义 'haiku' 别名或用于后台功能的模型
$env:ANTHROPIC_DEFAULT_HAIKU_MODEL = "deepseek-coder"


# ------------------------------------------------------------------------------
# 3. 高级配置 (可选)
#    通常不需要修改。
# ------------------------------------------------------------------------------

# 定义用于子代理 (sub-agents) 的模型
$env:CLAUDE_CODE_SUBAGENT_MODEL = "deepseek-chat"


# ==============================================================================
# 验证步骤 - 检查已设置的环境变量
# ==============================================================================
Write-Host ""
Write-Host "环境变量设置完成。以下是当前会话中已设置的相关变量：" -ForegroundColor Green

# 查找并以表格形式清晰地显示所有已设置的 ANTHROPIC_* 和 CLAUDE_CODE_* 变量
Get-ChildItem Env: | Where-Object { $_.Name -like 'ANTHROPIC_*' -or $_.Name -like 'CLAUDE_CODE_*' } | Format-Table -AutoSize

Write-Host ""
Write-Host "现在您可以在此窗口中运行 'claude' 命令，它将使用以上配置。" -ForegroundColor Cyan

# 例如:
# claude "你好，请问你是哪个模型？"
```