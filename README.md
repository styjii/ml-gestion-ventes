# 🛒 Projet ML — Gestion des Ventes & Prévision des Sorties de Stock

> Projet de Machine Learning appliqué à la gestion commerciale d'une boutique.  
> Prédiction des quantités vendues par produit à l'aide de modèles supervisés.

---

## 📋 Table des matières

- [Contexte](#contexte)
- [Structure du dépôt](#structure-du-dépôt)
- [Données](#données)
- [Installation](#installation)
- [Utilisation](#utilisation)
- [Méthodologie](#méthodologie)
- [Résultats](#résultats)
- [Questions de réflexion](#questions-de-réflexion)
- [Livrables](#livrables)

---

## Contexte

Une boutique dispose d'un système de gestion de stock enregistré dans un fichier Excel.  
L'objectif est d'exploiter ces données commerciales pour **prédire les ventes futures** (quantité vendue par produit) grâce à des algorithmes de Machine Learning supervisé.

Les catégories de produits présentes dans les données incluent :

| Catégorie | Exemples |
|-----------|----------|
| Smartphone | Smartphone Edge, Alpha, Nova, Ultra, Lite, Prime |
| Électroménager | Réfrigérateur, Climatiseur, Lave-linge, Congélateur… |
| Groupe électrogène | Groupes 2kVA, 5kVA, 6kVA, 8kVA, 12kVA |
| Instrument de musique | Piano numérique, Batterie, Guitare, Violon… |

---

## Structure du dépôt

```
ml-gestion-ventes/
│
├── data/
│   └── stock_boutique_simule.xlsx      # Fichier source (4 feuilles)
│
├── notebooks/
│   └── projet_ml_ventes.ipynb          # Notebook principal commenté
│
├── outputs/
│   ├── figures/                        # Graphiques exportés
│   │   ├── top_produits_vendus.png
│   │   ├── repartition_ventes_categorie.png
│   │   ├── correlation_heatmap.png
│   │   └── comparaison_modeles.png
│   └── rapport_ml_ventes.pdf           # Rapport final (3–5 pages)
│
├── requirements.txt
└── README.md
```

---

## Données

Le fichier `stock_boutique_simule.xlsx` est composé de **quatre feuilles** :

### Feuille 1 — `Liste des marchandises`
| Colonne | Description |
|---------|-------------|
| `Reference` | Identifiant unique du produit (ex. `SMP-0001`) |
| `Designation` | Nom du produit |
| `Categorie` | Catégorie (Smartphone, Électroménager, etc.) |
| `Seuil d'alerte` | Stock minimum avant réapprovisionnement |
| `Unite` | Unité de mesure (Pièce) |
| `Stock initial` | Quantité initiale en stock |
| `Prix Unitaire` | Prix en Ariary (MGA) |

### Feuille 2 — `Entrées`
| Colonne | Description |
|---------|-------------|
| `Date d'entrée` | Date de réception de la marchandise |
| `Reference produit` | Référence du produit réceptionné |
| `Quantité entrée` | Quantité reçue |

### Feuille 3 — `Sorties`
| Colonne | Description |
|---------|-------------|
| `Date de sortie` | Date de vente |
| `Reference produit` | Référence du produit vendu |
| `Quantité vendue` | **Variable cible Y** |

### Feuille 4 — `Inventaire`
| Colonne | Description |
|---------|-------------|
| `Reference produit` | Référence du produit |
| `Stock actuel` | Niveau de stock en temps réel |

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

### Contenu de `requirements.txt`

```
pandas
numpy
matplotlib
seaborn
scikit-learn
openpyxl
jupyter
```

### Lancer le notebook

```bash
jupyter lab notebooks/projet_ml_ventes.ipynb
```

---

## Utilisation

Ouvrir et exécuter les cellules du notebook `projet_ml_ventes.ipynb` dans l'ordre. Le notebook est organisé en quatre parties correspondant aux parties du projet.

---

## Méthodologie

### Partie 1 — Préparation des données

- Chargement des quatre feuilles Excel avec `pandas.read_excel()`
- Fusion des tables : `Sorties ← Liste des marchandises ← Entrées ← Inventaire`
- Définition des variables :
  - **Variable cible Y** : `Quantité vendue`
  - **Variables explicatives X** : Prix unitaire, Quantité entrée, Stock disponible, Catégorie (encodée)
- Traitement des valeurs manquantes (suppression ou imputation selon le contexte)
- Normalisation : `StandardScaler` (indispensable pour KNN) ou `MinMaxScaler`

### Partie 2 — Analyse exploratoire (EDA)

- Statistiques descriptives (`describe()`, distributions)
- Identification des produits les plus vendus (classement par quantité totale vendue)
- Visualisation de la répartition des ventes par catégorie
- Matrice de corrélation entre les variables numériques

### Partie 3 — Modélisation supervisée

Deux modèles de régression sont entraînés pour prédire `Y = Quantité vendue` :

#### Régression Linéaire

```python
from sklearn.linear_model import LinearRegression
model_lr = LinearRegression()
model_lr.fit(X_train, y_train)
```

#### KNN Regressor

```python
from sklearn.neighbors import KNeighborsRegressor
# Plusieurs valeurs de k testées : [1, 3, 5, 7, 10, 15, 20]
model_knn = KNeighborsRegressor(n_neighbors=k)
```

### Partie 4 — Comparaison des modèles

Les performances sont évaluées avec :

| Indicateur | Formule | Interprétation |
|------------|---------|----------------|
| **RMSE** | √(MSE) | Erreur quadratique moyenne — pénalise les grandes erreurs |
| **MAE** | mean(\|y - ŷ\|) | Erreur absolue moyenne — plus robuste aux outliers |
| **R²** | 1 - SS_res/SS_tot | Proportion de variance expliquée (1 = parfait) |

---

## Résultats

> *(À compléter après exécution du notebook)*

| Modèle | RMSE | MAE | R² |
|--------|------|-----|----|
| Régression Linéaire | — | — | — |
| KNN (k optimal) | — | — | — |

**Meilleur modèle** : *à déterminer selon les résultats obtenus*

---

## Questions de réflexion

**1. Pourquoi la normalisation est-elle particulièrement importante pour KNN ?**  
KNN repose sur le calcul de distances entre observations. Sans normalisation, les variables à grande échelle (ex. Prix unitaire en MGA) dominent la distance et biaisent les prédictions. La normalisation garantit que chaque variable contribue équitablement.

**2. Quels sont les avantages et les limites de la régression linéaire ?**  
- ✅ Simple, interprétable, rapide à entraîner, peu sensible aux données aberrantes
- ❌ Suppose une relation linéaire entre X et Y ; peu performante si la relation est complexe ou non-linéaire

**3. Dans quel cas un modèle KNN peut-il être préférable ?**  
KNN est préférable lorsque les relations entre variables sont non-linéaires et locales, et que le volume de données est modéré. Il n'impose aucune hypothèse sur la distribution des données.

**4. Comment pourrait-on améliorer davantage les prédictions ?**  
- Enrichir les features : saisonnalité (mois, jour de la semaine), historique de ventes glissant
- Tester d'autres modèles : Random Forest, Gradient Boosting (XGBoost, LightGBM)
- Optimiser les hyperparamètres avec `GridSearchCV`
- Croiser avec des données externes (événements locaux, promotions)

---

## Livrables

- [x] `notebooks/projet_ml_ventes.ipynb` — Notebook Python commenté
- [x] `outputs/figures/` — Graphiques produits
- [ ] `outputs/rapport_ml_ventes.pdf` — Rapport de 3 à 5 pages

---

## Auteurs

| Nom | Rôle |
|-----|------|
| *(à compléter)* | Développement & analyse |

---

## Licence

Ce projet est réalisé dans un cadre académique.
