# Pricing Tool — Architecture Audit Findings
_Date: 2026-07-29_

---

## A. Product & Structure

### What the product is

`pricing-tool` (Go module `pricing-tool`) is a backend service that helps Pricing Managers run pricing experiments fast and easily to optimise earnings. The README states: "Pricing tool Help Pricing Managers run pricing experiments fast and easy to optimize on earnings." The system includes an experiment state machine (creation_pending → created → segmentation submitted → calculation → price loading → benchmark/simulation), competitor data loading, BigQuery analytics, PubSub/Kafka event processing, and price-change orchestration.

---

### Top-level layout

```
cmd/
  server/
    main.go              — single binary entry point
    server/              — HTTP API server (goa endpoints)
    dailycron/           — nightly operations runner
    archivecron/         — price-change log archival cron
    purgenotificationscron/ — notifications TTL purge cron
    jobpoller/           — polling-based job runner
    kafkaoutboxpoller/   — Kafka outbox relay poller
    inboundservice/      — PubSub inbound subscription runner
    warehouseoperations/ — warehouse sync runner
    boilerplate/         — shared run/monitoring helpers
  scripts/               — one-off admin scripts
  testing/               — integration test runner entry point
internal/                — compiler-enforced boundary (all app code)
gen/                     — goa v3 generated HTTP glue (45 subdirs, openapi3.yaml)
config.yaml / config-test.yaml / config-local-to-stg-db.yaml
.mockery.yml             — mockery v2 config
go.mod / go.sum
```

---

### Key layers

| Layer | Path | Role |
|---|---|---|
| Inbound adapters | `internal/interface/` | HTTP controllers, goa design-first DSL, JWT auth, error handler |
| Use-case / application | `internal/service/` + `internal/serviceutils/` | Business logic; 62 subdirectories |
| Domain model | `internal/domain/` | Entities, aggregates, domain errors, repository *ports* |
| Infrastructure (outbound) | `internal/bigquery/`, `internal/pubsub/`, `internal/pkg/*manager` | BQ, PubSub, Kafka, DB, bucket, audit log |
| Generated | `gen/` | goa HTTP stubs, OpenAPI spec |

**`internal/interface/` sub-packages:**
- `controller/` — one sub-dir per resource (40+ controllers)
- `design/` — goa DSL definitions (`design.go`, `experiment.go`, `master_product.go`, etc.)
- `jwtauthorizer/` — middleware for JWT validation
- `errorhandler/` — centralised error mapping

**`internal/service/`** — 62 subdirectories covering every resource and background use-case.

**`internal/serviceutils/`** — 16 sub-packages for reusable service helpers (e.g. `updateitemroleutils`, `changepriceutils`, `smartsegmentationutils`).

**`internal/domain/repository/`** — 52 subdirectories; each holds both the port interface and the GORM implementation (see Finding C1).

**Cross-cutting helpers in `internal/pkg/`:**
- `transactioner/` — `TransactionManagerInterface` + `TransactionalDB` for unit-of-work
- `ctxvalues/` — typed context value helpers
- `loggermanager/` — `LoggerI` interface over zap/logrus
- `databasemanager/`, `bigquerymanager/`, `pubsubmanager/`, `kafkamanager/`, `bucketmanager/`, `auditlogmanager/` — infrastructure managers
- `filtersmanager/`, `idgenerator/`, `workerpool/`, `concurency/`, etc.

---

## B. How It Stays Clean (Evidence)

### B1. Repository port pattern

`internal/domain/repository/item/repository.go` lines 15–17 define:

```go
type TransactionalGorm interface {
    WithContext(ctx context.Context) *gorm.DB
}
```

Every repository package defines a narrow `TransactionalGorm` (or similar) interface and a concrete `Repository` struct that satisfies it. Services depend on per-service `RepositoryI` interfaces (defined inside each service package), not on the concrete repository types.

