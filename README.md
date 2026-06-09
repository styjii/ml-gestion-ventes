# 🛒 Projet ML — Gestion des Ventes & Prévision des Sorties de Stock

> Projet de Machine Learning appliqué à la gestion commerciale d'une boutique.  
> Prédiction de la quantité vendue par produit via Régression Linéaire et KNN Regressor.

**Filière :** Informatique | **Module :** Machine Learning | **Année :** 2024–2025

---

## 📋 Table des matières

- [Contexte](#contexte)
- [Structure du dépôt](#structure-du-dépôt)
- [Données](#données)
- [Installation](#installation)
- [Pipeline — guide d'exécution](#pipeline--guide-dexécution)
- [Méthodologie](#méthodologie)
- [Résultats](#résultats)
- [Questions de réflexion](#questions-de-réflexion)
- [Livrables](#livrables)

---

## Contexte

Une boutique dispose d'un système de gestion de stock réparti en quatre tables Excel.  
L'objectif est de prédire les **quantités vendues** (`quantite_vendue`) par produit à partir des caractéristiques disponibles (prix, stock, catégorie, entrées, données temporelles).

**Résultat principal :** La Régression Linéaire est retenue comme meilleur modèle (MAE = 18.45, R² = 0.008), les faibles R² obtenus révélant que les features disponibles n'entretiennent pas de relation linéaire simple avec les ventes — conclusion en elle-même opérationnellement utile.

---

## Structure du dépôt

```
ml-gestion-ventes/
│
├── data/
│   ├── stock_boutique_simule.xlsx          # Source brute (4 feuilles)
│   ├── stock_boutique_nettoye.xlsx         # Tables nettoyées (output NB 01)
│   └── base_apprentissage_ml.csv           # Base fusionnée prête pour le ML (output NB 02)
│
├── notebooks/
│   ├── 01_nettoyage_stock_boutique.ipynb   # Nettoyage des 4 tables
│   ├── 02_fusion_tables_ml_boutique.ipynb  # Jointures et construction de la base
│   ├── 03_analyse_exploratoire.ipynb       # EDA, statistiques, visualisations
│   └── 04_preprocessing_et_modelisation.ipynb  # Feature eng., modèles, comparaison
│
├── rapport_synthese_ml.pdf                 # Rapport de synthèse (5 pages)
├── requirements.txt
└── README.md
```

---

## Données

### `stock_boutique_simule.xlsx` — Source brute (4 feuilles)

| Feuille | Lignes | Description |
|---------|--------|-------------|
| `Liste des marchandises` | 500 | Catalogue produits : référence, désignation, catégorie, prix unitaire, stock initial |
| `Entrees` | 800 | Mouvements d'approvisionnement : date, référence, quantité entrée |
| `Sorties` | 900 | Transactions de vente : date, référence, **quantité vendue** ← variable cible |
| `Inventaire` | 500 | Stock final et statut par produit |

Période couverte : **01/01/2025 → 20/03/2026**

Catégories présentes : `Smartphone`, `Electroménager`, `Groupe électrogène`, `Instrument de musique`

### `stock_boutique_nettoye.xlsx` — Tables nettoyées (output NB 01)

Résultat du notebook 01 : 0 valeur manquante, 0 doublon, noms de colonnes normalisés, dates converties en `datetime`, 35 stocks négatifs corrigés via `clip(lower=0)`.

### `base_apprentissage_ml.csv` — Base d'apprentissage (output NB 02)

**900 lignes × 9 colonnes** — résultat de la fusion des 4 tables par jointures successives.

| Colonne | Type | Rôle |
|---------|------|------|
| `date` | datetime | Date de la vente |
| `reference` | str | Identifiant produit |
| `designation` | str | Nom du produit |
| `prix_unitaire` | float | Prix en Ariary (MGA) |
| `categorie` | str | Catégorie du produit |
| `quantite_entree` | float | Total des approvisionnements (agrégé par produit) |
| `stock_actuel` | float | Stock final de l'inventaire |
| `seuil_d_alerte` | int | Seuil de réapprovisionnement |
| `quantite_vendue` | int | **Variable cible Y** |

---

## Installation

### Prérequis

- Python 3.9+
- pip

### Cloner le dépôt

```bash
git clone https://github.com/<votre-utilisateur>/ml-gestion-ventes.git
cd ml-gestion-ventes
```

### Installer les dépendances

```bash
pip install -r requirements.txt
```

### `requirements.txt`

```
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
seaborn>=0.12
scikit-learn>=1.3
openpyxl>=3.1
jupyter>=1.0
```

### Lancer JupyterLab

```bash
jupyter lab
```

---

## Pipeline — guide d'exécution

Les notebooks sont **numérotés et doivent être exécutés dans l'ordre**. Chacun produit un fichier consommé par le suivant.

```
stock_boutique_simule.xlsx
        │
        ▼
[NB 01] 01_nettoyage_stock_boutique.ipynb
        └──► stock_boutique_nettoye.xlsx
                │
                ▼
        [NB 02] 02_fusion_tables_ml_boutique.ipynb
                └──► base_apprentissage_ml.csv
                        │
                        ▼
                [NB 03] 03_analyse_exploratoire.ipynb
                        └──► graphiques EDA
                                │
                                ▼
                        [NB 04] 04_preprocessing_et_modelisation.ipynb
                                └──► résultats des modèles + comparaison
```

---

## Méthodologie

### Notebook 01 — Nettoyage

- Chargement avec `pd.read_excel(..., sheet_name=None)`
- Normalisation des noms de colonnes (minuscules, underscores)
- Conversion des colonnes date en `datetime`
- Correction de 40 statuts d'inventaire erronés
- Correction des stocks négatifs : `stock_actuel.clip(lower=0)` (35 produits concernés, jusqu'à −237 unités)
- **Export :** `stock_boutique_nettoye.xlsx`

### Notebook 02 — Fusion

Pipeline de jointures en 3 étapes :

1. Jointure gauche `Sorties ← Inventaire` sur `reference` → chaque vente reçoit les données de stock
2. Agrégation des `Entrees` par produit (`SUM quantite GROUP BY reference`) → évite la duplication des lignes de vente
3. Jointure gauche `Base ← Entrees agrégées` → `NaN` imputés à 0 pour les produits sans entrée
- **Export :** `base_apprentissage_ml.csv` (900 × 9)

### Notebook 03 — Analyse exploratoire (EDA)

- Statistiques descriptives (distributions, skewness, kurtosis)
- Variable cible quasi-uniforme : skewness = 0.066, kurtosis = −1.18, plage [1, 80]
- Matrice de corrélation de Pearson — corrélations max < 0.21 avec `quantite_vendue`
- Identification des produits et catégories les plus vendus
- Visualisations : histogrammes, heatmap, barplots par catégorie

### Notebook 04 — Préprocessing & Modélisation

**Feature Engineering** (4 nouvelles variables construites) :

| Feature | Formule | Interprétation |
|---------|---------|----------------|
| `ratio_stock_alerte` | `stock_actuel / (seuil_d_alerte + 1)` | < 1 : risque de rupture, > 1 : stock confortable |
| `mois` | `date.dt.month` | Saisonnalité mensuelle (1–12) |
| `trimestre` | `date.dt.quarter` | Saisonnalité trimestrielle (1–4) |
| `jour_semaine` | `date.dt.dayofweek` | Effet jour de semaine (0 = lundi) |

**Pipeline complet :**

| Étape | Opération | Détail |
|-------|-----------|--------|
| 1 | Correction anomalies | `stock_actuel.clip(lower=0)` |
| 2 | Feature Engineering | +4 variables temporelles et métier |
| 3 | One-Hot Encoding | `categorie` → 3 dummies (`drop_first=True`, réf. : Électroménager) |
| 4 | Split Train/Test | 80 % / 20 %, `random_state=42` |
| 5 | StandardScaler | `fit` sur train uniquement, `transform` sur train + test |
| 6 | Modélisation | Régression Linéaire + KNN (k = 1 à 20) |

---

## Résultats

### Statistiques descriptives de la base

| Variable | Min | Moyenne | Max | Écart-type |
|----------|-----|---------|-----|------------|
| `prix_unitaire` (Ar) | 60 893 | 1 218 797 | 4 231 089 | 935 030 |
| `quantite_entree` | 0 | 94.5 | 485 | 85.7 |
| `seuil_d_alerte` | 3 | 14.5 | 25 | 6.6 |
| `quantite_vendue` (Y) | 1 | 39.8 | 80 | 22.9 |

### Comparaison des modèles

| Modèle | MAE | RMSE | R² test | R² CV 5-fold |
|--------|-----|------|---------|--------------|
| **Régression Linéaire** | **18.45** | **22.04** | **0.0083** | **0.0431** |
| KNN k=7 (meilleur k) | 18.81 | 22.56 | −0.039 | −0.086 |
| KNN k=3 | ~21.7 | ~25+ | −0.38 | — |
| KNN k=15 | ~19.5 | ~23 | −0.06 | — |

### ✅ Modèle retenu : Régression Linéaire

La régression linéaire domine le KNN sur toutes les métriques. Le coefficient le plus influent est `stock_actuel` (−7.36 après standardisation), cohérent avec la corrélation négative observée en EDA (−0.20).

Les faibles R² ne sont pas un échec méthodologique : ils indiquent que les features disponibles (prix, stock, catégorie, entrées, date) ne permettent pas d'expliquer les variations de ventes via des modèles simples. La principale cause est l'**absence de données contextuelles** (promotions, comportement client, événements locaux).

---

## Questions de réflexion

**Q1. Pourquoi la normalisation est-elle particulièrement importante pour KNN ?**  
Le KNN calcule des distances euclidiennes. Sans normalisation, `prix_unitaire` (jusqu'à 4 200 000 Ar) écraserait complètement `mois` (1–12) ou les dummies (0–1). Le `StandardScaler` (μ=0, σ=1) remet toutes les features sur un pied d'égalité.

**Q2. Avantages et limites de la régression linéaire ?**  
✅ Interprétable, rapide, faible variance, coefficients directement actionnables.  
❌ Suppose une relation strictement linéaire (invalidée ici, R² < 0.02), ne capte pas les interactions ni les effets non-linéaires, sensible aux outliers et à la multicolinéarité.

**Q3. Dans quel cas le KNN est-il préférable ?**  
Quand la relation X→Y est non-linéaire et locale, que le dataset est de taille modérée, et que le nombre de features est limité (<10–15). Il souffre ici de la **malédiction de la dimensionnalité** (10 features après encodage rendent les distances euclidiennes peu informatives).

**Q4. Comment améliorer les prédictions ?**  
1. **Modèles non-linéaires** : Random Forest, XGBoost, LightGBM
2. **Enrichissement des données** : historique promotions, fichier client, événements locaux
3. **Reformulation** : transformer en problème de classification (faible / moyen / fort volume)
4. **Séries temporelles** : SARIMA ou Prophet pour exploiter la structure temporelle des ventes

---

## Livrables

- [x] `notebooks/01_nettoyage_stock_boutique.ipynb`
- [x] `notebooks/02_fusion_tables_ml_boutique.ipynb`
- [x] `notebooks/03_analyse_exploratoire.ipynb`
- [x] `notebooks/04_preprocessing_et_modelisation.ipynb`
- [x] `data/stock_boutique_nettoye.xlsx`
- [x] `data/base_apprentissage_ml.csv`
- [x] `rapport_synthese_ml.pdf`

---

## Auteurs

| Nom | Rôle |
|-----|------|
| *(à compléter)* | Développement & analyse |

---

## Licence

Projet académique — Filière Informatique, Module Machine Learning, 2024–2025.
