# Patch: sprints and stories as folders

## Context

The project combines **sprint folders** and a **`sprint-status.yaml`** file co-located under `sprints/`:

```
{implementation_artifacts}/sprints/
├── sprint-status.yaml             ← global tracking file (moved here from impl_artifacts/ root)
├── sprint-1-started/              ← sprint N, status = folder name suffix
│   ├── 1-1-user-auth.md           ← story file
│   └── 1-2-account.md
└── sprint-2-planned/
    └── 2-1-dashboard.md
```

**Convention:**
- `sprints/sprint-{N}-{status}/` — status ∈ `{planned,started,ended,skipped}` = folder name suffix
- Stories: `sprints/sprint-{N}-{status}/{story-key}.md`
- Active sprint = first folder with suffix `-started` (fallback: first `-planned`)
- `sprint-status.yaml` lives under `sprints/` (not at the root of `{implementation_artifacts}`)
- Skills that change sprint or story state **update `sprint-status.yaml`**
- The folder structure is the **source of truth**; `sprint-status.yaml` is the tracking file maintained by skills

This file fixes the skills that:
1. Look for `sprint-status.yaml` at the root of `{implementation_artifacts}` (old location)
2. Create/read story files at the root of `{implementation_artifacts}` (instead of sprint folders)
3. Do not create sprint folders during sprint planning

## Application rules

- Apply each change in **both copies** of the file: `.claude/skills/…` AND `.agents/skills/…`
- Each change is **idempotent**: verify the guard is absent before replacing
- Only modify the indicated blocks — do not touch the rest of the file
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

**Guard:** if the file already contains `sprints_root` → `[SKIP]` (all replacements are considered applied)

---

### Replacement A — paths: sprint_status moved under sprints/ + add sprints_root (line ~70)

Search:
```
- `sprint_status` = `{implementation_artifacts}/sprint-status.yaml`
```

Replace with:
```
- `sprints_root` = `{implementation_artifacts}/sprints`
- `sprint_status` = `{implementation_artifacts}/sprints/sprint-status.yaml`
```

---

### Replacement B — default_output_file: story placed in the active sprint folder (line ~76)

Search:
```
- `default_output_file` = `{implementation_artifacts}/{{story_key}}.md`
```

Replace with:
```
- `default_output_file` = `{implementation_artifacts}/sprints/{{active_sprint_folder}}/{{story_key}}.md`
```

---

### Replacement C — Step 1: active sprint discovery via folder scan (lines ~98–128)

Search:
```
  <action>Check if {{sprint_status}} file exists for auto discover</action>
  <check if="sprint status file does NOT exist">
    <output>🚫 No sprint status file found and no story specified</output>
    <output>
      **Required Options:**
      1. Run `sprint-planning` to initialize sprint tracking (recommended)
      2. Provide specific epic-story number to create (e.g., "1-2-user-auth")
      3. Provide path to story documents if sprint status doesn't exist yet
    </output>
    <ask>Choose option [1], provide epic-story number, path to story docs, or [q] to quit:</ask>

    <check if="user chooses 'q'">
      <action>HALT - No work needed</action>
    </check>

    <check if="user chooses '1'">
      <output>Run sprint-planning workflow first to create sprint-status.yaml</output>
      <action>HALT - User needs to run sprint-planning</action>
    </check>

    <check if="user provides epic-story number">
      <action>Parse user input: extract epic_num, story_num, story_title</action>
      <action>Set {{epic_num}}, {{story_num}}, {{story_key}} from user input</action>
      <action>GOTO step 2a</action>
    </check>

    <check if="user provides story docs path">
      <action>Use user-provided path for story documents</action>
      <action>GOTO step 2a</action>
    </check>
  </check>
```