### B2. Mockery usage confirmed

`.mockery.yml`:
```yaml
inpackage: true
all: true
filename: "mock_{{.InterfaceName}}_test.go"
structname: 'Mock{{.InterfaceName}}'
packages:
  pricing-tool/internal:
    config:
      all: true
      recursive: true
```

Generated mock files exist throughout the codebase, for example:
- `internal/domain/repository/item/mock_TransactionalGorm_test.go`
- `internal/service/item/mock_RepositoryI_test.go` (1119 lines)
- `internal/service/masterproduct/mock_RepositoryI_test.go` (767 lines)

### B3. Goa design-first confirmed

`internal/interface/design/design.go` opens with:
```go
var _ = dsl.API("pricing-tool", func() {
    dsl.Title("Pricing Tool")
    dsl.Version("1.0")
    ...
})
```

45 design files cover every endpoint (e.g. `master_product.go` at 654 lines, `notification.go` at 626 lines, `experiment_item_details.go` at 573 lines). The `gen/` directory is produced from these designs via `goa gen`.

### B4. `internal/` compiler boundary

Go's module system prevents any code outside this repository from importing `internal/` packages, enforcing the external API surface to only what `cmd/` exposes.

### B5. Cross-cutting helpers

- `internal/pkg/transactioner/` — `TransactionManagerInterface` with `NewTransaction(ctx, fn)` abstracts DB transactions
- `internal/pkg/ctxvalues/ctx_values.go` — typed helpers for reading values from `context.Context`
- `internal/pkg/loggermanager/logger.go` — `LoggerI` interface
- Multiple `*manager` packages wrap external clients behind interfaces

---

## C. Audit Findings

### C1. Layering violation — GORM implementation inside `internal/domain/`

**Severity: High**

**Files:** Every subdirectory of `internal/domain/repository/` — confirmed for `item/`, `statistics/`, `competitor/`, `vendors/`, `zone/`, `excludedsku/`, `masterproductsuppliers/`, `zonevendor/`, `skumanualmatches/`, `processtracker/`, `catalogvendors/`, and more.

**Evidence:**
```
internal/domain/repository/item/list.go:7:   "gorm.io/gorm"
internal/domain/repository/item/list_with_additional.go:8: "gorm.io/gorm"
internal/domain/repository/item/repository.go:13: "gorm.io/gorm"
internal/domain/repository/item/batch_upsert.go:9: "gorm.io/gorm/clause"
internal/domain/repository/item/set_item_roles_and_ranks.go:8: "gorm.io/gorm"
```

The `internal/domain/` package is supposed to be the pure domain layer (entities, aggregates, errors, and repository *ports* only). Instead, every repository subdirectory contains the concrete GORM implementation alongside the port interface. This couples the domain ring to a specific ORM.

**What's wrong:** Swapping GORM for another database library requires changing files inside `internal/domain/`, which should be stable and infrastructure-agnostic.

**Fix:** Move GORM implementations to `internal/infrastructure/repository/` (or `internal/gorm/`). Keep only entity definitions, domain errors, and repository interfaces in `internal/domain/`. The `TransactionalGorm` interface in `repository.go` can remain as the adapter boundary.

---

### C2. Layering violation — `internal/service` imports `internal/interface/controller/utils`

**Severity: High**

**Files:**
- `internal/service/item/list.go` — `cutils "pricing-tool/internal/interface/controller/utils"`
- `internal/service/item/listwithadditional.go`
- `internal/service/masterproduct/list_with_competitors_details.go`
- `internal/service/masterproduct/list_with_competitors_details_csv.go`
- `internal/service/vendors/get_productvendor_prices_csv.go`
- `internal/service/priceupdate/vendor_products.go`
- `internal/service/competitor/list_items.go`
- `internal/service/simulation/get_price_review_list.go`
- `internal/service/notifications/service.go`

