# 📦 Guide Validation — dependencies-coherence
*(Domaine : **dependencies-coherence** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Le domaine **dependencies-coherence** garantit que :

- Les **imports externes** utilisés dans `src_new` sont cohérents avec les dépendances attendues dans la stack.
- Aucun import **non résolu** ou **incompatible** n'est détecté.
- Les dépendances inattendues sont détectées pour validation manuelle.
- Les composants clés ne dépendent pas de librairies interdites ou obsolètes selon les stack-guides.

Ce domaine ne résout **pas réellement** les dépendances (pas de build, pas de resolver NPM).  
Il réalise une **analyse déclarative** et **heuristique**.

---

## 2. 📦 Artefacts attendus

Dans :

```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/dependencies-coherence/
```

Fichiers générés :

- `dependencies-audit.report.json`
- `dependencies-audit.report.md`
- `.meta/validation.dependencies-coherence.meta.json`

---

## 3. 📄 Structure du rapport JSON

```jsonc
{
  "domain": "dependencies-coherence",
  "stageId": "73",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",

  "metrics": {
    "totalImports": 0,
    "externalImports": 0,
    "resolvedImports": 0,
    "unresolvedImports": 0,
    "unexpectedDependencies": 0
  },

  "unexpected": [
    {
      "module": "lodash-es",
      "reason": "Not listed in allowed dependencies for this stack",
      "suggestion": "Validate manually before continuing"
    }
  ],

  "unresolved": [
    {
      "import": "@mylib/form",
      "file": "components/Form.tsx",
      "line": 12,
      "suggestion": "Check file existence or stack compatibility"
    }
  ],

  "validation": {
    "status": "valid | warnings | rejected",
    "score": 0,
    "gate": "passed | failed"
  },

  "thresholds": {
    "maxUnresolved": 0,
    "maxUnexpected": 3,
    "minScore": 70
  }
}
```

---

## 4. 📄 Structure du rapport Markdown

```markdown
# 📦 Rapport de Validation — Dependencies Coherence

**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Score :** 92/100  
**Gate :** ✅ Passed  

---

## 📊 Résumé des métriques

- Total imports : 120  
- Imports externes : 34  
- Imports non résolus : 0  
- Dépendances inattendues : 1  

---

## ❗ Dépendances inattendues
- `lodash-es` : non listée dans la stack → **Validation manuelle requise**

---

## ❌ Imports non résolus
_(Aucun)_  

---

## 💡 Recommandations
1. Vérifier la pertinence des dépendances inattendues.  
2. Aligner les imports externes avec la stack cible.  

---

## ✅ Conclusion
Pas d’incohérence bloquante. Une vérification manuelle recommandée.

---
*Généré par ai-orchestrator-v4 — Stage 73*
```

---

## 5. 🧾 Métadonnées

```jsonc
{
  "stage": "73",
  "domain": "dependencies-coherence",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "generatedCode": "phase-3-generation/src_new/",
    "stackGuides": "stack/stack-guides/*.md"
  },
  "outputs": {
    "json": "dependencies-audit.report.json",
    "md": "dependencies-audit.report.md"
  }
}
```

---

## 6. 🧠 Règles de validation

### 6.1 Inputs requis
- Code généré `src_new`
- Stack-guides (décrivant les dépendances attendues/interdites)
- config.yaml

### 6.2 Checks obligatoires

1. **Extraction des imports externes**
2. **Vérification de résolution**
   - Un import est *non résolu* si aucun fichier ni module connu ne correspond.
3. **Détection des dépendances inattendues**
   - Module externe absent de la whitelist stack.
4. **Détection de versions ou patterns interdits** (si stack-guide les mentionne)
5. **Classification**
   - resolved / unresolved / unexpected

### 6.3 Score

Base 100 – pénalités :
- -40 par import non résolu
- -10 par dépendance inattendue
- -5 par dépendance suspecte

### 6.4 Gate

Gate **❌** si :
- `unresolvedImports > 0`  
- OU `score < minScore`

Gate **✅** sinon.

---

## 7. Auto-checks

- [ ] Stack-guides présents  
- [ ] Imports externes détectés  
- [ ] Non-resolved analysés  
- [ ] Score cohérent  
- [ ] JSON + MD générés  
