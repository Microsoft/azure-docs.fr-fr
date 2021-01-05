---
title: Créer une clé de partition synthétique dans Azure Cosmos DB
description: Découvrez comment utiliser des clés de partition synthétiques dans vos conteneurs Azure Cosmos pour distribuer les données et la charge de travail uniformément entre les clés
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/03/2019
author: markjbrown
ms.author: mjbrown
ms.openlocfilehash: 6b8bc44f1ba5624c37620205aaa574e618ef395f
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93340649"
---
# <a name="create-a-synthetic-partition-key"></a>Créer une clé de partition synthétique
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Il est recommandé de disposer d’une clé de partition comportant de nombreuses valeurs distinctes (plusieurs centaines ou plusieurs milliers). L'objectif est de répartir vos données et votre charge de travail de manière uniforme entre les éléments associés à ces valeurs de clé de partition. En l’absence d’une telle propriété dans vos données, vous pouvez construire une *clé de partition synthétique*. Ce document décrit plusieurs techniques de base permettant de générer une clé de partition synthétique pour votre conteneur Cosmos.

## <a name="concatenate-multiple-properties-of-an-item"></a>Concaténer plusieurs propriétés d’un élément

Vous pouvez former une clé de partition en concaténant plusieurs valeurs de propriété dans une propriété `partitionKey` artificielle. Ces clés sont appelées des clés synthétiques. Considérons l’exemple de document suivant :

```JavaScript
{
"deviceId": "abc-123",
"date": 2018
}
```

Pour le document précédent, une option consiste à définir /deviceId ou /date comme clé de partition. Utilisez cette option si vous souhaitez partitionner votre conteneur en fonction de l’ID de périphérique ou de la date. Une autre option consiste à concaténer ces deux valeurs dans une propriété `partitionKey` synthétique utilisée comme clé de partition.

```JavaScript
{
"deviceId": "abc-123",
"date": 2018,
"partitionKey": "abc-123-2018"
}
```

Dans le cadre de scénarios en temps réel, une base de données peut compter des milliers d’éléments. Plutôt que d’ajouter la clé synthétique manuellement, définissez une logique côté client pour concaténer les valeurs et insérer la clé synthétique dans les éléments de vos conteneurs Cosmos.

## <a name="use-a-partition-key-with-a-random-suffix"></a>Utiliser une clé de partition avec suffixe aléatoire

Pour répartir la charge de travail plus uniformément, une autre stratégie consiste à ajouter un nombre aléatoire à la fin de la valeur de la clé de partition. Cette répartition des éléments vous permet d'effectuer des opérations d'écriture parallèles sur des partitions.

Par exemple, si une clé de partition représente une date. Vous pouvez choisir un nombre aléatoire compris entre  1 et 400  et le concaténer en tant que suffixe de la date. Cette méthode aboutit à des valeurs de clé de partition telles que  `2018-08-09.1`, `2018-08-09.2`, etc., jusqu’à  `2018-08-09.400`. La clé de partition étant aléatoire, les opérations d'écriture effectuées quotidiennement sur le conteneur sont réparties de manière uniforme sur plusieurs partitions. Cette méthode optimise le parallélisme et le débit global.

## <a name="use-a-partition-key-with-pre-calculated-suffixes"></a>Utiliser une clé de partition avec suffixes précalculés 

La stratégie de suffixe aléatoire peut considérablement améliorer le débit d’écriture, mais il est difficile de lire un élément spécifique. Vous ne connaissez pas la valeur de suffixe utilisée au moment où vous avez écrit l’élément. Pour faciliter la lecture d’éléments individuels, utilisez la stratégie des suffixes précalculés. Au lieu d’utiliser un nombre aléatoire pour répartir les éléments entre les partitions, utilisez un nombre calculé en fonction de ce que vous souhaitez interroger.

Reprenons l’exemple précédent, dans lequel un conteneur utilise une date comme clé de partition. Supposons maintenant que chaque élément a un attribut  `Vehicle-Identification-Number` (`VIN`) auquel nous souhaitons accéder. Supposons également que vous exécutez souvent des requêtes pour rechercher des éléments par `VIN`, en plus de la date. Avant que votre application n'inscrive l'élément dans le conteneur, elle peut calculer un suffixe basé sur le NIV et l'ajouter à la date de la clé de partition. Le calcul peut générer un nombre compris entre 1 et 400, uniformément réparti. Ce résultat est semblable aux résultats obtenus avec la méthode de la stratégie de suffixe aléatoire. La valeur de la clé de partition correspond alors à la date concaténée avec le résultat calculé.

Avec cette stratégie, les écritures sont uniformément réparties sur les valeurs de clé de partition et sur les partitions. Vous pouvez facilement lire un élément et une date particuliers, car vous pouvez calculer la valeur de la clé de partition pour un `Vehicle-Identification-Number` spécifique. L’avantage de cette méthode est qu’elle vous évite de créer une clé de partition à chaud unique (la clé de partition qui prend toute la charge de travail). 

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le concept de partitionnement, consultez les articles suivants :

* En savoir plus sur les [partitions logiques](partitioning-overview.md).
* Découvrez comment [approvisionner le débit sur des conteneurs et bases de données Azure Cosmos](set-throughput.md).
* Découvrez comment [approvisionner le débit sur un conteneur Azure Cosmos](how-to-provision-container-throughput.md).
* Découvrez comment [approvisionner le débit sur une base de données Azure Cosmos](how-to-provision-database-throughput.md).
