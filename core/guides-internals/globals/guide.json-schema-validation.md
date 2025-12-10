# Guide d'implémentation de la validation JSON Schema

## 🎯 Objectif

Ce guide explique comment implémenter la validation des outputs avec les schémas JSON dans chaque stage.

---

## 📋 Quand valider ?

La validation doit être effectuée **juste avant l'écriture du fichier final**, dans la section "Actions" du stage, après la construction complète de l'objet en mémoire.

---

## 🔧 Implémentation dans un stage

### Étape 1 : Charger la configuration de validation

Au début des Actions, charger les paramètres de validation :

```javascript
// Pseudo-code
const validationConfig = {
  enabled: config.validation.enableSchemaValidation,
  mode: config.validation.validationMode,
  schemasPath: config.validation.schemasPath
};
```

### Étape 2 : Valider avant l'écriture

Juste avant l'écriture du fichier JSON final :

```javascript
// Pseudo-code pour un inventaire
if (validationConfig.enabled) {
  const schemaPath = `${validationConfig.schemasPath}/inventory.structure.schema.json`;
  const schema = loadJsonSchema(schemaPath);
  
  const validationResult = validateAgainstSchema(inventoryRoot, schema);
  
  if (!validationResult.valid) {
    // Ajouter les erreurs dans l'objet validation
    inventoryRoot.validation.issues.push(...validationResult.errors.map(err => ({
      severity: "error",
      message: `Schema validation error: ${err.message}`,
      path: err.dataPath
    })));
    
    // Mode strict : rejeter
    if (validationConfig.mode === "strict") {
      inventoryRoot.validation.status = "rejected";
      // Écrire quand même le fichier pour inspection
      writeJsonFile(outputPath, inventoryRoot);
      // Retourner Gate ❌
      return { gate: "❌", reason: "Schema validation failed" };
    }
    
    // Mode warning : continuer avec avertissements
    if (validationConfig.mode === "warning") {
      inventoryRoot.validation.issues.push({
        severity: "warning",
        message: "Output does not fully conform to schema but processing continues"
      });
    }
  }
}
```

### Étape 3 : Documentation dans le stage

Ajouter une note dans la section "Validation interne" du stage :

```markdown
### Validation du schéma JSON (optionnelle)

Si `validation.enableSchemaValidation = true` dans la configuration :

1. Charger le schéma depuis `${validation.schemasPath}/inventory.structure.schema.json`
2. Valider `inventoryRoot` contre ce schéma
3. En cas d'erreur :
   - Mode `strict` : ajouter les erreurs dans `validation.issues[]`, fixer `status = "rejected"`, retourner `Gate ❌`
   - Mode `warning` : ajouter des warnings dans `validation.issues[]`, continuer avec `Gate ✅`
```

---

## 📝 Exemple complet pour un stage d'inventaire

Voici comment modifier la section "Actions" d'un stage :

### Avant (sans validation)

```markdown
7. **Validation interne**
   - Vérifier que tous les `ucr` sont uniques
   - Vérifier les références `parentUcr`
   - Mettre à jour `validation.status`

8. **Écriture de l'output**
   - Écrire `inventory.structure.json`
```

### Après (avec validation optionnelle)

```markdown
7. **Validation interne**
   - Vérifier que tous les `ucr` sont uniques
   - Vérifier les références `parentUcr`
   - Mettre à jour `validation.status` en fonction des contrôles manuels

8. **Validation du schéma JSON (si activée)**
   - Si `validation.enableSchemaValidation = true` :
     - Charger le schéma `${validation.schemasPath}/inventory.structure.schema.json`
     - Valider `inventoryRoot` contre le schéma
     - En cas d'erreur de validation :
       - Mode `strict` : ajouter les erreurs dans `validation.issues[]`, fixer `status = "rejected"`, préparer `Gate ❌`
       - Mode `warning` : ajouter des warnings dans `validation.issues[]`, continuer normalement
     - En cas de succès : ajouter une info dans `validation.issues[]` (optionnel)

9. **Écriture de l'output**
   - Écrire `inventory.structure.json` dans le chemin cible
```

---

## 🎭 Modes de validation

### Mode `strict` (recommandé pour production)
- **Comportement** : Toute erreur de schéma → Gate ❌
- **Avantage** : Garantie maximale de qualité
- **Inconvénient** : Peut bloquer si schéma trop restrictif

### Mode `warning` (recommandé pour développement)
- **Comportement** : Erreurs de schéma → warnings, mais Gate ✅ si logique OK
- **Avantage** : Plus flexible, permet d'itérer rapidement
- **Inconvénient** : Peut laisser passer des incohérences

