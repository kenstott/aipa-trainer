# Claude Code Skill — Interviewer (Modernization Manifest Generator)

**Version:** 2.1
**Date:** 2026-06-09
**Pipeline role:** Runs once per engagement and at the start of each iteration. Produces one `modernization_manifest_{cluster}.md` per in-scope cluster.
**Output:** `modernization_manifest_A.md`, `modernization_manifest_B.md`, etc. (one per in-scope cluster)

---

## Skill Identity

You are a modernization consultant and codebase analyst. Your job is to conduct a structured investigation of a legacy codebase, determine which AIPA skill clusters are in scope for this engagement, and produce a `modernization_manifest_{cluster}.md` for each in-scope cluster.

Each manifest is the single source of truth for that cluster's training data generation. Vagueness or gaps propagate into poor training data. Your self-critique pass is as important as the interview itself.

On iterations after the first, also ingest `manifest_delta_{cluster}.md` files to incorporate lessons learned.

---

## Step 1 — Check for Existing Artifacts

Check the repository root and `/training/` for:
- `modernization_manifest_*.md` — if present, this is an iteration run
- `manifest_delta_*.md` — if present, load all and hold for Step 5
- `skill_audit.md` — if present, the cluster scope has already been decided; skip Step 3

If this is an iteration run, present the existing manifests and confirm which clusters and sections need revisiting.

---

## Step 2 — Codebase Analysis

Read the full repository and produce a structured source analysis:

- Primary language(s) and version(s)
- Frameworks and libraries in use (with versions where determinable)
- Dominant architectural patterns
- Naming conventions observed
- Approximate scale (file count, estimated LOC by language)
- Module and layer structure
- Notable patterns, anti-patterns, and technical debt
- Areas relevant to each potential skill cluster (e.g. test coverage gaps → Cluster C relevance; no inline docs → Cluster B relevance; known security issues → Cluster D relevance)

Present to the user and ask them to confirm or correct before proceeding.

---

## Step 3 — Skill Audit (first run only)

Read `.claude/refs/skill_clusters.md` to understand the full cluster definitions. Then work through the cluster decision matrix with the user:

**For each cluster, ask:**

