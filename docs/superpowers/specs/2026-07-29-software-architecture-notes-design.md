# Software Architecture Notes — Design Spec

_Date: 2026-07-29_

## Goal

Create a fine-grained set of study notes on software architecture in the
`🧠 brain/software-architecture/` folder (currently empty). The notes teach a
junior backend developer the **core principles and dependency-control
mechanisms** behind domain-centric architectures — not just the shape of each
diagram.

The guiding idea: individual architectures (Layered, Hexagonal, Clean, DDD) are
different answers to the **same** problem — controlling dependencies so business
logic stays isolated from technical details. The notes must make that shared
purpose visible through wikilinks and repeated principles.

## Audience & style

- Junior backend developer, quick-reference + study use.
- Simple, short English sentences. One idea per sentence.
- Code examples in **Go** (interfaces map cleanly to ports / DIP).
- Examples small and focused, not full applications.

## Format rules (every file must follow these)

These come from the project `CLAUDE.md` and the existing API folder style:

1. **Line 1:** two tags — one level tag + one status tag.
   - Level tags: `#aware` / `#understand` / `#essential` (assigned per file below).
   - Status tag: `#ai-draft` for all new files.
2. Blank line, then `# Title`.
3. The first `##` section right after the title has **no** `---` before it.
4. Every other `##` section has `---` on the line before it.
5. Use `==text==` to highlight key terms and important ideas.
6. Sentences inside a paragraph go on consecutive lines. No blank line between
   sentences in the same paragraph.
7. Second-to-last section: `## Related Notes` with Obsidian wikilinks
   (`[[filename]]`) and a short dash description each.
8. Last section (always): `## Review History` with a single `-` bullet.
9. Files are numbered by learning order (`1-...`, `2-...`). Numbers show read order.

## Learning order rationale

Principles → vocabulary → structural patterns → DDD → judgment.

- You learn **why** first (principles), then the **words** (core terms), then the
  **shapes** (patterns), then the **reward** (DDD), then the **judgment**
  (tradeoffs + how to choose).
- Structural patterns are ordered by increasing sophistication: Layered (naive
  default and its trap) → Hexagonal (fixes the trap with ports & adapters) →
  Clean (generalizes it into the dependency rule). Each motivates the next.
- The foundational half of DDD — "what is the domain, keep it pure" — is
  front-loaded at file #4. The full DDD methodology (strategic + tactical) comes
  at #10–#11 because it is the payoff that only makes sense after you know why
  you'd protect a core.

## File list

| # | Filename | Level | Purpose (one line) |
|---|----------|-------|--------------------|
| 1 | `1-core-design-principles` | `#essential` | SoC, Loose Coupling, High Cohesion — the goals every pattern serves. |
| 2 | `2-solid-principles` | `#essential` | All 5 SOLID; DIP highlighted as the key to Clean/Hexagonal. |
| 3 | `3-dependency-injection` | `#essential` | DI as a technique; clears up DIP-vs-DI confusion. |
| 4 | `4-domain-vs-infrastructure` | `#essential` | The central split: pure business rules vs technical tools. |
| 5 | `5-ports-and-adapters` | `#understand` | Port = interface (the hole), Adapter = implementation (the plug). |
| 6 | `6-dto-and-mappers` | `#understand` | DTOs + mapping as the firewall between layers. |
| 7 | `7-layered-architecture` | `#understand` | Presentation → Business → Data; common, and its coupling trap. |
| 8 | `8-hexagonal-architecture` | `#understand` | Core inside, adapters outside; built on ports & adapters. |
| 9 | `9-clean-architecture` | `#understand` | Concentric circles; the Dependency Rule (point inward only). |
| 10 | `10-domain-driven-design` | `#understand` | Strategic DDD: ubiquitous language, bounded context; why it's a method. |
| 11 | `11-ddd-building-blocks` | `#understand` | Tactical DDD: Entity, Value Object, Aggregate, Repository. |
| 12 | `12-tradeoffs-and-choosing` | `#essential` | Honest costs + a "when NOT to use this" decision guide. Capstone. |

## Per-file content specs

