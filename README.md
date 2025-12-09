# 🤖 AI Orchestrator V4 - Migration Legacy vers Stack Moderne

**Version :** 4.0 ✅ **OPÉRATIONNEL**  
**Date :** 7 Décembre 2025  
**Auteur :** Sylvain Chabaud

## 🚀 Démarrage rapide

Pour lancer l'orchestrateur, ouvre une conversation avec GitHub Copilot et tape :
> ```
> Exécute main-orchestrator.md
> ```

L'IA te guidera ensuite pas à pas pour configurer et lancer la migration.

---


## 📋 Vue d'ensemble

L'AI Orchestrator V4 est un système guidé par IA pour migrer automatiquement des pages d'une stack Legacy (React ancien, Vue, Angular...) vers une stack moderne cible (React 19, TypeScript, design system moderne, etc.).

Le processus est découpé en **4 phases** progressives avec validation à chaque étape via un système de **Gates**.

---

## 🎯 Objectif

**Entrée :** Une page Legacy existante  
**Sortie :** La même page migrée vers la stack moderne, avec :
- ✅ Code généré selon les standards de la stack cible
- ✅ Tests migrés et adaptés
- ✅ Documentation de la migration
- ✅ Traçabilité complète via UCR (Unique Canonical References)

---

## 📐 Architecture en 4 Phases

```
Phase 0: BOOTSTRAP (Stages 00, 01, 02)
   └─> Génération des guides de stack, validation cohérence, bridge Legacy↔DSL

Phase 1: ANALYSIS (Stages 10-26)
   └─> 16 inventaires du code Legacy + synthèse

Phase 2: INTERPRETATION (Stages 30-46)
   └─> 16 mappings Legacy→Stack Cible + synthèse

Phase 3: GENERATION (Stages 50-62) ✅ 13 STAGES
   └─> Génération du code moderne : types, services, stores, hooks, components, pages, routing, i18n, tests

Phase 4: VALIDATION (Stages 70-72)
   └─> Validation fonctionnelle et archivage
```

---

## 🚀 Démarrage rapide

### 1. Configuration

Éditer le fichier de configuration unique :

```yaml
# core/configs/project-config.yaml

project:
  pageName: "CampaignsDetail"  # ← Nom de la page à migrer

paths:
  legacySource: "src/packages/promo-boost/components/campaignsDetail/index.js"  # ← Chemin vers le code Legacy
```

### 2. Exécution

Demander à l'IA :

```
Lis le fichier core/configs/project-config.yaml, puis exécute tous les 
stages markdown en suivant l'ordre défini dans la section "stages" 
(phase 0 → phase 4), en t'arrêtant au premier Gate ❌ rencontré.
```

### 3. Suivi

L'orchestrateur génère tous les artefacts dans :
```
.ai-tools/ai-orchestrator-v4/workspace/projects/legacy-to-newStack-migration/
├── stack/                          ← Config commune (Phase 0)
└── pages/CampaignsDetail/          ← Artefacts de la page
    ├── phase-1-analysis/
    ├── phase-2-interpretation/
    ├── phase-3-generation/
    └── phase-4-validation/
```

---

## 📂 Structure du projet

```
.ai-tools/ai-orchestrator-v4/
├── core/                           ← Socle de l'orchestrateur (NE PAS MODIFIER)
│   ├── configs/
│   │   ├── project-config.yaml     ← Configuration principale
│   │   └── stacks/                 ← Définitions des stacks
│   ├── guides-internals/           ← Guides internes pour l'IA
│   │   ├── inventory/              ← Guides pour Phase 1
│   │   ├── mapping/                ← Guides pour Phase 2
│   │   └── globals/                ← Guides transverses (UCR, validation...)
│   ├── schemas/                    ← Schémas JSON de validation
│   │   ├── inventory.*.schema.json
│   │   └── mapping.*.schema.json
│   └── stages/                     ← Stages markdown (instructions IA)
│       ├── phase-0-bootstrap/
│       ├── phase-1-analysis/
│       ├── phase-2-interpretation/
│       ├── phase-3-generation/
│       └── phase-4-validation/
│
└── workspace/                      ← Tout ce qui est généré (GITIGNORE)
    └── projects/
        └── legacy-to-newStack-migration/
            ├── stack/
            └── pages/
```

