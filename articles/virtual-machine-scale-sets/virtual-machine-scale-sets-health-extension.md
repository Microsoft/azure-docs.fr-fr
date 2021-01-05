---
title: Utiliser l’extension Intégrité de l’application avec des groupes de machines virtuelles identiques Azure
description: Découvrez comment utiliser l’extension Intégrité de l’application pour surveiller l’intégrité de vos applications déployées sur des groupes de machines virtuelles identiques.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: extensions
ms.date: 05/06/2020
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: a38a715b45ab4d0810862ef4d016e4187ea507ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84783042"
---
# <a name="using-application-health-extension-with-virtual-machine-scale-sets"></a>Utilisation de l’extension Intégrité de l’application avec des groupes de machines virtuelles identiques
La surveillance de l’intégrité de votre application est un signal important pour la gestion et la mise à niveau votre déploiement. Les groupes de machines virtuelles identiques prennent en charge les [mises à niveau propagées](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model), notamment les [mises à niveau automatiques d’image de système d’exploitation](virtual-machine-scale-sets-automatic-upgrade.md), qui reposent sur l’analyse du fonctionnement des instances individuelles pour mettre à niveau votre déploiement. Vous pouvez également utiliser l’extension d’intégrité pour surveiller l’intégrité des applications de chaque instance de votre groupe identique et effectuer des réparations d’instance à l’aide de [réparations automatiques d’instances](virtual-machine-scale-sets-automatic-instance-repairs.md).

Cet article décrit comment utiliser l’extension Intégrité de l’application pour analyser le fonctionnement de vos applications déployées sur des groupes de machines virtuelles identiques.

## <a name="prerequisites"></a>Prérequis
Cet article suppose de connaître :
-   Les [extensions](../virtual-machines/extensions/overview.md) de machine virtuelle Azure
-   La [modification](virtual-machine-scale-sets-upgrade-scale-set.md) des groupes de machines virtuelles identiques

## <a name="when-to-use-the-application-health-extension"></a>Quand utiliser l’extension Intégrité de l’application
L’extension Intégrité de l’application est déployée à l’intérieur d’une instance de groupe de machines virtuelles identiques et rend compte de l’intégrité des machines virtuelles à partir de l’instance de groupe identique. Vous pouvez configurer l’extension pour sonder un point de terminaison d’application et mettre à jour l’état de l’application sur cette instance. Cet état de l’instance est vérifié par Azure pour déterminer si une instance est éligible pour des opérations de mise à niveau.

Étant donné que l’extension rend compte de l’intégrité à partir d’une machine virtuelle, elle peut être utilisée dans les situations où les sondes externes telles que les sondes d’intégrité d’application (qui utilisent des [sondes](../load-balancer/load-balancer-custom-probe-overview.md) Azure Load Balancer personnalisées) ne peuvent pas être utilisées.

## <a name="extension-schema"></a>Schéma d’extensions

Le JSON suivant montre le schéma pour l’extension Intégrité de l’application. L’extension nécessite au minimum une requête « tcp », « http » ou « https » avec respectivement un port ou un chemin d’accès à la demande associés.

```json
{
  "type": "extensions",
  "name": "HealthExtension",
  "apiVersion": "2018-10-01",
  "location": "<location>",  
  "properties": {
    "publisher": "Microsoft.ManagedServices",
    "type": "< ApplicationHealthLinux or ApplicationHealthWindows>",
    "autoUpgradeMinorVersion": true,
    "typeHandlerVersion": "1.0",
    "settings": {
      "protocol": "<protocol>",
      "port": "<port>",
      "requestPath": "</requestPath>"
    }
  }
}  
```

### <a name="property-values"></a>Valeurs de propriétés

| Nom | Valeur/Exemple | Type de données
| ---- | ---- | ---- 
| apiVersion | `2018-10-01` | Date |
| publisher | `Microsoft.ManagedServices` | string |
| type | `ApplicationHealthLinux` (Linux), `ApplicationHealthWindows` (Windows) | string |
| typeHandlerVersion | `1.0` | int |

### <a name="settings"></a>Paramètres

