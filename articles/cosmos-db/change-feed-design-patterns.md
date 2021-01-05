---
title: Modèles de conception des flux de modification dans Azure Cosmos DB
description: Vue d’ensemble des modèles de conception des flux de modification courants
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 04/08/2020
ms.openlocfilehash: 443d00e61e593daacca04a4451b90bb78cc7d854
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334581"
---
# <a name="change-feed-design-patterns-in-azure-cosmos-db"></a>Modèles de conception des flux de modification dans Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Le flux de modification Azure Cosmos DB permet de traiter efficacement les grands jeux de données avec un volume important d’écritures. Le flux de modification offre également une alternative à l’interrogation d’un jeu de données complet dans le but d’y identifier les changements. Ce document est axé sur les modèles de conception de flux de modification courants, les compromis de conception et les limitations des flux de modification.

Azure Cosmos DB est particulièrement bien adapté aux applications d’IoT, de jeux, de vente au détail et de journal des opérations. Ces applications intègrent souvent un modèle de conception qui consiste à utiliser des modifications de données pour déclencher des actions supplémentaires. Voici quelques exemples d’actions supplémentaires :

* Déclenchement d’une notification ou d’un appel d’API lorsqu’un élément est inséré ou modifié
* Traitement du flux de données en temps réel pour l’IoT ou traitement d’analyse des données opérationnelles en temps réel
* Déplacement de données tels que la synchronisation avec un cache, un moteur de recherche, un entrepôt de données ou un stockage à froid

Le flux de modification Azure Cosmos DB vous permet de créer des solutions efficaces et scalables pour chacun de ces modèles, comme illustré dans l’image suivante :

:::image type="content" source="./media/change-feed/changefeedoverview.png" alt-text="Utilisation du flux de modification d’Azure Cosmos DB pour bénéficier de l’analytique en temps réel et de calculs pilotés par les événements" border="false":::

## <a name="event-computing-and-notifications"></a>Calcul d’événements et notifications

Le flux de modification Azure Cosmos DB peut simplifier les scénarios qui doivent déclencher l’envoi d’une notification ou un appel à une API en fonction d’un certain événement. Vous pouvez utiliser la [bibliothèque de processeur de flux de modification](change-feed-processor.md) pour interroger automatiquement votre conteneur afin de connaître les modifications apportées, et appeler une API externe lors de chaque opération d’écriture ou de mise à jour.

Vous pouvez également déclencher de manière sélective une notification ou envoyer un appel à une API en fonction de critères spécifiques. Par exemple, si vous lisez à partir du flux de modification à l’aide d’[Azure Functions](change-feed-functions.md), vous pouvez placer une logique dans la fonction afin d’envoyer une notification uniquement si un critère spécifique a été satisfait. Le code Azure Functions s’exécutera pendant chaque écriture et mise à jour, mais la notification sera envoyée uniquement si des critères spécifiques ont été satisfaits.

## <a name="real-time-stream-processing"></a>Traitement du flux en temps réel

Le flux de modification Azure Cosmos DB peut être utilisé pour le traitement de flux en temps réel en vue du traitement de l’analytique en temps réel ou IoT sur des données opérationnelles.
Par exemple, vous pouvez recevoir et stocker des données d’événement à partir d’appareils, de capteurs, d’infrastructures et d’applications, et traiter ces événements en temps réel à l’aide de [Spark](../hdinsight/spark/apache-spark-overview.md). L’illustration suivante montre comment implémenter une architecture lambda à l’aide d’Azure Cosmos DB via le flux de modification :

:::image type="content" source="./media/change-feed/lambda.png" alt-text="Pipeline lambda Azure Cosmos DB pour l’ingestion et l’interrogation" border="false":::

Dans de nombreux cas, les implémentations de traitement de flux reçoivent d’abord un volume élevé de données entrantes dans une file d’attente de messages temporaire comme Azure Event Hub ou Apache Kafka. Le flux de modification est une alternative intéressante en raison de la capacité d’Azure Cosmos DB à prendre en charge un taux élevé et soutenu d’ingestion de données avec une latence de lecture et d’écriture faible et garantie. Les avantages du flux de modification Azure Cosmos DB par rapport à une file d’attente de messages sont les suivants :

### <a name="data-persistence"></a>Persistance des données

