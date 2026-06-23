---
stepsCompleted: ['step-01-init', 'step-02-context', 'step-03-starter', 'step-04-decisions', 'step-05-patterns', 'step-06-structure', 'step-07-validation', 'step-08-complete']
inputDocuments:
  - 'prjdocs/planning-artifacts/prd.md'
  - 'prjdocs/ux-design-specification.md'
  - 'docs/besoin.txt'
workflowType: 'architecture'
lastStep: 8
status: 'complete'
completedAt: '2026-03-28'
project_name: 'GenData'
user_name: 'Nouredine'
date: '2026-03-28'
---

# Architecture Decision Document — GenData

_Ce document se construit collaborativement étape par étape. Les sections sont ajoutées au fur et à mesure des décisions architecturales._

---

## Analyse du Contexte Projet

### Vue d'ensemble des Exigences

**Exigences Fonctionnelles (53 FRs en 8 domaines) :**

| Domaine | FRs | Implication architecturale clé |
|---|---|---|
| Gestion des Espaces | FR01–FR08 | CRUD hiérarchie, RBAC par espace, label N1 configurable |
| Gestion des Datasets | FR09–FR20 | Upload CSV, auto-détect, config, restauration, export |
| Génération de Données | FR21–FR26 | Moteur batch async, seed déterministe, jobs limités |
| Catalogue Typologies & Test Lab | FR27–FR33 | Catalogue YAML déclaratif, versioning, assistant création |
| Mode DDL | FR34–FR40 | Parsing SQL, graphe DAG FK, tri topologique, génération ordonnée |
| APIs & Intégration | FR41–FR45 | REST 15+ endpoints, auth, extraction par critère, CRUD |
| Sécurité & Administration | FR46–FR50 | SSO LDAP/AD, JWT, RBAC, audit trail, marquage TEST DATA |
| Suivi d’Activité | FR51–FR53 | Événements dataset, historique, statistiques espace |

**Exigences Non-Fonctionnelles critiques pour l’architecture :**

| NFR | Contrainte architecturale |
|---|---|
| NFR-P02 : 1M lignes/min | Spring Batch 5, chunks 10k, pool jobs |
| NFR-SC01 : 1 000 users concurrents | Stateless API, connection pool PostgreSQL |
| NFR-SC02 : 200 jobs simultanés | Queue Spring Batch, thread pool configurable |
| NFR-SC03 : datasets 10M lignes | Streaming HTTP, stockage partitionné |
| NFR-S01 : auth SSO LDAP | Spring Security + LDAP, JWT stateless |
| NFR-S04 : audit trail 100% | AOP/intercepteur transversal |
| NFR-R03 : reproductibilité | PRNG déterministe par colonne, version YAML typ. |
| NFR-I01 : versioning API | Header `API-Version`, N disponible ≥6 mois après N+1 |

### Évaluation de la Complexité

| Dimension | Niveau | Justification |
|---|---|---|
| Complexité globale | **Haute** | 53 FRs, 3 couches découplées, conformité banking |
| Frontend | **Haute** | Monaco Editor, React Flow, DataGrid, états async, polling |
| Backend / API | **Haute** | REST 15+ endpoints versionés, RBAC, LDAP, audit |
| Moteur | **Haute** | Spring Batch async, 200 jobs simultanés, streaming |
| Données | **Haute** | Partitionnement, purge planifiée, immutabilité config |

**Domaine principal :** Full-stack Web App Interne (SPA + API REST + Engine batch)

### Contraintes Techniques & Dépendances

- **Stack imposée :** React 18 + TypeScript 5, Spring Boot 3.x / Java 17+, PostgreSQL 15+
- **Auth :** LDAP/Active Directory bancaire existant — SSO obligatoire, pas de compte local
- **Brownfield :** 112 typologies YAML existantes avec format déjà défini — contrainte sur le moteur
- **Isolation :** Aucune visibilité cross-espace — RBAC vérifié à chaque requête
- **Exports volumineux :** Streaming HTTP obligatoire — jamais de bufferisation complète en mémoire
- **DDL :** Dialecte PostgreSQL en priorité MVP — JSQLParser

### Préoccupations Transversales Identifiées

1. **Audit trail** — 100% des générations loguées → AOP ou intercepteur HTTP
2. **Isolation espaces** — RBAC vérifié à chaque requête → filtre Spring Security
3. **Marquage TEST DATA** — en sortie de chaque export → enrichissement couche export
4. **Seed déterministe** — lié à la version YAML de la typo → versioning typologies obligatoire
5. **Jobs orphelins** — limite 5 jobs/user + scheduler nettoyage
6. **Streaming** — exports >10M lignes sans OOM → jamais de collect complète en mémoire
7. **Brownfield** — 112 typologies YAML existantes → format YAML = contrainte moteur

### Trois Domaines Techniques Distincts

L’analyse des 53 FRs fait émerger 3 boundary contexts métier + 2 services techniques orthogonaux :

| Contexte | Responsabilité | FRs concernés |
|---|---|---|
| `workspace-api` | CRUD hiérarchie, RBAC, membres, label N1 | FR01–FR08, FR46–FR50, FR51–FR53 |
| `dataset-api` | CSV upload, config colonnes, génération, export, restauration | FR09–FR26, FR34–FR45 |
| `typology-engine` | Catalogue YAML, Test Lab, assistant création, versioning | FR27–FR33 |
| `generation-engine` | Spring Batch, jobs async, chunking, streaming | NFR-P02, NFR-SC02 |
| `ddl-service` | Parsing SQL, DAG FK, tri topologique | FR34–FR38 |

