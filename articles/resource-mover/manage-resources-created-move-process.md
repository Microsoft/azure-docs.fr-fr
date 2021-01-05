---
title: Gérer les ressources créées lors du déplacement d’une machine virtuelle dans Azure Resource Mover
description: Apprenez à gérer les ressources créées lors du déplacement d’une machine virtuelle dans Azure Resource Mover
manager: evansma
author: rayne-wiselman
ms.service: resource-move
ms.topic: how-to
ms.date: 09/10/2020
ms.author: raynew
ms.openlocfilehash: 6d6f56fb47bfdaadc6704e2a13ebcf0e893b8b1c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90602351"
---
# <a name="manage-resources-created-for-the-vm-move"></a>Gérer les ressources créées pour le déplacement d’une machine virtuelle

Cet article explique comment gérer les ressources créées explicitement par [Azure Resource Mover](overview.md) pour faciliter le processus de déplacement d’une machine virtuelle. 

Après le déplacement de machines virtuelles entre régions, un certain nombre de ressources créées par le moteur de ressources doivent être nettoyées manuellement.

## <a name="delete-resources-created-for-vm-move"></a>Supprimer les ressources créées pour le déplacement de machines virtuelles

Supprimez manuellement la collection de déplacement et les ressources Site Recovery créées pour le déplacement des machines virtuelles.

1. Examinez les ressources dans le groupe de ressources ```ResourceMoverRG-<sourceregion>-<target-region>```.
2. Vérifiez que la machine virtuelle et toutes les autres ressources sources de la collection de déplacement ont été déplacées ou supprimées. Cette vérification permet de s’assurer qu’aucune ressource en attente ne les utilise.
2. Supprimez ces ressources.

    - Le nom de la collection de déplacement est ```movecollection-<sourceregion>-<target-region>```.
    - Le nom du compte de stockage de cache est ```resmovecache<guid>```.
    - Le nom du coffre est ```ResourceMove-<sourceregion>-<target-region>-GUID```.

## <a name="next-steps"></a>Étapes suivantes

Essayez de [déplacer une machine virtuelle](tutorial-move-region-virtual-machines.md) vers une autre région avec Resource Mover.