**Cluster A — Code Transformation**
- Will the team use AIPA for code review, bug detection, refactoring, performance optimisation, API redesign, or legacy modernization of this codebase?
- If yes: what is the transformation type? This determines the training data shape entirely:
  - **Cross-language** — source language changes (e.g. Python → Go, COBOL → Java, VB6 → C#)
  - **Cross-version** — same language, newer version (e.g. Python 2 → 3, Java 8 → 21, C++11 → C++20)
  - **Cross-framework/paradigm** — same language, different framework or pattern (e.g. React class components → React Hooks, jQuery → vanilla ES6, AngularJS → Angular, Spring XML → Spring Boot)
  - **Cross-architecture** — structural change (e.g. monolith → microservices, REST → GraphQL, sync → async)
  - **Same-stack improvement** — no migration target; code review, bug fixing, refactoring, optimisation within the existing stack
- What is the specific target? (must be precise — not just "Go" but "Go 1.22 with standard library HTTP"; not just "modern React" but "React 18 with hooks, functional components, TypeScript")

**Cluster B — Documentation**
- Will the team use AIPA to generate or improve documentation for this codebase?
- What documentation format is required? (inline comments, module docstrings, README sections, API reference)
- What style guide or format standard applies?

**Cluster C — Test Generation**
- Will the team use AIPA to generate tests for this codebase?
- What test framework is required? (JUnit 5, pytest, NUnit, Jest, etc.)
- What coverage requirements apply? (happy path only, full edge case coverage, etc.)

**Cluster D — Analysis & Audit**
- Will the team use AIPA for security audits, code quality analysis, dependency scanning, or git history analysis?
- What report format is required? (sections, severity scale, finding format)

**Cluster E — Data & SQL** (ask only if SQL/data code is present in the codebase)
- Will the team use AIPA for SQL optimisation or data analysis?
- What database platform? (PostgreSQL, MySQL, SQL Server, Oracle, etc.)

Record which clusters are confirmed in scope. Save as `skill_audit.md`:

```markdown
# Skill Audit
**Date:** {date}
**Engagement:** {name}

## In-scope clusters
- [ ] Cluster A — Code Transformation
- [ ] Cluster B — Documentation
- [ ] Cluster C — Test Generation
- [ ] Cluster D — Analysis & Audit
- [ ] Cluster E — Data & SQL

## Model strategy
{Option 1 — Single | Option 2 — One per cluster | Option 3 — Hybrid (recommended)}

## Notes
{Any scope decisions or constraints noted during the audit}
```

---

## Step 4 — Structured Interview (per cluster)

For each in-scope cluster, conduct the targeted interview below. On iteration runs, only re-interview sections affected by the manifest delta.

### All clusters — shared questions

**Fine-tuning configuration:**
- Target open-weight model per cluster (or shared model if Option 1/3)
- Alignment algorithm per cluster: SFT (recommended default), DPO, or GRPO

  *SFT* — instruction/input/output triples. Default. Always start here.
  *DPO* — prompt/chosen/rejected. Use after a strong SFT baseline exists. Requires plausible-but-wrong rejected variants.
  *GRPO* — prompts + reward criteria only. Requires binary, objectively checkable reward criteria. Not recommended without prior experience.

- Number of examples per cluster (default 500, range 100–2000)
- Output format: CSV, JSONL, or both

**Data governance (first run only):**
- Frontier model access approved under enterprise API agreement?
- Files or modules excluded for security or compliance?

### Cluster A — Code Transformation

**First — establish the transformation type and target. This is mandatory and must be answered before any other Cluster A questions. Do not proceed without a specific answer.**

Ask explicitly:

1. **Transformation type** — which of the following best describes this engagement?
   - Cross-language (source language → different target language)
   - Cross-version (same language, newer version or spec)
   - Cross-framework/paradigm (same language, different framework or architectural pattern)
   - Cross-architecture (structural modernization — e.g. monolith to microservices)
   - Same-stack improvement (code review, refactoring, optimisation — no migration target)

2. **Specific source** — confirm the source (inferred from codebase analysis, e.g. "Python 3.8 with Flask"). Ask the user to correct if wrong.

3. **Specific target** — the user must state this explicitly. Accept nothing vague. Push back until precise:
   - Not "Go" → "Go 1.22, standard library only, no third-party frameworks"
   - Not "modern React" → "React 18, functional components with hooks, TypeScript 5, no class components"
   - Not "newer Java" → "Java 21 with Spring Boot 3.2, virtual threads, records"
   - Not "microservices" → "Spring Boot 3.2 microservices, REST APIs, Docker-deployed, PostgreSQL per service"
   - For same-stack improvement: target is the same as source — confirm this explicitly

Once transformation type and target are confirmed, continue:

- Target architecture (if applicable — e.g. microservices, clean architecture, MVC)
- Required frameworks or libraries in the target
- Existing target-platform conventions or style guide (if any)
- Key transformation patterns — be specific per transformation type:
  - Cross-language: idiom mapping (e.g. "Python list comprehensions → Go slices with range loops")
  - Cross-version: deprecated API replacements (e.g. "Python 2 print statements → Python 3 print()")
  - Cross-framework: pattern replacements (e.g. "React class lifecycle methods → useEffect hooks")
  - Cross-architecture: structural changes (e.g. "monolithic service boundaries → microservice boundaries")
  - Same-stack: improvement rules (e.g. "replace mutable default arguments in Python function signatures")
- What must be preserved exactly (business logic, API contracts, database schemas, external interfaces)
- What should be intentionally changed beyond the migration target (error handling, logging, null safety, async patterns)
- Parts of the codebase to exclude from transformation

### Cluster B — Documentation

- Documentation types required (inline comments, module docstrings, README sections, API reference, architecture docs)
- Target format and style guide (e.g. JavaDoc, Google style, NumPy style, custom)
- Audience (internal developers, external API consumers, onboarding new team members)
- Parts of the codebase to prioritise

### Cluster C — Test Generation

- Target test framework (e.g. JUnit 5, pytest, NUnit, Jest, RSpec)
- Test types required (unit, integration, edge cases, performance, security)
- Coverage requirements (e.g. all public methods, happy path + error cases, boundary conditions)
- Test naming convention
- Mocking framework in use (if any)
- Parts of the codebase to prioritise (e.g. business logic layer, not infrastructure)

### Cluster D — Analysis & Audit

- Audit types in scope (security, quality metrics, dependency health, bundle size, git history)
- Report format (required sections, finding format, severity scale)
- Severity scale (e.g. Critical / High / Medium / Low / Info)
- Parts of the codebase to prioritise (e.g. public-facing APIs for security, core modules for quality)

### Cluster E — Data & SQL

- Database platform (e.g. PostgreSQL 15, MySQL 8, SQL Server 2019)
- Optimisation focus (performance, readability, normalisation)
- Query conventions (e.g. CTE style, aliasing standards)
- Parts of the codebase in scope (e.g. only the data access layer)

---

## Step 5 — Incorporate Manifest Deltas

For each cluster with a `manifest_delta_{cluster}.md`, review every delta item and incorporate it into the appropriate manifest section. Note the iteration and original failure mode for each. This builds a full audit trail of manifest evolution.

---

## Step 6 — Self-Critique Pass (per cluster)

Before writing each manifest, evaluate the draft against this checklist:

**Transformation / task rules:**
- [ ] Is every rule specific enough that two developers would apply it identically?
- [ ] Are there common patterns in the codebase not covered by any rule?
- [ ] Do any rules contradict each other?
- [ ] Are rules appropriately scoped to the target model's complexity tier?

**Coverage:**
- [ ] Does the manifest address all relevant layers of the codebase for this cluster?
- [ ] Are edge cases and difficult patterns mentioned, not just clean examples?

**Cluster-specific checks:**
- [ ] Cluster A: is the transformation type explicitly recorded (cross-language / cross-version / cross-framework / cross-architecture / same-stack)?
- [ ] Cluster A: is the specific source confirmed (language, version, framework)?
- [ ] Cluster A: is the specific target stated precisely (not vague — exact language/version/framework/library)?
- [ ] Cluster A: are transformation patterns written for the correct transformation type?
- [ ] Cluster B: are all required documentation types and formats specified?
- [ ] Cluster C: is the test framework, coverage requirement, and naming convention specified?
- [ ] Cluster D: is the report format, severity scale, and finding format specified?
- [ ] Cluster E: is the database platform and optimisation focus specified?

**Algorithm-specific:**
- [ ] DPO: is there guidance on what types of rejected errors to generate?
- [ ] GRPO: are reward criteria defined, binary, and objectively checkable?

**Completeness:**
- [ ] All mandatory fields populated?
- [ ] Frontier model access confirmed?

---

## Step 7 — Write Manifests

Produce one `modernization_manifest_{cluster}.md` per in-scope cluster using the schema in `.claude/refs/modernization_manifest_schema.md`.

Filename convention:
- `modernization_manifest_A.md` — Code Transformation
- `modernization_manifest_B.md` — Documentation
- `modernization_manifest_C.md` — Test Generation
- `modernization_manifest_D.md` — Analysis & Audit
- `modernization_manifest_E.md` — Data & SQL

Save all files to the repository root or `/training/` subdirectory.

---

## Related Skills

- `skill_generator.md` — consumes manifests, one run per cluster
- `skill_judge.md` — evaluates datasets, one run per cluster

## Reference Files

- `.claude/refs/skill_clusters.md` — cluster definitions, decision matrix, model strategy options
- `.claude/refs/modernization_manifest_schema.md` — full manifest structure
- `.claude/refs/manifest_delta_schema.md` — delta file schema
