# 养记 · 在线版

手机发视频链接/文字，AI 结合你的背景分析内容，按意图走不同通道（分析入库/讨论解惑/执行任务），把分析结果写成 Markdown 笔记推送到你的 GitHub 仓库。电脑 git pull 看笔记。

## 怎么用

1. 打开 [在线聊天页面](https://lingmingying1995.github.io/ling-ai-tools/yangji-agent/)
2. 第一次对话时填 GitHub 信息（token/owner/repo）
3. 发视频链接或文字内容
4. Agent 分析后问"要不要存下来？"
5. 确认后笔记推送到你的 GitHub 仓库
6. 电脑 git pull 看到笔记

## 前置条件

- GitHub 账号 + 仓库
- GitHub Personal Access Token（勾选 repo 权限）
- 仓库根目录有 `profile.md`（用户背景）

## 仓库结构

```
你的仓库/
├── profile.md          # 用户背景
├── intents/            # 意图记录（Agent自动维护）
│   └── 2026-08.md
└── notes/              # 笔记（Agent自动写入）
    └── 2026-08/
```

## 技术栈

- Agent 平台：Coze 扣子编程
- 存储：GitHub API
- 前端：Coze Web SDK
