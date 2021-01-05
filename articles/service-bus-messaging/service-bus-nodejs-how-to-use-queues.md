---
title: Guide pratique pour utiliser des files d’attente azure/service-bus en JavaScript
description: Découvrez comment écrire un programme JavaScript qui utilise la préversion la plus récente du package @azure/service-bus pour envoyer et recevoir des messages à partir d’une file d’attente Service Bus.
author: spelluru
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 11/09/2020
ms.author: spelluru
ms.custom: devx-track-js
ms.openlocfilehash: 009b060cc479c520dde8c4f595b24be7dcb4c6a6
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347759"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-javascript"></a>Envoyer et recevoir des messages à partir de files d’attente Azure Service Bus (JavaScript)
Dans ce tutoriel, vous allez découvrir comment utiliser le package [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) dans un programme JavaScript pour envoyer et recevoir des messages à partir d’une file d’attente Service Bus.

## <a name="prerequisites"></a>Prérequis
- Un abonnement Azure. Pour suivre ce tutoriel, vous avez besoin d’un compte Azure. Vous pouvez [activer les avantages de votre abonnement MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) ou vous inscrire pour un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si vous n’avez pas de file d’attente à utiliser, suivez les étapes de l’article [Utiliser le portail Azure pour créer une file d’attente Service Bus](service-bus-quickstart-portal.md) pour créer une file d’attente. Notez la **chaîne de connexion** pour votre espace de noms Service Bus et le nom de la **file d’attente** que vous avez créée.

> [!NOTE]
> - Ce didacticiel utilise des exemples que vous pouvez copier et exécuter à l’aide de [Nodejs](https://nodejs.org/). Pour obtenir des instructions sur la création d’une application Node.js, veuillez consulter les pages [Création et déploiement d’une application Node.js sur un site web Azure](../app-service/quickstart-nodejs.md) ou [Service cloud Node.js avec Windows PowerShell](../cloud-services/cloud-services-nodejs-develop-deploy-app.md).

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Utilisation de Node Package Manager (NPM) pour installer le package
Pour installer le package npm pour Service Bus, ouvrez une invite de commandes dont le chemin d’accès contient `npm`, remplacez le répertoire par le dossier où vous souhaitez stocker vos exemples, puis exécutez cette commande.

```bash
npm install @azure/service-bus
```

## <a name="send-messages-to-a-queue"></a>Envoi de messages à une file d'attente
L’exemple de code suivant illustre comment envoyer un message à une file d’attente.

1. Ouvrez votre éditeur favori, tel que [Visual Studio Code](https://code.visualstudio.com/).
2. Créez un fichier appelé `send.js` et collez-y le code ci-dessous. Ce code envoie un message vers votre file d’attente. Le message comporte une étiquette (Scientist) et un corps (Einstein).

    ```javascript
    const { ServiceBusClient } = require("@azure/service-bus");
    
    // connection string to your Service Bus namespace
    const connectionString = "<CONNECTION STRING TO SERVICE BUS NAMESPACE>"

    // name of the queue
    const queueName = "<QUEUE NAME>"
    
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
     
        // createSender() can also be used to create a sender for a topic.
        const sender = sbClient.createSender(queueName);
     
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
    
            // Send the last created batch of messages to the queue
            await sender.sendMessages(batch);
     
            console.log(`Sent a batch of messages to the queue: ${queueName}`);
                    
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
3. Remplacez `<CONNECTION STRING TO SERVICE BUS NAMESPACE>` par la chaîne de connexion de votre espace de noms Service Bus,
1. et remplacez `<QUEUE NAME>` par le nom de la file d’attente. 
1. Ensuite, exécutez la commande à partir d’une invite de commandes pour exécuter ce fichier.

    ```console
    node send.js 
    ```
1. Vous devez voir la sortie suivante.

    ```console
    Sent a batch of messages to the queue: myqueue
    ```

## <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente

1. Ouvrez votre éditeur favori, tel que [Visual Studio Code](https://code.visualstudio.com/)
2. Créez un fichier appelé `receive.js` et collez-y le code suivant.

    ```javascript
    const { delay, ServiceBusClient, ServiceBusMessage } = require("@azure/service-bus");

    // connection string to your Service Bus namespace
    const connectionString = "<CONNECTION STRING TO SERVICE BUS NAMESPACE>"

    // name of the queue
    const queueName = "<QUEUE NAME>"

     async function main() {
        // create a Service Bus client using the connection string to the Service Bus namespace
        const sbClient = new ServiceBusClient(connectionString);
     
        // createReceiver() can also be used to create a receiver for a subscription.
        const receiver = sbClient.createReceiver(queueName);
    
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
3. Remplacez `<CONNECTION STRING TO SERVICE BUS NAMESPACE>` par la chaîne de connexion de votre espace de noms Service Bus,
1. et remplacez `<QUEUE NAME>` par le nom de la file d’attente. 
1. Ensuite, exécutez la commande à partir d’une invite de commandes pour exécuter ce fichier.

    ```console
    node receive.js
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

Dans la page **Vue d’ensemble** de l’espace de noms Service Bus dans le portail Azure, vous pouvez voir le nombre de messages **entrants** et **sortants**. Vous devrez peut-être attendre environ une minute puis actualiser la page pour voir les valeurs les plus récentes. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Nombre de messages entrants et sortants":::

Sélectionnez la file d’attente dans cette page **Vue d’ensemble** pour accéder à la page **File d’attente Service Bus**. Le nombre de messages **entrants** et **sortants** est visible dans cette page, ainsi que d’autres informations telles que la **taille actuelle** de la file d’attente, la **taille maximale** et le **nombre de messages actifs**. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/queue-details.png" alt-text="Détails de la file d’attente":::
## <a name="next-steps"></a>Étapes suivantes
Voir la documentation et les exemples suivants : 

- [Bibliothèque de client Azure Service Bus pour JS](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/servicebus/service-bus/README.md)
- [Exemples](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/servicebus/service-bus/samples). Le dossier **javascript** contient des exemples JavaScript, et le dossier **typescript** contient des exemples TypeScript. 
- [Documentation de référence sur azure-servicebus](/javascript/api/overview/azure/service-bus)
