# 🧩 Stage ${stageId} – ${stageName}
**Phase:** ${phaseLabel}  
**Prev:** ${prevStageId} – ${prevStageName}  
**Next:** ${nextStageId} – ${nextStageName}

---

## 🎯 Objective
Explain in 2–3 sentences what this stage must achieve for page `${project.pageName}`.

- Be specific, but high-level.
- Mention if this stage works on **inventories**, **mappings**, **stack guides**, **generated code**, etc.

Example:
> Build the `${domain}` inventory for `${project.pageName}` by analyzing the legacy source at `${paths.legacySource}`, following the internal inventory guides and the UCR rules.

---

## ⚙️ Inputs

> **All inputs MUST be resolved from `core/configs/project.config.yaml`.  
> Never hard-code absolute paths.**  

List only what this stage really needs.

- **Configuration**
  - `core/configs/project.config.yaml`
    - keys used: `project.pageName`, `paths.*`, `stack.id`, `gates`, `ai`, `stages`

- **Legacy code (read-only)**
  - `${paths.legacySource}`
    - The legacy file must be read **in place** in the repo.
    - ❌ Do NOT copy it into `workspace/`.

- **Core guides (read-only, from `core/guides-internals`)**
  - `${paths.core}/guides-internals/inventory/guide.inventory.${domain}.md` (if relevant)
  - `${paths.core}/guides-internals/mapping/guide.mapping.${domain}.md` (if relevant)
  - `${paths.core}/guides-internals/globals/guide.ucr.md` (if relevant)

- **Stack guides (generated in Phase 0, read-only)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.${something}.md` (if relevant)

- **Previous stage outputs (from workspace)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/...`
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/...`

---

## 📤 Outputs

> List exactly what this stage MUST produce or update in the workspace.
> All outputs live under `${paths.workspace}`.

Examples:

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/inventory.${domain}.json`
- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/mapping.${domain}.json`
- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/...`
- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/...`

Rules:

- Outputs MUST respect the corresponding JSON schema in `core/schemas/*.schema.json`.
- Do not write anything into `core/`.
- Do not overwrite files from other phases unless responsible for them.

---

## 🧠 Actions

Describe the **ordered steps** the IA must follow:

1. **Load configuration and context**
2. **Analyze or Transform** inputs
3. **Generate / Update** outputs
4. **Run internal consistency checks**

Adapt these steps depending on the stage type (inventory, mapping, generation, validation…).

---

## ✅ Auto-Checks

Produce a short JSON summary:

```json
{
  "stageId": "${stageId}",
  "stageName": "${stageName}",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "schemaValidated": true,
    "outputsWritten": true
  }
}
```

This summary is **not written to disk**, only shown in the AI response.

---

## 🧩 Gate

Write exactly one of:

```markdown
## 🧩 Gate
Gate ✅
```

or

```markdown
## 🧩 Gate
Gate ❌
```

Use `Gate ❌` if any blocking error occurs.

---

## 📦 Next
> Continue with `${nextStageId}-${nextStageFileName}.md` if `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
