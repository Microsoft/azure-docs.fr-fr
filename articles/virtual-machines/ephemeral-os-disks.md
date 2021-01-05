---
title: Disques de système d’exploitation éphémères
description: En savoir plus sur les disques de système d’exploitation éphémères pour les machines virtuelles Azure.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 07/23/2020
ms.author: cynthn
ms.subservice: disks
ms.openlocfilehash: 52071b964412071d820745b173e8835c6f9e7d0e
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97510989"
---
# <a name="ephemeral-os-disks-for-azure-vms"></a>Disques de système d’exploitation éphémères pour les machines virtuelles Azure

Les disques de système d’exploitation éphémères sont créés sur le stockage local de la machine virtuelle (VM) et ne sont pas enregistrés dans le Stockage Azure à distance. Les disques de système d’exploitation éphémères conviennent particulièrement bien aux charges de travail sans état, car les applications tolèrent les défaillances individuelles des machines virtuelles, mais sont plus sensibles au temps de déploiement de machine virtuelle et de réinitialisation des instances de machines virtuelles individuelles. Comparé à un disque de système d’exploitation standard, un disque éphémère offre une latence plus faible pour les opérations de lecture/écriture et permet une réinitialisation plus rapide des machines virtuelles. 
 
Les principales caractéristiques des disques éphémères sont les suivantes : 
- Idéal pour les applications sans état.
- Ils peuvent être utilisés avec la Place de marché et des images personnalisées.
- Capacité de réinitialisation rapide de leurs machines virtuelles et instances de groupe identique à l’état de démarrage d’origine.  
- Faible latence, comparable à celle d’un disque temporaire. 
- Le disque de système d’exploitation éphémère est gratuit ; autrement dit, aucun coût de stockage ne s’applique à l’utilisation de celui-ci.
- Ils sont disponibles dans toutes les régions Azure. 
- Le disque de système d’exploitation éphémère est pris en charge par la [Galerie d’images partagées](./linux/shared-image-galleries.md). 
 

 
Différences clés entre les disques de système d’exploitation persistants et éphémères :

