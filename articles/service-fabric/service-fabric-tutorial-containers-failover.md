---
title: Basculer et mettre à l’échelle une application de conteneurs
description: Dans ce didacticiel, vous apprenez la gestion du basculement dans une application de conteneurs Azure Service Fabric.  Sachez également mettre à l’échelle les conteneurs et les services qui s’exécutent dans un cluster.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: f51a195f9ad43c7facb7dc413a4da83fc6b202b7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91531392"
---
# <a name="tutorial-demonstrate-fail-over-and-scaling-of-container-services-with-service-fabric"></a>Didacticiel : Présenter le basculement et la mise à l’échelle de services de conteneur avec Service Fabric

Ce tutoriel est le troisième de la série. Dans ce didacticiel, vous découvrez la gestion du basculement au sein d’applications de conteneur dans Service Fabric. Vous apprenez également à mettre à l’échelle des conteneurs. Dans ce tutoriel, vous allez :

> [!div class="checklist"]
> * En savoir plus sur le basculement de conteneur dans un cluster Service Fabric
> * Mettre à l’échelle les conteneurs web frontaux dans une application

## <a name="prerequisites"></a>Conditions préalables requises

À partir de la [Partie 2](service-fabric-tutorial-package-containers.md), l’application s’exécute dans un cluster Service Fabric actif.

## <a name="fail-over-a-container-in-a-cluster"></a>Basculer un conteneur dans un cluster

Service Fabric s’assure que vos instances de conteneur se déplacent automatiquement vers d’autres nœuds du cluster en cas d’échec. Vous pouvez aussi manuellement drainer un nœud de conteneurs, puis déplacer normalement ces conteneurs vers d’autres nœuds du cluster. Il existe plusieurs façon de mettre à l’échelle vos services, dans cet exemple, nous utilisons Service Fabric Explorer.

Pour basculer le conteneur frontal, effectuez les étapes suivantes :

1. Ouvrez Service Fabric Explorer dans votre cluster. Par exemple, `http://lin4hjim3l4.westus.cloudapp.azure.com:19080`.
2. Cliquez sur le nœud **fabric:/TestContainer/azurevotefront** dans l’arborescence, puis développez le nœud de la partition (représenté par un GUID). Remarquez le nom du nœud dans l’arborescence qui vous renseigne sur les nœuds en cours d’exécution dans le conteneur, par exemple `_nodetype_1`
3. Développez le nœud **Nœuds** dans l’arborescence. Cliquez sur les points de suspension (trois points) en regard du nœud qui exécute le conteneur.
4. Choisissez **Redémarrer** pour redémarrer ce nœud et confirmer l’action de redémarrage. Le redémarrage entraîne le basculement du conteneur vers un autre nœud du cluster.

![RedémarrageDuNœud][noderestart]

Remarquez comment le nom du nœud (indiquant l’emplacement où les conteneurs frontaux s’exécutent) est remplacé par un autre nœud du cluster. Au bout de quelques instants, vous devez être en mesure d’accéder de nouveau à l’application et la voir s’exécuter sur un autre nœud.

## <a name="scale-containers-and-services-in-a-cluster"></a>Mettre à l’échelle les conteneurs et services dans un cluster

Les conteneurs Service Fabric peuvent être mis à l’échelle dans un cluster pour s’adapter à la charge sur les services. Pour mettre à l’échelle un conteneur, vous modifiez le nombre d’instances s’exécutant dans le cluster.

Pour mettre à l’échelle le service web frontal, effectuez les étapes suivantes :

1. Ouvrez Service Fabric Explorer dans votre cluster. Par exemple, `http://lin4hjim3l4.westus.cloudapp.azure.com:19080`.
2. Cliquez sur les points de suspension (trois points) en regard du nœud **fabric:/TestContainer/azurevotefront** dans l’arborescence, puis choisissez **Mettre à l’échelle le service**.

![MettreÀL’échelleSfx][sfxscale]

Vous pouvez maintenant choisir de mettre à l’échelle le nombre d’instances du service web frontal.

1. Définissez le nombre sur **2**, puis cliquez sur **Scale Service** (Mettre à l’échelle le service).
1. Cliquez sur le nœud **fabric:/TestContainer/azurevotefront** dans l’arborescence, puis développez le nœud de la partition (représenté par un GUID).

![MiseÀL’échelleDeSfxEffectuée][sfxscaledone]

Vous constatez désormais que le service possède deux instances. Dans l’arborescence, vous voyez quels sont les nœuds sur lesquels les instances s’exécutent.

Par cette tâche de gestion simple, nous avons doublé les ressources disponibles pour permettre à notre service frontal de traiter la charge utilisateur. Il est important de comprendre que vous n’avez pas besoin de plusieurs instances d’un service pour que celui-ci s’exécute de manière fiable. En cas de défaillance d’un service, Service Fabric veille à ce qu'une nouvelle instance du service s’exécute dans le cluster.

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous a présenté le basculement d’un conteneur ainsi que la mise à l’échelle d’une application. Les étapes suivantes ont été effectuées :

> [!div class="checklist"]
> * En savoir plus sur le basculement de conteneur dans un cluster Service Fabric
> * Mettre à l’échelle les conteneurs web frontaux dans une application

Dans cette série de didacticiels, vous avez appris à :
> [!div class="checklist"]
> * Créer des images de conteneur
> * Envoyer des images conteneur à Azure Container Registry
> * Créer des packages de conteneurs pour Service Fabric à l’aide de Yeoman
> * Créer et exécuter une application Service Fabric avec des conteneurs
> * En savoir plus sur la gestion du basculement et de la mise à l’échelle dans Service Fabric

[noderestart]: ./media/service-fabric-tutorial-containers-failover/containersfailovertutorialnoderestart.png
[sfxscale]: ./media/service-fabric-tutorial-containers-failover/containersfailovertutorialscale.png
[sfxscaledone]: ./media/service-fabric-tutorial-containers-failover/containersfailovertutorialscaledone.png