<div align="center">

# 🐉 Discord Level Bot

**Text XP · Voice XP · Activity Roles · Daily Streaks · Dark & Red Rank Cards**

A self-hosted Discord levelling bot built on `discord.js` v14, SQLite and `@napi-rs/canvas`.
Text and voice activity are tracked separately, rendered as image cards, and rewarded with roles.
No dashboard, no subscription, no data leaving your server.

[![Version](https://img.shields.io/badge/version-1.5.0-E8381F)](CHANGELOG.md)
[![Node.js](https://img.shields.io/badge/node-%E2%89%A5%2020-3c873a?logo=node.js&logoColor=white)](https://nodejs.org)
[![discord.js](https://img.shields.io/badge/discord.js-v14-5865F2?logo=discord&logoColor=white)](https://discord.js.org)
[![SQLite](https://img.shields.io/badge/storage-SQLite-003B57?logo=sqlite&logoColor=white)](https://www.sqlite.org)
[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)

[Features](#features) · [Quick start](#quick-start) · [Commands](#commands) · [Configuration](#configuration) · [How it works](#how-it-works) · [Hosting](#hosting) · [Troubleshooting](#troubleshooting) · [Changelog](CHANGELOG.md)

</div>

<!--
Add screenshots to docs/images/ and uncomment:

<p align="center">
  <img src="docs/images/level-card.png"   width="49%" alt="Level card">
  <img src="docs/images/levelup-card.png" width="49%" alt="Level-up card">
</p>
-->

---

## Features

| | |
|---|---|
| 🎴 **Image cards** | Dark and red level card, level-up card and leaderboard rendered with `@napi-rs/canvas` — fully recolourable per server |
| 💬🎙️ **Text + voice XP** | Two independent XP tracks, levels, ranks and role ladders |
| 🏆 **Leaderboards** | Today / week / month / all-time, paged with buttons, with avatars and your own rank |
| 🎁 **Daily rewards** | `/daily` with a streak bonus (+10 % per day, up to +90 %) |
| 🎭 **Activity roles** | Auto-created or mapped to your own roles, stack or single mode, one-command resync |
| ⚡ **XP boost events** | Timed server-wide multipliers |
| 🛡️ **Anti-abuse** | 60 s cooldown, minimum length, duplicate-message filter, no voice XP when muted, alone, in AFK, or for bots |
| ⚙️ **Admin control** | `/config` for toggles, ignored channels, no-XP roles, XP rates, manual level overrides |
| 🚀 **Built for small hosts** | Batched SQLite writes, render limiter, avatar/background caches, skip-if-unchanged command deploys |
| 💾 **Self-contained** | One SQLite file, automatic migrations, owner-only `/backup` |

---

## Quick start

### Requirements

- **Node.js 20 or newer** (22 LTS and 24 both work)
- A Discord application with a bot user
- A host that can run a long-lived Node process

### 1. Create the bot

1. Open the [Developer Portal](https://discord.com/developers/applications) → **New Application**.
2. **Bot** tab → enable **Server Members Intent** and **Message Content Intent**, then **Reset Token** and copy it.
3. **OAuth2 → General** → copy the **Client ID**.

### 2. Invite it

**OAuth2 → URL Generator**, scopes `bot` and `applications.commands`, with these permissions:

```
View Channels · Send Messages · Embed Links · Attach Files
Read Message History · Manage Roles · Connect
```

> **Manage Roles** lets the bot hand out level roles. Drag the bot's role **above** your level roles in *Server Settings → Roles*.

### 3. Install and run

```bash
git clone https://github.com/yourusername/discord-level-bot.git
cd discord-level-bot
npm install
cp .env.example .env     # then fill it in
npm start
```

Slash commands are registered automatically on startup (and only re-registered when they change).

### 4. First-time server setup

```text
/set-channel #level-ups     # where level-up cards are posted
/setup-roles both           # create the default text + voice role ladders
/config view                # review everything
```

---

## Configuration

### Environment variables

| Variable | Required | Description |
|---|---|---|
| `DISCORD_TOKEN` | ✅ | Bot token from the Developer Portal |
| `CLIENT_ID` | ✅ | Application (client) ID |
| `GUILD_ID` | — | Register commands to one server (instant). Leave empty for global (up to 1 h) |
| `DB_PATH` | — | SQLite file location. Default `./data/bot.db` |

### Optional fonts

The cards work with system fonts, but look sharper with these files in `assets/fonts/`:

```
Anton-Regular.ttf          # big condensed headlines and level numbers
Montserrat-Bold.ttf        # small text
Montserrat-Regular.ttf
```

Both are free on Google Fonts. Restart the bot after adding them.

### Card colours

Defaults use the red theme. Override per server with `/set-card-color`:

| Element | Default | Controls |
|---|---|---|
| `accent` | `#E8381F` | Sun, slashes, border, avatar ring, role badge |
| `background` | `#0A0A0D` | Card background |
| `bar` | `#E8381F` | XP bars (voice bar is automatically a shade darker) |
| `text` | `#F4F4F5` | Names and labels |

---

## Commands

### Members

| Command | Description |
|---|---|
| `/level [user]` | Level card: text + voice XP, levels, ranks, role badge |
| `/leaderboard <text\|voice> [period]` | Paged rankings. Period: `day`, `week`, `month`, `lifetime` (UTC days) |
| `/daily` | Claim daily XP. Claim again within 48 h to keep the streak |
| `/rewards` | Level roles you can earn and how far away the next one is |
| `/stats` | Server totals and top users |
| `/help` | Command list (admins see the full list) |

### Admin — settings

| Command | Description |
|---|---|
| `/config view` | Every setting at a glance |
| `/config levelup-announcements <bool>` | Post level-up cards or not (roles are always given) |
| `/config voice-min-users <1-10>` | People needed in a voice channel before anyone earns XP (default 2) |
| `/config antispam <bool>` | No XP for repeating the same message |
| `/config daily-xp <0-5000>` | Base XP for `/daily` (`0` disables it) |
| `/set-channel [channel]` | Level-up channel (omit to use the triggering channel) |
| `/set-xp-rate …` | Text range, voice rate and permanent multiplier |
| `/set-card-color <element> <hex>` | Recolour the cards |
| `/ignore-channel <channel>` | Toggle XP in a channel (threads inherit it) |
| `/no-xp-role <role>` | Toggle a role's ability to earn text **and** voice XP |

### Admin — users and roles

| Command | Description |
|---|---|
| `/set-xp <user> <type> <level>` | Force a level (syncs roles) |
| `/reset-user <user> <text\|voice\|all>` | Wipe XP |
| `/setup-roles [type] [mode]` | Create the default role ladders (`stack` or `single`) |
| `/manage-roles set\|remove\|rename\|list` | Map your own roles to levels |
| `/sync-roles` | Rebuild everyone's roles from saved levels |
| `/xp-boost start\|stop\|status` | Timed multiplier events |

### Bot owner

| Command | Description |
|---|---|
| `/backup` | Sends a consistent copy of `bot.db` (ephemeral, owner only) |

---

## How it works

### XP rules

| Source | Rule |
|---|---|
| **Text** | 15–25 XP per message (configurable), 60 s cooldown, messages under 5 characters ignored, identical repeats ignored |
| **Voice** | 10 XP per minute while unmuted and undeafened, with at least `voice-min-users` humans in the channel. AFK channel, ignored channels, no-XP roles and bots earn nothing |
| **Daily** | `daily-xp × (1 + 0.1 × min(streak − 1, 9))`. Streak survives 48 h gaps |
| **Boost** | A running `/xp-boost` replaces the base multiplier until it expires |

### Level curve

XP needed to *reach* level `n` from `n − 1` is `floor(100 × n^1.5)`. Max level is 100.

| Level | XP for that level | Total XP |
|---:|---:|---:|
| 1 | 100 | 100 |
| 5 | 1,118 | 2,819 |
| 10 | 3,162 | 14,264 |
| 25 | 12,500 | 131,300 |
| 50 | 35,355 | 724,849 |
| 75 | 64,951 | 1,981,105 |
| 100 | 100,000 | 4,050,079 |

### Architecture

```text
src/
├── index.js                  entry: load commands/events, housekeeping, graceful shutdown
├── commands/                 one file per slash command
├── events/
│   ├── messageCreate.js      text XP, cooldown, anti-spam
│   ├── voiceStateUpdate.js   voice tracking + 60 s XP ticker
│   ├── ready.js              seeds the voice tracker, starts the ticker
│   └── interactionCreate.js  command router
├── database/
│   └── db.js                 SQLite, migrations, write-behind cache, XP math
└── utils/
    ├── cardRenderer.js       level / level-up / leaderboard cards (+ caches, limiter)
    ├── levelUp.js            shared flow: role → card → announcement
    ├── roleManager.js        create / assign / sync level roles
    ├── deployCommands.js     registers slash commands when they changed
    ├── theme.js              embed colour helper
    └── xpHelpers.js          validation + formatting
```

### Data and performance

- **Storage:** SQLite in WAL mode. Tables: `users`, `guilds`, `level_roles`, `xp_boosts`, `xp_history`.
- **Write-behind cache:** XP lives in memory and is flushed to disk in one transaction every 3 s and on shutdown. Stop the bot cleanly (SIGINT/SIGTERM) so nothing is lost.
- **History:** per-day XP totals power the day/week/month leaderboards and are pruned after ~5 weeks.
- **Rendering:** at most 2 cards are drawn at once; avatars (10 min LRU) and card backgrounds are cached.
- **Migrations:** run automatically on every start; existing databases upgrade in place.

---

## Hosting

<details>
<summary><b>Katabump / small panels</b></summary>

- Use Node 20, 22 or 24. `better-sqlite3` v12 ships prebuilt binaries for all three, so no compiler is needed.
- After changing `package.json`, delete `node_modules` and `package-lock.json`, then run `npm install`.
- CPU is capped, so the render limiter matters — don't raise it unless you have headroom.
- Stop the bot from the panel instead of killing the container, and download `data/bot.db` (or run `/backup`) regularly.

</details>

<details>
<summary><b>Railway / Render</b></summary>

Connect the repo, add the environment variables, and use `npm start`. On Render choose **Background Worker**. Attach a persistent volume and point `DB_PATH` at it, otherwise the database is wiped on redeploy.

</details>

<details>
<summary><b>VPS with pm2</b></summary>

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
git clone https://github.com/yourusername/discord-level-bot.git && cd discord-level-bot
npm install && cp .env.example .env && nano .env
npm install -g pm2
pm2 start src/index.js --name level-bot
pm2 save && pm2 startup
```

</details>

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| `Could not locate the bindings file — better_sqlite3.node` | Old `better-sqlite3` on Node 24. This project uses v12 — delete `node_modules` and `package-lock.json`, run `npm install`. |
| Slash commands missing | Global commands take up to an hour. Set `GUILD_ID` for instant updates while testing. |
| Bot ignores messages | Enable **Message Content Intent** in the Developer Portal. |
| No level-up cards | Check `/config view` (announcements on?) and that the bot can **Send Messages** and **Attach Files** in the channel. |
| Roles not assigned | Bot needs **Manage Roles** and its role must sit above the level roles. Run `/sync-roles` after changing milestones. |
| Voice XP not increasing | Needs 2+ people by default (`/config voice-min-users`), not muted/deafened, not in the AFK channel. |
| Cards look plain | Add `Anton-Regular.ttf` to `assets/fonts/` and restart. |
| `npm install` fails on old Linux | `sudo apt install -y libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev` |

---

## Contributing

Issues and pull requests are welcome.

1. Fork the repo and create a branch: `git checkout -b feat/my-change`
2. Keep commands in `src/commands/` and shared logic in `src/utils/`
3. Update [CHANGELOG.md](CHANGELOG.md) under **Unreleased**
4. Open a pull request describing what changed and why

## License

[MIT](LICENSE) — use it, fork it, just keep the license file.
