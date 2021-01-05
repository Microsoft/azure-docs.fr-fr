---
title: Étapes d’optimisation post-migration avec l’API Azure Cosmos DB pour MongoDB
description: Ce document fournit les techniques d’optimisation post-migration de MongoDB vers l’API Azure Cosmos DB pour Mongo DB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/20/2020
ms.author: chrande
ms.openlocfilehash: 300177b9d5a20ce8082db57837be3ff461fd51a0
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93361654"
---
# <a name="post-migration-optimization-steps-when-using-azure-cosmos-dbs-api-for-mongodb"></a>Étapes d’optimisation post-migration lors de l’utilisation de l’API Azure Cosmos DB pour MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Après avoir migré les données stockées dans la base de données MongoDB vers l’API Azure Cosmos DB pour MongoDB, vous pouvez vous connecter à Azure Cosmos DB et gérer les données. Ce guide fournit les étapes que vous pouvez effectuer après la migration. Pour connaître les étapes de migration, consultez le tutoriel [Migrer MongoDB vers l’API Azure Cosmos DB pour MongoDB](../dms/tutorial-mongodb-cosmos-db.md).

Dans ce guide, vous allez apprendre à :

- [Vous connecter à votre application](#connect-your-application)
- [Optimiser la stratégie d’indexation](#optimize-the-indexing-policy)
- [Configurer la distribution mondiale à l’aide de l’API Azure Cosmos DB pour MongoDB](#globally-distribute-your-data)
- [Définir le niveau de cohérence](#set-consistency-level)

> [!NOTE]
> La seule étape post-migration obligatoire au niveau de votre application consiste à changer la chaîne de connexion dans votre application pour qu’elle pointe vers votre nouveau compte Azure Cosmos DB. Toutes les autres étapes de migration sont des optimisations recommandées.
>

## <a name="connect-your-application"></a>Se connecter à votre application

1. Dans une nouvelle fenêtre, connectez-vous au [portail Azure](https://www.portal.azure.com/).
2. Dans le [portail Azure](https://www.portal.azure.com/), dans le volet gauche, ouvrez le menu **Toutes les ressources** et recherchez le compte Azure Cosmos DB vers lequel vous avez migré vos données.
3. Ouvrez le panneau **Chaîne de connexion**. Le volet droit contient toutes les informations dont vous avez besoin pour vous connecter à votre compte.
4. Utilisez les informations de connexion dans la configuration de votre application (ou d’autres emplacements pertinents) afin de refléter la connexion de l’API Azure Cosmos DB pour MongoDB dans votre application.
:::image type="content" source="./media/mongodb-post-migration/connection-string.png" alt-text="Capture d’écran montrant les paramètres d’une chaîne de connexion.":::

Pour plus d’informations, consultez la page [Connecter une application MongoDB à Azure Cosmos DB](connect-mongodb-account.md).

## <a name="optimize-the-indexing-policy"></a>Optimiser la stratégie d’indexation

Tous les champs de données sont automatiquement indexés par défaut, lors de la migration de données vers Azure Cosmos DB. Dans de nombreux cas, cette stratégie d’indexation par défaut est acceptable. En général, la suppression des index optimise les requêtes d’écriture, et l’application de la stratégie d’indexation par défaut (autrement dit, l’indexation automatique) optimise les requêtes de lecture.

Pour plus d’informations sur l’indexation, consultez [Indexation de données dans l’API Azure Cosmos DB pour MongoDB](mongodb-indexing.md) et les articles sur l’[indexation dans Azure Cosmos DB](index-overview.md).

## <a name="globally-distribute-your-data"></a>Distribuer vos données dans le monde

Azure Cosmos DB est disponible dans toutes les [régions Azure](https://azure.microsoft.com/regions/#services) à travers le monde. Après avoir sélectionné le niveau de cohérence par défaut pour votre compte Azure Cosmos DB, vous pouvez associer une ou plusieurs régions Azure (en fonction de vos besoins de distribution mondiale). Pour une haute disponibilité et une continuité d’activité, nous recommandons toujours une exécution dans au moins deux régions. Vous pouvez consulter les conseils d’[optimisation du coût des déploiements dans plusieurs régions dans Azure Cosmos DB](optimize-cost-regions.md).

Pour distribuer vos données à l’échelle mondiale, consultez [Distribuer des données dans le monde entier sur l’API Azure Cosmos DB pour MongoDB](tutorial-global-distribution-mongodb.md).

## <a name="set-consistency-level"></a>Définir le niveau de cohérence

Azure Cosmos DB offre cinq [niveaux de cohérence](consistency-levels.md) bien définis. Pour en savoir plus sur les correspondances entre les niveaux de cohérence MongoDB et Azure Cosmos DB, consultez [Niveaux de cohérence et API Azure Cosmos DB](./consistency-levels.md). Le niveau de cohérence par défaut est le niveau de cohérence de la session. Le changement du niveau de cohérence est facultatif, et vous pouvez l’optimiser pour votre application. Pour changer le niveau de cohérence à l’aide du portail Azure :

1. Accédez au panneau **Cohérence par défaut** sous Paramètres.
2. Sélectionnez votre [niveau de cohérence](consistency-levels.md)

La plupart des utilisateurs laissent le niveau de cohérence sur le paramètre de cohérence de session par défaut. Toutefois, il existe des [compromis entre disponibilité et performances pour différents niveaux de cohérence](./consistency-levels.md).

## <a name="next-steps"></a>Étapes suivantes

* [Connecter une application MongoDB à Azure Cosmos DB](connect-mongodb-account.md)
* [Se connecter à un compte Azure Cosmos DB à l’aide de Studio 3T](mongodb-mongochef.md)
* [Guide pratique pour distribuer à l’échelle mondiale des lectures par l’intermédiaire de l’API Azure Cosmos DB pour MongoDB](mongodb-readpreference.md)
* [Faire expirer des données avec l’API Azure Cosmos DB pour MongoDB](mongodb-time-to-live.md)
* [Niveaux de cohérence dans Azure Cosmos DB](consistency-levels.md)
* [Indexation dans Azure Cosmos DB](index-overview.md)
* [Unités de requête dans Azure Cosmos DB](request-units.md)