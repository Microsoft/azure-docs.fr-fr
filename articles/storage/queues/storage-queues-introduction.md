---
title: Présentation de Stockage File d’attente Azure – Stockage Azure
description: Consultez une introduction au Stockage File d’attente Azure, un service permettant de stocker un grand nombre de messages. Un service Stockage File d’attente contient un format d’URL, un compte de stockage, une file d’attente et un message.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 03/18/2020
ms.topic: overview
ms.service: storage
ms.subservice: queues
ms.openlocfilehash: 8c5c97fbb72934dd99ec784ccf8e08eec059c31b
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590577"
---
# <a name="what-is-azure-queue-storage"></a>Qu’est-ce que Stockage File d’attente Azure ?

Stockage File d’attente Azure est un service permettant de stocker un grand nombre de messages. Vous accédez aux messages depuis n’importe où dans le monde par le biais d’appels authentifiés à l’aide du protocole HTTP ou HTTPS. La taille maximale d’un message de file d’attente est de 64 Ko. Une file d’attente peut contenir des millions de messages, dans la limite de la capacité totale d’un compte de stockage. Les files d’attente sont couramment utilisées pour créer un backlog de travail à traiter de façon asynchrone.

## <a name="queue-storage-concepts"></a>Concepts de Stockage File d’attente

Le Stockage File d’attente est composé des éléments suivants :

![Diagramme montrant la relation qui existe entre un compte de stockage, les files d’attente et les messages.](./media/storage-queues-introduction/queue1.png)

- **Format d’URL :** les files d’attente sont adressables à l’aide du format d’URL suivant :

  `https://<storage account>.queue.core.windows.net/<queue>`

  L'URL suivante désigne une file d'attente du schéma :

  `https://myaccount.queue.core.windows.net/images-to-download`

- **Compte de stockage :** Tous les accès à Azure Storage passent par un compte de stockage. Pour plus d’informations sur la capacité du compte de stockage, consultez [Cibles de scalabilité et de performances pour les comptes de stockage standard](../common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json).

- **File d’attente :** une file d’attente contient un ensemble de messages. Le nom de la file d’attente **doit** être en minuscules. Pour plus d’informations sur le nommage des files d’attente, consultez [Affectation de noms pour les files d’attente et les métadonnées](/rest/api/storageservices/naming-queues-and-metadata).

- **Message :** message dans n’importe quel format d’une taille maximale de 64 Ko. Avant la version du 29 juillet 2017, la durée de vie maximale autorisée est de sept jours. Pour les versions du 29 juillet 2017 ou ultérieures, la durée de vie maximale peut être n’importe quel nombre positif, ou -1 indiquant que le message n’expire pas. Si ce paramètre est omis, la valeur par défaut de la durée de vie est de sept jours.

## <a name="next-steps"></a>Étapes suivantes

- [Créez un compte de stockage](../common/storage-account-create.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
- [Bien démarrer avec le Stockage File d’attente à l’aide de .NET](storage-dotnet-how-to-use-queues.md)
- [Bien démarrer avec le Stockage File d’attente à l’aide de Java](storage-java-how-to-use-queue-storage.md)
- [Bien démarrer avec le Stockage File d’attente à l’aide de Python](storage-python-how-to-use-queue-storage.md)
- [Bien démarrer avec le Stockage File d’attente à l’aide de Node.js](storage-nodejs-how-to-use-queues.md)
