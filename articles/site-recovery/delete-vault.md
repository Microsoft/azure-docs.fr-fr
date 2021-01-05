---
title: Supprimer un coffre Azure Site Recovery
description: Découvrez comment supprimer un coffre Recovery Services configuré pour Azure Site Recovery
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/05/2019
ms.author: sideeksh
ms.openlocfilehash: a33e04a24013d5450c98b91048fa418958d16886
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89426382"
---
# <a name="delete-a-site-recovery-services-vault"></a>Supprimer un coffre de services Site Recovery

Cet article explique comment supprimer un coffre Recovery Services pour Site Recovery. Pour supprimer un archivage utilisé dans Sauvegarde Microsoft Azure, voir [Supprimer un archivage Recovery Services](../backup/backup-azure-delete-vault.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]


## <a name="before-you-start"></a>Avant de commencer

Avant de pouvoir supprimer un coffre, vous devez supprimer les serveurs inscrits et les éléments du coffre. Ce que vous devez supprimer dépend des scénarios de réplication que vous avez déployés. 


## <a name="delete-a-vault-azure-vm-to-azure"></a>Supprimer un coffre : machine virtuelle Azure sur Azure

1. Suivez [ces instructions](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-azure-vm-azure-to-azure) pour supprimer toutes les machines virtuelles protégées.
2. Ensuite, supprimez le coffre.

## <a name="delete-a-vault-vmware-vm-to-azure"></a>Supprimer un coffre : machine virtuelle VMware sur Azure

1. Suivez [ces instructions](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) pour supprimer toutes les machines virtuelles protégées.
2. Suivez [ces étapes](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) pour supprimer toutes les stratégies de réplication.
3. Supprimez les références à vCenter à l’aide de [ces étapes](vmware-azure-manage-vcenter.md#delete-a-vcenter-server).
4. Suivez [ces instructions](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server) pour désactiver un serveur de configuration.
5. Ensuite, supprimez le coffre.


## <a name="delete-a-vault-hyper-v-vm-with-vmm-to-azure"></a>Supprimer un coffre : machine virtuelle Hyper-V (avec VMM) sur Azure

1. Suivez [ces étapes](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario) pour supprimer les machines virtuelles Hyper-V gérées par System Center VMM.
2. Dissociez et supprimez toutes les stratégies de réplication. Effectuez ce qui suit dans votre coffre > **Infrastructure Site Recovery** > **Pour System Center VMM** > **Stratégies de réplication**.
3. Suivez [ces étapes](site-recovery-manage-registration-and-protection.md#unregister-a-vmm-server) pour annuler l’inscription d’un serveur VMM connecté.
4. Ensuite, supprimez le coffre.

## <a name="delete-a-vault-hyper-v-vm-to-azure"></a>Supprimer un coffre : machine virtuelle Hyper-V sur Azure

1. Suivez [ces étapes](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure) pour supprimer toutes les machines virtuelles protégées.
2. Dissociez et supprimez toutes les stratégies de réplication. Effectuez ce qui suit dans votre coffre > **Infrastructure Site Recovery** > **Pour Sites Hyper-V** > **Stratégies de réplication**.
3. Suivez [ces instructions](site-recovery-manage-registration-and-protection.md#unregister-a-hyper-v-host-in-a-hyper-v-site) pour annuler l’inscription d’un hôte Hyper-V.
4. Supprimez le site Hyper-V.
5. Ensuite, supprimez le coffre.


## <a name="use-powershell-to-force-delete-the-vault"></a>Utiliser PowerShell pour forcer la suppression de l’archivage 

> [!Important]
> Si vous testez le produit et ne vous inquiétez pas de perdre des données, utilisez la méthode de suppression de force pour supprimer rapidement l’archivage et toutes ses dépendances.
> La commande PowerShell supprime tout le contenu de l’archivage et n’est **pas réversible**.

Pour supprimer l’archivage Site Recovery, même s’il contient des éléments protégés, utilisez les commandes suivantes :

```azurepowershell
Connect-AzAccount

Select-AzSubscription -SubscriptionName "XXXXX"

$vault = Get-AzRecoveryServicesVault -Name "vaultname"

Remove-AzRecoveryServicesVault -Vault $vault
```

Découvrez-en plus sur [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault) et [Remove-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/remove-azrecoveryservicesvault).
