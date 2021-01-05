---
title: Activer le chiffrement de bout en bout à l’aide du chiffrement sur l’hôte – Portail Azure – Disques managés
description: Utilisez le chiffrement sur l’hôte pour activer le chiffrement de bout en bout sur vos disques managés Azure – Portail Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 08/24/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: fd74872bc1c46f3dddda1b6d15f14f26dc2187cb
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499371"
---
# <a name="use-the-azure-portal-to-enable-end-to-end-encryption-using-encryption-at-host"></a>Utilisez le portail Azure pour activer le chiffrement de bout en bout à l’aide du chiffrement sur l’hôte

Quand vous activez le chiffrement sur l’hôte, les données stockées sur l’hôte de machine virtuelle sont chiffrées au repos et les flux sont chiffrés dans le service de stockage. Pour obtenir des informations conceptuelles sur le chiffrement sur l’hôte ainsi que sur d’autres types de chiffrement de disques managés, consultez :

* Linux : [Chiffrement à l’hôte : chiffrement de bout en bout pour vos données de machine virtuelle](./disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

* Windows : [Chiffrement à l’hôte : chiffrement de bout en bout pour vos données de machine virtuelle](./disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

## <a name="restrictions"></a>Restrictions

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### <a name="supported-regions"></a>Régions prises en charge

[!INCLUDE [virtual-machines-disks-encryption-at-host-regions](../../includes/virtual-machines-disks-encryption-at-host-regions.md)]

### <a name="supported-vm-sizes"></a>Tailles des machines virtuelles prises en charge

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

## <a name="prerequisites"></a>Prérequis

Afin de pouvoir utiliser le chiffrement sur l'hôte pour vos machines virtuelles ou groupes de machines virtuelles identiques, vous devez activer la fonctionnalité dans votre abonnement. Envoyez-nous un e-mail à l’adresse encryptionAtHost@microsoft. com avec vos ID d’abonnement pour activer la fonctionnalité.

Connectez-vous au portail Azure à l’aide du [lien fourni](https://aka.ms/diskencryptionupdates).

> [!IMPORTANT]
> Pour accéder au portail Azure, vous devez utiliser le [lien fourni](https://aka.ms/diskencryptionupdates). Le chiffrement au niveau de l’hôte n’est actuellement pas visible dans le portail Azure public si le lien n’est pas utilisé.

### <a name="create-an-azure-key-vault-and-disk-encryption-set"></a>Créer un coffre de clés Azure et un jeu de chiffrement de disque

Une fois la fonctionnalité activée, vous devez configurer un coffre de clés Azure et un jeu de chiffrement de disque, si ce n’est pas déjà fait.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-portal](../../includes/virtual-machines-disks-encryption-create-key-vault-portal.md)]

## <a name="deploy-a-vm"></a>Déployer une machine virtuelle

Vous devez déployer une nouvelle machine virtuelle pour activer le chiffrement au niveau de l’hôte, car il ne peut pas être activé sur les machines virtuelles existantes.

1. Recherchez **Machines virtuelles** et sélectionnez **+ Ajouter** pour créer une machine virtuelle.
1. Créez une nouvelle machine virtuelle, sélectionnez une région appropriée et une taille de machine virtuelle prise en charge.
1. Renseignez les autres valeurs du panneau **Informations de base** à votre guise, puis accédez au panneau **Disques**.

    :::image type="content" source="media/virtual-machines-disks-encryption-at-host-portal/disks-encryption-at-host-basic-blade.png" alt-text="Capture d’écran du panneau Informations de base de la création d’une machine virtuelle, la région et la taille de machine virtuelle sont mises en surbrillance.":::

1. Dans le panneau **Disques**, sélectionnez **Oui** pour **Chiffrement au niveau de l’hôte**.
1. Effectuez les sélections restantes comme vous le souhaitez.

    :::image type="content" source="media/virtual-machines-disks-encryption-at-host-portal/disks-encryption-at-host-disk-blade.png" alt-text="Capture d’écran du panneau Disques de la création d’une machine virtuelle, l’option Chiffrement au niveau de l’hôte est mise en surbrillance.":::

1. Terminez le processus de déploiement de la machine virtuelle en effectuant les sélections qui conviennent à votre environnement.

Vous avez maintenant déployé une machine virtuelle avec chiffrement au niveau de l’hôte, et tous les disques associés seront chiffrés à l’aide du chiffrement au niveau de l’hôte.

## <a name="next-steps"></a>Étapes suivantes

[Exemples de modèles Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)