# Wiki Schema

## Identity
- **Location-independent:** This Knowledge Base is a standalone Git repository. It has
  no fixed filesystem path — it may be cloned to any location, on any machine, and used
  with any Project. Never hardcode an absolute path in this schema or in any page; refer
  to paths relative to the repository root (e.g. `wiki/pages/`, `raw/development/`).
- **Domain:** Reusable engineering knowledge base spanning **multiple projects and
  multiple versions**. Captures and distills: architecture/system design and diagrams;
  source code, database schemas, and configuration; requirements, incidents, and test
  cases; Git history (commits, diffs, branches, tags, release notes); meeting notes;
  external documentation, URLs, and transcripts. Governing principle: raw evidence lives
  in `raw/`, every page is review-gated, and every claim must trace back to its original
  Source. The Knowledge Base is **technology-independent** — it must not assume a
  specific programming language, framework, database, or project layout.
- **Source types:** architecture/design documents, diagrams, source code, database
  schemas/configuration files, requirements documents, incident reports, test cases,
  Git commits/diffs/branches/tags, release notes, meeting notes, external documentation,
  URLs, transcripts
- **Created:** 2026-09-12

## Authority and Source Priority

When knowledge conflicts, resolve using this priority order, highest first:

1. **Actual Git / project repository** — the implementation itself
2. **Released Source Evidence** (`raw/released/`)
3. **Development Source Material** (`raw/development/`)
4. **Wiki Knowledge**
5. **AI Inference**

The project repository is the source of truth for implementation. The Wiki explains and
organizes knowledge but must never override the actual implementation. AI inference must
never override Source evidence. If the Wiki conflicts with the actual project Source, the
Source is trusted and the Wiki is corrected — not the other way around.

> Git says JWT uses ES256. Wiki says JWT uses RS256. → Trust ES256; update the Wiki.

**Git tells us what the system is. Wiki explains what the system means.**

## Project

A **Project** is the primary knowledge boundary. All project-specific knowledge belongs
to exactly one Project — an independent system, library, application, service, platform,
infrastructure project, or other logical software/product unit. Projects are never
silently mixed: knowledge from one Project does not automatically become knowledge of
another. A relationship between Projects must be explicit (see **Cross-Project
Relationships** below).

### Project identity
- `project.id` must be unique within the Knowledge Base, human-readable, lowercase
  kebab-case, and **must not contain a version** (`authentication`, not
  `authentication-v1`). It stays stable even if the display name changes.
- `project.name` is the human-readable display name; `project.id` is the primary
  identity every skill uses to key knowledge to a Project.
- Version is stored separately (see **Version**), never folded into `project.id`.

### Project registration workflow
Before creating a new Project, search for an existing one first:

```
New Project → Search Existing Projects → Existing? ──yes──→ Use Existing Project ID
                                              │
                                              no
                                              ↓
                                    Register New Project
                                              ↓
                                    Generate Project ID
                                              ↓
                                    Validate Uniqueness
```

Search existing `project:` values across `wiki/pages/*.md` frontmatter and any
project-profile pages (see below) before minting a new `project.id`. Never create a
duplicate Project because of a different display name. If two candidates might be the
same system and the identity is genuinely ambiguous, ask the user — never guess.

