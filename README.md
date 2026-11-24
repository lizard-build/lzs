# lzs

Coding agents in the cloud, one command per session. Run 50 at once without melting your laptop.

```sh
npm i -g @lizard-build/cli @lizard-build/lzs
lizard login
lzs init --new-project           # once: your base snapshot (agent logins, skills, MCPs)

lzs .                            # this repo, uncommitted work included, in a fresh sandbox

# best of 5: five agents on one task, your tests run in each when it finishes
lzs fan 5 . -p "fix the flaky checkout test" --check "npm test"
lzs watch                        # who's working, who's done, whose tests pass
lzs diff                         # the five side by side, passing ones first
lzs pick lzs-3                   # keep that one as branch lzs/lzs-3, delete the other four
```

Each session is its own [Lizard](https://lizard.build) sandbox (read the [Security model](#security-model) first):

- **ready in a few seconds** with the agent already running: Claude Code, Codex or OpenCode
  (all three are installed in the base; each needs its own login)
- **your repo goes in as it is**: history (as a git bundle), branch, uncommitted and untracked
  files, minus anything that looks like a secret (`.env`, keys). No push needed
- **the work comes back as a git branch** (`lzs pull`), or as a pull request (`lzs push`)
- **best of N**: `--check "npm test"` runs your tests in each sandbox when its agent finishes;
  `lzs diff` ranks the attempts, `lzs pick` keeps the winner
- **your phone buzzes when an agent is done**, laptop closed: `lzs init --notify <ntfy, Slack,
  Discord or Telegram URL>`; the sandbox itself sends it, with the check result
- **no token to paste** if you connected Claude on Lizard (`lizard agents login claude`)
- **your setup comes along**: `~/.claude` CLAUDE.md, skills, agents, commands, settings; MCP
  servers; `~/.agents/skills`; Codex's AGENTS.md, config and skills; OpenCode's config (secrets and
  local commands stripped, see below; `--no-local` to skip)
- **the conversations come back**: transcripts land in `~/.claude/projects` and `~/.codex/sessions`
  on `lzs pull`, `stop`, `rm` and `pick`, for review or training (`--no-sync` to skip)
- **keeps going when your laptop sleeps**: the agent runs in tmux in the sandbox
- **pauses itself when idle** (30 min) or after `--max-hours` (24 h) of running: compute
  billing stops, files and the conversation stay
- **wakes in ~3 s** and continues the conversation: `lzs resume`
- **follow 50 agents from one terminal**: `lzs watch`, `lzs peek`, `lzs send`

## Cost

A Nano sandbox (½ vCPU, 1 GB) is $0.00405 an hour. Fifty agents around the clock for a month:
about $148, and less, because idle ones pause themselves. Egress is free.

## Commands

```
Start
  lzs init --new-project | --project P [--agent claude|codex|opencode] [--agents LIST]
           [--size nano|mini|small] [--notify URL] [--idle-minutes 30] [--max-hours 24]
           [--no-local] [--include-mcp-secrets] [--no-sync]  build your base snapshot (again: rotate)
  lzs [. | DIR | --repo owner/name] [-p "task"] [-d]         a fresh session (. = this repo)
  lzs fan N [. | DIR | --repo owner/name] -p "task" [--check CMD]
                                                             N agents on one task (at most 50; asks
                                                             above 20); CMD runs in each when done
  lzs group N [.] [-p "task"]                                N sessions side by side (inside tmux)

Follow
  lzs ls [--json]                                            every session: working, your turn, asking
  lzs watch                                                  the same, live; a bell when one needs you
  lzs peek <name>                                            its screen, without attaching
  lzs send <name> "text" | --all "text"                      type into the agent without attaching
  lzs resume [name]                                          attach (wakes a paused one)

Compare
  lzs diff                                                   the last fan-out side by side
  lzs diff <name> [--stat]                                   one session's diff
  lzs check [name...] [--check CMD]                          run the check now, everywhere at once
  lzs pick <name> [--yes] [--keep]                           keep it (as a branch), delete the rest

Ship
  lzs pull <name> [--files] [--to DIR]                       the work as branch lzs/<name> in your repo
                                                             (--files: into a new, empty directory)
  lzs push <name>                                            push that branch and open a pull request
  lzs open <name> <port>                                     a URL for a dev server in the session

Tidy
  lzs stop <name> | --all                                    pause now (files kept, billing stops)
  lzs rm <name> | --all                                      delete
  lzs uninstall [--yes]                                      delete everything lzs made (below)
```

Inside a session: `Ctrl-b d` detaches (the agent keeps working), `Ctrl-b c` opens a shell next to it.
The terminal on lizard.build attaches to the same session.

```
$ lzs ls
lzs-1     working     claude           shop · Fix flaky checkout test · +40 −12
lzs-2     your turn   claude   ✓ check shop · Fix flaky checkout test · +18 −3 · idle 4m
lzs-3     your turn   claude   ✗ check shop · Fix flaky checkout test · +95 −40
lzs-4     paused      claude           shop · Add retry to payment webhook

4 sessions · 3 running · ~$0.012/h (nano) · 2 waiting for you
```

## Flags you'll want (scripts, CI, first run)

- `-p "task"` gives the agent its task; `lzs fan` requires it (a task without `-p` is an error).
- `-d` starts a session without attaching. Without a terminal (a script, CI, another agent) that
  is what happens anyway; `lzs fan` never attaches. Then `lzs peek <name>` shows the screen and
  `lzs send <name> "…"` types into it.
- `lzs ls --json` for scripts.
- `lzs pick <name> --yes` deletes the other attempts without asking (without a terminal and
  without `--yes` they are kept); `--keep` keeps them. A session with no changes is not picked
  without `--force`, and never deletes the others.
- `lzs pull <name> --files --to DIR` copies `/workspace` into a new or empty DIR instead of making
  a branch: no `.git`, no links, no execute bits.
- `--include-secrets` sends untracked files that look like secrets; `--yes` skips the question
  about untracked files; `lzs fan` above 20 needs `--yes` without a terminal.
- `--max-hours H` (default 24) pauses a session after H hours of running, even if the agent keeps
  busy; `--idle-minutes 0` is refused, `--no-idle-pause` says you mean it.
- `lzs rm --all` deletes the sessions (their transcripts come back first) but keeps your base
  snapshot. `lzs uninstall` deletes that too, plus `~/.config/lzs` (`--yes` to skip the question).
  It deletes only what this machine's lzs made, by id.
- When Lizard's sandbox limit is reached, lzs says so (how many run, the limit) and `lzs fan`
  stops starting more; `lzs stop` / `lzs rm` free a slot.

## Signing in

`lzs init` keeps the agents' logins in your base snapshot (`/root/.lzs/agent.env`). It installs
Claude Code, Codex and OpenCode (`--agents claude,codex` for fewer) and uses whatever logins it
finds, in order:

- **Connected on Lizard**: `lizard agents login claude` (or a key on the project's Agents page).
  Nothing to paste.
- **Your environment**: `CLAUDE_CODE_OAUTH_TOKEN` (from `claude setup-token`; one long-lived
  token is safe for many sessions at once, copying a normal login is not) or `ANTHROPIC_API_KEY`;
  `OPENAI_API_KEY` for Codex; `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `OPENROUTER_API_KEY` for
  OpenCode (a Claude subscription token does not work for OpenCode).
- **By hand**: on a terminal, `lzs init` opens the base's shell to sign in.

A session of an agent with no login in the base is refused before anything starts. An agent that
exits as soon as it starts is reported as a failure and its sandbox deleted.

**GitHub**: a `--repo owner/name` session asks Lizard for a grant to that one repository (needs
GitHub connected on Lizard and Lizard's GitHub App on the repository's owner); without one it
clones public repositories read-only. `--github-token` / `LZS_GH_TOKEN` gives one session a token
of your own instead: the agent can read it, so make it fine-grained, for that one repository. A
session on your local repo never gets GitHub credentials: `lzs push` pulls its branch here and
pushes it with your own git and gh logins.

## How it works

`lzs` drives the [`lizard` CLI](https://www.npmjs.com/package/@lizard-build/cli). `lzs init`
creates a sandbox in the project you name, installs tmux and the agents, copies your setup, saves
it as a snapshot and deletes the sandbox. A session is a new sandbox from that snapshot, made with
`lizard sandbox create --snapshot … --with-token`: Lizard gives it an API key of its own, scoped to
the project and deleted with the sandbox, which its watchdog uses to pause it. lzs unpacks your repo
on branch `lzs/<name>`, starts the agent in tmux, and opens the sandbox's terminal, which attaches
to it. `lzs pull` commits what is there and fetches it as a git bundle, so it lands in your repo
as a normal branch. The watchdog pauses the sandbox once neither the agent's screen nor your
keyboard has changed for the idle minutes, or after `--max-hours` of running; lzs itself pauses a
session it finds running well past that (if the watchdog was killed).

Local state lives in `~/.config/lzs` (`LZS_HOME` to move it), mode 700.

## Security model

The agents run with their permission prompts off: Claude Code with
`--dangerously-skip-permissions`, Codex with `--dangerously-bypass-approvals-and-sandbox`. Assume
anything in a session (the agent, or a repository it reads that tells it what to do) can run any
command there as root. lzs is built so that this reaches as little of yours as possible.

**What a session can see.** The agent's own login (`CLAUDE_CODE_OAUTH_TOKEN`, an API key): it
has to, to work, so a session can use and leak it; revoke it where you made it if in doubt, and
`lzs init` again to rotate. The sandbox's Lizard key (`~/.lizard/config.json`), and a GitHub
grant or token if you gave the session one (above). Not exported to the agent, but readable by
root in the sandbox: the notification URL. Never in a session: your `gh` login, your Lizard login,
your repo's `.git` directory (config, credentials in remote URLs, hooks), untracked `.env`/key
files, MCP servers' `env`/`headers` (unless `--include-mcp-secrets`), Claude Code hooks and helper
commands from your settings.

**What the sandbox's Lizard key can do.** It is scoped to the project, not to the one sandbox:
with it, a session can list the project's other sandboxes and run commands in them, restore its
snapshots, and anything else a project-scoped key may do there. It cannot reach your other
projects, account, billing or keys, and it is deleted with the sandbox. So give lzs a project of
its own: `lzs init --new-project` makes `lzs-<you>`, and `lzs init` requires you to name one.

**Who can open your base snapshot.** Snapshots are not private to you: anyone who can open the
project (its members, and its workspace's) can restore the base, which holds your agent login.
`lzs init` warns when the project has other members.

**What goes up.** For `lzs .`: tracked files as they are on disk, untracked files `.gitignore`
doesn't ignore (listed, and asked about on a terminal), and the history as a git bundle (HEAD and
the current branch). Untracked files that look like secrets (`.env*`, `.envrc`, `*.pem`, `*.key`,
`id_rsa*`, `id_ed25519*`, `*.p12`, `.npmrc`, `.pypirc`, `.netrc`, `credentials*`, …) are left out
unless `--include-secrets`; secret-looking files tracked in git are sent (they are in the history)
and lzs warns about them. The repo's `origin` URL goes in without any credentials in it.

**What comes back.** Everything from a session is treated as untrusted. `lzs pull` fetches a
branch from a git bundle (nothing runs). `lzs pull --files` writes regular files only into a new,
empty directory: no `.git`, no links, no execute bits, and it warns about `.envrc`. Transcripts
are only `.jsonl`/`.json` files with plain names, written mode 600 under `~/.claude/projects` and
`~/.codex/sessions`; they contain whatever the agent saw (including any secret it read), so treat
them like the session itself. Screen text, titles, diffs, commit subjects and check output lose
every terminal escape sequence except colors before they are printed.

**Notifications.** `--notify` sends the session's name, whether it is done or asking, the check
result and the agent's window title (the task, as the agent summarizes it) from the sandbox. An
ntfy topic is public to anyone who knows its name: pick a long random one, or use Slack, Discord
or Telegram.

**Billing.** Idle sessions pause after 30 minutes, every session after 24 hours of running
(`--max-hours`), and `lzs fan` stops at 50 and asks above 20.

**Rotating and removing.** `lzs init` again builds a new base with fresh logins and deletes the
old base snapshot (and lzs 0.4's project key) as soon as no session uses it. `lzs uninstall`
deletes every session, snapshot and key this machine's lzs made, by id, and `~/.config/lzs`.

## License

MIT
