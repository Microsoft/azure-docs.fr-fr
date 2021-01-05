---
title: Connecter Qlik Sense à Azure Cosmos DB et visualiser vos données
description: Cet article décrit les étapes nécessaires pour connecter Azure Cosmos DB à Qlik Sense et pour visualiser vos données.
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
author: SnehaGunda
ms.author: sngun
ms.topic: how-to
ms.date: 05/23/2019
ms.reviewer: sngun
ms.openlocfilehash: 2401786a82b6a3e5bf6c6a893a8e7cd3656f3402
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996953"
---
# <a name="connect-qlik-sense-to-azure-cosmos-db-and-visualize-your-data"></a>Connecter Qlik Sense à Azure Cosmos DB et visualiser vos données
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

Qlik Sense est un outil de visualisation de données qui regroupe des données provenant de différentes sources dans une même vue. Qlik Sense indexe chaque relation possible dans vos données, ce qui vous permet d’obtenir immédiatement des insights sur les données. Vous pouvez visualiser les données d’Azure Cosmos DB à avec Qlik Sense. Cet article décrit les étapes nécessaires pour connecter Azure Cosmos DB à Qlik Sense et pour visualiser vos données. 

> [!NOTE]
> La connexion de Qlik Sense à Azure Cosmos DB est prise en charge seulement pour les comptes des API SQL et Azure Cosmos DB pour MongoDB.

Vous pouvez connecter Qlik Sense à Azure Cosmos DB avec :

* L’API SQL de Cosmos DB en utilisant le connecteur ODBC.

* L’API Azure Cosmos DB pour MongoDB en utilisant le connecteur Qlik Sense MongoDB (actuellement en préversion).

* L’API d’Azure Cosmos DB pour MongoDB et l’API SQL en utilisant le connecteur de l’API REST dans Qlik Sense.

* L’API MongoDB de Cosmos DB en utilisant le connecteur gRPC pour Qlik Core.
Cet article décrit les détails de la connexion à l’API SQL de Cosmos DB avec le connecteur ODBC.

Cet article décrit les détails de la connexion à l’API SQL de Cosmos DB avec le connecteur ODBC.

## <a name="prerequisites"></a>Prérequis

Avant de suivre les instructions de cet article, veillez à disposer des ressources suivantes :

* Téléchargez [Qlik Sense Desktop](https://www.qlik.com/us/try-or-buy/download-qlik-sense) ou configurez Qlik Sense dans Azure en [installant l’élément Qlik Sense de la Place de marché](https://azuremarketplace.microsoft.com/marketplace/apps/qlik.qlik-sense).

* Téléchargez les [données sur les jeux vidéo](https://www.kaggle.com/gregorut/videogamesales). Ces exemples de données sont au format CSV. Stockez ces données dans un compte Cosmos DB et visualisez-les dans Qlik Sense.

* Créez un compte d’API SQL Azure Cosmos DB en suivant les étapes décrites dans la section [Créer un compte](create-sql-api-dotnet.md#create-account) de l’article de démarrage rapide.

* [Créez une base de données et une collection](create-sql-api-java.md#add-a-container) : vous pouvez utiliser la valeur du débit de la collection à 1 000 RU/s. 

* Chargez les exemples de données de ventes de jeux vidéo sur votre compte Cosmos DB. Vous pouvez importer les données avec l’outil de migration de données d’Azure Cosmos DB, en choisissant une importation des données [séquentielle](import-data.md#SQLSeqTarget) ou [en bloc](import-data.md#SQLBulkTarget). Environ 3 à 5 minutes sont nécessaires pour importer les données dans le compte Cosmos DB.

* Téléchargez, installez et configurez le pilote ODBC selon la procédure décrite dans l’article [Se connecter à Cosmos DB avec le pilote ODBC](odbc-driver.md). Les données des jeux vidéo étant un simple jeu de données, vous n’avez pas besoin de modifier le schéma : utilisez simplement le schéma de mappage par défaut de la collection.

## <a name="connect-qlik-sense-to-cosmos-db"></a>Connecter Qlik Sense à Cosmos DB

1. Ouvrez Qlik Sense, puis sélectionnez **Créer une application**. Spécifiez un nom pour votre application, puis sélectionnez **Créer**.

   :::image type="content" source="./media/visualize-qlik-sense/create-new-qlik-sense-app.png" alt-text="Créer une application Qlik Sense":::

2. Une fois la nouvelle application créée, sélectionnez **Ouvrir l’application** et choisissez **Ajouter des données à partir de fichiers et d’autres sources**. 

3. Parmi les sources de données, sélectionnez **ODBC** pour ouvrir la fenêtre de configuration de la nouvelle connexion. 

4. Passez à **DSN utilisateur** et choisissez la connexion ODBC que vous avez créée précédemment. Spécifiez un nom pour la connexion, puis sélectionnez **Créer**. 

   :::image type="content" source="./media/visualize-qlik-sense/create-new-connection.png" alt-text="Créer une connexion":::

5. Après avoir créé la connexion, vous pouvez choisir la base de données et la collection où se trouvent les données des jeux vidéo, puis les prévisualiser.

   :::image type="content" source="./media/visualize-qlik-sense/choose-database-and-collection.png" alt-text="Choisir la base de données et la collection"::: 

6. Sélectionnez ensuite **Ajouter des données** pour charger les données sur Qlik Sense. Une fois que vous avez chargé les données sur Qlik Sense, vous pouvez générer des insights et effectuer une analyse sur les données. Vous pouvez utiliser les insights ou créer votre propre application explorant les ventes de jeux vidéo. L’image suivante montre 

   :::image type="content" source="./media/visualize-qlik-sense/visualize-data.png" alt-text="Visualiser les données":::

### <a name="limitations-when-connecting-with-odbc"></a>Limitations lors de la connexion avec ODBC 

Cosmos DB est une base de données distribuée sans schéma, avec des pilotes modélisés autour des besoins des développeurs. Le pilote ODBC nécessite une base de données avec schéma pour inférer les colonnes, leur type de données et d’autres propriétés. La requête SQL standard ou la syntaxe DML avec des fonctionnalités relationnelles n’est pas applicable à l’API SQL de Cosmos DB, car l’API SQL n’est pas au standard SQL ANSI. Pour cette raison, les instructions SQL émises via le pilote ODBC sont traduites dans la syntaxe SQL spécifique à Cosmos DB, qui n’a pas d’équivalent pour toutes les constructions. Pour éviter ces problèmes de traduction, vous devez appliquer un schéma quand vous configurez la connexion ODBC. L’article [Se connecter avec le pilote ODBC](odbc-driver.md) contient des suggestions et des méthodes pour vous aider à configurer le schéma. Veillez à créer ce mappage pour chaque base de données/collection au sein du compte Cosmos DB.

## <a name="next-steps"></a>Étapes suivantes

Si vous utilisez un autre outil de visualisation, comme Power BI, vous pouvez vous y connecter en suivant les instructions du document suivant :

* [Visualiser des données Cosmos DB avec le connecteur Power BI](powerbi-visualize.md)
