# 🧩 Stage 71 – tests-audit
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 70 – static-consistency  
**Next:** 72 – functional-equivalence

---

## 🎯 Objective

Analyser la **couverture théorique** et la **cohérence structurelle** de la stratégie de tests générée pour `${project.pageName}`.  
Ce stage ne **lance** pas les tests : il inspecte statiquement les fichiers et vérifie que les UCR critiques sont correctement couvertes.

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `gates`, `stack.id`

- **Code généré (Phase 3)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`

- **Inventaires UCR & Mappings**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/*.json`
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/*.json`

- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.tests-audit.md` (guide principal pour ce domaine)

- **Stack-guides (optionnel mais recommandé)**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.tests.md`

---

## 📤 Outputs

Sous :  
```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/tests-audit/
```

- `tests-audit.report.json`
- `tests-audit.report.md`
- `.meta/validation.tests-audit.meta.json`

---

## 🧠 Actions

1. **Charger la configuration**
2. **Lister tous les fichiers de tests dans `src_new`**
   - Extensions : `.test.*`, `.spec.*`
   - Dossiers : `__tests__/`
3. **Identifier les UCR**
   - depuis inventaires & mappings
4. **Créer une matrice UCR ↔ tests**
   - Une UCR est *couverte* si un test mentionne :
     - son identifiant,
     - son scenario,
     - son composant associé.
5. **Calculer les métriques**
   - `ucrTotal`, `ucrCovered`, `ucrCritical`, `ucrCriticalCovered`
   - `coverageTheoretical`, `coverageCritical`
6. **Déterminer score & gate**
   - Gate ❌ si **une UCR critique n’est pas couverte**
7. **Générer rapport JSON + Markdown**
8. **Écrire `.meta/...meta.json`**

---

## ✅ Auto-Checks

```json
{
  "stageId": "71",
  "stageName": "tests-audit",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "ucrLoaded": true,
    "testFilesDetected": true,
    "coverageComputed": true,
    "reportsWritten": true
  }
}
```

---

## 🧩 Gate

Gate ✅

*(Gate ❌ si une UCR critique n’a pas de test associé.)*

---

## 📦 Next

> Continue with `72-functional-equivalence.md` if Gate ✅.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
