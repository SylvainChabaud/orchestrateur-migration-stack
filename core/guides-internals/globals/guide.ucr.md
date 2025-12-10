# 🧾 Guide Global — UCR (Unique Canonical Reference)

## 1. 🎯 Objectif

Les **UCR** (Unique Canonical References) sont des identifiants stables utilisés par tout l’orchestrateur pour :

- désigner de manière unique une entité (vue, logique, data, condition, event…) au sein d’une **page** ;
- permettre aux différents inventaires et mappings de parler de la **même chose** ;
- faciliter la génération et la validation en fin de pipeline.
> ⚠️ **Important** : Les UCR sont **créés en Phase 1** (Inventaires, stages 10-26).  
> Le bridge DSL créé en Phase 0 (stage 02) ne crée PAS d'UCR, il définit uniquement le catalogue de concepts DSL.
## 2. 🔐 Propriétés générales

1. Unicité locale à la page
2. Stabilité
3. Lisibilité

## 3. 🧱 Format des UCR

### 3.1. Regex générale

```regex
^[a-z0-9]+(?:-[a-z0-9]+)*$
```

### 3.2. Conventions par domaine

Chaque inventaire peut utiliser un préfixe cohérent :

| Domaine | Préfixe suggéré | Exemple |
|---------|-----------------|----------|
| Structure | `node-` | `node-root`, `node-table-campaigns` |
| Layout | `layout-` | `layout-main-grid`, `layout-sidebar` |
| Styles | `style-` | `style-theme-dark`, `style-btn-primary` |
| i18n | `i18n-` | `i18n-title-page`, `i18n-error-required` |
| Config | `cfg-` | `cfg-api-endpoint`, `cfg-feature-flag-x` |
| Logic | `logic-` | `logic-validate-form`, `logic-compute-total` |
| Conditions | `cond-` | `cond-is-admin`, `cond-has-rights` |
| Hooks | `hook-` | `hook-use-campaigns`, `hook-use-form-state` |
| Events | `evt-` | `evt-click-submit`, `evt-change-filter` |
| Dataflows | `flow-` | `flow-fetch-campaigns`, `flow-save-draft` |
| Async | `async-` | `async-retry-policy`, `async-polling-status` |
| Services | `svc-` | `svc-campaign-api`, `svc-auth-client` |
| Routing | `route-` | `route-campaigns-list`, `route-detail-id` |
| Effects | `effect-` | `effect-focus-input`, `effect-scroll-top` |
| Actions | `action-` | `action-create-campaign`, `action-delete-item` |
| Tests | `test-` | `test-create-success`, `test-validation-error` |

> 💡 **Note** : Ces préfixes sont **recommandés** mais pas obligatoires.  
> L'important est la cohérence au sein d'un inventaire donné.

### 3.3. Exemples complets

```
node-root
node-campaigns-detail-header
node-form-create-campaign
layout-main-grid-3cols
style-theme-enterprise-light
i18n-page-title-campaigns
cfg-api-base-url
logic-validate-required-fields
cond-user-has-edit-rights
hook-use-campaign-query
evt-click-save-button
flow-fetch-campaigns-paginated
async-retry-on-network-error
svc-campaigns-api-client
route-campaigns-detail-by-id
effect-focus-first-error-field
action-submit-campaign-form
test-scenario-create-campaign-success
```

## 4. 🧩 Rôles des inventaires

### 4.1. Inventaire Structure (stage 10)

- **Crée les UCR principales** : tous les nœuds de vue (root, containers, presentational, fragments)
- Format : `node-<nom>-<discriminant>`
- Ces UCR sont **réutilisés** par tous les autres inventaires via références

### 4.2. Autres inventaires (stages 11-25)

**Deux stratégies possibles :**

1. **Référencement** (le plus courant) :
   - L'inventaire **ne crée pas** de nouvelles UCR
   - Il référence les UCR de structure existantes via `structureUcrs[]`
   - Exemples : Layout, Styles, i18n, Routing (souvent liés à des vues)

