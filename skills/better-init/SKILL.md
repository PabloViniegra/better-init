---
name: better-init
description: Improved /init. Generates a lean, high-signal CLAUDE.md or AGENTS.md (max 200 lines, always English). Analyzes the codebase when the project is established; interviews the user when the project is empty or an untouched starter template. Use when the user wants to bootstrap or regenerate an agent context file.
---

# Better Init

Generate the agent context file for this project. The goal is a short, dense file that
prevents context-window saturation — never a long generic document.

## Step 1 — Pick the target file

- Running inside Claude Code → `CLAUDE.md`.
- Any other agent → `AGENTS.md`.

## Step 2 — Classify the project

Scan the repo: manifests, source tree, README, git log (if any). Decide between:

- **ESTABLISHED** — real domain code exists and the project purpose is inferable. → Step 3A.
- **EMPTY** — nothing, or an untouched archetype: a fresh scaffold (Vite, Astro, .NET,
  Next.js, Spring Initializr, etc.) containing only generated boilerplate. Signals: default
  README, only scaffold files, no domain-specific names, trivial git history. → Step 3B.
- **Signals conflict** (e.g. a monorepo mixing scaffold and established packages) — don't
  guess, ask the user which situation applies.

## Step 3A — Established project: analyze, then write

Gather evidence before writing. Do not guess what you can read:

1. Manifests (`package.json`, `*.csproj`, `pyproject.toml`, `go.mod`, `Cargo.toml`, ...)
   → technologies, scripts, dependencies.
2. Tool configs (`tsconfig`, `biome.json`, ESLint/Prettier, `.editorconfig`, formatter/linter configs, CI
   workflows) → commands and style rules actually enforced.
3. Source layout and a sample of key files → architecture, patterns, naming conventions.

On a large repo, don't read all of this inline: spawn an Explore subagent to do steps 1-3
and return a summary. Saturating the main context window here defeats the point of this
skill.

Then write the file following the Output rules.

## Step 3B — Empty project: interview, then write

You know nothing about the project, so interview the user. If a `grill-me` (or similar
interviewing) skill is available, use it. Otherwise run your own interview:

- Ask **one question at a time**, multiple-choice, mutually exclusive options.
  - AskUserQuestion tool available: use it, and don't add a manual "Other" — the tool
    appends free-text "Other" on its own.
  - Tool not available: fall back to plain text and end the list yourself with
    `Other (write your own answer)`.
- **Each answer shapes the next question.** Example: area = Frontend → next question offers
  frontend frameworks; area = Data → next question offers Python/Spark/dbt-style options.
- Skip anything the scaffold already answers (a fresh Astro repo fixes language and framework
  — do not ask).
- Be exhaustive: stop only when you can fill every section below with confidence. Cover at
  minimum:
  1. What the project is and who it is for (purpose).
  2. Area: Frontend / Backend / Fullstack / Data / Mobile / Other.
  3. Language and framework (options adapted to the area).
  4. Package manager and build tooling.
  5. Testing tools and approach.
  6. Architecture style (e.g. hexagonal, layered, feature folders, MVC — adapted to area).
  7. Code style: linting, formatting, strict conventions.
  8. Anything else worth recording (deploy target, CI, constraints).

Then write the file following the Output rules, combining answers with whatever the
archetype already provides.

## Output rules (both situations)

- **Always in English**, regardless of the conversation language.
- **Hard limit: 200 lines.** Fewer is better. Every line must earn its place.
- No filler, no generic best-practice advice, nothing the agent can trivially infer by
  opening one file.
- Exactly these sections, in this order. `Others` is the one optional section — omit it
  rather than pad it with a generic "nothing else to add"; a filler section is exactly the
  noise this skill exists to avoid.

```markdown
## Overview
What the project is. Answers "what does it consist of?" in 2–4 lines.

## Technology
The technologies it uses.

## Commands
Essential commands: run/start, build, lint, format, test.

## Architecture
Software architecture, design patterns, how the code is organized.

## Code Style
How the code is written, conventions followed, strict rules if any.

## Others
Anything else worth mentioning.
```

## Step 4 — Verify, then write

1. Draft the content per the Output rules above.
2. Count lines. If over 200, cut the lowest-signal content first — trim `Others`, shorten
   verbose bullets, drop restated defaults — and recount. Do not write while over the limit.
3. If the target file already exists, diff the draft against it, show the user a short
   summary of what would change, and wait for confirmation before overwriting.
4. Write the file.
5. Report the final line count and a one-line summary per section.