---

## Évaluation du Starter & Stack

### Domaine Technologique Principal

Full-stack Web App Interne : **SPA React** + **API REST Spring Boot** + **Moteur batch découplé**. Stack entièrement définie dans le PRD — évaluation confirme la cohérence des choix.

### Frontend — Vite + React 18 + TypeScript 5

**Commande d’initialisation :**

```bash
npm create vite@latest gendata-frontend -- --template react-ts
cd gendata-frontend
npm install
```

**Décisions architecturales établies par le starter :**

| Décision | Valeur |
|---|---|
| Framework | React 18 |
| Langage | TypeScript 5 strict |
| Build | Vite 5 (ESBuild, HMR < 50ms) |
| Config TS | `tsconfig.app.json` strict + paths |

**Dépendances post-init :**

```bash
# MUI v6 + DataGrid + TreeView
npm install @mui/material @mui/x-data-grid @mui/x-tree-view @emotion/react @emotion/styled

# Éditeur DDL
npm install @monaco-editor/react

# Graphe FK
npm install @xyflow/react

# State + Routing
npm install @tanstack/react-query zustand react-router-dom

# Formulaires typologies YAML
npm install @rjsf/core @rjsf/mui

# Charts métriques
npm install recharts
```

### Backend — Spring Boot 3.x + Java 17+

**Commande d’initialisation (Spring Initializr) :**

```bash
curl https://start.spring.io/starter.zip \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.2.x \
  -d groupId=com.gendata \
  -d artifactId=gendata-backend \
  -d javaVersion=17 \
  -d dependencies=web,data-jpa,postgresql,security,batch,validation,actuator,ldap \
  -o gendata-backend.zip
```

**Décisions architecturales établies par le starter :**

| Décision | Valeur |
|---|---|
| Framework | Spring Boot 3.2.x |
| Java | 17 LTS |
| Build | Maven (wrapper inclus) |
| Data | Spring Data JPA + Hibernate |
| Security | Spring Security 6 (lambda DSL) |
| Batch | Spring Batch 5 |
| DB dev | H2 in-memory (profil `dev`) |
| DB prod | PostgreSQL 15+ |
| Monitoring | Spring Actuator (`/health`, `/metrics`) |

**Dépendances additionnelles :**

```xml
<!-- Parsing SQL DDL -->
<dependency>
  <groupId>com.github.jsqlparser</groupId>
  <artifactId>jsqlparser</artifactId>
  <version>4.9</version>
</dependency>

<!-- YAML typologies -->
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>

<!-- JWT -->
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt-api</artifactId>
  <version>0.12.x</version>
</dependency>
```

### Structure du Projet Multi-Module

```
gendata/
├── gendata-frontend/          # Vite React TS
│   └── src/
│       ├── components/        # Composants MUI + custom C-01–C-07
│       ├── pages/             # Route-level screens
│       ├── features/          # workspace / dataset / typology / ddl
│       ├── hooks/             # React Query hooks
│       └── store/             # Zustand slices
├── gendata-backend/           # Spring Boot 3
│   └── src/main/java/com/gendata/
│       ├── workspace/         # Bounded context workspace
│       ├── dataset/           # Bounded context dataset
│       ├── typology/          # Bounded context typologies
│       ├── generation/        # Spring Batch engine
│       ├── ddl/               # DDL parsing + graphe FK
│       └── shared/            # Auth, audit, exceptions
├── prjdocs/                   # Artefacts planning
└── docker-compose.yml         # PostgreSQL + adminer dev local
```

**Note :** L’initialisation des deux projets via ces commandes constitue la **première story de Sprint 1**.

---

## Décisions Architecturales Fondamentales

### Analyse de Priorité

**Décisions Critiques (bloquent l’implémentation) :**
- D-04 Auth LDAP/JWT, D-05 RBAC espace, D-01 stockage lignes générées, D-02 versioning typologies

**Décisions Importantes (shapes architecture) :**
- D-08 async polling, D-09 streaming, D-11 state management, D-12 routing, D-13 feature folders

**Décisions Déférées Post-MVP :**
- WebSocket (D-08 Phase 2), VIEWER role granulaire (D-05 Phase 2), Kubernetes (D-16 Phase 3)

### Architecture des Données

**D-01 : Stockage des lignes générées**

Table `generated_rows` **partitionnée par `dataset_id`** (PostgreSQL native partitioning).
- Purge par partition = `DROP PARTITION` au lieu de `DELETE` massif
- Supporte 10M+ lignes/dataset sans dégradation des requêtes
- Index composé `(dataset_id, row_index)` pour l’extraction par critère (FR42, FR43)

**D-02 : Versioning des Typologies**

Chaque typo publiée est **immuable**. Version sémantique `MAJOR.MINOR` + SHA-256 du contenu YAML stocké en base. Un dataset référence `typology_id + version + sha` pour garantir la reproductibilité (NFR-R03) : même seed + même YAML = données identiques inter-versions.

**PRNG canonique (S-01) :** Column seed = `Long.hashCode(datasetSeed) ^ columnIndex` ; générateur = `new java.util.Random(columnSeed)` — aucun autre générateur autorisé (ThreadLocalRandom, SecureRandom, etc. sont interdits).

**D-03 : Config Dataset Immuable**

Colonne `original_config JSONB` (immuable, écrite à la création) + `current_config JSONB` (modifiable). Restauration = `current_config = original_config` (FR18, FR19).

### Authentification & Sécurité

**D-04 : Intégration LDAP/AD + JWT**

