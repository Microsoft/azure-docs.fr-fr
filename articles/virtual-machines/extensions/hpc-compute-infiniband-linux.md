---
title: Extension du pilote InfiniBand – Machines virtuelles Linux Azure
description: Extension Microsoft Azure pour l’installation des pilotes InfiniBand sur des machines virtuelles de calcul de série N et H sous Linux.
services: virtual-machines-linux
documentationcenter: ''
author: vermagit
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/20/2020
ms.author: amverma
ms.openlocfilehash: 2a5aa8983e6cbb0745e05ce275edeadeccb60736
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966035"
---
# <a name="infiniband-driver-extension-for-linux"></a>Extension du pilote InfiniBand pour Linux

Cette extension permet d’installer les pilotes OFED InfiniBand sur des machines virtuelles de la [série H](../sizes-hpc.md) et de la [série N](../sizes-gpu.md) compatibles SR-IOV et InfiniBand [tailles « r »] sous Linux. En fonction de la famille de machines virtuelles, l’extension installe les pilotes appropriés pour la carte réseau Connect-X.

Des instructions sur l’installation manuelle des pilotes OFED sont disponibles [ici](../workloads/hpc/enable-infiniband.md#manual-installation).

Une extension est également disponible pour installer les pilotes InfiniBand pour les [machines virtuelles Windows](hpc-compute-infiniband-windows.md).

## <a name="prerequisites"></a>Prérequis

### <a name="operating-system"></a>Système d’exploitation

Cette extension prend en charge les distributions suivantes de système d’exploitation, en fonction de la prise en charge par le pilote de la version spécifique du système d’exploitation.

| Distribution | Version |
|---|---|
| Linux : Ubuntu | 16.04 LTS, 18.04 LTS, 20.04 LTS |
| Linux : CentOS | 7.4, 7.5, 7.6, 7.7, 8.1, 8.2 |
| Linux : Red Hat Enterprise Linux | 7.4, 7.5, 7.6, 7.7, 8.1, 8.2 |

### <a name="internet-connectivity"></a>Connectivité Internet

L’extension Microsoft Azure pour les pilotes InfiniBand nécessite que la machine virtuelle cible soit connectée à Internet et dispose d’un accès.

## <a name="extension-schema"></a>Schéma d’extensions

Le JSON suivant illustre le schéma pour l’extension.

```json
{
  "name": "<myExtensionName>",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "InfiniBandDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="properties"></a>Propriétés

| Nom | Valeur/Exemple | Type de données |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | Date |
| publisher | Microsoft.HpcCompute | string |
| type | InfiniBandDriverLinux | string |
| typeHandlerVersion | 1.1 | int |



## <a name="deployment"></a>Déploiement


### <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager 

Les extensions de machines virtuelles Azure peuvent être déployées avec des modèles Azure Resource Manager. Les modèles sont idéaux lorsque vous déployez une ou plusieurs machines virtuelles nécessitant une configuration post-déploiement.

La configuration JSON pour une extension de machine virtuelle peut être imbriquée à l’intérieur de la ressource de machine virtuelle ou placée à la racine ou au niveau supérieur d’un modèle de Resource Manager JSON. Le positionnement de la configuration JSON affecte la valeur du nom de la ressource et son type. Pour plus d’informations, consultez [Définition du nom et du type des ressources enfants](../../azure-resource-manager/templates/child-resource-name-type.md). 

L’exemple suivant suppose que l’extension est imbriquée dans la ressource de machine virtuelle. Lors de l’imbrication de la ressource d’extension, le JSON est placé dans l’objet `"resources": []` de la machine virtuelle.

```json
{
  "name": "myExtensionName",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', myVM)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "InfiniBandDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="powershell"></a>PowerShell

```powershell
Set-AzVMExtension
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Location "southcentralus" `
    -Publisher "Microsoft.HpcCompute" `
    -ExtensionName "InfiniBandDriverLinux" `
    -ExtensionType "InfiniBandDriverLinux" `
    -TypeHandlerVersion 1.1 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name InfiniBandDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.1 
```

### <a name="add-extension-to-a-virtual-machine-scale-set"></a>Ajouter une extension à un groupe de machines virtuelles identiques

L’exemple suivant installe la version 1.1 de l’extension InfiniBandDriverLinux sur toutes les machines virtuelles RDMA d’un groupe de machines virtuelles identiques existant nommé *myVMSS* et déployé dans le groupe de ressources nommé *myResourceGroup* :

  ```powershell
  $VMSS = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS"
  Add-AzVmssExtension -VirtualMachineScaleSet $VMSS -Name "InfiniBandDriverLinux" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverLinux" -TypeHandlerVersion "1.1"
  Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "MyVMSS" -VirtualMachineScaleSet $VMSS
  Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS" -InstanceId "*"
```


## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure, ainsi qu’à l’aide d’Azure PowerShell et de l’interface de ligne de commande Azure. Pour afficher l’état du déploiement des extensions pour une machine virtuelle donnée, exécutez la commande suivante.

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

La sortie de l’exécution de l’extension est enregistrée dans le fichier suivant. Consultez ce fichier pour suivre l’état de l’installation, ainsi que pour résoudre les éventuels échecs.

```bash
/var/log/azure/ib-vmext-status
```

### <a name="exit-codes"></a>Codes de sortie

Le tableau suivant explique la signification et décrit l’action recommandée en fonction des codes de sortie du processus d’installation de l’extension.

| Code de sortie | Signification | Action possible |
| :---: | --- | --- |
| 0 | L’opération a réussi |
| 1 | Utilisation incorrecte de l’extension | Vérifiez le journal de sortie de l’exécution |
| 10 | Les services d’intégration Linux pour Hyper-V et Azure ne sont pas disponibles ou installés | Vérifiez la sortie de lspci |
| 11 | Mellanox InfiniBand introuvable sur cette taille de machine virtuelle | Utilisez une [taille de machine virtuelle et un système d’exploitation pris en charge](../sizes-hpc.md) |
| 12 | Offre d’image non prise en charge |
| 13 | Taille de machine virtuelle non prise en charge | Utilisez une machine virtuelle de [série H](../sizes-hpc.md) et de [série N](../sizes-gpu.md) compatible InfiniBand (taille « r ») à déployer |
| 14 | Échec de l’opération | Vérifiez le journal de sortie de l’exécution |


### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/community/). Vous pouvez également déposer une demande d’assistance via le [site du support Azure](https://azure.microsoft.com/support/options/). Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les tailles compatibles avec InfiniBand (tailles « r »), consultez les machines virtuelles de [série H](../sizes-hpc.md) et de [série N](../sizes-gpu.md).

> [!div class="nextstepaction"]
> [En savoir plus sur les extensions et les fonctionnalités des machines virtuelles Linux](features-linux.md)