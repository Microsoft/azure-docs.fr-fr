---
title: 'Didacticiel : gérer les disques Azure avec Azure PowerShell'
description: Dans ce didacticiel, vous allez apprendre à utiliser Azure PowerShell afin de créer et gérer des disques Azure pour des machines virtuelles
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: disks
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/29/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 528fe5dea533faf9447e03dd901568d783891ce9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88718932"
---
# <a name="tutorial---manage-azure-disks-with-azure-powershell"></a>Didacticiel : gérer les disques Azure avec Azure PowerShell

Les machines virtuelles utilisent des disques pour stocker leur système d’exploitation, leurs applications et leurs données. Lorsque vous créez une machine virtuelle, il est important de choisir une taille de disque et une configuration appropriées à la charge de travail prévue. Ce didacticiel décrit le déploiement et la gestion des disques de machine virtuelle. Vous en apprendrez davantage sur les points suivants :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attachement et préparation des disques de données

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="default-azure-disks"></a>Disques Azure par défaut

Lorsqu’une machine virtuelle Azure est créée, deux disques sont automatiquement attachés à celle-ci. 

**Disque de système d’exploitation** : la taille des disques de système d’exploitation peut atteindre 4 To ; ces disques hébergent le système d’exploitation des machines virtuelles. Si vous créez une machine virtuelle à partir d’une image de la [Place de marché Azure](https://azure.microsoft.com/marketplace/), sa taille est généralement de 127 Go (mais certaines images ont des tailles de disque de système d’exploitation plus petites). Le disque de système d’exploitation se voit attribuer la lettre de lecteur *C:* par défaut. La configuration de la mise en cache de disque de système d’exploitation est optimisée pour les performances du système d’exploitation. Le disque de système d’exploitation **ne doit pas** héberger d’applications ou de données. Pour héberger ce type de contenu, utilisez plutôt un disque de données, qui est décrit plus loin dans cet article.

**Disque temporaire** : les disques temporaires utilisent un disque SSD qui se trouve sur le même hôte Azure que la machine virtuelle. Les disques temporaires sont extrêmement performants et peuvent être utilisés pour des opérations telles que le traitement de données temporaires. Toutefois, si la machine virtuelle est déplacée vers un nouvel hôte, toutes les données stockées sur un disque temporaire sont supprimées. La taille du disque temporaire est déterminée par la [taille de la machine virtuelle](../sizes.md). Les disques temporaires se voient attribuer la lettre de lecteur *D:* par défaut.

## <a name="azure-data-disks"></a>Disques de données Azure

Des disques de données supplémentaires peuvent être ajoutés pour installer des applications et stocker des données. Les disques de données doivent être utilisés dans les cas où un stockage des données durable et réactif est nécessaire. La taille de la machine virtuelle détermine le nombre de disques de données pouvant être attachés à cette machine virtuelle.

## <a name="vm-disk-types"></a>Type de disque de machine virtuelle

Azure propose deux types de disque.

**Disques Standard** : ils s’appuient sur des disques durs et offrent un stockage économique qui n’en est pas moins performant. Les disques Standard constituent la solution idéale pour une charge de travail de développement et de test économique.

**Disques Premium** : ils reposent sur un disque SSD à faible latence et hautes performances. Ils conviennent parfaitement aux machines virtuelles exécutant une charge de travail en production. Les machines virtuelles dont le [nom de la taille](../vm-naming-conventions.md) contient un **S** prennent généralement en charge le stockage Premium. Par exemple, les machines virtuelles Azure des séries DS, DSv2, GS et FS prennent en charge le stockage Premium. Lorsque vous sélectionnez une taille de disque, la valeur est arrondie au type suivant. Par exemple, si la taille du disque est supérieure à 64 Go, mais inférieure à 128 Go, le type de disque est P10. 
<br>
[!INCLUDE [disk-storage-premium-ssd-sizes](../../../includes/disk-storage-premium-ssd-sizes.md)]

Lorsque vous configurez un disque de stockage Premium, contrairement au stockage standard, la capacité, les E/S par seconde et le débit de ce disque sont assurés. Par exemple, si vous créez un disque P50, Azure configure une capacité de stockage de 4 095 Go, 7 500 E/S par seconde et un débit de 250 Mo/s pour ce disque. Votre application peut utiliser tout ou partie de la capacité et des performances. Les disques SSD Premium sont conçus pour fournir des latences faibles de quelques millisecondes ainsi que l’IOPS et le débit cibles décrits dans le précédent tableau 99,9 % du temps.

Bien que le tableau ci-dessus identifie le nombre max. d’E/S par seconde par disque, un niveau de performances plus élevé est possible en entrelaçant plusieurs disques de données. Par exemple, 64 disques de données peuvent être attachés à la machine virtuelle Standard_GS5. Si chacun de ces disques est de type P30, vous pouvez atteindre un nombre maximum d’E/S par seconde de 80 000. Pour plus d’informations sur le nombre maximal d’E/S par seconde par machine virtuelle, consultez [Types et tailles des machines virtuelles](../sizes.md).

## <a name="create-and-attach-disks"></a>Créer et attacher des disques

Pour exécuter l’exemple dans ce didacticiel, vous devez disposer d’une machine virtuelle. Si nécessaire, créez une machine virtuelle à l’aide des commandes ci-dessous.

Définissez le nom d’utilisateur et le mot de passe pour le compte d’administrateur sur la machine virtuelle avec [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1) :


Créez la machine virtuelle avec [New-AzVM](/powershell/module/az.compute/new-azvm). Vous êtes invité à entrer un nom d’utilisateur et un mot de passe pour le compte administrateur de la machine virtuelle.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupDisk" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" 
```


Créez la configuration initiale avec [New-ADiskConfig](/powershell/module/az.compute/new-azdiskconfig). L’exemple suivant configure un disque d’une taille de 128 Go.

```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
    -Location "EastUS" `
    -CreateOption Empty `
    -DiskSizeGB 128
```

Créez le disque de données avec la commande [New-AzDisk](/powershell/module/az.compute/new-azdisk).

```azurepowershell-interactive
$dataDisk = New-AzDisk `
    -ResourceGroupName "myResourceGroupDisk" `
    -DiskName "myDataDisk" `
    -Disk $diskConfig
```

Obtenez la machine virtuelle que vous souhaitez ajouter au disque de données avec la commande [Get-AzVM](/powershell/module/az.compute/get-azvm).

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName "myResourceGroupDisk" -Name "myVM"
```

Ajoutez le disque de données à la configuration de la machine virtuelle avec la commande [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk).

```azurepowershell-interactive
$vm = Add-AzVMDataDisk `
    -VM $vm `
    -Name "myDataDisk" `
    -CreateOption Attach `
    -ManagedDiskId $dataDisk.Id `
    -Lun 1
```

Mettez à jour la machine virtuelle avec la commande [Update-AzVM](/powershell/module/az.compute/add-azvmdatadisk).

```azurepowershell-interactive
Update-AzVM -ResourceGroupName "myResourceGroupDisk" -VM $vm
```

## <a name="prepare-data-disks"></a>Préparer les disques de données

Une fois qu’un disque a été attaché à la machine virtuelle, le système d’exploitation doit être configuré pour utiliser le disque. L’exemple suivant montre comment configurer manuellement le premier disque ajouté à la machine virtuelle. Ce processus peut également être automatisé à l’aide de [l’extension de script personnalisé](./tutorial-automate-vm-deployment.md).

### <a name="manual-configuration"></a>Configuration manuelle

Créez une connexion RDP avec la machine virtuelle. Ouvrez PowerShell et exécutez ce script.

```azurepowershell
Get-Disk | Where partitionstyle -eq 'raw' |
    Initialize-Disk -PartitionStyle MBR -PassThru |
    New-Partition -AssignDriveLetter -UseMaximumSize |
    Format-Volume -FileSystem NTFS -NewFileSystemLabel "myDataDisk" -Confirm:$false
```

## <a name="verify-the-data-disk"></a>Vérifier le disque de données

Pour vérifier que le disque de données est attaché, affichez `StorageProfile` pour le `DataDisks` attaché.

```azurepowershell-interactive
$vm.StorageProfile.DataDisks
```

Le résultat suivant doit ressembler à ce qui suit :

```
Name            : myDataDisk
DiskSizeGB      : 128
Lun             : 1
Caching         : None
CreateOption    : Attach
SourceImage     :
VirtualHardDisk :
```


## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous a apporté des connaissances concernant les disques de machine virtuelle, notamment concernant les points suivants :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attachement et préparation des disques de données

Passez au didacticiel suivant pour en apprendre davantage sur l’automatisation de la configuration de machine virtuelle.

> [!div class="nextstepaction"]
> [Automatiser la configuration de machine virtuelle](./tutorial-automate-vm-deployment.md)
