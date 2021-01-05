---
title: Arrêter ou démarrer un groupe de conteneurs manuellement
description: Découvrez comment arrêter ou démarrer manuellement des conteneurs dans Azure Container Instances.
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: f697f18459959cf40e71d3ca90cd8b42aaa74239
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88799075"
---
# <a name="manually-stop-or-start-containers-in-azure-container-instances"></a>Arrêter ou démarrer manuellement des conteneurs dans Azure Container Instances

Le paramètre [Stratégie de redémarrage](container-instances-restart-policy.md) d’un groupe de conteneurs détermine la manière dont Container Instances démarre ou s'arrête par défaut. Vous pouvez remplacer le paramètre par défaut en arrêtant ou en démarrant manuellement un groupe de conteneurs.

[!INCLUDE [container-instances-restart-ip](../../includes/container-instances-restart-ip.md)]

## <a name="stop"></a>Arrêter

Arrêtez manuellement un groupe de conteneurs en cours d’exécution, par exemple, en utilisant la commande [az container stop][az-container-stop] ou le portail Azure. Pour certaines charges de travail de conteneur, vous pouvez vouloir arrêter un groupe de conteneurs à exécution longue après une période définie afin de réduire les coûts. 

*À l'état Arrêté, un groupe de conteneurs met fin et recycle tous les conteneurs du groupe. Il ne conserve pas l’état des conteneurs.*

Lorsque les conteneurs sont recyclés, les [ressources](container-instances-container-groups.md#resource-allocation) sont désallouées et la facturation s’arrête pour le groupe de conteneurs.

L'arrêt n’a aucun effet si le groupe de conteneurs est déjà terminé (état Réussite ou Échec). Par exemple, un groupe de conteneurs avec des tâches de conteneur à exécution unique correctement exécutées se termine dans un état Réussite. Toute tentative visant à arrêter un groupe dans cet état ne change pas ce dernier. 

## <a name="start"></a>Démarrer

Lorsqu'un groupe de conteneurs est arrêté - soit parce que les conteneurs s’arrêtent d’eux-mêmes, soit parce que vous avez arrêté manuellement le groupe - vous pouvez démarrer les conteneurs. Par exemple, utilisez la commande [az container start][az-container-start] ou le portail Azure pour démarrer manuellement les conteneurs du groupe. Si l’image d’un conteneur est mise à jour, une nouvelle image est extraite. 

Le démarrage d’un groupe de conteneurs commence un nouveau déploiement avec la même configuration de conteneur. Cette action peut vous aider à réutiliser rapidement une configuration de groupe de conteneurs connue qui fonctionne comme prévu. Vous n’avez pas besoin de créer un nouveau groupe de conteneurs pour exécuter la même charge de travail.

Cette action démarre tous les conteneurs d'un groupe de conteneurs. Vous ne pouvez pas démarrer un conteneur spécifique du groupe.

Après que vous avez démarré ou redémarré manuellement un groupe de conteneurs, le groupe de conteneurs fonctionne conformément à la stratégie de redémarrage configurée.
  
## <a name="restart"></a>Redémarrer

Vous pouvez redémarrer un groupe de conteneurs en cours d’exécution – par exemple, en utilisant la commande [az container restart][az-container-restart]. Cette action redémarre tous les conteneurs du groupe de conteneurs. Si l’image d’un conteneur est mise à jour, une nouvelle image est extraite. 

Le redémarrage d’un groupe de conteneurs est utile lorsque vous souhaitez résoudre un problème de déploiement. Par exemple, si une limitation temporaire des ressources empêche le bon fonctionnement de vos conteneurs, le redémarrage du groupe peut résoudre le problème.

Cette action redémarre tous les conteneurs d'un groupe de conteneurs. Vous ne pouvez pas redémarrer un conteneur spécifique du groupe.

Après avoir redémarré manuellement un groupe de conteneurs, le groupe de conteneurs fonctionne conformément à la stratégie de redémarrage configurée.

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur les [paramètres de stratégie de redémarrage](container-instances-restart-policy.md) dans Azure Container Instances.

En plus d'arrêter et de démarrer manuellement un groupe de conteneurs avec la configuration existante, vous pouvez [mettre à jour les paramètres](container-instances-update.md) d’un groupe de conteneurs en cours d’exécution.

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[az-container-restart]: /cli/azure/container?view=azure-cli-latest#az-container-restart
[az-container-start]: /cli/azure/container?view=azure-cli-latest#az-container-start
[az-container-stop]: /cli/azure/container?view=azure-cli-latest#az-container-stop
