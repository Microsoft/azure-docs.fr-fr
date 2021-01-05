---
title: Introduction à l’API Azure Cosmos DB pour MongoDB
description: Découvrez comment utiliser Azure Cosmos DB pour stocker et interroger de grandes quantités de données à l’aide de l’API Azure Cosmos DB pour MongoDB.
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: overview
ms.date: 11/25/2020
author: sivethe
ms.author: sivethe
ms.openlocfilehash: 8205ecab3abfc7a944c12db1aca2bf594f6cd98f
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349450"
---
# <a name="azure-cosmos-dbs-api-for-mongodb"></a>API d’Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

[Azure Cosmos DB](introduction.md) est le service de base de données multi-modèle de Microsoft distribué à l’échelle mondiale pour les applications stratégiques. Azure Cosmos DB fournit la [distribution mondiale clés en main](distribute-data-globally.md), la [mise à l’échelle élastique du débit et du stockage](partitioning-overview.md), des latences de l’ordre de quelques millisecondes dans le monde entier dans plus de 99 pour cent des cas, et une haute disponibilité garantie, le tout soutenu par nos [contrats de niveau de service de pointe](https://azure.microsoft.com/support/legal/sla/cosmos-db/). Azure Cosmos DB [indexe automatiquement les données](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf) sans avoir à s’occuper de la gestion des schémas et des index. Il est multi-modèle et prend en charge les modèles de données en colonnes, documents, graphes et clé-valeur. Le service Azure Cosmos DB implémente les protocoles de transmission des API NoSQL courantes, y compris Cassandra, MongoDB, Gremlin et Stockage Table Azure. Vous pouvez ainsi utiliser les outils et pilotes de clients NoSQL que vous connaissez pour interagir avec votre base de données Cosmos.

> [!NOTE]
> Le [mode de capacité serverless](serverless.md) est maintenant disponible sur l’API d’Azure Cosmos DB pour MongoDB.

## <a name="wire-protocol-compatibility"></a>Compatibilité des protocoles de transmission

Azure Cosmos DB implémente le protocole filaire pour MongoDB. Cette implémentation permet une compatibilité transparente avec les outils, les pilotes et les kits SDK clients MongoDB natifs. Azure Cosmos DB héberge le moteur de base de données MongoDB. Vous trouverez les détails des fonctionnalités prises en charge par MongoDB ici : 
- [Version 3.6 de l’API pour le moteur MongoDB d’Azure Cosmos DB](mongodb-feature-support-36.md)
- [Version 3.2 de l’API pour le moteur MongoDB d’Azure Cosmos DB](mongodb-feature-support.md)

Par défaut, les nouveaux comptes créés à l’aide de l’API Azure Cosmos DB pour MongoDB sont compatibles avec la version 3.6 du protocole filaire MongoDB. Les pilotes de client MongoDB comprenant cette version de protocole doivent être en mesure de se connecter de façon native à Cosmos DB.

:::image type="content" source="./media/mongodb-introduction/cosmosdb-mongodb.png" alt-text="API pour MongoDB d’Azure Cosmos DB" border="false":::

## <a name="key-benefits"></a>Principaux avantages

Les principaux avantages de l’utilisation de Cosmos DB comme base de données en tant que service entièrement gérée distribuée à l’échelle mondiale sont décrits [ici](introduction.md). En outre, en implémentant en mode natif des protocoles de transmission d’API NoSQL courantes, Cosmos DB offre les avantages suivants :

* Migrer facilement votre application vers Cosmos DB tout en conservant des parties importantes de la logique d’application.
* Préserver la mobilité de votre application et garder votre indépendance vis-à-vis des fournisseurs.
* Obtenez des contrats de niveau de service soutenus financièrement pour les API NoSQL courantes propulsées par Cosmos DB.
* Profitez d’une mise à l’échelle élastique du débit et du stockage approvisionnés pour vos bases de données Cosmos selon vos besoins et payez uniquement le débit et le stockage dont vous avez besoin. Vous réaliserez ainsi des économies considérables.
* Une distribution mondiale clé en main avec une réplication des écritures multirégions.

## <a name="cosmos-dbs-api-for-mongodb"></a>API Cosmos DB pour MongoDB

Suivez les guides de démarrage rapide pour créer un compte Azure Cosmos et migrer votre application MongoDB existante afin d’utiliser Azure Cosmos DB, ou en générer une nouvelle :

* [Migrer une application web Node.js MongoDB existante](create-mongodb-nodejs.md).
* [Générer une application web en utilisant l’API Azure Cosmos DB pour MongoDB et le kit de développement logiciel (SDK) .NET](create-mongodb-dotnet.md)
* [Générer une application console en utilisant l’API Azure Cosmos DB pour MongoDB et le kit de développement logiciel (SDK) Java](create-mongodb-java.md)

## <a name="next-steps"></a>Étapes suivantes

Voici quelques conseils pour vous aider à démarrer :

* Pour savoir comment obtenir les informations de chaîne de connexion de votre compte, suivez le didacticiel [Connecter une application MongoDB à Azure Cosmos DB](connect-mongodb-account.md).
* Pour savoir comment créer une connexion entre votre base de données Cosmos et l’application MongoDB dans Studio 3T, suivez le didacticiel [Utiliser Studio 3T avec Azure Cosmos DB](mongodb-mongochef.md).
* Pour importer vos données vers une base de données Cosmos, suivez le didacticiel [Importer des données MongoDB dans Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json).
* Connectez-vous à un compte Cosmos à l’aide de [Robo 3T](mongodb-robomongo.md).
* Découvrez comment [configurer des préférences de lecture pour les applications distribuées dans le monde entier](../cosmos-db/tutorial-global-distribution-mongodb.md).
* Recherchez les solutions aux erreurs couramment détectées dans notre [guide de résolution des problèmes](mongodb-troubleshoot.md)


<sup>Remarque : Cet article décrit une fonctionnalité d’Azure Cosmos DB qui assure la compatibilité des protocoles de transmission avec les bases de données MongoDB. Microsoft n’exécute pas les bases de données MongoDB pour fournir ce service. Azure Cosmos DB n’est pas affilié à MongoDB, Inc.</sup>