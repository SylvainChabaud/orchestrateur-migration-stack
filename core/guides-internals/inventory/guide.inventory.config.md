# 🔧 Guide Inventaire — Config (`inventory.config`)

*(Domaine d’inventaire : **Configuration** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **Config** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **paramètres de configuration** (feature flags, env vars, runtime settings, options, limites…).
2. Leurs **sources** (env, fichiers de config, modules, constantes).
3. Les **usages** dans la page (quelles vues / comportements ils pilotent).
4. Les **impacts fonctionnels** principaux (désactivation de sections, modification de comportements, etc.).

Il répond à la question :

> **“Quels paramètres configurables influencent cette page, d’où viennent-ils et sur quoi agissent-ils ?”**

Ce domaine ne :

- ne décrit pas les détails de la logique métier (`inventory.logic`),
- ne décrit pas les données (APIs, requêtes — `inventory.dataflows`, `inventory.services`),
- ne s’occupe pas du layout ni des styles (déjà couverts ailleurs).

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.config.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"config"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des paramètres de configuration (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "config",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/pages/SamplePage/index.js",
  "items": [],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

### 2.2. Schéma interne — `items[]`

Chaque élément de `items[]` représente un **paramètre de configuration logique** (*ConfigItem*).

```text
items[] : ConfigItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) du paramètre de config, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire Config.

- `configName` : string  
  Nom logique du paramètre, par exemple :
  - `"ENABLE_PROMOBOOST_EXPERIMENT"`,
  - `"campaignsDetailMaxItems"`,
  - `"apiBaseUrl"`,
  - `"showAdvancedFilters"`.

- `kind` : string  
  Type de paramètre parmi un ensemble contrôlé, par exemple :
  - `"featureFlag"`,
  - `"environmentVariable"`,
  - `"runtimeSetting"`,
  - `"experimentToggle"`,
  - `"pageSetting"`,
  - `"uiToggle"`.

- `source` : object  
  Décrit la **source technique** du paramètre, par exemple :
  - `type`: `"env" | "file" | "module" | "inline"`  
  - `identifier`: nom exact (ex. `"process.env.PROMOBOOST_ENABLED"`, `"config.campaignsDetail.maxItems"`),
  - `sourcePath`: chemin du fichier principal où il est défini.

- `targetStructureUcrs` : array de string  
  Liste des `ucr` de Structure (issus de `inventory.structure.json`) impactés par ce paramètre de configuration.

- `behaviorSummary` : string  
  Description courte et fonctionnelle de l’impact du paramètre, par exemple :
  - `"Active/désactive le bloc 'Promotions avancées'"`,
  - `"Limite le nombre de lignes visibles dans le tableau"`,
  - `"Change l’URL de base des appels API"`.

- `metadata` : object  
  Informations additionnelles, par exemple :
  - `defaultValue`: valeur par défaut,
  - `possibleValues`: tableau si connu,
  - `isDeprecated`: booléen,
  - `notes`: string optionnel.  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `constraints` : object  
  Contraintes connues sur la valeur :
  - `min`, `max`,
  - `allowedValues`,
  - `pattern` (regex logique si nécessaire).

- `dependsOn` : array de string  
  Liste de `configName` d’autres paramètres dont celui-ci dépend (logiquement).

- `relatedFeatures` : array de string  
  Liste de noms logiques de features ou modules fonctionnels impactés.

- `severity` : string  
  Impact en cas de mauvaise configuration, par ex. `"low"`, `"medium"`, `"high"`.

Tout champ optionnel utilisé doit être **documenté** ici et respecté dans toute la pipeline.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` de config doivent être **uniques** dans `inventory.config.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Si le DSL interne définit un domaine `config.*`, il peut inclure par exemple :

- `config.featureFlag`
- `config.environmentVariable`
- `config.runtimeSetting`
- `config.experimentToggle`
- `config.pageLevelSetting`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) peut fournir les patterns pour reconnaître ces usages dans le code.  
Si ces concepts n’existent pas encore, l’analyse se base sur :

- les modules `config`, `settings`, `featureFlags`, etc.,
- les usages d’`env` (`process.env.*`, etc.),
- les constantes globales qui pilotent le comportement de la page.

### 3.2. Règles d’analyse

L’inventaire Config doit :

1. Partir de `${paths.legacySource}` et :
   - repérer les **imports** de configuration,
   - repérer l’utilisation de **variables d’environnement**,
   - détecter les accès à des modules de **feature flags** ou **d’expérimentation**,
   - identifier les **constantes** qui conditionnent visiblement le comportement.
2. Pour chaque paramètre logique :
   - lui donner un `configName` stable,
   - documenter sa `source` (type, identifiant, chemin),
   - lier le paramètre aux `targetStructureUcrs` (vues impactées),
   - résumer son impact dans `behaviorSummary`,
   - capturer les valeurs importantes dans `metadata`.
3. Regrouper les occurrences d’un même paramètre dans un seul `ConfigItem`.

### 3.3. Restrictions

L’inventaire Config **ne doit pas** :

- s’occuper des détails de logique métier,
- modéliser toute la configuration globale de l’application (uniquement ce qui est **pertinent pour la page**),
- décider des mécanismes d’injection de config dans la stack cible,
- lier directement la config à des endpoints ou à des schémas de données (cela appartient aux inventaires Data).

---

## 4. 🔗 Relations avec les autres inventaires

- **Config ← Structure**
  - Utilise les `ucr` des vues pour documenter où la config a un impact concret.

- **Config ↔ Logic**
  - De nombreux paramètres piloteront des branches de logique.  
    Cette relation sera détaillée en Phase 1 (inventory.logic) / Phase 2 (mappings.logic).

- **Config ↔ Data / Routing**
  - Certains paramètres (endpoints, flags de routing) auront un impact sur les inventaires Dataflows, Services, Routing.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` de config sont uniques.
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `configName`, `kind`, `source`, `targetStructureUcrs`, `behaviorSummary`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "config",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "config-flag-enable-advanced-promoboost-1",
      "configName": "ENABLE_ADVANCED_PROMOBOOST",
      "kind": "featureFlag",
      "source": {
        "type": "env",
        "identifier": "process.env.ENABLE_ADVANCED_PROMOBOOST",
        "sourcePath": "src/config/featureFlags.js"
      },
      "targetStructureUcrs": ["view-container-advancedBlock-1"],
      "behaviorSummary": "Active/désactive l’affichage du bloc 'Promotions avancées' dans la page de détail de campagne.",
      "metadata": {
        "defaultValue": false,
        "possibleValues": [true, false],
        "isDeprecated": false
      }
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

---

### 6.2. Exemple invalide (commenté)

```json
{
  "domain": "config",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "config-flag-enable-advanced-promoboost-1",
      "configName": "ENABLE_ADVANCED_PROMOBOOST",
      "kind": "featureFlag",
      "source": {
        "type": "env",
        "identifier": "process.env.ENABLE_ADVANCED_PROMOBOOST",
        "sourcePath": "src/config/featureFlags.js"
      },
      "targetStructureUcrs": ["view-unknown-99"],
      "behaviorSummary": "Active/désactive l’affichage du bloc 'Promotions avancées'.",
      "metadata": {}
    }
  ],
  "validation": {
    "status": "valid",
    "issues": []
  }
}
```

Problèmes :

- `targetStructureUcrs` contient `view-unknown-99` qui n’existe pas dans `inventory.structure.json`.
- `validation.status` ne devrait pas être `"valid"`.

---

## 7. 📋 Checklist contractuelle finale

- [ ] `domain` est `"config"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` de config sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Se limiter aux paramètres **réellement utilisés** par la page.
- Ne pas tenter de cartographier toute la configuration globale de l’application.
- Utiliser `validation.issues` pour signaler :
  - les paramètres trouvés mais non utilisés,
  - les usages ambigus,
  - les dépendances circulaires ou difficiles à interpréter.
- S’appuyer systématiquement sur :
  - `inventory.structure.json`,
  - le bridge `config.*` si disponible,
  - les guides de stack pour la vision cible.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – Config*