### Project profile pages
Each registered Project SHOULD have one canonical profile page at
`wiki/pages/<project-id>.md`, categorized **Structure** (it describes what the project
is and how it's configured), carrying:

```yaml
project: <project-id>
```
```
project:
  id: authentication
  name: Authentication Library
  repository: github.com/example/authentication
  defaultBranch: main
  currentVersion: v1.2
```

This page is the registry entry for the Project — the first place both a human and an
ingest/query operation check when resolving whether a Project already exists. Project
configuration (repository, default branch, release/versioning strategy, project-specific
conventions, technology stack) belongs here, in Project scope — the global schema above
stays technology-independent.

## Version

A **Version belongs to a Project** — the identity is always `Project → Version`, never
`Version` alone (`authentication / v1.2` and `payment-wallet / v1.2` are unrelated). A
Version may be a semantic version, release number, Git tag, calendar version, deployment
version, commit, or any other project-defined identifier — SemVer is never required.

### Version resolution
- Resolve Version from reliable evidence: Git tag, release identifier, branch,
  deployment version, Project configuration, user-provided version, or commit metadata.
- **Never invent a Version.** If it cannot be determined reliably, ask the user, or mark
  it explicitly as `unknown` when the workflow allows that.
- For questions about the **current/production** system, prefer the latest **released**
  Version.
- For questions about **development**, use the relevant development Version.
- When a Version is **explicitly specified**, use exactly that Version.

## Source

A **Source** is the evidence a piece of knowledge was extracted from. Traceability from
knowledge back to its Source must always be preserved.

### Source identity
Identity depends on lifecycle state:

- **Development:** `Project → Version → Branch → Commit`
- **Released:** `Project → Version → Release Tag/Identifier → Commit`

Developer name is optional metadata, never the primary identity — the actual Git state
is authoritative.

### Source ID
`source-id` is a human-readable **organizational** label for a unit of development work
(`feature-jwt`, `feature-api-key`, `fix-signature`, `database-migration`) — it is not the
authoritative identity of the Source. The authoritative identity is the Git information
(`branch + commit`, or `release tag + commit`).

### Development Source
Represents work not yet an official release. Store under:

```
raw/
└── development/
    └── <project-id>/
        └── <version>/
            └── <source-id>/
```

Example:
```
raw/development/authentication/v1.2/feature-jwt/
raw/development/authentication/v1.2/feature-api-key/
raw/development/authentication/v1.2/feature-login/
```

Development Source metadata may include:
```yaml
project: authentication
version: v1.2
state: in-development       # in-development | merged | released
source:
  id: feature-jwt
  branch: feature/jwt-authentication
  commit: abc123
  baseRelease: v1.1
```

### Development Source state
States: `in-development`, `merged`, `released`.

**Merged ≠ Released.** A merged branch is not automatically an official release. A
Source becomes `released` only on reliable release evidence: an official Git tag, an
official release version, a deployment release identifier, or other project-defined
release evidence.

### Multiple developers and branches
A release may combine changes from multiple developers/branches. The final release
Source must reflect the actual final Git state **after** merge — never manually stitch
together separate branch snapshots and treat that as the release Source.

```
Developer A ──┐
Developer B ──┼──→ Merge → Main/Release Branch → Official Release
Developer C ──┘
```

### Released Source
Immutable evidence of an official release. Store under:

```
raw/
└── released/
    └── <project-id>/
        └── <version>/
```

Example: `raw/released/authentication/v1.2/`. Should carry enough metadata to identify
the exact released state:
```yaml
project: authentication
version: v1.2
release:
  tag: v1.2
  commit: abc123
```

### Development → Release workflow
```
Development Branch → Merge → Main/Release Branch → Official Release
  → Release Tag/Identifier → Capture Final Git State
  → Create Released Source Evidence → Update Wiki
  → Remove Temporary Development Material
```
A release must always be based on the actual final Git state.

### Development raw material lifecycle
`raw/development/` is temporary working ingestion material. `raw/released/` is
permanent release evidence. When a Project officially releases:

1. Confirm the final Git state.
2. Confirm the release identifier.
3. Create Released Source evidence under `raw/released/<project-id>/<version>/`.
4. Update or create the relevant Wiki knowledge, with traceability pointing at the new
   Released Source.
5. Remove the corresponding temporary material from
   `raw/development/<project-id>/<version>/<source-id>/`.

Never keep the same Source duplicated in both `raw/development/` and `raw/released/`.
No separate raw archive is required beyond this — Git history provides historical
tracking.

## Page Frontmatter
Every wiki page must start with:
```yaml
---
title: <page title>
category: <one of the Index Categories below>
summary: <one-line description — becomes this page's index entry>
tags: [tag1, tag2]
sources: [source-slug1]
created: YYYY-MM-DD
updated: YYYY-MM-DD
project: <project-id>     # optional — present when the concept is Project-scoped
version: <version>        # optional — present when the concept is Version-scoped
---
```

`category` and `summary` drive index generation (see **Index Generation** below);
`category` must match one of the wiki's Index Categories. `created` is set once when the
page is first written and never changes; `updated` bumps on every edit. `project` and
`version` are additive, Project-specific metadata — they never replace or narrow the
required fields above (see **Project Context in Wiki Pages**). A page with no `project`
field is either cross-project (a genuinely shared Concept/Standard) or non-technical
knowledge with no Project owner.

A **Sources**-category page (one page per ingested Source) should additionally carry
Source Identity so traceability survives independent of `raw/`:
```yaml
project: authentication
version: v1.2
source:
  id: feature-jwt
  state: released          # in-development | merged | released
  branch: feature/jwt-authentication
  commit: abc123
  release: v1.2            # present only once state: released
```

## Cross-References
- **link_style:** obsidian
- **link_style_rules:** config/link-style.md
- See `config/link-style.md` for the exact emit and parse rules. Every wiki skill
  reads that file to decide how to write new cross-references and how to scan
  existing ones.

## Concept Identity

The slug **is** the concept's identity — there is no separate id. A concept is the
page at `wiki/pages/<slug>.md`; everything that links to it uses `[[slug]]`. This only
works if the link graph is trustworthy, so these rules hold everywhere links are written:

1. **Links are verified, never invented.** Before writing any `[[slug]]`, the slug must
   resolve to an existing `wiki/pages/<slug>.md` **or** to a page being created in the
   same operation. List the existing page set first (`ls wiki/pages/`); never emit a
   link to a slug you have not confirmed. A `[[slug]]` that resolves to nothing is a
   hallucinated link — the failure this discipline exists to prevent.

2. **Homonyms get qualified slugs.** When a new concept collides with an existing slug
   for a *different* sense, qualify both with a discriminator rather than overloading
   one page:
   - `mercury-planet` / `mercury-element` / `mercury-mythology`
   - `transformer-ml` / `transformer-electrical`

   Pick the narrowest discriminator that disambiguates. `wiki-lint` warns when slugs
   sharing a base token look like an unintended collision.

3. **Cross-project collisions are qualified by Project, not by directory.** Because
   `wiki/pages/` stays flat (see **Wiki Pages** below), the same concept name can appear
   in multiple Projects. First decide whether it is a genuinely **shared Concept** (one
   page, `project` field omitted or listing multiple Projects) or a **Project-specific
   Concept** (qualified slug, one page per Project). When Project-specific, qualify with
   the Project id as a slug prefix/suffix:
   - `authentication-flow` (shared, or the primary/first Project)
   - `payment-wallet-authentication-flow` (payment-wallet's distinct flow)

   Never create a Project folder under `wiki/pages/` to resolve the collision instead.

Consolidating two pages that turn out to be the same concept (merge), or separating one
overloaded page into qualified pages (split), is the job of the `wiki-merge` skill.

## Wiki Pages Represent Concepts, Not Files

Wiki pages represent **concepts and knowledge**, never individual source files or a
single Project's directory layout. `jwt-authentication.md` represents the concept *JWT
Authentication* — not `src/Auth/JwtService.cs`. `wiki/pages/` must remain flat with no
Project subdirectories (**Project is context, stored in frontmatter; the page slug is
concept identity** — see **Project Context in Wiki Pages**).

### Page creation
Before creating a new page, search existing concepts:
```
New Knowledge → Search Existing Concepts → Same Concept? ──yes──→ Update Existing Page
                                                │
                                                no
                                                ↓
                                          Create New Page
```
Never create a duplicate page for the same concept because a new Source phrased it
differently. Never create a version-suffixed page (`jwt-authentication-v1.2.md`) as a
substitute for updating the concept page — see **Page Versioning Policy**.

### Page update
```
Existing Page + New Evidence → Review Existing Knowledge → Update Page
                                                              ↓
                                                    Add Source Traceability
```
Updates must preserve the page's conceptual identity; the page evolves as the Project
evolves, it does not get replaced.

### Page versioning policy
Pages normally represent a concept **across** versions — do not automatically create a
separate page per Version. Prefer:
```
jwt-authentication.md
  Current: v1.2 uses ES256.
  Previous: v1.1 used RS256.
```
Create a Version-specific page only when the Project explicitly requires it, the
concepts are genuinely different (not just an updated detail), or the Version-specific
knowledge cannot be represented clearly inside the existing concept page.

## Project Context in Wiki Pages

Because `wiki/pages/` is flat, Project ownership is carried in page metadata
(`project: <project-id>` in frontmatter) and content — never in directory nesting. A
page may reference or belong to multiple Projects when the concept is intentionally
shared (e.g. a Standard or a cross-cutting Concept). See **Concept Identity** rule 3 for
how to resolve a same-name concept that is *not* actually shared.

## Cross-Project Relationships

Projects may depend on other Projects. Dependencies must be explicit, never an implicit
copy of knowledge from one Project into another:
```yaml
dependency:
  project: authentication
  version: v1.2
```
If a dependency changes, review the affected Projects' knowledge for staleness.

## Citations

Cite every non-common-knowledge factual claim. "Common knowledge" = uncontroversial,
undergraduate-level facts in this wiki's domain. Granularity is paragraph or claim,
never per-sentence. If you cannot produce a citation in one of the forms below,
find one, weaken the claim, or drop it.

Format: Markdown footnotes. Two citation kinds, three valid targets.

**Quote citation** (preferred):
```
The model uses 8 attention heads.[^1]

[^1]: [[attention-is-all-you-need]] §3.2.2 L142-143 — "We employ h = 8 parallel attention layers"
```

**Synthesis citation** (when no single quote captures the claim):
```
The architecture is fundamentally an encoder-decoder with attention.[^2]

[^2]: [[attention-is-all-you-need]] §3.2-3.4 [synthesis] L138-202 — encoder, decoder, and
      attention sections together describe the full multi-head architecture
```

`L142-143` / `L138-202` are line ranges in the raw source file. For a quote they mark
the lines the quote is taken from; for a synthesis they mark the block being summarized.

Three rules for every footnote:

1. **The cited target is one of three forms:**
   - A slug reference to a Sources-category wiki page, written in the wiki's
     `link_style` (preferred for sources you've ingested via `wiki-ingest`)
   - A path under `raw/` or `assets/` — for a Project's evidence, this is
     `raw/development/<project-id>/<version>/<source-id>/<file>` or
     `raw/released/<project-id>/<version>/<file>` (for drive-by citations where a
     synthesis page isn't worth creating)
   - `<URL>` — a live URL, tweet, or ephemeral source (no local copy required)

   Never cite entity, concept, or analysis pages — those are syntheses, not sources.

2. **A locator is present.** Always a semantic locator: `§<section>`, `p.<n>`,
   `[HH:MM:SS]` for transcripts, URL anchor for web, or `(YYYY-MM-DD)` for dated posts.

   **Plus a line-range when the source is text-addressable.** If the resolved raw
   file is markdown, plaintext, code, or cached HTML, append a line-range token after
   the semantic locator:

   - `L<start>-<end>` — a range, e.g. `L142-145`
   - `L<n>` — a single line, e.g. `L142`
   - `L142-145,L201-203` — disjoint ranges

   The line range refers to lines in the **raw source file** resolved from the target
   (`[[slug]]` → its `**Source:**` raw path; or a direct `raw/...`/`assets/<file>`
   path). `raw/` is immutable, so these line numbers are stable references.

   A line-range is **required** for text-addressable sources and applies to BOTH
   citation kinds — a `[synthesis]` footnote marks the block it summarizes with `L…`
   just as a quote marks the lines it quotes. **Exempt** (semantic locator only, no
   `L…`): PDFs, transcripts, and live URLs with no local cached copy.

3. **Either a verbatim quote, or the `[synthesis]` tag plus a description** of
   what the cited range supports. No third option.

**Drive-by citation examples:**
```
[^3]: raw/development/authentication/v1.2/feature-jwt/jwt_service.py L40-52 — "signing_key = ES256"
[^4]: raw/released/authentication/v1.1/scaling-notes.pdf p.7 — "loss scales as a power law in compute"
[^5]: https://twitter.com/user/status/123 (2026-04-15) — "<tweet text>"
```

## Cross-Model Review

`wiki-audit strong` runs a second-opinion pass with a different-provider model and
stamps the audited page with an optional `review:` frontmatter block:
```
review:
  model: codex          # gemini | claude-sonnet
  provider: openai      # google | anthropic
  date: YYYY-MM-DD
  status: clean         # or: disputed
  findings: 2           # present only when status: disputed
```
- `status: clean` — the reviewer surfaced no disagreement with the normal audit.
- `status: disputed` — the reviewer flagged overreach or a contradiction the normal
  audit missed; `findings:` carries the count. The detail lives in the (local-only)
  audit report.
- `provider: anthropic` (the `claude-sonnet` fallback) means no different-provider CLI
  was available, so the check is same-provider and weaker.

This block is optional and is added only by `wiki-audit strong`. Pages never need it to
be valid.

## Contradiction Check

`wiki-ingest` runs a cheap contradiction check on the pages each ingest touches, before it
commits. It is a **gate, not an annotation**: every page that lands in git is clean.

- **Scope — touched neighbors only.** The check compares the pages an ingest wrote or
  edited against (a) themselves and (b) the pages that ingest already read (the entity /
  concept pages it updated and the neighbor pages from its backlink audit). It does NOT
  re-read the whole wiki — a conflict with a distant, untouched page is left to the
  periodic `wiki-lint` sweep.
- **Project scope.** Two Projects independently describing the same concept name
  differently is **not** a contradiction — it is two Project-scoped concepts that should
  use qualified slugs (**Concept Identity**, rule 3), or genuinely unrelated knowledge.
  Only treat pages as contradiction candidates across Projects when an explicit
  **Cross-Project Relationship** (dependency) puts the same concept in play.
- **Authority applies inside the check.** When a page's claim conflicts with its own
  cited Source (or with `raw/released/` outranking `raw/development/`, which outranks
  Wiki knowledge — see **Authority and Source Priority**), the Source wins and the page
  is corrected, not merely flagged.
- **Blocking vs. soft.** A **blocking** contradiction is a real factual conflict on the same
  entity under the same scope — incompatible dates, counts, names, or mutually-exclusive
  claims. A **soft** tension (differing emphasis, values within plausible version /
  measurement variance, claims that hold under different scope) is not a conflict.
- **The transient blocker flag.** When a blocking contradiction is found, a single line is
  written to the affected page's frontmatter and the ingest stops before committing:
  ```yaml
  contradiction-check: failed — <one-line reason naming the counterpart [[slug]] or "internal">
  ```
  The machine-readable token is the literal `contradiction-check: failed`. It exists ONLY
  while the conflict is unresolved; resolving the conflict **removes the line**. A committed
  page never carries it — there is no `passed` stamp, no severity history, nothing. Absence
  of the flag is the only "clean" state.
- **Soft tensions are surfaced, not recorded** — mentioned in the ingest summary so you can
  act if you wish, but never persisted and never blocking.

This flag is also what the **Pre-commit Gate** below scans staged files for.

## Pre-commit Gate

On a git wiki, `bin/hooks/pre-commit` (installed by `wiki-init` via
`git config core.hooksPath bin/hooks`) runs **two** deterministic gates before every commit —
no LLM:

1. **`bin/check-contradictions.py`** — scans the **staged** content of `wiki/pages/*.md`,
   frontmatter only, and **blocks the commit** if any page still carries a
   `contradiction-check: failed` flag. Backstop to the skill-level hold in `wiki-ingest`
   step 7b; on a healthy wiki it never fires. Resolve the contradiction and remove the
   `contradiction-check:` line, then re-stage.
2. **`bin/lint-mechanical.py --staged`** — scans the staged pages for **structural**
   problems and **blocks the commit** on any: missing required frontmatter, a broken
   `[[link]]`, or a slug collision (a bare slug clashing with a qualified one). Fix the page
   and re-stage.

- **Fresh clone:** `core.hooksPath` is repo-local config and is not cloned — re-run
  `git config core.hooksPath bin/hooks` once after cloning.
- **Override** an intentional commit with `git commit --no-verify`.

## Ingest Workflow

Beyond the generic `wiki-ingest` flow, this wiki's ingest process must resolve Project,
Version, and Source identity before writing any page:

```
Source
  → Identify Project (search existing; register only if genuinely new)
  → Identify Version (from reliable evidence; never invented)
  → Identify Source State (in-development | merged | released)
  → Validate Source Identity (branch+commit, or release tag+commit)
  → Store Source Material (raw/development/... or raw/released/...)
  → Run Knowledge Ingestion
  → Identify Concepts / Knowledge
  → Search Existing Wiki Pages
  → Create or Update Pages (with project/version frontmatter)
  → Add Source Traceability (citations per §Citations)
  → Contradiction Check (project-scoped, §Contradiction Check)
  → Validate / Lint
  → Commit
```

A page must never be created before checking whether the concept already exists.

## Query Workflow

Beyond the generic `wiki-query` flow, answering a question in this wiki must resolve
Project and Version context first:

```
Question → Identify Project → Identify Version Context
        → Search Relevant Wiki Knowledge → Verify Source When Necessary → Answer
```

- If the Project is ambiguous, **ask** — never silently pick one. Example: "How does
  authentication work?" when `authentication`, `payment-wallet`, and `platform-core` all
  have an authentication concept — ask which Project is intended unless context already
  makes it clear.
- Prefer **Released** knowledge for current/production questions, **Development**
  knowledge for development questions, and the **explicitly specified** Version when one
  is given (**Development vs Released Knowledge**, mirrors **Version resolution**).
  Development knowledge must never be presented as production behavior unless the Source
  confirms it has actually been released.

## Operation Log & Commit Convention
Operations: init, ingest, query, update, lint, audit, merge, split

**Git wiki — the git history is the operation log.** After an operation, the skill
suggests a commit message and commits on your confirmation (skills never auto-commit).
Render the human log on demand with `python bin/render-log.py`.

The suggested subject line follows the repo's commit convention:
1. **Detect an existing convention first** — scan recent `git log` and any `.gitmessage`,
   commitlint config, or `CONTRIBUTING.md`. If the repo already has a subject style,
   follow it.
2. **Default to Conventional Commits** when none is found, choosing the type by operation:

   | Operation        | Type                                  |
   |------------------|---------------------------------------|
   | init             | `chore`                               |
   | ingest           | `docs`                                |
   | update           | `docs`                                |
   | query (saved)    | `docs`                                |
   | lint             | `fix` if fixes applied, else `chore`  |
   | audit            | `fix` if fixes applied, else `chore`  |
   | merge / split    | `refactor`                            |

**Always append a `Wiki-Op:` trailer**, whatever the subject style — it is what
`render-log.py` keys on, decoupling the log from the subject convention. Which pages
changed is read from the commit diff, so no `Pages:` trailer is needed.
```
docs: summarize Attention Is All You Need

Wiki-Op: ingest
```

**Non-git wiki — fallback to `wiki/log.md`.** Append one entry per operation:
`## [YYYY-MM-DD] <operation> | <title>`.

## Index Generation
`wiki/index.md` is a generated, gitignored artifact — never hand-edit it. It is rebuilt
from page frontmatter by `bin/generate-index.py`:
- Run `python bin/generate-index.py` (or `python3`) **before reading the index**, and
  **after** any operation that adds, removes, renames, or re-categorizes a page.
- The generator groups pages by their `category` frontmatter, in the order categories are
  listed under **Index Categories** below; within a category it lists pages newest-first
  by `created`. Each entry is `- [[slug]] — summary _(created)_`.
- Pages whose filename matches `audit-*.md` are excluded (gitignored local-only
  artifacts). A page with an unrecognized or missing `category` lands in an
  `Uncategorized` section.
- The index groups by **category only** — it does not group by Project. To find all
  pages for a Project, search `wiki/pages/*.md` frontmatter for `project: <project-id>`
  (or the Project's profile page's backlinks) rather than relying on index.md structure.

## Index Categories

This wiki's categories are organized along a single dimension: the **question asked
during work** that a page must answer, not its format or topic. Each category answers
exactly one recurring question this wiki must support — writing/reviewing requirements,
designing architecture, understanding an existing system, investigating an incident,
onboarding.

| Category      | Question it answers                                                          |
|----------------|-------------------------------------------------------------------------------|
| Sources        | What did the original source say — one page per ingested source              |
| Requirements   | What must be done — goals, stakeholder needs, fit criteria                   |
| Standards      | What counts as "correct" and what terms mean — rules, compliance, glossary   |
| Concepts       | What is this idea, when is it used — general patterns, not project-specific  |
| Capabilities   | What can the system already do — features, APIs, contracts                   |
| Structure      | What is the system made of — components, data model, config, topology, and Project profile/registry pages |
| Flows          | How does it work step by step — sequences, solution/network paths            |
| Decisions      | Why was this chosen, and when does it stop applying — alternatives, tradeoffs|
| Playbooks      | How do you do this — procedures, runbooks, checklists                       |
| Lessons        | Have we been burned by this before — incidents, postmortems, test findings  |
| Changes        | What changed, and is it still current — timelines, migrations               |
| Relationships  | Who do you ask, what depends on what — ownership, escalation paths, Cross-Project Relationships |

Two rules keep this dimension from drifting:

1. **No "Analyses" category.** Analysis is a *format*, not a question. Route an analysis
   page by the question it actually answers: supports a decision → Decisions; explains
   how the system came to be built this way → Structure; distilled from a past failure →
   Lessons.
2. **No "Testing" category.** Testing content splits across three existing categories:
   acceptance/fit criteria → Requirements; how to run tests → Playbooks; what a test run
   revealed → Lessons.

Projects themselves are not a category — a Project is a cross-cutting scope carried in
`project:` frontmatter (see **Project Context in Wiki Pages**), and a Project's own
profile/registry page lives under **Structure**.

## Recommended Directory Structure

```
.
├── SCHEMA.md
├── .gitignore
├── bin/
│   ├── generate-index.py
│   ├── render-log.py
│   ├── check-contradictions.py
│   ├── lint-mechanical.py
│   └── hooks/pre-commit
├── config/
│   └── link-style.md
├── raw/                          ← Project- and Version-scoped, immutable
│   ├── development/
│   │   └── <project-id>/
│   │       └── <version>/
│   │           └── <source-id>/
│   └── released/
│       └── <project-id>/
│           └── <version>/
├── wiki/
│   ├── index.md                  ← generated, gitignored
│   ├── overview.md
│   ├── log.md                    ← non-git wikis only
│   └── pages/                    ← flat, no Project subdirectories
│       ├── <project-id>.md       ← Structure-category Project profile
│       └── <slug>.md
└── assets/
```

`wiki/pages/` must remain flat. `raw/` is Project- and Version-scoped.

## AI Rules

**Project**
- Always identify the Project before processing project-specific knowledge.
- Search existing Projects before creating a new one; never silently create a duplicate.
- Never mix knowledge between Projects without an explicit relationship.

**Version**
- Resolve Version from reliable evidence; never invent one.
- Prefer released Version for production/current questions; development Version for
  development questions; the specified Version when one is given.

**Source**
- Preserve Source traceability (§Citations).
- Use Git branch+commit as authoritative development identity; release tag/identifier
  + commit as authoritative released identity.

**Wiki**
- Search existing concepts before creating pages; update rather than duplicate.
- Never create Project folders under `wiki/pages/`.
- Never auto-create one page per Version (§Page Versioning Policy).

**Authority**
- Git is authoritative for implementation.
- Source evidence outranks Wiki knowledge; Wiki knowledge outranks AI inference.

**Uncertainty**
- If information cannot be determined reliably, ask the user rather than inventing it —
  applies to Project identity, Version, Source state, and factual claims alike.

## Golden Rules

1. Project is the primary knowledge boundary.
2. Project ID must be unique and stable, and never contain a version.
3. Version belongs to Project — `Project → Version`, not `Version` alone.
4. Never invent Project or Version information; ask instead.
5. Git is the source of truth for implementation.
6. Wiki explains knowledge; Git defines implementation.
7. Every non-common-knowledge claim must be traceable to a Source.
8. Development and Released knowledge must be distinguished.
9. Merged does not mean Released.
10. Released Source must represent the actual final Git state.
11. Development raw material (`raw/development/`) is temporary.
12. Released raw material (`raw/released/`) is permanent release evidence.
13. Never keep the same development Source in both `raw/development/` and `raw/released/`.
14. Wiki pages represent concepts, not source files.
15. `wiki/pages/` must remain flat.
16. Project context belongs in frontmatter, not page directories.
17. Search existing concepts before creating a Wiki page.
18. Update an existing concept instead of creating duplicates.
19. Never automatically create one Wiki page per Version.
20. Cross-project knowledge must be explicitly related (§Cross-Project Relationships).
21. Never silently mix knowledge between Projects.
22. AI inference must never override Source evidence.
23. When information is ambiguous, ask instead of guessing.
24. The schema must remain technology-independent.
25. `raw/` is immutable evidence; committed Wiki pages must never carry an unresolved
    `contradiction-check: failed` flag (§Contradiction Check, §Pre-commit Gate).

## Conventions
- raw/ is immutable — skills never modify it; it is split into `raw/development/<project-id>/<version>/<source-id>/` (temporary) and `raw/released/<project-id>/<version>/` (permanent) — see §Source
- operation log: git wikis record each op as a commit (see Operation Log & Commit Convention) and render it with bin/render-log.py; non-git wikis append to log.md (append-only, never rewritten)
- index.md is GENERATED by bin/generate-index.py and is gitignored — never hand-edit it; set page frontmatter (category, summary) and regenerate instead
- All pages live flat in wiki/pages/ — no subdirectories, no Project folders; Project ownership is a `project:` frontmatter field
- overview.md reflects the current synthesis across all sources
- Cross-reference and citation slug-targets follow `config/link-style.md` —
  every skill reads it before writing or scanning links
- contradiction check: ingest gates on blocking contradictions in touched pages via a transient `contradiction-check: failed` flag, removed before commit — committed pages are always clean, and cross-Project pages are never treated as contradicting absent an explicit dependency (see Contradiction Check)
- pre-commit gate: git wikis run bin/hooks/pre-commit (via core.hooksPath) → bin/check-contradictions.py, which blocks any commit staging a page that still carries the flag (see Pre-commit Gate); re-run `git config core.hooksPath bin/hooks` after a fresh clone
- README boundary: wiki pages must not duplicate README content. Extract structural signals; link to the README for operational content (setup, contributing, running). When ingesting any README, also evaluate it for gaps and suggest edits.
- authority: Git repository > Released Source > Development Source > Wiki > AI inference — a Wiki page that conflicts with its own Source is corrected, not the Source (see Authority and Source Priority)
