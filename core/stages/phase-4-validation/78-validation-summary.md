# 🧩 Stage 78 – validation-summary
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 77 – quality-compliance  
**Next:** (end of pipeline for Phase 4)

---

## 🎯 Objective

Agréger tous les résultats de la **Phase 4** pour `${project.pageName}`, calculer le **score global de qualité**, décider du **Gate global** de la phase, et produire :

- un **résumé synthétique** (JSON + Markdown),
- une **liste d’actions requises**,
- une **archive finale** des artefacts clés avec un `manifest.json`.

Ce stage ne refait aucune analyse du code :  
il se contente de **lire les rapports des stages 70 → 77** et de les consolider.

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `gates`, `stack.id`

- **Rapports Phase 4 (read-only)**  
  Tous sous :  
  `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/`

  - Static consistency (Stage 70)  
    - `static-consistency/static-consistency.report.json`
  - Tests audit (Stage 71)  
    - `tests-audit/tests-audit.report.json`
  - Functional equivalence (Stage 72)  
    - `functional-equivalence/equivalence-matrix.json`
  - Dependencies coherence (Stage 73)  
    - `dependencies-coherence/dependencies-audit.report.json`
  - Integration consistency (Stage 74)  
    - `integration-consistency/integration-consistency.report.json`
  - Accessibility heuristic (Stage 75)  
    - `accessibility-heuristic/accessibility-audit.report.json`
  - Performance patterns (Stage 76)  
    - `performance-patterns/performance-audit.report.json`
  - Quality compliance (Stage 77)  
    - `quality-compliance/quality-compliance.report.json`

- **Meta & thresholds (optionnel)**
  - `.meta/*.meta.json` des stages 70–77
  - Stack-guides de validation globales (`guide.validation.md`, `guide.quality-thresholds.md`)

---

## 📤 Outputs

Sous :  
`${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/`

1. **Résumé global**
   - `validation-summary.json`
   - `validation-summary.report.md`
   - `actions-required.md`

2. **Archive finale**  
   Dossier :  
   `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/archive/${timestamp}/`

   Contenant au minimum :
   - Copie des rapports JSON & MD des stages 70–78
   - Artefacts centraux des phases 0–3 (chemins à définir par la stack, ex.) :
     - `phase-0-bootstrap/*` (guides de stack, project-structure, bridge legacy-DSL, etc.)
     - `phase-1-analysis/inventories/*.json`
     - `phase-2-interpretation/mappings/*.json`
     - `phase-3-generation/src_new/**`
   - `manifest.json`

---

## 📦 Contract – validation-summary.json

```jsonc
{
  "pageName": "${project.pageName}",
  "projectName": "${project.name}",
  "timestamp": "2025-12-08T10:00:00.000Z",

  "scores": {
    "staticAnalysisScore": 0,
    "testsScore": 0,
    "functionalEquivalenceScore": 0,
    "dependenciesScore": 0,
    "integrationScore": 0,
    "accessibilityScore": 0,
    "performanceScore": 0,
    "qualityStandardsScore": 0,
    "globalScore": 0
  },

  "gates": {
    "stage70": "passed | failed",
    "stage71": "passed | failed",
    "stage72": "passed | failed",
    "stage73": "passed | failed",
    "stage74": "passed | failed",
    "stage75": "passed | failed",
    "stage76": "passed | failed",
    "stage77": "passed | failed",
    "phase4": "passed | failed"
  },

  "summary": {
    "status": "valid | warnings | rejected",
    "criticalIssues": [],
    "warnings": []
  },

  "actionsRequired": [
    {
      "priority": "high | medium | low",
      "category": "tests | functional | dependencies | integration | accessibility | performance | quality",
      "message": "Action à réaliser",
      "relatedStages": [70, 72]
    }
  ],

  "archive": {
    "path": "phase-4-validation/archive/2025-12-08T10-00-00Z/",
    "manifest": "phase-4-validation/archive/2025-12-08T10-00-00Z/manifest.json"
  }
}
```

---

## 🧾 Contract – validation-summary.report.md

