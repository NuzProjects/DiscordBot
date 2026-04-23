# NuzFlame V2

A feature-rich Discord bot built with [discord.py 2.7](https://discordpy.readthedocs.io/) featuring a FastAPI web dashboard, appeal system, ticket management, moderation tools, leveling, AI chat, and more.

---

## Requirements

- Python **3.11.7** (see `runtime.txt`)
- A Discord bot token — [Discord Developer Portal](https://discord.com/developers/applications)
- A Discord application with OAuth2 configured (for the web dashboard login)

---

## Installation

```bash
# 1. Clone the repository
git clone <your-repo-url>
cd bot-main

# 2. Install dependencies
pip install -r requirements.txt

# 3. Configure the bot (see Configuration section below)

# 4. Run
python main.py
```

---

## Configuration

All configuration lives in **`config.yaml`**. No `.env` file is needed — but environment variables (e.g. from a Pterodactyl panel) override config values where supported.

### Bot

```yaml
bot:
  token: "YOUR_BOT_TOKEN"       # Discord bot token
  status: "online"               # online | idle | dnd | invisible
  activity: "your status text"
  activity_type: "playing"       # playing | watching | listening | streaming
```

### Logging

```yaml
logging:
  level: "INFO"                  # DEBUG | INFO | WARNING | ERROR | CRITICAL
  file: "logs/bot.log"           # Rotates at 10 MB, keeps 5 backups
```

Logs are written to `logs/bot.log`. The folder is created automatically if missing.

### Web Dashboard

```yaml
web:
  enabled: true
  bind_host: "0.0.0.0"
  host: "your.domain.com"
  port: 10215
  public_url: "http://your.domain.com:10215"
  appeal_public_url: "http://your.domain.com:10215"   # Used in appeal DM links
  discord_client_id: "YOUR_APP_CLIENT_ID"
  discord_client_secret: "YOUR_APP_CLIENT_SECRET"
  session_secret: "RANDOM_32_CHAR_KEY"                # Regenerate for production
  allowed_user_id: "USER_ID_1,USER_ID_2"              # Comma-separated admin user IDs
```

> **Pterodactyl users:** The panel's allocated port is automatically picked up from `SERVER_PORT`, `PORT`, `WEB_PORT`, `PTERO_SERVER_PORT`, `ALLOCATED_PORT`, or `ALLOC_PORT` environment variables. This overrides the `port` value in config.

To get your Discord OAuth2 credentials: go to the [Developer Portal](https://discord.com/developers/applications) → your app → **OAuth2**. Set the redirect URI to `http://your.domain.com:PORT/auth/callback`.

### Channels

```yaml
channels:
  moderation_log: 0   # Channel for ban/kick/warn/mute logs
  appeal_log: 0       # Channel where appeal submissions appear (with Accept/Deny buttons)
  ticket_log: 0       # Channel for ticket open/close events and transcripts
```

### Ticket System

```yaml
tickets:
  panel_channel: 0                    # Channel to post the ticket creation panel (0 = disabled)
  category_name: "Tickets"            # Discord category for ticket channels
  channel_prefix: "🎫︱ticket-"
  autoclose_prefix: "⌛︱"
  autoclose_hours: 24
  transcript_history_limit: 500
  panel_title: "Create a Ticket"
  panel_description: "..."
  welcome_text: "..."
```

### AI (Groq)

```yaml
ai:
  api_key: "YOUR_GROQ_API_KEY"   # https://console.groq.com/
  channel_id: 0                  # Channel where the AI listens and auto-responds
  models:
    - "meta-llama/llama-4-scout-17b-16e-instruct"
    - "llama-3.3-70b-versatile"
    - ...
```

### Custom Emojis

All UI emojis are configurable. Replace any value with your own server's emoji ID, or set to `""` to fall back to a plain-text Unicode character.

```yaml
emojis:
  success:  "<:success:EMOJI_ID>"
  fail:     "<:fail:EMOJI_ID>"
  error:    "<:error:EMOJI_ID>"
  add:      "<:add:EMOJI_ID>"
  remove:   "<:remove:EMOJI_ID>"
  # ... (see config.yaml for full list)
```

---

## Data Storage

The bot uses flat JSON files — no database required. Files are created automatically on first use.

| File | Contents |
|------|----------|
| `data/tickets.json` | Open tickets, blacklist |
| `data/appeals.json` | Appeal submissions |
| `data/emoji.json` | Emoji/sticker usage cache |

Deleting any of these files is safe — they regenerate on next use. The `data/` folder itself must exist (or will be created automatically).

---

## Features & Commands

### 🎫 Tickets
Full support ticket system with transcripts, auto-close, and blacklist management.

| Command | Description | Permission |
|---------|-------------|------------|
| `/close [reason]` | Close a ticket immediately | Admin |
| `/autoclose <reason>` | Schedule a ticket to auto-close in 24h | Admin |
| `/add <user>` | Add a user to the current ticket | Admin |
| `/remove <user>` | Remove a user from the current ticket | Admin |
| `/ticket-blacklist <add\|remove> <user>` | Block or unblock a user from creating tickets | Admin |

**Buttons (available to ticket owner + admins):**
- **Close** — immediately closes and archives the ticket
- **Instant Close** — closes without a confirmation step
- **Deny Auto Close** — cancels a scheduled auto-close

Closing a ticket sends an HTML transcript to the ticket owner via DM and logs it in `channels.ticket_log`.

---

### 🛡️ Moderation

| Command | Description | Permission |
|---------|-------------|------------|
| `/warn <user> <reason>` | Issue a warning | Admin |
| `/warnlist <user>` | View a user's warnings | Admin |
| `/clearwarns <user>` | Clear all warnings for a user | Admin |
| `/kick <user> [reason]` | Kick a member | Admin |
| `/ban <user> [reason]` | Permanently ban a member | Admin |
| `/unban <user_id>` | Unban by Discord ID | Admin |
| `/mute <user> <duration> [reason]` | Timeout a member | Admin |
| `/unmute <user>` | Remove a timeout | Admin |
| `/purge <amount>` | Delete 1–100 messages | Admin |
| `/slowmode <seconds\|off>` | Set channel slowmode | Admin |
| `/lock [reason]` | Prevent members from sending messages | Admin |
| `/report <user> <reason>` | Submit a report to staff | Everyone |

All actions are logged to `channels.moderation_log` and users receive a DM.

---

### 🌐 Appeal System

Members who have been actioned can submit an appeal via a personal web link (sent by the bot via DM). Appeals are reviewed through the web dashboard.

The appeal page (`/appeal/<id>`) requires Discord OAuth2 login and validates that the correct account is submitting. Submissions appear in `channels.appeal_log` with **Accept** and **Deny** buttons for staff.

---

### 🤖 AI Chat

Powered by [Groq](https://console.groq.com/).

| Command | Description | Permission |
|---------|-------------|------------|
| `/ask <content> [image]` | Ask the AI anything | Everyone |
| `/feed <message>` | Add information to the AI knowledge base | Admin |
| `/blacklist <add\|remove> <user>` | Block/unblock a user from using AI | Admin |

The bot also auto-responds in the channel set by `ai.channel_id`.

---

### 📊 Levels & XP

Members earn XP by sending messages. Renders as a visual card using Pillow.

| Command | Description | Permission |
|---------|-------------|------------|
| `/level [user]` | View a level card | Everyone |
| `/lblevel` | Server XP leaderboard | Everyone |
| `/setxp <user> <amount>` | Manually set XP | Admin |
| `/resetxp <user>` | Reset a user's XP | Admin |

---

### 🎉 Giveaways

| Command | Description | Permission |
|---------|-------------|------------|
| `/gstart` | Start a giveaway | Admin |
| `/greroll <message_id>` | Reroll a winner | Admin |
| `/gend <message_id>` | Manually end a giveaway | Admin |

---

### 📨 Invites

| Command | Description |
|---------|-------------|
| `/invites [user]` | View invite stats for a member |
| `/invitelist [user]` | See who a member has invited |
| `/invitecodes [user]` | View all active invite codes |
| `/lbinvites` | Server invite leaderboard |

---

### 😄 Emoji & Stickers

| Command | Description | Permission |
|---------|-------------|------------|
| `/emojiinfo <emojis>` | Get info on up to 3 custom emojis | Everyone |
| `/copyemoji <emojis> [name]` | Copy up to 3 emojis into this server | Admin |
| `/stickerinfo <name\|url>` | Get info on a sticker | Everyone |
| `/stealsticker <message_id\|url> [name]` | Copy a sticker into this server | Admin |

---

### 📌 Sticky Messages

| Command | Description | Permission |
|---------|-------------|------------|
| `/sticky set <message>` | Pin a sticky message to a channel | Admin |
| `/sticky remove <id>` | Remove a sticky by ID | Admin |
| `/sticky list` | List all active stickies | Admin |

---

### 🔢 Counting Game

| Command | Description | Permission |
|---------|-------------|------------|
| `/csetup <channel>` | Configure the counting channel | Admin |
| `/cimport <file>` | Import data from a Countr JSON export | Admin |
| `/lbcounts` | Counting leaderboard | Everyone |

---

### 🛠️ Utility

| Command | Description | Permission |
|---------|-------------|------------|
| `/serverinfo` | Server statistics and info | Everyone |
| `/userinfo [user]` | Detailed member info | Everyone |
| `/avatar [user]` | Display a member's avatar | Everyone |
| `/banner [user]` | Display a member's banner | Everyone |
| `/ping` | Bot latency | Everyone |
| `/nick <user> [nickname]` | Change or reset a nickname | Admin |
| `/role add <user> <role>` | Assign a role | Admin |
| `/role remove <user> <role>` | Remove a role | Admin |
| `/role info <role>` | Role details | Admin |
| `/massrole type <role>` | Add/remove role from all members | Admin |

---

### 💬 Sender

Pre-configured messages stored as JSON files in the `data/messages/` folder.

| Command | Description | Permission |
|---------|-------------|------------|
| `/send <message_name>` | Send a pre-configured message | Admin |

---

### 🌍 Translator

Right-click any message → Apps → **Translate** to translate it to English using Google Cloud Translate.

Requires a Google Cloud service account with the Cloud Translation API enabled.

---

### 🟢 AFK

| Command | Description |
|---------|-------------|
| `/afk [reason]` | Set AFK status — auto-removed when you next chat |
| `/unafk` | Manually remove AFK status |

---

### 🎁 Server Backup

| Command | Description | Permission |
|---------|-------------|------------|
| `/backup <name>` | Create a full server backup (roles, channels, settings) | Admin |
| `/import-backup <name>` | Restore a backup | Admin |
| `/del-backup <name>` | Delete a saved backup | Admin |

---

### 💎 Booster Perks

Automatic role/perk assignment for server boosters. Configured per-server.

---

### 👋 Welcomer

| Command | Description | Permission |
|---------|-------------|------------|
| `/setwelcome <json_file>` | Upload a welcome embed config | Admin |
| `/setwelcomedm <json_file>` | Upload a DM welcome config | Admin |

**Example welcome JSON:**
```json
{
  "channel_id": 123456789012345678,
  "title": "Welcome {display_name}!",
  "description": "You are member #{member_count}.",
  "thumbnail_url": "{avatar}",
  "color": [88, 101, 242]
}
```

Available placeholders: `{username}`, `{display_name}`, `{mention}`, `{avatar}`, `{member_count}`, `{guild}`, `{guild_icon}`.

---

### 🤖 Bot Management

| Command | Description | Permission |
|---------|-------------|------------|
| `/botinfo` | Bot stats and version info | Admin |
| `/uptime` | How long the bot has been online | Admin |
| `/status` | CPU, memory, and latency | Admin |
| `/logs [lines]` | View the latest log entries | Admin |
| `/sync` | Sync slash commands | Admin |
| `/reload <cog>` | Reload a cog without restarting | Admin |
| `/load <cog>` | Load an unloaded cog | Admin |
| `/unload <cog>` | Unload a cog | Admin |
| `/restart` | Safely restart the bot | Admin |

---

## Web Dashboard

Accessible at `http://your.domain.com:PORT` after starting the bot.

- **Login** via Discord OAuth2 — only user IDs listed in `web.allowed_user_id` can access the dashboard.
- **Appeal review** — staff see submissions with Accept/Deny actions.
- **Status page** — live CPU, memory, shard, and uptime info.
- **Commands reference** — browsable command list.

The dashboard runs on FastAPI and is served by the bot process itself — no separate web server needed.

---

## Project Structure

```
bot-main/
├── main.py              # Entry point, bot class, cog loader, web server runner
├── config.yaml          # All configuration
├── requirements.txt
├── runtime.txt          # Python version (3.11.7)
├── cogs/                # Feature cogs (one per system)
│   ├── Ticket.py
│   ├── Moderation.py
│   ├── Level.py
│   ├── Giveaway.py
│   ├── Emoji.py
│   └── ...
├── utils/               # Shared utilities
│   ├── appeals.py       # Appeal load/submit logic
│   ├── emojis.py        # Custom emoji accessor (reads from config.yaml)
│   ├── logger.py        # Rotating file logger
│   ├── console_ui.py    # Rich terminal UI, startup banner
│   └── components_v2.py # discord.py Components V2 helpers
├── web/                 # FastAPI web dashboard
│   ├── api.py           # Routes, OAuth2, appeal API
│   └── site/            # Frontend (HTML/JS/CSS)
│       ├── appeal.html / appeal.js
│       ├── dashboard.js
│       └── styles.css
├── data/                # Auto-generated JSON storage
│   ├── tickets.json
│   └── appeals.json
└── logs/                # Rotating log files (auto-created)
    └── bot.log
```

---

## Deployment (Pterodactyl)

1. Set the **startup command** to `python main.py`.
2. Set the **allocated port** — the bot reads it automatically from environment variables (`SERVER_PORT`, `PORT`, etc.).
3. Set `web.allowed_user_id` in `config.yaml` to your Discord user ID.
4. Add your OAuth2 redirect URI (`http://host:PORT/auth/callback`) in the Discord Developer Portal.

The healthcheck endpoint is available at `GET /health`.
