---
title: Créer une machine virtuelle Windows avec le Générateur d’images Azure (préversion)
description: Créer une machine virtuelle Windows avec le Générateur d’images Azure.
author: cynthn
ms.author: cynthn
ms.date: 05/05/2020
ms.topic: how-to
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.openlocfilehash: 62d80426dec6f5d63d8fa5d67d64d6aafb881110
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91320011"
---
# <a name="preview-create-a-windows-vm-with-azure-image-builder"></a>Aperçu : Créer une machine virtuelle Windows avec le Générateur d’images Azure

Cet article vous montre comment créer une image Windows personnalisée à l’aide du Générateur d’images de machine virtuelle Azure. L’exemple de cet article utilise des [personnalisateurs](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json#properties-customize) pour la personnalisation de l’image :
- PowerShell (ScriptUri) – télécharger et exécuter un [script PowerShell](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/testPsScript.ps1).
- Redémarrage de Windows : redémarre la machine virtuelle.
- PowerShell (inline) : exécute des commandes spécifiques. Dans cet exemple, il crée un répertoire sur la machine virtuelle à l’aide de `mkdir c:\\buildActions`.
- Fichier : copier un fichier de GitHub vers la machine virtuelle. Cet exemple copie [index.md](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/exampleArtifacts/buildArtifacts/index.html) vers `c:\buildArtifacts\index.html` sur la machine virtuelle.
- buildTimeoutInMinutes : augmenter le temps de génération pour permettre l’exécution de builds plus longues. La valeur par défaut est de 240 minutes, et vous pouvez augmenter le temps de génération pour permettre l’exécution de builds plus longues.
- vmProfile : spécifier les propriétés vmSize et Network.
- osDiskSizeGB : augmenter la taille de l’image.
- identity : fournir une identité pour qu’Azure Image Builder l’utilise pendant la génération.


Vous pouvez également spécifier un `buildTimeoutInMinutes`. La valeur par défaut est de 240 minutes, et vous pouvez augmenter le temps de génération pour permettre l’exécution de builds plus longues.

Pour configurer l’image, nous allons utiliser un exemple de modèle .json. Le fichier .json que nous utilisons est : [helloImageTemplateWin.json](https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Windows_Managed_Image/helloImageTemplateWin.json). 


> [!IMPORTANT]
> Le Générateur d’images Azure est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="register-the-features"></a>Inscrire les fonctionnalités

Pour utiliser le Générateur d’images Azure pendant la préversion, vous devez inscrire la nouvelle fonctionnalité.

```azurecli-interactive
az feature register --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview
```

Vérifiez l’état d’inscription de la fonctionnalité.

```azurecli-interactive
az feature show --namespace Microsoft.VirtualMachineImages --name VirtualMachineTemplatePreview | grep state
```

Vérifiez votre inscription.

```azurecli-interactive
az provider show -n Microsoft.VirtualMachineImages | grep registrationState
az provider show -n Microsoft.KeyVault | grep registrationState
az provider show -n Microsoft.Compute | grep registrationState
az provider show -n Microsoft.Storage | grep registrationState
```

Si elle n’est pas inscrite, exécutez la commande suivante :

```azurecli-interactive
az provider register -n Microsoft.VirtualMachineImages
az provider register -n Microsoft.Compute
az provider register -n Microsoft.KeyVault
az provider register -n Microsoft.Storage
```


## <a name="set-variables"></a>Définition des variables

Nous allons utiliser certains éléments d’information à plusieurs reprises, donc nous allons créer des variables pour les stocker.


```azurecli-interactive
# Resource group name - we are using myImageBuilderRG in this example
imageResourceGroup=myWinImgBuilderRG
# Region location 
location=WestUS2
# Name for the image 
imageName=myWinBuilderImage
# Run output name
runOutputName=aibWindows
# name of the image to be created
imageName=aibWinImage
```

Créez une variable pour votre ID d’abonnement. Vous pouvez l’obtenir avec `az account show | grep id`.

```azurecli-interactive
subscriptionID=<Your subscription ID>
```
## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Ce groupe de ressources sert à stocker l’artefact de modèle de configuration d’image et l’image.


```azurecli-interactive
az group create -n $imageResourceGroup -l $location
```

## <a name="create-a-user-assigned-identity-and-set-permissions-on-the-resource-group"></a>Créer une identité affectée par l’utilisateur et définir des autorisations sur le groupe de ressources
Image Builder utilise l’[identité de l’utilisateur](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md#user-assigned-managed-identity) fourni pour injecter l’image dans le groupe de ressources. Dans cet exemple, vous allez créer une définition de rôle Azure qui dispose des actions granulaires pour distribuer l’image. La définition de rôle sera ensuite attribuée à l’identité de l’utilisateur.

## <a name="create-user-assigned-managed-identity-and-grant-permissions"></a>Créer une identité managée affectée par l’utilisateur et octroyer des autorisations 
```bash
# create user assigned identity for image builder to access the storage account where the script is located
idenityName=aibBuiUserId$(date +'%s')
az identity create -g $imageResourceGroup -n $idenityName

# get identity id
imgBuilderCliId=$(az identity show -g $imageResourceGroup -n $idenityName | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$imageResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$idenityName

# download preconfigured role definition example
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')

# update the definition
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json

# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role $imageRoleDefName \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup
```



## <a name="download-the-image-configuration-template-example"></a>Télécharger l’exemple de modèle de configuration d’image

Un modèle de configuration d’image paramétrable a été créé pour vous permettre d'essayer. Téléchargez l’exemple de fichier .json et configurez-le avec les variables précédemment définies.

```azurecli-interactive
curl https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/quickquickstarts/0_Creating_a_Custom_Windows_Managed_Image/helloImageTemplateWin.json -o helloImageTemplateWin.json

sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateWin.json
sed -i -e "s/<rgName>/$imageResourceGroup/g" helloImageTemplateWin.json
sed -i -e "s/<region>/$location/g" helloImageTemplateWin.json
sed -i -e "s/<imageName>/$imageName/g" helloImageTemplateWin.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateWin.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateWin.json

```

Vous pouvez modifier cet exemple dans le terminal à l’aide d’un éditeur de texte comme `vi`.

```azurecli-interactive
vi helloImageTemplateWin.json
```

> [!NOTE]
> Pour l’image source, vous devez toujours [spécifier une version](../linux/image-builder-troubleshoot.md#build--step-failed-for-image-version) ; vous ne pouvez pas utiliser `latest`.
> Si vous ajoutez ou modifiez le groupe de ressources où l’image est distribuée, vous devez vous assurer que les [autorisations sont définies](#create-a-user-assigned-identity-and-set-permissions-on-the-resource-group) sur le groupe de ressources.
 
## <a name="create-the-image"></a>Création de l’image

Envoyer la configuration de l’image au service Générateur d’images de votre machine virtuelle

```azurecli-interactive
az resource create \
    --resource-group $imageResourceGroup \
    --properties @helloImageTemplateWin.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateWin01
```

Une fois l’opération terminée, un message de réussite s’affiche à nouveau dans la console et crée un `Image Builder Configuration Template` dans `$imageResourceGroup`. Vous pouvez voir cette ressource dans le groupe de ressources dans le portail Azure si vous activez « Afficher les types masqués ».

En arrière-plan, le générateur d’images crée également un groupe de ressources de mise en lots dans votre abonnement. Ce groupe de ressources est utilisé pour générer l’image. Il se présente au format suivant : `IT_<DestinationResourceGroup>_<TemplateName>`

> [!Note]
> Vous ne devez pas supprimer directement le groupe de ressources de mise en lots. Commencez par supprimer l’artefact de modèle d’image ; cela supprimera le groupe de ressources de mise en lots.

Si le service signale un échec lors de la soumission du modèle de configuration d’image,
-  consultez ces étapes de [résolution des problèmes](../linux/image-builder-troubleshoot.md#troubleshoot-image-template-submission-errors). 
- Vous devez supprimer le modèle, à l’aide de l’extrait de code suivant, avant de retenter la soumission.

```azurecli-interactive
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateLinux01
```

## <a name="start-the-image-build"></a>Lancer la génération de l’image
Démarrez le processus de génération de l'image à l'aide d'[az resource invoke-action](/cli/azure/resource#az-resource-invoke-action).

```azurecli-interactive
az resource invoke-action \
     --resource-group $imageResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateWin01 \
     --action Run 
```

Attendez que la compilation soit terminée. Cela peut durer environ 15 minutes.

Si vous rencontrez des erreurs, passez en revue ces étapes de [résolution des problèmes](../linux/image-builder-troubleshoot.md#troubleshoot-common-build-errors).


## <a name="create-the-vm"></a>Création de la machine virtuelle

Créez la machine virtuelle avec l’image que vous avez créée. Remplacez *\<password>* par votre mot de passe pour le `aibuser` sur la machine virtuelle.

```azurecli-interactive
az vm create \
  --resource-group $imageResourceGroup \
  --name aibImgWinVm00 \
  --admin-username aibuser \
  --admin-password <password> \
  --image $imageName \
  --location $location
```

## <a name="verify-the-customization"></a>Vérifier la personnalisation

Créez une connexion Bureau à distance à la machine virtuelle avec le nom d’utilisateur et le mot de passe définis lors de la création de la machine virtuelle. Dans la machine virtuelle, ouvrez une invite de commande et tapez :

```console
dir c:\
```

Vous devez voir ces deux répertoires créés pendant la personnalisation de l’image :
- buildActions
- buildArtifacts

## <a name="clean-up"></a>Nettoyer

Lorsque vous avez terminé, supprimez les ressources.

### <a name="delete-the-image-builder-template"></a>Supprimer le modèle de générateur d’images

```azurecli-interactive
az resource delete \
    --resource-group $imageResourceGroup \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateWin01
```

### <a name="delete-the-role-assignment-role-definition-and-user-identity"></a>Supprimez l’attribution de rôle, la définition de rôle et l’identité de l’utilisateur.
```azurecli-interactive
az role assignment delete \
    --assignee $imgBuilderCliId \
    --role "$imageRoleDefName" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup

az role definition delete --name "$imageRoleDefName"

az identity delete --ids $imgBuilderId
```

### <a name="delete-the-image-resource-group"></a>Supprimer le groupe de ressources d’image

```azurecli-interactive
az group delete -n $imageResourceGroup
```


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les composants du fichier .json utilisé dans cet article, voir [Référence des modèles du Générateur d’images](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
