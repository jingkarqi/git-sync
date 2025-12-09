dddCLI项目
借助远程调用大语言模型，用自然语言在终端中执行命令

基本命令：
`ddd <自然语言>`
示例：
`ddd 帮我删除所有.txt结尾的文件`
`ddd 使用git命令推送当前项目到远程仓库`

高级命令：
`ddd help` 帮助文档

`ddd setting` 进入设置页面

`ddd whitelist` 命令白名单（始终执行无需同意）

`ddd blacklist` 命令黑名单（自动拒绝完全不同意）

环境：
Windows 11 PowerShell 5.1或更高版本

预期效果

```powershell
PS C:\test> ddd 删除所有.tmp文件

AI 建议执行：
Remove-Item *.tmp -Force

确认执行？[Y/n]: Y

已删除 5 个临时文件。
```

```powershell
PS C:\test> ddd setting
  Base_URL设置
→ API_Key设置
  Model_ID设置
```