The dependency direction is inverted. `internal/interface/controller/utils` lives in the inbound adapter layer; service layer code (a lower ring) must not import from a higher ring.

**What's wrong:** `controller/utils` contains pure utility functions (`StringToUUID`, `StringToDate`, `Round`, `Divide`, `ConvertToPercentage`, etc.) — none of which are HTTP-specific. They were placed in the controller package by convention but belong in a shared utility package.

**Fix:** Move `internal/interface/controller/utils/utils.go` to `internal/pkg/convutils/` (or similar). Both the controller and service layers can then import it without creating a ring violation.

---

### C3. Layering violation — domain test files import `internal/service/mockingutils`

**Severity: Medium**

**Files (test only, but still wrong):**
- `internal/domain/repository/statistics/*_test.go`
- `internal/domain/repository/vendors/*_test.go`
- `internal/domain/repository/competitor/*_test.go`
- `internal/domain/repository/zone/*_test.go`
- `internal/domain/repository/brand/fill_test.go`
- `internal/domain/repository/marketroundingparameters/get_test.go`

Example:
```
internal/domain/repository/statistics/upsert_general_statistics_test.go:
    "pricing-tool/internal/service/mockingutils"
```

Even in test files, `internal/domain` packages must not reach up to `internal/service`. This creates a hidden coupling and makes it impossible to test the domain layer without the service layer compiling.

**Fix:** Move `internal/service/mockingutils/mockingutils.go` to `internal/domain/testingutils/` (which already exists and has the right intent) or to `internal/pkg/testingutils/`.

---

### C4. God package — `internal/domain/repository/item/`

**Severity: Medium**

45 total files (35 non-test production files). The largest non-test files:

| File | Lines |
|---|---|
| `repository.go` (mapping functions) | 346 |
| `list_with_additional.go` | 576 |
| `list_with_competitor_prices.go` | 265 |
| `list.go` | 147 |
| `update_final_item_role2.go` | 145 |
| `update_final_item_role.go` | 127 |
| `set_item_roles_and_ranks.go` | 123 |
| `set_item_roles_predictions.go` | 108 |
| `list_with_categroy_details.go` | 102 |
| `items_with_extra_details.go` | 102 |
| Total (production) | ~2,800+ lines |

`repository.go` alone is a 346-line mapping file, not an interface definition. All read variants (`DenormalizedList`, `ListWithAdditional`, `ListWithCompetitorPrices`, `ListWithCategoryDetails`) live in one package with mixed responsibilities: mapping, SQL building, and GORM calls.

**Fix:** Split read projections into sub-packages (e.g. `item/listing/`, `item/roles/`, `item/benchmarks/`) aligned with use cases, after moving GORM impl out (see C1).

---

### C5. Duplicate / parallel implementations

**Severity: Medium**

**Pair 1: `masterproduct` vs `masterproductv2`**
- `internal/service/masterproduct/` — large service (30+ files) for full master product operations
- `internal/service/masterproductv2/` — separate service (`service.go`, `get_by_master_codes.go`, `get_details.go`, `list_master_products.go`, etc.) operating on `entity.MasterProductV2`
- `internal/domain/repository/masterproductv2/` — separate repository package
- `internal/domain/entity/master_product_v2.go` — parallel entity struct

Two independent code paths with overlapping responsibilities. It is unclear which is canonical.

**Pair 2: `update_final_item_role.go` vs `update_final_item_role2.go`**

Both files in `internal/domain/repository/item/`:
```go
// update_final_item_role.go
const calculatedFinalItemRole = "calculated_final_item_role"
func (r *Repository) UpdateFinalItemRole(...)

// update_final_item_role2.go
const calculatedFinalItemRole2 = "calculated_final_item_role"  // same value!
func (r *Repository) UpdateFinalItemRole2(...)
```

Both constants resolve to the same string `"calculated_final_item_role"`. The two functions implement similar but slightly different SQL strategies for computing final item roles. There is no comment explaining why both exist or which should be used when.