```markdown
# ✅ Phase 4 – Validation Summary

**Projet :** ${project.name}  
**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Score global :** 87/100  
**Gate Phase 4 :** ✅ Passed  

---

## 📊 Scores par domaine

- Static consistency : 90/100 (Gate ✅)  
- Tests audit : 85/100 (Gate ✅)  
- Functional equivalence : 88/100 (Gate ✅)  
- Dependencies : 92/100 (Gate ✅)  
- Integration : 86/100 (Gate ✅)  
- Accessibility : 80/100 (Gate ✅)  
- Performance : 78/100 (Gate ⚠️)  
- Quality compliance : 89/100 (Gate ✅)  

---

## 🚨 Points critiques

_(Lister ici les issues critiques encore ouvertes)_  

---

## 📝 Actions requises

1. [High] Compléter la couverture de tests pour les UCR critiques manquantes.  
2. [Medium] Optimiser les listes volumineuses sur la page principale.  

---

## 🧭 Décision globale

> **Statut :** _Ready / Needs work / Experimental_  
> **Recommandation :** _ex : intégration possible sous réserve des actions ci-dessus_  

---

*Généré par ai-orchestrator-v4 — Stage 78 (validation-summary)*
```

---

## 📦 Contract – manifest.json

Structure minimale attendue dans l’archive :

```jsonc
{
  "project": "${project.name}",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "phase4Gate": "passed | failed",
  "files": [
    {
      "path": "phase-0-bootstrap/stack/stack-guides/guide.stack.md",
      "hash": "sha256:...",
      "phase": "0"
    },
    {
      "path": "phase-3-generation/src_new/index.tsx",
      "hash": "sha256:...",
      "phase": "3"
    },
    {
      "path": "phase-4-validation/static-consistency/static-consistency.report.json",
      "hash": "sha256:...",
      "phase": "4"
    }
  ]
}
```

---

## 🧠 Actions du stage

1. **Charger la configuration & les seuils globaux**
   - `project-config.yaml`
   - `guide.validation.md`, `guide.quality-thresholds.md` (poids des scores, seuil global, etc.).

2. **Lire les rapports JSON des stages 70 → 77**
   - Extraire :
     - scores locaux,
     - statuts de validation,
     - Gates,
     - listes d’issues.

3. **Calculer le score global**
   - Appliquer les pondérations définies dans les guides (exemple) :
     - Static = 0.15
     - Tests = 0.20
     - Functional equivalence = 0.25
     - Dependencies = 0.10
     - Integration = 0.15
     - Accessibility = 0.05
     - Performance = 0.05
     - Quality = 0.05

4. **Déterminer le Gate de la Phase 4**
   - Gate Phase 4 ❌ si :
     - un stage critique (70, 72, 73, 74) est `failed`, ou
     - `globalScore` < seuil global.

5. **Synthétiser les issues et actions requises**
   - Extraire les issues critiques/WARN importantes.
   - Regrouper par catégorie (tests, functional, deps, integration, etc.).
   - Générer une liste priorisée d’actions dans `actions-required.md`.

6. **Générer les fichiers de sortie**
   - `validation-summary.json`
   - `validation-summary.report.md`
   - `actions-required.md`

7. **Construire l’archive**
   - Créer un sous-dossier `archive/${timestamp}/`.
   - Copier ou référencer les fichiers clés (Phases 0–4).
   - Générer `manifest.json` avec :
     - liste des fichiers,
     - hash (ou placeholder si non calculable),
     - phase d’origine.

---

## ✅ Auto-Checks

Le stage doit produire (dans la réponse de l’agent, pas sur disque) un court résumé JSON :

```json
{
  "stageId": "78",
  "stageName": "validation-summary",
  "pageName": "${project.pageName}",
  "checks": {
    "reportsLoaded": true,
    "globalScoreComputed": true,
    "phaseGateComputed": true,
    "summaryWritten": true,
    "archiveCreated": true
  }
}
```

Si un des champs est `false`, le rapport Markdown doit l’expliquer clairement.

---

## 🧩 Gate

Gate ✅  
*(Mettre Gate ❌ si conditions globales non atteintes : stage critique en échec, score global trop bas, etc.)*

---

## 📦 Next

> Fin de la Phase 4.  
> Le pipeline peut soit :
> - s’arrêter ici (validation finale),
> - soit déclencher une phase de revue humaine / déploiement selon la stack.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
