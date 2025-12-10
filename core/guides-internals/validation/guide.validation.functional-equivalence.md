# 🔄 Guide Validation — functional-equivalence
*(Domaine de validation : **functional-equivalence** — ai-orchestrator-v4)*

---

## 1. 🎯 Objectif du domaine

Le domaine **functional-equivalence** a pour but d'établir une **matrice déclarative d'équivalence fonctionnelle** entre :  
- les **UCR** (Use Case Requirements) et **behaviours** issus du legacy (Phases 1 & 2),  
- et les **implémentations générées** dans la stack cible (Phase 3).

Il répond à la question :

> **« Les comportements fonctionnels du legacy sont-ils représentés dans le code généré ? »**

Ce domaine **ne teste pas** l'exécution.  
Il réalise une **analyse statique** fondée sur :  
- les inventaires,  
- les mappings,  
- les stack-guides,  
- et le code généré `src_new`.

---

## 2. 📦 Artefacts attendus

Dans :  
```
${paths.workspace}/projects/${project.name}/pages/${project.pageName}/phase-4-validation/functional-equivalence/
```

Fichiers obligatoires :

- `equivalence-matrix.json`  
- `equivalence-matrix.report.md`  
- `regressions-detected.json`  
- `.meta/validation.functional-equivalence.meta.json`  

---

## 3. 📄 Structure du rapport JSON

```jsonc
{
  "domain": "functional-equivalence",
  "stageId": "72",
  "pageName": "${project.pageName}",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "summary": {
    "ucrTotal": 0,
    "ucrCritical": 0,
    "covered": 0,
    "partiallyCovered": 0,
    "notCovered": 0,
    "unknown": 0
  },
  "matrix": [
    {
      "ucr": "UCR-001",
      "description": "Description du behaviour",
      "coverage": "covered | partiallyCovered | notCovered | unknown",
      "evidence": [
        {
          "file": "path/to/file.tsx",
          "line": 42,
          "confidence": 0.92
        }
      ]
    }
  ],
  "regressions": [
    {
      "ucr": "UCR-CRIT-05",
      "message": "Critical UCR not covered",
      "severity": "critical"
    }
  ],
  "score": 0,
  "validation": {
    "status": "valid | warnings | rejected",
    "gate": "passed | failed"
  },
  "thresholds": {
    "minScore": 70,
    "criticalUcrMustBeCovered": true
  }
}
```

---

## 4. 📄 Structure du rapport Markdown

```markdown
# 🔄 Rapport de Validation — Functional Equivalence Matrix

**Page :** ${project.pageName}  
**Date :** 2025-12-08  
**Score :** 88/100  
**Gate :** ✅ Passed

---

## 📊 Vue d'ensemble

| Type UCR | Total | Couvert | Partiel | Non Couvert | Inconnu |
|----------|-------|---------|----------|--------------|---------|
| Standard | 10    | 8       | 1        | 1            | 0       |
| Critique | 3     | 3       | 0        | 0            | 0       |

---

## 🚨 UCR problématiques

### ❌ UCR non couvertes
- UCR-007 : Behaviour manquant dans la génération

### ⚠️ Couverture partielle
- UCR-004 : logique seulement partielle identifiée

---

## 💡 Recommandations
1. Ajouter les handlers manquants pour les UCR non couvertes.  
2. Compléter la logique partielle du composant `FooComponent.tsx`.  

---

## ✅ Conclusion
Le périmètre critique est couvert.  
Quelques compléments requis pour la couverture standard.

---
*Généré par ai-orchestrator-v4 — Stage 72 (functional-equivalence)*
```

---

## 5. 🧾 Métadonnées de validation

```jsonc
{
  "stage": "72",
  "domain": "functional-equivalence",
  "phase": "phase-4-validation",
  "timestamp": "2025-12-08T10:00:00.000Z",
  "gate": "passed | failed",
  "inputs": {
    "mappings": "phase-2-interpretation/mappings/*.json",
    "inventories": "phase-1-analysis/inventories/*.json",
    "generatedCode": "phase-3-generation/src_new/"
  },
  "outputs": {
    "matrix": "equivalence-matrix.json",
    "report": "equivalence-matrix.report.md"
  },
  "checksExecuted": 0,
  "errors": []
}
```

---

## 6. 🧠 Règles de validation functional-equivalence

### 6.1 Inputs requis
- Inventaires Phase 1  
- Mappings Phase 2  
- Code généré Phase 3  
- Stack-guides (optionnel)

### 6.2 Checks obligatoires

1. **Extraction UCR**
   - Récupérer toutes les UCR (standard + critiques).

2. **Association heuristique UCR → implémentation**
   - Par nommage, composants liés, handlers détectés.

3. **Classification**
   - `covered`: présence forte et claire
   - `partiallyCovered`: présence mais incomplète
   - `notCovered`: rien identifié
   - `unknown`: ambigu / indéterminable

4. **Détection des régressions**
   - Si UCR critique en `notCovered` → **régression critique**

5. **Scoring**
   - Score global basé sur taux de couverture pondéré.

### 6.3 Gate

**Gate ✅ si :**
- Aucune UCR critique en `notCovered`
- Score ≥ minScore (ex : 70)

**Gate ❌ sinon.**

---

## 7. Auto-checks recommandés

- [ ] Tous les mappings UCR présents  
- [ ] UCR critiques identifiées  
- [ ] Analyse code → UCR effectuée  
- [ ] Matrice générée  
- [ ] Score logique  


