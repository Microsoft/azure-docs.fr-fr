---
title: Créer une machine virtuelle Windows avec le Générateur d’images Azure avec PowerShell
description: Créer une machine virtuelle Windows avec le module PowerShell de Générateur d’images Azure.
author: cynthn
ms.author: cynthn
ms.date: 06/17/2020
ms.topic: how-to
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a221ba8fe14db37729183774197bfc2db8bf2baa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91328103"
---
# <a name="preview-create-a-windows-vm-with-azure-image-builder-using-powershell"></a>Aperçu : Créer une machine virtuelle Windows avec le Générateur d’images Azure avec PowerShell

Cet article vous montre comment créer une image Windows personnalisée à l’aide du module PowerShell du Générateur d’images de machine virtuelle Azure.

> [!CAUTION]
> Le Générateur d’images Azure est actuellement en préversion publique. Cette préversion est fournie sans contrat de niveau de service. Il n’est pas recommandé pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Tandis que les modules PowerShell **Az.ImageBuilder** et **Az.ManagedServiceIdentity** sont en préversion, vous devez les installer séparément à l’aide de l’applet de commande `Install-Module` et du paramètre `AllowPrerelease`. Une fois ces modules PowerShell disponibles, ils deviennent partie intégrante des versions futures du module Az PowerShell et disponibles en mode natif dans Azure Cloud Shell.

```azurepowershell-interactive
'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
```

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Si vous avez plusieurs abonnements Azure, sélectionnez l’abonnement approprié dans lequel les ressources doivent être facturées. Sélectionnez un abonnement spécifique avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

### <a name="register-features"></a>Inscrire des fonctionnalités

S’il s’agit de votre première utilisation du Générateur d’images Azure au cours de la préversion, inscrivez la nouvelle fonctionnalité **VirtualMachineTemplatePreview**.

```azurepowershell-interactive
Register-AzProviderFeature -ProviderNamespace Microsoft.VirtualMachineImages -FeatureName VirtualMachineTemplatePreview
```

Vérifiez l’état d’inscription de la fonctionnalité.

> [!NOTE]
> **RegistrationState** peut être à l’état `Registering` pendant plusieurs minutes avant de passer à `Registered`. Avant de continuer, attendez que l’état soit **Inscrit**.

```azurepowershell-interactive
Get-AzProviderFeature -ProviderNamespace Microsoft.VirtualMachineImages -FeatureName VirtualMachineTemplatePreview
```

Inscrivez les fournisseurs de ressources suivants avec votre abonnement Azure s’ils ne sont pas déjà inscrits.

- Microsoft.Compute
- Microsoft.KeyVault
- Microsoft.Storage
- Microsoft.VirtualMachineImages

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute, Microsoft.KeyVault, Microsoft.Storage, Microsoft.VirtualMachineImages |
  Where-Object RegistrationState -ne Registered |
    Register-AzResourceProvider
```

## <a name="define-variables"></a>Définir des variables

Vous utiliserez plusieurs types d’informations de façon répétée. Créez des variables pour stocker les informations.

```azurepowershell-interactive
# Destination image resource group name
$imageResourceGroup = 'myWinImgBuilderRG'

# Azure region
$location = 'WestUS2'

# Name of the image to be created
$imageTemplateName = 'myWinImage'

# Distribution properties of the managed image upon completion
$runOutputName = 'myDistResults'
```

Créez une variable pour votre ID d’abonnement Azure. Pour confirmer que la variable `subscriptionID` contient votre ID d’abonnement, vous pouvez exécuter la deuxième ligne de l’exemple suivant.

```azurepowershell-interactive
# Your Azure Subscription ID
$subscriptionID = (Get-AzContext).Subscription.Id
Write-Output $subscriptionID
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../../azure-resource-manager/management/overview.md) avec l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

L’exemple suivant crée un groupe de ressources en fonction du nom de la variable `$imageResourceGroup` dans la région spécifiée de la variable `$location`. Ce groupe de ressources sert à stocker l’artefact de modèle de configuration d’image et l’image.

```azurepowershell-interactive
New-AzResourceGroup -Name $imageResourceGroup -Location $location
```

## <a name="create-user-identity-and-set-role-permissions"></a>Créer une identité d’utilisateur et définir des autorisations de rôle

