---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-06-innovation', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
inputDocuments:
  - prjdocs/SYNTHESE_PROJET.md
  - prjdocs/FUNCTIONAL_SPEC.md
  - prjdocs/ux-design-specification.md
  - prjdocs/brainstorming/brainstorming-features-gendata-2026-03-25.md
  - prjdocs/typologies/README.md
  - docs/besoin.txt
workflowType: 'prd'
classification:
  projectType: 'Web App interne + API REST'
  domain: 'Fintech / Banking - Internal Developer Tool'
  complexity: 'high'
  projectContext: 'brownfield'
---

# Product Requirements Document - GenData

**Auteur :** Nouredine  
**Date :** 28 mars 2026

---

## Résumé Exécutif

**GenData** est une plateforme web interne de génération de données de test destinée aux 1 000 développeurs et équipes QA. Elle résout deux problèmes structurels qui dégradent la qualité des tests : l'absence d'uniformité des jeux de données entre équipes, et l'impossibilité de générer des volumes massifs de données réalistes sans infrastructure ad hoc.

L'outil permet à n'importe quelle équipe de créer, configurer et rejouer des jeux de données structurés à partir d'un fichier CSV existant, en assignant à chaque colonne une **typologie** — règle de génération sémantique (IBAN, UUID, montant, date de naissance, email…). Un **seed reproductible** garantit qu'un jeu de données peut être regénéré identiquement par une autre équipe, sur un autre environnement, à tout moment. La génération est conçue pour tenir à l'échelle : objectif 1 million de lignes/minute via Spring Batch et traitement asynchrone.

L'organisation du travail reflète la réalité des équipes : une hiérarchie **Espace (libellé libre) → Projet → Sous-projet** permet à chaque groupe de travailler dans son propre contexte sans pollution globale. Le mode **DDL** étend la plateforme à la génération de données relationnelles cohérentes à partir d'un schéma SQL, en respectant l'ordre des dépendances de clés étrangères.

**Utilisateurs cibles :** Développeurs, ingénieurs QA, analystes — profils techniques internes.  
**Périmètre MVP :** 112 typologies, hiérarchie configurable, mode DDL, APIs CRUD + extraction, suivi d'activité des datasets.  
**Horizon :** Architecture générique conçue pour une évolution SaaS multi-domaine — les typologies sont le vecteur d'extension métier.

---

### Ce qui rend ce produit spécial

L'outil est un **moteur générique** ; les typologies sont les **plugins métier**. Cette séparation architecture/contenu est le différenciateur fondamental : aujourd'hui orienté banking (IBAN Modulo-97, SWIFT, SEPA, montants conformes), demain extensible à d'autres domaines (healthcare, retail, télécoms) par simple ajout de typologies — sans toucher au moteur.

Ce qui crée la barrière à l'entrée n'est pas la technologie de génération elle-même, mais **la richesse du catalogue de typologies métier** combinée à la **reproductibilité seed** : deux propriétés qu'aucun outil générique du marché (Mockaroo, Faker, DBT) ne fournit nativement dans un contexte bancaire.

La plateforme transforme un problème récurrent (données de test de mauvaise qualité = tests qui ne veulent rien dire) en avantage compétitif interne : les équipes qui l'adoptent produisent des tests plus fiables, plus reproductibles, sur des volumes représentatifs de la production.

---

### Classification du Projet

| Dimension | Valeur |
|---|---|
| Type | Application Web + API REST (React 18 / Spring Boot 3) |
| Domaine | Fintech / Banking — Internal Developer Tool |
| Complexité | Haute (conformité bancaire, 1 000 utilisateurs, génération massive) |
| Contexte | Brownfield — Spec fonctionnelle v1.1, UX design steps 1-6, 112 typologies YAML |

---

## Critères de Succès

### Succès Utilisateur

| Critère | Définition | Cible |
|---|---|---|
| **Temps jusqu'au premier dataset** | Délai entre l'upload du CSV et la génération du premier jeu de données | < 5 minutes |
| **Taux d'auto-détection accepté** | % de typologies auto-détectées que l'utilisateur valide sans modification | ≥ 80% |
| **Reproductibilité** | Un dataset regénéré avec le même seed produit des données octets-identiques | 100% |
| **Adoption interne** | % des 1 000 devs/QA ayant créé au moins un dataset dans les 3 premiers mois | ≥ 60% |
| **Partage inter-équipes** | Des datasets partagés via leur URL sont consultables sans friction | 0 friction |

### Succès Business