---

## 🔑 Concepts clés

### UCR (Unique Canonical Reference)
Identifiant stable et unique pour chaque élément analysé/généré. Permet la traçabilité entre phases.

**Exemple :** `inv-struct-CampaignsDetail-root-view`

### DSL Interne
Langage de description générique des concepts (structure, layout, logic, events...) indépendant du framework Legacy.

**Exemple :** `structure.rootView`, `logic.formValidation`, `events.onClick`

### Bridge Legacy↔DSL
Mapping automatique entre les patterns du code Legacy et les concepts DSL génériques.

### Gates
Mécanisme de validation à la fin de chaque stage :
- **Gate ✅** : Le stage a réussi, continuer
- **Gate ❌** : Le stage a échoué, arrêter le pipeline

---

## 📊 Phases détaillées

### Phase 0 : Bootstrap (Stages 00-02)

**Objectif :** Préparer le contexte de migration

| Stage | Nom | Description |
|-------|-----|-------------|
| 00 | Stack Guides Builder | Génère les guides de la stack cible (React 19, routing, state, etc.) |
| 01 | Project Structure Spec | Définit la structure de dossiers du projet cible |
| 02 | Legacy→DSL Bridge | Crée le mapping Legacy↔DSL automatiquement |

**Outputs :**
- `stack/stack-guides/*.md`
- `stack/project-structure.json`
- `stack/bridge-legacy-to-dsl.json`

---

### Phase 1 : Analysis (Stages 10-26)

**Objectif :** Inventorier exhaustivement le code Legacy

**16 domaines d'inventaire :**
- Structure (10), Layout (11), Styles (12), i18n (13)
- Config (14), Logic (15), Conditions (16), Hooks (17)
- Events (18), Dataflows (19), Async (20), Services (21)
- Routing (22), Effects (23), Actions (24), Tests (25)
- **Synthèse (26)**

**Output :** Un JSON par domaine dans `phase-1-analysis/inventories/`

**Exemple :**
```json
{
  "domain": "structure",
  "pageName": "CampaignsDetail",
  "items": [
    {
      "ucr": "inv-struct-CampaignsDetail-root",
      "name": "CampaignsDetail",
      "type": "root",
      "childrenUcrs": ["inv-struct-CampaignsDetail-form", ...]
    }
  ]
}
```

---

### Phase 2 : Interpretation (Stages 30-46)

**Objectif :** Mapper chaque élément Legacy vers la stack cible

**16 domaines de mapping** (correspondant aux inventaires) :
- Structure (30), Layout (31), Styles (32), i18n (33)
- Config (34), Logic (35), Conditions (36), Hooks (37)
- Events (38), Dataflows (39), Async (40), Services (41)
- Routing (42), Effects (43), Actions (44), Tests (45)
- **Synthèse (46)**

**Output :** Un JSON par domaine dans `phase-2-interpretation/mappings/`

**Exemple :**
```json
{
  "domain": "structure",
  "items": [
    {
      "ucr": "map-struct-CampaignsDetail-root",
      "fromDsl": "structure.rootView",
      "toStack": {
        "stackKind": "pageComponent",
        "targetId": "CampaignsDetailPage",
        "targetPath": "src/pages/CampaignsDetail/CampaignsDetailPage.tsx",
        "targetLayer": "presentation"
      }
    }
  ]
}
```

---

### Phase 3 : Generation (Stages 50-58) 🚧 EN DÉVELOPPEMENT

**Objectif :** Générer le code moderne

Stages prévus :
- Components (50), Hooks (51), Services & Dataflows (52)
- Styles (53), Pages & Routing (54), i18n (55)
- Tests (56), Import Optimizer (57)
- **Synthèse (58)**

