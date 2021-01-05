---
title: Utiliser la parallélisation de requête et le scale-in dans Azure Stream Analytics
description: Cet article décrit comment mettre à l’échelle des tâches Stream Analytics en configurant des partitions d’entrée, en réglant la définition de requête et en configurant les unités de streaming d’un travail.
author: JSeb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: 326af3bc38ce70cc7cb205384bb4302c5ff73d28
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97704178"
---
# <a name="leverage-query-parallelization-in-azure-stream-analytics"></a>Profiter de la parallélisation de requête dans Azure Stream Analytics
Cet article explique comment tirer parti de la parallélisation dans Azure Stream Analytics. Vous découvrez comment mettre à l’échelle des travaux Stream Analytics en configurant des partitions d’entrée et en réglant la définition de requête Analytics.
Comme prérequis, vous pouvez vous familiariser avec la notion d’unité de streaming décrite dans [Comprendre et ajuster les unités de streaming](stream-analytics-streaming-unit-consumption.md).

## <a name="what-are-the-parts-of-a-stream-analytics-job"></a>Quelles sont les parties d’un travail Stream Analytics ?
La définition d’un travail Stream Analytics inclut au moins une entrée de streaming, une requête et une sortie. Les entrées correspondent à l’emplacement où le travail lit le flux de données. La requête permet de transformer le flux d’entrée de données, et la sortie correspond à l’emplacement où le travail envoie ses résultats.