Replace with:
```
  <action>Discover active sprint: list all `sprint-{N}-{status}` directories under {{sprints_root}}</action>
  <action>Set {{active_sprint_folder}} = first directory with suffix `-started` (fallback: first with suffix `-planned`, sorted by N ascending)</action>
  <check if="no sprint folder found AND {{sprint_status}} does not exist">
    <output>🚫 No sprint folder and no sprint-status.yaml found under {{sprints_root}}</output>
    <output>
      **Required Options:**
      1. Run `sprint-planning` to create the sprint folder structure (recommended)
      2. Provide specific epic-story number to create (e.g., "1-2-user-auth")
    </output>
    <ask>Choose option [1], provide epic-story number, or [q] to quit:</ask>

    <check if="user chooses 'q'">
      <action>HALT - No work needed</action>
    </check>

    <check if="user chooses '1'">
      <output>Run sprint-planning workflow first to create the sprint folder structure.</output>
      <action>HALT - User needs to run sprint-planning</action>
    </check>

    <check if="user provides epic-story number">
      <action>Parse user input: extract epic_num, story_num, story_title</action>
      <action>Set {{epic_num}}, {{story_num}}, {{story_key}} from user input</action>
      <action>GOTO step 2a</action>
    </check>
  </check>
  <check if="no sprint folder found AND {{sprint_status}} exists">
    <action>Fallback: load {{sprint_status}} for auto-discovery (legacy path)</action>
  </check>
```

---

### Replacement D — Step 1 auto-discover: read stories from sprint folder (lines ~130–141)

Search:
```
  <!-- Auto-discover from sprint status only if no user input -->
  <check if="no user input provided">
    <critical>MUST read COMPLETE {sprint_status} file from start to end to preserve order</critical>
    <action>Load the FULL file: {{sprint_status}}</action>
    <action>Read ALL lines from beginning to end - do not skip any content</action>
    <action>Parse the development_status section completely</action>

    <action>Find the FIRST story (by reading in order from top to bottom) where:
      - Key matches pattern: number-number-name (e.g., "1-2-user-auth")
      - NOT an epic key (epic-X) or retrospective (epic-X-retrospective)
      - Status value equals "backlog"
    </action>
```

Replace with:
```
  <!-- Auto-discover next story: sprint folder takes priority, sprint-status.yaml as fallback -->
  <check if="no user input provided">
    <check if="{{active_sprint_folder}} is set">
      <action>List all .md files in {implementation_artifacts}/sprints/{{active_sprint_folder}}/</action>
      <action>For each story file, read its `status:` frontmatter field</action>
      <action>Find the FIRST story file (sorted by name) where status is "backlog"</action>
      <action>If no backlog story file found in folder, fallback to reading {{sprint_status}} for backlog entries</action>
    </check>
    <check if="{{active_sprint_folder}} is NOT set">
      <critical>MUST read COMPLETE {sprint_status} file from start to end to preserve order</critical>
      <action>Load the FULL file: {{sprint_status}}</action>
      <action>Read ALL lines from beginning to end - do not skip any content</action>
      <action>Parse the development_status section completely</action>
      <action>Find the FIRST story (by reading in order from top to bottom) where:
        - Key matches pattern: number-number-name (e.g., "1-2-user-auth")
        - NOT an epic key (epic-X) or retrospective (epic-X-retrospective)
        - Status value equals "backlog"
      </action>
    </check>
```

---

### Replacement E — "no backlog" message, first occurrence (line ~144)

Search:
```
    <check if="no backlog story found">
      <output>📋 No backlog stories found in sprint-status.yaml
```

Replace with:
```
    <check if="no backlog story found">
      <output>📋 No backlog story found in active sprint ({{active_sprint_folder}} / sprint-status.yaml)
```

---

### Replacement F — sprint-status.yaml messages in epic checks (line ~114)

Search:
```
      <output>Run sprint-planning workflow first to create sprint-status.yaml</output>
```

Replace with:
```
      <output>Run sprint-planning workflow first to create the sprint folder structure and sprint-status.yaml.</output>
```

---

### Replacement G — "no backlog" message, second occurrence (line ~201)

Search:
```
  <check if="no backlog story found">
    <output>No backlog stories found in sprint-status.yaml
```

Replace with:
```
  <check if="no backlog story found">
    <output>No backlog story found in active sprint ({{active_sprint_folder}} / sprint-status.yaml)
```

---

### Replacement H — epic correction message pointing to sprint-status.yaml, first block (line ~175)

Search:
```
        <output>1. Manually change epic status back to 'in-progress' in sprint-status.yaml</output>
```

Replace with:
```
        <output>1. Manually change epic status back to 'in-progress' in {implementation_artifacts}/sprints/sprint-status.yaml</output>
```

