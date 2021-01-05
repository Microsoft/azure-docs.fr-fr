---
title: Fichier include
description: Fichier include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 04/02/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: de0cf260207747f4acb02a377819a13de8b8ba22
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002514"
---
[!INCLUDE [virtual-machines-disks-incremental-snapshots-description](virtual-machines-disks-incremental-snapshots-description.md)]

## <a name="regional-availability"></a>Disponibilité régionale
[!INCLUDE [virtual-machines-disks-incremental-snapshots-regions](virtual-machines-disks-incremental-snapshots-regions.md)]

### <a name="restrictions"></a>Restrictions

[!INCLUDE [virtual-machines-disks-incremental-snapshots-restrictions](virtual-machines-disks-incremental-snapshots-restrictions.md)]

## <a name="portal"></a>Portail


1. Connectez-vous au [portail Azure](https://portal.azure.com/) et accédez au disque dont vous voulez prendre une capture instantanée.
1. Sur votre disque, sélectionnez **Créer un instantané**

    :::image type="content" source="media/virtual-machines-disks-incremental-snapshots-portal/create-snapshot-button-incremental.png" alt-text="Capture d’écran. Le panneau de votre disque, avec **+Créer un instantané** mis en surbrillance, car c’est ce que vous devez sélectionner.":::

1. Sélectionnez le groupe de ressources que vous souhaitez utiliser, puis entrez un nom.
1. Sélectionnez **Incrémentiel**, puis **Vérifier + créer**

    :::image type="content" source="media/virtual-machines-disks-incremental-snapshots-portal/incremental-snapshot-create-snapshot-blade.png" alt-text="Capture d’écran. Créez un panneau d’instantané, indiquez le nom, sélectionnez Incrémentiel, puis créez votre instantané.":::

1. Sélectionnez **Créer**

    :::image type="content" source="media/virtual-machines-disks-incremental-snapshots-portal/create-incremental-snapshot-validation.png" alt-text="Capture d’écran. Page de validation de votre instantané, confirmez vos sélections, puis créez l’instantané.":::

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez voir un exemple de code illustrant la capacité différentielle des instantanés incrémentiels à l’aide de .NET, consultez [Copy Azure Managed Disks backups to another region with differential capability of incremental snapshots](https://github.com/Azure-Samples/managed-disks-dotnet-backup-with-incremental-snapshots) (Copier des sauvegardes de disques managés Azure dans une autre région avec la fonctionnalité différentielle des instantanés incrémentiels).
