---
title: Utiliser Azure Image Builder pour les machines virtuelles Linux autorisant l’accès à un réseau virtuel Azure existant (préversion)
description: Créer des images de machine virtuelle Linux avec Azure Image Builder autorisant l’accès à un réseau virtuel Azure existant
author: danielsollondon
ms.author: danis
ms.date: 08/10/2020
ms.topic: how-to
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.reviewer: danis
ms.openlocfilehash: cbff2358dcf8685298e31f18c430c35ec7de4948
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96437335"
---
# <a name="use-azure-image-builder-for-linux-vms-allowing-access-to-an-existing-azure-vnet"></a>Utiliser Azure Image Builder pour les machines virtuelles Linux autorisant l’accès à un réseau virtuel Azure existant

Cet article explique comment utiliser Azure Image Builder pour créer une image Linux personnalisée de base qui a accès aux ressources existantes sur un réseau virtuel. La machine virtuelle de build créée est déployée vers un réseau virtuel nouveau ou existant spécifié dans votre abonnement. Lorsque vous utilisez un réseau virtuel Azure existant, le service Azure Image Builder ne nécessite pas de connectivité de réseau public.

> [!IMPORTANT]
> Le Générateur d’images Azure est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="register-the-features"></a>Inscrire les fonctionnalités

Tout d’abord, vous devez vous inscrire au service Azure Image Builder. L’inscription octroie l’autorisation de service pour créer, gérer et supprimer un groupe de ressources mise en lots. Le service dispose également des droits nécessaires pour ajouter des ressources au groupe requises pour la génération de l’image.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

## <a name="set-variables-and-permissions"></a>Définir des variables et des autorisations 

Vous utiliserez certains types d’informations de façon répétée. Créez des variables pour stocker ces informations.

```azurecli-interactive
# set your environment variables here!!!!

# destination image resource group
imageResourceGroup=aibImageRG01

# location (see possible locations in main docs)
location=WestUS2

# your subscription
# get the current subID : 'az account show | grep id'
subscriptionID=$(az account show | grep id | tr -d '",' | cut -c7-)

# name of the image to be created
imageName=aibCustomLinuxImg01

# image distribution metadata reference name
runOutputName=aibCustLinManImg01ro


# VNET properties (update to match your existing VNET, or leave as-is for demo)
# VNET name
vnetName=myexistingvnet01
# subnet name
subnetName=subnet01
# VNET resource group name
# NOTE! The VNET must always be in the same region as the AIB service region.
vnetRgName=existingVnetRG
# Existing Subnet NSG Name or the demo will create it
nsgName=aibdemoNsg
```

Créez le groupe de ressources.

```azurecli-interactive
az group create -n $imageResourceGroup -l $location
```

## <a name="configure-networking"></a>Configurer la mise en réseau

Si vous n’avez pas de réseau virtuel\sous-réseau\groupe de sécurité réseau existant, utilisez le script suivant pour en créer un.

```bash

# Create a resource group

az group create -n $vnetRgName -l $location

# Create VNET

az network vnet create \
    --resource-group $vnetRgName \
    --name $vnetName --address-prefix 10.0.0.0/16 \
    --subnet-name $subnetName --subnet-prefix 10.0.0.0/24

# Create base NSG to simulate an existing NSG

az network nsg create -g $vnetRgName -n $nsgName

az network vnet subnet update \
    --resource-group $vnetRgName \
    --vnet-name $vnetName \
    --name $subnetName \
    --network-security-group $nsgName
    
#  NOTE! The VNET must always be in the same region as the Azure Image Builder service region.
```

### <a name="add-network-security-group-rule"></a>Ajouter une règle de groupe de sécurité réseau

Cette règle autorise la connectivité de l’équilibreur de charge Azure Image Builder à la machine virtuelle proxy. Le port 60001 est utilisé pour les systèmes d’exploitation Linux et le port 60000 pour les systèmes d’exploitation Windows. La machine virtuelle proxy se connecte à la machine virtuelle de build à l’aide du port 22 pour les systèmes d’exploitation Linux ou 5986 pour les systèmes d’exploitation Windows.

```azurecli-interactive
az network nsg rule create \
    --resource-group $vnetRgName \
    --nsg-name $nsgName \
    -n AzureImageBuilderNsgRule \
    --priority 400 \
    --source-address-prefixes AzureLoadBalancer \
    --destination-address-prefixes VirtualNetwork \
    --destination-port-ranges 60000-60001 --direction inbound \
    --access Allow --protocol Tcp \
    --description "Allow Image Builder Private Link Access to Proxy VM"
```

### <a name="disable-private-service-policy-on-subnet"></a>Désactiver la stratégie de service privé sur le sous-réseau

```azurecli-interactive
az network vnet subnet update \
  --name $subnetName \
  --resource-group $vnetRgName \
  --vnet-name $vnetName \
  --disable-private-link-service-network-policies true 
```

Pour plus d’informations sur la mise en réseau d’Image Builder, consultez [Options de mise en réseau du service Azure Image Builder](image-builder-networking.md).

## <a name="modify-the-example-template-and-create-role"></a>Modifier l’exemple de modèle et créer un rôle

