---
title: Générer une application en temps réel - Azure Functions et Azure SignalR Service
description: Découvrez comment développer une application web sans serveur en temps réel avec Azure SignalR Service à l’aide de l’exemple suivant.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: bf0b5f19d1aa224fa4a2e79c20f2900e3400fce0
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151133"
---
# <a name="build-real-time-apps-with-azure-functions-and-azure-signalr-service"></a>Compiler des applications en temps réel avec Azure Functions et Azure SignalR

Étant donné qu’Azure SignalR Service et Azure Functions sont des services entièrement gérés et hautement évolutifs qui vous permettent de vous concentrer sur la génération d’applications au lieu de la gestion d’infrastructure, il est courant de les utiliser ensemble pour fournir des communications en temps réel dans un environnement [serverless](https://azure.microsoft.com/solutions/serverless/).

> [!NOTE]
> Apprenez à utiliser SignalR et Azure Functions ensemble dans le tutoriel interactif [Activer les mises à jour automatiques dans une application web à l’aide d’Azure Functions et de SignalR Service](/learn/modules/automatic-update-of-a-webapp-using-azure-functions-and-signalr).

## <a name="integrate-real-time-communications-with-azure-services"></a>Intégrer des communications en temps réel avec les services Azure

Azure Functions permet d’écrire du code dans [plusieurs langages](../azure-functions/supported-languages.md) (notamment JavaScript, Python, C# et Java), qui se déclenche chaque fois que des événements se produisent dans le cloud. Voici des exemples d’événements :

* Requêtes HTTP et de webhook
* Minuteurs périodiques
* Événements de services Azure, tels que :
    - Event Grid
    - Event Hubs
    - Service Bus
    - Flux de modification Cosmos DB
    - Stockage (blob et files d’attente)
    - Connecteurs d’applications logiques, tels que Salesforce et SQL Server

En utilisant Azure Functions pour intégrer ces événements avec le service Azure SignalR, vous pouvez avertir des milliers de clients chaque fois que des événements se produisent.

Voici quelques scénarios courants pour la messagerie serverless en temps réel que vous pouvez implémenter avec Azure Functions et SignalR :

* Visualiser la télémétrie d’appareil IoT sur une carte ou un tableau de bord en temps réel
* Mettre à jour des données dans une application lors de la mise à jour de documents dans Cosmos DB
* Envoyer des notifications internes à l’application lors de la création de commandes dans Salesforce

## <a name="signalr-service-bindings-for-azure-functions"></a>Liaisons de service SignalR pour Azure Functions

Les liaisons de service SignalR pour Azure Functions permettent à une application Azure Functions de publier des messages destinés aux clients connectés au service SignalR. Les clients peuvent se connecter au service à l’aide d’un kit de développement logiciel (SDK) client SignalR disponible en .NET, JavaScript et Java (d’autres langages seront bientôt disponibles).

### <a name="an-example-scenario"></a>Exemple de scénario

L’utilisation des liaisons de service SignalR consiste, par exemple, en une intégration d’Azure Functions avec Azure Cosmos DB et SignalR pour envoyer des messages en temps réel lorsque de nouveaux événements apparaissent dans un flux de modification Cosmos DB.

![Cosmos DB, Azure Functions, service SignalR](media/signalr-concept-azure-functions/signalr-cosmosdb-functions.png)

1. Une modification est apportée dans une collection Cosmos DB.
2. L’événement de modification est propagé vers le flux de modification Cosmos DB.
3. Azure Functions est déclenché par l’événement de modification à l’aide du déclencheur Cosmos DB.
4. La liaison de sortie du service SignalR publie un message pour le service SignalR.
5. Le service SignalR publie le message pour tous les clients connectés.

### <a name="authentication-and-users"></a>Authentification et utilisateurs

Le service SignalR vous permet de diffuser des messages auprès de tous les clients ou uniquement à un sous-ensemble de clients, comme ceux appartenant à un même utilisateur. Les liaisons de service SignalR pour Azure Functions peuvent être combinées avec l’authentification App Service pour authentifier les utilisateurs avec des fournisseurs tels qu’Azure Active Directory, Facebook et Twitter. Vous pouvez alors envoyer des messages directement aux utilisateurs authentifiés.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez obtenu une vue d’ensemble de l’utilisation d’Azure Functions avec le service SignalR pour permettre un large éventail de scénarios de messagerie serverless en temps réel.

Pour plus d’informations sur l’utilisation d’Azure Functions et de SignalR Service ensemble, consultez les ressources suivantes :

* [Développement et configuration Azure Functions avec SignalR Service](signalr-concept-serverless-development-config.md)
* [Activer les mises à jour automatiques dans une application web à l’aide d’Azure Functions et de SignalR Service](/learn/modules/automatic-update-of-a-webapp-using-azure-functions-and-signalr)

Suivez l’un de ces guides de démarrage rapide pour en savoir plus.

* [Démarrage rapide du service Azure SignalR serverless - C#](signalr-quickstart-azure-functions-csharp.md)
* [Démarrage rapide du service Azure SignalR serverless - JavaScript](signalr-quickstart-azure-functions-javascript.md)