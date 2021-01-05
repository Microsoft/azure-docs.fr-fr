---
title: Surveiller les requêtes
titleSuffix: Azure Cognitive Search
description: Surveillez les performances et le volume des requêtes à l'aide des métriques. Collectez et analysez les entrées des chaînes de requêtes dans les journaux de ressources.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/18/2020
ms.openlocfilehash: a5589a46a63437fb395db280222f8a9e84775df3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935073"
---
# <a name="monitor-query-requests-in-azure-cognitive-search"></a>Surveiller les demandes de requête dans Recherche cognitive Azure

Cet article explique comment mesurer les performances et le volume des requêtes à l’aide des métriques et de la journalisation des ressources. Il explique également comment collecter les termes d'entrée utilisés dans les requêtes - informations nécessaires à l'évaluation de l'utilité et de l'efficacité de votre corpus de recherche.

Les données historiques qui alimentent les métriques sont conservées pendant 30 jours. Pour une conservation plus longue, ou pour générer des rapports sur les données opérationnelles et les chaînes de requêtes, activez un [paramètre de diagnostic](search-monitor-logs.md) spécifiant l'option de stockage relative aux métriques et événements consignés persistants.

Pour optimiser l'intégrité de la mesure des données, appliquez notamment les conditions suivantes :

+ Utilisez un service facturable (service créé au niveau De base ou Standard). Le service gratuit est partagé par différents abonnés, ce qui introduit une certaine volatilité à mesure que les charges se déplacent.

+ Si possible, utilisez un réplica et une partition uniques afin de créer un environnement autonome et isolé. Si vous utilisez différents réplicas, la moyenne des métriques de requête est calculée sur différents nœuds, ce qui peut nuire à la précision des résultats. De même, si vous utilisez différentes partitions, les données sont éparpillées, avec le risque que certaines partitions présentent des données différentes si une indexation est également en cours. Lors du paramétrage des performances des requêtes, un nœud et une partition uniques offrent un environnement plus stable pour les tests.

> [!Tip]
> Grâce à un code côté client supplémentaire et à Application Insights, vous pouvez également capturer des données de clics pour bénéficier d'insights plus approfondis sur ce qui suscite l'intérêt des utilisateurs de votre application. Pour plus d’informations, consultez [Analytique du trafic des recherches](search-traffic-analytics.md).

## <a name="query-volume-qps"></a>Volume de requêtes (RPS)

Le volume est mesuré en **requêtes de recherche par seconde** (RPS), une métrique intégrée qui peut être exprimée en tant que moyenne, nombre, minimum ou maximum de requêtes exécutées sur une période d'une minute. Pour les métriques, des intervalles d'une minute (TimeGrain = "PT1M") sont fixés dans le système.

Les requêtes sont souvent exécutées en quelques millisecondes, par conséquent seules les requêtes mesurées en secondes apparaissent dans les métriques.

| Type d’agrégation | Description |
|------------------|-------------|
| Average | Sur une période d'une minute, nombre moyen de secondes nécessaires à l'exécution de la requête.|
| Count | Nombre de métriques émises dans le journal pendant l'intervalle d'une minute. |
| Maximale | Nombre maximum de requêtes de recherche par seconde enregistrées en une minute. |
| Minimum | Nombre minimum de requêtes de recherche par seconde enregistrées en une minute.  |
| SUM | Somme de toutes les requêtes exécutées pendant l'intervalle d'une minute.  |

Par exemple, sur une minute, le schéma peut être le suivant : une seconde de charge élevée, qui représente votre valeur SearchQueriesPerSecond maximale, puis 58 secondes de charge moyenne, et enfin une seconde avec une seule requête, qui représente la valeur minimale.

Autre exemple : si un nœud émet 100 métriques, sachant que la valeur de chaque métrique est égale à 40, alors "Count" = 100, "Sum" = 4000, "Average" = 40 et "Max" = 40.

## <a name="query-performance"></a>Performances des requêtes

