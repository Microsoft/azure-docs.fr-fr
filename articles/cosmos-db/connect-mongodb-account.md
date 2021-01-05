---
title: Connecter une application MongoDB à Azure Cosmos DB
description: Découvrir comment connecter une application MongoDB à Azure Cosmos DB à l’aide d’une chaîne de connexion à partir du Portail Azure
author: timsander1
ms.author: tisande
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/19/2020
ms.reviewer: sngun
ms.openlocfilehash: 06b03ac1cfe37307a5839be0d5ca3b1003db35af
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93072407"
---
# <a name="connect-a-mongodb-application-to-azure-cosmos-db"></a>Connecter une application MongoDB à Azure Cosmos DB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Découvrez comment connecter votre application MongoDB à une instance Azure Cosmos DB à l’aide d’une chaîne de connexion MongoDB. Vous pouvez ensuite utiliser une base de données Azure Cosmos en tant que magasin de données pour votre application MongoDB.

Ce didacticiel fournit deux façons de récupérer les informations de la chaîne de connexion :

- [Méthode du guide de démarrage rapide](#get-the-mongodb-connection-string-by-using-the-quick-start), à utiliser avec les pilotes .NET, Node.js, MongoDB Shell, Java et Python
- [Méthode de la chaîne de connexion personnalisée](#get-the-mongodb-connection-string-to-customize), à utiliser avec d’autres pilotes.

## <a name="prerequisites"></a>Prérequis

- Un compte Azure. Si vous ne possédez pas de compte Azure, vous pouvez créer un [compte Azure gratuit](https://azure.microsoft.com/free/) dès maintenant.
- Un compte Cosmos. Pour obtenir des instructions, consultez [Générer une application web en utilisant l’API pour MongoDB d’Azure Cosmos DB et le kit SDK .NET](create-mongodb-dotnet.md).

## <a name="get-the-mongodb-connection-string-by-using-the-quick-start"></a>Obtenir la chaîne de connexion MongoDB à l’aide du démarrage rapide

1. Dans un navigateur Internet, connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le panneau **Azure Cosmos DB** , sélectionnez l’API.
3. Dans le volet gauche du panneau Compte, cliquez sur **Démarrage rapide**.
4. Choisissez votre plateforme ( **.NET** , **Node.js** , **MongoDB Shell** , **Java** ou **Python** ). Si votre pilote ou outil n’est pas répertorié, ne vous inquiétez pas, nous développons en permanence de nouveaux extraits de code de connexion. Entrez ci-dessous un commentaire sur ce que vous voudriez voir. Pour savoir comment créer votre propre connexion, lisez [Obtenir les informations de chaîne de connexion du compte](#get-the-mongodb-connection-string-to-customize).
5. Copiez et collez l’extrait de code dans votre application MongoDB.

    :::image type="content" source="./media/connect-mongodb-account/QuickStartBlade.png" alt-text="Panneau Démarrage rapide":::

## <a name="get-the-mongodb-connection-string-to-customize"></a>Obtenir la chaîne de connexion MongoDB à personnaliser

1. Dans un navigateur Internet, connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans le panneau **Azure Cosmos DB** , sélectionnez l’API.
3. Dans le volet gauche du panneau Compte, cliquez sur **Chaîne de connexion**.
4. Le panneau **Chaîne de connexion** s’ouvre. Il affiche toutes les informations nécessaires pour la connexion au compte à l’aide d’un pilote pour MongoDB, dont une chaîne de connexion prédéfinie.

   :::image type="content" source="./media/connect-mongodb-account/ConnectionStringBlade.png" alt-text="Panneau Chaîne de connexion" lightbox= "./media/connect-mongodb-account/ConnectionStringBlade.png" :::

## <a name="connection-string-requirements"></a>Exigences relatives à la chaîne de connexion

> [!Important]
> Azure Cosmos DB obéit à des normes et à des exigences strictes en matière de sécurité. Les comptes Azure Cosmos DB nécessitent une authentification et une communication sécurisée via *TLS*.

Azure Cosmos DB prend en charge le format d’URI de la chaîne de connexion MongoDB standard sous certaines conditions spécifiques : Les comptes Azure Cosmos DB nécessitent une authentification et une communication sécurisée via TLS. Le format de chaîne de connexion est donc :

`mongodb://username:password@host:port/[database]?ssl=true`

Les valeurs de cette chaîne sont disponibles dans le panneau **Chaîne de connexion** illustré précédemment :

* Nom d’utilisateur (obligatoire) : Nom du compte Cosmos.
* Mot de passe (obligatoire) : Mot de passe du compte Cosmos.
* Hôte (obligatoire) : Nom de domaine complet du compte Cosmos.
* Port (obligatoire) : 10255.
* Base de données (facultatif) : Base de données utilisée par la connexion. Si aucune base de données n’est fournie, la base de données par défaut est « test ».
* ssl = true (obligatoire)

Par exemple, observez le compte figurant dans le panneau **Chaîne de connexion**. Exemple de chaîne de connexion valide :

`mongodb://contoso123:0Fc3IolnL12312asdfawejunASDF@asdfYXX2t8a97kghVcUzcDv98hawelufhawefafnoQRGwNj2nMPL1Y9qsIr9Srdw==@contoso123.documents.azure.com:10255/mydatabase?ssl=true`

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [utiliser Studio 3T](mongodb-mongochef.md) avec l’API Azure Cosmos DB pour MongoDB.
- Découvrez comment [utiliser Robo 3T](mongodb-robomongo.md) avec l’API Azure Cosmos DB pour MongoDB.
- Explorez les [exemples](mongodb-samples.md) MongoDB avec l’API Azure Cosmos DB pour MongoDB.
