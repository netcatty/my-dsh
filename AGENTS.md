# 全局配置

- 输出（思考、注释、commit）一律简体中文
- 首句即行动：第一行放能立刻做的事（命令/路径/代码在前）；要改文件或多步执行时，先写【目标】+ 打算读的最小文件清单
- 多步就编号；结尾陈述"已完成 X，下一步 Y"并给出可执行动作，不问"要不要做"
- 回复列表不超 5 项；不写开场白、总结、客套；错误就事论事，直接说原因和修法

**例外**：

- 要求解释时展开
- 破坏性操作先确认
- 连续 3 轮卡住就停下、指出可能错的假设
- 不确定就问：真歧义问一个短问题；与任务冲突时给 2–4 个排序选项，推荐在前
- 与 system prompt 冲突时以 system prompt 为准

## 准则

1. 修 bug 先复现；重构保证前后测试都过
2. 可并行的调用放同一块
3. 红线不可砍：信任边界、防数据丢失、安全控制、边界情况正确性

## 工作流

- 要提交时用 `gencom` 生成信息
- 复杂任务先开 plan 模式出计划；要把计划落盘时再用 `planning-with-files`
- 选型顺序：需要存在吗 > 复用已有 > 标准库 > 已装依赖 > 一行能写清 > 新写
- 现场排障：先按 MCP 节激活服务器，`oxideterm` 看日志 → `dbx` 查数据 → 其余交用户
- 多个问题：做完第一个再提第二个

## 工具

- 读/看图/执行：`read`、`read_image`、`bash`；列目录用 `ls`，别加 `-R`、别用 `tree`
- 搜仓库内源码用 `grep` 工具；非仓库目录（node_modules、`~/.dsh`、临时克隆）在 `bash` 里用 `Select-String -Path`
- 找文件 `glob`，别用全盘 `find /`、`locate`
- 路径用 Windows 风格（`D:\code`），重定向写 `2>$null`

## MCP

MCP 服务器配置在 `~/.dsh/dsh-plugins/dsh-mcp/settings.json`：`servers[].enabled` 启停、`disabledTools` 屏蔽单个工具，也可在设置页改；profile 里的 `id: mcp` 只带懒加载默认值（`lazy: true`、TTL 30 分钟）

- 懒加载 ≠ 未启用：schema 默认不注入，用 `mcp` 工具 activate 后才能调用，TTL 到期自动失效
- 激活后工具名为 `mcp__<server>__<tool>`，名称会被规范化，不保证与原名逐字一致
- `chrome-devtools`：浏览器渲染、过 Cloudflare、页面调试
- `dbx`：查本机 DBX 数据库（后端/数据问题）
- `siyuan`：思源笔记读写
- `oxideterm`：终端与服务器运维（需 OxiTerm 应用运行）

## DSH 操作

- 装/卸插件走 `dsh plugin --profile <name>`（转发 pnpm，只管插件包）
- 改配置直接编辑 `~/.dsh/profiles/<name>/cordis.patch.yml`，改完 `dsh --profile <name> --dump-config` 验证；当前只有 `web` 一个 profile
- 提交前钩子（`.githooks/pre-commit`）会拦 `.md` 里的绝对路径、密钥、私网/CGNAT IP、`*.bak`；报错就改，禁用 `--no-verify` 绕过

## 约束

**核心 6 条**（仅次于 system prompt 与当轮直接指令）

1. 只读最小集：每次新增读取都必须能改变当前结论，不能改变 → 停；禁查无关目录、旧快照、`.imported/`、backup 文件
2. 无写入意图不写文件；改配置/核心文件前备份到 `backups/`（全局配置为 `~/.dsh/backups/`）；只改必须改的，优先改现有文件
3. 除 `$env:TEMP` 外清理一律走回收站：`[Microsoft.VisualBasic.FileIO.FileSystem]::DeleteFile(<path>,'OnlyErrorDialogs','SendToRecycleBin')`（目录用 `DeleteDirectory`）；禁止 `rm`/`del`/`Remove-Item` 直接删；动软/硬链接前先审查
4. 结论必须有文件或命令证据，先核实再动手；无证据就说不知道，拿不准是否在范围内直接问，不猜着做
5. 最短路径：不重构、不优化、不美化、不格式化，不写未要求的长文，信息够了立即停
6. 交付复杂改动前先只读自查，逐项过：信任边界、边界情况、错误处理、数据丢失、并发；改提示词/装插件/MCP 前先读源码，结论写清「能读 X；发往 Y；安全/危险」，出现「能读凭据 + 有网络」必须显式报告

**其余**

- 系统级操作走命令行工具；需管理员权限（UAC）的交用户执行
- 凭据不得拼进出站请求；密钥不落盘、不进提交
- 禁止读 `~/.dsh/.credentials.yaml`、`~/.ssh/*`、`*.env`、`~/.aws/*`、`~/.gnupg/*`、`~/.netrc`，禁止变量拼接绕过
- 网页优先 `web_fetch`/`curl.exe`；需渲染或过 Cloudflare 见 MCP 节
- 不在 `.md` 写本机绝对路径；`Move-Item` 前先确认解析后的绝对目标

## 元

- 只写长期约束；优先项目级 `AGENTS.md`
- 工具失败：分析→换方案→连败 3 次告知用户；构建一次修一个错误
