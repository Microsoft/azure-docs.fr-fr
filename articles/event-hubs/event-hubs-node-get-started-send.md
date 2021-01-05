---
title: Recevoir des événements d’Azure Event Hubs ou lui en envoyer en utilisant JavaScript (dernière version)
description: Cet article décrit la procédure à suivre pour créer une application JavaScript qui reçoit des événements d’Azure Event Hubs et lui en envoie à l’aide du dernier package azure/event-hubs.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-js
ms.openlocfilehash: 2e5f297d5edb9a271843db060f948209dd076074
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97106085"
---
# <a name="send-events-to-or-receive-events-from-event-hubs-by-using-javascript--azureevent-hubs"></a>Recevoir des événements des hubs d’événements ou leur en envoyer à l’aide de JavaScript (azure/event-hubs)
Ce guide de démarrage rapide montre comment recevoir des événements d’un hub d’événements et lui en envoyer à l’aide du package JavaScript **azure/event-hubs**. 


## <a name="prerequisites"></a>Prérequis
Si vous débutez avec Azure Event Hubs, consultez la [vue d’ensemble d’Event Hubs](event-hubs-about.md) avant de suivre ce guide de démarrage rapide. 

Pour effectuer ce démarrage rapide, vous avez besoin de ce qui suit :

- **Abonnement Microsoft Azure**. Pour utiliser les services Azure, y compris Azure Event Hubs, vous avez besoin d’un abonnement.  Si vous n’avez pas de compte Azure, vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/free/) ou utiliser les avantages de votre abonnement MSDN quand vous [créez un compte](https://azure.microsoft.com).
- Node.j version 8.x ou ultérieure. Téléchargez la [version LTS (prise en charge à long terme)](https://nodejs.org) la plus récente.  
- Visual Studio Code (recommandé) ou tout autre environnement de développement intégré (IDE).  
- Un hub d’événements et un espace de noms Event Hubs actifs. Pour les créer, effectuez les étapes suivantes : 

   1. Dans le [portail Azure](https://portal.azure.com), créez un espace de noms de type *Event Hubs*, puis obtenez les informations de gestion nécessaires à votre application pour communiquer avec le hub d’événements. 
   1. Pour créer l’espace de noms et le hub d’événements, suivez les instructions du [Guide de démarrage rapide : Créer un hub d’événement à l’aide du portail Azure](event-hubs-create.md).
   1. Continuez en suivant les instructions de ce guide de démarrage rapide. 
   1. Pour obtenir la chaîne de connexion de votre espace de noms Event Hub, suivez les instructions indiquées dans [Obtenir une chaîne de connexion](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Enregistrez cette chaîne de connexion pour l’utiliser plus tard dans ce guide de démarrage rapide.
- **Créez un espace de noms Event Hubs et un Event Hub**. La première étape consiste à utiliser le [portail Azure](https://portal.azure.com) pour créer un espace de noms de type Event Hubs et obtenir les informations de gestion nécessaires à votre application pour communiquer avec le concentrateur d’événements. Pour créer un espace de noms et un hub d’événements, suivez la procédure décrite dans [cet article](event-hubs-create.md). Ensuite, obtenez la **chaîne de connexion de l’espace de noms Event Hubs** en suivant les instructions à partir de l’article : [Obtenir la chaîne de connexion](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Vous utiliserez la chaîne de connexion plus tard dans ce guide de démarrage rapide.

### <a name="install-the-npm-package"></a>Installer le package npm
Pour installer le [package npm (Node Package Manager) d’Event Hubs](https://www.npmjs.com/package/@azure/event-hubs), ouvrez une invite de commandes dont le chemin d’accès contient *npm*, remplacez le répertoire par le dossier où vous souhaitez conserver vos exemples, puis exécutez cette commande :

```shell
npm install @azure/event-hubs
```

Pour le côté récepteur, vous avez besoin d’installer deux autres packages. Dans ce guide de démarrage rapide, vous utilisez le stockage Blob Azure pour conserver les points de contrôle afin que le programme ne lise pas les événements qu’il a déjà lus. Il effectue des points de contrôle de métadonnées sur les messages reçus à intervalles réguliers dans un blob. Cette approche permet ultérieurement de continuer à recevoir des messages à partir du point où vous vous étiez arrêté.

Exécutez les commandes suivantes :

```shell
npm install @azure/storage-blob
```

```shell
npm install @azure/eventhubs-checkpointstore-blob
```

## <a name="send-events"></a>Envoyer des événements

Dans cette section, vous créez une application JavaScript qui envoie des événements à un hub d’événements.

1. Ouvrez votre éditeur favori, tel que [Visual Studio Code](https://code.visualstudio.com).
1. Créez un fichier appelé *send.js* et collez-y le code suivant :

    ```javascript
    const { EventHubProducerClient } = require("@azure/event-hubs");

    const connectionString = "EVENT HUBS NAMESPACE CONNECTION STRING";
    const eventHubName = "EVENT HUB NAME";

    async function main() {

      // Create a producer client to send messages to the event hub.
      const producer = new EventHubProducerClient(connectionString, eventHubName);

      // Prepare a batch of three events.
      const batch = await producer.createBatch();
      batch.tryAdd({ body: "First event" });
      batch.tryAdd({ body: "Second event" });
      batch.tryAdd({ body: "Third event" });    

      // Send the batch to the event hub.
      await producer.sendBatch(batch);

      // Close the producer client.
      await producer.close();

      console.log("A batch of three events have been sent to the event hub");
    }

    main().catch((err) => {
      console.log("Error occurred: ", err);
    });
    ```
1. Dans le code, utilisez des valeurs réelles pour remplacer les valeurs suivantes :
    * `EVENT HUBS NAMESPACE CONNECTION STRING` 
    * `EVENT HUB NAME`
1. Exécutez `node send.js` pour exécuter ce fichier. Cette commande envoie un lot de trois événements à votre hub d’événements.
1. Dans le portail Azure, vérifiez que le hub d’événements a reçu les messages. Dans la section **Métriques**, basculez vers l’affichage **Messages**. Actualisez la page pour mettre à jour le graphique. Cela peut prendre quelques secondes pour indiquer que les messages ont été reçus.

    [![Vérifier que le hub d’événements a reçu les messages](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png)](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png#lightbox)

    > [!NOTE]
    > Pour obtenir le code source complet, y compris des commentaires d’informations supplémentaires, accédez à la [page GitHub sendEvents.js](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/event-hubs/samples/javascript/sendEvents.js).

Félicitations ! Vous venez d’envoyer des événements à un hub d’événements.


## <a name="receive-events"></a>Recevoir des événements
Dans cette section, vous recevez des événements d’un hub d’événements à l’aide d’un magasin de points de contrôle de stockage Blob Azure dans une application JavaScript. Il effectue des points de contrôle de métadonnées sur les messages reçus à intervalles réguliers dans un blob Stockage Azure. Cette approche permet ultérieurement de continuer à recevoir des messages à partir du point où vous vous étiez arrêté.

> [!WARNING]
> Si vous exécutez ce code sur Azure Stack Hub, vous rencontrerez des erreurs d’exécution, sauf si vous ciblez une version spécifique de l’API Stockage. Ceci est dû au fait que le SDK Event Hubs utilise la dernière API Stockage Azure disponible dans Azure, qui peut ne pas être pas disponible sur votre plateforme Azure Stack Hub. Azure Stack Hub peut prendre en charge une autre version du SDK Stockage Blob que celles généralement disponibles sur Azure. Si vous utilisez Stockage Blob Azure comme magasin de points de contrôle, vérifiez la [version de l’API Stockage Azure prise en charge pour votre build Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) et ciblez cette version dans votre code. 
>
> Par exemple, si vous exécutez sur Azure Stack Hub version 2005, la version la plus élevée disponible pour le service Stockage est la version 2019-02-02. Par défaut, la bibliothèque de client du SDK Event Hubs utilise la version la plus récente disponible sur Azure (2019-07-07 au moment de la publication du SDK). Dans ce cas, en plus des étapes suivantes de cette section, vous devrez également ajouter du code pour cibler la version 2019-02-02 de l’API du service Stockage. Pour obtenir un exemple sur la façon de cibler une version spécifique de l’API de stockage, consultez les exemples [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript/receiveEventsWithApiSpecificStorage.js) et [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript/src/receiveEventsWithApiSpecificStorage.ts) sur GitHub. 


### <a name="create-an-azure-storage-account-and-a-blob-container"></a>Créer un compte de stockage Azure et un conteneur d’objets blob
Pour créer un compte de stockage Azure et un conteneur d’objets blob dans celui-ci, effectuez les actions suivantes :

1. [Créer un compte de stockage Azure](../storage/common/storage-account-create.md?tabs=azure-portal)  
2. [Créer un conteneur d’objets blob dans le compte de stockage](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)  
3. [Obtenir la chaîne de connexion au compte de stockage](../storage/common/storage-configure-connection-string.md)

Veillez à enregistrer la chaîne de connexion et le nom du conteneur en vue de les utiliser dans le code de réception.

### <a name="write-code-to-receive-events"></a>Écrire du code pour recevoir des événements

1. Ouvrez votre éditeur favori, tel que [Visual Studio Code](https://code.visualstudio.com).
1. Créez un fichier appelé *receive.js* et collez-y le code suivant :

    ```javascript
    const { EventHubConsumerClient } = require("@azure/event-hubs");
    const { ContainerClient } = require("@azure/storage-blob");    
    const { BlobCheckpointStore } = require("@azure/eventhubs-checkpointstore-blob");

    const connectionString = "EVENT HUBS NAMESPACE CONNECTION STRING";    
    const eventHubName = "EVENT HUB NAME";
    const consumerGroup = "$Default"; // name of the default consumer group
    const storageConnectionString = "AZURE STORAGE CONNECTION STRING";
    const containerName = "BLOB CONTAINER NAME";

    async function main() {
      // Create a blob container client and a blob checkpoint store using the client.
      const containerClient = new ContainerClient(storageConnectionString, containerName);
      const checkpointStore = new BlobCheckpointStore(containerClient);

      // Create a consumer client for the event hub by specifying the checkpoint store.
      const consumerClient = new EventHubConsumerClient(consumerGroup, connectionString, eventHubName, checkpointStore);

      // Subscribe to the events, and specify handlers for processing the events and errors.
      const subscription = consumerClient.subscribe({
          processEvents: async (events, context) => {
            for (const event of events) {
              console.log(`Received event: '${event.body}' from partition: '${context.partitionId}' and consumer group: '${context.consumerGroup}'`);
            }
            // Update the checkpoint.
            await context.updateCheckpoint(events[events.length - 1]);
          },

          processError: async (err, context) => {
            console.log(`Error : ${err}`);
          }
        }
      );

      // After 30 seconds, stop processing.
      await new Promise((resolve) => {
        setTimeout(async () => {
          await subscription.close();
          await consumerClient.close();
          resolve();
        }, 30000);
      });
    }

    main().catch((err) => {
      console.log("Error occurred: ", err);
    });    
    ```
1. Dans le code, utilisez des valeurs réelles pour remplacer les valeurs suivantes :
    - `EVENT HUBS NAMESPACE CONNECTION STRING`
    - `EVENT HUB NAME`
    - `AZURE STORAGE CONNECTION STRING`
    - `BLOB CONTAINER NAME`
1. Ensuite, exécutez `node receive.js` dans une invite de commandes pour exécuter ce fichier. La fenêtre doit afficher les messages relatifs aux événements reçus.

    > [!NOTE]
    > Pour obtenir le code source complet, y compris des commentaires d’informations supplémentaires, accédez à la [page GitHub receiveEventsUsingCheckpointStore.js](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript/receiveEventsUsingCheckpointStore.js).

Félicitations ! Vous recevez maintenant des événements depuis votre hub d’événements. Le programme récepteur recevra les événements de toutes les partitions du groupe de consommateurs par défaut dans le hub d’événements.

## <a name="next-steps"></a>Étapes suivantes
Consultez ces exemples sur GitHub :

- [Exemples JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/javascript)
- [Exemples TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/typescript)