```
Client → POST /auth/login { username, password }
Spring Security → LdapAuthenticationProvider → Active Directory
AD → OK → JWT généré (expiry 8h, secret configurable en env var)
Client → Authorization: Bearer <token> sur toutes les requêtes suivantes
BearerTokenAuthenticationFilter → valide JWT à chaque requête
```

URL LDAP + Base DN configurés dans `application.yml` — modifiables sans recompilation (NFR-I04).

**D-05 : RBAC par Espace**

- **Rôles MVP :** `SPACE_ADMIN`, `SPACE_CONTRIBUTOR` (VIEWER en Phase 2)
- **Storage :** Table `space_memberships(user_id, space_id, role)`
- **Filtre :** `SpaceSecurityFilter` Spring Security — vérifie membership à chaque requête sur `/api/workspaces/{id}/**`
- **JWT claims :** Liste des `space_id` accessibles encodée dans le token (revalidation DB sur les opérations sensibles)
- **DDL URL (S-03) :** Endpoints DDL sous `/api/workspaces/{wid}/projects/{pid}/subprojects/{spid}/ddl/{did}/**` — chemin imbriqué garantissant la couverture par `SpaceSecurityFilter`. Le préfixe plat `/api/ddl/**` est interdit.
- **Export URL (S-04) :** Endpoints d'export sous `/api/workspaces/{wid}/datasets/{did}/export` — membership vérifié avant tout streaming. L'export n'est jamais exposé hors du chemin workspace.

**D-06 : Audit Trail — AOP**

Annotation `@Auditable` sur toutes les méthodes de génération **et d'export** (S-06). `AuditTrailAspect` (`@AfterReturning`) logue en table `audit_logs` : `user_id`, `timestamp_utc`, `action`, `dataset_id`, `volume`. Transparent pour le code métier.

### Patterns API & Communication

**D-07 : Versioning API**

Header `API-Version: 1` — pas de préfixe `/v1/` dans l’URL. `HandlerMapping` Spring résout la version. Version N disponible ≥6 mois après sortie N+1 (NFR-I01).

**Comportement header absent (S-05) :** Header `API-Version` manquant → `400 BAD_REQUEST`, code `MISSING_API_VERSION` — jamais de fallback silencieux sur une version par défaut.

**D-08 : Async Génération — Polling MVP**

Polling client React Query `refetchInterval: 2000ms` sur `GET /api/datasets/{id}/status`. Format retour :
```json
{ "status": "RUNNING", "progress": 73, "processedColumns": ["transaction_id", "iban"] }
```
WebSocket sera ajouté en Phase 2 sans casser l’API polling existante.

**Single writer status (S-02) :** Seul `GenerationJobListener` (Spring Batch `@AfterJob`) écrit les transitions `PENDING → RUNNING → COMPLETED/FAILED` après soumission. `dataset-api` appelle `generationEngine.requestCancel(jobId)` et ne modifie **jamais** `generation_jobs.status` directement — écriture concurrente interdite.

**React Query invalidation (S-07) :** Clés canoniques — `[‘workspaces’, wid]`, `[‘datasets’, wid, pid, spid]`, `[‘dataset’, did]`, `[‘generation’, did]`. Sur `COMPLETED` : invalider `[‘dataset’, did]` **et** `[‘datasets’, wid, pid, spid]` pour rafraîchir la liste et la fiche.

**D-09 : Streaming Export**

Spring `StreamingResponseBody` + `Transfer-Encoding: chunked`. Premièr byte ≤3s quel que soit le volume (NFR-P04). Jamais de bufferisation complète.

**D-10 : Format d’Erreur Standard**

```json
{
  "error": "TYPOLOGY_NOT_FOUND",
  "message": "La typologie 'iban-fr' version 2 est introuvable.",
  "field": "typology_id",
  "timestamp": "2026-03-28T09:00:00Z"
}
```
`@ControllerAdvice` global — aucune stack trace exposée (sécurité + UX pattern 7.2).
**P-03 : Format Standard des Réponses API REST**

Toutes les réponses API utilisent deux enveloppes JSON selon le type :

*Ressource unique :*
```json
{
  "data": {
    "id": "uuid",
    "name": "Mon Dataset",
    "status": "completed"
  }
}
```

*Collection paginée :*
```json
{
  "data": [
    { "id": "uuid-1", "name": "Dataset A" },
    { "id": "uuid-2", "name": "Dataset B" }
  ],
  "pagination": {
    "page": 1,
    "size": 20,
    "total": 127,
    "totalPages": 7
  }
}
```

*Opération asynchrone :*
```json
{
  "data": {
    "jobId": "uuid",
    "status": "PENDING",
    "datasetId": "uuid",
    "submittedAt": "2026-03-28T10:00:00Z"
  }
}
```

*Erreur :*
```json
{
  "error": "VALIDATION_ERROR",
  "message": "Le champ 'nbRows' doit être un entier positif.",
  "field": "nbRows",
  "timestamp": "2026-03-28T10:00:00Z"
}
```

*Header obligatoire :* `API-Version: 1` sur tous les appels.

*Sémantique HTTP :* 200/201/202/204 succès, 400 erreur client, 401/403 auth, 404 absent, 500 erreur serveur.

*Implémentation :* Classes enveloppe génériques `ApiResponse<T>`, `PagedApiResponse<T>`, `ApiErrorResponse`. Intercepteur `ApiVersionInterceptor` pour valider le header.
### Architecture Frontend

**D-11 : Séparation Server State / UI State**

