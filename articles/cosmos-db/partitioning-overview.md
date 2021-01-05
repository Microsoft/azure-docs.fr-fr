---
title: Partitionnement et mise à l’échelle horizontale dans Azure Cosmos DB
description: En savoir plus sur le partitionnement de partitions logiques et physiques dans Azure Cosmos DB, les meilleures pratiques lors du choix d’une clé de partition et la gestion des partitions logiques
author: deborahc
ms.author: dech
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 10/12/2020
ms.openlocfilehash: 7c05ca6462d49d1d41791e5b93b7723ac681d448
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93080830"
---
# <a name="partitioning-and-horizontal-scaling-in-azure-cosmos-db"></a>Partitionnement et mise à l’échelle horizontale dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB utilise le partitionnement pour procéder à la mise à l’échelle des conteneurs au sein d’une base de données afin de répondre aux besoins de performances de votre application. Avec le partitionnement, les éléments d’un conteneur sont répartis en sous-ensembles distincts, appelés *partitions logiques*. Les partitions logiques sont formées en fonction de la valeur d’une *clé de partition* associée à chaque élément d’un conteneur. Tous les éléments d’une partition logique possèdent la même valeur de clé de partition.

Par exemple, un conteneur contient des éléments. Chaque élément a une valeur unique pour la propriété `UserID`. Si `UserID` sert de clé de partition pour les éléments présents dans un conteneur et qu’il existe 1 000 valeurs `UserID` uniques, 1 000 partitions logiques sont créées pour le conteneur.