**Fix for both:** Document which version is current. Delete the superseded version or unify them under a strategy parameter. The `v2` pattern signals unfinished migration.

---

### C6. Naming inconsistency — `handle*` services mixed with domain services

**Severity: Low**

14 service packages start with `handle`:
```
handleautoloadcompetitorsglobally
handlecatalogvendorproductkafkastream
handlecatalogvendorsubscription
handledeletingexpiredscrappeddata
handlejobpoller
handlekafkaoutbox
handleloadingunmatchedcompetitors
handlemasterdetails
handleproductdetails
handlesharemainexresults
handlesimulationcalculation
handlesmartsegmentation
handlevendorproductdetails
handlevendorproductnotifications
handlevendorproductsupplierprice
```

These sit alongside domain-named services (`item/`, `zone/`, `experiment/`, `competitor/`). The `handle*` prefix suggests event-driven or orchestration use cases, but this distinction is not documented or enforced structurally. There is also a typo: `handezoneperformancesync` (missing the `l`).

**Fix:** Group event-handler services under `internal/service/handlers/` and domain services under `internal/service/domain/`. Fix the typo. Consider a top-level `internal/usecase/` distinction.

---

### C7. Naming inconsistency — `service` vs `serviceutils` split

**Severity: Low**

`internal/service/` (62 dirs) and `internal/serviceutils/` (16 dirs) exist side by side. The distinction is unclear: `serviceutils` contains `updateitemroleutils`, `changepriceutils`, `smartsegmentationutils` — these are not generic utilities but domain-specific operations. `internal/service/utils/` also exists. Three locations for "utility" code at the same layer.

**Fix:** Merge `serviceutils` sub-packages into the service package that owns them, or document a clear boundary rule.

---

### C8. Concrete type dependency — `processtracker.Tracker` in `item` service

**Severity: Low**

`internal/service/item/service.go` line 243:
```go
processTracker *processtracker.Tracker
```

`processtracker.Tracker` is a concrete struct (`type Tracker struct` in `internal/serviceutils/processtracker/process_tracker.go`). The `item` service holds a direct pointer to the concrete type, making it impossible to mock `processTracker` in unit tests without a real implementation. All other dependencies in the same struct use `I`-suffixed interfaces.

**Fix:** Extract a `ProcessTrackerI` interface from `processtracker.Tracker` and depend on that interface in `item.Service`.

---

### C9. Oversized single-function file

**Severity: Low**

`internal/service/zone/copyexpermentzone.go` — 462 lines containing a single exported function `CopyExperimentToZone`. The function orchestrates an entire transaction, copies experiment data, creates notifications, and sends PubSub messages. Note also the typo in the filename (`experment` instead of `experiment`).

**Fix:** Extract the steps inside `processCopyExperiment` into named helper functions in separate files. Fix the filename typo.

---

## Summary Table

| # | Theme | Severity | Path(s) |
|---|---|---|---|
| C1 | GORM impl inside `internal/domain/` | High | `internal/domain/repository/*/` |
| C2 | Service imports interface/controller/utils | High | 9 service files |
| C3 | Domain tests import service/mockingutils | Medium | `internal/domain/repository/*/` test files |
| C4 | God package: item repository (45 files) | Medium | `internal/domain/repository/item/` |
| C5 | Duplicate implementations (v2 twins) | Medium | `masterproductv2/`, `update_final_item_role2.go` |
| C6 | `handle*` naming mix + typo | Low | `internal/service/handle*/` |
| C7 | `service` vs `serviceutils` split unclear | Low | `internal/service/utils/`, `internal/serviceutils/` |
| C8 | Concrete `Tracker` struct in item service | Low | `internal/service/item/service.go:243` |
| C9 | 462-line single-function file + typo | Low | `internal/service/zone/copyexpermentzone.go` |