- **React Query** = server state (datasets, typologies, jobs, workspaces) — cache automatique, invalidation, polling
- **Zustand** = UI state (drawer ouvert, onglet actif, label N1, écran sélectionné)
- **Pas de Redux** — complexité non justifiée pour ce périmètre

**D-12 : Routing Deep-Link**

```
/workspaces/:wid/projects/:pid/subprojects/:spid                   → Sous-projet
/workspaces/:wid/projects/:pid/subprojects/:spid/datasets/:did     → Config Dataset
/workspaces/:wid/projects/:pid/subprojects/:spid/ddl/:did          → Vue DDL
/typologies                                                        → Typology Browser
```
URL = état de navigation — partage et deep-link natif (FR17).

**D-13 : Organisation par Feature**

```
src/features/
  workspace/    # TreeView sidebar, CRUD espace/projet/sous-projet
  dataset/      # Config colonnes, preview, génération
  typology/     # Browser, Test Lab, assistant création
  ddl/          # Éditeur Monaco, FK Graph React Flow
```
Chaque feature = composants + hooks React Query + client API + types TypeScript.

### Infrastructure & Déploiement

**D-14 : Dev Local**

```yaml
# docker-compose.yml
services:
  postgres:
    image: postgres:15
    environment: { POSTGRES_DB: gendata_dev, POSTGRES_USER: gendata, POSTGRES_PASSWORD: devpassword }
    ports: ["5432:5432"]
  adminer:
    image: adminer
    ports: ["8081:8080"]
```

**D-15 : CI/CD Pipeline MVP**

1. Push → GitHub Actions
2. `mvn verify` (tests Spring Boot)
3. `npm run build` (Vite)
4. `npx cypress-axe` (accessibilité automatique)
5. Docker build + push registry interne

**D-16 : Déploiement**

Deux containers : `gendata-frontend` (Nginx statique) + `gendata-backend` (JAR Spring Boot). PostgreSQL external (infra bancaire existante). VM interne MVP — pas de Kubernetes avant Phase 3.

### Analyse d’Impact — Séquence d’Implémentation

| Priorité | Décision | Bloque |
|---|---|---|
| 🔴 Sprint 1 | D-04 Auth LDAP + JWT | Toute l’API |
| 🔴 Sprint 1 | D-05 RBAC + Space filter | Isolation espaces |
| 🔴 Sprint 1 | D-01 Table `generated_rows` partitionnée | Moteur batch |
| 🔴 Sprint 1 | D-13 Feature folders frontend | Structure code |
| 🟠 Sprint 2 | D-08 Polling async | UX génération |
| 🟠 Sprint 2 | D-02 Versioning typologies | Reproductibilité |
| 🟡 Sprint 3 | D-09 Streaming export | Exports volumineux |
| 🟡 Sprint 3 | D-06 Audit AOP | Conformité banking |

---

## Patterns d'Implémentation & Règles de Cohérence

_Ces patterns définissent les conventions obligatoires pour tous les agents implémentant GenData. Toute déviation doit être justifiée et approuvée._

### P-01 : Nommage Base de Données

| Élément | Convention | Exemple |
|---|---|---|
| Tables | `snake_case` pluriel | `generated_rows`, `space_memberships` |
| Clé primaire | `id UUID DEFAULT gen_random_uuid()` | `id UUID PRIMARY KEY DEFAULT gen_random_uuid()` |
| Clé étrangère | `{table_singulier}_id` | `dataset_id`, `workspace_id` |
| Index | `idx_{table}_{colonnes}` | `idx_generated_rows_dataset_id` |
| Timestamps | `created_at`, `updated_at` TIMESTAMPTZ UTC | `created_at TIMESTAMPTZ DEFAULT now()` |
| Enum PostgreSQL | `snake_case` | `generation_status`, `export_format` |

### P-02 : Nommage API REST

| Élément | Convention | Exemple |
|---|---|---|
| Segments de chemin | `snake_case` pluriel | `/api/workspaces`, `/api/datasets` |
| Path parameters | `camelCase` | `/{datasetId}`, `/{workspaceId}` |
| Query parameters | `camelCase` | `?pageSize=50&sortBy=createdAt` |
| Headers personnalisés | `X-GenData-*` | `X-GenData-Job-Id`, `X-GenData-Version` |
| Verbes HTTP | Sémantique stricte | GET=lecture, POST=création, PUT=remplacement, PATCH=partiel, DELETE |

### P-03 : Format Réponses API

```json
// Liste paginée
{
  "data": [...],
  "pagination": { "page": 0, "size": 20, "total": 142 }
}

// Ressource unique
{ "data": { ... } }

// Opération asynchrone (202 Accepted)
{
  "jobId": "550e8400-e29b-41d4-a716-446655440000",
  "statusUrl": "/api/datasets/{id}/status"
}

// Erreur (D-10)
{
  "error": "SNAKE_CASE_CODE",
  "message": "Message lisible en français",
  "field": "nomDuChamp",
  "timestamp": "2026-03-28T09:00:00Z"
}
```

**Codes HTTP utilisés :** 200 OK · 201 Created · 202 Accepted · 204 No Content · 400 Bad Request · 401 Unauthorized · 403 Forbidden · 404 Not Found · 409 Conflict · 500 Internal Server Error

### P-04 : Nommage Code

**Java (Backend) :**

