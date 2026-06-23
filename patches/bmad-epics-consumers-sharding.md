# Patch: sharded epics — consumer skills

## Context

The `bmad-create-epics-and-stories` skill now produces a sharded structure:

```
{planning_artifacts}/epics/
├── index.md                   ← requirements, FR map, epic list (with links)
├── epic-1-{title}.md          ← goal + stories for epic 1
├── epic-2-{title}.md          ← goal + stories for epic 2
└── ...
```

This file fixes the skills that **consume** this output and still reference the old monolithic `epics.md`.

## Application rules

- Apply each change in **both copies** of the file: `.claude/skills/…` AND `.agents/skills/…`
- Each change is **idempotent**: verify the old text is present before replacing
- Only modify the indicated lines — do not touch the rest of the file
- After each replacement (or attempt), display the result following the protocol below

## Reporting protocol

After each operation on a file, display a status line:

| Result | Format |
|--------|--------|
| Already up to date (guard triggered) | `[SKIP] <path> — already patched` |
| Replacement applied successfully | `[OK]   <path> — <replacement label>` |
| Source text not found in file | `[ERR]  <path> — text not found: "<beginning of searched text…>"` |
| File missing | `[ERR]  <path> — file not found` |

At the end of all operations, display a summary:

```
=== Patch result ===
OK    : N
SKIP  : N
ERROR : N
```

If at least one error is present, list the corrective actions to take.

---

## File 1 — `bmad-create-story/SKILL.md`

**Paths:**
- `.claude/skills/bmad-create-story/SKILL.md`
- `.agents/skills/bmad-create-story/SKILL.md`

**Guard:** if the file already contains `epics/index.md` on the relevant line → `[SKIP]`

### Replacement

Search (line ~71):
```
- `epics_file` = `{planning_artifacts}/epics.md`
```

Replace with:
```
- `epics_file` = `{planning_artifacts}/epics/index.md`
```

---

## File 2 — `bmad-sprint-planning/SKILL.md`

**Paths:**
- `.claude/skills/bmad-sprint-planning/SKILL.md`
- `.agents/skills/bmad-sprint-planning/SKILL.md`

**Guard:** if the file already contains `Search for sharded version first` → `[SKIP]` (all three replacements are considered applied)

### Replacement A — Epic Discovery Process block (lines ~88–95)

Search:
```
1. **Search for whole document first** - Look for `epics.md`, `bmm-epics.md`, or any `*epic*.md` file
2. **Check for sharded version** - If whole document not found, look for `epics/index.md`
3. **If sharded version found**:
   - Read `index.md` to understand the document structure
   - Read ALL epic section files listed in the index (e.g., `epic-1.md`, `epic-2.md`, etc.)
   - Process all epics and their stories from the combined content
   - This ensures complete sprint status coverage
4. **Priority**: If both whole and sharded versions exist, use the whole document
```

Replace with:
```
1. **Search for sharded version first** - Look for `epics/index.md`
2. **If sharded version found**:
   - Read `index.md` to understand the document structure
   - Read ALL epic section files listed in the index (e.g., `epic-1-user-auth.md`, `epic-2-content.md`, etc.)
   - Process all epics and their stories from the combined content
   - This ensures complete sprint status coverage
3. **Fallback (legacy only)** - If `epics/index.md` not found, look for `epics.md`, `bmm-epics.md`, or any `*epic*.md` file
```

### Replacement B — fuzzy matching line (line ~97)

Search:
```
**Fuzzy matching**: Be flexible with document names - users may use variations like `epics.md`, `bmm-epics.md`, `user-stories.md`, etc.
```

Replace with:
```
**Fuzzy matching**: Be flexible with document names - canonical form is `epics/index.md` (sharded); legacy fallbacks include `epics.md`, `bmm-epics.md`, `user-stories.md`.
```

### Replacement C — action tag (line ~105)

