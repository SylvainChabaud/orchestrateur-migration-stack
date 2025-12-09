# 🚀 main-orchestrator.md — Point d’entrée utilisateur

## 🎯 But

Ce fichier sert à configurer puis lancer l’orchestrateur IA **en posant chaque question une par une**, dans un dialogue séquentiel.  
L’IA ne doit jamais poser la question suivante tant que l’utilisateur n’a **pas répondu à la précédente**.

L’orchestrateur doit :

1. Poser 4 questions **séquentielles** à l’utilisateur  
2. Mettre à jour `core/configs/project.config.yaml`  
3. Demander si l’utilisateur souhaite lancer l’orchestrateur  
4. Exécuter les stages dans l’ordre, en tenant compte de `runtime.regenerateStackGuides`

---

## 🧠 Instructions pour l’IA (obligatoires)

### ⚠️ RÈGLE ABSOLUE  
**Tu dois poser les questions UNE PAR UNE.**  
Après chaque réponse de l’utilisateur, tu continues au point suivant.  
Ne jamais poser plusieurs questions dans un seul message.

---

## 1️⃣ Séquence de questions

### **Question 1 — à poser seule**
« Quel est le nom de la page / module à migrer ?  
(ex: CampaignsDetail, UserProfile…) »

→ Une fois la réponse fournie, stocker la valeur dans :  
`project.pageName`

---

### **Question 2 — à poser seulement après réponse à Q1**
« Quel est le chemin exact du fichier legacy principal ?  
(ex: src/.../index.js) »

→ Stocker ensuite la valeur dans :  
`paths.legacySource`

---

### **Question 3 — à poser seulement après réponse à Q2**

« Je vais maintenant valider la configuration de ta stack finale. »

**Étape 3.1 : Chargement et validation technique**

1. Charger `${stack.custom}`
2. Vérifier :
   - ✅ Le fichier existe et est lisible
   - ✅ Le YAML est syntaxiquement valide
   - ✅ La section `metadata` est présente
   - ✅ La section `tools` est présente
   - ✅ Les sous-sections obligatoires de `metadata` sont présentes :
     - `architecture` (type, folderStructure, packageManagement)
     - `naming` (files, functions, constants)
     - `projectStructure` (srcLayout)
     - `performance` (heavyLibraries, optimization, targets)
     - `accessibility` (standard, requirements, tools, targets)
     - `qualityThresholds` (validation, globalScore)
     - `layouts` (available, responsive)
   - ✅ Les sections obligatoires de `tools` sont présentes :
     - `runtime`, `frontend`, `routing`, `i18n`, `design`, `stateManagement`,
     - `api`, `validation`, `forms`, `build`, `tests`, `auth`

Si un élément manque → afficher un message d'erreur détaillé et arrêter.

**Étape 3.2 : Afficher un résumé visuel**

```
╔══════════════════════════════════════════════════════════╗
║     📋 RÉSUMÉ DE LA STACK FINALE                        ║
╚══════════════════════════════════════════════════════════╝

🏗️  Architecture
    • Type : ${metadata.architecture.type}
    • Structure : ${metadata.architecture.folderStructure}
    • Package Manager : ${metadata.architecture.packageManagement}

🎨  Frontend Stack
    • Frontend : ${tools.frontend.library} ${tools.frontend.libraryVersion}
    • Runtime : ${tools.runtime.language} ${tools.runtime.languageVersion}
    • Design : ${tools.design.designSystem} ${tools.design.designSystemVersion}
    • UI Library : ${tools.design.uiLibrary} ${tools.design.uiLibraryVersion}

🔄  Core Libraries
    • Routing : ${tools.routing.router} ${tools.routing.routerVersion}
    • i18n : ${tools.i18n.library} ${tools.i18n.libraryVersion}
    • State (global) : ${tools.stateManagement.globalState.library} ${tools.stateManagement.globalState.version}
    • State (server) : ${tools.stateManagement.serverState.library} ${tools.stateManagement.serverState.version}

🔌  Data & Forms
    • API Client : ${tools.api.httpClient.library} ${tools.api.httpClient.version}
    • Forms : ${tools.forms.formLibrary.library} ${tools.forms.formLibrary.version}
    • Validation : ${tools.validation.schemaValidation.library} ${tools.validation.schemaValidation.version}

🧪  Testing & Quality
    • Unit Tests : ${tools.tests.unit.runner} ${tools.tests.unit.version}
    • Component Tests : ${tools.tests.component.library} ${tools.tests.component.version}
    • E2E : ${tools.tests.e2e.runner} ${tools.tests.e2e.version}
    • Build : ${tools.build.bundler.library} ${tools.build.bundler.version}

🔐  Authentication
    • OIDC : ${tools.auth.oidcClient.library} ${tools.auth.oidcClient.version}

📊  Métriques qualité définies
    • Performance TTI : ${metadata.performance.targets.tti}
    • Accessibility : ${metadata.accessibility.standard}
    • Test Coverage : ${tools.tests.unit.coverage}
    • Global Score min : ${metadata.qualityThresholds.globalScore.minGlobalScore}

📚  Guides qui seront générés : 26
    ├── Stack & Structure (5)
    ├── UI & Design (6)
    ├── State & Data (6)
    ├── Quality & Testing (5)
    └── Infrastructure (4)

╔══════════════════════════════════════════════════════════╗
```

