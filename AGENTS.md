# Repository Guidelines

## Project Structure & Module Organization
This repository is a documentation-first security knowledge base. Content is organized by topic folder at the root (e.g., `Enjeksiyon/`, `Erisim_Kontrolu/`, `Oturum_Yonetimi/`, `OWASP_Top_10/`, `Isletim_Sistemi/`, `Donanim/`, `Mobil/`, `Guvenlik_Yapilandirmasi/`, `Arastirmalar/`). Reusable templates live in `_Sablonlar/` (see `_Sablonlar/guvenlik_acigi_prompt_sablonu.md`). `README.md` serves as the main index—update it when you add or rename documents.

## Build, Test, and Development Commands
There is no build system or automated test suite. Typical workflow is:
- Edit Markdown in your editor and use its preview for formatting checks.
- Manually verify links and headings for new documents.
- Optional sanity search: `rg "CVE-"` to spot new entries or typos.

## Coding Style & Naming Conventions
- Language: Turkish is the default for documentation.
- File naming pattern (from existing content): `[Kategori]/[Baslik]_[Teknoloji].md`.
  Example: `Enjeksiyon/MySQL2_Tip_Manipulasyonu_NodeJS.md`.
- Use clear Markdown structure: `##` for sections, short paragraphs, and bullet lists for steps.
- Prefer code fences with a language tag (e.g., ```bash, ```python) and keep examples minimal but reproducible.

## Content Template Expectations
For vulnerability write-ups, follow the structure in `_Sablonlar/guvenlik_acigi_prompt_sablonu.md` (summary/impact, technical detail, PoC, risk, mitigations, code, checklist, monitoring). If you deviate, explain why in the document intro.

## Testing Guidelines
No automated tests are defined. If you add scripts or data that require verification, document exact steps in the new file and mention any assumptions.

## Commit & Pull Request Guidelines
Git history shows short, imperative summaries and occasional type prefixes like `docs:`. Please keep commit subjects concise (50–72 chars) and include CVE IDs when relevant. For PRs, include:
- A brief scope summary (what was added/changed).
- The list of new/updated documents.
- Any sources or references used.
- Updates to `README.md` if the index changed.
