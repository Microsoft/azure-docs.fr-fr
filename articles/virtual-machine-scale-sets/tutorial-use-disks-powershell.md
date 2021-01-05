---
title: Tutoriel - Créer et utiliser des disques pour les groupes identiques avec Azure PowerShell
description: Découvrez comment utiliser Azure PowerShell pour créer et utiliser la fonctionnalité Disques managés avec des groupes identiques de machines virtuelles, notamment comment ajouter, préparer, répertorier et détacher des disques.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 9e995e88b80bf14f9c7784f465bcd3d89d0bed65
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92367956"
---
# <a name="tutorial-create-and-use-disks-with-virtual-machine-scale-set-with-azure-powershell"></a>Didacticiel : Créer et utilisez les disques avec un groupe de machines virtuelles identiques à l’aide de Azure PowerShell

Les groupes de machines virtuelles identiques utilisent des disques pour stocker le système d’exploitation, les applications et les données de l’instance de machine virtuelle. Lorsque vous créez et gérez un groupe identique, il est important de choisir une taille de disque et une configuration appropriées à la charge de travail prévue. Ce didacticiel explique comment créer et gérer des disques de machine virtuelle. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attacher et préparer des disques de données

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="default-azure-disks"></a>Disques Azure par défaut
Lorsqu’un groupe identique est créé ou mis à l’échelle, deux disques sont automatiquement attachés à chaque instance de machine virtuelle. 

**Disque de système d’exploitation** : la taille des disques de système d’exploitation peut atteindre 2 To. Ces disques hébergent le système d’exploitation de l’instance de machine virtuelle. Le disque de système d’exploitation est nommé */dev/sda* par défaut. La configuration de la mise en cache de disque de système d’exploitation est optimisée pour les performances du système d’exploitation. En raison de cette configuration, le disque de système d’exploitation **ne doit pas** héberger d’applications ou de données. Pour héberger ce type de contenu, utilisez plutôt des disques de données, qui sont décrits plus loin dans cet article. 

**Disque temporaire** : les disques temporaires utilisent un disque SSD qui se trouve sur le même hôte Azure que l’instance de machine virtuelle. Ce sont des disques extrêmement performants qui peuvent être utilisés pour diverses opérations telles que le traitement de données temporaires. Toutefois, si l’instance de machine virtuelle est déplacée vers un nouvel hôte, toutes les données stockées sur le disque temporaire concerné sont supprimées. La taille du disque temporaire est déterminée par la taille de l’instance de machine virtuelle. Les disques temporaires sont nommés */dev/sdb* et ont un point de montage */mnt*.

### <a name="temporary-disk-sizes"></a>Tailles du disque temporaire
| Type | Tailles courantes | Taille maximale du disque temporaire (Gio) |
|----|----|----|
| [Usage général](../virtual-machines/sizes-general.md) | Séries A, B et D | 1 600 |
| [Optimisé pour le calcul](../virtual-machines/sizes-compute.md) | Série F | 576 |
| [Mémoire optimisée](../virtual-machines/sizes-memory.md) | Séries D, E, G et M | 6144 |
| [Optimisé pour le stockage](../virtual-machines/sizes-storage.md) | Série L | 5630 |
| [GPU](../virtual-machines/sizes-gpu.md) | Série N | 1440 |
| [Hautes performances](../virtual-machines/sizes-hpc.md) | Séries A et H | 2000 |


## <a name="azure-data-disks"></a>Disques de données Azure
Des disques de données supplémentaires peuvent être ajoutés si vous avez besoin d’installer des applications et de stocker des données. Les disques de données doivent être utilisés dans les cas où un stockage des données durable et réactif est souhaité. Chaque disque de données offre une capacité maximale de 4 To. La taille de l’instance de machine virtuelle détermine le nombre de disques de données pouvant être attachés. Pour chaque processeur virtuel de la machine virtuelle, deux disques de données peuvent être attachés.

