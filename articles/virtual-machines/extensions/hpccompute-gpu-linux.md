---
title: Extension du pilote GPU NVIDIA - Machines virtuelles Linux Azure
description: Extension Microsoft Azure pour l’installation des pilotes GPU NVIDIA sur des machines virtuelles de calcul de gamme N sous Linux.
services: virtual-machines-linux
documentationcenter: ''
author: vermagit
manager: gwallace
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/11/2019
ms.author: akjosh
ms.openlocfilehash: c06d8e4ab368934182ed67b91dedb9ba9bcfc199
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965321"
---
# <a name="nvidia-gpu-driver-extension-for-linux"></a>Extension du pilote GPU NVIDIA pour Linux

## <a name="overview"></a>Vue d’ensemble

Cette extension installe des pilotes GPU NVIDIA sur des machines virtuelles Linux de gamme N. En fonction de la famille de machine virtuelle, l’extension installe des pilotes CUDA ou GRID. Lorsque vous installez des pilotes NVIDIA à l’aide de cette extension, vous acceptez les termes du [contrat de licence utilisateur final NVIDIA](https://go.microsoft.com/fwlink/?linkid=874330). Pendant le processus d’installation, la machine virtuelle peut redémarrer pour terminer l’installation du pilote.

Des instructions sur l’installation manuelle des pilotes et les versions actuellement prises en charge sont disponibles [ici](../linux/n-series-driver-setup.md).
Une extension est également disponible pour installer les pilotes GPU NVIDIA sur [des machines virtuelles de gamme N Windows](hpccompute-gpu-windows.md).

## <a name="prerequisites"></a>Prérequis

### <a name="operating-system"></a>Système d’exploitation

Cette extension prend en charge les distributions suivantes de système d’exploitation, en fonction de la prise en charge par le pilote de la version spécifique du système d’exploitation.

| Distribution | Version |
|---|---|
| Linux : Ubuntu | 16.04 LTS, 18.04 LTS |
| Linux : Red Hat Enterprise Linux | 7.3, 7.4, 7.5, 7.6, 7.7 |
| Linux : CentOS | 7.3, 7.4, 7.5, 7.6, 7.7 |

### <a name="internet-connectivity"></a>Connectivité Internet

L’extension Microsoft Azure pour les pilotes GPU NVIDIA nécessite que la machine virtuelle cible soit connectée à Internet et dispose d’un accès.

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
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.3",
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
| type | NvidiaGpuDriverLinux | string |
| typeHandlerVersion | 1.3 | int |

### <a name="settings"></a>Paramètres

Tous les paramètres sont facultatifs. Le comportement par défaut consiste à ne pas mettre à jour le noyau si cela n’est pas requis pour l’installation du pilote et à installer le dernier pilote pris en charge ainsi que le kit de ressources CUDA (le cas échéant).

| Nom | Description | Valeur par défaut | Valeurs valides | Type de données |
| ---- | ---- | ---- | ---- | ---- |
| updateOS | Mettre à jour le noyau, même si cela n’est pas requis pour l’installation du pilote | false | true, false | boolean |
| driverVersion | NV : version du pilote GRID<br> NC/ND : version du kit de ressources CUDA. Les derniers pilotes pour le kit de ressources CUDA choisi sont installés automatiquement. | latest | GRID: "430.30", "418.70", "410.92", "410.71", "390.75", "390.57", "390.42"<br> CUDA: "10.0.130", "9.2.88", "9.1.85" | string |
| installCUDA | Installer le kit de ressources CUDA. S’applique uniquement aux machines virtuelles de la série NC/ND. | true | true, false | boolean |


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
    "type": "NvidiaGpuDriverLinux",
    "typeHandlerVersion": "1.3",
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
    -ExtensionName "NvidiaGpuDriverLinux" `
    -ExtensionType "NvidiaGpuDriverLinux" `
    -TypeHandlerVersion 1.3 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

L’exemple suivant reflète les exemples Azure Resource Manager et PowerShell ci-dessus.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.3 
```

L’exemple suivant ajoute également deux paramètres personnalisés facultatifs afin d’illustrer l’installation personnalisée du pilote. Plus précisément, il met à jour le noyau du système d’exploitation vers la dernière version et installe un pilote de la version spécifique du kit de ressources CUDA. Là encore, notez que les paramètres « --settings » sont facultatifs et définis par défaut. Notez que la mise à jour du noyau peut accroître la durée d’installation de l’extension. Il se peut également que le choix d’une version spécifique (plus ancienne) du kit de ressources CUDA ne soit pas toujours compatible avec les nouveaux noyaux.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.3 \
  --settings '{ \
    "updateOS": true, \
    "driverVersion": "10.0.130" \
  }'
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

La sortie de l’exécution de l’extension est enregistrée dans le fichier suivant. Consultez ce fichier pour suivre l’état de toute installation (de longue durée), ainsi que pour résoudre les éventuels échecs.

```bash
/var/log/azure/nvidia-vmext-status
```

### <a name="exit-codes"></a>Codes de sortie

| Code de sortie | Signification | Action possible |
| :---: | --- | --- |
| 0 | L’opération a réussi |
| 1 | Utilisation incorrecte de l’extension | Vérifiez le journal de sortie de l’exécution |
| 10 | Les services d’intégration Linux pour Hyper-V et Azure ne sont pas disponibles ou installés | Vérifiez la sortie de lspci |
| 11 | GPU NVIDIA introuvable sur cette taille de machine virtuelle | Utilisez une [taille de machine virtuelle et un système d’exploitation pris en charge](../linux/n-series-driver-setup.md) |
| 12 | Offre d’image non prise en charge |
| 13 | Taille de machine virtuelle non prise en charge | Utilisez une machine virtuelle de série N pour déployer. |
| 14 | Échec de l’opération | Vérifiez le journal de sortie de l’exécution |


### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/community/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les extensions, consultez la page [Extensions et fonctionnalités de machine virtuelle pour Linux](features-linux.md).

Pour plus d’informations sur les machines virtuelles de gamme N, consultez la page [Tailles de machine virtuelle à GPU optimisé](../sizes-gpu.md).