Search:
```
<action>Could be a single `epics.md` file or multiple `epic-1.md`, `epic-2.md` files</action>
```

Replace with:
```
<action>Expected structure: sharded `epics/index.md` + `epics/epic-{N}-{title}.md` files. Legacy fallback: single `epics.md`.</action>
```

---

## File 3 — `bmad-quick-dev/step-01-clarify-and-route.md`

**Paths:**
- `.claude/skills/bmad-quick-dev/step-01-clarify-and-route.md`
- `.agents/skills/bmad-quick-dev/step-01-clarify-and-route.md`

**Guard:** if the file already contains `epics/epic-<N>-*.md` → `[SKIP]`

### Replacement

Search (line ~62):
```
spawn a sub-agent with `./compile-epic-context.md` as its prompt. Pass it the epic number, the epics file path, the `{planning_artifacts}` directory, and the output path `{implementation_artifacts}/epic-<N>-context.md`.
```

Replace with:
```
spawn a sub-agent with `./compile-epic-context.md` as its prompt. Pass it the epic number, the epic file path (resolve as `{planning_artifacts}/epics/epic-<N>-*.md`; fallback to `{planning_artifacts}/epics.md` if sharded structure absent), the `{planning_artifacts}` directory, and the output path `{implementation_artifacts}/epic-<N>-context.md`.
```

---

## File 4 — `bmad-create-epics-and-stories/customize.toml` _(low priority)_

**Paths:**
- `.claude/skills/bmad-create-epics-and-stories/customize.toml`
- `.agents/skills/bmad-create-epics-and-stories/customize.toml`

**Guard:** if the file already contains `epics/ folder` → `[SKIP]`

### Replacement

Search (line ~38):
```
# user confirms [C] Complete — after the epics.md is saved and bmad-help is invoked.
```

Replace with:
```
# user confirms [C] Complete — after the epics/ folder is complete (index.md + epic files) and bmad-help is invoked.
```

---

## File 5 — `bmad-create-epics-and-stories` — **producer** sharding

This patch updates the **producer** skill that currently writes a monolithic
`{planning_artifacts}/epics.md` to instead produce a **sharded folder**:

```
{planning_artifacts}/epics/
├── index.md                     ← requirements, FR map, epic list
├── epic-1-{title}.md            ← epic 1 goal + stories
├── epic-2-{title}.md            ← epic 2 goal + stories
└── ...
```

**Paths (all):**
- `.claude/skills/bmad-create-epics-and-stories/steps/step-XX-*.md`
- `.agents/skills/bmad-create-epics-and-stories/steps/step-XX-*.md`
- `.claude/skills/bmad-create-epics-and-stories/templates/epics-template.md`
- `.agents/skills/bmad-create-epics-and-stories/templates/epics-template.md`

**Guard:** if the file already contains `{planning_artifacts}/epics/` (path with trailing slash for folder) → `[SKIP]` (all replacements in that file are considered applied).

---

### File 5A — `step-01-validate-prerequisites.md`

**Paths:**
- `.claude/skills/bmad-create-epics-and-stories/steps/step-01-validate-prerequisites.md`
- `.agents/skills/bmad-create-epics-and-stories/steps/step-01-validate-prerequisites.md`

**Guard:** if the file already contains `{planning_artifacts}/epics/` → `[SKIP]`

#### Replacement 1 — execution protocol output path (line ~35)

Search:
```
- 💾 Populate {planning_artifacts}/epics.md with extracted requirements
```

Replace with:
```
- 💾 Populate {planning_artifacts}/epics/index.md with extracted requirements
```

#### Replacement 2 — template initialization (line ~72)

Search:
```
Before proceeding, Ask the user if there are any other documents to include for analysis, and if anything found should be excluded. Wait for user confirmation. Once confirmed, create the {planning_artifacts}/epics.md from the ../templates/epics-template.md and in the front matter list the files in the array of `inputDocuments: []`.
```

