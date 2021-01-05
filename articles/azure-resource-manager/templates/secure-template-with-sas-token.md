---
title: Déployer un modèle avec un jeton SAS en toute sécurité
description: Déployez des ressources sur Azure avec un modèle Azure Resource Manager protégé par un jeton SAP. Affiche Azure PowerShell et Azure CLI.
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 8b35e82da8ebca98ec9fe1fb7441612bf61fb142
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88855654"
---
# <a name="deploy-private-arm-template-with-sas-token"></a>Déployer un modèle privé ARM avec un jeton SAS

Lorsque votre modèle Azure Resource Manager (modèle ARM) se trouve dans un compte de stockage, vous pouvez restreindre l’accès au modèle pour éviter de l’exposer publiquement. Vous accédez à un modèle sécurisé en créant un jeton de signature d’accès partagé (SAS) pour le modèle et en fournissant ce jeton pendant le déploiement. Cet article explique comment utiliser Azure PowerShell ou Azure CLI pour déployer un modèle avec un jeton SAP.

> [!IMPORTANT]
> Plutôt que de sécuriser votre modèle avec un jeton SAS, envisagez d’utiliser les [spécifications de modèle](template-specs.md). Les spécifications de modèle vous permettent de partager vos modèles avec d’autres utilisateurs de votre organisation et de gérer l’accès aux modèles par le biais d’Azure RBAC.

## <a name="create-storage-account-with-secured-container"></a>Créer un compte de stockage avec un conteneur sécurisé

Le script suivant crée un compte de stockage et un conteneur avec l’accès public désactivé.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup `
  -Name ExampleGroup `
  -Location "Central US"
New-AzStorageAccount `
  -ResourceGroupName ExampleGroup `
  -Name {your-unique-name} `
  -Type Standard_LRS `
  -Location "Central US"
Set-AzCurrentStorageAccount `
  -ResourceGroupName ExampleGroup `
  -Name {your-unique-name}
New-AzStorageContainer `
  -Name templates `
  -Permission Off
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create \
  --name "ExampleGroup" \
  --location "Central US"
az storage account create \
    --resource-group ExampleGroup \
    --location "Central US" \
    --sku Standard_LRS \
    --kind Storage \
    --name {your-unique-name}
connection=$(az storage account show-connection-string \
    --resource-group ExampleGroup \
    --name {your-unique-name} \
    --query connectionString)
az storage container create \
    --name templates \
    --public-access Off \
    --connection-string $connection
```

---

## <a name="upload-template-to-storage-account"></a>Charger le modèle dans le compte de stockage

Maintenant, vous pouvez charger votre modèle dans le compte de stockage. Indiquez le chemin d’accès au modèle que vous souhaitez utiliser.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Set-AzStorageBlobContent `
  -Container templates `
  -File c:\Templates\azuredeploy.json
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az storage blob upload \
    --container-name templates \
    --file azuredeploy.json \
    --name azuredeploy.json \
    --connection-string $connection
```

---

## <a name="provide-sas-token-during-deployment"></a>Fournir un jeton SAP au cours du déploiement

Pour déployer un modèle dans un compte de stockage privé, générez un jeton SAP et incluez-le dans l’URI du modèle. Définissez le délai d’expiration de façon à laisser suffisamment de temps pour terminer le déploiement.

> [!IMPORTANT]
> L’objet blob contenant le modèle n’est accessible qu’au propriétaire du compte. Toutefois, lorsque vous créez un jeton SAP pour l’objet blob, celui-ci est accessible à toute personne ayant cet URI. Si un autre utilisateur intercepte l’URI, il pourra accéder au modèle. Un jeton SAP est un bon moyen de limiter l’accès à vos modèles, mais vous ne devez pas inclure de données sensibles comme des mots de passe directement dans le modèle.
>

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
# get the URI with the SAS token
$templateuri = New-AzStorageBlobSASToken `
  -Container templates `
  -Blob azuredeploy.json `
  -Permission r `
  -ExpiryTime (Get-Date).AddHours(2.0) -FullUri

# provide URI with SAS token during deployment
New-AzResourceGroupDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri $templateuri
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

L’exemple suivant fonctionne avec l’environnement Bash dans Cloud Shell. D’autres environnements peuvent nécessiter une syntaxe différente pour créer l’heure d’expiration du jeton SAS.

```azurecli-interactive
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group ExampleGroup \
    --name {your-unique-name} \
    --query connectionString)
token=$(az storage blob generate-sas \
    --container-name templates \
    --name azuredeploy.json \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
    --container-name templates \
    --name azuredeploy.json \
    --output tsv \
    --connection-string $connection)
az deployment group create \
  --resource-group ExampleGroup \
  --template-uri $url?$token
```

---

Pour accéder à un exemple d’utilisation d’un jeton SAP avec des modèles liés, consultez [Utilisation de modèles liés avec Azure Resource Manager](linked-templates.md).


## <a name="next-steps"></a>Étapes suivantes
* Pour une introduction au déploiement de modèles, voir [Déployer des ressources à l’aide de modèles ARM et d’Azure PowerShell](deploy-powershell.md).
* Pour définir des paramètres dans le modèle, consultez [Création de modèles](template-syntax.md#parameters).
