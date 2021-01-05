---
title: Fichier include
description: Fichier include
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 04/08/2020
ms.service: storage
ms.subservice: blobs
ms.topic: include
ms.reviewer: hux
ms.custom: include file
ms.openlocfilehash: a369eb7000fb8622a69f4205ffcc232ae9c9d242
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95545927"
---
Pour lire des données dans le stockage archive, vous devez d’abord attribuer un niveau chaud ou froid à l’objet blob. Ce processus est appelé « réalimentation » et peut prendre des heures. Nous recommandons des tailles d’objet blob importantes pour obtenir des performances de réalimentation optimales. La réalimentation simultanée de plusieurs petits objets blob peut demander du temps supplémentaire. Il existe actuellement deux priorités de réalimentation, haute et standard, qui peuvent être définies via la propriété facultative *x-ms-rehydrate-priority* sur une opération [Définir un niveau d’objet blob](/rest/api/storageservices/set-blob-tier) ou [Copier l’objet blob](/rest/api/storageservices/copy-blob).

* **Priorité standard** : La requête de réalimentation est traitée dans l’ordre dans lequel elle a été reçue et peut prendre jusqu’à 15 heures.
* **Haute priorité** : La demande de réalimentation, prioritaire par rapport aux demandes standard, peut se terminer en moins d’une heure pour les objets de taille inférieure à 10 Go. 

> [!NOTE]
> La priorité standard est l’option de réalimentation par défaut pour les archives. La haute priorité est une option plus rapide plus coûteuse que la réalimentation de priorité standard et est généralement réservée à une utilisation dans des situations de restauration de données urgentes.
>
> La haute priorité peut durer plus de 1 heure en fonction de la taille de l’objet blob et de la requête actuelle. Les requêtes haute priorité sont assurées d’être prioritaires sur les requêtes de priorité standard.

Une fois qu’une requête de réalimentation est lancée, elle ne peut pas être annulée. Pendant le processus de réalimentation, la propriété blob *x-ms-access-tier* continue à s’afficher en tant qu’archive jusqu’à ce que la réalimentation soit terminée sur un niveau en ligne. Pour confirmer l’état et la progression de la réalimentation, vous pouvez appeler [Get Blob Properties](/rest/api/storageservices/get-blob-properties) pour vérifier les propriétés blob *x-ms-archive-status* et *x-ms-rehydrate-priority*. L’état de l’archive peut afficher « réalimentation-vers-chaud » ou « réalimentation-vers-froid » selon le niveau de réalimentation choisi. La priorité de réalimentation indique la vitesse « haute » ou « standard ». Une fois l’opération terminée, les propriétés État de l’archive et Priorité de réalimentation sont supprimées, et la propriété blob du niveau d’accès est mise à jour pour refléter le niveau chaud ou froid sélectionné.