Accordez des autorisations au Générateur d’images Azure pour créer des images dans le groupe de ressources spécifié à l’aide de l’exemple suivant. Sans cette autorisation, le processus de génération de l’image ne se termine pas correctement.

Créer des variables pour les noms de définition de rôle et d’identité. Ces valeurs doivent être uniques.

```azurepowershell-interactive
[int]$timeInt = $(Get-Date -UFormat '%s')
$imageRoleDefName = "Azure Image Builder Image Def $timeInt"
$identityName = "myIdentity$timeInt"
```

Créez une identité d’utilisateur.

```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName
```

Stockez les ressources d’identité et les ID du principal dans les variables.

```azurepowershell-interactive
$identityNameResourceId = (Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId = (Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
```

### <a name="assign-permissions-for-identity-to-distribute-images"></a>Attribuer des autorisations d’identité pour distribuer des images

Téléchargez le fichier de configuration .json et modifiez-le en fonction des paramètres définis dans cet article.

```azurepowershell-interactive
$myRoleImageCreationUrl = 'https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json'
$myRoleImageCreationPath = "$env:TEMP\myRoleImageCreation.json"

Invoke-WebRequest -Uri $myRoleImageCreationUrl -OutFile $myRoleImageCreationPath -UseBasicParsing

$Content = Get-Content -Path $myRoleImageCreationPath -Raw
$Content = $Content -replace '<subscriptionID>', $subscriptionID
$Content = $Content -replace '<rgName>', $imageResourceGroup
$Content = $Content -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName
$Content | Out-File -FilePath $myRoleImageCreationPath -Force
```

Créez la définition de rôle.

```azurepowershell-interactive
New-AzRoleDefinition -InputFile $myRoleImageCreationPath
```

Accordez la définition de rôle au principal du Générateur d’images.

```azurepowershell-interactive
$RoleAssignParams = @{
  ObjectId = $identityNamePrincipalId
  RoleDefinitionName = $imageRoleDefName
  Scope = "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
}
New-AzRoleAssignment @RoleAssignParams
```

> [!NOTE]
> Si vous recevez le message d’erreur : « _New-AzRoleDefinition : Limite de définitions de rôle dépassée. Aucune autre définition de rôle ne peut être créée._  », consultez [résoudre les problèmes DANS Azure RBAC](../../role-based-access-control/troubleshooting.md).

## <a name="create-a-shared-image-gallery"></a>Créer une galerie Shared Image Gallery

Créez la galerie.

```azurepowershell-interactive
$myGalleryName = 'myImageGallery'
$imageDefName = 'winSvrImages'

New-AzGallery -GalleryName $myGalleryName -ResourceGroupName $imageResourceGroup -Location $location
```

Créez une définition de galerie.

```azurepowershell-interactive
$GalleryParams = @{
  GalleryName = $myGalleryName
  ResourceGroupName = $imageResourceGroup
  Location = $location
  Name = $imageDefName
  OsState = 'generalized'
  OsType = 'Windows'
  Publisher = 'myCo'
  Offer = 'Windows'
  Sku = 'Win2019'
}
New-AzGalleryImageDefinition @GalleryParams
```

## <a name="create-an-image"></a>Créer une image

Créez un objet source pour le Générateur d’images Azure. Pour connaître les valeurs de paramètres, consultez [Rechercher des images de machine virtuelle Windows sur la Place de marché Microsoft Azure avec Azure PowerShell](./cli-ps-findimage.md).

```azurepowershell-interactive
$SrcObjParams = @{
  SourceTypePlatformImage = $true
  Publisher = 'MicrosoftWindowsServer'
  Offer = 'WindowsServer'
  Sku = '2019-Datacenter'
  Version = 'latest'
}
$srcPlatform = New-AzImageBuilderSourceObject @SrcObjParams
```

Créez un objet de distribution pour le Générateur d’images Azure.

```azurepowershell-interactive
$disObjParams = @{
  SharedImageDistributor = $true
  ArtifactTag = @{tag='dis-share'}
  GalleryImageId = "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup/providers/Microsoft.Compute/galleries/$myGalleryName/images/$imageDefName"
  ReplicationRegion = $location
  RunOutputName = $runOutputName
  ExcludeFromLatest = $false
}
$disSharedImg = New-AzImageBuilderDistributorObject @disObjParams
```