---

### Replacement I — epic correction message pointing to sprint-status.yaml, second block (line ~232)

Search:
```
      <output>1. Manually change epic status back to 'in-progress' in sprint-status.yaml</output>
```

Replace with:
```
      <output>1. Manually change epic status back to 'in-progress' in {implementation_artifacts}/sprints/sprint-status.yaml</output>
```

---

### Replacement J — Step 6: sprint-status update + story path confirmation (lines ~399–407)

Search:
```
  <!-- Update sprint status -->
  <check if="sprint status file exists">
    <action>Update {{sprint_status}}</action>
    <action>Load the FULL file and read all development_status entries</action>
    <action>Find development_status key matching {{story_key}}</action>
    <action>Verify current status is "backlog" (expected previous state)</action>
    <action>Update development_status[{{story_key}}] = "ready-for-dev"</action>
    <action>Update last_updated field to current date</action>
    <action>Save file, preserving ALL comments and structure including STATUS DEFINITIONS</action>
  </check>
```

Replace with:
```
  <!-- Update sprint status — file lives under sprints/ -->
  <action>Confirm story file saved to: {implementation_artifacts}/sprints/{{active_sprint_folder}}/{{story_key}}.md</action>
  <check if="sprint status file exists at {{sprint_status}}">
    <action>Update {{sprint_status}}</action>
    <action>Load the FULL file and read all development_status entries</action>
    <action>Find development_status key matching {{story_key}}</action>
    <action>Verify current status is "backlog" (expected previous state)</action>
    <action>Update development_status[{{story_key}}] = "ready-for-dev"</action>
    <action>Update last_updated field to current date</action>
    <action>Save file, preserving ALL comments and structure including STATUS DEFINITIONS</action>
  </check>
```

---

## File 2 — `bmad-sprint-planning/SKILL.md`

**Paths:**
- `.claude/skills/bmad-sprint-planning/SKILL.md`
- `.agents/skills/bmad-sprint-planning/SKILL.md`

**Guard:** if the file already contains `sprints_root` → `[SKIP]` (all replacements are considered applied)

---

### Replacement A — Goal + Role: include sprint folder creation (lines ~8–10)

Search:
```
**Goal:** Generate sprint status tracking from epics, detecting current story statuses and building a complete sprint-status.yaml file.

**Your Role:** You are a Developer generating and maintaining sprint tracking. Parse epic files, detect story statuses, and produce a structured sprint-status.yaml.
```

Replace with:
```
**Goal:** Generate sprint folder structure from epics and produce `sprints/sprint-status.yaml`. Each sprint becomes a `sprints/sprint-{N}-{status}/` folder; stories are `.md` files inside.

**Your Role:** You are a Developer generating and maintaining sprint tracking. Parse epic files, detect story statuses, create sprint folder structures, and maintain `sprint-status.yaml` under `sprints/`.
```

---

### Replacement B — paths: status_file moved under sprints/ + add sprints_root (line ~72)

Search:
```
- `status_file` = `{implementation_artifacts}/sprint-status.yaml`
```

Replace with:
```
- `sprints_root` = `{implementation_artifacts}/sprints`
- `status_file` = `{implementation_artifacts}/sprints/sprint-status.yaml`
```

---

### Replacement C — Step 2: build YAML structure → build folder + YAML structure (lines ~123–140)

Search:
```
<step n="2" goal="Build sprint status structure">
<action>For each epic found, create entries in this order:</action>

1. **Epic entry** - Key: `epic-{num}`, Default status: `backlog`
2. **Story entries** - Key: `{epic}-{story}-{title}`, Default status: `backlog`
3. **Retrospective entry** - Key: `epic-{num}-retrospective`, Default status: `optional`

**Example structure:**

```yaml
development_status:
  epic-1: backlog
  1-1-user-authentication: backlog
  1-2-account-management: backlog
  epic-1-retrospective: optional
```

</step>
```