| Critère | Cible | Horizon |
|---|---|---|
| **Remplacement des solutions bricolées** | Aucune équipe ne génère manuellement des données de test | 6 mois post-launch |
| **Réduction du temps de setup de test** | Gain moyen de temps de préparation des données de test par sprint | ≥ 70% |
| **Extensibilité démontrée** | Ajout d'un nouveau domaine métier (ex. healthcare) par simple YAML sans code | ≤ 3 jours |
| **Base pour SaaS** | Architecture validée prête pour multi-tenant | Fin MVP |

### Succès Technique

| Critère | Cible |
|---|---|
| **Débit de génération** | ≥ 1 000 000 lignes/minute (Spring Batch asynchrone) |
| **Disponibilité** | ≥ 99,5% (outil interne) |
| **Conformité IBAN** | 100% Modulo-97 valide sur les typologies banking |
| **Couverture typologies** | 112 typologies disponibles au launch |
| **Temps de réponse API** | < 200ms sur les endpoints CRUD pour jeux < 10 000 lignes |

### Résultats Mesurables

- Un développeur peut qualifier ses colonnes CSV et lancer une génération de 10 000 lignes en moins de 5 minutes
- Deux équipes différentes obtiennent des données identiques en partageant uniquement un seed + une config
- Le moteur génère 1M lignes en < 60 secondes sur l'infrastructure cible
- L'ajout d'une nouvelle typologie est faisable par un développeur junior via YAML en < 30 minutes

---

## Parcours Utilisateurs

### Parcours 1 — Yann, développeur backend : le chemin nominal

**Situation :** Yann prépare un sprint de tests pour une nouvelle API de virement SEPA. Il a un fichier `virement_exemple.csv` avec 1 ligne d'exemple. Les IBAN fictifs ne passent pas le checksum de leur suite de tests automatisés.

**Scène d'ouverture :** Lundi 9h, Yann ouvre GenData. Il navigue dans la sidebar : *Département Tech → Projet Paiements → Sous-projet SEPA*. L'espace existe déjà.

**Déroulé :** Il uploade son CSV. L'auto-détection analyse les en-têtes : `transaction_id` → UUID ✓ 99%, `creditor_iban` → IBAN ✓ 97%, `amount` → Montant ✓ 87%, `status` → Valeur libre ⚠️ 45%. Il corrige `status` avec une liste personnalisée (`PENDING`, `SETTLED`, `REJECTED`). Seed à `42`, 50 000 lignes.

**Climax :** Génération async. 12 secondes. Notif : *“50 000 lignes générées — seed: 42”*. Tous les IBAN passent le Modulo-97.

**Résolution :** Yann partage l'URL du dataset avec seed `42`. Sa collègue Fatima régénère les mêmes données depuis son poste. Identique.

*Capabilities révélées :* Import CSV, auto-détection avec confiance, liste de valeurs personnalisée, génération async, seed reproductible, export JSON, URL partageable.

---

### Parcours 2 — Fatima, ingénieure QA : cas limite et récupération

**Situation :** Fatima teste le module KYC. Son CSV a des en-têtes translittérés (à `tarikh_milad`) — l'auto-détection échoue.

**Déroulé :** Badge orange 12% de confiance. Elle ouvre le Typology Browser, tape « naissance », trouve `DateNaissance`, teste via Test Lab : 5 dates entre 1940 et 2005. Elle assigne manuellement. Configure une distribution pondérée pour `nationalite` (80% FR, 10% MA, 10% autres).

**Climax :** Génère 10 000 lignes. En prévisualisation : 3 dates de naissance dans le futur. Elle restaure la copie originale de sa config, corrige la plage, regénère.

**Résolution :** Dataset corrigé en 2 minutes. Elle ouvre un ticket sur le cas limite du générateur.

*Capabilities révélées :* Typology Browser + Test Lab, assignation manuelle, liste pondérée, prévisualisation, copie originale + restauration.

---

### Parcours 3 — Karim, responsable d'espace : administrateur

**Situation :** Karim gère 40 personnes. Un junior a créé 200 datasets orphelins dans un sous-projet inutile.

**Déroulé :** Il ouvre les paramètres de l'espace. Tableau d'activité : 200 datasets jamais téléchargés. Il archive le sous-projet, identifie les 12 datasets actifs (consultés > 3 fois).

**Climax :** Il renomme l'espace de « Tribu Tech » en « Département SI » — le libellé se propage dans toute l'UI.

**Résolution :** L'espace est propre. Il partage les URLs des datasets de référence pour les nouveaux arrivants.

*Capabilities révélées :* Paramètres d'espace, suivi d'activité, archivage, libellé N1 configurable.

---

### Parcours 4 — Pipeline CI/CD : consommateur API