Créez un objet de personnalisation pour le Générateur d’images Azure.

```azurepowershell-interactive
$ImgCustomParams = @{
  PowerShellCustomizer = $true
  CustomizerName = 'settingUpMgmtAgtPath'
  RunElevated = $false
  Inline = @("mkdir c:\\buildActions", "echo Azure-Image-Builder-Was-Here  > c:\\buildActions\\buildActionsOutput.txt")
}
$Customizer = New-AzImageBuilderCustomizerObject @ImgCustomParams
```

Créez un modèle Azure pour le Générateur d’images Azure.

```azurepowershell-interactive
$ImgTemplateParams = @{
  ImageTemplateName = $imageTemplateName
  ResourceGroupName = $imageResourceGroup
  Source = $srcPlatform
  Distribute = $disSharedImg
  Customize = $Customizer
  Location = $location
  UserAssignedIdentityId = $identityNameResourceId
}
New-AzImageBuilderTemplate @ImgTemplateParams
```

Une fois l’exécution terminée, un message est retourné et un modèle de configuration pour le Générateur d’images Azure est créé dans `$imageResourceGroup`.

Pour déterminer si le processus de création du modèle a réussi, vous pouvez utiliser l’exemple suivant.

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup |
  Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState
```

En outre, en arrière-plan, le générateur d’images crée un groupe de ressources de mise en lots dans votre abonnement. Ce groupe de ressources est utilisé pour générer l’image. Il est au format : `IT_<DestinationResourceGroup>_<TemplateName>`.

> [!WARNING]
> Ne supprimez pas directement le groupe de ressources de mise en lots. Supprimez l’artefact de modèle d’image ; cela supprimera le groupe de ressources de mise en lots.

Si le service signale un échec lors de la soumission du modèle de configuration d’image,

- Consultez [Résolution des problèmes liés aux défaillances de création d’image de machine virtuelle Azure](../linux/image-builder-troubleshoot.md).
- Supprimez le modèle à l’aide de l’exemple suivant avant de réessayer.

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup
```

## <a name="start-the-image-build"></a>Lancer la génération de l’image

Envoyez la configuration de l’image au service Générateur d’images de la machine virtuelle.

```azurepowershell-interactive
Start-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName
```

Attendez la fin du processus de génération de l’image. Cette étape peut prendre une heure.

Si vous rencontrez des problèmes, consultez [Résolution des problèmes liés aux défaillances de création d’image de machine virtuelle Azure](../linux/image-builder-troubleshoot.md).

## <a name="create-a-vm"></a>Créer une machine virtuelle

Stockez les informations d’identification de connexion pour la machine virtuelle dans une variable. Le mot de passe doit être complexe.

```azurepowershell-interactive
$Cred = Get-Credential
```

Créez la machine virtuelle avec l’image que vous avez créée.

```azurepowershell-interactive
$ArtifactId = (Get-AzImageBuilderRunOutput -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup).ArtifactId

New-AzVM -ResourceGroupName $imageResourceGroup -Image $ArtifactId -Name myWinVM01 -Credential $Cred
```

## <a name="verify-the-customization"></a>Vérifier la personnalisation

Créez une connexion Bureau à distance à la machine virtuelle avec le nom d’utilisateur et le mot de passe définis lors de la création de la machine virtuelle. À l’intérieur de la machine virtuelle, ouvrez PowerShell et exécutez `Get-Content` comme indiqué dans l’exemple suivant :

```azurepowershell-interactive
Get-Content -Path C:\buildActions\buildActionsOutput.txt
```

Vous devez voir la sortie en fonction du contenu du fichier créé pendant le processus de personnalisation de l’image.

```Output
Azure-Image-Builder-Was-Here
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin des ressources que vous avez créées dans cet article, vous pouvez les supprimer en exécutant l’exemple suivant.

### <a name="delete-the-image-builder-template"></a>Supprimer le modèle de générateur d’images

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName
```

### <a name="delete-the-image-resource-group"></a>Supprimer le groupe de ressources d’image

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de cet article existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $imageResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les composants du fichier .json utilisé dans cet article, voir [Référence des modèles du Générateur d’images](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
