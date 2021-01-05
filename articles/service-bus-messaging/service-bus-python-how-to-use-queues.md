---
title: Utiliser des files d’attente Azure Service Bus avec le package Python azure-servicebus version 7.0.0
description: Cet article explique comment utiliser Python pour envoyer et recevoir des messages à partir des files d’attente Azure Service Bus.
author: spelluru
documentationcenter: python
ms.devlang: python
ms.topic: quickstart
ms.date: 11/18/2020
ms.author: spelluru
ms.custom: seo-python-october2019, devx-track-python
ms.openlocfilehash: 7275e33e44c20ece6eb7d620e2c1e8032be41a7b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498657"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-python"></a>Envoyer et recevoir des messages à partir de files d’attente Azure Service Bus (Python)
Cet article explique comment utiliser Python pour envoyer et recevoir des messages à partir des files d’attente Azure Service Bus. 

## <a name="prerequisites"></a>Prérequis
- Un abonnement Azure. Vous pouvez activer les [avantages de votre abonnement Visual Studio ou MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) ou vous inscrire pour créer un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si vous n’avez pas de file d’attente à utiliser, suivez les étapes de l’article [Utiliser le portail Azure pour créer une file d’attente Service Bus](service-bus-quickstart-portal.md) pour créer une file d’attente. Notez la **chaîne de connexion** pour votre espace de noms Service Bus et le nom de la **file d’attente** que vous avez créée.
- Python 2.7 ou version ultérieure, avec le package [Python Azure Service Bus](https://pypi.python.org/pypi/azure-servicebus) installé. Pour plus d’informations, consultez le [Guide d’installation Python](/azure/developer/python/azure-sdk-install). 

## <a name="send-messages-to-a-queue"></a>Envoi de messages à une file d'attente

1. Ajoutez l’instruction d’importation suivante : 

    ```python
    from azure.servicebus import ServiceBusClient, ServiceBusMessage
    ```
2. Ajoutez les constantes suivantes. 

    ```python
    CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
    QUEUE_NAME = "<QUEUE NAME>"
    ```

    > [!IMPORTANT]
    > - Remplacez `<NAMESPACE CONNECTION STRING>` par la chaîne de connexion de votre espace de noms Service Bus,
    > - et remplacez `<QUEUE NAME>` par le nom de la file d’attente. 
3. Ajoutez une méthode pour envoyer un message unique.

    ```python
    def send_single_message(sender):
        # create a Service Bus message
        message = ServiceBusMessage("Single Message")
        # send the message to the queue
        sender.send_messages(message)
        print("Sent a single message")
    ```

    L’expéditeur est un objet qui agit en tant que client pour la file d’attente que vous avez créée. Vous le créerez ultérieurement et l’enverrez en tant qu’argument à cette fonction. 
4. Ajoutez une méthode pour envoyer une liste de messages.

    ```python
    def send_a_list_of_messages(sender):
        # create a list of messages
        messages = [ServiceBusMessage("Message in list") for _ in range(5)]
        # send the list of messages to the queue
        sender.send_messages(messages)
        print("Sent a list of 5 messages")
    ```
5. Ajoutez une méthode pour envoyer un lot de messages.

    ```python
    def send_batch_message(sender):
        # create a batch of messages
        batch_message = sender.create_message_batch()
        for _ in range(10):
            try:
                # add a message to the batch
                batch_message.add_message(ServiceBusMessage("Message inside a ServiceBusMessageBatch"))
            except ValueError:
                # ServiceBusMessageBatch object reaches max_size.
                # New ServiceBusMessageBatch object can be created here to send more data.
                break
        # send the batch of messages to the queue
        sender.send_messages(batch_message)
        print("Sent a batch of 10 messages")
    ```
6. Créez un client Service Bus, puis un objet expéditeur de file d’attente pour envoyer des messages.

    ```python
    # create a Service Bus client using the connection string
    servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)
    with servicebus_client:
        # get a Queue Sender object to send messages to the queue
        sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)
        with sender:
            # send one message        
            send_single_message(sender)
            # send a list of messages
            send_a_list_of_messages(sender)
            # send a batch of messages
            send_batch_message(sender)
    
    print("Done sending messages")
    print("-----------------------")
    ```
 
## <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente
Ajoutez le code suivant après l’instruction print. Ce code reçoit continuellement de nouveaux messages jusqu’à ce qu’il n’en reçoive plus pendant cinq (`max_wait_time`) secondes. 

```python
with servicebus_client:
    # get the Queue Receiver object for the queue
    receiver = servicebus_client.get_queue_receiver(queue_name=QUEUE_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            # complete the message so that the message is removed from the queue
            receiver.complete_message(msg)
```

## <a name="full-code"></a>Code complet

```python
# import os
from azure.servicebus import ServiceBusClient, ServiceBusMessage

CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
QUEUE_NAME = "<QUEUE NAME>"

def send_single_message(sender):
    message = ServiceBusMessage("Single Message")
    sender.send_messages(message)
    print("Sent a single message")

def send_a_list_of_messages(sender):
    messages = [ServiceBusMessage("Message in list") for _ in range(5)]
    sender.send_messages(messages)
    print("Sent a list of 5 messages")

def send_batch_message(sender):
    batch_message = sender.create_message_batch()
    for _ in range(10):
        try:
            batch_message.add_message(ServiceBusMessage("Message inside a ServiceBusMessageBatch"))
        except ValueError:
            # ServiceBusMessageBatch object reaches max_size.
            # New ServiceBusMessageBatch object can be created here to send more data.
            break
    sender.send_messages(batch_message)
    print("Sent a batch of 10 messages")

servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)

with servicebus_client:
    sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)
    with sender:
        send_single_message(sender)
        send_a_list_of_messages(sender)
        send_batch_message(sender)

print("Done sending messages")
print("-----------------------")

with servicebus_client:
    receiver = servicebus_client.get_queue_receiver(queue_name=QUEUE_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            receiver.complete_message(msg)
```

## <a name="run-the-app"></a>Exécuter l’application
Lorsque vous exécutez l’application, vous devez voir la sortie suivante : 

```console
Sent a single message
Sent a list of 5 messages
Sent a batch of 10 messages
Done sending messages
-----------------------
Received: Single Message
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
```

Dans le portail Azure, accédez à votre espace de noms Service Bus. Dans la page **Vue d’ensemble**, vérifiez que le nombre de messages **entrants** et **sortants** est égal à 16. Si ces chiffres ne s’affichent pas, actualisez la page après quelques minutes. 

:::image type="content" source="./media/service-bus-python-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Nombre de messages entrants et sortants":::

Sélectionnez la file d’attente dans cette page **Vue d’ensemble** pour accéder à la page **File d’attente Service Bus**. Vous pouvez également voir le nombre de messages **entrants** et **sortants** dans cette page, ainsi que d’autres informations telles que la **taille actuelle** de la file d’attente et le **nombre de messages actifs**. 

:::image type="content" source="./media/service-bus-python-how-to-use-queues/queue-details.png" alt-text="Détails de la file d’attente":::


## <a name="next-steps"></a>Étapes suivantes
Voir la documentation et les exemples suivants : 

- [Bibliothèque de client Azure Service Bus pour Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus)
- [Exemples](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus/samples). 
    - Le dossier **sync_samples** contient des exemples qui montrent comment interagir avec Service Bus de manière synchrone. Dans ce guide de démarrage rapide, vous avez utilisé cette méthode. 
    - Le dossier **async_samples** contient des exemples qui montrent comment interagir avec Service Bus de manière asynchrone. 
- [Documentation de référence sur azure-servicebus](/python/api/azure-servicebus/azure.servicebus?preserve-view=true&view=azure-python-preview)