À l'échelle du service, les performances des requêtes sont mesurées en termes de latence de recherche (durée d'exécution d'une requête) et de requêtes limitées qui ont été abandonnées en raison d'un conflit de ressources.

### <a name="search-latency"></a>Latence de recherche

| Type d’agrégation | Latence | 
|------------------|---------|
| Average | Durée moyenne de la requête en millisecondes. | 
| Count | Nombre de métriques émises dans le journal pendant l'intervalle d'une minute. |
| Maximale | Requête la plus longue de l'échantillon. | 
| Minimum | Requête la plus courte de l'échantillon.  | 
| Total | Durée d'exécution totale de toutes les requêtes de l'échantillon, exécutées dans l'intervalle (une minute).  |

Penchons-nous sur l'exemple suivant de métriques de **latence de recherche** : 86 requêtes ont été échantillonnées, avec une durée moyenne de 23,26 millisecondes. Un minimum de 0 indique que certaines requêtes ont été abandonnées. La requête la plus longue s'est exécutée en 1 000 millisecondes. La durée d'exécution totale a été de 2 secondes.

![Agrégations de latence](./media/search-monitor-usage/metrics-latency.png "Agrégations de latence")

### <a name="throttled-queries"></a>Requêtes limitées

Les requêtes limitées font référence aux requêtes qui sont abandonnées au lieu d'être traitées. Dans la plupart des cas, la limitation fait partie intégrante de l'exécution du service.  Cela ne traduit pas nécessairement un problème.

La limitation intervient lorsque le nombre de demandes en cours de traitement dépasse les ressources disponibles. Vous pouvez constater une augmentation du nombre de demandes limitées lorsqu'un réplica est retiré de la rotation ou pendant l'indexation. Les demandes de requête et d'indexation sont gérées par le même ensemble de ressources.

Le service détermine s'il faut abandonner les demandes en fonction de la consommation des ressources. Le pourcentage de ressources consommées sur la mémoire, le processeur et les E/S disque est calculé en moyenne sur une période donnée. Si ce pourcentage dépasse un certain seuil, toutes les demandes adressées à l'index sont limitées jusqu'à ce que le volume de demandes baisse. 

Selon votre client, une demande limitée peut être signalée comme suit :

+ Un service renvoie une erreur « Trop grand nombre de requêtes. Veuillez réessayer plus tard. » 
+ Un code d'erreur 503 est renvoyé pour indiquer que le service est indisponible. 
+ Si vous utilisez le portail (par exemple, l'Explorateur de recherche), la requête est abandonnée en mode silencieux et vous devez à nouveau cliquer sur Rechercher.

Pour confirmer les requêtes limitées, utilisez la métrique **Requêtes de recherche limitées**. Vous pouvez explorer les métriques sur le portail ou créer une métrique d'alerte comme décrit dans cet article. Pour les requêtes qui ont été abandonnées au cours de l'intervalle d'échantillonnage, utilisez *Total* afin d'obtenir le pourcentage de requêtes qui n'ont pas été exécutées.

| Type d’agrégation | Limitation |
|------------------|-----------|
| Average | Pourcentage de requêtes abandonnées pendant l'intervalle. |
| Count | Nombre de métriques émises dans le journal pendant l'intervalle d'une minute. |
| Maximale | Pourcentage de requêtes abandonnées pendant l'intervalle.|
| Minimum | Pourcentage de requêtes abandonnées pendant l'intervalle. |
| Total | Pourcentage de requêtes abandonnées pendant l'intervalle. |

Pour **Pourcentage de requêtes de recherche limitées**, les valeurs minimales, maximales, moyennes et totales seront identiques : il s'agit du pourcentage de requêtes de recherche qui ont été limitées, en fonction du nombre total de requêtes de recherche pendant une minute.

Dans la capture d'écran suivante, la première valeur correspond au nombre de métriques envoyées au journal. Les agrégations supplémentaires, qui apparaissent en haut ou en pointant sur la métrique, incluent la moyenne, le maximum et le total. Dans cet exemple, aucune demande n'a été abandonnée.

![Agrégations limitées](./media/search-monitor-usage/metrics-throttle.png "Agrégations limitées")

## <a name="explore-metrics-in-the-portal"></a>Explorer les métriques sur le portail

Pour un aperçu rapide des valeurs actuelles, l'onglet **Surveillance** de la page de présentation du service affiche trois mesures (**Latence de recherche**, **Requêtes de recherche par seconde (par unité de recherche)** , **Pourcentage de requêtes de recherche limitées**) sur des intervalles fixes mesurés en heures, jours et semaines, avec la possibilité de modifier le type d'agrégation.

Pour une exploration plus approfondie, ouvrez Metrics Explorer à partir du menu **Surveillance**. Vous pourrez ainsi ajouter des données, effectuer un zoom sur celles-ci et les visualiser afin d'explorer les tendances ou les anomalies. Pour en savoir plus sur Metrics Explorer, suivez ce [tutoriel consacré à la création d'un graphique de métriques](../azure-monitor/learn/tutorial-metrics-explorer.md).

