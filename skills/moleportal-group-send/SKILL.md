---
name: moleportal-group-send
description: 通过服务器端 moleportal_mcp（MCP）创建库米云「客户群群发」任务 —— 按群名自动解析群、支持统一话术与个性话术、默认存草稿确认后才发送。触发词：客户群群发、群发任务、给客户群发消息、群发话术、企微群发、customer-group-send、群发到所有群。
agent_created: true
---

# 库米云 · 客户群群发（MCP 版）

一句话创建客户群群发任务，不用手工拼请求体。

> 分工：本技能走服务器 MCP 执行写操作，凭据在服务器端。
> 不要改用本地 CLI（本地没有 OpenAPI 凭据，会报 `Missing OpenAPI config`）。

---

## 一、接入 MCP 服务（首次使用必做）

⚠️ **前置检查：先确认本机已配置 `moleportal` 这个 MCP 服务**（WorkBuddy 看 `~/.workbuddy/mcp.json`，其它客户端各有各的配置文件，字段一致）。

若配置里**没有** `moleportal`，**停下来向用户索要**这两样，不要猜、不要凭记忆编造：

1. **MCP 服务地址** —— 形如 `https://<host>/mcp`
2. **Bearer Token** —— 形如 `Bearer <一串字符串>`

拿到后写进配置文件的 `mcpServers` 段：

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

> 改完通常需要重启客户端、或在连接器管理页点信任才生效。

可用工具：`moleportal_cli_run`（执行）、`moleportal_cli_help`（查帮助）、`moleportal_cli_version`（版本号）。

**业务侧凭据（callerId / security）已固化在服务器 MCP 进程环境中，调用方无需也不应传。**

---

## 二、调用规范（两个必踩的坑）

### ⚠️ 坑 1：只用 `argv` 传参，禁止用 `options`

服务端 `options` 字段存在 pydantic 校验缺陷，传字符串或对象都会报
`Input should be a valid string`，**该字段实际不可用**。所有参数一律平铺进 `argv`：

```json
{"argv": ["openapi","--allow-risk","write","customer-group-send","create","--name","寒露关怀","..."], "timeout_s": 120}
```

### ⚠️ 坑 2：`--allow-risk` 是 `openapi` 的**组级**参数

必须放在 `"openapi"` **之后**、子命令之前：

```json
{"argv": ["openapi","--allow-risk","write","customer-group-send","create", "..."]}
```

漏了会被直接拒绝。**别担心误发** —— 没带这个参数命令根本不执行，而且默认还是草稿。

### 其它约定

- `json_output` 保持默认 `true`
- `timeout_s` 给 **120**（群多时翻页需要时间）
- argv 里**不要**写二进制名，从 `openapi` 开始
- 项目由服务端凭据决定，**调用方选不了**，`k_project` 对 `openapi` 子命令无效

---

## 三、`customer-group-send create` —— 创建群发任务

### 参数表

| 参数 | 必填 | 默认 | 说明 |
|---|---|---|---|
| `--name` | ✅ | — | 任务名称 |
| `--group-name` | 否 | — | 目标群名，**可重复**传多个 |
| `--group-id` | 否 | — | 直接给群 ID（chatId），可重复，跳过名字查询 |
| `--content` | 二选一 | — | **统一话术**：所有群同一句 |
| `--contents-json` | 二选一 | — | **个性话术**：JSON 数组，每群不同 |
| `--send` | 否 | 不加 | **不加就是存草稿**，加了才真发 |
| `--end-time` | 否 | 今天 23:59:59 | 失效时间 `YYYY-MM-DD HH:MM:SS` |
| `--send-type` | 否 | `SENDNOW` | 发送类型 |
| `--delayed-send-time` | 否 | 空 | 定时发送时间 |
| `--description` | 否 | 空 | 任务描述 |
| `--script-type` | 否 | `PUBLIC` | 话术类型 |
| `--dry-run` | 否 | false | 只打印请求体，不调用创建 |

### 选群规则（重要）

| 用户怎么说 | 怎么做 |
|---|---|
| 点了某几个群名 | 传 `--group-name`（每个一次），`ifAllGroup=false` |
| 没点名，就说"发给所有群" | 不传任何群参数，走 `ifAllGroup=true`（命令会打印群总数提醒） |

> ⚠️ 全选会覆盖该项目**所有**客户群（可能有几百个）。命令会提示总数，
> 但**务必先跟用户确认**再执行，尤其是要真发的时候。

