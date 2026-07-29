# Software Architecture Notes Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create 12 numbered software-architecture study notes plus a real-codebase architecture writeup of the pricing-tool project, all as Obsidian markdown.

**Architecture:** Deliverable 1 is a fine-grained set of `brain/` notes ordered principles → terms → patterns → DDD → tradeoffs, cross-linked with wikilinks. Deliverable 2 is a `reference/` note that documents and audits the real pricing-tool Go codebase. This is a documentation task; each task's "test" is a format-and-wikilink verification checklist, not a unit test.

**Tech Stack:** Obsidian markdown. Go code snippets inside Deliverable 1 notes. Read-only Go source at `/Users/yeongseo.na/IdeaProjects/pricing-tool` for Deliverable 2.

**Spec:** `docs/superpowers/specs/2026-07-29-software-architecture-notes-design.md` (read it before starting; it holds full per-file content specs).

## Global Constraints

- Writing style: simple, short English sentences; one idea per sentence.
- Deliverable 1 files live in `🧠 brain/software-architecture/` and MUST follow all brain format rules (below). Go for all code examples.
- Deliverable 2 file lives in `🐟 reference/pricing-tool/` and MUST NOT use tags, numbering, or a Review History section (reference-path rule).
- Never modify the pricing-tool project. Read only.
- Commit style: conventional prefix (`docs`), single sentence, no body, no Claude co-author, max 50 chars.
- Highlight key terms with `==text==`.

## Brain note format rules (Deliverable 1 — applies to every note)

Every Deliverable 1 file MUST follow this exact skeleton:

```markdown
#<level> #ai-draft

# <Title>

<intro paragraph and/or first ## section — NO `---` before the first ## >

## <First Section>
<content>

---
## <Next Section>
<sentences on consecutive lines, no blank line between sentences in a paragraph>

---
## Related Notes

- [[<filename-without-extension>]] — short description

## Review History

-
```

Rules:
1. Line 1 = two tags: one level tag (`#aware`/`#understand`/`#essential`, per task) + `#ai-draft`.
2. Blank line, then `# Title`.
3. First `##` section has NO `---` before it. Every later `##` has `---` on its own line before it.
4. `==highlight==` for key terms.
5. Sentences in a paragraph go on consecutive lines (no blank line between them).
6. Second-to-last section `## Related Notes` with wikilinks.
7. Last section always `## Review History` with a single `-`.

## Per-note verification checklist (Deliverable 1)

Run this after writing each note (`FILE` = the note just written):
- Line 1 has exactly two tags, one level + `#ai-draft`.
- `## Review History` is the last section and contains a `-`.
- No `---` immediately before the first `##`; every later `##` has one.
- All `[[wikilinks]]` name files that exist (or will exist) in the file list below.

File list (target filenames, no extension shown):
`1-core-design-principles`, `2-solid-principles`, `3-dependency-injection`, `4-domain-vs-infrastructure`, `5-ports-and-adapters`, `6-dto-and-mappers`, `7-layered-architecture`, `8-hexagonal-architecture`, `9-clean-architecture`, `10-domain-driven-design`, `11-ddd-building-blocks`, `12-tradeoffs-and-choosing`.

---

# Deliverable 1 — software-architecture study notes

### Task 1: Note 1 — core design principles

**Files:**
- Create: `🧠 brain/software-architecture/1-core-design-principles.md`

**Level tag:** `#essential`

- [ ] **Step 1: Write the note**

Cover, in this order:
- Framing: architecture is mostly about managing ==dependencies== and ==change==.
- ==Separation of Concerns (SoC)==: split the system into modules with distinct, non-overlapping jobs. Example: don't mix business calculation, DB access, and HTTP handling in one place.
- ==Loose Coupling==: minimize connections between modules so they change independently.
- ==High Cohesion==: keep related things together in a module.
- Closing line: low coupling + high cohesion is the recurring target of every later pattern.

