# GeoR2LLMs — Phase 2 : Exploration Multi-LLMs et Multi-Datasets

**Master 2 Intelligence Artificielle — Université Toulouse III Paul Sabatier**
Projet GeoR2LLMs — Encadré par Lynda Tamine-Lechani (IRIT)
Année universitaire 2025–2026

---

## Objectif

Explorer les performances de différents LLMs dans une configuration RAG pour la question-réponse géographique, en variant :
1. **Les modèles de génération** (5 LLMs de familles et tailles différentes)
2. **Les datasets** (GeoSQA, GKMC, Tourism-QA)
3. **Les configurations** (Baseline sans RAG vs RAG avec Wikipedia/Reviews)

---

## Modèles évalués

### Requis
| Modèle | Famille | Taille | HuggingFace ID |
|--------|---------|--------|----------------|
| Qwen3-0.6B | Qwen | 0.6B | `Qwen/Qwen3-0.6B` |
| Qwen3-1.7B | Qwen | 1.7B | `Qwen/Qwen3-1.7B` |
| Llama-3.2-1B | Meta | 1.0B | `meta-llama/Llama-3.2-1B-Instruct` |
| Llama-3.2-3B | Meta | 3.0B | `meta-llama/Llama-3.2-3B-Instruct` |
| Ministral-3B | Mistral | 3.0B | `mistralai/Ministral-3-3B-Instruct-2512` |

### Optionnels (si GPU suffisant)
| Modèle | Famille | Taille | HuggingFace ID |
|--------|---------|--------|----------------|
| Qwen3-8B | Qwen | 8B | `Qwen/Qwen3-8B` |
| Llama-3.1-8B | Meta | 8B | `meta-llama/Llama-3.1-8B-Instruct` |

### Encodeur (commun à tous)
- **BAAI/bge-m3** (1024 dimensions, multilingue)
- Index : FAISS IndexFlatIP (cosine similarity)

---

## Datasets

| Dataset | Source | Format | Tâche | Corpus RAG |
|---------|--------|--------|-------|------------|
| **GeoSQA** | `rfr2003/Geo_Benchmark` config `GeoSQA` | QCM (A/B/C/D) | Géographie générale | Wikipedia FR |
| **GKMC** | `rfr2003/Geo_Benchmark` config `GKMC` | QCM (A/B/C/D) | Géographie (similaire GeoSQA) | Wikipedia FR |
| **Tourism-QA** | `rfr2003/Geo_Benchmark` config `TourismQA` | Recommandation POI | Tourisme | Reviews utilisateurs |

---

## Structure des notebooks

```
Phase2/
├── Phase2a_Comparaison_LLMs.ipynb     ← Comparaison 5 LLMs sur GeoSQA
├── Phase2b_GKMC.ipynb                 ← Évaluation sur GKMC
├── Phase2c_TourismQA.ipynb            ← Évaluation sur Tourism-QA (recommandation POI)
├── Phase2d_Analyse_Facteurs.ipynb     ← Analyse factorielle croisée
├── results/                           ← Résultats produits par les notebooks
│   ├── phase2a_all_results.json
│   ├── phase2a_comparison.csv
│   ├── phase2b_all_results.json
│   ├── phase2b_comparison.csv
│   ├── phase2c_all_results.json
│   ├── phase2c_comparison.csv
│   ├── phase2d_unified_results.csv
│   ├── phase2d_analysis_summary.json
│   └── *.png (figures)
└── README.md
```

---

## Ordre d'exécution

Les notebooks doivent être exécutés dans l'ordre suivant :

1. **Phase2a** → Résultats GeoSQA (requis pour Phase2d)
2. **Phase2b** → Résultats GKMC (requis pour Phase2d)
3. **Phase2c** → Résultats Tourism-QA (requis pour Phase2d)
4. **Phase2d** → Analyse croisée (charge les résultats de 2a/2b/2c)

> **Note** : Phase2d peut être exécuté même si certains résultats manquent — il génère des données de démonstration en fallback.

---

## Utilisation

### Pour chaque notebook (2a, 2b, 2c) :
1. Ouvrir dans Google Colab
2. Activer le GPU T4 : `Environnement d'exécution` → `Modifier le type` → **GPU T4**
3. Exécuter tout : `Exécution` → `Tout exécuter`
4. Durée estimée : ~1-3h par notebook (5 modèles × N questions)

### Pour Phase2d :
1. Copier les fichiers JSON résultats de 2a/2b/2c dans `results/`
2. Exécuter le notebook (pas de GPU nécessaire)

---

## Métriques

### Génération
- **Accuracy (MCQ)** : correspondance de la lettre prédite (GeoSQA, GKMC)
- **Accuracy@1** : POI correct en première position (Tourism-QA)
- **Exact Match** : correspondance exacte du texte
- **F1 Score** : F1 au niveau des tokens
- **Contains** : présence des mots-clés

### Retrieval
- **Hit Rate@K** (K=1, 3, 5) : réponse dans les top-K documents
- **MRR@K** : Mean Reciprocal Rank

### Analyse factorielle
- **Corrélation taille/performance** : Pearson
- **Comparaison familles** : ANOVA
- **Impact RAG** : t-test apparié
- **Efficacité** : Accuracy / (Taille × Temps)

---

## Facteurs analysés dans Phase2d

| Facteur | Description | Analyse |
|---------|-------------|---------|
| Taille du modèle | 0.6B → 3B | Corrélation Pearson |
| Famille de modèle | Qwen vs Llama vs Mistral | ANOVA + comparaison |
| Type de dataset | GeoSQA vs GKMC vs Tourism-QA | Difficulté relative |
| Impact du RAG | Baseline vs RAG | Delta + t-test |
| Latence | Temps/question | Compromis perf/vitesse |

---

## Références

- [Qwen3 — HuggingFace](https://huggingface.co/collections/Qwen/qwen3)
- [Llama 3.2 — HuggingFace](https://huggingface.co/collections/meta-llama/llama-32-66f448ffc8c32f949b04c8cf)
- [Ministral 3 — HuggingFace](https://huggingface.co/collections/mistralai/ministral-3)
- [BGE-M3 — arXiv](https://arxiv.org/abs/2402.03216)
- [GeoSQA — arXiv](https://arxiv.org/abs/1908.07855)
- [Tourism-QA — ACM](https://dl.acm.org/doi/10.1145/3459637.3482320)
- [FAISS — Facebook Research](https://github.com/facebookresearch/faiss)
- [RAG — Lewis et al. 2020](https://arxiv.org/abs/2005.11401)

---

## Auteurs

Projet GeoR2LLMs — Master 2 Intelligence Artificielle
Université Toulouse III — Paul Sabatier
Année universitaire 2025–2026
