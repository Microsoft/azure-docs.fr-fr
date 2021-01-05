---
title: Redimensionner une machine virtuelle Windows dans Azure
description: Modifier la taille de la machine virtuelle utilisée pour une machine virtuelle Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: sizes
ms.workload: infrastructure
ms.topic: how-to
ms.date: 01/13/2020
ms.author: cynthn
ms.openlocfilehash: 4860422fefb5a95fe41912b4898f02867f7b3aeb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87832244"
---
# <a name="resize-a-windows-vm"></a>Redimensionner une machine virtuelle Windows

Cet article vous explique comment modifier la [taille d’une machine virtuelle](../sizes.md).

Une fois que vous avez créé une machine virtuelle, vous pouvez la mettre à l’échelle en modifiant sa taille. Dans certains cas, vous devez commencer par libérer la machine virtuelle. Cela peut se produire si la nouvelle taille n’est pas disponible sur le cluster matériel qui héberge actuellement la machine virtuelle.

Si votre machine virtuelle utilise le stockage Premium, assurez-vous de choisir une version **s** de la taille pour obtenir un support de stockage Premium. Par exemple, choisissez Standard_E4**s**_v3 au lieu de Standard_E4_v3.

## <a name="use-the-portal"></a>Utiliser le portail

1. Ouvrez le [portail Azure](https://portal.azure.com).
1. Ouvrez la page de la machine virtuelle.
1. Dans le menu de gauche, sélectionnez **Taille**.
1. Choisissez une nouvelle taille dans la liste des tailles disponibles, puis sélectionnez **Redimensionner**.


Si la machine virtuelle est en cours d’exécution et que vous modifiez sa taille, elle redémarre. L’arrêt de la machine virtuelle peut révéler des tailles supplémentaires.

## <a name="use-powershell-to-resize-a-vm-not-in-an-availability-set"></a>Utilisez PowerShell pour redimensionner une machine virtuelle qui ne se trouve pas dans un groupe à haute disponibilité

Définissez des variables. Remplacez les valeurs par vos propres informations.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

Liste les tailles de machines virtuelles qui sont disponibles sur le cluster matériel sur lequel la machine virtuelle est hébergée. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

Si la taille voulue est répertoriée, exécutez les commandes suivantes pour redimensionner la machine virtuelle. Si la taille souhaitée n’est pas répertoriée, passez à l’étape 3.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMsize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

Si la taille voulue n’est pas répertoriée, exécutez les commandes suivantes pour libérer la machine virtuelle, la redimensionner et la redémarrer. Remplacez **\<newVMsize>** par la taille voulue.
   
```powershell
Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName
```

> [!WARNING]
> Le fait de libérer la machine virtuelle libère toutes les adresses IP dynamiques affectées à la machine virtuelle. Les disques de données et du système d’exploitation ne sont pas affectés. 
> 
> 

## <a name="use-powershell-to-resize-a-vm-in-an-availability-set"></a>Utilisez PowerShell pour redimensionner une machine virtuelle qui se trouve dans un groupe à haute disponibilité

Si la nouvelle taille d’une machine virtuelle se trouvant dans un groupe à haute disponibilité n’est pas disponible sur le cluster matériel qui l’héberge actuellement, toutes les machines virtuelles du groupe à haute disponibilité doivent être libérées pour redimensionner la machine virtuelle. Vous devrez peut-être mettre à jour la taille des autres machines virtuelles du groupe à haute disponibilité après le redimensionnement d’une des machines virtuelles. Pour redimensionner une machine virtuelle dans un groupe à haute disponibilité, procédez comme suit.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

Liste les tailles de machines virtuelles qui sont disponibles sur le cluster matériel sur lequel la machine virtuelle est hébergée. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

Si la taille voulue n’est pas répertoriée, exécutez les commandes suivantes pour redimensionner la machine virtuelle. Si elle n’est pas répertoriée, passez à la section suivante.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName 
$vm.HardwareProfile.VmSize = "<newVmSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
    
Si la taille voulue n’est pas répertoriée, passez aux étapes suivantes pour libérer toutes les machines virtuelles du groupe à haute disponibilité, les redimensionner et les redémarrer.

Arrêtez toutes les machines virtuelles du groupe à haute disponibilité.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines |  Stop-AzVM -Force -NoWait  
```

Redimensionnez et redémarrez toutes les machines virtuelles du groupe à haute disponibilité.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$newSize = "<newVmSize>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines | Foreach-Object { $_.HardwareProfile.VmSize = $newSize }
$virtualMachines | Update-AzVM
$virtualMachines | Start-AzVM
```

## <a name="next-steps"></a>Étapes suivantes

Pour une évolutivité supplémentaire, exécutez plusieurs instances de machine virtuelle et effectuez un scale-out. Pour plus d’informations, consultez [Mise à l’échelle automatique des machines Windows dans un groupe de machines virtuelles identiques](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md).