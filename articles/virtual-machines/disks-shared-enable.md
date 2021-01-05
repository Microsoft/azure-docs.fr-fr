---
title: Activer la fonctionnalité Disques partagés pour les disques managés Azure
description: Configurer un disque managé Azure avec la fonctionnalité Disques partagés afin de pouvoir le partager avec plusieurs machines virtuelles
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 09/30/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: e8bb97196fcceea0c86f58fe4a63146e681c427e
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500731"
---
# <a name="enable-shared-disk"></a>Activer la fonctionnalité Disques partagés

Cet article explique comment activer la fonctionnalité Disques partagés pour les disques managés Azure. La fonctionnalité Disques partagés Azure est une nouvelle fonctionnalité pour disques managés Azure qui vous permet de connecter simultanément un disque managé à plusieurs machines virtuelles. La connexion d'un disque managé à plusieurs machines virtuelles vous permet de déployer de nouvelles applications en cluster ou de migrer des applications en cluster existantes vers Azure. 

Si vous recherchez des informations conceptuelles relatives aux disques managés sur lesquels la fonctionnalité Disques partagés est activée, consultez [Disques partagés Azure](disks-shared.md).

## <a name="limitations"></a>Limites

[!INCLUDE [virtual-machines-disks-shared-limitations](../../includes/virtual-machines-disks-shared-limitations.md)]

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