Outre une clé de partition qui détermine la partition logique de l’élément, chaque élément présent dans un conteneur possède un *ID d’élément* (unique dans une partition logique). La combinaison de la clé de partition et de l’ *ID d’élément* crée l’ *index* de l’élément, qui identifie l’élément de façon unique. Le [choix d’une clé de partition](#choose-partitionkey) est une décision importante qui déterminera les performances de votre application.

Cet article explique la relation entre les partitions logiques et physiques. Il présente également les bonnes pratiques pour le partitionnement et donne une vue détaillée du fonctionnement de la mise à l’échelle horizontale dans Azure Cosmos DB. Il n’est pas nécessaire de comprendre ces détails internes pour sélectionner votre clé de partition, mais nous les avons abordés afin que vous compreniez bien comment fonctionne Azure Cosmos DB.

## <a name="logical-partitions"></a>Partitions logiques

Une partition logique est constituée d’un ensemble d’éléments dotés de la même clé de partition. Par exemple, dans un conteneur qui contient des données sur la nutrition, tous les éléments contiennent une propriété `foodGroup`. Vous pouvez utiliser `foodGroup` comme clé de partition pour le conteneur. Des groupes d’éléments ayant des valeurs spécifiques pour `foodGroup`, telles que `Beef Products`, `Baked Products` et `Sausages and Luncheon Meats`, forment des partitions logiques distinctes. Vous n’avez pas à vous soucier de la suppression d’une partition logique quand les données sous-jacentes sont supprimées.

Une partition logique définit également la portée des transactions de base de données. Vous pouvez mettre à jour les éléments d’une partition logique via une [transaction avec isolement de capture instantanée](database-transactions-optimistic-concurrency.md). Lorsque de nouveaux éléments sont ajoutés à un conteneur, de nouvelles partitions logiques sont créées de façon transparente par le système.

Il n’existe aucune limite au nombre de partitions logiques dans votre conteneur. Chaque partition logique peut stocker jusqu’à 20 Go de données. Les bons choix de clé de partition procurent une large gamme de valeurs possibles. Par exemple, dans un conteneur dont tous les éléments contiennent une propriété `foodGroup`, les données de la partition logique `Beef Products` peuvent croître jusqu’à 20 Go. La [sélection d’une clé de partition](#choose-partitionkey) avec une large gamme de valeurs possibles garantit que le conteneur peut être mis à l’échelle.

## <a name="physical-partitions"></a>Partitions physiques

Un conteneur est mis à l’échelle en distribuant les données et le débit sur des partitions physiques. En interne, une ou plusieurs partitions logiques sont mappées à une seule partition physique. En général, les petits conteneurs ont de nombreuses partitions logiques, mais ils n’ont besoin que d’une seule partition physique. Contrairement aux partitions logiques, les partitions physiques sont une implémentation interne du système et sont entièrement gérées par Azure Cosmos DB.

Le nombre de partitions physiques dans votre conteneur dépend de la configuration suivante :

* La quantité de débit approvisionnée (chaque partition physique peut fournir un débit allant jusqu’à 10 000 unités de requête par seconde).
* Le stockage de données total (chaque partition physique peut stocker jusqu’à 50 Go de données).

Il n’existe aucune limite au nombre total de partitions physiques dans votre conteneur. À mesure qu’augmente la taille des données ou du débit provisionné, Azure Cosmos DB crée automatiquement des partitions physiques en divisant celles qui existent déjà. Les divisions des partitions physiques n’impactent pas la disponibilité de votre application. Après la division des partitions physiques, toutes les données d’une seule partition logique sont toujours stockées sur la même partition physique. La division d’une partition physique crée simplement un mappage de partitions logiques sur des partitions physiques.

Le débit approvisionné pour un conteneur est uniformément réparti entre les partitions physiques. Une conception de clé de partition qui ne distribue pas les requêtes équitablement peut entraîner la direction d’un trop grand nombre de requêtes vers un petit sous-ensemble de partitions dont le niveau d’accès devient « chaud ». Les partitions « chaudes » peuvent entraîner une utilisation inefficace du débit approvisionné et, par conséquent, une limitation du débit et une augmentation des coûts.

Vous pouvez voir les partitions physiques de votre conteneur dans la section **Stockage** du **panneau Métriques** du portail Azure :

:::image type="content" source="./media/partitioning-overview/view-partitions-zoomed-out.png" alt-text="Affichage du nombre de partitions physiques" lightbox="./media/partitioning-overview/view-partitions-zoomed-in.png" ::: 

Dans la capture d’écran ci-dessus, un conteneur a `/foodGroup` comme clé de partition. Chacune des trois barres du graphique représente une partition physique. Dans l’image, la **plage de clés de partition** est identique à une partition physique. La partition physique sélectionnée contient trois partitions logiques : `Beef Products`, `Vegetable and Vegetable Products` et `Soups, Sauces, and Gravies`.

Si vous approvisionnez un débit de 18 000 unités de requête par seconde (RU/s), chacune des trois partitions physiques peut utiliser 1/3 du débit total approvisionné. Au sein de la partition physique sélectionnée, les clés de partition logique `Beef Products`, `Vegetable and Vegetable Products` et `Soups, Sauces, and Gravies` peuvent, collectivement, utiliser les 6 000  RU/s provisionnées de la partition physique. Le débit provisionné étant réparti uniformément entre les partitions physiques de votre conteneur, il est important de choisir une clé de partition qui répartit uniformément la consommation du débit en [choisissant la clé de partition logique appropriée](#choose-partitionkey). 

> [!NOTE]
> Si vous choisissez une clé de partition qui répartit uniformément la consommation du débit entre les partitions logiques, vous vous assurez que la consommation du débit sur les partitions physiques est équilibrée.

## <a name="managing-logical-partitions"></a>Gestion des partitions logiques

Azure Cosmos DB gère de manière transparente et automatique le positionnement des partitions logiques sur les partitions physiques afin de répondre efficacement aux besoins d’extensibilité et de performances du conteneur. À mesure que le débit et le stockage requis par une application augmentent, Azure Cosmos DB déplace les partitions logiques pour répartir automatiquement la charge sur un plus grand nombre de partitions physiques. En savoir plus sur les [partitions physiques](partitioning-overview.md#physical-partitions).

Azure Cosmos DB utilise un partitionnement basé sur un hachage pour répartir les partitions logiques sur des partitions physiques. Azure Cosmos DB hache la valeur de la clé de partition d’un élément. Le résultat du hachage détermine la partition physique. Azure Cosmos DB alloue ensuite l’espace des hachages de clé de partition uniformément entre les partitions physiques.

Les transactions (dans les procédures stockées ou dans les déclencheurs) ne sont autorisées que pour les éléments d’une même partition logique.

En savoir plus sur la [façon dont Azure Cosmos DB gère les partitions](partitioning-overview.md). (Il n’est pas nécessaire de connaître le fonctionnement interne de la création ou de l’exécution de vos applications mais il n’est décrit ici qu’à titre d’information.)

## <a name="replica-sets"></a>Jeux de réplicas

Chaque partition physique se compose d’un ensemble de réplicas, également appelé [*jeu de réplicas*](global-dist-under-the-hood.md). Chaque jeu de réplicas héberge une instance du moteur de base de données. Grâce au jeu de réplicas, les données stockées dans la partition physique sont durables, hautement disponibles et cohérentes. Chaque réplica qui constitue la partition physique hérite du quota de stockage de la partition. Tous les réplicas d’une partition physique prennent collectivement en charge le débit alloué à la partition physique. Azure Cosmos DB gère automatiquement les jeux de réplicas.

En général, les petits conteneurs nécessitent uniquement une seule partition physique, mais ils ont toujours au moins quatre réplicas.

L’illustration suivante montre comment les partitions logiques sont mappées sur des partitions physiques mondialement distribuées :

:::image type="content" source="./media/partitioning-overview/logical-partitions.png" alt-text="Image illustrant le partitionnement dans Azure Cosmos DB" border="false":::

## <a name="choosing-a-partition-key"></a><a id="choose-partitionkey"></a>Choix d’une clé de partition

Une clé de partition a deux composants : le **chemin de clé de partition** et la **valeur de clé de partition**. Par exemple, considérez un élément { "userId" : "Andrew", "worksFor": "Microsoft" } si vous choisissez « userId » comme clé de partition, voici les deux composants de la clé de partition :

* Chemin de clé de partition (par exemple : « /userId »). Le chemin de clé de partition accepte les caractères alphanumériques et les traits de soulignement (_). Vous pouvez également utiliser des objets imbriqués à l’aide de la notation de chemin standard (/).

* La valeur de clé de partition (par exemple : « Andrew »). La valeur de clé de partition peut être de type chaîne ou numérique.

Pour en savoir plus sur les limites du débit, du stockage et de la longueur de la clé de partition, consultez l’article [Quotas de service Azure Cosmos DB](concepts-limits.md).

La sélection de votre clé de partition est un choix de conception simple, mais important dans Azure Cosmos DB. Une fois que vous avez sélectionné votre clé de partition, il n’est pas possible de la modifier sur place. Si vous avez besoin de modifier votre clé de partition, vous devez déplacer vos données vers un nouveau conteneur à l’aide de la nouvelle clé de partition de votre choix.

Pour **tous les conteneurs** , votre clé de partition doit :

* Être une propriété qui a une valeur qui ne change pas. Si une propriété est votre clé de partition, vous ne pouvez pas mettre à jour la valeur de cette propriété.

* Avoir une cardinalité élevée. En d’autres termes, la propriété doit avoir une large gamme de valeurs possibles.

* Répartir la consommation d’unités de requête (RU) et le stockage des données uniformément sur toutes les partitions logiques. Cela garantit une consommation des RU et une répartition du stockage uniformes sur les partitions physiques.

Si vous avez besoin de [transactions ACID à plusieurs éléments](database-transactions-optimistic-concurrency.md#multi-item-transactions) dans Azure Cosmos DB, vous devez utiliser [des procédures stockées ou des déclencheurs](how-to-write-stored-procedures-triggers-udfs.md#stored-procedures). Toutes les procédures stockées et tous les déclencheurs basés sur JavaScript sont limités à une seule partition logique.

## <a name="partition-keys-for-read-heavy-containers"></a>Clés de partition pour les conteneurs à lecture intensive

Pour la plupart des conteneurs, les critères ci-dessus sont tout ce que vous devez prendre en compte lors de la sélection d’une clé de partition. Toutefois, pour les conteneurs à lecture intensive, vous souhaiterez peut-être choisir une clé de partition qui apparaît fréquemment comme filtre dans vos requêtes. Les requêtes peuvent être [efficacement acheminées uniquement vers les partitions physiques concernées](how-to-query-container.md#in-partition-query) en incluant la clé de partition dans le prédicat de filtre.

Si la plupart des demandes de votre charge de travail sont des requêtes et que la plupart de vos requêtes ont un filtre d’égalité sur la même propriété, cette propriété peut être un bon choix de clé de partition. Par exemple, si vous exécutez fréquemment une requête qui filtre sur `UserID`, la sélection de `UserID` comme clé de partition réduit le nombre de [requêtes dans plusieurs partitions](how-to-query-container.md#avoiding-cross-partition-queries).

Toutefois, si votre conteneur est petit, vous n’avez probablement pas assez de partitions physiques pour avoir à vous soucier de l’impact sur les performances des requêtes dans plusieurs partitions. La plupart des petits conteneurs dans Azure Cosmos DB nécessitent uniquement une ou deux partitions physiques.

Si votre conteneur peut atteindre plus de quelques partitions physiques, vous devez vous assurer de choisir une clé de partition qui minimise les requêtes dans plusieurs partitions. Votre conteneur nécessitera plus de quelques partitions physiques lorsque l’une des conditions suivantes est remplie :

* Votre conteneur aura plus de 30 000 RU configurées

* Votre conteneur stockera plus de 100 Go de données

## <a name="using-item-id-as-the-partition-key"></a>Utilisation de l’ID d’élément comme clé de partition

Si votre conteneur a une propriété qui présente une large gamme de valeurs possibles, il s’agit probablement d’un bon choix de clé de partition. L’ *ID d’élément* est un exemple possible d’une telle propriété. Pour les petits conteneurs à lecture intensive ou les conteneurs à écriture intensive de toute taille, l’ *ID d’élément* est naturellement un bon choix pour la clé de partition.

La propriété système *ID d’élément* existe dans chaque élément de votre conteneur. Vous pouvez avoir d’autres propriétés qui représentent un ID logique de votre élément. Dans de nombreux cas, il s’agit également d’excellents choix de clé de partition pour les mêmes raisons que pour l’ *ID d’élément*.

L’ *ID d’élément* est un excellent choix de clé de partition pour les raisons suivantes :

* Il existe un large éventail de valeurs possibles (un *ID d’élément* unique par élément).
* Étant donné qu’il existe un *ID d’élément* unique par élément, l’ *ID d’élément* fait un excellent travail pour équilibrer la consommation des RU et le stockage des données.
* Vous pouvez facilement effectuer des lectures de point efficaces, car vous connaissez toujours la clé de partition d’un élément si vous connaissez son *ID d’élément*.

Voici quelques points à prendre en compte lors de la sélection de l’ *ID d’élément* comme clé de partition :

* Si l’ *ID d’élément* est la clé de partition, il deviendra un identificateur unique dans tout le conteneur. Vous ne pouvez pas avoir d’éléments dont l’ *ID d’élément* est dupliqué.
* Si vous disposez d’un conteneur à lecture intensive qui comporte un grand nombre de [partitions physiques](partitioning-overview.md#physical-partitions), les requêtes seront plus efficaces si elles sont dotées d’un filtre d’égalité avec l’ *ID d’élément*.
* Vous ne pouvez pas exécuter de procédures stockées ni de déclencheurs sur plusieurs partitions logiques.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur le [débit approvisionné dans Azure Cosmos DB](request-units.md).
* Renseignez-vous sur la [distribution globale dans Azure Cosmos DB](distribute-data-globally.md).
* Découvrez comment [approvisionner le débit sur un conteneur Azure Cosmos](how-to-provision-container-throughput.md).
* Découvrez comment [approvisionner le débit sur une base de données Azure Cosmos](how-to-provision-database-throughput.md).
