# AGENTS.md

## What this repo is

Personal **Obsidian vault** for Pichyy — not a software project. No build, test, lint, or CI.

The agent's job here: **เขียน จัดหมวดหมู่ จัดเรียง ตรวจproofread อ่านโน้ต** — write, categorize, rearrange, proofread, and read notes.

## Critical layout gotcha

- Obsidian vault root is `Pichyy/` — that's where `.obsidian/` lives.
- **All notes go inside `Pichyy/`**, never at repo root. A note at `D:\Obsidian-vault\foo.md` is invisible to Obsidian.
- Git repo root is one level up (`D:\Obsidian-vault`), currently has no commits.

## Writing conventions

- Notes are in **Thai**. Keep Thai text natural; don't translate to English unless asked.
- Obsidian features in active use — use them: wiki-links `[[Note]]`, tags `#tag`, YAML frontmatter properties, callouts `> [!note]`, checkboxes, tables.
- Monospace font is set to **TH SarabunPSK** (Thai font) — avoid relying on ASCII-art alignment in code blocks.
- Core plugins enabled: daily-notes, templates, backlinks, tag-pane, bases, canvas. No community plugins.

## Verify (there are no code tests)

After writing or editing a note, re-read the file and check:

- frontmatter YAML parses (keys indented correctly, no tabs)
- wiki-links match an existing filename exactly (case-sensitive)
- callout syntax `> [!type]` on its own line, body lines prefixed with `>`
- tables have a valid `| --- |` separator row
- Thai text renders (no mojibake / stray `?` characters)

## Git

- Don't commit unless explicitly asked. Vault is synced via Obsidian Sync; treat files as user-owned.