Replace with:
```
Before proceeding, Ask the user if there are any other documents to include for analysis, and if anything found should be excluded. Wait for user confirmation. Once confirmed, create the {planning_artifacts}/epics/index.md from the ../templates/epics-template.md (which will be split into index.md + per-epic files) and in the front matter list the files in the array of `inputDocuments: []`.
```

#### Replacement 3 — template load + init (line ~163)

Search:
```
### 7. Load and Initialize Template

Load ../templates/epics-template.md and initialize {planning_artifacts}/epics.md:
1. Copy the entire template to {planning_artifacts}/epics.md
2. Replace {{project_name}} with the actual project name
3. Replace placeholder sections with extracted requirements:
   - {{fr_list}} → extracted FRs
   - {{nfr_list}} → extracted NFRs
   - {{additional_requirements}} → extracted additional requirements (from Architecture)
   - {{ux_design_requirements}} → extracted UX Design Requirements (if UX document exists)
4. Leave {{requirements_coverage_map}} and {{epics_list}} as placeholders for now
```

Replace with:
```
### 7. Load and Initialize Template

Load ../templates/epics-template.md and initialize {planning_artifacts}/epics/index.md:
1. Copy the entire template to {planning_artifacts}/epics/index.md
2. Replace {{project_name}} with the actual project name
3. Replace placeholder sections with extracted requirements:
   - {{fr_list}} → extracted FRs
   - {{nfr_list}} → extracted NFRs
   - {{additional_requirements}} → extracted additional requirements (from Architecture)
   - {{ux_design_requirements}} → extracted UX Design Requirements (if UX document exists)
4. Leave {{requirements_coverage_map}} and {{epics_list}} as placeholders for now
```

#### Replacement 4 — content save (line ~209)

Search:
```
After extraction and confirmation, update {planning_artifacts}/epics.md with:
```

Replace with:
```
After extraction and confirmation, update {planning_artifacts}/epics/index.md with:
```

#### Replacement 5 — menu handling (line ~228)

Search:
```
- IF C: Save all to {planning_artifacts}/epics.md, update frontmatter, then read fully and follow: ./step-02-design-epics.md
```

Replace with:
```
- IF C: Save all to {planning_artifacts}/epics/index.md, update frontmatter, then read fully and follow: ./step-02-design-epics.md
```

#### Replacement 6 — critical step completion (line ~233)

Search:
```
ONLY WHEN C is selected and all requirements are saved to document and frontmatter is updated, will you then read fully and follow: ./step-02-design-epics.md to begin epic design step.
```

Replace with:
```
ONLY WHEN C is selected and all requirements are saved to {planning_artifacts}/epics/index.md and frontmatter is updated, will you then read fully and follow: ./step-02-design-epics.md to begin epic design step.
```

---

### File 5B — `step-02-design-epics.md`

**Paths:**
- `.claude/skills/bmad-create-epics-and-stories/steps/step-02-design-epics.md`
- `.agents/skills/bmad-create-epics-and-stories/steps/step-02-design-epics.md`

**Guard:** if the file already contains `{planning_artifacts}/epics/` → `[SKIP]`

#### Replacement 1 — execution protocol (line ~36)

Search:
```
- 💾 Update {{epics_list}} in {planning_artifacts}/epics.md
```

Replace with:
```
- 💾 Update {{epics_list}} in {planning_artifacts}/epics/index.md
```

#### Replacement 2 — load epics (line ~44)

Search:
```
Load {planning_artifacts}/epics.md and review:
```

Replace with:
```
Load {planning_artifacts}/epics/index.md and review:
```

#### Replacement 3 — content to update (line ~194)

Search:
```
After approval, update {planning_artifacts}/epics.md:
```

Replace with:
```
After approval, update {planning_artifacts}/epics/index.md:
```

#### Replacement 4 — menu handling (line ~208)

Search:
```
- IF C: Save approved epics_list to {planning_artifacts}/epics.md, update frontmatter, then read fully and follow: ./step-03-create-stories.md
```

