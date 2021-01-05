---
title: Tutoriel - Utiliser une image de machine virtuelle personnalisée dans un groupe identique avec Azure PowerShell
description: Découvrez comment utiliser Azure PowerShell afin de créer une image de machine virtuelle personnalisée que vous pouvez utiliser pour déployer un groupe de machines virtuelles identiques
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.topic: tutorial
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: akjosh
ms.openlocfilehash: 8a0ee8565d22e935535bbe1e830d8399b2489f5c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86527521"
---
# <a name="tutorial-create-and-use-a-custom-image-for-virtual-machine-scale-sets-with-azure-powershell"></a>Tutoriel : Créer et utiliser une image personnalisée pour des groupes de machines virtuelles identiques avec Azure PowerShell

Lorsque vous créez un groupe identique, vous spécifiez une image à utiliser lors du déploiement des instances de machine virtuelle. Pour réduire le nombre de tâches une fois que les instances de machine virtuelle sont déployées, vous pouvez utiliser une image de machine virtuelle personnalisée. Cette image de machine virtuelle personnalisée inclut les configurations ou installations des applications requises. Toutes les instances de machine virtuelle créées dans le groupe identique utilisent l’image de machine virtuelle personnalisée et sont prêtes à répondre au trafic des applications. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image
> * Créer une version d’image
> * Créer un groupe identique à partir d’une image 
> * Partager une galerie d’images

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="before-you-begin"></a>Avant de commencer

Les étapes ci-dessous expliquent comment prendre une machine virtuelle existante et la transformer en une image personnalisée réutilisable pour créer un groupe identique.

Pour exécuter l’exemple dans ce didacticiel, vous devez disposer d’une machine virtuelle. Si nécessaire, vous pouvez consulter le [guide de démarrage rapide PowerShell](quick-create-powershell.md) pour créer une machine virtuelle à utiliser pour ce tutoriel. Au cours du tutoriel, remplacez les noms des ressources si nécessaire.

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.


## <a name="get-the-vm"></a>Obtenir la machine virtuelle

Pour afficher une liste de machines virtuelles disponibles dans un groupe de ressources, utilisez [Get-AzVM](/powershell/module/az.compute/get-azvm). Une fois le nom de la machine virtuelle et le groupe de ressources connus, vous pouvez réutiliser `Get-AzVM` pour obtenir l’objet de machine virtuelle et le stocker dans une variable à des fins d’utilisation ultérieure. Cet exemple obtient une machine virtuelle nommée *sourceVM* à partir du groupe de ressources « myResourceGroup » et l’assigne à la variable *$vm*. 

```azurepowershell-interactive
$sourceVM = Get-AzVM `
   -Name sourceVM `
   -ResourceGroupName myResourceGroup
```
## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Dans l’exemple suivant, un groupe de ressources nommé *myGalleryRG* est créé dans la région *EastUS* :

```azurepowershell-interactive
$resourceGroup = New-AzResourceGroup `
   -Name 'myGalleryRG' `
   -Location 'EastUS'
```

## <a name="create-an-image-gallery"></a>Créer une galerie d’images 

Une galerie d’images est la ressource principale utilisée pour activer le partage d’image. Les caractères autorisés pour le nom de galerie sont les lettres majuscules ou minuscules, les chiffres et les points. Le nom de galerie ne peut pas contenir de tirets. Les noms de galerie doivent être uniques dans votre abonnement. 

Créez une galerie d’images à l’aide de [New-AzureRmGallery](/powershell/module/az.compute/new-azgallery). L’exemple suivant crée une galerie nommée *myGallery* dans le groupe de ressources *myGalleryRG*.

```azurepowershell-interactive
$gallery = New-AzGallery `
   -GalleryName 'myGallery' `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -Description 'Shared Image Gallery for my organization'  
```


## <a name="create-an-image-definition"></a>Créer une définition d’image 

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur les versions d’image créées au sein de celles-ci. Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](../virtual-machines/windows/shared-image-galleries.md#image-definitions).

Créez la définition d’image à l’aide de [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). Dans cet exemple, l’image de la galerie se nomme *myGalleryImage* et est créée pour une image spécialisée. 

```azurepowershell-interactive
$galleryImage = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-an-image-version"></a>Créer une version d’image

Créez une version d’image à partir d’une machine virtuelle à l’aide de [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Les caractères autorisés pour la version d’image sont les nombres et les points. Les nombres doivent être un entier 32 bits. Format: *MajorVersion*.*MinorVersion*.*Patch*.

Dans cet exemple, la version d’image est *1.0.0* et est répliquée dans les deux centres de données *USA Est* et *USA Centre Sud*. Lors du choix des régions cibles pour la réplication, vous devez inclure la région *source* en tant que cible pour la réplication.

Pour créer une version d’image à partir de la machine virtuelle, utilisez `$vm.Id.ToString()` comme `-Source`.

```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='East US';ReplicaCount=2}
$targetRegions = @($region1,$region2)

New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -TargetRegion $targetRegions  `
   -Source $sourceVM.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-01'
```

La réplication de l’image à l’ensemble des régions cibles peut prendre un certain temps.

