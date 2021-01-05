---
title: Étapes de prémigration pour la migrations de données vers l’API Azure Cosmos DB pour MongoDB
description: Ce document fournit une vue d’ensemble des prérequis pour une migration de données de MongoDB vers Cosmos DB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 09/01/2020
ms.author: chrande
ms.openlocfilehash: 337341daf0e092def639a4e8f6fc8ee0a9b57c75
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349416"
---
# <a name="pre-migration-steps-for-data-migrations-from-mongodb-to-azure-cosmos-dbs-api-for-mongodb"></a>Étapes de prémigration pour les migrations de données de MongoDB vers l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Avant de migrer vos données depuis MongoDB (localement ou dans le cloud) vers l’API Azure Cosmos DB pour MongoDB, vous devez :

1. [Lire les considérations clés relatives à l’utilisation de l’API Azure Cosmos DB pour MongoDB](#considerations)
2. [Choisir une option pour migrer vos données](#options)
3. [Estimer le débit nécessaire pour vos charges de travail](#estimate-throughput)
4. [Choisir une clé de partition optimale pour vos données](#partitioning)
5. [Comprendre la stratégie d’indexation que vous pouvez définir sur vos données](#indexing)

Si vous avez déjà effectué les étapes prérequises ci-dessus pour la migration, vous pouvez [migrer les données MongoDB vers l’API Azure Cosmos DB pour MongoDB à l’aide d’Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db.md). En outre, si vous n’avez pas créé de compte, vous pouvez parcourir n’importe quel [guide de démarrage rapide](create-mongodb-dotnet.md) illustrant les étapes de création d’un compte.

## <a name="considerations-when-using-azure-cosmos-dbs-api-for-mongodb"></a><a id="considerations"></a>Considérations relatives à l’utilisation de l’API Azure Cosmos DB pour MongoDB

Les caractéristiques spécifiques à l’API Azure Cosmos DB pour MongoDB sont les suivantes :

- **Modèle de capacité** : La capacité de base de données dans Azure Cosmos DB est basée sur un modèle basé sur le débit. Ce modèle est basé sur les [unités de requête par seconde](request-units.md) et constitue une unité qui représente le nombre d’opérations de base de données pouvant être exécutées par seconde sur une collection. Cette capacité peut être allouée [au niveau d’une collection ou d’une base de données](set-throughput.md), et elle peut être approvisionnée sur un modèle d’allocation ou à l’aide du [débit approvisionné en mode de mise à l’échelle automatique](provision-throughput-autoscale.md).

- **Unités de requête** : Chaque opération de base de données a un coût d’unités de requête (RU) associé dans Azure Cosmos DB. Une fois l’opération exécutée, ce coût est soustrait du niveau d’unités de requête disponible à une seconde donnée. Si une demande nécessite plus d’unités de requête que le nombre d’unités de requête actuellement allouées/s, il existe deux options pour résoudre le problème : augmenter le nombre d’unités de requête ou attendre que la seconde suivante démarre, puis recommencer l’opération.

- **Capacité élastique** : La capacité d’une collection ou d’une base de données donnée peut changer à tout moment. Cela permet à la base de données de s’adapter de manière élastique aux exigences de débit de votre charge de travail.

- **Partitionnement automatique** : Azure Cosmos DB fournit un système de partitionnement automatique qui requiert un seul partitionnement (ou une clé de partition). Le [mécanisme de partitionnement automatique](partitioning-overview.md) est partagé entre toutes les API Azure Cosmos DB et permet une mise à l’échelle transparente des données et du débit via une distribution horizontale.

## <a name="migration-options-for-azure-cosmos-dbs-api-for-mongodb"></a><a id="options"></a>Options de migration pour l’API Azure Cosmos DB pour MongoDB

Le service [Azure Database Migration Service pour l’API Azure Cosmos DB pour MongoDB](../dms/tutorial-mongodb-cosmos-db.md) simplifie la migration des données en fournissant une plateforme d’hébergement complètement managée, des options de surveillance de la migration et une gestion automatique de la limitation. La liste complète des options est la suivante :

|**Type de migration**|**Solution**|**Considérations**|
|---------|---------|---------|
|En ligne|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB <br/>&bull; Adapté aux jeux de données volumineux et chargé de la réplication des modifications dynamiques <br/>&bull; Fonctionne uniquement avec d’autres sources MongoDB|
|Hors connexion|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB <br/>&bull; Adapté aux jeux de données volumineux et chargé de la réplication des modifications dynamiques <br/>&bull; Fonctionne uniquement avec d’autres sources MongoDB|
|Hors connexion|[Azure Data Factory](../data-factory/connector-azure-cosmos-db.md).|&bull; Facile à configurer et prise en charge de plusieurs sources <br/>&bull; Utilise la bibliothèque d’exécuteurs en bloc Azure Cosmos DB <br/>&bull; Adapté aux jeux de données volumineux <br/>&bull; L’absence de points de contrôle signifie que tout problème au cours de la migration impose le redémarrage de l’ensemble du processus de migration<br/>&bull; L’absence de file d’attente de lettres mortes signifie que quelques fichiers erronés peuvent arrêter l’ensemble du processus de migration <br/>&bull; Nécessite du code personnalisé pour augmenter le débit de lecture pour certaines sources de données|
|Hors connexion|[Outils Mongo existants (mongodump, mongorestore, Studio3T)](https://azure.microsoft.com/resources/videos/using-mongodb-tools-with-azure-cosmos-db/)|&bull; Facilité de configuration et d’intégration <br/>&bull; Nécessite une gestion personnalisée des limitations|

## <a name="estimate-the-throughput-need-for-your-workloads"></a><a id="estimate-throughput"></a> Estimer le débit nécessaire pour vos charges de travail

Dans Azure Cosmos DB, le débit est provisionné à l’avance et mesuré en unités de requête par seconde. Contrairement aux machines virtuelles ou aux serveurs locaux, il est facile d’effectuer un scale-up ou scale-down des unités de requête à tout moment. Vous pouvez instantanément modifier le nombre d’unités de requête provisionnées. Pour en savoir plus, voir [Unités de requête dans Azure Cosmos DB](request-units.md).

Vous pouvez utiliser la [calculatrice de capacité Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/) pour déterminer la quantité d’unités de requête en fonction de la configuration de votre compte de base de données, de la quantité de données, de la taille du document et des opérations de lecture et d’écriture requises par seconde.

Les principaux facteurs qui affectent le nombre d’unités de requête nécessaires sont les suivants :
- **Taille du document** : plus la taille d’un élément/document augmente, plus le nombre d’unités de requête consommées pour le lire ou l’écrire augmente.

- **Nombre de propriétés de document** : le nombre d’unités de requête consommées pour créer ou mettre à jour un document est lié au nombre, à la complexité et à la longueur de ses propriétés. Vous pouvez réduire la consommation d’unités de requête pour les opérations d’écriture en [limitant le nombre de propriétés indexées](mongodb-indexing.md).

- **Modèles de requête** : la complexité d’une requête a une incidence sur le nombre d’unités de requête qu’elle consomme. 

La meilleure façon de comprendre le coût des requêtes consiste à utiliser des exemples de données dans Azure Cosmos DB et à [exécuter des exemples de requête dans l’interpréteur de commandes MongoDB](connect-mongodb-account.md) à l’aide de la commande `getLastRequestStastistics` pour obtenir les frais de requêtes, ce qui fournira le nombre d’unités de requête consommées :

`db.runCommand({getLastRequestStatistics: 1})`

Cette commande génère un document JSON semblable au suivant :

```{  "_t": "GetRequestStatisticsResponse",  "ok": 1,  "CommandName": "find",  "RequestCharge": 10.1,  "RequestDurationInMilliSeconds": 7.2}```

Vous pouvez également utiliser [les paramètres de diagnostic](cosmosdb-monitor-resource-logs.md) pour comprendre la fréquence et les modèles des requêtes exécutées sur Azure Cosmos DB. Les résultats des journaux de diagnostic peuvent être envoyés à un compte de stockage, à une instance EventHub ou à [Azure Log Analytics](../azure-monitor/log-query/log-analytics-tutorial.md).  

## <a name="choose-your-partition-key"></a><a id="partitioning"></a>Choisir votre clé de partition
Le partitionnement est un point clé à prendre en compte avant de migrer des données. Azure Cosmos DB utilise un partitionnement complètement managé pour augmenter la capacité d’une base de données à répondre aux exigences de stockage et de débit. Cette fonctionnalité n’a pas besoin de l’hébergement ni de la configuration des serveurs de routage.   

D’une façon similaire, la fonctionnalité de partitionnement ajoute automatiquement de la capacité et rééquilibre les données en conséquence. Pour plus d’informations et pour obtenir des suggestions concernant le choix de la clé de partition appropriée pour vos données, consultez l’article [Choix d’une de clé de partition](partitioning-overview.md#choose-partitionkey). 

## <a name="index-your-data"></a><a id="indexing"></a>Indexer vos données

L’API d’Azure Cosmos DB pour le serveur MongoDB version 3.6 indexe automatiquement uniquement le champ `_id`. Ce champ ne peut pas être supprimé. Elle applique automatiquement l’unicité du champ `_id` par clé de partition. Pour indexer des champs supplémentaires, appliquez les commandes de gestion d’index MongoDB. Cette stratégie d’indexation par défaut est différente de celle de l’API SQL Azure Cosmos DB, qui indexe tous les champs par défaut.

Les capacités d’indexation fournies par Azure Cosmos DB incluent l’ajout d’index composites, d’index uniques et d’index de durée de vie (index TTL). L’interface de gestion d’index est mappée à la commande `createIndex()`. Pour en savoir plus, consultez [Indexation dans l’API Azure Cosmos DB pour MongoDB](mongodb-indexing.md).

[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db.md) migre automatiquement les collections MongoDB avec des index uniques. Toutefois, les index uniques doivent être créés avant la migration. Azure Cosmos DB ne prend pas en charge la création d’index uniques quand il existe déjà des données dans vos collections. Pour plus d’informations, consultez [Clés uniques dans Azure Cosmos DB](unique-keys.md).

## <a name="next-steps"></a>Étapes suivantes
* [Migrer vos données MongoDB vers Cosmos DB à l’aide de Database Migration Service](../dms/tutorial-mongodb-cosmos-db.md) 
* [Provisionner le débit sur les conteneurs et les bases de données Azure Cosmos](set-throughput.md)
* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
* [Distribution de données mondiale avec Azure Cosmos DB](distribute-data-globally.md)
* [Indexation dans Azure Cosmos DB](index-overview.md)
* [Unités de requête dans Azure Cosmos DB](request-units.md)