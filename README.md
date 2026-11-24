# lzs

Run coding agents in the cloud, one command per session.

```sh
npm i -g @lizard-build/cli @lizard-build/lzs
lizard login
lzs init     # once: builds your base snapshot
lzs          # a fresh sandbox with Claude Code running, and you're in it
```

Each session is its own [Lizard](https://lizard.build) sandbox:

- **starts in ~2.5 s** from your base snapshot, with the agent already running
- **Claude Code, Codex or OpenCode** (`--agent codex`), signed in, permissions mode on
- **your setup comes along**: `~/.claude` CLAUDE.md, skills, agents, commands and MCP servers
- **keeps going when your laptop sleeps**: the agent runs in tmux in the sandbox
- **pauses itself when idle** (30 min by default): compute billing stops, files and history stay
- **wakes in ~2.5 s** and continues the last conversation: `lzs resume`
- **side by side**: `lzs group 4` opens four sessions in tmux panes

## Cost

A Nano sandbox (½ vCPU, 1 GB) is $0.00405 an hour. Fifty agents running around the clock
for a month: about $148. A paused session costs only its disk.

## Commands

```
lzs init [--agent claude|codex|opencode] [--size nano|mini|small] [--project P]
         [--idle-minutes 30] [--no-local]       build the base snapshot (run once, or to update it)
lzs [--agent A] [--name N] [-- agent args]      new session: sandbox + agent, attached
lzs resume [name]                               wake a paused session and attach (default: newest)
lzs ls                                          sessions and their status
lzs stop <name>... | --all                      pause now
lzs rm <name>... | --all                        delete sessions
lzs pull <name> [--to DIR]                      copy /workspace + the agent's history here
lzs group [N]                                   N sessions side by side (inside tmux)
```

Inside a session: `Ctrl-b d` detaches (the agent keeps working), `Ctrl-b c` opens another shell.
The terminal on lizard.build attaches to the same session.

## Signing in the agent

`lzs init` asks for a token once and keeps it in your private base snapshot:

- **Claude Code**: run `claude setup-token` on your machine and paste the token
  (or set `CLAUDE_CODE_OAUTH_TOKEN` / `ANTHROPIC_API_KEY` before `lzs init`).
  One long-lived token is safe for many sessions at once; copying a normal login is not,
  because sessions refreshing the same login sign each other out.
- **Codex**: `OPENAI_API_KEY`, or sign in by hand when `lzs init` opens the base shell.
- **OpenCode**: a provider key, or `opencode auth login` in the base shell.

## How it works

`lzs` drives the [`lizard` CLI](https://www.npmjs.com/package/@lizard-build/cli). `lzs init`
creates a sandbox from the `claude` template, installs tmux and the agent, copies your setup,
saves it as a private snapshot and deletes the sandbox. `lzs` restores that snapshot, starts
the agent in a tmux session, and opens the sandbox's terminal, which attaches to it. A small
watchdog in each sandbox pauses it, with a key scoped to the project, once neither the
agent's screen nor your keyboard has changed for `LZS_IDLE_MINUTES`.

Local state lives in `~/.config/lzs` (`LZS_HOME` to move it).

## License

MIT
