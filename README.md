<div align="center">

# SafeRelay

## Private Chat Relay Bot — Anti-Harassment · Zero Cost · Serverless

[![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-yellow?logo=javascript&logoColor=white)](https://developer.mozilla.org/docs/Web/JavaScript) [![Cloudflare Workers](https://img.shields.io/badge/Cloudflare%20Workers-100K%2Fday-orange?logo=cloudflare&logoColor=white)](https://workers.cloudflare.com/) [![Telegram Bot](https://img.shields.io/badge/Telegram%20Bot-API%208.0+-blue?logo=telegram&logoColor=white)](https://core.telegram.org/bots/api) [![License](https://img.shields.io/badge/License-GPL--3.0-green)](LICENSE)

**English** | [简体中文](#简体中文)

</div>

---

## ✨ Features

| Module | Description |
| ---- | ---- |
| **Message Relay** | Two-way guest ↔ admin forwarding, with edit sync and forum topic mode |
| **Human Verification** | Cloudflare Turnstile + local quiz, effective against bots |
| **Federation Ban** | Shared ban list: a user banned once is blocked everywhere |
| **Fraud Detection** | Local fraud database checked in real time |
| **Trusted List** | Trusted guests skip all checks and are forwarded directly |
| **Local Moderation** | Block / unblock / reset verification by replying to a message |
| **Automatic Messages** | Custom welcome message and auto-reply (10-minute cooldown) |
| **Graphical Panel** | Inline keyboard menus for one-tap configuration |
| **Forwarding Modes** | Switch between private chat and forum topics |
| **Broadcast** | Send a message to all verified users (HTML supported, 24-hour cooldown) |
| **Edit Sync** | Guest and admin message edits are synced in real time |
| **Statistics** | Daily message counts and active users |
| **🌍 Bilingual UI** | English by default, switchable to Simplified Chinese per user (see below) |
| **Zero Cost** | Runs long-term on the Cloudflare Workers free tier |

---

## 🚀 Quick Start

See the full deployment guide: **[DEPLOY.md](./DEPLOY.md)**.

Deployment, environment variables, webhook activation, Turnstile setup, and topic mode are all documented there.

---

## 🌍 Language / 语言切换

- **Default language: English.** A fresh deployment talks to everyone in English.
- **Switch per user:** anyone — guests and admins — can send `/language` (or `/lang`), or tap the **🌍 Language** button (guests: on the `/start` welcome message; admins: in the main menu), and pick **English** or **简体中文**.
- **Stored per user:** the choice is saved in KV under the key `lang:<userId>`, so every user keeps their own language. Admin messages use the admin's language; guest messages use the guest's language; a notification *about* a guest uses the receiving admin's language.
- **Deployment default:** set the environment variable `DEFAULT_LANG` (alias: `BOT_LANG`) to `zh` to make Simplified Chinese the default for users who have not chosen yet. Accepted values: `en` (default) / `zh`.
- **No automatic switching:** the bot deliberately does **not** follow Telegram's `language_code`. The language stays English until the user switches with `/language` or `DEFAULT_LANG` is set.

---

## 🎨 Emoji Legend

SafeRelay uses one consistent emoji set across all menus and messages:

| Emoji | Meaning | Emoji | Meaning |
| ---- | ---- | ---- | ---- |
| 🛡️ | Verification / security | 📊 | Statistics |
| 🧹 | Spam filter | 🌍 | Language |
| 🗑️ | Spam topic | 🟢 / 🔴 | On / off |
| 🤖 | AI detection | ✅ | Success |
| 🌐 | Federation ban | ❌ | Error / cancel |
| 👥 | User management | ⚠️ | Warning |
| 👋 | Welcome message | ◀️ | Back |
| 💬 | Forwarding | 🔄 | Refresh |

---

## 🤖 Admin Commands

For most commands, simply **reply** to a forwarded guest message — the bot extracts the target user ID automatically.

### Basics

| Command | Description | Example |
|:----:|:----:|:----:|
| *(reply)* | Reply directly to the guest | (just type and send) |
| `/help` | Show help | `/help` |
| `/menu` | Open the graphical admin panel | `/menu` |
| `/language` | Switch language (English / 简体中文) | `/language` |

### User Management

| Command | Description | Example |
|:----:|:----:|:----:|
| `/ban` | Block a user | Reply with `/ban`, or `/ban 123456` |
| `/unban` | Unblock a user | Reply with `/unban`, or `/unban 123456` |
| `/reset` | Reset a user's verification | Reply with `/reset`, or `/reset 123456` |
| `/trust` | Trust a user (skip verification) | Reply with `/trust`, or `/trust 123456` |
| `/untrust` | Remove trust | Reply with `/untrust`, or `/untrust 123456` |

### Message Settings

| Command | Description | Example |
|:----:|:----:|:----:|
| `/welcome` | Set the welcome message | `/welcome Hello! Please verify first.` |
| `/autoreply` | Set the auto-reply | `/autoreply Thanks, we received your message.` |
| `/broadcast` | Broadcast to all verified users | `/broadcast Maintenance notice` |
| `/bcancel` | Cancel an ongoing broadcast | `/bcancel` |

### System

| Command | Description | Example |
|:----:|:----:|:----:|
| `/cleanup` | Clean up invalid topic mappings | `/cleanup` or `/cleanup --dry-run` |
| `/cachestats` | View cache statistics | `/cachestats` |
| `/clearcache` | Clear all caches | `/clearcache` |
| `/restore` | Restore a message from the spam topic | Reply to a spam-topic message with `/restore` |

Guests can use `/start` and `/language` / `/lang`.

---

## 💬 Topic Mode (Forum)

Want every guest conversation organized in a Telegram forum topic?

1. Set the `GROUP_ID` environment variable to your forum group.
2. Make sure the bot is an admin in that group with the **Manage Topics** permission.
3. Send `/menu` → **💬 Forwarding** → **💬 Forum Topics**.

The bot then creates a topic for each new guest and forwards their messages there. Reply inside the topic to answer the guest.

---

## 🧠 How It Works

```text
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│    Guest    │ --> │     Bot     │ --> │    Admin    │
│  (verify)   │     │ (check/fwd) │     │  (reply)    │
└─────────────┘     └─────────────┘     └─────────────┘
       ^                                      │
       └──────────────────────────────────────┘
                  (replies are relayed back)
```

1. A new guest sends a message → human verification starts
2. Verification passed → the message is forwarded to the admin
3. The admin replies → the reply is relayed back to the guest
4. Verified guests → messages are forwarded directly

---

## 🛡️ Security

- **Webhook secret validation** — checks the `X-Telegram-Bot-Api-Secret-Token` header to reject forged requests
- **Federation ban** — shared ban system blocks known malicious users
- **Fraud detection** — local fraud database, checked in real time
- **Anti-flood protection** — at most 5 messages within 5 seconds
- **Human verification** — Cloudflare Turnstile keeps bots out
- **Log redaction** — tokens, secrets, and signatures are filtered from logs automatically

---

## ⚠️ Notes

1. **Verification delay**: Cloudflare KV is eventually consistent; a passed verification may take up to ~30 seconds to propagate to all edge locations.
2. **Trusted list priority**: trusted users skip all checks (verification, block list, fraud detection).
3. **Mapping expiry**: forwarding mappings are kept for 48 hours; older messages can no longer be replied to.
4. **Broadcast cooldown**: broadcasts have a 24-hour cooldown and send to at most 500 users per batch.
5. **Edit limit**: Telegram only allows editing messages within 48 hours.

---

## 🎯 Use Cases

- Support / customer-service bots
- Anonymous submission bots
- Private chat relay bots
- Feedback collection bots
- Community reception bots

---

## 🛠 Tech Stack

- JavaScript (ES6+)
- Telegram Bot API
- Cloudflare Workers + KV + Cache API
- Cloudflare Turnstile

---

## 📂 Project Structure

```
SafeRelay/
├── worker.js          # Main program
├── DEPLOY.md          # Deployment guide
├── README.md          # This document
├── LICENSE            # GPL-3.0 license
└── data/
    └── fraud.db       # Fraud user database
```

### Fraud Database

`data/fraud.db` contains known fraudulent user IDs, one per line. The bot checks it automatically and blocks matches.

**Custom fraud database**:
1. Edit `data/fraud.db`
2. Add one user ID per line
3. After pushing to GitHub it takes effect within ~1 hour (or immediately after a Worker restart)

---

## 简体中文

### 功能特性

| 模块 | 说明 |
| ---- | ---- |
| **消息中转** | 访客 ↔ 管理员双向转发，支持编辑同步与话题模式 |
| **人机验证** | Cloudflare Turnstile + 本地题库双重验证，有效拦截机器人 |
| **联合封禁** | 接入中心化封禁系统，恶意用户一次封禁全网拦截 |
| **欺诈检测** | 本地欺诈数据库实时比对，自动识别可疑用户 |
| **信任名单** | 信任用户跳过所有检查，直接转发 |
| **本地管理** | 拉黑 / 解封 / 重置验证，回复消息即可操作 |
| **自动消息** | 自定义欢迎消息与自动回复（10 分钟冷却） |
| **图形面板** | Inline Keyboard 菜单，配置一键完成 |
| **转发模式** | 私聊 ↔ 话题群组一键切换 |
| **广播推送** | 向所有已验证用户群发消息，支持 HTML，24 小时冷却 |
| **双语界面** | 默认英文，可按用户切换为简体中文（见上方 Language 章节） |
| **零成本运行** | Cloudflare Workers 免费额度长期稳定运行 |

### 快速开始

请直接查看详细部署教程：**[DEPLOY.md](./DEPLOY.md)**。

### 语言说明

- **默认语言为英文。** 发送 `/language`（或 `/lang`），或点击 🌍 语言按钮，即可切换为简体中文；选择按用户保存在 KV 的 `lang:<userId>` 中。
- 部署默认语言可用环境变量 `DEFAULT_LANG`（别名 `BOT_LANG`）设置为 `zh`。
- 机器人**不会**根据 Telegram 的 `language_code` 自动切换语言；未切换前始终使用英文（或 `DEFAULT_LANG` 指定的默认语言）。

### 管理员指令

基础：`/help` 帮助、`/menu` 管理面板、`/language` 切换语言。
用户管理：`/ban` 封禁、`/unban` 解封、`/reset` 重置验证、`/trust` 信任、`/untrust` 取消信任（回复访客消息使用，或附带用户 ID）。
消息设置：`/welcome` 欢迎消息、`/autoreply` 自动回复、`/broadcast` 广播、`/bcancel` 取消广播。
系统管理：`/cleanup` 清理失效话题、`/cachestats` 缓存统计、`/clearcache` 清空缓存、`/restore` 从垃圾话题恢复消息。

---

## 🙏 Acknowledgements

This project builds on the following open-source projects:

| Project | Author | License | Main Contribution |
|:----:|:----:|:------:|:---------|
| [NFD](https://github.com/LloydAsp/nfd) | LloydAsp | GPL-3.0 | Core architecture, message relay |
| [NFD 3.0](https://www.nodeseek.com/post-545453-1) | NodeSeek | GPL-3.0 | Turnstile verification |
| [RelayGo](https://github.com/abcxyz-123456/RelayGo) | abcxyz-123456 | GPL-3.0 | Federation ban, admin panel |
| [telegram-verify-bot](https://github.com/Squarelan/telegram-verify-bot) | Squarelan | GPL-3.0 | Trusted list, fraud detection |
| [telegram_private_chatbot](https://github.com/jikssha/telegram_private_chatbot) | jikssha | MIT | Local quiz, verification, security practices, deployment flow |

Thanks to all of the authors! ❤️

---

## 📜 License

This project is based on [NFD](https://github.com/LloydAsp/nfd) and is released under the [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.html) (GPL-3.0).

As a derivative work of GPL-3.0 software, this project follows the same license terms. You may freely use, modify, and distribute it, provided you comply with GPL-3.0, including:
- Keeping copyright notices
- Releasing derivative works under the same license (GPL-3.0)
- Providing the source code

See the [LICENSE](./LICENSE) file for details.

---

## ⭐ Star History

<a href="https://www.star-history.com/?repos=qianqi32%2FSafeRelay&type=date&legend=top-left">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/chart?repos=qianqi32/SafeRelay&type=date&theme=dark&legend=top-left" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/chart?repos=qianqi32/SafeRelay&type=date&legend=top-left" />
    <img alt="Star History Chart" src="https://api.star-history.com/chart?repos=qianqi32/SafeRelay&type=date&theme=dark&legend=top-left" />
  </picture>
</a>

---

<div align="center">

**If this project helps you, please give it a ⭐ Star!**

**Made with ❤️ by [qianqi32](https://github.com/qianqi32/)**

</div>
