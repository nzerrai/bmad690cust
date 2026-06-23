# Adversarial Boundary Review — GenData Architecture

## Verdict

The document fixes the high-level boundaries well but leaves seven concrete seams where two builders following every written rule will produce incompatible code.

---

## Scenarios constructed

### S-01: PRNG algorithm — same seed, different rows
**Builder A (typology-engine):** Implements `YamlTypologyParser` and the seed-to-PRNG wiring using Java's `java.util.Random(seed)` per column.
**Builder B (generation-engine):** Implements `RowGeneratorItemWriter` using `java.util.concurrent.ThreadLocalRandom` seeded from the dataset seed + column index for performance.
**Clash:** D-02 mandates `typology_id + version + sha256` reference for reproducibility (NFR-R03), but never specifies *which PRNG class* or *how the column-level seed is derived from the dataset seed* (e.g., `datasetSeed XOR columnIndex`? `datasetSeed + columnIndex`?). Builder A and B each comply with D-02 and still produce different rows for identical inputs.
**Severity:** Critical
**Fix:** Add a binding rule: "Column seed = `Long.hashCode(datasetSeed) ^ columnIndex`; PRNG = `java.util.Random(columnSeed)` — no other generator allowed."

---

### S-02: GenerationJob ownership — write conflict on status field
**Builder A (generation-engine):** `GenerationJobConfig` (Spring Batch listener) writes `RUNNING`, `COMPLETED`, `FAILED` to the `generation_jobs` table via `GenerationJobListener`.
**Builder B (dataset-api):** `GenerationService.submitJob()` creates the job row with `PENDING` and also has a `cancelJob()` method that writes `CANCELLED` to `generation_jobs`.
**Clash:** Both packages own write access to `generation_jobs.status`. There is no rule about who is the single writer or what the legal state-transition graph is (e.g., can `dataset-api` set `CANCELLED` while Batch is mid-write of `COMPLETED`?). A race produces a permanently inconsistent status.
**Severity:** Critical
**Fix:** Declare a single writer rule: "Only `generation-engine` (Spring Batch listener) writes `status` transitions after job submission; `dataset-api` calls a `generationEngine.requestCancel(jobId)` method and never writes `status` directly."

---

### S-03: SpaceSecurityFilter scope — DDL endpoints uncovered
**Builder A (security):** Implements `SpaceSecurityFilter` covering `/api/workspaces/{id}/**` exactly as D-05 states.
**Builder B (ddl-service):** Implements `DdlController` under `/api/ddl/{did}` (a natural flat path for a cross-cutting service, matching the `com.gendata.ddl` package which is peer to `workspace` and `dataset`).
**Clash:** D-05's filter path `/api/workspaces/{id}/**` does not cover `/api/ddl/**`. Builder B's DDL endpoints bypass RBAC entirely. The document maps DDL to FR34–40 under `dataset-api` in one table, but places the code in `com.gendata.ddl` which has no declared URL prefix — two builders choose incompatible paths.
**Severity:** Critical
**Fix:** Declare the canonical URL prefix for DDL: `/api/workspaces/{wid}/projects/{pid}/subprojects/{spid}/ddl/{did}/**` — this places it under the `SpaceSecurityFilter` path automatically.

---

### S-04: Export + RBAC — cross-space leak risk
**Builder A (export):** Implements `ExportController` at `/api/export/{datasetId}` with `StreamingResponseBody` (D-09). Calls `DatasetRepository.findById(datasetId)` and streams rows.
**Builder B (security):** Implements `SpaceSecurityFilter` covering `/api/workspaces/{id}/**` only.
**Clash:** If `ExportController` is not under `/api/workspaces/{id}/**`, `SpaceSecurityFilter` never runs before the export. Builder A can export any dataset by UUID without space membership check, enabling cross-space data exfiltration. D-06 names `@Auditable` on export methods but that is a logging annotation, not an authorization check.
**Severity:** Critical
**Fix:** Require export to be nested under the workspace path: `/api/workspaces/{wid}/datasets/{did}/export` — or explicitly state that `ExportService` must call `SpaceSecurityFilter`-equivalent membership check before streaming.

---

### S-05: API-Version header — absent header behavior undefined
**Builder A (API interceptor):** Implements `ApiVersionInterceptor` that rejects requests without `API-Version` header with `400 Bad Request`.
**Builder B (API client):** Implements `client.ts` Axios interceptor that adds `API-Version: 1` on all calls — but a third-party integration team calling the REST API directly omits the header and expects the API to default to version 1.
**Clash:** D-07 says the `HandlerMapping` "résout la version" from the header, but does not specify the fallback behavior when the header is absent. The two builders implement incompatible contracts: one rejects, one silently defaults.
**Severity:** High
**Fix:** Add one line to D-07: "Absent `API-Version` header → reject with `400 BAD_REQUEST`, error code `MISSING_API_VERSION`."

---

### S-06: Audit trail scope — export coverage gap
**Builder A (audit):** Implements `AuditableAspect` and annotates methods in `GenerationService` with `@Auditable`. Reads D-06: "toutes les méthodes de génération + export."
**Builder B (export):** Implements `ExportService` without `@Auditable` because P-09 rule 4 only says "Toute génération de données déclenche `@Auditable`" — export is not listed in P-09 rule 4, and Builder B reads P-09 as the authoritative rule list.
**Clash:** D-06 includes export in the audit scope; P-09 rule 4 omits it. Two compliant builders produce inconsistent audit coverage, violating NFR-S04 (audit trail 100%).
**Severity:** High
**Fix:** Align P-09 rule 4: change "Toute génération de données déclenche `@Auditable`" to "Toute génération ou export de données déclenche `@Auditable`."

---

### S-07: React Query invalidation — stale dataset state after generation completes
**Builder A (generation feature):** Implements `useGenerationStatus` hook that polls `GET /api/datasets/{id}/status`. On `status === 'COMPLETED'`, it calls `queryClient.invalidateQueries(['generation', datasetId])`.
**Builder B (dataset feature):** Implements `useDatasets` hook with query key `['datasets', workspaceId]`. After generation completes, the dataset list still shows the old row count because Builder A only invalidated the `generation` query key, not the `datasets` query key.
**Clash:** D-11 mandates React Query for all server state but defines no query key taxonomy or cross-feature invalidation contract. Two builders define disjoint query keys; generation completion never refreshes the dataset list.
**Severity:** Medium
**Fix:** Add a query key registry rule: "Canonical query key shapes: `['workspaces', wid]`, `['datasets', wid, pid, spid]`, `['dataset', did]`, `['generation', did]`. On generation `COMPLETED`, invalidate `['dataset', did]` and `['datasets', wid, pid, spid]`."

---

## Summary

| ID | Boundary | Severity |
|---|---|---|
| S-01 | PRNG algorithm + column seed derivation | Critical |
| S-02 | GenerationJob status — single writer | Critical |
| S-03 | SpaceSecurityFilter URL scope — DDL blind spot | Critical |
| S-04 | Export RBAC — cross-space leak | Critical |
| S-05 | API-Version absent header behavior | High |
| S-06 | Audit scope — export coverage gap D-06 vs P-09 | High |
| S-07 | React Query key taxonomy + cross-feature invalidation | Medium |

**File:** `reviews/review-adversarial.md`