```bash
# download the example and configure it with your vars

curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/1a_Creating_a_Custom_Linux_Image_on_Existing_VNET/existingVNETLinux.json -o existingVNETLinux.json
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json -o aibRoleNetworking.json
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" existingVNETLinux.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" existingVNETLinux.json
sed -i -e "s/<region>/$location/g" existingVNETLinux.json
sed -i -e "s/<imageName>/$imageName/g" existingVNETLinux.json
sed -i -e "s/<runOutputName>/$runOutputName/g" existingVNETLinux.json

sed -i -e "s/<vnetName>/$vnetName/g" existingVNETLinux.json
sed -i -e "s/<subnetName>/$subnetName/g" existingVNETLinux.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" existingVNETLinux.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleNetworking.json
sed -i -e "s/<vnetRgName>/$vnetRgName/g" aibRoleNetworking.json

```

## <a name="set-permissions-on-the-resource-group"></a>Définir des autorisations sur le groupe de ressources

Image Builder utilise l’[identité managée affectée par l’utilisateur](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) fournie pour injecter l’image dans Azure Shared Image Gallery (SIG). Dans cet exemple, vous allez créer une définition de rôle Azure qui dispose des actions granulaires pour distribuer l’image à la galerie SIG. La définition de rôle sera ensuite attribuée à l’identité de l’utilisateur.

```bash
# create user assigned identity for image builder
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $imageResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# update the template
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" existingVNETLinux.json

# make role name unique, to avoid clashes in the same Azure Active Directory domain
imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')
netRoleDefName="Azure Image Builder Network Def"$(date +'%s')

# update the definitions
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Networking Role/$netRoleDefName/g" aibRoleNetworking.json
```

Plutôt que d’octroyer à Image Builder un niveau de granularité inférieur et des privilèges accrus, vous pouvez créer deux rôles. L’un d’eux donne au générateur des autorisations pour créer une image, l’autre l’autorise à connecter la machine virtuelle de build et l’équilibreur de charge à votre réseau virtuel.

```bash
# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json
az role definition create --role-definition ./aibRoleNetworking.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role assignment create \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$vnetRgName
```

Pour plus d’informations sur les autorisations, consultez [Configurer les autorisations du service Azure Image Builder à l’aide d’Azure CLI](image-builder-permissions-cli.md) ou [Configurer les autorisations du service Azure Image Builder à l’aide de PowerShell](image-builder-permissions-powershell.md).

## <a name="create-the-image"></a>Création de l’image

Envoyez la configuration de l’image au service Générateur d’images Azure.

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @existingVNETLinux.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n existingVNETLinuxTemplate01

# Wait approximately 1-3 mins (validation, permissions etc.)
```

Lancez la génération de l’image.

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n existingVNETLinuxTemplate01 \
     --action Run 

# Wait approximately 15 mins
```

La création de l’image et sa réplication dans les deux régions peuvent prendre un certain temps. Attendez la fin de cette partie pour passer à la création d’une machine virtuelle.


## <a name="create-the-vm"></a>Création de la machine virtuelle

Créez une machine virtuelle à partir de la version de l’image créée par le Générateur d’images Azure.

```azurecli-interactive
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgVm0001 \
  --admin-username aibuser \
  --image $imageName \
  --location $location \
  --generate-ssh-keys
```

Connectez-vous avec SSH à la machine virtuelle.

```bash
ssh aibuser@<publicIpAddress>
```

L’image est personnalisée avec un *Message du jour* dès que la connexion SSH est établie.

```console
*******************************************************
**            This VM was built from the:            **
**      !! AZURE VM IMAGE BUILDER Custom Image !!    **
**         You have just been Customized :-)         **
*******************************************************
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez à présent essayer de personnaliser à nouveau la version de l’image pour créer une nouvelle version de la même image, ignorez les étapes suivantes et passez à [Utiliser Azure Image Builder pour créer une autre version de l’image](image-builder-gallery-update-image-version.md).


La procédure suivante permet de supprimer l’image créée ainsi que tous les autres fichiers de ressources. Terminez ce déploiement avant de supprimer les ressources.

En ce qui concerne la suppression des ressources de la bibliothèque d’images, il est nécessaire de supprimer toutes les versions de l’image pour pouvoir supprimer la définition de l’image utilisée pour les créer. Supprimer une galerie implique de supprimer au préalable toutes les définitions de l’image qu’elle comporte.

Supprimez le modèle du Générateur d’images.

```azurecli
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n existingVNETLinuxTemplate01
```

Supprimer des affectations d’autorisations, des rôles et des identités
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role assignment delete \
    --assignee $imgBuilderCliId \
    --role $netRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$vnetRgName


az role definition delete --name "$imageRoleDefName"
az role definition delete --name "$netRoleDefName"

az identity delete --ids $imgBuilderId
```

Supprimez le groupe de ressources.

```azurecli-interactive
az group delete -n $imageResourceGroup
```

Si vous avez créé un réseau virtuel pour ce démarrage rapide, vous pouvez supprimer le réseau virtuel s’il n’est plus utilisé.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [galeries d’images partagées Azure](shared-image-galleries.md).