Les disques partagés prennent en charge plusieurs systèmes d’exploitation. Consultez les sections [Windows](./disks-shared.md#windows) et [Linux](./disks-shared.md#linux) de l’article conceptuel relatif aux systèmes d’exploitation pris en charge.

## <a name="disk-sizes"></a>Tailles du disque

[!INCLUDE [virtual-machines-disks-shared-sizes](../../includes/virtual-machines-disks-shared-sizes.md)]

## <a name="deploy-shared-disks"></a>Déployer des disques partagés

### <a name="deploy-a-premium-ssd-as-a-shared-disk"></a>Déployer un disque SSD Premium en tant que disque partagé

Pour déployer un disque managé avec la fonctionnalité de disque partagé activée, utilisez la nouvelle propriété `maxShares` et définissez une valeur supérieure à 1. Cela permet de partager le disque sur plusieurs machines virtuelles.

> [!IMPORTANT]
> La valeur de `maxShares` peut être définie ou modifiée uniquement lorsqu’un disque est démonté de toutes les machines virtuelles. Consultez les [tailles de disque](#disk-sizes) afin de connaître les valeurs autorisées pour `maxShares`.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az disk create -g myResourceGroup -n mySharedDisk --size-gb 1024 -l westcentralus --sku Premium_LRS --max-shares 2
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$dataDiskConfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType Premium_LRS -CreateOption Empty -MaxSharesCount 2

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $dataDiskConfig
```

# <a name="resource-manager-template"></a>[Modèle Resource Manager](#tab/azure-resource-manager)

Avant d’utiliser le modèle suivant, remplacez `[parameters('dataDiskName')]`, `[resourceGroup().location]`, `[parameters('dataDiskSizeGB')]` et `[parameters('maxShares')]` par vos propres valeurs.

[SSD Premium modèle de disque partagé](https://aka.ms/SharedPremiumDiskARMtemplate)

---

### <a name="deploy-an-ultra-disk-as-a-shared-disk"></a>Déployer un disque Ultra en tant que disque partagé

Pour déployer un disque managé avec la fonctionnalité de disque partagé activée, définissez le paramètre `maxShares` sur une valeur supérieure à 1. Cela permet de partager le disque sur plusieurs machines virtuelles.

> [!IMPORTANT]
> La valeur de `maxShares` peut être définie ou modifiée uniquement lorsqu’un disque est démonté de toutes les machines virtuelles. Consultez les [tailles de disque](#disk-sizes) afin de connaître les valeurs autorisées pour `maxShares`.


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

##### <a name="regional-disk-example"></a>Exemple de disque régional

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

##### <a name="zonal-disk-example"></a>Exemple de disque zonale

Cet exemple est quasiment identique au précédent, sauf qu’il crée un disque dans la zone de disponibilité 1.

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1 --zone 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

##### <a name="regional-disk-example"></a>Exemple de disque régional

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

##### <a name="zonal-disk-example"></a>Exemple de disque zonale

Cet exemple est quasiment identique au précédent, sauf qu’il crée un disque dans la zone de disponibilité 1.

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5 -Zone 1

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

# <a name="resource-manager-template"></a>[Modèle Resource Manager](#tab/azure-resource-manager)

##### <a name="regional-disk-example"></a>Exemple de disque régional

Avant d’utiliser le modèle suivant, remplacez `[parameters('dataDiskName')]`, `[resourceGroup().location]`, `[parameters('dataDiskSizeGB')]`, `[parameters('maxShares')]`, `[parameters('diskIOPSReadWrite')]`, `[parameters('diskMBpsReadWrite')]`, `[parameters('diskIOPSReadOnly')]` et `[parameters('diskMBpsReadOnly')]` par vos propres valeurs.

[Modèle d’ultra-disques partagés régionaux](https://aka.ms/SharedUltraDiskARMtemplateRegional)

##### <a name="zonal-disk-example"></a>Exemple de disque zonale

Avant d’utiliser le modèle suivant, remplacez `[parameters('dataDiskName')]`, `[resourceGroup().location]`, `[parameters('dataDiskSizeGB')]`, `[parameters('maxShares')]`, `[parameters('diskIOPSReadWrite')]`, `[parameters('diskMBpsReadWrite')]`, `[parameters('diskIOPSReadOnly')]` et `[parameters('diskMBpsReadOnly')]` par vos propres valeurs.

[Modèle d’ultra-disques partagés zonaux](https://aka.ms/SharedUltraDiskARMtemplateZonal)

---

## <a name="using-azure-shared-disks-with-your-vms"></a>Utilisation de disques partagés Azure avec vos machines virtuelles

Une fois que vous avez déployé un disque partagé avec `maxShares>1`, vous pouvez monter le disque sur une ou plusieurs de vos machines virtuelles.

> [!NOTE]
> Si vous déployez un ultra-disque, assurez-vous qu’il correspond à la configuration requise. Pour plus d’informations, consultez [Utilisation de disques Ultra Azure](disks-enable-ultra-ssd.md).

```azurepowershell-interactive

$resourceGroup = "myResourceGroup"
$location = "WestCentralUS"

$vm = New-AzVm -ResourceGroupName $resourceGroup -Name "myVM" -Location $location -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress"

$dataDisk = Get-AzDisk -ResourceGroupName $resourceGroup -DiskName "mySharedDisk"

$vm = Add-AzVMDataDisk -VM $vm -Name "mySharedDisk" -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0

update-AzVm -VM $vm -ResourceGroupName $resourceGroup
```

## <a name="supported-scsi-pr-commands"></a>Commandes PR SCSI prises en charge

Une fois que vous avez monté le disque partagé sur vos machines virtuelles dans votre cluster, vous pouvez établir un quorum et lire/écrire sur le disque à l’aide de demandes de tirage (pull requests) SCSI. Les commandes PR suivantes sont disponibles lors de l’utilisation de disques partagés Azure :

Pour interagir avec le disque, démarrez avec la liste persistent-reservation-action :

```
PR_REGISTER_KEY 

PR_REGISTER_AND_IGNORE 

PR_GET_CONFIGURATION 

PR_RESERVE 

PR_PREEMPT_RESERVATION 

PR_CLEAR_RESERVATION 

PR_RELEASE_RESERVATION 
```

Lorsque vous utilisez PR_RESERVE, PR_PREEMPT_RESERVATION ou PR_RELEASE_RESERVATION, fournissez l’un des persistent-reservation-type suivants :

```
PR_NONE 

PR_WRITE_EXCLUSIVE 

PR_EXCLUSIVE_ACCESS 

PR_WRITE_EXCLUSIVE_REGISTRANTS_ONLY 

PR_EXCLUSIVE_ACCESS_REGISTRANTS_ONLY 

PR_WRITE_EXCLUSIVE_ALL_REGISTRANTS 

PR_EXCLUSIVE_ACCESS_ALL_REGISTRANTS 
```

Vous devez également fournir une clé persistent-reservation-key lorsque vous utilisez PR_RESERVE, PR_REGISTER_AND_IGNORE, PR_REGISTER_KEY, PR_PREEMPT_RESERVATION, PR_CLEAR_RESERVATION ou PR_RELEASE-RESERVATION.


## <a name="next-steps"></a>Étapes suivantes

Si vous préférez utiliser des modèles d’Azure Resource Manager pour déployer votre disque, les exemples de modèles suivants sont disponibles :
- [SSD Premium](https://aka.ms/SharedPremiumDiskARMtemplate)
- [Ultra-disques régionaux](https://aka.ms/SharedUltraDiskARMtemplateRegional)
- [Ultra-disques zonaux](https://aka.ms/SharedUltraDiskARMtemplateZonal)