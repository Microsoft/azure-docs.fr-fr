---
title: Choisir un outil pour déplacer des ressources Azure entre les régions
description: Passer en revue les options et outils permettant de déplacer des ressources Azure entre les régions
author: rayne-wiselman
manager: evansma
ms.service: resource-move
ms.topic: overview
ms.date: 09/09/2020
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: 1b233028b52175842c73660ff116ac592d8296e0
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/16/2020
ms.locfileid: "90603354"
---
# <a name="choose-a-tool-for-moving-azure-resources"></a>Choisir un outil pour déplacer des ressources Azure

Vous pouvez déplacer des ressources dans Azure comme suit :

- **Déplacer des ressources entre les régions** : Déplacez une ressource depuis le hub Resource Mover ou depuis un groupe de ressources. 
- **Déplacer des ressources entre les groupes de ressources/abonnements** : Déplacez les ressources depuis un groupe de ressources. 
- **Déplacer des ressources entre les clouds Azure** : Utilisez le service Azure Site Recovery pour déplacer des ressources entre des clouds publics et gouvernementaux.
- **Déplacer des ressources entre des zones de disponibilité de la même région** : Utilisez le service Azure Site Recovery pour déplacer des ressources entre des zones de disponibilité de la même région Azure.


## <a name="compare-move-tools"></a>Comparer les outils de déplacement

**Outil** | **Quand utiliser** | **En savoir plus**
--- | --- | ---
**Déplacer au sein du groupe de ressources** | Déplacer des ressources vers un autre groupe de ressources/abonnement ou entre différentes régions.<br/><br/> Si vous changez de région, dans le groupe de ressources, sélectionnez les ressources à déplacer, puis accédez au hub Resource Mover pour vérifier les dépendances avant de déplacer les ressources vers la région cible. | [Déplacer des ressources vers un autre groupe de ressources/abonnement.](../azure-resource-manager/management/move-resource-group-and-subscription.md)<br/><br/> [Déplacer des ressources vers une autre région à partir d’un groupe de ressources](move-region-within-resource-group.md).
**Déplacer à partir du hub Resource Mover** | Déplacer des ressources entre les régions. <br/><br/> Vous pouvez déplacer vers une région cible, une zone de disponibilité spécifique ou un groupe à haute disponibilité, au sein de la région cible. | [Déplacer des ressources entre les régions dans le hub Resource Mover]().
**Déplacer des machines virtuelles avec Site Recovery** | Utiliser cet outil pour déplacer des machines virtuelles Azure entre des clouds publics et gouvernementaux.<br/><br/> Utiliser cet outil pour déplacer des machines virtuelles entre des zones de disponibilité de la même région. |[Déplacer des ressources entre des clouds publics/gouvernementaux](../site-recovery/region-move-cross-geos.md), [Déplacer des ressources vers des zones de disponibilité de la même région](../site-recovery/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery.md).

## <a name="next-steps"></a>Étapes suivantes

[Découvrez-en plus](about-move-process.md) sur les composants de Resource Mover et le processus de déplacement.