### <a name="max-data-disks-per-vm"></a>Disques de données max. par machine virtuelle
| Type | Tailles courantes | Disques de données max. par machine virtuelle |
|----|----|----|
| [Usage général](../virtual-machines/sizes-general.md) | Séries A, B et D | 64 |
| [Optimisé pour le calcul](../virtual-machines/sizes-compute.md) | Série F | 64 |
| [Mémoire optimisée](../virtual-machines/sizes-memory.md) | Séries D, E, G et M | 64 |
| [Optimisé pour le stockage](../virtual-machines/sizes-storage.md) | Série L | 64 |
| [GPU](../virtual-machines/sizes-gpu.md) | Série N | 64 |
| [Hautes performances](../virtual-machines/sizes-hpc.md) | Séries A et H | 64 |


## <a name="vm-disk-types"></a>Type de disque de machine virtuelle
Azure propose deux types de disque.

### <a name="standard-disk"></a>Disque Standard
Le stockage Standard s’appuie sur des disques HDD et offre un stockage économique et performant. Les disques Standard constituent la solution idéale pour une charge de travail de développement et de test économique.

### <a name="premium-disk"></a>Disque Premium
Les disques Premium reposent sur un disque SSD à faible latence et hautes performances. Ces disques sont recommandés pour les machines virtuelles qui exécutent des charges de travail de production. Le stockage Premium prend en charge les machines virtuelles des séries DS, DSv2, GS et FS. Lorsque vous sélectionnez une taille de disque, la valeur est arrondie au type suivant. Par exemple, si la taille du disque est inférieure à 128 Go, le type de disque est P10. Si la taille du disque est entre 129 Go et 512 Go, le type de disque est P20. Au-dessus de 512 Go, le type de disque est P30.

### <a name="premium-disk-performance"></a>Performances du disque Premium
|Type de disque de stockage Premium | P4 | P6 | P10 | P20 | P30 | P40 | P50 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Taille du disque (arrondie) | 32 Go | 64 Go | 128 Go | 512 Go | 1 024 Go (1 To) | 2 048 Go (2 To) | 4 095 Go (4 To) |
| Nb max. d'E/S par seconde par disque | 120 | 240 | 500 | 2 300 | 5 000 | 7 500 | 7 500 |
Débit par disque | 25 Mo/s | 50 Mo/s | 100 Mo/s | 150 Mo/s | 200 Mo/s | 250 Mo/s | 250 Mo/s |

Bien que le tableau ci-dessus identifie le nombre max. d’E/S par seconde par disque, un niveau de performances plus élevé est possible en entrelaçant plusieurs disques de données. Par exemple, une machine virtuelle Standard_GS5 peut atteindre un nombre maximum d’E/S par seconde de 80 000. Pour plus d’informations sur le nombre maximal d’E/S par seconde par machine virtuelle, consultez [Tailles des machines virtuelles Windows](../virtual-machines/sizes.md).


## <a name="create-and-attach-disks"></a>Créer et attacher des disques
Vous pouvez créer et attacher des disques lorsque vous créez un groupe identique, ou avec un groupe identique existant.

