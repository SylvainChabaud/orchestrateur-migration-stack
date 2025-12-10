# 🧩 Guide Validation — integration-consistency
*(Domaine : **integration-consistency** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Le domaine **integration-consistency** vérifie que le code généré pour `${project.pageName}` :

- s’insère correctement dans la **structure cible** (dossiers, fichiers, points d’entrée),
- respecte les règles d’**intégration et de routing** de la stack,
- ne crée pas de **collisions** de routes, de composants ou de fichiers,
- ne casse pas les **contrats d’intégration** (layout, providers, navigation, etc.).

Il répond à la question :

> **« La page générée est-elle prête à être branchée dans l’application cible sans incohérence structurelle majeure ? »**

Ce domaine reste **déclaratif** :  
il ne lance pas d’application, ne vérifie pas le runtime, mais s’appuie sur la structure des fichiers, le routing et les stack-guides.

---

## 2. 📦 Artefacts attendus

Répertoire :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/integration-consistency/
```

Fichiers à générer :

- `integration-consistency.report.json`
- `integration-consistency.report.md`
- `.meta/validation.integration-consistency.meta.json`

---

## 3. 📄 Structure du rapport JSON

```jsonc
{
  "domain": "integration-consistency",
  "stageId": "74",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",

  "metrics": {
    "routesDeclared": 0,
    "routeConflicts": 0,
    "layoutConflicts": 0,
    "missingEntryFiles": 0
  },

  "conflicts": [
    {
      "type": "route | layout | file",
      "message": "Conflict description",
      "location": "path/to/file.ext",
      "suggestion": "How to fix it"
    }
  ],

  "missing": [
    {
      "type": "entry | layout | provider",
      "expected": "app/page.tsx",
      "message": "Expected entry file missing for this route"
    }
  ],

  "validation": {
    "status": "valid | warnings | rejected",
    "score": 0,
    "gate": "passed | failed"
  },

  "thresholds": {
    "maxRouteConflicts": 0,
    "maxLayoutConflicts": 0,
    "maxMissingEntryFiles": 0,
    "minScore": 70
  }
}
```

---

## 4. 📄 Structure du rapport Markdown

```markdown
# 🧩 Rapport de Validation — Integration Consistency

**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Score :** 93/100  
**Gate :** ✅ Passed  

---

## 📊 Résumé

- Routes déclarées : 3  
- Conflits de routes : 0  
- Fichiers d’entrée manquants : 0  
- Conflits de layout : 0  

---

## ⚠️ Détails des conflits

### ❌ Conflits
_(Aucun)_  

### ⚠️ Manques
_(Aucun)_  

---

## 💡 Recommandations

1. Vérifier régulièrement la table de routing après ajout de nouvelles pages.  
2. Harmoniser les layouts utilisés pour les sections similaires.  

---

## ✅ Conclusion

La page générée semble s’intégrer correctement dans la structure cible.

---
*Généré par ai-orchestrator-v4 — Stage 74 (integration-consistency)*
```

---

## 5. 🧾 Métadonnées

```jsonc
{
  "stage": "74",
  "domain": "integration-consistency",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "projectStructure": "project-structure.json",
    "generatedCode": "phase-3-generation/src_new/",
    "stackGuides": "stack/stack-guides/*.md"
  },
  "outputs": {
    "jsonReport": "integration-consistency.report.json",
    "mdReport": "integration-consistency.report.md"
  }
}
```

---

## 6. 🧠 Règles de validation integration-consistency

### 6.1 Inputs requis

- `project-structure.json` (ou équivalent) généré en Phase 0/2  
- Code généré `src_new`  
- Stack-guides (routing, structure, layout, etc.)  
- Configuration projet (paths, stack id)

### 6.2 Checks obligatoires

1. **Correspondance structurelle**
   - Les fichiers générés existent aux emplacements attendus par `project-structure.json`.
   - Pas de fichier critique manquant (page, layout, entry).

2. **Routing**
   - Les routes générées respectent les conventions de la stack.
   - Aucune collision de route (deux artefacts pour la même route sans justification explicite).

3. **Layouts / Providers / Wrappers**
   - Les composants générés utilisent les bons layouts/wrappers si spécifiés dans les stack-guides.
   - Aucune utilisation incohérente (ex : section devant utiliser `AdminLayout` mais utilisant `PublicLayout`).

4. **Intégration aux points d’entrée**
   - Vérifier que la page générée est référencée par le point d’entrée attendu (ou structure équivalente).

### 6.3 Score & Gate

- Score de base : 100  
- Pénaliser :
  - -40 par conflit de route
  - -20 par fichier d’entrée manquant
  - -10 par conflit de layout

Gate **❌** si :
- `routeConflicts > 0`
- ou `missingEntryFiles > 0`
- ou `score < minScore`

Gate **✅** sinon.

---

## 7. ✅ Auto-checks recommandés

- [ ] `project-structure.json` présent  
- [ ] `src_new` analysé  
- [ ] Routes extraites  
- [ ] Conflits et manques identifiés  
- [ ] Score cohérent  
- [ ] JSON + MD générés  