Related Notes:
- `[[2-solid-principles]]` — the object-level rules that support these goals
- `[[4-domain-vs-infrastructure]]` — the first big separation to make
- `[[7-layered-architecture]]` — the most common way to separate concerns

- [ ] **Step 2: Verify** — run the per-note verification checklist above.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/1-core-design-principles.md"
git commit -m "docs: add core design principles note"
```

### Task 2: Note 2 — SOLID principles

**Files:**
- Create: `🧠 brain/software-architecture/2-solid-principles.md`

**Level tag:** `#essential`

- [ ] **Step 1: Write the note**

Cover all five, one short `##` section each, each with a one-line Go-ish or plain example:
- ==SRP== (Single Responsibility): one reason to change.
- ==OCP== (Open/Closed): open to extension, closed to modification.
- ==LSP== (Liskov Substitution): subtypes must be usable through the base type.
- ==ISP== (Interface Segregation): many small interfaces beat one fat interface.
- ==DIP== (Dependency Inversion): high-level modules (business logic) must not depend on low-level modules (DB, external API); both depend on ==abstractions== (interfaces).
- Add a short callout: DIP is the *principle*; DI (see next note) is the *technique*. Emphasize DIP is what later lets Clean/Hexagonal flip dependency direction.

Related Notes:
- `[[1-core-design-principles]]` — the higher-level goals SOLID serves
- `[[3-dependency-injection]]` — the technique that realizes DIP
- `[[8-hexagonal-architecture]]` — DIP applied to isolate the core
- `[[9-clean-architecture]]` — DIP as the dependency rule

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/2-solid-principles.md"
git commit -m "docs: add SOLID principles note"
```

### Task 3: Note 3 — dependency injection

**Files:**
- Create: `🧠 brain/software-architecture/3-dependency-injection.md`

**Level tag:** `#essential`

- [ ] **Step 1: Write the note**

Cover:
- ==Dependency Injection (DI)==: give an object its dependencies from outside instead of creating them inside.
- Go example (constructor injection):

```go
type Repository interface {
    Save(o Order) error
}

type Service struct {
    repo Repository
}

func NewService(repo Repository) *Service { // dependency passed in
    return &Service{repo: repo}
}
```

- Clarify the classic confusion: ==DIP = design principle== (depend on abstractions); ==DI = technique== (pass dependencies in). DIP is usually realized via DI.
- Why DI makes testing easy: pass a fake/mock that satisfies the interface. (This is the only place testing gets its own callout.)

Related Notes:
- `[[2-solid-principles]]` — DIP, the principle DI helps achieve
- `[[5-ports-and-adapters]]` — where injected interfaces become ports

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/3-dependency-injection.md"
git commit -m "docs: add dependency injection note"
```

### Task 4: Note 4 — domain vs infrastructure

**Files:**
- Create: `🧠 brain/software-architecture/4-domain-vs-infrastructure.md`

**Level tag:** `#essential`

- [ ] **Step 1: Write the note**

Cover:
- ==Domain==: pure business rules that exist without any framework or DB. Examples: "a coupon cannot be used twice", "cannot pay if balance is too low".
- ==Infrastructure==: technical tools that run and support the domain. Examples: MySQL, Redis, a web framework, REST API, AWS S3.
- Why the split matters: it is the line every later pattern draws.
- Note this is the foundational half of DDD, learned early before the full DDD notes.

Related Notes:
- `[[5-ports-and-adapters]]` — how the domain talks to infrastructure
- `[[8-hexagonal-architecture]]` — domain in the center, infra outside
- `[[10-domain-driven-design]]` — modeling the domain in depth

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/4-domain-vs-infrastructure.md"
git commit -m "docs: add domain vs infrastructure note"
```

### Task 5: Note 5 — ports and adapters

**Files:**
- Create: `🧠 brain/software-architecture/5-ports-and-adapters.md`

**Level tag:** `#understand`

- [ ] **Step 1: Write the note**