Replace with:
```
<step n="2" goal="Build sprint structure (folders + YAML entries)">
<action>Group all stories into sprints (default: one sprint per epic, or N stories per sprint if configured).</action>
<action>For each sprint N, determine initial status: `planned` (no sprint in progress yet).</action>

**Sprint folders to create:**
- `{sprints_root}/sprint-{N}-planned/` — one folder per sprint

**Story files to create in each folder:**
- `{story-key}.md` — stub with `status: backlog` in frontmatter

**Corresponding YAML entries in `sprint-status.yaml`:**

1. **Epic entry** - Key: `epic-{num}`, Default status: `backlog`
2. **Story entries** - Key: `{epic}-{story}-{title}`, Default status: `backlog`
3. **Retrospective entry** - Key: `epic-{num}-retrospective`, Default status: `optional`

**Example structure:**
```
sprints/
├── sprint-status.yaml
├── sprint-1-planned/
│   ├── 1-1-user-authentication.md   (status: backlog)
│   └── 1-2-account-management.md   (status: backlog)
└── sprint-2-planned/
    └── 2-1-dashboard.md             (status: backlog)
```

</step>
```

---

### Replacement D — Step 3: story file detection updated for sprint folder (lines ~145–153)

Search:
```
**Story file detection:**

- Check: `{story_location_absolute}/{story-key}.md` (e.g., `stories/1-1-user-authentication.md`)
- If exists → upgrade status to at least `ready-for-dev`

**Preservation rule:**

- If existing `{status_file}` exists and has more advanced status, preserve it
- Never downgrade status (e.g., don't change `done` to `ready-for-dev`)
```

Replace with:
```
**Story file detection:**

- Check: `{sprints_root}/sprint-{N}-{status}/{story-key}.md`
- If story file exists in a sprint folder → upgrade status to at least `ready-for-dev`
- Sprint status from folder suffix: `planned` → backlog, `started` → in-progress/ready-for-dev, `ended` → done

**Preservation rule:**

- If a story already has a more advanced status in `{status_file}` or in its sprint folder, preserve it
- Never downgrade status (e.g., don't change `done` to `ready-for-dev`)
- Sprint folder suffix takes precedence for sprint-level status
```

---

### Replacement E — Step 4: Generate sprint status file → Create folders + YAML (lines ~162–163)

Search:
```
<step n="4" goal="Generate sprint status file">
<action>Create or update {status_file} with:</action>
```

Replace with:
```
<step n="4" goal="Create sprint folders and generate sprint-status.yaml">
<action>For each sprint N, create directory `{sprints_root}/sprint-{N}-planned/` if it does not already exist</action>
<action>For each story assigned to sprint N, create stub file `{sprints_root}/sprint-{N}-planned/{story-key}.md` if it does not already exist (stub must contain at minimum `status: backlog` in frontmatter)</action>
<action>Create or update {status_file} with:</action>
```

---

### Replacement F — Step 5: validation checklist (lines ~223–228)

Search:
```
- [ ] Every epic in epic files appears in {status_file}
- [ ] Every story in epic files appears in {status_file}
- [ ] Every epic has a corresponding retrospective entry
- [ ] No items in {status_file} that don't exist in epic files
- [ ] All status values are legal (match state machine definitions)
- [ ] File is valid YAML syntax
```

Replace with:
```
- [ ] Every epic in epic files appears in {status_file}
- [ ] Every story in epic files appears in {status_file}
- [ ] Every epic has a corresponding retrospective entry
- [ ] No items in {status_file} that don't exist in epic files
- [ ] All status values are legal (match state machine definitions)
- [ ] File is valid YAML syntax
- [ ] Every sprint N has a corresponding `{sprints_root}/sprint-{N}-{status}/` folder
- [ ] Every story in {status_file} has a stub `.md` file in its sprint folder
```

---

### Replacement G — Step 5: File Location in summary (line ~241)

Search:
```
- **File Location:** {status_file}
```

Replace with:
```
- **File Location:** {status_file}
- **Sprint Root:** {sprints_root}/
```

---

## File 3 — `bmad-sprint-status/SKILL.md`

**Paths:**
- `.claude/skills/bmad-sprint-status/SKILL.md`
- `.agents/skills/bmad-sprint-status/SKILL.md`

**Guard:** if the file already contains `sprints_root` → `[SKIP]` (all replacements are considered applied)

---

### Replacement A — sprint_status_file moved under sprints/ + add sprints_root (line ~64)

Search:
```
- `sprint_status_file` = `{implementation_artifacts}/sprint-status.yaml`
```