## <a name="partitions-in-inputs-and-outputs"></a>Partitions dans les entrées et sorties
Le partitionnement vous permet de répartir les données en sous-ensembles basés sur une [clé de partition](../event-hubs/event-hubs-scalability.md#partitions). Si votre entrée (par exemple Event Hubs) est partitionnée par une clé, il est vivement recommandé de spécifier cette clé de partition lors de l’ajout d’une entrée à votre travail Stream Analytics. La mise à l’échelle d’un travail Stream Analytics tire parti des partitions dans l’entrée et la sortie. Un travail Stream Analytics peut consommer et écrire différentes partitions en parallèle, ce qui augmente le débit. 

### <a name="inputs"></a>Entrées
Toutes les entrées Azure Stream Analytics peuvent tirer parti du partitionnement :
-   EventHub (nécessité de définir la clé de partition explicitement avec le mot clé PARTITION BY si vous utilisez le niveau de compatibilité 1.1 ou un niveau inférieur)
-   IoT Hub (nécessité de définir la clé de partition explicitement avec le mot clé PARTITION BY si vous utilisez le niveau de compatibilité 1.1 ou un niveau inférieur)
-   Stockage d'objets blob

### <a name="outputs"></a>Sorties

Quand vous utilisez Stream Analytics, vous pouvez tirer parti du partitionnement dans les sorties :
-   Azure Data Lake Storage
-   Azure Functions
-   table Azure
-   Stockage Blob (possibilité de définir la clé de partition explicitement)
-   Cosmos DB (définir la clé de partition explicitement)
-   Event Hubs (définir la clé de partition explicitement)
-   IoT Hub (nécessité de définir la clé de partition explicitement)
-   Service Bus
- SQL et Azure Synapse Analytics avec partitionnement facultatif : consultez plus d’informations sur la [page Sortie dans Azure SQL Database](./stream-analytics-sql-output-perf.md).

Power BI ne prend pas en charge le partitionnement. Toutefois, vous pouvez toujours partitionner l’entrée comme décrit dans [cette section](#multi-step-query-with-different-partition-by-values) 

Pour plus d’informations sur les partitions, consultez les articles suivants :

* [Vue d’ensemble des fonctionnalités des concentrateurs d’événements](../event-hubs/event-hubs-features.md#partitions)
* [Partitionnement des données](/azure/architecture/best-practices/data-partitioning)


## <a name="embarrassingly-parallel-jobs"></a>Travaux massivement parallèles
Un travail *massivement parallèle* est le scénario le plus évolutif d’Azure Stream Analytics. Elle permet de connecter une partition de l’entrée à une instance de la requête, puis de connecter celle-ci à une partition de la sortie. Ce parallélisme comporte les exigences suivantes :

1. Si votre logique de requête dépend de la clé qui est actuellement traitée par la même instance de requête, vous devez vous assurer que les événements atteignent la même partition de votre entrée. Pour Event Hubs ou IoT Hub, cela signifie que vous devez définir la valeur de **PartitionKey** pour les données d’événement. Par ailleurs, vous pouvez utiliser des expéditeurs partitionnés. Pour le stockage d’objets blob, cela signifie que les événements sont envoyés vers le même dossier de partition. Il peut s’agir, par exemple, d’une instance de requête qui agrège les données par userID, le hub d’événements d’entrée étant partitionné à l’aide de l’identificateur userID comme clé de partition. Toutefois, si votre logique de requête ne demande pas la même clé pour être traitée par la même instance de requête, vous pouvez ignorer cette exigence. Un exemple de cette logique serait une requête simple du type select/project/filter.  

2. L’étape suivante consiste à partitionner votre requête. Pour les travaux dont le niveau de compatibilité est supérieur ou égal à 1.2 (recommandé), vous pouvez spécifier une colonne personnalisée en tant que Clé de partition dans les paramètres d’entrée, afin que le travail soit automatiquement exécuté en parallèle. Pour les travaux avec un niveau de compatibilité 1.0 ou 1.1, vous devez utiliser **PARTITION BY PartitionId** dans toutes les étapes de votre requête. Les étapes multiples sont autorisées, mais elles doivent être partitionnées à l’aide de la même clé. 

3. La plupart des sorties prises en charge dans Stream Analytics peuvent tirer parti du partitionnement. Si vous utilisez un type de sortie qui ne prend pas en charge le partitionnement, votre travail ne sera pas *massivement parallèle*. Pour les sorties du hub d’événements, vérifiez que la **colonne de clé de partition** est définie sur la même clé de partition que celle utilisée dans la requête. Reportez-vous à la [section relative aux sorties](#outputs) pour plus d’informations.

4. Le nombre de partitions d’entrée doit être égal à celui des partitions de sortie. La sortie du Stockage Blob peut prendre en charge les partitions et hériter du schéma de partitionnement de la requête en amont. Lorsqu’une clé de partition du Stockage Blob est spécifiée, les données sont partitionnées par partition d’entrée ; le résultat reste donc entièrement parallèle. Voici des exemples de valeurs de partition qui permettent la création d’un travail entièrement parallèle :

   * 8 partitions d’entrée de concentrateur Event Hub et 8 partitions de sortie de concentrateur Event Hub
   * 8 partitions d’entrée de concentrateur Event Hub et une sortie de stockage d’objets blob
   * 8 partitions d’entrée Event Hub et une sortie de Stockage Blob partitionnée par un champ personnalisé avec cardinalité arbitraire
   * 8 partitions d’entrée de stockage d’objets blob et une sortie de stockage d’objets blob
   * 8 partitions d’entrée de stockage d’objets blob et 8 partitions de sortie de concentrateur Event Hub

Les sections ci-après présentent quelques exemples de parallélisme massif.

### <a name="simple-query"></a>Requête simple

* Entrée : concentrateur Event Hub avec 8 partitions
* Sortie : le hub d’événements à huit partitions (« Colonne de clé de partition » doit être défini de façon à utiliser « PartitionId »)

Requête :

```SQL
    --Using compatibility level 1.2 or above
    SELECT TollBoothId
    FROM Input1
    WHERE TollBoothId > 100
    
    --Using compatibility level 1.0 or 1.1
    SELECT TollBoothId
    FROM Input1 PARTITION BY PartitionId
    WHERE TollBoothId > 100
```

Cette requête est un filtre simple. Par conséquent, nous n’avons pas à nous préoccuper du partitionnement de l’entrée qui est envoyée au concentrateur Event Hub. Notez que les travaux dont le niveau de compatibilité est inférieur à 1.2 doivent inclure la clause **PARTITION BY PartitionId** afin de répondre à l’exigence n°2 précitée. Pour la sortie, nous devons configurer la sortie du hub d’événements dans le travail afin que la clé de partition ait la valeur **PartitionId**. La dernière vérification consiste à s’assurer que le nombre de partitions d’entrée est égal au nombre de partitions de sortie.

### <a name="query-with-a-grouping-key"></a>Requête avec clé de regroupement

* Entrée : concentrateur Event Hub avec 8 partitions
* Sortie : Stockage d'objets blob

Requête :

```SQL
    --Using compatibility level 1.2 or above
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1
    GROUP BY TumblingWindow(minute, 3), TollBoothId
    
    --Using compatibility level 1.0 or 1.1
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
```

Cette requête comporte une clé de regroupement. Par conséquent, les événements regroupés doivent être envoyés à la même partition Event Hub. Étant donné que, dans cet exemple, le regroupement est effectué selon TollBoothID, nous devons vérifier que TollBoothID est utilisé comme clé de partition quand les événements sont envoyés à la partition Event Hub. Ensuite, dans ASA, nous pouvons utiliser **PARTITION BY PartitionId** pour hériter de ce schéma de partition et activer la parallélisation complète. Étant donné que la sortie est un stockage d’objets blob, nous n’avons pas à nous soucier de la configuration d’une valeur de clé de partition, conformément à l’exigence n°4.

## <a name="example-of-scenarios-that-are-not-embarrassingly-parallel"></a>Exemples de scénarios qui n’impliquent *pas* un parallélisme massif

Dans la section précédente, nous vous avons présenté certains scénarios impliquant un parallélisme massif. Dans cette section, nous étudions des cas dans lesquels toutes les exigences ne sont pas remplies pour un parallélisme massif. 

### <a name="mismatched-partition-count"></a>Nombre de partitions d’entrée et de sortie différent
* Entrée : concentrateur Event Hub avec 8 partitions
* Sortie : concentrateur Event Hub avec 32 partitions

Si le nombre de partitions en entrée ne correspond pas au nombre de partitions en sortie, la topologie n’est pas massivement parallèle, quelle que soit la requête. Toutefois, nous pouvons tout de même obtenir un certain niveau ou une certaine parallélisation.

### <a name="query-using-non-partitioned-output"></a>Requête avec une sortie non partitionnée
* Entrée : concentrateur Event Hub avec 8 partitions
* Sortie : Power BI

Pour le moment, la sortie Power BI ne prend pas en charge le partitionnement. Par conséquent, ce scénario n’est pas de type massivement parallèle.

### <a name="multi-step-query-with-different-partition-by-values"></a>Requête à plusieurs étapes avec différentes valeurs PARTITION BY
* Entrée : concentrateur Event Hub avec 8 partitions
* Sortie : concentrateur Event Hub avec 8 partitions
* Niveau de compatibilité : 1.0 ou 1.1

Requête :

```SQL
    WITH Step1 AS (
    SELECT COUNT(*) AS Count, TollBoothId, PartitionId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1 Partition By TollBoothId
    GROUP BY TumblingWindow(minute, 3), TollBoothId
```

Comme vous pouvez le voir, la deuxième étape utilise **TollBoothId** comme clé de partitionnement. Cette étape n’est pas la même que la première. Nous devons donc apporter quelques modifications. 

### <a name="multi-step-query-with-different-partition-by-values"></a>Requête à plusieurs étapes avec différentes valeurs PARTITION BY
* Entrée : concentrateur Event Hub avec 8 partitions
* Sortie : le hub d’événements à huit partitions (« Colonne de clé de partition » doit être défini de façon à utiliser « TollBoothId »)
* Niveau de compatibilité : 1.2 ou supérieur

Requête :

```SQL
    WITH Step1 AS (
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1
    GROUP BY TumblingWindow(minute, 3), TollBoothId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute, 3), TollBoothId
```

le niveau de compatibilité 1.2 ou supérieur permet l’exécution de requête en parallèle par défaut. Par exemple, la requête de la section précédente est partitionnée tant que la colonne « TollBooth If » est définie en tant que clé de partition d’entrée. La clause PARTITION BY PartitionId n’est pas obligatoire.

## <a name="calculate-the-maximum-streaming-units-of-a-job"></a>Calcul du nombre maximum d'unités de diffusion en continu pour un travail
Le nombre total d'unités de diffusion en continu qui peut être utilisé par un travail Stream Analytics varie selon le nombre d'étapes de la requête définie pour le travail et le nombre de partitions pour chaque étape.

### <a name="steps-in-a-query"></a>Étapes dans une requête
Une requête peut avoir une ou plusieurs étapes. Chaque étape est une sous-requête définie par le mot-clé **WITH**. La requête qui se trouve en dehors du mot-clé **WITH** (une seule requête) est également comptabilisée comme une étape, par exemple, l’instruction **SELECT** dans la requête suivante :

Requête :

```SQL
    WITH Step1 AS (
        SELECT COUNT(*) AS Count, TollBoothId
        FROM Input1 Partition By PartitionId
        GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )
    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute,3), TollBoothId
```

Cette requête compte deux étapes.

> [!NOTE]
> Cette requête est approfondie plus loin dans cet article.
>  

### <a name="partition-a-step"></a>Partitionnement d'une étape
Les conditions suivantes doivent être respectées pour procéder au partitionnement d'une étape :

* La source d'entrée doit être partitionnée. 
* L’instruction **SELECT** de la requête doit lire à partir d’une source d’entrée partitionnée.
* La requête de l’étape doit contenir le mot clé **PARTITION BY**.

Lorsqu’une requête est partitionnée, les événements d’entrée sont traités et agrégés dans des groupes de partition distincts, et les événements de sortie sont générés pour chacun des groupes. Si vous souhaitez procéder à un agrégat combiné, vous devez créer une deuxième étape non partitionnée à agréger.

### <a name="calculate-the-max-streaming-units-for-a-job"></a>Calcul des unités de diffusion en continu maximum pour un travail
Toutes les étapes non partitionnées ensemble peuvent être mises à l’échelle jusqu’à atteindre six unités SU par travail Stream Analytics. En outre, vous pouvez ajouter 6 unités de streaming pour chaque partition dans une étape partitionnée.
Vous pouvez voir quelques **exemples** dans le tableau ci-dessous.

| Requête                                               | Nombre d’unités SU maximal pour le travail |
| --------------------------------------------------- | ------------------- |
| <ul><li>La requête contient une étape.</li><li>L'étape n'est pas partitionnée.</li></ul> | 6 |
| <ul><li>Le flux de données d’entrée est partitionné par 16.</li><li>La requête contient une étape.</li><li>L'étape est partitionnée.</li></ul> | 96 (6 * 16 partitions) |
| <ul><li>La requête contient 2 étapes.</li><li>Aucune des étapes n'est partitionnée.</li></ul> | 6 |
| <ul><li>Le flux de données d'entrée est partitionné par 3.</li><li>La requête contient 2 étapes. L'étape d'entrée est partitionnée et la deuxième étape ne l'est pas.</li><li>L’instruction <strong>SELECT</strong> lit dans l’entrée partitionnée.</li></ul> | 24 (18 pour les étapes partitionnées + 6 pour les étapes non partitionnées) |

### <a name="examples-of-scaling"></a>Exemples de mise à l’échelle

La requête suivante calcule le nombre de voitures, dans une fenêtre de trois minutes, qui traversent un poste de péage pourvu de trois cabines de péage. Cette requête peut être mise à l’échelle jusqu’à six unités SU.

```SQL
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
```

Pour utiliser plus d’unités SU pour la requête, le flux de données d’entrée et la requête doivent être partitionnés. Comme la partition de flux de données est définie sur 3, la requête modifiée suivante peut être mise à l’échelle jusqu’à compter 18 unités SU :

```SQL
    SELECT COUNT(*) AS Count, TollBoothId
    FROM Input1 Partition By PartitionId
    GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
```

Lorsqu'une requête est partitionnée, les événements d'entrée sont traités et agrégées dans des groupes de partition distincts. Les événements de sortie sont également générés pour chacun des groupes. Le partitionnement peut provoquer des résultats inattendus si le champ **GROUP BY** n’est pas la clé de partition dans le flux de données d’entrée. Par exemple, dans la requête précédente, le champ **TollBoothId** n’est pas la clé de partition **d’Input1**. Le résultat est le suivant : les données de la cabine de péage « TollBooth 1 » peuvent être réparties dans plusieurs partitions.

Chaque partition **Input1** est traitée séparément par Stream Analytics. En conséquence, plusieurs enregistrements du nombre de voitures pour la même cabine de péage sont créés dans la même fenêtre bascule. Si la clé de partition d’entrée ne peut pas être modifiée, ce problème peut être résolu en ajoutant une étape non partitionnée pour agréger des valeurs sur plusieurs partitions, comme dans l’exemple suivant :

```SQL
    WITH Step1 AS (
        SELECT COUNT(*) AS Count, TollBoothId
        FROM Input1 Partition By PartitionId
        GROUP BY TumblingWindow(minute, 3), TollBoothId, PartitionId
    )

    SELECT SUM(Count) AS Count, TollBoothId
    FROM Step1
    GROUP BY TumblingWindow(minute, 3), TollBoothId
```

Cette requête peut être mise à l’échelle jusqu’à comporter 24 unités SU.

> [!NOTE]
> Si vous joignez deux flux de données, assurez-vous qu’ils sont partitionnés par la clé de partition de la colonne que vous utilisez pour créer les jointures. Assurez-vous également d’avoir le même nombre de partitions dans les deux flux de données.
> 
> 

## <a name="achieving-higher-throughputs-at-scale"></a>Obtention de débits supérieurs à grande échelle

Un [travail massivement en parallèle](#embarrassingly-parallel-jobs) est nécessaire mais insuffisant pour maintenir un débit plus élevé à grande échelle. Chaque système de stockage et sa sortie Stream Analytics correspondante varient quant à la manière d’obtenir le meilleur débit possible en écriture. Comme dans tout scénario à grande échelle, il est possible de résoudre certains problèmes en utilisant les configurations appropriées. Cette section présente les configurations pour obtenir quelques sorties communes et fournit des exemples permettant de maintenir des taux d’ingestion de 1 000, 5 000 et 10 000 événements par seconde.

Les observations suivantes utilisent un travail Stream Analytics avec une requête sans état (passthrough), un UDF JavaScript de base qui écrit dans Event Hub, Azure SQL DB ou Cosmos DB.

#### <a name="event-hub"></a>Event Hub

|Taux d’ingestion (événements par seconde) | Unités de diffusion en continu | Ressources de sortie  |
|--------|---------|---------|
| 1 000     |    1    |  2 TU   |
| 5 000     |    6    |  6 TU   |
| 10 000    |    12   |  10 TU  |

La solution [Event Hub](https://github.com/Azure-Samples/streaming-at-scale/tree/main/eventhubs-streamanalytics-eventhubs) met à l’échelle de manière linéaire en termes d’unités de streaming et de débit, ce qui en fait le moyen le plus efficace et le plus performant d’analyser et de diffuser des données à partir de Stream Analytics. Les travaux peuvent être mis à l’échelle jusqu’à 192 unités de streaming, ce qui correspond approximativement au traitement de 200 Mo/s, soit 19 billions d’événements par jour.

#### <a name="azure-sql"></a>Azure SQL
|Taux d’ingestion (événements par seconde) | Unités de diffusion en continu | Ressources de sortie  |
|---------|------|-------|
|    1 000   |   3  |  S3   |
|    5 000   |   18 |  P4   |
|    10 000  |   36 |  P6   |

[SQL Azure](https://github.com/Azure-Samples/streaming-at-scale/tree/main/eventhubs-streamanalytics-azuresql) prend en charge l’écriture en parallèle, appelée Inherit Partitioning, qui n’est pas activée par défaut. Toutefois, l’activation de la fonctionnalité Inherit Partitioning avec une requête entièrement parallèle peut ne pas suffire pour atteindre des débits supérieurs. Les débits en écriture SQL dépendent considérablement de la configuration et du schéma de table de votre base de données. L’article [Performances en sortie SQL](./stream-analytics-sql-output-perf.md) contient des informations plus détaillées sur les paramètres susceptibles d’optimiser votre débit en écriture. Comme indiqué dans l’article [Sortie d’Azure Stream Analytics dans Azure SQL Database](./stream-analytics-sql-output-perf.md#azure-stream-analytics), cette solution n’est pas mise à l’échelle de manière linéaire en tant que pipeline entièrement parallèle au-delà de 8 partitions, et peut nécessiter un repartitionnement avant la sortie SQL (voir [INTO](/stream-analytics-query/into-azure-stream-analytics#into-shard-count)). Des références (SKU) Premium sont nécessaires pour prendre en charge des taux d’E/S élevés, ainsi que la surcharge liée aux sauvegardes de fichiers journaux toutes les quelques minutes.

#### <a name="cosmos-db"></a>Cosmos DB
|Taux d’ingestion (événements par seconde) | Unités de diffusion en continu | Ressources de sortie  |
|-------|-------|---------|
|  1 000   |  3    | 20 000 unités de requête  |
|  5 000   |  24   | 60 000 unités de requête  |
|  10 000  |  48   | 120 000 unités de requête |

La sortie de [Cosmos DB](https://github.com/Azure-Samples/streaming-at-scale/tree/main/eventhubs-streamanalytics-cosmosdb) à partir de Stream Analytics a été mise à jour pour utiliser une intégration native sous le [niveau de compatibilité 1.2](./stream-analytics-documentdb-output.md#improved-throughput-with-compatibility-level-12). Le niveau de compatibilité 1.2 permet un débit sensiblement supérieur, et réduit la consommation d’unités de requête par rapport au niveau 1.1 qui est le niveau de compatibilité par défaut pour les nouveaux travaux. La solution utilise des conteneurs CosmosDB partitionnés sur /deviceId et le reste de la solution est configuré de manière identique.

Tous les [exemples Azure de diffusion en continu à grande échelle](https://github.com/Azure-Samples/streaming-at-scale) utilisent un Event Hub alimenté par des clients de test simulant une charge. Chaque événement en entrée est un document JSON de 1 Ko, qui traduit facilement les taux d’ingestion configurés en débits (1 Mo/s, 5 Mo/s et 10 Mo/s). Les événements simulent un appareil IoT envoyant les données JSON suivantes (sous une forme abrégée) pour jusqu’à 1 000 appareils :

```
{
    "eventId": "b81d241f-5187-40b0-ab2a-940faf9757c0",
    "complexData": {
        "moreData0": 51.3068118685458,
        "moreData22": 45.34076957651598
    },
    "value": 49.02278128887753,
    "deviceId": "contoso://device-id-1554",
    "type": "CO2",
    "createdAt": "2019-05-16T17:16:40.000003Z"
}
```

> [!NOTE]
> Les configurations sont sujettes à modification en raison des divers composants utilisés dans la solution. Pour une estimation plus précise, personnalisez les échantillons en fonction de votre scénario.

### <a name="identifying-bottlenecks"></a>Identification des goulots d’étranglement

Utilisez le volet Métriques de votre travail Azure Stream Analytics pour identifier les goulots d’étranglement de votre pipeline. Examinez les **événements d’entrée/sortie** pour le débit, ainsi que le [ « Délai en filigrane »](https://azure.microsoft.com/blog/new-metric-in-azure-stream-analytics-tracks-latency-of-your-streaming-pipeline/) ou les **Événements en backlog**, pour voir si le travail suit la vitesse d’entrée. Pour les métriques Event Hub, recherchez les **Demandes limitées** et ajustez les Unités de seuil en conséquence. Pour les métriques de Cosmos DB, examinez la valeur **Nombre maximal de RU/s consommées par groupe de clés de partition** sous Débit pour vous assurer que les groupes de clés de partition sont consommés de manière uniforme. Pour Azure SQL DB, surveillez **E/S journal** et **UC**.

## <a name="get-help"></a>Obtenir de l’aide

Pour obtenir de l’aide supplémentaire, essayez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)

<!--Image references-->

[img.stream.analytics.monitor.job]: ./media/stream-analytics-scale-jobs/StreamAnalytics.job.monitor-NewPortal.png
[img.stream.analytics.configure.scale]: ./media/stream-analytics-scale-jobs/StreamAnalytics.configure.scale.png
[img.stream.analytics.perfgraph]: ./media/stream-analytics-scale-jobs/perf.png
[img.stream.analytics.streaming.units.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsStreamingUnitsExample.jpg
[img.stream.analytics.preview.portal.settings.scale]: ./media/stream-analytics-scale-jobs/StreamAnalyticsPreviewPortalJobSettings-NewPortal.png   

<!--Link references-->

[microsoft.support]: https://support.microsoft.com
[azure.event.hubs.developer.guide]: /previous-versions/azure/dn789972(v=azure.100)

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/