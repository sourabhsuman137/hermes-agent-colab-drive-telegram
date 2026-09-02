# Hermes Agent v3 → v4 — Change Log & Guide

## Before anything else: how this was built, and one real limitation

Your v3 notebook is installed on top of **[NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)**
— a large, real, actively developed open-source agent (not a small custom script). That
changes the right design for almost every upgrade you asked for: Hermes already has a
native identity file (`SOUL.md`), native persistent memory (`MEMORY.md`/`USER.md`),
a native skills system, native per-task model routing, and more. Building a second,
parallel version of any of that inside the wrapper notebook would fight the framework
instead of extending it — so v4's real job, for most of the 12 upgrades, was
**configuring and writing good content for Hermes's own systems**, not inventing new
ones. Everything below explains, upgrade by upgrade, which of those two v4 actually is.

**One honest limitation:** I was not able to programmatically fetch the raw content of
your actual `Hermes_Agent_Colab_Drive_Telegram_v3.ipynb` from GitHub — `.ipynb` files
render through GitHub's JS notebook viewer rather than as plain text, and the direct
raw-file path is blocked to automated fetches. I *did* fully retrieve your wrapper
repo's own README, which documents v3's exact behavior and flow in detail, and I
cross-checked every command and config key below against Hermes's current official
documentation and source. v4 is built from that combination, faithfully preserving
everything the README describes v3 as doing. If your actual v3 code has extra custom
logic beyond what the README describes (a particular preflight check, bash quirks
specific to your setup), paste the specific cell(s) and I'll merge them in precisely
rather than guess.

---

## Change log, by upgrade

**1 — SOUL.md.** Hermes already loads a `SOUL.md` first, in every session, as its
primary identity. v4 writes a complete one covering everything you asked for:
understanding intent without over-asking, answer-first structure, fact vs. assumption
labeling, anti-hallucination rules, tool-use judgment, Hindi/Hinglish/English
adaptation, concise-vs-detailed calibration, and a pre-send quality checklist. Written
once; never overwritten on later runs unless you set `FORCE_RESET_IDENTITY_FILES`.

**2 — Persistent memory.** `MEMORY.md`/`USER.md` are native — Hermes's own agent loop
writes and maintains them (duplicate-prevented, security-scanned). v4's job was making
sure they're enabled and that `~/.hermes/memories/` is included in the native
`hermes backup`/`hermes import` cycle, which it is by construction.

**3 — Skills.** Written as real `SKILL.md` files (correct frontmatter, correct
directory layout) into Hermes's native skills folder, for all 8 requested domains:
deep reasoning, research, coding, fact-checking, writing, decision-making, data
analysis, personal assistant. Hermes's own progressive-disclosure system (short
index always loaded, full skill body loaded only when relevant) is what gives you
"loaded only when relevant" — no custom loader was written, because one already
exists. Existing/edited skill files are never overwritten.

**4 & 5 — Model routing.** Primary model selection now scores OpenRouter's *live* free
model list on more than context length: real capability signals where OpenRouter
exposes them (tool-calling and reasoning support via `supported_parameters`), plus an
openly-editable, clearly-labeled heuristic nudge — not a claim of ground truth.
`fallback_providers` is set to several other free models across different families.
Auxiliary tasks (compression, title generation, background memory review, vision,
subagent delegation) are pinned to an explicit fast/cheap free model via Hermes's
native per-task `auxiliary.*` config, which **is** the native version of "fast model
for simple tasks, auxiliary model for subtasks."

One thing this does *not* do, on purpose: dynamically switch the **primary** chat
model per incoming message based on a live complexity classification. Hermes's
gateway runs one configured primary model per session; there's no native hook to swap
it mid-stream per message, and wrapping the gateway to intercept and reclassify every
message before Hermes sees it would mean fighting the framework's own architecture,
not extending it. Since these are free models, this only costs a small latency
difference on simple questions — not money. If you want it later, `/model` is
available for manual switching, or it's worth raising upstream as a feature request.

**6 — Verification/QC.** Hermes's native `agent.verify_on_stop` refuses to accept a
final answer on a turn where code was edited but no fresh test/build/lint evidence was
produced — this is enabled explicitly (not left on `"auto"`, which defaults **off**
for messaging surfaces like Telegram). That covers the coding half of this upgrade
natively. There's no native general-purpose "verify every complex answer" pipeline
stage for non-coding claims; the safest compatible version of that is the self-check
section written into SOUL.md (point 8), since a pipeline stage would require patching
Hermes's own source, which the wrapper notebook doesn't do.

**7 — Tool usage.** Governed by SOUL.md section 5 plus the skill files' own "Tool
Preferences" sections, on top of Hermes's existing 40+ built-in tools — no new tool
system, just better guidance for the one that's already there.

**8 — Response style.** SOUL.md section 6 (language mirroring including Hinglish
code-switching, length calibration, no filler, Telegram-appropriate formatting).

