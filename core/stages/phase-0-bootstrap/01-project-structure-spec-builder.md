# 🧩 Stage 01 – Project Structure Spec Builder
**Phase:** Phase 0 – Bootstrap  
**Prev:** 00 – Stack Guides Builder  
**Next:** 10 – Inventory Structure Builder

---

## 🎯 Objective

Generate the **project structure specification** for `${project.pageName}`, defining all folders, file paths, naming conventions, and architectural rules required by the orchestrator.  
This stage transforms the output of Stage 00 (stack guides + summary) into a **formal JSON structure** that every subsequent stage (inventories, mappings, generation, validation) will rely upon.

The goal is to produce a single authoritative specification file describing:

- where the orchestrator must write generated code,
- how pages, components, hooks, logic, tests, routes, and locales must be organized,
- naming conventions derived from the stack guides and contracts,
- constraints imposed by the stack (React 19, DS Peaksys, Zustand, React Query, i18n, routing, tests…).

---

## ⚙️ Inputs

> All inputs MUST be resolved from `core/configs/project.config.yaml`.  
> Never hard‑code paths.

### Configuration
- `core/configs/project.config.yaml`  
  Keys used:
  - `project.pageName`
  - `project.name`
  - `paths.core`
  - `paths.workspace`
  - `stack.custom`

### Stack guides (generated during Stage 00)
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/*.md`
- `${paths.workspace}/projects/${project.name}/stack/stack-guides/stack-guides-summary.json`

### No legacy code is used here
- ❌ `${paths.legacySource}` is **not** used in this stage.

---

## 📤 Outputs

All outputs must be written under:

```
${paths.workspace}/projects/${project.name}/stack/
```

This stage MUST generate:

### 1. Project structure specification JSON  
`project-structure.json`

It MUST contain at least:

```json
{
  "projectRoot": "src/",
  "pages": {
    "${project.pageName}": {
      "root": "src/pages/${project.pageName}",
      "components": "src/pages/${project.pageName}/components",
      "hooks": "src/pages/${project.pageName}/hooks",
      "logic": "src/pages/${project.pageName}/logic",
      "tests": "src/pages/${project.pageName}/__tests__",
      "i18n": "src/locales",
      "routes": "src/routes"
    }
  },
  "naming": {
    "component": "PascalCase",
    "hook": "useCamelCase",
    "file": "kebab-case",
    "test": "${ComponentName}.test.tsx"
  },
  "generationRules": {
    "mustCreateFolders": true,
    "cleanEmptyFolders": false
  }
}
```

(Values will depend on the stack guides.)

---

## 🧠 Actions

1. **Load configuration**
   - Read project name, page name, workspace path.
   - Resolve path of the stack guides directory.

2. **Load stack guides summary**
   - Read `stack-guides-summary.json`.
   - Identify:
     - routing conventions,
     - component conventions,
     - i18n conventions,
     - testing strategy,
     - folder layout rules.

3. **Parse relevant guides**
   - Extract structure rules from:
     - `guide.structure.md`
     - `guide.ui-components.md`
     - `guide.routing.md`
     - `guide.tests.md`
     - `guide.i18n.md`
     - any guide mentioning folder or file structure.
   - Normalize conflicting rules (if any).

4. **Build internal project structure model**
   - Derive:
     - root folder,
     - page‑specific folder tree,
     - components folder convention,
     - naming conventions,
     - rules for i18n, logic, hooks, routes, tests,
     - code generation targets required for Phase 3.

5. **Generate `project-structure.json`**
   - Write a complete JSON description of the target layout.
   - Include:
     - folders,
     - file naming conventions,
     - rules for future automatic code generation,
     - domain mapping (components, hooks, tests, logic, routes).

6. **Generate `project-structure-checklist.md`**
   - Provide human‑readable summary of all rules.
   - Must match the JSON spec exactly.

7. **Internal consistency checks**
   - Ensure all required folders are defined.
   - Ensure naming conventions exist and are valid.
   - Ensure no contradictions with stack guides.
   - Ensure the JSON schema is valid (if schema exists).

---

## ✅ Auto-Checks

```json
{
  "stageId": "01",
  "stageName": "Project Structure Spec Builder",
  "pageName": "${project.pageName}",
  "checks": {
    "stackGuidesLoaded": true,
    "summaryReadable": true,
    "jsonGenerated": true,
    "checklistGenerated": true,
    "structureConsistent": true
  },
  "notes": [
    "project-structure.json generated successfully",
    "naming conventions extracted from stack guides"
  ]
}
```

---

## 🧩 Gate

```markdown
## 🧩 Gate
Gate ✅
```

OR

```markdown
## 🧩 Gate
Gate ❌
```

Use **Gate ❌** if:
- stack guides missing or unreadable,
- no naming conventions can be derived,
- JSON invalid or incomplete,
- contradictions detected.

---

## 📦 Next

> Continue with `10-inventory.structure.md` if `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
