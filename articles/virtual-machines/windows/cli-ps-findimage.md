---
title: Rechercher et utiliser des images et des plans de la Place de marché Azure
description: Utilisez Azure PowerShell pour rechercher et utiliser des informations sur l’éditeur, l’offre, la référence SKU, la version et le plan pour des images de machine virtuelle de la Place de marché.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 12/07/2020
ms.author: cynthn
ms.openlocfilehash: 45e6b157dba5ef7410d8a5c0223fd3ecb52f39d0
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906265"
---
# <a name="find-and-use-azure-marketplace-vm-images-with-azure-powershell"></a>Rechercher et utiliser des images de machine virtuelle de la Place de marché Azure avec Azure PowerShell     

Cet article décrit comment utiliser Azure PowerShell pour rechercher des images de machine virtuelle sur la Place de marché Microsoft Azure. Vous pouvez ensuite spécifier des informations sur une image et un plan de la Place de marché lorsque vous créez une machine virtuelle.

Vous pouvez également parcourir les offres et images disponibles à l’aide de la vitrine [Place de marché Microsoft Azure](https://azuremarketplace.microsoft.com/), du [Portail Azure](https://portal.azure.com) ou d’[Azure CLI](../linux/cli-ps-findimage.md). 


[!INCLUDE [virtual-machines-common-image-terms](../../../includes/virtual-machines-common-image-terms.md)]


## <a name="create-a-vm-from-vhd-with-plan-information"></a>Créer une machine virtuelle à partir d’un VHD avec des informations de plan

Si vous disposez d’un VHD existant qui a été créé à l’aide d’une image de la Place de marché Azure, vous devrez peut-être fournir les informations du plan d’achat lors de la création d’une machine virtuelle à partir de ce VHD.

Si vous avez toujours la machine virtuelle d’origine, ou une autre machine virtuelle créée à partir de la même image, vous pouvez obtenir le nom du plan, l’éditeur et les informations sur le produit à l’aide de Get-AzVM. Cet exemple obtient une machine virtuelle nommée *myVM* dans le groupe de ressources *myResourceGroup*, puis affiche les informations du plan d’achat.

```azurepowershell-interactive
$vm = Get-azvm `
   -ResourceGroupName myResourceGroup `
   -Name myVM
$vm.Plan
```

Si vous n’avez pas obtenu les informations du plan avant la suppression de la machine virtuelle d’origine, vous pouvez effectuer une [demande de support](https://ms.portal.azure.com/#create/Microsoft.Support). Les techniciens de support auront besoin du nom de la machine virtuelle, de l’ID d’abonnement et de l’horodatage de l’opération de suppression.

Pour créer une machine virtuelle à l’aide d’un VHD, consultez l’article [Créer une machine virtuelle à partir d’un VHD spécialisé](create-vm-specialized.md) et ajoutez une ligne pour ajouter les informations du plan à la configuration de la machine virtuelle à l’aide de [Set-AzVMPlan](/powershell/module/az.compute/set-azvmplan), comme ceci :

```azurepowershell-interactive
$vmConfig = Set-AzVMPlan `
   -VM $vmConfig `
   -Publisher "publisherName" `
   -Product "productName" `
   -Name "planName"
```

## <a name="create-a-new-vm-from-a-marketplace-image"></a>Créer une machine virtuelle à partir d’une image de la Place de marché

Si vous avez déjà les informations sur l’image que vous souhaitez utiliser, vous pouvez transmettre ces informations dans l’applet de commande [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage) pour ajouter les informations sur l’image à la configuration de la machine virtuelle. Pour rechercher et lister les images disponibles sur la Place de marché, consultez les sections suivantes.

Certaines images payantes vous demandent également de fournir des informations sur le plan d’achat à l’aide de [Set-AzVMPlan](/powershell/module/az.compute/set-azvmplan). 

```powershell
...

$vmConfig = New-AzVMConfig -VMName "myVM" -VMSize Standard_D1

# Set the Marketplace image
$offerName = "windows-data-science-vm"
$skuName = "windows2016"
$version = "19.01.14"
$vmConfig = Set-AzVMSourceImage -VM $vmConfig -PublisherName $publisherName -Offer $offerName -Skus $skuName -Version $version

# Set the Marketplace plan information, if needed
$publisherName = "microsoft-ads"
$productName = "windows-data-science-vm"
$planName = "windows2016"
$vmConfig = Set-AzVMPlan -VM $vmConfig -Publisher $publisherName -Product $productName -Name $planName

...
```

Vous transmettez ensuite la configuration de la machine virtuelle en même temps que les autres objets de configuration à l’applet de commande `New-AzVM`. Pour obtenir un exemple détaillé de l’utilisation d’une configuration de machine virtuelle avec PowerShell, consultez ce [script](https://github.com/Azure/azure-docs-powershell-samples/blob/master/virtual-machine/create-vm-detailed/create-windows-vm-detailed.ps1).

Si vous recevez un message concernant l’acceptation des conditions de l’image, consultez la section [Accepter les conditions](#accept-the-terms) plus loin dans cet article.

## <a name="list-images"></a>Répertorier des images

Pour rechercher une image à un emplacement, une méthode consiste à exécuter les cmdlets [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher), [Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer) et [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) :

1. en répertoriant les éditeurs d’images ;
2. pour un éditeur donné, en répertoriant ses offres ;
3. pour une offre donnée, en répertoriant ses références SKU.

Ensuite, pour une référence SKU sélectionnée, exécutez la cmdlet [Get-AzVMImage](/powershell/module/az.compute/get-azvmimage) afin de dresser la liste des versions à déployer.

1. Répertoriez les éditeurs d’images :

    ```powershell
    $locName="<Azure location, such as West US>"
    Get-AzVMImagePublisher -Location $locName | Select PublisherName
    ```

2. Indiquez le nom de l’éditeur choisi et répertoriez les offres :

    ```powershell
    $pubName="<publisher>"
    Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
    ```

3. Indiquez le nom de l’offre choisie et répertoriez les références SKU :

    ```powershell
    $offerName="<offer>"
    Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
    ```

4. Indiquez le nom de la référence choisie et obtenez la version de l’image :

    ```powershell
    $skuName="<SKU>"
    Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
    ```
    
Dans la sortie de la commande `Get-AzVMImage`, vous pouvez sélectionner une image de version pour déployer une nouvelle machine virtuelle.

L’exemple suivant présente l’intégralité de la séquence des commandes et leurs sorties :

```powershell
$locName="West US"
Get-AzVMImagePublisher -Location $locName | Select PublisherName
```

Résultat partiel :

```
PublisherName
-------------
...
abiquo
accedian
accellion
accessdata-group
accops
Acronis
Acronis.Backup
actian-corp
actian_matrix
actifio
activeeon
adgs
advantech
advantech-webaccess
advantys
...
```

Pour l’éditeur *MicrosoftWindowsServer* :

```powershell
$pubName="MicrosoftWindowsServer"
Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
```

Sortie :

```
Offer
-----
Windows-HUB
WindowsServer
WindowsServerSemiAnnual
```

Pour l’offre *WindowsServer* :

```powershell
$offerName="WindowsServer"
Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
```

Résultat partiel :

```
Skus
----
2008-R2-SP1
2008-R2-SP1-smalldisk
2012-Datacenter
2012-Datacenter-smalldisk
2012-R2-Datacenter
2012-R2-Datacenter-smalldisk
2016-Datacenter
2016-Datacenter-Server-Core
2016-Datacenter-Server-Core-smalldisk
2016-Datacenter-smalldisk
2016-Datacenter-with-Containers
2016-Datacenter-with-RDSH
2019-Datacenter
2019-Datacenter-Core
2019-Datacenter-Core-smalldisk
2019-Datacenter-Core-with-Containers
...
```

Ensuite, pour la référence SKU *2019-Datacenter* :

```powershell
$skuName="2019-Datacenter"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Sku $skuName | Select Version
```

Maintenant, vous pouvez combiner l’éditeur, l’offre, la référence SKU et la version sélectionnés en un schéma URN (valeurs séparées par :). Transmettez cet URN avec le paramètre `--image` lorsque vous créez une machine virtuelle à l'aide de la cmdlet [New-AzVM](/powershell/module/az.compute/new-azvm). Vous pouvez également remplacer le numéro de version dans l’URN par « latest » (dernière) afin d’obtenir la dernière version de l’image.

Si vous déployez une machine virtuelle avec un modèle Resource Manager, vous définissez alors les paramètres d’image de manière individuelle dans les propriétés `imageReference`. Consultez la [référence de modèle](/azure/templates/microsoft.compute/virtualmachines).

[!INCLUDE [virtual-machines-common-marketplace-plan](../../../includes/virtual-machines-common-marketplace-plan.md)]

### <a name="view-plan-properties"></a>Afficher les propriétés du plan

Pour afficher les informations du plan d’achat d’une image, exécutez la cmdlet `Get-AzVMImage`. Si la propriété `PurchasePlan` dans la sortie n’est pas `null`, l’image a des conditions que vous devez accepter avant le déploiement par programmation.  

Par exemple, l’image *Windows Server 2016 Datacenter* ne possède pas de conditions supplémentaires, car l’information `PurchasePlan` est `null` :

```powershell
$version = "2016.127.20170406"
Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName -Version $version
```

Sortie :

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/MicrosoftWindowsServer/ArtifactTypes/VMImage/Offers/WindowsServer/Skus/2016-Datacenter/Versions/2019.0.20190115
Location         : westus
PublisherName    : MicrosoftWindowsServer
Offer            : WindowsServer
Skus             : 2019-Datacenter
Version          : 2019.0.20190115
FilterExpression :
Name             : 2019.0.20190115
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : null
DataDiskImages   : []

```

L’exemple ci-dessous montre une commande similaire pour l’image *Data Science Virtual Machine - Windows 2016*, ayant les propriétés `PurchasePlan` suivantes : `name`, `product` et `publisher`. Certaines images ont également une propriété `promotion code`. Pour déployer cette image, consultez les sections suivantes pour accepter les conditions et activer le déploiement par programmation.

```powershell
Get-AzVMImage -Location "westus" -PublisherName "microsoft-ads" -Offer "windows-data-science-vm" -Skus "windows2016" -Version "0.2.02"
```

Sortie :

```
Id               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/microsoft-ads/ArtifactTypes/VMImage/Offers/windows-data-science-vm/Skus/windows2016/Versions/19.01.14
Location         : westus
PublisherName    : microsoft-ads
Offer            : windows-data-science-vm
Skus             : windows2016
Version          : 19.01.14
FilterExpression :
Name             : 19.01.14
OSDiskImage      : {
                     "operatingSystem": "Windows"
                   }
PurchasePlan     : {
                     "publisher": "microsoft-ads",
                     "name": "windows2016",
                     "product": "windows-data-science-vm"
                   }
DataDiskImages   : []

```

### <a name="accept-the-terms"></a>Accepter les conditions

Pour afficher les termes du contrat de licence, utilisez la cmdlet [Get-AzMarketplaceterms](/powershell/module/az.marketplaceordering/get-azmarketplaceterms) et transmettez les paramètres de plan d'achat. La sortie contient un lien vers les conditions de l’image de la Marketplace et indique si vous les avez acceptées précédemment. Veillez à utiliser uniquement des lettres minuscules dans les valeurs de paramètre.

```powershell
Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"
```

Sortie :

```
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DVM%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : 2UMWH6PHSAIM4U22HXPXW25AL2NHUJ7Y7GRV27EBL6SUIDURGMYG6IIDO3P47FFIBBDFHZHSQTR7PNK6VIIRYJRQ3WXSE6BTNUNENXA
Accepted          : False
Signdate          : 1/25/2019 7:43:00 PM
```

Utilisez la cmdlet [Set-AzMarketplaceterms](/powershell/module/az.marketplaceordering/set-azmarketplaceterms) pour accepter ou rejeter les conditions. Vous ne devez accepter qu’une fois les conditions par abonnement pour l’image. Veillez à utiliser uniquement des lettres minuscules dans les valeurs de paramètre. 

```powershell
$agreementTerms=Get-AzMarketplaceterms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016"

Set-AzMarketplaceTerms -Publisher "microsoft-ads" -Product "windows-data-science-vm" -Name "windows2016" -Terms $agreementTerms -Accept
```

Sortie :

```
Publisher         : microsoft-ads
Product           : windows-data-science-vm
Plan              : windows2016
LicenseTextLink   : https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_MICROSOFT%253a2DADS%253a24WINDOWS%253a2DDATA%253a2DSCIENCE%253a2DV
                    M%253a24WINDOWS2016%253a24OC5SKMQOXSED66BBSNTF4XRCS4XLOHP7QMPV54DQU7JCBZWYFP35IDPOWTUKXUC7ZAG7W6ZMDD6NHWNKUIVSYBZUTZ245F44SU5AD7Q.txt
PrivacyPolicyLink : https://www.microsoft.com/EN-US/privacystatement/OnlineServices/Default.aspx
Signature         : XXXXXXK3MNJ5SROEG2BYDA2YGECU33GXTD3UFPLPC4BAVKAUL3PDYL3KBKBLG4ZCDJZVNSA7KJWTGMDSYDD6KRLV3LV274DLBXXXXXX
Accepted          : True
Signdate          : 2/23/2018 7:49:31 PM
```



## <a name="next-steps"></a>Étapes suivantes

Pour créer rapidement une machine virtuelle avec la cmdlet `New-AzVM` en utilisant des informations d’image de base, consultez [Créer une machine virtuelle Windows avec PowerShell](quick-create-powershell.md).

Pour plus d’informations sur la création d’images personnalisées dans une galerie Shared Image Gallery à l’aide d’images de la Place de marché, consultez [Mention des informations de plan d’achat Azure Marketplace lors de la création d’images](../marketplace-images.md).