**Situation :** L'équipe Core Banking a intégré GenData dans Jenkins. À chaque merge request, les tests tournent avec un dataset frais.

**Déroulé :**
- `GET /api/datasets/{id}/generate?seed=42&rows=1000` → 1 000 lignes JSON en 800ms
- `GET /api/datasets/{id}/rows?transaction_id=UUID-XXXX` → ligne exacte pour test de cas limite
- `POST /api/datasets/{id}/config` → mise à jour du schéma après modification de la table

**Résolution :** Chaque MR tourne avec des données cohérentes, reproductibles, conformes au schéma courant. Zéro maintenance manuelle des fixtures.

*Capabilities révélées :* API REST génération, extraction par critère (UUID, valeur colonne), API CRUD datasets.

---

### Parcours 5 — Omar, nouveau venu : onboarding

**Situation :** Omar rejoint l'équipe KYC. On lui dit « utilise GenData ». Il ne connaît pas l'outil.

**Déroulé :** Il ouvre l'URL. La sidebar affiche l'espace *Département KYC* avec les projets existants. Il voit 3 datasets créés par ses collègues, clique sur *Régénérer avec seed: 42*. Dataset identique en 10 secondes.

**Climax :** Curieux, il explore le Typology Browser, catégorie *Compliance / KYC*. Découvre `NIR` et `PassportNumber`. Crée son premier dataset custom en 8 minutes.

**Résolution :** Autonome le jour 1. Sans doc, sans ticket, sans attente.

*Capabilities révélées :* Navigation par espace/projet, régénération par seed, Typology Browser par catégorie, valeur sociale des datasets partagés.

---

### Synthèse des Capabilities Révélées

| Capability | Parcours |
|---|---|
| Import CSV + auto-détection avec confiance | P1, P2 |
| Assignation manuelle de typologie | P2 |
| Typology Browser + Test Lab | P2, P5 |
| Génération asynchrone + seed reproductible | P1, P4 |
| Export JSON/CSV + URL partageable | P1, P3, P5 |
| API REST (generate, extract, CRUD) | P4 |
| Suivi d'activité (consulté/modifié/téléchargé) | P3 |
| Restauration copie originale | P2 |
| Hiérarchie Espace [libellé libre] → Projet → S-P | P3, P5 |
| Paramètres et gestion de l'espace | P3 |
| Liste de valeurs personnalisée / pondérée | P1, P2 |

---

## Exigences Spécifiques au Domaine

### Conformité & Réglementaire

- **RGPD** — GenData génère de fausses données, non des données personnelles réelles. Cette distinction doit être explicite dans l'UI (« données fictives — aucune donnée personnelle réelle »). Exigence de traçabilité sur qui génère quoi (audit trail).
- **Isolation environnements** — Les données générées ne doivent jamais être introduites en production. Les outputs sont marqués « TEST DATA ONLY ».
- **PCI-DSS awareness** — Les numéros de carte générés (`CardNumber`) sont syntaxiquement valides (Luhn) mais utilisent un préfixe BIN inexistant (`9999`) pour éviter tout usage frauduleux.
- **Conformité SEPA / SWIFT** — IBAN Modulo-97 valide par construction, BIC/SWIFT 8 ou 11 caractères, cohérence pays IBAN ↔ BIC.
- **Audit trail** — Toute génération loguée (utilisateur, horodatage, dataset, volume) pour conformité interne.

### Contraintes Techniques

- **Aucune donnée réelle en entrée** — L'UI affiche un avertissement explicite à l'upload de CSV. Mode anonymisation prévu en Phase 2.
- **Authentification** — SSO / LDAP interne (Active Directory bancaire). Pas de gestion de mots de passe locaux en MVP.
- **Isolation des espaces** — Un utilisateur ne peut accéder qu'aux espaces auxquels il appartient. Aucune visiblité cross-espace.
- **Rétention** — Durée de conservation des datasets configurable par admin. Purge automatique après expiration.
- **Performance à l'échelle** — 1 000 utilisateurs concurrents, pics lors des sprints. File d'attente prioritaire configurable. Objectif : 1M lignes/minute via Spring Batch chunking.

### Patterns Domaine Banking

- **Cohérence inter-champs** — Montant positif, date opération ≤ date valeur, statut terminal incompatible avec date valeur future. Ces règles sont encodées dans les typologies avancées, pas dans le moteur générique.
- **Formats locaux** — IBAN par pays (FR, DE, ES, IT…), RIB français, formats téléphone E.164.
- **Volumes représentatifs** — En banking, un jeu utile pour les tests de performance commence à 100 000 lignes. Le moteur doit supporter des batches de 10M+ lignes.

