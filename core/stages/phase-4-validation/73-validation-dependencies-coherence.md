# 🧩 Stage 73 – dependencies-coherence
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 72 – functional-equivalence  
**Next:** 74 – integration-consistency

---

## 🎯 Objective

Analyser la **cohérence des dépendances** pour `${project.pageName}` :  
imports externes, modules inattendus, imports non résolus.  
Ce stage ne réalise pas de build, il se base uniquement sur une analyse statique.

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `stack.id`, `gates`

- **Code généré (Phase 3)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`

- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.dependencies-coherence.md` (guide principal pour ce domaine)

- **Stack-guides**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.validation.md`
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.performance.md`
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tests.md`

---

## 📤 Outputs

Dans :  
```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/dependencies-coherence/
```

- `dependencies-audit.report.json`
- `dependencies-audit.report.md`
- `.meta/validation.dependencies-coherence.meta.json`

---

## 🧠 Actions

1. **Charger la configuration**  
2. **Charger les stack-guides pour récupérer la liste des dépendances attendues**  
3. **Scanner tout le code de `src_new`**
   - Extraire imports externes
   - Détecter imports non résolus
4. **Déterminer les dépendances "inattendues"**
   - Non listées dans la stack cible  
5. **Construire les métriques**
   - totalImports  
   - resolved / unresolved  
   - unexpectedDependencies  
6. **Calculer le score**
7. **Déterminer Gate**
   - Gate ❌ si:
     - `unresolvedImports > 0`, ou
     - `score < minScore`
8. **Produire les fichiers JSON, MD et meta**

---

## ✅ Auto-Checks

```json
{
  "stageId": "73",
  "stageName": "dependencies-coherence",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "importsExtracted": true,
    "analysisDone": true,
    "reportsWritten": true
  }
}
```

---

## 🧩 Gate

Gate ✅  
*(Gate ❌ si import non résolu ou score insuffisant.)*

---

## 📦 Next

> Continue with `74-integration-consistency.md` if Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
