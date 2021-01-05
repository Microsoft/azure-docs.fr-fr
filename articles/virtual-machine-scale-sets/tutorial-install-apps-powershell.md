---
title: Tutoriel - Installer des applications dans un groupe identique avec Azure PowerShell
description: Découvrez comment utiliser Azure PowerShell pour installer des applications dans des groupes identiques de machines virtuelles avec l’extension de script personnalisé
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: powershell
ms.date: 11/08/2018
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: e783f7f0a9be413679e509e4d6124d50bb811821
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87059283"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-azure-powershell"></a>Didacticiel : Installer des applications dans des groupes identiques de machines virtuelles avec Azure PowerShell

Pour exécuter des applications sur des instances de machine virtuelle d’un groupe identique, vous devez d’abord installer les composants d’application et les fichiers requis. Dans un didacticiel précédent, vous avez appris à créer et utiliser une image personnalisée de machine virtuelle pour déployer vos instances de machine virtuelle. Cette image personnalisée comprenait l’installation et la configuration manuelles d’applications. Vous pouvez également automatiser l’installation des applications pour un groupe identique après le déploiement de chaque instance de machine virtuelle, ou mettre à jour une application déjà exécutée dans un groupe identique. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Installer automatiquement des applications dans votre groupe identique
> * Utiliser l’extension de script personnalisé Azure
> * Mettre à jour une application en cours d’exécution dans un groupe identique

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="what-is-the-azure-custom-script-extension"></a>Qu’est-ce que l’extension de script personnalisé Azure ?
L’extension de script personnalisé télécharge et exécute des scripts sur des machines virtuelles Azure. Cette extension est utile pour la configuration post-déploiement, l’installation de logiciels ou toute autre tâche de configuration ou de gestion. Des scripts peuvent être téléchargés à partir de Stockage Azure ou de GitHub, ou fournis dans le portail Azure lors de l’exécution de l’extension.

L’extension de script personnalisé est compatible avec les modèles Azure Resource Manager. Elle peut également être utilisée avec Azure CLI, Azure PowerShell, le portail Azure ou l’API REST. Pour plus d’informations, consultez [Vue d’ensemble de l’extension de script personnalisé](../virtual-machines/extensions/custom-script-windows.md).

Pour voir l’extension de script personnalisé en action, créez un groupe identique qui installe le serveur web IIS et qui affiche le nom d’hôte de l’instance de machine virtuelle du groupe identique. La définition de l’extension de script personnalisé télécharge un exemple de script à partir de GitHub, installe les packages requis, puis écrit le nom d’hôte de l’instance de machine virtuelle sur une page HTML de base.


## <a name="create-a-scale-set"></a>Créer un groupe identique
À présent, créez un groupe de machines virtuelles identiques avec [New-AzVmss](/powershell/module/az.compute/new-azvmss). Pour distribuer le trafic aux différentes instances de machine virtuelle, un équilibreur de charge est également créé. L’équilibreur de charge inclut des règles pour distribuer le trafic sur le port TCP 80. Il autorise également le trafic du Bureau à distance sur le port TCP 3389 et la communication à distance PowerShell sur le port TCP 5985. Quand vous y êtes invité, vous pouvez définir vos propres informations d’identification d’administration pour les instances de machine virtuelle du groupe identique :

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic"
```

La création et la configuration des l’ensemble des ressources et des machines virtuelles du groupe identique prennent quelques minutes.


## <a name="create-custom-script-extension-definition"></a>Créer une définition d’extension de script personnalisé
Azure PowerShell utilise une table de hachage pour stocker le fichier à télécharger et la commande à exécuter. L’exemple suivant utilise un exemple de script tiré de GitHub. Commencez d’abord par créer cet objet de configuration comme suit :

```azurepowershell-interactive
$customConfig = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}
```


Ensuite, appliquez l’extension de script personnalisé avec [Add-AzVmssExtension](/powershell/module/az.Compute/Add-azVmssExtension). L’objet de configuration défini précédemment est passé à l’extension. Mettez à jour et exécutez l’extension sur les instances de machine virtuelle avec [Update-AzVmss](/powershell/module/az.compute/update-azvmss).


```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Add the Custom Script Extension to install IIS and configure basic website
$vmss = Add-AzVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.9 `
  -Setting $customConfig

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Chaque instance de machine virtuelle dans le groupe identique télécharge et exécute le script à partir de GitHub. Dans un exemple plus complexe, les composants et fichiers de plusieurs applications peuvent être installés. Si le groupe identique monte en puissance, les nouvelles instances de machine virtuelle appliquent automatiquement la même définition d’extension de script personnalisé et installent l’application requise.


## <a name="allow-traffic-to-application"></a>Autoriser le trafic vers l’application

Pour autoriser l’accès à l’application web de base, créez un groupe de sécurité réseau avec [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) et [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). Pour plus d’informations, consultez [Fonctionnalités réseau pour les groupes de machines virtuelles identiques Azure](virtual-machine-scale-sets-networking.md).

```azurepowershell-interactive

