# Schémas JSON de l'Orchestrateur V4

Ce dossier contient les schémas JSON Schema (draft-07) pour valider les artefacts générés par l'orchestrateur.

## 📋 Structure des schémas

### Inventaires (Phase 1)
- `inventory.structure.schema.json` - Validation de l'inventaire de structure
- `inventory.layout.schema.json` - (À créer) Validation de l'inventaire de layout
- `inventory.logic.schema.json` - (À créer) Validation de l'inventaire de logique
- ... (autres domaines)

### Mappings (Phase 2)
- `mapping.structure.schema.json` - Validation du mapping de structure
- `mapping.layout.schema.json` - (À créer) Validation du mapping de layout
- `mapping.logic.schema.json` - (À créer) Validation du mapping de logique
- ... (autres domaines)

### Synthèses
- `inventories-summary.schema.json` - (À créer) Validation du résumé des inventaires
- `mappings-summary.schema.json` - (À créer) Validation du résumé des mappings

## 🔍 Utilisation dans les stages

Les schémas sont utilisés pour valider les outputs **avant écriture sur disque** :

```javascript
// Pseudo-code d'utilisation dans un stage
const schema = loadSchema('inventory.structure.schema.json');
const isValid = validateAgainstSchema(inventoryData, schema);

if (!isValid) {
  // Ajouter les erreurs dans validation.issues
  // Fixer validation.status = "rejected"
  // Retourner Gate ❌
}
```

## ⚙️ Configuration dans les stages

Chaque stage peut activer/désactiver la validation stricte :

- **Mode strict** : Validation échoue → Gate ❌
- **Mode warning** : Validation échoue → warnings dans validation.issues, mais Gate ✅ si pas d'erreur bloquante

## 🎯 État d'implémentation

### ✅ Schémas implémentés (Couverture minimale suffisante)

Les 3 schémas critiques sont créés et validés dans les stages correspondants :

1. ✅ `inventory.structure.schema.json` - Stage 10 (Phase 1 - Entrée)
2. ✅ `mapping.structure.schema.json` - Stage 30 (Phase 2 - Transformation)
3. ✅ `generation.types.schema.json` - Stage 50 (Phase 3 - Génération)

**Cette couverture minimale assure :**
- ✅ Validation de la structure d'inventaire (entrée du pipeline)
- ✅ Validation du mapping de structure (cœur de la transformation)
- ✅ Validation de la génération de types (sortie de génération)

**Conclusion** : L'implémentation actuelle est **suffisante** pour garantir la cohérence structurelle du pipeline. Les schémas suivants sont **optionnels** selon vos besoins de robustesse.

---

### 📋 Extension possible (optionnelle)

Si vous souhaitez renforcer la validation, voici les schémas à créer par ordre de priorité :

**Priorité 2 (Important pour validation stricte)**
- `inventory.logic.schema.json` - Stage 15 (validation logique métier)
- `mapping.logic.schema.json` - Stage 35 (validation logique mappée)
- `inventory.actions.schema.json` - Stage 24 (validation actions)
- `mapping.actions.schema.json` - Stage 44 (validation actions mappées)

**Priorité 3 (Souhaitable pour synthèses)**
- `inventory.layout.schema.json` - Stage 11
- `mapping.layout.schema.json` - Stage 31
- `inventories-summary.schema.json` - Stage 26
- `mappings-summary.schema.json` - Stage 46

**Priorité 4 (Autres domaines)**
- Créer au besoin pour les 13+ autres domaines (dataflows, i18n, routing, etc.)

## 📝 Conventions

1. **Nommage** : `{phase}.{domain}.schema.json`
2. **$id** : `https://ai-orchestrator-v4/schemas/{filename}`
3. **Required fields** : Toujours spécifier les champs obligatoires
4. **Enums** : Utiliser pour les valeurs contraintes
5. **Patterns** : Utiliser regex pour les formats stricts (UCR, DSL tags)

## 🔄 Ajout d'un nouveau schéma (si nécessaire)

Si vous décidez d'ajouter un nouveau schéma :

1. **Créer le fichier** `.schema.json` dans ce dossier
2. **Mettre à jour** ce README (section "Extension possible")
3. **Ajouter la section de validation** dans le stage correspondant (voir `guide.json-schema-validation.md`)
4. **Référencer le guide** : `${paths.core}/guides-internals/globals/guide.json-schema-validation.md`
5. **Tester** avec des données réelles du pipeline

**Note** : Avant de créer un nouveau schéma, évaluez si la validation est réellement nécessaire. Les 3 schémas actuels couvrent les points critiques du pipeline.
