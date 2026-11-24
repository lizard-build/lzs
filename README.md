# lzs

Coding agents in the cloud, one command per session. Run 50 at once without melting your laptop.

```sh
npm i -g @lizard-build/cli @lizard-build/lzs
lizard login
lzs init                         # once: your base snapshot (agent login, skills, MCPs)

lzs .                            # this repo, uncommitted work included, in a fresh sandbox

# best of 5: five agents on one task, your tests run in each when it finishes
lzs fan 5 . -p "fix the flaky checkout test" --check "npm test"
lzs watch                        # who's working, who's done, whose tests pass
lzs diff                         # the five side by side, passing ones first
lzs pick lzs-3                   # keep that one as branch lzs/lzs-3, delete the other four
```

Each session is its own [Lizard](https://lizard.build) sandbox:

- **ready in ~3 s** with the agent already running: Claude Code, Codex or OpenCode
- **your repo goes in as it is**: history, branch, uncommitted and untracked files. No push needed
- **the work comes back as a git branch** (`lzs pull`), or as a pull request (`lzs push`)
- **best of N**: `--check "npm test"` runs your tests in each sandbox when its agent finishes;
  `lzs diff` ranks the attempts, `lzs pick` keeps the winner
- **your phone buzzes when an agent is done**, laptop closed: `lzs init --notify <ntfy, Slack,
  Discord or Telegram URL>`; the sandbox itself sends it, with the check result
- **no token to paste** if you connected Claude on Lizard (`lizard agents login claude`)
- **your setup comes along**: `~/.claude` CLAUDE.md, skills, agents, commands, MCP servers
- **keeps going when your laptop sleeps**: the agent runs in tmux in the sandbox
- **pauses itself when idle** (30 min): compute billing stops, files and the conversation stay
- **wakes in ~3 s** and continues the conversation: `lzs resume`
- **follow 50 agents from one terminal**: `lzs watch`, `lzs peek`, `lzs send`

## Cost

A Nano sandbox (½ vCPU, 1 GB) is $0.00405 an hour. Fifty agents around the clock for a month:
about $148, and less, because idle ones pause themselves. Egress is free.

## Commands

```
Start
  lzs init [--agent claude|codex|opencode] [--size nano|mini|small] [--github]
           [--notify URL] [--idle-minutes 30] [--project P]  build your base snapshot (once)
  lzs [. | DIR | --repo owner/name] [-p "task"] [-d]         a fresh session (. = this repo)
  lzs fan N [. | --repo owner/name] -p "task" [--check CMD]  N agents on one task; CMD (your tests)
                                                             runs in each one when it finishes
  lzs group N [.] [-p "task"]                                N sessions side by side (inside tmux)

Follow
  lzs ls                                                     every session: working, your turn, asking
  lzs watch                                                  the same, live; a bell when one needs you
  lzs peek <name>                                            its screen, without attaching
  lzs send <name> "text" | --all "text"                      type into the agent without attaching
  lzs resume [name]                                          attach (wakes a paused one)

Compare
  lzs diff                                                   the last fan-out side by side
  lzs diff <name> [--stat]                                   one session's diff
  lzs check [name...] [--check CMD]                          run the check now, everywhere at once
  lzs pick <name>                                            keep it (as a branch), delete the rest

Ship
  lzs pull <name>                                            the work as branch lzs/<name> in your repo
  lzs push <name>                                            push that branch and open a pull request
  lzs open <name> <port>                                     a URL for a dev server in the session

Tidy
  lzs stop <name> | --all                                    pause now (files kept, billing stops)
  lzs rm <name> | --all                                      delete
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

## Signing in

`lzs init` keeps the agent's login in your private base snapshot. In order:

- **Connected on Lizard**: if you connected Claude (or an API key) for the project with
  `lizard agents login claude` or on the Agents page, `lzs init` takes it from there. Nothing to paste.
- **Claude Code**: otherwise run `claude setup-token` on your machine and paste the token (or set
  `CLAUDE_CODE_OAUTH_TOKEN` / `ANTHROPIC_API_KEY`). One long-lived token is safe for many sessions
  at once; copying a normal login is not, because sessions refreshing it sign each other out.
- **Codex**: `OPENAI_API_KEY`, or sign in by hand when `lzs init` opens the base shell.
- **OpenCode**: a provider key, or `opencode auth login` in the base shell.
- **GitHub** (optional, `--github`): your `gh` login, so sessions can clone private repos with
  `--repo` and `lzs push` can open pull requests. Without it, `lzs .` and `lzs pull` still work.

## How it works

`lzs` drives the [`lizard` CLI](https://www.npmjs.com/package/@lizard-build/cli). `lzs init`
creates a sandbox, installs tmux and the agent, copies your setup, saves it as a private snapshot
and deletes the sandbox. A session restores that snapshot, unpacks your repo on branch
`lzs/<name>`, starts the agent in tmux, and opens the sandbox's terminal, which attaches to it.
`lzs pull` commits what is there and fetches it as a git bundle, so it lands in your repo as a
normal branch. A small watchdog in each sandbox pauses it, with a key scoped to the project, once
neither the agent's screen nor your keyboard has changed for `LZS_IDLE_MINUTES`.

Local state lives in `~/.config/lzs` (`LZS_HOME` to move it).

## License

MIT
