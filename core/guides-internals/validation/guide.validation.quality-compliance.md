# 🧹 Guide Validation — quality-compliance
*(Domaine : **quality-compliance** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Le domaine **quality-compliance** vérifie que le code généré pour `${project.pageName}` respecte :

- les **règles de qualité internes** du projet (style, patterns, conventions),
- les **règles imposées par la stack** (React, Next.js, Vue, Angular, etc.),
- les **règles transverses** définies par l'organisation (naming, patterns globaux, conventions de domaine).

Cette validation est **statique**, **déclarative** et **heuristique**.

---

## 2. 📦 Artefacts attendus

Répertoire :

```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/quality-compliance/
```

Fichiers :

- `quality-compliance.report.json`
- `quality-compliance.report.md`
- `.meta/validation.quality-compliance.meta.json`

---

## 3. 📄 Structure du rapport JSON

```jsonc
{
  "domain": "quality-compliance",
  "stageId": "77",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "metrics": {
    "lintLikeIssues": 0,
    "namingInconsistencies": 0,
    "forbiddenPatternsDetected": 0,
    "missingRequiredPatterns": 0,
    "folderStructureViolations": 0
  },
  "issues": [
    {
      "severity": "warning | error",
      "category": "naming | forbidden | required | structure",
      "message": "Description du problème",
      "location": {
        "file": "src_new/components/Button.tsx",
        "line": 14
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
    "minScore": 80,
    "maxForbiddenPatterns": 0
  }
}
```

---

## 4. 📄 Structure du rapport Markdown

```markdown
# 🧹 Rapport de Validation — Quality Compliance

**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Score :** 90/100  
**Gate :** ✅ Passed  

---

## 📊 Résumé des métriques

- Problèmes de style / lint-like : 3  
- Incohérences de nommage : 1  
- Patterns interdits détectés : 0  
- Patterns obligatoires manquants : 0  
- Violations structurelles : 0  

---

## 🚨 Problèmes détectés

### ❌ Erreurs
_(Aucune dans cet exemple)_

### ⚠️ Avertissements
- Composant `Button` ne respecte pas le PascalCase strict.

---

## 💡 Recommandations

1. Vérifier la cohérence du nommage avec les conventions du projet.  
2. Éviter les patterns interdits listés dans les guides globaux.  
3. Vérifier la structure des dossiers recommandée pour la stack.

---

## ✅ Conclusion

Qualité globalement satisfaisante. Quelques corrections mineures recommandées.

---
*Généré par ai-orchestrator-v4 — Stage 77 (quality-compliance)*
```

---

## 5. 🧾 Métadonnées

```jsonc
{
  "stage": "77",
  "domain": "quality-compliance",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "generatedCode": "phase-3-generation/src_new/",
    "stackGuides": "stack/stack-guides/*.md",
    "globalGuides": "core/guides-internals/globals/*.md"
  },
  "outputs": {
    "reportJson": "quality-compliance.report.json",
    "reportMd": "quality-compliance.report.md"
  }
}
```

---

## 6. 🧠 Règles de validation quality-compliance

### 6.1 Inputs requis

- Code généré  
- Stack-guides  
- Guides internes (naming, patterns globaux, structure, etc.)  
- Config projet

### 6.2 Checks obligatoires

1. **Conventions de nommage**
   - noms de composants (PascalCase),
   - hooks (camelCase commençant par `use`),
   - fichiers conformes aux conventions stack (ex : `.tsx` pour React).

2. **Patterns interdits**
   - usage direct de `any` si interdit par stack-guides,
   - appels interdits (`document.querySelector` dans React strict),
   - patterns UI obsolètes si mentionnés.

3. **Patterns obligatoires**
   - hooks obligatoires à utiliser pour certains types de données,
   - handlers standardisés (`handleXxx`, `onXxx`) si imposés.

4. **Structure des dossiers**
   - vérifier que les fichiers générés se situent dans les bonnes sections de la structure cible.

5. **Lint-like heuristics**
   - lignes trop longues,
   - structures incohérentes,
   - bloc de code inutile.

### 6.3 Score & Gate

- Score de base : 100  
- Penalties :
  - -15 pour patterns interdits,
  - -10 par incohérence de nommage majeure,
  - -5 pour structure,
  - -1 ou -2 pour lint-like mineurs.

Gate **❌** si :
- `forbiddenPatternsDetected > maxForbiddenPatterns`
- ou `score < minScore`.

Gate **✅** sinon.

---

## 7. Auto-checks recommandés

- [ ] Stack-guides chargés  
- [ ] Scanning complet du code  
- [ ] Issues correctement classées  
- [ ] Score calculé  
- [ ] Rapports générés  