## <a name="create-a-scale-set-from-the-image"></a>Créer un groupe identique à partir de l’image
Créez à présent un groupe identique avec [New-AzVmss](/powershell/module/az.compute/new-azvmss) qui utilise le paramètre `-ImageName` pour définir l’image de machine virtuelle personnalisée créée à l’étape précédente. Pour distribuer le trafic aux différentes instances de machine virtuelle, un équilibreur de charge est également créé. L’équilibreur de charge inclut des règles pour distribuer le trafic sur le port TCP 80, ainsi que pour autoriser le trafic Bureau à distance sur le port TCP 3389 et le trafic Accès distant PowerShell sur le port TCP 5985. Lorsque vous y êtes invité, fournissez vos propres informations d’identification d’administration souhaitées pour les instances de machine virtuelle dans le groupe identique :

```azurepowershell-interactive
# Define variables for the scale set
$resourceGroupName = "myVMSSRG3"
$scaleSetName = "myScaleSet3"
$location = "East US"

# Create a resource group
New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location

# Create a networking pieces
$subnet = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.0.0/24
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName $resourceGroupName `
  -Name "myVnet" `
  -Location $location `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnet
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName $resourceGroupName `
  -Location $location `
  -AllocationMethod Static `
  -Name "myPublicIP"
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "myFrontEndPool" `
  -PublicIpAddress $publicIP
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name "myBackEndPool"
$inboundNATPool = New-AzLoadBalancerInboundNatPoolConfig `
  -Name "myRDPRule" `
  -FrontendIpConfigurationId $frontendIP.Id `
  -Protocol TCP `
  -FrontendPortRangeStart 50001 `
  -FrontendPortRangeEnd 50010 `
  -BackendPort 3389
# Create the load balancer and health probe
$lb = New-AzLoadBalancer `
  -ResourceGroupName $resourceGroupName `
  -Name "myLoadBalancer" `
  -Location $location `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -InboundNatPool $inboundNATPool
Add-AzLoadBalancerProbeConfig -Name "myHealthProbe" `
  -LoadBalancer $lb `
  -Protocol TCP `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2
Add-AzLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol TCP `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe (Get-AzLoadBalancerProbeConfig -Name "myHealthProbe" -LoadBalancer $lb)
Set-AzLoadBalancer -LoadBalancer $lb

# Create IP address configurations
$ipConfig = New-AzVmssIpConfig `
  -Name "myIPConfig" `
  -LoadBalancerBackendAddressPoolsId $lb.BackendAddressPools[0].Id `
  -LoadBalancerInboundNatPoolsId $inboundNATPool.Id `
  -SubnetId $vnet.Subnets[0].Id

# Create a configuration 
$vmssConfig = New-AzVmssConfig `
    -Location $location `
    -SkuCapacity 2 `
    -SkuName "Standard_DS2" `
    -UpgradePolicyMode "Automatic"

# Reference the image version
Set-AzVmssStorageProfile $vmssConfig `
  -OsDiskCreateOption "FromImage" `
  -ImageReferenceId $galleryImage.Id

# Complete the configuration
 
Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name "network-config" `
  -Primary $true `
  -IPConfiguration $ipConfig 

# Create the scale set 
New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Name $scaleSetName `
  -VirtualMachineScaleSet $vmssConfig
```

La création et la configuration des l’ensemble des ressources et des machines virtuelles du groupe identique prennent quelques minutes.


## <a name="share-the-gallery"></a>Partager la galerie

Nous vous recommandons de partager l’accès au niveau de la galerie d’image. Utilisez une adresse e-mail et la cmdlet [Get-AzADUser](/powershell/module/az.resources/get-azaduser) pour obtenir l’ID d’objet de l’utilisateur, puis utilisez [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment) pour leur donner accès à la galerie. Remplacez l’exemple d’e-mail, alinne_montes@contoso.com dans cet exemple, par vos propres informations.

```azurepowershell-interactive
# Get the object ID for the user
$user = Get-AzADUser -StartsWith alinne_montes@contoso.com
# Grant access to the user for our gallery
New-AzRoleAssignment `
   -ObjectId $user.Id `
   -RoleDefinitionName Reader `
   -ResourceName $gallery.Name `
   -ResourceType Microsoft.Compute/galleries `
   -ResourceGroupName $resourceGroup.ResourceGroupName
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du groupe de ressources et de toutes les ressources associées, vous pouvez utiliser l’applet de commande [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) pour les supprimer :

```azurepowershell-interactive
# Delete the gallery 
Remove-AzResourceGroup -Name myGalleryRG

# Delete the scale set resource group
Remove-AzResourceGroup -Name myResoureceGroup
```

## <a name="azure-image-builder"></a>Azure Image Builder

Azure propose également un service, basé sur Packer, nommé [Azure VM Image Builder](../virtual-machines/windows/image-builder-overview.md). Il vous suffit de décrire vos personnalisations dans un modèle, et celui-ci gère la création d’images. 

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à créer et utiliser une image de machine virtuelle personnalisée pour vos groupes identiques au moyen d’Azure PowerShell :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image
> * Créer une version d’image
> * Créer un groupe identique à partir d’une image 
> * Partager une galerie d’images

Passez au didacticiel suivant pour apprendre à déployer des applications dans votre groupe identique.

> [!div class="nextstepaction"]
> [Déployer des applications dans vos groupes identiques](tutorial-install-apps-powershell.md)
