````markdown
# 🔍 Guide Validation — Template

*(Domaine de validation : **${domain}** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine de validation

Le guide de validation **${domain}** décrit comment **auditer la qualité, la cohérence et l'équivalence fonctionnelle** du code généré en Phase 3 de manière **heuristique** :

1. À partir du **code généré** dans `phase-3-generation/src_new/`
2. En référence aux **mappings Phase 2** (`mapping.*.json`)
3. En référence aux **inventaires Phase 1** (`inventory.*.json`)
4. En conformité avec les **stack guides Phase 0**
5. En comparaison **déclarative** avec le **code Legacy** (via inventaires)

Il répond à la question :

> **"Comment garantir que le code généré est cohérent, crédible, fonctionnellement équivalent (déclarativement) et prêt pour intégration avec validation manuelle minimale ?"**

## 🎯 Approche Réaliste de Phase 4

La Phase 4 effectue un **audit de qualité heuristique** sans exécution réelle de build/tests :

- ✅ **Analyse statique approfondie** (imports, types, patterns)
- ✅ **Vérifications déclaratives** (structure, cohérence, équivalence)
- ✅ **Détection de risques** (smells, collisions, violations)
- ✅ **Scoring heuristique** basé sur règles et patterns
- ❌ **PAS d'exécution** de compilateur, tests, ou build réel
- ❌ **PAS de runtime validation** (serveur, browser, etc.)

Le domaine Validation **${domain}** :

- **analyse** le code généré de façon **statique/heuristique**,
- **valide** la cohérence interne (imports, types, structure),
- **vérifie** l'équivalence fonctionnelle **déclarative** via UCR,
- **détecte** les risques (smells, patterns, violations),
- **produit** des rapports détaillés (JSON + Markdown),
- **calcule** un score de qualité heuristique pour ce domaine.

Il **ne fait pas** :

- d'exécution de compilateur ou build réel,
- d'exécution de tests unitaires/intégration,
- de génération ou modification de code,
- de validation runtime (browser, serveur).

---

## 2. 📦 Format de sortie attendu

### 2.1. Artefacts générés

Le guide de validation **${domain}** produit des rapports dans :

```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/${domain}/
```

**Fichiers obligatoires :**

- `${domain}.report.json` — Rapport machine-readable structuré
- `${domain}.report.md` — Rapport human-readable formaté
- `${domain}.meta.json` — Métadonnées de validation (dans `.meta/`)

**Fichiers optionnels selon le domaine :**

- `${domain}-details.json` — Données détaillées supplémentaires
- `${domain}-matrix.json` — Matrices de comparaison/équivalence
- `${domain}-violations.json` — Liste des violations détectées
- `${domain}.log` — Logs d'analyse heuristique

### 2.2. Structure du rapport JSON (`${domain}.report.json`)

Schéma contractuel commun à tous les rapports de validation :

```jsonc
{
  "domain": "${domain}",
  "stageId": "${stageId}",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "validation": {
    "status": "valid" | "warnings" | "rejected",
    "score": 95.5,              // Score 0-100 (heuristique)
    "gate": "passed" | "failed",
    "issues": [
      {
        "severity": "error" | "warning" | "info",
        "category": "string",
        "message": "string",
        "location": {
          "file": "string",
          "line": 42,
          "column": 10
        },
        "suggestion": "string"
      }
    ]
  },
  "metrics": {
    // Métriques heuristiques spécifiques au domaine
  },
  "summary": {
    "totalChecks": 150,
    "passedChecks": 143,
    "failedChecks": 2,
    "warningChecks": 5,
    "skippedChecks": 0
  },
  "thresholds": {
    // Seuils utilisés pour ce domaine
  },
  "recommendations": [
    "Action recommandée 1",
    "Action recommandée 2"
  ]
}
```

### 2.3. Structure du rapport Markdown (`${domain}.report.md`)

Format human-readable standardisé :

```markdown
# 🔍 Rapport de Validation — ${Domain}

**Page :** ${project.pageName}  
**Date :** 2025-12-08 10:00:00  
**Statut :** ✅ Valid / ⚠️ Warnings / ❌ Rejected  
**Score :** 95.5/100  
**Gate :** ✅ Passed / ❌ Failed

---

## 📊 Résumé

- **Total vérifications :** 150
- **Réussies :** 143 ✅
- **Échouées :** 2 ❌
- **Avertissements :** 5 ⚠️
- **Ignorées :** 0

---

## 📈 Métriques (Analyse Heuristique)

| Métrique | Valeur | Seuil | Statut |
|----------|--------|-------|--------|
| ... | ... | ... | ✅/❌ |

---

## 🚨 Issues détectées

### ❌ Erreurs (2)

1. **[Category] Message**
   - Fichier : `path/to/file.ts:42:10`
   - Suggestion : Action corrective

### ⚠️ Avertissements (5)

1. **[Category] Message**
   - Fichier : `path/to/file.ts:100:5`
   - Suggestion : Amélioration recommandée

---

## 💡 Recommandations

1. Recommandation 1
2. Recommandation 2

---

## ✅ Conclusion

_(Synthèse finale du statut de validation)_

---

*Généré par ai-orchestrator-v4 — Phase 4 — Stage ${stageId} (Analyse Heuristique)*
```

### 2.4. Métadonnées de validation (`${domain}.meta.json`)

Métadonnées techniques de traçabilité :

```jsonc
{
  "stage": "${stageId}",
  "domain": "${domain}",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed" | "failed",
  "inputs": {
    "generatedCode": "phase-3-generation/src_new/",
    "mappings": ["mapping.structure.json", "mapping.logic.json"],
    "inventories": ["inventory.structure.json"],
    "stackGuides": ["guide.validation.md"],
    "legacySource": "src/legacy/..."
  },
  "outputs": {
    "report": "phase-4-validation/${domain}/${domain}.report.json",
    "reportMd": "phase-4-validation/${domain}/${domain}.report.md"
  },
  "checksExecuted": 150,
  "duration": "12s",
  "errors": []
}
```

---

## 3. 🧠 Règles de validation — Niveau générique

### 3.1. Concepts de validation utilisés

Le domaine **${domain}** valide principalement les aspects suivants de manière **heuristique** :

_(Liste des aspects validés pour ce domaine)_

Exemples selon le domaine (approche réaliste) :

- **static-consistency** (Stage 70) : cohérence imports, détection code smells, vérif types/props basique
- **tests-audit** (Stage 71) : présence fichiers tests, cartographie UCR↔tests, couverture théorique
- **functional-equivalence** (Stage 72) : matrice behaviours legacy vs code généré (déclarative)
- **dependencies-coherence** (Stage 73) : deps utilisées, cross-check package.json, imports non résolus
- **integration-consistency** (Stage 74) : cohérence chemins/structure, routing, collisions noms
- **accessibility-heuristic** (Stage 75) : analyse markup/JSX, ARIA, labels, structure sémantique
- **performance-patterns** (Stage 76) : imports lourds non lazy, composants suspects, mémoïsation
- **quality-compliance** (Stage 77) : nommage, patterns archi, documentation, types/PropTypes

### 3.2. Inputs principaux

Pour valider le domaine **${domain}**, le stage doit consulter :

1. **Code généré Phase 3** (obligatoire)
   - `phase-3-generation/src_new/**/*` — Code à valider

2. **Mappings Phase 2** (obligatoire)
   - `mapping.${domain}.json` — Contexte de génération
   - Autres mappings liés selon `relations`

3. **Inventaires Phase 1** (obligatoire pour équivalence fonctionnelle)
   - `inventory.*.json` — Référence Legacy DSL

4. **Code Legacy** (optionnel, uniquement pour comparaison)
   - `${paths.legacySource}` — Source Legacy originale

5. **Stack guides Phase 0** (obligatoire)
   - `guide.validation.md` — Règles générales
   - `guide.${domain}.md` — Règles spécifiques au domaine
   - `guide.quality-thresholds.md` — Seuils de qualité

6. **Configuration projet** (obligatoire)
   - `package.json` — Dépendances disponibles
   - Config files (`.eslintrc`, `tsconfig.json`, etc.) — référence uniquement, pas d'exécution

### 3.3. Stratégie de validation

#### Étape 1 : Préparation du contexte d'analyse

Avant de valider, vérifier que :

- Tous les **inputs requis** sont disponibles (code, mappings, inventories)
- Les **règles heuristiques** sont chargées (depuis stack-guides)
- Les **seuils de qualité** sont définis
- Les **patterns de détection** sont prêts

```typescript
// Exemple de préparation (analyse heuristique, pas d'exécution)
const validationContext = {
  generatedCodePath: resolveGeneratedCodePath(),
  mappings: loadMappings(['structure', 'logic', 'tests']),
  inventories: loadInventories(['structure', 'logic']),
  stackGuides: loadStackGuides(['validation', domain]),
  thresholds: loadQualityThresholds(domain),
  heuristicRules: loadHeuristicRules(domain),
  patterns: {
    codeSmells: loadCodeSmellPatterns(),
    a11yRules: loadA11yRules(),
    perfRisks: loadPerformanceRiskPatterns()
  }
};
```

#### Étape 2 : Exécution des analyses heuristiques

Pour chaque analyse du domaine :

```typescript
const checks = [
  {
    id: 'check-1',
    name: 'Descriptive check name',
    category: 'consistency' | 'smells' | 'patterns' | 'coherence' | ...,
    severity: 'error' | 'warning' | 'info',
    type: 'heuristic', // Pas d'exécution réelle
    execute: async () => {
      // Analyse statique/heuristique du code généré
      // Pas de compilation, pas de runtime
      const analysis = analyzeCodeHeuristically(generatedFiles, rules);
      
      return {
        passed: analysis.issuesCount === 0,
        message: analysis.summary,
        details: analysis.findings
      };
    }
  },
  // ... autres checks
];

// Exécuter toutes les analyses
const results = await Promise.all(
  checks.map(check => executeCheck(check))
);
```

#### Étape 3 : Collecte des métriques heuristiques

Collecter les métriques déclaratives/heuristiques spécifiques au domaine :

```typescript
// Exemple pour tests-audit (Stage 71)
const metrics = {
  expectedTestFiles: 45,
  presentTestFiles: 43,
  missingTestFiles: 2,
  ucrWithTests: 38,
  ucrWithoutTests: 7,
  theoreticalCoverage: 84.4, // % UCR avec au moins 1 test associé
  criticalUcrCovered: true
};

// Exemple pour static-consistency (Stage 70)
const metrics = {
  totalImports: 120,
  unresolvedImports: 2,
  circularDependencies: 0,
  typeInconsistencies: 3,
  detectedSmells: {
    longFunctions: 2,
    deepNesting: 1,
    duplicateCode: 0
  },
  consistencyScore: 92.5 // Score heuristique
};

// Exemple pour functional-equivalence (Stage 72)
const metrics = {
  totalBehaviors: 28,
  covered: 24,
  partiallyCovered: 3,
  notCovered: 1,
  unknown: 0,
  criticalNotCovered: 0,
  equivalenceScore: 85.7 // Score déclaratif
};
```

#### Étape 4 : Calcul du score heuristique

Calculer un score de 0 à 100 pour le domaine (approche heuristique) :

```typescript
// Formule heuristique de scoring
const score = calculateHeuristicScore({
  checks: results,
  metrics: metrics,
  thresholds: validationContext.thresholds,
  weights: {
    criticalIssues: 0.4,    // Issues critiques = 40% du score
    warnings: 0.2,          // Avertissements = 20% du score
    metricsHeuristic: 0.3,  // Métriques heuristiques = 30% du score
    patternCompliance: 0.1  // Conformité patterns = 10% du score
  }
});

// Score normalisé 0-100
const normalizedScore = Math.max(0, Math.min(100, score));
```

**Critères de scoring heuristique recommandés :**

- **100** : Parfait, aucune issue, conformité totale aux patterns
- **85-99** : Excellent, quelques warnings mineurs non-bloquants
- **70-84** : Bon, améliorations recommandées mais pas critiques
- **60-69** : Acceptable, corrections nécessaires mais non bloquantes
- **0-59** : Insuffisant, issues critiques détectées, Gate ❌

#### Étape 5 : Détermination du Gate

Déterminer si le Gate est **passed** ou **failed** (approche réaliste) :

```typescript
const gate = determineGate({
  score: normalizedScore,
  criticalIssues: results.filter(r => r.severity === 'error' && r.critical),
  thresholds: validationContext.thresholds,
  domainCriticality: getDomainCriticality(domain)
});

// Règles réalistes de Gate par criticité de stage :
// 
// Stages CRITIQUES (70, 72, 73, 74) :
// - Gate ❌ si score < 70
// - Gate ❌ si au moins 1 issue critique non résolue
// - Gate ✅ sinon
//
// Stages NON-CRITIQUES (75, 76, 77) :
// - Gate ✅ même avec warnings si score ≥ 60
// - Issues documentées pour correction manuelle
// - Gate ❌ uniquement si score < 60 ou issue bloquante structurelle
```

#### Étape 6 : Génération des recommandations

Produire des recommandations actionnables :

```typescript
const recommendations = generateRecommendations({
  issues: results.filter(r => !r.passed),
  metrics: metrics,
  thresholds: validationContext.thresholds,
  bestPractices: loadBestPractices(domain)
});

// Exemples de recommandations :
// - "Augmenter la couverture théorique de 82% à 90% minimum"
// - "Résoudre les 2 imports non résolus dans CampaignCard.tsx"
// - "Ajouter des attributs ARIA manquants sur les 5 boutons interactifs"
```

#### Étape 7 : Génération des rapports

Produire les rapports JSON et Markdown :

```typescript
// Rapport JSON
const jsonReport = {
  domain,
  stageId,
  pageName,
  timestamp: new Date().toISOString(),
  validation: {
    status: determineStatus(score, results),
    score: normalizedScore,
    gate,
    issues: formatIssues(results)
  },
  metrics,
  summary: {
    totalChecks: checks.length,
    passedChecks: results.filter(r => r.passed).length,
    failedChecks: results.filter(r => !r.passed && r.severity === 'error').length,
    warningChecks: results.filter(r => !r.passed && r.severity === 'warning').length,
    skippedChecks: results.filter(r => r.skipped).length
  },
  thresholds: validationContext.thresholds,
  recommendations
};

await writeJSON(`${domain}.report.json`, jsonReport);

// Rapport Markdown
const markdownReport = generateMarkdownReport(jsonReport);
await writeFile(`${domain}.report.md`, markdownReport);

// Métadonnées
const metaReport = generateMetaReport(jsonReport, validationContext);
await writeJSON(`.meta/validation.${domain}.meta.json`, metaReport);
```

---

## 4. 🎯 Checks spécifiques au domaine **${domain}**

_(À compléter selon le domaine spécifique)_

### 4.1. Check : [Nom du check 1]

**Objectif :** Description précise du check

**Catégorie :** `consistency` | `smells` | `patterns` | `coherence` | ...

**Sévérité :** `error` | `warning` | `info`

**Type :** `heuristic` (analyse statique, pas d'exécution)

**Critères de succès :**
- Condition 1
- Condition 2

**Exemple de validation :**

```typescript
async function checkExample() {
  // Analyse heuristique du code
  const result = await analyzeHeuristically(generatedCode, rules);
  
  return {
    passed: result.isValid,
    message: result.isValid 
      ? "Check passed successfully"
      : `Check failed: ${result.error}`,
    details: {
      // Détails additionnels
    }
  };
}
```

**Issue générée si échec :**

```json
{
  "severity": "error",
  "category": "example-category",
  "message": "Descriptive error message",
  "location": {
    "file": "src/components/Example.tsx",
    "line": 42,
    "column": 10
  },
  "suggestion": "How to fix this issue"
}
```

### 4.2. Check : [Nom du check 2]

_(Répéter pour chaque check spécifique au domaine)_

---

## 5. 📊 Métriques et seuils

### 5.1. Métriques collectées

Liste des métriques heuristiques mesurées :

| Métrique | Description | Unité | Seuil minimal | Seuil optimal |
|----------|-------------|-------|---------------|---------------|
| ... | ... | ... | ... | ... |

**Exemple pour tests-audit (Stage 71) :**

| Métrique | Description | Unité | Seuil minimal | Seuil optimal |
|----------|-------------|-------|---------------|---------------|
| `theoreticalCoverage` | % UCR avec tests associés | % | 75 | 90 |
| `criticalUcrCovered` | UCR critiques avec tests | boolean | true | true |
| `missingTestFiles` | Fichiers tests manquants | count | 5 max | 0 |

**Exemple pour static-consistency (Stage 70) :**

| Métrique | Description | Unité | Seuil minimal | Seuil optimal |
|----------|-------------|-------|---------------|---------------|
| `consistencyScore` | Score cohérence globale | score | 70 | 90 |
| `unresolvedImports` | Imports non résolus | count | 0 | 0 |
| `circularDependencies` | Dépendances circulaires | count | 0 | 0 |

### 5.2. Seuils configurables

Les seuils sont définis dans :

1. **Stack guides** (`guide.quality-thresholds.md`)
2. **Configuration projet** (fichiers de config spécifiques)
3. **Valeurs par défaut** (hardcodées dans le guide)

Ordre de priorité : Config projet > Stack guides > Défaut

```typescript
const thresholds = resolveThresholds({
  domain,
  projectConfig: loadProjectConfig(),
  stackGuides: loadStackGuides(['quality-thresholds']),
  defaults: {
    minScore: 70,          // Stages critiques
    minScoreNonCritical: 60, // Stages non-critiques
    maxCriticalIssues: 0
  }
});
```

### 5.3. Calcul du score par métriques

Formule de scoring heuristique basée sur les métriques :

```typescript
function calculateMetricsScore(metrics, thresholds) {
  const scores = [];
  
  for (const [key, value] of Object.entries(metrics)) {
    const threshold = thresholds[key];
    if (!threshold) continue;
    
    // Score normalisé : 0 si sous minimum, 100 si au-dessus optimal
    const score = normalizeScore(
      value,
      threshold.min,
      threshold.optimal
    );
    
    scores.push({
      metric: key,
      value,
      threshold,
      score,
      weight: threshold.weight || 1
    });
  }
  
  // Score pondéré
  const totalWeight = scores.reduce((sum, s) => sum + s.weight, 0);
  const weightedScore = scores.reduce(
    (sum, s) => sum + (s.score * s.weight),
    0
  ) / totalWeight;
  
  return Math.round(weightedScore);
}
```

---

## 6. 🔗 Relations avec autres domaines de validation

### 6.1. Dépendances amont (pré-requis)

Le domaine **${domain}** peut dépendre de :

- **Domaine X** : Description de la dépendance
- **Domaine Y** : Description de la dépendance

**Exemple :**
- `tests-audit` (71) peut bénéficier de `static-consistency` (70) pour vérifier la cohérence avant
- `integration-consistency` (74) dépend de `dependencies-coherence` (73)

### 6.2. Dépendances avales (utilisateurs)

Les résultats de **${domain}** sont utilisés par :

- **validation-summary** (Stage 78) : Agrégation finale des scores
- Autres stages de validation qui s'appuient sur ce domaine

### 6.3. Relations croisées

Certains domaines partagent des données :

- **functional-equivalence** (72) ↔ **tests-audit** (71) : Les tests couvrent les behaviours
- **static-consistency** (70) ↔ **quality-compliance** (77) : Cohérence et qualité liées
- **dependencies-coherence** (73) ↔ **performance-patterns** (76) : Deps impactent la perf

---

## 7. ✅ Validation et auto-checks

### 7.1. Checks à l'entrée (Pre-validation)

Avant de valider, vérifier :

- [ ] Le code généré Phase 3 est disponible
- [ ] Les mappings Phase 2 sont disponibles
- [ ] Les inventaires Phase 1 sont disponibles (si requis)
- [ ] Les stack guides sont lisibles
- [ ] Les règles heuristiques sont chargées
- [ ] Les seuils de qualité sont définis
- [ ] Les patterns de détection sont prêts

### 7.2. Checks à la sortie (Post-validation)

Après validation, vérifier :

- [ ] Le rapport JSON est généré et valide
- [ ] Le rapport Markdown est généré et formaté
- [ ] Les métadonnées sont créées
- [ ] Le score est calculé (0-100)
- [ ] Le Gate est déterminé (passed/failed)
- [ ] Les issues sont listées et catégorisées
- [ ] Les recommandations sont générées
- [ ] Tous les checks ont été exécutés (ou skipped explicitement)

### 7.3. Auto-validation du rapport

Le rapport de validation doit lui-même être valide :

```typescript
function validateReport(report) {
  const errors = [];
  
  // Vérifier la structure
  if (!report.domain) errors.push("Missing domain");
  if (!report.validation) errors.push("Missing validation");
  if (typeof report.validation.score !== 'number') {
    errors.push("Score must be a number");
  }
  if (report.validation.score < 0 || report.validation.score > 100) {
    errors.push("Score must be between 0 and 100");
  }
  if (!['valid', 'warnings', 'rejected'].includes(report.validation.status)) {
    errors.push("Invalid status");
  }
  if (!['passed', 'failed'].includes(report.validation.gate)) {
    errors.push("Invalid gate");
  }
  
  // Vérifier la cohérence
  if (report.validation.gate === 'failed' && report.validation.score > 80) {
    errors.push("Inconsistent: gate failed but score > 80");
  }
  
  return { valid: errors.length === 0, errors };
}
```

---

## 8. 🚨 Gestion des erreurs

### 8.1. Erreurs bloquantes (Gate ❌)

Les situations suivantes **doivent** déclencher un `Gate ❌` selon la criticité du stage :

**Stages CRITIQUES (70, 72, 73, 74) :**

- **Issue critique structurelle** : Incohérence majeure détectée (imports cycliques critiques, collision de noms, etc.)
- **Score < 70** : Le score heuristique est insuffisant
- **UCR critique non couverte** : (Stage 72) Behaviour critique notCovered
- **Dépendance critique manquante** : (Stage 73) Import non résolu critique

**Stages NON-CRITIQUES (75, 76, 77) :**

- **Score < 60** : Score vraiment trop bas
- **Violation structurelle bloquante** : Pattern architectural complètement cassé

**Exemples selon le domaine (approche réaliste) :**

- `static-consistency` (70) : Imports cycliques sur composants critiques, types props incohérents majeurs
- `tests-audit` (71) : 0% couverture théorique sur use-cases critiques (Gate ❌ si critique)
- `functional-equivalence` (72) : UCR critiques notCovered, regressions bloquantes
- `dependencies-coherence` (73) : Imports critiques non résolus
- `integration-consistency` (74) : Collisions de routes critiques, chemins invalides
- `accessibility-heuristic` (75) : Composants interactifs sans label (mais non-bloquant si < 5)
- `performance-patterns` (76) : Imports massifs non lazy sur route principale (souvent warning)
- `quality-compliance` (77) : Pattern archi complètement violé sur composants principaux

### 8.2. Avertissements (non-bloquants)

Les situations suivantes génèrent un **warning** mais ne bloquent pas le Gate :

- **Issue non-critique** : Issue de sévérité `warning`
- **Métrique sous optimal** : Métrique entre seuil minimal et optimal
- **Best practice non respectée** : Recommandation non suivie
- **Check optionnel échoué** : Check non-critique échoué

**Exemples selon le domaine :**

- `static-consistency` : Quelques code smells détectés (fonctions longues)
- `tests-audit` : Couverture théorique 82% (sous optimal 90% mais > minimal 75%)
- `accessibility-heuristic` : Quelques labels ARIA manquants sur composants secondaires
- `performance-patterns` : Absence de mémoïsation sur listes moyennes

### 8.3. Erreurs d'infrastructure (limitées dans l'approche heuristique)

Erreurs liées au contexte d'analyse (pas au code généré) :

- **Inputs manquants** : Code généré, mappings ou inventaires absents
- **Stack-guides illisibles** : Fichiers de règles corrompus ou introuvables
- **Règles heuristiques invalides** : Patterns de détection malformés
- **Timeout d'analyse** : L'analyse statique a dépassé le temps maximum

Ces erreurs doivent être **remontées clairement** et ne doivent **pas** être confondues avec des issues du code validé.

```json
{
  "validation": {
    "status": "rejected",
    "gate": "failed",
    "infrastructureError": {
      "type": "input-missing",
      "message": "Generated code directory not found: phase-3-generation/src_new/",
      "suggestion": "Ensure Phase 3 completed successfully before running Phase 4"
    }
  }
}
```

**Note importante :** Dans l'approche heuristique, il n'y a **PAS** d'erreurs liées à :
- Outils manquants (compilateur, linter, test runner) — car non exécutés
- Configuration runtime invalide — car pas d'exécution
- Ressources insuffisantes — analyse statique légère

---

## 9. 📚 Exemples complets

### 9.1. Exemple : Validation réussie

**Contexte :** Validation de `static-consistency` (Stage 70) avec code cohérent

**Rapport JSON :**

```json
{
  "domain": "static-consistency",
  "stageId": "70",
  "pageName": "CampaignsDetail",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "validation": {
    "status": "valid",
    "score": 95,
    "gate": "passed",
    "issues": []
  },
  "metrics": {
    "totalImports": 87,
    "unresolvedImports": 0,
    "circularDependencies": 0,
    "typeInconsistencies": 0,
    "detectedSmells": {
      "longFunctions": 0,
      "deepNesting": 0,
      "duplicateCode": 0
    },
    "consistencyScore": 95.0
  },
  "summary": {
    "totalChecks": 6,
    "passedChecks": 6,
    "failedChecks": 0,
    "warningChecks": 0,
    "skippedChecks": 0
  },
  "thresholds": {
    "maxUnresolvedImports": 0,
    "maxCircularDeps": 0,
    "minConsistencyScore": 70
  },
  "recommendations": []
}
```

**Rapport Markdown :**

```markdown
# 🔍 Rapport de Validation — Static Consistency & Code Smells

**Page :** CampaignsDetail  
**Date :** 2025-12-08 10:00:00  
**Statut :** ✅ Valid  
**Score :** 95/100  
**Gate :** ✅ Passed

---

## 📊 Résumé

- **Total vérifications :** 6
- **Réussies :** 6 ✅
- **Échouées :** 0 ❌
- **Avertissements :** 0 ⚠️

---

## 📈 Métriques (Analyse Heuristique)

| Métrique | Valeur | Seuil | Statut |
|----------|--------|-------|--------|
| Imports non résolus | 0 | 0 max | ✅ |
| Dépendances circulaires | 0 | 0 max | ✅ |
| Incohérences types/props | 0 | 0 max | ✅ |
| Fonctions longues | 0 | - | ✅ |
| Imbrication profonde | 0 | - | ✅ |
| Code dupliqué | 0 | - | ✅ |
| Score de cohérence | 95.0 | ≥ 70 | ✅ |

---

## ✅ Conclusion

Le code généré est structurellement cohérent. Tous les imports sont résolvables,
aucune dépendance circulaire détectée, pas de code smell majeur.
Aucune amélioration nécessaire.

---

*Généré par ai-orchestrator-v4 — Phase 4 — Stage 70 (Analyse Heuristique)*
```

### 9.2. Exemple : Validation avec warnings

**Contexte :** Validation de `tests-audit` (Stage 71) avec bonne couverture théorique mais quelques UCR sans tests

**Rapport JSON :**

```json
{
  "domain": "tests-audit",
  "stageId": "71",
  "pageName": "CampaignsDetail",
  "timestamp": "2025-12-08T10:05:00.000Z",
  "validation": {
    "status": "warnings",
    "score": 82,
    "gate": "passed",
    "issues": [
      {
        "severity": "warning",
        "category": "coverage-theoretical",
        "message": "UCR without associated test file",
        "location": {
          "ucr": "logic-validation-campaignForm-1"
        },
        "suggestion": "Create test file for form validation logic"
      },
      {
        "severity": "warning",
        "category": "coverage-theoretical",
        "message": "UCR without associated test file",
        "location": {
          "ucr": "logic-computation-priceCalculation-1"
        },
        "suggestion": "Create test file for price calculation logic"
      }
    ]
  },
  "metrics": {
    "expectedTestFiles": 42,
    "presentTestFiles": 40,
    "missingTestFiles": 2,
    "totalUcrs": 45,
    "ucrWithTests": 37,
    "ucrWithoutTests": 8,
    "criticalUcrWithTests": 15,
    "criticalUcrWithoutTests": 0,
    "theoreticalCoverage": 82.2
  },
  "summary": {
    "totalChecks": 5,
    "passedChecks": 3,
    "failedChecks": 0,
    "warningChecks": 2,
    "skippedChecks": 0
  },
  "thresholds": {
    "minTheoreticalCoverage": 75,
    "optimalTheoreticalCoverage": 90,
    "criticalUcrMustHaveTests": true
  },
  "recommendations": [
    "Add test files for 2 missing test scenarios",
    "Increase theoretical coverage from 82.2% to 90% by testing all UCR",
    "Prioritize testing the 8 UCR without associated tests"
  ]
}
```

### 9.3. Exemple : Validation échouée

**Contexte :** Validation de `functional-equivalence` (Stage 72) avec UCR critiques non couverts

**Rapport JSON :**

```json
{
  "domain": "functional-equivalence",
  "stageId": "72",
  "pageName": "CampaignsDetail",
  "timestamp": "2025-12-08T10:10:00.000Z",
  "validation": {
    "status": "rejected",
    "score": 58,
    "gate": "failed",
    "issues": [
      {
        "severity": "error",
        "category": "critical-regression",
        "message": "Critical behavior not covered in generated code",
        "location": {
          "ucr": "behavior-campaign-validation-duplicateName-1",
          "legacyFile": "src/legacy/CampaignsDetail/validation.js"
        },
        "suggestion": "Implement duplicate name validation in campaign form"
      },
      {
        "severity": "error",
        "category": "critical-regression",
        "message": "Critical use-case not implemented",
        "location": {
          "ucr": "usecase-campaign-bulk-delete-1",
          "legacyFile": "src/legacy/CampaignsDetail/actions.js"
        },
        "suggestion": "Implement bulk delete functionality for campaigns"
      },
      {
        "severity": "warning",
        "category": "partial-coverage",
        "message": "Behavior partially covered - edge case missing",
        "location": {
          "ucr": "behavior-campaign-save-offline-1"
        },
        "suggestion": "Add offline handling for campaign save operation"
      }
    ]
  },
  "metrics": {
    "totalBehaviors": 32,
    "covered": 22,
    "partiallyCovered": 5,
    "notCovered": 5,
    "unknown": 0,
    "criticalBehaviors": 8,
    "criticalCovered": 6,
    "criticalNotCovered": 2,
    "equivalenceScore": 68.8
  },
  "summary": {
    "totalChecks": 8,
    "passedChecks": 5,
    "failedChecks": 3,
    "warningChecks": 1,
    "skippedChecks": 0
  },
  "thresholds": {
    "minEquivalenceScore": 80,
    "criticalBehaviorsMustBeCovered": true
  },
  "recommendations": [
    "CRITICAL: Implement the 2 missing critical behaviors before integration",
    "Address the 5 fully not-covered behaviors",
    "Improve partial coverage for 5 behaviors",
    "Target equivalence score of 80% minimum (currently 68.8%)"
  ]
}
```

---

## 10. 🔄 Évolution et maintenance

### 10.1. Ajout d'un nouveau check

Pour ajouter un nouveau check de validation :

1. **Documenter le check** dans la section 4 de ce guide
2. **Définir les critères** de succès/échec
3. **Ajouter le check** au code du stage
4. **Mettre à jour les seuils** si nécessaire
5. **Ajouter des exemples** de validation réussie/échouée
6. **Mettre à jour le scoring** si le check impacte le score

### 10.2. Modification d'un seuil existant

Pour modifier un seuil de qualité :

1. **Justifier le changement** (contexte projet, évolution standards)
2. **Mettre à jour** `guide.quality-thresholds.md`
3. **Documenter** l'impact sur le scoring
4. **Tester** avec du code existant
5. **Communiquer** le changement à l'équipe

### 10.3. Ajout d'un nouveau domaine de validation

Pour ajouter un nouveau domaine (ex: `security-audit`) :

1. **Créer** `guide.validation.security-audit.md` à partir de ce template
2. **Définir** les checks spécifiques au domaine
3. **Ajouter** le domaine à la Phase 4 specification
4. **Créer** le stage markdown correspondant
5. **Mettre à jour** le stage `validation-summary` (78) pour inclure le nouveau domaine

---

## 11. 📖 Références

### 11.1. Guides liés

- `PHASE-4-SPECIFICATION.md` — Spécification complète Phase 4
- `guide.ucr.md` — Règles de nommage UCR
- `guide.json-schema-validation.md` — Validation JSON
- `core/templates/stage-markdown.template.md` — Template de stage
- Stack guides Phase 0 (selon le domaine)

### 11.2. Artefacts référencés

- Phase 3 : `phase-3-generation/src_new/**/*` — Code généré
- Phase 2 : `mapping.*.json` — Mappings
- Phase 1 : `inventory.*.json` — Inventaires
- Phase 0 : `stack-guides/guide.*.md` — Guides de stack
- Legacy : `${paths.legacySource}` — Code Legacy (référence déclarative)

### 11.3. Outils et approche

**Important :** Dans l'approche réaliste de Phase 4, **aucun outil externe n'est exécuté**.

La validation est purement **heuristique et déclarative** :

- **Pas de compilation réelle** : Analyse statique des types/imports via parsing AST ou regex
- **Pas de lint exécuté** : Détection de patterns via règles heuristiques
- **Pas de tests exécutés** : Audit déclaratif de la stratégie de tests
- **Pas de build réel** : Vérification de cohérence structurelle
- **Pas de runtime** : Aucun serveur, browser, ou environnement d'exécution

Les **règles heuristiques** et **patterns de détection** sont définis dans les stack guides :

- `guide.validation.md` — Règles générales d'audit
- `guide.quality-thresholds.md` — Seuils de qualité
- `guide.accessibility.md` — Patterns d'accessibilité à vérifier
- `guide.performance.md` — Patterns de performance à détecter

---

© 2025 Sylvain Chabaud — ai-orchestrator-v4 — Phase 4

````
