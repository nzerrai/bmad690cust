# Rubric Review — GenData Architecture

## Verdict

Architecture solide et exhaustive, couvrant les 53 FRs et les décisions critiques, mais avec trois lacunes structurelles bloquantes : la frontière `generation-engine` laisse les builders diverger sur la propriété des données partagées, l'observabilité est silencieuse, et le format D-01/P-01 ne porte pas les champs Binds/Prevents/Rule exigés par la spine.

---

## Critical findings

### C-01 — Frontière generation-engine non définie (divergence réelle)

La table des bounded contexts (ligne 85) liste `generation-engine` comme contexte distinct de `dataset-api`, mais la structure physique les place dans le même Spring Boot (`com.gendata.generation`). Trois questions sans réponse qui créent une divergence directe entre builders :

- **Qui déclenche le job ?** Est-ce un appel direct `JobLauncher.run()` depuis `DatasetService` ? Un événement interne ? Un endpoint séparé ? D-08 montre `GET /api/datasets/{id}/status` — ce endpoint vit dans `dataset/` ou `generation/` ?
- **Qui lit `generated_rows` ?** `ExportService` (dans `export/`) doit lire la table `generated_rows` (propriété de `generation/`). L'accès cross-package n'est pas défini : appel de service ? Query directe ? Repository partagé ?
- **Où vit le quota "5 jobs/user" (NFR-SC02) ?** Mentionné dans les cross-cutting concerns mais aucun mécanisme d'enforcement défini — `GenerationService` ? Un `JobExecutionListener` Spring Batch ? Un filtre ?

Conséquence : deux builders implémentant `dataset-api` et `generation-engine` indépendamment produiront des interfaces incompatibles.

### C-02 — Observabilité entièrement silencieuse

Outil bancaire avec NFR-P02 (1M lignes/min) et NFR-SC01 (1 000 utilisateurs simultanés) — l'observabilité est une dimension structurelle, pas opérationnelle. Le document cite Spring Actuator mais ne définit rien :

- Format de logs (JSON structuré ? Logback text ?) et niveau par défaut
- Corrélation IDs sur les requêtes HTTP → jobs Batch → lignes audit
- Qui consomme `/health` et `/metrics` (sonde k8s ? Prometheus ?)
- Pas de stack de log shipping mentionnée (ELK, Loki, etc.)

Un builder implémentant la couche logging produira un format incompatible avec un autre qui fait de même.

### C-03 — Format non conforme spine (AD-n avec Binds/Prevents/Rule)

Le document utilise D-01/P-01 sans les champs structurés `Binds:` / `Prevents:` / `Rule:` requis par le format ARCHITECTURE-SPINE.md. Conséquence : la revue de cohérence mécanique (`lint_spine.py`) ne peut pas valider les règles, et les agents aval ne peuvent pas citer les décisions par ID stable. Format gap non bloquant pour la compréhension humaine, mais bloquant pour le tooling BMad.

---

## High findings

### H-01 — CORS policy non définie

`WebConfig.java` est mentionné comme point d'entrée CORS mais aucun détail : quelles origines sont autorisées ? (outil interne bancaire = restrictif, possiblement `*.banque-interne.fr` uniquement). Un builder implémentant CORS sans directive créera soit une politique trop permissive (risque sécurité) soit trop restrictive (bloque le frontend dev).

### H-02 — Gestion des secrets non adressée

JWT secret "configurable en env var" (D-04) — mais pas de stratégie de secrets management. En contexte bancaire, un env var brut est insuffisant. Aucune mention de Vault, k8s Secrets, ou équivalent. Builders assumera l'env var ; sécurité assumera autre chose.

### H-03 — Stratégie de migration DB (ongoing) absente

7 fichiers Flyway V1–V7 sont listés (seed) mais aucune règle sur les migrations futures : convention de nommage ? Qui peut merger une migration ? Tests de migration obligatoires ? Sans règle, deux builders produiront des migrations V5bis et V5 qui entrent en conflit.

### H-04 — Section "Deferred" absente et éparpillée

Les décisions déférées (WebSocket Phase 2, VIEWER Phase 2, Kubernetes Phase 3, SMTP Phase 2) sont disséminées dans le document sans section consolidée. Risque : un builder ne les voit pas, implémente partiellement une feature Phase 2, et crée une dette architecturale non trackée.

---

## Medium / Low

### M-01 — Spring Boot 3.2.x potentiellement dépassé

Spring Boot 3.4.x est sorti fin 2024. 3.2.x reste en support mais sa EOL est novembre 2025. Pour un projet démarrant en 2026, 3.4.x ou 3.5.x serait plus prudent. (Non bloquant si la version est fixée délibérément pour compatibilité interne.)

### M-02 — React 18 vs React 19

React 19 est stable depuis décembre 2024. React 18 reste valide et largement supporté — non bloquant, mais `@xyflow/react` (React Flow v12) supporte React 19 ; vérifier que `@rjsf/mui` et `@mui/x-*` ont bien des releases React 19-compatibles si mise à jour envisagée.

### M-03 — Lint findings (9 low, tous faux positifs)

Le linter signale 9 `placeholder` (lignes 273, 288, 449, 450, 501, 513, 585, 793) — tous sont des variables de naming convention dans les tableaux P-01/P-04 (`{id}`, `{table_singulier}`, `{feature}`, `{jwt}`) ou des exemples de patterns URL. Aucun TBD réel. Verdict lint : propre sémantiquement.

### L-01 — Test strategy partiellement définie

P-05 définit les conventions de nommage et la structure des tests + couverture minimale (80/70/60%). Pas de mention de tests de contrat API (Spring Cloud Contract ? Pact ?), ni de stratégie pour tester l'isolation RBAC entre espaces. Non bloquant pour Sprint 1 mais utile avant Sprint 2.

---

## Checklist coverage

| Item | Statut | Note |
|---|---|---|
| 1. Divergence points couverts | ⚠️ PARTIEL | C-01 : generation-engine boundary manquante |
| 2. Règles enforceables | ✅ OUI | P-09 avec 10 règles explicites + D-01→D-16 clairs |
| 3. Section Deferred | ⚠️ PARTIEL | Décisions déférées présentes mais éparpillées (H-04) |
| 4. Format AD-n | ❌ MANQUANT | D-01/P-01 sans Binds/Prevents/Rule (C-03) |
| 5. Tech versions courantes | ⚠️ PARTIEL | Spring Boot 3.2.x vieillissant (M-01), reste cohérent |
| 6. Brownfield ratification | ✅ OUI | 112 typologies YAML, LDAP/AD, PG15 tous intégrés sans contradiction |
| 7. Couverture spec 53 FR / 26 NFR | ✅ OUI | Mapping complet dans la section validation |
| 8. Enveloppe opérationnelle | ⚠️ PARTIEL | D-14/15/16 présents ; secrets et log shipping absents |
| 9. Dimensions silencieuses | ❌ MANQUANT | Observabilité silencieuse (C-02), CORS (H-01), secrets (H-02), migration ongoing (H-03) |
| 10. Frontière generation-engine | ❌ MANQUANT | Propriété `generated_rows`, déclencheur job, quota enforcement (C-01) |