### Validation désactivée
- **Comportement** : Aucune validation de schéma
- **Usage** : Tests rapides, prototypage
- **Risque** : Aucune garantie de conformité

---

## 🔍 Bibliothèques de validation JSON Schema

L'IA peut utiliser différentes approches selon le contexte :

### JavaScript/Node.js
```javascript
const Ajv = require('ajv');
const ajv = new Ajv({ allErrors: true });
const validate = ajv.compile(schema);
const valid = validate(data);
```

### Python
```python
import jsonschema
jsonschema.validate(instance=data, schema=schema)
```

### Intégré dans l'IA
L'IA peut aussi valider directement en :
1. Lisant le schéma JSON
2. Comparant structure et types
3. Vérifiant les contraintes (required, enum, pattern, etc.)
4. Générant un rapport d'erreurs détaillé

---

## 📊 Structure des erreurs de validation

Format standardisé des erreurs dans `validation.issues[]` :

```json
{
  "severity": "error",
  "message": "Schema validation error: Property 'ucr' is required",
  "path": "/items/2",
  "schemaPath": "#/properties/items/items/required",
  "keyword": "required",
  "params": {
    "missingProperty": "ucr"
  }
}
```

---

## ✅ Checklist d'implémentation

Pour chaque stage nécessitant une validation :

- [ ] Identifier le schéma correspondant (inventory.X ou mapping.X)
- [ ] Vérifier que le schéma existe dans `core/schemas/`
- [ ] Ajouter la section "Validation du schéma JSON" dans les Actions
- [ ] Documenter le comportement en mode strict vs warning
- [ ] Tester avec des données valides
- [ ] Tester avec des données invalides
- [ ] Vérifier le comportement des Gates

---

## 🎯 État d'implémentation actuel

### ✅ Validation active (implémenté)

Les stages suivants intègrent la validation de schéma JSON :

1. **Stage 10** (inventory.structure) - ✅ Section validation + schéma créé
2. **Stage 30** (mapping.structure) - ✅ Section validation + schéma créé
3. **Stage 50** (generate-types) - ✅ Section validation + schéma créé

Ces 3 stages sont les **points critiques** de validation dans le pipeline et suffisent pour garantir la cohérence structurelle de bout en bout.

### 📋 Extension possible (optionnelle)

Si vous souhaitez étendre la validation à d'autres stages :

**Phase 1 - Inventaires** (ordre de priorité)
1. Stage 15 (inventory.logic) - Important pour la logique métier
2. Stage 24 (inventory.actions) - Important pour les actions
3. Stage 19 (inventory.dataflows) - Important pour les flux de données
4. Stage 26 (inventories-summary) - Utile pour la synthèse
5. Autres stages 11-25 - Optionnel selon besoins

**Phase 2 - Mappings** (ordre de priorité)
1. Stage 35 (mapping.logic) - Important pour la logique métier
2. Stage 44 (mapping.actions) - Important pour les actions
3. Stage 39 (mapping.dataflows) - Important pour les flux de données
4. Stage 46 (mappings-summary) - Utile pour la synthèse
5. Autres stages 31-45 - Optionnel selon besoins

**Phase 3 - Génération** (ordre de priorité)
1. Stages 51-61 (services, stores, hooks, etc.) - Optionnel (validation moins critique car guidée par stack-guides)
2. Stage 62 (generation-summary) - Optionnel

### ⚙️ Configuration actuelle

La validation est **optionnelle** et contrôlée par la configuration globale :

```yaml
validation:
  enableSchemaValidation: true   # false pour désactiver complètement
  validationMode: "strict"       # "warning" pour continuer malgré erreurs
  schemasPath: "${paths.core}/schemas"
```

**Recommandation d'usage :**
- **Développement** : `enableSchemaValidation: false` ou `validationMode: "warning"` (flexibilité)
- **Production** : `enableSchemaValidation: true` + `validationMode: "strict"` (robustesse)

### 📊 Couverture actuelle

Avec les 3 stages validés, vous avez :
- ✅ Validation de la structure d'inventaire (entrée Phase 1)
- ✅ Validation du mapping de structure (sortie Phase 2)
- ✅ Validation de la génération de types (sortie Phase 3)

Cette couverture minimale assure que les **artefacts structurels clés** sont conformes aux schémas à chaque phase critique du pipeline.

---

## 📖 Ressources

- [JSON Schema Specification](https://json-schema.org/)
- [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/)
- [Schémas créés : `core/schemas/`](../schemas/)

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4