À compter de la version `2019-07-01` de l’API, vous pouvez définir la taille du disque du système d’exploitation dans un groupe de machines virtuelles identiques avec la propriété [storageProfile.osDisk.diskSizeGb](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosdisk). Après le provisionnement, il peut être nécessaire d’étendre ou de repartitionner le disque pour utiliser tout l’espace. Découvrez plus en détail [l’extension du disque ici](../virtual-machines/windows/expand-os-disk.md#expand-the-volume-within-the-os).

### <a name="attach-disks-at-scale-set-creation"></a>Attacher des disques lors de la création d’un groupe identique
Créez un groupe de machines virtuelles identiques avec [New-AzVmss](/powershell/module/az.compute/new-azvmss). Lorsque vous y êtes invité, saisissez un nom d’utilisateur et le mot de passe correspondant pour les instances de machine virtuelle. Pour distribuer le trafic aux différentes instances de machine virtuelle, un équilibreur de charge est également créé. L’équilibreur de charge inclut des règles pour distribuer le trafic sur le port TCP 80, ainsi que pour autoriser le trafic Bureau à distance sur le port TCP 3389 et le trafic Accès distant PowerShell sur le port TCP 5985.

Deux disques sont créés avec le paramètre `-DataDiskSizeGb`. Le premier disque fait *64* Go et le second disque fait *128* Go. Lorsque vous y êtes invité, fournissez vos propres informations d’identification d’administration souhaitées pour les instances de machine virtuelle dans le groupe identique :

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic" `
  -DataDiskSizeInGb 64,128
```

La création et la configuration de l’ensemble des ressources et des instances de machine virtuelle du groupe identique prennent quelques minutes.

### <a name="attach-a-disk-to-existing-scale-set"></a>Attacher un disque à un groupe identique existant
Vous pouvez également attacher des disques à un groupe identique existant. Utilisez le groupe identique créé à l’étape précédente pour ajouter un autre disque avec la commande [Add-AzVmssDataDisk](/powershell/module/az.compute/add-azvmssdatadisk). Dans l’exemple suivant, un disque supplémentaire de *128* Go est associé à un groupe identique existant :

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Attach a 128 GB data disk to LUN 2
Add-AzVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -CreateOption Empty `
  -Lun 2 `
  -DiskSizeGB 128

# Update the scale set to apply the change
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="prepare-the-data-disks"></a>Préparer les disques de données
Les disques créés et attachés aux instances de machine virtuelle de votre groupe identique sont des disques bruts. Avant que vous ne puissiez les utiliser avec vos données et vos applications, les disques doivent être préparés. Pour préparer les disques, vous devez créer une partition, créer un système de fichiers, et les monter.

Pour automatiser le processus sur plusieurs instances de machine virtuelle dans un groupe identique, vous pouvez utiliser l’extension de script personnalisé Azure. Cette extension peut exécuter des scripts localement sur chaque instance de machine virtuelle, par exemple pour préparer les disques de données attachés. Pour plus d’informations, consultez [Vue d’ensemble de l’extension de script personnalisé](../virtual-machines/extensions/custom-script-windows.md).


L’exemple suivant présente l’exécution d’un script à partir d’un dépôt d’exemples GitHub sur chaque instance de machine virtuelle avec la commande [Add-AzVmssExtension](/powershell/module/az.compute/Add-AzVmssExtension) qui prépare tous les disques de données attachés bruts :


```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Define the script for your Custom Script Extension to run
$publicSettings = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File prepare_vm_disks.ps1"
}

# Use Custom Script Extension to prepare the attached data disks
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Pour confirmer que les disques ont été correctement préparés, utilisez le protocole RDP sur l’une des instances de machine virtuelle. 

Tout d’abord, obtenez l’objet d’équilibrage de charge avec [Get-AzLoadBalancer](/powershell/module/az.network/Get-AzLoadBalancer). Affichez ensuite les règles NAT de trafic entrant avec [Get-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/Get-AzLoadBalancerInboundNatRuleConfig). Les règles NAT répertorient le *port du serveur frontal* pour chaque instance de machine virtuelle qui écoute le protocole RDP. Enfin, obtenez l’adresse IP publique de l’équilibreur de charge avec [Get-AzPublicIpAddress](/powershell/module/az.network/Get-AzPublicIpAddress) :


```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort

# View the public IP address of the load balancer
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" -Name myPublicIPAddress | Select IpAddress
```

Pour vous connecter à votre machine virtuelle, spécifiez votre propre adresse IP publique et le numéro de port de l’instance de machine virtuelle, comme indiqué dans les commandes précédentes. À l’invite, saisissez les informations d’identification que vous avez utilisées lors de la création du groupe identique. Si vous utilisez Azure Cloud Shell, effectuez cette étape à partir d’une invite de commandes PowerShell locale ou du client Bureau à distance. L’exemple suivant établit une connexion à l’instance de machine virtuelle *1* :

```powershell
mstsc /v 52.168.121.216:50001
```

