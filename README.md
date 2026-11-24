# lzs

**Run Claude Code, Codex or OpenCode in cloud sandboxes. One command per session, dozens at once, and they keep going when your laptop sleeps.**

[![npm](https://img.shields.io/npm/v/@lizard-build/lzs?color=cb3837&label=npm)](https://www.npmjs.com/package/@lizard-build/lzs)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Runs on Lizard](https://img.shields.io/badge/runs%20on-Lizard-2ea44f)](https://lizard.build)

```sh
npm i -g @lizard-build/cli @lizard-build/lzs
lizard login && lzs init --new-project     # once (see Setup: a few minutes)
lzs .                                      # Claude Code on this repo, in its own sandbox
```

```console
$ lzs fan 5 . -p "fix the flaky checkout test" --check "npm test"
lzs starting 5 claude sessions on shop…
  ✓ lzs-1 3.1 s
  ✓ lzs-2 3.4 s
  ✓ lzs-3 3.2 s
  ✓ lzs-4 3.6 s
  ✓ lzs-5 3.3 s

5 of 5 running after 4.0 s · lzs watch to follow them; each runs npm test when it finishes · lzs diff to compare · lzs pick <name> to keep the best

$ lzs ls
lzs-1     working     claude           shop · Fix flaky checkout test · +40 −12
lzs-2     your turn   claude   ✓ check shop · Fix flaky checkout test · +18 −3 · idle 4m
lzs-3     your turn   claude   ✗ check shop · Fix flaky checkout test · +95 −40
lzs-4     asking you  claude           shop · Fix flaky checkout test · +7 −2
lzs-5     paused      claude           shop · Fix flaky checkout test

5 sessions · 4 running · ~$0.016/h (nano) · 3 waiting for you

$ lzs pick lzs-2 --yes        # keep it as branch lzs/lzs-2, delete the other four
```

---

## Why

- **Your laptop isn't the bottleneck any more.** Each agent gets its own Linux sandbox, booted from your warm snapshot in about 3 seconds. Run one, or forty, without your fans noticing.
- **It keeps going while you sleep.** The agent runs in tmux in the cloud. Close the lid, get a notification on your phone when it's done.
- **Best of N, not hope.** Give the same task to N agents, run your tests in each, keep the one that passes.
- **Cheap when idle.** A session pauses itself after 30 idle minutes and wakes in ~3 s with the conversation intact. Paused sessions don't pay for compute.
- **Your setup comes along.** Your `CLAUDE.md`, skills, agents, commands, settings and MCP servers (secrets stripped), Codex's `AGENTS.md` and config, OpenCode's config.
- **The work comes back as git.** `lzs pull` lands it as a normal branch; `lzs push` opens a pull request. Transcripts sync back to `~/.claude/projects` and `~/.codex/sessions`.

## Setup

**1. A Lizard account.** Sign up at [lizard.build](https://lizard.build). New accounts start with a **7-day Pro trial that needs a card** ($5 of usage included, then $19/month). See [Limits](#how-many-agents-at-once) and [Known limitations](#known-limitations) before you start on a trial.

**2. Install and sign in.**

```sh
npm i -g @lizard-build/cli @lizard-build/lzs
lizard login
```

Today, sessions need `lizard` signed in with an **API key** (a `liz_…` key), not just the browser login. Make one and sign in with it:

```sh
lizard keys create lzs          # prints a liz_… key; copy it
lizard login --token liz_…      # or: export LIZARD_TOKEN=liz_…
```

**3. Connect your agent's login.** For Claude Code, make a long-lived token and add it to the project lzs will use (agent logins on Lizard are **per project**):

```sh
claude setup-token                       # prints a long-lived OAuth token
lizard agents add claude -p <project>    # paste it (or use the project's Agents page on lizard.build)
```

Or skip this: `lzs init` also takes `CLAUDE_CODE_OAUTH_TOKEN` / `ANTHROPIC_API_KEY` from your environment, and on a terminal asks you to paste one.

**4. Build your base snapshot** (once; run it again any time to rotate logins or pick up new config):

```sh
lzs init --new-project              # a project for lzs alone (recommended)
lzs init --project <name>           # or an existing project
```

`lzs init` installs tmux, Claude Code, Codex and OpenCode, copies your setup, saves it all as a snapshot and deletes the build sandbox. Every session boots from it.

**5. Go.**

```sh
lzs .                               # this repo (uncommitted work included) in a fresh sandbox
lzs . -p "add dark mode"            # ...with a task
lzs --repo owner/name               # a GitHub repo instead of a local one
```

Inside a session, `Ctrl-b d` detaches (the agent keeps working) and `Ctrl-b c` opens a shell next to it. `lzs resume` gets you back in.

## Commands

| | Command | What it does |
|---|---|---|
| **Start** | `lzs init --new-project \| --project P` | Build your base snapshot (again = rotate) |
| | `lzs [. \| DIR \| --repo owner/name] [-p "task"] [-d]` | A fresh session; `-d` starts it without attaching |
| | `lzs fan N [.] -p "task" [--check CMD]` | N agents on one task (max 50, asks above 20) |
| | `lzs group N [.] [-p "task"]` | N sessions side by side in tmux (run inside tmux) |
| **Follow** | `lzs ls [--json]` | Every session: working, your turn, asking you, paused |
| | `lzs watch` | The same, live, with a bell when one needs you |
| | `lzs peek <name>` | Its screen, without attaching |
| | `lzs send <name> "text"` / `--all "text"` | Type into the agent without attaching |
| | `lzs resume [name]` | Attach (wakes a paused one) |
| **Compare** | `lzs diff` | The last fan-out side by side, passing checks first |
| | `lzs diff <name> [--stat]` | One session's diff |
| | `lzs check [name...] [--check CMD]` | Run the check now, in all of them at once |
| | `lzs pick <name> [--yes] [--keep]` | Keep it as a branch, delete the rest |
| **Ship** | `lzs pull <name> [--files] [--to DIR]` | The work as branch `lzs/<name>` in your repo |
| | `lzs push <name>` | Push that branch and open a pull request |
| | `lzs open <name> <port>` | A public URL for a dev server in the session |
| **Tidy** | `lzs stop <name> \| --all` | Pause now (files kept, compute billing stops) |
| | `lzs rm <name> \| --all` | Delete (transcripts come back first) |
| | `lzs uninstall [--yes]` | Delete everything this machine's lzs made |

`lzs --help` has the full list of flags.

<details>
<summary><b>Flags you'll want</b> (agents, sizes, scripts, CI)</summary>

- `--agent claude|codex|opencode` on `lzs init` sets the default agent (Claude Code if you don't say); `--agents claude,codex` installs fewer.
- `--size nano|mini|small` on `lzs init` sets the sandbox size (default `nano`: ½ vCPU, 1 GB).
- `--idle-minutes 30` / `--max-hours 24` set when a session pauses itself; `--no-idle-pause` turns the idle pause off.
- `--no-local` skips copying your local agent config; `--no-sync` skips bringing transcripts back.
- `-d` starts a session without attaching. Without a terminal (a script, CI, another agent) that is what happens anyway, and `lzs fan` never attaches.
- `lzs ls --json` for scripts.
- `lzs pick <name> --yes` deletes the other attempts without asking; `--keep` keeps them. A session with no changes isn't picked without `--force`.
- `lzs pull <name> --files --to DIR` copies `/workspace` into a new, empty directory instead of making a branch (no `.git`, no links, no execute bits).
- `--include-secrets` also sends untracked files that look like secrets; `--yes` skips the question about untracked files.

</details>

## Best of N

The workflow lzs is built around: let several agents try, let your tests decide.

```sh
lzs fan 5 . -p "fix the flaky checkout test" --check "npm test"
lzs watch          # who's working, who's done, whose tests pass
lzs diff           # all five side by side, passing ones first
lzs pick lzs-3     # keep it as branch lzs/lzs-3, delete the other four
lzs push lzs-3     # ...and open a pull request
```

`--check` runs in each sandbox when its agent finishes. `lzs check` re-runs it everywhere at once, e.g. after you `lzs send --all "also handle the empty cart"`.

## Notifications

Get pinged when an agent is done or asking you something, laptop closed. The sandbox sends it itself, with the check result.

```sh
lzs init --new-project --notify https://ntfy.sh/<long-random-topic>
```

Works with [ntfy](https://ntfy.sh), Slack incoming webhooks, Discord webhooks and Telegram bot URLs (`api.telegram.org`). An ntfy topic is readable by anyone who knows its name: make it long and random.

## Pricing

A **Nano** sandbox (½ vCPU, 1 GB) costs **$0.00405 an hour** per agent. Egress is free. Paused sessions pay nothing for compute.

| 50 agents for one hour | Cost |
|---|---|
| **lzs on Lizard** (Nano) | **$0.20** |
| Vercel Sandbox (smallest: 1 vCPU / 2 GB) | $8.52 |

That's **42× cheaper** with CPU maxed out. A typical agent mostly waits on the model, which Vercel bills less for, so in practice it's about **18× cheaper**.

<sub>Fine print: Vercel figure = 50 × ($0.128 CPU + $0.042 RAM) per hour at full CPU, from Vercel's public pricing. Lizard prices are per second of sandbox life; volumes and bigger sizes cost more. The Lizard account itself has a plan (below).</sub>

### How many agents at once

**Runs on Pro ($19/mo): up to 40 agents at once on Nano.** Need more? Pro $99 runs 200+ (220).

| Plan | Agents at once (Nano) |
|---|---|
| **Pro $19/month** (or pay-as-you-go) | **40** |
| Pro $99/month | 220 |
| 7-day trial (card required) | 2, each deleted after 1 hour (see [Known limitations](#known-limitations)) |

Limits count vCPU, and a Nano session counts as 1. When you hit the limit, lzs says so (how many run, the limit) and `lzs fan` stops starting more. `lzs stop` / `lzs rm` free a slot.

## Requirements

- macOS, Linux or WSL (native Windows is not supported)
- Node.js 18 or newer, and git
- The [`lizard` CLI](https://www.npmjs.com/package/@lizard-build/cli) (`@lizard-build/cli`), signed in
- tmux on your machine only for `lzs group`
- A login for your agent: a Claude subscription token (`claude setup-token`) or Anthropic API key; an OpenAI API key or ChatGPT login for Codex; an Anthropic, OpenAI or OpenRouter key for OpenCode

## Known limitations

- **Trial accounts can't run `lzs init` yet.** lzs makes sandboxes that live until they pause, and trial sandboxes are capped at 1 hour. Add credit or start a plan first.
- **Sessions need an API-key login.** After a plain `lizard login`, starting a session fails with "`--with-token` expects a liz_ API key". Sign in with a key from `lizard keys create` ([Setup, step 2](#setup)).

## Security model

The agents run with their permission prompts **off** (Claude Code with `--dangerously-skip-permissions`, Codex with `--dangerously-bypass-approvals-and-sandbox`). Assume anything in a session, including a repository that tells the agent what to do, can run any command there as root. lzs is built so that this reaches as little of yours as possible. **Give lzs a project of its own** (`lzs init --new-project`).

<details>
<summary><b>What a session can and can't see</b></summary>

**Can see:** the agent's own login (it needs it to work, so a session can use and leak it; revoke it where you made it and run `lzs init` again to rotate); the sandbox's own Lizard key; a GitHub grant or token if you gave the session one; and, readable by root but not exported to the agent, the notification URL.

**Never in a session:** your `gh` login, your own Lizard login, your repo's `.git` directory (config, credentials in remote URLs, hooks), untracked `.env`/key files, MCP servers' `env`/`headers` (unless `--include-mcp-secrets`), Claude Code hooks and helper commands from your settings.

</details>

<details>
<summary><b>What the sandbox's Lizard key can do</b></summary>

Each session gets a Lizard API key of its own, made by Lizard, scoped to the project and deleted with the sandbox. Its watchdog uses it to pause the session. It is scoped to the **project**, not the one sandbox: a session can list the project's other sandboxes, run commands in them, restore its snapshots, and anything else a project-scoped key may do there. It cannot reach your other projects, account, billing or keys. That's why lzs wants a project of its own, and why `lzs init` makes you name one.

</details>

<details>
<summary><b>Who can open your base snapshot</b></summary>

Snapshots aren't private to you: anyone who can open the project (its members, and its workspace's) can restore the base, which holds your agent login. `lzs init` warns when the project has other members.

</details>

<details>
<summary><b>What goes up</b></summary>

For `lzs .`: tracked files as they are on disk, untracked files `.gitignore` doesn't ignore (listed, and asked about on a terminal), and the history as a git bundle (HEAD and the current branch). No push needed.

Untracked files that look like secrets (`.env*`, `.envrc`, `*.pem`, `*.key`, `id_rsa*`, `id_ed25519*`, `*.p12`, `.npmrc`, `.pypirc`, `.netrc`, `credentials*`, …) are left out unless `--include-secrets`. Secret-looking files tracked in git are sent (they're in the history anyway) and lzs warns about them. The repo's `origin` URL goes in without any credentials in it.

</details>

<details>
<summary><b>What comes back</b></summary>

Everything from a session is treated as untrusted.

- `lzs pull` fetches a branch from a git bundle; nothing runs.
- `lzs pull --files` writes regular files only, into a new, empty directory: no `.git`, no links, no execute bits, and it warns about `.envrc`.
- Transcripts are only `.jsonl`/`.json` files with plain names, written mode 600 under `~/.claude/projects` and `~/.codex/sessions`. They contain whatever the agent saw (including any secret it read), so treat them like the session itself.
- Screen text, titles, diffs, commit subjects and check output lose every terminal escape sequence except colors before they're printed.

</details>

<details>
<summary><b>GitHub</b></summary>

A `--repo owner/name` session asks Lizard for a grant to that one repository (needs GitHub connected on Lizard and Lizard's GitHub App on the repository's owner); without one it clones public repositories read-only. `--github-token` / `LZS_GH_TOKEN` gives one session a token of your own instead: the agent can read it, so make it fine-grained, for that one repository. A session on your local repo never gets GitHub credentials: `lzs push` pulls its branch here and pushes it with your own git and gh logins.

</details>

<details>
<summary><b>Notifications, billing, rotating</b></summary>

- **Notifications:** `--notify` sends the session's name, whether it's done or asking, the check result and the agent's window title (the task as the agent summarizes it), from the sandbox.
- **Billing guards:** idle sessions pause after 30 minutes, every session after 24 hours of running (`--max-hours`), and `lzs fan` stops at 50 and asks above 20. If a session's watchdog was killed, lzs pauses it when it finds it running well past its limit.
- **Rotating:** `lzs init` again builds a new base with fresh logins and deletes the old base snapshot as soon as no session uses it.
- **Removing:** `lzs uninstall` deletes every session, snapshot and key this machine's lzs made, by id, and `~/.config/lzs`.

</details>

## How it works

`lzs` drives the [`lizard` CLI](https://www.npmjs.com/package/@lizard-build/cli). A session is a new sandbox from your base snapshot (`lizard sandbox create --snapshot … --with-token`). lzs unpacks your repo on branch `lzs/<name>`, starts the agent in tmux, and opens the sandbox's terminal, which attaches to it (the terminal on lizard.build attaches to the same session). `lzs pull` commits what's there and fetches it as a git bundle. A watchdog in the sandbox pauses it once neither the agent's screen nor your keyboard has changed for the idle minutes, or after `--max-hours` of running.

Local state lives in `~/.config/lzs` (mode 700; set `LZS_HOME` to move it).

## FAQ

<details>
<summary><b>"no claude login" / a session is refused before it starts</b></summary>

Your base has no login for that agent. Agent logins connected on Lizard belong to **one project**: add it to the project lzs uses (`lizard agents add claude -p <project>`, or that project's Agents page), or set `CLAUDE_CODE_OAUTH_TOKEN` (from `claude setup-token`), then run `lzs init --project <project>` again. Note `lizard agents login` is the ChatGPT device-code sign-in for Codex, not Claude.

</details>

<details>
<summary><b>"lzs init needs the project its sandboxes live in"</b></summary>

Pass `--new-project` (recommended: a project for lzs alone) or `--project <name>`.

</details>

<details>
<summary><b>"--with-token expects a liz_ API key"</b></summary>

You're signed in to `lizard` with the browser login. Create a key with `lizard keys create lzs` and sign in with it (`lizard login --token liz_…`, or set `LIZARD_TOKEN`).

</details>

<details>
<summary><b>"Lizard refused a sandbox: limit reached"</b></summary>

You're at your plan's limit of agents at once ([table above](#how-many-agents-at-once)). `lzs ls` shows what's running; `lzs stop <name>` pauses one, `lzs rm <name>` deletes one.

</details>

<details>
<summary><b>Can I use one Claude subscription for many sessions?</b></summary>

Yes, with a token from `claude setup-token`: one long-lived token is safe for many sessions at once. Copying a normal interactive login into many machines is not.

</details>

<details>
<summary><b>Does my laptop need to stay on?</b></summary>

No. The agent runs in the sandbox. Close the lid; `lzs resume` reattaches later, and a paused session wakes in ~3 s with the conversation intact.

</details>

## Uninstall

```sh
lzs uninstall              # deletes every session, snapshot and key this machine's lzs made, plus ~/.config/lzs
npm rm -g @lizard-build/lzs
```

`lzs rm --all` deletes just the sessions and keeps your base snapshot.

## Credits

Inspired by [fxs](https://github.com/fazxes/fxs) by [@fazxes](https://github.com/fazxes) (MIT).

## License

[MIT](LICENSE)