Cover:
- ==Port==: a standardized interface into or out of the business logic (the hole).
- ==Adapter==: the concrete implementation that fits a port (the plug). Examples: a MySQL adapter, an external PG (payment gateway) adapter.
- Inbound vs outbound ports (driving vs driven), briefly.
- Go example:

```go
// Port (defined by the core)
type PaymentPort interface {
    Pay(amount int) error
}

// Adapter (infrastructure implements the port)
type TossPaymentAdapter struct{ /* client, config */ }

func (a TossPaymentAdapter) Pay(amount int) error { /* call Toss API */ return nil }
```

Related Notes:
- `[[3-dependency-injection]]` — how the adapter is supplied to the core
- `[[6-dto-and-mappers]]` — how data crossing a port is shaped
- `[[8-hexagonal-architecture]]` — the pattern built on ports and adapters

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/5-ports-and-adapters.md"
git commit -m "docs: add ports and adapters note"
```

### Task 6: Note 6 — DTO and mappers

**Files:**
- Create: `🧠 brain/software-architecture/6-dto-and-mappers.md`

**Level tag:** `#understand`

- [ ] **Step 1: Write the note**

Cover:
- ==DTO (Data Transfer Object)==: a plain object to move data across a boundary.
- ==Mapper==: converts between DTOs and domain objects.
- Why: outside data (JSON, DB rows) must not leak straight into business logic. DTO + mapper act as a ==firewall== that keeps layers independent.
- Go example: a `CreateOrderRequest` DTO mapped to an `Order` domain struct.

```go
type CreateOrderRequest struct { // DTO from the outside
    SKU   string `json:"sku"`
    Count int    `json:"count"`
}

func toOrder(r CreateOrderRequest) Order { // mapper
    return Order{SKU: r.SKU, Count: r.Count}
}
```

Related Notes:
- `[[5-ports-and-adapters]]` — DTOs travel through ports
- `[[4-domain-vs-infrastructure]]` — the boundary DTOs protect

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/6-dto-and-mappers.md"
git commit -m "docs: add DTO and mappers note"
```

### Task 7: Note 7 — layered architecture

**Files:**
- Create: `🧠 brain/software-architecture/7-layered-architecture.md`

**Level tag:** `#understand`

- [ ] **Step 1: Write the note**

Cover:
- Structure: ==Presentation → Business → Data Access==. Include a simple ASCII stack diagram.
- The most intuitive and common layout.
- The ==trap==: business logic easily couples to the data layer, so DB changes ripple into business code.
- Forward pointer: Hexagonal and Clean fix this by inverting the dependency on the data layer.

Related Notes:
- `[[1-core-design-principles]]` — the separation this pattern applies
- `[[8-hexagonal-architecture]]` — the fix for the coupling trap
- `[[9-clean-architecture]]` — a stricter fix
- `[[12-tradeoffs-and-choosing]]` — when layered is enough

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/7-layered-architecture.md"
git commit -m "docs: add layered architecture note"
```

### Task 8: Note 8 — hexagonal architecture

**Files:**
- Create: `🧠 brain/software-architecture/8-hexagonal-architecture.md`

**Level tag:** `#understand`

- [ ] **Step 1: Write the note**

Cover:
- ==Inside (core)== vs ==Outside (adapters)==. Include an ASCII hexagon or box diagram.
- Business core in the center; all technology pushed out to adapters that plug into ports.
- Built on ports & adapters (note 5) and DIP (note 2).
- Go sketch: the core depends on a port interface; a DB adapter implements it (reuse the `PaymentPort` idea or a `Repository` port).

Related Notes:
- `[[2-solid-principles]]` — DIP, the mechanism behind it
- `[[5-ports-and-adapters]]` — the building blocks
- `[[4-domain-vs-infrastructure]]` — what goes inside vs outside
- `[[9-clean-architecture]]` — the same idea, generalized

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/8-hexagonal-architecture.md"
git commit -m "docs: add hexagonal architecture note"
```

### Task 9: Note 9 — clean architecture

**Files:**
- Create: `🧠 brain/software-architecture/9-clean-architecture.md`

**Level tag:** `#understand`

