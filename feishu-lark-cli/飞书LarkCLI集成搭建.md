# 飞书LarkCLI集成搭建

> 日期：2026-09-29
> 一句话：打通「AI agent 读写飞书文档/wiki」，飞书官方 Lark CLI 路线，任何 agent 通用（不绑定 opencode）。

## 一、成果

AI agent 能以**用户本人身份**读写飞书云文档和知识库——包括别人分享给你、你只有编辑权限的共创库。读、追加、精确改块、删块全链路可用。

## 二、选型事实（为什么是 Lark CLI）

| 路线 | 事实结论 |
|------|---------|
| 远程 MCP（open.feishu.cn/page/mcp） | Beta 灰度，普通账号无权限进不去；2026-02-24 起新链接 7 天过期；官方声明个人授权流会逐步淘汰 |
| 本地 MCP（npm `@larksuiteoapi/lark-mcp`） | 官方明确限制：**不能直接编辑云文档**（只能导入和读取），不能传下载文件 |
| **Lark CLI（npm `@larksuite/cli`）** | 官方主推新路线，开源（github.com/larksuite/cli）；用户身份授权可读写文档/wiki/表格/多维表格/日历/邮件；**选定** |

关键权限事实：**朋友共享的库、自己只有编辑权 → 应用身份（bot token）走不通，必须走用户身份授权**（用户 token 权限=本人权限）。

## 三、搭建步骤（可复刻）

**前置**：Node.js + 任意 AI agent（opencode / Claude Code / Cursor / Codex / TRAE / Gemini CLI 均可）。

1. **建专用目录**（别在主项目仓库里跑后面所有命令，安装器会往 cwd 塞 `.agents/skills/` + `skills-lock.json`）：
   ```powershell
   # 例如 D:\AI programs\feishu-workspace（跟主项目同级）
   ```

2. **装 CLI 和官方 skills**（在专用目录下跑）：
   ```powershell
   npm install -g @larksuite/cli
   npx -y skills add https://open.feishu.cn --skill -y
   ```

3. **创建自建应用**（CLI 会引导浏览器建应用，App ID/Secret 自动管理，不用手动抄）：
   ```powershell
   lark-cli config init --new   # 阻塞式（约10分钟），必须后台跑：把 stdout/stderr 重定向到文件，轮询读取出链接后发给用户
   # Windows：Start-Process -FilePath cmd.exe -ArgumentList "/c","lark-cli","config","init","--new" -RedirectStandardOutput out.txt -RedirectStandardError err.txt
   # macOS/Linux：nohup lark-cli config init --new > out.txt 2>&1 &
   # 然后轮询读 out.txt/err.txt，提取 https://open.feishu.cn/page/cli?... 链接发给用户，等用户浏览器完成
   ```

4. **用户身份授权**（关键一步，token 权限=本人权限）：
   ```powershell
   lark-cli auth login --recommend   # 同样后台跑 + 提取 verification_url 给用户（算法同上）
   lark-cli auth qrcode "<URL>" --output qrcode.png   # 配套二维码（手机扫）
   # 等用户完成授权后跑 lark-cli auth status 确认 identity=user
   ```

5. **验证闭环**（读写全测一遍）：
   ```powershell
   lark-cli auth status
   lark-cli docs +fetch --doc "<文档URL>" --doc-format markdown --as user
   # 写入（内容放 UTF-8 文件，见铁律）
   lark-cli docs +update --doc "<文档URL>" --command append --content "@./content.md" --doc-format markdown --as user
   # 回读验证
   lark-cli docs +fetch --doc "<文档URL>" --doc-format markdown --as user
   ```

## 四、跨 agent 兼容（核心设计）

**根本事实：lark-cli 是普通 CLI**。任何能跑 shell 命令的 agent 都能直接用，兼容性不依赖任何 agent 的插件机制。要做的只是让每个 agent 知道「怎么用」：

