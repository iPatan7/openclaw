---
summary: "Comprehensive guide to using OpenClaw to its fullest"
read_when:
  - You want to get the most out of OpenClaw
  - You've completed onboarding and want to optimize
title: "Comprehensive Usage Guide"
---

# OpenClaw Comprehensive Guide

How to use OpenClaw to its absolute best: setup, security, channels, models, tools, and workflows.

## 1. Complete Onboarding

If you haven't finished the wizard yet, run:

```bash
openclaw onboard --install-daemon
```

Select **Yes** at the security warning, then walk through:

1. **Model/Auth** — Choose Anthropic (recommended), OpenAI, or a custom provider. Pick a default model.
2. **Workspace** — Default `~/.openclaw/workspace` is fine.
3. **Gateway** — Port 18789, token auth (auto-generated).
4. **Channels** — Add WhatsApp, Telegram, Discord, Slack, Signal, or WebChat.
5. **Daemon** — Installs systemd (Linux) or launchd (macOS) so the gateway runs in the background.
6. **Skills** — Install recommended skills.

For **Advanced** mode, pass `--advanced` to expose every configuration step.

---

## 2. Fastest First Chat

No channel setup needed: use the Control UI.

```bash
openclaw dashboard
```

Opens the browser at `http://127.0.0.1:18789/`. Chat directly with your assistant.

---

## 3. Model Best Practices

### Recommended model stack

- **Primary:** Anthropic Claude Opus 4.6 (best prompt-injection resistance, long context).
- **Fallback:** Add `anthropic/claude-sonnet-4-5` or `openai/gpt-5.2` in `agents.defaults.model.fallbacks`.
- **Image model:** Set `agents.defaults.imageModel.primary` if your primary can't handle images.

### Set your model

```bash
openclaw models set anthropic/claude-opus-4-6
openclaw models status
```

### In chat: switch models

```
/model          # Interactive picker
/model list     # Numbered list
/model 3        # Pick by number
/model anthropic/claude-sonnet-4-5
```

### Auth tip

For Anthropic, prefer the setup-token flow:

```bash
claude setup-token
openclaw models status
```

---

## 4. Web Search & Knowledge

Enable `web_search` so the agent can look things up.

### Brave Search (default, free tier)

```bash
openclaw configure --section web
```

