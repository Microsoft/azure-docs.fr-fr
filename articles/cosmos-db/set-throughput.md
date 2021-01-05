---
title: Provisionner le débit sur les conteneurs et les bases de données Azure Cosmos
description: Découvrez comment définir un débit provisionné pour vos conteneurs et bases de données Azure Cosmos.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 11/10/2020
ms.openlocfilehash: 4fea027663b55e87822eae1fd0cdb2d67dbc630b
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96170819"
---
# <a name="introduction-to-provisioned-throughput-in-azure-cosmos-db"></a>Introduction au débit approvisionné dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB vous permet de définir le débit approvisionné sur vos bases de données et conteneurs. Il existe deux types de débit approvisionné, standard (manuel) ou avec mise à l’échelle automatique. Cet article présente le fonctionnement du débit approvisionné. 

Une base de données Azure Cosmos est une unité de gestion pour un ensemble de conteneurs. Une base de données se compose d’un ensemble de conteneurs sans schéma. Un conteneur Azure Cosmos est l’unité d’extensibilité pour le stockage et le débit. Un conteneur est partitionné horizontalement sur un ensemble de machines au sein d’une région Azure et réparti entre toutes les régions Azure associées à votre compte Azure Cosmos.

Avec Azure Cosmos DB, vous pouvez configurer le débit selon deux niveaux de granularité :
 
- Conteneurs Cosmos Azure
- Bases de données Azure Cosmos

## <a name="set-throughput-on-a-container"></a>Définir le débit sur un conteneur  

Le débit provisionné sur un conteneur Azure Cosmos est exclusivement réservé au conteneur. Le conteneur reçoit en permanence le débit provisionné. Le débit provisionné sur un conteneur est directement associé à des contrats SLA. Pour découvrir comme configurer le débit standard (manuel) sur un conteneur, consultez [Approvisionner le débit sur un conteneur Azure Cosmos](how-to-provision-container-throughput.md). Pour découvrir comme configurer le débit avec mise à l’échelle automatique sur un conteneur, consultez [Approvisionner le débit avec mise à l’échelle automatique](how-to-provision-autoscale-throughput.md).

L’option la plus fréquemment utilisée est le paramétrage d’un débit provisionné sur un conteneur. Vous pouvez mettre à l’échelle le débit pour un conteneur de manière élastique en provisionnant une quantité quelconque de débit à l’aide [d’unités de requête (RU)](request-units.md). 

Le débit approvisionné pour un conteneur est réparti uniformément entre ses partitions physiques, et en supposant l’utilisation d’une clé de partition appropriée qui répartit uniformément les partitions logiques entre les partitions physiques, le débit est également réparti uniformément entre tous les partitions logiques du conteneur. Vous ne pouvez pas spécifier sélectivement le débit pour les partitions logiques. Dans la mesure où une ou plusieurs partitions logiques d’un conteneur sont hébergées par une partition physique, les partitions physiques appartiennent exclusivement au conteneur et prennent en charge le débit provisionné sur ce conteneur. 

Si la charge de travail d’une partition logique consomme plus que le débit alloué à la partition physique sous-jacente, vos opérations peuvent être limitées en termes de débit. Le phénomène de _partition à chaud_ se produit lorsqu’une partition logique a un nombre de requêtes qui est supérieur de façon disproportionnée à celui des autres valeurs de clé de partition.

En cas de limitation, vous pouvez augmenter le débit aprovisionné pour l’intégralité du conteneur ou retenter les opérations. Vous devez également vous assurer que vous choisissez une clé de partition qui distribue uniformément le volume de stockage et de demande. Pour plus d’informations sur le partitionnement, consultez [Partitionnement et mise à l’échelle horizontale dans Azure Cosmos DB](partitioning-overview.md).

Il vous est recommandé de configurer le débit au niveau de la granularité du conteneur lorsque vous souhaitez que les performances du conteneur soient prévisibles.

L’illustration suivante montre comment une partition physique héberge une ou plusieurs partitions logiques d’un conteneur :

