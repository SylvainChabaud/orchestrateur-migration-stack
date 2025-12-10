# 🔧 Guide Génération — Summary (Résumé de Génération)

*(Domaine de génération : **generation-summary** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif

Le domaine **generation-summary** décrit comment produire le **résumé global de la Phase 3** pour `${project.pageName}`, sous deux formes complémentaires :

1. un **fichier JSON** (`generation-summary.json`) exploitable par d’autres outils / étapes de pipeline ;  
2. un **rapport Markdown** (`generation-report.md`) lisible par un humain.

Ce résumé sert à :

- vérifier la complétude de la Phase 3 (stages 50 → 61) ;
- synthétiser les statistiques de génération (fichiers, lignes de code, par domaine) ;
- recenser les issues et warnings ;
- décider si la **Phase 4 (validation fonctionnelle)** peut commencer (`readyForPhase4`).

---

## 2. 🔌 Entrées du domaine

### 2.1. Métadonnées de stages de génération (50 → 61)

Sources :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/.meta/generation.*.meta.json`

Pour chaque stage 50 → 61, les métadonnées doivent fournir :

- `domain` (ex. `"services"`, `"stores"`, `"pages"`, `"tests"`, `"imports"`, …) ;
- `stageId` ;
- `filesGenerated[]` (avec au moins les chemins) ;
- `statistics` (ex. nombre de fichiers, LOC, etc., si disponible) ;
- `validation.status` (`"valid"`, `"invalid"`, `"missing"`, …) ;
- `validation.issues[]` ;
- éventuellement un indicateur de Gate (`gateStatus` ou équivalent).

En absence de métadonnée pour un stage donné, le générateur doit **marquer ce stage comme manquant**.

### 2.2. Fichiers générés (optionnel)

Le domaine peut, en plus, scanner les dossiers :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/`

pour :

- vérifier l’existence des fichiers référencés dans les métadonnées ;
- détecter d’éventuelles collisions ou incohérences.

### 2.3. Mappings & structure

Pour certaines vérifications (non obligatoires) :

- `mappings-summary.json` (Phase 2) ;
- `${paths.workspace}/projects/${project.name}/stack/project-structure.json`.

Ces fichiers permettent de croiser :

- ce qui **aurait dû** être généré (d’après mappings / structure) ;
- ce qui a **réellement** été généré (d’après les métadonnées).

---

## 3. 📥 Structure d’entrée typique

Exemple simplifié de métadonnées de stage :

```jsonc
{
  "domain": "services",
  "stageId": "51",
  "pageName": "CampaignsCreate",
  "filesGenerated": [
    { "path": "services/CreateCampaignService.ext", "linesOfCode": 120 }
  ],
  "statistics": {
    "filesCount": 3,
    "linesOfCode": 420
  },
  "validation": {
    "status": "valid",
    "issues": [],
    "checks": {
      "schemaValid": true
    }
  },
  "gateStatus": "Gate ✅"
}
```

Le domaine `generation-summary` se contente de **consommer** ces métadonnées, sans les modifier.

---

## 4. 📤 Outputs attendus

### 4.1. `generation-summary.json`

Fichier JSON écrit sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/generation-summary.json`

Structure recommandée :

```jsonc
{
  "domain": "generation-summary",
  "pageName": "${project.pageName}",
  "phase": 3,
  "statistics": {
    "filesGenerated": 142,
    "linesOfCode": 8450,
    "typesGenerated": 45,
    "servicesGenerated": 8,
    "storesGenerated": 4,
    "hooksGenerated": 22,
    "componentsGenerated": 38,
    "pagesGenerated": 1,
    "routesGenerated": 5,
    "translationsGenerated": 2,
    "testsGenerated": 65
  },
  "byStage": {
    "50-types": { "files": 6, "loc": 450, "status": "valid" },
    "51-services": { "files": 8, "loc": 1200, "status": "valid" }
  },
  "validation": {
    "allStagesCompleted": true,
    "allFilesWritten": true,
    "noCollisions": true,
    "importsResolvable": true,
    "compilationSuccess": true
  },
  "phase3Complete": true,
  "readyForPhase4": true,
  "issues": []
}
```

Notes :

- Les clés précises dans `statistics` peuvent être adaptées à tes besoins (ex. `atomsGenerated`, `containersGenerated`, etc.).  
- `byStage` doit refléter au minimum le nombre de fichiers, LOC (si dispo) et un statut par stage.

### 4.2. `generation-report.md`

Fichier Markdown écrit sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/src_new/generation-report.md`

Contenu recommandé :

1. **En-tête**  
   - Page, date, statut global de Phase 3.  

2. **Résumé exécutif**  
   - Nombre de fichiers générés ;  
   - Volume approximatif de code ;  
   - Statut de la Phase 3 (succès, partiel, échec).  

3. **Statistiques par stage** (tableau)  

   | Stage | Domaine   | Fichiers | LOC  | Statut | Issues |
   |-------|-----------|----------|------|--------|--------|
   | 50    | types     | 6        | 450  | ✅     | 0      |
   | 51    | services  | 8        | 1200 | ✅     | 1 warn |

4. **Validation globale**  
   - Collisions de fichiers ;  
   - Imports résolvables (si checké) ;  
   - Résultat d’une éventuelle compilation (si renseigné dans les métadonnées).  

5. **Issues & warnings**  
   - Liste consolidée ;  
   - Classification bloquant / warning.  

6. **Prêt pour Phase 4**  
   - Décision explicite :  
     - ✅ « Phase 4 (functional-audit) peut démarrer »  
     - ou ❌ « Phase 4 ne peut pas démarrer pour les raisons suivantes : … »

---

## 5. ✅ Règles de décision `readyForPhase4`

### 5.1. Conditions minimales

Le flag `readyForPhase4` doit être **true** seulement si :

- tous les stages 50 → 61 ont :  
  - un meta de génération accessible,  
  - un `validation.status` non bloquant (ex. `"valid"`) ;  
- tous les fichiers attendus ont été écrits (ou au moins aucun fichier critique manquant) ;  
- aucune **issue bloquante** n’est listée (collisions, erreurs structurantes, etc.).  

### 5.2. Pseudocode de décision

```
readyForPhase4 = 
  allStagesCompleted AND
  allFilesWritten AND
  noCollisions AND
  noBlockingIssues
```

Où :

- `allStagesCompleted` : tous les stages 50 → 61 ont des métadonnées avec `validation.status` non bloquant ;  
- `allFilesWritten` : pas de `filesGenerated` critiques manquants ;  
- `noCollisions` : aucun `path` de fichier généré en double pour deux contenus incompatibles ;  
- `noBlockingIssues` : aucune issue marquée comme bloquante dans les métadonnées.  

Si l’une de ces conditions échoue → `readyForPhase4 = false`.

---

## 6. 🔍 Patterns de traitement

### 6.1. Agrégation des métadonnées

- Lire tous les fichiers `.meta/generation.*.meta.json` ;
- Remplir une structure indexée par `stageId` et/ou `domain` ;
- Agréger les stats (fichiers, LOC).

### 6.2. Détection de collisions

- Collecter tous les `filesGenerated[].path` ;  
- Détecter les doublons ;  
- En cas de collision, les consigner comme issues (et potentiellement bloquantes).

### 6.3. Validation des imports / compilation (optionnel)

Si certains stages fournissent déjà des résultats de check (ex. `importsResolvable`, `compilationSuccess`), les **remonter** dans :

- `generation-summary.json.validation.*` ;  
- et les exposer dans `generation-report.md`.

Le domaine `generation-summary` ne lance pas lui-même les outils de compilation ou de vérification d’import :  
il **agrège** uniquement les résultats fournis par d’autres étapes.

---

## 7. Gestion des erreurs

### 7.1. Erreurs bloquantes (Gate ❌ pour Stage 62)

- Un stage 50 → 61 en échec critique (ex. `Gate ❌` + fichiers essentiels manquants) ;
- Fichiers de sortie (`generation-summary.json`, `generation-report.md`) non écrits ;
- Schéma JSON invalide pour les fichiers de sortie (si la validation de schéma est activée).

### 7.2. Warnings non bloquants

- Informations manquantes sur certaines stats (LOC non renseignés, etc.) ;
- fichiers générés mais non référencés par les métadonnées ;
- incohérences mineures de formatage.

Ces warnings doivent être listés dans `issues[]`, mais ne bloquent pas forcément `readyForPhase4`.

---

## 8. ✅ Checklist de génération pour `generation-summary`

Avant de considérer que le résumé de génération est correct pour `${project.pageName}` :

- [ ] Tous les fichiers de métadonnées des stages 50 → 61 ont été lus ou marqués comme manquants  
- [ ] Les statistiques globales (`filesGenerated`, `linesOfCode`) sont agrégées correctement  
- [ ] Les statistiques par stage (`byStage`) sont cohérentes avec les métadonnées  
- [ ] Les issues et warnings des stages sont bien remontés dans le résumé  
- [ ] `generation-summary.json` est écrit et respecte la structure attendue  
- [ ] `generation-report.md` est généré avec un résumé lisible et une décision claire pour la Phase 4  
- [ ] `readyForPhase4` est cohérent avec l’état réel des stages 50 → 61  
- [ ] En cas d’erreur bloquante, Stage 62 est en `Gate ❌` et `readyForPhase4 = false`

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
