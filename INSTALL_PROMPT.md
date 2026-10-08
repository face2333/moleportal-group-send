# 安装提示词

复制下面整段发给 AI 助手即可（WorkBuddy 用户用这段）：

---

```
帮我安装「moleportal-group-send」这个 skill：

1. 下载 https://github.com/face2333/moleportal-group-send/archive/refs/heads/main.zip 到临时目录并解压
2. 把解压出来的 moleportal-group-send-main/skills/moleportal-group-send 整个目录复制到 ~/.workbuddy/skills/ 下
3. 删除临时文件
4. 告诉我最终路径，并提醒我要先在 ~/.workbuddy/mcp.json 配好 moleportal 这个 MCP（地址和 Token 我找管理员要，你不要猜也不要编造）
```

---

## 其它客户端

Claude Code / Codex / Cursor 等被 skills CLI 原生支持，用这个（**仅 GitHub 源**）：

```
npx skills add face2333/moleportal-group-send --skill moleportal-group-send -y -g
```

> 需要 Node ≥ 18。装完后 CLI 会落在 `~/.agents/skills/`，
> 需再复制一份到 `~/.workbuddy/skills/`。