---

### Phase 4 : Validation (Stages 70-72) 🚧 EN DÉVELOPPEMENT

**Objectif :** Valider et archiver

Stages prévus :
- Functional Audit (70)
- Final Validation (71)
- Archive Run (72)

---

## ⚙️ Configuration avancée

### Validation des schémas JSON

```yaml
validation:
  # Activer la validation des outputs contre les schémas JSON
  enableSchemaValidation: true
  
  # Mode de validation
  # "strict" → Gate ❌ si erreur de schéma
  # "warning" → warnings uniquement
  validationMode: "strict"
```

**Schémas disponibles :**
- ✅ `inventory.structure.schema.json`
- ✅ `mapping.structure.schema.json`
- 🚧 Autres domaines en cours de création

### Régénération des guides de stack

```yaml
runtime:
  # Si true → exécute le Stage 00
  # Si false → skip le Stage 00
  regenerateStackGuides: false
```

---

## 📖 Documentation

### Guides internes (pour l'IA)
- `core/guides-internals/inventory/` - Guides d'inventaire
- `core/guides-internals/mapping/` - Guides de mapping
- `core/guides-internals/globals/` - Guides transverses

### Documentation utilisateur
- Ce README
- `CORRECTIONS-2025-12-07.md` - Historique des corrections
- `core/schemas/README.md` - Documentation des schémas JSON

---

## 🐛 Débogage

### Vérifier la configuration
```bash
cat core/configs/project-config.yaml
```

### Vérifier les outputs d'une phase
```bash
ls -la workspace/projects/*/pages/*/phase-1-analysis/inventories/
```

### Vérifier les Gates
Chercher "Gate ❌" dans les logs de l'IA pour identifier où le pipeline a échoué.

### Vérifier la validation
```bash
# Vérifier qu'un inventaire est valide
cat workspace/projects/*/pages/*/phase-1-analysis/inventories/inventory.structure.json | jq '.validation'
```

---

## 🔄 Changelog

### v4.1 (7 décembre 2025)
- ✅ Correction incohérence project.name/pageName
- ✅ Ajout validation JSON Schema optionnelle
- ✅ Création schémas structure (inventory + mapping)
- ✅ Documentation complète de la validation

### v4.0 (novembre 2025)
- ✅ Phases 0, 1, 2 complètes
- ✅ 16 domaines d'inventaire
- ✅ 16 domaines de mapping
- ✅ Système de Gates
- ✅ Concept UCR
- ✅ Bridge Legacy↔DSL

---

## 📊 État actuel

| Phase | Progression | État |
|-------|-------------|------|
| Phase 0 | 100% | ✅ Complet |
| Phase 1 | 100% | ✅ Complet |
| Phase 2 | 100% | ✅ Complet |
| Phase 3 | 0% | 🚧 En développement |
| Phase 4 | 0% | 🚧 En développement |
| **Total** | **45%** | ⚠️ Partiellement fonctionnel |

---

## 🤝 Contribution

### Pour ajouter un nouveau domaine d'inventaire/mapping
1. Créer le stage dans `core/stages/`
2. Créer le guide dans `core/guides-internals/`
3. Créer le schéma dans `core/schemas/`
4. Référencer dans `project-config.yaml`
5. Documenter dans ce README

### Pour corriger un bug
1. Identifier le stage concerné
2. Modifier le fichier markdown du stage
3. Tester avec l'IA
4. Documenter dans `CORRECTIONS-*.md`

---

## 📞 Support

Pour toute question :
- Consulter les guides dans `core/guides-internals/`
- Vérifier `CORRECTIONS-2025-12-07.md`
- Analyser les fichiers de validation dans `workspace/`

---

## 📜 Licence

© 2025 Sylvain Chabaud - Usage interne

---

**Note :** Les Phases 3 et 4 sont en cours de développement. L'orchestrateur est actuellement fonctionnel pour l'analyse et l'interprétation (Phases 0-2).
