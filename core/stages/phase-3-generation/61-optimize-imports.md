# 🧩 Stage 61 – optimize-imports
**Phase:** Phase 3 – Generation  
**Prev:** 60 – generate-tests  
**Next:** 62 – generation-summary  

---

## 🎯 Objective

Optimiser **tous les imports** des artefacts générés pour `${project.pageName}` en Phase 3, en :

- supprimant les imports inutilisés ;
- unifiant les chemins d’import (relatifs/absolus) conformément aux stack-guides ;
- regroupant / factorisant certains imports lorsque la stack le recommande ;
- appliquant les conventions de style (ordre, alias, regroupement) décrites dans les stack-guides.

Le stage **ne modifie aucune logique métier** :  
il ne fait qu’ajuster les **instructions d’import** dans les fichiers générés de Phase 3 pour `${project.pageName}`.

Il reste **agnostique de la stack et du langage** (TS, JS, autre) :  
la forme exacte des imports, leurs mots-clés et options viennent des **stack-guides**.

---

## 🔌 Inputs

> Toutes les entrées sont résolues à partir de `core/configs/project.config.yaml`.  
> Aucun chemin absolu ne doit être codé en dur.

### 1. Configuration globale

Depuis `core/configs/project.config.yaml` :

- `project.name`  
- `project.pageName`  
- `paths.root`  
- `paths.core`  
- `paths.workspace`  
- `paths.legacySource`  
- `paths.stages`  
- `stack.custom`  
- `gates.*`  
- `stages.*`  

### 2. Artefacts Phase 0 – Stack & structure

Depuis `${paths.workspace}/projects/${project.name}/stack/` :

- `project-structure.json`  
- `stack-guides/guide.stack.md`  
- `stack-guides/guide.imports.md` (obligatoire)  

Le stack-guide `guide.imports.md` définit au minimum :

- le **style d’import** recommandé (ex. import absolu vs relatif, alias de modules, etc.) ;
- l’**ordre des imports** (librairies externes, modules core, modules internes…) ;
- les conventions de **regroupement** (imports multi-lignes, types vs valeurs, etc.) ;
- les règles spécifiques à la stack (ex. chemins racine, alias, barrel files).

### 3. Artefacts Phase 3 – Fichiers à optimiser

Depuis `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/` :

Tous les dossiers générés par les stages précédents peuvent contenir des imports à optimiser, notamment :

- `services/` (stage 51)  
- `stores/` (stage 52)  
- `hooks-logic/` (stage 53)  
- `hooks-data/` (stage 54)  
- `components/atoms/` (stage 55)  
- `components/containers/` (stage 56)  
- `pages/` (stage 57)  
- `routing/` (stage 58)  
- `i18n/` (stage 59)  
- `tests/` (stage 60)  

Le stage 61 ne doit intervenir **que** dans `phase-3-generation/` pour `${project.pageName}`.

### 4. DSL, UCR et bridge (optionnel pour ce stage)

- `Spec Dsl Orchestrator`  
- `Spec Ucr Orchestrator`  
- `bridge-legacy-to-dsl.json`  

Ces artefacts ne sont pas strictement nécessaires à l’optimisation des imports, mais peuvent être utiles pour :

- conserver des commentaires / métadonnées UCR en place lors du refactoring ;
- s’assurer de ne pas supprimer des imports utilisés uniquement dans des annotations UCR.

---

## 📤 Outputs

Ce stage ne crée pas de nouveaux artefacts métier ; il :

1. **réécrit les fichiers** existants dans `phase-3-generation/` pour y optimiser les imports ;  
2. génère un fichier de **métadonnées d’optimisation** :  

   - `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/imports/imports.meta.json`  

   Contenant au minimum :

   ```jsonc
   {
     "pageName": "${project.pageName}",
     "filesScanned": 0,
     "filesModified": 0,
     "unusedImportsRemoved": 0,
     "importsNormalized": 0,
     "issues": []
   }
   ```

Règles :

- ne pas déplacer des fichiers ;
- ne pas modifier les signatures publiques (exports) ;
- ne pas modifier le contenu métier, uniquement les instructions d’import (et éventuellement les exports associés, si guidé par `guide.imports.md`).

---

## 🧠 Actions

### Étape 1 – Charger configuration & stack-guides

1.1. Charger `core/configs/project.config.yaml` et résoudre `${paths.*}`.  
1.2. Charger `project-structure.json`.  
1.3. Charger `stack-guides/guide.imports.md` pour :  