Replace with:
```
- `sprints_root` = `{implementation_artifacts}/sprints`
- `sprint_status_file` = `{implementation_artifacts}/sprints/sprint-status.yaml`
```

---

### Replacement B — Step 1: file location enriched with folder scan (lines ~92–101)

Search:
```
<step n="1" goal="Locate sprint status file">
  <action>Load {project_context} for project-wide patterns and conventions (if exists)</action>
  <action>Try {sprint_status_file}</action>
  <check if="file not found">
    <output>sprint-status.yaml not found.
Run `/bmad:bmm:workflows:sprint-planning` to generate it, then rerun sprint-status.</output>
    <action>Exit workflow</action>
  </check>
  <action>Continue to Step 2</action>
</step>
```

Replace with:
```
<step n="1" goal="Locate sprint status file and sprint folders">
  <action>Load {project_context} for project-wide patterns and conventions (if exists)</action>
  <action>Try {sprint_status_file}</action>
  <action>List all `sprint-{N}-{status}` directories under {sprints_root}</action>
  <action>Set {{active_sprint_folder}} = first folder with suffix `-started` (fallback: first with suffix `-planned`, sorted by N ascending)</action>
  <check if="sprint_status_file not found AND no sprint folder found">
    <output>No sprint-status.yaml and no sprint folders found under {sprints_root}.
Run `sprint-planning` to generate the sprint structure, then rerun sprint-status.</output>
    <action>Exit workflow</action>
  </check>
  <action>Continue to Step 2</action>
</step>
```

---

### Replacement C — Step 2: YAML reading enriched with folder scan (lines ~103–104)

Search:
```
<step n="2" goal="Read and parse sprint-status.yaml">
  <action>Read the FULL file: {sprint_status_file}</action>
```

Replace with:
```
<step n="2" goal="Read and parse sprint-status.yaml and sprint folders">
  <action>Read the FULL file: {sprint_status_file} (if present)</action>
  <action>For each `sprint-{N}-{status}` folder under {sprints_root}: list story .md files and read their `status:` frontmatter field</action>
  <action>Reconcile: if a story's folder-derived status is more advanced than the YAML value, use the folder value (source of truth = folders)</action>
```

---

### Replacement D — sprint-status.yaml staleness check (line ~153)

Search:
```
- IF `last_updated` timestamp is more than 7 days old (or `last_updated` is missing, fall back to `generated`): warn "sprint-status.yaml may be stale"
```

Replace with:
```
- IF `last_updated` timestamp is more than 7 days old (or `last_updated` is missing, fall back to `generated`): warn "sprint-status.yaml may be stale — verify against sprint folders"
- IF active sprint folder has suffix `-started` AND all story files have status `done`: recommend running `retrospective` then renaming the folder to `sprint-{N}-ended`
```

---

### Replacement E — Step 4: Status file → Sprint root + Status file (line ~176)

Search:
```
- Status file: {sprint_status_file}
```

Replace with:
```
- Status file: {sprint_status_file}
- Sprint root: {sprints_root}/
- Active sprint: {{active_sprint_folder}}
```

---

### Replacement F — option 3 "Show raw sprint-status.yaml" (line ~199)

Search:
```
3) Show raw sprint-status.yaml
```

Replace with:
```
3) Show sprint-status.yaml and sprint folder listing
```

---

### Replacement G — Step 5, choice == 3 action (lines ~219–221)

Search:
```
  <check if="choice == 3">
    <action>Display the full contents of {sprint_status_file}</action>
  </check>
```

Replace with:
```
  <check if="choice == 3">
    <action>Display the full contents of {sprint_status_file}</action>
    <action>List all sprint folders under {sprints_root} with their story files and status values</action>
  </check>
```

---

### Replacement H — Step 30: Validate sprint-status.yaml and folder structure (lines ~254–294)