### Risques et Mitigations

| Risque | Mitigation |
|---|---|
| Utilisation accidentelle d'un IBAN valide en prod | Watermark « TEST » dans les données, documentation, formation |
| Génération de PAN utilisable pour fraude | Luhn valide mais préfixe BIN `9999` (non émis) |
| Saturation de la file de génération | Queue prioritaire, limite de volume par utilisateur configurable |
| Accès cross-espace | RBAC par espace, isolation au niveau API |
| Mauvaise qualité des typologies banking | Test Lab intégré + suite de tests de validation automatiques par typologie |

---

## Innovation & Patterns Nouveaux

### Zones d'Innovation Détectées

**1. Architecture Typologies-as-Plugins (catalogue déclaratif YAML)**

Le différenciateur architectural fondamental est la séparation stricte entre le **moteur de génération** (générique, agnostique au domaine) et les **typologies** (règles de génération déclaratives en YAML). Un expert métier peut décrire une nouvelle règle de génération — avec ses contraintes, ses distributions, ses exemples — sans écrire une ligne de code.

Ce paradigme est inédit dans les outils de génération de données d'entreprise : les solutions existantes (Mockaroo, Faker, DBT) encodent leurs types en dur dans le code source, rendant l'extension dépendante d'un développeur. Un **assistant de création de typologie** (formulaire guidé + prévisualisation live) est inclus dans le MVP pour permettre aux non-développeurs d'enrichir le catalogue de manière autonome.

**Impact :** Le catalogue de typologies devient un actif métier autonome, maintenable par des experts domaine, versionnable en Git, et transférable entre projets ou organisations.

**2. Génération Relationnelle FK-Aware (Mode DDL)**

L'analyse automatique d'un schéma SQL DDL pour construire un graphe de dépendances de clés étrangères, puis générer les données dans l'ordre topologique correct, est une fonctionnalité absente des outils de génération grand public. L'utilisateur n'a pas à connaître l'ordre d'insertion — le moteur le déduit.

**Impact :** Génération de fixtures de base de données complètes et cohérentes en une seule opération, sans scripts d'insertion manuels ni connaissance de l'architecture DB.

### Contexte Marché & Positionnement

| Outil | Forces | Lacune comblée par GenData |
|---|---|---|
| Mockaroo | Simple, rapide | Pas de typologies métier, pas de FK-aware, pas de seed reproductible |
| Faker (lib) | Flexible en code | Nécessite un dev, pas d'UI, pas de partage |
| DBT Seed | Intégration data pipeline | Pas de génération, fichiers statiques uniquement |
| Synthea / Jailer | FK-aware | Domaine unique (healthcare / DB copy), pas extensible |

**Positionnement unique :** GenData est le premier outil combinant catalogue de typologies déclaratif extensible + génération FK-aware + reproductibilité seed + organisation en espaces partagés.

### Approche de Validation

- **Catalogue déclaratif YAML** — Un expert métier non-développeur crée une nouvelle typologie via l'assistant en < 30 minutes, sans assistance technique.
- **FK-aware** — Test avec un schéma bancaire réel (banque → compte → virement → mouvement) : cohérence référentielle 100% sans intervention manuelle.
- **Adoption** — 60% des devs/QA actifs à 3 mois (critère de succès déjà défini).

### Mitigation des Risques d'Innovation

| Risque | Mitigation |
|---|---|
| Format YAML trop complexe pour les non-devs | Assistant guidé (formulaire) + exemples inline dans le browser |
| Analyse DDL incomplète (dialectes SQL) | Support PostgreSQL en priorité (stack cible) ; dialectes additionnels en post-MVP |
| Mauvaise inférence de l'ordre FK | Détection des cycles, avertissement explicite, ordre manuel en fallback |

---

## Exigences Techniques — Application Web + API REST

### Vue d'ensemble

GenData est une Single Page Application (SPA) React consommant une API REST Spring Boot, avec un moteur de génération asynchrone découplé via Spring Batch. Trois couches distinctes : **UI** (navigation/configuration), **API** (orchestration/CRUD), **Engine** (génération batch).

### Architecture Frontend

| Aspect | Décision |
|---|---|
| Framework | React 18 + TypeScript 5 |
| UI Components | MUI v6 (DataGrid, TreeView, Dialog) |
| Éditeur SQL | Monaco Editor (mode DDL) |
| Graphe FK | React Flow |
| Formulaires typologies | react-jsonschema-form |
| État global | React Query (server state) + Zustand (UI state) |
| Routing | React Router v6 — deep-link par espace/projet/dataset |
| Build | Vite |

### Architecture Backend

