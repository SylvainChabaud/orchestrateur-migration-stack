# 🔧 Guide Inventaire — i18n (`inventory.i18n`)

*(Domaine d’inventaire : **i18n** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine d’inventaire

L’inventaire **i18n** décrit, pour une page ou un module donné (`${project.pageName}`) :

1. Les **keys de traduction** utilisées ou à prévoir.
2. Les **namespaces / fichiers** de traduction impliqués.
3. Les **usages concrets** de ces clés dans la page (labels, titres, messages d’erreur, tooltips, etc.).
4. Les **liens entre ces usages et les vues** décrites dans `inventory.structure.json` (via les `ucr`).
5. Les aspects **contextuels** importants (pluriels, formats, contextes spécifiques).

Il répond à la question :

> **“Quels textes sont affichés dans cette page, comment sont-ils ou devraient-ils être représentés dans le système i18n, et où sont-ils utilisés ?”**

Ce domaine ne :

- ne décrit pas la logique métier (`inventory.logic`),
- ne gère pas les styles (`inventory.styles`),
- ne décide pas de l’implémentation exacte du système i18n dans la stack cible (Phase 2 & 3).

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

### 2.1. Racine du JSON

La racine du JSON `inventory.i18n.json` doit respecter le schéma suivant :

- `domain` : string — doit valoir exactement `"i18n"`
- `pageName` : string — nom logique de la page/module (souvent `${project.pageName}`)
- `sourceEntry` : string — chemin Legacy du fichier d’entrée principal (ex : `${paths.legacySource}`)
- `items` : array d’objets — liste des unités i18n (voir 2.2)
- `validation` : object — statut et éventuelles anomalies

Exemple minimal :

```json
{
  "domain": "i18n",
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

Chaque élément de `items[]` représente une **unité i18n** (I18nItem).  
Le modèle recommandé est **centré sur la key**, éventuellement avec plusieurs usages.

```text
items[] : I18nItem
```

#### 2.2.1. Champs obligatoires

- `ucr` : string  
  Identifiant canonique unique (UCR) de l’unité i18n, conforme à `guide.ucr.md`.  
  - Doit être **unique** dans l’inventaire i18n.

- `key` : string  
  Clé i18n, par exemple :
  - `"campaigns.detail.title"`,
  - `"errors.network.generic"`,
  - `"actions.save"`.
  - Si le Legacy ne fournit pas de clé explicite (texte en dur), une clé **proposée** peut être construite selon les conventions de la stack (ex. `campaigns.detail.title.proposed`). Cela doit être documenté dans `metadata`.

- `namespace` : string  
  Namespace ou fichier logique auquel la clé appartient, par exemple :
  - `"campaigns"`,
  - `"common"`,
  - `"errors"`.
  - Peut être une valeur générique (`"default"`) si le projet ne distingue pas les namespaces.

- `targetStructureUcrs` : array de string  
  Liste des `ucr` (issus de `inventory.structure.json`) où cette clé est utilisée ou devrait être utilisée.

- `usageKind` : string  
  Type d’usage principal de cette clé :
  - `"label"`,
  - `"title"`,
  - `"subtitle"`,
  - `"description"`,
  - `"helperText"`,
  - `"errorMessage"`,
  - `"successMessage"`,
  - `"tooltip"`,
  - `"cta"` (Call To Action),
  - etc.

- `sampleText` : string  
  Un exemple de texte localisé (dans la langue de référence, souvent français/anglais) :
  - soit extrait du Legacy si déjà présent,
  - soit proposé à partir du contexte si le texte est en dur ou manquant.

- `metadata` : object  
  Informations additionnelles :
  - `isFromLiteral`: booléen (true si le texte provient d’un literal dans le code),
  - `isExistingKey`: booléen (true si la clé existait déjà dans le système i18n Legacy),
  - `notes`: string optionnel pour commentaires (ex : “clé à renommer”, “conflit potentiel”, etc.).  
  Peut être `{}` au minimum.

#### 2.2.2. Champs optionnels suggérés

- `pluralization` : object  
  Informations sur la gestion du pluriel, par exemple :
  ```json
  {
    "hasPlural": true,
    "forms": ["one", "other"]
  }
  ```

- `formatting` : object  
  Informations sur les formats spéciaux :
  - dates,
  - nombres,
  - montants,
  - placeholders (ex : `{count}`, `{name}`).

- `contexts` : array de string  
  Contexte(s) d’utilisation particulier(s) :
  - `"modal"`,
  - `"notification"`,
  - `"inline"`,
  - `"tableHeader"`,
  - etc.

- `languages` : array de string  
  Liste des langues dans lesquelles la key est déjà traduite (si cette information est disponible via le Legacy ou les fichiers de traduction).

Tout champ optionnel utilisé doit être **documenté** ici.

---

### 2.3. Contraintes contractuelles

- Tous les `ucr` i18n doivent être **uniques** dans `inventory.i18n.json`.
- Tous les `targetStructureUcrs` doivent référencer des `ucr` valides de `inventory.structure.json`.
- Aucune clé inconnue ne doit être ajoutée en racine ou dans les items.
- Le JSON doit être **strictement sérialisable**.

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

### 3.1. Concepts DSL utilisés

Si le DSL interne définit un domaine `i18n.*`, il peut inclure par exemple :

- `i18n.keyUsage`
- `i18n.namespace`
- `i18n.pluralization`
- `i18n.formatting`

Le bridge Legacy → DSL (`bridge-legacy-to-dsl.json`) peut alors fournir des patterns pour détecter ces usages dans le code.  
Si ces concepts n’existent pas encore, l’analyse se fait de façon plus générique en se basant sur :

- les appels à des fonctions i18n,
- les composants de traduction,
- les conventions de nommage des clés.

### 3.2. Règles d’analyse

L’inventaire i18n doit :

1. Parcourir le code Legacy à partir de `${paths.legacySource}` pour :
   - détecter les appels aux API i18n (fonctions, hooks, composants),
   - repérer les textes littéraux destinés à être localisés,
   - identifier les fichiers/objets de traduction déjà existants si accessibles.
2. Pour chaque clé ou texte identifié :
   - déterminer le `namespace`,
   - associer les `targetStructureUcrs` via la structure de la page,
   - classifier l’`usageKind` (label, title, errorMessage, etc.),
   - proposer un `sampleText` pertinent.
3. Regrouper les occurrences par key + namespace pour produire un `I18nItem` par unité logique.

### 3.3. Restrictions

L’inventaire i18n **ne doit pas** :

- faire de la génération de texte libre hors `sampleText`,
- décider de la structure globale du système i18n (fichiers, chargement lazy, etc.),
- mélanger la description i18n avec la logique métier ou les styles,
- introduire des dépendances directes à une librairie i18n spécifique (les noms de fonctions peuvent apparaître dans les commentaires ou `metadata`, mais pas comme contrat structurant).

---

## 4. 🔗 Relations avec les autres inventaires

- **i18n ← Structure**
  - Utilise les `ucr` de Structure (`targetStructureUcrs`) pour localiser les usages de texte dans l’arbre de vues.

- **i18n ↔ Layout (optionnel)**
  - Peut relier certains textes à des régions de layout en fonction des besoins,
  - mais ce n’est pas obligatoire au niveau de ce domaine.

- **i18n ↔ Styles (optionnel)**
  - Certains textes peuvent être stylés de manière spécifique (badges, tags), mais ces informations sont décrites dans `inventory.styles`.

---

## 5. 🧪 Validation interne (local checks)

Avant de valider l’étape, l’IA doit vérifier au minimum :

- [ ] Tous les `ucr` i18n sont uniques.
- [ ] Tous les `targetStructureUcrs` pointent vers des `ucr` valides de `inventory.structure.json`.
- [ ] Tous les champs obligatoires (`ucr`, `key`, `namespace`, `targetStructureUcrs`, `usageKind`, `sampleText`, `metadata`) sont présents.
- [ ] `validation.status` et `validation.issues` sont cohérents.
- [ ] Le JSON est strictement valide.

---

## 6. 📘 Exemples de JSON

### 6.1. Exemple valide minimal

```json
{
  "domain": "i18n",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "i18n-key-campaigns-detail-title-1",
      "key": "campaigns.detail.title",
      "namespace": "campaigns",
      "targetStructureUcrs": ["view-root-1"],
      "usageKind": "title",
      "sampleText": "Détails de la campagne",
      "metadata": {
        "isFromLiteral": false,
        "isExistingKey": true
      }
    },
    {
      "ucr": "i18n-key-campaigns-detail-save-cta-1",
      "key": "campaigns.detail.saveCta",
      "namespace": "campaigns",
      "targetStructureUcrs": ["view-container-1"],
      "usageKind": "cta",
      "sampleText": "Enregistrer",
      "metadata": {
        "isFromLiteral": true,
        "isExistingKey": false,
        "notes": "Clé proposée à partir d’un texte en dur."
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
  "domain": "i18n",
  "pageName": "CampaignsDetail",
  "sourceEntry": "src/packages/promo-boost/components/campaignsDetail/index.js",
  "items": [
    {
      "ucr": "i18n-key-campaigns-detail-title-1",
      "key": "campaigns.detail.title",
      "namespace": "campaigns",
      "targetStructureUcrs": ["view-unknown-99"],
      "usageKind": "title",
      "sampleText": "Détails de la campagne",
      "metadata": {
        "isFromLiteral": false,
        "isExistingKey": true
      }
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

- [ ] `domain` est `"i18n"`  
- [ ] `pageName` est correctement renseigné  
- [ ] `sourceEntry` pointe vers le bon fichier Legacy  
- [ ] Tous les `ucr` i18n sont uniques  
- [ ] Tous les `targetStructureUcrs` sont valides vis-à-vis de `inventory.structure.json`  
- [ ] Le JSON respecte le schéma contractuel du domaine  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] `validation.issues` est cohérent avec les erreurs détectées  
- [ ] Le fichier est un JSON strictement valide  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

- Ne pas inventer de keys i18n sans les documenter clairement dans `metadata` comme « proposées ».
- Toujours s’appuyer sur :
  - `inventory.structure.json` pour localiser les vues,
  - le bridge (si pertinent) pour interpréter les patterns i18n,
  - les guides de stack pour rester cohérent avec la stratégie i18n cible.
- Utiliser `validation.issues` pour :
  - lister les textes non externalisés,
  - les conflits de keys,
  - les cas où le namespace n’est pas déterminable précisément.

---

© 2025 — ai-orchestrator-v4  
*Guide Inventaire – i18n*
