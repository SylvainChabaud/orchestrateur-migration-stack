# 🚀 Guide Validation — performance-patterns
*(Domaine : **performance-patterns** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Le domaine **performance-patterns** détecte les **risques de performance évidents** dans le code généré pour `${project.pageName}`, en se basant sur une analyse statique et sur les bonnes pratiques définies par la stack.

Il s’agit d’un audit **heuristique**, qui vise à repérer notamment :

- l’usage de **libs lourdes** sans lazy-loading,
- les **listes volumineuses** rendues sans optimisation,
- les composants **monolithiques** ou trop complexes,
- des patterns pouvant provoquer des **re-renders inutiles**.

Aucun bundle n’est réellement calculé, aucun profiling n’est exécuté.

---

## 2. 📦 Artefacts attendus

Répertoire :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/performance-patterns/
```

Fichiers à générer :

- `performance-audit.report.json`
- `performance-audit.report.md`
- `.meta/validation.performance-patterns.meta.json`

---

## 3. 📄 Structure du rapport JSON

```jsonc
{
  "domain": "performance-patterns",
  "stageId": "76",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",

  "metrics": {
    "totalComponents": 0,
    "potentiallyHeavyComponents": 0,
    "largeListsDetected": 0,
    "listsWithoutKeys": 0,
    "missingMemoizationHooks": 0,
    "heavyImports": 0
  },

  "heavyImportsDetail": [
    {
      "module": "chart.js",
      "file": "components/Statistics.tsx",
      "suggestion": "Consider dynamic import / lazy loading"
    }
  ],

  "issues": [
    {
      "severity": "error | warning",
      "category": "heavyImport | list | memo | other",
      "message": "Description du problème",
      "location": {
        "file": "path/to/file.tsx",
        "line": 42
      },
      "suggestion": "Recommandation de correction"
    }
  ],

  "score": 0,
  "validation": {
    "status": "valid | warnings | rejected",
    "gate": "passed | failed"
  },

  "thresholds": {
    "minScore": 70,
    "maxHeavyImportsWithoutLazyLoading": 0,
    "maxLargeListsWithoutOptimization": 0
  }
}
```

---

## 4. 📄 Structure du rapport Markdown

```markdown
# 🚀 Rapport de Validation — Performance Patterns Audit

**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Score :** 88/100  
**Gate :** ✅ Passed  

---

## 📊 Résumé des métriques

- Composants analysés : 18  
- Libs lourdes sans lazy-loading : 1  
- Grandes listes détectées : 2  
- Listes sans key stable : 1  
- Manque de mémoïsation sur composants critiques : 1  

---

## 🚨 Points d’attention

### ❌ Problèmes critiques

- Grande liste rendue sans key stable dans `components/ProductList.tsx`  

### ⚠️ Avertissements

- Import direct de `chart.js` dans `components/Stats.tsx` (pas de lazy-loading)

---

## 💡 Recommandations

1. Appliquer du **lazy-loading** pour les composants / libs lourdes.  
2. Ajouter des **keys stables** sur les éléments de listes.  
3. Utiliser `memo`, `useMemo`, `useCallback` (ou équivalents) pour les composants critiques.  

---

## ✅ Conclusion

Performances globalement acceptables, mais des optimisations sont possibles sur certains points.

---
*Généré par ai-orchestrator-v4 — Stage 76 (performance-patterns)*
```

---

## 5. 🧾 Métadonnées

```jsonc
{
  "stage": "76",
  "domain": "performance-patterns",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "generatedCode": "phase-3-generation/src_new/",
    "stackGuides": "stack/stack-guides/guide.performance.md"
  },
  "outputs": {
    "reportJson": "performance-audit.report.json",
    "reportMd": "performance-audit.report.md"
  }
}
```

---

## 6. 🧠 Règles de validation performance-patterns

### 6.1 Inputs requis

- Code généré `src_new`  
- Stack-guide de performance : `guide.performance.md`  
- Configuration projet (paths, stack id)

### 6.2 Checks obligatoires

1. **Heavy imports**
   - Identifier les imports de libs typiquement lourdes (ex : graphiques, data-grid, visualisation).
   - Vérifier si des mécaniques de lazy-loading ou dynamic import sont recommandées dans `guide.performance.md`.

2. **Listes volumineuses**
   - Repérer les rendus de listes (`map`, `for`, etc.).
   - Vérifier la présence de keys stables.
   - Vérifier l’éventuelle recommandation de virtualisation / pagination.

3. **Composants critiques / complexes**
   - Détecter les composants très gros (taille, nombre de props, logique complexe).
   - Vérifier l’usage (ou l’absence) de mémoïsation (`memo`, `useMemo`, `useCallback`, etc. si stack React).

4. **Hooks / sélecteurs**
   - Vérifier que les sélecteurs de store / hooks intensifs ne sont pas appelés de façon naïve dans de grosses listes.

### 6.3 Score & Gate

- Score de base : 100  
- Pénalités en fonction :
  - nombre de heavy imports non lazy-loadés,
  - grandes listes non optimisées,
  - composants critiques non mémoïsés.

Gate **❌** si :
- `score < minScore`
- OU un problème critique identifié par la stack (par ex. liste énorme non optimisée sur un écran clé).

Gate **✅** sinon.

---

## 7. ✅ Auto-checks recommandés

- [ ] `guide.performance.md` présent  
- [ ] Code scanné  
- [ ] Heavy imports détectés  
- [ ] Listes analysées  
- [ ] Score cohérent  
- [ ] Rapports JSON + MD générés  