| Aspect | Décision |
|---|---|
| Framework | Spring Boot 3.x, Java 17+ |
| API style | REST JSON — versioning par header `API-Version` |
| Génération | Spring Batch 5 — chunks de 10 000 lignes, JobRepository PostgreSQL |
| Base de données | PostgreSQL 15+ (production), H2 (dev local) |
| Auth | Spring Security + SSO/LDAP (Active Directory bancaire) — JWT en interne |
| Async | Jobs Spring Batch asynchrones — notification WebSocket ou polling |

### Spécification des Endpoints Clés

#### Espaces & Hiérarchie
```
GET    /api/workspaces                     — lister les espaces accessibles
POST   /api/workspaces                     — créer un espace
PUT    /api/workspaces/{id}/label-name     — renommer le libellé du N1
GET    /api/workspaces/{id}/projects       — lister les projets
POST   /api/workspaces/{id}/projects       — créer un projet
GET    /api/projects/{id}/subprojects      — lister les sous-projets
POST   /api/projects/{id}/subprojects      — créer un sous-projet
```

#### Datasets
```
POST   /api/subprojects/{id}/datasets      — créer une config dataset
GET    /api/datasets/{id}                  — lire la config
PUT    /api/datasets/{id}/config           — mettre à jour la config
POST   /api/datasets/{id}/generate         — lancer une génération (async)
GET    /api/datasets/{id}/status           — état du job de génération
GET    /api/datasets/{id}/rows             — lire les lignes (pagination + filtres)
GET    /api/datasets/{id}/rows?{col}={val} — extraction par valeur de colonne
GET    /api/datasets/{id}/rows/{uuid}      — extraction par UUID
GET    /api/datasets/{id}/export?format=json|csv — téléchargement
POST   /api/datasets/{id}/restore         — restaurer la copie originale
DELETE /api/datasets/{id}                  — archiver
```

#### Typologies
```
GET    /api/typologies                     — lister (filtres : catégorie, search)
GET    /api/typologies/{id}               — détail + exemples
POST   /api/typologies/{id}/sample        — générer N exemples (Test Lab)
POST   /api/typologies                    — créer une nouvelle typo (assistant)
PUT    /api/typologies/{id}               — mettre à jour
```

#### DDL
```
POST   /api/subprojects/{id}/ddl          — uploader un DDL SQL
GET    /api/ddl/{id}/graph                — graphe FK (noeuds + arêtes)
POST   /api/ddl/{id}/generate             — générer les données relationnelles
```

### Modèle d'Authentification & Permissions

- **Authentification** : SSO LDAP/AD — token JWT émis par Spring Security après validation AD
- **Isolation des espaces** : RBAC par espace — un utilisateur appartient à un ou plusieurs espaces, accès refusé aux autres
- **Rôles par espace** : `VIEWER` (lecture), `CONTRIBUTOR` (créer/modifier datasets), `ADMIN` (paramètres espace, archivage)
- **MVP** : rôles simplifiés — tout membre d'un espace est `CONTRIBUTOR`, le créateur est `ADMIN`

### Génération Asynchrone — Flux Technique

```
Client         →  POST /api/datasets/{id}/generate
API            →  crée un Job Spring Batch, retourne jobId + 202 Accepted
Spring Batch   →  lit la config, génère par chunks de 10k lignes → PostgreSQL
Client         →  poll GET /api/datasets/{id}/status  (ou WebSocket notification)
API            →  retourne PENDING | RUNNING (progress%) | COMPLETED | FAILED
Client         →  GET /api/datasets/{id}/export  quand COMPLETED
```

### Parsing DDL — Flux Technique

```
Client         →  POST DDL SQL (upload ou texte)
API            →  parsé via JSQLParser (PostgreSQL dialect)
Engine         →  construit le graphe FK (DAG)
Engine         →  détecte les cycles → avertissement
Engine         →  tri topologique → ordre de génération
Engine         →  génération table par table, FK résolues
Client         →  graphe FK visualisé (React Flow) + bouton Générer débloqué
```

### Considérations d'Implémentation

- **Export volumineux** : streaming HTTP (chunked transfer encoding) pour éviter les timeouts sur les gros volumes — pas de bufferisation complète en mémoire
- **Seed** : utilisation de `java.util.Random(seed)` avec séquence déterministe par colonne — la reproductibilité est garantie toutes versions confondues si le YAML de la typo ne change pas
- **Stockage des données générées** : table PostgreSQL `generated_rows` partitionnée par `dataset_id` — purge planifiée par scheduler
- **Limite par utilisateur** : max 5 jobs simultanés par utilisateur, configurable par admin

---

