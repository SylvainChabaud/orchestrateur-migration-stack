# 🧩 Stage 62 – generation-summary
**Phase :** Phase 3 – Generation  
**Prev :** 61 – optimize-imports  
**Next :** 70 – functional-audit (Phase 4)

---

## 🎯 Objectif

Produire le **rapport final de la Phase 3** pour `${project.pageName}`, en agrégeant :

- les métadonnées de tous les stages 50 → 61,
- les statistiques de génération (fichiers, lignes de code, par domaine),
- l’état de validation de chaque stage (Gate ✅ / ❌, validation JSON, issues),
- un **flag global** `readyForPhase4` indiquant si la Phase 4 (validation fonctionnelle) peut démarrer.

Ce stage **ne génère pas de code métier** :  
il consolide uniquement des **rapports** (JSON + Markdown) et une **métadonnée de stage**.

---

## 🔌 Entrées (Inputs)

> ⚠️ Tous les chemins sont résolus à partir de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### 1. Configuration globale

Depuis `core/configs/project.config.yaml` :

- `project.name`  
- `project.pageName`  
- `paths.core`  
- `paths.workspace`  
- `validation.enableSchemaValidation`  
- `validation.schemasPath`  

### 2. Artefacts Phase 0

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`  
- `bridge-legacy-to-dsl.json`  

Aucun stack-guide spécifique n’est requis pour ce stage.

### 3. Métadonnées de génération Phase 3 (stages 50 → 61)

Pour chaque stage de génération :

- 50 – types  
- 51 – services  
- 52 – stores  
- 53 – hooks-logic  
- 54 – hooks-data  
- 55 – components-atoms  
- 56 – components-containers  
- 57 – pages  
- 58 – routing  
- 59 – i18n  
- 60 – tests  
- 61 – optimize-imports  

Le stage lit les fichiers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.*.meta.json`

Chaque fichier de métadonnées de stage doit contenir au minimum :

- l’identifiant de domaine (ex. `domain: "pages"`),  
- le `stageId` (ex. `"57"`),  
- la liste des fichiers générés (`filesGenerated[]`),  
- un bloc `validation` (`status`, `issues[]`, etc.),  
- l’état du Gate pour le stage (directement ou indirectement).

Si un fichier de métadonnées est **manquant** pour un stage donné, ce stage est considéré comme :

- **Gate ❌**,  
- `validation.status != "valid"`.

### 4. Guides internes

Depuis `${paths.core}/guides-internals/` :

- `${paths.core}/guides-internals/generation/guide.generation.generation-summary.md`  
- `${paths.core}/guides-internals/globals/guide.ucr.md`  

Ils décrivent :

- la forme attendue de `generation-summary.json` et `generation-report.md`,
- la façon de lier les UCR aux statistiques de génération.

---

## 📤 Sorties (Outputs)

### 1. Rapports de Phase 3 (par page)

Sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/`

Le stage doit écrire :

1. `generation-summary.json`  
   - Vue machine du résumé de Phase 3, avec statistiques agrégées et `readyForPhase4`.  

2. `generation-report.md`  
   - Rapport lisible (Markdown) pour un humain, détaillant :  
     - un résumé exécutif,  
     - les stats globales,  
     - les stats par stage,  
     - les issues éventuelles,  
     - la décision finale : **Phase 4 possible / non possible**.

### 2. Métadonnées du stage 62

Sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/`

Le stage écrit :

- `generation.generation-summary.meta.json`

Structure minimale attendue :

```jsonc
{
  "domain": "generation-summary",
  "stageId": "62",
  "pageName": "${project.pageName}",
  "filesGenerated": [
    { "path": "src_new/generation-summary.json", "ucr": [], "linesOfCode": 0 },
    { "path": "src_new/generation-report.md", "ucr": [], "linesOfCode": 0 }
  ],
  "statistics": {
    "totalFiles": 0,
    "totalLinesOfCode": 0
  },
  "validation": {
    "status": "valid",
    "issues": [],
    "checks": {}
  },
  "readyForPhase4": true
}
```

Les champs peuvent être enrichis selon `guide.generation.generation-summary.md`.

---

## 🧠 Actions

### Étape 1 – Charger configuration & contexte

1.1. Charger `core/configs/project.config.yaml` et résoudre `${paths.core}` et `${paths.workspace}`.  
1.2. Charger `${paths.workspace}/projects/${project.name}/stack/project-structure.json`.  
1.3. Charger `${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json` (optionnel, pour traçabilité UCR).

### Étape 2 – Charger les guides de résumé

2.1. Charger `${paths.core}/guides-internals/generation/guide.generation.generation-summary.md`.  
2.2. Charger `${paths.core}/guides-internals/globals/guide.ucr.md`.  

Ces guides définissent :  

- les statistiques à calculer,  
- la structure détaillée de `generation-summary.json`,  
- les règles de construction de `generation-report.md`.

### Étape 3 – Collecter toutes les métadonnées de génération (stages 50 → 61)

3.1. Définir la liste ordonnée des stages :  

```jsonc
["50","51","52","53","54","55","56","57","58","59","60","61"]
```

3.2. Pour chaque stageId de cette liste :

- rechercher un fichier de métadonnées correspondant dans :  
  `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/`  
  avec un nom du type `generation.*.meta.json` ;
- charger ce fichier et extraire :
  - `domain` (types, services, stores, …) ;
  - `stageId` ;
  - `filesGenerated[]` (path, ucr éventuels, LOC éventuel) ;
  - `validation.status` ;
  - `validation.issues[]` ;
  - un indicateur Gate (si présent dans le meta ; sinon déduit de `validation.status`).  

3.3. Si un fichier de métadonnées est introuvable pour un stage :  