Replace with:
```
- IF C: Save approved epics_list to {planning_artifacts}/epics/index.md, update frontmatter, then read fully and follow: ./step-03-create-stories.md
```

#### Replacement 5 — critical step completion (line ~220)

Search:
```
ONLY WHEN C is selected and the approved epics_list is saved to document, will you then read fully and follow: ./step-03-create-stories.md to begin story creation step.
```

Replace with:
```
ONLY WHEN C is selected and the approved epics_list is saved to {planning_artifacts}/epics/index.md, will you then read fully and follow: ./step-03-create-stories.md to begin story creation step.
```

---

### File 5C — `step-03-create-stories.md`

**Paths:**
- `.claude/skills/bmad-create-epics-and-stories/steps/step-03-create-stories.md`
- `.agents/skills/bmad-create-epics-and-stories/steps/step-03-create-stories.md`

**Guard:** if the file already contains `{planning_artifacts}/epics/` → `[SKIP]`

#### Replacement 1 — execution protocol (line ~36)

Search:
```
- 💾 Append epics and stories to {planning_artifacts}/epics.md following template
```

Replace with:
```
- 💾 Append epics and stories to {planning_artifacts}/epics/index.md (header) and per-epic files under {planning_artifacts}/epics/ following template
```

#### Replacement 2 — load approved epics (line ~44)

Search:
```
Load {planning_artifacts}/epics.md and review:
```

Replace with:
```
Load {planning_artifacts}/epics/index.md and review:
```

#### Replacement 3 — append to document (line ~168)

Search:
```
- Append it to {planning_artifacts}/epics.md following template structure
```

Replace with:
```
- Append it to the per-epic file `{planning_artifacts}/epics/epic-{N}-{title}.md` following template structure (create the file if it does not exist yet; append the story block under the appropriate epic section)
```

#### Replacement 4 — menu handling (line ~219)

Search:
```
- IF C: Save content to {planning_artifacts}/epics.md, update frontmatter, then read fully and follow: ./step-04-final-validation.md
```

Replace with:
```
- IF C: Save content to {planning_artifacts}/epics/index.md (header + requirements) and per-epic files under {planning_artifacts}/epics/ (epic goals + stories), update frontmatter, then read fully and follow: ./step-04-final-validation.md
```

#### Replacement 5 — critical step completion (line ~231)

Search:
```
ONLY WHEN [C continue option] is selected and [all epics and stories saved to document following the template structure exactly], will you then read fully and follow: `./step-04-final-validation.md` to begin final validation phase.
```

Replace with:
```
ONLY WHEN [C continue option] is selected and [all epics and stories saved to {planning_artifacts}/epics/index.md (header + requirements) and per-epic files under {planning_artifacts}/epics/ (epic goals + stories) following the template structure exactly], will you then read fully and follow: `./step-04-final-validation.md` to begin final validation phase.
```

---

### File 5D — `step-04-final-validation.md`

**Paths:**
- `.claude/skills/bmad-create-epics-and-stories/steps/step-04-final-validation.md`
- `.agents/skills/bmad-create-epics-and-stories/steps/step-04-final-validation.md`

**Guard:** if the file already contains `{planning_artifacts}/epics/` → `[SKIP]`

#### Replacement — save final (line ~126)

Search:
```
- Save the final epics.md
```

Replace with:
```
- Save the final {planning_artifacts}/epics/index.md and all per-epic files under {planning_artifacts}/epics/
```

#### Replacement — step completion (line ~133)

Search:
```
When C is selected, the workflow is complete and the epics.md is ready for development.
```

Replace with:
```
When C is selected, the workflow is complete and the epics/ folder (index.md + per-epic files) is ready for development.
```

---

### File 5E — `templates/epics-template.md`