Les données écrites dans Azure Cosmos DB apparaissent dans le flux de modification et sont conservées jusqu’à leur suppression. Les files d’attente de messages ont généralement une durée de conservation maximale. Par exemple, [Azure Event Hub](https://azure.microsoft.com/services/event-hubs/) offre une durée de conservation des données maximale de 90 jours.

### <a name="querying-ability"></a>Capacité d’interrogation

En plus de lire à partir du flux de modification d’un conteneur Cosmos, vous pouvez exécuter des requêtes SQL sur les données stockées dans Azure Cosmos DB. Le flux de modification n’est pas une duplication des données qui se trouvent déjà dans le conteneur, mais plutôt un mécanisme de lecture des données différent. Par conséquent, si vous lisez des données à partir du flux de modification, elles sont toujours cohérentes avec les requêtes du même conteneur Azure Cosmos DB.

### <a name="high-availability"></a>Haute disponibilité

Azure Cosmos DB offre jusqu’à 99,999 % de disponibilité en lecture et en écriture. Contrairement à de nombreuses files d’attente de messages, il est facile de configurer et de distribuer les données Azure Cosmos DB mondialement avec un [objectif de temps de récupération (RTO)](./consistency-levels.md#rto) de zéro.

Après avoir traité des éléments dans le flux de modification, vous pouvez créer une vue matérialisée et conserver les valeurs agrégées dans Azure Cosmos DB. Si vous utilisez Azure Cosmos DB pour créer un jeu, vous pouvez, par exemple, utiliser le flux de modification pour implémenter des classements en temps réel basés sur des scores de jeux terminés.

## <a name="data-movement"></a>Déplacement des données

Vous pouvez également lire le flux de modification pour le déplacement des données en temps réel.

Par exemple, le flux de modification vous aide à effectuer efficacement les tâches suivantes :

* Mettre à jour un cache, un index de recherche ou un entrepôt de données avec des données stockées dans Azure Cosmos DB

* Effectuer des migrations sans aucun temps d’arrêt vers un autre compte ou un autre conteneur Azure Cosmos avec une clé de partition logique différente

* Implémenter une hiérarchisation et un archivage des données au niveau de l’application. Par exemple, vous pouvez stocker des « données à chaud » dans Azure Cosmos DB et vieillir les « données à froid » vers d’autres systèmes de stockage, tels que [Stockage Blob Azure](../storage/common/storage-introduction.md)

Quand vous devez [dénormaliser des données dans des partitions et des conteneurs](how-to-model-partition-example.md#v2-introducing-denormalization-to-optimize-read-queries
), vous pouvez lire le flux de modification de votre conteneur en tant que source pour cette réplication de données. La réplication des données en temps réel avec le flux de modification peut uniquement garantir la cohérence finale. Vous pouvez [superviser le retard du processeur de flux de modification](how-to-use-change-feed-estimator.md) dans le traitement des modifications dans votre conteneur Cosmos.

## <a name="event-sourcing"></a>Provisionnement en événements

Le [modèle de provisionnement en événements](/azure/architecture/patterns/event-sourcing) implique l’utilisation d’un magasin avec ajout uniquement pour enregistrer la série complète d’actions sur ces données. Le flux de modification d’Azure Cosmos DB est un bon choix de magasin de données central dans les architectures de provisionnement en événements où toute l’ingestion des données est modélisée en tant qu’écritures (aucune mise à jour ou suppression). Dans ce cas, chaque écriture dans Azure Cosmos DB est un « événement », et vous aurez un enregistrement complet des événements passés dans le flux de modification. Les utilisations les plus courantes des événements publiés par le magasin d’événements central concernent la gestion des vues matérialisées ou l’intégration à des systèmes externes. Étant donné qu’il n’existe pas de limite de temps pour la conservation dans le flux de modification, vous pouvez relire tous les événements passés en lisant à partir du début du flux de modification de votre conteneur Cosmos.

Vous pouvez faire en sorte que [plusieurs consommateurs de flux de modification s’abonnent au flux de modification du même conteneur](how-to-create-multiple-cosmos-db-triggers.md#optimizing-containers-for-multiple-triggers). Hormis le débit provisionné du [conteneur de baux](change-feed-processor.md#components-of-the-change-feed-processor), il n’y a aucun coût d’utilisation associé au flux de modification. Le flux de modification est disponible dans chaque conteneur, qu’il soit utilisé ou non.

Azure Cosmos DB est un excellent magasin de données avec ajout uniquement, persistant et centralisé dans le modèle de provisionnement en événements en raison de ses atouts en matière de scalabilité horizontale et de haute disponibilité. En outre, la bibliothèque du processeur de flux de modification offre une garantie [« au moins une fois »](change-feed-processor.md#error-handling), qui permet d’être certain que tous les événements seront traités.

## <a name="current-limitations"></a>Limites actuelles

Le flux de modification présente des limitations importantes que vous devez comprendre. Même si les éléments d’un conteneur Cosmos seront toujours conservés dans le flux de modification, celui-ci n’est pas un journal des opérations complet. Il y a des points importants à prendre en compte lors de la conception d’une application qui utilise le flux de modification.

### <a name="intermediate-updates"></a>Mises à jour intermédiaires

Seule la dernière modification d’un élément donné est incluse dans le flux de modification. Quand le traitement changera, vous lirez la dernière version de l’élément disponible. Si plusieurs mises à jour ont été apportées au même élément dans un laps de temps réduit, il est possible que certaines mises à jour intermédiaires ne soient pas traitées. Si vous souhaitez effectuer le suivi des mises à jour et pouvoir relire les mises à jour précédentes d’un élément, nous vous recommandons de modéliser plutôt ces mises à jour en tant que séries d’écritures.

### <a name="deletes"></a>Suppressions

Le flux de modification ne capture pas les suppressions. Si vous supprimez un élément de votre conteneur, il est également retiré du flux de modification. La méthode la plus courante pour gérer cela consiste à ajouter un marqueur souple sur les éléments qui sont supprimés. Vous pouvez ajouter une propriété appelée « deleted » et lui affecter la valeur « true » au moment de la suppression. Cette mise à jour de document apparaîtra dans le flux de modification. Vous pouvez définir une durée de vie (TTL) sur cet élément afin qu’il puisse être supprimé automatiquement ultérieurement.

### <a name="guaranteed-order"></a>Ordre garanti

L’ordre est garanti dans le flux de modification au sein d’une valeur de clé de partition, mais pas entre les valeurs de clés de partitions. Vous devez sélectionner une clé de partition qui vous donne une garantie d’ordre significative.

Par exemple, considérez une application de vente au détail utilisant le modèle de conception de provisionnement en événements. Dans cette application, différentes actions de l’utilisateur sont chaque « événement » qui sont modélisés en tant qu’écritures dans Azure Cosmos DB. Imaginez que des exemples d’événements se produisent dans l’ordre suivant :

1. Le client ajoute l’article A à son panier d’achat
2. Le client ajoute l’article B à son panier d’achat
3. Le client supprime l’article A de son panier d’achat
4. Le client valide son achat et le contenu du panier est expédié

Une vue matérialisée du contenu actuel du panier d’achat est tenue à jour pour chaque client. Cette application doit s’assurer que ces événements sont traités dans l’ordre dans lequel ils se produisent. Si, par exemple, la validation du panier était traitée avant la suppression de l’élément A, il est probable que le client recevrait l’article A, plutôt que l’article B. Afin de garantir que ces quatre événements sont traités dans l’ordre de leur occurrence, ils doivent être dans la même valeur de clé de partition. Si vous sélectionnez **username** (chaque client a un nom d’utilisateur unique) comme clé de partition, vous pouvez garantir que ces événements apparaissent dans le flux de modification dans l’ordre dans lequel ils sont écrits dans Azure Cosmos DB.

## <a name="examples"></a>Exemples

Voici quelques exemples de code de flux de modification réels qui vont au-delà de la portée des exemples fournis dans Microsoft Docs :

- [Introduction au flux de modification](https://azurecosmosdb.github.io/labs/dotnet/labs/08-change_feed_with_azure_functions.html)
- [Cas d’usage IoT axé sur le flux de modification](https://github.com/AzureCosmosDB/scenario-based-labs)
- [Cas d’usage de vente au détail centré sur le flux de modification](https://github.com/AzureCosmosDB/scenario-based-labs)

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble du flux de modification](change-feed.md)
* [Options de lecture du flux de modification](read-change-feed.md)
* [Utilisation du flux de modification avec Azure Functions](change-feed-functions.md)