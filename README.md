# pi-skills

Cross-harness skill discovery for the [pi coding agent](https://github.com/badlogic/pi-mono).

Most of your team's skills live where other agent harnesses put them —
`.claude/skills`, `.codex/skills`, `.opencode/skills`, project and user
scope, plus `~/.config/opencode/skills`. This extension teaches pi to
find, read, and run them.

## What it adds

| Tool | Purpose |
|---|---|
| `get_available_skills` | List/rank discovered skills (metadata is a routing hint, not an instruction) |
| `read_skill_file` | Read a file inside a discovered skill; paths are contained to the skill directory |
| `run_skill_script` | Run a registered executable script inside a skill, confined and shell-free |

It also injects an `<available-skills>` catalog into the system prompt at
each run start, and invalidates its skill cache on compaction.

## Why not just use `pi-agent-skills`?

This project is a derivative of
[`pi-agent-skills`](https://www.npmjs.com/package/pi-agent-skills) 0.2.3
(MIT) — see [`NOTICE`](./NOTICE) — with one deliberate divergence: we
removed its regex risk scanner and the interactive **"Approve skill
script"** confirmation. In daily use that gate fires on any script
containing the word `curl` or `export` — false positives on your own,
audited skills — and it re-prompts on every single run with no memory.
It also *fails closed* in headless sessions, silently blocking helper
agents from running perfectly ordinary scripts.

We kept the containment that actually limits blast radius and removed the
content guessing:

- scripts run only from the skill's own registry (discovered under the skill directory)
- paths are contained lexically **and** by realpath — no traversal out of the skill
- no shell: scripts spawn as `argv`, stdin is `/dev/null`
- a hard timeout (SIGKILL) and a 64 KiB output cap
- control characters and forged HTML tags in skill metadata are sanitized before they reach the prompt

If you want an approval gate, upstream is the project to use.

## Install

```sh
pi install git:github.com/TESD-Tech/pi-skills
```

Or try it once without installing:

```sh
pi -e /path/to/pi-skills/index.ts
```

> If you previously installed `pi-agent-skills`, uninstall it first
> (`pi uninstall npm:pi-agent-skills`) so the two do not register
> duplicate tools.

## Configuration (env)

| Variable | Default | Meaning |
|---|---|---|
| `PI_SKILLS_ROOTS` | built-in roots | Colon-separated list replacing the default skill roots |
| `PI_SKILLS_SCRIPT_TIMEOUT_MS` | `30000` | Kill time for `run_skill_script` |
| `PI_SKILLS_DEBUG` | off | `1/true/yes/on` logs discovery to stderr |

## Skill format

A skill is a directory containing `SKILL.md` with YAML-ish frontmatter:

```markdown
---
name: my-skill
description: What this skill does and when to use it.
trigger: optional, short trigger phrase for ranking
disable-model-invocation: true   # optional; hides from catalog, still readable by name
---

Instructions for the agent...
```

Executable files anywhere under the skill directory (shebang scripts,
`+x` ELF/Mach-O binaries) are registered as runnable scripts and listed
when the agent reads the `SKILL.md`.

## License

MIT — see [`LICENSE`](./LICENSE) and [`NOTICE`](./NOTICE). Derived from
`pi-agent-skills` Copyright (c) 2026 metalbolicx.