**Étape 3.3 : Poser la question de validation**

« Cette configuration est-elle correcte ? (oui/non) »

- Si **non** → 
  ```
  ⚠️  Pour mettre à jour ta configuration :
  
  📄 Fichier : `${stack.custom}`
  
  📖 Guide de remplissage :
     .ai-tools/ai-orchestrator-v4/core/configs/stacks/STACK-YAML-TEMPLATE.md
  
  Sections à vérifier :
  ✓ metadata.architecture
  ✓ metadata.naming
  ✓ metadata.projectStructure
  ✓ metadata.performance
  ✓ metadata.accessibility
  ✓ metadata.qualityThresholds
  ✓ metadata.layouts
  ✓ tools.* (toutes les sections)
  
  Une fois les modifications effectuées, relance l'orchestrateur.
  ```
  → Arrêter le processus.

- Si **oui** →  
  Afficher :
  ```
  ✅ Configuration validée. Passage à l'étape suivante.
  ```
  → Passer à la Question 4.

---

### **Question 4 — à poser seulement après réponse à Q3**
« Souhaites-tu **regénérer les guides de stack (Stage 00)**,  
ou **réutiliser les guides déjà générés** ?  
Réponds par :  
- `regenerer`  
- `reutiliser` »

- Si `regenerer` → `runtime.regenerateStackGuides = true`  
- Si `reutiliser` → `runtime.regenerateStackGuides = false`

---

## 2️⃣ Mise à jour du fichier YAML

Une fois les 4 réponses obtenues :

- Charger `core/configs/project.config.yaml`
- Modifier **uniquement** :
  - `project.pageName`
  - `paths.legacySource`
  - `runtime.regenerateStackGuides`

Afficher ensuite le YAML mis à jour dans un bloc :

```yaml
# core/configs/project.config.yaml (mis à jour)
...
```

---

## 3️⃣ Proposer d’exécuter l’orchestrateur

Après avoir mis à jour la config, poser **seulement alors** :

> « Souhaites-tu lancer l’orchestrateur maintenant ? (oui / non) »

- Si **non** → s’arrêter immédiatement.  
- Si **oui** → passer à l’étape 4.

---

## 4️⃣ Lancer le pipeline complet

Si l’utilisateur a répondu **oui** :

1. Lire `core/configs/project.config.yaml`
2. Lire la valeur de `runtime.regenerateStackGuides`

### Cas A — regenerateStackGuides = true
Exécuter tous les stages dans l’ordre :

- Phase 0 (incluant `00-stack-guides-builder.md`)
- Phase 1  
- Phase 2  
- Phase 3  
- Phase 4  

### Cas B — regenerateStackGuides = false
- **Ne pas exécuter** `00-stack-guides-builder.md`
- Commencer à : `01-project-structure-spec-builder.md`
- Avant d’aller plus loin :
  - vérifier que les guides existent :
    - `stack-guides/`
    - `stack-guides-summary.json`
  - si absents → arrêter et dire :
    > « Les guides de stack ne sont pas présents.  
    > Remets `runtime.regenerateStackGuides` à `true` et relance. »

---

## 5️⃣ Logique d’exécution des stages

Pour chaque stage exécuté :

1. Ouvrir le markdown correspondant  
2. Exécuter ses instructions  
3. À la fin, l’IA du stage doit écrire un bloc :

```markdown
## 🧩 Gate
Gate ✅
```
ou

```markdown
## 🧩 Gate
Gate ❌
```

4. L’orchestrateur doit :
   - lire cette valeur dans **la réponse du stage** (pas dans le fichier markdown)
   - extraire le diagnostic complet si `Gate ❌`
   
   **Si `Gate ✅` :**
   - Logger le succès
   - Afficher : `✅ Stage ${stageId} (${stageName}) terminé avec succès`
   - Passer au stage suivant
   
   **Si `Gate ❌` :**
   - **ARRÊTER LE PIPELINE IMMÉDIATEMENT**
   - Afficher le rapport d'erreur complet avec :
     - Diagnostic du stage
     - Cause racine
     - Impact sur le pipeline
     - Solutions proposées
   - Proposer les options de récupération :
     1. Corriger et relancer ce stage
     2. Relancer depuis un stage antérieur
     3. Relancer toute la phase
     4. Afficher plus d'informations
     5. Abandonner le pipeline
   - **ATTENDRE la réponse de l'utilisateur**

5. **Gestion des commandes de récupération**
   
   - `"Relance le stage ${stageId}"` → réexécuter uniquement ce stage
   - `"Relance depuis le stage ${previousStageId}"` → réexécuter depuis ce point
   - `"Relance la Phase ${phaseNumber}"` → réexécuter toute la phase
   - `"Explique l'erreur du stage ${stageId}"` → afficher détails + guides
   - `"Arrête le pipeline"` → sauvegarder l'état et terminer

---

## 6️⃣ Fin du pipeline

Lorsque tous les stages ont passé `Gate ✅` :

Afficher où se trouvent dans `${paths.workspace}` :

- inventaires  
- mappings  
- code généré  
- rapports  
- archive  

et terminer avec :

> « 🎉 Migration terminée avec succès. »

---

© 2025 — ai-orchestrator-v4
