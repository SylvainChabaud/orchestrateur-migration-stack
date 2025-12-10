# 🔍 Guide Validation — static-consistency
*(Domaine de validation : **static-consistency** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine de validation

Le domaine **static-consistency** valide, de manière **statique et heuristique**, que le code généré en Phase 3 est :
- structurellement cohérent (imports, fichiers, modules),
- libre de problèmes bloquants évidents (imports non résolus, cycles critiques),
- raisonnablement propre du point de vue des **code smells** (fonctions trop longues, fichiers gigantesques),
- aligné avec les règles de base de typage/props définies par la stack.

Il répond à la question :

> **« Le code généré est-il structurellement sain et raisonnablement propre avant de pousser plus loin la validation ? »**

---

## 2. 📦 Artefacts et emplacements

Les artefacts générés pour **static-consistency** doivent être produits dans :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/static-consistency/
```

Fichiers obligatoires :

- `static-consistency.report.json` — Rapport machine-readable
- `static-consistency.report.md` — Rapport human-readable
- `.meta/validation.static-consistency.meta.json` — Métadonnées de validation

---

## 3. 📄 Structure du rapport JSON

Fichier : `static-consistency.report.json`

```jsonc
{
  "domain": "static-consistency",
  "stageId": "70",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "validation": {
    "status": "valid | warnings | rejected",
    "score": 0,
    "gate": "passed | failed",
    "issues": [
      {
        "severity": "error | warning | info",
        "category": "imports | types | smells | structure",
        "message": "Description courte du problème",
        "location": {
          "file": "path/relatif/au/fichier.ext",
          "line": 42,
          "column": 10
        },
        "suggestion": "Proposition de correction"
      }
    ]
  },
  "metrics": {
    "totalFiles": 0,
    "totalImports": 0,
    "unresolvedImports": 0,
    "circularDependencies": 0,
    "typeInconsistencies": 0,
    "detectedSmells": {
      "longFunctions": 0,
      "deepNesting": 0,
      "duplicateCode": 0
    },
    "consistencyScore": 0
  },
  "summary": {
    "totalChecks": 0,
    "passedChecks": 0,
    "failedChecks": 0,
    "warningChecks": 0,
    "skippedChecks": 0
  },
  "thresholds": {
    "maxUnresolvedImports": 0,
    "maxCircularDeps": 0,
    "minConsistencyScore": 70
  },
  "recommendations": [
    "Action recommandée 1",
    "Action recommandée 2"
  ]
}
```

Règles :
- `score` et `consistencyScore` sont compris entre **0 et 100**.
- `validation.status` :
  - `valid` si aucun problème bloquant et peu de warnings,
  - `warnings` si des améliorations sont nécessaires mais non bloquantes,
  - `rejected` s’il y a au moins une **issue critique**.
- `validation.gate` :
  - `passed` si les conditions de Gate✅ sont réunies (cf. section 6),
  - `failed` sinon.

---

## 4. 📄 Structure du rapport Markdown

Fichier : `static-consistency.report.md`

```markdown
# 🔍 Rapport de Validation — Static Consistency & Code Smells

**Page :** ${project.pageName}  
**Date :** 2025-12-08 10:00:00  
**Statut :** ✅ Valid / ⚠️ Warnings / ❌ Rejected  
**Score :** 95/100  
**Gate :** ✅ Passed / ❌ Failed

---

## 📊 Résumé

- **Total vérifications :** 6  
- **Réussies :** 6 ✅  
- **Échouées :** 0 ❌  
- **Avertissements :** 0 ⚠️  

---

## 📈 Métriques (Analyse statique heuristique)

| Métrique                | Valeur | Seuil         | Statut |
|-------------------------|--------|---------------|--------|
| Imports non résolus     | 0      | 0 max         | ✅     |
| Dépendances circulaires | 0      | 0 max         | ✅     |
| Incohérences de types   | 0      | 0 max         | ✅     |
| Fonctions longues       | 0      | -             | ✅     |
| Imbrication profonde    | 0      | -             | ✅     |
| Code dupliqué           | 0      | -             | ✅     |
| Score de cohérence      | 95.0   | ≥ 70          | ✅     |

---

## 🚨 Issues détectées

### ❌ Erreurs

_(Lister ici les erreurs, ou « Aucune »)_

### ⚠️ Avertissements

_(Lister ici les warnings, ou « Aucun »)_

---

## 💡 Recommandations

1. Exemple : découper les fonctions trop longues en sous-fonctions lisibles.
2. Exemple : réduire l’imbrication de blocs conditionnels.

---

## ✅ Conclusion

_Synthèse courte : niveau de confiance, risques identifiés, prochaines étapes._

---

*Généré par ai-orchestrator-v4 — Phase 4 — Stage 70 (static-consistency)*
```

---

## 5. 🧾 Métadonnées de validation

Fichier : `.meta/validation.static-consistency.meta.json`

```jsonc
{
  "stage": "70",
  "domain": "static-consistency",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "generatedCode": "phase-3-generation/src_new/",
    "mappings": ["mapping.structure.json", "mapping.logic.json"],
    "inventories": ["inventory.structure.json"],
    "stackGuides": [
      "guide.validation.md",
      "guide.quality-thresholds.md"
    ],
    "legacySource": "${paths.legacySource}"
  },
  "outputs": {
    "report": "phase-4-validation/static-consistency/static-consistency.report.json",
    "reportMd": "phase-4-validation/static-consistency/static-consistency.report.md"
  },
  "checksExecuted": 0,
  "duration": "0s",
  "errors": []
}
```

---

## 6. 🧠 Règles de validation static-consistency

### 6.1. Inputs principaux

Le stage 70 doit impérativement lire :

- `core/configs/project-config.yaml`
- Code généré :  
  `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`
- Stack guides :  
  `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.validation.md`  
  `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.quality-thresholds.md`
- Inventaires/mappings utiles pour rattacher les problèmes à des UCR (si possible).

### 6.2. Types de checks

Le domaine **static-consistency** couvre, au minimum, les familles de checks suivantes :

1. **Imports & modules**
   - Imports pointant vers des fichiers inexistants
   - Imports externes manifestement incohérents avec la stack
   - Cycles simples entre modules critiques

2. **Types & props (heuristique)**
   - Composants majeurs sans typage/PropTypes alors que recommandé
   - Incohérences évidentes entre usage de props et définition (ex : props supposées requises mais non fournies)

3. **Code smells structurels**
   - Fonctions ou composants anormalement longs (selon seuils des stack-guides)
   - Imbrication très profonde de blocs (`if`, `switch`, `loops`)
   - Duplications manifestes de blocs de code (très grossières)

### 6.3. Métriques minimales

Le rapport JSON doit au minimum renseigner :

- `totalFiles`, `totalImports`
- `unresolvedImports`, `circularDependencies`
- `typeInconsistencies`
- `detectedSmells.*`
- `consistencyScore`

### 6.4. Scoring & Gate

Recommandation réaliste :

- Score de base = 100
- Retirer des points pour :
  - chaque import non résolu,
  - chaque cycle critique,
  - chaque incohérence de types,
  - chaque smell important.
- Normaliser entre 0 et 100.

**Gate ✅** si :
- `unresolvedImports == 0` sur les fichiers critiques,
- `circularDependencies == 0`,
- `consistencyScore >= minConsistencyScore` (ex. 70).

**Gate ❌** sinon.

---

## 7. ✅ Auto-checks recommandés

Avant de conclure :

- [ ] Le répertoire `src_new` existe et contient au moins un fichier.
- [ ] Les stack-guides nécessaires sont présents.
- [ ] Le JSON final respecte la structure attendue.
- [ ] Le score est bien entre 0 et 100.
- [ ] Le Gate est cohérent avec les issues et les seuils.
