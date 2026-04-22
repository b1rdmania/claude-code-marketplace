---
name: readme-generator
description: Generates a complete, opinionated README.md by reading the actual project — code, config, SKILL.md, scripts, and structure. Works for software projects, Claude Code skills, and non-standard tool repos. Output is publication-ready on first pass, not a template with TODOs.
allowed-tools: ["Read", "Glob", "Grep", "Write", "Bash"]
version: 2.0.0
author: b1rdmania (forked from GLINCKER)
license: Apache-2.0
keywords: [documentation, readme, markdown, project, skills]
---

# README Generator

Reads the project, writes the README. One pass, no placeholders.

## What This Skill Does

- Detects project type: software package, CLI tool, Claude Code skill, script collection, or non-standard repo
- Reads actual code and config — not just file names
- Writes for the real audience: users installing the thing, not the person who built it
- Produces a complete README on first pass — no TODO sections, no "add your description here"
- Adapts tone and structure to project type rather than applying a generic template

## Instructions

### Step 0 — Read before writing

Before generating anything, understand what this project actually is and who will use it. Do not start with the README structure — start with the project.

Run these in parallel:
```bash
ls -la
cat package.json 2>/dev/null || cat pyproject.toml 2>/dev/null || cat Cargo.toml 2>/dev/null || cat go.mod 2>/dev/null || echo "no standard config"
```

Then:
- Glob for `SKILL.md`, `CLAUDE.md`, `*.mjs`, `*.py`, `*.ts`, `*.sh` — read the ones that explain what the project does
- Read the existing README if one exists — note what's already right and what's missing
- Check `scripts/` or `bin/` — read the entry points, not just the filenames
- Grep for the main exported function, CLI command, or slash command name

Ask: **who installs this and what do they do first?** That is the README's job.

---

### Step 1 — Classify the project

| Type | Signals | README focus |
|---|---|---|
| **Claude Code skill** | `SKILL.md` present, slash command defined | Installation, command syntax, what it does vs. doesn't do |
| **CLI tool** | `bin/` entry, `#!/usr/bin/env` shebang | Install command, usage with flags, example output |
| **npm package** | `package.json` with `main`/`exports` | Install, API surface, quick example |
| **Python package** | `pyproject.toml` or `setup.py` | pip install, usage, environment setup |
| **Script collection** | Multiple `.mjs`/`.py`/`.sh`, no package | What each script does, how to run, dependencies |
| **Non-standard repo** | None of the above | Describe what's in it, how to navigate it |

If `SKILL.md` is present: read it fully. The README should accurately reflect the skill's commands, tools, and scope — it is the public face of the skill.

---

### Step 2 — Write the README

Structure adapts by project type. Use only the sections that apply.

---

**`# [Project Name]`**

One sentence. What it does and who it's for. Not "a tool that helps you" — just what it does.

Good: `Generates pre-call intelligence briefs on founder-operated businesses using Perplexity deep research and Companies House data.`
Bad: `A powerful CLI tool that helps developers streamline their workflow.`

---

**`## Install`** (if installable)

Exact commands. No explanation unless something is non-obvious.

For a Claude Code skill:
```bash
cp -r skills/readme-generator ~/.claude/skills/
# or symlink for development
ln -s $(pwd)/skills/readme-generator ~/.claude/skills/readme-generator
```

For npm:
```bash
npm install @scope/package
```

For pip:
```bash
pip install package-name
```

---

**`## Usage`**

Show the real thing, not a toy example. If it's a slash command, show the actual command and what the output looks like. If it's a script, show the real invocation with realistic arguments.

For Claude Code skills, show:
- The slash command and any flags
- A concrete example with realistic input
- What the output looks like

---

**`## What it does`**

For non-obvious projects: a short list of what happens when you run it. Bullet points, active verbs, specific not vague.

Skip this section for simple tools where usage makes it obvious.

---

**`## What it doesn't do`**

One of the most useful sections. Prevents the wrong people from installing it and sets accurate expectations for the right ones. Include whenever there are likely misconceptions.

Examples:
- Does not send emails or make external API calls on your behalf
- Does not analyse code logic — only file structure and config
- Requires Perplexity MCP to be configured — will not work without it

---

**`## Requirements`**

Only list things that aren't obvious or aren't automatically installed. If it's just Node 18+, skip it. If it needs a specific MCP server or API key configured, say so explicitly — name the variable, show the format.

---

**`## Configuration`** (if applicable)

Environment variables, settings files, or one-time setup steps. Be specific — name the exact variable, show the format.

```bash
PERPLEXITY_API_KEY=pplx-...
NOTION_TOKEN=secret_...
```

---

**`## Project structure`** (for non-standard repos)

A short annotated tree showing what's where and why. Not exhaustive — just enough to navigate.

```
scripts/          # zero-dependency .mjs scripts, run directly with node
schemas/          # JSON schemas for intake and audit data
clients/          # per-client folders (gitignored)
skills/           # Claude Code skills for the pipeline
```

---

**`## Contributing`** (optional)

Only include if contributions are genuinely welcome and you have a process. Skip the boilerplate "open an issue or PR" if you don't actually mean it.

---

**`## License`**

One line.

---

### Step 3 — Voice rules

Apply throughout. Non-negotiable.

- **Orwell rules**: no word that can be cut, no passive where active works
- **No jargon without a referent**: "MCP server" is fine, "leveraging synergistic tooling" is not
- **No emojis** unless the user explicitly asks
- **Banned words**: "simply", "easily", "just", "powerful", "robust", "seamless", "streamline", "leverage", "comprehensive" (as a filler adjective)
- **No marketing tone**: write for the person installing it, not for a product page
- **Present tense, declarative sentences**
- **Code blocks for all commands** — never inline a command without a code block

---

### Step 4 — Output

1. Show the generated README in the conversation
2. Ask: "Write to README.md?" — if one already exists, confirm before overwriting
3. If the user approves, write it
4. Offer one round of targeted edits if something doesn't look right

Do not add TODO placeholders. If a section genuinely can't be filled from the project files, skip it or ask the user one specific question to fill the gap.

---

## Error handling

**No config files found**: Read source files directly — don't give up. Glob for `.md`, `.mjs`, `.py`, `.sh` and read the most likely entry point.

**Existing README**: Read it first. Preserve anything that looks intentional. Don't overwrite without confirming.

**SKILL.md present but incomplete**: Generate from what's there, flag the gap to the user rather than inventing content.

**Can't determine project type**: Ask one question — "What does someone do first after cloning this?" — then proceed.

---

## Limitations

- Cannot generate screenshots or demo GIFs
- Does not run the project to verify commands work — review before publishing
- For large codebases, reads entry points and config only, not every file

---

## Changelog

### 2.0.0
- Forked from GLINCKER/claude-code-marketplace
- Added Claude Code skill detection and SKILL.md cross-referencing
- Added script collection and non-standard repo project types
- Rewrote voice rules — Orwell, no marketing filler, banned words list
- Output is complete on first pass — no TODO placeholders
- Added "What it doesn't do" section pattern
- Removed generic boilerplate sections

### 1.0.0 (GLINCKER original)
- Initial release
- Python, Node.js, Rust, Go support
