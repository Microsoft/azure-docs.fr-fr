---
title: Modèles de solution Azure Stream Analytics
description: Découvrez des modèles de solution courants pour Azure Stream Analytics, comme les tableaux de bord, les messages d’événements, les magasins de données, l’enrichissement des données de référence et le monitorage.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/21/2019
ms.openlocfilehash: b82dd4ea8a74c51606b7a388b8d6ede07b1057c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89461542"
---
# <a name="azure-stream-analytics-solution-patterns"></a>Modèles de solution Azure Stream Analytics

Comme de nombreux autres services présents dans Azure, Stream Analytics trouve sa grande utilité lorsqu’il est associé à d’autres services pour créer une solution de bout en bout plus étendue. Cet article aborde les solutions Azure Stream Analytics simples ainsi que différents modèles d’architecture. Vous pouvez vous appuyer sur ces modèles pour développer des solutions plus complexes. Les modèles décrits dans cet article sont exploitables dans un large éventail de scénarios. Des exemples de modèles propres aux scénarios sont disponibles dans les [Architectures de solution Azure](https://azure.microsoft.com/solutions/architecture/?product=stream-analytics).

## <a name="create-a-stream-analytics-job-to-power-real-time-dashboarding-experience"></a>Créer un travail Stream Analytics pour alimenter l’expérience des tableaux de bord en temps réel

Avec Azure Stream Analytics, vous pouvez rapidement mettre en place des alertes et des tableaux de bord en temps réel. Une solution simple ingère des événements à partir d’Event Hubs ou d’IoT Hub, puis [alimente le tableau de bord Power BI avec un jeu de données en streaming](/power-bi/service-real-time-streaming). Pour plus d’informations, consultez le tutoriel détaillé [Analyser les données d’appel téléphonique avec Stream Analytics et visualiser les résultats dans un tableau de bord Power BI](stream-analytics-manage-job.md).

![Tableau de bord Power BI ASA](media/stream-analytics-solution-patterns/power-bi-dashboard.png)

Cette solution peut être générée en quelques minutes seulement à partir du portail Azure. Aucun codage complexe n’est impliqué, et la logique métier s’exprime avec le langage SQL.

Ce modèle de solution offre la plus faible latence, de la source d’événements au tableau de bord Power BI dans un navigateur. Azure Stream Analytics est le seul service Azure comportant cette fonctionnalité intégrée.

## <a name="use-sql-for-dashboard"></a>Utiliser SQL pour le tableau de bord

Le tableau de bord Power BI offre une faible latence, mais il est inutilisable dans la production de rapports Power BI complets. Un modèle de création de rapports courant se veut tout d’abord sortir vos données dans une base de données SQL. Ainsi, utilisez le connecteur SQL de Power BI pour interroger SQL sur les données les plus récentes.

![Tableau de bord SQL ASA](media/stream-analytics-solution-patterns/sql-dashboard.png)

L’utilisation d’une base de données SQL vous offre plus de flexibilité, mais elle se fait au détriment d’une latence légèrement plus élevée. Cette solution est optimale pour les travaux dont les exigences en termes de latence sont supérieures à une seconde. Avec cette méthode, vous pouvez optimiser les fonctionnalités de Power BI pour segmenter et traiter les données destinées aux rapports, et de nombreuses autres options de visualisation. Vous gagnez également en flexibilité avec l’utilisation d’autres solutions de tableau de bord, telles que Tableau.

SQL n’est pas un magasin de données à débit élevé. Le débit maximal, vers une base de données SQL depuis Azure Stream Analytics, se situe actuellement autour de 24 Mo/s. Si les sources d’événements de votre solution produisent des données à un taux plus élevé, vous devez utiliser une logique de traitement dans Stream Analytics pour réduire le taux de sortie vers SQL. Des techniques, telles que le filtrage, les agrégats fenêtrés, les critères spéciaux avec des jointures temporelles, et des fonctions analytiques, peuvent être utilisées. Il est possible d’optimiser davantage le taux de sortie vers SQL en recourant aux techniques décrites dans l’article [Sortie d’Azure Stream Analytics dans Azure SQL Database](stream-analytics-sql-output-perf.md).

## <a name="incorporate-real-time-insights-into-your-application-with-event-messaging"></a>Incorporer des insights en temps réel dans votre application avec la messagerie d’événements

La deuxième utilisation la plus appréciée de Stream Analytics est la création d’alertes en temps réel. Dans ce modèle de solution, vous pouvez utiliser une logique métier dans Stream Analytics afin de détecter les [modèles temporels et spatiaux](stream-analytics-geospatial-functions.md) ou les [anomalies](stream-analytics-machine-learning-anomaly-detection.md), et produire ensuite des signaux d’alerte. Toutefois, contrairement à la solution de tableau de bord dans laquelle Stream Analytics se sert de Power BI comme point de terminaison par défaut, certains récepteurs de données intermédiaires peuvent être utilisés. Ces récepteurs comprennent Event Hubs, Service Bus et Azure Functions. Vous, en tant que créateur d’applications, devez décider quel récepteur de données convient le mieux à votre scénario.

La logique de consommateur d’événements en aval doit être implémentée pour générer des alertes dans votre workflow d’entreprise existant. À partir du moment où vous pouvez implémenter une logique personnalisée dans Azure Functions, Azure Functions représente le moyen le plus rapide pour vous d’effectuer cette intégration. Un tutoriel sur l’utilisation d’Azure Functions en tant que sortie d’une tâche Stream Analytics est disponible dans [Exécuter Azure Functions à partir des travaux Azure Stream Analytics](stream-analytics-with-azure-functions.md). Azure Functions prend également en charge différents types de notifications, y compris le texte et les e-mails. Vous pouvez aussi utiliser Logic Apps pour ce genre d’intégration, en plaçant Event Hubs entre Stream Analytics et Logic Apps.

![Application de messagerie d’événements ASA](media/stream-analytics-solution-patterns/event-messaging-app.png)

Event Hubs, par ailleurs, offre le point d’intégration le plus souple. De nombreux autres services, tels qu’Azure Data Explorer et Time Series Insights, peuvent consommer des événements à partir d’Event Hubs. Des services peuvent être connectés directement au récepteur Event Hubs à partir d’Azure Stream Analytics pour terminer la solution. Event Hubs est également le répartiteur de messagerie au débit le plus élevé, disponible sur Azure pour de tels scénarios d’intégration.

## <a name="dynamic-applications-and-websites"></a>Sites web et applications dynamiques

Vous pouvez créer des visualisations en temps réel personnalisées, telles que la visualisation de tableaux de bord ou de cartes, avec Azure Stream Analytics et Azure SignalR Service. L’utilisation de SignalR permet la mise à jour des clients web et l’affichage du contenu dynamique en temps réel.

![Application dynamique ASA](media/stream-analytics-solution-patterns/dynamic-app.png)

## <a name="incorporate-real-time-insights-into-your-application-through-data-stores"></a>Incorporer à votre application des insights en temps réel par le biais de magasins de données

La plupart des services et applications web utilisent aujourd’hui un modèle de requête/réponse pour gérer la couche de présentation. Le modèle de requête/réponse est simple à créer et peut être facilement mis à l’échelle dans un temps de réponse faible, avec un front-end sans état et des magasins scalables, comme Cosmos DB.

Les volumes importants de données génèrent souvent des goulots d’étranglement dans un système CRUD. Le [modèle de solution de provisionnement en événements](/azure/architecture/patterns/event-sourcing) s’utilise pour résoudre les goulots d’étranglement au niveau des performances. Les insights et modèles temporels sont également ardus et inefficaces pour l’extraction à partir d’un magasin de données classique. Les applications modernes basées sur les données volumineuses adoptent souvent une architecture basée sur les dataflows. Azure Stream Analytics, en tant que moteur de calcul pour les données en mouvement, est un élément central dans cette architecture.

![Application de provisionnement en événements ASA](media/stream-analytics-solution-patterns/event-sourcing-app.png)

Dans ce modèle de solution, les événements sont traités et agrégés dans des magasins de données par Azure Stream Analytics. La couche d’application interagit avec les magasins de données par l’intermédiaire du modèle de requête/réponse classique. Du fait de la capacité de Stream Analytics à traiter un grand nombre d’événements en temps réel, l’application s’en trouve extrêmement évolutive, sans nécessiter la fourniture en bloc de la couche magasin de données. La couche magasin de données est avant tout une vue matérialisée dans le système. La [sortie d’Azure Stream Analytics vers Azure Cosmos DB](stream-analytics-documentdb-output.md) décrit l’utilisation de Cosmos DB en tant que sortie Stream Analytics.

Dans les applications réelles, où la logique de traitement est complexe et où il est nécessaire de mettre à niveau certaines parties de la logique séparément, plusieurs travaux Stream Analytics peuvent être combinés ensemble avec Event Hubs, en tant que répartiteur d’événements intermédiaire.

![Application de provisionnement d’événements complexe ASA](media/stream-analytics-solution-patterns/event-sourcing-app-complex.png)

Ce modèle améliore la résilience et facilite la gestion du système. Même si Stream Analytics garantit le traitement en une seule et unique fois, il existe malgré tout une faible probabilité pour que les événements en double puissent se retrouver dans l’instance Event Hubs placée en intermédiaire. Il est important pour le travail Stream Analytics en aval de dédupliquer les événements à l’aide de clés logiques dans une fenêtre de recherche arrière. Pour plus d’informations sur la remise d’événements, consultez la référence [Garanties de remise d’événements](/stream-analytics-query/event-delivery-guarantees-azure-stream-analytics).

## <a name="use-reference-data-for-application-customization"></a>Utiliser les données de référence pour la personnalisation des applications

La fonctionnalité des données de référence Azure Stream Analytics est prévue plus particulièrement pour la personnalisation de l’utilisateur final, telles que le seuil d’alerte, les règles de traitement et les [limites géographiques](geospatial-scenarios.md). La couche d’application peut accepter les modifications apportées aux paramètres et les stocker dans une base de données SQL. Le travail Stream Analytics interroge régulièrement sur les modifications à partir de la base de données, et rend les paramètres de personnalisation accessibles par le biais d’une jointure de données de référence. Pour plus d’informations sur l’utilisation des données de référence pour la personnalisation d’application, consultez [Données de référence SQL](sql-reference-data.md) et [Jointure de données de référence](/stream-analytics-query/reference-data-join-azure-stream-analytics).

Ce modèle peut être également utilisé pour implémenter un moteur de règles, où les seuils des règles sont définis à partir de données de référence. Pour plus d’informations sur les règles, consultez [Traiter des règles configurables basées sur un seuil dans Azure Stream Analytics](stream-analytics-threshold-based-rules.md).

![Application de données de référence ASA](media/stream-analytics-solution-patterns/reference-data-app.png)

## <a name="add-machine-learning-to-your-real-time-insights"></a>Ajouter Machine Learning à vos insights en temps réel

Le [modèle de détection d’anomalie](stream-analytics-machine-learning-anomaly-detection.md) intégrée d’Azure Stream Analytics est un moyen pratique pour introduire Machine Learning dans votre application en temps réel. Pour un plus large éventail des besoins Machine Learning, consultez [Azure Stream Analytics s’intègre au service de scoring d’Azure Machine Learning](stream-analytics-machine-learning-integration-tutorial.md).

Pour les utilisateurs expérimentés qui souhaitent intégrer le scoring et la formation en ligne dans le même pipeline Stream Analytics, consultez cet exemple pour savoir comment procéder en utilisant la [régression linéaire](stream-analytics-high-frequency-trading.md).

![Application Machine Learning ASA](media/stream-analytics-solution-patterns/machine-learning-app.png)

## <a name="real-time-data-warehousing"></a>Entreposage de données en temps réel

L’entreposage de données en quasi temps réel, également appelé entrepôt de données de streaming est un autre modèle courant. En plus des événements arrivant dans Event Hubs et IoT Hub depuis votre application, vous pouvez utiliser [Azure Stream Analytics running on IoT Edge](stream-analytics-edge.md) pour répondre aux besoins de nettoyage, de réduction ainsi que de stockage et transfert de données. Stream Analytics running on IoT Edge peut gérer convenablement les problèmes de connectivité et de limitation de la bande passante dans le système. Stream Analytics peut prendre en charge des taux de débit allant jusqu’à 200 Mo/s lors de l’écriture dans Azure Synapse Analytics.

![Entreposage de données ASA](media/stream-analytics-solution-patterns/data-warehousing.png)


## <a name="archiving-real-time-data-for-analytics"></a>Archivage des données en temps réel pour l’analytique

La plupart des activités d’analytique et de science des données s’effectuent encore hors connexion. Les données peuvent être archivées par Azure Stream Analytics via les formats de sortie Azure Data Lake Store Gen2 et Parquet. Cette fonctionnalité supprime la difficulté d’alimenter directement en données Azure Data Lake Analytics, Azure Databricks et Azure HDInsight. Azure Stream Analytics sert de moteur ETL en quasi temps réel dans cette solution. Vous pouvez explorer des données archivées dans Data Lake avec différents moteurs de calcul.

> [!div class="mx-imgBorder"]
> ![ASA offline analytics](media/stream-analytics-solution-patterns/offline-analytics.png)

## <a name="use-reference-data-for-enrichment"></a>Utiliser les données de référence pour l’enrichissement

L’enrichissement des données constitue souvent une condition exigée pour les moteurs ETL. Azure Stream Analytics prend en charge l’enrichissement des données avec les [données de référence](stream-analytics-use-reference-data.md) provenant de la base de données SQL et du stockage Blob Azure. L’enrichissement des données peut être effectué pour l’arrivage de données dans Azure Data Lake et dans Azure Synapse Analytics.


![Analytique ASA en mode hors connexion avec enrichissement des données](media/stream-analytics-solution-patterns/offline-analytics-enriched.png)

## <a name="operationalize-insights-from-archived-data"></a>Rendre des insights opérationnels à partir de données archivées

Si vous combinez le modèle d’analytique hors connexion avec le modèle d’application en quasi temps réel, vous pouvez créer une boucle de rétroaction. La boucle de rétroaction permet à l’application de s’ajuster automatiquement aux modifications des modèles dans les données. Cette boucle de rétroaction peut se révéler aussi simple qu’une modification de la valeur du seuil d’alerte, ou aussi complexe que le réentraînement des modèles Machine Learning. La même architecture de solution peut être appliquée autant aux travaux ASA s’exécutant dans le cloud que sur IoT Edge.

![Opérationnalisation d’insights ASA](media/stream-analytics-solution-patterns/insights-operationalization.png)

## <a name="how-to-monitor-asa-jobs"></a>Guide pratique pour superviser les travaux ASA

Un travail Azure Stream Analytics peut s’exécuter 24/24 7/7 pour traiter des événements entrants sans interruption en temps réel. Sa garantie de durée de fonctionnement est cruciale pour l’intégrité de l’application dans son ensemble. Même si Stream Analytics est le seul service d’analytique en streaming du secteur qui offre une [garantie de disponibilité de 99,9 %](https://azure.microsoft.com/support/legal/sla/stream-analytics/v1_0/), vous risquez tout de même de connaître un certain niveau de temps d’arrêt. Au fil des années, Stream Analytics a introduit les métriques, les journaux et les états des tâches pour refléter l’intégrité des travaux. Tous sont exposés via le service Azure Monitor, et peuvent même être exportés vers OMS. Pour plus d’informations, consultez [Présentation de la supervision des travaux Stream Analytics et des requêtes](stream-analytics-monitoring.md).

![Supervision ASA](media/stream-analytics-solution-patterns/monitoring.png)

Il y a deux éléments clés à surveiller :

- [État d’échec du travail](job-states.md)

    Avant toute chose, vous devez vous assurer que le travail est en cours d’exécution. Sans travail associé à un état en cours d’exécution, aucune nouvelle métrique et aucun journal n’est généré. Les travaux peuvent afficher un état d’échec pour diverses raisons, entre autres avoir un niveau élevé d’utilisation de SU (autrement dit, manquer de ressources).

- [Métrique du délai en filigrane](https://azure.microsoft.com/blog/new-metric-in-azure-stream-analytics-tracks-latency-of-your-streaming-pipeline/)

    Cette métrique reflète le retard pris par votre pipeline de traitement en temps Horloge (secondes). Un certain délai est attribué à la logique de traitement inhérente. En conséquence, la supervision de la tendance croissante est beaucoup plus importante que la surveillance de la valeur absolue. Le délai de l’état stable devrait être corrigé par la conception de votre application, pas par la supervision ou les alertes.

En cas d’échec, les journaux d’activité et [les journaux de diagnostic](stream-analytics-job-diagnostic-logs.md) sont les meilleurs endroits où commencer à rechercher des erreurs.

## <a name="build-resilient-and-mission-critical-applications"></a>Créer des applications stratégiques et résilientes

Quels que soient la garantie du contrat SLA d’Azure Stream Analytics utilisé et le soin que vous apportez à exécuter votre application de bout en bout, des pannes se produisent. Si votre application est stratégique, vous devez être prêt à réagir face aux interruptions, afin de procéder à une récupération correctement.

Pour les applications de génération d’alertes, le plus important est de détecter l’alerte suivante. Vous pouvez choisir de redémarrer le travail à l’heure en cours au moment de la récupération, et donc d’ignorer les alertes passées. La sémantique de l’heure de début du travail s’établit en fonction de la première heure de sortie, pas de la première heure d’entrée. L’entrée est rembobinée vers l’arrière, jusqu’à une quantité appropriée de temps permettant de garantir que la première sortie à l’heure spécifiée est complète et correcte. En procédant ainsi, vous n’obtenez aucun agrégat partiel et ne déclenchez aucune alerte de façon inattendue.

Vous pouvez également choisir de démarrer la sortie à partir d’une certaine quantité de temps dans le passé. Les stratégies de conservation Event Hubs et IoT Hub contiennent une quantité raisonnable de données permettant d’autoriser le traitement dans le passé. Le compromis tient dans la rapidité avec laquelle vous pouvez rattraper le retard par rapport à l’heure actuelle, et commencer à générer de nouvelles alertes dans les délais. Les données perdent rapidement leur valeur au fil du temps, il est donc important de rattraper au plus vite l’heure actuelle. Il existe deux façons de rattraper rapidement un retard :

- Provisionner davantage de ressources (SU) lors du rattrapage du retard.
- Redémarrer à partir de l’heure actuelle.

Le redémarrage à l’heure actuelle est simple à réaliser, en acceptant le compromis de devoir laisser un intervalle vide lors du traitement. Le redémarrage par ce moyen peut accessoirement fonctionner pour les scénarios d’alerte, mais s’avérer problématique avec les scénarios de tableau de bord, et être voué à l’échec pour l’archivage et les scénarios d’entreposage de données.

Le provisionnement de ressources supplémentaires peut accélérer le processus, mais l’effet d’une augmentation brutale de la vitesse de traitement est complexe.

- Vérifiez que votre travail peut être mis à l’échelle sur un plus grand nombre de SU. Les requêtes ne sont pas toutes adaptables. Vous devez vous assurer que votre requête est [parallélisée](stream-analytics-parallelization.md).

- Veillez à ce qu’il existe suffisamment de partitions en amont dans les services Event Hubs ou IoT Hub pour que vous puissiez ajouter plus d’unités de débit (TU) et ainsi mettre à l’échelle le débit d’entrée. N’oubliez pas, chaque TU Event Hubs atteint la limite maximale à une vitesse de sortie de 2 Mo/s.

- Assurez-vous d’avoir provisionné assez de ressources dans les récepteurs de sortie (c’est-à-dire SQL Database, Cosmos DB), afin qu’ils ne répriment pas l’afflux important en sortie, qui peut parfois entraîner le blocage du système.

L’essentiel est plus de tout d’anticiper la modification de la vitesse de traitement, de tester ces scénarios avant la mise en production, et d’être prêt à la mise à l’échelle correcte du traitement pendant la durée de récupération après une défaillance.

Dans le scénario extrême où les événements entrants sont tous retardés, [il est possible que tous les événements différés soient supprimés](stream-analytics-time-handling.md) si vous avez appliqué une plage d’arrivée tardive à votre travail. La suppression des événements peut sembler un comportement étrange au début, mais si l’on considère que Stream Analytics est un moteur de traitement en temps réel, celui-ci s’attend donc à ce que les événements entrants soient proches du temps Horloge. Il doit donc supprimer les événements qui violent ces contraintes.

### <a name="lambda-architectures-or-backfill-process"></a>Architectures lambda ou processus de renvoi

Heureusement, le modèle d’archivage de données précédent peut être utilisé pour traiter normalement ces événements tardifs. L’idée est que le travail d’archivage traite les événements entrants à l’heure d’arrivée, et archive les événements dans le compartiment du moment opportun, dans un objet blob Azure ou dans Azure Data Lake Store, avec leur heure d’événement. Peu importe l’heure tardive à laquelle un événement arrive, il n’est jamais supprimé. Il est toujours placé dans le compartiment du moment opportun. Lors de la récupération, il est possible de retraiter les événements archivés et de renvoyer les résultats dans le magasin voulu. Cela est similaire à la façon dont les modèles lambda sont implémentés.

![Renvoi ASA](media/stream-analytics-solution-patterns/back-fill.png)

Le processus de renvoi doit être effectué avec un système de traitement par lots hors connexion ; celui-ci a probablement un autre modèle de programmation qu’Azure Stream Analytics. Cela signifie que vous devez implémenter de nouveau et en entier la logique de traitement.

Pour un renvoi, il demeure important de provisionner au moins temporairement plus de ressources sur les récepteurs de sortie, afin de gérer un débit plus élevé que les besoins du traitement de l’état stable.

|Scénarios  |Redémarrer à partir de maintenant uniquement  |Redémarrer à partir de l’heure du dernier arrêt |Redémarrer à partir de maintenant + renvoyer avec des événements archivés|
|---------|---------|---------|---------|
|**Tableaux de bord**   |Crée l’intervalle vide    |OK pour une brève interruption    |Utiliser pour une longue interruption |
|**Alertes**   |Acceptable |OK pour une brève interruption    |Pas nécessaire |
|**Application de provisionnement en événements** |Acceptable |OK pour une brève interruption    |Utiliser pour une longue interruption |
|**Entrepôt de données**   |Perte de données  |Acceptable |Pas nécessaire |
|**Analytique hors connexion**  |Perte de données  |Acceptable |Pas nécessaire|

## <a name="putting-it-all-together"></a>Exemple complet

Il est facile d’imaginer que tous les modèles de solution mentionnés ci-dessus sont combinables entre eux, dans un système de bout en bout complexe. Le système composé peut inclure des fonctionnalités de tableaux de bord, d’alertes, d’application de provisionnement en événements, d’entreposage de données et d’analytique hors connexion.

L’essentiel est de concevoir votre système dans des modèles composables, de telle sorte que chaque sous-système peut être généré, testé, mis à niveau et récupéré séparément.

## <a name="next-steps"></a>Étapes suivantes

Vous venez de découvrir différents modèles de solution avec Azure Stream Analytics. Pour approfondir ces connaissances et créer votre premier travail Stream Analytics, consultez les articles suivants :

* [Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md).
* [Créer un travail Stream Analytics à l’aide d’Azure PowerShell](stream-analytics-quick-create-powershell.md).
* [Créer un travail Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md).
