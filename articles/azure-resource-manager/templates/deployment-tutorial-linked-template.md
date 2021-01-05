---
title: 'Tutoriel : Déployer un modèle lié'
description: Découvrez comment déployer un modèle lié
ms.date: 03/13/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: ''
ms.openlocfilehash: 81a8b6cd5044b31ee80f50448d6610ba6a57281e
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97585647"
---
# <a name="tutorial-deploy-a-linked-template"></a>Tutoriel : Déployer un modèle lié

Dans les [tutoriels précédents](./deployment-tutorial-local-template.md), vous avez appris à déployer un modèle qui est stocké sur votre ordinateur local. Pour déployer des solutions complexes, vous pouvez diviser un modèle en plusieurs modèles, puis les déployer au moyen d’un modèle principal. Dans ce tutoriel, vous allez apprendre à déployer un modèle principal qui contient la référence à un modèle lié. Lorsque le modèle principal est déployé, il déclenche le déploiement du modèle lié. Vous allez également apprendre à stocker et à sécuriser le modèle lié à l’aide d’un jeton SAS. Comptez environ **12 minutes** pour suivre ce tutoriel.

## <a name="prerequisites"></a>Prérequis

Nous vous recommandons de suivre le tutoriel précédent, mais ce n’est pas obligatoire.

## <a name="review-template"></a>Vérifier le modèle

Dans les tutoriels précédents, vous avez déployé un modèle qui crée un compte de stockage, un plan App Service et une application web. Le modèle utilisé était :

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/local-template/azuredeploy.json":::

## <a name="create-a-linked-template"></a>Créer un modèle lié

Vous pouvez séparer la ressource de compte de stockage en un modèle lié :

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/linkedStorageAccount.json":::

Le modèle suivant est le modèle principal. L’objet `Microsoft.Resources/deployments` mis en surbrillance montre comment appeler un modèle lié. Le modèle lié ne peut pas être stocké en tant que fichier local ni fichier qui est uniquement disponible sur votre réseau local. Vous pouvez seulement fournir une valeur d’URI qui inclut HTTP ou HTTPS. Resource Manager doit être en mesure d’accéder au modèle. Une possibilité consiste à placer votre modèle lié dans un compte de stockage et à utiliser l’URI de cet élément. L’URI est passé au modèle à l’aide d’un paramètre. Consultez la définition de paramètre en surbrillance.

:::code language="json" source="~/resourcemanager-templates/get-started-deployment/linked-template/azuredeploy.json" highlight="27-32,40-58":::

Enregistrez une copie du modèle principal sur votre ordinateur local avec l’extension _.json_, par exemple _azuredeploy.json_. Vous n’avez pas besoin d’enregistrer une copie du modèle lié. Le modèle lié est copié à partir d’un dépôt GitHub vers un compte de stockage.

## <a name="store-the-linked-template"></a>Stocker le modèle lié

Le script PowerShell suivant crée un compte de stockage, crée un conteneur et copie le modèle lié à partir d’un dépôt GitHub vers le conteneur. Une copie du modèle lié est stockée dans [GitHub](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json).

Sélectionnez **Essayer** pour ouvrir Cloud Shell et **Copier** pour copier le script PowerShell, puis cliquez avec le bouton droit sur le volet de l’interpréteur de commandes pour coller le script :

> [!IMPORTANT]
> Ce nom doit comprendre entre 3 et 24 caractères, uniquement des lettres en minuscules et des nombres. Le nom doit être unique. Dans le modèle, le nom du compte de stockage est le nom du projet auquel **store** est ajouté, et le nom du projet doit comprendre entre 3 et 11 caractères. Le nom du projet doit donc respecter les exigences du nom du compte de stockage et comporter moins de 11 caractères.

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name:"   # This name is used to generate names for Azure resources, such as storage account name.
$location = Read-Host -Prompt "Enter a location (i.e. centralus)"

$resourceGroupName = $projectName + "rg"
$storageAccountName = $projectName + "store"
$containerName = "templates" # The name of the Blob container to be created.

$linkedTemplateURL = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/get-started-deployment/linked-template/linkedStorageAccount.json" # A completed linked template used in this tutorial.
$fileName = "linkedStorageAccount.json" # A file name used for downloading and uploading the linked template.

