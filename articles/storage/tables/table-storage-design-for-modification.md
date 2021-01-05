---
title: Concevoir un stockage Table Azure pour la modification de données | Microsoft Docs
description: Concevoir des tables pour la modifications de données dans le stockage Table Azure. Optimiser les opérations d’ajout, de mise à jour et de suppression. Assurer la cohérence de vos entités stockées.
services: storage
ms.service: storage
author: tamram
ms.author: tamram
ms.topic: article
ms.date: 04/23/2018
ms.subservice: tables
ms.openlocfilehash: 25785bc4b945f469e67f2a71eb6676940e091d56
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88236758"
---
# <a name="design-for-data-modification"></a>Conception pour la modification de données
Cet article décrit les conceptions qui permettent d’optimiser les insertions, les mises à jour et les suppressions. Dans certains cas, vous devez évaluer le compromis entre les conceptions optimisées pour l'interrogation et celles optimisées pour la modification des données, comme vous le feriez dans les conceptions de bases de données relationnelles (bien que les techniques permettant de gérer les compromis de conception sont différentes dans une base de données relationnelle). La section Modèles de conception de table décrit plusieurs modèles de conception détaillés pour le service de Table et apporte des informations sur certains de ces compromis. En pratique, vous constaterez que les nombreuses conceptions optimisées pour l'interrogation des entités fonctionnent aussi bien pour la modification des entités.  

## <a name="optimize-the-performance-of-insert-update-and-delete-operations"></a>Optimiser les performances des opérations d'insertion, de mise à jour et de suppression
Pour mettre à jour ou supprimer une entité, vous devez être en mesure de l’identifier à l’aide des valeurs de **PartitionKey** et de **RowKey**. À cet égard, le choix de la **PartitionKey** et de la **RowKey** pour la modification des entités doit respecter les mêmes critères que ceux déterminant votre choix de prise en charge des requêtes de pointage, afin d’identifier les entités aussi efficacement que possible. Il ne vaut mieux pas utiliser d’analyse de table ou de partition inefficace pour localiser une entité afin de découvrir les valeurs de **PartitionKey** et de **RowKey** que vous devez mettre à jour ou supprimer.  

Les modèles suivants de la section Modèles de conception de table permettent d’optimiser les performances ou vos opérations d’insertion, de mise à jour et de suppression :  

* [Modèle de suppression de volume élevé](table-storage-design-patterns.md#high-volume-delete-pattern) : permet de supprimer un volume élevé d’entités en stockant toutes les entités en vue d’une suppression simultanée à partir de leurs propres tables distinctes. Vous supprimez les entités en supprimant la table.  
* [Modèle de série de données](table-storage-design-patterns.md#data-series-pattern) : stocke des séries de données complètes dans une seule entité pour réduire le nombre de vos demandes.  
* [Modèle d’entités larges](table-storage-design-patterns.md#wide-entities-pattern) : utilisez plusieurs entités physiques pour stocker des entités logiques avec plus de 252 propriétés.  
* [Modèle d’entités volumineuses](table-storage-design-patterns.md#large-entities-pattern) : utilisez le stockage d’objets blob pour stocker des valeurs de propriétés volumineuses.  

## <a name="ensure-consistency-in-your-stored-entities"></a>Assurer la cohérence de vos entités stockées
L'autre facteur déterminant pour votre choix de clés en vue de l'optimisation des modifications de données consiste à assurer la cohérence à l'aide de transactions atomiques. Vous pouvez uniquement utiliser une EGT pour mener des opérations sur des entités stockées dans la même partition.  

Les modèles suivants de l’article [Modèles de conception de table](table-storage-design-patterns.md) abordent la gestion de la cohérence :  

* [Modèle d’index secondaire intra-partition](table-storage-design-patterns.md#intra-partition-secondary-index-pattern) : stockez plusieurs copies de chaque entité en utilisant différentes valeurs de **RowKey** (dans la même partition) pour pouvoir mener des recherches rapides et efficaces et alterner des commandes de tri à l’aide de différentes valeurs de **RowKey**.  
* [Modèle d’index secondaire entre les partitions](table-storage-design-patterns.md#inter-partition-secondary-index-pattern) : stockez plusieurs copies de chaque entité à l’aide de différentes valeurs de RowKey dans des partitions ou des tables distinctes pour mener des recherches rapides et efficaces et alterner des commandes de tri à l’aide de différentes valeurs de **RowKey** .  
* [Modèle de transactions cohérentes](table-storage-design-patterns.md#eventually-consistent-transactions-pattern) : permet de conserver un comportement cohérent entre les limites de partition ou les limites du système de stockage à l’aide des files d’attente Azure.
* [Modèle d’entités d’index](table-storage-design-patterns.md#index-entities-pattern) : mettez à jour des entités d’index pour mener des recherches efficaces qui retournent des listes d’entités.  
* [Modèle de dénormalisation](table-storage-design-patterns.md#denormalization-pattern) : combinez des données liées dans une seule entité pour pouvoir récupérer toutes les données dont vous avez besoin en utilisant une seule requête de pointage.  
* [Modèle de série de données](table-storage-design-patterns.md#data-series-pattern) : stocke des séries de données complètes dans une seule entité pour réduire le nombre de vos demandes.  

Pour plus d’informations sur les transactions de groupe d’entités, consultez la section [Transactions de groupe d’entités](table-storage-design.md#entity-group-transactions).  

## <a name="ensure-your-design-for-efficient-modifications-facilitates-efficient-queries"></a>Faire en sorte que votre conception pour des modifications efficaces facilite les requêtes efficaces
Dans de nombreux cas, une conception visant à obtenir une interrogation efficace entraîne des modifications efficaces, mais vous devez toujours évaluer si cela est le cas pour votre scénario. Certains des modèles de l’article [Modèles de conception de table](table-storage-design-patterns.md) évaluent clairement les compromis entre l’interrogation et la modification des entités. Vous devez toujours prendre en compte le nombre de chaque type d’opération.  

Les modèles suivants de l’article [Modèles de conception de table](table-storage-design-patterns.md) répondent aux compromis entre la conception pour des requêtes efficaces et la conception pour une modification de données efficace :  

* [Modèle de clé composée](table-storage-design-patterns.md#compound-key-pattern) : utilisez les valeurs de **RowKey** composée pour permettre à un client de rechercher des données associées en utilisant une seule requête de pointage.  
* [Modèle de fin de journal](table-storage-design-patterns.md#log-tail-pattern) : récupérez les *n* entités récemment ajoutées à une partition en utilisant une valeur de **RowKey** qui effectue un tri dans l’ordre inverse de la date et de l’heure.  
