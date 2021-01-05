---
title: Concevoir un Stockage Table Azure pour les requêtes | Microsoft Docs
description: Concevez des tables pour les requêtes dans un Stockage Table Azure. Choisissez une clé de partition appropriée, optimisez les requêtes et triez les données du service Table.
services: storage
author: tamram
ms.author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.subservice: tables
ms.openlocfilehash: 43ae21d97bc9d8292270ae62006e649f4bcf540b
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93316160"
---
# <a name="design-for-querying"></a>Conception pour l'interrogation
Les solutions de service de Table peuvent lire ou écrire de façon intensive, ou effectuer une combinaison des deux. Cet article décrit les éléments à prendre en compte quand vous concevez votre service de Table pour prendre en charge efficacement les opérations de lecture. En règle générale, une conception qui prend en charge les opérations de lecture de manière efficace le sera également pour des opérations d'écriture. Toutefois, vous devez prendre en compte d’autres éléments quand la conception a pour but de prendre en charge les opérations d’écriture. Ces éléments sont décrits dans l’article [Concevoir pour la modification de données](table-storage-design-for-modification.md).

Un bon point de départ pour concevoir votre solution de service de Table afin de vous permettre de lire les données efficacement consiste à vous demander quelles requêtes votre application devra exécuter pour extraire du service de Table les données dont elle aura besoin.  

> [!NOTE]
> Avec le service de Table, il est important de mettre en place une conception efficace dès le début, car il est difficile et coûteux de la modifier ultérieurement. Par exemple, dans une base de données relationnelle, il est souvent possible de résoudre les problèmes de performances simplement en ajoutant des index à une base de données existante : cela n’est pas possible avec le service de Table.  
> 
> 

Cette section se concentre sur les problèmes à traiter lors de la conception de vos tables pour l'interrogation. Les sujets abordés dans cette section sont les suivants :