:::image type="content" source="./media/set-throughput/resource-partition.png" alt-text="Partition physique hébergeant une ou plusieurs partitions logiques d'un conteneur" border="false":::

## <a name="set-throughput-on-a-database"></a>Définir le débit sur une base de données

Quand vous provisionnez le débit sur une base de données Azure Cosmos, il est partagé entre tous les conteneurs (nommés conteneurs de base de données partagée) de la base de données. Sauf si vous avez spécifié un débit provisionné sur des conteneurs spécifiques dans la base de données. Le partage de débit provisionné au niveau de la base de données entre ses conteneurs est analogue à l’hébergement d’une base de données sur un cluster de machines. Tous les conteneurs d’une base de données partageant les ressources disponibles sur une machine, vous ne pouvez pas prévoir les performances d'un conteneur spécifique. Pour découvrir comment configurer le débit provisionné sur une base de données, consultez [Configurer le débit provisionné sur une base de données Azure Cosmos](how-to-provision-database-throughput.md). Pour découvrir comme configurer le débit avec mise à l’échelle automatique sur une base de données, consultez [Approvisionner le débit avec mise à l’échelle automatique](how-to-provision-autoscale-throughput.md).

Tous les conteneurs d’une base de données partageant le débit provisionné, Azure Cosmos DB n’offre pas de garantie de débit prévisible pour un conteneur donné de cette base de données. La portion de débit qu'un conteneur spécifique peut recevoir dépend de ce qui suit :

* Nombre de conteneurs.
* Choix des clés de partition pour différents conteneurs.
* Distribution de la charge de travail entre les différentes partitions logiques des conteneurs 

Il est recommandé de configurer le débit sur une base de données lorsque vous souhaitez partager ce débit entre plusieurs conteneurs, et ne pas le dédier à un conteneur spécifique. 

Les exemples suivants démontrent des situations où il est préférable de provisionner le débit au niveau de la base de données :

* Partager le débit approvisionné d’une base de données dans un ensemble de conteneurs est utile dans le cadre d’une application mutualisée. Chaque utilisateur peut être représenté par un conteneur Azure Cosmos distinct.

* Partager le débit approvisionné d’une base de données sur un ensemble de conteneurs est utile quand vous migrez une base de données NoSQL (par exemple, MongoDB ou Cassandra) hébergée sur un cluster de machines virtuelles ou à partir de serveurs physiques locaux vers Azure Cosmos DB. Vous pouvez considérer le débit provisionné configuré sur votre base de données Azure Cosmos en tant qu’équivalent logique (mais plus rentable et élastique) de la capacité de calcul de votre cluster MongoDB ou Cassandra.  

Tous les conteneurs créés à l’intérieur d’une base de données avec un débit provisionné doivent être créés avec une [clé de partition](partitioning-overview.md). À un moment donné, le débit alloué à un conteneur au sein d’une base de données est réparti entre toutes les partitions logiques de ce conteneur. En présence de conteneurs partageant un débit provisionné configuré sur une base de données, vous ne pouvez pas appliquer de manière sélective le débit à un conteneur ou une partition logique spécifique. 

Si la charge de travail d'une partition logique consomme plus que le débit alloué à une partition logique spécifique, vos opérations sont limitées en termes de débit. En cas de limitation, vous pouvez augmenter le débit pour l’intégralité de la base de données ou retenter les opérations. Pour plus d’informations sur le partitionnement, consultez [Partitions logiques](partitioning-overview.md).

Les conteneurs dans une base de données à débit partagé partagent le débit (RU/s) alloué à cette base de données. Avec un débit provisionné standard (manuel), vous pouvez avoir jusqu’à 25 conteneurs avec un minimum de 400 RU/s sur la base de données. Avec le débit provisionné avec mise à l’échelle automatique, vous pouvez avoir jusqu’à 25 conteneurs dans une base de données avec une mise à l’échelle automatique de 4 000 RU/s au maximum (mise à l’échelle entre 400 et 4 000 RU/s).

