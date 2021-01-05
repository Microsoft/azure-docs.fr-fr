---
title: Configurer des autorisations de service Azure Image Builder à l’aide d’Azure CLI
description: Configurer les exigences pour le service Azure VM Image Builder, y compris les autorisations et les privilèges à l’aide d’Azure CLI
author: cynthn
ms.author: danis
ms.date: 05/06/2020
ms.topic: article
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: f598ed3e2aef2f53b90b213d0bd1b002f56bd30c
ms.sourcegitcommit: 5831eebdecaa68c3e006069b3a00f724bea0875a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94517748"
---
# <a name="configure-azure-image-builder-service-permissions-using-azure-cli"></a>Configurer des autorisations de service Azure Image Builder à l’aide d’Azure CLI

Le service Azure Image Builder requiert la configuration des autorisations et des privilèges avant la génération d’une image. Les sections suivantes détaillent comment configurer les scénarios possibles à l’aide d’Azure CLI.

> [!IMPORTANT]
> Le Générateur d’images Azure est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="register-the-features"></a>Inscrire les fonctionnalités

Tout d’abord, vous devez vous inscrire au service Azure Image Builder. L’inscription octroie l’autorisation de service pour créer, gérer et supprimer un groupe de ressources mise en lots. Le service dispose également des droits nécessaires pour ajouter des ressources au groupe requises pour la génération de l’image.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

## <a name="create-an-azure-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur Azure

Azure Image Builder vous oblige à créer une [identité managée attribuée par l’utilisateur Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md). Azure Image Builder utilise l’identité managée attribuée par l’utilisateur pour lire et écrire des images et accéder aux comptes de stockage Azure. Vous octroyez l’autorisation d’identité pour effectuer des actions spécifiques dans votre abonnement.

> [!NOTE]
> Auparavant, Azure Image Builder utilisait le nom de principal du service (SPN) d’Azure Image Builder pour octroyer des autorisations aux groupes de ressources d’image. L’utilisation du SPN sera déconseillée. Utilisez une identité managée attribuée par l’utilisateur à la place.

L’exemple suivant montre comment créer une identité managée attribuée par l’utilisateur Azure. Remplacez les paramètres d’espaces réservés pour définir vos variables.

| Paramètre | Description |
|---------|-------------|
| \<Resource group\> | Le groupe de ressources dans lequel créer l’identité managée attribuée par l'utilisateur. |

```azurecli-interactive
identityName="aibIdentity"
imageResourceGroup=<Resource group>

az identity create \
    --resource-group $imageResourceGroup \
    --name $identityName
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

```azurecli-interactive
# Subscription ID - You can get this using `az account show | grep id` or from the Azure portal.
subscriptionID=<Subscription ID>
# Resource group - For Preview, image builder will only support creating custom images in the same Resource Group as the source managed image.
imageResourceGroup=<Resource group>
identityName="aibIdentity"

# Use *cURL* to download the a sample JSON description 
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

# Create a unique role name to avoid clashes in the same Azure Active Directory domain
imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')

# Update the JSON definition using stream editor
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json

# Create a custom role from the sample aibRoleImageCreation.json description file.
az role definition create --role-definition ./aibRoleImageCreation.json

# Get the user-assigned managed identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $identityName | grep "clientId" | cut -c16- | tr -d '",')

# Grant the custom role to the user-assigned managed identity for Azure Image Builder.
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```

### <a name="existing-vnet-azure-role-example"></a>Exemple de rôle Azure du réseau virtuel existant

L’exemple suivant crée un rôle Azure pour utiliser et distribuer une image du réseau virtuel existant. Vous octroyez ensuite le rôle personnalisé à l’identité managée attribuée par l’utilisateur pour Azure Image Builder.

Pour simplifier le remplacement de valeurs dans l’exemple, définissez les variables suivantes en premier. Remplacez les paramètres d’espaces réservés pour définir vos variables.

| Paramètre | Description |
|---------|-------------|
| \<Subscription ID\> | L'identifiant de votre abonnement Azure. |
| \<Resource group\> | Groupe de ressources du réseau virtuel |

```azurecli-interactive
# Subscription ID - You can get this using `az account show | grep id` or from the Azure portal.
subscriptionID=<Subscription ID>
VnetResourceGroup=<Resource group>
identityName="aibIdentity"

# Use *cURL* to download the a sample JSON description 
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json -o aibRoleNetworking.json

# Create a unique role name to avoid clashes in the same domain
netRoleDefName="Azure Image Builder Network Def"$(date +'%s')

# Update the JSON definition using stream editor
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleNetworking.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" aibRoleNetworking.json
sed -i -e "s/Azure Image Builder Service Networking Role/$netRoleDefName/g" aibRoleNetworking.json

# Create a custom role from the aibRoleNetworking.json description file.
az role definition create --role-definition ./aibRoleNetworking.json

# Get the user-assigned managed identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $identityName | grep "clientId" | cut -c16- | tr -d '",')

# Grant the custom role to the user-assigned managed identity for Azure Image Builder.
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$VnetResourceGroup
```

## <a name="using-managed-identity-for-azure-storage-access"></a>Utilisation de l’identité managée pour l’accès au Stockage Azure

Si vous voulez authentifier de façon transparente avec le Stockage Azur et utiliser des conteneurs privés, Azure Image Builder a besoin d’une identité managée attribuée par l’utilisateur. Azure Image Builder utilise l’identité pour authentifier avec le Stockage Azure.

> [!NOTE]
> Azure Image Builder n’utilise que l’identité au moment de la soumission du modèle d’image. La machine virtuelle de build n’a pas accès à l’identité pendant la génération de l’image.

Utilisez Azure CLI pour créer l’identité managée attribuée par l’utilisateur.

```azurecli
az role assignment create \
    --assignee <Image Builder client ID> \
    --role "Storage Blob Data Reader" \
    --scope /subscriptions/<Subscription ID>/resourceGroups/<Resource group>/providers/Microsoft.Storage/storageAccounts/$scriptStorageAcc/blobServices/default/containers/<Storage account container>
```

Dans le modèle Image Builder, vous devez fournir l’identité managée attribuée par l’utilisateur.

```json
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "apiVersion": "2019-05-01-preview",
    "location": "<Region>",
    ..
    "identity": {
    "type": "UserAssigned",
          "userAssignedIdentities": {
            "<Image Builder ID>": {}     
        }
```

Remplacez les paramètres d’espaces réservés suivants :

| Paramètre | Description |
|---------|-------------|
| \<Region\> | Région du modèle |
| \<Resource group\> | Resource group |
| \<Storage account container\> | Nom du conteneur de compte de stockage |
| \<Subscription ID\> | Abonnement Azure |

Pour plus d’informations sur l’utilisation d’une identité managée attribuée par l’utilisateur, consultez la rubrique [Créer une image personnalisée qui utilisera une identité managée attribuée par l’utilisateur Azure pour accéder de façon transparente aux fichiers du Stockage Azure](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts/7_Creating_Custom_Image_using_MSI_to_Access_Storage#create-a-custom-image-that-will-use-an-azure-user-assigned-managed-identity-to-seemlessly-access-files-azure-storage). Le Guide de démarrage rapide vous guide dans la création et la configuration de l’identité managée attribuée par l’utilisateur pour accéder à un compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Vue d’ensemble d’Azure Image Builder](image-builder-overview.md).