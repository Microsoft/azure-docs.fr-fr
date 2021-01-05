---
title: Présentation de l’API Détecteur d’anomalies
titleSuffix: Azure Cognitive Services
description: Utilisez les algorithmes de l’API Détecteur d’anomalies pour appliquer la détection d’anomalies à vos données de séries chronologiques.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: overview
ms.date: 11/23/2020
ms.author: mbullwin
keywords: détection d’anomalie, Machine Learning, algorithmes
ms.custom: cog-serv-seo-aug-2020
ms.openlocfilehash: 1a13b962a000e6e3b2143dc23f94b63c122bd963
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95999096"
---
# <a name="what-is-the-anomaly-detector-api"></a>Présentation de l’API Détecteur d’anomalies

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

L’API Détecteur d’anomalies vous permet de superviser et de détecter des anomalies dans vos données de séries chronologiques sans avoir à connaître le machine learning. Les algorithmes de l’API Détecteur d’anomalies s’adaptent en identifiant et en appliquant automatiquement les modèles les mieux adaptés à vos données, indépendamment du secteur d’activité, du scénario ou du volume de données. À l’aide de vos données de série chronologique, l’API détermine les limites pour la détection des anomalies, les valeurs attendues et les points de données qui constituent des anomalies.

![Détecter des modifications de modèle dans les demandes de service](./media/anomaly_detection2.png)

L’utilisation du détecteur d'anomalies ne nécessite aucune expérience préalable de l’apprentissage automatique et l’API RESTful permet d’intégrer facilement le service dans vos applications et processus.

## <a name="features"></a>Fonctionnalités

Grâce au détecteur d’anomalies, vous pouvez automatiquement détecter des anomalies, dans l’ensemble de vos données de série chronologique ou en temps réel.

|Fonctionnalité  |Description  |
|---------|---------|
|Détection d’anomalie en temps réel. | Détectez les anomalies dans vos données de diffusion en continu à l’aide des points de données préalablement vus pour déterminer si le dernier point est une anomalie. Cette opération génère un modèle à l’aide des points de données que vous envoyez et détermine si le point cible est une anomalie. En appelant l’API avec chaque nouveau point de données que vous générez, vous pouvez surveiller vos données au moment de leur création. |
|Détecter les anomalies tout au long de votre jeu de données par lots. | Utilisez votre série chronologique pour détecter d’éventuelles anomalies dans l’ensemble de vos données. Cette opération génère un modèle à l’aide de vos données de série chronologique complètes, chaque point étant analysé avec le même modèle.         |
|Détecter les points de changement tout au long de votre jeu de données dans un même lot. | Utilisez votre série chronologique pour détecter les points de changement de tendance présents dans vos données. Cette opération génère un modèle à l’aide de vos données de série chronologique complètes, chaque point étant analysé avec le même modèle.    |
| Obtenir des informations supplémentaires sur vos données. | Obtenez des détails utiles sur vos données et sur les anomalies constatées, notamment les valeurs attendues ainsi que les limites et les positions des anomalies. |
| Ajuster les limites de détection des anomalies. | L’API Détecteur d’anomalies crée automatiquement des limites pour la détection des anomalies. Ajustez ces limites pour augmenter ou diminuer la sensibilité de l’API aux anomalies de données et mieux l’adapter à vos données. |

## <a name="demo"></a>Démonstration

