---
title: Introduction au modèle de ressource Azure Service Fabric
description: Découvrez le modèle de ressource Service Fabric, une approche simplifiée de la définition d’applications Service Fabric mesh.
author: vturecek
ms.topic: conceptual
ms.date: 10/23/2018
ms.author: vturecek
ms.custom: mvc, devcenter
ms.openlocfilehash: a7f25d4156c3d2671e1a15a1253ed7ba22265af0
ms.sourcegitcommit: d95cab0514dd0956c13b9d64d98fdae2bc3569a0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91354868"
---
# <a name="introduction-to-service-fabric-resource-model"></a>Présentation du modèle de ressource Azure Service Fabric

Le modèle de ressource Service Fabric décrit une approche simple de la définition des ressources qui composent une application Service Fabric Mesh. Les ressources individuelles peuvent être déployées dans n’importe quel environnement Service Fabric.  Le modèle de ressource Service Fabric est également compatible avec le modèle Azure Resource Manager. Les types de ressources actuellement pris en charge dans ce modèle sont les suivants :

- Applications et services
- Réseaux
- Passerelles
- Secrets et secrets/valeurs
- Volumes

Chaque ressource est décrite de manière déclarative dans un fichier de ressources, qui est un simple document YAML ou JSON décrivant l’application mesh, et approvisionnée par la plateforme Service Fabric.

## <a name="applications-and-services"></a>Applications et services

Une ressource d’application est l’unité de déploiement, le contrôle de version et la durée de vie d’une application mesh. Elle est composée d’une ou plusieurs ressources Service qui représentent un microservice. Chaque ressource Service, à son tour, se compose d’un ou plusieurs packages de code qui décrivent tout ce qui est nécessaire pour exécuter l’image conteneur associée au package de code.

![Applications et services][Image1]

Une ressource Service déclare les éléments suivants :

- Nom, version et registre du conteneur
- Ressources de processeur et de mémoire requises pour chaque conteneur
- Points de terminaison réseau
- Références à d’autres ressources telles que les réseaux, les volumes et les secrets 

Tous les packages de code définis comme faisant partie d’une ressource de service sont déployés et activés ensemble en tant que groupe. La ressource de service décrit également le nombre d’instances du service à exécuter, et référence d’autres ressources (par exemple, des ressources réseau) dont elle dépend.

Si une application mesh est composée de plusieurs services, il n’est pas garanti que ceux-ci s’exécutent ensemble sur le même nœud. De même, pendant une mise à niveau de l’application, l’échec de mise à niveau d’un seul service entraîne la restauration de la version précédente de tous les services.

Comme mentionné précédemment, le cycle de vie de chaque instance d’application peut être géré indépendamment. Par exemple, une instance d’application peut être mise à niveau indépendamment des autres instances d’application. Généralement, vous maintenez un nombre relativement faible de services dans une application car, plus vous placez de services dans une application, plus il devient difficile de gérer chacun d’eux de façon indépendante.

## <a name="networks"></a>Réseaux

Une ressource réseau est une ressource déployable individuellement, indépendante de toute ressource d’application ou de service qui pourrait y faire référence en tant que dépendance. Elles permet de créer un réseau pour vos applications. Plusieurs services de différentes applications peuvent faire partie d’un même réseau.  Pour plus d’informations, consultez [Réseau dans les applications Service Fabric Mesh](service-fabric-mesh-networks-and-gateways.md).

> [!NOTE]
> La préversion actuelle ne prend en charge que le mappage un-à-un entre les applications et les réseaux.

![Réseau et passerelle][Image2]

## <a name="gateways"></a>Passerelles
Une ressource Passerelle connecte deux réseaux et achemine le trafic.  Une passerelle permet à vos services de communiquer avec des clients externes et fournit une entrée dans votre ou vos services.  Vous pouvez aussi l’utiliser pour connecter votre application Mesh à votre propre réseau virtuel existant. Pour plus d’informations, consultez [Réseau dans les applications Service Fabric Mesh](service-fabric-mesh-networks-and-gateways.md).