| Élément | Convention | Exemple |
|---|---|---|
| Classes | `PascalCase` | `DatasetGenerationService`, `WorkspaceController` |
| Méthodes / Variables | `camelCase` | `generateRows()`, `datasetId` |
| Packages | `com.gendata.{feature}` | `com.gendata.dataset`, `com.gendata.typology` |
| Tests unitaires | `{Classe}Test` | `DatasetServiceTest` |
| Tests intégration | `{Classe}IntegrationTest` | `WorkspaceControllerIntegrationTest` |
| Constantes | `UPPER_SNAKE_CASE` | `MAX_ROWS_PER_CHUNK`, `DEFAULT_PAGE_SIZE` |

**TypeScript (Frontend) :**

| Élément | Convention | Exemple |
|---|---|---|
| Composants | `PascalCase.tsx` | `DatasetCard.tsx`, `WorkspaceHeader.tsx` |
| Hooks React Query | `use{Nom}.ts` | `useDatasets.ts`, `useGeneration.ts` |
| Stores Zustand | `use{Nom}Store.ts` | `useWorkspaceStore.ts`, `useUIStore.ts` |
| Services API | `{feature}Api.ts` | `datasetApi.ts`, `workspaceApi.ts` |
| Types / Interfaces | `PascalCase.ts` | `Dataset.ts`, `GenerationJob.ts` |
| Constantes | `UPPER_SNAKE_CASE` | `MAX_POLLING_INTERVAL`, `API_BASE_URL` |

### P-05 : Organisation des Tests

**Backend :**
```
src/test/java/com/gendata/
  {feature}/
    {Classe}Test.java              # Tests unitaires (Mockito)
    {Classe}IntegrationTest.java   # Tests intégration (@SpringBootTest)
  fixtures/
    {Feature}TestData.java         # Données de test partagées
```

**Frontend :**
```
src/features/{feature}/__tests__/
  {Composant}.test.tsx             # Tests Jest + Testing Library
src/components/C-0x/
  {Composant}.stories.tsx          # Storybook — une story par état
```

**Règle de couverture minimale :** Services métier ≥80% · Controllers ≥70% · Composants UI ≥60%

### P-06 : Formats de Données

| Type | Format | Exemple |
|---|---|---|
| Dates | ISO 8601 UTC | `"2026-03-28T09:00:00Z"` |
| UUID | Lowercase avec tirets | `"550e8400-e29b-41d4-a716-446655440000"` |
| Montants / Décimaux | String 2 décimales | `"1234.56"` (jamais `float`) |
| Booléens | `true` / `false` | Jamais `0`/`1` ni `"yes"`/`"no"` |
| Valeurs nulles | `null` explicite | Jamais champ absent |
| Pagination | `page` 0-indexed, `size`, `total` | `{ "page": 0, "size": 20, "total": 142 }` |
| Encodage | UTF-8 partout | Headers, CSV, JSON, DB |

### P-07 : Gestion Erreurs Frontend

- **React Query** pour tous les appels API — gestion automatique retry/error state
- **React Error Boundary** sur chaque page feature-level (pas global)
- **Jamais** `error.stack` ni `error.message` technique dans l'UI — messages en français
- Pattern standard :
```typescript
const { data, isLoading, error } = useQuery(...);
if (error) return <ErrorMessage message={getDisplayMessage(error)} />;
```

### P-08 : États de Chargement

| Situation | Composant | Règle |
|---|---|---|
| Chargement initial page | `<DatasetListSkeleton />` | Skeleton mimant la vraie structure |
| Action inline | `<CircularProgress size={16} />` | Inline à côté du bouton |
| Soumission formulaire | Bouton disabled + spinner | Jamais spinner global |
| Pagination / tri | Overlay léger sur la table | Pas de remount complet |

**Interdit :** Spinner plein écran bloquant toute l'UI

### P-09 : Règles OBLIGATOIRES pour Agents

> Ces 10 règles sont NON-NÉGOCIABLES. Tout agent implémentant du code GenData DOIT les respecter.

| # | Règle | Référence |
|---|---|---|
| 1 | Tables PostgreSQL en `snake_case` pluriel | P-01 |
| 2 | Format erreur `{error, message, field, timestamp}` strict | D-10 |
| 3 | RBAC exclusivement via `SpaceSecurityFilter` | D-05 |
| 4 | Toute génération **ou export** de données déclenche `@Auditable` (S-06) | D-06 |
| 5 | Exports via `StreamingResponseBody` — jamais `byte[]` complet | D-09 |
| 6 | Toutes les dates en `TIMESTAMPTZ UTC` dans la DB | P-06 |
| 7 | Organisation par feature folders `src/features/{feature}/` | D-13 |
| 8 | Messages d'erreur UI en français | P-07 |
| 9 | Typologies référencées par `typology_id + version + sha256` | D-02 |
| 10 | Server state via React Query — jamais `useState` pour data API | D-11 |

---

## Structure du Projet & Frontières Architecturales

_Structure physique complète basée sur toutes les décisions architecturales D-01→D-16 et patterns P-01→P-09._

### Mapping FRs → Modules

| Domaine FR | Module Backend | Module Frontend |
|---|---|---|
| FR01–08 Espaces | `com.gendata.workspace` | `src/features/workspace/` |
| FR09–20 Datasets | `com.gendata.dataset` | `src/features/dataset/` |
| FR21–26 Génération | `com.gendata.generation` | `src/features/generation/` |
| FR27–33 Typologies | `com.gendata.typology` | `src/features/typology/` |
| FR34–40 Mode DDL | `com.gendata.ddl` | `src/features/ddl/` |
| FR41–45 APIs & Export | `com.gendata.export` | `src/features/api-integration/` |
| FR46–50 Sécurité | `com.gendata.security` | `src/features/auth/` |
| FR51–53 Activité | `com.gendata.activity` | `src/features/activity/` |

