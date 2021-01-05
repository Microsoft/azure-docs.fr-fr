---
title: Utiliser le séquencement d’extensions avec des groupes de machines virtuelles identiques Azure
description: Découvrez comment séquencer le provisionnement des extensions quand vous déployez plusieurs extensions dans des groupes de machines virtuelles identiques.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: extensions
ms.date: 01/30/2019
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 3271041b9f4db100cd05588129c7d714d4478f10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83121029"
---
# <a name="sequence-extension-provisioning-in-virtual-machine-scale-sets"></a>Séquencer le provisionnement des extensions dans des groupes de machines virtuelles identiques
Les extensions de machine virtuelle Azure apportent diverses fonctionnalités, telles que la configuration et la gestion après le déploiement, la supervision, la sécurité, et bien plus encore. Les déploiements de production utilisent généralement une combinaison de plusieurs extensions configurées pour les instances de machine virtuelle afin d’obtenir les résultats souhaités.

Si vous utilisez plusieurs extensions sur une machine virtuelle, il est important de vous assurer que les extensions nécessitant les mêmes ressources de système d’exploitation n’essaieront pas d’acquérir ces ressources au même moment. De plus, certaines extensions peuvent dépendre d’autres extensions pour fournir les configurations requises, comme les paramètres d’environnement et les secrets. Sans un classement et un séquencement corrects des différentes extensions, les déploiements des extensions dépendantes peuvent échouer.

Cet article explique comment séquencer les extensions configurées pour les instances de machine virtuelle dans les groupes de machines virtuelles identiques.

## <a name="prerequisites"></a>Prérequis
Cet article suppose de connaître :
-   Les [extensions](../virtual-machines/extensions/overview.md) de machine virtuelle Azure
-   La [modification](virtual-machine-scale-sets-upgrade-scale-set.md) des groupes de machines virtuelles identiques

## <a name="when-to-use-extension-sequencing"></a>Quand utiliser le séquencement des extensions ?
Le séquencement des extensions n’est pas obligatoire pour les groupes identiques et, sauf indication contraire, les extensions peuvent être provisionnées sur une instance de groupe identique dans n’importe quel ordre.

Par exemple, si votre modèle de groupe identique spécifie deux extensions, ExtensionA et ExtensionB, les deux séquences de provisionnement suivantes peuvent se produire :
-   ExtensionA -> ExtensionB
-   ExtensionB -> ExtensionA

Si votre application demande que la séquence ExtensionA soit toujours provisionnée avant la séquence ExtensionB, vous devez utiliser le séquencement des extensions comme expliqué dans cet article. Après le séquencement des extensions, une seule séquence peut se produire :
-   ExtensionA -> ExtensionB

Toutes les extensions non spécifiées dans une séquence de provisionnement définie peuvent être provisionnées à tout moment, y compris avant, après ou pendant une séquence définie. Le séquencement des extensions indique seulement qu’une extension spécifique doit être provisionnée après une autre extension spécifique. Il n’impacte pas le provisionnement des autres extensions définies dans le modèle.

Par exemple, si votre modèle de groupe identique spécifie trois extensions, ExtensionA, ExtensionB et ExtensionC, où ExtensionC est configurée pour être provisionnée après ExtensionA, les séquences de provisionnement suivantes peuvent se produire :
-   ExtensionA -> ExtensionC -> ExtensionB
-   ExtensionB -> ExtensionA -> ExtensionC
-   ExtensionA -> ExtensionB -> ExtensionC

Si vous voulez empêcher le provisionnement d’une autre extension durant l’exécution de la séquence d’extensions définie, nous vous recommandons de séquencer toutes les extensions dans votre modèle de groupe identique. Dans l’exemple ci-dessus, ExtensionB peut être configurée pour être provisionnée après ExtensionC afin qu’une seule séquence puisse se produire :
-   ExtensionA -> ExtensionC -> ExtensionB


## <a name="how-to-use-extension-sequencing"></a>Comment utiliser le séquencement des extensions ?
Pour séquencer le provisionnement des extensions, vous devez mettre à jour la définition d’extension dans le modèle de groupe identique afin d’inclure la propriété « provisionAfterExtensions », qui accepte un tableau de noms d’extensions. Les extensions indiquées dans le tableau de la propriété doivent être pleinement définies dans le modèle de groupe identique.

### <a name="template-deployment"></a>Déploiement de modèle
L’exemple suivant définit un modèle où le groupe identique a trois extensions, ExtensionA, ExtensionB et ExtensionC, configurées pour être provisionnées dans cet ordre :
-   ExtensionA -> ExtensionB -> ExtensionC

```json
"virtualMachineProfile": {
  "extensionProfile": {
    "extensions": [
      {
        "name": "ExtensionA",
        "properties": {
          "publisher": "ExtensionA.Publisher",
          "settings": {},
          "typeHandlerVersion": "1.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionA"
        }
      },
      {
        "name": "ExtensionB",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionA"
          ],
          "publisher": "ExtensionB.Publisher",
          "settings": {},
          "typeHandlerVersion": "2.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionB"
        }
      }, 
      {
        "name": "ExtensionC",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionB"
          ],
          "publisher": "ExtensionC.Publisher",
          "settings": {},
          "typeHandlerVersion": "3.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionC"                   
        }
      }
    ]
  }
}
```

Étant donné que la propriété « provisionAfterExtensions » accepte un tableau de noms d’extensions, l’exemple ci-dessus peut être modifié pour qu’ExtensionC soit provisionnée après ExtensionA et ExtensionB, mais sans ordre imposé entre ExtensionA et ExtensionB. Le modèle suivant permet de réaliser ce scénario :

