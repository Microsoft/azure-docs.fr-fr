---
title: Convertir les disques non managés d’une machine virtuelle Windows en disques managés
description: Conversion d’une machine virtuelle Windows qui utilise des disques non managés pour qu’elle utilise des disques managés à l’aide de PowerShell dans le modèle de déploiement Resource Manager
author: roygara
ms.service: virtual-machines-windows
ms.subservice: disks
ms.topic: how-to
ms.date: 07/12/2018
ms.author: rogarana
ms.openlocfilehash: 2168b3e3c55eaf06b93358f4623b181f134b4f9e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017342"
---
# <a name="convert-a-windows-virtual-machine-from-unmanaged-disks-to-managed-disks"></a>Convertir les disques non managés d’une machine virtuelle Windows en disques managés

Si vos machines virtuelles Windows existantes utilisent des disques non managés, vous pouvez les convertir pour qu’elles utilisent des disques managés par le biais d’[Azure Disques managés](../managed-disks-overview.md). Ce processus convertit le disque du système d’exploitation ainsi que tous les autres disques de données attachés.

 

## <a name="before-you-begin"></a>Avant de commencer


* Consultez [Planification de la migration vers la fonctionnalité Disques managés](on-prem-to-azure.md#plan-for-the-migration-to-managed-disks).

* Consultez les [questions fréquentes (FAQ) sur la migration vers la fonctionnalité Disques managés](../faq-for-disks.md#migrate-to-managed-disks).

[!INCLUDE [virtual-machines-common-convert-disks-considerations](../../../includes/virtual-machines-common-convert-disks-considerations.md)]

* Les disques durs virtuels et le compte de stockage d’origine utilisé par la machine virtuelle avant la conversion ne sont pas supprimés. Ils continuent à être facturés. Pour éviter la facturation de ces artefacts, supprimez les objets BLOB du disque dur virtuel d’origine après avoir vérifié que la conversion est terminée. Si vous avez besoin de trouver ces disques libres afin de les supprimer, consultez notre article [Rechercher et supprimer les disques managés et non managés Azure non attachés](find-unattached-disks.md).


## <a name="convert-single-instance-vms"></a>Convertir des machines virtuelles à instance unique
Cette section explique comment convertir vos machines virtuelles Azure à instance unique à partir de disques non managés vers des disques managés. (Si vos machines virtuelles sont dans un groupe à haute disponibilité, voir la section suivante.) 

1. Libérez la machine virtuelle à l’aide de la cmdlet [Stop-AzVM](/powershell/module/az.compute/stop-azvm). L’exemple suivant libère la machine virtuelle nommée `myVM` dans le groupe de ressources nommé `myResourceGroup` : 

   ```azurepowershell-interactive
   $rgName = "myResourceGroup"
   $vmName = "myVM"
   Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force
   ```

2. Convertissez la machine virtuelle pour qu’elle utilise des disques managés à l’aide de la cmdlet [ConvertTo-AzVMManagedDisk](/powershell/module/az.compute/convertto-azvmmanageddisk). Le processus suivant convertit la machine virtuelle précédente, y compris le disque du système d’exploitation et tous les disques de données, et démarre la machine virtuelle :

   ```azurepowershell-interactive
   ConvertTo-AzVMManagedDisk -ResourceGroupName $rgName -VMName $vmName
   ```



## <a name="convert-vms-in-an-availability-set"></a>Convertir des machines virtuelles dans un groupe à haute disponibilité

Si les machines virtuelles que vous souhaitez convertir pour utiliser des disques managés se trouvent dans un groupe à haute disponibilité, vous devez tout d’abord convertir ce dernier en groupe à haute disponibilité géré.

1. Convertissez le groupe à haute disponibilité à l’aide de la cmdlet [Update-AzAvailabilitySet](/powershell/module/az.compute/update-azavailabilityset). L’exemple suivant met à jour le groupe à haute disponibilité nommé `myAvailabilitySet` dans le groupe de ressources nommé `myResourceGroup` :

   ```azurepowershell-interactive
   $rgName = 'myResourceGroup'
   $avSetName = 'myAvailabilitySet'

   $avSet = Get-AzAvailabilitySet -ResourceGroupName $rgName -Name $avSetName
   Update-AzAvailabilitySet -AvailabilitySet $avSet -Sku Aligned 
   ```

   Si la région où votre groupe à haute disponibilité se situe n’a que 2 domaines d’erreur gérés, mais que le nombre de domaines d’erreur non gérés est de 3, cette commande affiche une erreur semblable à « Le nombre de domaines d’erreur spécifié 3 doit être compris entre 1 et 2 ». Pour résoudre l’erreur, mettez à jour le domaine par défaut sur 2 et `Sku` sur `Aligned`, comme suit :

   ```azurepowershell-interactive
   $avSet.PlatformFaultDomainCount = 2
   Update-AzAvailabilitySet -AvailabilitySet $avSet -Sku Aligned
   ```

2. Désallouer et convertir des machines virtuelles dans le groupe à haute disponibilité. Le script suivant libère chaque machine virtuelle à l’aide de la cmdlet [Stop-AzVM](/powershell/module/az.compute/stop-azvm), la convertit à l’aide de [ConvertTo-AzVMManagedDisk](/powershell/module/az.compute/convertto-azvmmanageddisk) et la redémarre automatiquement dans le cadre du processus de conversion :

   ```azurepowershell-interactive
   $avSet = Get-AzAvailabilitySet -ResourceGroupName $rgName -Name $avSetName

   foreach($vmInfo in $avSet.VirtualMachinesReferences)
   {
     $vm = Get-AzVM -ResourceGroupName $rgName | Where-Object {$_.Id -eq $vmInfo.id}
     Stop-AzVM -ResourceGroupName $rgName -Name $vm.Name -Force
     ConvertTo-AzVMManagedDisk -ResourceGroupName $rgName -VMName $vm.Name
   }
   ```


## <a name="troubleshooting"></a>Dépannage

Si une erreur survient lors de la conversion, ou si une machine virtuelle est dans un état d’échec en raison de problèmes dans une conversion précédente, exécutez l’applet de commande `ConvertTo-AzVMManagedDisk` à nouveau. Généralement, une simple nouvelle tentative suffit à débloquer la situation.
Avant de lancer la conversion, assurez-vous que toutes les extensions de machine virtuelle sont dans l’état « Provisionnement réussi ». Sinon, la conversion échoue avec le code d’erreur 409.

## <a name="convert-using-the-azure-portal"></a>Convertir à l’aide du portail Azure

Vous pouvez également convertir des disques non managés en disques managés à l’aide du portail Azure.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez la machine virtuelle dans la liste des machines virtuelles sur le portail.
3. Dans le panneau de la machine virtuelle, sélectionnez **Disques** dans le menu.
4. En haut du panneau **Disques**, sélectionnez **Migrer vers des disques managés**.
5. Si votre machine virtuelle se trouve dans un groupe à haute disponibilité, un avertissement apparaît sur le panneau **Migrer vers des disques managés**, indiquant que vous devez commencer par convertir le groupe à haute disponibilité. L’avertissement doit avoir un lien sur lequel vous pouvez cliquer pour convertir le groupe à haute disponibilité. Une fois le groupe à haute disponibilité converti, ou si votre machine virtuelle ne se trouve pas dans un groupe à haute disponibilité, cliquez sur **Migrate** pour démarrer le processus de migration de vos disques vers des disques managés.

Lorsque la migration est terminée, la machine virtuelle est arrêtée, puis redémarrée.

## <a name="next-steps"></a>Étapes suivantes

[Convertir des disques managés standard en premium](convert-disk-storage.md)

Créez une copie en lecture seule d’une machine virtuelle en utilisant des [captures instantanées](snapshot-copy-managed-disk.md).