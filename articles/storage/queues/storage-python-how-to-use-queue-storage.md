---
title: Guide pratique pour utiliser Stockage Files d’attente à partir de Python - Stockage Azure
description: Apprenez à utiliser le service de File d'attente Azure à partir de Python pour créer et supprimer des files d'attente, ainsi que pour insérer, récupérer et supprimer des messages.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 08/25/2020
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: seo-javascript-october2019, devx-track-python
ms.openlocfilehash: ac75b89548d346945901d752672ef0f08601ccfb
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93345650"
---
# <a name="how-to-use-azure-queue-storage-from-python"></a>Guide pratique pour utiliser Stockage Files d’attente à partir de Python

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>Vue d’ensemble

Cet article présente des scénarios courants d’utilisation du service Stockage File d’attente Azure. Les scénarios traités comprennent l'insertion, la consultation rapide, la récupération et la suppression de messages de la file d'attente. Le code de création et de suppression de files d'attente est également abordé.

Les exemples de cet article sont écrits en Python et utilisent la [bibliothèque de client Stockage File d'attente Azure pour Python]. Pour plus d’informations sur les files d’attente, consultez la section [Étapes suivantes](#next-steps).

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="download-and-install-azure-storage-sdk-for-python"></a>Télécharger et installer le SDK Stockage Azure pour Python

Le [SDK Stockage Azure pour Python](https://github.com/azure/azure-storage-python) nécessite Python versions 2.7, 3.3 et ultérieures.

### <a name="install-via-pypi"></a>Effectuer l'installation via PyPi

Pour effectuer l’installation via PyPI (Python Package Index), tapez :

# <a name="python-v12"></a>[Python v12](#tab/python)

```console
pip install azure-storage-queue
```

# <a name="python-v2"></a>[Python v2](#tab/python2)

```console
pip install azure-storage-queue==2.1.0
```

---

> [!NOTE]
> Si vous mettez à niveau la version 0.36 ou antérieure du SDK Stockage Azure pour Python, désinstallez l’ancien SDK à l’aide de `pip uninstall azure-storage` avant d’installer le dernier package.

Pour connaître les autres méthodes d'installation, consultez [SDK Azure pour Python].

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="configure-your-application-to-access-queue-storage"></a>Configuration de votre application pour accéder au stockage de files d'attente

# <a name="python-v12"></a>[Python v12](#tab/python)

L'objet [QueueClient](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient) vous permet d'utiliser une file d'attente. Ajoutez le code suivant au début de chaque fichier Python dans lequel vous souhaitez accéder à une file d'attente Azure par programme :

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_ImportStatements":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

L'objet [QueueService](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2) permet d'utiliser des files d'attente. Le code suivant crée un objet `QueueService`. Ajoutez le code suivant au début de chaque fichier Python dans lequel vous souhaitez accéder à Stockage Azure par programme :

```python
from azure.storage.queue import (
        QueueService,
        QueueMessageFormat
)

import os, uuid
```

---

Le package `os` permet d'extraire une variable d'environnement. Le package `uuid` permet de générer un identificateur unique pour un nom de file d'attente.

## <a name="create-a-queue"></a>Créer une file d’attente

La chaîne de connexion est extraite de la variable d'environnement `AZURE_STORAGE_CONNECTION_STRING` définie précédemment.

# <a name="python-v12"></a>[Python v12](#tab/python)

Le code suivant crée un objet `QueueClient` à l'aide de la chaîne de connexion de stockage.

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_CreateQueue":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Le code suivant crée un objet `QueueService` à l'aide de la chaîne de connexion de stockage.

```python
# Retrieve the connection string from an environment
# variable named AZURE_STORAGE_CONNECTION_STRING
connect_str = os.getenv("AZURE_STORAGE_CONNECTION_STRING")

# Create a unique name for the queue
queue_name = "queue-" + str(uuid.uuid4())

# Create a QueueService object which will
# be used to create and manipulate the queue
print("Creating queue: " + queue_name)
queue_service = QueueService(connection_string=connect_str)

# Create the queue
queue_service.create_queue(queue_name)
```

---

## <a name="insert-a-message-into-a-queue"></a>Insertion d'un message dans une file d'attente

# <a name="python-v12"></a>[Python v12](#tab/python)

Pour insérer un message dans une file d'attente, utilisez la méthode [send_message](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#send-message-content----kwargs-).

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_AddMessage":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Pour insérer un message dans une file d’attente, utilisez la méthode [put_message](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#put-message-queue-name--content--visibility-timeout-none--time-to-live-none--timeout-none-) afin de créer un message et l’ajouter à la file d’attente.

```python
message = u"Hello World"
print("Adding message: " + message)
queue_service.put_message(queue_name, message)
```

---

Les messages de la file d’attente Azure sont stockés sous forme de texte. Si vous souhaitez stocker des données binaires, configurez les fonctions d'encodage et de décodage Base64 avant de placer un message dans la file d'attente.

# <a name="python-v12"></a>[Python v12](#tab/python)

Configurez les fonctions d'encodage et de décodage Base64 sur l'objet client de la file d'attente.

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_EncodeMessage":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Configurez les fonctions d'encodage et de décodage Base64 sur l'objet service de la file d'attente.

```python
# Setup Base64 encoding and decoding functions
queue_service.encode_function = QueueMessageFormat.binary_base64encode
queue_service.decode_function = QueueMessageFormat.binary_base64decode
```

---

## <a name="peek-at-messages"></a>Afficher un aperçu des messages

# <a name="python-v12"></a>[Python v12](#tab/python)

Vous pouvez lire furtivement les messages sans les supprimer de la file d'attente en appelant la méthode [peek_messages](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#peek-messages-max-messages-none----kwargs-). Par défaut, `peek_messages` lit furtivement un seul message.

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_PeekMessage":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Vous pouvez lire furtivement les messages sans les supprimer de la file d'attente en appelant la méthode [peek_messages](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#peek-messages-queue-name--num-messages-none--timeout-none-). Par défaut, `peek_messages` lit furtivement un seul message.

```python
messages = queue_service.peek_messages(queue_name)

for peeked_message in messages:
    print("Peeked message: " + peeked_message.content)
```

---

## <a name="change-the-contents-of-a-queued-message"></a>Modification du contenu d'un message en file d'attente

Vous pouvez modifier le contenu d'un message placé dans la file d'attente. Si le message représente une tâche, vous pouvez utiliser cette fonctionnalité pour mettre à jour l'état de la tâche.

# <a name="python-v12"></a>[Python v12](#tab/python)

Le code ci-dessous utilise la méthode [update_message](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#update-message-message--pop-receipt-none--content-none----kwargs-) pour mettre à jour un message. Ce délai de visibilité est défini sur 0, ce qui signifie que le message s’affiche immédiatement et que le contenu est mis à jour.

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_ChangeMessage":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Le code ci-dessous utilise la méthode [update_message](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#update-message-queue-name--message-id--pop-receipt--visibility-timeout--content-none--timeout-none-) pour mettre à jour un message. Ce délai de visibilité est défini sur 0, ce qui signifie que le message s’affiche immédiatement et que le contenu est mis à jour.

```python
messages = queue_service.get_messages(queue_name)

for message in messages:
    queue_service.update_message(
        queue_name, message.id, message.pop_receipt, 0, u"Hello World Again")
```

---

## <a name="get-the-queue-length"></a>Obtention de la longueur de la file d'attente

Vous pouvez obtenir une estimation du nombre de messages dans une file d'attente.

# <a name="python-v12"></a>[Python v12](#tab/python)

La méthode [get_queue_properties](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#get-queue-properties---kwargs-) demande au service de file d'attente de renvoyer des propriétés relatives à la file d'attente, y compris `approximate_message_count`.

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_GetQueueLength":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

La méthode [get_queue_metadata](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#get-queue-metadata-queue-name--timeout-none-) demande au service de file d'attente de renvoyer les métadonnées relatives à la file d'attente, y compris `approximate_message_count`.

```python
metadata = queue_service.get_queue_metadata(queue_name)
count = metadata.approximate_message_count
print("Message count: " + str(count))
```

---

Le résultat est seulement approximatif, car des messages peuvent être ajoutés ou supprimés après que le service de file d'attente a répondu à votre demande.

## <a name="dequeue-messages"></a>Enlever des messages d’une file d’attente

Supprimez un message d'une file d'attente en deux étapes. Si votre code ne parvient pas à traiter un message, ce processus en deux étapes vous permet de récupérer le même message et de réessayer. Appelez `delete_message` une fois le message traité.

# <a name="python-v12"></a>[Python v12](#tab/python)

Lorsque vous appelez [receive_messages](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-), vous obtenez par défaut le message suivant de la file d'attente. Un message renvoyé par `receive_messages` devient invisible par les autres codes lisant les messages de cette file d'attente. Par défaut, ce message reste invisible pendant 30 secondes. Pour finaliser la suppression du message de la file d’attente, vous devez aussi appeler [delete_message](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#delete-message-message--pop-receipt-none----kwargs-).

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_DequeueMessages":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Lorsque vous appelez [get_messages](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#get-messages-queue-name--num-messages-none--visibility-timeout-none--timeout-none-), vous obtenez par défaut le message suivant de la file d'attente. Un message renvoyé par `get_messages` devient invisible par les autres codes lisant les messages de cette file d'attente. Par défaut, ce message reste invisible pendant 30 secondes. Pour finaliser la suppression du message de la file d’attente, vous devez aussi appeler [delete_message](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#delete-message-queue-name--message-id--pop-receipt--timeout-none-).

```python
messages = queue_service.get_messages(queue_name)

for message in messages:
    print("Deleting message: " + message.content)
    queue_service.delete_message(queue_name, message.id, message.pop_receipt)
```

---

Il existe deux façons de personnaliser la récupération des messages à partir d'une file d'attente. Premièrement, vous pouvez obtenir un lot de messages (jusqu'à 32). Deuxièmement, vous pouvez définir un délai d'expiration de l'invisibilité plus long ou plus court afin d'accorder à votre code plus ou moins de temps pour traiter complètement chaque message.

# <a name="python-v12"></a>[Python v12](#tab/python)

L'exemple de code suivant utilise la méthode [receive_messages](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-) pour récupérer des messages par lots. Il traite ensuite chacun des messages de chaque lot en utilisant une boucle `for` imbriquée. Il définit également le délai d'expiration de l'invisibilité sur cinq minutes pour chaque message.

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_DequeueByPage":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

L'exemple de code suivant utilise la méthode [get_messages](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#get-messages-queue-name--num-messages-none--visibility-timeout-none--timeout-none-) pour récupérer 16 messages en un appel. Ensuite, il traite chaque message à l'aide d'une boucle `for`. Il définit également le délai d'expiration de l'invisibilité sur cinq minutes pour chaque message.

```python
messages = queue_service.get_messages(queue_name, num_messages=16, visibility_timeout=5*60)

for message in messages:
    print("Deleting message: " + message.content)
    queue_service.delete_message(queue_name, message.id, message.pop_receipt)
```

---

## <a name="delete-a-queue"></a>Suppression d'une file d'attente

# <a name="python-v12"></a>[Python v12](#tab/python)

Pour supprimer une file d’attente et tous les messages qu’elle contient, appelez la méthode [delete_queue](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueclient#delete-queue---kwargs-).

:::code language="python" source="~/azure-storage-snippets/queues/howto/python/python-v12/python-howto-v12.py" id="Snippet_DeleteQueue":::

# <a name="python-v2"></a>[Python v2](#tab/python2)

Pour supprimer une file d’attente et tous les messages qu’elle contient, appelez la méthode [delete_queue](/azure/developer/python/sdk/storage/azure-storage-queue/azure.storage.queue.queueservice.queueservice?view=storage-py-v2#delete-queue-queue-name--fail-not-exist-false--timeout-none-).

```python
print("Deleting queue: " + queue_name)
queue_service.delete_queue(queue_name)
```

---

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez les bases du stockage de files d’attente, consultez les liens suivants pour en savoir plus.

- [Informations de référence sur l’API Python Files d’attente Azure](/python/api/azure-storage-queue)
- [Centre de développement Python](https://azure.microsoft.com/develop/python/)
- [API REST des services d’Azure Storage](/rest/api/storageservices/)

[Bibliothèque de client Stockage File d'attente Azure pour Python]: https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue
[SDK Azure pour Python]: https://github.com/azure/azure-sdk-for-python
[Azure Storage Team Blog]: https://techcommunity.microsoft.com/t5/azure-storage/bg-p/AzureStorageBlog