#Create a rule to allow traffic over port 80
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow

#Create a network security group and associate it with the rule
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName  "myResourceGroup" `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName  "myResourceGroup" `
  -Name myVnet

$frontendSubnet = $vnet.Subnets[0]

$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name mySubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend

Set-AzVirtualNetwork -VirtualNetwork $vnet

```



## <a name="test-your-scale-set"></a>Tester votre groupe identique
Pour voir votre serveur web en action, obtenez l’adresse IP publique de votre équilibreur de charge avec [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress). L’exemple suivant affiche l’adresse IP créée dans le groupe de ressources *myResourceGroup* :

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" | Select IpAddress
```

Saisissez l’adresse IP publique de l’équilibreur de charge dans un navigateur web. L’équilibreur de charge répartit le trafic vers l’une de vos instances de machine virtuelle, comme illustré dans l’exemple suivant :

![Page web de base dans IIS](media/tutorial-install-apps-powershell/running-iis.png)

Laissez le navigateur web ouvert afin que vous puissiez voir une version mise à jour à l’étape suivante.


## <a name="update-app-deployment"></a>Déployer une mise à jour d’application
Tout au long du cycle de vie d’un groupe identique, vous devrez peut-être déployer une version mise à jour de votre application. Avec l’extension de script personnalisé, vous pouvez faire référence à un script de déploiement de mises à jour puis réappliquer l’extension à votre groupe identique. Une fois le groupe identique créé à l’étape précédente, `-UpgradePolicyMode` a été défini sur *Automatique*. Ce paramètre permet aux instances de machine virtuelle dans le groupe identique de mettre automatiquement à jour et d’appliquer la dernière version de votre application.

Créez une nouvelle définition de configuration nommée *customConfigv2*. Cette définition exécute une version *v2* mise à jour du script d’installation de l’application :

```azurepowershell-interactive
$customConfigv2 = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis-v2.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis-v2.ps1"
}
```

Mettez à jour la configuration de l’extension de script personnalisé pour les instances de machine virtuelle dans votre groupe identique. Cette définition *customConfigv2* est utilisée pour appliquer la version mise à jour de l’application :

```azurepowershell-interactive
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"
 
$vmss.VirtualMachineProfile.ExtensionProfile[0].Extensions[0].Settings = $customConfigv2
 
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Toutes les instances de machine virtuelle dans le groupe identique sont automatiquement mises à jour avec la dernière version de la page web exemple. Pour afficher la version mise à jour, actualisez le site web dans votre navigateur :

![Page web mise à jour dans IIS](media/tutorial-install-apps-powershell/running-iis-updated.png)


## <a name="clean-up-resources"></a>Nettoyer les ressources
Pour supprimer votre groupe identique et d’autres ressources, supprimez le groupe de ressources et toutes ses ressources avec [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). Le paramètre `-Force` confirme que vous souhaitez supprimer les ressources sans passer par une invite supplémentaire à cette fin. Le paramètre `-AsJob` retourne le contrôle à l’invite de commandes sans attendre que l’opération se termine.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris comment installer et mettre à jour automatiquement des applications dans votre groupe identique avec Azure PowerShell :

> [!div class="checklist"]
> * Installer automatiquement des applications dans votre groupe identique
> * Utiliser l’extension de script personnalisé Azure
> * Mettre à jour une application en cours d’exécution dans un groupe identique

Passez au didacticiel suivant pour apprendre à mettre automatiquement à l’échelle votre groupe identique.

> [!div class="nextstepaction"]
> [Mettre automatiquement à l’échelle vos groupes identiques](tutorial-autoscale-powershell.md)
