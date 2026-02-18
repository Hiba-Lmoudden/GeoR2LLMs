# Analyse critique de l'ancienne Phase 1

**Document justificatif — Pourquoi l'ancienne implémentation était incorrecte**

Ce document détaille les problèmes identifiés dans l'implémentation originale (`Phase1/`) et justifie pourquoi une réécriture complète était nécessaire.

---

## 1. Mauvais modèle LLM (Critique)

**Problème** : L'ancienne version utilise `Qwen/Qwen2.5-1.5B-Instruct` (code du notebook) alors que le cahier des charges spécifie explicitement **Qwen3-0.6B**.

**Preuves** :
- Cellule 13 du notebook original : `model_name = 'Qwen/Qwen2.5-1.5B-Instruct'`
- Fichier `phase1_metrics.json` : `"llm": "Qwen/Qwen2.5-0.5B-Instruct"` (encore un autre modèle !)
- Le `README.md` mentionne `Qwen2.5-1.5B-Instruct`
- Le `rapport.pdf` mentionne `Qwen2.5-0.5B-Instruct`

**Conséquence** : Trois modèles différents sont référencés dans les fichiers, ce qui crée une incohérence totale et ne respecte pas le cahier des charges. Qwen2.5 et Qwen3 sont des familles de modèles différentes avec des architectures et des capacités distinctes.

**Correction** : Utilisation de `Qwen/Qwen3-0.6B` comme requis, avec le template de chat Qwen3 et `enable_thinking=False`.

---

## 2. Dataset GeoSQA non chargé (Critique)

**Problème** : Le code tente de charger GeoSQA avec `load_dataset('rfr2003/Geo_Benchmark', split='test')` **sans spécifier le config name** `'GeoSQA'`.

**Preuves** :
- Cellule 9 du notebook : le chargement échoue avec l'erreur `Config name is missing`
- Le code bascule vers **12 questions codées en dur** dans un bloc `except`
- La sortie de la cellule affiche clairement : `"Erreur GeoSQA : Config name is missing."`
- Les résultats portent donc sur 12 questions manuelles et non sur le vrai dataset GeoSQA

**Conséquence** : Toute l'évaluation est invalide. Les 12 questions hardcodées ne sont pas représentatives du vrai benchmark GeoSQA (4110 questions). Les métriques rapportées ne reflètent pas la performance réelle du système sur le dataset officiel.

**Correction** : `load_dataset('rfr2003/Geo_Benchmark', 'GeoSQA', split='test')` — avec le config name correct.

---

## 3. Fuite de données (Data Leakage) (Critique)

**Problème** : Le code contient un `CORPUS_GARANTI` de 9 articles Wikipedia fabriqués à la main qui contiennent **mot pour mot** les réponses aux questions hardcodées.

**Preuves** (exemples directs du code, cellule 10) :

| Question | Réponse attendue | Texte du corpus garanti |
|----------|-----------------|------------------------|
| "Quelle est la distance entre Paris et Lyon ?" | "Environ 465 km" | *"La distance entre Paris et Lyon est d'environ **465 kilomètres**."* |
| "Quelle est la population de Bordeaux ?" | "Environ 260 000 habitants" | *"La population de Bordeaux est d'environ **260 000 habitants**."* |
| "Dans quelle direction se trouve Marseille par rapport à Paris ?" | "Sud-Est" | *"Par rapport à Paris, Marseille se trouve dans la direction **sud-est**."* |
| "Dans quelle direction se trouve Strasbourg par rapport à Paris ?" | "Est" | *"Par rapport à Paris, Strasbourg se trouve dans la direction **est**."* |

**Conséquence** : Le retriever trouve forcément la bonne réponse car elle est littéralement dans le corpus. Cela invalide toute l'évaluation RAG : on ne mesure pas la capacité du système à trouver des informations pertinentes, mais simplement sa capacité à recopier une réponse préfabriquée.

**Correction** : Le corpus est chargé uniquement depuis Wikipedia (streaming HuggingFace), sans aucun article fabriqué.

---

## 4. Absence de métriques de retrieval (Important)

**Problème** : L'ancienne version n'évalue que la génération (Exact Match, F1, Contains) mais **aucune métrique de retrieval** n'est calculée.

