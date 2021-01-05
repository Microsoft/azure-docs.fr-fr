---
title: Mise à l’échelle des fonctions Machine Learning studio (classique) dans Azure Stream Analytics
description: Cet article explique comment mettre à l’échelle des travaux Stream Analytics qui utilisent des fonctions Machine Learning studio (classique) en configurant des unités de partitionnement et de streaming.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/16/2020
ms.openlocfilehash: feeb709f67a0e75f5980ec0520b95feb7edd5960
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018815"
---
# <a name="scale-your-stream-analytics-job-with-azure-machine-learning-studio-classic-functions"></a>Mettre à l’échelle votre travail Stream Analytics avec des fonctions Azure Machine Learning Studio (classique)

> [!TIP]
> Il est vivement recommandé d’utiliser des [UDF Azure Machine Learning](machine-learning-udf.md) au lieu d’une UDF Azure Machine Learning Studio (classique) pour améliorer les performances et la fiabilité.

Cet article explique comment mettre efficacement à l’échelle des travaux Azure Stream Analytics qui utilisent des fonctions Azure Machine Learning studio (classique). Pour plus d’informations sur la procédure de mise à l’échelle des travaux Stream Analytics en général, voir l’article [Mise à l’échelle des travaux](stream-analytics-scale-jobs.md).

## <a name="what-is-a-studio-classic-function-in-stream-analytics"></a>Qu’est-ce qu’une fonction studio (classique) dans Stream Analytics ?

Une fonction Machine Learning studio (classique) dans Stream Analytics peut être utilisée comme un appel de fonction standard dans le langage de requête Stream Analytics. En arrière-plan toutefois, ces appels de fonction constituent en fait des demandes de service web studio (classique).

Il est possible d’améliorer le débit des demandes de service web studio (classique) en traitant « par lots » plusieurs lignes dans le même appel d’API de service web. Ce regroupement est appelé un mini-lot. Pour plus d’informations, consultez [Services web Azure Machine Learning Studio (classique)](../machine-learning/classic/consume-web-services.md). La prise en charge de studio (classique) dans Stream Analytics est en préversion.

## <a name="configure-a-stream-analytics-job-with-studio-classic-functions"></a>Configuration d’un travail Stream Analytics avec des fonctions studio (classique)

Deux paramètres permettent de configurer la fonction studio (classique) utilisée par le travail Stream Analytics :

* Taille de lot des appels de fonction studio (classique).
* Nombre d’unités streaming (SU) approvisionnées pour le travail Stream Analytics.

Pour déterminer les valeurs appropriées de SU, choisissez d’optimiser la latence du travail Stream Analytics ou le débit de chaque SU. Il est toujours possible d’ajouter des SU à un travail pour augmenter le débit d’une requête Stream Analytics correctement partitionnée. Des SU supplémentaires augmentent le coût d’exécution du travail.

Déterminez la *tolérance* de latence pour votre travail Stream Analytics. Une augmentation de la taille de lot aura pour effet d’accroître la latence des demandes studio (classique) et du travail Stream Analytics.

L’augmentation de la taille de lot permet au travail Stream Analytics de traiter **davantage d’événements** avec le **même nombre** de demandes de service web studio (classique). La latence du service web studio (classique) croît généralement moins vite que la taille de lot. 

Quelle que soit la situation, il est important de prendre en compte la taille de lot la plus économique pour un service web studio (classique). La taille de lot par défaut pour les demandes de service web est de 1 000. Vous pouvez modifier cette taille par défaut à l’aide de [l’API REST Stream Analytics](/previous-versions/azure/mt653706(v=azure.100) "API REST Stream Analytics") ou du [client PowerShell pour Stream Analytics](stream-analytics-monitor-and-manage-jobs-use-powershell.md).

Lorsque vous avez choisi une taille de lot, vous pouvez définir le nombre d’unités de streaming (SU) en fonction du nombre d’événements que la fonction doit traiter par seconde. Pour plus d’informations sur les unités de diffusion en continu, voir [Mise à l’échelle des travaux Stream Analytics](stream-analytics-scale-jobs.md).

Chaque ensemble de 6 SU bénéficie de 20 connexions simultanées au service web studio (classique). Toutefois, 1 travail SU et 3 travaux SU obtiennent 20 connexions simultanées.  

Si votre application génère 200 000 événements par seconde, et que taille du lot est de 1 000, la latence de service web qui en résulte est de 200 ms. Avec ce débit, chaque connexion peut envoyer cinq requêtes au service web studio (classique) par seconde. Dans le cas de 20 connexions, le travail Stream Analytics peut traiter 20 000 événements en 200 ms et 100 000 événements en une seconde.