**9 — Drive persistence.** Switched from (what the README describes as) direct file
copying to Hermes's own native `hermes backup` / `hermes import` — a tested,
zip-based export/import Hermes ships specifically for this. It matters concretely:
Hermes's session database is SQLite in WAL mode, and `hermes backup` snapshots it
through SQLite's own backup API rather than copying the file directly, which avoids a
real corruption risk that a naive `cp`/`rsync` of a live WAL database has.

**10 — Security.** Carries forward the Telegram allow-list (now a hard failure if
empty, rather than a soft warning), adds `.env` file permissions (600), and turns on
two native protections that are off by default upstream: `skills.guard_agent_created`
(scans skill files the agent writes to itself) and explicit `approvals.mode: smart`
for risky commands. Colab's own default of running as root is a platform property,
not something a notebook can change — flagged rather than silently ignored.

**11 — Error handling.** A shared `run()` helper distinguishes required steps (raise
with a clear message, stopping "Run all" at an understandable point) from best-effort
ones (warn and continue). The gateway loop restarts a crashed process up to 5 times
before giving up loudly instead of restart-looping forever, and always backs up before
each restart attempt.

**12 — Benchmark.** Not natively covered by Hermes (its own `evals/` folder is for
Hermes's own development, not for comparing candidate backend models for your use
case), so this is genuinely new code — a small, honestly-scoped harness with visible
heuristic checks (does an expected value appear, does generated code parse, was a
format constraint followed), explicitly *not* presented as a rigorous benchmark.

**Also included, from mid-conversation follow-ups:**
- **First-run credential prompting, fixed.** The likely bug: checking whether `.env`
  *exists* rather than whether the specific required *values* are actually inside it
  (an empty or partial file reads as "already configured"). v4's credential cell reads
  the three required values individually and prompts only for genuinely missing ones,
  running strictly after Drive restore so a real first run (nothing restored) still
  triggers all three prompts, and a returning run (valid backup restored) triggers
  none.
- **Live browser access.** Hermes's built-in interactive browser tools (navigate,
  click, type, read page content — not just one-shot search) are enabled. As of this
  writing the default backend is a lightweight local browser Hermes manages itself
  (no Chromium/Playwright install needed for it), with automatic fallback to real
  Chrome for anything that needs it. No screenshots in the lightweight default mode —
  it works text-first.

