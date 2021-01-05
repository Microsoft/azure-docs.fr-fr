---
title: Portail Azure - Activez les clés gérées par le client avec les disques managés - SSE
description: Activez les clés gérées par le client sur vos disques managés à l’aide du portail Azure.
author: roygara
ms.date: 08/24/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 5452b1e2ffc3ca8a11f3acb591c6288806445e1b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499388"
---
# <a name="use-the-azure-portal-to-enable-server-side-encryption-with-customer-managed-keys-for-managed-disks"></a>Utiliser le portail Azure pour activer le chiffrement côté serveur à l'aide de clés gérées par le client pour les disques managés

Le Stockage sur disque Azure vous permet de gérer vos propres clés lors de l’utilisation du chiffrement côté serveur (SSE) pour les disques managés, si vous le souhaitez. Pour obtenir des informations conceptuelles sur SSE avec des clés gérées par le client ainsi que d’autres types de chiffrement de disque managés, consultez la section **Clés gérées par le client** de notre article sur le chiffrement de disque :

- Pour Linux : [Clés gérées par le client](./disk-encryption.md#customer-managed-keys)
- Pour Windows : [Clés gérées par le client](./disk-encryption.md#customer-managed-keys)

## <a name="restrictions"></a>Restrictions

Pour le moment, les clés gérées par le client sont soumises aux restrictions suivantes :

- Si cette fonctionnalité est activée pour votre disque, vous ne pouvez pas la désactiver.
    Pour contourner le problème, vous devez copier toutes les données sur un autre disque managé qui n’utilise pas de clés gérées par le client :

    - Pour Linux : [Copier un disque managé](./linux/disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk)

    - Pour Windows : [Copier un disque managé](./windows/disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk)

- Seules les [clés RSA HSM et de logiciels](../key-vault/keys/about-keys.md) des tailles une tailles 2 048 bits, 3 072 bits et 4 096 bits sont prises en charge ; aucune autre clé ou taille n’est prise en charge.
    - Les clés [HSM](../key-vault/keys/hsm-protected-keys.md) nécessitent le niveau **Premium** de coffres de clés Azure.
[!INCLUDE [virtual-machines-managed-disks-customer-managed-keys-restrictions](../../includes/virtual-machines-managed-disks-customer-managed-keys-restrictions.md)]

Les sections suivantes expliquent comment activer et utiliser les clés gérées par le client pour les disques managés :

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-portal](../../includes/virtual-machines-disks-encryption-create-key-vault-portal.md)]

## <a name="deploy-a-vm"></a>Déployer une machine virtuelle

Maintenant que vous avez créé et configuré votre coffre de clés et le jeu de chiffrement de disque, vous pouvez déployer une machine virtuelle à l’aide du chiffrement.
Le processus de déploiement de la machine virtuelle est similaire au processus de déploiement standard, les seules différences sont que vous devez déployer la machine virtuelle dans la même région que vos autres ressources et que vous choisissez d’utiliser une clé gérée par le client.

1. Recherchez **Machines virtuelles** et sélectionnez **+ Ajouter** pour créer une machine virtuelle.
1. Sous l’onglet **De base**, sélectionnez la région de votre jeu de chiffrement de disque et d’Azure Key Vault.
1. Renseignez les autres valeurs du panneau **De base** à votre guise.

    ![Capture d’écran de l’expérience de création de machines virtuelles, dont la valeur de région est mise en surbrillance.](media/virtual-machines-disk-encryption-portal/server-side-encryption-create-a-vm-region.png)

1. Sur le panneau **Disques**, sélectionnez **Chiffrement au repos avec une clé gérée par le client**.
1. Sélectionnez votre jeu de chiffrement de disque dans la liste déroulante **Jeu de chiffrement de disque**.
1. Effectuez les sélections restantes comme vous le souhaitez.

    ![Capture d’écran de l’expérience de création de machines virtuelles, dans le panneau Disques. Avec la liste déroulante Jeu de chiffrement de disque mise en surbrillance.](media/virtual-machines-disk-encryption-portal/server-side-encryption-create-vm-select-customer-managed-key-disk-encryption-set.png)

## <a name="enable-on-an-existing-disk"></a>Activer sur un disque existant

> [!CAUTION]
> L’activation du chiffrement de disque sur les disques attachés à une machine virtuelle nécessite l’arrêt de celle-ci.
    
1. Accédez à une machine virtuelle qui se trouve dans la même région que l’un de vos jeux de chiffrement de disque.
1. Ouvrez la machine virtuelle et sélectionnez **Arrêter**.

    ![Capture d’écran de la superposition principale pour votre exemple de machine virtuelle, avec le bouton Arrêter mis en évidence.](media/virtual-machines-disk-encryption-portal/server-side-encryption-stop-vm-to-encrypt-disk-fix.png)

1. Une fois l’arrêt de la machine virtuelle terminé, sélectionnez **Disques** puis sélectionnez le disque que vous souhaitez chiffrer.

    ![Capture d’écran de votre exemple de machine virtuelle, avec le panneau Disques ouvert. Le disque du système d’exploitation est mis en surbrillance, comme exemple de disque à sélectionner.](media/virtual-machines-disk-encryption-portal/server-side-encryption-existing-disk-select.png)

1. Sélectionnez **Chiffrement**, **Chiffrement au repos avec une clé gérée par le client**, puis sélectionnez votre jeu de chiffrement de disque dans la liste déroulante.
1. Sélectionnez **Enregistrer**.

    ![Capture d’écran de votre exemple de disque de système d’exploitation. Le panneau Chiffrement est ouvert, le chiffrement au repos avec une clé gérée par le client est sélectionné, ainsi que votre exemple Azure Key Vault. Après avoir effectué ces sélections, le bouton Enregistrer est sélectionné.](media/virtual-machines-disk-encryption-portal/server-side-encryption-encrypt-existing-disk-customer-managed-key.png)

1. Répétez ce processus pour tous les autres disques attachés à la machine virtuelle que vous souhaitez chiffrer.
1. S’il n’y a pas d’autres disques attachés que vous souhaitez chiffrer lorsque vos disques ont fini de basculer vers les clés gérées par le client, vous pouvez démarrer votre machine virtuelle.

> [!IMPORTANT]
> Les clés managées par le client s’appuient sur des identités managées pour ressources Azure, une fonctionnalité d’Azure Active Directory (Azure AD). Quand vous configurez des clés gérées par le client, une identité managée est automatiquement affectée à vos ressources à l’arrière plan. Si, par la suite, vous déplacez l’abonnement, le groupe de ressources ou le disque managé d’un répertoire Azure AD à un autre, l’identité managée associée aux disques managés n’est pas transférée vers le nouveau locataire, de sorte que les clés gérées par le client peuvent ne plus fonctionner. Pour plus d’informations, consultez [Transfert d’un abonnement entre des répertoires Azure AD](../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="next-steps"></a>Étapes suivantes

- [Découvrez les modèles Azure Resource Manager permettant de créer des disques chiffrés avec des clés gérées par le client](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [Qu’est-ce qu’Azure Key Vault ?](../key-vault/general/overview.md)
- [Répliquer des machines avec des disques activés par les clés gérées par le client](../site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks.md)
- [Configurer la reprise d’activité des machines virtuelles VMware sur Azure avec PowerShell](../site-recovery/vmware-azure-disaster-recovery-powershell.md#replicate-vmware-vms)
- [Configurer la récupération d’urgence dans Azure pour les machines virtuelles Hyper-V à l’aide de PowerShell et d’Azure Resource Manager](../site-recovery/hyper-v-azure-powershell-resource-manager.md#step-7-enable-vm-protection)