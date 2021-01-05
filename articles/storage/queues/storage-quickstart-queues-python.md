---
title: 'Démarrage rapide : Bibliothèque de client Stockage File d’attente Azure v12 – Python'
description: Découvrez comment utiliser la bibliothèque de client Stockage File d’attente Azure v12 pour Python, afin de créer une file d’attente et d’y ajouter des messages. Apprenez ensuite à lire et à supprimer des messages de la file d’attente. Vous apprendrez également à supprimer une file d’attente.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 12/10/2019
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-python
ms.openlocfilehash: 0eeb1333952533cdf392b93a926f256097557151
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585494"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-python"></a>Démarrage rapide : Bibliothèque de client Stockage File d’attente Azure v12 pour Python

Familiarisez-vous avec la bibliothèque de client Storage File d’attente Azure v12 pour Python. Le Stockage File d’attente Azure est un service permettant de stocker un grand nombre de messages dans le but de les récupérer et de les traiter plus tard. Suivez les étapes suivantes pour installer le package et essayer un exemple de code pour les tâches de base.

Utilisez la bibliothèque de client Stockage File d’attente Azure v12 pour Python afin d’effectuer les opérations suivantes :

- Créer une file d’attente
- Ajouter des messages à une file d’attente
- Afficher un aperçu des messages d’une file d’attente
- Mettre à jour un message dans une file d’attente
- Réception des messages d'une file d'attente
- Supprimer des messages d’une file d’attente
- Suppression d'une file d'attente

Ressources supplémentaires :

