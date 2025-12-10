# 🧩 Stage 76 – performance-patterns
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 75 – accessibility-heuristic  
**Next:** 77 – quality-compliance

---

## 🎯 Objective

Identifier les principaux **risques de performance** dans le code généré pour `${project.pageName}`, en appliquant les règles définies dans les stack-guides de performance.

Aucun bundle réel n’est calculé, aucun profilage n’est exécuté :  
ce stage se limite à une **analyse statique** des imports, composants, listes et hooks.

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `stack.id`

- **Code généré (Phase 3)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`

- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.performance-patterns.md` (guide principal pour ce domaine)

- **Stack-guide de performance**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.performance.md`

---

## 📤 Outputs

Dans :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/performance-patterns/
```

- `performance-audit.report.json`
- `performance-audit.report.md`
- `.meta/validation.performance-patterns.meta.json`

---

## 🧠 Actions

1. **Charger la configuration & le guide de performance**
   - Lire `project-config.yaml`
   - Lire `guide.performance.md` pour identifier :
     - les libs considérées comme "lourdes",
     - les bonnes pratiques d’optimisation (lazy-loading, virtualisation, mémoïsation…).

2. **Scanner le code généré**
   - Lister les composants, hooks et modules analysables dans `src_new`.
   - Identifier :
     - les imports de libs potentiellement lourdes,
     - les rendus de listes (ex. `array.map(...)`).

3. **Analyser les imports lourds**
   - Marquer comme `heavyImports` les imports de modules explicitement listés comme lourds dans le guide.
   - Vérifier, si possible, la présence de mécanismes de lazy-loading/dynamic import.

4. **Analyser les listes et collections**
   - Détecter les grandes listes (heuristique : basées sur les noms, les usages ou les guides).
   - Vérifier la présence de **keys stables**.
   - Vérifier les recommandations de virtualisation / pagination si mentionnées.

5. **Analyser la complexité des composants**
   - Repérer les composants très volumineux ou avec beaucoup de logique imbriquée.
   - Vérifier si une mémoïsation est recommandée (ex. composants critiques d’affichage).

6. **Construire les métriques & le score**
   - Remplir les champs `metrics` définis dans le guide.
   - Calculer un score (0–100) en partant de 100 et en appliquant des pénalités progressives.

7. **Déterminer le Gate**
   - Gate ❌ si :
     - `score < minScore`, ou
     - au moins un problème classé "critique" selon les stack-guides.

8. **Générer les outputs**
   - Écrire `performance-audit.report.json` avec toutes les métriques et issues.
   - Écrire `performance-audit.report.md` avec :
     - résumé,
     - métriques,
     - problèmes critiques,
     - recommandations,
     - conclusion.
   - Écrire `.meta/validation.performance-patterns.meta.json` :
     - inputs, outputs, score, gate, timestamp, etc.

---

## ✅ Auto-Checks

```json
{
  "stageId": "76",
  "stageName": "performance-patterns",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "performanceGuideLoaded": true,
    "codeScanned": true,
    "metricsComputed": true,
    "reportsWritten": true
  }
}
```

---

## 🧩 Gate

Gate ✅  
*(Gate ❌ si score insuffisant ou problème critique de performance détecté.)*

---

## 📦 Next

> Continue with `77-quality-compliance.md` if Gate ✅.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
