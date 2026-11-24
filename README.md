# lzs

Coding agents in the cloud, one command per session. Run 50 at once without melting your laptop.

```sh
npm i -g @lizard-build/cli @lizard-build/lzs
lizard login
lzs init                         # once: your base snapshot (agent login, skills, MCPs)

lzs .                            # this repo, uncommitted work included, in a fresh sandbox
lzs fan 10 . -p "fix the flaky checkout test"   # ten agents on it, in the background
lzs watch                        # who's working, who's done, who's asking; a bell when one needs you
lzs pull lzs-3                   # the winner's work, back here as branch lzs/lzs-3
```

Each session is its own [Lizard](https://lizard.build) sandbox:

- **ready in ~3 s** with the agent already running: Claude Code, Codex or OpenCode
- **your repo goes in as it is**: history, branch, uncommitted and untracked files. No push needed
- **the work comes back as a git branch** (`lzs pull`), or as a pull request (`lzs push`)
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
           [--idle-minutes 30] [--project P] [--no-local]    build your base snapshot (once)
  lzs [. | DIR | --repo owner/name] [-p "task"] [-d]         a fresh session (. = this repo)
  lzs fan N [. | --repo owner/name] -p "task"                N agents on one task, in the background
  lzs group N [.] [-p "task"]                                N sessions side by side (inside tmux)

Follow
  lzs ls                                                     every session: working, your turn, asking
  lzs watch                                                  the same, live; a bell when one needs you
  lzs peek <name>                                            its screen, without attaching
  lzs send <name> "text" | --all "text"                      type into the agent without attaching
  lzs resume [name]                                          attach (wakes a paused one)

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
lzs-1     working     claude   shop · Fix flaky checkout test · 2 commits
lzs-2     your turn   claude   shop · Add retry to payment webhook · 3 changed · idle 4m
lzs-3     asking you  codex    shop · Migrate orders table
lzs-4     paused      claude   shop · write tests

4 sessions · 3 running · ~$0.012/h (nano) · 2 waiting for you
```

## Signing in

`lzs init` asks once and keeps the answers in your private base snapshot:

- **Claude Code**: run `claude setup-token` on your machine and paste the token (or set
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
