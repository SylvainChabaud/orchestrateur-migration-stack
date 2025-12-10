# 🧩 Stage 77 – quality-compliance
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 76 – performance-patterns  
**Next:** 78 – validation-summary

---

## 🎯 Objective

Valider que le code généré pour `${project.pageName}` respecte les **standards internes**, les **conventions de la stack cible**, et les **patterns globaux** définis dans les guides.

Il s’agit d’une validation **statique**, **heuristique**, inspirée d’un audit "lint-like".

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `paths.core`, `stack.id`

- **Code généré**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`

- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.quality-compliance.md` (guide principal pour ce domaine)
  - `${paths.core}/guides-internals/globals/*.md`  
    (naming, structure, conventions internes)

- **Stack-guides**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/*.md`

---

## 📤 Outputs

Répertoire :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/quality-compliance/
```

Fichiers générés :

- `quality-compliance.report.json`
- `quality-compliance.report.md`
- `.meta/validation.quality-compliance.meta.json`

---

## 🧠 Actions

1. **Charger la configuration & les guides**
2. **Scanner le code**
   - composants, hooks, effets, noms de fichiers, structures
3. **Analyser la conformité au naming**
   - composant ≠ PascalCase → warning
   - hook ≠ `useXxx` → warning
4. **Détecter patterns interdits**
   - usage de `any` si interdit
   - accès direct au DOM si stack orientée declarative
5. **Vérifier les patterns obligatoires**
   - utilisation de handlers standards (`onXxx`, `handleXxx`)
6. **Vérifier la structure des dossiers**
   - conformité à `project-structure.json` si présent
7. **Construire les métriques & score**
8. **Déterminer le Gate**
   - Gate ❌ si patterns interdits détectés OU score < minScore
9. **Générer JSON, MD et meta**

---

## ✅ Auto-Checks

```json
{
  "stageId": "77",
  "stageName": "quality-compliance",
  "pageName": "${project.pageName}",
  "checks": {
    "guidesLoaded": true,
    "codeScanned": true,
    "issuesDetected": true,
    "scoreComputed": true,
    "reportsWritten": true
  }
}
```

---

## 🧩 Gate

Gate ✅  
*(Gate ❌ si patterns interdits détectés ou score insuffisant.)*

---

## 📦 Next

> Continue with `78-validation-summary.md` if Gate ✅.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
