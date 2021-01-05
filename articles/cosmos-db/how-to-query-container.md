---
title: Interroger des conteneurs dans Azure Cosmos DB
description: Découvrez comment interroger des conteneurs dans Azure Cosmos DB à l’aide de requêtes dans une partition et dans plusieurs partitions
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 3/18/2019
ms.author: mjbrown
ms.openlocfilehash: 0f08ca84597b08b9a236b7bfb0fc9c849423a752
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93335889"
---
# <a name="query-an-azure-cosmos-container"></a>Interroger un conteneur Azure Cosmos
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article explique comment interroger un conteneur (collection, graphe ou table) dans Azure Cosmos DB. En particulier, il aborde la façon dont les requêtes de partition et sur plusieurs partitions fonctionnent dans Azure Cosmos DB.

## <a name="in-partition-query"></a>Requête dans une partition

Quand vous interrogez des données de conteneurs, si un filtre de clé de partition est spécifié dans la requête, Azure Cosmos DB optimise la requête automatiquement. Il route la requête vers les [partitions physiques](partitioning-overview.md#physical-partitions) qui correspondent aux valeurs de clé de partition spécifiées dans le filtre.

Par exemple, considérez la requête ci-dessous avec un filtre d’égalité sur `DeviceId`. Si nous exécutons cette requête sur un conteneur partitionné sur `DeviceId`, cette requête filtre sur une seule partition physique.

```sql
    SELECT * FROM c WHERE c.DeviceId = 'XMS-0001'
```

Comme dans l’exemple précédent, cette requête filtre également sur une partition unique. L’ajout du filtre supplémentaire sur `Location` ne modifie pas cela :

```sql
    SELECT * FROM c WHERE c.DeviceId = 'XMS-0001' AND c.Location = 'Seattle'
```

Voici une requête qui comporte un filtre de plage sur la clé de partition et qui n’est pas limitée à une seule partition physique. Pour être une requête dans une partition, la requête doit avoir un filtre d’égalité qui comprend la clé de partition :

```sql
    SELECT * FROM c WHERE c.DeviceId > 'XMS-0001'
```

## <a name="cross-partition-query"></a>Requête dans plusieurs partitions

La requête suivante n’a pas de filtre sur la clé de partition (`DeviceId`). Par conséquent, elle doit se distribuer sur toutes les partitions physiques où elle est exécutée sur l’index de chaque partition :

```sql
    SELECT * FROM c WHERE c.Location = 'Seattle`
```

Chaque partition physique a son propre index. Par conséquent, lorsque vous exécutez une requête sur plusieurs partitions sur un conteneur, vous exécutez de fait une requête *par* partition physique. Azure Cosmos DB regroupe automatiquement les résultats sur différentes partitions physiques.

Les index de différentes partitions physiques sont indépendants les uns des autres. Il n’y a pas d’index global dans Azure Cosmos DB.

## <a name="parallel-cross-partition-query"></a>Requête dans plusieurs partitions parallèles

Les kits SDK Azure Cosmos DB 1.9.0 (et versions ultérieures) prennent en charge les options d’exécution de requêtes parallèles. Les requêtes dans plusieurs partitions parallèles vous permettent d’exécuter des requêtes dans plusieurs partitions avec une faible latence.

Vous pouvez gérer l’exécution de requêtes parallèles en réglant les paramètres suivants :

- **MaxConcurrency** : définit le nombre maximal de connexions réseau simultanées aux partitions du conteneur. Si vous affectez la valeur `-1` à cette propriété, le kit SDK gère le degré de parallélisme. Si la  `MaxConcurrency` est définie sur `0`, il existe une seule connexion réseau aux partitions du conteneur.

- **MaxBufferedItemCount**  : limite la latence des requêtes par rapport à l’utilisation de la mémoire côté client. Si cette option est omise ou si elle a la valeur -1, le kit SDK gère le nombre d’éléments mis en mémoire tampon durant l’exécution de requêtes parallèles.

En raison de la capacité d’Azure Cosmos DB à paralléliser les requêtes entre les partitions, la latence des requêtes est généralement correctement mise à l’échelle lorsque le système ajoute des [partitions physiques](partitioning-overview.md#physical-partitions). Toutefois, les frais de RU augmenteront considérablement avec le nombre total de partitions physiques.

Lorsque vous exécutez une requête sur plusieurs partitions, vous effectuez essentiellement une requête distincte par partition physique individuelle. Même si les requêtes sur plusieurs partitions utilisent l’index, s’il est disponible, elles ne sont toujours pas aussi efficaces que les requêtes dans une seule partition.

## <a name="useful-example"></a>Exemple utile

Voici une analogie pour mieux comprendre les requêtes sur plusieurs partitions :

Supposons que vous êtes un livreur qui doit remettre des colis à différents complexes d’appartements. Chaque complexe d’appartements a une liste sur place qui contient tous les numéros d’unité des résidents. Nous pouvons comparer chaque complexe d’appartements à une partition physique et chaque liste à l’index de la partition physique.

Nous pouvons comparer les requêtes dans la partition et les requêtes sur plusieurs partitions à l’aide de l’exemple suivant :

### <a name="in-partition-query"></a>Requête dans une partition

Si le livreur connaît le bon complexe d’appartements (la bonne partition physique), il peut immédiatement se rediriger vers le bon bâtiment. Le livreur peut vérifier la liste des numéros d’unité des résidents (l’index) du complexe d’appartements et livrer rapidement les colis correspondants. Dans ce cas, le livreur ne perd pas de temps ou d’efforts à conduire jusqu’à un complexe d’appartements pour vérifier si des destinataires de colis habitent ici.

### <a name="cross-partition-query-fan-out"></a>Requête dans plusieurs partitions (distribution ramifiée)

Si le livreur ne connaît pas le bon complexe d’appartements (la bonne partition physique), il devra aller jusqu’à chaque complexe d’appartements unique et vérifier la liste avec tous les numéros d’unité des résidents (l’index). Une fois qu’il arrive à chaque complexe d’appartements, il peut toujours utiliser la liste des adresses de chaque résident. Toutefois, il doit vérifier la liste de chaque complexe d’appartements, que des destinataires de colis s’y trouvent ou non. C’est ainsi que les requêtes sur plusieurs partitions fonctionnent. Même si elles peuvent utiliser l’index (elles n’ont pas besoin d’aller frapper à chaque porte), elles doivent vérifier séparément l’index pour chaque partition physique.

### <a name="cross-partition-query-scoped-to-only-a-few-physical-partitions"></a>Requête sur plusieurs partitions (limitée à quelques partitions physiques uniquement)

Si le livreur sait que tous les destinataires de colis vivent dans un certain nombre de complexes d’appartements, il n’a pas besoin de conduire jusqu’à chaque complexe. Même si conduire jusqu’à quelques complexes nécessitera toujours plus de travail que la visite d’un seul bâtiment, le livreur économisera ainsi beaucoup de temps et d’efforts. Si une requête a la clé de partition dans son filtre avec le mot clé `IN`, elle ne vérifie que les index de la partition physique appropriée pour les données.

## <a name="avoiding-cross-partition-queries"></a>Prévention des requêtes sur plusieurs partitions

Pour la plupart des conteneurs, il est inévitable que vous ayez des requêtes sur plusieurs partitions. Avoir des requêtes sur plusieurs partitions n’est pas un problème en soi ! Presque toutes les opérations de requête sont prises en charge sur plusieurs partitions (aussi bien pour les clés de partition logique que les partitions physiques). Azure Cosmos DB offre également de nombreuses optimisations dans le moteur de requête et les kits de développement logiciel clients pour paralléliser l’exécution des requêtes sur les partitions physiques.

Pour la plupart des scénarios de lecture intensive, nous vous recommandons de sélectionner simplement la propriété la plus courante dans vos filtres de requête. Vous devez également vous assurer que votre clé de partition adhère aux autres [meilleures pratiques de sélection de clé de partition](partitioning-overview.md#choose-partitionkey).

Il n’est généralement important d’éviter les requêtes sur plusieurs partitions qu’avec les conteneurs volumineux. Vous êtes facturé un minimum d’environ 2,5 RU chaque fois que vous vérifiez l’index d’une partition physique pour obtenir des résultats, même si aucun élément de la partition physique ne correspond au filtre de la requête. Par conséquent, si vous n’avez qu’une ou un petit nombre de partitions physiques, les requêtes sur plusieurs partitions ne consomment pas beaucoup plus de RU que les requêtes dans une seule partition.

Le nombre de partitions physiques est lié à la quantité de RU configurées. Chaque partition physique autorise jusqu’à 10 000 RU configurées et peut stocker jusqu’à 50 Go de données. Azure Cosmos DB gérera automatiquement les partitions physiques. Le nombre de partitions physiques dans votre conteneur dépend de votre débit configuré et du stockage consommé.

Vous devez essayer d’éviter les requêtes sur plusieurs partitions si votre charge de travail répond aux critères ci-dessous :
- Vous envisagez de configurer plus de 30 000 RU
- Vous envisagez de stocker plus de 100 Go de données

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants pour en savoir plus sur le partitionnement dans Azure Cosmos DB :

- [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
- [Create a synthetic partition key](synthetic-partition-keys.md) (Créer une clé de partition synthétique)