1. Dans la section Surveillance, sélectionnez **Métriques** pour ouvrir Metrics Explorer en veillant à ce que l'étendue soit définie en fonction de votre service de recherche.

1. Sous Métriques, choisissez-en une dans la liste déroulante et passez en revue la liste des agrégations disponibles pour sélectionner le type de votre choix. L'agrégation définit la manière dont les valeurs collectées seront échantillonnées sur chaque intervalle de temps.

   ![Metrics Explorer pour la métrique RPS](./media/search-monitor-usage/metrics-explorer-qps.png "Metrics Explorer pour la métrique RPS")

1. Dans le coin supérieur droit, définissez l'intervalle de temps.

1. Choisissez une visualisation. La visualisation par défaut est un graphique en courbes.

1. Ajoutez des agrégations supplémentaires en choisissant **Ajouter des métriques** et en sélectionnant différentes agrégations.

1. Zoomez sur une zone d'intérêt du graphique en courbes. Placez le pointeur de la souris au début de la zone, cliquez et maintenez le bouton gauche de la souris enfoncé, faites glisser de l’autre côté de la zone, puis relâchez le bouton. Cet intervalle de temps sera agrandi dans le graphique.

## <a name="identify-strings-used-in-queries"></a>Identifier les chaînes utilisées dans les requêtes

Quand vous activez la journalisation des ressources, le système capture les demandes de requête dans la table **AzureDiagnostics**. Comme prérequis, vous devez avoir déjà activé la [journalisation des ressources](search-monitor-logs.md), en spécifiant un espace de travail d’analytique des journaux ou une autre option de stockage.

1. Dans la section Surveillance, sélectionnez **Journaux** afin d'ouvrir une fenêtre de requête vide dans Log Analytics.

1. Exécutez l'expression suivante pour rechercher des opérations Query.Search. Vous obtiendrez un jeu de résultats, sous forme de tableau, avec le nom de l'opération, la chaîne de requête, l'index interrogé et le nombre de documents trouvés. Les deux dernières instructions excluent les chaînes de requêtes correspondant à une recherche vide ou non spécifiée, sur un exemple d'index, ce qui réduit le bruit dans vos résultats.

   ```
   AzureDiagnostics
   | project OperationName, Query_s, IndexName_s, Documents_d
   | where OperationName == "Query.Search"
   | where Query_s != "?api-version=2020-06-30&search=*"
   | where IndexName_s != "realestate-us-sample-index"
   ```

1. Vous pouvez également définir un filtre de colonne sur *Query_s* pour effectuer une recherche sur une syntaxe ou une chaîne spécifique. Par exemple, vous pouvez appliquer le filtre suivant : *est égal à* `?api-version=2020-06-30&search=*&%24filter=HotelName`).

   ![Chaînes de requêtes consignées](./media/search-monitor-usage/log-query-strings.png "Chaînes de requêtes consignées")

