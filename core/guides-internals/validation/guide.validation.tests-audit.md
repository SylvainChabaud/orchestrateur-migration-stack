# 🧪 Guide Validation — tests-audit
*(Domaine de validation : **tests-audit** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Le domaine **tests-audit** vérifie la **couverture théorique** et la **cohérence structurelle** de la stratégie de tests générée en Phase 3.  
Il ne **lance pas** les tests : il s’agit d’une validation **statique, heuristique**.

Il répond à :
> « Les tests générés couvrent-ils raisonnablement les UCR / behaviours critiques et respectent-ils la structure attendue par la stack ? »

---

## 2. 📦 Artefacts attendus

Dans :  
```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/tests-audit/
```

Fichiers obligatoires :  
- `tests-audit.report.json`  
- `tests-audit.report.md`  
- `.meta/validation.tests-audit.meta.json`  

---

## 3. 📄 Structure du rapport JSON

```jsonc
{
  "domain": "tests-audit",
  "stageId": "71",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "validation": {
    "status": "valid | warnings | rejected",
    "score": 0,
    "gate": "passed | failed"
  },
  "metrics": {
    "totalTests": 0,
    "ucrTotal": 0,
    "ucrCovered": 0,
    "ucrCritical": 0,
    "ucrCriticalCovered": 0,
    "coverageTheoretical": 0,
    "coverageCritical": 0
  },
  "issues": [
    {
      "severity": "error | warning",
      "ucr": "UCR-XYZ",
      "message": "UCR critique non couvert",
      "suggestion": "Créer un test unitaire ou d'intégration correspondant"
    }
  ],
  "recommendations": [
    "Ajouter des tests unitaires pour les UCR critiques manquants",
    "Réorganiser les dossiers de tests pour respecter les conventions"
  ],
  "thresholds": {
    "minCoverage": 60,
    "minCriticalCoverage": 100
  }
}
```

---

## 4. 📄 Structure du rapport Markdown

```markdown
# 🧪 Rapport Validation — Tests Audit

**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Statut :** ✅ / ⚠️ / ❌  
**Score :** 90/100  
**Gate :** ✅ Passed

---

## 📊 Résumé des métriques

- **UCR totales :** 12  
- **UCR couvertes :** 10  
- **UCR critiques couvertes :** 4/4 → ✅  
- **Couverture théorique :** 83%  
- **Couverture critique :** 100%  

---

## 🚨 Issues détectées

### ❌ Erreurs
_(exemple)_  
- UCR-005 : non couvert alors que critique

### ⚠️ Avertissements
_(exemple)_  
- Certains tests sont présents mais trop génériques

---

## 💡 Recommandations
1. Ajouter un test dédié pour UCR-005  
2. Scinder les tests de composants trop larges en tests plus ciblés

---

## ✅ Conclusion
Couverture critique satisfaisante.  
Quelques compléments recommandés pour renforcer la robustesse.

---

*Généré par ai-orchestrator-v4 — Stage 71 (tests-audit)*
```

---

## 5. 🧾 Métadonnées de validation

```jsonc
{
  "stage": "71",
  "domain": "tests-audit",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "generatedCode": "phase-3-generation/src_new/",
    "ucrMappings": "phase-2-interpretation/mappings/*.json",
    "testFiles": "phase-3-generation/src_new/**/__tests__/*"
  },
  "outputs": {
    "reportJson": "tests-audit.report.json",
    "reportMd": "tests-audit.report.md"
  },
  "checksExecuted": 0,
  "errors": []
}
```

---

## 6. 🧠 Règles de validation tests-audit

### 6.1 Inputs requis
- Configuration : `project-config.yaml`
- Code Phase 3 : `src_new`
- Inventaires UCR Phase 1
- Mappings Phase 2
- Stack-guides tests (si fournis)

### 6.2 Checks obligatoires

1. **Détection des fichiers de tests**
   - Respect des conventions : `.test`, `.spec`, dossiers `__tests__`  
2. **Lien UCR → tests**
   - Chaque UCR critique doit avoir au moins un test associé  
3. **Calcul de couverture théorique**
   - `coverage = (ucrCovered / ucrTotal) * 100`  
   - `coverageCritical = (ucrCriticalCovered / ucrCritical) * 100`  
4. **Analyse structurelle des tests**
   - Tests trop génériques  
   - Tests inexistants pour composants critiques  

### 6.3 Score et Gate

- Score basé sur la couverture + qualité structurelle
- **Gate ✅ si :**
  - `coverageCritical == 100%`
  - `coverageTheoretical >= minCoverage` (souvent 60–70%)

---

## 7. Auto-checks recommandés

- [ ] Tous les UCR critiques mappés ?  
- [ ] Des tests existent pour les UCR critiques ?  
- [ ] Aucune incohérence structurelle grave dans les tests  

