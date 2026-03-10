# Repository Guidelines

## Project Structure & Module Organization
This repository is a documentation-first security knowledge base in Turkish. Content is organized into two layers:

1. **Category folders** at the root contain general-purpose security guides (fundamentals, checklists, secure coding):
   - `OWASP_Top_10/` — OWASP Top 10 2025 category write-ups (A01–A10)
   - `Enjeksiyon/` — SQL Injection, Command Injection fundamentals
   - `Erisim_Kontrolu/` — Broken Access Control, IDOR guides
   - `Oturum_Yonetimi/` — CSRF, Session Management guides
   - `Arastirmalar/` — In-depth research reports and threat analyses

2. **`_TODO/`** contains all CVE-specific vulnerability analyses as detailed 8-section write-ups (48 files). Each TODO tracks a single vulnerability or risk with a checklist. Summary dashboard: `_TODO/OZET.md`.

Supporting files:
- `_Sablonlar/` — Document templates (`todo_sablonu.md`)
- `.claude/` — Claude Code commands and project configuration
- `CLAUDE.md` — Claude Code workflow instructions (research processing, TODO creation)
- `INDEX.md` — Comprehensive project index with CVE table and content map
- `README.md` — Main project description and content listing

## Build, Test, and Development Commands
There is no build system or automated test suite. Typical workflow is:
- Edit Markdown in your editor and use its preview for formatting checks.
- Manually verify links and headings for new documents.
- Optional sanity search: `rg "CVE-"` to spot new entries or typos.

## Coding Style & Naming Conventions
- Language: Turkish is the default for documentation.
- Category folder docs: `[Kategori]/[Baslik]_[Teknoloji].md`
  Example: `Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md`
- TODO files: `_TODO/TODO_[slug_snake_case].md`
  Example: `_TODO/TODO_cve_2026_22708_cursor_shell_bypass.md`
- Use clear Markdown structure: `##` for sections, short paragraphs, and bullet lists for steps.
- Prefer code fences with a language tag (e.g., ```bash, ```python) and keep examples minimal but reproducible.

## Content Template Expectations
- **TODO vulnerability analyses:** Follow `_Sablonlar/todo_sablonu.md` (8-section format: meta, summary, technical detail, PoC, risk, mitigations, code, checklist, monitoring)
- General vulnerability write-ups should follow the same 8-section structure where applicable.
- If you deviate from the template, explain why in the document intro.

## TODO Is Akisi
- Sablon: `_Sablonlar/todo_sablonu.md`
- Ozet panosu: `_TODO/OZET.md`
- Tetikleme: "Bu arastirmayi isle" veya `/arastirma-isle` komutu
- Her TODO bir zafiyet takip eder (ACIK/KAPANDI)
- Detayli is akisi: `CLAUDE.md` dosyasina bakin

## Testing Guidelines
No automated tests are defined. If you add scripts or data that require verification, document exact steps in the new file and mention any assumptions.

## Commit & Pull Request Guidelines
Git history shows short, imperative summaries and occasional type prefixes like `docs:`. Please keep commit subjects concise (50-72 chars) and include CVE IDs when relevant. For PRs, include:
- A brief scope summary (what was added/changed).
- The list of new/updated documents.
- Any sources or references used.
- Updates to `README.md` and `INDEX.md` if the index changed.