- [ ] **Step 1: Write the note**

Cover:
- Concentric circles: ==Entities → Use Cases → Interface Adapters → Frameworks==. Include an ASCII circles diagram.
- ==The Dependency Rule==: source-code dependencies point only inward. Inner circles know nothing about outer circles.
- Relationship to Hexagonal: Clean generalizes the same idea into an explicit rule.
- Note DIP is how the inner-pointing rule is enforced in practice.

Related Notes:
- `[[8-hexagonal-architecture]]` — the close cousin
- `[[2-solid-principles]]` — DIP enforces the dependency rule
- `[[11-ddd-building-blocks]]` — what lives in the entity core
- `[[12-tradeoffs-and-choosing]]` — the cost of this strictness

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/9-clean-architecture.md"
git commit -m "docs: add clean architecture note"
```

### Task 10: Note 10 — domain-driven design

**Files:**
- Create: `🧠 brain/software-architecture/10-domain-driven-design.md`

**Level tag:** `#understand`

- [ ] **Step 1: Write the note**

Cover:
- DDD is a ==methodology==, not a file layout.
- ==Ubiquitous Language==: business experts and developers use one shared language to design the domain.
- ==Bounded Context==: a boundary where a model and its language stay consistent.
- How DDD complements Clean/Hexagonal: those give the protected shell, DDD fills the core with a rich domain model.

Related Notes:
- `[[4-domain-vs-infrastructure]]` — the domain idea this builds on
- `[[11-ddd-building-blocks]]` — the tactical objects of DDD
- `[[9-clean-architecture]]` — the shell DDD fills

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/10-domain-driven-design.md"
git commit -m "docs: add domain-driven design note"
```

### Task 11: Note 11 — DDD building blocks

**Files:**
- Create: `🧠 brain/software-architecture/11-ddd-building-blocks.md`

**Level tag:** `#understand`

- [ ] **Step 1: Write the note**

Cover tactical patterns, one short section each:
- ==Entity==: has identity that persists over time.
- ==Value Object==: defined only by its values, immutable, no identity.
- ==Aggregate==: a cluster of objects treated as one unit, with an aggregate root.
- ==Repository==: gives collection-like access to aggregates and hides storage.
- Small Go examples where useful (e.g. a `Money` value object; an `Order` aggregate root).
- Note the Repository is where DDD meets ports/adapters (note 5).

```go
type Money struct { // value object: compared by value, immutable
    Amount   int
    Currency string
}
```

Related Notes:
- `[[10-domain-driven-design]]` — the methodology these serve
- `[[5-ports-and-adapters]]` — the Repository as a port
- `[[4-domain-vs-infrastructure]]` — these objects live in the domain

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/11-ddd-building-blocks.md"
git commit -m "docs: add DDD building blocks note"
```

### Task 12: Note 12 — tradeoffs and choosing

**Files:**
- Create: `🧠 brain/software-architecture/12-tradeoffs-and-choosing.md`

**Level tag:** `#essential`

- [ ] **Step 1: Write the note**

Cover:
- Honest ==benefits==: easy unit testing, freedom to swap framework/DB, clear protection of business logic.
- Honest ==costs==: many more files/classes (boilerplate), simple CRUD must pass through several layers, steeper learning curve.
- ==When NOT to use== full Clean/Hexagonal: small CRUD apps, thin domains, short-lived projects. Match the architecture to the complexity of the domain.
- Closing: no architecture is always right; understand the price.

Related Notes:
- `[[7-layered-architecture]]` — the cheaper default
- `[[8-hexagonal-architecture]]` — cost vs isolation
- `[[9-clean-architecture]]` — cost vs strictness
- `[[10-domain-driven-design]]` — worth it only for rich domains

- [ ] **Step 2: Verify** — run the per-note verification checklist.

- [ ] **Step 3: Commit**