**Explicitly preserved:** Drive mount, `.env` persistence, Telegram bot + allow-list,
OpenRouter free-model selection with automatic fallback, Hermes skills, session
persistence, automatic Drive backup, Telegram live-progress behavior (native to
Hermes's gateway), and Colab compatibility — including re-implemented (not
byte-identical, since the original code wasn't accessible) versions of v3's three
documented installer fixes: the PATH issue, the messaging dependency, and root-mode
install paths.

---

## Setup instructions

1. **Get three things ready** (2 minutes):
   - An OpenRouter API key — [openrouter.ai/keys](https://openrouter.ai/keys)
   - A Telegram bot token — message [@BotFather](https://t.me/BotFather) → `/newbot`
   - Your Telegram numeric user ID — message [@userinfobot](https://t.me/userinfobot)
2. **Upload the notebook** to Google Colab (`File → Upload notebook`).
3. **If you're upgrading from v3**, open the config cell and set
   `DRIVE_BACKUP_FOLDER` to the exact Drive folder your v3 notebook already used, so
   v4 continues from your existing state instead of starting fresh. If unsure, check
   `My Drive` for a folder containing a `.env` or `config.yaml`.
4. **Runtime → Run all.** On a genuine first run, one cell pauses to ask for the three
   items from step 1 — enter them once. Everything else is automatic.
5. **Message your bot on Telegram.** Once the last cell shows "Starting Telegram
   gateway," it's live.
6. **To stop:** press the ■ stop button on the running cell (or interrupt it) — this
   triggers one final Drive backup automatically before it exits.
7. **Next time:** just `Runtime → Run all` again. Nothing will be asked a second time.

---

## Testing instructions — 10–20 prompts to check v4 actually improved things

Send these to your bot on Telegram. What to look for is next to each one.

1. **"What's 17 × 24?"** — Should be exactly `408`, computed (not eyeballed).
2. **"A farmer has 17 sheep. All but 9 die. How many are left?"** — Correct answer is
   `9` (a classic trap); watch for the deep-reasoning skill catching it instead of a
   quick wrong `8`.
3. **"मुझे कल की meeting के बारे में बताओ"** (or any Hinglish message) — Should reply
   naturally in Hindi/Hinglish, not stiff English or a literal translation.
4. **"Write a Python function that checks if a number is prime, then actually run it
   on a few test cases."** — Should show real execution output, not just code.
5. **"What's the current USD to INR exchange rate?"** — Should use a tool (search or
   browser) rather than quoting a stale training-data number.
6. **"Go to [any news homepage] and tell me the top headline right now."** — Tests the
   new interactive browser toolset specifically (not just search).
7. **"Should I use PostgreSQL or MongoDB for a small inventory app?"** — Should give an
   actual recommendation tied to stated/reasonable criteria, not just a neutral list.
8. **"Summarize [paste a paragraph] in exactly one sentence."** — Tests instruction
   following on a hard length constraint.
9. **"Is it true that [pick a specific, checkable current claim]?"** — Should verify
   rather than assert from memory, and say plainly if it can't confirm something.
10. **Ask something you told it in an earlier session** (e.g., a preference you
    mentioned days ago) — Tests that `MEMORY.md`/`USER.md` actually persisted through
    the Drive backup/restore cycle.
11. **"What is 2+2?"** — Should be instant and short, with no unnecessary verification
    overhead for something trivial.
12. **Deliberately introduce a bug** (paste broken code and ask for a fix) — Should
    reproduce the bug, fix it, and show a passing re-run (native `verify_on_stop`).
13. **"Give me 3 bullet points, no more, no less, about [topic]."** — Exact-count
    instruction following.
14. **Ask for something requiring 3+ sequential steps** (e.g., "find X, then compute Y
    from it, then tell me Z") — Tests multi-step tool use in one turn.
15. **Restart the Colab runtime and "Run all" again** — Should ask for nothing (this
    is the specific bug that was fixed) and should resume with memory intact.
16. **Send a message from a Telegram account *not* in your allow-list** (e.g., ask a
    friend to try, or a second account) — Should be silently ignored/refused, not
    answered.
17. **"What model are you running on right now?"** or check the setup cells' printed
    output — Confirms which model the live scoring actually picked, so you're not
    flying blind on what's serving you.

---

## Troubleshooting

| Symptom | Likely cause / fix |
|---|---|
| `hermes: command not found` in a later cell | PATH fix didn't find the venv. Run the [DEBUG] cell; check `~/.hermes/hermes-agent/venv/bin` exists. |
| `ModuleNotFoundError: No module named 'dotenv'` | Something invoked the raw `~/.hermes/hermes-agent/hermes` file with system Python instead of the venv launcher — re-run Step 2. |
| Bot doesn't respond on Telegram at all | Check `hermes status` and `hermes logs errors -n 40` in the [DEBUG] section; confirm your Telegram user ID in `TELEGRAM_ALLOWED_USERS` exactly matches what @userinfobot gave you. |
| Credentials asked again even though you set them before | `DRIVE_BACKUP_FOLDER` probably doesn't match the folder your previous run backed up to — check `My Drive` for the actual folder name. |
| "Telegram will reject concurrent polling for the same token" / weird double-replies | An old v3 (or an earlier v4) Colab runtime is still running with the same bot token. Stop that runtime first. |
| A configured free model suddenly errors as unavailable | Free models on OpenRouter do get retired. Re-run the Step 7 model-routing cell to re-score against the current live list, or run `hermes migrate` to clean up stale references. |
| OpenRouter rate-limit errors under heavy use | Expected on the free tier under load — the fallback chain should absorb it. For a harder ceiling, see the optional NVIDIA NIM fallback cell. |
| Model-scoring cell prints a warning and picks nothing | Usually a bad/missing `OPENROUTER_API_KEY` or OpenRouter itself briefly down — check the key, retry, or set a model manually with `hermes model`. |
| Browser tool calls fail | Run `hermes doctor -v`; on some networks the self-managed local browser process needs a moment on first use — retry once before assuming it's broken. |
| Colab disconnects mid-session | Normal on the free Colab tier after some hours idle. Your last backup (at most `BACKUP_INTERVAL_SECONDS` old) is safe on Drive — just "Run all" again. |
| Want to reset SOUL.md/skills back to this notebook's defaults | Set `FORCE_RESET_IDENTITY_FILES = True` in the config cell once, run, then set it back to `False`. |

---

## Performance & cost notes

Everything here runs on **free** OpenRouter models by default, so "cost" mostly means
**request volume against free-tier rate limits**, not money:

- **Auxiliary tasks add calls you don't see directly**: title generation (once per new
  conversation), background memory review (after turns where something worth
  remembering happened), compression (on long conversations), vision (only if an
  image is sent). All are pinned to the cheap/fast model chosen in Step 7, not the
  primary model, specifically to keep these light.
- **`verify_on_stop`** adds a verification pass on turns that edited code — the
  tradeoff is more reliable code changes for a modest number of extra calls; turn it
  off (`hermes config set agent.verify_on_stop false`) if you want fewer calls and
  code correctness matters less for your use case.
- **The benchmark cell** (optional, off by default) makes `len(models) × len(test cases)`
  real model calls — expect it to take a few minutes and to visibly consume your
  per-minute rate limit if you run it.
- **Skills cost very little**: only a short name+description per skill sits in every
  request by default (progressive disclosure); the full body of a skill loads only
  when it's actually relevant.
- **Monitor actual usage** any time with `hermes insights --days 7` (the [DEBUG]
  section runs this for you) — token counts and call volume, not guesswork.
- OpenRouter's free tier enforces both a per-minute and a per-day request cap; the
  exact numbers are set by OpenRouter and do change, so check
  [openrouter.ai/docs](https://openrouter.ai/docs) if you want current figures rather
  than relying on a number here that could be stale by the time you read it.
