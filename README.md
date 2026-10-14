# Day Mark — MCP for your personal work record

**Day Mark** is a to-do app for iPhone, iPad and Mac that remembers what you actually finished: a calendar heatmap shows what got done on any day, and every task shows its lead time from creation to completion.

The Mac app has a **built-in MCP (Model Context Protocol) endpoint**, so AI agents such as Claude Code can read your tasks, search your history, add tasks and write journal notes — with every agent write clearly labeled.

- App Store: https://apps.apple.com/us/app/day-mark-work-log/id6815580518
- Privacy: https://www.figedu.com/daymark/privacy
- This repository contains **documentation only**. Day Mark itself is a closed-source App Store app.

> **Availability:** the built-in MCP endpoint ships in **Day Mark 1.2 for Mac** (in review). Day Mark 1.1 offers the same actions through Shortcuts and Siri on iPhone, iPad and Mac.

---

## Connect (Mac)

1. Open **Day Mark → Settings → AI Agents**, turn on **Allow AI agents (MCP)**.
2. Click **Copy Claude Code setup command** and run it in Terminal. It looks like:

   ```bash
   claude mcp add --scope user --transport http daymark http://127.0.0.1:47731/mcp \
     --header "Authorization: Bearer <your-token>"
   ```

3. Optional: turn on **Open at login** so the endpoint is always available.

Any MCP client that supports the **Streamable HTTP** transport with a custom `Authorization` header can connect the same way.

| | |
|---|---|
| Endpoint | `http://127.0.0.1:47731/mcp` |
| Transport | MCP Streamable HTTP (stateless; JSON responses, no SSE) |
| Protocol versions | `2025-06-18`, `2025-03-26`, `2024-11-05` |
| Auth | `Authorization: Bearer <token>` (token stored in the macOS Keychain; resettable in Settings) |

## Tools

| Tool | What it does | Writes? |
|---|---|---|
| `daymark_list_lists` | List task lists | — |
| `daymark_list_tasks` | Open tasks, My Day, starred, or a list | — |
| `daymark_search` | Keyword search over tasks (incl. completed) and journal notes | — |
| `daymark_get_work_record` | What got done each day: completions with lead time and source, plus journal notes | — |
| `daymark_add_task` | Create a task (optionally planned for a day) | ✓ |
| `daymark_update_task` | Rename, note, move, plan, star | ✓ |
| `daymark_add_day_note` | Write a journal note for a day | ✓ |
| `daymark_complete_task` / `daymark_uncomplete_task` | Mark done / reopen (idempotent) | ✓ |
| `daymark_delete_task` / `daymark_delete_day_note` | Soft-delete — **only items an agent created** | ✓ |

Full contract: [`docs/protocol.md`](docs/protocol.md).

## Trust model

- **Every write carries its source** (`app`, `shortcut`, `agent`). Tasks an agent completes show *“By agent”* in the day's record; notes it writes show *“Written by agent”*.
- **Agents can only delete what they created.** Anything you wrote yourself returns `forbidden`.
- Completing is idempotent — a retrying agent never creates duplicate “done” records.
- Agents cannot set reminders or edit your journal entries.

## Security

- Listens on **127.0.0.1 only** — not reachable from other devices.
- **Off by default**; you turn it on.
- Requires the bearer token; requests from web pages with a foreign `Origin` are rejected.
- Your data never leaves your devices except through your own private iCloud. There is no Day Mark server.

---

## 中文简介

**Day Mark** 是一款 iPhone / iPad / Mac 通用的待办 App，它记住的是「你做完了什么」：月历热力图显示每一天完成了什么，每件事都显示从创建到完成的耗时。

Mac 版内置 **MCP 端点**（1.2 起），Claude Code 等 AI 助手可以读取任务、搜索历史、添加任务和写日记；**AI 写入的每一条都会被标出来，而且它只能删除自己建的东西。** 端点只在本机（127.0.0.1）可连、需要访问令牌、默认关闭。

接入：Day Mark → 设置 → AI Agent → 打开「允许 AI Agent 访问（MCP）」→ 复制 Claude Code 接入命令。

本仓库只有文档，Day Mark 本身是 App Store 上的闭源应用。
