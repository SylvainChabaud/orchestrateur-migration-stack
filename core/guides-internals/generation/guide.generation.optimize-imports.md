# 🔧 Guide Génération — Imports (Optimisation)

*(Domaine de génération : **optimize-imports** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine de génération

Le domaine **optimize-imports** décrit comment :

- nettoyer,
- normaliser,
- et harmoniser

les **instructions d’import** dans tous les artefacts générés pour `${project.pageName}` en Phase 3.

L’objectif est de :

- supprimer les imports inutilisés ;
- appliquer un style cohérent d’imports (chemins, ordre, regroupement) ;
- réduire le bruit dans les diffs ;
- faciliter l’évolution et la lisibilité du code généré.

Ce domaine ne modifie **pas la logique métier** :  
il agit uniquement sur les **entêtes d’import / require / équivalents** en respectant les conventions décrites dans les **stack-guides d’imports**.

---

## 2. 🔌 Entrées du domaine de génération

### 2.1. Configuration & chemins

Depuis `core/configs/project.config.yaml` :

- `project.name`
- `project.pageName`
- `paths.root`
- `paths.core`
- `paths.workspace`
- `paths.legacySource`
- `paths.stages`
- `stack.custom`

Les fichiers à optimiser se trouvent sous :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/`

### 2.2. Artefacts Phase 0 — Stack & imports

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`
- `stack-guides/guide.stack.md`
- `stack-guides/guide.imports.md`

Le guide d’imports doit préciser :

- quels **langages / extensions** sont concernés (`.ts`, `.tsx`, `.js`, etc.) ;
- s’il faut privilégier les **imports absolus** ou **relatifs**, et via quels alias ;
- l’**ordre et le regroupement** des imports :
  - librairies externes ;
  - modules core ;
  - modules internes partagés ;
  - modules propres à la page ;
- les règles concernant :
  - la séparation types / valeurs ;
  - les alias de modules ;
  - les barrel files éventuels.

### 2.3. Artefacts Phase 3 — Fichiers cibles

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/` :

Les dossiers typiques contenant des imports :

- `services/`
- `stores/`
- `hooks-logic/`
- `hooks-data/`
- `components/atoms/`
- `components/containers/`
- `pages/`
- `routing/`
- `i18n/`
- `tests/`

Le domaine **optimize-imports** ne doit **jamais** intervenir sur :

- les fichiers d’inputs (legacy, inventaires, mappings) ;
- les fichiers d’autres phases ;
- les fichiers de meta (sauf mention contraire explicite).

### 2.4. DSL + UCR + bridge (optionnel)

- `Spec Dsl Orchestrator`
- `Spec Ucr Orchestrator`
- `bridge-legacy-to-dsl.json`

Ces artefacts sont secondaires ici ; ils peuvent toutefois aider à :

- ne pas supprimer des imports utilisés uniquement dans des annotations ou commentaires liés aux UCR ;
- comprendre la criticité d’un fichier (et donc la prudence à avoir).

---

## 3. 🧠 Règles générales d’optimisation des imports

### 3.1. Principe de non-régression

Les refactorings d’imports ne doivent **jamais** :

- introduire de comportements différents ;
- casser des exports publics ;
- altérer des signatures de fonctions / composants / classes.

Si un doute subsiste, l’optimisation doit rester conservatrice.

### 3.2. Suppression des imports inutilisés

Le domaine peut supprimer :

- les specifiers dont les symboles ne sont plus utilisés dans le fichier ;
- les instructions d’import entières qui n’ont plus de specifier.

Il doit toutefois :

- prendre en compte les usages indirects (ex. dans des annotations, configs stack) si possible ;
- s’appuyer sur un minimum d’analyse statique pour éviter les faux positifs.

### 3.3. Normalisation des chemins d’import

Selon `guide.imports.md`, le domaine doit :

- privilégier certains types de chemins :
  - alias de module (ex. `@app/…`) ;
  - chemins relatifs courts ;
- éviter les chemins cassants (`../../../../`) lorsqu’un alias existe ;
- corriger les incohérences (mélange de variantes de chemins pour le même module).

### 3.4. Regroupement et tri

Le domaine peut regrouper les imports lorsque cela améliore la lisibilité :

- fusionner plusieurs imports issus d’un même module ;
- regrouper les imports de valeurs et de types (ou les séparer selon les règles de la stack).

Il doit aussi :

- trier les imports par catégories et éventuellement par ordre alphabétique, si les stack-guides le recommandent ;
- conserver les commentaires structurants (bannières, TODO, tags spécifiques).

### 3.5. Conventions types / valeurs

Dans les stacks typées, les stack-guides peuvent imposer une séparation :

- imports de **types** ;
- imports de **valeurs**.

Le domaine doit :

- suivre ces conventions (ex. groupements spécifiques, mots-clés pour types) ;
- ne pas dégrader la distinction types/valeurs.

---

## 4. 🧬 Processus conceptuel d’optimisation

### 4.1. Découverte des fichiers éligibles

Pour `${project.pageName}` :

- lister tous les fichiers dans `phase-3-generation` correspondant aux extensions ciblées ;
- filtrer selon les règles d’exclusion :
  - dossiers ignorés ;
  - fichiers explicitement marqués comme non optimisables.

### 4.2. Parsing des imports

Pour chaque fichier :

- extraire les blocs d’import et, si nécessaire, d’export ;
- construire un modèle interne listant :
  - la source (module) ;
  - les specifiers (noms, alias, type/valeur) ;
  - les usages repérés dans le code.

### 4.3. Analyse d’usage

- analyser le reste du fichier pour :
  - compter les occurrences de chaque symbole importé ;
  - repérer les usages dans les types, les annotations, les commentaires structurés.

Les symboles sans usage (dans les limites de cette analyse) sont candidats à suppression.

### 4.4. Application des règles de style

- appliquer les règles de style définies dans `guide.imports.md` :
  - ordre des imports ;
  - regroupement ;
  - conversions de chemins ;
  - séparation types/valeurs ;
  - alias, etc.

### 4.5. Réécriture sécurisée

- réécrire le fichier avec des imports optimisés :  
  - conserver tout le corps du fichier inchangé ;
  - ne pas modifier ce qui suit les imports, sauf pour ajuster de très légers détails si la stack le demande (par ex. imports/exports groupés).

---

## 5. 🗂 Métadonnées d’optimisation : `imports.meta.json`

Le domaine doit produire, pour `${project.pageName}` :

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/imports/imports.meta.json`

Ce fichier contient au minimum :

```jsonc
{
  "pageName": "${project.pageName}",
  "filesScanned": 18,
  "filesModified": 14,
  "unusedImportsRemoved": 32,
  "importsNormalized": 27,
  "issues": [
    "Could not parse imports in components/containers/LegacyWrapper.ext"
  ]
}
```

Ce rapport est utile pour :

- diagnostiquer l’impact de l’optimisation ;
- repérer les fichiers problématiques ;
- alimenter les étapes de validation ou de reporting global (Phase 4).

---

## 6. ✅ Checklist de génération pour `optimize-imports`

Avant de considérer que le domaine `optimize-imports` est correctement appliqué :

- [ ] Les stack-guides d’imports (`guide.imports.md`) sont présents et interprétés  
- [ ] Les fichiers ciblés sont uniquement ceux de `phase-3-generation/` pour `${project.pageName}`  
- [ ] Les imports inutilisés évidents ont été supprimés (sans casser le code)  
- [ ] Les chemins d’import respectent les conventions d’alias / relatif définies par la stack  
- [ ] L’ordre des imports respecte les catégories (externes / internes / locaux) définies par la stack  
- [ ] Aucun changement de logique métier n’a été introduit  
- [ ] `imports.meta.json` a été généré avec les compteurs et les issues éventuelles  
- [ ] Les erreurs de parsing ou d’application des règles sont limitées à des cas isolés, sans impacter le reste du pipeline

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
