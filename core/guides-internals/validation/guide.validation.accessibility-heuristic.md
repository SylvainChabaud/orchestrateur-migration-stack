# ♿ Guide Validation — accessibility-heuristic
*(Domaine : **accessibility-heuristic** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Le domaine **accessibility-heuristic** évalue, de manière statique et heuristique, le niveau d’accessibilité de la page générée `${project.pageName}`.

Il vérifie notamment :

- la présence de texte alternatif pour les contenus non textuels,
- la structure sémantique (titres, listes, zones),
- l’accessibilité des éléments interactifs (boutons, liens, champs),
- l’utilisation d’attributs ARIA lorsque nécessaire.

> ⚠️ Ce domaine **ne mesure pas** les contrastes réels ni le comportement clavier ;  
> il applique des **règles heuristiques** décrites dans les stack-guides.

---

## 2. 📦 Artefacts attendus

Répertoire :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/accessibility-heuristic/
```

Fichiers :

- `accessibility-audit.report.json`
- `accessibility-audit.report.md`
- `.meta/validation.accessibility-heuristic.meta.json`

---

## 3. 📄 Structure du rapport JSON

```jsonc
{
  "domain": "accessibility-heuristic",
  "stageId": "75",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",

  "metrics": {
    "totalInteractiveElements": 0,
    "interactiveWithoutLabel": 0,
    "imagesTotal": 0,
    "imagesWithoutAlt": 0,
    "headingsTotal": 0,
    "headingsOutOfOrder": 0,
    "landmarksTotal": 0
  },

  "issues": [
    {
      "severity": "error | warning",
      "category": "label | alt | heading | landmark | aria",
      "message": "Description du problème",
      "location": {
        "file": "path/to/file.tsx",
        "line": 42
      },
      "suggestion": "Correction recommandée"
    }
  ],

  "score": 0,
  "validation": {
    "status": "valid | warnings | rejected",
    "gate": "passed | failed"
  },

  "thresholds": {
    "minScore": 70,
    "maxInteractiveWithoutLabel": 0,
    "maxImagesWithoutAlt": 0
  }
}
```

---

## 4. 📄 Structure du rapport Markdown

```markdown
# ♿ Rapport de Validation — Accessibility Heuristic Audit

**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Score :** 91/100  
**Gate :** ✅ Passed  

---

## 📊 Résumé des métriques

- Éléments interactifs : 25  
- Sans label accessible : 0  
- Images : 4  
- Sans `alt` : 0  
- Titres hors ordre hiérarchique : 1  

---

## 🚨 Issues détectées

### ❌ Problèmes critiques
- Bouton sans label accessible dans `components/PrimaryButton.tsx:32`

### ⚠️ Avertissements
- Hiérarchie de titres non linéaire dans `pages/FooPage.tsx`

---

## 💡 Recommandations

1. Ajouter des labels explicites sur tous les éléments interactifs.  
2. Structurer les titres de façon hiérarchique (h1 → h2 → h3, etc.).  

---

## ✅ Conclusion

Accessibilité globalement satisfaisante, quelques améliorations recommandées.

---
*Généré par ai-orchestrator-v4 — Stage 75 (accessibility-heuristic)*
```

---

## 5. 🧾 Métadonnées

```jsonc
{
  "stage": "75",
  "domain": "accessibility-heuristic",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "generatedCode": "phase-3-generation/src_new/",
    "stackGuides": "stack/stack-guides/guide.accessibility.md"
  },
  "outputs": {
    "reportJson": "accessibility-audit.report.json",
    "reportMd": "accessibility-audit.report.md"
  }
}
```

---

## 6. 🧠 Règles de validation accessibility-heuristic

### 6.1 Inputs requis

- Code généré `src_new`  
- Stack-guide accessibilité : `guide.accessibility.md`  
- Config projet (pour les paths)

### 6.2 Checks obligatoires

1. **Éléments interactifs**
   - Boutons, liens, inputs, etc.
   - Vérifier la présence d’un label (texte ou attribut ARIA pertinent).

2. **Images**
   - Présence d’attribut `alt` (ou équivalent accessibilité) pour les images importants.
   - Ignorer ou marquer comme décoratives les images explicitement signalées comme telles.

3. **Structure des titres**
   - Vérifier l’ordre logique (h1 → h2 → h3, etc.).
   - Signaler les ruptures manifestes (passage h1 → h3 sans h2, etc.).

4. **Landmarks / zones**
   - Vérifier la présence éventuelle de landmarks (`main`, `nav`, `header`, etc.) si recommandé par les stack-guides.

5. **ARIA**
   - Vérifier l’usage d’attributs ARIA seulement lorsqu’ils sont nécessaires (pas de sur-aria).

### 6.3 Score & Gate

- Score initial : 100  
- Pénalités par problème critique ou warning, selon les seuils du stack-guide.

Gate **❌** si :
- `interactiveWithoutLabel > maxInteractiveWithoutLabel`
- ou `imagesWithoutAlt > maxImagesWithoutAlt`
- ou `score < minScore`

Gate **✅** sinon.

---

## 7. ✅ Auto-checks recommandés

- [ ] `guide.accessibility.md` présent  
- [ ] Code scanné  
- [ ] Métriques calculées  
- [ ] Score cohérent  
- [ ] JSON + MD générés  
