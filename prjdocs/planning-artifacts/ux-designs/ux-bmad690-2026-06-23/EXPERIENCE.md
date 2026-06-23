---
name: GenData
description: Internal test dataset generation platform — behavioral and interaction spec
status: final
updated: 2026-06-23
---

# GenData — Expérience

> Spine comportementale. L'identité visuelle (couleurs, typographie, espacements, composants visuels) vit dans `DESIGN.md`. Ce document spécifie *comment ça fonctionne* ; DESIGN.md spécifie *comment ça ressemble*. Les tokens visuels sont référencés par nom (`{colors.primary}`, `{components.confidence-badge-high}`, etc.) — ne pas dupliquer leurs valeurs ici.

---

## Fondation

Application web uniquement. React TypeScript, desktop-first (1920×1080 cible), responsive vers tablet (1024px min). Souris et clavier primaires. Pas de mode hors-ligne. Mobile < 768px hors périmètre MVP.

**Système UI :** MUI v6 fournit la surface de composants (layout, formulaires, tables, navigation, TreeView). Le document `DESIGN.md` est la référence d'identité visuelle — il définit les deltas de marque sur les défauts MUI v6. Ne pas personnaliser les composants MUI non listés dans `DESIGN.md.Components`.

Bibliothèques spécialisées par surface :

| Bibliothèque | Surface |
|---|---|
| `@mui/x-data-grid` | Prévisualisation dataset — virtualisation jusqu'à 1 M lignes |
| `@mui/x-tree-view` | Sidebar Tribu → Projet → Sous-projet |
| Monaco Editor | Éditeur SQL inline (import DDL) |
| React Flow | Graphe de dépendances FK |
| `react-jsonschema-form` | Formulaires de configuration typologie générés depuis `configSchema` |
| Recharts | Métriques d'usage du workspace |

---

## Architecture de l'information

→ Référence visuelle : `mockups/key-dashboard.html`. La spine prime en cas de conflit.

### Hiérarchie de navigation

```
Workspace
└── N1 (Tribu / Département / Équipe / Domaine / BU — configurable à la création)
    └── Projet
        └── Sous-projet
            ├── Dataset (CSV → génération)
            └── DDL (SQL → schéma)
```

Le label N1 est configurable une fois à la création du workspace. Après création, il reste stable ; le modifier est une action de configuration avancée (via `LabelSettingsPanel`).

### Surfaces

| Surface | URL | Atteinte depuis | Rôle |
|---|---|---|---|
| Dashboard workspace | `/workspaces/:id` | Connexion · sidebar workspace | KPI, projets récents, fil d'activité |
| Liste de projets | `/workspaces/:id/projects` | Sidebar nœud N1 | Vue plate des projets du workspace |
| Détail projet | `/workspaces/:id/projects/:pid` | Sidebar nœud Projet | Sous-projets + datasets |
| Configuration dataset | `/workspaces/:id/projects/:pid/subprojects/:spid/datasets/:did` | Sidebar nœud Dataset | Config colonnes, génération, seed |
| Import DDL | `/workspaces/:id/projects/:pid/subprojects/:spid/ddl/:did` | Sidebar nœud DDL | Éditeur SQL, graphe FK, config génération par table |
| Navigateur de typologies | Drawer (overlay) | Bouton ⚙ sur ColumnConfigRow | Correction manuelle de typologie + Test Lab |
| Prévisualisation dataset | Drawer (overlay) | Bouton [Aperçu N lignes] | 5 à 20 lignes générées avant bulk |
| Paramètres workspace | `/workspaces/:id/settings` | Sidebar bas | N1 label, membres, export |
| Page mobile hors périmètre | Tout chemin < 768px | Navigateur mobile | Message "outil non disponible sur mobile" |

### Modèle de navigation

- **Sidebar 260px fixe sur desktop** — jamais repliable sur desktop (≥ 1024px). Sur tablet < 1024px : overlay hamburger, consultation seulement, pas de config colonne.
- **Breadcrumb** sur toutes les vues ; dernier élément non cliquable.
- **Pas de bouton retour explicite** : breadcrumb + sidebar suffisent.
- **Tabs** (filtre de contenu) : le contenu se filtre immédiatement sans navigation.
- **Deep-links stables** : copier l'URL = partager le contexte exact, y compris la colonne active et le seed. Aucune donnée de navigation ne vit uniquement en mémoire de session.

---

## Voix et ton

Microcopy. La voix de marque et la posture esthétique vivent dans `DESIGN.md.Brand & Style`.

