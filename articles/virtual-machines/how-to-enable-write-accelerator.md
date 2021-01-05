---
title: Accélérateur d’écriture Azure
description: Documentation sur la façon d’activer et d’utiliser un accélérateur des écritures
author: raiye
manager: markkie
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure
ms.date: 2/20/2019
ms.author: raiye
ms.subservice: disks
ms.openlocfilehash: e4f6cefd56c12162b370c78b6df2cd29ece030f1
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92515710"
---
# <a name="enable-write-accelerator"></a>Activer l’Accélérateur des écritures

L’Accélérateur des écritures est une fonctionnalité de disque des machines virtuelles Série M sur stockage Premium avec Azure Disques managés exclusivement. Comme son nom l’indique, cette fonctionnalité vise à améliorer la latence d’E/S des écritures dans le stockage Azure Premium. L’Accélérateur des écritures est particulièrement adapté lorsque les mises à jour du fichier journal doivent être conservées sur disque dans un système à hautes performances pour répondre aux besoins des bases de données modernes.

L’Accélérateur des écritures est généralement disponible pour les machines virtuelles Série M dans le cloud public.

## <a name="planning-for-using-write-accelerator"></a>Planification de l’utilisation de l’Accélérateur des écritures

Utilisez l’Accélérateur des écritures pour les volumes qui stockent le journal des transactions ou les journaux d’activité de restauration d’un SGBD (système de gestion de base de données). Il n’est pas recommandé d’utiliser l’Accélérateur des écritures pour les volumes de données d’une SGBD car la fonctionnalité a été optimisée pour être utilisée sur les disques journaux.

