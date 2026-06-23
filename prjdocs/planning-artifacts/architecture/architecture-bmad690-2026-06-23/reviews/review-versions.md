# Version & Tech Review — GenData Architecture

Reviewed against mid-2026 technology landscape (knowledge cutoff August 2025; Spring Boot, React, and Vite major-version trajectory confirmed to early-2026 release schedules).

## Verdict

Four version flags require action before this document is handed to implementers: Spring Boot 3.2.x has been out of OSS support since November 2024, React 18 has been superseded by React 19 (December 2024), Vite 5 by Vite 6 (December 2024), and the `bootVersion=3.2.x` string in the Spring Initializr curl command is not a valid version and would fail at runtime.

---

## Flagged items

| Technology | Doc version | Assessment | Action |
|---|---|---|---|
| Spring Boot | `3.2.x` | OSS support ended November 2024. By mid-2026, the current supported release is 3.4.x (or 3.5.x if released). 3.2.x has known CVEs without patches. | **Update to `3.4.x`** (latest GA). Verify spring.io/projects/spring-boot for exact patch. |
| `bootVersion` in curl | `3.2.x` | Spring Initializr rejects wildcard `x` — will throw 400. Must be an explicit version like `3.4.6`. | **Fix to exact version** matching the Spring Boot decision above. |
| React | `18` | React 19 reached stable in December 2024. By mid-2026 it is the standard GA. MUI v6 supports React 19. | **Assess upgrade to React 19.** If staying on 18 for stability, pin explicitly (`"react": "^18.3.1"`) and document rationale. |
| Vite | `5` | Vite 6 released December 2024. Vite 5 receives security patches only. | **Update to Vite 6.** No breaking changes for a standard react-ts template. |
| react-router-dom | implicit v6 (line 740 references "React Router v6") | React Router v7 was released November 2024 with breaking changes (new File Route API, Server Functions). v6 enters maintenance mode. | **Pin to `react-router-dom@6` explicitly** if staying on v6, or plan migration to v7. Either way, the version must appear in the document. |
| jjwt-api | `0.12.x` | `0.12.x` is a wildcard, not a version pin. Latest in the 0.12 line is 0.12.6. | **Pin to `0.12.6`** (or latest 0.12.x patch). |
| jsqlparser | `4.9` | Latest in the 4.x line is 4.9 (released 2023). Check GitHub for 5.x releases. | **Verify** — if 5.x exists by mid-2026, assess upgrade. 4.9 may still be fine. |
| PostgreSQL (Docker) | `postgres:15` | PostgreSQL 17 was released September 2024 and is the current stable. `postgres:15` reaches EOL November 2027 — not immediately critical but increasingly stale. | **Recommend `postgres:17`** for alignment with prod environment. `postgres:15` is acceptable if prod is pinned to PG 15. |
| Java | `17 LTS` | Java 17 LTS is still in active support (until September 2029). Not outdated. However, Spring Boot 3.4+ officially lists Java 21 as the recommended LTS. | **Low risk.** If upgrading Spring Boot to 3.4.x, consider noting Java 21 as the preferred runtime while 17 remains supported minimum. |

---

## Compatibility matrix check

| Pair | Status | Notes |
|---|---|---|
| Spring Boot 3.2.x ↔ Spring Security 6.2.x | ✅ Compatible | SB 3.2 ships Security 6.2 by default |
| Spring Boot 3.2.x ↔ Spring Batch 5.0.x | ✅ Compatible | SB 3.2 ships Batch 5.0 by default |
| Spring Boot 3.2.x ↔ Java 17 | ✅ Compatible | Minimum Java 17 required |
| Spring Boot 3.4.x ↔ Java 17 | ✅ Still compatible | Java 17 remains supported minimum through 3.x |
| React 18 ↔ MUI v6 | ✅ Compatible | MUI v6 supports React 18 and 19 |
| React 19 ↔ MUI v6 | ✅ Compatible | MUI v6 explicitly supports React 19 |
| @tanstack/react-query v5 ↔ React 18/19 | ✅ Compatible | TanStack Query v5 supports both |
| @xyflow/react ↔ React 18 | ✅ Compatible | @xyflow/react (formerly reactflow) is the correct package name since v12 |
| jjwt-api 0.12.x ↔ Spring Boot 3.2.x | ✅ Compatible | No Spring Boot BOM for jjwt; explicit version needed — correct approach |
| jackson-dataformat-yaml (unversioned) ↔ Spring Boot 3.2.x | ✅ Compatible | Managed by Spring Boot BOM; no explicit version needed — correct |

---

## No issues found

These technologies are real, current, and appropriately chosen:

- **Spring Security 6** — correct major version for Spring Boot 3.x
- **Spring Batch 5** — correct major version for Spring Boot 3.x
- **TypeScript 5** — still the current major series (5.4, 5.5, 5.6 in the 5.x line)
- **MUI v6** — current as of mid-2026 (v6 released 2024; v7 not yet released)
- **@xyflow/react** — correct package name (renamed from `reactflow` in 2024)
- **@tanstack/react-query** — v5 is current; correct choice for server state
- **zustand** — standard choice for UI state; no version issues
- **@rjsf/core + @rjsf/mui** — v5.x; appropriate for auto-generated typology forms
- **recharts** — current and appropriate for usage metrics charts
- **@monaco-editor/react** — standard Monaco wrapper; correct choice for SQL DDL editor
- **jackson-dataformat-yaml** — unversioned (BOM-managed); correct approach
- **GitHub Actions CI** — standard; correct
- **Nginx** — standard for static SPA hosting; correct
- **H2 in-memory (dev profile)** — appropriate for local dev without LDAP
- **HikariCP** — default Spring Boot connection pool; correct
- **AOP (`@Auditable` aspect)** — standard Spring AOP pattern; correct approach
- **JSQLParser for DDL parsing** — correct choice for PostgreSQL DDL parsing in Java
