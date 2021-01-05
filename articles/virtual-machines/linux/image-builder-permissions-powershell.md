---
title: Configurer des autorisations de service Azure Image Builder à l’aide de PowerShell
description: Configurer les exigences pour le service Azure VM Image Builder, y compris les autorisations et les privilèges à l’aide de PowerShell
author: danielsollondon
ms.author: danis
ms.date: 05/06/2020
ms.topic: article
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: cfe3efc77e065ac3685b72d0eab501034609b59b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88067982"
---
# <a name="configure-azure-image-builder-service-permissions-using-powershell"></a>Configurer des autorisations de service Azure Image Builder à l’aide de PowerShell

Le service Azure Image Builder requiert la configuration des autorisations et des privilèges avant la génération d’une image. Les sections suivantes détaillent comment configurer les scénarios possibles à l’aide de PowerShell.

> [!IMPORTANT]
> Le Générateur d’images Azure est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="register-the-features"></a>Inscrire les fonctionnalités

Tout d’abord, vous devez vous inscrire au service Azure Image Builder. L’inscription octroie l’autorisation de service pour créer, gérer et supprimer un groupe de ressources mise en lots. Le service dispose également des droits nécessaires pour ajouter des ressources au groupe requises pour la génération de l’image.

```powershell-interactive
Register-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages
```

## <a name="create-an-azure-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur Azure

Azure Image Builder vous oblige à créer une [identité managée attribuée par l’utilisateur Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md). Azure Image Builder utilise l’identité managée attribuée par l’utilisateur pour lire et écrire des images et accéder aux comptes de stockage Azure. Vous octroyez l’autorisation d’identité pour effectuer des actions spécifiques dans votre abonnement.

> [!NOTE]
> Auparavant, Azure Image Builder utilisait le nom de principal du service (SPN) d’Azure Image Builder pour octroyer des autorisations aux groupes de ressources d’image. L’utilisation du SPN sera déconseillée. Utilisez une identité managée attribuée par l’utilisateur à la place.

L’exemple suivant montre comment créer une identité managée attribuée par l’utilisateur Azure. Remplacez les paramètres d’espaces réservés pour définir vos variables.

| Paramètre | Description |
|---------|-------------|
| \<Resource group\> | Le groupe de ressources dans lequel créer l’identité managée attribuée par l'utilisateur. |

```powershell-interactive
## Add AZ PS module to support AzUserAssignedIdentity
Install-Module -Name Az.ManagedServiceIdentity

$parameters = @{
    Name = 'aibIdentity'
    ResourceGroupName = '<Resource group>'
}
# create identity
New-AzUserAssignedIdentity @parameters
```

Pour plus d’informations sur les identités attribuées par l’utilisateur Azure, consultez la documentation [Identité managée attribuée Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md) sur la création d’une identité.

## <a name="allow-image-builder-to-distribute-images"></a>Autoriser Image Builder à distribuer des images

Pour que Azure Image Builder distribue des images (images managées/Shared Image Gallery), le service Azure Image Builder doit être autorisé à injecter les images dans ces groupes de ressources. Pour octroyer les autorisations requises, vous devez créer une identité managée attribuée par l’utilisateur et lui octroyer des droits sur le groupe de ressources dans lequel l’image est générée. Azure image Builder ne dispose **pas** d’autorisation d’accès aux ressources d’autres groupes de ressources dans l’abonnement. Vous devez effectuer des actions explicites pour permettre l’accès afin d’éviter l’échec de vos builds.

Vous n’avez pas besoin d’octroyer au contributeur d’identités managées attribuées par l’utilisateur des droits sur le groupe de ressources pour distribuer des images. Toutefois, l’identité managée attribuée par l’utilisateur a besoin des autorisations `Actions` Azure suivantes dans le groupe de ressources de distribution :

```Actions
Microsoft.Compute/images/write
Microsoft.Compute/images/read
Microsoft.Compute/images/delete
```

Si vous distribuez à une galerie d’images partagées, vous avez également besoin des éléments suivants :

```Actions
Microsoft.Compute/galleries/read
Microsoft.Compute/galleries/images/read
Microsoft.Compute/galleries/images/versions/read
Microsoft.Compute/galleries/images/versions/write
```

## <a name="permission-to-customize-existing-images"></a>Autorisation de personnaliser des images existantes

Pour qu’Azure Image Builder génère des images à partir d’images personnalisées sources (Images managées / Shared Image Gallery) le service Azure Image Builder doit être autorisé à lire les images dans ces groupes de ressources. Pour octroyer les autorisations requises, vous devez créer une identité managée attribuée par l’utilisateur et lui octroyer des droits sur le groupe de ressources dans lequel se trouve l’image.

Générer à partir d’une image personnalisée existante :

```Actions
Microsoft.Compute/galleries/read
```

Générer à partir d’une version Shared Image Gallery existante :

```Actions
Microsoft.Compute/galleries/read
Microsoft.Compute/galleries/images/read
Microsoft.Compute/galleries/images/versions/read
```

## <a name="permission-to-customize-images-on-your-vnets"></a>Autorisation de personnaliser des images sur vos réseaux virtuels

Azure Image Builder a la capacité de déployer et d’utiliser un réseau virtuel existant dans votre abonnement, ce qui permet aux personnalisations d’accéder aux ressources connectées.