* [Impact de votre choix de PartitionKey et RowKey sur les performances des requêtes](#how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance)
* [Choix d’une PartitionKey appropriée](#choosing-an-appropriate-partitionkey)
* [Optimisation des requêtes pour le service de Table](#optimizing-queries-for-the-table-service)
* [Tri des données dans le service de Table](#sorting-data-in-the-table-service)

## <a name="how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance"></a>Impact de votre choix de PartitionKey et RowKey sur les performances des requêtes
Les exemples suivants supposent que le service de Table stocke les entités relatives aux employés (employee) en utilisant la structure suivante (la plupart des exemples omettent la propriété **Timestamp** par souci de clarté) :  

| *Nom de la colonne* | *Type de données* |
| --- | --- |
| **PartitionKey** (nom du service) |String |
| **RowKey** (ID d’employé) |String |
| **FirstName** |String |
| **LastName** |String |
| **Age** |Integer |
| **EmailAddress** |String |

L’article [Vue d’ensemble du Stockage Table Azure](table-storage-overview.md) décrit certaines des principales fonctionnalités du service de Table Azure qui ont un impact direct sur la conception des requêtes. Il en résulte les conseils suivants, qui vous aideront à concevoir des requêtes de service de Table. Notez que la syntaxe de filtre utilisée dans les exemples ci-dessous provient de l’API REST du service de Table. Pour en savoir plus, consultez la rubrique [Interrogation d’entités](/rest/api/storageservices/Query-Entities).  

* Une * **requête de pointage** _ constitue la méthode de recherche la plus efficace. Elle est recommandée pour les recherches sur de gros volumes ou des recherches nécessitant la latence la plus faible. Une telle requête peut utiliser les index pour localiser une entité individuelle très efficacement en spécifiant les valeurs de _ *PartitionKey* * et de **RowKey**. Par exemple : $filter=(PartitionKey eq ’Sales’) and (RowKey eq ’2’)  
* La deuxième méthode conseillée consiste à utiliser une * **requête de plage** _ de données qui utilise la valeur de _ *PartitionKey* * et des filtres sur une plage de valeurs de **RowKey** pour retourner plusieurs entités. La valeur de **PartitionKey** identifie une partition spécifique, tandis que la valeur de **RowKey** identifie un sous-ensemble des entités de cette partition. Par exemple : $filter=PartitionKey eq ’Sales’ and RowKey ge ’S’ and RowKey lt ’T’  
* La troisième méthode conseillée consiste à effectuer une * **analyse de partition** _ qui utilise la valeur de _ *PartitionKey* * et des filtres sur une autre propriété sans clé afin de renvoyer plusieurs entités. La valeur de **PartitionKey** identifie une partition spécifique et les valeurs des propriétés sélectionnent un sous-ensemble d’entités dans cette partition. Par exemple : $filter=PartitionKey eq ’Sales’ and LastName eq ’Smith’  
* Une * **analyse de table** _ n’inclut pas la valeur de _ *PartitionKey* * et s’avère particulièrement inefficace, car elle lance une recherche sur toutes les partitions qui composent la table pour toutes les entités correspondantes. Elle effectue une analyse de table, que votre filtre utilise la valeur de **RowKey** ou non. Par exemple : $filter=LastName eq ’Jones’  
* Les requêtes qui retournent plusieurs entités les retournent triées dans l’ordre de la **PartitionKey** et de la **RowKey**. Pour éviter un nouveau tri des entités dans le client, sélectionnez une valeur de **RowKey** qui définit l’ordre de tri le plus répandu.  

Notez que l’utilisation d’un connecteur «  **or**  » pour spécifier un filtre selon les valeurs de **RowKey** déclenche une analyse de partition et n’est pas traitée en tant que requête de plage de données. Par conséquent, vous devez éviter les requêtes qui utilisent des filtres comme : $filter=PartitionKey eq ’Sales’ and (RowKey eq ’121’ or RowKey eq ’322’)  

Pour obtenir des exemples de code côté client qui utilisent la bibliothèque cliente de stockage pour exécuter des requêtes efficaces, consultez les pages suivantes :  

* [Exécuter une requête de pointage à l’aide de la bibliothèque cliente de stockage](table-storage-design-patterns.md#executing-a-point-query-using-the-storage-client-library)
* [Récupérer plusieurs entités à l’aide de LINQ](table-storage-design-patterns.md#retrieving-multiple-entities-using-linq)
* [Projection côté serveur](table-storage-design-patterns.md#server-side-projection)  

Pour obtenir des exemples de code côté client pouvant gérer plusieurs types d'entité stockés dans la même table, consultez la page :  

* [Utiliser des types d’entités hétérogènes](table-storage-design-patterns.md#working-with-heterogeneous-entity-types)  

## <a name="choosing-an-appropriate-partitionkey"></a>Choix d’une PartitionKey appropriée
Votre choix de **PartitionKey** peut équilibrer le besoin d’utiliser des transactions EGT (pour garantir la cohérence) avec l’exigence de distribution de vos entités sur plusieurs partitions (pour assurer une solution évolutive).  

D'un côté, vous pouvez stocker toutes vos entités dans une seule partition, mais cela peut limiter l'extensibilité de votre solution et empêcher le service de Table d'équilibrer les demandes. D’un autre côté, vous pouvez stocker une seule entité par partition, ce qui améliorerait sensiblement l’extensibilité et permet au service de Table d’équilibrer la charge des demandes, mais vous empêcherait d’utiliser des transactions de groupe d’entités.  

Une **PartitionKey** idéale vous permet d’utiliser des requêtes efficaces et possède assez de partitions pour garantir l’extensibilité de votre solution. En règle générale, vous trouverez que vos entités auront une propriété appropriée qui les distribue sur des partitions suffisantes.

> [!NOTE]
> Par exemple, dans un système qui stocke des informations sur des utilisateurs ou des employés, l’ID utilisateur peut être une bonne PartitionKey. Vous pouvez avoir plusieurs entités qui utilisent un ID utilisateur donné comme clé de partition. Les différentes entités qui stockent des données relatives à un utilisateur sont regroupées en une seule partition et sont ainsi accessibles via des transactions de groupe d’entités, tout en étant hautement évolutives.
> 
> 

Au moment de choisir la valeur de **PartitionKey** , vous devez vous demander comment vous allez insérer, mettre à jour et supprimer des entités. Pour plus d’informations, consultez [Conception de tables pour la modifications de données](table-storage-design-for-modification.md).  

## <a name="optimizing-queries-for-the-table-service"></a>Optimisation des requêtes pour le service de Table
Le service de Table indexe automatiquement vos entités en utilisant les valeurs de **PartitionKey** et **RowKey** dans un seul index en cluster. Voilà pourquoi les requêtes de pointage sont les plus efficaces. Toutefois, il n’existe aucun autre index que celui de l’index en cluster sur la **PartitionKey** et la **RowKey**.

De nombreuses conceptions doivent répondre aux conditions requises pour permettre la recherche d'entités basée sur plusieurs critères. Par exemple, la localisation des entités relatives aux employés (employee) en fonction de leurs adresses e-mail, de leur ID employé ou de leur nom. Les modèles décrits dans [Modèles de conception de table](table-storage-design-patterns.md) répondent à ces types de besoin et décrivent les différentes manières de contourner le fait que le service de Table ne fournit pas d’index secondaire :  

* [Modèle d’index secondaire intra-partition](table-storage-design-patterns.md#intra-partition-secondary-index-pattern) : stockez plusieurs copies de chaque entité en utilisant différentes valeurs de **RowKey** (dans la même partition) pour pouvoir mener des recherches rapides et efficaces et alterner des commandes de tri à l’aide de différentes valeurs de **RowKey**.  
* [Modèle d’index secondaire entre les partitions](table-storage-design-patterns.md#inter-partition-secondary-index-pattern) : stockez plusieurs copies de chaque entité à l’aide de différentes valeurs de **RowKey** dans des partitions ou des tables distinctes pour mener des recherches rapides et efficaces et alterner des commandes de tri à l’aide de différentes valeurs de **RowKey**.  
* [Modèle d’entités d’index](table-storage-design-patterns.md#index-entities-pattern) : mettez à jour des entités d’index pour mener des recherches efficaces renvoyant des listes d’entités.  

## <a name="sorting-data-in-the-table-service"></a>Tri des données dans le service de Table
Le service de Table renvoie des entités triées dans l’ordre croissant selon la **PartitionKey** , puis la **RowKey**. Ces clés correspondent à des valeurs de chaîne. Pour vous assurer que les valeurs numériques permettent des tris corrects, vous devez les convertir en une longueur fixe et les remplir avec des zéros. Par exemple, si la valeur d’ID d’un employé que vous utilisez comme **RowKey** est une valeur de nombre entier, vous devez convertir l’ID de cet employé, **123** , en **00000123**.  

De nombreuses applications ont des conditions d'utilisation pour l'utilisation des données triées dans différents ordres : par exemple, le tri des employés par nom ou par date d'arrivée. Les modèles suivants permettent de comprendre comment alterner des commandes de tri pour vos entités :  

* [Modèle d’index secondaire intra-partition](table-storage-design-patterns.md#intra-partition-secondary-index-pattern) : stockez plusieurs copies de chaque entité en utilisant différentes valeurs de RowKey (dans la même partition) pour pouvoir mener des recherches rapides et efficaces et alterner des commandes de tri à l’aide de différentes valeurs de RowKey.  
* [Modèle d’index secondaire entre les partitions](table-storage-design-patterns.md#inter-partition-secondary-index-pattern) : stockez plusieurs copies de chaque entité à l’aide de différentes valeurs de RowKey dans des partitions ou des tables distinctes pour mener des recherches rapides et efficaces et alterner des commandes de tri à l’aide de différentes valeurs de RowKey.
* [Modèle de fin de journal](table-storage-design-patterns.md#log-tail-pattern) : récupérez les *n* entités récemment ajoutées à une partition en utilisant une valeur de **RowKey** qui effectue un tri dans l’ordre inverse de la date et de l’heure.  

## <a name="next-steps"></a>Étapes suivantes

- [Modèles de conception de table](table-storage-design-patterns.md)
- [Modélisation des relations](table-storage-design-modeling.md)
- [Chiffrer des données de table](table-storage-design-encrypt-data.md)
- [Conception pour la modification de données](table-storage-design-for-modification.md)