- considérer que ce stage est en **échec** (Gate ❌) ;  
- créer une entrée synthétique avec :
  - `filesGenerated = []` ;
  - `validation.status = "missing"` ;
  - `validation.issues = ["Missing generation meta for stage <id>"]`.

### Étape 4 – Agréger les statistiques et l’état global

4.1. Agréger les statistiques globales, par exemple :

- `totalFiles` : somme de tous les `filesGenerated.length` ;
- `totalLinesOfCode` : somme des `linesOfCode` si disponibles, sinon `null` ou estimation ;
- statistiques par domaine (types, services, stores, hooks, components, pages, routing, i18n, tests).

4.2. Construire une vue `byStage` indexée par stageId ou par `domain` :

```jsonc
"byStage": {
  "50-types": { "files": 0, "loc": 0, "status": "valid" },
  "51-services": { "files": 0, "loc": 0, "status": "valid" }
}
```

4.3. Récupérer et consolider toutes les `issues` provenant des métadonnées de stages.

### Étape 5 – Déterminer `readyForPhase4`

5.1. Un stage 50 → 61 est considéré comme **OK** si :

- son Gate est ✅,  
- ET `validation.status = "valid"` (ou équivalent “pas d’erreur critique”).

5.2. La Phase 3 est considérée comme **complète** si :

- tous les stages 50 → 61 sont **OK**,  
- ET aucun stage ne remonte d’issue bloquante,  
- ET le nombre total de fichiers générés est > 0.

5.3. Dans ce cas :

- `readyForPhase4 = true` dans :
  - `generation-summary.json` ;
  - `generation.generation-summary.meta.json`.

5.4. Sinon :

- `readyForPhase4 = false` ;  
- consigner les raisons (stages manquants, Gate ❌, validation invalide, 0 fichier généré) dans :
  - `generation-summary.json.issues[]` ;
  - `generation.generation-summary.meta.json.validation.issues[]`.

---

## 🧾 Génération des fichiers de sortie

### Étape 6 – Construire `generation-summary.json`

6.1. Construire un objet JSON avec au minimum :

- `domain: "generation-summary"`  
- `pageName: "${project.pageName}"`  
- `phase: 3`  
- `statistics` (globale + par domaine)  
- `byStage` (détaillé par stage 50 → 61)  
- `validation` (agrégée : `allStagesCompleted`, `allFilesWritten`, `noCollisions`, `importsResolvable`, etc., selon `guide.generation.generation-summary.md`)  
- `phase3Complete` (booléen : tous stages 50 → 61 OK)  
- `readyForPhase4` (booléen)  
- `issues[]` (liste des problèmes bloquants ou warnings importants)

6.2. Écrire ce JSON sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/generation-summary.json`

### Étape 7 – Construire `generation-report.md`

7.1. À partir de `generation-summary.json` et des guides :

- générer un rapport Markdown avec :  
  - **en-tête** (page, date, statut global) ;  
  - **résumé exécutif** (nombre de fichiers, LOC, statut Phase 3) ;  
  - **tableau par stage** (files, LOC, statut, issues majeures) ;  
  - **section Validation** (compilation, imports, collisions si ces infos existent dans les métadonnées) ;  
  - **section Prêt pour Phase 4** (explicite).  

7.2. Écrire ce rapport sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/generation-report.md`

### Étape 8 – Générer la métadonnée de stage 62

8.1. Construire l’objet `meta62` avec :

- `domain: "generation-summary"`  
- `stageId: "62"`  
- `pageName: "${project.pageName}"`  
- `filesGenerated` :
  - `src_new/generation-summary.json`
  - `src_new/generation-report.md`
- `statistics` :
  - `totalFiles` (reprise de `generation-summary.json.statistics.totalFiles`) ;
  - `totalLinesOfCode` (reprise ou agrégat).  
- `validation` :
  - `status` = `"valid"` si les deux fichiers sont écrits et cohérents ;  
  - `issues[]` le cas échéant.  
- `readyForPhase4` (même valeur que dans `generation-summary.json`).

8.2. Écrire :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.generation-summary.meta.json`

### Étape 9 – Validation schéma (optionnelle)

9.1. Si `validation.enableSchemaValidation = true` :

- charger les schémas JSON depuis `${validation.schemasPath}` ;  
- valider :
  - `generation-summary.json` ;
  - `generation.generation-summary.meta.json`.  

9.2. En cas d’erreur de schéma :

- ajouter les messages dans `validation.issues[]` ;  
- si l’erreur est bloquante selon `guide.generation.generation-summary.md`, forcer **Gate ❌**.

---

## 🧩 Gate

### Gate ✅

Utiliser :

```markdown
## 🧩 Gate
Gate ✅
```

si et seulement si :

- les métadonnées de tous les stages 50 → 61 ont été lues (ou marquées comme manquantes),  
- `generation-summary.json` et `generation-report.md` ont été écrits,  
- `generation.generation-summary.meta.json` a été écrit avec `validation.status = "valid"`,  
- `readyForPhase4 = true`.

### Gate ❌

Utiliser :

```markdown
## 🧩 Gate
Gate ❌
```

si :

- un stage précédent critique (50 → 61) est en échec (Gate ❌),  
- ou les fichiers de sortie n’ont pas pu être écrits,  
- ou la validation schéma échoue de manière bloquante.

Dans ce cas, **forcer** :

- `readyForPhase4 = false` dans :
  - `generation-summary.json` ;
  - `generation.generation-summary.meta.json`.

---

## 📦 Next

> Continuer avec **70 – functional-audit (Phase 4)** uniquement si :  

- Stage 62 a `Gate ✅`  
- et `readyForPhase4 = true`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
