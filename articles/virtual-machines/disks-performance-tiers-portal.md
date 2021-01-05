---
title: Modification du niveau de performance des disques managés Azure avec le Portail Azure
description: Découvrez comment modifier le niveau de performance de nouveaux disques managés et de disques existants avec le Portail Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 11/19/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: a868d5d236cf4c5a8d29f15490909c2f1a53546f
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "96016572"
---
# <a name="change-your-performance-tier-using-the-azure-portal"></a>Modification du niveau de performance avec le Portail Azure

[!INCLUDE [virtual-machines-disks-performance-tiers-intro](../../includes/virtual-machines-disks-performance-tiers-intro.md)]

## <a name="restrictions"></a>Restrictions

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](../../includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## <a name="getting-started"></a>Prise en main

### <a name="new-disks"></a>Nouveaux disques

La procédure suivante montre comment modifier le niveau de performance d’un disque lors de sa création initiale :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Accédez à la machine virtuelle pour laquelle vous souhaitez créer un disque.
1. Lorsque vous sélectionnez le nouveau disque, commencez par choisir la taille dont vous avez besoin.
1. Une fois que vous avez sélectionné une taille, sélectionnez un autre niveau de performance pour modifier ses performances.
1. Sélectionnez **OK** pour créer le disque.

:::image type="content" source="media/disks-performance-tiers-portal/new-disk-change-performance-tier.png" alt-text="Capture d’écran du panneau de création de disque, avec un disque et la liste déroulante du niveau de performance surlignés" lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::


## <a name="existing-disks"></a>Disques existants

La procédure suivante montre comment modifier le niveau de performance d’un disque existant :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Accédez à la machine virtuelle contenant le disque que vous souhaitez modifier.
1. Libérez la machine virtuelle ou détachez le disque.
1. Sélectionnez votre disque.
1. Sélectionnez **Taille + performances**.
1. Dans la liste déroulante **Niveau de performance**, sélectionnez un niveau différent de la ligne de base actuelle du disque.
1. Sélectionnez **Redimensionner**.

:::image type="content" source="media/disks-performance-tiers-portal/change-tier-existing-disk.png" alt-text="Capture d’écran du panneau Taille + performances, avec le niveau de performance surligné" lightbox="media/disks-performance-tiers-portal/performance-tier-settings.png":::

## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin de redimensionner un disque pour tirer parti de niveaux de performance supérieurs, consultez les articles suivants :

- [Étendre des disques durs virtuels sur une machine virtuelle Linux avec Azure CLI](linux/expand-disks.md)
- [Développer un disque managé attaché à une machine virtuelle Windows](windows/expand-os-disk.md)