### Arborescence Complète du Projet

```
gendata/
├── README.md
├── .gitignore
├── .env.example
├── docker-compose.yml                    # D-14 : postgres:15 + adminer
├── .github/
│   └── workflows/
│       └── ci.yml                        # D-15 : mvn verify + npm build + cypress-axe
├── gendata-backend/                      # Maven — Spring Boot 3.2.x / Java 17
│   ├── pom.xml
│   ├── Dockerfile
│   └── src/
│       ├── main/
│       │   ├── java/com/gendata/
│       │   │   ├── GendataApplication.java
│       │   │   ├── config/
│       │   │   │   ├── SecurityConfig.java           # D-04 LDAP + JWT
│       │   │   │   ├── BatchConfig.java              # Spring Batch 5
│       │   │   │   ├── JpaConfig.java
│       │   │   │   └── WebConfig.java                # CORS, streaming
│       │   │   ├── security/
│       │   │   │   ├── LdapAuthProvider.java         # D-04
│       │   │   │   ├── JwtTokenService.java          # JWT 8h stateless
│       │   │   │   ├── JwtAuthFilter.java
│       │   │   │   ├── SpaceSecurityFilter.java      # D-05 RBAC
│       │   │   │   └── AuditableAspect.java          # D-06 @Auditable AOP
│       │   │   ├── common/
│       │   │   │   ├── ApiResponse.java              # P-03 wrapper standard
│       │   │   │   ├── PagedResponse.java
│       │   │   │   ├── ApiException.java             # D-10 format erreur
│       │   │   │   ├── GlobalExceptionHandler.java
│       │   │   │   └── Auditable.java                # annotation AOP
│       │   │   ├── workspace/                        # FR01–08
│       │   │   │   ├── Workspace.java
│       │   │   │   ├── SpaceMembership.java
│       │   │   │   ├── WorkspaceRepository.java
│       │   │   │   ├── WorkspaceService.java
│       │   │   │   ├── WorkspaceController.java
│       │   │   │   └── dto/
│       │   │   ├── dataset/                          # FR09–20
│       │   │   │   ├── Dataset.java
│       │   │   │   ├── DatasetColumn.java
│       │   │   │   ├── DatasetRepository.java
│       │   │   │   ├── DatasetService.java
│       │   │   │   ├── DatasetController.java
│       │   │   │   ├── CsvImportService.java
│       │   │   │   └── dto/
│       │   │   ├── generation/                       # FR21–26
│       │   │   │   ├── GenerationJob.java
│       │   │   │   ├── GeneratedRow.java             # D-01 table partitionnée
│       │   │   │   ├── GenerationJobRepository.java
│       │   │   │   ├── GenerationService.java
│       │   │   │   ├── GenerationController.java
│       │   │   │   ├── batch/
│       │   │   │   │   ├── GenerationJobConfig.java  # Spring Batch 5
│       │   │   │   │   ├── RowGeneratorItemWriter.java
│       │   │   │   │   └── GenerationJobListener.java
│       │   │   │   └── dto/
│       │   │   ├── typology/                         # FR27–33
│       │   │   │   ├── Typology.java
│       │   │   │   ├── TypologyVersion.java          # D-02 versioning sha256
│       │   │   │   ├── TypologyRepository.java
│       │   │   │   ├── TypologyService.java
│       │   │   │   ├── TypologyController.java
│       │   │   │   ├── YamlTypologyParser.java
│       │   │   │   └── dto/
│       │   │   ├── ddl/                              # FR34–40
│       │   │   │   ├── DdlParserService.java
│       │   │   │   ├── SchemaGraph.java              # DAG FK
│       │   │   │   ├── TopologicalSorter.java        # tri topologique
│       │   │   │   ├── DdlGenerationService.java
│       │   │   │   ├── DdlController.java
│       │   │   │   └── dto/
│       │   │   ├── export/                           # FR15, FR41–45
│       │   │   │   ├── ExportService.java
│       │   │   │   ├── ExportController.java         # D-09 StreamingResponseBody
│       │   │   │   ├── CsvExporter.java
│       │   │   │   ├── JsonExporter.java
│       │   │   │   └── SqlExporter.java
│       │   │   └── activity/                         # FR51–53
│       │   │       ├── ActivityEvent.java
│       │   │       ├── AuditLog.java
│       │   │       ├── ActivityRepository.java
│       │   │       ├── ActivityService.java
│       │   │       └── ActivityController.java
│       │   └── resources/
│       │       ├── application.yml
│       │       ├── application-dev.yml
│       │       ├── application-prod.yml
│       │       └── db/migration/
│       │           ├── V1__init_schema.sql
│       │           ├── V2__workspaces.sql
│       │           ├── V3__datasets.sql
│       │           ├── V4__generation_rows_partitioned.sql  # D-01
│       │           ├── V5__typologies.sql
│       │           ├── V6__audit_logs.sql
│       │           └── V7__activity_events.sql
│       └── test/
│           └── java/com/gendata/
│               ├── workspace/
│               │   ├── WorkspaceServiceTest.java
│               │   └── WorkspaceControllerIntegrationTest.java
│               ├── dataset/
│               │   ├── DatasetServiceTest.java
│               │   └── DatasetControllerIntegrationTest.java
│               ├── generation/
│               │   ├── GenerationServiceTest.java
│               │   └── GenerationJobIntegrationTest.java
│               ├── security/
│               │   ├── JwtTokenServiceTest.java
│               │   └── SpaceSecurityFilterTest.java
│               └── fixtures/
│                   ├── WorkspaceTestData.java
│                   ├── DatasetTestData.java
│                   └── TypologyTestData.java
└── gendata-frontend/                     # Vite + React 18 + TypeScript 5
    ├── package.json
    ├── vite.config.ts
    ├── tsconfig.json
    ├── .eslintrc.json
    ├── .prettierrc
    ├── Dockerfile
    ├── nginx.conf                        # D-16 Nginx statique
    ├── index.html
    ├── public/assets/
    └── src/
        ├── main.tsx
        ├── App.tsx
        ├── router.tsx                    # D-12 React Router v6 deep-links
        ├── api/
        │   ├── client.ts                 # Axios + interceptors (D-07 API-Version)
        │   ├── workspaceApi.ts
        │   ├── datasetApi.ts
        │   ├── generationApi.ts
        │   ├── typologyApi.ts
        │   ├── ddlApi.ts
        │   └── activityApi.ts
        ├── stores/                       # D-11 Zustand
        │   ├── useWorkspaceStore.ts
        │   ├── useUIStore.ts
        │   └── useAuthStore.ts
        ├── hooks/
        │   ├── useWorkspaces.ts
        │   ├── useDatasets.ts
        │   └── useGenerationStatus.ts    # D-08 polling 2000ms
        ├── components/
        │   ├── C-01-WorkspaceNav/
        │   ├── C-02-DatasetExplorer/
        │   ├── C-03-TypologyCard/
        │   ├── C-04-GenerationPanel/
        │   ├── C-05-DDLEditor/           # Monaco Editor
        │   ├── C-06-SchemaGraph/         # React Flow
        │   ├── C-07-ActivityFeed/
        │   └── shared/
        │       ├── ErrorBoundary.tsx
        │       ├── SkeletonLoader.tsx    # P-08
        │       ├── ErrorMessage.tsx
        │       └── PageLayout.tsx
        ├── features/                     # D-13 feature folders
        │   ├── auth/
        │   ├── workspace/               # FR01–08
        │   ├── dataset/                 # FR09–20
        │   ├── generation/              # FR21–26
        │   ├── typology/                # FR27–33
        │   ├── ddl/                     # FR34–40
        │   └── activity/               # FR51–53
        └── types/
            ├── Workspace.ts
            ├── Dataset.ts
            ├── GenerationJob.ts
            ├── Typology.ts
            ├── DdlSchema.ts
            └── api.ts
```

