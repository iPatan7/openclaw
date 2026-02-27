---
summary: "Telegram integration + cross-platform posting for gym content and nasheed"
read_when:
  - Setting up Telegram for digital marketing
  - Automating posts across messaging platforms
  - Creating gym/fitness content with nasheed
title: "Digital Marketing Workflow"
---

# Digital Marketing Workflow

Telegram integration, cross-platform broadcast, and automated workflows for gym content and nasheed.

## 1. Telegram Setup

### Step 1: Create your bot

1. Open Telegram and message **@BotFather**.
2. Send `/newbot`.
3. Follow the prompts (name + username, e.g. `@YourGymBot`).
4. Copy the token (e.g. `1234567890:ABCdefGHI...`).

### Step 2: Configure OpenClaw

Add to `~/.openclaw/openclaw.json` (or run `openclaw configure` and edit):

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "YOUR_BOT_TOKEN_FROM_BOTFATHER",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

Or use env: `TELEGRAM_BOT_TOKEN=...` (set in `~/.openclaw/.env`).

### Step 3: Start and approve

```bash
pnpm openclaw gateway
```

DM your bot on Telegram. You'll get a pairing code. Approve:

```bash
pnpm openclaw pairing list telegram
pnpm openclaw pairing approve telegram <CODE>
```

### Step 4: Telegram channel for marketing

Create a **Telegram channel** (not group) for your gym/nasheed content:

1. Telegram → Menu → New Channel.
2. Choose a name and add your bot as admin with "Post Messages".
3. Get the channel ID (e.g. `@YourGymChannel` or `-1001234567890`).

Add to config so the bot can post there:

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false, groupPolicy: "open" },
      },
    },
  },
}
```

Replace `-1001234567890` with your channel ID. For public channels, use `@channelname`.

---

## 2. Cross-Platform Posting

OpenClaw can **broadcast** to multiple channels at once: Telegram, Discord, Slack, WhatsApp.

### What it covers

| Platform   | Supported | Notes                                  |
|-----------|-----------|----------------------------------------|
| Telegram  | Yes       | Channels, groups, DMs                  |
| Discord   | Yes       | Channels, DMs                          |
| Slack     | Yes       | Channels, DMs                          |
| WhatsApp  | Yes       | Groups, DMs                            |
| Instagram | No        | No native API; see workarounds below   |
| TikTok    | No        | No native API; see workarounds below   |
| YouTube   | No        | No native API; see workarounds below   |

### Broadcast CLI

Post the same message to several targets/channels:

```bash
# Single channel, multiple targets
pnpm openclaw message broadcast \
  --channel telegram \
  --targets "@YourGymChannel" "@BackupChannel" \
  --message "New gym clip dropping today! 💪🕌" \
  --media "https://example.com/video.mp4"

# All configured channels (Telegram + Discord + Slack + WhatsApp)
pnpm openclaw message broadcast \
  --channel all \
  --targets "@YourGymChannel" "channel:DISCORD_ID" "#marketing" \
  --message "Check out today's nasheed mix 🔥" \
  --media "https://example.com/audio.mp3"
```

Enable broadcast in config (default: on):

```json5
{
  tools: {
    message: {
      broadcast: { enabled: true },
    },
  },
}
```

---

## 3. Automated Workflow (Cron)

Schedule posts so the agent runs on a timer.

### Daily content reminder

```bash
pnpm openclaw cron add \
  --name "Daily content prompt" \
  --cron "0 8 * * *" \
  --tz "America/New_York" \
  --session isolated \
  --message "Reminder: draft today's gym post with nasheed suggestion. Keep it short and motivating." \
  --announce \
  --channel telegram \
  --to "@YourGymChannel"
```

### Weekly planning

```bash
pnpm openclaw cron add \
  --name "Weekly content plan" \
  --cron "0 9 * * 1" \
  --tz "America/New_York" \
  --session isolated \
  --message "Create a 7-day content calendar for gym clips + nasheed. Mix workout tips, nasheed recommendations, and motivational quotes." \
  --announce \
  --channel telegram \
  --to "@YourGymChannel"
```

The agent will generate content; you can edit and broadcast via `/send` or CLI.

---

## 4. Agent Prompt for Gym + Nasheed

Add this to your agent's system prompt or workspace `AGENTS.md` so it understands your niche:

```markdown
## Content focus
- Gym/fitness content with Islamic nasheed background
- Motivating, short-form posts
- Blend fitness tips with spiritual inspiration
- Suggest nasheed tracks that fit workout energy (upbeat for cardio, calm for cooldown)

## Post style
- Short captions (1–2 lines for Telegram/Instagram)
- Hashtags: #GymMotivation #Nasheed #IslamicFitness #FitnessJourney
- Emoji use: moderate, motivational
```

---

## 5. Instagram / TikTok / YouTube

OpenClaw does not post directly to Instagram, TikTok, or YouTube. Options:

### A. Telegram-first strategy

1. Post to Telegram channel (and Discord/Slack if you use them).
2. Copy from Telegram and paste to Instagram/TikTok/YouTube manually.
3. Use OpenClaw to draft captions, suggest nasheed, and plan timing.

### B. Third-party schedulers

Use Buffer, Hootsuite, Later, or Creator Studio. Have the agent:

- Draft posts and captions.
- You paste into the scheduler or use their API (if you build a small skill).

### C. Browser tool (advanced)

With `browser` tool enabled and a paired node, the agent could help automate posting via the web (e.g. Instagram web, TikTok upload). This is advanced, requires careful setup, and can break when UIs change.

---

## 6. Quick Reference

| Task              | Command / config                                      |
|-------------------|-------------------------------------------------------|
| Telegram config   | `channels.telegram.botToken` in `openclaw.json`       |
| Approve DM        | `openclaw pairing approve telegram <CODE>`            |
| Broadcast         | `openclaw message broadcast --channel all --targets X Y --message "..."` |
| Schedule post     | `openclaw cron add --name "..." --cron "0 8 * * *" --message "..." --announce` |
| List cron         | `openclaw cron list`                                  |
| Channel status    | `openclaw channels status`                            |

---

## 7. After Anthropic Setup

When you add your Anthropic API key tomorrow:

1. `pnpm openclaw onboard` → pick Anthropic, paste key.
2. Set model: `pnpm openclaw models set anthropic/claude-opus-4-6`.
3. Add web search: `pnpm openclaw configure --section web` (for research/nasheed discovery).
4. The agent will handle content drafting, nasheed suggestions, and cron-triggered workflows.

Docs: [Telegram](/channels/telegram) · [Cron](/automation/cron-jobs) · [Broadcast](/cli/message#broadcast)