2. **Création locale** (pour entités indépendantes) :
   - L'inventaire **crée ses propres UCR** pour des entités non directement liées à une vue
   - Exemples :
     - **Dataflows** : `flow-fetch-campaigns` (peut exister indépendamment d'une vue)
     - **Services** : `svc-campaigns-api` (façade technique réutilisable)
     - **Tests** : `test-scenario-create-success` (scénario E2E transverse)
     - **Actions** : `action-submit-form` (chaîne logique end-to-end)
     - **Hooks** : `hook-use-form-validation` (hook custom réutilisable)

### 4.3. Règle de cohérence

Si une entité est **directement attachée à une vue** :
- Ne pas créer de nouvelle UCR
- Référencer l'UCR de structure via `structureUcrs[]` ou `parentUcr`

Si une entité est **transverse ou réutilisable** :
- Créer une UCR locale avec préfixe approprié
- Documenter les liens via `relations.*Ucrs[]`

## 5. 🔁 Cycle de vie des UCR

### Phase 0 — Bootstrap (stages 00-02)
- ❌ **Aucune UCR n'est créée**
- Stage 02 crée uniquement le **bridge DSL** (catalogue de concepts : `structure.*`, `logic.*`, etc.)

### Phase 1 — Analysis (stages 10-26)
- ✅ **Création des UCR** dans les inventaires
- Stage 10 (Structure) : UCR principales (`node-*`)
- Stages 11-25 : UCR secondaires ou références aux UCR de structure
- Format JSON : `{ "ucr": "node-root", "dslTags": ["structure.rootView"], ... }`

### Phase 2 — Interpretation (stages 30-46)
- ✅ **Référencement des UCR** depuis les inventaires
- Chaque mapping item référence : `sourceInventoryRef.itemUcr`
- Création UCR de mapping : `map-<domain>-${item.ucr}` (ex: `map-structure-node-root`)
- Relations : `relations.structureUcrs[]`, `relations.logicUcrs[]`, etc.

### Phase 3 — Generation (stages 50-62)
- ✅ **Traçabilité UCR** dans le code généré
- Commentaires de traçabilité : `/* UCR: node-root | Mapping: map-structure-node-root */`
- Meta.json : `ucrTrace[]` pour chaque fichier généré
- Permet de remonter du code généré vers le legacy

### Phase 4 — Validation (future)
- ✅ **Vérification de la couverture UCR**
- Tous les UCR d'inventaire ont-ils un mapping ?
- Tous les mappings ont-ils généré du code ?
- UCR orphelins, manquants, ou dupliqués ?  

## 6. 🔧 Règles IA

### 6.1. Unicité stricte

❌ **Interdit** :
```json
{ "ucr": "node-form", ... }
{ "ucr": "node-form", ... }  // Doublon !
```

✅ **Correct** :
```json
{ "ucr": "node-form-create", ... }
{ "ucr": "node-form-edit", ... }
```

### 6.2. Stabilité (même entité = même UCR)

Si le code Legacy ne change pas, l'UCR ne doit pas changer entre deux exécutions.

**Stratégie** :
- Baser l'UCR sur des éléments stables : nom de composant, fonction, rôle métier
- Éviter les indices générés aléatoirement
- Si besoin d'un discriminant : utiliser l'ordre d'apparition dans le code

### 6.3. Lisibilité

❌ **Éviter** :
```
node-c1
node-x42
node-tmp-abc123
```

✅ **Préférer** :
```
node-campaigns-list-table
node-filter-panel
node-action-buttons-group
```

### 6.4. Résolution des ambiguïtés

**Cas 1 : Plusieurs composants similaires**

Si plusieurs composants ont le même nom logique, ajouter un discriminant :
```
node-modal-confirm-delete
node-modal-confirm-archive
node-modal-confirm-duplicate
```

**Cas 2 : Composant anonyme ou inconnu**

Si impossible de déterminer le rôle :
```json
{
  "ucr": "node-unknown-1",
  "metadata": { "reason": "anonymous function component without displayName" },
  "validation": {
    "issues": ["UCR could not be determined from code structure"]
  }
}
```

**Cas 3 : Composant dynamique**

Pour des composants générés dynamiquement :
```
node-dynamic-card-0
node-dynamic-card-1
node-dynamic-card-2
```
OU
```
node-campaign-card-featured
node-campaign-card-regular
```
(préférer la deuxième approche si le rôle est identifiable)

### 6.5. Cohérence cross-inventaire

**Règle** : Si un inventaire référence une UCR de structure, cette UCR **doit exister** dans `inventory.structure.json`.

✅ **Correct** :
```json
// inventory.structure.json
{ "ucr": "node-form-campaign", ... }

// inventory.events.json
{
  "ucr": "evt-submit-form",
  "structureUcrs": ["node-form-campaign"],  // Référence valide
  ...
}
```

❌ **Invalide** :
```json
// inventory.structure.json
{ "ucr": "node-form-campaign", ... }

// inventory.events.json
{
  "ucr": "evt-submit-form",
  "structureUcrs": ["node-form-xxx"],  // Référence inexistante !
  ...
}
```

### 6.6. Checklist avant génération inventaire

- [ ] Tous les UCR respectent le format regex
- [ ] Pas de doublons d'UCR dans l'inventaire
- [ ] Toutes les références UCR (`parentUcr`, `childrenUcrs[]`, `structureUcrs[]`) pointent vers des UCR existants
- [ ] Les UCR sont stables (reproductibles si on relance sur le même code)
- [ ] Les UCR sont lisibles (pas de `node-1`, `node-tmp`, etc.)

---

© 2025 — ai-orchestrator-v4