L’Accélérateur des écritures fonctionne uniquement en association avec des [disques managés Azure](https://azure.microsoft.com/services/managed-disks/).

> [!IMPORTANT]
> L’activation de l’Accélérateur des écritures sur le disque système de la machine virtuelle provoque le redémarrage de la machine virtuelle.
>
> Si vous souhaitez activer l’Accélérateur des écritures sur un disque Azure existant qui ne fait PAS partie d’un volume constitué de plusieurs disques à l’aide des gestionnaires de disques ou de volumes Windows, des espaces de stockage Windows, de la fonctionnalité SOFS Windows, de LVM ou MDADM pour Linux, vous devez arrêter la charge de travail qui accède au disque Azure. Les applications de base de données qui utilisent le disque Azure DOIVENT également être arrêtées.
>
> Si vous souhaitez activer ou désactiver l’Accélérateur des écritures pour un volume existant constitué de plusieurs disques de stockage Azure Premium et agrégé par bandes à l’aide des gestionnaires de disques ou de volumes Windows, des espaces de stockage Windows, de la fonctionnalité SOFS Windows, de LVM ou MDADM pour Linux, vous devez l’activer ou le désactiver sur tous les disques du volume en effectuant plusieurs étapes distinctes. **Avant d’activer ou de désactiver l’Accélérateur des écritures Azure dans cette configuration, arrêtez la machine virtuelle Azure** .

L’activation de l’Accélérateur des écritures sur les disques système n’est généralement pas nécessaire pour les configurations de machines virtuelles SAP.

### <a name="restrictions-when-using-write-accelerator"></a>Restrictions relatives à l’utilisation de l’Accélérateur des écritures

Quand vous utilisez l’Accélérateur des écritures sur un disque/disque dur virtuel Azure, les restrictions suivantes s’appliquent :

- La mise en cache du disque Premium doit être définie sur Aucune ou Lecture seule. Les autres modes de mise en cache ne sont pas pris en charge.
- Les instantanés ne sont pas pris en charge pour les disques où l’Accélérateur d’écriture est activé. Durant la sauvegarde, le service Sauvegarde Azure exclut automatiquement les disques avec activation de l’Accélérateur d’écriture, qui sont attachés à la machine virtuelle.
- Seules les tailles d’E/S plus petites (<=512 Kio) prennent le chemin accéléré. Dans les cas de charge de travail où les données sont chargées en bloc ou les tampons de journal des transactions des différents systèmes de gestion de base de données (SGBD) sont davantage remplis avant d’être conservés dans le stockage, les E/S écrites sur le disque risquent de ne pas prendre le chemin d’accès accéléré.

L’Accélérateur des écritures prend en charge un nombre limité de disques durs virtuels de stockage Azure Premium par machine virtuelle. Les limites actuelles sont :

| Référence de la machine virtuelle | Nombre de disques avec Accélérateur des écritures | IOPS de disque avec Accélérateur des écritures par machine virtuelle |
| --- | --- | --- |
| M416ms_v2, M416s_v2| 16 | 20000 |
| M208ms_v2, M208s_v2| 8 | 10000 |
| M128ms, M128s | 16 | 20000 |
| M64ms, M64ls, M64s | 8 | 10000 |
| M32ms, M32ls, M32ts, M32s | 4 | 5 000 |
| M16ms, M16s | 2 | 2 500 |
| M8ms, M8s | 1 | 1250 |

Les limites d’IOPS s’entendent par machine virtuelle et *non* par disque. Tous les disques avec Accélérateur des écritures partagent la même limite d’IOPS par machine virtuelle. Les disques attachés ne peuvent pas dépasser la limite d’IOPS d’accélérateur d’écriture pour une machine virtuelle. Par exemple, même si les disques attachés peuvent effectuer 30 000 IOPS, le système n’autorise pas les disques à aller au-delà de 20 000 IOPS pour M416ms_v2.

## <a name="enabling-write-accelerator-on-a-specific-disk"></a>Activation de l’Accélérateur des écritures sur un disque spécifique

Les sections ci-après décrivent comment activer l’Accélérateur des écritures sur des disques durs virtuels de stockage Azure Premium.

### <a name="prerequisites"></a>Prérequis

Assurez-vous de remplir les prérequis suivants pour pouvoir utiliser l’Accélérateur des écritures :

- Les disques sur lesquels vous voulez activer l’Accélérateur des écritures Azure doivent être des [disques managés Azure](https://azure.microsoft.com/services/managed-disks/) sur stockage Premium.
- Vous devez utiliser une machine virtuelle Série M

## <a name="enabling-azure-write-accelerator-using-azure-powershell"></a>Activation de l’Accélérateur des écritures Azure à l’aide d’Azure PowerShell

Dans les versions 5.5.0 et ultérieures du module Azure PowerShell, des modifications ont été apportées aux applets de commande concernées pour permettre l’activation ou la désactivation de l’Accélérateur des écritures sur des disques de stockage Azure Premium spécifiques.
Pour permettre l’activation ou le déploiement de disques pris en charge par l’Accélérateur des écritures, les commandes PowerShell suivantes ont été modifiées et étendues avec un paramètre pour l’accélérateur.

Le nouveau paramètre de commutateur **-WriteAccelerator** a été ajouté aux applets de commande suivantes :

- [Set-AzVMOsDisk](/powershell/module/az.compute/set-azvmosdisk?view=azurermps-6.0.0)
- [Add-AzVMDataDisk](/powershell/module/az.compute/Add-AzVMDataDisk?view=azurermps-6.0.0)
- [Set-AzVMDataDisk](/powershell/module/az.compute/Set-AzVMDataDisk?view=azurermps-6.0.0)
- [Add-AzVmssDataDisk](/powershell/module/az.compute/Add-AzVmssDataDisk?view=azurermps-6.0.0)

Si le paramètre n’est pas spécifié, la propriété est définie sur false, et les disques déployés ne sont pas pris en charge par l’Accélérateur des écritures.

Le nouveau paramètre de commutateur **-OsDiskWriteAccelerator** a été ajouté aux applets de commande suivantes :

- [Set-AzVmssStorageProfile](/powershell/module/az.compute/Set-AzVmssStorageProfile?view=azurermps-6.0.0)

Si le paramètre n’est pas spécifié, la propriété est définie sur false par défaut, et retournant les disques déployés n’utilisant pas l’Accélérateur des écritures.

Le nouveau paramètre booléen **-OsDiskWriteAccelerator** (paramètre facultatif et qui n’accepte pas les valeurs Null) a été ajouté aux applets de commande suivantes :

- [Update-AzVM](/powershell/module/az.compute/Update-AzVM?view=azurermps-6.0.0)
- [Update-AzVmss](/powershell/module/az.compute/Update-AzVmss?view=azurermps-6.0.0)

Spécifiez $true ou $false pour activer ou désactiver la prise en charge de l’Accélérateur des écritures Azure avec les disques.

Exemples de commandes :

```powershell
New-AzVMConfig | Set-AzVMOsDisk | Add-AzVMDataDisk -Name "datadisk1" | Add-AzVMDataDisk -Name "logdisk1" -WriteAccelerator | New-AzVM

Get-AzVM | Update-AzVM -OsDiskWriteAccelerator $true

New-AzVmssConfig | Set-AzVmssStorageProfile -OsDiskWriteAccelerator | Add-AzVmssDataDisk -Name "datadisk1" -WriteAccelerator:$false | Add-AzVmssDataDisk -Name "logdisk1" -WriteAccelerator | New-AzVmss

Get-AzVmss | Update-AzVmss -OsDiskWriteAccelerator:$false
```

Les scripts montrés dans les sections suivantes correspondent à deux scénarios courants.

### <a name="adding-a-new-disk-supported-by-write-accelerator-using-powershell"></a>Ajout d’un nouveau disque pris en charge par l’Accélérateur des écritures à l’aide de PowerShell

Vous pouvez utiliser ce script pour ajouter un nouveau disque à votre machine virtuelle. Le disque créé avec ce script utilise l’Accélérateur des écritures.

Remplacez `myVM`, `myWAVMs`, `log001`, la taille du disque et le LunID du disque par les valeurs appropriées pour votre déploiement spécifique.

```powershell
# Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "log001"
#LUN Id
$lunid=8
#size
$size=1023
#Pulls the VM info for later
$vm=Get-AzVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Add-AzVMDataDisk -CreateOption empty -DiskSizeInGB $size -Name $vmname-$datadiskname -VM $vm -Caching None -WriteAccelerator:$true -lun $lunid
#Updates the VM with the disk config - does not require a reboot
Update-AzVM -ResourceGroupName $rgname -VM $vm
```

### <a name="enabling-write-accelerator-on-an-existing-azure-disk-using-powershell"></a>Activation de l’Accélérateur des écritures sur un disque Azure existant à l’aide de PowerShell

Vous pouvez utiliser ce script pour activer l’Accélérateur des écritures sur un disque existant. Remplacez `myVM`, `myWAVMs`, et `test-log001` par les valeurs appropriées pour votre déploiement spécifique. Le script ajoute l’Accélérateur des écritures à un disque existant si le paramètre **$newstatus** est défini sur « $true ». Si ce paramètre est défini sur $false, l’Accélérateur des écritures est désactivé sur le disque.

```powershell
#Specify your VM Name
$vmName="myVM"
#Specify your Resource Group
$rgName = "myWAVMs"
#data disk name
$datadiskname = "test-log001" 
#new Write Accelerator status ($true for enabled, $false for disabled) 
$newstatus = $true
#Pulls the VM info for later
$vm=Get-AzVM -ResourceGroupName $rgname -Name $vmname
#add a new VM data disk
Set-AzVMDataDisk -VM $vm -Name $datadiskname -Caching None -WriteAccelerator:$newstatus
#Updates the VM with the disk config - does not require a reboot
Update-AzVM -ResourceGroupName $rgname -VM $vm
```

> [!Note]
> L’exécution du script ci-dessus détache le disque spécifié, active l’Accélérateur des écritures sur le disque, puis réattache le disque

## <a name="enabling-write-accelerator-using-the-azure-portal"></a>Activation de l’Accélérateur des écritures à l’aide du Portail Azure

Vous pouvez activer l’Accélérateur des écritures sur le portail, en spécifiant vos paramètres de mise en cache du disque :

![Accélérateur des écritures sur le Portail Azure](./media/virtual-machines-common-how-to-enable-write-accelerator/wa_scrnsht.png)

## <a name="enabling-write-accelerator-using-the-azure-cli"></a>Activation de l’Accélérateur des écritures à l’aide de Azure CLI

Vous pouvez utiliser [Azure CLI](/cli/azure/?view=azure-cli-latest) pour activer l’accélérateur d’écriture.

Pour activer l’Accélérateur des écritures sur un disque existant, utilisez la commande [az vm update](/cli/azure/vm?view=azure-cli-latest#az-vm-update). Vous pouvez utiliser les exemples suivants si vous remplacez diskName, VMName et ResourceGroup par vos propres valeurs : `az vm update -g group1 -n vm1 -write-accelerator 1=true`

Pour attacher un disque avec l’Accélérateur des écritures activé, utilisez la commande [az vm disk attach](/cli/azure/vm/disk?view=azure-cli-latest#az-vm-disk-attach). Vous pouvez utiliser l’exemple suivant si vous indiquez vos propres valeurs : `az vm disk attach -g group1 -vm-name vm1 -disk d1 --enable-write-accelerator`

Pour désactiver l’Accélérateur des écritures, utilisez la commande [az vm update](/cli/azure/vm?view=azure-cli-latest#az-vm-update) et définissez les propriétés sur false : `az vm update -g group1 -n vm1 -write-accelerator 0=false 1=false`

## <a name="enabling-write-accelerator-using-rest-apis"></a>Activation de l’Accélérateur des écritures à l’aide des API Rest

Pour effectuer le déploiement à l’aide de l’API Rest Azure, vous devez installer Azure armclient.

### <a name="install-armclient"></a>Installer armclient

Pour pouvoir utiliser armclient, installez-le via Chocolatey, à l’aide de cmd.exe ou PowerShell. Exécutez ces commandes avec des droits élevés (« Exécuter en tant qu’administrateur »).

Avec cmd.exe, exécutez la commande suivante : `@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"`

Avec Power Shell, exécutez la commande suivante : `Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))`

Vous pouvez maintenant installer armclient à l’aide de la commande suivante dans cmd.exe ou PowerShell `choco install armclient`

### <a name="getting-your-current-vm-configuration"></a>Obtenir la configuration actuelle de votre machine virtuelle

Pour changer les attributs de configuration du disque, vous devez d’abord obtenir la configuration actuelle et l’enregistrer dans un fichier JSON. Pour cela, exécutez la commande suivante : `armclient GET /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 > <<filename.json>>`

Remplacez le contenu entre les balises <<   >> par vos propres données, dont le nom souhaité pour le fichier JSON.

Le résultat doit être semblable à ceci :

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"

```

Ensuite, mettez à jour le fichier JSON et à activer l’Accélérateur des écritures sur le disque appelé « log1 ». Ce peut être effectué en ajoutant cet attribut après l’entrée du cache du disque dans le fichier JSON.

```JSON
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
```

Ensuite, mettez à jour le déploiement existant avec cette commande : `armclient PUT /subscriptions/<<subscription-ID<</resourceGroups/<<ResourceGroup>>/providers/Microsoft.Compute/virtualMachines/<<virtualmachinename>>?api-version=2017-12-01 @<<filename.json>>`

Le résultat doit ressembler à l’exemple ci-dessous. Vous pouvez voir que l’Accélérateur des écritures est activé pour un seul disque.

```JSON
{
  "properties": {
    "vmId": "2444c93e-f8bb-4a20-af2d-1658d9dbbbcb",
    "hardwareProfile": {
      "vmSize": "Standard_M64s"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "SUSE",
        "offer": "SLES-SAP",
        "sku": "12-SP3",
        "version": "latest"
      },
      "osDisk": {
        "osType": "Linux",
        "name": "mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a",
        "createOption": "FromImage",
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS",
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/mylittlesap_OsDisk_1_754a1b8bb390468e9b4c429b81cc5f5a"
        },
        "diskSizeGB": 30
      },
      "dataDisks": [
        {
          "lun": 0,
          "name": "data1",
          "createOption": "Attach",
          "caching": "None",
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data1"
          },
          "diskSizeGB": 1023
        },
        {
          "lun": 1,
          "name": "log1",
          "createOption": "Attach",
          "caching": "None",
          "writeAcceleratorEnabled": true,
          "managedDisk": {
            "storageAccountType": "Premium_LRS",
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/disks/data2"
          },
          "diskSizeGB": 1023
        }
      ]
    },
    "osProfile": {
      "computerName": "mylittlesapVM",
      "adminUsername": "pl",
      "linuxConfiguration": {
        "disablePasswordAuthentication": false
      },
      "secrets": []
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Network/networkInterfaces/mylittlesap518"
        }
      ]
    },
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "https://mylittlesapdiag895.blob.core.windows.net/"
      }
    },
    "provisioningState": "Succeeded"
  },
  "type": "Microsoft.Compute/virtualMachines",
  "location": "westeurope",
  "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/mylittlesap/providers/Microsoft.Compute/virtualMachines/mylittlesapVM",
  "name": "mylittlesapVM"
```

Une fois ce changement effectué, le disque est normalement pris en charge par l’Accélérateur des écritures.
