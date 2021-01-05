---
title: 'Tutoriel : Créer des images de machine virtuelle personnalisées avec Azure PowerShell'
description: Dans ce tutoriel, vous allez découvrir comment utiliser Azure PowerShell pour créer une image de machine virtuelle personnalisée Windows stockée dans une galerie d’images partagées Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 05/01/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: cf8fc9916384c9eef24c4c50f7647632c0e6b7a2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87077458"
---
# <a name="tutorial-create-windows-vm-images-with-azure-powershell"></a>Tutoriel : Créer des images de machines virtuelles Windows avec Azure PowerShell

Vous pouvez utiliser des images pour démarrer des déploiements et en garantir la cohérence sur plusieurs machines virtuelles. Dans ce tutoriel, vous allez créer votre propre image spécialisée d’une machine virtuelle Azure à l’aide de PowerShell et la stocker dans Shared Image Gallery. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image
> * Créer une version d’image
> * Créer une machine virtuelle à partir d’une image 
> * Partager une galerie d’images



## <a name="before-you-begin"></a>Avant de commencer

Les étapes ci-dessous expliquent comment prendre une machine virtuelle existante et la transformer en une image personnalisée réutilisable pour créer de nouvelles machines virtuelles.

Pour exécuter l’exemple dans ce didacticiel, vous devez disposer d’une machine virtuelle. Si nécessaire, vous pouvez consulter le [guide de démarrage rapide PowerShell](quick-create-powershell.md) pour créer une machine virtuelle à utiliser pour ce tutoriel. Au cours du tutoriel, remplacez les noms des ressources si nécessaire.

## <a name="overview"></a>Vue d’ensemble

Une [galerie d’images partagées](shared-image-galleries.md) simplifie considérablement le partage d’images personnalisées dans votre organisation. Les images personnalisées sont comme des images de la Place de marché, sauf que vous les créez vous-même. Les images personnalisées peuvent être utilisées pour amorcer des configurations comme le préchargement des applications, les configurations d’application et d’autres configurations de système d’exploitation. 

Shared Image Gallery vous permet de partager vos images de machines virtuelles personnalisées avec d’autres utilisateurs. Choisissez les images à partager, les régions dans lesquelles vous souhaitez les rendre disponibles et les personnes avec lesquelles vous voulez les partager. 

La fonctionnalité Galerie d’images partagées a plusieurs types de ressources :

[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]


## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="get-the-vm"></a>Obtenir la machine virtuelle

Pour voir la liste de machines virtuelles disponibles dans un groupe de ressources, utilisez [Get-AzVM](/powershell/module/az.compute/get-azvm). Une fois le nom de la machine virtuelle et le groupe de ressources connus, vous pouvez réutiliser `Get-AzVM` pour obtenir l’objet de machine virtuelle et le stocker dans une variable afin de l’utiliser ultérieurement. Cet exemple obtient une machine virtuelle nommée *sourceVM* à partir du groupe de ressources « myResourceGroup » et l’assigne à la variable *$sourceVM*. 

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

Les définitions d’image créent un regroupement logique des images. Elles sont utilisées pour gérer les informations sur les versions d’image créées au sein de celles-ci. Les noms de définition d’image peuvent contenir des lettres majuscules ou minuscules, des chiffres, des tirets et des points. Pour plus d’informations sur les valeurs que vous pouvez spécifier pour une définition d’image, consultez [Définitions d’image](./shared-image-galleries.md#image-definitions).

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


## <a name="create-a-vm"></a>Créer une machine virtuelle 

Une fois que vous disposez d’une image spécialisée, vous pouvez créer une ou plusieurs nouvelles machines virtuelles. Utilisation de l’applet de commande [New-AzVM](/powershell/module/az.compute/new-azvm). Pour vous servir de l’image, utilisez `Set-AzVMSourceImage` et définissez `-Id` sur l’ID de définition d’image ($galleryImage.Id dans le cas présent) afin de toujours utiliser la version la plus récente de l’image. 

Remplacez les noms de ressources en fonction des besoins dans cet exemple. 

```azurepowershell-interactive
# Create some variables for the new VM.
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "mySpecializedVM"

# Create a resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create the network resources.
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name $vmName -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using $imageVersion.Id to specify the image version.
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1_v2 | `
Set-AzVMSourceImage -Id $galleryImage.Id | `
Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```


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

# Delete the VM
Remove-AzResourceGroup -Name myResoureceGroup
```

## <a name="azure-image-builder"></a>Azure Image Builder

Azure propose également un service, basé sur Packer, nommé [Azure VM Image Builder](./image-builder-overview.md). Il vous suffit de décrire vos personnalisations dans un modèle, et celui-ci gère la création d’images. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé une image de machine virtuelle spécialisée. Vous avez appris à :

> [!div class="checklist"]
> * Créer une galerie Shared Image Gallery
> * Créer une définition d’image
> * Créer une version d’image
> * Créer une machine virtuelle à partir d’une image 
> * Partager une galerie d’images

Passez au tutoriel suivant pour en savoir plus sur la création de machines virtuelles hautement disponibles.

> [!div class="nextstepaction"]
> [Créer des machines virtuelles hautement disponibles](tutorial-availability-sets.md)
