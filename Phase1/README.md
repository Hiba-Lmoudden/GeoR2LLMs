# GeoR2LLMs — Phase 1 : Système RAG Géographique

**Master 2 Intelligence Artificielle — Université Toulouse III Paul Sabatier**  
Projet européen GoR2LLMs

---

## Présentation

Ce dépôt contient l'implémentation de la Phase 1 du projet GeoR2LLMs : un système de Question-Réponse géographique basé sur l'architecture RAG (Retrieval-Augmented Generation).

L'objectif est de mesurer l'apport du RAG sur des questions géographiques en français, en comparant un LLM seul (baseline) à un LLM augmenté d'un corpus Wikipedia.

### Architecture

| Composant | Choix |
|-----------|-------|
| LLM | Qwen/Qwen2.5-1.5B-Instruct |
| Embeddings | BAAI/bge-m3 |
| Index vectoriel | FAISS |
| Dataset | GeoSQA (rfr2003/Geo_Benchmark) |
| Corpus | Wikipedia français |
| Environnement | Google Colab (GPU T4) |

---

## Utilisation

Tout le code se trouve dans un seul notebook autonome. Aucune installation locale n'est requise.

**[▶ Ouvrir dans Google Colab](https://colab.research.google.com/github/VOTRE_USERNAME/VOTRE_REPO/blob/main/Phase1_GeoR2LLMs_Colab.ipynb)**

### Étapes

1. Ouvrir le notebook dans Google Colab (lien ci-dessus)
2. Activer le GPU : `Environnement d'exécution` → `Modifier le type d'exécution` → **GPU T4**
3. Lancer tout : `Exécution` → `Tout exécuter`
4. Durée estimée : ~20–30 min (première exécution, téléchargements inclus)

---

## Contenu du notebook

Le notebook est entièrement autonome et contient dans l'ordre :

- **Étape 1** — Installation automatique de toutes les dépendances
- **Étape 2** — Définition des modules (Retriever, Generator, Evaluator)
- **Étape 3** — Chargement des données (GeoSQA + Wikipedia français)
- **Étape 4** — Initialisation de BGE-M3 et Qwen2.5-1.5B
- **Étape 5** — Évaluation baseline (LLM seul, sans RAG)
- **Étape 6** — Évaluation RAG (BGE-M3 + Wikipedia + Qwen)
- **Étape 7** — Visualisations et tableau comparatif
- **Étape 8** — Analyse qualitative : succès et échecs
- **Étape 9** — Sauvegarde des résultats (JSON, CSV, PNG) sur Google Drive

---

## Résultats attendus

### Métriques globales

| Métrique | Baseline (sans RAG) | RAG (avec Wikipedia) | Amélioration |
|----------|--------------------|-----------------------|--------------|
| Exact Match | ~40–50% | ~60–75% | +15–25 pts |
| F1 Score | ~45–55% | ~60–70% | +10–20 pts |
| Contains | ~50–60% | ~70–85% | +15–25 pts |

### Performance par type de question

| Type | Baseline | RAG | Commentaire |
|------|----------|-----|-------------|
| Factuelle simple | ~60% | ~80% | Bonne amélioration avec contexte |
| Factuelle numérique | ~40% | ~60% | LLM tend à halluciner les chiffres |
| Spatiale — distance | ~15% | ~35% | Calcul géométrique nécessaire |
| Spatiale — direction | ~10% | ~30% | Idem, limites du RAG textuel |

---

## Difficultés identifiées

### 1. Questions spatiales (distances et directions)
Les questions comme "Quelle est la distance entre Paris et Lyon ?" obtiennent de faibles scores même avec RAG, car Wikipedia ne contient pas de matrice de distances prête à l'emploi. Le LLM hallucine des chiffres avec confiance.

**Solution prévue en Phase 2** : module de calcul spatial (formule de Haversine pour les distances, azimut pour les directions).

### 2. Qualité du corpus
Les 100 premiers articles de Wikipedia sont aléatoires et peuvent ne pas couvrir les villes ou régions demandées. Un corpus garanti de 9 articles géographiques clés est inclus pour compenser.

**Solution prévue en Phase 2** : corpus spécialisé (GeoNames, IGN, sources touristiques).

### 3. Taille du LLM
Le modèle Qwen2.5-0.5B est trop petit pour le français — il produit des réponses inexactes même sur des faits simples. Le passage à 1.5B améliore significativement la qualité.

**Solution prévue en Phase 2** : comparaison systématique de Qwen2.5-1.5B, Llama-3.2-3B, Mistral-3B.

---

## Structure du dépôt

```
GeoR2LLMs-Phase1/
├── Phase1_GeoR2LLMs_Colab.ipynb   ← Notebook principal (tout est ici)
├── results                        ← Resultats produits par le notebook
    ├── fig1_comparison.png
    ├── fig2_by_type.png
    ├── phase1_metrics.json
    └── phase1_predictions.csv
├── rapport.pdf                    ← Rapport qui analyse et explique les resultats trouvés
└── README.md                      ← Ce fichier

```

Les résultats générés par le notebook (JSON, CSV, graphiques) sont aussi sauvegardés automatiquement sur Google Drive dans `Mon Drive/GeoR2LLMs/Phase1/`.

---

## Feuille de route

```
[x]-Phase 1 (Jan–Mars 2026)    
   └─ Baseline RAG : BGE-M3 + Qwen2.5-1.5B + GeoSQA + Wikipedia

[]-Phase 2a (Mars 2026)      
  └─ Comparaison LLMs : Qwen2.5-1.5B vs Llama-3.2-3B vs Mistral-3B

[]-Phase 2b (Avril 2026)
  └─ Module spatial : calcul Haversine (distances), azimut (directions)

[]-Phase 2c (Avril–Mai 2026)
  └─ Nouveaux datasets : GKMC, Tourism-QA
```

---

## Références

- [Qwen2.5 — HuggingFace](https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct)
- [BGE-M3 — Paper arXiv](https://arxiv.org/abs/2402.03216)
- [GeoSQA Dataset](https://huggingface.co/datasets/rfr2003/Geo_Benchmark)
- [FAISS — Facebook Research](https://github.com/facebookresearch/faiss)
- [RAG — Lewis et al. 2020](https://arxiv.org/abs/2005.11401)

---

## Auteurs

Projet GeoR2LLMs — Master 2 Intelligence Artificielle  
Université Toulouse III — Paul Sabatier  
Année universitaire 2025–2026
