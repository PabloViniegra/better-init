# better-init

A sharper `/init` for AI coding agents. Instead of dumping a long, generic
context file, `better-init` produces a short, high-signal `CLAUDE.md` or
`AGENTS.md` — capped at 200 lines — built either from a real analysis of your
codebase or, if the project is empty, from a short adaptive interview.

## Why

The stock `/init` command tends to generate verbose, low-signal context
files that eat into the agent's context window without earning their place.
It's also close to useless on a freshly scaffolded project (Vite, Astro,
.NET, Next.js, ...) where there's no domain code to read yet.

`better-init` fixes both problems:

- **Established project** → the agent reads manifests, tool configs, and
  source layout, then writes a dense summary instead of guessing.
- **Empty / scaffold-only project** → the agent interviews you with
  adaptive, multiple-choice questions instead of producing filler content.

## Install

```bash
npx skills add PabloViniegra/better-init
```

This installs the skill into your agent's skills directory (e.g.
`.claude/skills/better-init/`).

## Usage

Once installed, ask your agent to bootstrap or regenerate its context file,
for example:

```
run better-init
```

The skill picks the right target file for the agent in use (`CLAUDE.md` for
Claude Code, `AGENTS.md` otherwise) and decides on its own whether to
analyze the codebase or interview you, based on the project's state.

## Output contract

Every generated file follows the same contract:

- Always written in English, regardless of conversation language.
- Hard limit of 200 lines — fewer is better.
- No filler, no generic best-practice advice, nothing trivially inferable
  by opening one file.
- Exactly these sections, in order: `Overview`, `Technology`, `Commands`,
  `Architecture`, `Code Style`, and an optional `Others`.

## How it works

1. **Pick the target file** — `CLAUDE.md` or `AGENTS.md` depending on the
   agent running the skill.
2. **Classify the project** — established, empty/scaffold, or conflicting
   signals (asks the user to disambiguate in that case).
3. **Established** → analyze manifests, tool configs, and source layout
   (delegated to a subagent on large repos to avoid saturating context).
   **Empty** → run a one-question-at-a-time, answer-driven interview.
4. **Verify** — enforce the line limit, diff against any existing file, and
   confirm before overwriting.

See [`skills/better-init/SKILL.md`](skills/better-init/SKILL.md) for the
full instruction set.