Search:
```
<step n="30" goal="Validate sprint-status file">
  <action>Check that {sprint_status_file} exists</action>
  <check if="missing">
    <template-output>is_valid = false</template-output>
    <template-output>error = "sprint-status.yaml missing"</template-output>
    <template-output>suggestion = "Run sprint-planning to create it"</template-output>
    <action>Return</action>
  </check>

<action>Read and parse {sprint_status_file}</action>

<action>Validate required metadata fields exist: generated, project, project_key, tracking_system, story_location (last_updated is optional for backward compatibility)</action>
<check if="any required field missing">
<template-output>is_valid = false</template-output>
<template-output>error = "Missing required field(s): {{missing_fields}}"</template-output>
<template-output>suggestion = "Re-run sprint-planning or add missing fields manually"</template-output>
<action>Return</action>
</check>

<action>Verify development_status section exists with at least one entry</action>
<check if="development_status missing or empty">
<template-output>is_valid = false</template-output>
<template-output>error = "development_status missing or empty"</template-output>
<template-output>suggestion = "Re-run sprint-planning or repair the file manually"</template-output>
<action>Return</action>
</check>

<action>Validate all status values against known valid statuses:</action>

- Stories: backlog, ready-for-dev, in-progress, review, done (legacy: drafted)
- Epics: backlog, in-progress, done (legacy: contexted)
- Retrospectives: optional, done
  <check if="any invalid status found">
  <template-output>is_valid = false</template-output>
  <template-output>error = "Invalid status values: {{invalid_entries}}"</template-output>
  <template-output>suggestion = "Fix invalid statuses in sprint-status.yaml"</template-output>
  <action>Return</action>
  </check>

<template-output>is_valid = true</template-output>
<template-output>message = "sprint-status.yaml valid: metadata complete, all statuses recognized"</template-output>
```

Replace with:
```
<step n="30" goal="Validate sprint-status.yaml and sprint folder structure">
  <action>Check that {sprint_status_file} exists at {sprints_root}/sprint-status.yaml</action>
  <check if="missing">
    <template-output>is_valid = false</template-output>
    <template-output>error = "sprint-status.yaml missing under sprints/"</template-output>
    <template-output>suggestion = "Run sprint-planning to create it"</template-output>
    <action>Return</action>
  </check>

<action>Read and parse {sprint_status_file}</action>

<action>Validate required metadata fields exist: generated, project, project_key, tracking_system, story_location (last_updated is optional for backward compatibility)</action>
<check if="any required field missing">
<template-output>is_valid = false</template-output>
<template-output>error = "Missing required field(s): {{missing_fields}}"</template-output>
<template-output>suggestion = "Re-run sprint-planning or add missing fields manually"</template-output>
<action>Return</action>
</check>

<action>Verify development_status section exists with at least one entry</action>
<check if="development_status missing or empty">
<template-output>is_valid = false</template-output>
<template-output>error = "development_status missing or empty"</template-output>
<template-output>suggestion = "Re-run sprint-planning or repair the file manually"</template-output>
<action>Return</action>
</check>

<action>Validate all status values against known valid statuses:</action>

- Stories: backlog, ready-for-dev, in-progress, review, done (legacy: drafted)
- Epics: backlog, in-progress, done (legacy: contexted)
- Retrospectives: optional, done
  <check if="any invalid status found">
  <template-output>is_valid = false</template-output>
  <template-output>error = "Invalid status values: {{invalid_entries}}"</template-output>
  <template-output>suggestion = "Fix invalid statuses in sprint-status.yaml"</template-output>
  <action>Return</action>
  </check>

<action>Validate sprint folder structure under {sprints_root}:</action>
<action>Each sprint folder must match `sprint-{N}-{status}` where status ∈ {planned,started,ended,skipped}</action>
<check if="any folder has invalid name">
<template-output>is_valid = false</template-output>
<template-output>error = "Invalid sprint folder name(s): {{invalid_folders}}"</template-output>
<template-output>suggestion = "Rename folders to follow sprint-{N}-{status} convention"</template-output>
<action>Return</action>
</check>

<action>Cross-check: every story in development_status should have a corresponding .md file in its sprint folder</action>
<check if="stories in YAML have no corresponding folder file">
<template-output>is_valid = false</template-output>
<template-output>error = "YAML/folder mismatch: {{mismatched_stories}}"</template-output>
<template-output>suggestion = "Re-run sprint-planning or create missing story stub files"</template-output>
<action>Return</action>
</check>

<template-output>is_valid = true</template-output>
<template-output>message = "sprint-status.yaml valid and sprint folder structure consistent"</template-output>
```
