<div align="center">

# Discord Level Bot

A levelling bot for Discord that tracks both chat and voice activity. Generates rank cards, handles leaderboards, and lets admins customise pretty much everything. Built with Node.js and discord.js v14.

[![Node.js](https://img.shields.io/badge/Node.js-18%2B-brightgreen?logo=node.js)](https://nodejs.org)
[![discord.js](https://img.shields.io/badge/discord.js-v14-5865F2?logo=discord)](https://discord.js.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

</div>

---

I made this because every level bot I tried either only tracked messages (ignoring voice completely), cost money for basic features, or stored everything on their own servers. This one's self-hosted, free, and tracks both text and voice XP separately — same idea as Many Level Bots but you run it yourself.

---

## What's included

- Rank cards with your avatar, text XP bar, voice XP bar, current level, and server rank — rendered as an image
- Leaderboards for text and voice, also rendered as image cards (top 10, medals for the top 3)
- Level-up announcements that send your new card to whatever channel you pick
- Anti-spam so people can't just spam one-word messages to farm XP
- Voice XP that ticks up every minute you're in a channel — going muted or deafened pauses it
- Full card colour customisation per server — accent colour, background, XP bars, text
- XP rate controls including a global multiplier (useful for events)
- Channel and role blocklists so you can exclude #bot-spam or muted members
- Admin commands that are actually hidden from regular users, not just gated by a permission check
- Slash commands that register themselves on startup — no separate deploy step, works on any host

---

## Getting started

### Step 1 — Create a bot account

Go to [discord.com/developers/applications](https://discord.com/developers/applications), hit **New Application**, and give it a name.

Once it's created, go to the **Bot** tab. You'll need to do a few things here:

- Scroll down to **Privileged Gateway Intents** and turn on **Server Members Intent** and **Message Content Intent**. The bot won't work without both of these.
- Click **Reset Token**, copy the token, and paste it somewhere safe. You won't be able to see it again.

Then go to **OAuth2 → General** and grab your **Client ID** — it's the big number near the top of the page.

### Step 2 — Invite the bot to your server

Go to **OAuth2 → URL Generator** and select these scopes:

```
bot
applications.commands
```

Then select these permissions:

```
View Channels
Send Messages
Embed Links
Attach Files
Read Message History
Connect
```

The Connect permission is what lets it see who's in voice channels. Without it, voice XP won't track.

Copy the URL it generates and open it in your browser to invite the bot.

### Step 3 — Install

```bash
git clone https://github.com/yourusername/discord-level-bot.git
cd discord-level-bot
npm install
```

You need Node.js 18 or newer. If you're not sure what version you have, run `node --version`.

### Step 4 — Set up your .env

```bash
cp .env.example .env
```

Open `.env` and fill in your values:

```env
DISCORD_TOKEN=your_bot_token_here
CLIENT_ID=your_client_id_here

# Set this to your server's ID while testing — commands show up instantly instead of waiting an hour
# Leave it blank when you're ready to go global
GUILD_ID=

DB_PATH=./data/bot.db
```

To find your server ID, turn on Developer Mode in Discord settings, then right-click your server icon and click "Copy Server ID".

### Step 5 — Run it

```bash
npm start
```

The bot will register all its slash commands automatically, then log in. You should see it go online in Discord within a few seconds.

---

## Commands

### For everyone

| Command | Description |
|---|---|
| `/level` | Your rank card showing both Text and Voice XP |
| `/level @someone` | Check another member's card |
| `/leaderboard text` | Top 10 by chat XP |
| `/leaderboard voice` | Top 10 by voice XP |
| `/stats` | Server totals — how many members are tracked, combined XP, highest levels |
| `/help` | Shows all available commands |

### For admins

These don't show up in Discord's autocomplete for regular members at all — not just blocked, genuinely hidden.

| Command | Description |
|---|---|
| `/set-xp @user text 25` | Set a user's text level directly |
| `/set-xp @user voice 10` | Set a user's voice level directly |
| `/set-channel #announcements` | Where to send level-up cards |
| `/set-channel` | Run with no argument to turn off level-up messages |
| `/set-card-color accent #FF6B6B` | Change the border and glow colour |
| `/set-card-color background #1A1A2E` | Change the card background |
| `/set-card-color bar #FF6B6B` | Change the XP bar colour |
| `/set-card-color text #FFFFFF` | Change text colour |
| `/set-xp-rate` | Check the current XP settings |
| `/set-xp-rate multiplier 2.0` | Run a double XP event |
| `/set-xp-rate text-min 10 text-max 30` | Change how much XP messages give |
| `/set-xp-rate voice-xp 15` | Change how much XP voice gives per minute |
| `/ignore-channel #bot-spam` | Stop a channel from giving XP (run again to re-enable) |
| `/no-xp-role @SomeRole` | Stop a role from earning XP (run again to re-enable) |
| `/reset-user @user text` | Wipe someone's text XP |
| `/reset-user @user voice` | Wipe someone's voice XP |
| `/reset-user @user all` | Full reset |

---

## Customising the card

Run `/set-card-color`, pick an element from the dropdown, and type a hex code. Both `#FF5733` and `FF5733` work fine.

| Element | Default | What it affects |
|---|---|---|
| Accent | `#5865F2` | Left edge stripe, avatar ring, glow |
| Background | `#23272A` | The card background |
| Bar | `#5865F2` | Text XP progress bar |
| Text | `#FFFFFF` | Username, level labels, XP numbers |

The voice XP bar shifts hue automatically from whatever the bar colour is set to, so you don't need to configure it separately.

A few combos worth trying:

```
Cyberpunk:  accent #00FFFF  background #0D0D1A  bar #FF00FF
Sunset:     accent #FF6B6B  background #1A1A2E  bar #FF8E53
Forest:     accent #57CC99  background #1B2226  bar #38A3A5
Gold:       accent #FFD700  background #1C1C1C  bar #FFA500
```

---

## How XP works

**Text XP** — each message gives between 15 and 25 XP by default (random, so it doesn't feel mechanical). There's a 60-second cooldown between grants, and messages under 5 characters don't count. The amount is multiplied by whatever the server multiplier is set to.

**Voice XP** — every 60 seconds, anyone in a voice channel who isn't muted or deafened gets 10 XP (default). If the bot restarts while people are in voice, it picks them up immediately when it comes back online.

**The level curve** — XP requirements go up exponentially. The formula is `floor(100 × (level + 1)^1.5)`, which in practice looks like this:

| Level | XP to reach this level | Total XP from scratch |
|---|---|---|
| 1 | 100 | 100 |
| 5 | 245 | 886 |
| 10 | 332 | 2,181 |
| 25 | 510 | 7,747 |
| 50 | 722 | 21,238 |
| 100 | 1,020 | 59,814 |

Early levels go fast to keep things engaging. Getting to level 100 takes real time without being absurd.

---

## Hosting

### Railway

Push your code to GitHub, create a new Railway project from that repo, add your environment variables under Variables, and deploy. Railway picks up `npm start` automatically and the commands register on boot so there's nothing extra to do.

### Render

Same idea — connect the repo, paste your env vars, set the start command to `npm start`. Pick **Background Worker** as the service type, not Web Service.

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

The last command (`pm2 startup`) prints a command you need to run to make it survive reboots. Just copy and run whatever it outputs.

### Better-looking cards (optional)

The bot uses system fonts by default which is fine, but Montserrat looks much cleaner. Download it free from [Google Fonts](https://fonts.google.com/specimen/Montserrat) and drop the files here:

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
│   ├── index.js                    # entry point, deploys commands then logs in
│   ├── commands/
│   │   ├── level.js
│   │   ├── leaderboard.js
│   │   ├── stats.js
│   │   ├── help.js
│   │   ├── setXp.js
│   │   ├── setChannel.js
│   │   ├── setCardColor.js
│   │   ├── setXpRate.js
│   │   ├── ignoreChannel.js
│   │   ├── noXpRole.js
│   │   └── resetUser.js
│   ├── events/
│   │   ├── ready.js                # scans voice channels on startup
│   │   ├── messageCreate.js        # text XP logic and anti-spam
│   │   ├── voiceStateUpdate.js     # voice join/leave/mute tracking
│   │   └── interactionCreate.js    # command router
│   ├── database/
│   │   └── db.js                   # SQLite, all queries, XP math
│   └── utils/
│       ├── cardRenderer.js         # draws the rank and leaderboard cards
│       ├── deployCommands.js       # registers slash commands with Discord
│       └── xpHelpers.js            # small utilities
├── data/                           # auto-created, this is where bot.db lives
├── assets/fonts/                   # optional Montserrat fonts go here
├── .env.example
└── package.json
```

---

## Troubleshooting

**Slash commands aren't appearing**
If you left `GUILD_ID` blank, global commands can take up to an hour to show up. Set it to your server ID for instant updates while you're testing, then remove it when you're done.

**Bot is online but nothing happens when I send a message**
Almost always means **Message Content Intent** isn't enabled. Go back to the developer portal, open your bot, scroll down on the Bot page, and make sure that toggle is on.

**Level-up messages aren't being sent**
You need to set a channel first — run `/set-channel #whatever` in your server. If no channel is set, the bot tries to send in the same channel as the message, and if it doesn't have permission there it'll just silently fail.

**`npm install` crashes with canvas errors**
`@napi-rs/canvas` ships pre-built so it usually just works, but on older Linux systems it sometimes needs these:

```bash
sudo apt install -y libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev
```

**Voice XP isn't tracking after a restart**
It should start tracking automatically — the bot checks all voice channels on startup. If it's still not working, double-check that the bot has the **Connect** permission in those voice channels.

---

## License

MIT. Use it, fork it, do whatever — just keep the license file in there.