- [Documentation de référence de l’API](/python/api/azure-storage-queue/index)
- [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue)
- [Package (Python Package Index)](https://pypi.org/project/azure-storage-queue/)
- [Exemples](../common/storage-samples-python.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/)
- Compte de stockage Azure : [créez un compte de stockage](../common/storage-account-create.md)
- [Python](https://www.python.org/downloads/) pour votre système d’exploitation - 2.7, 3.5 ou version ultérieure

## <a name="setting-up"></a>Configuration

Cette section vous guide tout au long de la préparation d’un projet à utiliser avec la bibliothèque de client Stockage File d’attente Azure v12 pour Python.

### <a name="create-the-project"></a>Créer le projet

Créez une application Python nommée `queues-quickstart-v12`.

1. Dans une fenêtre de console (telle que cmd, PowerShell ou Bash), créez un nouveau répertoire pour le projet.

    ```console
    mkdir queues-quickstart-v12
    ```

1. Basculez vers le répertoire `queues-quickstart-v12` créé.

    ```console
    cd queues-quickstart-v12
    ```

### <a name="install-the-package"></a>Installer le package

Installez le package de la bibliothèque de client Stockage Blob Azure pour Python à l’aide de la commande `pip install`.

```console
pip install azure-storage-queue
```

Cette commande installe le package de la bibliothèque de client Stockage File d’attente Azure pour Python et toutes les bibliothèques dont elle dépend. Dans ce cas, il s’agit seulement de la bibliothèque principale d’Azure pour Python.

### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

1. Ouvrez un nouveau fichier texte dans votre éditeur de code
1. Ajoutez les instructions `import`
1. Créez la structure du programme, y compris la gestion des exceptions de base

    Voici le code :

    ```python
    import os, uuid
    from azure.storage.queue import QueueServiceClient, QueueClient, QueueMessage

    try:
        print("Azure Queue storage v12 - Python quickstart sample")
        # Quick start code goes here
    except Exception as ex:
        print('Exception:')
        print(ex)

    ```

1. Enregistrez le nouveau fichier sous `queues-quickstart-v12.py` dans le répertoire `queues-quickstart-v12`.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Modèle objet

Stockage File d’attente Azure est un service permettant de stocker un grand nombre de messages. La taille maximale d’un message de file d’attente est de 64 Ko. Une file d’attente peut contenir des millions de messages, dans la limite de la capacité totale d’un compte de stockage. Les files d’attente sont couramment utilisées pour créer un backlog de travail à traiter de façon asynchrone. Le Stockage File d’attente offre trois types de ressources :

- Le compte de stockage
- Une file d’attente dans le compte de stockage
- Les messages dans la file d’attente

Le diagramme suivant montre la relation entre ces ressources.

![Diagramme de l’architecture du Stockage File d’attente](./media/storage-queues-introduction/queue1.png)

Utilisez les classes Python suivantes pour interagir avec ces ressources :

- [`QueueServiceClient`](/python/api/azure-storage-queue/azure.storage.queue.queueserviceclient) : `QueueServiceClient` vous permet de gérer toutes les files d’attente de votre compte de stockage.
- [`QueueClient`](/python/api/azure-storage-queue/azure.storage.queue.queueclient) : la classe `QueueClient` vous permet de gérer et de manipuler une file d’attente individuelle et ses messages.
- [`QueueMessage`](/python/api/azure-storage-queue/azure.storage.queue.queuemessage) : la classe `QueueMessage` représente les objets individuels retournés lors de l’appel de [`receive_messages`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-) dans une file d’attente.

## <a name="code-examples"></a>Exemples de code

Ces exemples d’extraits de code vous montrent comment effectuer les actions suivantes avec la bibliothèque de client Stockage File d’attente Azure pour Python :

- [Obtenir la chaîne de connexion](#get-the-connection-string)
- [Créer une file d’attente](#create-a-queue)
- [Ajouter des messages à une file d’attente](#add-messages-to-a-queue)
- [Afficher un aperçu des messages d’une file d’attente](#peek-at-messages-in-a-queue)
- [Mettre à jour un message dans une file d’attente](#update-a-message-in-a-queue)
- [Recevoir les messages d’une file d’attente](#receive-messages-from-a-queue)
- [Supprimer des messages d’une file d’attente](#delete-messages-from-a-queue)
- [Supprimer une file d’attente](#delete-a-queue)

### <a name="get-the-connection-string"></a>Obtenir la chaîne de connexion

Le code suivant récupère la chaîne de connexion du compte de stockage. La chaîne de connexion est stockée dans la variable d’environnement créée dans la section [Configurer votre chaîne de connexion de stockage](#configure-your-storage-connection-string).

Ajoutez ce code dans le bloc `try` :

```python
    # Retrieve the connection string for use with the application. The storage
    # connection string is stored in an environment variable on the machine
    # running the application called AZURE_STORAGE_CONNECTION_STRING. If the
    # environment variable is created after the application is launched in a
    # console or with Visual Studio, the shell or application needs to be
    # closed and reloaded to take the environment variable into account.
    connect_str = os.getenv('AZURE_STORAGE_CONNECTION_STRING')
```

### <a name="create-a-queue"></a>Créer une file d’attente

Choisissez un nom pour la nouvelle file d’attente. Le code suivant ajoute une valeur UUID au nom de la file d’attente pour s’assurer qu’il est unique.

> [!IMPORTANT]
> Les noms de file d’attente peuvent contenir uniquement des lettres minuscules, des chiffres et des traits d’union, et doivent commencer par une lettre ou un nombre. Chaque trait d’union doit être précédé et suivi d’un caractère autre qu’un tiret. Le nom doit avoir entre 3 et 63 caractères. Pour plus d’informations, consultez [Affectation de noms pour les files d’attente et les métadonnées](/rest/api/storageservices/naming-queues-and-metadata).

Créez une instance de la classe [`QueueClient`](/python/api/azure-storage-queue/azure.storage.queue.queueclient). Ensuite, appelez la méthode [`create_queue`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#create-queue---kwargs-) pour créer la file d’attente dans votre compte de stockage.

Ajoutez ce code à la fin du bloc `try` :

```python
    # Create a unique name for the queue
    queue_name = "quickstartqueues-" + str(uuid.uuid4())

    print("Creating queue: " + queue_name)

    # Instantiate a QueueClient which will be
    # used to create and manipulate the queue
    queue_client = QueueClient.from_connection_string(connect_str, queue_name)

    # Create the queue
    queue_client.create_queue()
```

### <a name="add-messages-to-a-queue"></a>Ajouter des messages à une file d’attente

L’extrait de code suivant ajoute des messages à la file d’attente en appelant la méthode [`send_message`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#send-message-content----kwargs-). Il enregistre également l’élément [`QueueMessage`](/python/api/azure-storage-queue/azure.storage.queue.queuemessage) retourné par le troisième appel de `send_message`. Le `saved_message` est utilisé pour mettre à jour le contenu du message ultérieurement dans le programme.

Ajoutez ce code à la fin du bloc `try` :

```python
    print("\nAdding messages to the queue...")

    # Send several messages to the queue
    queue_client.send_message(u"First message")
    queue_client.send_message(u"Second message")
    saved_message = queue_client.send_message(u"Third message")
```

### <a name="peek-at-messages-in-a-queue"></a>Afficher un aperçu des messages d’une file d’attente

Affichez un aperçu des messages de la file d’attente en appelant la méthode [`peek_messages`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#peek-messages-max-messages-none----kwargs-). Cette méthode récupère un ou plusieurs messages du début de la file d’attente, mais ne modifie pas la visibilité du message.

Ajoutez ce code à la fin du bloc `try` :

```python
    print("\nPeek at the messages in the queue...")

    # Peek at messages in the queue
    peeked_messages = queue_client.peek_messages(max_messages=5)

    for peeked_message in peeked_messages:
        # Display the message
        print("Message: " + peeked_message.content)
```

### <a name="update-a-message-in-a-queue"></a>Mettre à jour un message dans une file d’attente

Mettez à jour le contenu d’un message en appelant la méthode [`update_message`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#update-message-message--pop-receipt-none--content-none----kwargs-). Cette méthode peut changer le contenu et le délai d’expiration de la visibilité d’un message. Le contenu du message doit être une chaîne encodée en UTF-8 d’une taille maximale de 64 Ko. Avec le nouveau contenu, transmettez les valeurs du message qui a été enregistré dans le code. Les valeurs `saved_message` identifient le message à mettre à jour.

```python
    print("\nUpdating the third message in the queue...")

    # Update a message using the message saved when calling send_message earlier
    queue_client.update_message(saved_message, pop_receipt=saved_message.pop_receipt, \
        content="Third message has been updated")
```

### <a name="receive-messages-from-a-queue"></a>Réception des messages d'une file d'attente

Téléchargez les messages ajoutés en appelant la méthode [`receive_messages`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-).

Ajoutez ce code à la fin du bloc `try` :

```python
    print("\nReceiving messages from the queue...")

    # Get messages from the queue
    messages = queue_client.receive_messages(messages_per_page=5)
```

### <a name="delete-messages-from-a-queue"></a>Supprimer des messages d’une file d’attente

Supprimez les messages de la file d’attente une fois qu’ils sont reçus et traités. Dans ce cas, le traitement affiche simplement le message sur la console.

Avant de traiter et de supprimer les messages, l’application s’interrompt dans l’attente d’une entrée de l’utilisateur en appelant `input`. Vérifiez dans votre [portail Azure](https://portal.azure.com) que les ressources ont été créées correctement avant d’être supprimées. Tout message qui n’est pas supprimé explicitement sera de nouveau visible dans la file d’attente et pourra donc faire l’objet d’un nouveau traitement.

Ajoutez ce code à la fin du bloc `try` :

```python
    print("\nPress Enter key to 'process' messages and delete them from the queue...")
    input()

    for msg_batch in messages.by_page():
            for msg in msg_batch:
                # "Process" the message
                print(msg.content)
                # Let the service know we're finished with
                # the message and it can be safely deleted.
                queue_client.delete_message(msg)
```

### <a name="delete-a-queue"></a>Suppression d'une file d'attente

Le code suivant nettoie les ressources créées par l’application en supprimant la file d’attente avec la méthode [`delete_queue`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#delete-queue---kwargs-).

Ajoutez ce code à la fin du bloc `try` et enregistrez le fichier :

```python
    print("\nPress Enter key to delete the queue...")
    input()

    # Clean up
    print("Deleting queue...")
    queue_client.delete_queue()

    print("Done")
```

## <a name="run-the-code"></a>Exécuter le code

Cette application crée trois messages et les ajoute à une file d’attente Azure. Le code liste les messages dans la file d’attente, puis les récupère et les supprime avant de supprimer la file d’attente.

Dans la fenêtre de votre console, accédez au répertoire contenant le fichier `queues-quickstart-v12.py`, puis utilisez la commande `python` suivante pour exécuter l’application.

```console
python queues-quickstart-v12.py
```

La sortie de l’application ressemble à l’exemple suivant :

```output
Azure Queue Storage client library v12 - Python quickstart sample
Creating queue: quickstartqueues-cac365be-7ce6-4065-bd65-3756ea052cb8

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Receiving messages from the queue...

Press Enter key to 'process' messages and delete them from the queue...

First message
Second message
Third message has been updated

Press Enter key to delete the queue...

Deleting queue...
Done
```

Quand l’application s’interrompt avant de recevoir des messages, vérifiez votre compte de stockage dans le [portail Azure](https://portal.azure.com). Vérifiez que les messages se trouvent dans la file d’attente.

Appuyez sur la touche `Enter` pour recevoir et supprimer les messages. Quand vous y êtes invité, réappuyez sur la touche `Enter` pour supprimer la file d’attente et terminer la démonstration.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez appris à créer une file d’attente et à y ajouter des messages à l’aide de code Python. Ensuite, vous avez appris à afficher un aperçu des messages, à les récupérer et à les supprimer. Enfin, vous avez appris à supprimer une file d’attente de messages.

Pour obtenir des tutoriels, des exemples, des guides de démarrage rapide et d’autres documents, visitez :

> [!div class="nextstepaction"]
> [Azure for Python developers](/azure/python/) (Azure pour les développeurs Python)

- Pour plus d’informations, consultez les [bibliothèques Stockage Azure pour Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage).
- Pour d’autres exemples d’applications Stockage File d’attente Azure, consultez [Exemples de bibliothèques de client Stockage File d’attente Azure v12 pour Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue/samples).