**Paths:**
- `.claude/skills/bmad-create-epics-and-stories/templates/epics-template.md`
- `.agents/skills/bmad-create-epics-and-stories/templates/epics-template.md`

**Guard:** if the file already contains `epics/index.md` → `[SKIP]`

#### Replacement — split into two templates

The current template produces a single monolithic file.  Replace it with **two**
templates:

**5E-1 — `templates/index-template.md`** (replaces the existing file):

```markdown
---
stepsCompleted: []
inputDocuments: []
---

# {{project_name}} - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for {{project_name}}, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

{{fr_list}}

### NonFunctional Requirements

{{nfr_list}}

### Additional Requirements

{{additional_requirements}}

### UX Design Requirements

{{ux_design_requirements}}

### FR Coverage Map

{{requirements_coverage_map}}

## Epic List

{{epics_list}}

<!-- Repeat for each epic in epics_list (N = 1, 2, 3...) -->

## Epic {{N}}: {{epic_title_N}}

{{epic_goal_N}}

<!-- Repeat for each story (M = 1, 2, 3...) within epic N -->

### Story {{N}}.{{M}}: {{story_title_N_M}}

As a {{user_type}},
I want {{capability}},
So that {{value_benefit}}.

**Acceptance Criteria:**

<!-- for each AC on this story -->

**Given** {{precondition}}
**When** {{action}}
**Then** {{expected_outcome}}
**And** {{additional_criteria}}

<!-- End story repeat -->
```

**5E-2 — `templates/epic-template.md`** (new file, one per epic):

```markdown
---
stepsCompleted: []
epic_num: {{N}}
epic_title: {{epic_title_N}}
inputDocuments: []
---

## Epic {{N}}: {{epic_title_N}}

{{epic_goal_N}}

<!-- Repeat for each story (M = 1, 2, 3...) within epic N -->

### Story {{N}}.{{M}}: {{story_title_N_M}}

As a {{user_type}},
I want {{capability}},
So that {{value_benefit}}.

**Acceptance Criteria:**

<!-- for each AC on this story -->

**Given** {{precondition}}
**When** {{action}}
**Then** {{expected_outcome}}
**And** {{additional_criteria}}

<!-- End story repeat -->
```

**Application:**
1. Rename (or copy) the existing `templates/epics-template.md` → `templates/index-template.md` with the content above.
2. Create a new file `templates/epic-template.md` with the content above.
3. Update references in steps 01–03 from `../templates/epics-template.md` to `../templates/index-template.md` (for index init) and from `../templates/epics-template.md` to `../templates/epic-template.md` (for per-epic story append).

**Specific replacements in step-01 (lines ~163):**

Search:
```
Load ../templates/epics-template.md and initialize {planning_artifacts}/epics/index.md:
1. Copy the entire template to {planning_artifacts}/epics/index.md
```

Replace with:
```
Load ../templates/index-template.md and initialize {planning_artifacts}/epics/index.md:
1. Copy the entire template to {planning_artifacts}/epics/index.md
```

**Specific replacements in step-03 (lines ~168):**

Search:
```
- Append it to the per-epic file `{planning_artifacts}/epics/epic-{N}-{title}.md` following template structure (create the file if it does not exist yet; append the story block under the appropriate epic section)
```

Replace with:
```
- Append it to the per-epic file `{planning_artifacts}/epics/epic-{N}-{title}.md` following `../templates/epic-template.md` structure (create the file if it does not exist yet; append the story block under the appropriate epic section)
```

**Paths:**
- `.claude/skills/bmad-create-epics-and-stories/customize.toml`
- `.agents/skills/bmad-create-epics-and-stories/customize.toml`

**Guard:** if the file already contains `epics/ folder` → `[SKIP]`

### Replacement

Search (line ~38):
```
# user confirms [C] Complete — after the epics.md is saved and bmad-help is invoked.
```

Replace with:
```
# user confirms [C] Complete — after the epics/ folder is complete (index.md + epic files) and bmad-help is invoked.
```