```json
"virtualMachineProfile": {
  "extensionProfile": {
    "extensions": [
      {
        "name": "ExtensionA",
        "properties": {
          "publisher": "ExtensionA.Publisher",
          "settings": {},
          "typeHandlerVersion": "1.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionA"
        }
      },
      {
        "name": "ExtensionB",
        "properties": {
          "publisher": "ExtensionB.Publisher",
          "settings": {},
          "typeHandlerVersion": "2.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionB"
        }
      }, 
      {
        "name": "ExtensionC",
        "properties": {
          "provisionAfterExtensions": [
            "ExtensionA","ExtensionB"
          ],
          "publisher": "ExtensionC.Publisher",
          "settings": {},
          "typeHandlerVersion": "3.0",
          "autoUpgradeMinorVersion": true,
          "type": "ExtensionC"                   
        }
      }
    ]
  }
}
```

### <a name="rest-api"></a>API REST
L’exemple suivant ajoute une nouvelle extension nommée ExtensionC à un modèle de groupe identique. ExtensionC a des dépendances sur ExtensionA et ExtensionB, qui ont déjà été définies dans ce modèle.

```
PUT on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/ExtensionC?api-version=2018-10-01`
```

```json
{ 
  "name": "ExtensionC",
  "properties": {
    "provisionAfterExtensions": [
      "ExtensionA","ExtensionB"
    ],
    "publisher": "ExtensionC.Publisher",
    "settings": {},
    "typeHandlerVersion": "3.0",
    "autoUpgradeMinorVersion": true,
    "type": "ExtensionC" 
  }                  
}
```

Si ExtensionC a été définie précédemment dans le modèle de groupe identique et que vous souhaitez maintenant ajouter ses dépendances, vous pouvez exécuter un `PATCH` pour modifier les propriétés de l’extension déjà déployée.

```
PATCH on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/extensions/ExtensionC?api-version=2018-10-01`
```

```json
{ 
  "properties": {
    "provisionAfterExtensions": [
      "ExtensionA","ExtensionB"
    ]
  }                  
}
```
Les modifications apportées aux instances du groupe identique existantes sont appliquées à la [mise à niveau](virtual-machine-scale-sets-upgrade-scale-set.md#how-to-bring-vms-up-to-date-with-the-latest-scale-set-model) suivante.

### <a name="azure-powershell"></a>Azure PowerShell
Utilisez l’applet de commande [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) pour ajouter l’extension Intégrité de l’application à la définition du modèle de groupe identique. Le séquencement des extensions nécessite l’utilisation d’Az PowerShell 1.2.0 ou ultérieur.

L’exemple suivant ajoute l’[extension Intégrité de l’application](virtual-machine-scale-sets-health-extension.md) au `extensionProfile` dans le modèle d’un groupe identique basé sur Windows. L’extension Intégrité de l’application est provisionnée après le provisionnement de l’[extension Script personnalisé](../virtual-machines/extensions/custom-script-windows.md), déjà définie dans le groupe identique.

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
  -ProvisionAfterExtension "CustomScriptExtension" `
  -AutoUpgradeMinorVersion $True

# Update the scale set
Update-AzVmss -ResourceGroupName $vmScaleSetResourceGroup `
  -Name $vmScaleSetName `
  -VirtualMachineScaleSet $vmScaleSet
```

### <a name="azure-cli-20"></a>Azure CLI 2.0
Utilisez la commande [az vmss extension set](/cli/azure/vmss/extension#az-vmss-extension-set) pour ajouter l’extension Intégrité de l’application à la définition du modèle de groupe identique. Le séquencement des extensions nécessite l’utilisation d’Azure CLI 2.0.55 ou ultérieur.

L’exemple suivant ajoute l’[extension Intégrité de l’application](virtual-machine-scale-sets-health-extension.md) au modèle d’un groupe identique basé sur Windows. L’extension Intégrité de l’application est provisionnée après le provisionnement de l’[extension Script personnalisé](../virtual-machines/extensions/custom-script-windows.md), déjà définie dans le groupe identique.

```azurecli-interactive
az vmss extension set \
  --name ApplicationHealthWindows \
  --publisher Microsoft.ManagedServices \
  --version 1.0 \
  --resource-group <myVMScaleSetResourceGroup> \
  --vmss-name <myVMScaleSet> \
  --provision-after-extensions CustomScriptExtension \
  --settings ./extension.json
```


## <a name="troubleshoot"></a>Dépanner

### <a name="not-able-to-add-extension-with-dependencies"></a>Vous ne pouvez pas ajouter d’extension avec des dépendances ?
1. Assurez-vous que les extensions spécifiées dans provisionAfterExtensions sont définies dans le modèle de groupe identique.
2. Vérifiez qu’il n’y a pas de dépendances circulaires présentes. Par exemple, la séquence suivante n’est pas autorisée : ExtensionA -> ExtensionB -> ExtensionC -> ExtensionA
3. Vérifiez que chaque extension sur laquelle il existe des dépendances a une propriété « settings » définie sous l’élément « properties » de l’extension. Par exemple, si ExtensionB doit être provisionnée après ExtensionA, ExtensionA doit avoir la propriété « settings » spécifiée sous « properties » dans ExtensionA. Vous pouvez spécifier une propriété « settings » vide si l’extension n’a pas de paramètres obligatoires.

### <a name="not-able-to-remove-extensions"></a>Vous ne pouvez pas supprimer les extensions ?
Vérifiez que les extensions à supprimer ne sont pas listées sous provisionAfterExtensions pour une autre extension.

## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [déployer votre application](virtual-machine-scale-sets-deploy-app.md) sur des groupes de machines virtuelles identiques.