**Pourquoi c'est un problème** : Dans un système RAG, il est essentiel d'évaluer séparément :
1. La qualité du **retriever** (est-ce qu'il trouve les bons documents ?)
2. La qualité du **générateur** (est-ce qu'il produit la bonne réponse ?)

Sans métriques de retrieval, on ne peut pas savoir si les erreurs viennent du retriever ou du générateur. Le rapport bibliographique mentionne pourtant les métriques Hit Rate@K, MRR et NDCG comme standards.

**Correction** : Ajout de Hit Rate@1, @3, @5 et MRR@5 dans la nouvelle version.

---

## 5. Format d'évaluation incorrect (Important)

**Problème** : GeoSQA est un dataset à **choix multiples** (QCM) avec 4 options (A, B, C, D). L'ancienne version traite les questions comme des QA ouvertes avec des réponses libres.

**Preuves** :
- Le vrai dataset GeoSQA a les colonnes : `question`, `scenario`, `A`, `B`, `C`, `D`, `answer` (lettre)
- L'ancienne version utilise des questions ouvertes ("Quelle est la capitale de la France ?") avec des réponses textuelles ("Paris")
- Ce ne sont même pas des questions du dataset GeoSQA

**Conséquence** : La métrique d'**accuracy** (pourcentage de bonnes réponses MCQ) — qui est la métrique naturelle pour un QCM — n'est pas calculée. L'évaluation utilise des métriques inadaptées au format du dataset.

**Correction** : Évaluation en mode QCM (accuracy par lettre) ET en mode ouvert (EM, F1) pour une analyse complète.

---

## 6. Artefacts de génération en anglais (Modéré)

**Problème** : Le modèle génère des textes en anglais au milieu de réponses en français.

**Preuves** (depuis `phase1_predictions.csv`) :

- Question "Dans quelle région se trouve Toulouse ?" → Prédiction RAG : *"InconnueHuman resources management is a crucial aspect of any organization's success and growth..."*

- Question "Quelle est la deuxième ville de France ?" → Prédiction RAG : *"MarseilleHuman resources management is a crucial aspect of any organization's success and growth..."*

**Cause** : Le modèle Qwen2.5-0.5B est trop petit pour gérer correctement le français et produit des hallucinations en anglais. L'extraction de la réponse (`extract_answer`) ne coupe pas suffisamment ces artefacts.

**Correction** : Utilisation de Qwen3-0.6B (meilleur multilingue) avec `enable_thinking=False` pour des réponses directes et concises.

---

## 7. Incohérences entre fichiers (Modéré)

**Problème** : Les différents fichiers de la Phase 1 se contredisent sur le modèle utilisé.

| Fichier | Modèle mentionné |
|---------|------------------|
| Code du notebook (cellule 13) | `Qwen/Qwen2.5-1.5B-Instruct` |
| `phase1_metrics.json` | `Qwen/Qwen2.5-0.5B-Instruct` |
| `README.md` | `Qwen/Qwen2.5-1.5B-Instruct` |
| `rapport.pdf` (titre) | `Qwen2.5-0.5B-Instruct` |

**Conséquence** : Impossible de savoir quel modèle a réellement été utilisé pour produire les résultats. La reproductibilité est compromise.

**Correction** : Un seul modèle (`Qwen/Qwen3-0.6B`) référencé de manière cohérente partout (CONFIG dict centralisé).

---

## Tableau récapitulatif

| # | Problème | Sévérité | Ancienne version | Version corrigée |
|---|----------|----------|-----------------|------------------|
| 1 | Mauvais modèle LLM | **Critique** | Qwen2.5-0.5B/1.5B (incohérent) | Qwen3-0.6B (conforme au cahier des charges) |
| 2 | GeoSQA non chargé | **Critique** | 12 questions codées en dur | Dataset GeoSQA complet (config name correct) |
| 3 | Fuite de données | **Critique** | Corpus garanti avec réponses | Wikipedia streaming, sans données fabriquées |
| 4 | Pas de métriques retrieval | **Important** | Aucune | Hit Rate@1/3/5, MRR@5 |
| 5 | Format d'évaluation | **Important** | QA ouvert (incorrect) | QCM (accuracy) + QA ouvert (EM, F1) |
| 6 | Artefacts anglais | **Modéré** | Texte anglais dans les réponses | Qwen3 + enable_thinking=False |
| 7 | Incohérences fichiers | **Modéré** | 3 modèles différents cités | CONFIG centralisé, cohérent partout |

---

## Conclusion

L'ancienne Phase 1 présente **3 problèmes critiques** qui invalidaient complètement les résultats :
1. Le dataset GeoSQA n'était pas réellement utilisé
2. Le corpus contenait directement les réponses (data leakage)
3. Le modèle ne correspondait pas au cahier des charges

La nouvelle version corrige tous ces problèmes et ajoute les métriques de retrieval manquantes, offrant une évaluation fiable et conforme aux exigences du projet.