## Scoping & Développement par Phases

### Stratégie MVP & Philosophie

**Approche MVP :** MVP « Experience » — livrer une expérience utilisateur complète sur le périmètre défini plutôt qu'un périmètre réduit avec une UX dégradée. L'adoption interne de 1 000 utilisateurs exige un outil qui inspire confiance dès le premier usage.

**Ressources estimées :** 12-17 personnes, 3-4 mois, développement parallèle en 4-5 subteams (UI, API/Core, Engine/Batch, Typologies, DevOps).

### MVP — Fonctionnalités Essentielles (Phase 1)

**Parcours utilisateurs supportés :** Tous les 5 parcours définis (Yann, Fatima, Karim, CI/CD, Omar)

| Capacité | Justification MVP |
|---|---|
| Import CSV + auto-détection typologies | Sans ça, le produit n'existe pas |
| 112 typologies banking + génériques | Le catalogue complet est le différenciateur — moins = pas crédible |
| Hiérarchie Espace [libellé libre] → Projet → Sous-projet | Organisation requise dès le jour 1 pour 1 000 users |
| Génération async + seed reproductible | La reproductibilité est le cœur de valeur |
| Mode DDL (schéma SQL → données FK-aware) | Reclassé MVP — trop de cas d'usage sans ça |
| Export JSON / CSV | Minimum pour être utile |
| API REST (generate, extract, CRUD) | Intégration CI/CD requise dès le MVP |
| Typology Browser + Test Lab | Adoption et confiance — critique |
| Assistant Création de Typologie | Extensibilité par les non-devs — différenciateur stratégique |
| Suivi d'activité + restauration originale | Gestion opérationnelle minimum |
| Auth SSO/LDAP | Non négociable en contexte bancaire |
| RBAC simplifié par espace | Isolation requise (compliance) |
| Audit trail des générations | Conformité interne |

### Phase 2 — Croissance (Post-MVP, T+4 à T+8 mois)

- Corrélations inter-colonnes (code postal ↔ ville, IBAN ↔ BIC cohérents)
- Distributions statistiques avancées (gaussienne, Pareto, loi personnalisée)
- Export SQL INSERT, Parquet, Avro
- Webhooks + intégration CI/CD native (plugin Jenkins, GitHub Actions)
- Gestion des rôles granulaire par espace (VIEWER / CONTRIBUTOR / ADMIN)
- Historique de versions des configurations datasets
- Mode anonymisation CSV (données réelles → pseudonymisées)

### Phase 3 — Expansion (T+8 mois+)

- SaaS multi-tenant (isolation complète par organisation)
- Marketplace typologies communautaires
- GenAI-assisted : décrire un dataset en langage naturel → proposition de schéma
- Extension domaines : healthcare, retail, télécoms (nouvelles typologies YAML)
- Génération incrémentale (enrichir sans recréer)

### Stratégie de Mitigation des Risques

| Risque | Mitigation |
|---|---|
| Parsing DDL sur dialectes variés | Scope PostgreSQL uniquement en MVP ; fallback ordre manuel documenté |
| Performances 1M lignes/min sous charge | Benchmark Spring Batch dès sprint 1 ; ajustement chunk size |
| Faible adoption malgré outil complet | Onboarding guidé + datasets de référence par équipe au lancement |
| Délai MVP > 4 mois | Couper corrélations et distributions avancées si besoin |
| Données fictives utilisées en prod | Watermark systématique + sensibilisation au lancement |

---

## Exigences Fonctionnelles

### 1. Gestion des Espaces de Travail

- **FR01 :** Un administrateur peut créer un espace de travail avec un libellé de premier niveau personnalisable (Tribu, Département, Équipe, ou terme libre)
- **FR02 :** Un administrateur peut modifier le libellé du premier niveau d'un espace — le changement se propage dans toute l'interface
- **FR03 :** Un utilisateur peut créer un Projet dans un espace auquel il appartient
- **FR04 :** Un utilisateur peut créer un Sous-projet dans un Projet
- **FR05 :** Un utilisateur ne peut voir et accéder qu'aux espaces dont il est membre
- **FR06 :** Un administrateur peut consulter l'activité globale d'un espace (membres, datasets, volumes générés)
- **FR07 :** Un administrateur peut archiver un sous-projet et ses contenus
- **FR08 :** Un utilisateur peut naviguer dans la hiérarchie Espace → Projet → Sous-projet depuis une sidebar persistante

### 2. Gestion des Datasets

