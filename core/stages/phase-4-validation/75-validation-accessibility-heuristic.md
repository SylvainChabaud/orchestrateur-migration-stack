# 🧩 Stage 75 – accessibility-heuristic
**Phase:** Phase 4 – Validation & Quality Assurance  
**Prev:** 74 – integration-consistency  
**Next:** 76 – performance-patterns

---

## 🎯 Objective

Évaluer l’**accessibilité** de la page `${project.pageName}` de manière **statique et heuristique**, en s’appuyant sur le code généré et le stack-guide d’accessibilité.

Ce stage ne mesure pas les contrastes réels ni le comportement clavier ;  
il vérifie principalement la présence de labels, d’alternatives textuelles et d’une structure sémantique raisonnable.

---

## ⚙️ Inputs

- **Configuration**
  - `core/configs/project-config.yaml`
    - keys: `project.pageName`, `project.name`, `paths.workspace`, `stack.id`

- **Code généré (Phase 3)**
  - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`
- **Guides internes**
  - `${paths.core}/guides-internals/validation/guide.validation.accessibility-heuristic.md` (guide principal pour ce domaine)
- **Stack-guide d’accessibilité**
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/guide.accessibility.md`

---

## 📤 Outputs

Dans :

```text
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/accessibility-heuristic/
```

- `accessibility-audit.report.json`
- `accessibility-audit.report.md`
- `.meta/validation.accessibility-heuristic.meta.json`

---

## 🧠 Actions

1. **Charger la configuration & les guides**
   - Lire `project-config.yaml`
   - Lire `guide.accessibility.md` pour obtenir les règles métiers spécifiques (si présentes).

2. **Scanner le code généré**
   - Identifier les composants / templates contenant :
     - boutons, liens, inputs, autres contrôles interactifs,
     - images / illustrations,
     - titres (balises de type heading),
     - landmarks ou zones (si pertinentes pour la stack).

3. **Analyser les éléments interactifs**
   - Compter `totalInteractiveElements`.
   - Déterminer lesquels n’ont pas :
     - de label textuel,
     - ou d’attribut ARIA approprié.
   - Incrémenter `interactiveWithoutLabel` et créer des issues détaillées.

4. **Analyser les images**
   - Compter `imagesTotal`.
   - Détecter les images sans alternative textuelle (`imagesWithoutAlt`).

5. **Analyser les titres et la structure sémantique**
   - Compter `headingsTotal`.
   - Détecter les ruptures flagrantes de hiérarchie (ex : h1 → h3 sans h2).  
   - Incrémenter `headingsOutOfOrder` au besoin.

6. **Calculer les métriques et le score**
   - Remplir la section `metrics` du rapport JSON.
   - Calculer `score` sur 0–100 en appliquant des pénalités selon les seuils des stack-guides.

7. **Déterminer le Gate**
   - Gate ❌ si :
     - `interactiveWithoutLabel > maxInteractiveWithoutLabel`, ou
     - `imagesWithoutAlt > maxImagesWithoutAlt`, ou
     - `score < minScore`.

8. **Générer les fichiers de sortie**
   - Écrire `accessibility-audit.report.json`.
   - Générer `accessibility-audit.report.md` (résumé, métriques, issues, recommandations, conclusion).
   - Écrire `.meta/validation.accessibility-heuristic.meta.json`.

---

## ✅ Auto-Checks

```json
{
  "stageId": "75",
  "stageName": "accessibility-heuristic",
  "pageName": "${project.pageName}",
  "checks": {
    "configLoaded": true,
    "accessibilityGuideLoaded": true,
    "codeScanned": true,
    "metricsComputed": true,
    "reportsWritten": true
  }
}
```

---

## 🧩 Gate

Gate ✅  
*(Gate ❌ si seuils d’accessibilité non atteints.)*

---

## 📦 Next

> Continue with `76-performance-patterns.md` if Gate ✅.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
