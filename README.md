# Obsidian Git 同步仓库

## 仓库定位
- 本仓库用于保存一个可直接在 Obsidian 中打开的知识库，并通过 Git 管理版本与多端同步。
- 配置文件（`.obsidian`）随仓库分发，可保证不同设备获得一致的编辑体验。
- `.gitignore` 已排除个性化视图（如 `workspace.json`）与插件二进制，避免同步本地使用痕迹。

## 目录结构
- `2025/`：按年度拆分的通用资料，便于基于时间的检索与归档。
- `智慧林业/`：针对特定主题或项目的集中整理目录。
- `.obsidian/`：Obsidian 的核心配置，包含图谱、应用参数与插件列表；若需要二次配置，可在本地修改后自行决定是否提交。

## 使用方式
1. `git clone git@your.git.server:git-sync.git`（按需替换仓库地址），或直接在现有目录下执行 `git pull` 更新。
2. 在 Obsidian 中选择“打开现有库”，指向 `git-sync` 目录即可加载全部内容。
3. 关闭安全模式，启用仓库中列出的社区插件：
   - `obsidian-git`：提供自动/手动的 Git 同步命令。
   - `obsidian-custom-attachment-location`：把附件统一写入自定义路径，保持媒体文件整洁。
   - `obsidian-enhancing-export`：用于导出更友好的排版格式。
4. 根据需要调整主题、面板布局等本地偏好，这些设置不会被纳入版本控制。

## 同步流程
- **日常写作**：使用 `Ctrl+S` 触发 Obsidian 自动保存；`obsidian-git` 会在设定的间隔内检测改动。
- **手动同步**：通过命令面板运行 `Obsidian Git: Commit and push`，或在终端执行 `git status && git add -A && git commit -m "..." && git push`。
- **切换设备**：在另一台设备开始工作前执行 `git pull`，确保本地配置与文件保持最新。
- **解决冲突**：优先使用外部 Git 工具查看差异，必要时在 Obsidian 中手动合并，再执行一次提交。

## 写作与组织约定
- 保持相对路径链接（`useMarkdownLinks = true`），以便在不同文件系统下仍能正常跳转。
- 创建新页面时优先放入匹配的时间或专题目录，避免出现无归属的散落文件。
- 附件请使用 Obsidian 的“插入附件”功能，交由 `obsidian-custom-attachment-location` 插件处理存放位置。
- 若新增社区插件，请在 `.obsidian/community-plugins.json` 中登记并写入本说明，方便其他设备同步安装。

## 维护建议
- 定期运行 `git pull --rebase`，减少同步冲突。
- 提交信息使用简短的中文概括（例如“调整目录结构”“新增插件配置”）以便回溯。
- 如需共享导出结果，可通过 `obsidian-enhancing-export` 生成 PDF/HTML，而无需直接同步生成文件。