- **FR09 :** Un utilisateur peut créer un dataset dans un sous-projet en uploadant un fichier CSV (en-tête + au moins une ligne d'exemple)
- **FR10 :** Le système propose automatiquement une détection de typologie pour chaque colonne du CSV, avec un score de confiance
- **FR11 :** Un utilisateur peut accepter, modifier ou rejeter la détection automatique pour chaque colonne
- **FR12 :** Un utilisateur peut assigner une liste de valeurs personnalisée à une colonne
- **FR13 :** Un utilisateur peut assigner une liste de valeurs pondérée à une colonne (probabilité par valeur)
- **FR14 :** Un utilisateur peut configurer le nombre de lignes à générer et un seed de reproductibilité
- **FR15 :** Un utilisateur peut consulter les données d'un dataset en prévisualisation (50 lignes minimum)
- **FR16 :** Un utilisateur peut télécharger un dataset au format JSON ou CSV
- **FR17 :** Un utilisateur peut partager un dataset via une URL stable (deep-link)
- **FR18 :** Un utilisateur peut restaurer la configuration originale d'un dataset après modification
- **FR19 :** Le système conserve une copie immuable de la configuration initiale de chaque dataset
- **FR20 :** Un administrateur peut archiver ou supprimer un dataset

### 3. Génération de Données

- **FR21 :** Le système génère les données de manière asynchrone et notifie l'utilisateur à la fin du job
- **FR22 :** Un utilisateur peut consulter l'état d'avancement d'un job de génération en cours (progression en %)
- **FR23 :** Un dataset regénéré avec le même seed et la même configuration produit des données identiques
- **FR24 :** Le système génère des données conformes aux contraintes de la typologie assignée (ex. IBAN Modulo-97 valide)
- **FR25 :** Un utilisateur peut regénérer un dataset en modifiant uniquement le seed ou le nombre de lignes
- **FR26 :** Le système limite le nombre de jobs simultanés par utilisateur (configurable par admin)

### 4. Catalogue de Typologies & Test Lab

- **FR27 :** Un utilisateur peut parcourir le catalogue de typologies filtré par catégorie, nom ou tag
- **FR28 :** Un utilisateur peut consulter la définition, les contraintes et des exemples de valeurs pour chaque typologie
- **FR29 :** Un utilisateur peut générer des exemples de valeurs à la demande pour une typologie (Test Lab)
- **FR30 :** Un utilisateur peut créer une nouvelle définition de typologie via un assistant guidé (formulaire + prévisualisation live)
- **FR31 :** L'assistant de création de typologie valide la syntaxe de la définition et génère 5 exemples avant enregistrement
- **FR32 :** Un utilisateur peut exporter la définition d'une typologie au format YAML
- **FR33 :** Un administrateur peut publier une nouvelle typologie pour qu'elle soit disponible à tous les membres

### 5. Mode DDL — Génération Relationnelle

- **FR34 :** Un utilisateur peut importer un schéma SQL DDL (upload de fichier ou saisie directe dans un éditeur)
- **FR35 :** Le système analyse le schéma DDL et construit un graphe de dépendances entre tables (clés étrangères)
- **FR36 :** Un utilisateur peut visualiser le graphe de dépendances FK sous forme de diagramme interactif
- **FR37 :** Le système détecte les cycles de dépendances dans le schéma et avertit l'utilisateur
- **FR38 :** Un utilisateur peut générer des données pour toutes les tables du schéma dans l'ordre correct des dépendances FK
- **FR39 :** Un utilisateur peut configurer le nombre de lignes à générer par table
- **FR40 :** Un utilisateur peut assigner des typologies aux colonnes d'un schéma DDL avant génération

### 6. APIs & Intégration

- **FR41 :** Un système externe peut déclencher la génération d'un dataset via API REST (avec authentification)
- **FR42 :** Un système externe peut extraire une ligne spécifique d'un dataset par identifiant UUID via API
- **FR43 :** Un système externe peut extraire des lignes d'un dataset par valeur d'une ou plusieurs colonnes via API
- **FR44 :** Un système externe peut lire, créer, modifier et supprimer des configurations de datasets via API CRUD
- **FR45 :** Un système externe peut télécharger un dataset généré au format JSON ou CSV via API

### 7. Sécurité & Administration

- **FR46 :** Un utilisateur s'authentifie via SSO LDAP/Active Directory (pas de compte local)
- **FR47 :** Un administrateur peut ajouter ou retirer des membres d'un espace
- **FR48 :** Le système journalise chaque opération de génération (utilisateur, date, dataset, volume) dans un audit trail
- **FR49 :** Les données générées sont marquées comme « TEST DATA ONLY » dans les exports
- **FR50 :** Un administrateur peut configurer la durée de rétention des datasets générés et déclencher une purge

### 8. Suivi d'Activité

- **FR51 :** Le système enregistre les événements sur chaque dataset : consulté, modifié, téléchargé
- **FR52 :** Un utilisateur peut consulter l'historique d'activité d'un dataset (qui, quand, quelle action)
- **FR53 :** Un administrateur peut consulter les statistiques d'usage de l'espace (datasets actifs, volumes, membres actifs)

---

## Exigences Non-Fonctionnelles

### Performance

| NFR | Critère |
|---|---|
| **NFR-P01** Temps de réponse API CRUD | Les endpoints CRUD sur datasets répondent en < 200ms pour des jeux de ≤ 10 000 lignes — P95 |
| **NFR-P02** Débit de génération | Le moteur génère ≥ 1 000 000 lignes/minute en régime nominal (Spring Batch, chunks 10k, PostgreSQL) |
| **NFR-P03** Latence de démarrage de job | Un job de génération passe en RUNNING dans les 5 secondes suivant la requête |
| **NFR-P04** Export streaming | Le début d'un téléchargement (premier byte) démarre en < 3 secondes quel que soit le volume — streaming HTTP chunked |
| **NFR-P05** Chargement SPA | L'application React est interactive (TTI) en < 2 secondes sur connexion interne standard |
| **NFR-P06** Parsing DDL | L'analyse d'un schéma DDL de ≤ 50 tables et la construction du graphe FK s'effectuent en < 3 secondes |

### Sécurité

| NFR | Critère |
|---|---|
| **NFR-S01** Authentification | Toute session nécessite une authentification SSO LDAP/AD validée — aucun endpoint fonctionnel accessible sans token JWT valide |
| **NFR-S02** Isolation des espaces | Un utilisateur ne peut accéder à aucune ressource d'un espace dont il n'est pas membre — validé à chaque requête |
| **NFR-S03** Transport | Toutes les communications client ↔ serveur s'effectuent en HTTPS/TLS 1.2+ — HTTP non chiffré refusé |
| **NFR-S04** Audit trail | 100% des opérations de génération sont loguées : utilisateur, horodatage UTC, dataset ID, volume — rétention minimum 1 an |
| **NFR-S05** Marquage TEST DATA | 100% des exports (JSON, CSV) incluent un champ `_gendata_marker: TEST_DATA_ONLY` |
| **NFR-S06** Données générées | Aucune donnée réelle ne transite dans le moteur — les CSV uploadés ne servent qu'à l'analyse des en-têtes et la première ligne d'exemple |
| **NFR-S07** Expiration des tokens | Les tokens JWT expirent après 8h maximum — aucune session permanente |

### Scalabilité

| NFR | Critère |
|---|---|
| **NFR-SC01** Utilisateurs simultanés | 1 000 utilisateurs connectés simultanément sans dégradation (< 20% d'augmentation du P95) |
| **NFR-SC02** Jobs concurrents | ≥ 200 jobs de génération simultanés sans perte ni corruption |
| **NFR-SC03** Volume de données | Un dataset peut contenir jusqu'à 10 000 000 de lignes sans erreur de génération ni d'export |
| **NFR-SC04** Catalogue typologies | Jusqu'à 500 typologies sans dégradation du temps de chargement du browser |
| **NFR-SC05** Extensibilité future | L'architecture supporte l'ajout d'un deuxième tenant sans refactoring majeur du core |

### Fiabilité

| NFR | Critère |
|---|---|
| **NFR-R01** Disponibilité | ≥ 99,5% en heures ouvrables (8h–20h, lundi–vendredi) — tolérance 3,6h/mois |
| **NFR-R02** Cohérence des jobs | Un job interrompu ne produit pas de données partiellement écrites — reprise propre ou annulation |
| **NFR-R03** Reproductibilité | Régénération avec le même seed + même YAML de typo = données binaires identiques, garanti inter-versions |
| **NFR-R04** Sauvegarde config | La configuration initiale de chaque dataset est immuable et non écrasable par des opérations utilisateur normales |

### Intégration

| NFR | Critère |
|---|---|
| **NFR-I01** Versioning API | L'API REST est versionnée via header `API-Version` — version N disponible ≥ 6 mois après sortie de N+1 |
| **NFR-I02** Formats d'échange | Tous les endpoints acceptent et retournent du JSON UTF-8 (`Content-Type: application/json`) |
| **NFR-I03** Codes HTTP | Sémantique HTTP respectée : 200/201/202 succès, 400 erreur client, 401/403 auth, 404 absent, 500 erreur serveur |
| **NFR-I04** SSO LDAP | Intégration LDAP v3 et LDAPS (port 636) — configurable sans recompilation |

