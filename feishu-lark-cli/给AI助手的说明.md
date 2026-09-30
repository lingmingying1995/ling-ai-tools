# 给AI助手的说明（飞书集成交接）

> **目标读者**：AI 助手（WorkBuddy / opencode / Claude Code / Cursor 等皆可）。
> 用户从朋友那里拿到这个目录，会对你说：「按这篇文档帮我完成飞书与本地的打通」。
> 你读完本说明，配合同目录的《飞书LarkCLI集成搭建.md》直接执行，不需要用户一步步指挥。

---

## 交接就是一句话

用户把《飞书LarkCLI集成搭建.md》（就在本目录）发给你，说「按这篇文档帮我完成飞书与本地的打通」。之后你能自主完成六步，全程只需要用户点两次浏览器。

## 你能自主完成的六步（按顺序做完）

1. **建专用目录**：在用户主项目仓库**外面**建（如 `D:\AI programs\feishu-workspace`，安装器会往 cwd 塞 `.agents/skills/` + `skills-lock.json`，别污染主项目）。后续所有命令在这个目录下跑。
2. **装 CLI 和官方 skills**：
   ```powershell
   npm install -g @larksuite/cli
   npx -y skills add https://open.feishu.cn --skill -y
   ```
3. **后台拉起配置进程提取链接**：`lark-cli config init --new` 是阻塞式的（约10分钟），**必须后台跑**——stdout/stderr 重定向到文件，轮询读取，一出 `https://open.feishu.cn/page/cli?...` 链接立刻发给用户（命令模板见文档第三节第3步）。
4. **引导授权**：`lark-cli auth login --recommend` 同样后台跑，提取 `verification_url` 发给用户（可配 `lark-cli auth qrcode "<URL>" --output qrcode.png` 出二维码手机扫）。
5. **验证闭环**：`lark-cli auth status` 确认 `identity=user`；然后 docs +fetch → +append 写入 → +fetch 回读，**读写全链路测一遍**。
6. **生成你自己的 AGENTS.md**：把文档附录的模板放进专用目录（若你的 agent 不读 AGENTS.md，把铁律抄进它认的规则文件，如 CLAUDE.md / .cursor/rules）。

## 用户的参与点（只有两次，都是点浏览器）

1. 点你发的**建应用链接** → 浏览器创建应用（App ID/Secret 由 CLI 自动管理，不用手动抄）
2. 点你发的 **OAuth 授权链接** → 用**她自己的**飞书账号授权

> 注意：她的授权跟文档作者（朋友）的授权**互相独立**，账号不互通，不需要朋友这边做任何操作。

## 执行铁律（运行时按文档第五、六节展开）

- 身份一律 `--as user`（bot 身份访问不了共享给个人的库）
- 传中文内容必须写 UTF-8 文件再 `--content "@./content.md"`，**禁止 PowerShell 管道传 stdin**（中文全变问号，且返回 ok 不报错）
- 写入后必须 `docs +fetch` 回读验证，不能只看返回 ok
- 删库/删文档等高风险操作先问用户

## 完成标志

验证闭环通过 + 铁律落到专用目录的 AGENTS.md + 授权长期有效（token 自动刷新，日常不用管）。完成后告诉用户：平时怎么用（直接对她说"帮我把XX写进飞书文档"即可）。
