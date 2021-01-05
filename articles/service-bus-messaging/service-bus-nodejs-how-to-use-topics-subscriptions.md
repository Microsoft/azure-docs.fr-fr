---
title: Utiliser la préversion d’azure/service-bus JavaScript avec des rubriques et des abonnements
description: Découvrez comment écrire un programme JavaScript qui utilise la préversion la plus récente du package @azure/service-bus pour envoyer des messages à une rubrique Service Bus et recevoir des messages à partir d’un abonnement à la rubrique.
author: spelluru
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 11/09/2020
ms.author: spelluru
ms.custom: devx-track-js
ms.openlocfilehash: aac3b6339c318c76e9b0c9abd0bc3778f2563a6f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498691"
---
# <a name="quickstart-service-bus-topics-and-subscriptions-with-nodejs-and-the-preview-azureservice-bus-package"></a>Démarrage rapide : Rubriques et abonnements Service Bus avec Node.js et le package azure/service-bus en préversion
Dans ce tutoriel, vous allez découvrir comment utiliser le package [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) dans un programme JavaScript pour envoyer des messages à une rubrique Service Bus et recevoir des messages à partir d’un abonnement Service Bus à cette rubrique.

## <a name="prerequisites"></a>Prérequis
- Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez [activer les avantages de votre abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) ou [vous inscrire pour un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Suivez les étapes dans [Démarrage rapide : Utiliser le portail Azure pour créer une rubrique Service Bus et des abonnements à cette rubrique](service-bus-quickstart-topics-subscriptions-portal.md). Notez la chaîne de connexion, le nom de la rubrique et un nom d’abonnement. Vous n’utiliserez qu’un seul abonnement dans ce guide de démarrage rapide. 

> [!NOTE]
> - Ce didacticiel utilise des exemples que vous pouvez copier et exécuter à l’aide de [Nodejs](https://nodejs.org/). Pour obtenir des instructions sur la création d’une application Node.js, veuillez consulter les pages [Création et déploiement d’une application Node.js sur un site web Azure](../app-service/quickstart-nodejs.md) ou [Service cloud Node.js avec Windows PowerShell](../cloud-services/cloud-services-nodejs-develop-deploy-app.md).

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Utilisation de Node Package Manager (NPM) pour installer le package
Pour installer le package npm pour Service Bus, ouvrez une invite de commandes dont le chemin d’accès contient `npm`, remplacez le répertoire par le dossier où vous souhaitez stocker vos exemples, puis exécutez cette commande.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-topic"></a>Envoi de messages à une rubrique
L’exemple de code suivant montre comment envoyer un lot de messages à une rubrique Service Bus. Pour plus d’informations, consultez les commentaires du code. 

1. Ouvrez votre éditeur favori, tel que [Visual Studio Code](https://code.visualstudio.com/)
2. Créez un fichier appelé `sendtotopic.js` et collez-y le code ci-dessous. Ce code envoie un message vers votre rubrique.

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus");
    
    const connectionString = "<SERVICE BUS NAMESPACE CONNECTION STRING>"
    const topicName = "<TOPIC NAME>";
    
    const messages = [
        { body: "Albert Einstein" },
        { body: "Werner Heisenberg" },
        { body: "Marie Curie" },
        { body: "Steven Hawking" },
        { body: "Isaac Newton" },
        { body: "Niels Bohr" },
        { body: "Michael Faraday" },
        { body: "Galileo Galilei" },
        { body: "Johannes Kepler" },
        { body: "Nikolaus Kopernikus" }
     ];
    
     async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createSender() can also be used to create a sender for a queue.
        const sender = sbClient.createSender(topicName);
     
        try {
            // Tries to send all messages in a single batch.
            // Will fail if the messages cannot fit in a batch.
            // await sender.sendMessages(messages);
     
            // create a batch object
            let batch = await sender.createMessageBatch(); 
            for (let i = 0; i < messages.length; i++) {
                // for each message in the arry         
    
                // try to add the message to the batch
                if (!batch.tryAddMessage(messages[i])) {            
                    // if it fails to add the message to the current batch
                    // send the current batch as it is full
                    await sender.sendMessages(batch);
    
                    // then, create a new batch 
                    batch = await sender.createBatch();
     
                    // now, add the message failed to be added to the previous batch to this batch
                    if (!batch.tryAddMessage(messages[i])) {
                        // if it still can't be added to the batch, the message is probably too big to fit in a batch
                        throw new Error("Message too big to fit in a batch");
                    }
                }
            }
    
            // Send the last created batch of messages to the topic
            await sender.sendMessages(batch);
     
            console.log(`Sent a batch of messages to the topic: ${topicName}`);
                    
            // Close the sender
            await sender.close();
        } finally {
            await sbClient.close();
        }
    }
    
    // call the main function
    main().catch((err) => {
        console.log("Error occurred: ", err);
        process.exit(1);
     });    
    ```
3. Remplacez `<SERVICE BUS NAMESPACE CONNECTION STRING>` par la chaîne de connexion de votre espace de noms Service Bus,
1. et remplacez `<TOPIC NAME>` par le nom de la rubrique. 
1. Ensuite, exécutez la commande à partir d’une invite de commandes pour exécuter ce fichier.

    ```console
    node sendtotopic.js 
    ```
1. Vous devez voir la sortie suivante.

    ```console
    Sent a batch of messages to the topic: mytopic
    ```

## <a name="receive-messages-from-a-subscription"></a>Réception des messages d’un abonnement
1. Ouvrez votre éditeur favori, tel que [Visual Studio Code](https://code.visualstudio.com/)
2. Créez un fichier nommé **receivefromsubscription.js** et collez-y le code suivant. Pour plus d’informations, consultez les commentaires du code. 

    ```javascript
    const { delay, ServiceBusClient, ServiceBusMessage } = require("@azure/service-bus");
    
    const connectionString = "<SERVICE BUS NAMESPACE CONNECTION STRING>"
    const topicName = "<TOPIC NAME>";
    const subscriptionName = "<SUBSCRIPTION NAME>";
    
     async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createReceiver() can also be used to create a receiver for a queue.
        const receiver = sbClient.createReceiver(topicName, subscriptionName);
    
        // function to handle messages
        const myMessageHandler = async (messageReceived) => {
            console.log(`Received message: ${messageReceived.body}`);
        };
    
        // function to handle any errors
        const myErrorHandler = async (error) => {
            console.log(error);
        };
    
        // subscribe and specify the message and error handlers
        receiver.subscribe({
            processMessage: myMessageHandler,
            processError: myErrorHandler
        });
    
        // Waiting long enough before closing the sender to send messages
        await delay(5000);
    
        await receiver.close(); 
        await sbClient.close();
    }
    
    // call the main function
    main().catch((err) => {
        console.log("Error occurred: ", err);
        process.exit(1);
     });    
    ```
3. Remplacez `<SERVICE BUS NAMESPACE CONNECTION STRING>` par la chaîne de connexion à l’espace de noms, 
1. et remplacez `<TOPIC NAME>` par le nom de la rubrique. 
1. Remplacez `<SUBSCRIPTION NAME>` par le nom de l’abonnement à la rubrique. 
1. Ensuite, exécutez la commande à partir d’une invite de commandes pour exécuter ce fichier.

    ```console
    node receivefromsubscription.js
    ```
1. Vous devez voir la sortie suivante.

    ```console
    Received message: Albert Einstein
    Received message: Werner Heisenberg
    Received message: Marie Curie
    Received message: Steven Hawking
    Received message: Isaac Newton
    Received message: Niels Bohr
    Received message: Michael Faraday
    Received message: Galileo Galilei
    Received message: Johannes Kepler
    Received message: Nikolaus Kopernikus
    ```

Dans le portail Azure, accédez à votre espace de noms Service Bus et sélectionnez la rubrique dans le volet inférieur pour afficher la page **Rubrique Service Bus** correspondant à votre rubrique. Dans cette page, vous devez voir trois messages entrants et trois messages sortants dans le graphique **Messages**. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/topic-page-portal.png" alt-text="Messages entrants et sortants":::

Si la fois suivante vous exécutez uniquement l’application d’envoi, la page **Rubrique Service Bus** mentionne six messages entrants (trois nouveaux) mais trois messages sortants. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/updated-topic-page.png" alt-text="Page de rubrique mise à jour":::

Dans cette page, si vous sélectionnez un abonnement, vous accédez à la page **Abonnement Service Bus**. Elle indique entre autres le nombre de messages actifs et le nombre de messages de lettres mortes. Dans cet exemple, trois messages actifs n’ont pas encore été reçus par un récepteur. 

:::image type="content" source="./media/service-bus-java-how-to-use-topics-subscriptions/active-message-count.png" alt-text="Nombre de messages actifs":::

## <a name="next-steps"></a>Étapes suivantes
Voir la documentation et les exemples suivants : 

- [Bibliothèque de client Azure Service Bus pour Python](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/servicebus/service-bus/README.md)
- [Exemples](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/samples). Le dossier **JavaScript** contient des exemples JavaScript, et le dossier **typescript** contient des exemples TypeScript. 
- [Documentation de référence sur azure-servicebus](/javascript/api/overview/azure/service-bus)