### 1 — `1-core-design-principles` `#essential`
- Frame: architecture is mostly about managing dependencies and change.
- ==Separation of Concerns (SoC)==: split the system into modules with distinct,
  non-overlapping jobs. Example: don't mix business calculation, DB access, and
  HTTP handling in one place.
- ==Loose Coupling==: minimize connections between modules so they can change
  independently.
- ==High Cohesion==: keep related things together inside a module.
- Short note that low coupling + high cohesion is the recurring target of every
  later pattern.
- Related: `[[2-solid-principles]]`, `[[4-domain-vs-infrastructure]]`,
  `[[7-layered-architecture]]`.

### 2 — `2-solid-principles` `#essential`
- All five: SRP, OCP, LSP, ISP, DIP. One short section each with a one-line Go-ish
  example or plain example.
- ==DIP (Dependency Inversion Principle)==: high-level modules (business logic)
  must not depend on low-level modules (DB, external API); both depend on
  ==abstractions== (interfaces).
- Emphasize DIP as the mechanism that later lets Clean/Hexagonal flip the
  dependency direction. This is the bridge to the pattern notes.
- Contrast note: DIP is the *principle*; DI (#3) is the *technique*.
- Related: `[[1-core-design-principles]]`, `[[3-dependency-injection]]`,
  `[[8-hexagonal-architecture]]`, `[[9-clean-architecture]]`.

### 3 — `3-dependency-injection` `#essential`
- ==Dependency Injection (DI)==: give an object its dependencies from outside
  instead of creating them inside.
- Go example: a service that receives a repository interface via its constructor
  (`NewService(repo Repository)`), not one that news up a concrete DB struct.
- Clear the classic confusion: ==DIP = design principle==, ==DI = technique that
  helps achieve it==. You can do DI without DIP, and DIP is usually realized via DI.
- Note on why DI makes testing easy (pass a fake). This is the only place testing
  gets its own callout (testing-by-layer was intentionally left out as a file).
- Related: `[[2-solid-principles]]`, `[[5-ports-and-adapters]]`.

### 4 — `4-domain-vs-infrastructure` `#essential`
- ==Domain==: pure business rules that exist without any framework or DB.
  Examples: "a coupon cannot be used twice", "cannot pay if balance is too low".
- ==Infrastructure==: technical tools that run and support the domain. Examples:
  MySQL, Redis, Spring Boot / web framework, REST API, AWS S3.
- Why the split matters: it is the line every later pattern draws.
- This note carries the foundational half of the DDD idea (the domain concept)
  early, before the full DDD notes.
- Related: `[[5-ports-and-adapters]]`, `[[8-hexagonal-architecture]]`,
  `[[10-domain-driven-design]]`.

### 5 — `5-ports-and-adapters` `#understand`
- ==Port==: a standardized interface into or out of the business logic (the hole).
- ==Adapter==: the concrete implementation that fits a port (the plug). Examples:
  a MySQL adapter, an external PG (payment gateway) adapter.
- Inbound vs outbound ports (driving vs driven) — brief.
- Go example: a `PaymentPort` interface + a `TossPaymentAdapter` implementing it.
- Related: `[[3-dependency-injection]]`, `[[6-dto-and-mappers]]`,
  `[[8-hexagonal-architecture]]`.

### 6 — `6-dto-and-mappers` `#understand`
- ==DTO (Data Transfer Object)==: a plain object to move data across a boundary.
- ==Mapper==: converts between DTOs and domain objects.
- Why: outside data (JSON, DB rows) must not leak straight into business logic.
  The DTO + mapper acts as a ==firewall== that keeps layers independent.
- Go example: `CreateOrderRequest` (DTO) → mapper → `Order` (domain).
- Related: `[[5-ports-and-adapters]]`, `[[4-domain-vs-infrastructure]]`.

### 7 — `7-layered-architecture` `#understand`
- Structure: ==Presentation → Business → Data Access==.
- Most intuitive and common.
- The trap: business logic easily leaks into / couples to the data layer, so DB
  changes ripple into business code.
- Forward pointer: Hexagonal and Clean fix this trap by inverting the dependency
  on the data layer.
- Related: `[[1-core-design-principles]]`, `[[8-hexagonal-architecture]]`,
  `[[9-clean-architecture]]`, `[[12-tradeoffs-and-choosing]]`.

### 8 — `8-hexagonal-architecture` `#understand`
- ==Inside (core)== vs ==Outside (adapters)==.
- Business core sits in the center; all technology is pushed out to adapters that
  plug into ports.
- Built directly on ports & adapters (#5) and DIP (#2).
- Simple diagram (ASCII hexagon or box) plus a Go sketch: core depends on a port
  interface; the DB adapter implements it.
- Related: `[[2-solid-principles]]`, `[[5-ports-and-adapters]]`,
  `[[4-domain-vs-infrastructure]]`, `[[9-clean-architecture]]`.

### 9 — `9-clean-architecture` `#understand`
- Concentric circles: ==Entities → Use Cases → Interface Adapters → Frameworks==.
- ==The Dependency Rule==: source-code dependencies point only inward. Inner
  circles know nothing about outer circles.
- Relationship to Hexagonal: Clean generalizes the same idea into an explicit rule.
- Note DIP is how the inner-pointing rule is enforced in practice.
- Related: `[[8-hexagonal-architecture]]`, `[[2-solid-principles]]`,
  `[[11-ddd-building-blocks]]`, `[[12-tradeoffs-and-choosing]]`.

### 10 — `10-domain-driven-design` `#understand`
- DDD is a ==methodology==, not a file layout.
- ==Ubiquitous Language==: business experts and developers use one shared language
  to design the domain.
- ==Bounded Context==: a boundary where a model and its language are consistent.
- Explain how DDD complements Clean/Hexagonal: those give the protected shell,
  DDD fills the core with a rich domain model.
- Related: `[[4-domain-vs-infrastructure]]`, `[[11-ddd-building-blocks]]`,
  `[[9-clean-architecture]]`.

### 11 — `11-ddd-building-blocks` `#understand`
- Tactical patterns:
  - ==Entity==: has identity that persists over time.
  - ==Value Object==: defined only by its values, immutable, no identity.
  - ==Aggregate==: a cluster of objects treated as one unit, with an aggregate root.
  - ==Repository==: gives collection-like access to aggregates, hides storage.
- One small Go example per concept where it helps (e.g. a `Money` value object,
  an `Order` aggregate root).
- Note the Repository is where DDD meets ports/adapters (#5).
- Related: `[[10-domain-driven-design]]`, `[[5-ports-and-adapters]]`,
  `[[4-domain-vs-infrastructure]]`.

### 12 — `12-tradeoffs-and-choosing` `#essential`
- Honest ==benefits==: easy unit testing, freedom to swap framework/DB, clear
  protection of business logic.
- Honest ==costs==: many more files/classes (boilerplate), simple CRUD must pass
  through several layers, steeper learning curve.
- ==When NOT to use== full Clean/Hexagonal: small CRUD apps, thin domains,
  short-lived projects. Match the architecture to the complexity of the domain.
- Reinforce: no architecture is always right; understand the price.
- Related: `[[7-layered-architecture]]`, `[[8-hexagonal-architecture]]`,
  `[[9-clean-architecture]]`, `[[10-domain-driven-design]]`.

## Wikilink map (the shared-problem web)

- DIP (#2) is referenced from #3, #8, #9 — the mechanism that inverts dependencies.
- Domain vs Infra (#4) and Ports & Adapters (#5) are referenced from #8, #9, #11 —
  the shared vocabulary reused by every pattern.
- Layered (#7) links forward to #8/#9 as "the fix" for its coupling trap.
- Tradeoffs (#12) links back to all four patterns.

## Out of scope

- No separate "testing strategy by layer" file (a short DI/testing callout lives in
  #3 instead).
- No CQRS, event sourcing, or microservices notes.
- No intro/index file (the folder relies on numbered order, like the API folder).
- `personal/` and `reference/` rules do not apply here; these are `brain/` notes.

## Definition of done

- 12 files created in `🧠 brain/software-architecture/`, numbered 1–12.
- Each file follows all format rules above (tags, `---` placement, highlights,
  Related Notes, Review History).
- All wikilinks resolve to real filenames in the list.
- Go examples compile-plausibly (syntactically sane), small, and illustrative.
