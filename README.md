# Hermes Agent — Colab + Google Drive + Telegram Setup

A one-click Google Colab notebook that installs and runs [Hermes Agent](https://github.com/NousResearch/hermes-agent) (by [Nous Research](https://nousresearch.com)) as a persistent Telegram bot — with Google Drive backing so your config, credentials, and memory survive across Colab sessions, and zero recurring API cost.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sourabhsuman137/hermes-agent-colab-drive-telegram/blob/main/Hermes_Agent_Colab_Drive_Telegram_v3.ipynb) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/sourabhsuman137/hermes-agent-colab-drive-telegram/blob/main/LICENSE)

> This repo does **not** contain Hermes Agent's source code. It's a setup notebook that installs the official Hermes Agent via its own installer and configures it for a specific environment (Colab + Drive + Telegram). All credit for the agent itself goes to [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) (MIT licensed).

## What this does

- 📦 Installs Hermes Agent fresh in every Colab session (fast, local disk) — no manual setup steps after the first run
- 💾 Persists config, `.env`, skills, and conversation memory in a `Hermes Agent` folder on **your Google Drive**, so nothing is lost when the Colab runtime resets
- 🤖 Connects the agent to a **Telegram bot** — chat with your agent from your phone, anywhere
- ✍️ Tool activity is shown as a **live-edited Telegram message** (one bubble updates in place) instead of spamming new messages
- 🆓 Defaults to **OpenRouter's free models** ($0 per token) — lists live free-model options at setup time, or auto-picks the best available one
- 🔁 Configures **automatic fallback**: if the selected model errors out or gets rate-limited, Hermes automatically retries on a free model
- 🩺 Includes a pre-flight self-check cell that catches misconfiguration before the bot goes live, plus fixes for a few real Colab-specific installer quirks (uv path mismatch, missing Telegram dependency, root-mode install paths)

## Requirements

- A Google account (for Colab + Drive)
- A Telegram account
  - Create a bot via [@BotFather](https://t.me/BotFather) → get a bot token
  - Message [@userinfobot](https://t.me/userinfobot) → get your numeric Telegram user ID
- A free [OpenRouter](https://openrouter.ai/keys) account → API key (no credit card required)

## Quickstart

1. Click **"Open in Colab"** above (or open the `.ipynb` file in Google Colab manually)
2. `Runtime → Run all`
3. When prompted, enter your Telegram bot token, Telegram user ID, and OpenRouter API key (one-time only)
4. Pick a free model from the list, or press Enter to auto-select
5. Once the last cell is running, message your bot on Telegram

On every future run, just `Run all` again — your credentials and settings are restored automatically from Drive.

## How it's structured

The notebook is a linear sequence of cells: mount Drive → install Hermes Agent → install Telegram support → restore saved config from Drive → pick a model → first-time credential setup (skipped after the first run) → pre-flight check → backup to Drive → start the Telegram gateway (runs until you stop it, auto-backing up every 5 minutes).

## Cost

$0, as long as you stay on OpenRouter's free-model tier (20 requests/minute, 50/day — or 1,000/day if you've ever added a one-time $10 credit balance to your OpenRouter account). See [OpenRouter's pricing](https://openrouter.ai/docs) for current limits.

## Limitations

- Colab's free tier disconnects idle/long-running sessions after a few hours — fine for personal/testing use, not a substitute for a real always-on server (e.g. a small VPS) if you need 24/7 uptime
- Free OpenRouter models are rate-limited and occasionally get delisted/rotated — the built-in fallback handles temporary model issues, not the daily request cap

## Credits

- [Hermes Agent](https://github.com/NousResearch/hermes-agent) by Nous Research — MIT License
- [OpenRouter](https://openrouter.ai) for free model access

## License

The setup notebook and files in this repo are released under the [MIT License](https://github.com/sourabhsuman137/hermes-agent-colab-drive-telegram/blob/main/LICENSE). Hermes Agent itself is a separate project, MIT licensed by Nous Research.