### Frontières Architecturales

**Frontières API :**

| Frontière | Mécanisme | Détail |
|---|---|---|
| Frontend ↔ Backend | REST JSON over HTTP | Header `API-Version: 1` (D-07), `Authorization: Bearer {jwt}` |
| Authentification | `JwtAuthFilter` Spring Security 6 | Valide JWT sur toutes routes sauf `/api/auth/**` |
| Autorisation Espace | `SpaceSecurityFilter` | Injecte `spaceRole` dans `SecurityContext` avant chaque handler |
| Export streaming | `StreamingResponseBody` | Chunked transfer, jamais de bufferisation complète |

**Frontières Données :**

| Frontière | Mécanisme | Détail |
|---|---|---|
| `generated_rows` | Partitionnement PostgreSQL par `dataset_id` | Isolation queries, TRUNCATE/DELETE par dataset (D-01) |
| Typologies publiées | Immuabilité `version + sha256` | Modification = nouvelle version (D-02) |
| Configs dataset | `original_config JSONB` + `current_config JSONB` | Restauration FR19 (D-03) |
| Audit | AOP `@Auditable` → table `audit_logs` | Transparent pour couche service (D-06) |

**Frontières State Frontend :**

| Store | Responsabilité |
|---|---|
| React Query | Tout l'état serveur : workspaces, datasets, jobs, typologies |
| `useWorkspaceStore` (Zustand) | Workspace actif, projet actif, navigation |
| `useUIStore` (Zustand) | Modals ouverts, notifications, thème |
| `useAuthStore` (Zustand) | Utilisateur courant, JWT, rôles |

### Intégrations Externes

| Système | Sens | Implémentation |
|---|---|---|
| LDAP / Active Directory | Entrant | `LdapAuthProvider.java` → `spring-security-ldap` |
| PostgreSQL 15+ | Bidirectionnel | Spring Data JPA + HikariCP pool configurable |
| Registry Docker interne | Sortant | GitHub Actions → push image (D-15/D-16) |
| SMTP (Phase 2) | Sortant | Spring Mail — non implémenté MVP |

---

## Résultats de la Validation Architecturale

### Validation de la Cohérence ✅

**Compatibilité des Décisions :**

| Vérification | Résultat |
|---|---|
| Spring Boot 3.2.x + Java 17 | ✅ LTS GA, compatible Spring Security 6 + Batch 5 |
| Spring Security 6 + LDAP + JWT stateless | ✅ `JwtAuthFilter` + `SpaceSecurityFilter` bien séquencés |
| Spring Batch 5 + PostgreSQL 15 partitionné | ✅ Chunks 10k sur table partitionnée, HikariCP |
| React 18 + React Query + Zustand | ✅ Séparation server/client state claire (D-11) |
| D-08 Polling 2000ms + D-09 StreamingResponseBody | ✅ Non-conflictuels — polling status, streaming download |
| D-04 JWT 8h + D-05 RBAC SpaceSecurityFilter | ✅ JWT validé avant SpaceSecurityFilter dans la chaîne |

**Cohérence des Patterns :**