![Réseau et passerelle][Image2]

## <a name="secrets"></a>Secrets

Les ressources Secrets peuvent être déployées, indépendantes de toute ressource d’application ou de service qui pourrait y faire référence en tant que dépendance. Vous pouvez les utiliser pour fournir des secrets à vos applications en toute sécurité. Plusieurs services de différentes applications peuvent référencer des valeurs d’un même secret.

## <a name="volumes"></a>Volumes

Les conteneurs mettent souvent des disques temporaires à disposition. Toutefois, les disques temporaires sont éphémères. Quand un conteneur plante, vous obtenez un nouveau disque temporaire et perdez vos informations. Il est également difficile de partager des informations sur les disques temporaires avec d’autres conteneurs. Les volumes sont des répertoires montés à l’intérieur de vos instances de conteneur, que vous pouvez utiliser pour conserver l’état. Les volumes vous offrent un stockage de fichiers universel et vous permettent de lire/écrire des fichiers à l’aide d’API de fichier d’E/S de disque normal. La ressource Volume est un moyen déclaratif de décrire la façon dont un répertoire est monté et le stockage de sauvegarde qui y est associé (Azure Files Volume ou Service Fabric Reliable Volume).  Pour plus d’informations, consultez [Stockage d’état](service-fabric-mesh-storing-state.md#volumes).

![Le diagramme montre un flux qui passe d’un service à un volume de disque, et qui passe ensuite à Service Fabric Reliable Volume (puis à un disque local répliqué) et à Azure Files Volume (puis au stockage réseau).][Image3]

## <a name="programming-models"></a>Modèles de programmation
Une ressource de service ne requiert qu’une image conteneur pour s’exécuter, qui est référencée dans les packages de code associés à la ressource. Vous pouvez exécuter n’importe quel code, écrit dans n’importe quel langage, à l’aide de n’importe quelle infrastructure au sein du conteneur sans avoir besoin de connaître ou d’utiliser des API spécifiques de Service Fabric mesh. 

Votre code d’application reste portable, même en dehors de Service Fabric mesh, et vos déploiements d’applications restent cohérents indépendamment du langage ou de l’infrastructure utilisés pour implémenter vos services. Que votre application soit ASP.NET Core, Go, ou simplement un ensemble de processus et de scripts, le modèle de déploiement de ressource Service Fabric mesh reste le même. 

## <a name="packaging-and-deployment"></a>Empaquetage et déploiement

Suivant le modèle de ressource, les applications Service Fabric Mesh sont empaquetées en tant que conteneurs Docker.  Service Fabric Mesh est un environnement multilocataire partagé et les conteneurs vous offrent un niveau élevé d’isolation.  Ces applications sont décrites au format JSON ou YAML (qui est ensuite converti en JSON). Lorsque vous déployez une application Mesh sur Azure Service Fabric Mesh, le code JSON utilisé pour décrire l’application est un modèle Azure Resource Manager. Les ressources sont mappées aux ressources Azure.  Lorsque vous déployez une application Mesh sur un cluster Service Fabric (autonome ou hébergé par Azure), le code JSON utilisé pour décrire l’application est un format similaire à un modèle Azure Resource Manager.  Une fois déployées, les applications Mesh peuvent être gérées via des interfaces HTTP ou Azure CLI. 


## <a name="next-steps"></a>Étapes suivantes 
Pour en savoir plus sur Service Fabric mesh, voir la
- [Vue d’ensemble de Service Fabric mesh](service-fabric-mesh-overview.md)

[Image1]: media/service-fabric-mesh-service-fabric-resources/AppsAndServices.png
[Image2]: media/service-fabric-mesh-service-fabric-resources/NetworkAndGateway.png
[Image3]: media/service-fabric-mesh-service-fabric-resources/volumes.png