Pour traiter 200 000 événements par seconde, le travail Stream Analytics a besoin de 40 connexions simultanées, ce qui correspond à 12 SU. Le diagramme ci-dessous illustre les demandes effectuées du travail Stream Analytics vers le point de terminaison de service web studio (classique). Pour chaque ensemble de 6 unités de streaming, il existe 20 connexions simultanées au service web studio (classique) maximum.

![Mise à l’échelle de Stream Analytics avec des fonctions studio (classique) – Exemple de deux travaux](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-00.png "Mise à l’échelle de Stream Analytics avec des fonctions studio (classique) – Exemple de deux travaux")

Généralement, **_B_* _ pour la taille de lot, _*_L_*_ pour la latence du service web en millisecondes pour la taille de lot B, le débit d’un travail Stream Analytics avec _*_N_*_ unités de diffusion en continu est le suivant :

![Mise à l’échelle de Stream Analytics avec des fonctions studio (classique) – Formule](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-02.png "Mise à l’échelle de Stream Analytics avec des fonctions studio (classique) – Formule")

Vous pouvez également configurer le « nombre maximal d’appels simultanés » sur le service web studio (classique). Il est recommandé de définir ce paramètre sur la valeur maximale (200 actuellement).

Pour plus d’informations sur ce paramètre, consultez l’article [Mise à l’échelle pour les services web Machine Learning studio (classique)](../machine-learning/classic/create-endpoint.md).

## <a name="example--sentiment-analysis"></a>Exemple – Analyse de sentiments
L’exemple suivant comprend un travail Stream Analytics avec la fonction studio (classique) d’analyse des sentiments (cf. [Tutoriel d’intégration de Machine Learning studio (classique) à Stream Analytics](stream-analytics-machine-learning-integration-tutorial.md)).

La requête est une requête simple entièrement partitionnée suivie de la fonction _ *sentiment**, comme indiqué dans l’exemple suivant :

```SQL
    WITH subquery AS (
        SELECT text, sentiment(text) as result from input
    )

    Select text, result.[Score]
    Into output
    From subquery
```

Commençons par examiner la configuration nécessaire pour créer un travail Stream Analytics qui analyse le sentiment de tweets à un débit de 10 000 tweets par seconde.

Avec 1 SU, ce travail Stream Analytics pourrait-il de gérer le trafic ? Le travail peut prendre en charge le trafic d’entrée en utilisant la taille de lot par défaut de 1 000. La latence par défaut du service web studio (classique) d’analyse des sentiment (avec une taille de lot par défaut de 1 000) ne crée pas plus d’une seconde de latence.

La latence **totale** ou de bout en bout du travail Stream Analytics serait généralement de l’ordre de quelques secondes. Examinons ce travail Stream Analytics plus en détail, *en particulier* les appels de fonction studio (classique). Avec une taille de lot de 1 000, un débit de 10 000 événements nécessitera l’envoi de 10 requêtes environ au service web. Même avec 1 unité de streaming, le nombre de connexions simultanées est suffisant pour prendre en charge ce trafic d’entrée.

Si le taux d’événements d’entrée est multiplié par 100, le travail Stream Analytics devra alors traiter 1 000 000 de tweets par seconde. Deux options permettent de gérer cette augmentation :

1. Augmentez la taille du lot.
2. Partitionnez le flux d’entrée pour traiter les événements en parallèle.

Si nous choisissons la première option, la **latence** du travail augmente.

Avec la seconde option, vous devrez provisionner plus de SU pour générer plus de demandes de service web studio (classique) simultanées. Ce plus grand nombre de SU entraîne une augmentation du **coût** du travail.

Observons la mise à l’échelle à l’aide des mesures de latence suivantes pour chaque taille de lot :

| Latence | Taille du lot |
| --- | --- |
| 200 ms | lots de 1 000 événements maximum |
| 250 ms | lots de 5 000 événements |
| 300 ms | lots de 10 000 événements |
| 500 ms | lots de 25 000 événements |