```bash
git add "🧠 brain/software-architecture/12-tradeoffs-and-choosing.md"
git commit -m "docs: add tradeoffs and choosing note"
```

### Task 13: Cross-link verification pass (Deliverable 1)

**Files:**
- Read: all 12 files in `🧠 brain/software-architecture/`

- [ ] **Step 1: Check every wikilink resolves**

Run:
```bash
cd "🧠 brain/software-architecture"
# List every wikilink target used
grep -rho "\[\[[^]]*\]\]" . | sed 's/\[\[//;s/\]\]//' | sort -u
# List actual filenames (without .md)
ls *.md | sed 's/\.md$//'
```
Expected: every wikilink target on the first list appears in the second list. Fix any mismatch (typo or missing file).

- [ ] **Step 2: Check format basics across all files**

Run:
```bash
cd "🧠 brain/software-architecture"
for f in *.md; do
  head -1 "$f" | grep -q "#ai-draft" || echo "MISSING tag: $f"
  tail -3 "$f" | grep -q "Review History" || echo "MISSING Review History: $f"
done
```
Expected: no output. Fix any file that prints.

- [ ] **Step 3: Commit any fixes** (skip if nothing changed)

```bash
git add "🧠 brain/software-architecture"
git commit -m "docs: fix architecture note cross-links"
```

---

# Deliverable 2 — pricing-tool Architecture.md

### Task 14: Explore and audit the pricing-tool codebase

**Files:**
- Read-only: `/Users/yeongseo.na/IdeaProjects/pricing-tool` (do NOT modify)
- Scratch (optional): keep findings in working notes for Task 15/16

**Goal of this task:** gather concrete, file-anchored evidence. No prose writing yet.

- [ ] **Step 1: Map the layers**

Run and record output:
```bash
cd /Users/yeongseo.na/IdeaProjects/pricing-tool
find internal -maxdepth 2 -type d | sort
find cmd -maxdepth 2 -type d | sort
```
Record which dirs are inbound (`internal/interface`), use-case (`internal/service`, `internal/serviceutils`), domain (`internal/domain`), and outbound infra (`internal/bigquery`, `internal/pubsub`, `internal/pkg`).

- [ ] **Step 2: Confirm the "stays clean" mechanisms**

For each item below, find a concrete file/line as evidence:
- Repository interfaces as ports (e.g. `internal/domain/repository/item/repository.go` → `type TransactionalGorm interface`).
- mockery config: `.mockery.yml`; find a generated mock file.
- goa design-first: `internal/interface/design`.
- multiple entry-point binaries: `cmd/server/*`.
- cross-cutting helpers: `internal/pkg/transactioner`, `internal/pkg/ctxvalues`.

- [ ] **Step 3: Gather audit evidence (thorough)**

For each theme, record specific files/packages that show the issue:
```bash
cd /Users/yeongseo.na/IdeaProjects/pricing-tool
# Layering leak: gorm inside domain
grep -rl "gorm" internal/domain --include="*.go" | grep -v _test | head
# God packages: largest packages by file count
for d in internal/service internal/domain/repository/item internal/pkg; do echo "$d: $(ls "$d" | wc -l) entries"; done
# v1/v2 duplication
find internal -name "*v2*" -o -name "*2.go" | grep -v _test | head
ls internal/service | grep -i master
# Naming: service vs serviceutils split
ls internal/service | head; ls internal/serviceutils | head
```
Record findings grouped by theme (layering, god packages, naming, testability, dependency direction, duplication) with a high/medium/low severity guess each.

- [ ] **Step 4: Sanity-check dependency direction**

Run:
```bash
cd /Users/yeongseo.na/IdeaProjects/pricing-tool
# Does the domain import service or interface layers? (should be none)
grep -rn "pricing-tool/internal/service\|pricing-tool/internal/interface" internal/domain --include="*.go" | grep -v _test | head
```
Record whether any inner layer imports an outer layer.

- [ ] **Step 5: No commit** (investigation only). Confirm you have file-anchored evidence for every "stays clean" point and every audit finding before moving on.

