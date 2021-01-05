---
title: Stocker et utiliser des secrets d’application Azure Service Fabric Mesh
description: Service Fabric Mesh prend en charge les secrets comme des ressources Azure. Voici comment stocker et gérer les secrets avec des applications Service Fabric Mesh.
author: erikadoyle
ms.author: edoyle
ms.date: 10/25/2018
ms.topic: conceptual
ms.openlocfilehash: da4eaa34840f14714616b85e24fd62cf65602b84
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76277638"
---
# <a name="service-fabric-mesh-application-secrets"></a>Secrets d’application Service Fabric Mesh
Service Fabric Mesh prend en charge les secrets comme des ressources Azure. Un secret Service Fabric Mesh peut correspondre à n’importe quel type de texte sensible, comme des chaînes de connexion de stockage, des mots de passe ou d’autres valeurs devant être stockées et transmises de manière sécurisée.

![Présentation des secrets Mesh][sf-mesh-secrets-overview]

## <a name="mesh-secrets-resources"></a>Ressources Secrets Mesh
Un secret d’application Mesh comprend :
* Une ressource **Secrets**, qui est un conteneur où sont stockés les secrets texte. Les secrets contenus dans la ressource **Secrets** sont stockés et transmis de manière sécurisée.
* Une ou plusieurs ressources **Secrets/Valeurs** qui sont stockées dans le conteneur de ressources **Secrets**. Chaque ressource **Secrets/Valeurs** se distingue par son numéro de version.

## <a name="next-steps"></a>Étapes suivantes 
Pour plus d’informations sur les secrets Service Fabric Mesh, consultez :
- [Gérer les secrets d’application Service Fabric Mesh](service-fabric-mesh-howto-manage-secrets.md)
- [Présentation du modèle de ressource Azure Service Fabric](service-fabric-mesh-service-fabric-resources.md)

<!-- pics -->
[sf-mesh-secrets-overview]: ./media/service-fabric-mesh-secrets-overview/MeshAppSecretsOverview.png
