# moleportal-group-send

一个 Agent Skill：通过 MCP 创建库米云「客户群群发」任务。

说人话就能用，不用手工拼请求体，也不用操心群 ID 该填哪个字段。

---

## 安装

把 `INSTALL_PROMPT.md` 里那段话整段复制，发给你的 AI 助手即可。

装好后技能落在：

```
~/.workbuddy/skills/moleportal-group-send/
```

## 前置条件

使用前**必须**先配好 `moleportal` 这个 MCP 服务，缺两项信息：

1. **MCP 服务地址**，形如 `https://<host>/mcp`
2. **Bearer Token**，形如 `Bearer <一串字符串>`

这两项找管理员要。技能会在发现配置缺失时停下来向你索要，**不会凭记忆编造**。

配置写在本机的 `~/.workbuddy/mcp.json`（其它客户端各有各的配置文件，字段一致）：

```json
{
  "mcpServers": {
    "moleportal": {
      "type": "http",
      "url": "<管理员给你的 MCP 地址>",
      "headers": {
        "Authorization": "Bearer <管理员给你的 Token>"
      }
    }
  }
}
```

改完通常需要重启客户端、或在连接器管理页点信任才生效。

> **OpenAPI 业务凭据固化在服务端 MCP 进程里，你不需要也不应该传。**

## 能做什么

| 你说 | 它做 |
|---|---|
| 给 A 群、B 群发同一句话 | 指定群 + 统一话术 |
| 给每个群发不同的话术 | 指定群 + 个性话术 |
| 发给所有客户群 | 全选（会先提示群总数） |
| 先建好，等我确认再发 | 默认就是草稿，不发 |

**默认存草稿**，必须你说"立即发出"才会真的投递 —— 群发不可撤回，这是刻意设计的安全阀。

## 注意

- **项目不是你能选的**：服务端凭据绑定了具体项目，能看到哪些群、能发给谁都由它决定。
- 群名必须一字不差（"蒙阴"和"蒙明"是两个名字）。不确定就用 `customer-group-send groups` 查。
- 全选会覆盖该项目所有客户群，可能几百个 —— 执行前务必确认。

完整参数表、调用示例和避坑清单见 `skills/moleportal-group-send/SKILL.md`。
