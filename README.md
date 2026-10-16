# Appliquer le weighted voting de Golub au dataset WDBC

Projet individuel : application de la méthode de weighted voting
(Golub et al., 1999) au dataset Wisconsin Diagnostic Breast Cancer (WDBC),
avec comparaison à une régression logistique.

## Contenu du dépôt

| Fichier / dossier | Rôle |
|---|---|
| `script.ipynb` | Pipeline complet : vérification des données, weighted voting, validation croisée, stabilité de la sélection, régression logistique, figures |
| `data/wdbc.csv` | Données WDBC figées (569 patientes, 30 variables), non modifiées |
| `data/MANIFEST.json` | Provenance et effectifs des classes |
| `figures/` | Figures générées par le notebook |
| `pyproject.toml`, `uv.lock`, `.python-version` | Environnement reproductible géré avec uv |

## Prérequis

- [uv] installé
- Aucune autre installation n'est nécessaire : uv télécharge la bonne version de Python et les dépendances.

## Exécuter tout le pipeline (une seule commande)

```bash
uv run jupyter nbconvert --to notebook --execute script.ipynb --output script_executed.ipynb
```

Cette commande :
1. crée l'environnement à partir de `uv.lock` (versions exactes) ;
2. exécute le notebook de bout en bout (tableaux, métriques, stabilité, comparaison) ;
3. génère les figures dans `figures/` ;
4. écrit le notebook exécuté, avec toutes les sorties, dans `script_executed.ipynb`.

Les résultats sont déterministes : la graine est fixée dans le notebook
(`SEED = 26`) et utilisée à l'identique pour le weighted voting et la
régression logistique, donc les deux méthodes sont évaluées sur les 15 mêmes découpages.

## Ce que fait le pipeline

1. **Vérification des données** : dimensions, doublons, valeurs manquantes, effectifs des classes (357 bénignes, 212 malignes).
2. **Weighted voting (Golub)** : poids `w_j = (μ_j,malin − μ_j,bénin) / (σ_j,malin + σ_j,bénin)`, sélection des 10 variables avec |w_j| maximal, vote `V(x)`, prédiction « malin » si `V(x) ≥ 0`. Toutes les statistiques sont estimées sur le pli d'entraînement uniquement.
3. **Évaluation** : validation croisée stratifiée 5 plis × 3 répétitions (`RepeatedStratifiedKFold`), balanced accuracy, AUC, matrice de confusion d'un modèle exemple, moyennes sur les 15 modèles.
4. **Stabilité de la sélection** : fréquence de sélection de chaque variable, similarité de Jaccard moyenne entre les 15 ensembles.
5. **Comparaison** : régression logistique (`liblinear`, `C=1.0`, `max_iter=1000`), 30 variables standardisées dans chaque pli d'entraînement, seuil 0,5, mêmes 15 découpages.

## Ordre d'exécution

Les cellules du notebook dépendent les unes des autres et doivent être
exécutées dans l'ordre (c'est ce que fait la commande ci-dessus).

## Reda LAHLOU KASSI