Cette technique fonctionne pour une investigation ad hoc, mais la création d'un rapport vous permettra de regrouper les chaînes de requêtes et de les présenter dans un format plus propice à l'analyse.

## <a name="identify-long-running-queries"></a>Identifier les requêtes longues

Ajoutez une colonne de durée pour obtenir les valeurs de toutes les requêtes, et pas seulement celles récupérées en tant que métriques. Le tri de ces données vous indique quelles requêtes prennent le plus de temps.

1. Dans la section Surveillance, sélectionnez **Journaux** pour rechercher des informations de journal.

1. Exécutez la requête suivante pour renvoyer des requêtes, triées par durée en millisecondes. Les requêtes longues figurent en haut.

   ```
   AzureDiagnostics
   | project OperationName, resultSignature_d, DurationMs, Query_s, Documents_d, IndexName_s
   | where OperationName == "Query.Search"
   | sort by DurationMs
   ```

   ![Trier les requêtes par durée](./media/search-monitor-usage/azurediagnostics-table-sortby-duration.png "Trier les requêtes par durée")

## <a name="create-a-metric-alert"></a>Créer une alerte de métrique

Une alerte de métrique fixe le seuil à partir duquel vous recevez une notification ou déclenchez une action corrective définie à l'avance. 

Dans un service de recherche, il est fréquent de créer une alerte de métrique pour la latence de recherche et les requêtes limitées. Si vous savez quand les requêtes sont abandonnées, vous pouvez rechercher des solutions pour réduire la charge ou augmenter la capacité. Par exemple, si les requêtes limitées augmentent pendant l'indexation, vous pouvez reporter celle-ci jusqu'à ce que l'activité de requête diminue.

Lorsque vous repoussez les limites d'une configuration réplica-partition particulière, il convient également de définir des alertes pour les seuils de volumes de requêtes (RPS).

1. Dans la section Surveillance, sélectionnez **Alertes**, puis cliquez sur **+ Nouvelle règle d'alerte**. Veillez à ce que votre service de recherche soit sélectionné comme ressource.

1. Sous Condition, cliquez sur **Ajouter**.

1. Configurez la logique du signal. Pour le type de signal, choisissez **métriques**, puis sélectionnez le signal.

1. Après avoir sélectionné le signal, vous pouvez utiliser un graphique afin de visualiser les données historiques et prendre une décision éclairée sur le mode de configuration des conditions.

1. Faites ensuite défiler jusqu'à Logique d'alerte. Pour la preuve de concept, vous pouvez spécifier une valeur artificiellement faible à des fins de test.

   ![Logique d'alerte](./media/search-monitor-usage/alert-logic-qps.png "Logique d'alerte")

1. Ensuite, spécifiez ou créez un groupe d'actions. Il s'agit de la réponse à appeler lorsque le seuil est atteint. Il peut s'agir d'une notification Push ou d'une réponse automatisée.

1. Enfin, spécifiez les détails de l'alerte. Nommez et décrivez l'alerte, attribuez une valeur de gravité et spécifiez si la règle doit être activée ou désactivée.

   ![Détails de l'alerte](./media/search-monitor-usage/alert-details.png "Détails de l’alerte")

Si vous avez spécifié une notification par e-mail, vous recevrez un e-mail de « Microsoft Azure » dont l'objet sera : « Azure : Gravité activée : 3 `<your rule name>` ».

<!-- ## Report query data

Power BI is an analytical reporting tool useful for visualizing data, including log information. If you are collecting data in Blob storage, a Power BI template makes it easy to spot anomalies or trends. Use this link to download the template. -->

## <a name="next-steps"></a>Étapes suivantes

Si ce n’est déjà fait, passez en revue les principes de base de la surveillance des services de recherche pour en savoir plus sur les différentes capacités de surveillance disponibles.

> [!div class="nextstepaction"]
> [Surveiller les opérations et l'activité dans Recherche cognitive Azure](search-monitor-usage.md)