---
stepsCompleted: ['step-01-validate-prerequisites', 'step-02-design-epics', 'step-03-create-stories', 'step-04-final-validation']
status: final
inputDocuments:
  - prjdocs/planning-artifacts/prd.md
  - prjdocs/planning-artifacts/architecture.md
  - prjdocs/planning-artifacts/ux-designs/ux-bmad690-2026-06-23/DESIGN.md
  - prjdocs/planning-artifacts/ux-designs/ux-bmad690-2026-06-23/EXPERIENCE.md
---

# GenData - Epic Breakdown

## Overview

Ce document fournit la décomposition complète des epics et stories pour GenData, en décomposant les exigences du PRD, de la spécification UX (DESIGN.md + EXPERIENCE.md) et des décisions architecturales en stories implémentables.

## Requirements Inventory

### Functional Requirements

FR01: Un administrateur peut créer un espace de travail avec un libellé de premier niveau personnalisable (Tribu, Département, Équipe, ou terme libre)
FR02: Un administrateur peut modifier le libellé du premier niveau d'un espace — le changement se propage dans toute l'interface
FR03: Un utilisateur peut créer un Projet dans un espace auquel il appartient
FR04: Un utilisateur peut créer un Sous-projet dans un Projet
FR05: Un utilisateur ne peut voir et accéder qu'aux espaces dont il est membre
FR06: Un administrateur peut consulter l'activité globale d'un espace (membres, datasets, volumes générés)
FR07: Un administrateur peut archiver un sous-projet et ses contenus
FR08: Un utilisateur peut naviguer dans la hiérarchie Espace → Projet → Sous-projet depuis une sidebar persistante
FR09: Un utilisateur peut créer un dataset dans un sous-projet en uploadant un fichier CSV (en-tête + au moins une ligne d'exemple)
FR10: Le système propose automatiquement une détection de typologie pour chaque colonne du CSV, avec un score de confiance
FR11: Un utilisateur peut accepter, modifier ou rejeter la détection automatique pour chaque colonne
FR12: Un utilisateur peut assigner une liste de valeurs personnalisée à une colonne
FR13: Un utilisateur peut assigner une liste de valeurs pondérée à une colonne (probabilité par valeur)
FR14: Un utilisateur peut configurer le nombre de lignes à générer et un seed de reproductibilité
FR15: Un utilisateur peut consulter les données d'un dataset en prévisualisation (50 lignes minimum)
FR16: Un utilisateur peut télécharger un dataset au format JSON ou CSV
FR17: Un utilisateur peut partager un dataset via une URL stable (deep-link)
FR18: Un utilisateur peut restaurer la configuration originale d'un dataset après modification
FR19: Le système conserve une copie immuable de la configuration initiale de chaque dataset
FR20: Un administrateur peut archiver ou supprimer un dataset
FR21: Le système génère les données de manière asynchrone et notifie l'utilisateur à la fin du job
FR22: Un utilisateur peut consulter l'état d'avancement d'un job de génération en cours (progression en %)
FR23: Un dataset regénéré avec le même seed et la même configuration produit des données identiques
FR24: Le système génère des données conformes aux contraintes de la typologie assignée (ex. IBAN Modulo-97 valide)
FR25: Un utilisateur peut regénérer un dataset en modifiant uniquement le seed ou le nombre de lignes
FR26: Le système limite le nombre de jobs simultanés par utilisateur (configurable par admin)
FR27: Un utilisateur peut parcourir le catalogue de typologies filtré par catégorie, nom ou tag
FR28: Un utilisateur peut consulter la définition, les contraintes et des exemples de valeurs pour chaque typologie
FR29: Un utilisateur peut générer des exemples de valeurs à la demande pour une typologie (Test Lab)
FR30: Un utilisateur peut créer une nouvelle définition de typologie via un assistant guidé (formulaire + prévisualisation live)
FR31: L'assistant de création de typologie valide la syntaxe de la définition et génère 5 exemples avant enregistrement
FR32: Un utilisateur peut exporter la définition d'une typologie au format YAML
FR33: Un administrateur peut publier une nouvelle typologie pour qu'elle soit disponible à tous les membres
FR34: Un utilisateur peut importer un schéma SQL DDL (upload de fichier ou saisie directe dans un éditeur)
FR35: Le système analyse le schéma DDL et construit un graphe de dépendances entre tables (clés étrangères)
FR36: Un utilisateur peut visualiser le graphe de dépendances FK sous forme de diagramme interactif
FR37: Le système détecte les cycles de dépendances dans le schéma et avertit l'utilisateur
FR38: Un utilisateur peut générer des données pour toutes les tables du schéma dans l'ordre correct des dépendances FK
FR39: Un utilisateur peut configurer le nombre de lignes à générer par table
FR40: Un utilisateur peut assigner des typologies aux colonnes d'un schéma DDL avant génération
FR41: Un système externe peut déclencher la génération d'un dataset via API REST (avec authentification)
FR42: Un système externe peut extraire une ligne spécifique d'un dataset par identifiant UUID via API
FR43: Un système externe peut extraire des lignes d'un dataset par valeur d'une ou plusieurs colonnes via API
FR44: Un système externe peut lire, créer, modifier et supprimer des configurations de datasets via API CRUD
FR45: Un système externe peut télécharger un dataset généré au format JSON ou CSV via API
FR46: Un utilisateur s'authentifie via SSO LDAP/Active Directory (pas de compte local)
FR47: Un administrateur peut ajouter ou retirer des membres d'un espace
FR48: Le système journalise chaque opération de génération (utilisateur, date, dataset, volume) dans un audit trail
FR49: Les données générées sont marquées comme « TEST DATA ONLY » dans les exports
FR50: Un administrateur peut configurer la durée de rétention des datasets générés et déclencher une purge
FR51: Le système enregistre les événements sur chaque dataset : consulté, modifié, téléchargé
FR52: Un utilisateur peut consulter l'historique d'activité d'un dataset (qui, quand, quelle action)
FR53: Un administrateur peut consulter les statistiques d'usage de l'espace (datasets actifs, volumes, membres actifs)

### NonFunctional Requirements

NFR-P01: Les endpoints CRUD sur datasets répondent en < 200ms pour des jeux de ≤ 10 000 lignes — P95
NFR-P02: Le moteur génère ≥ 1 000 000 lignes/minute en régime nominal (Spring Batch, chunks 10k, PostgreSQL)
NFR-P03: Un job de génération passe en RUNNING dans les 5 secondes suivant la requête
NFR-P04: Le début d'un téléchargement (premier byte) démarre en < 3 secondes quel que soit le volume — streaming HTTP chunked
NFR-P05: L'application React est interactive (TTI) en < 2 secondes sur connexion interne standard
NFR-P06: L'analyse d'un schéma DDL de ≤ 50 tables et la construction du graphe FK s'effectuent en < 3 secondes
NFR-S01: Toute session nécessite une authentification SSO LDAP/AD validée — aucun endpoint fonctionnel accessible sans token JWT valide
NFR-S02: Un utilisateur ne peut accéder à aucune ressource d'un espace dont il n'est pas membre — validé à chaque requête
NFR-S03: Toutes les communications client ↔ serveur s'effectuent en HTTPS/TLS 1.2+ — HTTP non chiffré refusé
NFR-S04: 100% des opérations de génération sont loguées : utilisateur, horodatage UTC, dataset ID, volume — rétention minimum 1 an
NFR-S05: 100% des exports (JSON, CSV) incluent un champ `_gendata_marker: TEST_DATA_ONLY`
NFR-S06: Aucune donnée réelle ne transite dans le moteur — les CSV uploadés ne servent qu'à l'analyse des en-têtes et la première ligne d'exemple
NFR-S07: Les tokens JWT expirent après 8h maximum — aucune session permanente
NFR-SC01: 1 000 utilisateurs connectés simultanément sans dégradation (< 20% d'augmentation du P95)
NFR-SC02: ≥ 200 jobs de génération simultanés sans perte ni corruption
NFR-SC03: Un dataset peut contenir jusqu'à 10 000 000 de lignes sans erreur de génération ni d'export
NFR-SC04: Jusqu'à 500 typologies sans dégradation du temps de chargement du browser
NFR-SC05: L'architecture supporte l'ajout d'un deuxième tenant sans refactoring majeur du core
NFR-R01: ≥ 99,5% en heures ouvrables (8h–20h, lundi–vendredi) — tolérance 3,6h/mois
NFR-R02: Un job interrompu ne produit pas de données partiellement écrites — reprise propre ou annulation
NFR-R03: Régénération avec le même seed + même YAML de typo = données binaires identiques, garanti inter-versions
NFR-R04: La configuration initiale de chaque dataset est immuable et non écrasable par des opérations utilisateur normales
NFR-I01: L'API REST est versionnée via header `API-Version` — version N disponible ≥ 6 mois après sortie de N+1
NFR-I02: Tous les endpoints acceptent et retournent du JSON UTF-8 (`Content-Type: application/json`)
NFR-I03: Sémantique HTTP respectée : 200/201/202 succès, 400 erreur client, 401/403 auth, 404 absent, 500 erreur serveur
NFR-I04: Intégration LDAP v3 et LDAPS (port 636) — configurable sans recompilation

### Additional Requirements

- **Starter Frontend** : `npm create vite@latest gendata-frontend -- --template react-ts` — décisions de build établies par ce starter (React 18, TypeScript 5 strict, Vite)
- **Starter Backend** : Spring Initializr — Spring Boot 3.4.x, Java 17, Maven, dépendances web/data-jpa/postgresql/security/batch/validation/actuator/ldap
- **D-01 Table partitionnée** : `generated_rows` partitionnée par `dataset_id` (PostgreSQL native) — purge par DROP PARTITION, index composé `(dataset_id, row_index)`
- **D-02 Versioning typologies** : chaque typo publiée est immuable — `typology_id + version + sha256` référencé par chaque dataset pour garantir NFR-R03
- **D-03 Config dataset immuable** : colonnes `original_config JSONB` (immuable) + `current_config JSONB` (modifiable) — restauration = copie
- **D-04 Auth LDAP + JWT** : `LdapAuthenticationProvider` → JWT 8h stateless — URL LDAP + Base DN dans `application.yml`
- **D-05 RBAC par espace** : `SpaceSecurityFilter` Spring Security — vérifie membership sur `/api/workspaces/{id}/**` et chemins imbriqués (DDL, export)
- **D-06 Audit AOP** : annotation `@Auditable` sur toutes méthodes de génération ET export — `AuditTrailAspect @AfterReturning`
- **D-07 API versioning** : header `API-Version: 1` — absent header → rejet `400 MISSING_API_VERSION`
- **D-08 Polling async** : `refetchInterval: 2000ms` React Query sur `GET /api/datasets/{id}/status`
- **D-09 Export streaming** : `StreamingResponseBody` + `Transfer-Encoding: chunked` — jamais de bufferisation complète
- **D-10 Format erreur standard** : `{error, message, field, timestamp}` — `@ControllerAdvice` global, jamais de stack trace
- **D-11 State management** : React Query = server state ; Zustand = UI state ; pas de Redux
- **D-12 Routing deep-link** : React Router v6 — URL canonical et stable par ressource, paramètres session synchronisés dans URL
- **D-13 Feature folders** : `src/features/{workspace|dataset|typology|ddl|generation|activity}/`
- **D-14 Dev local** : `docker-compose.yml` — postgres:15 + adminer
- **D-15 CI/CD** : GitHub Actions — `mvn verify` + `npm run build` + `npx cypress-axe` + Docker build
- **D-16 Déploiement MVP** : 2 containers (Nginx statique + JAR Spring Boot) sur VM interne — pas de Kubernetes avant Phase 3
- **PRNG canonique** : Column seed = `Long.hashCode(datasetSeed) ^ columnIndex` ; PRNG = `new java.util.Random(columnSeed)` — aucun autre générateur autorisé
- **Single writer jobs** : seul `GenerationJobListener` (Batch) écrit les transitions de status après soumission — `dataset-api` appelle `requestCancel()` sans écrire directement
- **Quota jobs** : `GenerationService.submitJob()` vérifie le count avant création — rejette `429` si > 5 jobs actifs/user
- **Observabilité D-17** : format log JSON structuré (Logback + logstash-logback-encoder), MDC avec `correlationId` propagé HTTP→Batch, Actuator sur port admin
- **CORS** : origines autorisées restreintes en prod (domaine bancaire interne uniquement), `localhost:5173` en dev
- **React Query keys** : formes canoniques `['workspaces', wid]`, `['datasets', wid, pid, spid]`, `['dataset', did]`, `['generation', did]` — on COMPLETED invalider `['dataset', did]` + `['datasets', wid, pid, spid]`
- **A1** : `CycleDetectedException` dans `TopologicalSorter` avant génération DDL
- **A2** : Profil `dev` auth in-memory fallback quand LDAP indisponible
- **A3** : `YamlTypologyParser` valide JSON Schema sur YAML soumis avant enregistrement

### UX Design Requirements

UX-DR1: Implémentation du brand layer MUI v6 — theme override avec les tokens DESIGN.md : primary #1565c0, success #2e7d32, warning #f57c00, error #d32f2f, police Inter (body), Roboto Mono (données), border-radius sm/md/lg (4/6/8px), base unit 4px
UX-DR2: Composant C-01 ConfidenceBadge — pill couleur par seuil (≥80% vert #2e7d32, 50-79% orange #f57c00, <50% rouge #d32f2f), texte blanc, `role="status"`, `aria-label="Confiance {v}%"`, tooltip explicatif au survol
UX-DR3: Composant C-02 SeedBadge — Roboto Mono, bg #f5f5f5 border #e0e0e0 radius 6px, clic copie clipboard, animation checkmark 1.5s retour défaut, `aria-label="Seed de reproductibilité : {v}. Cliquer pour copier"`, toujours visible et proéminent
UX-DR4: Composant C-03 TypologyChip — états validé (bg #e3f2fd, text #1565c0) / avertissement (bg #fff3e0, text #f57c00) / manuel (violet pâle) ; radius 4px ; clic ouvre drawer filtré sur la colonne ; `role="button"`, `aria-label="Changer la typologie : {nom}"`
UX-DR5: Composant C-04 ColumnConfigRow — anatomie : # · nom colonne · TypologyChip · ConfidenceBadge · toggle Nullable (aria-checked) · toggle Unique (aria-checked) · bouton ⚙ ; états : normal (bg white) / attention (bg #fff8e1, border-left 3px #f57c00) / verrouillé FK (bg #e3f2fd, ⚙ désactivé, tooltip)
UX-DR6: Composant C-05 FKGraphCanvas — React Flow, nœuds parent (bg #2e7d32 blanc) / enfant (bg #1565c0 blanc) / sélectionné (border #90caf9) ; drag repositionnement ; hover flèche FK → tooltip `TABLE_A.col → TABLE_B.col` ; clic nœud → focus dans table config ; ARIA `role="button"` + aria-label par nœud et flèche
UX-DR7: Composant C-06 GenerationProgressCard — 4 états (pending skeleton / running barre animée + compteurs / completed alerte verte + 3 actions / error message + Retry) ; `role="progressbar"`, `aria-valuenow`, `aria-valuemax` ; jamais de stack trace
UX-DR8: Composant C-07 LabelSettingsPanel — panneau inline sous en-tête sidebar (pas Drawer, pas Dialog) ; chips Tribu/Département/Équipe/Domaine/BU + saisie libre ; bouton OK ; fermeture Échap / clic hors / OK
UX-DR9: Layout trois zones — topbar 48px bg #1565c0 z-100 sticky, sidebar 260px fixe bg #ffffff overflow-y auto, contenu flex:1 bg #f5f5f5 padding 24px overflow-y auto — jamais de sidebar repliable sur desktop ≥ 1024px
UX-DR10: Navigation sidebar TreeView — item actif bg #e3f2fd text #1565c0 ; responsive tablet <1024px : overlay hamburger consultation seule (config colonne et génération désactivés) ; mobile <768px : page "outil non disponible sur mobile"
UX-DR11: Deep-links URL — URL canonique stable par ressource ; paramètres session (colonne active, drawer ouvert, seed saisi) synchronisés dans URL ; aucune donnée de navigation en mémoire session seule ; breadcrumb sur toutes vues, dernier élément non cliquable
UX-DR12: Drawer configuration typologique — 400px slide-in droite, table config visible gauche, Typology Browser (recherche debounce 300ms + chips catégories) + Test Lab inline, footer sticky Annuler / Assigner, Échap ferme sans piège focus ; Dialog uniquement pour confirmations destructives
UX-DR13: Accessibilité WCAG 2.1 AA comportementale — focus ring #90caf9 2px offset sur tous interactifs (jamais outline:none sans alternative) ; ARIA complet (C-01 à C-07) ; keyboard nav tab logique sidebar→table→drawers sans piège (sauf Dialog) ; `prefers-reduced-motion` désactive animations checkmark et auto-detect ; `lang="fr"` sur html ; cibles tactiles 32×32px desktop / 44×44px tablet
UX-DR14: États vides et chargement — skeleton MUI sur chargement initial (pas spinner global) ; CircularProgress 16px inline colonne Confiance pendant auto-détection ; état vide : icône illustrative + message contextuel + CTA ; erreur inline + Retry (jamais page d'erreur complète ni stack trace) ; reset filtres visible uniquement si filtre actif
UX-DR15: Voix et ton — messages d'erreur UI en français (jamais stack trace technique) ; Snackbar 5s auto-dismiss succès / dismiss manuel erreur ; féliciter avant informer sur succès ("50 000 lignes générées." avant le détail) ; format professionnel direct sans affect

### FR Coverage Map

FR01: Epic 1 — Création d'un espace de travail avec libellé personnalisable
FR02: Epic 1 — Modification du libellé de premier niveau (propagation UI)
FR03: Epic 1 — Création d'un Projet dans un espace
FR04: Epic 1 — Création d'un Sous-projet dans un Projet
FR05: Epic 1 — Visibilité restreinte aux espaces dont l'utilisateur est membre
FR06: Epic 7 — Dashboard d'activité globale pour administrateur
FR07: Epic 1 — Archivage d'un sous-projet et de ses contenus
FR08: Epic 1 — Navigation hiérarchique depuis la sidebar persistante
FR09: Epic 2 — Création d'un dataset par upload CSV
FR10: Epic 2 — Détection automatique de typologie par colonne avec score de confiance
FR11: Epic 2 — Acceptation, modification ou rejet de la détection automatique
FR12: Epic 2 — Assignation d'une liste de valeurs personnalisée à une colonne
FR13: Epic 2 — Assignation d'une liste de valeurs pondérée à une colonne
FR14: Epic 2 — Configuration du nombre de lignes et du seed de reproductibilité
FR15: Epic 2 — Prévisualisation dataset (50 lignes minimum)
FR16: Epic 2 — Téléchargement dataset au format JSON ou CSV
FR17: Epic 2 — Partage dataset via URL stable (deep-link)
FR18: Epic 2 — Restauration de la configuration originale d'un dataset
FR19: Epic 2 — Conservation immuable de la configuration initiale (original_config JSONB)
FR20: Epic 2 — Archivage ou suppression d'un dataset par l'administrateur
FR21: Epic 3 — Génération asynchrone avec notification de fin de job
FR22: Epic 3 — Consultation de la progression d'un job en cours (%)
FR23: Epic 3 — Reproductibilité garantie : même seed + même config → données identiques
FR24: Epic 3 — Données conformes aux contraintes de la typologie (ex. IBAN Modulo-97)
FR25: Epic 3 — Regénération avec seed ou nombre de lignes différent
FR26: Epic 3 — Limitation du nombre de jobs simultanés par utilisateur (configurable)
FR27: Epic 2 — Parcours du catalogue de typologies filtré par catégorie, nom ou tag
FR28: Epic 2 — Consultation de la définition, contraintes et exemples d'une typologie
FR29: Epic 4 — Génération d'exemples à la demande pour une typologie (Test Lab)
FR30: Epic 4 — Création d'une nouvelle définition de typologie via assistant guidé
FR31: Epic 4 — Validation syntaxique + génération de 5 exemples avant enregistrement
FR32: Epic 4 — Export de la définition d'une typologie au format YAML
FR33: Epic 4 — Publication d'une nouvelle typologie par l'administrateur
FR34: Epic 5 — Import d'un schéma SQL DDL (upload ou éditeur Monaco)
FR35: Epic 5 — Analyse DDL et construction du graphe de dépendances FK
FR36: Epic 5 — Visualisation interactive du graphe FK (React Flow)
FR37: Epic 5 — Détection des cycles de dépendances avec avertissement
FR38: Epic 5 — Génération de données pour toutes les tables dans l'ordre FK correct
FR39: Epic 5 — Configuration du nombre de lignes par table
FR40: Epic 5 — Assignation de typologies aux colonnes d'un schéma DDL
FR41: Epic 6 — Déclenchement de génération via API REST (authentifié)
FR42: Epic 6 — Extraction d'une ligne par identifiant UUID via API
FR43: Epic 6 — Extraction de lignes par valeur de colonne(s) via API
FR44: Epic 6 — CRUD de configurations de datasets via API
FR45: Epic 6 — Téléchargement de dataset généré via API (JSON ou CSV)
FR46: Epic 1 — Authentification SSO LDAP/Active Directory (pas de compte local)
FR47: Epic 1 — Gestion des membres d'un espace par l'administrateur
FR48: Epic 7 — Journal d'audit des opérations de génération (utilisateur, date, dataset, volume)
FR49: Epic 2 — Marquage TEST DATA ONLY dans tous les exports
FR50: Epic 7 — Configuration de la durée de rétention et déclenchement de purge
FR51: Epic 7 — Enregistrement des événements par dataset (consulté, modifié, téléchargé)
FR52: Epic 7 — Historique d'activité d'un dataset (qui, quand, quelle action)
FR53: Epic 7 — Statistiques d'usage de l'espace pour l'administrateur

## Epic List

### Epic 1 : Authentification & Organisation des espaces de travail
Les équipes peuvent s'authentifier via SSO et organiser la hiérarchie Espace → Projet → Sous-projet. Socle sur lequel tous les autres epics s'appuient.
**FRs couverts :** FR01, FR02, FR03, FR04, FR05, FR07, FR08, FR46, FR47

---

### Epic 2 : Création, configuration et export de datasets
Un développeur peut uploader un CSV, laisser le système détecter automatiquement les typologies, naviguer dans le catalogue, configurer les colonnes, prévisualiser les données et télécharger le résultat.
**FRs couverts :** FR09, FR10, FR11, FR12, FR13, FR14, FR15, FR16, FR17, FR18, FR19, FR20, FR27, FR28, FR49

---

### Epic 3 : Génération asynchrone et reproductibilité
Un utilisateur peut déclencher la génération de millions de lignes en arrière-plan, suivre la progression en temps réel et obtenir des données strictement identiques en réutilisant le même seed.
**FRs couverts :** FR21, FR22, FR23, FR24, FR25, FR26

---

### Epic 4 : Test Lab & Gestion des typologies
Un utilisateur peut tester une typologie à la volée dans le Test Lab, créer de nouvelles définitions via l'assistant guidé et les publier pour toute l'équipe.
**FRs couverts :** FR29, FR30, FR31, FR32, FR33

---

### Epic 5 : Import DDL & Génération multi-tables
Un utilisateur peut importer un schéma SQL complet, visualiser le graphe de dépendances FK, détecter les cycles et générer des données cohérentes pour toutes les tables dans le bon ordre de résolution.
**FRs couverts :** FR34, FR35, FR36, FR37, FR38, FR39, FR40

---

### Epic 6 : Intégration API REST
Un système CI/CD ou une application externe peut piloter GenData entièrement via API — déclencher la génération, extraire des lignes ciblées et gérer les configurations de datasets.
**FRs couverts :** FR41, FR42, FR43, FR44, FR45

---

### Epic 7 : Administration, suivi d'activité & conformité
Un administrateur dispose d'une visibilité complète sur l'usage de la plateforme, peut piloter la rétention des données et s'assurer de la conformité des traces d'audit.
**FRs couverts :** FR06, FR48, FR50, FR51, FR52, FR53

---

## Epic 1 — Authentification & Organisation des espaces de travail

**Objectif :** Les équipes peuvent s'authentifier via SSO et organiser la hiérarchie Espace → Projet → Sous-projet. Socle sur lequel tous les autres epics s'appuient.
**FRs :** FR01, FR02, FR03, FR04, FR05, FR07, FR08, FR46, FR47
**NFRs clés :** NFR-S01, NFR-S03, NFR-S07, NFR-P05
**UX-DRs :** UX-DR1, UX-DR9, UX-DR10, UX-DR11, UX-DR15

---

### Story 1.0 : Initialisation du projet depuis les starters

En tant que développeur,
je veux initialiser le dépôt frontend (Vite react-ts) et le projet backend (Spring Initializr) depuis les starters officiels,
afin que l'équipe démarre avec une structure validée et les dépendances exactes prescrites par l'architecture.

**Critères d'acceptation :**

**Given** le dépôt `gendata` est vide
**When** j'exécute `npm create vite@latest gendata-frontend -- --template react-ts` (D-14)
**Then** la structure `src/features/`, `src/components/`, `tsconfig.json`, `vite.config.ts` est générée
**And** les dépendances exactes sont ajoutées : `@mui/material@^6`, `@tanstack/react-query@^5`, `zustand`, `react-router-dom@^6`, `@xyflow/react`, `@monaco-editor/react`, `recharts`, `axios`, `@rjsf/mui` (D-11, D-13)
**And** `npm run build` passe sans erreur

**Given** le module backend est initialisé via Spring Initializr
**When** le projet est généré avec : Spring Boot 3.4.x, Java 17, Maven, dépendances `web / data-jpa / postgresql / security / batch / validation / actuator / ldap`
**Then** `mvn verify` passe (tests vides mais compilation propre)
**And** `docker-compose.yml` avec `postgres:15` + adminer est présent (D-14)
**And** le profil `dev` inclut le fallback auth in-memory (A2)

**Given** la structure de features est créée
**When** les dossiers `src/features/{workspace,dataset,typology,ddl,generation,activity}/` sont créés (D-13)
**Then** le projet compile avec `npm run build` et passe `mvn verify`

_Aucune table créée, aucun endpoint exposé — bootstrapping uniquement._
_Décisions : D-13, D-14, D-15, A2_

---

### Story 1.1 : Authentification SSO LDAP/JWT côté serveur

En tant qu'administrateur système,
je veux que les utilisateurs s'authentifient via LDAP et reçoivent un JWT stateless,
afin que toutes les requêtes API soient sécurisées sans état de session côté serveur.

**Critères d'acceptation :**

**Given** un username/password LDAP valide
**When** POST `/api/auth/login` avec `API-Version: 1`
**Then** le serveur valide via `LdapAuthenticationProvider`, retourne un JWT signé (expiry 8h) — 200 OK
**And** le payload JWT contient : userId, displayName, roles

**Given** des credentials invalides
**When** POST `/api/auth/login`
**Then** 401 Unauthorized au format `{error, message, timestamp}` — jamais de stack trace (D-10)

**Given** header `API-Version` absent
**When** tout endpoint protégé est appelé
**Then** 400 BAD_REQUEST, code `MISSING_API_VERSION` (D-07)

**Given** JWT expiré
**When** tout endpoint protégé est appelé
**Then** 401, code `EXPIRED_TOKEN`

**NFRs :** NFR-S01, NFR-S03, NFR-S07 | **Décisions :** D-04, D-07

---

### Story 1.2 : Interface de connexion et état d'authentification (frontend)

En tant qu'utilisateur GenData,
je veux une page de connexion et une persistance de session dans le navigateur,
afin de naviguer librement sans me reconnecter à chaque page.

**Critères d'acceptation :**

**Given** j'ouvre l'application sans JWT valide
**When** je tente d'accéder à une route protégée
**Then** je suis redirigé vers `/login`

**Given** je soumets des credentials valides
**When** la réponse inclut un JWT
**Then** le JWT est stocké en mémoire via Zustand (pas localStorage) (D-11)
**And** toutes les requêtes Axios suivantes incluent `Authorization: Bearer {token}` et `API-Version: 1`
**And** je suis redirigé vers la route initialement demandée

**Given** credentials invalides
**When** l'API retourne 401
**Then** message français : "Identifiant ou mot de passe incorrect." — sans stack trace (UX-DR15)

**Given** JWT expiré pendant la session
**When** une requête API retourne 401
**Then** redirection vers `/login`, et après re-connexion retour à la page interrompue

**Décisions :** D-07, D-11, D-12

---

### Story 1.3 : Création et personnalisation d'un espace de travail

En tant qu'administrateur,
je veux créer un espace de travail avec un libellé de premier niveau personnalisable,
afin que mon équipe s'organise sous une hiérarchie qui reflète sa structure (Tribu, Département, etc.).

**Critères d'acceptation :**

**Given** je suis authentifié en tant qu'administrateur
**When** POST `/api/workspaces` `{name: "KYC Ops", labelType: "TRIBU"}` avec `API-Version: 1`
**Then** l'espace est créé — 201 Created, réponse inclut id, name, labelType, createdAt
**And** le créateur est automatiquement ajouté comme premier membre (workspace_members)

**Given** je veux renommer le libellé
**When** PATCH `/api/workspaces/{id}` `{labelType: "DEPARTEMENT"}`
**Then** labelType mis à jour — 200 OK
**And** le changement se propage dans l'UI (breadcrumb, sidebar) sans rechargement de page

**Given** nom d'espace déjà existant
**When** POST `/api/workspaces`
**Then** 409 Conflict avec message français clair

**Given** non-administrateur
**When** tentative de création
**Then** 403 Forbidden

**Tables créées :** `workspaces`, `workspace_members` | **Décisions :** D-05, D-10

---

### Story 1.4 : Gestion de la hiérarchie Projet / Sous-projet

En tant que membre d'un espace de travail,
je veux créer des Projets et Sous-projets dans mon espace,
afin d'organiser les datasets par domaine métier et par itération.

**Critères d'acceptation :**

**Given** je suis membre de l'espace {wid}
**When** POST `/api/workspaces/{wid}/projects` `{name: "Crédit Hypothécaire"}`
**Then** le projet est créé — 201 Created avec project ID

**Given** je suis membre du projet {pid}
**When** POST `.../projects/{pid}/subprojects` `{name: "Sprint 12 — Données UAT"}`
**Then** le sous-projet est créé — 201 Created avec subproject ID

**Given** je ne suis pas membre de {wid}
**When** toute opération sur `/api/workspaces/{wid}/**`
**Then** `SpaceSecurityFilter` retourne 403 (D-05)

**Given** nom dépassant 128 caractères
**When** création
**Then** 400 Bad Request avec message de validation en français

**Tables créées :** `projects`, `subprojects` | **Décisions :** D-05

---

### Story 1.5 : Gestion des membres et contrôle de visibilité

En tant qu'administrateur,
je veux ajouter ou retirer des membres de mon espace,
afin que seules les personnes autorisées accèdent aux projets et datasets.

**Critères d'acceptation :**

**Given** je suis administrateur de l'espace {wid}
**When** POST `/api/workspaces/{wid}/members` `{userId: "jdupont"}`
**Then** l'utilisateur est ajouté — 200 OK

**When** DELETE `/api/workspaces/{wid}/members/jdupont`
**Then** l'utilisateur est retiré — 204 No Content

**Given** jdupont N'est PAS membre de {wid}
**When** jdupont appelle GET `/api/workspaces/{wid}`
**Then** `SpaceSecurityFilter` retourne 403 (FR05, D-05)

**Given** jdupont EST membre de {wid} uniquement
**When** GET `/api/workspaces`
**Then** seul l'espace {wid} apparaît dans sa liste

**Table modifiée :** `workspace_members`

---

### Story 1.6 : Shell applicatif — layout trois zones et navigation sidebar

En tant qu'utilisateur GenData,
je veux une navigation persistante affichant ma hiérarchie d'espaces,
afin de passer d'un Projet à un Sous-projet en un clic sans perdre le contexte.

**Critères d'acceptation :**

**Given** je suis connecté
**When** l'application charge
**Then** le layout trois zones s'affiche : topbar 48px (#1565c0), sidebar 260px (#fff), contenu flex:1 (#f5f5f5, padding 24px) (UX-DR9)

**Given** la sidebar charge
**When** la hiérarchie est récupérée via React Query (`['workspaces', wid]`)
**Then** le TreeView affiche Espace → Projets → Sous-projets (UX-DR10)
**And** l'item actif est mis en évidence (bg #e3f2fd, text #1565c0)
**And** la sidebar reste fixe sur desktop ≥ 1024px

**Given** je clique un sous-projet
**When** la route change
**Then** l'URL devient `/workspaces/{wid}/projects/{pid}/subprojects/{spid}` (UX-DR11)
**And** un breadcrumb Espace > Projet > Sous-projet apparaît en haut du contenu (dernier élément non cliquable)

**Given** tablet < 1024px
**When** la page charge
**Then** la sidebar est masquée, accessible via hamburger (UX-DR10)
**And** les actions de configuration et génération sont désactivées avec tooltip

**Given** un utilisateur copie et partage l'URL
**When** un autre membre autorisé l'ouvre
**Then** il arrive sur la même vue (UX-DR11)

**NFR :** NFR-P05 (TTI < 2s) | **UX-DRs :** UX-DR1, UX-DR9, UX-DR10, UX-DR11 | **Décisions :** D-11, D-12, D-13

---

### Story 1.7 : Archivage d'un sous-projet

En tant qu'administrateur,
je veux archiver un sous-projet terminé,
afin qu'il ne pollue pas la navigation active tout en restant consultable.

**Critères d'acceptation :**

**Given** je suis administrateur de l'espace
**When** PATCH `/api/workspaces/{wid}/projects/{pid}/subprojects/{spid}/archive`
**Then** status du sous-projet passe à `ARCHIVED` — 200 OK

**Given** un sous-projet est ARCHIVED
**When** GET `.../subprojects` sans paramètre
**Then** les sous-projets archivés N'apparaissent PAS dans la liste par défaut
**And** GET `.../subprojects?includeArchived=true` les retourne avec `status: ARCHIVED`

**Given** non-administrateur
**When** tentative d'archivage
**Then** 403 Forbidden

**Table modifiée :** `subprojects` (ajout : `status`, `archived_at`)

---

## Epic 2 — Création, configuration et export de datasets

**Objectif :** Un développeur peut uploader un CSV, détecter automatiquement les typologies, naviguer dans le catalogue, configurer les colonnes, prévisualiser et télécharger le résultat.
**FRs :** FR09, FR10, FR11, FR12, FR13, FR14, FR15, FR16, FR17, FR18, FR19, FR20, FR27, FR28, FR49
**NFRs clés :** NFR-P01 (CRUD < 200ms), NFR-P04 (premier byte < 3s), NFR-S05 (marquage TEST DATA), NFR-S06 (pas de données réelles)
**UX-DRs :** UX-DR2, UX-DR3, UX-DR4, UX-DR5, UX-DR8, UX-DR12, UX-DR13, UX-DR14, UX-DR15

---

### Story 2.1 : Upload CSV et création d'un dataset

En tant que développeur,
je veux créer un dataset en uploadant un fichier CSV (en-tête + au moins une ligne d'exemple),
afin que la plateforme extraie automatiquement la structure de mes colonnes.

**Critères d'acceptation :**

**Given** je suis sur la page d'un sous-projet actif
**When** je soumets un fichier CSV via POST `/api/workspaces/{wid}/.../datasets` (multipart)
**Then** le serveur analyse l'en-tête et la première ligne d'exemple uniquement (NFR-S06)
**And** un dataset est créé — 201 Created avec dataset ID et liste des colonnes extraites (nom + type Java inféré)
**And** la configuration initiale est sauvegardée dans `original_config JSONB` (FR19, D-03, immuable)

**Given** le CSV ne contient pas d'en-tête ou de ligne d'exemple
**When** upload
**Then** 400 Bad Request : "Le fichier CSV doit contenir une ligne d'en-tête et au moins une ligne de données."

**Given** le sous-projet est archivé
**When** tentative de création de dataset
**Then** 403 Forbidden

**Given** aucun dataset dans le sous-projet
**When** la page du sous-projet charge
**Then** état vide : icône illustrative + "Importez un fichier CSV pour créer votre premier dataset." + bouton CTA (UX-DR14)

**Tables créées :** `datasets`, `dataset_columns` | **NFR :** NFR-S06, NFR-P01

---

### Story 2.2 : Détection automatique de typologies avec score de confiance

En tant que développeur,
je veux que le système propose automatiquement une typologie pour chaque colonne avec un score de confiance,
afin d'éviter de tout configurer manuellement sur les datasets bien structurés.

**Critères d'acceptation :**

**Given** un dataset vient d'être créé
**When** POST `.../datasets/{did}/detect` est appelé
**Then** chaque colonne reçoit `suggestedTypology` (id YAML) et `confidenceScore` (0–100)
**And** les suggestions s'appuient sur les 112 typologies YAML existantes (chargées au démarrage)

**Given** la détection est en cours
**When** l'UI affiche la table de colonnes
**Then** un `CircularProgress` 16px inline s'affiche à la place du score de confiance (UX-DR14)

**Given** la détection est terminée
**When** l'UI reçoit les résultats
**Then** chaque colonne affiche un `ConfidenceBadge` (C-01) : ≥ 80% vert / 50–79% orange / < 50% rouge (UX-DR2)
**And** `role="status"`, `aria-label="Confiance {v}%"` sur chaque badge (UX-DR13)

**Given** aucune correspondance fiable (score < seuil minimal)
**When** la détection se termine
**Then** badge rouge < 50%, tooltip : "Aucune correspondance fiable — assignez une typologie manuellement."

**Tables modifiées :** `dataset_columns` (ajout : `suggested_typology`, `confidence_score`) | **NFR :** NFR-P01

---

### Story 2.3 : Configuration des colonnes — typologies et contraintes

En tant que développeur,
je veux accepter, modifier ou rejeter la détection automatique et configurer les contraintes de chaque colonne,
afin que les données générées correspondent exactement à mes besoins métier.

**Critères d'acceptation :**

**Given** je consulte la table de configuration d'un dataset
**When** la liste des colonnes s'affiche
**Then** chaque ligne suit l'anatomie C-04 : # · nom · TypologyChip · ConfidenceBadge · toggle Nullable (aria-checked) · toggle Unique (aria-checked) · bouton ⚙ (UX-DR5)
**And** une colonne en attention (confidence < 50%) affiche `border-left: 3px solid #f57c00` et fond #fff8e1 (UX-DR5)

**Given** je clique le bouton ⚙ d'une colonne
**When** le Drawer s'ouvre (400px, slide-in droite)
**Then** je peux toggler Nullable et Unique avec aria-checked (UX-DR12, UX-DR13)
**And** Échap ou clic hors du Drawer le ferme sans piège focus

**Given** je veux assigner une liste de valeurs personnalisée (FR12)
**When** je saisis les valeurs et valide
**Then** `current_config` est mis à jour — `original_config` reste immuable (D-03)
**And** PATCH `.../datasets/{did}/columns/{colId}` retourne 200 OK

**Given** je veux assigner une liste pondérée (FR13)
**When** j'associe une probabilité à chaque valeur (ex. MALE: 0.6, FEMALE: 0.4)
**Then** la configuration pondérée est sauvegardée et validée (somme des probabilités = 1.0)
**And** si somme ≠ 1.0 : 400 Bad Request avec message français explicite

**UX-DRs :** UX-DR4, UX-DR5, UX-DR12, UX-DR13 | **Tables modifiées :** `dataset_columns` (update `current_config` JSONB)

---

### Story 2.4 : Catalogue de typologies et drawer de sélection

En tant que développeur,
je veux parcourir le catalogue de typologies et consulter leur définition depuis le drawer de configuration,
afin de choisir la bonne typologie pour chaque colonne en toute confiance.

**Critères d'acceptation :**

**Given** je clique sur un TypologyChip dans une ligne de colonne
**When** le Drawer de sélection s'ouvre
**Then** la liste des typologies du catalogue s'affiche (FR27)
**And** un champ de recherche debounce 300ms filtre par nom, catégorie ou tag (UX-DR12)
**And** des chips de catégorie permettent un filtrage rapide

**Given** je survole ou sélectionne une typologie dans la liste
**When** la fiche de détail s'affiche (FR28)
**Then** je vois : description, contraintes, et 3 exemples de valeurs générées

**Given** je clique "Assigner" sur une typologie
**When** le Drawer se ferme
**Then** le TypologyChip de la colonne se met à jour (UX-DR4)
**And** `current_config` de la colonne est mis à jour — `original_config` reste immuable

**Given** le filtre actif ne retourne aucun résultat
**When** la liste se vide
**Then** état vide + bouton "Réinitialiser les filtres" visible uniquement si filtre actif (UX-DR14)

**NFR :** NFR-SC04 (≤ 500 typologies sans dégradation) | **UX-DRs :** UX-DR4, UX-DR12, UX-DR14

---

### Story 2.5 : Paramètres de génération — nombre de lignes et seed

En tant que développeur,
je veux configurer le nombre de lignes à générer et un seed de reproductibilité,
afin de contrôler le volume et garantir des données identiques entre deux runs.

**Critères d'acceptation :**

**Given** je suis sur la vue dataset
**When** je saisis le nombre de lignes et un seed
**Then** les valeurs sont sauvegardées dans `current_config`

**Given** le SeedBadge (C-02) est affiché
**When** l'UI rend le badge
**Then** il est toujours visible et proéminent : bg #f5f5f5, border #e0e0e0, Roboto Mono (UX-DR3)
**And** un clic copie le seed dans le clipboard avec animation checkmark 1.5s puis retour état initial
**And** `aria-label="Seed de reproductibilité : {v}. Cliquer pour copier"` (UX-DR13)

**Given** le nombre de lignes ≤ 0 ou non numérique
**When** validation frontend
**Then** message d'erreur français inline, soumission bloquée

**Given** aucun seed saisi au moment de la génération
**When** POST `.../generate` sans seed
**Then** un seed aléatoire est généré côté serveur et retourné dans la réponse
**And** le SeedBadge est mis à jour avec le seed généré

**UX-DRs :** UX-DR3, UX-DR13 | **Tables modifiées :** `datasets` (ajout : `row_count`, `seed`)

---

### Story 2.6 : Prévisualisation du dataset

En tant que développeur,
je veux prévisualiser les 50 premières lignes d'un dataset généré,
afin de vérifier que les données correspondent à mes attentes avant de télécharger.

**Critères d'acceptation :**

**Given** un dataset a été généré avec succès
**When** j'ouvre l'onglet Prévisualisation
**Then** GET `.../datasets/{did}/preview` retourne les 50 premières lignes — 200 OK (FR15)
**And** les données s'affichent dans un tableau paginé (en-têtes = noms de colonnes)

**Given** la prévisualisation charge
**When** les données sont en transit
**Then** skeleton MUI s'affiche (pas de spinner global) (UX-DR14)

**Given** le dataset n'a pas encore été généré
**When** j'ouvre l'onglet Prévisualisation
**Then** état vide illustré : "Générez les données pour afficher une prévisualisation." + CTA "Générer" (UX-DR14)

**NFR :** NFR-P01 (< 200ms pour ≤ 10 000 lignes)

---

### Story 2.7 : Téléchargement, marquage TEST DATA et partage du dataset

En tant que développeur,
je veux télécharger le dataset au format JSON ou CSV et partager un lien stable,
afin de l'injecter dans mes outils de test sans configuration supplémentaire.

**Critères d'acceptation :**

**Given** un dataset est en status COMPLETED
**When** je clique "Télécharger JSON" ou "Télécharger CSV"
**Then** GET `.../datasets/{did}/export?format=json|csv` démarre en streaming (StreamingResponseBody, D-09)
**And** le premier byte arrive en < 3 secondes quel que soit le volume (NFR-P04)
**And** chaque ligne exportée contient `"_gendata_marker": "TEST_DATA_ONLY"` (FR49, NFR-S05)

**Given** je clique "Copier le lien"
**When** l'URL est copiée
**Then** l'URL pointe sur `/workspaces/{wid}/.../datasets/{did}` — stable et partageable (FR17, UX-DR11)

**Given** un non-membre tente le téléchargement via URL directe
**When** GET `.../export`
**Then** 403 Forbidden (D-05 — export nested sous `/api/workspaces/{wid}/**`)

**NFR :** NFR-P04, NFR-S05 | **Décisions :** D-05, D-09

---

### Story 2.8 : Restauration de la configuration et gestion du dataset

En tant que développeur,
je veux restaurer la configuration originale d'un dataset ou le supprimer si nécessaire,
afin de corriger les erreurs de configuration ou nettoyer les datasets obsolètes.

**Critères d'acceptation :**

**Given** j'ai modifié la configuration d'un dataset
**When** je clique "Restaurer la configuration d'origine" et confirme dans la Dialog de confirmation
**Then** POST `.../datasets/{did}/restore` copie `original_config` vers `current_config` (D-03)
**And** 200 OK, l'UI reflète la configuration restaurée
**And** `original_config` reste inchangé après la restauration (FR18, FR19)

**Given** je suis administrateur et veux supprimer un dataset
**When** je confirme la suppression dans la Dialog (action destructive)
**Then** DELETE `.../datasets/{did}` supprime le dataset et ses lignes générées — 204 No Content
**And** la Dialog est le seul pattern utilisé pour les confirmations destructives (UX-DR12)

**Given** je veux archiver un dataset sans le supprimer (FR20)
**When** PATCH `.../datasets/{did}/archive`
**Then** status passe à `ARCHIVED` — le dataset n'apparaît plus dans la liste par défaut

**Décisions :** D-03 | **UX-DRs :** UX-DR12

---

## Epic 3 — Génération asynchrone et reproductibilité

**Objectif :** Un utilisateur peut déclencher la génération de millions de lignes en arrière-plan, suivre la progression en temps réel et obtenir des données strictement identiques en réutilisant le même seed.
**FRs :** FR21, FR22, FR23, FR24, FR25, FR26
**NFRs clés :** NFR-P02 (≥ 1M lignes/min), NFR-P03 (RUNNING < 5s), NFR-SC02 (≥ 200 jobs simultanés), NFR-R03 (reproductibilité)
**UX-DRs :** UX-DR7, UX-DR13, UX-DR14, UX-DR15

---

### Story 3.1 : Soumission d'un job de génération asynchrone

En tant que développeur,
je veux déclencher la génération d'un dataset et la laisser tourner en arrière-plan,
afin de continuer à travailler sans attendre la fin du job.

**Critères d'acceptation :**

**Given** un dataset est configuré (typologies, nombre de lignes, seed)
**When** POST `.../datasets/{did}/generate`
**Then** un job est créé avec status PENDING — 202 Accepted avec job ID
**And** le job est soumis à Spring Batch dans les 5 secondes (NFR-P03)
**And** `GenerationService.submitJob()` vérifie le count actif — si > 5 pour l'utilisateur, retourne 429 Too Many Requests (FR26)

**Given** l'utilisateur a déjà 5 jobs actifs
**When** POST `.../generate`
**Then** 429 : "Vous avez atteint la limite de 5 générations simultanées. Attendez qu'un job se termine."

**Given** le job démarre
**When** Spring Batch le prend en charge
**Then** SEUL `GenerationJobListener` écrit les transitions de status (PENDING → RUNNING → COMPLETED/FAILED) — `dataset-api` n'écrit JAMAIS status directement (S-02)

**Given** la génération se termine avec succès
**When** status → COMPLETED
**Then** Snackbar : "[N] lignes générées avec succès." (5s auto-dismiss) (UX-DR15)
**And** React Query invalide `['dataset', did]` ET `['datasets', wid, pid, spid]` (D-11, S-07)

**NFRs :** NFR-P03, NFR-SC02 | **Tables créées :** `generation_jobs`, `generated_rows` (partitionnée par `dataset_id`, D-01)

---

### Story 3.2 : Suivi de la progression d'un job en temps réel

En tant que développeur,
je veux voir la progression en temps réel d'un job de génération,
afin de savoir quand mes données seront prêtes.

**Critères d'acceptation :**

**Given** un job est RUNNING
**When** la vue dataset est ouverte
**Then** C-06 `GenerationProgressCard` s'affiche avec barre animée + compteurs de lignes (UX-DR7)
**And** `role="progressbar"`, `aria-valuenow={rows_done}`, `aria-valuemax={rows_total}` (UX-DR13)
**And** React Query poll GET `.../datasets/{did}/status` toutes les 2000ms (D-08)

**Given** le job est PENDING
**When** la card se rend
**Then** état skeleton : "En attente de démarrage..." — pas de spinner global (UX-DR7, UX-DR14)

**Given** le job est COMPLETED
**When** le polling retourne COMPLETED
**Then** la card affiche : alerte verte + count de lignes + 3 boutons : Télécharger / Copier le seed / Partager
**And** le polling s'arrête (refetchInterval supprimé sur terminal state)

**Given** le job est FAILED
**When** status retourne FAILED
**Then** message d'erreur français sans stack trace + bouton Retry (UX-DR7, UX-DR14)

**UX-DRs :** UX-DR7, UX-DR13, UX-DR14 | **Décisions :** D-08

---

### Story 3.3 : Génération de données conformes aux contraintes des typologies

En tant que développeur,
je veux que les données générées respectent strictement les contraintes de chaque typologie assignée,
afin de faire confiance à la validité des données pour mes tests.

**Critères d'acceptation :**

**Given** une colonne est assignée à la typologie "IBAN_FR"
**When** le job de génération s'exécute
**Then** toutes les valeurs IBAN générées passent la validation Modulo-97 (FR24)
**And** le seed de colonne est calculé : `Long.hashCode(datasetSeed) ^ columnIndex` ; PRNG = `new java.util.Random(columnSeed)` — aucun autre générateur autorisé (S-01)

**Given** une colonne a Nullable=true avec probabilité 0.1
**When** la génération s'exécute
**Then** environ 10% des valeurs sont null (tolérance ±5% pour ≥ 10 000 lignes)

**Given** une colonne a Unique=true
**When** la génération s'exécute
**Then** toutes les valeurs générées sont distinctes
**And** si la cardinalité de la typologie < nombre de lignes, le job échoue avec message français explicite

**Given** une colonne a une liste pondérée (Story 2.3)
**When** la génération s'exécute
**Then** la distribution des valeurs approxime les poids configurés (tolérance ±5% pour ≥ 10 000 lignes)

**NFR :** NFR-P02 (≥ 1M lignes/min), NFR-R03 | **Architecture :** S-01 (PRNG canonique)

---

### Story 3.4 : Reproductibilité par seed et regénération

En tant que développeur,
je veux regénérer un dataset avec le même ou un nouveau seed,
afin de reproduire des scénarios de test exacts ou d'obtenir des données fraîches.

**Critères d'acceptation :**

**Given** un dataset a été généré avec seed {s} et typologies en version sha256 {h}
**When** je relance la génération avec le même seed et les mêmes versions de typologies
**Then** les données générées sont binaires-identiques au run précédent (FR23, NFR-R03)
**And** la dérivation du seed de colonne est : `Long.hashCode(datasetSeed) ^ columnIndex` (S-01)

**Given** je génère avec un seed différent (FR25)
**When** POST `.../generate` avec `{seed: newSeed}`
**Then** nouvelles données générées, `generated_rows` précédentes remplacées
**And** le nouveau seed s'affiche dans le SeedBadge (UX-DR3)

**Given** je change uniquement le nombre de lignes (FR25)
**When** POST `.../generate` avec `{rowCount: newCount, seed: sameSeed}`
**Then** les N premières lignes (N = min(newCount, prevCount)) sont identiques au run précédent
**And** les lignes supplémentaires continuent la séquence PRNG de façon déterministe

**Given** une nouvelle version de la typologie est publiée après la génération initiale
**When** le dataset est regénéré avec le même seed
**Then** avertissement : "La version de la typologie [X] a changé. Les données ne seront pas identiques au run précédent."

**Architecture :** D-02 (versioning sha256), S-01, NFR-R03

---

## Epic 4 — Test Lab & Gestion des typologies

**Objectif :** Un utilisateur peut tester une typologie à la volée dans le Test Lab, créer de nouvelles définitions via l'assistant guidé et les publier pour toute l'équipe.
**FRs :** FR29, FR30, FR31, FR32, FR33
**NFRs clés :** NFR-SC04 (≤ 500 typologies), NFR-R03 (reproductibilité)
**Architecture :** A3 (YamlTypologyParser validation JSON Schema)

---

### Story 4.1 : Test Lab — génération d'exemples à la demande

En tant que développeur,
je veux générer des exemples de valeurs pour n'importe quelle typologie à la demande,
afin d'évaluer si elle convient à ma colonne avant de l'assigner.

**Critères d'acceptation :**

**Given** je suis dans le drawer de sélection de typologies (Story 2.4) et je sélectionne une typologie
**When** je clique "Générer des exemples" dans le panneau Test Lab (FR29)
**Then** POST `.../typologies/{tid}/test-generate` retourne 5 exemples en < 2s
**And** les exemples s'affichent inline dans le drawer, sans ouvrir une nouvelle vue

**Given** je modifie le seed dans le Test Lab (champ optionnel)
**When** je clique "Actualiser"
**Then** de nouveaux exemples sont générés avec le nouveau seed — résultats déterministes pour le même seed

**Given** la génération d'exemples échoue ou time-out
**When** la requête retourne une erreur
**Then** message d'erreur en français inline + bouton Retry (UX-DR14)

---

### Story 4.2 : Assistant de création de typologies

En tant que développeur,
je veux créer une nouvelle définition de typologie via un assistant guidé avec prévisualisation live,
afin que mon équipe puisse partager des patterns de données métier personnalisés.

**Critères d'acceptation :**

**Given** je clique "Nouvelle typologie" dans le catalogue
**When** l'assistant s'ouvre (FR30)
**Then** les champs disponibles sont : nom, catégorie, description, contraintes (formulaire JSON Schema via @rjsf/mui), et une zone de prévisualisation live de 5 exemples

**Given** je modifie la définition des contraintes
**When** la saisie est debounce 500ms
**Then** 5 exemples sont générés et affichés live (FR31)
**And** si la définition est invalide, un message de validation français s'affiche inline

**Given** je clique "Valider et enregistrer"
**When** POST `.../typologies` est appelé
**Then** le serveur valide le JSON Schema via `YamlTypologyParser` (A3)
**And** génère 5 exemples côté serveur comme vérification finale (FR31)
**And** si valide : 201 Created, status = DRAFT (en attente de publication admin)
**And** si invalide : 400 Bad Request avec message français détaillant l'erreur de validation

---

### Story 4.3 : Export YAML et publication d'une typologie

En tant qu'administrateur,
je veux exporter une définition de typologie en YAML et publier les typologies DRAFT pour les rendre disponibles à toute l'équipe,
afin de maintenir un catalogue partagé et versionné.

**Critères d'acceptation :**

**Given** je suis sur la fiche d'une typologie
**When** je clique "Exporter YAML" (FR32)
**Then** GET `.../typologies/{tid}/export?format=yaml` retourne le fichier (Content-Type: application/x-yaml)
**And** le fichier est syntaxiquement valide selon le format des 112 typologies existantes (contrainte brownfield)

**Given** je suis administrateur et une typologie a status DRAFT
**When** je clique "Publier" (FR33)
**Then** PATCH `.../typologies/{tid}/publish` passe status à PUBLISHED
**And** la typologie devient visible dans le catalogue pour tous les membres
**And** Snackbar : "Typologie [name] publiée." (5s auto-dismiss) (UX-DR15)

**Given** je ne suis pas administrateur
**When** je consulte une fiche de typologie DRAFT
**Then** le bouton "Publier" est absent — aucun appel API possible

---

## Epic 5 — Import DDL & Génération multi-tables

**Objectif :** Un utilisateur peut importer un schéma SQL complet, visualiser le graphe de dépendances FK, détecter les cycles et générer des données cohérentes pour toutes les tables dans le bon ordre.
**FRs :** FR34, FR35, FR36, FR37, FR38, FR39, FR40
**NFRs clés :** NFR-P06 (analyse DDL ≤ 50 tables en < 3s)
**Architecture :** A1 (CycleDetectedException), D-05 (RBAC sur DDL endpoints — S-03), @monaco-editor/react, @xyflow/react

---

### Story 5.1 : Import d'un schéma SQL DDL

En tant que développeur,
je veux importer un schéma SQL DDL en uploadant un fichier ou en le saisissant dans un éditeur,
afin que la plateforme puisse analyser mes tables et leurs dépendances.

**Critères d'acceptation :**

**Given** je suis en mode DDL d'un sous-projet
**When** j'uploade un fichier .sql via POST `/api/workspaces/{wid}/.../ddl-schemas` (multipart) OU je colle du SQL dans l'éditeur Monaco et clique "Analyser" (FR34)
**Then** le SQL est reçu, parsé par JSQLParser, et un DDL schema resource est créé — 201 Created avec schema ID
**And** l'éditeur Monaco affiche le SQL avec coloration syntaxique (@monaco-editor/react)

**Given** le SQL contient des erreurs de syntaxe
**When** JSQLParser échoue
**Then** 400 Bad Request avec la position de l'erreur et un message en français

**Given** le fichier DDL dépasse 1MB
**When** upload
**Then** 413 Payload Too Large avec message français

**Given** l'endpoint DDL est appelé
**When** SpaceSecurityFilter s'exécute
**Then** le chemin `/api/workspaces/{wid}/projects/{pid}/subprojects/{spid}/ddl/{did}/**` est sous le filtre RBAC (S-03 fix, D-05)

---

### Story 5.2 : Analyse DDL, graphe de dépendances FK et détection de cycles

En tant que développeur,
je veux que le système construise automatiquement le graphe de dépendances FK et me signale les cycles,
afin de savoir si mon schéma est valide pour la génération ordonnée.

**Critères d'acceptation :**

**Given** un schéma DDL a été importé
**When** POST `.../ddl-schemas/{sid}/analyze` est appelé
**Then** le serveur construit le graphe FK (FR35) et exécute le `TopologicalSorter`
**And** analyse complète en < 3s pour ≤ 50 tables (NFR-P06)

**Given** aucun cycle détecté
**When** l'analyse se termine
**Then** 200 OK avec : liste des tables, arêtes FK, ordre topologique de génération

**Given** un cycle est détecté
**When** `TopologicalSorter` lève `CycleDetectedException` (A1, FR37)
**Then** 422 Unprocessable Entity : "Cycle de dépendances détecté entre les tables : [TABLE_A → TABLE_B → TABLE_A]. Corrigez le schéma avant de générer."
**And** les tables circulaires sont identifiées dans la réponse JSON

---

### Story 5.3 : Visualisation interactive du graphe FK

En tant que développeur,
je veux voir un diagramme interactif des dépendances FK entre tables,
afin de comprendre visuellement la structure et configurer la génération en connaissance de cause.

**Critères d'acceptation :**

**Given** un schéma DDL a été analysé sans cycles
**When** j'ouvre la vue Graphe FK (FR36)
**Then** C-05 `FKGraphCanvas` (React Flow) rend : tables parentes (bg #2e7d32), tables enfants (bg #1565c0) (UX-DR6)
**And** les flèches FK affichent un tooltip au survol : "TABLE_A.col → TABLE_B.col"
**And** les nœuds sont repositionnables par drag (UX-DR6)

**Given** je clique sur un nœud de table
**When** le clic est traité
**Then** les colonnes de cette table sont mises en évidence dans le panneau de configuration en dessous (FR36)

**Given** je navigue au clavier dans le graphe
**When** je Tab entre les nœuds
**Then** chaque nœud a `role="button"` et `aria-label="Table {name} — {n} colonnes"` (UX-DR13)
**And** chaque flèche FK a `aria-label="Clé étrangère : {TABLE_A.col} → {TABLE_B.col}"`

**UX-DRs :** UX-DR6, UX-DR13

---

### Story 5.4 : Configuration des colonnes DDL et génération multi-tables

En tant que développeur,
je veux configurer les typologies des colonnes DDL et déclencher la génération pour toutes les tables dans l'ordre FK,
afin d'obtenir un dataset relationnel cohérent et sans violation de contraintes.

**Critères d'acceptation :**

**Given** le graphe FK est analysé et les colonnes sont affichées
**When** j'assigne des typologies aux colonnes (FR40)
**Then** l'interface est identique à Epic 2 (ColumnConfigRow, TypologyChip, Drawer 400px)
**And** les colonnes FK (références vers une table parente) sont en état verrouillé : bg #e3f2fd, ⚙ désactivé, tooltip "Valeur déterminée par la clé étrangère" (UX-DR5)

**Given** je configure le nombre de lignes par table (FR39)
**When** je saisis les valeurs par table dans le panneau de config
**Then** chaque count est stocké indépendamment dans la config du schéma DDL

**Given** je clique "Générer toutes les tables" (FR38)
**When** POST `.../ddl-schemas/{sid}/generate`
**Then** la génération s'exécute dans l'ordre topologique (tables parentes en premier)
**And** les colonnes FK reçoivent des valeurs issues des lignes déjà générées dans les tables parentes (intégrité référentielle garantie)
**And** la `GenerationProgressCard` (C-06) affiche la progression par table (UX-DR7)
**And** le PRNG canonique s'applique : `Long.hashCode(datasetSeed) ^ columnIndex` par table (S-01)

---

## Epic 6 — Intégration API REST

**Objectif :** Un système CI/CD ou une application externe peut piloter GenData entièrement via API — déclencher la génération, extraire des lignes ciblées et gérer les configurations de datasets.
**FRs :** FR41, FR42, FR43, FR44, FR45
**NFRs clés :** NFR-I01 (versioning API), NFR-I02 (JSON UTF-8), NFR-I03 (sémantique HTTP), NFR-P04 (premier byte < 3s)
**Architecture :** D-05 (RBAC S-04 fix export), D-07 (API-Version), D-09 (streaming), D-10 (format erreur)

---

### Story 6.1 : Authentification API externe et versioning

En tant que système externe,
je veux m'authentifier avec des credentials LDAP et obtenir un JWT,
afin d'appeler tous les endpoints GenData depuis mon pipeline CI/CD.

**Critères d'acceptation :**

**Given** un système externe appelle POST `/api/auth/login` avec des credentials LDAP et `API-Version: 1`
**When** les credentials sont valides
**Then** un JWT 8h est retourné (même flow que Story 1.1)
**And** toutes les requêtes suivantes peuvent utiliser `Authorization: Bearer {token}`

**Given** le header `API-Version` est absent
**When** n'importe quel endpoint est appelé
**Then** 400 BAD_REQUEST, code `MISSING_API_VERSION` (D-07, S-05)
**And** le comportement est documenté : "Header absent → 400, jamais de défaut silencieux"

**Given** le JWT expire pendant une session CI longue
**When** un appel API retourne 401 EXPIRED_TOKEN
**Then** le système externe doit appeler à nouveau `/api/auth/login` pour obtenir un nouveau token

**NFRs :** NFR-I01, NFR-I02, NFR-I03

---

### Story 6.2 : CRUD de configurations de datasets via API

En tant que système externe,
je veux créer, lire, modifier et supprimer des configurations de datasets via REST,
afin de provisionner des données de test automatiquement dans mon pipeline.

**Critères d'acceptation :**

**Given** un appel authentifié avec JWT valide et API-Version: 1
**When** POST `/api/workspaces/{wid}/.../datasets` avec config JSON
**Then** un dataset est créé — 201 Created (mêmes règles que Stories 2.1 + 2.5) (FR44)
**And** le RBAC est appliqué : l'appelant doit être membre du workspace (D-05)

**Given** GET `/api/workspaces/{wid}/.../datasets/{did}`
**Then** la configuration complète du dataset est retournée — 200 OK

**Given** PATCH `/api/workspaces/{wid}/.../datasets/{did}` avec delta de config
**Then** `current_config` est mis à jour — 200 OK (mêmes règles que Story 2.3)

**Given** DELETE `/api/workspaces/{wid}/.../datasets/{did}`
**Then** dataset supprimé — 204 No Content

**Given** l'appelant n'est pas membre du workspace
**When** toute opération CRUD
**Then** 403 Forbidden (D-05)

**NFRs :** NFR-I02, NFR-I03

---

### Story 6.3 : Déclenchement de génération et polling de statut via API

En tant que système externe,
je veux déclencher la génération et poller le statut jusqu'à COMPLETED,
afin que mon pipeline attende les données avant de lancer les tests.

**Critères d'acceptation :**

**Given** POST `/api/workspaces/{wid}/.../datasets/{did}/generate` avec `API-Version: 1`
**When** appelé par un système externe authentifié
**Then** 202 Accepted avec `{jobId, status: "PENDING"}` (FR41)
**And** la génération s'exécute de façon asynchrone (même moteur que Story 3.1)

**Given** GET `/api/workspaces/{wid}/.../datasets/{did}/status`
**When** le système externe poll
**Then** `{status: PENDING|RUNNING|COMPLETED|FAILED, progress: N, rowsGenerated: M}` — 200 OK
**And** le système externe contrôle son propre intervalle de polling (pas de Server-Sent Events en MVP)

**Given** status est COMPLETED
**When** le système externe poll une fois de plus
**Then** `{status: COMPLETED, rowsGenerated: {total}}` — données prêtes pour extraction

**NFRs :** NFR-P03, NFR-I02, NFR-I03

---

### Story 6.4 : Extraction de lignes et téléchargement de dataset via API

En tant que système externe,
je veux extraire des lignes spécifiques ou télécharger le dataset complet,
afin d'injecter des données de test ciblées dans mon environnement.

**Critères d'acceptation :**

**Given** GET `/api/workspaces/{wid}/.../datasets/{did}/rows/{uuid}`
**When** appelé avec un UUID de ligne valide
**Then** la ligne est retournée en JSON — 200 OK (FR42)
**And** la réponse inclut `"_gendata_marker": "TEST_DATA_ONLY"` (NFR-S05)

**Given** GET `.../rows?columnName={col}&value={val}`
**When** appelé avec des paramètres de filtre
**Then** toutes les lignes correspondant à colonne=valeur sont retournées — 200 OK (FR43)
**And** résultats incluent `"_gendata_marker": "TEST_DATA_ONLY"`

**Given** GET `.../datasets/{did}/export?format=json|csv` (FR45)
**Then** le dataset stream en `Transfer-Encoding: chunked` (D-09)
**And** premier byte en < 3s (NFR-P04)
**And** toutes les lignes incluent `"_gendata_marker": "TEST_DATA_ONLY"` (NFR-S05)

**Given** l'appelant n'est pas membre du workspace
**When** tout endpoint d'extraction
**Then** 403 Forbidden (D-05 — S-04 fix : export nested sous `/api/workspaces/{wid}/**`)

---

## Epic 7 — Administration, suivi d'activité & conformité

**Objectif :** Un administrateur dispose d'une visibilité complète sur l'usage de la plateforme, peut piloter la rétention des données et s'assurer de la conformité des traces d'audit.
**FRs :** FR06, FR48, FR50, FR51, FR52, FR53
**NFRs clés :** NFR-S04 (audit trail 100%, rétention ≥ 1 an), NFR-R01 (disponibilité 99,5%)
**Architecture :** D-06 (@Auditable AOP), S-06 (audit scope — génération ET export)

---

### Story 7.1 : Audit trail — enregistrement automatique des opérations

En tant que responsable conformité,
je veux que chaque opération de génération et d'export soit automatiquement journalisée dans un audit trail immuable,
afin de démontrer la conformité réglementaire.

**Critères d'acceptation :**

**Given** toute opération de génération ou d'export est déclenchée (UI ou API, par tout utilisateur)
**When** l'opération se termine (succès ou échec)
**Then** `AuditTrailAspect @AfterReturning` enregistre : userId, timestamp UTC, type d'opération, dataset ID, nombre de lignes (FR48, D-06)
**And** P-09 règle 4 couvre GÉNÉRATION ET EXPORT : "Toute génération ou export de données déclenche `@Auditable`" (S-06 fix)

**Given** un audit entry est créé
**When** n'importe quel utilisateur tente de le supprimer
**Then** 405 Method Not Allowed — les entrées d'audit sont immuables (NFR-S04)

**Given** les logs sont conservés
**When** une entrée dépasse la période de rétention configurée
**Then** l'entrée N'est PAS supprimée — la rétention s'applique uniquement à `generated_rows`, pas à `audit_trail` (NFR-S04 : minimum 1 an)

**Tables créées :** `audit_trail` (id, user_id, operation, dataset_id, row_count, timestamp_utc)

---

### Story 7.2 : Historique d'activité d'un dataset

En tant que développeur,
je veux consulter l'historique des actions sur un dataset (qui a fait quoi et quand),
afin de tracer les changements et partager le contexte avec mon équipe.

**Critères d'acceptation :**

**Given** un dataset existe et a eu des activités
**When** j'ouvre l'onglet "Historique" du dataset
**Then** GET `.../datasets/{did}/activity` retourne les événements : consulté, modifié, téléchargé, généré (FR51, FR52)
**And** chaque événement affiche : nom d'affichage de l'utilisateur, action, timestamp, métadonnées optionnelles (ex. nombre de lignes pour les téléchargements)

**Given** je filtre l'activité par plage de dates
**When** j'applique le filtre
**Then** seuls les événements dans la plage sont retournés

**Given** le dataset n'a aucune activité
**When** l'onglet charge
**Then** skeleton MUI pendant le chargement, puis état vide : "Aucune activité enregistrée pour ce dataset." (UX-DR14)

---

### Story 7.3 : Dashboard d'activité et statistiques d'espace

En tant qu'administrateur,
je veux un dashboard affichant les membres actifs, les datasets et les volumes générés,
afin de piloter l'usage de la plateforme et justifier les capacités.

**Critères d'acceptation :**

**Given** je suis administrateur de l'espace {wid}
**When** j'ouvre le dashboard de l'espace
**Then** GET `/api/workspaces/{wid}/stats` retourne : count de membres actifs, total datasets, total lignes générées (30 derniers jours), top 5 datasets les plus actifs (FR06, FR53)

**Given** le dashboard charge
**When** les données sont récupérées
**Then** skeleton loaders s'affichent pour chaque métrique (UX-DR14)
**And** un graphe recharts affiche l'évolution hebdomadaire du volume de génération

**Given** je ne suis pas administrateur
**When** je navigue vers l'URL du dashboard
**Then** 403 Forbidden, message UI approprié en français

---

### Story 7.4 : Configuration de la rétention et purge des données

En tant qu'administrateur,
je veux configurer la politique de rétention des données et déclencher des purges,
afin de respecter nos exigences de minimisation des données.

**Critères d'acceptation :**

**Given** je suis administrateur
**When** j'ouvre Paramètres Espace > Rétention
**Then** je peux configurer `retention_days` (défaut : 90) pour les `generated_rows` du workspace

**Given** je clique "Lancer la purge maintenant"
**When** je confirme dans la Dialog (action destructive, UX-DR12)
**Then** POST `/api/workspaces/{wid}/admin/purge` déclenche le job de purge
**And** les `generated_rows` plus anciennes que `retention_days` sont supprimées via DROP PARTITION (D-01)
**And** Snackbar : "{N} lignes supprimées." (5s auto-dismiss) (UX-DR15) (FR50)

**Given** la purge automatique nocturne s'exécute
**When** le job de purge tourne
**Then** SEULES les `generated_rows` sont purgées — `audit_trail` n'est JAMAIS supprimé (NFR-S04)
**And** les entrées de la table `datasets` (métadonnées) sont préservées — seules les données générées sont purgées
