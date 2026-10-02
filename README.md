<div align="center">

# Discord Level Bot

**Text XP · Voice XP · Activity Roles · XP Boost Events · Custom Cards**

A self-hosted Discord levelling bot built with Node.js. Tracks both chat and voice activity separately, generates rank cards, hands out roles as people level up, and lets admins run timed XP events. No dashboard, no subscription, no data leaving your server.

[![Node.js](https://img.shields.io/badge/Node.js-18%2B-brightgreen?logo=node.js)](https://nodejs.org)
[![discord.js](https://img.shields.io/badge/discord.js-v14-5865F2?logo=discord)](https://discord.js.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

I built this because every level bot I tried either ignored voice completely, locked activity roles behind a paid tier, or stored everything on someone else's servers. This one is self-hosted, free, and tracks text and voice XP independently — same idea as Arcane but you own it.

---

## What's included

- Rank cards rendered as images — avatar, two XP bars (text and voice), current levels, server ranks, and your highest activity role shown as a badge
- Leaderboards for text and voice, also rendered as image cards, with medals for the top 3
- Activity roles that get awarded automatically when someone hits a level milestone — works for both text and voice XP
- Default role sets the bot creates for you with one command, or you can map any existing role to any level
- Stack mode (members keep every role they've earned) or single mode (only the highest role at any time)
- XP boost events — start a timed multiplier for the whole server, announce it automatically, cancel it early if needed
- Level-up announcements with the new rank card sent to whatever channel you pick
- Anti-spam: 60-second cooldown per person, minimum message length, muted/deafened users don't earn voice XP
- Full card colour customisation — accent, background, XP bars, and text, all configurable per server with a hex code
- Admin commands that are actually hidden from regular members, not just permission-gated
- Slash commands register themselves on startup, so hosting services that don't run a separate deploy step work fine

---

## Setup

### 1. Create a bot account

Go to [discord.com/developers/applications](https://discord.com/developers/applications) and click **New Application**.

Go to the **Bot** tab and do three things:

- Click **Add Bot**
- Under **Privileged Gateway Intents**, enable both **Server Members Intent** and **Message Content Intent** — the bot won't work without these
- Click **Reset Token**, copy the token, and save it somewhere

Then go to **OAuth2 → General** and copy your **Client ID** from the top of the page.

### 2. Invite it to your server

Go to **OAuth2 → URL Generator** and select these scopes:

```
bot
applications.commands
```

And these permissions:

```
View Channels
Send Messages
Embed Links
Attach Files
Read Message History
Manage Roles     ← needed to assign activity roles
Connect          ← needed to see who's in voice
```

The Manage Roles permission is new — the bot needs it to assign and remove activity roles when people level up. Open the generated URL in your browser to invite it.

### 3. Install

```bash
git clone https://github.com/yourusername/discord-level-bot.git
cd discord-level-bot
npm install
```

Node.js 18 or higher is required. Check yours with `node --version`.

### 4. Configure

```bash
cp .env.example .env
```

Open `.env` and fill it in:

```env
DISCORD_TOKEN=your_bot_token_here
CLIENT_ID=your_client_id_here

# Set this to your server's ID while testing — commands update instantly
# Remove it when you go live (global commands take up to an hour to propagate)
GUILD_ID=

DB_PATH=./data/bot.db
```

To find your server ID: turn on Developer Mode in Discord settings (under Advanced), then right-click your server icon and hit "Copy Server ID".

### 5. Start it

```bash
npm start
```

The bot registers all slash commands automatically on startup, then logs in. It should come online in a few seconds.

---

## First-time server setup

Once the bot is running, there are two things worth doing right away:

**Set a level-up channel** so announcements go somewhere specific instead of the same channel as the triggering message:
```
/set-channel #level-ups
```

**Set up activity roles** — this creates all the default level roles automatically:
```
/setup-roles both
```

That's all you need. The bot will start handing out roles as people hit milestones. You can customise everything later.

---

## Commands

### For everyone

| Command | What it does |
|---|---|
| `/level` | Your rank card with Text XP, Voice XP, levels, ranks, and your current role badge |
| `/level @someone` | Check another member's card |
| `/leaderboard text` | Top 10 by chat XP |
| `/leaderboard voice` | Top 10 by voice XP |
| `/stats` | Server totals — members tracked, combined XP, top users |
| `/help` | All commands. Admins see the full list; regular members see just the user commands |

### XP and configuration (admins only)

| Command | What it does |
|---|---|
| `/set-xp @user text 25` | Force a user to level 25 text XP — also syncs their roles |
| `/set-xp @user voice 10` | Same for voice |
| `/set-channel #channel` | Where level-up cards get posted |
| `/set-channel` | No argument — turns off level-up announcements |
| `/set-card-color accent #FF6B6B` | Change the card accent colour |
| `/set-card-color background #1A1A2E` | Change the card background |
| `/set-card-color bar #FF6B6B` | Change the XP bar colour |
| `/set-card-color text #FFFFFF` | Change text colour |
| `/set-xp-rate` | View current XP settings |
| `/set-xp-rate multiplier 2.0` | Permanently double XP (use `/xp-boost` for timed events) |
| `/set-xp-rate text-min 10 text-max 30` | Change per-message XP range |
| `/set-xp-rate voice-xp 15` | Change voice XP per minute |
| `/ignore-channel #bot-spam` | Toggle XP off in a channel (run again to re-enable) |
| `/no-xp-role @role` | Stop a role from earning XP (run again to re-enable) |
| `/reset-user @user text` | Wipe someone's text XP |
| `/reset-user @user voice` | Wipe someone's voice XP |
| `/reset-user @user all` | Full reset |

### Activity roles (admins only)

| Command | What it does |
|---|---|
| `/setup-roles both` | Creates all default text and voice level roles automatically |
| `/setup-roles text stack` | Text roles only, stacking mode (members keep all earned roles) |
| `/setup-roles voice single` | Voice roles only, single mode (only highest role kept) |
| `/manage-roles set 20 @role` | Map any existing role to level 20 (text by default) |
| `/manage-roles set 20 @role voice` | Same but for voice XP |
| `/manage-roles rename 20 "🔥 Veteran"` | Rename the role at level 20 (renames the actual Discord role too) |
| `/manage-roles remove 20` | Remove the level 20 role milestone |
| `/manage-roles list` | See every configured milestone for text and voice |
| `/sync-roles` | Rebuilds everyone's roles based on their current saved level — run this after changing milestones |

### XP boost events (admins only)

| Command | What it does |
|---|---|
| `/xp-boost start 2h` | 2× XP for 2 hours, announces in the level-up channel |
| `/xp-boost start 1h 3.0 "Weekend Bonus"` | Custom multiplier and event name |
| `/xp-boost stop` | Cancel early |
| `/xp-boost status` | See the active boost and when it expires |

---

## Activity roles

The default role set (created by `/setup-roles`) looks like this:

| Level | Text role | Voice role |
|---|---|---|
| 5 | 📝 Newcomer | 🎧 Listener |
| 10 | 💬 Chatter | 🎙️ Talker |
| 20 | 🗣️ Regular | 📻 Regular |
| 35 | ⭐ Veteran | 🔊 Veteran |
| 50 | 🔥 Elite | 🎵 Elite |
| 75 | 💎 Legend | 👑 Voice King |

You can change any of these names with `/manage-roles rename`, or point any level to a role you already have with `/manage-roles set`. The bot won't touch roles it doesn't know about.

**Stack vs single mode** — in stack mode, someone who hits level 35 keeps Newcomer, Chatter, Regular, and Veteran all at once. In single mode, they only have Veteran. You set this when running `/setup-roles`, or change it any time and run `/sync-roles` to rebuild everyone's roles from scratch.

**The bot's role needs to be above the level roles** in your server's role list for Discord to let it assign them. Drag the bot's role above the highest level role in Server Settings → Roles.

---

## Customising the card

Run `/set-card-color`, pick an element from the dropdown, and enter a hex code. Both `#FF5733` and `FF5733` work.

| Element | Default | Controls |
|---|---|---|
| Accent | `#5865F2` | Left edge stripe, avatar glow ring, role badge colour |
| Background | `#23272A` | Card background |
| Bar | `#5865F2` | Text XP bar |
| Text | `#FFFFFF` | Username, level labels, XP numbers |

The voice XP bar automatically shifts hue from the bar colour — you don't need to set it separately.

Some combinations worth trying:

```
Cyberpunk:   accent #00FFFF  background #0D0D1A  bar #FF00FF
Sunset:      accent #FF6B6B  background #1A1A2E  bar #FF8E53
Forest:      accent #57CC99  background #1B2226  bar #38A3A5
Gold:        accent #FFD700  background #1C1C1C  bar #FFA500
```

---

## How XP works

**Text XP** — each message earns between 15 and 25 XP by default (random, so it doesn't feel mechanical). There's a 60-second cooldown between grants, and messages under 5 characters don't count.

**Voice XP** — 10 XP every 60 seconds while you're in a voice channel and not muted or deafened. If the bot restarts while people are in voice, it picks them up immediately when it comes back.

**XP boosts** — `/xp-boost` temporarily overrides the multiplier for the whole server. The base multiplier from `/set-xp-rate` is still there underneath; when the boost expires, it goes back to that.

**Level curve** — `floor(100 × (level + 1)^1.5)` per level. Early levels go fast, higher levels take real time:

| Level | XP to reach this level | Total XP from 0 |
|---|---|---|
| 1 | 100 | 100 |
| 5 | 245 | 886 |
| 10 | 332 | 2,181 |
| 25 | 510 | 7,747 |
| 50 | 722 | 21,238 |
| 75 | 884 | 38,943 |
| 100 | 1,020 | 59,814 |

---

## Hosting

### Railway

Push your code to GitHub, create a new Railway project from that repo, add your environment variables under Variables, and deploy. Railway picks up `npm start` automatically. Commands register on boot, nothing extra needed.

### Render

Same idea — connect the repo, add env vars, set the start command to `npm start`, and use **Background Worker** as the service type.

### VPS

```bash
# Get Node.js 18
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Set up the bot
git clone https://github.com/yourusername/discord-level-bot.git
cd discord-level-bot
npm install
cp .env.example .env
nano .env

# Keep it running with pm2
npm install -g pm2
pm2 start src/index.js --name level-bot
pm2 save
pm2 startup
```

Run whatever command `pm2 startup` prints — it sets up auto-restart on reboot.

### Optional: Montserrat font

The bot falls back to system fonts which is fine, but Montserrat looks better on the cards. Download it free from [Google Fonts](https://fonts.google.com/specimen/Montserrat) and drop the files here:

```
assets/fonts/Montserrat-Bold.ttf
assets/fonts/Montserrat-Regular.ttf
```

Restart the bot after adding them.

---

## File structure

```
discord-level-bot/
├── src/
│   ├── index.js                    # entry point — deploys commands then logs in
│   ├── commands/
│   │   ├── level.js                # rank card
│   │   ├── leaderboard.js          # top 10 image card
│   │   ├── stats.js                # server totals
│   │   ├── help.js                 # command list
│   │   ├── setupRoles.js           # create default level roles
│   │   ├── manageRoles.js          # set / remove / rename / list role milestones
│   │   ├── syncRoles.js            # rebuild everyone's roles from saved levels
│   │   ├── xpBoost.js              # timed XP events
│   │   ├── setXp.js                # force-set a user's level
│   │   ├── setChannel.js           # level-up announcement channel
│   │   ├── setCardColor.js         # card colour customisation
│   │   ├── setXpRate.js            # XP rate and multiplier config
│   │   ├── ignoreChannel.js        # toggle XP in a channel
│   │   ├── noXpRole.js             # block a role from earning XP
│   │   └── resetUser.js            # wipe a user's XP
│   ├── events/
│   │   ├── ready.js                # populates voice tracker on startup
│   │   ├── messageCreate.js        # text XP, cooldown, role awards
│   │   ├── voiceStateUpdate.js     # voice join/leave/mute tracking and XP ticker
│   │   └── interactionCreate.js    # command router
│   ├── database/
│   │   └── db.js                   # SQLite, all queries, XP math, role and boost APIs
│   └── utils/
│       ├── cardRenderer.js         # draws rank cards and leaderboard cards
│       ├── roleManager.js          # create default roles, assign/remove, sync
│       ├── deployCommands.js       # registers slash commands with Discord
│       └── xpHelpers.js            # hex validation, colour helpers
├── data/                           # auto-created — bot.db lives here
├── assets/fonts/                   # optional Montserrat fonts
├── .env.example
└── package.json
```

---

## Troubleshooting

**Slash commands aren't appearing**
Global commands take up to an hour. Set `GUILD_ID` in your `.env` to your server's ID for instant updates while testing, then remove it when you're done.

**Bot is online but doesn't respond to messages**
Message Content Intent isn't enabled. Go to the developer portal, open your bot's page, scroll down to Privileged Gateway Intents, and turn it on.

**Level-up messages aren't sending**
Run `/set-channel #your-channel`. Without a configured channel the bot tries to send in the same channel as the triggering message — if it doesn't have permission there, it fails silently.

**Activity roles aren't being assigned**
Two things to check: the bot's role needs to be above the level roles in Server Settings → Roles, and the bot needs the Manage Roles permission. Also make sure you've run `/setup-roles` or `/manage-roles set` to configure which roles go with which levels.

**`npm install` fails**
`@napi-rs/canvas` ships pre-built for most systems, but on older Linux distros it sometimes needs:
```bash
sudo apt install -y libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev
```

**Voice XP not tracking after a restart**
The bot scans all voice channels on startup and picks up anyone already there. If it's still not working, check the bot has the Connect permission in those channels.

**Roles are stacking when I want single mode, or vice versa**
Run `/setup-roles` again with the mode argument — either `stack` or `single` — then run `/sync-roles` to rebuild everyone's roles based on the new setting.

---

## License

MIT. Use it, fork it, do whatever — just keep the license file in there.