### Task 15: Write Architecture.md — overview and how it stays clean

**Files:**
- Create/fill: `🐟 reference/pricing-tool/Architecture.md`

**Format:** reference-path — NO tags, NO numbering, NO Review History. Use `==highlight==` and wikilinks to `[[API Design & REST Principles]]` and `[[Error Handling]]`.

- [ ] **Step 1: Write sections 1–3**

1. **What the product is** — pricing-experiments tool for pricing managers (from README).
2. **The big picture** — the DDD-flavored layered / Clean-influenced design. Include an ASCII diagram, e.g.:

```
        inbound                     outbound
 cmd/server ──▶ interface ──▶ service ──▶ domain ◀── bigquery / pubsub / pkg
 (binaries)     (goa,          (use       (entities,      (DB, Kafka, PubSub,
                controllers)   cases)     aggregates,      BigQuery, buckets)
                                          repo ports)
        gen/  = goa-generated glue
```
   Explain dependency direction points toward the domain.
3. **How it keeps the architecture clean** — one short paragraph or bullet per mechanism from Task 14 Step 2, each citing a concrete file/package. Link REST points to `[[API Design & REST Principles]]` and error points to `[[Error Handling]]`.

- [ ] **Step 2: Verify** — file has no tags/numbering/Review History; wikilinks name real sibling files (`API Design & REST Principles.md`, `Error Handling.md`); every "stays clean" claim names a real file.

- [ ] **Step 3: Commit**

```bash
git add "🐟 reference/pricing-tool/Architecture.md"
git commit -m "docs: add pricing-tool architecture overview"
```

### Task 16: Write Architecture.md — thorough audit section

**Files:**
- Modify: `🐟 reference/pricing-tool/Architecture.md`

- [ ] **Step 1: Add section 4 — Suggested improvements (thorough audit)**

Write a `## Suggested Improvements` section. Group findings by theme; for EACH finding give: the concrete file/package evidence, why it hurts the architecture, a concrete fix, and a severity (high/medium/low). Themes to include (only those with real evidence from Task 14):
- Layering violations (e.g. gorm implementations inside `internal/domain/repository/*` couple the domain to gorm; move to an infra/adapter package).
- God packages / oversized packages (e.g. large `internal/service`, `internal/domain/repository/item`); how to split.
- Naming consistency (`service` vs `serviceutils`, `handle*` services, `manager` suffixes).
- Testability gaps (code depending on concretes where an interface/mock is missing).
- Dependency-direction issues (any inner layer importing an outer one).
- Duplicated/parallel implementations (e.g. `masterproduct` vs `masterproductv2`, `*2.go` files).

- [ ] **Step 2: Verify** — every finding cites a concrete file/package (no generic advice); severities present; at least one ASCII diagram exists in the file; pricing-tool project is unmodified (`cd /Users/yeongseo.na/IdeaProjects/pricing-tool && git status --short` shows no changes).

- [ ] **Step 3: Commit**

```bash
git add "🐟 reference/pricing-tool/Architecture.md"
git commit -m "docs: add pricing-tool architecture audit"
```

---

## Self-Review (completed by plan author)

**Spec coverage:**
- Deliverable 1: all 12 notes have a task (Tasks 1–12) with level tags matching the spec; cross-link pass (Task 13) covers the wikilink-map requirement. ✓
- Deliverable 2: overview + big picture + how-it-stays-clean (Task 15) and thorough audit (Task 16), with exploration (Task 14) feeding them. Reference-path format rules and read-only constraint captured. ✓

**Placeholder scan:** Content points per note are concrete and drawn from the spec; the actual prose is written during execution (expected for a docs task). No "TBD"/"similar to Task N" left in steps. ✓

**Type consistency:** Wikilink target names match the file list exactly (e.g. `4-domain-vs-infrastructure` used consistently). Go snippet names (`Repository`, `PaymentPort`, `Money`) are illustrative and self-contained per note. ✓
