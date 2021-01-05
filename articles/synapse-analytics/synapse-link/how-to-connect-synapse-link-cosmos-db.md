---
title: Se connecter à Azure Synapse Link (préversion) pour Cosmos DB
description: Découvrez comment connecter une base de données Azure Cosmos DB à un espace de travail Azure Synapse avec Azure Synapse Link
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 04/21/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 0e4bcc13a2eaddbf394d8468b29eeff475119e2d
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459018"
---
# <a name="connect-to-azure-synapse-link-for-azure-cosmos-db"></a>Se connecter à Azure Synapse Link pour Cosmos DB

Cet article explique comment accéder à une base de données Azure Cosmos DB à partir d’Azure Synapse Analytics Studio avec Azure Synapse Link.

## <a name="prerequisites"></a>Prérequis

Avant de connecter une base de données Azure Cosmos DB à votre espace de travail, les points suivants doivent être respectés :

* Une base de données Azure Cosmos DB existante, ou créez un compte en suivant les étapes de [Démarrage rapide : Gérer un compte Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/how-to-manage-database-account).
* Un espace de travail Azure Synapse existant, ou créez un espace de travail en suivant les étapes de [Démarrage rapide : Créer un espace de travail Synapse](https://docs.microsoft.com/azure/synapse-analytics/quickstart-create-workspace).

> [!IMPORTANT]
> Azure Synapse Link pour Azure Cosmos DB est actuellement pris en charge pour les espaces de travail pour lesquels un réseau virtuel managé n’est pas activé.

## <a name="enable-synapse-link-on-an-azure-cosmos-db-database-account"></a>Activer Synapse Link sur un compte de base de données Azure Cosmos DB

Pour effectuer de l’analytique à grande échelle dans Azure Cosmos DB sans affecter vos performances opérationnelles, nous vous recommandons d’activer Synapse Link pour Azure Cosmos DB. Synapse Link apporte une capacité HTAP pour un conteneur et un support intégré dans Azure Synapse.

## <a name="go-to-synapse-studio"></a>Accéder à Synapse Studio

Dans votre espace de travail Azure Synapse, sélectionnez **Lancer Synapse Studio**. Dans la page d’accueil de Synapse Studio, sélectionnez **Données** pour accéder à l’Explorateur d’objets de données.

## <a name="connect-an-azure-cosmos-db-database-to-an-azure-synapse-workspace"></a>Connecter une base de données Azure Cosmos DB à un espace de travail Azure Synapse

La connexion d’une base de données Azure Cosmos DB s’effectue en tant que service lié. Avec un service lié à Azure Cosmos DB, vous pouvez parcourir et explorer les données, lire et écrire à partir d’Apache Spark pour Azure Synapse Analytics ou SQL dans Azure Cosmos DB.

À partir de l’Explorateur d’objets de données, vous pouvez vous connecter directement à une base de données Azure Cosmos DB en suivant ces étapes :

1. Sélectionnez l’icône **+** à proximité de **Données**.
1. Sélectionnez **Se connecter à des données externes**.
1. Sélectionnez l’API à laquelle vous voulez vous connecter, par exemple **API SQL** ou **API pour MongoDB**.
1. Sélectionnez **Continuer**.
1. Utilisez un nom convivial pour nommer le service lié. Le nom apparaît dans l’Explorateur d’objets de données et est utilisé par les runtimes Azure Synapse pour se connecter à la base de données et aux conteneurs.
1. Sélectionnez le **nom du compte Azure Cosmos DB** et le **nom de la base de données**.
1. (Facultatif) Si aucune région n’est spécifiée, les opérations du runtime Azure Synapse sont routées vers la région la plus proche où le magasin analytique est activé. Vous pouvez aussi définir manuellement la région que vos utilisateurs doivent utiliser pour accéder au magasin analytique Azure Cosmos DB. Sélectionnez **Propriétés de connexion supplémentaires**, **Nouveau**. Sous **Nom de la propriété**, entrez **PreferredRegions**. Définissez la **Valeur** sur la région de votre choix, par exemple **WestUS2**. (Il n’y a pas d’espace entre les mots et le chiffre.)
1. Sélectionnez **Créer**.

Les bases de données Azure Cosmos DB apparaissent sous l’onglet **Lié** dans la section **Azure Cosmos DB**. Avec Azure Cosmos DB, vous pouvez différencier un conteneur activé pour HTAP d’un conteneur limité à l’OLTP via les icônes suivantes :

**Conteneur limité à l’OLTP** :

![Visualisation montrant l’icône de conteneur OLTP.](../media/quickstart-connect-synapse-link-cosmosdb/oltp-container.png)

**Conteneur activé pour HTAP** :

![Visualisation montrant l’icône de conteneur HTAP.](../media/quickstart-connect-synapse-link-cosmosdb/htap-container.png)

## <a name="quickly-interact-with-code-generated-actions"></a>Interagir rapidement avec des actions générées par le code

Quand vous cliquez avec le bouton droit dans un conteneur, vous obtenez une liste des mouvements qui vont déclencher un runtime Spark ou SQL. L’écriture de données dans un conteneur aura lieu via le magasin transactionnel d’Azure Cosmos DB et consommera des unités de requête.  

## <a name="next-steps"></a>Étapes suivantes

* [Découvrir ce qui est pris en charge entre Azure Synapse et Azure Cosmos DB](./concept-synapse-link-cosmos-db-support.md)
* [Découvrir comment interroger un magasin analytique avec Spark](./how-to-query-analytical-store-spark.md)
