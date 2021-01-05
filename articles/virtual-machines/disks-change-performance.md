---
title: Niveaux de performance des disques managés Azure
description: En savoir plus sur les niveaux de performance des disques managés.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 11/19/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 28980756ac9e41c9477d687ea9df608b512759e3
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94986779"
---
# <a name="performance-tiers-for-managed-disks"></a>Niveaux de performance des disques managés

Les performances d’un disque managé Azure sont définies lors de sa création, par son niveau de performance. Le niveau de performance détermine ses IOPS et son débit. Lorsque vous définissez la taille approvisionnée de votre disque, un niveau de performance est sélectionné automatiquement. Il peut être modifié au moment du déploiement ou ultérieurement, sans changer de taille de disque.

La modification du niveau de performance vous permet de préparer le disque et de répondre à une demande plus importante sans utiliser la capacité de bursting de votre disque. Il peut être plus rentable de changer de niveau de performance plutôt que de recourir au bursting, selon la durée pendant laquelle la performance supplémentaire est nécessaire. Cette solution est idéale pour les événements qui nécessitent temporairement un niveau de performances systématiquement plus élevé, comme les achats pendant les fêtes, les tests de performance ou l’exécution d’un environnement de formation. Pour gérer ces événements, vous pouvez utiliser un niveau de performance plus élevé aussi longtemps que vous en avez besoin. Vous pouvez ensuite revenir au niveau d’origine lorsque vous n’avez plus besoin de performances supplémentaires.

## <a name="how-it-works"></a>Fonctionnement

Quand vous déployez ou provisionnez un disque pour la première fois, le niveau de performance de base de référence pour ce disque est défini en fonction de la taille de disque provisionnée. Vous pouvez utiliser un niveau de performances supérieur à la ligne de base d’origine pour répondre à une demande plus élevée. Lorsque vous n’avez plus besoin de ce niveau de performance, vous pouvez revenir au niveau de performance initial.

Votre facturation change à mesure que votre niveau de performance change. Par exemple, si vous provisionnez un disque P10 (128 Gio), votre niveau de performance de base de référence est défini comme P10 (500 IOPS et 100 Mo/s). Vous êtes facturé au tarif P10. Vous pouvez augmenter le niveau pour qu’il corresponde aux performances P50 (7 500 E/S par seconde et 250 Mbits/s) sans que la taille du disque soit augmentée. Au moment de la mise à niveau, vous êtes facturé au tarif P50. Lorsque vous n’avez plus besoin de performances supérieures, vous pouvez revenir au niveau P10. Le disque sera à nouveau facturé au tarif P10.

| Taille du disque | Niveau de performance de base de référence | Mise à niveau possible vers |
|----------------|-----|-------------------------------------|
| 4 Gio | P1 | P2, P3, P4, P6, P10, P15, P20, P30, P40, P50 |
| 8 Gio | P2 | P3, P4, P6, P10, P15, P20, P30, P40, P50 |
| 16 Gio | P3 | P4, P6, P10, P15, P20, P30, P40, P50 | 
| 32 Gio | P4 | P6, P10, P15, P20, P30, P40, P50 |
| 64 Gio | P6 | P10, P15, P20, P30, P40, P50 |
| 128 Go | P10 | P15, P20, P30, P40, P50 |
| 256 Gio | P15 | P20, P30, P40, P50 |
| 512 Go | P20 | P30, P40, P50 |
| 1 Tio | P30 | P40, P50 |
| 2 Tio | P40 | P50 |
| 4 Tio | P50 | Aucun |
| 8 Tio | P60 |  P70, P80 |
| 16 Tio | P70 | P80 |
| 32 Tio | P80 | Aucun |

Pour plus d’informations sur la facturation, consultez [Tarification des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/).

## <a name="restrictions"></a>Restrictions

[!INCLUDE [virtual-machines-disks-performance-tiers-restrictions](../../includes/virtual-machines-disks-performance-tiers-restrictions.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment modifier votre niveau de performance, consultez les articles pour le [portail](disks-performance-tiers-portal.md) ou pour [PowerShell/CLI](disks-performance-tiers.md).