**Registre :** outil professionnel bancaire. Direct, sans affect. L'outil parle à des développeurs et des QA — pas besoin de les encourager.

| Faire | Éviter |
|---|---|
| "3 colonnes nécessitent votre attention." | "Attention ! Des colonnes ont besoin de vous 🚨" |
| "50 000 lignes générées." | "Super ! Votre dataset est prêt ✓" |
| "Seed copié." | "Le seed a bien été copié dans le presse-papiers !" |
| "Colonne verrouillée — contrainte FK DDL." | "Vous ne pouvez pas modifier cette colonne." |
| "Aucun cycle FK détecté." | "Tout va bien, pas de cycle !" |
| "Réessayer" sur erreur. | "Quelque chose s'est mal passé." sans action. |
| Féliciter avant d'informer sur succès. | Commencer par le décompte technique. |

**Règle absolue :** ne jamais afficher une stack trace brute à l'utilisateur. Les erreurs techniques se résument en une phrase actionnable + bouton "Réessayer" ou instruction concrète.

**Micro-émotions à cultiver :** confiance (checkmark IBAN visible, aperçu avant bulk), progression (étapes numérotées), appartenance (avatar tribu, compteur membres), maîtrise (seed toujours visible, reproductibilité démontrée). Micro-émotions à éviter : doute, confusion, honte de données fausses, sentiment d'isolement.

---

## Composants — Comportements

Comportemental uniquement. Les spécifications visuelles (couleurs, radius, typographie) vivent dans `DESIGN.md.Components`.

→ Référence visuelle : `mockups/key-dataset-config.html` (C-01 à C-04), `mockups/key-typology-drawer.html` (C-01, C-03), `mockups/key-ddl-fk-graph.html` (C-02, C-05), `mockups/key-dashboard.html` (grille projets, KPI cards).

### C-01 ConfidenceBadge

| Attribut | Valeur |
|---|---|
| Rôle ARIA | `role="status"` |
| Label | `aria-label="Confiance {v}%"` |
| Interaction | Non interactif — indicateur lecture seule |
| Seuils | ≥ 80 % → `{components.confidence-badge-high}` · 50–79 % → `{components.confidence-badge-medium}` · < 50 % → `{components.confidence-badge-low}` |
| Tooltip | Survol : explication du score de confiance de l'auto-détection |

### C-02 SeedBadge

| Attribut | Valeur |
|---|---|
| Interaction | Clic → copie dans le presse-papiers |
| États | Défaut · Copié (checkmark, animation 1,5 s, puis retour défaut) |
| Label ARIA | `aria-label="Seed de reproductibilité : {v}. Cliquer pour copier"` |
| Placement | Toujours visible et proéminent — jamais caché derrière un clic ou un panneau |
| Typographie | `{typography.mono}` — signal visuel "donnée réelle" |

### C-03 TypologyChip

| Attribut | Valeur |
|---|---|
| États | Validé (détection ≥ seuil) · Avertissement (ambiguïté) · Manuel (assigné manuellement, violet pâle) |
| Interaction | Clic → ouvre le Drawer de configuration typologique filtré sur cette colonne |
| Accessibilité | `role="button"` · `aria-label="Changer la typologie : {nom}"` |

### C-04 ColumnConfigRow

Anatomie : `#` · nom de colonne · `TypologyChip` · `ConfidenceBadge` · toggle Nullable · toggle Unique · bouton ⚙

| État | Déclencheur | Comportement |
|---|---|---|
| Normal | — | Fond `{components.column-row-normal}` |
| Attention | Score confiance < seuil OU typologie non détectée | Fond `{components.column-row-attention}` — guide le regard vers les lignes à corriger |
| Verrouillé | Colonne issue d'un DDL avec contrainte FK | Fond `{components.column-row-locked}` · bouton ⚙ désactivé · tooltip "Verrouillé — contrainte FK DDL" |

Toggles Nullable et Unique : `aria-checked` sur l'état courant. Mise à jour optimiste côté client, rollback sur erreur serveur.

### C-05 FKGraphCanvas

