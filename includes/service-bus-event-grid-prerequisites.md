---
title: Fichier include
description: Fichier include
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 10/15/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: d12df7197945a514ed8d3d0dca77271fb4bd0903
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96509247"
---
## <a name="prerequisites"></a>Prérequis
Si vous n’avez pas [d’abonnement Azure](../articles/guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

## <a name="create-a-service-bus-namespace"></a>Création d’un espace de noms Service Bus
Suivez les instructions de ce tutoriel : [Démarrage rapide : utiliser le portail Azure pour créer une rubrique Service Bus et des abonnements à cette rubrique](../articles/service-bus-messaging/service-bus-quickstart-topics-subscriptions-portal.md) pour effectuer les tâches suivantes :

- Créer un espace de noms Service Bus **premium**. 
- Obtenir la chaîne de connexion. 
- Créer une rubrique Service Bus.
- Créer un abonnement à la rubrique. Vous n’avez besoin que d’un seul abonnement dans ce tutoriel. Il n’est donc pas nécessaire de créer des abonnements S2 et S3 

## <a name="send-messages-to-the-service-bus-topic"></a>Envoyer des messages à la rubrique Service Bus
Lors de cette étape, vous allez utiliser un exemple d’application pour envoyer des messages à la rubrique Service Bus créée à l’étape précédente. 

1. Clonez le [dépôt GitHub azure-service-bus](https://github.com/Azure/azure-service-bus/).
2. Dans Visual Studio, accédez au dossier *\samples\DotNet\Azure.Messaging.ServiceBus\ServiceBusEventGridIntegration*, puis ouvrez le fichier *SBEventGridIntegration.sln*.
3. Dans la fenêtre Explorateur de solutions, développez le projet **MessageSender**, puis sélectionnez **Program.cs**.
4. Remplacez `<SERVICE BUS NAMESPACE - CONNECTION STRING>` par la chaîne de connexion à votre espace de noms Service Bus et `<TOPIC NAME>` par le nom de la rubrique. 

    ```csharp
    const string ServiceBusConnectionString = "<SERVICE BUS NAMESPACE - CONNECTION STRING>";
    const string TopicName = "<TOPIC NAME>";
    ```
5. Générez et exécutez le programme pour envoyer cinq messages de test (`const int numberOfMessages = 5;`) à la rubrique Service Bus. 

    :::image type="content" source="./media/service-bus-event-grid-prerequisites/console-app-output.png" alt-text="Sortie d’application console":::
