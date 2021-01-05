---
title: Comment préserver la confidentialité des données à l’aide des packages WhiteNoise (préversion)
titleSuffix: Azure Machine Learning
description: Découvrez comment appliquer les meilleures pratiques en matière de confidentialité différentielle aux modèles Azure Machine Learning à l’aide des packages WhiteNoise.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: slbird
author: slbird
ms.reviewer: luquinta
ms.date: 07/09/2020
ms.openlocfilehash: 355d96fe5a617effab89fbd038f7f1785215f88f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90897694"
---
# <a name="use-differential-privacy-in-azure-machine-learning-preview"></a>Utiliser la confidentialité différentielle dans Azure Machine Learning (préversion)



Découvrez comment appliquer les meilleures pratiques en matière de confidentialité différentielle aux modèles Azure Machine Learning à l’aide des packages Python WhiteNoise.

La confidentialité différentielle est la définition de référence de la confidentialité. Les systèmes qui adhèrent à cette définition de la confidentialité fournissent de solides garanties contre un large éventail d’attaques de reconstruction et de réidentification des données, notamment les attaques par des adversaires qui possèdent des informations auxiliaires. En savoir plus sur le [fonctionnement de la confidentialité différentielle](./concept-differential-privacy.md).

> [!NOTE]
> Veuillez noter que nous allons renommer le kit de ressources et présenterons le nouveau nom dans les semaines à venir. 

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte gratuit avant de commencer. Essayez la [version gratuite ou payante d’Azure Machine Learning](https://aka.ms/AMLFree) dès aujourd’hui.
- [Python 3](https://www.python.org/downloads/)

## <a name="install-whitenoise-packages"></a>Installer les packages WhiteNoise

### <a name="standalone-installation"></a>Installation autonome

Les bibliothèques sont conçues pour fonctionner à partir de clusters Spark distribués et peuvent être installées comme n’importe quel autre package.

Les instructions ci-dessous supposent que vos commandes `python` et `pip` sont mappées à `python3` et `pip3`.

Utilisez pip pour installer les [packages Python WhiteNoise](https://pypi.org/project/opendp-whitenoise/).

`pip install opendp-whitenoise`

Pour vérifier que les packages sont installés, lancez une invite Python et saisissez :

```python
import opendp.whitenoise.core
import opendp.whitenoise.sql
```

Si les importations ont été effectuées correctement, les bibliothèques sont installées et prêtes à l’emploi.

### <a name="docker-image"></a>Image Docker

Vous pouvez également utiliser des packages WhiteNoise avec Docker.

Extrayez l’image `opendp/whitenoise` pour utiliser les bibliothèques à l’intérieur d’un conteneur Docker qui comprend Spark, Jupyter et un exemple de code.

```sh
docker pull opendp/whitenoise:privacy
```

Une fois que vous avez extrait l’image, lancez le serveur Jupyter :

```sh
docker run --rm -p 8989:8989 --name whitenoise-run opendp/whitenoise:privacy
```

Cela démarre un serveur Jupyter sur le port `8989` de votre `localhost`, avec le mot de passe `pass@word99`. En supposant que vous avez utilisé la ligne de commande ci-dessus pour démarrer le conteneur portant le nom `whitenoise-privacy`, vous pouvez ouvrir un terminal bash dans le serveur Jupyter en exécutant :

```sh
docker exec -it whitenoise-run bash
```

L’instance Docker efface tous les états à l’arrêt. Vous perdez donc tous les notebooks que vous créez dans l’instance en cours d’exécution. Pour y remédier, vous pouvez lier le montage d’un dossier local au conteneur lors de son lancement :

```sh
docker run --rm -p 8989:8989 --name whitenoise-run --mount type=bind,source=/Users/your_name/my-notebooks,target=/home/privacy/my-notebooks opendp/whitenoise:privacy
```

Tous les notebooks que vous créez dans le dossier *my-notebooks* sont stockés dans votre système de fichiers local.

## <a name="perform-data-analysis"></a>Effectuer une analyse des données

Pour préparer une version privée différentielle, vous devez choisir une source de données, une statistique et certains paramètres de confidentialité, en indiquant le niveau de protection de la confidentialité.  

Cet exemple fait référence aux microdonnées à usage public de Californie (PUMS), qui représentent les enregistrements anonymes des données démographiques des citoyens :

```python
import os
import sys
import numpy as np
import opendp.whitenoise.core as wn

data_path = os.path.join('.', 'data', 'PUMS_california_demographics_1000', 'data.csv')
var_names = ["age", "sex", "educ", "race", "income", "married", "pid"]
```

Dans cet exemple, nous calculons la moyenne et la variance de l’âge.  Nous utilisons une `epsilon` totale de 1 (epsilon est notre paramètre de confidentialité, en répartissant notre budget dédié à la confidentialité sur les deux quantités que nous voulons calculer). En savoir plus sur les [métriques de confidentialité](concept-differential-privacy.md#differential-privacy-metrics).

```python
with wn.Analysis() as analysis:
    # load data
    data = wn.Dataset(path = data_path, column_names = var_names)

    # get mean of age
    age_mean = wn.dp_mean(data = wn.cast(data['age'], type="FLOAT"),
                          privacy_usage = {'epsilon': .65},
                          data_lower = 0.,
                          data_upper = 100.,
                          data_n = 1000
                         )
    # get variance of age
    age_var = wn.dp_variance(data = wn.cast(data['age'], type="FLOAT"),
                             privacy_usage = {'epsilon': .35},
                             data_lower = 0.,
                             data_upper = 100.,
                             data_n = 1000
                            )
analysis.release()

print("DP mean of age: {0}".format(age_mean.value))
print("DP variance of age: {0}".format(age_var.value))
print("Privacy usage: {0}".format(analysis.privacy_usage))
```

Les résultats ressemblent à ceci :

```text
DP mean of age: 44.55598845931517
DP variance of age: 231.79044646429134
Privacy usage: approximate {
  epsilon: 1.0
}
```

Quelques points importants sont à noter dans cet exemple.  Tout d’abord, l’objet `Analysis` représente un graphique de traitement des données.  Dans cet exemple, la moyenne et la variance sont calculées à partir du même nœud source.  Toutefois, vous pouvez inclure des expressions plus complexes qui combinent des entrées et des sorties de manière arbitraire.

Le graphique d’analyse comprend les métadonnées `data_upper` et `data_lower`, spécifiant les limites inférieure et supérieure pour les âges.  Ces valeurs sont utilisées pour étalonner précisément le bruit afin de garantir la confidentialité différentielle.  Ces valeurs sont également utilisées pour le traitement de valeurs aberrantes ou manquantes.

Enfin, le graphique d’analyse permet de suivre le budget total consacré à la confidentialité.

Vous pouvez utiliser la bibliothèque pour composer des graphiques d’analyse plus complexes, avec plusieurs mécanismes, statistiques et fonctions utilitaires :

| Statistiques    | Mécanismes | Services  |
| ------------- |------------|------------|
| Count         | Gaussienne   | Caster       |
| Histogramme     | Géométrique  | Limiter   |
| Moyenne          | Laplace    | Numériser   |
| Quantiles     |            | Filtrer     |
| SUM           |            | Imputer |
| Variance/covariance |      | Transformer  |

Pour plus d’informations, consultez le [notebook relatif à l’analyse des données](https://github.com/opendifferentialprivacy/whitenoise-samples/blob/master/analysis/basic_data_analysis.ipynb).

## <a name="approximate-utility-of-differentially-private-releases"></a>Utilitaire approximatif de versions privées différentielles

Comme la confidentialité différentielle fonctionne par calibrage du bruit, l’utilitaire des versions peut varier en fonction du risque pour la confidentialité.  En règle générale, le bruit nécessaire à la protection de chaque individu devient négligeable à mesure que la taille des échantillons augmente, mais le résultat submerge les versions qui ciblent une seule personne.  Les analystes peuvent examiner les informations relatives à l’exactitude d’une version pour déterminer l’utilité de cette dernière :

```python
with wn.Analysis() as analysis:
    # load data
    data = wn.Dataset(path = data_path, column_names = var_names)

    # get mean of age
    age_mean = wn.dp_mean(data = wn.cast(data['age'], type="FLOAT"),
                          privacy_usage = {'epsilon': .65},
                          data_lower = 0.,
                          data_upper = 100.,
                          data_n = 1000
                         )
analysis.release()

print("Age accuracy is: {0}".format(age_mean.get_accuracy(0.05)))
```

Le résultat de cette opération doit se présenter comme suit :

```text
Age accuracy is: 0.2995732273553991
```

Cet exemple calcule la moyenne comme indiqué ci-dessus et utilise la fonction `get_accuracy` pour demander une exactitude à `alpha` de 0,05. Une `alpha` de 0,05 représente un intervalle de 95 %, c’est-à-dire que la valeur publiée sera comprise entre les limites d’exactitude rapportées environ 95 % du temps.  Dans cet exemple, l’exactitude rapportée est de 0,3, ce qui signifie que la valeur publiée sera comprise dans un intervalle de largeur de 0,6, environ 95 % du temps.  Il n’est pas correct de considérer cette valeur comme une barre d’erreur, car la valeur publiée se situera en dehors de la plage d’exactitude indiquée au taux spécifié par `alpha`, et les valeurs en dehors de la plage peuvent l’être dans l’une ou l’autre direction.

Les analystes peuvent interroger `get_accuracy` pour connaître les différentes valeurs de `alpha` afin d’obtenir des intervalles de confiance plus étroits ou plus larges, sans occasionner de coûts supplémentaires pour la confidentialité.

## <a name="generate-a-histogram"></a>Générer un histogramme

La fonction `dp_histogram` intégrée crée des histogrammes privés de manière différentielle sur l’un des types de données suivants :

- Une variable continue, où l’ensemble des nombres doit être divisé en compartiments
- Une variable booléenne ou dichotomique, qui ne peut prendre que deux valeurs
- Une variable catégorique, où des catégories distinctes sont énumérées en tant que chaînes

Voici un exemple d’une `Analysis` spécifiant des compartiments pour un histogramme de variables continues :

```python
income_edges = list(range(0, 100000, 10000))

with wn.Analysis() as analysis:
    data = wn.Dataset(path = data_path, column_names = var_names)

    income_histogram = wn.dp_histogram(
            wn.cast(data['income'], type='int', lower=0, upper=100),
            edges = income_edges,
            upper = 1000,
            null_value = 150,
            privacy_usage = {'epsilon': 0.5}
        )
```

Comme les individus sont répartis de manière disjointe entre les compartiments de l’histogramme, le coût de la confidentialité n’est encouru qu’une seule fois par histogramme, même si l’histogramme comprend de nombreux compartiments.

Pour plus d’informations sur les histogrammes, consultez le [notebook relatif aux histogrammes](https://github.com/opendifferentialprivacy/whitenoise-samples/blob/master/analysis/histograms.ipynb).

## <a name="generate-a-covariance-matrix"></a>Générer une matrice de covariance

WhiteNoise propose trois fonctionnalités différentes avec sa fonction `dp_covariance` :

- Covariance entre deux vecteurs
- Matrice de covariance d’une matrice
- Matrice de covariance croisée d’une paire de matrices

Voici un exemple de calcul d’une covariance scalaire :

```python
with wn.Analysis() as analysis:
    wn_data = wn.Dataset(path = data_path, column_names = var_names)

    age_income_cov_scalar = wn.dp_covariance(
      left = wn.cast(wn_data['age'], 
      type = "FLOAT"), 
      right = wn.cast(wn_data['income'], 
      type = "FLOAT"), 
      privacy_usage = {'epsilon': 1.0},
      left_lower = 0., 
      left_upper = 100., 
      left_n = 1000, 
      right_lower = 0., 
      right_upper = 500_000.,
      right_n = 1000)
```

Pour plus d’informations, consultez le [notebook relatif à la covariance](
https://github.com/opendifferentialprivacy/whitenoise-samples/blob/master/analysis/covariance.ipynb).

## <a name="next-steps"></a>Étapes suivantes

- Explorez des [exemples de notebooks WhiteNoise](https://github.com/opendifferentialprivacy/whitenoise-samples/tree/master/analysis).