Consultez cette [démonstration interactive](https://aka.ms/adDemo) pour comprendre le fonctionnement du détecteur d’anomalies.
Pour exécuter la démonstration, vous devez créer une ressource Détecteur d’anomalies et récupérer la clé API ainsi que le point de terminaison.

## <a name="notebook"></a>Notebook

Pour savoir comment appeler l’API Détecteur d’anomalies, essayez ce [notebook](https://aka.ms/adNotebook). Ce notebook Jupyter montre comment envoyer une demande d’API et visualiser le résultat.

Pour exécuter le notebook, procédez comme suit :

1. Obtenez une clé d’abonnement valide pour l’API Détecteur d’anomalies et un point de terminaison d’API. La section ci-dessous donne les instructions pour s’inscrire.
1. Connectez-vous, puis cliquez sur Cloner en haut à droite.
1. Désactivez l’option « public » dans la boîte de dialogue avant de procéder à l’opération de clonage. Dans le cas contraire, votre notebook, avec toutes les clés d’abonnement, sera public.
1. Cliquez sur **Exécuter sur le calcul gratuit**
1. Sélectionnez l’un des notebooks.
1. Ajoutez votre clé d’abonnement à l’API Détecteur d'anomalies valide à la variable `subscription_key`.
1. Modifiez la variable `endpoint` à votre point de terminaison. Par exemple : `https://westus2.api.cognitive.microsoft.com/anomalydetector/v1.0/timeseries/last/detect`
1. Dans la barre de menus supérieure, cliquez sur **Cellule**, puis sur **Exécuter tout**.

## <a name="workflow"></a>Workflow

L’API Détecteur d'anomalies étant un service web RESTful, elle peut être facilement appelée à partir de n’importe quel langage de programmation capable d’exécuter des requêtes HTTP et d’analyser des réponses JSON.

[!INCLUDE [cognitive-services-anomaly-detector-data-requirements](../../../includes/cognitive-services-anomaly-detector-data-requirements.md)]

[!INCLUDE [cognitive-services-anomaly-detector-signup-requirements](../../../includes/cognitive-services-anomaly-detector-signup-requirements.md)]

Après l’inscription :

1. Prenez vos données de série chronologique et convertissez-les en format JSON valide. Utilisez les [meilleures pratiques](concepts/anomaly-detection-best-practices.md) lors de la préparation de vos données pour obtenir les meilleurs résultats.
1. Envoyez une demande à l’API Détecteur d’anomalies avec vos données.
1. Traitez la réponse de l’API en analysant le message JSON renvoyé.

## <a name="algorithms"></a>Algorithmes

* Pour plus d’informations sur les algorithmes utilisés, consultez les blogs techniques suivants :
    * [Présentation de l’API Détecteur d’anomalies](https://techcommunity.microsoft.com/t5/AI-Customer-Engineering-Team/Introducing-Azure-Anomaly-Detector-API/ba-p/490162)
    * [Présentation de l’algorithme SR-CNN dans le Détecteur d’anomalies Azure](https://techcommunity.microsoft.com/t5/AI-Customer-Engineering-Team/Overview-of-SR-CNN-algorithm-in-Azure-Anomaly-Detector/ba-p/982798)

Pour plus d’informations sur les algorithmes SR-CNN de pointe développés par Microsoft, lisez le document [Time-Series Anomaly Detection Service at Microsoft](https://arxiv.org/abs/1906.03821) (accepté par KDD 2019) .


> [!VIDEO https://www.youtube.com/embed/ERTaAnwCarM]

## <a name="deploy-on-premises-using-docker-containers"></a>Déployer localement en utilisant des conteneurs Docker

[Utilisez les conteneurs Détecteur d'anomalies](anomaly-detector-container-howto.md) pour déployer localement des fonctionnalités d’API. Les conteneurs Docker vous donnent la possibilité de rapprocher davantage le service de vos données, ce qui peut être souhaitable pour des raisons de conformité, de sécurité ou opérationnelles.

## <a name="join-the-anomaly-detector-community"></a>Rejoindre la communauté du détecteur d’anomalies

* Rejoindre le [groupe Anomaly Detector Advisors sur Microsoft Teams](https://aka.ms/AdAdvisorsJoin)
* Voir le [contenu généré par l’utilisateur](user-generated-content.md) sélectionné

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Détecter des anomalies dans vos données de séries chronologiques avec l’API Détecteur d’anomalies](quickstarts/detect-data-anomalies-csharp.md)
* [Démonstration en ligne](https://notebooks.azure.com/AzureAnomalyDetection/projects/anomalydetector) de l’API Détecteur d’anomalies
* La [référence d’API REST](https://aka.ms/anomaly-detector-rest-api-ref) du Détecteur d'anomalies