|                             | Disque de système d’exploitation persistant                          | Disque de système d’exploitation éphémère                              |
|-----------------------------|---------------------------------------------|------------------------------------------------|
| **Limite de taille du disque de système d’exploitation**      | 2 Tio                                                                                        | Taille du cache pour la taille de la machine virtuelle ou 2 Tio, la valeur la plus faible étant retenue. Pour la **taille du cache en Gio**, consultez [DS](sizes-general.md), [ES](sizes-memory.md), [M](sizes-memory.md), [FS](sizes-compute.md), and [GS](sizes-previous-gen.md#gs-series)              |
| **Tailles de machines virtuelles prises en charge**          | Tous                                                                                          | Tailles de machine virtuelle prenant en charge le stockage Premium, par exemple, DSv1, DSv2, DSv3, Esv3, Fs, FsV2, GS ou M                                               |
| **Prise en charge du type de disque**           | Disque de système d’exploitation managé et non managé                                                                | Disque de système d’exploitation managé uniquement                                                               |
| **Prise en charge des régions**              | Toutes les régions                                                                                  | Toutes les régions                              |
| **Persistance des données**            | Les données écrites sur le disque de système d’exploitation sont stockées dans le Stockage Azure                                  | Les données écrites sur le disque du système d’exploitation sont stockées sur le stockage local de la machine virtuelle et ne sont pas persistantes dans le Stockage Azure. |
| **État arrêté/libéré**      | Les machines virtuelles et instances de groupe identique peuvent être arrêtées-libérées et redémarrées à partir de l’état arrêté-libéré | Les machines virtuelles et instances de groupe identique ne peuvent pas être arrêtées-libérées                                  |
| **Prise en charge des disques de système d’exploitation spécialisés** | Oui                                                                                          | Non                                                                                 |
| **Redimensionnement du disque de système d’exploitation**              | Pris en charge durant la création de la machine virtuelle et une fois que la machine virtuelle est arrêtée-libérée                                | Prise en charge lors de la création d’une machine virtuelle uniquement                                                  |
| **Redimensionnement des machines virtuelles**   | Les données du disque de système d’exploitation sont conservées                                                                    | Les données sur le disque du système d’exploitation sont supprimées, le système d’exploitation est redéployé       
| **Emplacement du fichier d’échange**   | Pour Windows, le fichier d’échange est stocké sur le disque de ressources                                              | Pour Windows, le fichier d’échange est stocké sur le disque du système d’exploitation   |

## <a name="size-requirements"></a>Exigences de taille

Vous pouvez déployer des machines virtuelles et images d’instance jusqu’à la taille du cache des machines virtuelles. Par exemple, les images Windows Server Standard de la place de marché sont d’environ 127 Go, ce qui signifie que vous avez besoin d’une taille de machine virtuelle qui a un cache supérieur à 127 Go. Dans ce cas, la référence [Standard_DS2_v2](dv2-dsv2-series.md) a une taille de cache de 86 Gio, ce qui n’est pas suffisant. La référence Standard_DS3_v2 a une taille de cache de 172 Gio, ce qui est suffisant. Dans ce cas, la référence Standard_DS3_v2 est la taille minimale de la série DSv2 que vous pouvez utiliser avec cette image. Les images Linux de base dans les images de la place de marché et Windows Server qui sont signalées par `[smallsize]` ont tendance à être d’environ 30 Gio et peuvent utiliser la plupart des tailles de machine virtuelle disponibles.

Les disques éphémères nécessitent également que la taille de machine virtuelle prenne en charge le stockage Premium. Les tailles ont généralement (mais pas toujours) un `s` dans le nom, comme DSv2 et EsV3. Pour plus d’informations, consultez [Tailles de machine virtuelle Azure](sizes.md) pour plus d’informations sur les tailles prenant en charge le stockage Premium.

## <a name="preview---ephemeral-os-disks-can-now-be-stored-on-temp-disks"></a>Préversion - Les disques de système d’exploitation éphémères peuvent désormais être stockés sur des disques temporaires
Les disques de système d’exploitation éphémères peuvent désormais être stockés sur le disque temporaire/ressources de la machine virtuelle en plus du cache de la machine virtuelle. À présent, vous pouvez donc utiliser des disques de système d’exploitation éphémères avec des machines virtuelles qui n’ont pas de cache ou qui ne disposent pas d’un cache suffisant, mais qui disposent d’un disque temporaire/ressources pour stocker le disque de système d’exploitation éphémère, tel que Dav3, Dav4, Eav4 et Eav3. Si une machine virtuelle dispose d’un cache et d’un espace temporaire suffisants, vous pouvez également spécifier où vous voulez stocker le disque de système d’exploitation éphémère à l’aide d’une nouvelle propriété appelée [DiffDiskPlacement](/rest/api/compute/virtualmachines/list#diffdiskplacement). Avec cette fonctionnalité, lorsqu’une machine virtuelle Windows est provisionnée, nous configurons le fichier d’échange pour qu’il se trouve sur le disque du système d’exploitation. Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Pour commencer, [demandez l’accès](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR6cQw0fZJzdIsnbfbI13601URTBCRUZPMkQwWFlCOTRIMFBSNkM1NVpQQS4u).

## <a name="powershell"></a>PowerShell

Pour utiliser un disque éphémère pour un déploiement de machine virtuelle avec PowerShell, utilisez [Set-AzVMOSDisk](/powershell/module/az.compute/set-azvmosdisk) dans votre configuration de machine virtuelle. Définissez `-DiffDiskSetting` sur `Local` et `-Caching` sur `ReadOnly`.     

```powershell
Set-AzVMOSDisk -DiffDiskSetting Local -Caching ReadOnly
```

Pour les déploiements de groupes identiques, utilisez l’applet de commande [Set-AzVmssStorageProfile](/powershell/module/az.compute/set-azvmssstorageprofile) dans votre configuration. Définissez `-DiffDiskSetting` sur `Local` et `-Caching` sur `ReadOnly`.


```powershell
Set-AzVmssStorageProfile -DiffDiskSetting Local -OsDiskCaching ReadOnly
```

## <a name="cli"></a>Interface de ligne de commande

Pour utiliser un disque éphémère pour un déploiement de machine virtuelle par CLI, définissez le paramètre `--ephemeral-os-disk` dans [az vm create](/cli/azure/vm#az-vm-create) sur `true` et le paramètre `--os-disk-caching` sur `ReadOnly`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --ephemeral-os-disk true \
  --os-disk-caching ReadOnly \
  --admin-username azureuser \
  --generate-ssh-keys
```

Pour les groupes identiques, vous utilisez le même paramètre `--ephemeral-os-disk true` pour [az-vmss-create](/cli/azure/vmss#az-vmss-create) et définissez le paramètre `--os-disk-caching` sur `ReadOnly`.

## <a name="portal"></a>Portail

Dans le portail Azure, vous pouvez choisir d’utiliser des disques éphémères lors du déploiement d’une machine virtuelle en ouvrant la section **Avancé** de l’onglet **Disques**. Pour **Utiliser un disque de système d’exploitation éphémère** sélectionnez **Oui**.

![Capture d’écran montrant le bouton radio pour choisir d’utiliser un disque de système d’exploitation éphémère](./media/virtual-machines-common-ephemeral/ephemeral-portal.png)

Si l’option pour l’utilisation d’un disque éphémère est grisée, vous avez peut-être sélectionné une taille de machine virtuelle qui n’a pas une taille de cache supérieure à l’image du système d’exploitation ou qui ne prend pas en charge le stockage Premium. Revenez à la page **Notions de base** et essayez de choisir une autre taille de machine virtuelle.

Vous pouvez également créer des groupes identiques avec des disques de système d’exploitation éphémères en utilisant le portail. Veillez simplement à sélectionner une taille de machine virtuelle avec une taille de cache suffisamment grande, puis dans **Utiliser un disque du système d’exploitation éphémère**, sélectionnez **Oui**.

![Capture d’écran montrant le bouton radio pour choisir d’utiliser un disque de système d’exploitation éphémère pour votre groupe identique](./media/virtual-machines-common-ephemeral/scale-set.png)

## <a name="scale-set-template-deployment"></a>Déploiement de modèle de groupe identique  
Le processus de création d’un groupe identique utilisant un disque de système d’exploitation éphémère consiste à ajouter la propriété `diffDiskSettings` au type de ressource `Microsoft.Compute/virtualMachineScaleSets/virtualMachineProfile` dans le modèle. En outre, la stratégie de mise en cache doit être définie sur `ReadOnly` pour le disque de système d’exploitation éphémère. 


```json
{ 
  "type": "Microsoft.Compute/virtualMachineScaleSets", 
  "name": "myScaleSet", 
  "location": "East US 2", 
  "apiVersion": "2018-06-01", 
  "sku": { 
    "name": "Standard_DS2_v2", 
    "capacity": "2" 
  }, 
  "properties": { 
    "upgradePolicy": { 
      "mode": "Automatic" 
    }, 
    "virtualMachineProfile": { 
       "storageProfile": { 
        "osDisk": { 
          "diffDiskSettings": { 
            "option": "Local" 
          }, 
          "caching": "ReadOnly", 
          "createOption": "FromImage" 
        }, 
        "imageReference":  { 
          "publisher": "Canonical", 
          "offer": "UbuntuServer", 
          "sku": "16.04-LTS", 
          "version": "latest" 
        } 
      }, 
      "osProfile": { 
        "computerNamePrefix": "myvmss", 
        "adminUsername": "azureuser", 
        "adminPassword": "P@ssw0rd!" 
      } 
    } 
  } 
}  
```

## <a name="vm-template-deployment"></a>Déploiement de modèle de machine virtuelle 
Vous pouvez déployer une machine virtuelle avec un disque de système d’exploitation éphémère à l’aide d’un modèle. Le processus de création d’une machine virtuelle utilisant des disques de système d’exploitation éphémères consiste à ajouter la propriété `diffDiskSettings` au type de ressource Microsoft.Compute/virtualMachines dans le modèle. En outre, la stratégie de mise en cache doit être définie sur `ReadOnly` pour le disque de système d’exploitation éphémère. 

```json
{ 
  "type": "Microsoft.Compute/virtualMachines", 
  "name": "myVirtualMachine", 
  "location": "East US 2", 
  "apiVersion": "2018-06-01", 
  "properties": { 
       "storageProfile": { 
            "osDisk": { 
              "diffDiskSettings": { 
                "option": "Local" 
              }, 
              "caching": "ReadOnly", 
              "createOption": "FromImage" 
            }, 
            "imageReference": { 
                "publisher": "MicrosoftWindowsServer", 
                "offer": "WindowsServer", 
                "sku": "2016-Datacenter-smalldisk", 
                "version": "latest" 
            }, 
            "hardwareProfile": { 
                 "vmSize": "Standard_DS2_v2" 
             } 
      }, 
      "osProfile": { 
        "computerNamePrefix": "myvirtualmachine", 
        "adminUsername": "azureuser", 
        "adminPassword": "P@ssw0rd!" 
      } 
    } 
 } 
```


## <a name="reimage-a-vm-using-rest"></a>Réinitialiser une machine virtuelle à l’aide de REST
Vous pouvez réinitialiser une instance de machine virtuelle avec un disque de système d’exploitation éphémère à l’aide d’une API REST (comme décrit ci-dessous) et à partir du portail Azure en accédant au volet Vue d’ensemble de la machine virtuelle. Pour les groupes identiques, cette opération est déjà disponible via Powershell, l’interface CLI et le Portail.

```
POST https://management.azure.com/subscriptions/{sub-
id}/resourceGroups/{rgName}/providers/Microsoft.Compute/VirtualMachines/{vmName}/reimage?a pi-version=2018-06-01" 
```
 
## <a name="frequently-asked-questions"></a>Forum aux questions

**Q : Quelle est la taille des disques de système d’exploitation locaux ?**

A : Nous prenons en charge des plateformes et/ou images personnalisées atteignant la taille du cache de la machine virtuelle, dans lequel toutes les écritures/lecture sur le disque de système d’exploitation seront effectuées à un niveau local sur le même nœud que la machine virtuelle. 

**Q : Le disque de système d’exploitation éphémère peut-il être redimensionné ?**

A : Non, une fois le disque de système d’exploitation éphémère configuré, il ne peut plus être redimensionné. 

**Q : Puis-je attacher un disque managé à une machine virtuelle éphémère ?**

A : Oui, vous pouvez attacher un disque de données managé à une machine virtuelle qui utilise un disque de système d’exploitation éphémère. 

**Q : Toutes les tailles de machine virtuelle sont-elles prises en charge par les disques de système d’exploitation éphémères ?**

A : Non, la plupart des tailles de machine virtuelle Stockage Premium sont prises en charge (DS, ES, FS, GS, M, etc.). Pour savoir si une taille de machine virtuelle particulière prend en charge les disques de système d’exploitation éphémères, vous pouvez :

Appeler l’applet de commande PowerShell `Get-AzComputeResourceSku`
```azurepowershell-interactive
 
$vmSizes=Get-AzComputeResourceSku | where{$_.ResourceType -eq 'virtualMachines' -and $_.Locations.Contains('CentralUSEUAP')} 

foreach($vmSize in $vmSizes)
{
   foreach($capability in $vmSize.capabilities)
   {
       if($capability.Name -eq 'EphemeralOSDiskSupported' -and $capability.Value -eq 'true')
       {
           $vmSize
       }
   }
}
```
 
**Q : Le disque de système d’exploitation éphémère peut-il être appliqué aux machines virtuelles et groupes identiques existants ?**

A : Non, le disque de système d’exploitation éphémère peut uniquement être utilisé au cours de la création de machines virtuelles et de groupes identiques. 

**Q : Peut-on combiner des disques de système d’exploitation normaux et éphémères dans un groupe identique ?**

A : Non, vous ne pouvez pas combiner des instances de disques de système d’exploitation éphémères et persistants dans le même groupe identique. 

**Q : Le disque de système d’exploitation éphémère peut-il être créé à l’aide de Powershell ou de l’interface CLI ?**

A : Oui, vous pouvez créer des machines virtuelles avec un disque de système d’exploitation éphémère à l’aide de REST, des modèles, de PowerShell et de l’interface CLI.

**Q : Quelles fonctionnalités ne sont pas prises en charge avec le disque de système d’exploitation éphémère ?**

A : Les disques éphémères ne prennent pas en charge :
- Capture des images de machine virtuelle
- Captures instantanées de disque 
- Azure Disk Encryption 
- Sauvegarde Azure
- Azure Site Recovery  
- Échange du disque de système d’exploitation 
 
## <a name="next-steps"></a>Étapes suivantes
Vous pouvez créer une machine virtuelle avec un disque de système d’exploitation éphémère à l’aide [d’Azure CLI](/cli/azure/vm#az-vm-create).
