---
title: 'Tutoriel : Créer des applications mobiles avec Xamarin et Azure Cosmos DB'
description: 'Tutoriel : Didacticiel qui permet de créer une application Xamarin iOS, Android ou Forms à l’aide d’Azure Cosmos DB. Azure Cosmos DB est une base de données cloud, mondiale et rapide, pour les applications mobiles.'
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: d1164c14b406fb66638e6a5333fe38180272aa14
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359328"
---
# <a name="tutorial-build-mobile-applications-with-xamarin-and-azure-cosmos-db"></a>Tutoriel : Créer des applications mobiles avec Xamarin et Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.JS](sql-api-nodejs-application.md)
> * [Python](./create-sql-api-python.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

La plupart des applications mobiles doivent stocker des données dans le cloud. Azure Cosmos DB est une base de données cloud conçue pour ce type d’application. Elle a tout ce dont un développeur mobile a besoin. Il s’agit d’une base de données en tant que service, entièrement gérée, qui évolue en fonction de la demande. Elle peut fournir des données à votre application de manière transparente, où que vos utilisateurs se trouvent dans le monde. Avec le [Kit de développement logiciel (SDK) Azure Cosmos DB .NET Core](sql-api-sdk-dotnet-core.md), vous permettez aux applications mobiles Xamarin d’interagir directement avec Azure Cosmos DB, sans intermédiaire.

Dans cet article, nous proposons un didacticiel pour la création d’applications mobiles avec Xamarin et Azure Cosmos DB. Vous trouverez le code source complet de ce didacticiel dans [Xamarin et Azure Cosmos DB sur GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/xamarin), avec des explications sur la gestion des utilisateurs et des autorisations.

## <a name="azure-cosmos-db-capabilities-for-mobile-apps"></a>Fonctionnalités d’Azure Cosmos DB pour les applications mobiles
Azure Cosmos DB fournit les fonctionnalités clés suivantes aux développeurs d’applications mobiles :

:::image type="content" source="media/mobile-apps-with-xamarin/documentdb-for-mobile.png" alt-text="Fonctionnalités d’Azure Cosmos DB pour les applications mobiles":::

* Requêtes riches sur des données sans schéma. Azure Cosmos DB stocke les données sous la forme de documents JSON sans schéma dans des collections hétérogènes. Il propose des [requêtes riches et rapides](./sql-query-getting-started.md), qui ne nécessitent aucun schéma ou index.
* Débit rapide. Seules quelques millisecondes sont nécessaires pour lire et écrire des documents avec Azure Cosmos DB. Les développeurs peuvent spécifier le débit dont ils ont besoin, et Azure Cosmos DB l’honore avec une disponibilité à 99,99 % pour tous les comptes à région unique et à plusieurs régions avec cohérence souple, ainsi qu’une disponibilité de lecture à 99,999 % pour tous les comptes de base de données à plusieurs régions.
* Mise à l’échelle illimitée. Vos conteneurs Azure Cosmos [augmentent à mesure que votre application grandit](partitioning-overview.md). Vous pouvez commencer par des données de petite taille et un débit réduit, soit quelques centaines de requêtes par seconde. Vos collections ou bases de données peuvent inclure jusqu’à plusieurs pétaoctets de données et un débit arbitrairement élevé, soit des centaines de millions de requêtes par seconde.
* Distribution globale. Les utilisateurs d’applications mobiles sont en déplacement, souvent aux quatre coins du monde. Azure Cosmos DB est une [base de données mondialement distribuée](distribute-data-globally.md). Cliquez sur la carte pour rendre les données accessibles à vos utilisateurs.
* Autorisation riche intégrée. Avec Azure Cosmos DB, vous implémentez facilement des modèles très répandus, comme les [données par utilisateur](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/xamarin/UserItems) ou les données partagées par plusieurs utilisateurs, sans recourir à un code d’autorisation complexe personnalisé.
* Requêtes géospatiales. Beaucoup d’applications mobiles proposent aujourd’hui des expériences géocontextuelles. Grâce à la prise en charge de première classe des [types géospatiaux](./sql-query-geospatial-intro.md), Azure Cosmos DB rend la création de ces expériences facile à accomplir.
* Pièces jointes binaires. Vos données d’application comprennent souvent des blobs binaires. La prise en charge native des pièces jointes simplifie l’utilisation d’Azure Cosmos DB comme guichet unique pour les données de votre application.

## <a name="azure-cosmos-db-and-xamarin-tutorial"></a>Didacticiel Azure Cosmos DB et Xamarin
Le didacticiel suivant explique comment créer une application mobile à l’aide de Xamarin et d’Azure Cosmos DB. Vous trouverez le code source complet du didacticiel dans [Xamarin et Azure Cosmos DB sur GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/xamarin).

### <a name="get-started"></a>Bien démarrer
La prise en main d’Azure Cosmos DB est simple. Utilisez le portail Azure pour créer un compte Azure Cosmos DB. Cliquez sur l’onglet **Démarrage rapide**. Téléchargez l’exemple de liste de tâches Xamarin Forms, qui est déjà connecté à votre compte Azure Cosmos DB. 

:::image type="content" source="media/mobile-apps-with-xamarin/cosmos-db-quickstart.png" alt-text="Démarrage rapide d’Azure Cosmos DB pour les applications mobiles":::

Si vous disposez déjà d’une application Xamarin, vous pouvez également ajouter le [package NuGet d’Azure Cosmos DB](sql-api-sdk-dotnet-core.md). Azure Cosmos DB prend en charge les bibliothèques partagées Xamarin.iOS, Xamarin.Android et Xamarin Forms.

### <a name="work-with-data"></a>Utilisation des données
Vos enregistrements de données sont stockés dans Azure Cosmos DB sous la forme de documents JSON sans schéma dans des collections hétérogènes. Vous pouvez stocker des documents avec des structures différentes dans la même collection :

```cs
    var result = await client.CreateDocumentAsync(collectionLink, todoItem);
```

Dans vos projets Xamarin, vous pouvez utiliser des requêtes intégrées au langage sur des données sans schéma :

```cs
    var query = await client.CreateDocumentQuery<ToDoItem>(collectionLink)
                    .Where(todoItem => todoItem.Complete == false)
                    .AsDocumentQuery();

    Items = new List<TodoItem>();
    while (query.HasMoreResults) {
        Items.AddRange(await query.ExecuteNextAsync<TodoItem>());
    }
```
### <a name="add-users"></a>Ajouter des utilisateurs
À l’instar de nombreux exemples de prise en main, l’exemple d’Azure Cosmos DB que vous avez téléchargé s’authentifie auprès du service à l’aide de la clé principale codée en dur dans le code de l’application. Ce comportement par défaut n’est pas judicieux si vous envisagez d’exécuter une application n’importe où sauf sur votre émulateur local. Si un utilisateur non autorisé obtient la clé principale, toutes les données de votre compte Azure Cosmos DB risquent d’être compromises. En revanche, vous souhaitez que votre application puisse uniquement accéder aux enregistrements de l’utilisateur connecté. Azure Cosmos DB permet aux développeurs d’accorder à l’application un accès en lecture ou en lecture/écriture à une collection, un ensemble de documents regroupés par clé de partition, ou un document spécifique. 

Pour transformer l’application de liste de tâches en liste multi-utilisateur, procédez comme suit : 

  1. Ajoutez une connexion à votre application, à l’aide de Facebook, d’Active Directory ou d’un autre fournisseur.

  2. Créez une collection Azure Cosmos DB UserItems avec la clé de partition **/userId**. Le fait de spécifier une clé de partition pour votre collection permet une mise à l’échelle infinie d’Azure Cosmos DB, à mesure que le nombre d’utilisateurs de l’application augmente, tout en offrant des requêtes toujours rapides.

  3. Ajoutez Azure Cosmos DB Ressource Token Broker. Cette API web simple authentifie les utilisateurs et émet des jetons de courte durée pour les utilisateurs connectés, avec un accès limité aux documents se trouvant au sein de la partition de l’utilisateur. Dans cet exemple, nous hébergeons Resource Token Broker dans App Service.

  4. Modifiez l’application pour qu’elle propose une authentification auprès de Resource Token Broker avec Facebook et demande les jetons de ressource pour l’utilisateur Facebook connecté. Vous pouvez ensuite accéder à leurs données dans la collection UserItems.  

Un exemple de code complet de ce modèle est disponible sur [Resource Token Broker sur GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/xamarin/UserItems). Ce diagramme illustre la solution :

:::image type="content" source="media/mobile-apps-with-xamarin/documentdb-resource-token-broker.png" alt-text="Utilisateurs Azure Cosmos DB et répartiteur d’autorisations" border="false":::

Si vous souhaitez que deux utilisateurs puissent accéder à la même liste des tâches, vous pouvez ajouter des autorisations supplémentaires au jeton d’accès dans Resource Token Broker.

### <a name="scale-on-demand"></a>Mise à l’échelle à la demande
Azure Cosmos DB est une base de données gérée en tant que service. À mesure que votre base d’utilisateurs augmente, vous n’avez pas à vous soucier de la configuration des machines virtuelles ou de l’augmentation de la mémoire à tores magnétiques. Il vous suffit d’indiquer à Azure Cosmos DB, le nombre d’opérations par seconde (débit) dont votre application a besoin. Vous pouvez spécifier le débit via l’onglet **Mise à l’échelle** à l’aide d’une mesure de débit appelée Unité de requête par seconde (RU). Par exemple, une opération de lecture d’un document de 1 Ko nécessite une RU. Vous pouvez également ajouter des alertes à la mesure **Débit**, afin de surveiller la croissance du trafic et de modifier le débit par programme lorsque des alertes se déclenchent.

:::image type="content" source="media/mobile-apps-with-xamarin/cosmos-db-xamarin-scale.png" alt-text="Débit de mise à l’échelle d’Azure Cosmos DB à la demande":::

### <a name="go-planet-scale"></a>Développement à l’échelle de la planète
À mesure que votre application gagne en popularité, vous pouvez acquérir des utilisateurs aux quatre coins du monde. Peut-être souhaitez-vous pouvoir faire face à des événements imprévus. Accédez au portail Azure et ouvrez votre compte Azure Cosmos DB. Cliquez sur la carte permettre la réplication continue de vos données dans différentes régions du monde. Grâce à cette fonctionnalité, vos données sont disponibles où que se trouvent vos utilisateurs. Vous pouvez également ajouter des stratégies de basculement, afin d’être prêt à affronter toute situation d’urgence.

:::image type="content" source="media/mobile-apps-with-xamarin/cosmos-db-xamarin-replicate.png" alt-text="Mise à l’échelle d’Azure Cosmos DB dans différentes régions géographiques" border="false":::

Félicitations ! Vous avez terminé la solution et vous disposez d’une application mobile avec Xamarin et Azure Cosmos DB. Vous pouvez utiliser une procédure similaire pour créer des applications Cordova, à l’aide du Kit de développement logiciel (SDK) JavaScript DocumentDB, ainsi que des applications iOS/Android natives, à l’aide des API REST DocumentDB.

## <a name="next-steps"></a>Étapes suivantes
* Affichez le code source de [Xamarin et Azure Cosmos DB sur GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/xamarin).
* Téléchargez le [SDK .NET Core Azure Cosmos DB](sql-api-sdk-dotnet-core.md).
* Consultez davantage d’exemples de code pour les [applications .NET](sql-api-dotnet-samples.md).
* Découvrez plus en détail les [riches fonctionnalités d’interrogation d’Azure Cosmos DB](./sql-query-getting-started.md).
* Découvrez plus en détail la [prise en charge géospatiale dans Azure Cosmos DB](./sql-query-geospatial-intro.md).