| Attribut | Valeur |
|---|---|
| Bibliothèque | React Flow |
| Nœuds | Parent (aucun FK entrant) → `{components.fk-node-parent}` · Enfant (FK entrant) → `{components.fk-node-child}` · Sélectionné → bordure `{colors.border-hover}` |
| Interaction | Drag pour repositionner · Hover sur FK → tooltip `TABLE_A.colonne → TABLE_B.colonne` · Clic nœud → highlight colonnes FK dans la table de config |
| Cycle détecté | Alerte erreur bloquante avant l'affichage du graphe — "Cycle FK détecté : TABLE_A → TABLE_B → TABLE_A. Corrigez le DDL avant de continuer." |
| Accessibilité | Chaque nœud `role="button"` + `aria-label="Table {nom}, {N} colonnes"` ; flèches FK `aria-label="FK : {TABLE_A}.{col} → {TABLE_B}.{col}"` |

### C-06 GenerationProgressCard

| État | Comportement |
|---|---|
| En attente | Skeleton loader — `{typography.body}` "Génération en attente…" |
| En cours | Barre de progression animée (`role="progressbar"` · `aria-valuenow` · `aria-valuemax`) · compteurs colonnes en cours / terminées / échouées |
| Terminé | Alerte succès + boutons [Télécharger] [Copier le seed] [Partager le lien] |
| Erreur partielle | Alerte avertissement : N colonnes générées, M échouées — [Réessayer les colonnes échouées] |
| Erreur totale | Alerte erreur — message résumé + [Réessayer] — jamais de stack trace |

### C-07 LabelSettingsPanel

- Déclenché par ⚙ dans l'en-tête sidebar.
- S'ouvre en panneau inline sous l'en-tête (pas un Dialog, pas un Drawer).
- Anatomie : chips prédéfinies (Tribu / Département / Équipe / Domaine / BU) + champ libre + bouton [OK].
- Fermeture : clic [OK] · clic hors du panneau · touche Échap.
- Modifie le label N1 pour tout le workspace — action visible de tous les membres.

### Hiérarchie des boutons

Un seul bouton primaire par vue. Règle absolue : deux boutons primaires concurrents sur la même surface = erreur de conception.

| Variante | Usage | Irréversibilité |
|---|---|---|
| Primaire | Action principale unique de la vue | Réversible |
| Succès | Génération bulk (⚡ Générer N lignes) | **Irréversible** — une fois lancée, la génération ne s'annule pas |
| Secondaire | Aperçu, actions réversibles | Réversible |
| Ghost/Icône | ⚙ · ··· — actions contextuelles | — |
| Danger | Actions destructives dans un dropdown | Irréversible — confirmer via Dialog |

### Formulaires

- Labels toujours au-dessus du champ — jamais placeholder seul.
- Validation inline au `blur` — HelperText rouge sous le champ.
- Champs requis : astérisque rouge + note "Champs requis" en bas du formulaire.
- Valeurs par défaut : seed = aléatoire pré-rempli · lignes = 1 000 · format = JSON.
- Inputs numériques (seed, lignes) : pas de flèches haut/bas — saisie directe uniquement. Largeurs contextuelles (seed ≈ 160px, lignes ≈ 120px).

---

## Plancher d'accessibilité

Comportemental. Le contraste visuel vit dans `DESIGN.md.Colors`.

- **Cible WCAG 2.1 AA** sur l'intégralité de la surface web.
- **Bague de focus** sur tous les éléments interactifs — `{colors.border-hover}` 2px, offset 2px. Jamais `outline: none` sans alternative visible.
- **ARIA :**
  - `ConfidenceBadge` : `role="status"` + `aria-label="Confiance {v}%"`
  - `SeedBadge` : `aria-label="Seed de reproductibilité : {v}. Cliquer pour copier"`
  - Toggles Nullable/Unique : `aria-checked`
  - `GenerationProgressCard` : `role="progressbar"` + `aria-valuenow` + `aria-valuemax`
  - Chips de catégorie actives : `aria-pressed="true"`
- **Navigation clavier** : ordre Tab logique dans la sidebar, la table de configuration, les drawers. Pas de piège au focus hors Dialog.
- **Cibles tactiles** : min 32×32px desktop, 44×44px tablet.
- **`prefers-reduced-motion`** : désactive l'animation de checkmark du SeedBadge et l'animation subtile de l'auto-détect > 95 %. Les transitions de layout restent.
- **Langue** : `lang="fr"` sur `<html>`. Les libellés ARIA en français.
- **Texte désactivé** (`{colors.text-disabled}`) : jamais seul porteur d'une information fonctionnelle — toujours doublé par forme, position ou icône.

---

## Patterns d'état

→ Référence visuelle : `mockups/key-dataset-config.html` (états colonnes), `mockups/key-dashboard.html` (état peuplé).

### Chargement initial