Vous n’avez pas besoin d’octroyer au contributeur d’identités managées attribuées par l’utilisateur des droits sur le groupe de ressources pour déployer une machine virtuelle sur le réseau virtuel existant. Toutefois, l’identité managée attribuée par l’utilisateur a besoin des autorisations `Actions` Azure suivantes dans le groupe de ressources du réseau virtuel :

```Actions
Microsoft.Network/virtualNetworks/read
Microsoft.Network/virtualNetworks/subnets/join/action
```

## <a name="create-an-azure-role-definition"></a>Créer une définition de rôle Azure

Les exemples suivants créent une définition de rôle Azure à partir des actions décrites dans les sections précédentes. Les exemples sont appliqués au niveau du groupe de ressources. Évaluez et testez si les exemples sont suffisamment granulaires pour répondre à vos exigences. Pour votre scénario, vous devrez peut-être l’affiner dans une galerie d’images partagées spécifique.

Les actions d’image autorisent la lecture et l’écriture. Décidez des éléments appropriés pour votre environnement. Par exemple, créez un rôle pour permettre à Azure Image Builder de lire des images à partir du groupe de ressources *exemple-rg-1* et d’écrire des images dans le groupe de ressources *exemple-rg-2*.

### <a name="custom-image-azure-role-example"></a>Exemple de rôle d’image personnalisée Azure

L’exemple suivant crée un rôle Azure pour utiliser et distribuer une image personnalisée source. Vous octroyez ensuite le rôle personnalisé à l’identité managée attribuée par l’utilisateur pour Azure Image Builder.

Pour simplifier le remplacement de valeurs dans l’exemple, définissez les variables suivantes en premier. Remplacez les paramètres d’espaces réservés pour définir vos variables.

| Paramètre | Description |
|---------|-------------|
| \<Subscription ID\> | L'identifiant de votre abonnement Azure. |
| \<Resource group\> | Groupes de ressources pour l’image personnalisée |

```powershell-interactive
$sub_id = "<Subscription ID>"
# Resource group - For Preview, image builder will only support creating custom images in the same Resource Group as the source managed image.
$imageResourceGroup = "<Resource group>"
$identityName = "aibIdentity"

# Use a web request to download the sample JSON description
$sample_uri="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json"
$role_definition="aibRoleImageCreation.json"

Invoke-WebRequest -Uri $sample_uri -Outfile $role_definition -UseBasicParsing

# Create a unique role name to avoid clashes in the same Azure Active Directory domain
$timeInt=$(get-date -UFormat "%s")
$imageRoleDefName="Azure Image Builder Image Def"+$timeInt

# Update the JSON definition placeholders with variable values
((Get-Content -path $role_definition -Raw) -replace '<subscriptionID>',$sub_id) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace '<rgName>', $imageResourceGroup) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $role_definition

# Create a custom role from the aibRoleImageCreation.json description file. 
New-AzRoleDefinition -InputFile $role_definition

# Get the user-identity properties
$identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId

# Grant the custom role to the user-assigned managed identity for Azure Image Builder.
$parameters = @{
    ObjectId = $identityNamePrincipalId
    RoleDefinitionName = $imageRoleDefName
    Scope = '/subscriptions/' + $sub_id + '/resourceGroups/' + $imageResourceGroup
}

New-AzRoleAssignment @parameters
```

### <a name="existing-vnet-azure-role-example"></a>Exemple de rôle Azure du réseau virtuel existant

L’exemple suivant crée un rôle Azure pour utiliser et distribuer une image du réseau virtuel existant. Vous octroyez ensuite le rôle personnalisé à l’identité managée attribuée par l’utilisateur pour Azure Image Builder.

Pour simplifier le remplacement de valeurs dans l’exemple, définissez les variables suivantes en premier. Remplacez les paramètres d’espaces réservés pour définir vos variables.

| Paramètre | Description |
|---------|-------------|
| \<Subscription ID\> | L'identifiant de votre abonnement Azure. |
| \<Resource group\> | Groupe de ressources du réseau virtuel |

```powershell-interactive
$sub_id = "<Subscription ID>"
$res_group = "<Resource group>"
$identityName = "aibIdentity"

# Use a web request to download the sample JSON description
$sample_uri="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json"
$role_definition="aibRoleNetworking.json"

Invoke-WebRequest -Uri $sample_uri -Outfile $role_definition -UseBasicParsing

# Create a unique role name to avoid clashes in the same AAD domain
$timeInt=$(get-date -UFormat "%s")
$networkRoleDefName="Azure Image Builder Network Def"+$timeInt

# Update the JSON definition placeholders with variable values
((Get-Content -path $role_definition -Raw) -replace '<subscriptionID>',$sub_id) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace '<vnetRgName>', $res_group) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace 'Azure Image Builder Service Networking Role',$networkRoleDefName) | Set-Content -Path $role_definition

# Create a custom role from the aibRoleNetworking.json description file
New-AzRoleDefinition -InputFile $role_definition

# Get the user-identity properties
$identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId

# Assign the custom role to the user-assigned managed identity for Azure Image Builder
$parameters = @{
    ObjectId = $identityNamePrincipalId
    RoleDefinitionName = $networkRoleDefName
    Scope = '/subscriptions/' + $sub_id + '/resourceGroups/' + $res_group
}

New-AzRoleAssignment @parameters
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Vue d’ensemble d’Azure Image Builder](image-builder-overview.md).