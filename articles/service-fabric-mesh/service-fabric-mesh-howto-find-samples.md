---
title: Rechercher des exemples Azure Service Fabric Mesh
description: Voici l’index des exemples d’applications Service Fabric Mesh disponibles. Le code source de ces exemples montre comment réaliser un scénario particulier à l’aide du modèle de ressource Service Fabric.
ms.date: 12/03/2018
ms.topic: conceptual
ms.openlocfilehash: 64d9e4e112f26f83eedd02b034c8191c6f677618
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86261272"
---
# <a name="find-service-fabric-mesh-samples"></a>Rechercher des exemples Service Fabric Mesh

Ce tableau décrit les exemples d’applications Service Fabric Mesh disponibles. Le code source de ces exemples montre comment réaliser un scénario particulier à l’aide du modèle de ressource Service Fabric.

Pour plus d’informations sur le déploiement des modèles directement dans Azure, consultez la [page GitHub concernant les exemples de modèles](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/README.md).

|Exemple de modèle|Description du scénario|Code source|Outils de développement|
|------------|--------------------|----------|----------------------|
| [Application Hello World](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/templates/helloworld) | Page web statique hébergée dans un conteneur. Elle utilise nginx pour Linux et IIS pour Windows. | [Code source](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/helloworld) | Aucune exigence |
| [Application de compteur pour les volumes Azure Files](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/templates/counter/readme.md) | État du stockage après le montage d’un volume Azure Files dans le conteneur. <br><br> **Remarque :** Ce modèle nécessite un partage de fichiers Azure Files pour être préprovisionné [Instructions](../storage/files/storage-how-to-create-file-share.md) | [Code source](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/counter) | Visual Studio Mesh Tooling |
| [TodoListApp](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/templates/todolist) | Créez une application avec un service front-end et back-end qui utilise la résolution DNS. Utilisée pour un tutoriel [ici](./service-fabric-mesh-tutorial-create-dotnetcore.md) | [Code source](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/todolistapp) | Visual Studio Mesh Tooling |
| [Application Visual Objects](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/templates/visualobjects) | Effectuez la mise à l’échelle et la mise à niveau de microservices au sein d’une application. | [Code source](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/visualobjects) |  Visual Studio Mesh Tooling |
| [Application Voting](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/templates/voting) | Créez une application avec un service front-end et back-end qui utilise la résolution DNS. | [Code source](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/votingapp) | Visual Studio Mesh Tooling pour la version Windows ; VS Code/l’interface CLI .NET peut être utilisé pour la version Linux. |
| [Application de compteur pour des volumes Service Fabric fiables](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/templates/counter/readme.sfreliablevolume.md)| État du stockage après le montage d’un volume Service Fabric Reliable Disk dans le conteneur.| [Code source](https://github.com/Azure-Samples/service-fabric-mesh/tree/master/src/counter) | Visual Studio Mesh Tooling |
