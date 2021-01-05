---
title: API de détection des anomalies d’Azure Machine Learning - Team Data Science Process
description: L’API de détection des anomalies est un exemple d’API généré avec Microsoft Azure Machine Learning. Elle détecte des anomalies dans les données de séries chronologiques présentant des valeurs numériques qui sont réparties uniformément dans le temps.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=alokkirpal, previous-ms.author=alok
ms.openlocfilehash: e210c1683d5f14181bc0549e73a892eb91d2e746
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305697"
---
# <a name="machine-learning-anomaly-detection-api"></a>API de détection des anomalies Machine Learning

> [!NOTE]
> Cet élément est en cours de maintenance. Nous vous encourageons à utiliser le [service d’API de détecteur d’anomalies](https://azure.microsoft.com/services/cognitive-services/anomaly-detector/) alimenté par des algorithmes de Machine Learning sous Azure Cognitive Services pour détecter les anomalies à partir de métriques d’entreprise, opérationnelles et IoT.

## <a name="overview"></a>Vue d’ensemble
L’[API de détection des anomalies](https://gallery.azure.ai/MachineLearningAPI/Anomaly-Detection-2) est un exemple d’API généré avec Microsoft Azure Machine Learning. Elle détecte des anomalies dans les données de séries chronologiques présentant des valeurs numériques qui sont réparties uniformément dans le temps.

Cette API peut détecter les types suivants de schémas anormaux dans les données de séries chronologiques :

* **Tendances positives et négatives**  : par exemple, lors de la surveillance de l’utilisation de la mémoire d’un système informatique, une tendance à la hausse peut être digne d’intérêt, car elle est susceptible d’être le signe d’une fuite de mémoire.
* **Modifications dans la plage dynamique de valeurs**  : par exemple, lors de la surveillance des exceptions déclenchées par un service cloud, tout changement de la plage dynamique de valeurs peut indiquer une instabilité de l’intégrité du service.
* **Pics et creux**  : par exemple, quand vous surveillez le nombre d’échecs de connexion pour un service ou le nombre de validations dans un site de e-commerce, les pics ou les creux d’activité peuvent indiquer un comportement anormal.

Ces détecteurs Machine Learning effectuent le suivi de tels changements de valeurs au fil du temps, signalant les changements en continu au sein de leurs valeurs en tant que résultats d’anomalies. Ils ne nécessitent pas de réglage du seuil ad hoc et leurs notations peuvent être utilisées pour contrôler le taux de faux positifs. L’API de détection des anomalies est utile dans plusieurs scénarios, tels que la surveillance des services via le suivi des indicateurs de performance clés au fil du temps, la surveillance de l’utilisation via des mesures comme le nombre de recherches et le nombre de clics, la surveillance des performances via des compteurs comme la mémoire, les processeurs, le nombre de lectures de fichiers, etc. au fil du temps.

L’offre de détection des anomalies inclut des outils utiles pour bien commencer.

* [L’application web](https://anomalydetection-aml.azurewebsites.net/) vous aide à évaluer et à visualiser les résultats de l’exécution de l’API de détection des anomalies sur vos données.

> [!NOTE]
> Essayez la **solution IT Anomaly Insights** basée sur [cette API](https://gallery.azure.ai/MachineLearningAPI/Anomaly-Detection-2).
>
<!-- This Solution is no longer available
> To get this end to end solution deployed to your Azure subscription <a href="https://gallery.cortanaintelligence.com/Solution/Anomaly-Detection-Pre-Configured-Solution-1" target="_blank">**Start here >**</a>
-->

## <a name="api-deployment"></a>Déploiement de l’API
Pour pouvoir utiliser l’API, vous devez la déployer dans votre abonnement Azure où elle sera hébergée comme un service web Azure Machine Learning.  Vous pouvez effectuer cette opération dans la [galerie Azure AI](https://gallery.azure.ai/MachineLearningAPI/Anomaly-Detection-2).  Cette action déploie deux services web Azure Machine Learning Studio (classique) (et leurs ressources associées) dans votre abonnement Azure : un pour la détection d’anomalies avec détection de saisonnalité et un autre sans détection de saisonnalité.  Une fois le déploiement terminé, vous serez en mesure de gérer vos API à partir de la page des [services web Azure Machine Learning Studio (classique)](https://services.azureml.net/webservices/).  Sur cette page, vous pourrez trouver l’emplacement de vos points de terminaison, de vos clés API, ainsi que les exemples de code pour appeler l’API.  Des instructions plus détaillées sont disponibles [ici](../classic/manage-new-webservice.md).

## <a name="scaling-the-api"></a>Mise à l’échelle de l’API
Par défaut, votre déploiement est assorti d’un plan de facturation Dev/Test gratuit comprenant 1 000 transactions et 2 heures de calcul par mois.  Vous pouvez mettre à niveau votre plan en fonction de vos besoins.  Vous pouvez retrouver des informations sur la tarification des différents plans [ici](https://azure.microsoft.com/pricing/details/machine-learning/) sous « Tarification des API web de production ».

## <a name="managing-aml-plans"></a>Gestion des plans AML
Vous pouvez gérer votre plan de facturation [ici](https://services.azureml.net/plans/).  Le nom du plan doit être basé sur le nom du groupe de ressources que vous avez choisi lors du déploiement de l’API, plus une chaîne unique à votre abonnement.  Vous pouvez retrouver des instructions sur la mise à niveau de votre plan [ici](../classic/manage-new-webservice.md) dans la section « Gestion des plans de facturation ».

## <a name="api-definition"></a>Définition de l’API
Le service web fournit des API REST sur HTTPS qui peuvent être utilisées de différentes façons, notamment une application web ou mobile, R, Python, Excel, etc.  L’envoi des données de séries chronologiques à ce service se fait via un appel d’API REST ; celui-ci exécute une combinaison des trois types de détection d’anomalie décrits ci-dessous.

## <a name="calling-the-api"></a>Appel de l’API
Pour appeler l’API, vous devez connaître l’emplacement du point de terminaison et la clé d’API.  Ces deux exigences, ainsi que l’exemple de code pour l’appel de l’API, sont disponibles sur la page [Services web Azure Machine Learning Studio (classique)](https://services.azureml.net/webservices/).  Accédez à l’API souhaitée, puis cliquez sur l’onglet « Utiliser » pour les trouver.  Vous pouvez appeler l’API en tant qu’API Swagger (avec le paramètre d’URL `format=swagger`) ou API non-Swagger (sans le paramètre d’URL `format`).  L’exemple de code utilise le format Swagger.  Voici un exemple de demande et de réponse au format non-Swagger.  Ces exemples sont relatifs au point de terminaison de saisonnalité.  Le point de terminaison sans saisonnalité est similaire.

### <a name="sample-request-body"></a>Exemple de corps de la demande
La demande contient deux objets : `Inputs` et `GlobalParameters`.  Dans l’exemple de demande ci-dessous, certains paramètres sont envoyés de façon explicite tandis que d’autres ne le sont pas (faites défiler pour voir la liste complète des paramètres de chaque point de terminaison).  Les paramètres qui ne sont pas envoyés de façon explicite dans la demande utilisent les valeurs par défaut indiquées ci-dessous.

```json
{
            "Inputs": {
                    "input1": {
                            "ColumnNames": ["Time", "Data"],
                            "Values": [
                                    ["5/30/2010 18:07:00", "1"],
                                    ["5/30/2010 18:08:00", "1.4"],
                                    ["5/30/2010 18:09:00", "1.1"]
                            ]
                    }
            },
    "GlobalParameters": {
        "tspikedetector.sensitivity": "3",
        "zspikedetector.sensitivity": "3",
        "bileveldetector.sensitivity": "3.25",
        "detectors.spikesdips": "Both"
    }
}
```

### <a name="sample-response"></a>Exemple de réponse
Pour voir le champ `ColumnNames`, vous devez inclure `details=true` comme paramètre d’URL dans votre demande.  Consultez les tableaux ci-dessous pour connaître la signification de chacun de ces champs.

```json
{
    "Results": {
        "output1": {
            "type": "table",
            "value": {
                "Values": [
                    ["5/30/2010 6:07:00 PM", "1", "1", "0", "0", "-0.687952590518378", "0", "-0.687952590518378", "0", "-0.687952590518378", "0"],
                    ["5/30/2010 6:08:00 PM", "1.4", "1.4", "0", "0", "-1.07030497733224", "0", "-0.884548154298423", "0", "-1.07030497733224", "0"],
                    ["5/30/2010 6:09:00 PM", "1.1", "1.1", "0", "0", "-1.30229513613974", "0", "-1.173800281031", "0", "-1.30229513613974", "0"]
                ],
                "ColumnNames": ["Time", "OriginalData", "ProcessedData", "TSpike", "ZSpike", "BiLevelChangeScore", "BiLevelChangeAlert", "PosTrendScore", "PosTrendAlert", "NegTrendScore", "NegTrendAlert"],
                "ColumnTypes": ["DateTime", "Double", "Double", "Double", "Double", "Double", "Int32", "Double", "Int32", "Double", "Int32"]
            }
        }
    }
}
```


## <a name="score-api"></a>API Score
L’API Score est utilisée pour la détection des anomalies dans les données de séries chronologiques non saisonnières. L’API exécute un certain nombre de détecteurs d’anomalies sur les données et renvoie leurs résultats d’anomalies.
La figure ci-dessous illustre un exemple d’anomalies détectables par l’API Score. Cette série chronologique présente deux changements de niveau et trois pics. Les points rouges indiquent l’heure à laquelle le changement de niveau a été détecté, tandis que les points noirs signalent les pics détectés.
![API Score][1]

### <a name="detectors"></a>Détecteurs
L’API de détection d’anomalie prend en charge les détecteurs de trois grandes catégories. Le tableau suivant contient des informations détaillées sur les paramètres d’entrée spécifiques et les sorties de chaque détecteur.

| Catégorie de détecteurs | Détecteur | Description | Paramètres d’entrée | Outputs |
| --- | --- | --- | --- | --- |
| Détecteurs de pics |Détecteurs TSpike |Détecter des pics et des creux en fonction de l’écart des valeurs par rapport aux premier et troisième quartiles |*tspikedetector.sensitivity :* prend une valeur entière comprise entre 1 et 10, par défaut : 3 ; plus la valeur est élevée, moins la sensibilité est importante |TSpike : valeurs binaires (1 si un pic/creux est détecté, 0 dans le cas contraire) |
| Détecteurs de pics | Détecteur ZSpike |Détecter des pics et des creux en fonction de l’écart des points de données par rapport à leur moyenne |*zspikedetector.sensitivity :* prend une valeur entière comprise entre 1 et 10, par défaut : 3 ; plus la valeur est élevée, moins la sensibilité est importante |ZSpike : valeurs binaires (1 si un pic/creux est détecté, 0 dans le cas contraire) |
| Détecteur de tendances lentes |Détecteur de tendances lentes |Détecter les tendances positives lentes suivant la sensibilité définie |*trenddetector.sensitivity :* seuil sur le score du détecteur (par défaut : 3,25, 3,25 - 5 est une plage raisonnable pour opérer une sélection ; plus la valeur est élevée, plus la sensibilité est basse) |tscore : nombre flottant représentant le résultat d’anomalies pour une tendance |
| Détecteurs de changements de niveau | Détecteur de changements de niveau bidirectionnels |Détecter les changements de niveau vers le haut et vers le bas suivant la sensibilité définie |*bileveldetector.sensitivity :* seuil sur le score du détecteur (par défaut : 3,25, 3,25 - 5 est une plage raisonnable pour opérer une sélection ; plus la valeur est élevée, plus la sensibilité est basse) |rscore : nombre flottant représentant le résultat d’anomalies pour un changement de niveau vers le haut et vers le bas |

### <a name="parameters"></a>Paramètres
Le tableau ci-dessous contient des informations plus détaillées sur ces paramètres d’entrée :

| Paramètres d’entrée | Description | Paramètre par défaut | Type | Plage valide | Plage suggérée |
| --- | --- | --- | --- | --- | --- |
| detectors.historywindow |Historique (en nombre de points de données) utilisé pour le calcul du résultat d’anomalies |500 |entier |10-2000 |Dépend des séries chronologiques |
| detectors.spikesdips | Détection des pics uniquement, des creux uniquement ou des deux |Les deux |enumerated |Les deux, pics, creux |Les deux |
| bileveldetector.sensitivity |Sensibilité du détecteur de changements de niveau bidirectionnels. |3.25 |double |None |3.25-5 (plus la valeur est basse, plus la sensibilité est importante) |
| trenddetector.sensitivity |Sensibilité du détecteur de tendances positives |3.25 |double |None |3.25-5 (plus la valeur est basse, plus la sensibilité est importante) |
| tspikedetector.sensitivity |Sensibilité du détecteur TSpike |3 |entier |1-10 |3-5 (plus la valeur est basse, plus la sensibilité est importante) |
| zspikedetector.sensitivity |Sensibilité du détecteur ZSpike |3 |entier |1-10 |3-5 (plus la valeur est basse, plus la sensibilité est importante) |
| postprocess.tailRows |Nombre de points de données récents à conserver dans les résultats de sortie |0 |entier |0 (conserver tous les points de données) ou spécifier le nombre de points à conserver dans les résultats |N/A |

### <a name="output"></a>Output
L’API exécute tous les détecteurs sur vos données de séries chronologiques et renvoie les résultats d’anomalies et les indicateurs de pics binaires pour chaque point dans le temps. Le tableau ci-dessous répertorie les sorties de l’API.

| Outputs | Description |
| --- | --- |
| Temps |Horodatages issus des données brutes ou des données agrégées (et/ou) imputées si l’imputation des données agrégées (et/ou) manquantes est appliquée. |
| Données |Valeurs issues des données brutes ou des données agrégées (et/ou) imputées si l’imputation des données agrégées (et/ou) manquantes est appliquée. |
| TSpike |Indicateur binaire signalant si un pic est détecté par le détecteur TSpike |
| ZSpike |Indicateur binaire signalant si un pic est détecté par le détecteur ZSpike |
| rpscore |Nombre flottant représentant le résultat d’anomalies pour un changement de niveau bidirectionnel |
| rpalert |Valeur de 1/0 indiquant s’il y a une anomalie de changement de niveau bidirectionnel vers le haut en fonction de la sensibilité d’entrée |
| tscore |Nombre flottant représentant le résultat d’anomalies pour une tendance positive |
| talert |Valeur de 1/0 indiquant s’il y a une anomalie de tendance positive en fonction de la sensibilité d’entrée |

## <a name="scorewithseasonality-api"></a>API ScoreWithSeasonality
L’API ScoreWithSeasonality est utilisée pour la détection des anomalies dans les séries chronologiques qui présentent des schémas saisonniers. Cette API est utile pour détecter les écarts dans les schémas saisonniers.
La figure suivante illustre un exemple d’anomalies détectées dans une série chronologique saisonnière. Cette série chronologique présente un pic (le 1er point noir), deux creux (le 2e point noir et un autre à la fin) et un changement de niveau (le point rouge). Tant le creux au milieu de la série chronologique que le changement de niveau ne sont visibles qu’une fois les composantes saisonnières supprimées de la série.
![API Saisonnalité][2]

### <a name="detectors"></a>Détecteurs
Les détecteurs du point de terminaison du caractère saisonnier sont similaires à ceux du point de terminaison sans saisonnalité, mais avec des noms de paramètres légèrement différents (répertoriés ci-après).

### <a name="parameters"></a>Paramètres

Le tableau ci-dessous contient des informations plus détaillées sur ces paramètres d’entrée :

| Paramètres d’entrée | Description | Paramètre par défaut | Type | Plage valide | Plage suggérée |
| --- | --- | --- | --- | --- | --- |
| preprocess.aggregationInterval |Intervalle d’agrégation en secondes pour l’agrégation de séries chronologiques d’entrée |0 (aucune agrégation n’est effectuée) |entier |0 : ignorer l’agrégation, > 0 autrement |5 minutes à 1 jour, dépend des séries chronologiques |
| preprocess.aggregationFunc |Fonction utilisée pour agréger les données dans l’intervalle d’agrégation spécifié |mean |enumerated |mean, sum, length |N/A |
| preprocess.replaceMissing |Valeurs utilisées pour imputer les données manquantes |lkv (dernière valeur connue) |enumerated |zero, lkv, mean |N/A |
| detectors.historywindow |Historique (en nombre de points de données) utilisé pour le calcul du résultat d’anomalies |500 |entier |10-2000 |Dépend des séries chronologiques |
| detectors.spikesdips | Détection des pics uniquement, des creux uniquement ou des deux |Les deux |enumerated |Les deux, pics, creux |Les deux |
| bileveldetector.sensitivity |Sensibilité du détecteur de changements de niveau bidirectionnels. |3.25 |double |None |3.25-5 (plus la valeur est basse, plus la sensibilité est importante) |
| postrenddetector.sensitivity |Sensibilité du détecteur de tendances positives |3.25 |double |None |3.25-5 (plus la valeur est basse, plus la sensibilité est importante) |
| negtrenddetector.sensitivity |Sensibilité du détecteur de tendances négatives. |3.25 |double |None |3.25-5 (plus la valeur est basse, plus la sensibilité est importante) |
| tspikedetector.sensitivity |Sensibilité du détecteur TSpike |3 |entier |1-10 |3-5 (plus la valeur est basse, plus la sensibilité est importante) |
| zspikedetector.sensitivity |Sensibilité du détecteur ZSpike |3 |entier |1-10 |3-5 (plus la valeur est basse, plus la sensibilité est importante) |
| seasonality.enable |Exécution de l’analyse de saisonnalité |true |boolean |true, false |Dépend des séries chronologiques |
| seasonality.numSeasonality |Nombre maximal de cycles périodiques à détecter |1 |entier |1, 2 |1-2 |
| seasonality.transform |Suppression des composantes de tendances (et) saisonnières avant l’exécution de la détection des anomalies |deseason |enumerated |none, deseason, deseasontrend |N/A |
| postprocess.tailRows |Nombre de points de données récents à conserver dans les résultats de sortie |0 |entier |0 (conserver tous les points de données) ou spécifier le nombre de points à conserver dans les résultats |N/A |

### <a name="output"></a>Output
L’API exécute tous les détecteurs sur vos données de séries chronologiques et renvoie les résultats d’anomalies et les indicateurs de pics binaires pour chaque point dans le temps. Le tableau ci-dessous répertorie les sorties de l’API.

| Outputs | Description |
| --- | --- |
| Temps |Horodatages issus des données brutes ou des données agrégées (et/ou) imputées si l’imputation des données agrégées (et/ou) manquantes est appliquée. |
| OriginalData |Valeurs issues des données brutes ou des données agrégées (et/ou) imputées si l’imputation des données agrégées (et/ou) manquantes est appliquée. |
| ProcessedData |L’une des options suivantes : <ul><li>série chronologique ajustée de façon saisonnière si un caractère saisonnier important a été détectée et si l’option deseason est sélectionnée ;</li><li>série chronologique redressée et ajustée de façon saisonnière si un caractère saisonnier important a été détectée et si l’option deseasontrend est sélectionnée ;</li><li>autrement, cette option est identique à OriginalData.</li> |
| TSpike |Indicateur binaire signalant si un pic est détecté par le détecteur TSpike |
| ZSpike |Indicateur binaire signalant si un pic est détecté par le détecteur ZSpike |
| BiLevelChangeScore |Nombre flottant représentant le résultat d’anomalies pour un changement de niveau |
| BiLevelChangeAlert |Valeur 1/0 indiquant s’il y a une anomalie de changement de niveau en fonction de la sensibilité d’entrée |
| PosTrendScore |Nombre flottant représentant le résultat d’anomalies pour une tendance positive |
| PosTrendAlert |Valeur de 1/0 indiquant s’il y a une anomalie de tendance positive en fonction de la sensibilité d’entrée |
| NegTrendScore |Nombre flottant représentant le résultat d’anomalies pour une tendance négative |
| NegTrendAlert |Valeur de 1/0 indiquant s’il y a une anomalie de tendance négative en fonction de la sensibilité d’entrée |

[1]: ./media/apps-anomaly-detection-api/anomaly-detection-score.png
[2]: ./media/apps-anomaly-detection-api/anomaly-detection-seasonal.png