---
title: Sorties d’Azure Stream Analytics
description: Cet article décrit les options de sortie des données disponibles pour Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.custom: contperf-fy21q1
ms.date: 12/9/2020
ms.openlocfilehash: c11e0702a7825b719417288c5ebbf93defde186c
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029242"
---
# <a name="outputs-from-azure-stream-analytics"></a>Sorties d’Azure Stream Analytics

Un travail Azure Stream Analytics se compose d’une entrée, d’une requête et d’une sortie. Plusieurs types de sorties vous permettent d’envoyer des données transformées. Cet article répertorie les sorties de Stream Analytics prises en charge. Lorsque vous concevez votre requête Stream Analytics, faites référence au nom de la sortie à l’aide de la [clause INTO](/stream-analytics-query/into-azure-stream-analytics). Vous pouvez utiliser une seule sortie par travail ou, si nécessaire, plusieurs sorties par travail de diffusion en continu en ajoutant plusieurs clauses INTO à la requête.

Pour créer, modifier et tester des sorties de travaux Stream Analytics, vous pouvez utiliser le [portail Microsoft Azure](stream-analytics-quick-create-portal.md#configure-job-output), [Azure PowerShell](stream-analytics-quick-create-powershell.md#configure-output-to-the-job), [l’API .NET](/dotnet/api/microsoft.azure.management.streamanalytics.ioutputsoperations), [l’API REST](/rest/api/streamanalytics/) et [Visual Studio](stream-analytics-quick-create-vs.md).

Certains types de sorties prennent en charge le [partitionnement](#partitioning), et les [tailles de lots de sortie](#output-batch-size) varient afin d’optimiser le débit. Le tableau suivant répertorie les fonctionnalités prises en charge pour chaque type de sortie.

| Type de sortie | Partitionnement | Sécurité | 
|-------------|--------------|----------|
|[Azure Data Lake Storage Gen 1](azure-data-lake-storage-gen1-output.md)|Oui|Utilisateur Azure Active Directory </br> , Identité managée|
|[Azure SQL Database](sql-database-output.md)|Oui, facultatif.|Authentification utilisateur SQL, </br> Identité managée (préversion)|
|[Azure Synapse Analytics](azure-synapse-analytics-output.md)|Oui|Authentification utilisateur SQL, </br> Identité managée (préversion)|
|[Stockage Blob et Azure Data Lake Gen 2](blob-storage-azure-data-lake-gen2-output.md)|Oui|Clé d’accès, </br> Identité managée (préversion)|
|[Azure Event Hubs](event-hubs-output.md)|Oui, vous devez définir la colonne clé de partition dans la configuration de sortie.|Clé d’accès, </br> Identité managée (préversion)|
|[Power BI](power-bi-output.md)|Non|Utilisateur Azure Active Directory, </br> Identité managée|
|[Stockage Table Azure](table-storage-output.md)|Oui|Clé de compte|
|[Files d’attente Azure Service Bus](service-bus-queues-output.md)|Oui|Clé d’accès|
|[Rubriques Azure Service Bus](service-bus-topics-output.md)|Oui|Clé d’accès|
|[Azure Cosmos DB](azure-cosmos-db-output.md)|Oui|Clé d’accès|
|[Azure Functions](azure-functions-output.md)|Oui|Clé d’accès|

## <a name="partitioning"></a>Partitionnement

Stream Analytics prend en charge les partitions pour toutes les sorties, à l’exception de Power BI. Pour plus d’informations sur les clés de partition et le nombre de générateurs de sortie, consultez l’article correspondant au type de sortie spécifique qui vous intéresse. Tous les articles de sortie sont liés dans la section précédente.  

De plus, pour un réglage avancé des partitions, vous pouvez contrôler le nombre d’enregistreurs de sortie à l’aide d’une clause `INTO <partition count>` (voir [INTO](/stream-analytics-query/into-azure-stream-analytics#into-shard-count)) dans votre requête, ce qui peut ête utile pour atteindre la topologie de travail souhaitée. Si votre adaptateur de sortie n'est pas partitionné, l'absence de données dans une partition d'entrée entraîne un retard pouvant aller jusqu'au délai d'arrivée tardive. Dans ce cas, la sortie est fusionnée dans un seul enregistreur, ce qui peut créer des goulots d’étranglement dans votre pipeline. Pour en savoir plus sur la stratégie d’arrivée tardive, consultez l’article [Considérations relatives à l’ordre des événements Azure Stream Analytics](./stream-analytics-time-handling.md).

## <a name="output-batch-size"></a>Taille de lot de sortie

Toutes les sorties prennent en charge le traitement par lot, mais seule une certaine taille de lot de prise en charge est explicite. Azure Stream Analytics utilise des lots de taille variable pour traiter les événements et générer les sorties. En général, le moteur Stream Analytics n’écrit pas les messages un par un ; il utilise des lots pour plus d’efficacité. Lorsque le taux des événements entrants et sortants est élevé, Stream Analytics utilise des lots plus volumineux. Lorsque le taux de sortie est faible, il utilise des lots plus petits pour maintenir une faible latence.

## <a name="parquet-output-batching-window-properties"></a>Propriétés de la fenêtre de traitement par lot de la sortie Parquet

Lorsque vous utilisez le déploiement de modèle Azure Resource Manager ou l’API REST, les deux propriétés de la fenêtre de traitement par lot sont les suivantes :

1. *timeWindow*

   Le délai d’attente maximum par lot. La valeur doit être une chaîne TimeSpan. Par exemple, « 00:02:00 » pour deux minutes. À l’issue de cette période, le lot est écrit dans la sortie même si l’exigence de lignes minimum n’est pas remplie. La valeur par défaut est de 1 minute et la valeur maximum autorisée est de 2 heures. Si la sortie de votre objet BLOB a une fréquence de modèle de chemin d’accès, le délai d’attente ne peut pas être supérieur à l’intervalle de temps de la partition.

2. *sizeWindow*

   Le nombre minimum de lignes par lot. Pour Parquet, chaque lot crée un fichier. La valeur par défaut actuelle est de 2 000 lignes et la valeur maximum autorisée est de 10 000 lignes.

Ces propriétés de fenêtre de traitement par lot sont uniquement prises en charge par la version d’API **2017-04-01-preview**. Voici un exemple de la charge utile JSON pour un appel d’API REST :

```json
"type": "stream",
      "serialization": {
        "type": "Parquet",
        "properties": {}
      },
      "timeWindow": "00:02:00",
      "sizeWindow": "2000",
      "datasource": {
        "type": "Microsoft.Storage/Blob",
        "properties": {
          "storageAccounts" : [
          {
            "accountName": "{accountName}",
            "accountKey": "{accountKey}",
          }
          ],
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>
> [Démarrage rapide : Créer un travail Stream Analytics à l’aide du portail Azure](stream-analytics-quick-create-portal.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/