| Nom | Valeur/Exemple | Type de données
| ---- | ---- | ----
| protocol | `http` ou `https` ou `tcp` | string |
| port | Facultatif si le protocole est `http` ou `https`, obligatoire si le protocole est `tcp` | int |
| requestPath | Obligatoire si le protocole est `http` ou `https`, non autorisé si le protocole est `tcp` | string |

## <a name="deploy-the-application-health-extension"></a>Déployer l’extension Intégrité de l’application
Il existe plusieurs façons de déployer l’extension Intégrité de l’application sur vos groupes identiques, comme indiqué dans les exemples ci-dessous.

### <a name="rest-api"></a>API REST

L'exemple suivant ajoute l'extension Intégrité de l'application (avec le nom myHealthExtension) à extensionProfile dans le modèle d'un groupe identique basé sur Windows.

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/myHealthExtension?api-version=2018-10-01`
```

```json
{
  "name": "myHealthExtension",
  "properties": {
    "publisher": " Microsoft.ManagedServices",
    "type": "< ApplicationHealthWindows>",
    "autoUpgradeMinorVersion": true,
    "typeHandlerVersion": "1.0",
    "settings": {
      "protocol": "<protocol>",
      "port": "<port>",
      "requestPath": "</requestPath>"
    }
  }
}
```
Utilisez `PATCH` pour modifier une extension déjà déployée.

### <a name="azure-powershell"></a>Azure PowerShell

Utilisez l’applet de commande [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) pour ajouter l’extension Intégrité de l’application à la définition du modèle de groupe identique.

L'exemple suivant ajoute l'extension Intégrité de l'application à `extensionProfile` dans le modèle d'un groupe identique basé sur Windows. L'exemple utilise le nouveau module Az PowerShell.

```azurepowershell-interactive
# Define the scale set variables
$vmScaleSetName = "myVMScaleSet"
$vmScaleSetResourceGroup = "myVMScaleSetResourceGroup"

# Define the Application Health extension properties
$publicConfig = @{"protocol" = "http"; "port" = 80; "requestPath" = "/healthEndpoint"};
$extensionName = "myHealthExtension"
$extensionType = "ApplicationHealthWindows"
$publisher = "Microsoft.ManagedServices"

# Get the scale set object
$vmScaleSet = Get-AzVmss `
  -ResourceGroupName $vmScaleSetResourceGroup `
  -VMScaleSetName $vmScaleSetName

# Add the Application Health extension to the scale set model
Add-AzVmssExtension -VirtualMachineScaleSet $vmScaleSet `
  -Name $extensionName `
  -Publisher $publisher `
  -Setting $publicConfig `
  -Type $extensionType `
  -TypeHandlerVersion "1.0" `
  -AutoUpgradeMinorVersion $True

# Update the scale set
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `
  -Name $vmScaleSetName `
  -VirtualMachineScaleSet $vmScaleSet
```


### <a name="azure-cli-20"></a>Azure CLI 2.0

Utilisez la commande [az vmss extension set](/cli/azure/vmss/extension#az-vmss-extension-set) pour ajouter l’extension Intégrité de l’application à la définition du modèle de groupe identique.

L’exemple suivant ajoute l’extension Intégrité de l’application au modèle d’un groupe identique basé sur Linux.

```azurecli-interactive
az vmss extension set \
  --name ApplicationHealthLinux \
  --publisher Microsoft.ManagedServices \
  --version 1.0 \
  --resource-group <myVMScaleSetResourceGroup> \
  --vmss-name <myVMScaleSet> \
  --settings ./extension.json
```
Contenu du fichier d’extension.json.

```json
{
  "protocol": "<protocol>",
  "port": "<port>",
  "requestPath": "</requestPath>"
}
```


## <a name="troubleshoot"></a>Dépanner
La sortie de l’exécution de l’extension est journalisées dans des fichiers figurant dans les répertoires suivants :

```Windows
C:\WindowsAzure\Logs\Plugins\Microsoft.ManagedServices.ApplicationHealthWindows\<version>\
```

```Linux
/var/lib/waagent/Microsoft.ManagedServices.ApplicationHealthLinux-<extension_version>/status
/var/log/azure/applicationhealth-extension
```

Les journaux d’activité capturent également régulièrement l’état d’intégrité de l’application.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [déployer votre application](virtual-machine-scale-sets-deploy-app.md) sur des groupes de machines virtuelles identiques.
