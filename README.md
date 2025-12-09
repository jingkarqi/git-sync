# 我的 Obsidian 小宇宙

写作、灵感、随手的碎碎念都住在这个仓库里。它是我在不同电脑之间同步 Obsidian 的办法，只要有 Git 就能把整个空间拎走。这里没有公开内容的细节，只有怎样一起维护这个空间的一些小提示。

## 文件都放哪儿了
- `2025/`：当年的所有记录都塞进这里，方便以后按年份翻回去。
- `智慧林业/`：给某些长期折腾的话题留的位置，省得在一堆日志里找。
- `.obsidian/`：我的主题、面板和插件配置。带上它们，就能在任何机器上看到熟悉的布局。

## 我一般怎么用
1. `git clone <仓库地址>` 或者在老设备上 `git pull` 一下，就能拿到最新版。
2. 打开 Obsidian，选“打开已有库”，指到这个目录，等它把配置都读进去。
3. 关掉安全模式，把我常备的几个插件打开：
   - `obsidian-git`：定时自动提交，懒得敲命令的时候全靠它。
   - `obsidian-custom-attachment-location`：附件不再到处乱飞。
   - `obsidian-enhancing-export`：偶尔需要导出，排版好看不少。
4. 主题、面板什么的随便调，反正 `.gitignore` 已经把个性化文件忽略了。

## 同步的节奏
- 平时就是写完随手保存，让 `obsidian-git` 自动 commit。
- 要是心里没底，就打开命令面板手动执行 `Obsidian Git: Commit and push`，或在终端跑 `git status && git add -A && git commit && git push`。
- 换设备前一定记得 `git pull`，这样不会把另一台机器的更改顶掉。
- 真遇到冲突就别慌，VS Code 或任何你顺手的 diff 工具看一眼，合并完再 push。

## 一些约定
- 我默认用 Markdown 相对路径链接（`useMarkdownLinks = true`），所以别手痒改成 Wiki 链接。
- 新文件想想要放在哪个目录，时间序列就丢 `2025/`，专题内容去对应文件夹。
- 附件记得用 Obsidian 的“插入附件”按钮，让插件安排好去处。
- 如果装了新插件，别忘了把名字写进 `.obsidian/community-plugins.json`，顺手也在这里记一句。

## 维护 tips
- 偶尔 `git pull --rebase`，减少因为不同步导致的噪音。
- 提交信息就用中文短句，方便我日后追溯“当时在干嘛”。
- 需要把内容分享给别人？直接用 `obsidian-enhancing-export` 导出 PDF/HTML，仓库本身还是保持干净。

就这样，一个轻量又带点个人味道的 README。如果你也在一起维护这个空间，希望它能帮你快速上手。
