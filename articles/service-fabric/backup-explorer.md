---
title: Lire et mettre à jour une sauvegarde de collection fiable localement
description: Utilisez l’Explorateur de sauvegarde dans Azure Service Fabric pour lire et mettre à jour une sauvegarde de collection fiable locale.
author: athinanthny
ms.topic: conceptual
ms.date: 07/01/2020
ms.author: atsenthi
ms.openlocfilehash: 1da70c37c8a6ed93e7abe1b5d329e808c592e43a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86034719"
---
# <a name="read-and-update-a-reliable-collections-backup-by-using-backup-explorer"></a>Lire et mettre à jour une sauvegarde de collection fiable en utilisant Explorateur de sauvegarde

L’Explorateur de sauvegarde Azure Service Fabric aide à la correction des données si les données sont endommagées dans les collections fiables de Service Fabric. L’état actuel des données peut être endommagé par un bogue introduit dans une application ou par des entrées incorrectes dans un cluster actif.

À l’aide de l’Explorateur de sauvegarde, vous pouvez effectuer les tâches suivantes :
-   Interrogez les métadonnées de la collection.
-   Affichez l’état actuel et ses entrées dans la collection de la sauvegarde chargée.
-   Répertoriez les transactions effectuées depuis le dernier point de contrôle.
-   Mettez à jour la collection en ajoutant, en mettant à jour ou en supprimant des entrées dans la collection.
-   Effectuez une nouvelle sauvegarde en utilisant l’état mis à jour.

> [!NOTE]
> L’Explorateur de sauvegarde de Service Fabric prend actuellement en charge l’affichage et la modification des collections fiables dans la sauvegarde.
>

## <a name="access-the-backup"></a>Accéder à la sauvegarde

L’Explorateur de sauvegarde de Service Fabric peut être utilisé de l’une des manières suivantes pour afficher ou mettre à jour les collections fiables dans la sauvegarde :
-   **Binaire** : Utilisez un package NuGet pour afficher et modifier les collections fiables.
-   **HTTP/REST** : Utilisez un serveur REST basé sur le protocole HTTP pour afficher et modifier les collections fiables.
-   **bkpctl** : Utilisez l’interface de ligne de commande (CLI) de l’Explorateur de sauvegarde de Service Fabric pour afficher et modifier une sauvegarde de collections fiables.

L’Explorateur de sauvegarde dispose d’une bibliothèque C# pour les utilisateurs expérimentés. Vous pouvez utiliser la bibliothèque dans l’application directement pour travailler avec des collections fiables similaires à la façon dont les clients travaillent avec le gestionnaire d’État dans leurs services existants. Pour les utilisateurs de base et dans un cas d’usage de base, l’Explorateur dispose également d’un serveur REST autonome qui expose les API pour inspecter les sauvegardes. L’outil bkpctl de l’interface CLI fonctionne sur les API REST et est basé sur Python. Vous pouvez utiliser l’interface CLI pour lire et mettre à jour les sauvegardes, et pour effectuer de nouvelles sauvegardes à l’aide des lignes de commande.

Pour plus d’informations, consultez le référentiel GitHub [Explorateur de sauvegarde de Service Fabric ](https://github.com/microsoft/service-fabric-backup-explorer). Le référentiel contient des informations sur la source et la version, ainsi que des instructions d’installation.

Vous pouvez également créer le référentiel localement et travailler sur des sauvegardes.
 
NuGet pour l’Explorateur de sauvegarde (Microsoft.ServiceFabric.ReliableCollectionBackup.Parser) sera disponible sur [nuget.org](https://www.nuget.org/). 

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [Collections fiables dans les services avec état d’Azure Service Fabric](service-fabric-reliable-services-reliable-collections.md).
* Consultez les [bonnes pratiques Service Fabric](service-fabric-best-practices-overview.md).