Stores `BRAVE_API_KEY` from [brave.com/search/api](https://brave.com/search/api). Get a key from the Data for Search plan.

### Perplexity (AI-synthesized answers)

Set `OPENROUTER_API_KEY` or `PERPLEXITY_API_KEY` in `~/.openclaw/.env`, or configure:

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

`web_fetch` (URL fetching) is enabled by default. No key needed.

---

## 5. Security Baseline

### Run the audit regularly

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

### DM pairing (recommended)

Unknown senders get a pairing code; you approve them:

```bash
openclaw pairing list telegram
openclaw pairing approve telegram ABC12345
```

### Groups: require mention

In `openclaw.json`:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

### Session isolation for shared inboxes

If multiple people can DM the bot:

```json5
{
  session: { dmScope: "per-channel-peer" },
}
```

### Doctor check

```bash
openclaw doctor
```

Fixes permissions, suggests config improvements, surfaces risky DM policies.

---

## 6. Channels in Depth

### Connect channels

During onboarding, or later:

```bash
openclaw configure
```

Per-channel docs: [Channels](/channels).

### Pair new DMs

| Channel   | Approve command                      |
|----------|--------------------------------------|
| Telegram | `openclaw pairing approve telegram <code>` |
| WhatsApp | `openclaw pairing approve whatsapp <code>` |
| Discord  | `openclaw pairing approve discord <code>` |
| Slack    | `openclaw pairing approve slack <code>`   |
| Signal   | `openclaw pairing approve signal <code>`  |

### Channel status

```bash
openclaw channels status
openclaw channels status --probe
```

---

## 7. Agent from CLI

Run the agent directly (no inbound message):

```bash
# Basic
pnpm openclaw agent --message "Summarize today's tasks"

# With session target
pnpm openclaw agent --to +15555550123 --message "Status update"

# Deliver reply to channel
pnpm openclaw agent --message "Report" --deliver --channel slack --reply-to "#reports"

# Thinking level (GPT-5.2, Codex)
pnpm openclaw agent --message "Debug this" --thinking high
```

---

## 8. Essential Slash Commands

Use these in chat (WhatsApp, Telegram, Discord, etc.):

| Command       | Purpose                           |
|---------------|-----------------------------------|
| `/help`       | Show help                         |
| `/status`     | Gateway + model status            |
| `/model`      | Switch model                      |
| `/reset`      | New conversation                  |
| `/think high` | Enable extended thinking          |
| `/verbose on` | Show tool output (debug only)     |
| `/skill &lt;name&gt;` | Run a skill by name       |
| `/whoami`     | Your sender ID                    |

Directives (e.g. `/think`, `/verbose`) persist when sent as **standalone** messages. Inline in a message, they apply once only.

---

## 9. Skills

Skills extend the agent with custom tools and workflows.

### Configure skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/my-skills"],
      watch: true,
    },
    entries: {
      peekaboo: { enabled: true },
      sag: { enabled: false, apiKey: { source: "env", id: "SAG_API_KEY" } },
    },
  },
}
```

### Run a skill

In chat: `/skill &lt;name&gt; [input]`

Or reference it in your message; the agent will invoke it when appropriate.

### Per-skill env vars

When using a sandbox, set env via `agents.defaults.sandbox.docker.env` or `skills.entries.<name>.env`.

---

## 10. Multi-Agent Setup

Create separate agents for different purposes:

```bash
openclaw agents add ops
openclaw agents add research
```

Configure bindings so different channels or groups route to different agents. See [Multi-Agent](/concepts/multi-agent).

---

## 11. Gateway Operations

### Run in foreground (debug)

```bash
pnpm openclaw gateway --port 18789 --verbose
```

### Watch mode (dev loop)

```bash
pnpm gateway:watch
```

Auto-reloads on TypeScript changes.

### Check status

```bash
openclaw gateway status
openclaw status --all
openclaw status --deep
```

---

## 12. Sending Messages

```bash
pnpm openclaw message send --to +15555550123 --message "Hello from OpenClaw"
pnpm openclaw message send --target "#general" --channel slack --message "Announcement"
```

Target formats match channel conventions (phone numbers, Discord channels, Slack channels).

---

## 13. Tool Profiles

Control what tools the agent can use:

- `messaging` — messaging only
- `minimal` — basic tools, no exec/browser
- `full` — all tools (default for personal use)

Per-agent overrides in `agents.list[].tools`:

```json5
{
  agents: {
    list: [{
      id: "public",
      tools: {
        allow: ["web_search", "web_fetch"],
        deny: ["exec", "browser", "read", "write"],
      },
    }],
  },
}
```

---

## 14. Cron Jobs

Schedule recurring tasks from the Control UI or via config. The agent can create cron jobs when `cron` tool is allowed.

---

## 15. Nodes (iOS/Android/macOS)

Pair mobile or macOS nodes for:

- Remote commands
- Voice wake / talk mode
- Device actions

```bash
openclaw devices list
openclaw devices approve <requestId>
```

From Telegram: `/pair` → paste setup code in app → `/pair approve`.

---

## 16. Checklist for Best Experience

- [ ] Complete `openclaw onboard --install-daemon`
- [ ] Set primary model (Anthropic Opus 4.6 recommended)
- [ ] Add web search key: `openclaw configure --section web`
- [ ] Run `openclaw security audit --fix`
- [ ] Configure DM pairing for channels you use
- [ ] Set `session.dmScope: "per-channel-peer"` if shared inbox
- [ ] Enable `requireMention` in groups
- [ ] Open Control UI: `openclaw dashboard`
- [ ] Install useful skills during onboarding
- [ ] Run `openclaw doctor` after config changes

---

## 17. Digital Marketing (Telegram + Cross-Platform)

For gym content, nasheed, and cross-platform posting: [Digital Marketing Workflow](/start/digital-marketing-workflow).

---

## 18. Quick Reference URLs

- [Getting Started](/start/getting-started)
- [Onboarding Wizard](/start/wizard)
- [Models](/concepts/models)
- [Security](/gateway/security)
- [Web Tools](/tools/web)
- [Pairing](/channels/pairing)
- [Slash Commands](/tools/slash-commands)
- [Control UI](/web/control-ui)