> [!NOTE]
> En février 2020, nous avons apporté un changement qui vous permet de bénéficier d'un maximum de 25 conteneurs dans une base de données à débit partagée, afin de mieux partager le débit entre les conteneurs. Après les 25 premiers conteneurs, vous ne pouvez ajouter d'autres conteneurs à la base de données que s'ils sont [approvisionnés avec un débit dédié](#set-throughput-on-a-database-and-a-container) distinct du débit partagé de la base de données.<br>
Si votre compte Azure Cosmos DB contient déjà une base de données à débit partagée dotée de >= 25 conteneurs, le compte et tous les autres comptes du même abonnement Azure sont exemptés de ce changement. Si vous avez des commentaires ou des questions, n'hésitez pas à [contacter le support produit](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). 

Si vos charges de travail impliquent la suppression et la recréation de toutes les collections d’une base de données, il est recommandé de supprimer la base de données vide et de recréer une nouvelle base de données avant la création de la collection. L’illustration suivante montre comment une partition physique peut héberger une ou plusieurs partitions logiques, appartenant à différents conteneurs, au sein d’une base de données :

:::image type="content" source="./media/set-throughput/resource-partition2.png" alt-text="Partition physique hébergeant une ou plusieurs partitions logiques appartenant à différents conteneurs" border="false":::

## <a name="set-throughput-on-a-database-and-a-container"></a>Définir le débit sur une base de données et un conteneur

Vous pouvez combiner les deux modèles. Provisionner le débit sur la base de données et le conteneur est autorisé. L’exemple suivant montre comment approvisionner le débit standard (manuel) sur une base de données et un conteneur Azure Cosmos :

* Vous pouvez créer une base de données Azure Cosmos nommée *« Z »* avec le débit approvisionné standard (manuel) des unités de requête *« K »* . 
* Créez ensuite cinq conteneurs nommés *A*, *B*, *C*, *D* et *E* dans la base de données. Lors de la création du conteneur B, assurez-vous d’activer **Fournir un débit dédié pour cette option conteneur** et configurez explicitement *"P"* . RU de débit provisionné sur ce conteneur. Vous pouvez configurer le débit partagé et dédié uniquement lors de la création de la base de données et du conteneur. 

   :::image type="content" source="./media/set-throughput/coll-level-throughput.png" alt-text="Définition du débit au niveau du conteneur":::

* Le début des unités de requête *« K »* est partagé entre les quatre conteneurs *A*, *C*, *D*, et *E*. La quantité exacte de débit disponible pour *A*, *C*, *D*, ou *E* varie. Il n’existe pas de contrat SLA correspondant au débit de chaque conteneur individuel.
* Le conteneur nommé *B* est assuré de bénéficier en permanence du débit des unités de requête *« P »* . Il est associé à des contrats SLA.

> [!NOTE]
> Un conteneur avec un débit approvisionné ne peut pas être converti en conteneur de base de données partagée. À l’inverse, un conteneur de base de données partagée ne peut pas être converti pour disposer d’un débit dédié.

## <a name="update-throughput-on-a-database-or-a-container"></a>Mise à jour du débit sur une base de données ou un conteneur

Après avoir créé un conteneur Azure Cosmos ou une base de données, vous pouvez mettre à jour le débit provisionné. Il n’y a pas de limite sur le débit maximum que vous pouvez provisionner sur la base de données ou le conteneur.

### <a name="current-provisioned-throughput"></a><a id="current-provisioned-throughput"></a> Débit approvisionné actuel

Vous pouvez récupérer le débit approvisionné d’un conteneur ou d’une base de données dans le portail Azure ou à l’aide des Kits de développement logiciel (SDK) suivants :

* [Container.ReadThroughputAsync](/dotnet/api/microsoft.azure.cosmos.container.readthroughputasync?view=azure-dotnet&preserve-view=true) sur le Kit de développement logiciel (SDK) .NET.
* [CosmosContainer.readThroughput](/java/api/com.azure.cosmos.cosmosasynccontainer.readthroughput?view=azure-java-stable&preserve-view=true) sur le Kit de développement logiciel (SDK) Java.