| Pattern | Alignement |
|---|---|
| Feature folders (D-13) ↔ Packages backend | ✅ Symétrie frontend/backend par domaine |
| P-01 snake_case DB ↔ P-02 camelCase API ↔ P-04 PascalCase Java | ✅ Convention distincte par couche |
| P-03 format réponse ↔ D-10 format erreur | ✅ Complémentaires, non-conflictuels |
| D-06 @Auditable AOP ↔ P-09 règle 4 | ✅ Décision + règle obligatoire alignées |

### Couverture des Exigences ✅

**53 FRs — Couverture complète :**

| Domaine | FRs | Support Architectural |
|---|---|---|
| Espaces (FR01–08) | 8 | ✅ `workspace/` + RBAC `SpaceSecurityFilter` |
| Datasets (FR09–20) | 12 | ✅ `dataset/` + `CsvImportService` + `original/current_config JSONB` |
| Génération (FR21–26) | 6 | ✅ Spring Batch 5 + `GeneratedRow` partitionné (D-01) |
| Typologies (FR27–33) | 7 | ✅ `YamlTypologyParser` + `TypologyVersion` sha256 |
| DDL (FR34–40) | 7 | ✅ `SchemaGraph` DAG + `TopologicalSorter` |
| APIs (FR41–45) | 5 | ✅ `ExportController` StreamingResponseBody (D-09) |
| Sécurité (FR46–50) | 5 | ✅ LDAP + JWT + RBAC + Audit |
| Activité (FR51–53) | 3 | ✅ `ActivityEvent` + `AuditLog` |

**26 NFRs — Couverture complète :**

| NFR clé | Couverture |
|---|---|
| NFR-P02 : 1M lignes/min | ✅ Spring Batch chunks 10k + thread pool configurable |
| NFR-SC01 : 1000 users | ✅ API stateless JWT + HikariCP pool |
| NFR-SC03 : datasets 10M lignes | ✅ Streaming D-09 + partitionnement D-01 |
| NFR-S01 : SSO LDAP | ✅ `LdapAuthProvider` + Spring Security LDAP |
| NFR-S04 : audit 100% | ✅ AOP `@Auditable` → `audit_logs` |
| NFR-R03 : reproductibilité | ✅ PRNG déterministe + `typology_id + version + sha256` |

### Points d'Attention identifiés par Revue Multi-Équipe (Party Mode)

> Trois enrichissements non-bloquants identifiés lors de la revue d'équipe :

| # | Action | Priorité | Responsable |
|---|---|---|---|
| A1 | `TopologicalSorter` : détecter cycles FK avant tri (exception `CycleDetectedException`) | 🔴 Sprint 1 | Dev |
| A2 | `application-dev.yml` : profil auth in-memory fallback si LDAP indisponible | 🔴 Sprint 1 | Dev |
| A3 | `YamlTypologyParser` : validation JSON Schema sur YAML soumis (éviter injection) | 🟠 Sprint 2 | Dev |

### Checklist de Complétude ✅

- [x] Analyse du contexte (53 FRs, 26 NFRs, 5 bounded contexts)
- [x] Décisions architecturales D-01→D-16 documentées avec justifications
- [x] Stack technique versionnée (Spring Boot 3.2.x, Java 17, React 18, PG 15+)
- [x] Patterns P-01→P-09 définissant les règles obligatoires agents
- [x] Structure projet complète (arborescence `gendata-backend/` + `gendata-frontend/`)
- [x] Frontières API, données, state documentées
- [x] Mapping 53 FRs → modules/fichiers
- [x] Intégrations externes (LDAP, PostgreSQL, Docker, CI/CD)
- [x] Validation cohérence complète
- [x] 3 enrichissements intégrés (A1/A2/A3)

### Verdict Final

> **🟢 ARCHITECTURE PRÊTE POUR L'IMPLÉMENTATION**
>
> Niveau de confiance : **ÉLEVÉ** — Toutes les décisions critiques documentées, tous les patterns définis, structure complète et cohérente. Les agents dev peuvent démarrer sans ambiguïté architecturale. Les 3 actions A1/A2/A3 sont à inclure dans les premières stories techniques.

---

## Guide de Passation — Implémentation

_Ce document est la source unique de vérité pour toutes les décisions techniques de GenData. Tout agent dev doit le lire intégralement avant de produire du code._

### Priorité d'Implémentation Sprint 1

| Composant | Décision | Pourquoi en premier |
|---|---|---|
| Auth LDAP + JWT | D-04 | Bloque toute l'API |
| RBAC `SpaceSecurityFilter` | D-05 | Bloque l'isolation des espaces |
| Table `generated_rows` partitionnée | D-01 | Bloque le moteur batch |
| Feature folders frontend | D-13 | Structure de base du code |
| Profil `dev` auth in-memory | A2 | Développement sans LDAP d'entreprise |
| `CycleDetectedException` dans DDL | A1 | Sécurité du tri topologique |

### Références Rapides pour Agents

```
Architecture  → prjdocs/planning-artifacts/architecture.md (ce document)
PRD           → prjdocs/planning-artifacts/prd.md
UX Design     → prjdocs/ux-design-specification.md
Maquette      → prjdocs/mockup/index.html
```

### Prochaines Étapes Recommandées

1. **`bmad-create-epics-and-stories`** — Décomposer les 53 FRs en epics et stories implémentables
2. **Re-run `bmad-check-implementation-readiness`** — Valider avec architecture + UX complètes
3. **Sprint Planning S1** — Auth, RBAC, schéma DB, feature folders

---

_Architecture GenData — Complétée le 28 mars 2026 — Nouredine_
