---
title: Portail Azure – Restreindre l’accès en importation/exportation à des disques managés avec des liaisons privées
description: Activez des liaisons privées pour vos disques managés avec le portail Azure. Vous permet d’exporter et d’importer des disques de manière sécurisée au sein de votre réseau virtuel.
author: roygara
ms.service: virtual-machines
ms.topic: overview
ms.date: 08/24/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 68a308952b63d15aa6db90021400f826ef575c0c
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94537184"
---
# <a name="use-the-azure-portal-to-restrict-importexport-access-for-managed-disks-with-private-links"></a>Utiliser le portail Azure afin de restreindre l’accès par importation/exportation aux disques managés avec des liaisons privées

La prise en charge des liaisons privées pour des disques managés vous permet de limiter l’exportation et l’importation de disques managés afin qu’elles ne se produisent que dans votre réseau virtuel Azure. Vous pouvez générer un URI de signature d’accès partagé (SAS) limité dans le temps pour les instantanés et les disques managés non attachés afin d’exporter les données vers une autre région dans le cadre d’une expansion régionale, d’une reprise d’activité après sinistre ou de la lecture des données à des fins d’analyse forensique. Vous pouvez également utiliser l’URI SAS pour charger directement le disque dur virtuel sur un disque vide à partir de votre site local. Le trafic réseau entre clients sur leurs réseau virtuel et disques managés transite uniquement sur le réseau virtuel et une liaison privée sur le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public.

Vous pouvez créer une ressource d’accès au disque et la lier à votre réseau virtuel dans le même abonnement en créant un point de terminaison privé. Vous devez associer un disque ou un instantané avec accès au disque pour exporter et importer les données au moyen de liaisons privées. Vous devez également définir la propriété NetworkAccessPolicy du disque ou de l’instantané avec `AllowPrivate`. 

Vous pouvez définir la propriété NetworkAccessPolicy avec `DenyAll` pour empêcher toute personne de générer l’URI SAS pour un disque ou un instantané. La valeur par défaut de la propriété NetworkAccessPolicy est `AllowAll`.

## <a name="limitations"></a>Limites

[!INCLUDE [virtual-machines-disks-private-links-limitations](../../includes/virtual-machines-disks-private-links-limitations.md)]


## <a name="create-a-disk-access-resource"></a>Créer une ressource d’accès au disque

1. Connectez-vous au portail Azure et suivez [ce lien](https://aka.ms/disksprivatelinks) pour accéder à **Accès au disque**.

    > [!IMPORTANT]
    > Vous devez utiliser le [lien fourni](https://aka.ms/disksprivatelinks) pour accéder au panneau Accès au disque. Celui-ci n’est pas actuellement visible dans le portail public.

1. Sélectionnez **+ Ajouter** pour créer une ressource d’accès au disque.
1. Dans le panneau de création, sélectionnez votre abonnement, un groupe de ressources, entrez un nom, puis sélectionnez une région.
1. Sélectionnez **Revoir + créer**.

    :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-create-basics.png" alt-text="Capture d’écran du panneau de création d’accès au disque. Indiquez un nom, sélectionnez une région et un groupe de ressources, puis continuez.":::

Une fois votre ressource créée, accédez directement à celle-ci.

:::image type="content" source="media/disks-enable-private-links-for-import-export-portal/screenshot-resource-button.png" alt-text="Capture d’écran du bouton Accéder à la ressource dans le portail":::

## <a name="create-a-private-endpoint"></a>Créer un Private Endpoint

Maintenant que vous disposez d’une ressource d’accès au disque, vous pouvez l’utiliser pour gérer l’accès aux opérations d’exportation/importation de votre disque à l’aide de points de terminaison privés. Vous devez donc créer un point de terminaison privé et le configurer pour l’accès au disque.

1. À partir de votre ressource d’accès au disque, sélectionnez **Connexions des points de terminaison privés**.
1. Sélectionnez **+ Point de terminaison privé**.

    :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-main-private-blade.png" alt-text="Capture d’écran du panneau de vue d’ensemble de votre ressource d’accès au disque. L’option Connexions des points de terminaison privés est mise en évidence.":::

1. Sélectionner un groupe de ressources
1. Indiquez un nom et sélectionnez la même région que celle dans laquelle votre ressource d’accès au disque a été créée.
1. Sélectionnez **Suivant : Ressource >**

    :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-private-endpoint-first-blade.png" alt-text="Capture d’écran du workflow de création d’un point de terminaison privé, premier panneau. Si vous ne sélectionnez pas la région appropriée, vous risquez de rencontrer des problèmes par la suite.":::

1. Dans le panneau **Ressource**, sélectionnez **Se connecter à une ressource Azure dans mon annuaire**.
1. Pour **Type de ressource**, sélectionnez **Microsoft.Compute/diskAccesses**.
1. Pour **Resource**, sélectionnez la ressource d’accès au disque créée précédemment.
1. Pour **Sous-ressource ciblen**, conservez **disks**.
1. Sélectionnez **Suivant : Configuration >** .

    :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-private-endpoint-second-blade.png" alt-text="Capture d’écran du workflow de création d’un point de terminaison privé, deuxième panneau. Toutes les valeurs sont mises en évidence : Type de ressource, Ressource, Sous-ressource cible.":::

1. Sélectionnez le réseau virtuel auquel vous souhaitez limiter l’exportation du disque. Les autres réseaux virtuels ne pourront pas exporter votre disque.

    > [!NOTE]
    > Si un groupe de sécurité réseau (NSG) est activé pour le sous-réseau sélectionné, il sera désactivé pour les points de terminaison privés sur ce sous-réseau uniquement. Les autres ressources de ce sous-réseau seront toujours soumises au NSG.

1. Sélectionner le sous-réseau approprié
1. Sélectionnez **Revoir + créer**.

    :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-private-endpoint-third-blade.png" alt-text="Capture d’écran du workflow de création d’un point de terminaison privé, troisième panneau. Les options Réseau virtuel et Sous-réseau sont mise en évidence.":::

## <a name="enable-private-endpoint-on-your-disk"></a>Activer un point de terminaison privé sur votre disque

1. Accédez au disque que vous souhaitez configurer.
1. Sélectionner **Mise en réseau**
1. Sélectionnez **Point de terminaison privé (via l’accès de disque)** et sélectionnez l’accès au disque créé précédemment.
1. Sélectionnez **Enregistrer**.

    :::image type="content" source="media/disks-enable-private-links-for-import-export-portal/disk-access-managed-disk-networking-blade.png" alt-text="Capture d’écran du panneau Réseau pour l’accès au disque. La sélection du point de terminaison privé et de l’accès au disque sont mises en évidence. Enregistrez pour configurer votre disque avec cet accès.":::

Vous venez de terminer la configuration des liaisons privées, que vous pouvez maintenant utiliser lors de l’importation ou de l’exportation de votre disque managé.

## <a name="next-steps"></a>Étapes suivantes

- [Questions fréquentes (FAQ) sur les liaisons privées](./faq-for-disks.md#private-links-for-securely-exporting-and-importing-managed-disks)
- [Exporter/copier des instantanés managés en tant que disque dur virtuel vers un compte de stockage dans une région différente avec PowerShell](./scripts/virtual-machines-powershell-sample-copy-snapshot-to-storage-account.md)