Ouvrez une session PowerShell locale sur l’instance de machine virtuelle et examinez les disques connectés avec [Get-Disk](/powershell/module/storage/get-disk) :

```powershell
Get-Disk
```

L’exemple de sortie suivant montre que trois disques de données sont attachés à l’instance de machine virtuelle.

```powershell
Number  Friendly Name      HealthStatus  OperationalStatus  Total Size  Partition Style
------  -------------      ------------  -----------------  ----------  ---------------
0       Virtual HD         Healthy       Online                 127 GB  MBR
1       Virtual HD         Healthy       Online                  14 GB  MBR
2       Msft Virtual Disk  Healthy       Online                  64 GB  MBR
3       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
4       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
```

Examinez les systèmes de fichiers et les points de montage sur l’instance de machine virtuelle comme suit :

```powershell
Get-Partition
```

L’exemple de sortie suivant montre que trois disques de données se voient attribuer des lettres de lecteur :

```powershell
   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000001

PartitionNumber  DriveLetter  Offset   Size  Type
---------------  -----------  ------   ----  ----
1                F            1048576  64 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000002

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                G            1048576  128 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000003

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                H            1048576  128 GB  IFS
```

Les disques sur chaque instance de machine virtuelle de votre groupe identique sont préparés automatiquement de la même façon. Lorsque votre groupe identique monte en puissance, les disques de données requis sont attachés aux nouvelles instances de machine virtuelle. L’extension de script personnalisé s’exécute aussi pour préparer automatiquement les disques.

Fermez la session de connexion Bureau à distance avec l’instance de machine virtuelle.


## <a name="list-attached-disks"></a>Afficher les disques attachés
Pour afficher des informations sur les disques attachés à un groupe identique, utilisez [Get-AzVmss](/powershell/module/az.compute/get-azvmss) comme suit :

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
```

Sous la propriété *VirtualMachineProfile.StorageProfile*, la liste des *disques de données* est affichée. Les informations sur la taille du disque, le niveau de stockage et le numéro d’unité logique s’affichent. L’exemple de sortie suivant détaille les trois disques de données attachés au groupe identique :

```powershell
DataDisks[0]                            :
  Lun                                   : 0
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 64
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[1]                            :
  Lun                                   : 1
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[2]                            :
  Lun                                   : 2
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
```


## <a name="detach-a-disk"></a>Détacher un disque
Lorsque vous n’avez plus besoin d’un disque donné, vous pouvez le détacher du groupe identique. Le disque est supprimé de toutes les instances de machine virtuelle dans le groupe identique. Pour détacher un disque d’un groupe identique, utilisez la commande [Remove-AzVmssDataDisk](/powershell/module/az.compute/remove-azvmssdatadisk) et spécifiez le numéro d’unité logique du disque. Les numéros d’unité logique sont indiqués dans la sortie de la commande [Get-AzVmss](/powershell/module/az.compute/get-azvmss) mentionnée dans la section précédente. L’exemple ci-après illustre le détachement du numéro d’unité logique *3* du groupe identique :

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Detach a disk from the scale set
Remove-AzVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -Lun 2

# Update the scale set and detach the disk from the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="clean-up-resources"></a>Nettoyer les ressources
Pour supprimer votre groupe identique et vos disques, supprimez le groupe de ressources et toutes ses ressources avec [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). Le paramètre `-Force` confirme que vous souhaitez supprimer les ressources sans passer par une invite supplémentaire à cette fin. Le paramètre `-AsJob` retourne le contrôle à l’invite de commandes sans attendre que l’opération se termine.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à créer et utiliser des disques avec des groupes identiques au moyen d’Azure PowerShell :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attacher et préparer des disques de données

Passez à l’étape suivante du didacticiel afin d’apprendre à utiliser une image personnalisée pour les instances de machine virtuelle de votre groupe identique.

> [!div class="nextstepaction"]
> [Utiliser une image personnalisée pour les instances de machine virtuelle d’un groupe identique](tutorial-use-custom-image-powershell.md)