| 层 | 作用 | 兼容方式 |
|----|------|---------|
| CLI 本体 | 干活的 | agent 无关，shell 就行 |
| 使用铁律 | 防坑规则（身份/编码/验证） | 写进 `AGENTS.md`——跨 agent 事实标准：opencode、Codex 原生读；Claude Code、Cursor 新版支持读；旧版 Claude Code 用 `CLAUDE.md`、旧版 Cursor 用 `.cursor/rules/`，内容抄同一份铁律即可 |
| 官方 lark skills | 各业务域用法手册（docs/wiki/drive/...28个） | 本质是 Markdown 文档，`lark-cli skills read <名字>` 或直接读文件；注册机制因 agent 而异，**不注册也能用** |
| 飞书官方安装指令 | 换 agent 重装时用 | 把安装指南 URL 发给任意 agent 让它自己装：`Help me install Feishu CLI: https://open.feishu.cn/document/no_class/mcp-archive/feishu-cli-installation-guide.md` |

**最小兼容集 = 专用目录放一份 AGENTS.md（铁律+命令速查）+ 任何 agent 调 lark-cli**。新 agent 接入只需确认它读 AGENTS.md（或把铁律抄进它认的规则文件）。

## 五、使用铁律（防坑）

1. **身份一律 `--as user`**（bot 身份访问不了共享给个人的库）
2. **传中文内容必须用文件**：写成 UTF-8 文件再 `--content "@./content.md"`。禁止 PowerShell 管道传 stdin（PS 5.1 默认 ASCII，中文全变 `?` 写进文档且返回 ok 不报错）；内容含引号等特殊符号同样走文件
3. **写入后必须回读验证**，不能只看返回 ok
4. 删库/删文档等高风险操作先跟用户确认

## 六、踩坑记录

- **PowerShell 管道传中文 → 全变问号**：`@'中文'@ | lark-cli --content -` 写入"成功"但内容全是 `?`。根因 PS 5.1 `$OutputEncoding` 默认 ASCII。修复=走 @file。
- **中文引号截断参数**：内容里的 “” 会被 lark-cli 参数解析器当成分隔符，报 "positional arguments are not supported"。修复=走 @file。
- **远程 MCP 平台无权限**不是账号问题，是灰度，别死磕，直接走 Lark CLI。
- **在主项目仓库跑安装命令**会把 `.agents/skills/`（28 个 skill 目录）+ `skills-lock.json` 塞进仓库。安装类命令在专用目录跑。

## 七、维护

- skill 更新：专用目录下重跑 `npx -y skills add https://open.feishu.cn --skill -y`
- 授权过期：`lark-cli auth status` 查状态，`lark-cli auth login --recommend` 重新授权（token 自动刷新，长期不用手动管）
- 凭证位置：lark-cli 的应用凭证和 token 都在用户主目录，专用目录不含 key，可安全同步
- 本机实例：目录 `feishu-workspace/`（含 AGENTS.md 实例细节：App ID、共创文档 URL 等，不进分享文档）

## 附录：专用目录 AGENTS.md 模板（复制即用）

> 建好后放进专用目录，opencode/Codex/Claude Code 等新窗口自动加载；旧版不读 AGENTS.md 的 agent，把内容抄进它认的规则文件（CLAUDE.md / .cursor/rules 等）。

```markdown
# AGENTS.md — 飞书工作区

## 环境
- lark-cli（飞书官方 CLI）已配置自建应用 + 用户身份授权（token 自动刷新）
- .agents/skills/ 有飞书官方 lark skill，做任务前先读：lark-cli skills read lark-doc/references/<文件>.md

## 铁律
1. 身份一律 --as user（bot 身份访问不了共享给个人的库）
2. 传中文内容必须写 UTF-8 文件再 --content "@./content.md"，禁止 PowerShell 管道传 stdin（中文全变问号）
3. 写入后必须 docs +fetch 回读验证，不能只看返回 ok
4. 删库/删文档等高风险操作先跟用户确认

## 常用命令（<doc> 可传 wiki/docx URL 或 token）
lark-cli auth status
lark-cli docs +fetch --doc "<doc>" --doc-format markdown --as user
lark-cli docs +update --doc "<doc>" --command append --content "@./content.md" --doc-format markdown --as user
lark-cli docs +fetch --doc "<doc>" --detail with-ids --as user   # 拿 block ID 再精确改
```

**交接方式**：把这个文档发给对方，说「按这篇文档帮我完成飞书与本地的打通」，AI 即可自主执行，全程只需要用户点两次浏览器链接（建应用 + 授权）。
