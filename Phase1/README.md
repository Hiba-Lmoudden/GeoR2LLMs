# GeoR2LLMs — Phase 1 : Système RAG Géographique (Version Corrigée)

**Master 2 Intelligence Artificielle — Université Toulouse III Paul Sabatier**
Projet GeoR2LLMs — Encadré par Lynda Tamine-Lechani (IRIT)
Année universitaire 2025–2026

---

## Présentation

Ce dossier contient la **version corrigée** de la Phase 1 du projet GeoR2LLMs : un système de Question-Réponse géographique basé sur l'architecture RAG (Retrieval-Augmented Generation).

L'objectif est de mesurer l'apport du RAG sur des questions géographiques en comparant :
- **Baseline** : un LLM seul (sans contexte externe)
- **RAG** : un LLM augmenté de passages récupérés depuis Wikipedia

### Configuration

| Composant | Choix |
|-----------|-------|
| LLM | Qwen/Qwen3-0.6B |
| Embeddings | BAAI/bge-m3 (1024 dimensions) |
| Index vectoriel | FAISS (IndexFlatIP, cosine similarity) |
| Dataset | GeoSQA (`rfr2003/Geo_Benchmark`, config `GeoSQA`) |
| Corpus | Wikipedia français (`wikimedia/wikipedia`, `20231101.fr`) |
| Chunking | 512 caractères, overlap 50 |
| Top-K | 5 documents récupérés |
| Environnement | Google Colab (GPU T4) |

---

## Utilisation

Tout le code se trouve dans un seul notebook autonome, exécutable sur Google Colab.

### Étapes

1. Ouvrir le notebook `Phase1_GeoR2LLMs.ipynb` dans Google Colab
2. Activer le GPU : `Environnement d'exécution` → `Modifier le type d'exécution` → **GPU T4**
3. Lancer tout : `Exécution` → `Tout exécuter`
4. Les résultats sont automatiquement sauvegardés dans le dossier `results/`

---

## Contenu du notebook

Le notebook est organisé en **10 étapes** :

| Étape | Description |
|-------|-------------|
| 1 | Installation automatique des dépendances |
| 2 | Définition des modules (Retriever BGE-m3, Generator Qwen3, Evaluator) |
| 3 | Chargement des données (GeoSQA + Wikipedia français) |
| 4 | Indexation du corpus et initialisation des modèles |
| 5 | Évaluation Baseline (LLM seul, sans RAG) |
| 6 | Évaluation RAG (LLM + contexte Wikipedia) |
| 7 | Métriques de Retrieval (Hit Rate@K, MRR@K) |
| 8 | Visualisations (graphiques comparatifs) |
| 9 | Analyse qualitative (succès, échecs, difficultés) |
| 10 | Sauvegarde des résultats (JSON, CSV, PNG, Google Drive) |

---

## Métriques évaluées

### Génération (LLM)
- **Accuracy (MCQ)** : correspondance de la lettre prédite avec la réponse correcte
- **Exact Match** : correspondance exacte du texte (après normalisation)
- **F1 Score** : F1 au niveau des tokens
- **Contains** : présence des mots-clés de la vérité dans la prédiction

### Retrieval (BGE-m3 + FAISS)
- **Hit Rate@K** (K=1, 3, 5) : proportion de questions où la bonne réponse apparaît dans les top-K documents
- **MRR@K** : Mean Reciprocal Rank du premier document pertinent

---

## Structure du dépôt

```
Phase1/
├── Phase1_GeoR2LLMs.ipynb        ← Notebook principal (tout est ici)
├── results/                      ← Résultats produits par le notebook
│   ├── phase1_metrics.json       ← Toutes les métriques (JSON)
│   ├── phase1_predictions.csv    ← Prédictions détaillées (CSV)
│   ├── fig1_baseline_vs_rag.png  ← Comparaison Baseline vs RAG
│   ├── fig2_retrieval_metrics.png ← Métriques de retrieval
│   └── fig3_f1_distribution.png  ← Distribution des scores F1
├── rapport/                      
│   ├── rapport_phase1.pdf        ← Rapport qui analyse et explique les résultats trouvés
│   ├── rapport_phase1.tex        ← Rapport au format latex
│   └── references.bib            ← Références bibliographiques utilisées dans le rapport
└── README.md                     ← Ce fichier
```

## Prochaines étapes (Phase 2)

- **Phase 2a** : Comparaison de LLMs (Qwen3-1.7B, Llama-3.2-1B/3B, Ministral-3B)
- **Phase 2b** : Module spatial (Haversine pour distances, azimut pour directions)
- **Phase 2c** : Nouveaux datasets (GKMC, Tourism-QA)
- **Phase 2d** : Analyse factorielle des résultats

---

## Références

- [Qwen3-0.6B — HuggingFace](https://huggingface.co/Qwen/Qwen3-0.6B)
- [BGE-M3 — Paper arXiv](https://arxiv.org/abs/2402.03216)
- [GeoSQA Dataset](https://huggingface.co/datasets/rfr2003/Geo_Benchmark)
- [FAISS — Facebook Research](https://github.com/facebookresearch/faiss)
- [RAG — Lewis et al. 2020](https://arxiv.org/abs/2005.11401)

---

## Auteurs

Projet GeoR2LLMs — Master 2 Intelligence Artificielle
Université Toulouse III — Paul Sabatier
Année universitaire 2025–2026
