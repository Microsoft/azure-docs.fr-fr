---
title: Fichier include
description: Fichier include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 03/05/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: edec192009b9cc9b71114287e705c161183273dc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95992925"
---
Les instantanés incrémentiels sont des sauvegardes des disques managés exécutées à un moment donné et contenant uniquement les modifications apportées depuis la dernière capture instantanée. Lorsque vous restaurez un disque à partir d’un instantané incrémentiel, le système reconstruit le disque complet qui représente la sauvegarde du disque au moment où l’instantané incrémentiel a été capturé. Cette nouvelle fonctionnalité de capture instantanée des disques managés améliore leur rentabilité, car vous n’êtes pas obligé de stocker l'intégralité du disque avec chaque instantané. Comme pour les instantanés complets, les instantanés incrémentiels peuvent être utilisés pour créer un disque managé complet ou un instantané complet.

Il existe un certain nombre de différences entre un instantané incrémentiel et un instantané complet. Les instantanés incrémentiels utilisent toujours le stockage HDD Standard, quel que soit le type de stockage du disque, alors que les instantanés complets peuvent utiliser des disques SSD premium. Si vous utilisez des instantanés complets sur le Stockage Premium pour effectuer un scale-up de déploiements de machines virtuelles, nous vous recommandons d’utiliser des images personnalisées sur le stockage standard dans [Shared Image Gallery](../articles/virtual-machines/linux/shared-image-galleries.md). Cela vous aidera à atteindre une plus grande échelle à moindre coût. En outre, les instantanés incrémentiels offrent potentiellement une meilleure fiabilité avec un [stockage redondant interzone](../articles/storage/common/storage-redundancy.md) (ZRS). Si le stockage ZRS est disponible dans la région sélectionnée, l'instantané incrémentiel l'utilisera automatiquement. Si le stockage ZRS n'est pas disponible dans la région, l'instantané utilisera par défaut le [stockage localement redondant](../articles/storage/common/storage-redundancy.md) (LRS). Vous pouvez remplacer ce comportement et sélectionner un autre type de stockage manuellement, mais nous vous le déconseillons.

Les instantanés incrémentiels offrent également une capacité différentielle, qui est disponible uniquement pour les disques managés. Ils vous permettent d'obtenir les modifications apportées entre deux instantanés incrémentiels des mêmes disques managés, jusqu'au niveau bloc. Vous pouvez utiliser cette fonctionnalité pour réduire l'empreinte de vos données lors de la copie d'instantanés d'une région à une autre.  Par exemple, vous pouvez télécharger le premier instantané incrémentiel en tant qu’objet blob de base dans une autre région. Pour les instantanés incrémentiels suivants, vous pouvez copier uniquement les modifications apportées depuis le dernier instantané sur l’objet blob de base. Après avoir copié les modifications, vous pouvez prendre des instantanés de l’objet blob de base qui représente votre sauvegarde du disque dans une autre région à un moment donné. Vous pouvez restaurer votre disque à partir de l’objet blob de base ou à partir d’un instantané de l’objet blob de base dans une autre région.

:::image type="content" source="media/virtual-machines-disks-incremental-snapshots-description/incremental-snapshot-diagram.png" alt-text="Diagramme illustrant des instantanés incrémentiels copiés dans plusieurs régions. Les instantanés effectuent différents appels d’API jusqu’à ce qu’ils forment éventuellement des objets blob de pages pour chaque instantané.":::

Les instantanés incrémentiels sont facturés uniquement pour la taille utilisée. Vous pouvez connaître la taille utilisée de vos instantanés en examinant le [rapport d’utilisation d’Azure](../articles/cost-management-billing/understand/review-individual-bill.md). Par exemple, si la taille de données utilisée d’un instantané est 10 Gio, le rapport d’utilisation **quotidien** affiche 10 Gio/(31 jours) = 0,3226 Gio comme quantité consommée.