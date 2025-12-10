# 🧩 Stage 02 – Legacy → DSL Bridge Builder  
**Phase:** Phase 0 – Bootstrap  
**Prev:** 01 – Project Structure Spec Builder  
**Next:** 10 – Inventory Structure

---

## 🎯 Objective

Ce stage génère automatiquement le guide de correspondance **Legacy ↔ DSL interne**, utilisé par *tous* les inventaires et mappings.

Il analyse la stack Legacy du projet (React, Vue, Angular, etc.) et génère :

- `bridge-legacy-to-dsl.json`

Ce fichier définit, pour chaque **concept générique du DSL**, les **patterns concrets** permettant de reconnaître ce concept dans le code Legacy.

> ⚠️ **Important** : Ce stage crée uniquement le **catalogue de concepts DSL** (bridge-legacy-to-dsl.json).  
> Les **UCR** (Unique Canonical References) sont créés plus tard, en **Phase 1** (stages 10-26, inventaires).  
> Le bridge DSL sert de "dictionnaire sémantique", les UCR sont les "identifiants d'instances".

---

## ⚙️ Inputs

### Configuration
- `core/configs/project.config.yaml`
  - `project.name`
  - `paths.workspace`
  - `stack.custom`

### Guides préalables
- Stack guides (issus du Stage 00) :
  - `${paths.workspace}/projects/${project.name}/stack/stack-guides/*`

### Template du Bridge
- `${paths.core}/templates/guide.bridge.legacy-to-dsl.template.md`

---

## 📤 Outputs

Generated in :

```
${paths.workspace}/projects/${project.name}/stack/
```

### 1. `bridge-legacy-to-dsl.json`

**Format :** JSON strictement conforme au schéma défini dans le template.

**Contenu :** Mapping complet Legacy ↔ DSL avec tous les concepts DSL documentés.

---

## 🧠 Actions

1. Lire la configuration projet.
2. Identifier la stack Legacy du projet.
3. Charger le template DSL.
4. Pour chaque concept DSL :
   - remplir description,
   - remplir legacyPatterns[] selon le framework Legacy.
5. **Générer le fichier JSON** `bridge-legacy-to-dsl.json`.
   - Le JSON doit être strictement conforme au schéma du template.
   - Tous les concepts DSL du template doivent avoir une entrée.
6. Vérifier l'exhaustivité.
7. Sauvegarder le fichier JSON.

---

## ✅ Auto-Checks

```json
{
  "stageId": "02",
  "stageName": "Legacy to DSL Bridge Builder",
  "checks": {
    "configLoaded": true,
    "stackDetected": true,
    "dslExhaustive": true,
    "patternsGenerated": true,
    "jsonWritten": true
  }
}
```

---

## 🧩 Gate

```markdown
## 🧩 Gate
Gate ✅
```

Ou :

```markdown
## 🧩 Gate
Gate ❌
```

---

## 📦 Next

> Continuer avec `10-inventory.structure.md`.

---

© 2025 — ai-orchestrator-v4