# Download the template
Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$fileName"

# Create a resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create a storage account
$storageAccount = New-AzStorageAccount `
    -ResourceGroupName $resourceGroupName `
    -Name $storageAccountName `
    -Location $location `
    -SkuName "Standard_LRS"

$context = $storageAccount.Context

# Create a container
New-AzStorageContainer -Name $containerName -Context $context -Permission Container

# Upload the template
Set-AzStorageBlobContent `
    -Container $containerName `
    -File "$home/$fileName" `
    -Blob $fileName `
    -Context $context

Write-Host "Press [ENTER] to continue ..."
```

## <a name="deploy-template"></a>Déployer un modèle

Pour déployer un modèle dans un compte de stockage privé, générez un jeton SAP et incluez-le dans l’URI du modèle. Définissez le délai d’expiration de façon à laisser suffisamment de temps pour terminer le déploiement. L’objet blob contenant le modèle n’est accessible qu’au propriétaire du compte. Toutefois, lorsque vous créez un jeton SAP pour l’objet blob, celui-ci est accessible à toute personne ayant cet URI. Si un autre utilisateur intercepte l’URI, il pourra accéder au modèle. Un jeton SAP est un bon moyen de limiter l’accès à vos modèles, mais vous ne devez pas inclure de données sensibles comme des mots de passe directement dans le modèle.

Si vous n’avez pas créé le groupe de ressources, consultez [Créer un groupe de ressources](./deployment-tutorial-local-template.md#create-resource-group).

> [!NOTE]
> Dans le code Azure CLI ci-dessous, le paramètre `date` `-d` n’est pas un argument valide sur macOS. Si vous utilisez macOS, indiquez `-v+2H` pour ajouter 2 heures à l’heure actuelle dans le terminal sur macOS.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell

$projectName = Read-Host -Prompt "Enter a project name:"   # This name is used to generate names for Azure resources, such as storage account name.
$templateFile = Read-Host -Prompt "Enter the main template file and path"

$resourceGroupName="${projectName}rg"
$storageAccountName="${projectName}store"
$containerName = "templates"
$fileName = "linkedStorageAccount.json" # A file name used for downloading and uploading the linked template.

$key = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
$context = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $key

# Generate a SAS token
$linkedTemplateUri = New-AzStorageBlobSASToken `
    -Context $context `
    -Container $containerName `
    -Blob $fileName `
    -Permission r `
    -ExpiryTime (Get-Date).AddHours(2.0) `
    -FullUri

# Deploy the template
New-AzResourceGroupDeployment `
  -Name DeployLinkedTemplate `
  -ResourceGroupName $resourceGroupName `
  -TemplateFile $templateFile `
  -projectName $projectName `
  -linkedTemplateUri $linkedTemplateUri `
  -verbose
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli

echo "Enter a project name that is used to generate resource names:"
read projectName
echo "Enter the main template file:"
read templateFile

resourceGroupName="${projectName}rg"
storageAccountName="${projectName}store"
containerName="templates"
fileName="linkedStorageAccount.json"

key=$(az storage account keys list -g $resourceGroupName -n $storageAccountName --query [0].value -o tsv)

linkedTemplateUri=$(az storage blob generate-sas \
  --account-name $storageAccountName \
  --account-key $key \
  --container-name $containerName \
  --name $fileName \
  --permissions r \
  --expiry `date -u -d "120 minutes" '+%Y-%m-%dT%H:%MZ'` \
  --full-uri)

linkedTemplateUri=$(echo $linkedTemplateUri | sed 's/"//g')
az deployment group create \
  --name DeployLinkedTemplate \
  --resource-group $resourceGroupName \
  --template-file $templateFile \
  --parameters projectName=$projectName linkedTemplateUri=$linkedTemplateUri \
  --verbose
```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Nettoyez les ressources que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à déployer un modèle lié. Dans le tutoriel suivant, vous allez apprendre à créer un pipeline DevOps pour déployer un modèle.

> [!div class="nextstepaction"]
> [Créer un pipeline](./deployment-tutorial-pipeline.md)