1. Utilisation de la première option (**ne pas** provisionner plus d’unités de stockage). Vous pourriez augmenter la taille du lot pour atteindre **25 000**. Cette augmentation permettra au travail de traiter un million d’événements avec 20 connexions simultanées au service web studio (classique) (avec une latence de 500 ms par appel). La latence supplémentaire du travail Stream Analytics découlant des demandes de la fonction sentiment par rapport aux demandes de service web studio (classique) passerait donc de **200 ms** à **500 ms**. Toutefois, la taille de lot **ne peut pas** être augmentée à l’infini, car les services web studio (classique) exigent que la taille utile d’une requête soit de 4 Mo maximum et imposent un délai d’expiration de 100 secondes pour les demandes de service web.
1. Si nous utilisions la deuxième option, en conservant une taille de lot de 1000, avec une latence de service web de 200 ms, chaque ensemble de 20 connexions simultanées au service web est en mesure de traiter 20 * 1000 * 5 événements = 100 000 événements par seconde. Pour traiter 1 000 000 d’événements par seconde, le travail aurait donc besoin de 60 unités de diffusion en continu. Par rapport à la première option, le travail Stream Analytics effectuerait un plus grand nombre de demandes de service web par lot, ce qui entraînerait alors une augmentation des coûts.

Le tableau ci-après présente le débit du travail Stream Analytics pour différentes unités de diffusion en continu et tailles de lot (en nombre d’événements par seconde).

| Taille de lot (latence Machine Learning) | 500 (200 ms) | 1 000 (200 ms) | 5 000 (250 ms) | 10 000 (300 ms) | 25 000 (500 ms) |
| --- | --- | --- | --- | --- | --- |
| **1 unité de diffusion en continu** |2 500 |5 000 |20 000 |30,000 |50 000 |
| **3 unités de diffusion en continu** |2 500 |5 000 |20 000 |30,000 |50 000 |
| **6 unités de diffusion en continu** |2 500 |5 000 |20 000 |30,000 |50 000 |
| **12 unités de diffusion en continu** |5 000 |10 000 |40 000 |60 000 |100 000 |
| **18 unités de diffusion en continu** |7 500 |15,000 |60 000 |90 000 |150 000 |
| **24 unités de diffusion en continu** |10 000 |20 000 |80 000 |120 000 |200 000 |
| **…** |… |… |… |… |… |
| **60 unités de diffusion en continu** |25 000 |50 000 |200 000 |300 000 |500 000 |

Vous avez déjà une bonne compréhension du fonctionnement des fonctions studio (classique) dans Stream Analytics. Vous devez également avoir compris que les travaux Stream Analytics tirent (pull) les données des sources de données, et que chaque tirage retourne un lot d’événements destinés à être traités par le travail Stream Analytics. Comment ce modèle de transmission de type pull affecte-t-il les demandes de service web studio (classique) ?

En règle générale, la taille de lot définie pour les fonctions studio (classique) n’est pas exactement divisible par le nombre d’événements retournés par chaque tirage (pull) de travail Stream Analytics. Dans ce cas, le service web studio (classique) est appelé avec des lots « partiels ». L’utilisation de lots partiels évite un surcroît de latence de travail induit par le regroupement des événements issus des différentes transmissions de type pull.

## <a name="new-function-related-monitoring-metrics"></a>Nouvelles métriques de surveillance associées aux fonctions
Dans la zone de surveillance d’un travail Stream Analytics, trois nouvelles métriques associées aux fonctions ont été ajoutées. Il s’agit des métriques **DEMANDES DE FONCTION**, **ÉVÉNEMENTS DE FONCTION** et **DEMANDES DE FONCTION AYANT ÉCHOUÉ**, comme illustré dans le graphique ci-après.

![Mise à l’échelle de Stream Analytics avec des fonctions studio (classique) – Métriques](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-01.png "Mise à l’échelle de Stream Analytics avec des fonctions studio (classique) – Métriques")

Les définitions de ces métriques sont les suivantes :

**DEMANDES DE FONCTION** : Nombre de demandes de fonction.

**ÉVÉNEMENTS DE FONCTION** : Nombre d’événements dans les demandes de fonction.

**DEMANDES DE FONCTION AYANT ÉCHOUÉ** : Nombre de demandes de fonction ayant échoué.

## <a name="key-takeaways"></a>Points clés

Pour mettre à l’échelle un travail Stream Analytics avec des fonctions studio (classique), prenez en compte les facteurs suivants :

1. Taux d’événements d’entrée.
2. Latence tolérée pour le travail Stream Analytics en cours d’exécution (et donc taille de lot des demandes de service web studio (classique)).
3. SU Stream Analytics provisionnées et nombre de demandes de service web studio (classique) (coûts associés aux fonctions supplémentaires).

Notre exemple portait sur une requête Stream Analytics entièrement partitionnée. Si vous avez besoin d’une requête plus complexe, la [Page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html) est une précieuse ressource pour obtenir une aide supplémentaire auprès de l’équipe Stream Analytics.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur Stream Analytics, consultez :

* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)