### 话术两种方式

**统一话术**：

```json
{"argv": ["openapi","--allow-risk","write","customer-group-send","create",
          "--name","寒露关怀","--content","寒露至，注意添衣保暖"]}
```

**个性话术**（每群不同内容）：

```json
{"argv": ["openapi","--allow-risk","write","customer-group-send","create",
          "--name","寒露关怀",
          "--contents-json","[{\"groupName\":\"A群\",\"content\":\"给A的话术\"},{\"groupName\":\"B群\",\"content\":\"给B的话术\"}]"]}
```

`--contents-json` 每项可用 `groupName` 或 `groupId`，二者其一 + `content`。

### 调用示例（指定两群 + 个性话术 + 真发）

```json
{"argv": ["openapi","--allow-risk","write","customer-group-send","create",
          "--name","1008群发测试",
          "--contents-json","[{\"groupName\":\"土秀才临沂蒙阴37群\",\"content\":\"寒露至，天渐凉，注意添衣保暖，愿您安康。\"},{\"groupName\":\"土秀才临沂蒙阴35群\",\"content\":\"寒露节气，昼夜温差大，记得添衣，保重身体。\"}]",
          "--end-time","2026-10-08 18:00:00",
          "--send"], "timeout_s": 120}
```

### 返回

```json
{"status":"success","activityId":"0000000426100815","activityIdSource":"lookup-by-name",
 "ifDraft":false,"ifAllGroup":false,"itemCount":2,"endTime":"2026-10-08 18:00:00",
 "verify":{"listed":{...}}}
```

`activityIdSource` 说明 id 从哪来：`create`（接口直接返回）/ `lookup-by-name`（接口没返回，按名回查）/
`unknown`。若显示 `success_unverified` 说明创建成功但没拿到 id，需要去后台确认。

---

## 四、`customer-group-send groups` —— 查客户群（只读）

```json
{"argv": ["openapi","customer-group-send","groups","--limit","20"], "timeout_s": 120}
```

可加 `--keyword` 按名字过滤。**只读，不需要 `--allow-risk`。**

当用户说的群名不确定、或报找不到群时，先用这个查准确群名。

---

## 五、标准作业流（写操作，别跳步）

1. **问清四件事**：任务名、发给哪些群、话术（统一还是个性）、是否现在发。
2. **群名不确定就先查** `customer-group-send groups`，拿到准确名字再用。
3. **先跑 `--dry-run`**，把请求体和目标群数给用户过目 —— 群发不可撤回，别盲发。
4. 用户确认后，**去掉 `--dry-run`，并按需加 `--send`**：
   - 只要先建好待发 → 不加 `--send`（默认草稿，安全）
   - 确认要立刻发出 → 加 `--send`
5. 回报 `activityId`，并说明是草稿还是已发送、覆盖了几个群。

> 用户说"立即发出"才加 `--send`。**没明说就默认草稿**，这是本技能的安全约定。

---

## 六、避坑清单

| 现象 | 原因 / 处理 |
|---|---|
| `Command risk is 'write'` | 漏了 `--allow-risk write`，或位置错 → 放在 `openapi` 之后 |
| `Input should be a valid string` | 用了 `options` 参数 → 改平铺进 `argv` |
| `GROUP_NOT_FOUND` | 群名不在当前项目里，或写错字 → 用 `groups` 查准确名字（注意"蒙阴"别写成"蒙明"） |
| 提示 `ALL GROUPS selected: N groups` | 没指定群 → 走全选。**务必跟用户确认**再继续 |
| `activityId: null` + `success_unverified` | create 接口不返回 id 是常态 → 已自动按名回查；仍为空就去后台确认 |
| `Connection reset by peer` | 上游网关偶发 → 命令内置重试；仍失败就再跑一次 |
| `notExecuteNumber` 大于 0 | 有群没执行成功（可能当天已发过、或非好友）→ 提示用户去后台看 |
| `Missing OpenAPI config` | 服务端 MCP 缺凭据 → 联系管理员，**不要在参数里塞密钥** |
| `No such command 'customer-group-send'` | 服务器还没部署该命令 → 管理员需部署并重启 |

---

## 七、附：为什么默认草稿

群发一旦发出不可撤回，而且全选时会覆盖几百个群。所以本技能把默认值设成
`ifDraft=true`，只有显式 `--send` 才真发 —— 宁可多一步确认，也不要发出去再道歉。
