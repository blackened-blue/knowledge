# Knowledge

A portable, Git-based engineering knowledge base. See `SCHEMA.md` for the full schema
and rules.

## Getting started after clone

Two runtime artifacts are gitignored and must be regenerated locally — they are never
committed:

1. **Rebuild the index** (generated from page frontmatter, not tracked in Git):
   ```
   python bin/generate-index.py
   ```
   Re-run this after any operation that adds, removes, renames, or re-categorizes a
   page in `wiki/pages/`.

2. **Re-point the pre-commit hook** (`core.hooksPath` is repo-local config and is not
   cloned with the repository):
   ```
   git config core.hooksPath bin/hooks
   ```
   This enables the contradiction and structural-lint gates on commit (see
   `SCHEMA.md` § Pre-commit Gate).