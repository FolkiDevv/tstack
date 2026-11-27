# tstack

The agent skills I use every day, cleaned up so anyone can use them. They work
in Claude Code and Codex. Some use [T3 Code](https://github.com/pingdotgg/t3code)
features when they are available and fall back to native tools when they are
not.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `full-send` | "full send" | Takes a change to a merged PR with nobody watching: proves it, reviews it, files it, babysits it, merges it. |
| `full-stop` | "full stop" | Same as `full-send`, but stops when the PR is ready for you to merge. |
| `council` | "council" | Gets review from three models from different providers, and loops until they agree. |
| `review` | "review" | Runs `neon-loop` and `neon-security-loop` on a change at the same time, until both agree. |
| `opus-review` | "opus review" | Same as `review`, with `opus-loop` and `opus-security-loop` as the two loops. |
| `opus-loop` | "opus loop" | Reviews a change with Claude Opus 5.5 at high effort in a loop until both agents agree. |
| `opus-security-loop` | "opus security loop" | Looks for security holes a change opens, with Claude Opus 5.5 at high effort, in a loop until both agents agree. |
| `sol-loop` | "sol loop" | Reviews a change with GPT Sol in a loop until both agents agree. |
| `neon-loop` | "neon loop" | Reviews a change with Gemini Neon at medium effort in a loop until both agents agree. |
| `neon-security-loop` | "neon security loop" | Looks for security holes a change opens, with Gemini Neon at medium effort, in a loop until both agents agree. |
| `file-pr` | "file a PR" | Files a short PR that leads with the problem, not an inventory of changes. |
| `takeover` | "take over" | Finishes someone else's PR, branch, or agent thread, and keeps their credit. |
| `babysit-pr` | "babysit" | Watches a PR through CI and review bots, and fixes the real findings. |
| `modernize` | "modernize" | Rebases an old PR and decides if it should still exist. |
| `leaving-pr-comments` | (automatic) | Formats comments that an agent posts on your behalf. |

Some skills call others. Install the ones they need too:

- `full-send` uses `review`, `file-pr`, and `babysit-pr`, and `opus-review`
  for large changes.
- `full-stop` uses `full-send`, and through it `review`, `opus-review`,
  `file-pr`, and `babysit-pr`.
- `review` uses `neon-loop` and `neon-security-loop`.
- `opus-review` uses `review`, `opus-loop`, and `opus-security-loop`.
- `takeover` uses `modernize`, `file-pr`, `babysit-pr`, and `leaving-pr-comments`.
- `file-pr`, `babysit-pr`, and `modernize` use `leaving-pr-comments`.
- `babysit-pr` also uses `review`, for fixes it makes after the PR is filed.

## Install

```bash
git clone https://github.com/t3dotgg/tstack
cp -R tstack/skills/* ~/.claude/skills/   # Claude Code
cp -R tstack/skills/* ~/.codex/skills/    # Codex
```

## Requirements

- An authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- `python3` for `babysit-pr`'s `pr-wait` script.
- `council`, `sol-loop`, `neon-loop`, and `neon-security-loop` need subagents
  on other providers' models. T3 Code does this with `delegate_task`. Otherwise
  you need a harness that can run those models. The Neon loops run Gemini Neon
  through Antigravity.
- `opus-loop` and `opus-security-loop` need a subagent that runs Claude Opus
  5.5 at high reasoning effort: `delegate_task` in T3 Code, or the `Agent`
  tool in Claude Code.

## Make them yours

These are starting points. Edit them. Put personal settings, like which models
sit on your council, in your `CLAUDE.md` or `AGENTS.md`, so you can update the
skills without losing them.

`neon-loop` and `opus-loop` copy a "Review notes" section from a project's
`CLAUDE.md` or `AGENTS.md` into the reviewer's brief. Write there what a
reviewer cannot see in a diff: which processes share state, what the
platform can kill or recreate, which older code does the same job, and
which input is untrusted.

## License

[MIT](LICENSE)