- lire les préférences de style d’import ;
- connaître les alias root éventuels ;
- connaître l’ordre souhaité des imports (externes, internes, locaux…).  

### Étape 2 – Découvrir les fichiers à traiter

2.1. Lister récursivement tous les fichiers **éligibles à l’optimisation** dans :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/`  

en respectant les extensions/langages définis par la stack (ex. `.ts`, `.tsx`, `.js`, `.jsx`, etc., si précisés dans `guide.stack.md` ou `guide.imports.md`).  

2.2. Exclure explicitement :  

- les fichiers purement data (JSON de meta, schema, etc.),
- les fichiers indiqués comme exclus de l’optimisation dans `project-structure.json` ou `guide.imports.md`.  

### Étape 3 – Analyser les imports de chaque fichier

Pour chaque fichier cible :

3.1. Parser les instructions d’import / require / équivalent, en suivant les instructions génériques de `guide.imports.md`.  
3.2. Construire une représentation interne :  

```jsonc
{
  "file": "components/containers/MyContainer.ext",
  "imports": [
    {
      "raw": "import ... from '...'",
      "source": "module/path",
      "specifiers": [
        { "name": "Something", "alias": null, "isType": false }
      ]
    }
  ],
  "exports": [...],
  "usage": {
    "Something": true
  }
}
```

3.3. Détecter les symboles **non utilisés** dans le fichier (via analyse statique minimale).  

### Étape 4 – Appliquer les règles d’optimisation

En se basant sur `guide.imports.md` :

4.1. **Suppression des imports inutilisés** :  

- supprimer les specifiers non utilisés ;
- si un import n’a plus aucun specifier, supprimer l’instruction d’import complète.  

4.2. **Normalisation des chemins d’import** :  

- convertir certains imports relatifs en imports alias (ou l’inverse) si prescrit par les stack-guides ;  
- corriger les chemins `../..` en chemins plus stables si un alias root est défini.  

4.3. **Regroupement / réorganisation** :  

- regrouper les imports provenant du même module si séparés ;  
- trier les import par catégories (externes → internes → locaux) et éventuellement par ordre alphabétique, selon les conventions ;  
- séparer les imports de type (ex. type-only) si la stack le recommande.  

4.4. **Préservation de la lisibilité** :  

- si des commentaires spécifiques (ex. tags UCR, warnings) sont attachés à un import, les conserver au bon endroit ;  
- éviter les refactorings qui rendraient le fichier difficile à relire (ex. regroupements extrêmes).  

### Étape 5 – Réécrire les fichiers

5.1. Si des modifications ont été appliquées à un fichier (imports supprimés / modifiés / réordonnés) :  

- réécrire le fichier avec les imports optimisés en tête, suivis du contenu inchangé.  

5.2. S’assurer que :  

- la structure globale du fichier est respectée ;  
- les exports publics ne sont pas altérés, sauf si `guide.imports.md` le prévoit explicitement (ex. export barrel).  

### Étape 6 – Mise à jour des métadonnées `imports.meta.json`

6.1. Pour chaque fichier scanné, mettre à jour un accumulateur :  

- `filesScanned` ;  
- `filesModified` ;  
- `unusedImportsRemoved` (compteur global) ;  
- `importsNormalized` (nombre de modifications de normalisation) ;  
- `issues` le cas échéant (ex. parsing impossible, règles stack-guides ambiguës…).  

6.2. Sérialiser ce rapport dans :  

- `${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-3-generation/imports/imports.meta.json`  

### Étape 7 – Validation et Gate

7.1. Le stage **ne doit pas** empêcher la suite de la pipeline si :  

- seuls des optimisations non-critiques échouent ;  
- certains fichiers ne peuvent pas être parsés (ils doivent simplement être listés dans `issues`).  

7.2. Cas typiques de **Gate ❌** :  

- erreurs systémiques (par ex. aucun fichier ne peut être analysé à cause d’un mauvais paramétrage) ;  
- impossibilité d’écrire `imports.meta.json`.  

7.3. Sinon, si au moins un passage d’optimisation a été mené à bien et que `imports.meta.json` est écrit → **Gate ✅**.

---

## 🧩 Gate

```markdown
## 🧩 Gate
Gate ✅
```

ou

```markdown
## 🧩 Gate
Gate ❌
```

Utiliser `Gate ❌` seulement en cas de problème bloquant global (configuration corrompue, impossibilité d’écrire les sorties).

---

## 📦 Next

> Continuer avec `62-generation-summary.md` si `Gate ✅`.

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