- **Skeleton loaders MUI** sur toutes les surfaces — pas de spinner global.
- Chargement inline (auto-détection colonne) : `CircularProgress` 16px dans la colonne Confiance.
- Jamais : page blanche + spinner centré.

### État vide

| Surface | Message | CTA |
|---|---|---|
| Liste de projets (workspace neuf) | Icône illustrative + "Créez votre premier projet pour commencer." | [+ Créer un projet] |
| Sous-projet sans dataset | "Aucun dataset dans ce sous-projet." | [+ Nouveau Dataset] |
| Résultats de recherche vides | "Aucune typologie ne correspond à « {terme} »." | — |
| Workspace sans activité | "Aucune activité récente." | — |

### Erreur de chargement

Message inline (pas de page d'erreur complète) + bouton [Réessayer]. Jamais de stack trace.

### Succès

- Alerte verte (composant `{components.alert-success}`) + Snackbar auto-dismiss 5 s.
- Snackbar sur erreur persistante : dismiss manuel uniquement.
- Règle : féliciter avant d'informer ("50 000 lignes générées avec succès." — puis détails).

### Feedback par type

| Type | Composant | Comportement |
|---|---|---|
| Succès | `{components.alert-success}` + Snackbar | Auto-dismiss 5 s |
| Info / guidage | `{components.alert-info}` | Manuel |
| Attention requise | `{components.alert-warning}` | Manuel |
| Erreur récupérable | `{components.alert-error}` | Manuel — toujours avec [Réessayer] |

---

## Primitives d'interaction

### Souris

- Clic pour agir. Hover révèle les actions contextuelles (⚙ · ···) sur les lignes.
- Drag-and-drop CSV sur la zone d'upload — feedback visuel pendant le survol.
- Drag pour repositionner les nœuds FK dans `FKGraphCanvas`.
- `box-shadow` au hover sur les cartes → signal d'interactivité.

### Clavier

- Navigation Tab logique dans la sidebar, la table de config, les drawers — pas de piège au focus (sauf modals, voir ci-dessous).
- Raccourcis sur les surfaces clés :

| Touche | Action |
|---|---|
| `Tab` | Navigation entre éléments interactifs dans l'ordre de lecture |
| `Espace` / `Entrée` | Active le bouton ou toggle focalisé |
| `Échap` | Ferme le drawer ou dialog actif |
| `⌘C` / `Ctrl+C` sur SeedBadge focalisé | Copie le seed |

- Aucun raccourci clavier global (style vim) en v1.

### Piège au focus

Uniquement dans les Dialogs destructifs (confirmation). Les Drawers ne piègent pas le focus — Tab sort du drawer et revient à la table sous-jacente.

### Modals et overlays

| Cas | Composant | Justification |
|---|---|---|
| Config typologie + Test Lab | Drawer 400px slide-in droite | Table de config visible à gauche — contexte conservé |
| Confirmation destructive | Dialog centré — titre explicite + bouton danger + annuler | Irréversibilité justifie l'interruption |
| Config label N1 | Panneau inline dans l'en-tête sidebar | Pas de rupture de navigation |
| Aide / documentation | Tooltip enrichi au survol | Contextuel, non bloquant |

**Règle :** les Dialogs sont réservés aux confirmations destructives uniquement. Tout le reste = Drawer ou panneau inline.

---

## Génération et reproductibilité

Le cycle de génération est l'action centrale de GenData. La reproductibilité est sa valeur clé — cible émotionnelle "Je maîtrise".

**Cycle asynchrone :**

1. L'utilisateur configure seed, nombre de lignes, format, puis clique [⚡ Générer N lignes].
2. Un job asynchrone est créé côté serveur. L'UI affiche immédiatement `GenerationProgressCard` (état "En cours").
3. La progression est mise à jour en temps réel (polling ou WebSocket) — barre de progression + compteurs colonnes.
4. **Climax (succès) :** `GenerationProgressCard` passe à l'état "Terminé". Alerte verte + trois actions : [Télécharger] · [Copier le seed] · [Partager le lien].
5. L'utilisateur peut naviguer ailleurs pendant la génération — le job continue. La progression est accessible depuis la sidebar (badge ou entrée dédiée).
6. Timeout : si le job dépasse 5 minutes sans réponse, alerte orange + [Vérifier le statut].

**Règle :** ne jamais bloquer l'UI pendant une génération. La génération est toujours asynchrone, même pour 100 lignes.

**Seed :**

- Le seed est **toujours visible et proéminent** sur toute surface liée à la génération (`SeedBadge`). Jamais caché.
- Le seed par défaut est pré-rempli aléatoirement — l'utilisateur peut le modifier avant génération.
- Après génération, le seed est copiable en un clic et partageable via URL.
- Reproductibilité collaborative : cliquer [Régénérer avec le seed] sur le dataset d'un collègue lance une génération identique < 10 s pour 1 000 lignes.

---

## Pipeline d'import DDL

→ Référence visuelle : `mockups/key-ddl-fk-graph.html`.

1. **Éditeur SQL (Monaco)** : l'utilisateur colle ou saisit les instructions `CREATE TABLE`. Coloration syntaxique SQL activée. Bouton [Analyser le DDL].
2. **Analyse FK** (< 3 s) : détection des contraintes FK, construction du graphe de dépendances.
3. **Détection de cycles** : si un cycle FK est détecté, alerte erreur bloquante avec le chemin du cycle. L'utilisateur doit corriger le DDL avant de continuer. Pas de génération partielle en cas de cycle.
4. **FKGraphCanvas** : graphe React Flow interactif. Nœuds repositionnables. Hover FK → tooltip. Clic nœud → focus dans la table de config.
5. **Configuration par table** : chaque table a sa propre ligne (seed, nombre de lignes, format). L'ordre de génération est déduit du graphe (parents avant enfants) et affiché explicitement.
6. **Génération schéma** : [⚡ Générer le schéma complet] lance la génération dans l'ordre calculé. Intégrité référentielle garantie par l'ordre de génération.

---

## Flux clés

### Flow 1 — Génération nominale (Yann, développeur backend, 10h30 mardi)

Yann doit générer un dataset de virements SEPA pour ses tests d'intégration.

1. Yann navigue dans la sidebar : Tribu KYC → Virements SEPA → Tests unitaires.
2. Il clique [+ Nouveau Dataset].
3. Un formulaire de création s'affiche — il saisit "virement_test_v3" et glisse-dépose son CSV.
4. L'auto-détection tourne (< 2 s) — `CircularProgress` dans chaque cellule Confiance.
5. La table de configuration s'affiche : lignes vertes (confiance ≥ 80 %) et lignes orange (attention). Alerte orange en haut : "3 colonnes nécessitent votre attention."
6. Yann corrige les 3 lignes orange via le Drawer (voir Flow 2 pour le détail).
7. En bas de page, il confirme : 1 000 lignes, format JSON, seed pré-rempli aléatoirement.
8. Il clique [Aperçu 5 lignes] (secondaire) — un Drawer d'aperçu s'ouvre à droite avec 5 lignes en Roboto Mono. Il vérifie que les IBANs ressemblent à de vrais IBANs.
9. Il clique [⚡ Générer 1 000 lignes] (succès).
10. **Climax :** `GenerationProgressCard` s'affiche. En 8 secondes, la barre passe à 100 %. Alerte verte : "1 000 lignes générées." Trois boutons : [Télécharger] · [Copier le seed] · [Partager le lien]. Yann télécharge et colle le fichier dans son test CI. Il copie le seed dans un commentaire de code.

Échec : auto-détection timeout → alerte orange + [Réessayer l'auto-détection] + option assignation manuelle complète.

### Flow 2 — Correction manuelle + Test Lab (Fatima, QA engineer, 14h15 mercredi)

Fatima a un CSV avec des en-têtes non standards — l'auto-détection a des scores bas.

1. Elle est sur la table de config de son dataset — 2 lignes orange, `ConfidenceBadge` rouge < 50 %.
2. Elle clique ⚙ sur la ligne `code_operation` (badge rouge 43 %).
3. Le Drawer de configuration typologique s'ouvre à 400px depuis la droite. La table reste visible à gauche.
4. Elle tape "opération" dans le Navigateur de typologies — recherche en temps réel (debounce 300 ms).
5. "Code opération bancaire" apparaît en premier avec `ConfidenceBadge` vert 89 %. Elle clique sur la ligne pour la mettre en surbrillance.
6. Elle ouvre le **Test Lab** (inline dans le Drawer). Elle configure : Nullable OFF, Unique OFF.
7. Le Test Lab génère 5 exemples en < 500 ms — elle voit "SCT", "SDD", "SCT", "SDD-R", "SCT" en Roboto Mono. Reconnaissable immédiatement.
8. Elle clique [Assigner "Code opération bancaire"].
9. **Climax :** le Drawer se ferme. La ligne `code_operation` est maintenant verte — `TypologyChip` bleu "Code opération bancaire", `ConfidenceBadge` vert 89 %. Le nombre de colonnes en attention dans l'alerte orange passe de 2 à 1. Fatima ressent "cet outil me comprend."

Cas d'erreur : aucune typologie ne correspond à la recherche → "Aucune typologie ne correspond à « {terme} »." + lien vers la documentation des 112 typologies.

### Flow 3 — Import DDL et graphe FK (Omar, architecte data, 16h00 jeudi)

Omar importe un schéma SQL de 6 tables avec des contraintes FK complexes.

1. Omar navigue : Transactions SWIFT → Tests intégration. Il clique [+ Nouveau DDL].
2. Il colle 6 instructions `CREATE TABLE` dans l'éditeur Monaco. Coloration syntaxique active.
3. Il clique [Analyser le DDL].
4. Analyse FK (< 3 s) — `CircularProgress` sur le bouton. Résultat : aucun cycle.
5. Alerte verte : "Aucun cycle FK détecté. Ordre de génération : swift_message → contrepartie → …"
6. Le `FKGraphCanvas` s'affiche — 6 nœuds, 4 flèches FK. Omar drag-repositionne pour clarifier la lecture.
7. Il survole une flèche FK → tooltip "swift_message.id_message → contrepartie.id_message".
8. Dans la table de config par table, il ajuste : 10 000 lignes par table, format JSON, seeds distincts.
9. Il assigne les typologies manquantes colonne par colonne (certaines ont été auto-détectées, d'autres non).
10. Il clique [⚡ Générer le schéma complet].
11. **Climax :** `GenerationProgressCard` montre 6 barres de progression — une par table, dans l'ordre des dépendances. Les tables parents terminent en premier ; les tables enfants démarrent après confirmation de l'intégrité. 60 000 lignes cohérentes sont générées. Omar partage le lien via Slack — ses collègues peuvent régénérer à l'identique.

Cycle FK détecté : alerte erreur bloquante avant le graphe — "Cycle FK détecté : TABLE_A → TABLE_B → TABLE_A. Corrigez le DDL avant de continuer." Pas de génération possible.

### Flow 4 — Reproductibilité collaborative (Omar, premier jour, 09h00 lundi)

Omar arrive dans l'équipe. Il veut reproduire le dataset de test standard de l'équipe.

1. Il se connecte — la sidebar affiche les espaces existants de son organisation.
2. Il clique sur "KYC Department" (son département).
3. Il voit les projets de l'équipe (référence visuelle : `mockups/key-dashboard.html`).
4. Il clique sur "Virements SEPA" → "Tests unitaires" → dataset "virement_test_v3" partagé par Yann.
5. La page de configuration s'ouvre en lecture — toutes les typologies assignées, seed = 42 visible dans le `SeedBadge`.
6. Il clique [Régénérer avec ce seed].
7. **Climax :** job asynchrone lancé. En < 10 s pour 1 000 lignes, un dataset identique à l'original est généré. Omar télécharge — les IBANs, montants et codes sont strictement identiques au dataset de référence. Il a confiance dans ses tests dès le premier jour. Sentiment : "Je maîtrise."

Option d'exploration : depuis cette page, Omar peut ouvrir le Navigateur de typologies et le Test Lab pour comprendre comment chaque colonne est configurée — sans modifier le dataset original.

---

## Responsive et plateforme

| Breakpoint | Comportement |
|---|---|
| Desktop ≥ 1440px | Layout complet. Sidebar 260px fixe. Contenu sans `max-width`. |
| Desktop 1280–1440px | Identique — layout fluide. |
| Tablet ≥ 1024px | Même layout. Cibles tactiles 44px min. |
| Tablet < 1024px | Sidebar en overlay hamburger — **consultation uniquement**. Config colonne, Drawer typologique et génération désactivés sur cette taille. |
| Mobile < 768px | Hors périmètre MVP. Afficher la page "Outil non disponible sur mobile" avec le message "GenData est optimisé pour desktop. Connectez-vous depuis un ordinateur." |

**Approche :** desktop-first. Les adaptations tablet se font via `@media (max-width: 1023px)`. Pas de breakpoints entre 768px et 1023px — pas de surface intermédiaire.

**Conseil aux développeurs :** les composants MUI sont responsive par défaut ; ne pas surcharger leurs breakpoints internes. Les adaptations GenData portent uniquement sur la sidebar et la désactivation des surfaces d'édition sur tablet < 1024px.
