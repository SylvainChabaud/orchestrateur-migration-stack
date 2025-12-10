# 🔧 Guide Inventaire — Template Générique  
*(À utiliser pour créer tous les guides d’inventaires du système ai-orchestrator-v4 — framework-agnostique)*

---

## 1. 🎯 Objectif du domaine d’inventaire

Décrire précisément et sans ambiguïté, pour un **domaine d’inventaire donné** (Structure, Logic, Conditions, Layout, Data, etc.) :

1. Ce que doit capturer cet inventaire.
2. Le périmètre exact (ce qui est inclus / exclu).
3. Le rôle de cet inventaire dans la pipeline globale (Analyse → Interprétation → Génération).
4. La relation entre cet inventaire et les autres inventaires.

> Toujours répondre clairement à : **“Pourquoi cet inventaire existe-t-il ?”**

Le guide doit rester **agnostique du framework Legacy** (React, Vue, Angular, Twig, etc.).  
Les détails concrets de détection (patterns Legacy) sont décrits dans le **bridge** :

```text
${paths.workspace}/projects/${project.name}/stack/bridge-legacy-to-dsl.json
```

---

## 2. 📦 Format JSON attendu (Schéma contractuel)

Définir ici le schéma contractuel complet, lisible par l’IA et par l’orchestrateur.

### 2.1. Racine du JSON

Lister toutes les clés obligatoires au niveau racine, dans l’ordre :

- `domain` : string — nom interne de l’inventaire (ex: "structure", "logic", "conditions")
- `pageName` : string — page / module cible
- `sourceEntry` : string — fichier d’entrée Legacy
- `items` ou `components` ou `nodes` : array — selon le domaine
- `validation` : object — statut et erreurs

> L’ordre et le nom exact des clés doivent rester **stables** dans toute la pipeline.

---

### 2.2. Schéma interne (items)

Décrire ici l’objet principal du tableau :

```text
items[] : InventoryItem (ou ComponentNode, ConditionNode, etc.)
```

Puis détailler, en distinguant :

#### Champs obligatoires
Exemples typiques :
- `ucr` : identifiant unique stable interne  
- `name` : nom logique de l’entité (view, hook, data source, condition, etc.)
- `type` : classification interne (ex: "component", "condition", "datasource", "eventHandler")
- `sourcePath` : chemin Legacy de définition
- `metadata` : informations dérivées du Legacy (peut être {} au minimum)

#### Champs optionnels
Selon le domaine, par exemple :
- `props`, `state`, `children`, `conditions`, `dataSources`, `events`, etc.

> Tous les champs doivent être clairement listés et décrits dans ce guide, même s’ils sont optionnels.

---

### 2.3. Contraintes contractuelles

Écrire les contraintes obligatoires, par exemple :

- unicité des `ucr`,
- interdiction des cycles (si l’inventaire décrit un graphe),
- cohérence des liens (`children[]`, `targets[]`, etc.),
- aucune clé inconnue dans le JSON final,
- toutes les clés obligatoires doivent être présentes,
- le JSON doit être **strictement sérialisable** (pas d’AST, pas de structures JS).

---

## 3. 🧠 Règles d’extraction (Analyse) — Niveau générique

Décrire ici les **règles d’extraction logiques**, indépendantes du framework :

### 3.1. Concepts DSL utilisés

Lister les concepts du DSL interne pertinents pour ce domaine, par exemple :

- `structure.viewNode`, `structure.viewHierarchy`  
- `logic.localState`, `logic.effect`  
- `condition.visibility`  
- `data.dataSource`, `data.query`  
- etc.

> Le mapping de ces concepts vers le code Legacy concret est fourni par le **bridge Legacy → DSL** (`bridge-legacy-to-dsl.json`).

### 3.2. Règles d’analyse

Exemples selon le domaine :

- comment identifier une entité (item) à extraire,
- comment parcourir la hiérarchie d’appel ou de composition,
- comment lier une entité à une autre (parent → enfant, source → cible),
- comment lier les entités à des fichiers (`sourcePath`).

### 3.3. Restrictions

Préciser clairement ce que l’inventaire **ne doit pas faire** :

- ex : l’inventaire Structure ne doit pas interpréter la logique métier,
- ex : l’inventaire Conditions ne doit pas générer de code cible,
- ex : l’inventaire Data ne doit pas se soucier de layout ou de routing.

---

## 4. 🔗 Relations avec les autres inventaires

Décrire comment cet inventaire :

- **alimente** d’autres inventaires,
- **dépend** de certains inventaires,
- doit rester cohérent avec l’ensemble du pipeline.

Exemples :

- Structure → fournit les `ucr` aux inventaires Logic, Conditions, Layout…
- Logic → s’appuie sur Structure et Data.
- Conditions → dépend de Structure, Logic, et éventuellement Feature Flags.
- Layout → dépend de Structure, éventuellement Décorations UI.

---

## 5. 🧪 Validation interne (local checks)

Lister les **checks minimaux** que tout stage doit effectuer avant de valider son Gate :

Exemples de checks :

- tous les items ont un `ucr` unique,
- toutes les références (`children[]`, `targets[]`, `sources[]`) pointent vers des UCR existants,
- pas de cycle dans les graphes si le domaine l’interdit,
- les valeurs obligatoires sont présentes,
- `validation.status` est cohérent avec `validation.issues`.

---

## 6. 📘 Exemples de JSON valides et invalides

### Exemple valide minimal

Adapter cet exemple au domaine cible (ici, illustré pour une structure générique) :

```json
{
  "domain": "example-domain",
  "pageName": "SamplePage",
  "sourceEntry": "src/legacy/SamplePage/index.js",
  "items": [
    {
      "ucr": "item-1",
      "name": "SampleEntity",
      "type": "exampleType",
      "sourcePath": "src/legacy/SamplePage/Entity.js",
      "metadata": {}
    }
  ],
  "validation": { "status": "valid", "issues": [] }
}
```

### Exemple invalide (commenté)

```json
{
  "domain": "example-domain",
  "pageName": "SamplePage",
  "items": [], // ❌ aucun item alors que la page devrait contenir des entités
  "validation": {} // ❌ pas de statut ni de liste d'issues
}
```

---

## 7. 📋 Checklist contractuelle finale

Adapter et compléter, par exemple :

- [ ] Le domaine (`domain`) est correctement défini  
- [ ] Toutes les clés racines obligatoires sont présentes  
- [ ] Les `ucr` sont uniques  
- [ ] Toutes les références pointent vers des UCR existants  
- [ ] Le JSON final respecte le schéma contractuel du domaine  
- [ ] Le fichier est sérialisable (JSON valide)  
- [ ] `validation.status` est `"valid"` ou `"rejected"`  
- [ ] Les chemins de fichiers (`sourcePath`) sont cohérents avec le Legacy  
- [ ] Le guide n’introduit aucune dépendance directe à un framework particulier

---

## 8. 🧩 Notes d’implémentation IA

Rappeler ici le comportement attendu du modèle :

- produire un JSON strict, sans pollution de texte libre,
- ne jamais inventer de chemins ou d’entités qui n’existent pas réellement,
- ne pas interpréter la stack cible dans les inventaires (l’inventaire décrit le Legacy),
- s’appuyer sur :
  - le **DSL interne**,
  - le **bridge Legacy → DSL**,
  - les autres guides internes,
- en cas d’ambiguïté, documenter dans `validation.issues` sans casser la structure.

---

© 2025 — ai-orchestrator-v4  
*Guide Template – Inventaires (Framework-agnostic)*
