# 🧩 Stage 72 – functional-equivalence
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 71 – tests-audit  
**Next:** 73 – dependencies-coherence

---

## 🎯 Objective

Établir une **matrice déclarative d’équivalence fonctionnelle** entre le legacy et le code généré pour `${project.pageName}`.  
Ce stage vérifie si chaque UCR / behaviour identifié en Phase 1 & 2 possède une implémentation plausible dans la stack cible (Phase 3).

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `gates`

- **Inventaires UCR (Phase 1)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-1-analysis/inventories/*.json`

- **Mappings UCR (Phase 2)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-2-interpretation/mappings/*.json`

- **Code généré (Phase 3)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`

- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.functional-equivalence.md` (guide principal pour ce domaine)

- **Stack-guides** (optionnel)  
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.validation.md`

---

## 📤 Outputs

Dans :  
```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/functional-equivalence/
```

- `equivalence-matrix.json`  
- `equivalence-matrix.report.md`  
- `regressions-detected.json`  
- `.meta/validation.functional-equivalence.meta.json`  

---

## 🧠 Actions

1. **Charger la configuration et les UCR**  
2. **Rassembler behaviours / UCR depuis inventaires & mappings**  
3. **Analyser le code pour trouver les implémentations plausibles**  
   - Recherche par nom, logique associée, handlers, fichiers correspondants  
4. **Établir la matrice UCR → implémentation**  
   - coverage = `covered | partiallyCovered | notCovered | unknown`
5. **Identifier les régressions**  
   - UCR critiques en `notCovered` ⇒ régression critique  
6. **Calculer le score global**  
7. **Générer :**  
   - JSON détaillé  
   - Markdown lisible  
   - Fichier `.meta`  
8. **Déterminer Gate**  
   - Gate ❌ si une UCR critique n’est pas couverte  

---

## ✅ Auto-Checks

```json
{
  "stageId": "72",
  "stageName": "functional-equivalence",
  "pageName": "${project.pageName}",
  "checks": {
    "inputsAvailable": true,
    "ucrLoaded": true,
    "codeLoaded": true,
    "matrixBuilt": true,
    "reportsWritten": true
  }
}
```

---

## 🧩 Gate

Gate ✅  
*(Gate ❌ si une UCR critique est en `notCovered`.)*

---

## 📦 Next

> Continue with `73-dependencies-coherence.md` if Gate ✅`.

---
© 2025 Sylvain Chabaud — ai-orchestrator-v4
