---
title: Créer un fournisseur de ressources
description: Décrit comment créer un fournisseur de ressources et déployer ses types de ressources personnalisés.
author: MSEvanhi
ms.topic: tutorial
ms.date: 06/24/2020
ms.author: evanhi
ms.openlocfilehash: 23ae69f49d37a210dd9fa1bfeedeca6c1b461e20
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888859"
---
# <a name="quickstart-create-a-custom-provider-and-deploy-custom-resources"></a>Démarrage rapide : Créer un fournisseur personnalisé et déployer des ressources personnalisées

Dans ce guide de démarrage rapide, vous allez créer votre propre fournisseur de ressources et déployer des types de ressources personnalisés pour celui-ci. Pour plus d’informations sur les fournisseurs personnalisés, consultez [Présentation de la préversion d’Azure Custom Providers](overview.md).

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Pour effectuer les étapes mentionnées dans ce guide, vous devez appeler des opérations `REST`. Il existe [différentes façons d’envoyer des demandes REST](/rest/api/azure/).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Préparez votre environnement pour l’interface Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

Les exemples Azure CLI utilisent `az rest` pour les demandes `REST`. Pour plus d’informations, consultez [az rest](/cli/azure/reference-index#az-rest).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Les commandes PowerShell sont exécutées localement à l’aide de PowerShell 7 ou une version ultérieure et les modules Azure PowerShell. Pour plus d’informations, consultez l’article [Installation et configuration d’Azure PowerShell](/powershell/azure/install-az-ps).
- Si vous ne disposez pas déjà d’un outil pour les opérations `REST`, installez [ARMClient](https://github.com/projectkudu/ARMClient). Il s’agit d’un outil de ligne de commande open source qui simplifie l’appel de l’API Azure Resource Manager, est nécessaire.
- Une fois **ARMClient** installé, vous pouvez afficher les informations d’utilisation à partir d’une invite de commandes PowerShell en tapant : `armclient.exe`. Vous pouvez également accéder au [wiki ARMClient](https://github.com/projectkudu/ARMClient/wiki).

---

## <a name="deploy-custom-provider"></a>Déployer un fournisseur personnalisé

Pour configurer le fournisseur personnalisé, vous devez déployer un [exemple de modèle](https://github.com/Azure/azure-docs-json-samples/blob/master/custom-providers/customprovider.json) sur votre abonnement Azure.

Après avoir déployé le modèle, votre abonnement affiche les ressources suivantes :

- Une application de fonction avec les opérations pour les ressources et les actions.
- Un compte de stockage pour stocker les utilisateurs créés via le fournisseur personnalisé.
- Un fournisseur personnalisé qui définit les types de ressources personnalisées et les actions. Il utilise le point de terminaison de l’application de fonction pour envoyer les demandes.
- Des ressources personnalisées provenant du fournisseur personnalisé.

Pour déployer le fournisseur personnalisé, utilisez Azure CLI, PowerShell ou le portail Azure :

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Cet exemple vous invite à entrer un groupe de ressources, un emplacement et le nom de l’application de fonction du fournisseur. Les noms sont stockés dans des variables qui sont utilisées dans d’autres commandes. Les commandes [az group create](/cli/azure/group#az-group-create) et [az deployment group create](/cli/azure/deployment/group#az-deployment-group-create) déploient les ressources.

```azurecli-interactive
read -p "Enter a resource group name:" rgName &&
read -p "Enter the location (i.e. eastus):" location &&
read -p "Enter the provider's function app name:" funcName &&
templateUri="https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/custom-providers/customprovider.json" &&
az group create --name $rgName --location "$location" &&
az deployment group create --resource-group $rgName --template-uri $templateUri --parameters funcName=$funcName &&
echo "Press [ENTER] to continue ..." &&
read
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cet exemple vous invite à entrer un groupe de ressources, un emplacement et le nom de l’application de fonction du fournisseur. Les noms sont stockés dans des variables qui sont utilisées dans d’autres commandes. Les commandes [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) et [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) déploient les ressources.

```powershell
$rgName = Read-Host -Prompt "Enter a resource group name"
$location = Read-Host -Prompt "Enter the location (i.e. eastus)"
$funcName = Read-Host -Prompt "Enter the provider's function app name"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/custom-providers/customprovider.json"
New-AzResourceGroup -Name $rgName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $rgName -TemplateUri $templateUri -funcName $funcName
Read-Host -Prompt "Press [ENTER] to continue ..."
```

---

Vous pouvez également déployer la solution à partir du portail Azure. Sélectionnez le bouton **Déployer sur Azure** pour ouvrir le modèle dans le portail Azure.

[![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-docs-json-samples%2Fmaster%2Fcustom-providers%2Fcustomprovider.json)

## <a name="view-custom-provider-and-resource"></a>Afficher le fournisseur personnalisé et la ressource

Dans le portail, le fournisseur personnalisé est un type de ressource masqué. Pour vérifier que le fournisseur de ressources a été déployé, accédez au groupe de ressources. Sélectionnez l’option **Afficher les types masqués**.

![Afficher les types de ressources masqués](./media/create-custom-provider/show-hidden.png)

Pour voir le type de ressource personnalisé que vous avez déployé, utilisez l’opération `GET` sur votre type de ressource.

```http
GET https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users?api-version=2018-09-01-preview
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
subID=$(az account show --query id --output tsv)
requestURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users?api-version=2018-09-01-preview"
az rest --method get --uri $requestURI
```

Vous recevez la réponse :

```json
{
  "value": [
    {
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/santa",
      "name": "santa",
      "properties": {
        "FullName": "Santa Claus",
        "Location": "NorthPole",
        "provisioningState": "Succeeded"
      },
      "resourceGroup": "<rg-name>",
      "type": "Microsoft.CustomProviders/resourceProviders/users"
    }
  ]
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$subID = (Get-AzContext).Subscription.Id
$requestURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users?api-version=2018-09-01-preview"

armclient GET $requestURI
```

Vous recevez la réponse :

```json
{
  "value": [
    {
      "properties": {
        "provisioningState": "Succeeded",
        "FullName": "Santa Claus",
        "Location": "NorthPole"
      },
      "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/santa",
      "name": "santa",
      "type": "Microsoft.CustomProviders/resourceProviders/users"
    }
  ]
}
```

---

## <a name="call-action"></a>Appeler une action

Votre fournisseur personnalisé dispose également d’une action nommée `ping`. Le code qui traite la demande est implémenté dans l’application de fonction. L’action `ping` répond par un message d’accueil.

Pour envoyer une demande `ping`, utilisez l’opération `POST` sur votre fournisseur personnalisé.

```http
POST https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/ping?api-version=2018-09-01-preview
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
pingURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/ping?api-version=2018-09-01-preview"
az rest --method post --uri $pingURI
```

Vous recevez la réponse :

```json
{
  "message": "hello <function-name>.azurewebsites.net",
  "pingcontent": {
    "source": "<function-name>.azurewebsites.net"
  }
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$pingURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/ping?api-version=2018-09-01-preview"

armclient POST $pingURI
```

Vous recevez la réponse :

```json
{
  "pingcontent": {
    "source": "<function-name>.azurewebsites.net"
  },
  "message": "hello <function-name>.azurewebsites.net"
}
```

---

## <a name="create-a-resource-type"></a>Créer un type de ressource

Pour créer le type de ressource personnalisé, vous pouvez déployer la ressource dans un modèle. Cette approche est illustrée dans le modèle que vous avez déployé dans ce guide. Vous pouvez également envoyer une demande `PUT` pour le type de ressource.

```http
PUT https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/<resource-name>?api-version=2018-09-01-preview

{"properties":{"FullName": "Test User", "Location": "Earth"}}
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
addURI="https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users/testuser?api-version=2018-09-01-preview"
az rest --method put --uri $addURI --body "{'properties':{'FullName': 'Test User', 'Location': 'Earth'}}"
```

Vous recevez la réponse :

```json
{
  "id": "/subscriptions/<sub-ID>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/testuser",
  "name": "testuser",
  "properties": {
    "FullName": "Test User",
    "Location": "Earth",
    "provisioningState": "Succeeded"
  },
  "resourceGroup": "<rg-name>",
  "type": "Microsoft.CustomProviders/resourceProviders/users"
}
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```powershell
$addURI = "https://management.azure.com/subscriptions/$subID/resourceGroups/$rgName/providers/Microsoft.CustomProviders/resourceProviders/$funcName/users/testuser?api-version=2018-09-01-preview"
$requestBody = "{'properties':{'FullName': 'Test User', 'Location': 'Earth'}}"

armclient PUT $addURI $requestBody
```

Vous recevez la réponse :

```json
{
  "properties": {
    "provisioningState": "Succeeded",
    "FullName": "Test User",
    "Location": "Earth"
  },
  "id": "/subscriptions/<sub-ID>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceProviders/<provider-name>/users/testuser",
  "name": "testuser",
  "type": "Microsoft.CustomProviders/resourceProviders/users"
}
```

---

## <a name="custom-resource-provider-commands"></a>Commandes de fournisseur de ressources personnalisé

Utilisez les commandes [custom-providers](/cli/azure/ext/custom-providers/custom-providers/resource-provider) pour travailler avec votre fournisseur de ressources personnalisé.

### <a name="list-custom-resource-providers"></a>Lister les fournisseurs de ressources personnalisés

Utilisez la commande `list` pour afficher tous les fournisseurs de ressources personnalisés disponibles dans un abonnement. La valeur par défaut liste les fournisseurs de ressources personnalisés de l’abonnement actuel, mais vous pouvez également spécifier le paramètre `--subscription`. Pour lister un groupe de ressources, utilisez le paramètre `--resource-group`.

```azurecli-interactive
az custom-providers resource-provider list --subscription $subID
```

```json
[
  {
    "actions": [
      {
        "endpoint": "https://<provider-name>.azurewebsites.net/api/{requestPath}",
        "name": "ping",
        "routingType": "Proxy"
      }
    ],
    "id": "/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/Microsoft.CustomProviders/resourceproviders/<provider-name>",
    "location": "eastus",
    "name": "<provider-name>",
    "provisioningState": "Succeeded",
    "resourceGroup": "<rg-name>",
    "resourceTypes": [
      {
        "endpoint": "https://<provider-name>.azurewebsites.net/api/{requestPath}",
        "name": "users",
        "routingType": "Proxy, Cache"
      }
    ],
    "tags": {},
    "type": "Microsoft.CustomProviders/resourceproviders",
    "validations": null
  }
]
```

### <a name="show-the-properties"></a>Afficher les propriétés

Utilisez la commande `show` pour afficher les propriétés du fournisseur de ressources personnalisé. Le format de sortie ressemble à la sortie `list`.

```azurecli-interactive
az custom-providers resource-provider show --resource-group $rgName --name $funcName
```

### <a name="create-a-new-resource"></a>Créer une nouvelle ressource

Utilisez la commande `create` pour créer ou mettre à jour un fournisseur de ressources personnalisé. Cet exemple met à jour `actions` et `resourceTypes`.

```azurecli-interactive
az custom-providers resource-provider create --resource-group $rgName --name $funcName \
--action name=ping endpoint=https://myTestSite.azurewebsites.net/api/{requestPath} routing_type=Proxy \
--resource-type name=users endpoint=https://myTestSite.azurewebsites.net/api{requestPath} routing_type="Proxy, Cache"
```

```json
"actions": [
  {
    "endpoint": "https://myTestSite.azurewebsites.net/api/{requestPath}",
    "name": "ping",
    "routingType": "Proxy"
  }
],

"resourceTypes": [
  {
    "endpoint": "https://myTestSite.azurewebsites.net/api{requestPath}",
    "name": "users",
    "routingType": "Proxy, Cache"
  }
],
```

### <a name="update-the-providers-tags"></a>Mettre à jour les étiquettes du fournisseur

La commande `update` met uniquement à jour les étiquettes d’un fournisseur de ressources personnalisé. Dans le portail Azure, le service d’application du fournisseur de ressources personnalisé affiche l’étiquette.

```azurecli-interactive
az custom-providers resource-provider update --resource-group $rgName --name $funcName --tags new=tag
```

```json
"tags": {
  "new": "tag"
},
```

### <a name="delete-a-custom-resource-provider"></a>Supprimer un fournisseur de ressources personnalisé

La commande `delete` vous invite à supprimer uniquement le fournisseur de ressources personnalisé. Le compte de stockage, le service d’application et le plan de service d’application ne sont pas supprimés. Une fois le fournisseur supprimé, vous êtes redirigé vers une invite de commandes.

```azurecli-interactive
az custom-providers resource-provider delete --resource-group $rgName --name $funcName
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir une présentation des fournisseurs personnalisés, consultez l’article suivant :

> [!div class="nextstepaction"]
> [Vue d’ensemble des fournisseurs personnalisés Azure en préversion](overview.md)