La réponse de ces méthodes contient également le [débit minimal approvisionné](concepts-limits.md#storage-and-database-operations) pour le conteneur ou la base de données :

* [ThroughputResponse.MinThroughput](/dotnet/api/microsoft.azure.cosmos.throughputresponse.minthroughput?view=azure-dotnet&preserve-view=true) sur le Kit de développement logiciel (SDK) .NET.
* [ThroughputResponse.getMinThroughput()](/java/api/com.azure.cosmos.models.throughputresponse.getminthroughput?view=azure-java-stable&preserve-view=true) sur me Kit de développement logiciel (SDK) Java.

La valeur RU/s minimale réelle peut varier en fonction de la configuration de votre compte. Mais en général, il s’agit de la valeur maximale :

* 400 RU/s 
* Le stockage actuel en Go * 10 RU/s (sauf si votre conteneur ou base de données contient plus de 1 To de données, consultez notre [programme sur le stockage étendu / débit faible](#high-storage-low-throughput-program))
* Valeur RU/s la plus élevée provisionnée sur la base de données ou le conteneur / 100

### <a name="changing-the-provisioned-throughput"></a>Modification du débit approvisionné

Vous pouvez mettre à l’échelle le débit approvisionné d’un conteneur ou d’une base de données via le portail Azure ou à l’aide des Kits de développement logiciel (SDK) suivants :

* [Container.ReplaceThroughputAsync](/dotnet/api/microsoft.azure.cosmos.container.replacethroughputasync?view=azure-dotnet&preserve-view=true) sur le Kit de développement logiciel (SDK) .NET.
* [CosmosContainer.replaceThroughput](/java/api/com.azure.cosmos.cosmosasynccontainer.replacethroughput?view=azure-java-stable&preserve-view=true) sur le Kit de développement logiciel (SDK) Java.

Si vous **réduisez le débit approvisionné**, vous pouvez le faire jusqu’à la valeur [minimale](#current-provisioned-throughput).

Si vous **augmentez le débit approvisionné**, la plupart du temps, l’opération est instantanée. Toutefois, dans certains cas, l’opération peut prendre plus de temps en raison des tâches du système pour approvisionner les ressources requises. Dans ce cas, une tentative de modification du débit approvisionné pendant que cette opération est en cours génère une réponse HTTP 423 avec un message d’erreur indiquant qu’une autre opération de mise à l’échelle est en cours.

> [!NOTE]
> Si vous planifiez une charge de travail d’ingestion très importante qui nécessite une forte augmentation du débit approvisionné, gardez à l’esprit que l’opération de mise à l’échelle n’a pas de contrat SLA et, comme indiqué dans le paragraphe précédent, elle peut prendre beaucoup de temps lorsque l’augmentation est importante. Il serait bon de planifier et de commencer la mise à l’échelle avant que la charge de travail ne démarre et d’utiliser les méthodes ci-dessous pour vérifier la progression.

Vous pouvez vérifier par programmation la progression de la mise à l’échelle en lisant la section [Débit approvisionné actuel](#current-provisioned-throughput) et en utilisant :

* [ThroughputResponse.IsReplacePending](/dotnet/api/microsoft.azure.cosmos.throughputresponse.isreplacepending?view=azure-dotnet&preserve-view=true) sur le Kit de développement logiciel (SDK) .NET.
* [ThroughputResponse.isReplacePending()](/java/api/com.azure.cosmos.models.throughputresponse.isreplacepending?view=azure-java-stable&preserve-view=true) sur le Kit de développement logiciel (SDK) Java.

Vous pouvez utiliser les [métriques Azure Monitor](monitor-cosmos-db.md#view-operation-level-metrics-for-azure-cosmos-db) pour voir l’historique du débit provisionné (RU/s) et du stockage sur une ressource.

## <a name="high-storage--low-throughput-program"></a><a id="high-storage-low-throughput-program"></a> Programme de stockage élevé/faible débit

Comme décrit dans la section [Débit approvisionné actuel](#current-provisioned-throughput) ci-dessus, le débit minimal que vous pouvez approvisionner sur un conteneur ou une base de données dépend de plusieurs facteurs. L’un d’entre eux est la quantité de données actuellement stockées, puisqu’Azure Cosmos DB impose un débit minimal de 10 RU/s par Go de stockage.

Cela peut être un problème dans les situations où vous devez stocker de grandes quantités de données, mais où les exigences en matière de débit sont faibles en comparaison. Pour mieux prendre en charge ces scénarios, Azure Cosmos DB a mis en place un **programme « stockage élevé/faible débit »** qui réduit la contrainte de RU/s par Go sur les comptes éligibles.

Vous devez disposer d’au moins un conteneur ou une base de données à débit partagé contenant plus de 1 To de données dans votre compte pour que ce dernier soit éligible. Pour adhérer à ce programme et évaluer votre pleine admissibilité, il vous suffit de remplir [cette enquête](https://customervoice.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRzBPrdEMjvxPuDm8fCLUtXpUREdDU0pCR0lVVFY5T1lRVEhWNUZITUJGMC4u). L’équipe d’Azure Cosmos DB assurera ensuite le suivi et procédera à votre intégration.

## <a name="comparison-of-models"></a>Comparaison des modèles
Ce tableau présente une comparaison entre l’approvisionnement du débit standard (manuel) sur une base de données et sur un conteneur. 

|**Paramètre**  |**Débit standard (manuel) sur une base de données**  |**Débit standard (manuel) sur un conteneur**|**Débit avec mise à l’échelle automatique sur une base de données** | **Débit avec mise à l’échelle automatique sur un conteneur**|
|---------|---------|---------|---------|---------|
|Point d’entrée (RU/s minimum) |400 RU/s. Peut avoir jusqu’à 25 conteneurs sans minimum de RU/s par conteneur.</li> |400| Mise à l’échelle automatique entre 400 et 4 000 RU/s. Peut avoir jusqu’à 25 conteneurs sans minimum de RU/s par conteneur.</li> | Mise à l’échelle automatique entre 400 et 4 000 RU/s.|
|RU/s minimales par conteneur|--|400|--|Mise à l’échelle automatique entre 400 et 4000 RU/s|
|Unités de requête maximales|Illimitées, sur la base de données.|Illimitées, sur le conteneur.|Illimitées, sur la base de données.|Illimitées, sur le conteneur.
|Unités de requête allouées ou disponibles sur un conteneur spécifique|Aucune garantie. Les unités de requête allouées à un conteneur donné dépendent des propriétés. Les propriétés peuvent être le choix de clés de partition des conteneurs qui partagent le débit, la distribution de la charge de travail et le nombre de conteneurs. |Toutes les unités de requête configurées sur le conteneur sont exclusivement réservées à ce conteneur.|Aucune garantie. Les unités de requête allouées à un conteneur donné dépendent des propriétés. Les propriétés peuvent être le choix de clés de partition des conteneurs qui partagent le débit, la distribution de la charge de travail et le nombre de conteneurs. |Toutes les unités de requête configurées sur le conteneur sont exclusivement réservées à ce conteneur.|
|Stockage maximal pour un conteneur|Illimité.|Illimité|Illimité|Illimité|
|Débit maximal par partition logique d’un conteneur|10 000 RU/s|10 000 RU/s|10 000 RU/s|10 000 RU/s|
|Débit maximal (données + index) par partition logique d’un conteneur|20 Go|20 Go|20 Go|20 Go|

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [partitions logiques](partitioning-overview.md).
* Découvrez comment [approvisionner le débit standard (manuel) sur un conteneur Azure Cosmos](how-to-provision-container-throughput.md).
* Découvrez comment [approvisionner le débit standard (manuel) sur une base de données Azure Cosmos](how-to-provision-database-throughput.md).
* Découvrez comment [approvisionner le débit avec mise à l’échelle automatique sur un conteneur ou une base de données Azure Cosmos](how-to-provision-autoscale-throughput.md).
