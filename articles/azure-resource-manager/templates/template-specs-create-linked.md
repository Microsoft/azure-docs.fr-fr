---
title: Créer une spec de modèle avec des modèles liés
description: Découvrez comment créer une spec de modèle avec des modèles liés.
ms.topic: conceptual
ms.date: 11/17/2020
ms.openlocfilehash: 038fb3e6bbb6086535b83a69de7a3fe49a8e60bb
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518887"
---
# <a name="tutorial-create-a-template-spec-with-linked-templates-preview"></a>Tutoriel : Créer une spec de modèle avec des modèles liés (préversion)

Découvrez comment créer une [spec de modèle](template-specs.md) avec un modèle principal et un [modèle lié](linked-templates.md#linked-template). Les specs de modèle permettent de partager des modèles ARM avec d’autres utilisateurs de votre organisation. Cet article explique comment créer une spec de modèle pour empaqueter un modèle principal et ses modèles liés à l’aide de la nouvelle propriété `relativePath` de la [ressources de déploiement](/azure/templates/microsoft.resources/deployments).

## <a name="prerequisites"></a>Prérequis

Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

> [!NOTE]
> La fonctionnalité Specs de modèle est actuellement en préversion. Pour l’utiliser avec Azure PowerShell, vous devez installer la [version 5.0.0 ou ultérieure](/powershell/azure/install-az-ps). Pour l’utiliser avec l’interface de ligne de commande Azure CLI, utilisez la [version 2.14.2 ou ultérieure](/cli/azure/install-azure-cli).

## <a name="create-linked-templates"></a>Créer des modèles liés

Créez le modèle principal et le modèle lié.

Pour lier un modèle, ajoutez une [ressource de déploiement](/azure/templates/microsoft.resources/deployments) à votre modèle principal. Dans la propriété `templateLink`, spécifiez le chemin d’accès relatif du modèle lié, conformément au chemin d’accès du modèle parent.

Le modèle lié est nommé **linkedTemplate.json**, et il est stocké dans un sous-dossier appelé **artefacts** dans le chemin d’accès de l’emplacement où est stocké le modèle principal.  Pour relativePath, vous pouvez utiliser une des valeurs suivantes :

- `./artifacts/linkedTemplate.json`
- `/artifacts/linkedTemplate.json`
- `artifacts/linkedTemplate.json`

La propriété `relativePath` est toujours relative au fichier de modèle dans lequel `relativePath` est déclaré. Ainsi, s’il existe un autre linkedTemplate2.json appelé à partir de linkedTemplate.json, et si linkedTemplate2.json est stocké dans le même sous-dossier artefacts, le relativePath spécifié dans linkedTemplate.json est simplement `linkedTemplate2.json`.

1. Créez le modèle principal avec le code JSON suivant. Enregistrez le modèle principal sous **azuredeploy.json** sur votre ordinateur local. Ce tutoriel part du principe que vous avez enregistré dans le chemin d’accès **c:\Templates\linkedTS\azuredeploy.json**, mais vous pouvez utiliser n’importe quel chemin.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "location": {
          "type": "string",
          "defaultValue": "westus2",
          "metadata":{
            "description": "Specify the location for the resources."
          }
        },
        "storageAccountType": {
          "type": "string",
          "defaultValue": "Standard_LRS",
          "metadata":{
            "description": "Specify the storage account type."
          }
        }
      },
      "variables": {
        "appServicePlanName": "[concat('plan', uniquestring(resourceGroup().id))]"
      },
      "resources": [
        {
          "type": "Microsoft.Web/serverfarms",
          "apiVersion": "2016-09-01",
          "name": "[variables('appServicePlanName')]",
          "location": "[parameters('location')]",
          "sku": {
            "name": "B1",
            "tier": "Basic",
            "size": "B1",
            "family": "B",
            "capacity": 1
          },
          "kind": "linux",
          "properties": {
            "perSiteScaling": false,
            "reserved": true,
            "targetWorkerCount": 0,
            "targetWorkerSizeId": 0
          }
        },
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-06-01",
          "name": "createStorage",
          "properties": {
            "mode": "Incremental",
            "templateLink": {
              "relativePath": "artifacts/linkedTemplate.json"
            },
            "parameters": {
              "storageAccountType": {
                "value": "[parameters('storageAccountType')]"
              }
            }
          }
        }
      ]
    }
    ```

    > [!NOTE]
    > La valeur apiVersion de `Microsoft.Resources/deployments` doit être 2020-06-01 ou une version ultérieure.

1. Créez un répertoire appelé **artefacts** dans le dossier dans lequel le modèle principal est enregistré.
1. Créez le modèle lié avec le code JSON suivant :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageAccountType": {
          "type": "string",
          "defaultValue": "Standard_LRS",
          "allowedValues": [
            "Standard_LRS",
            "Standard_GRS",
            "Standard_ZRS",
            "Premium_LRS"
          ],
          "metadata": {
            "description": "Storage Account type"
          }
        },
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        }
      },
      "variables": {
        "storageAccountName": "[concat('store', uniquestring(resourceGroup().id))]"
      },
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2019-04-01",
          "name": "[variables('storageAccountName')]",
          "location": "[parameters('location')]",
          "sku": {
            "name": "[parameters('storageAccountType')]"
          },
          "kind": "StorageV2",
          "properties": {}
        }
      ],
      "outputs": {
        "storageAccountName": {
          "type": "string",
          "value": "[variables('storageAccountName')]"
        }
      }
    }
    ```

1. Enregistrez le modèle sous **linkedTemplate.json** dans le dossier **artefacts**.

## <a name="create-template-spec"></a>Créer une spec de modèle

Les specs de modèle sont stockées dans des groupes de ressources.  Créez un groupe de ressources, puis créez une spec de modèle avec le script suivant. Le nom de la spec de modèle est **webSpec**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name templateSpecRG `
  -Location westus2

New-AzTemplateSpec `
  -Name webSpec `
  -Version "1.0.0.0" `
  -ResourceGroupName templateSpecRG `
  -Location westus2 `
  -TemplateFile "c:\Templates\linkedTS\azuredeploy.json"
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az group create \
  --name templateSpecRG \
  --location westus2

az ts create \
  --name webSpec \
  --version "1.0.0.0" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "c:\Templates\linkedTS\azuredeploy.json"
```

---

Lorsque vous avez terminé, vous pouvez afficher la spec de modèle à partir du portail Azure ou à l’aide de la cmdlet suivante :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Get-AzTemplateSpec -ResourceGroupName templatespecRG -Name webSpec
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az ts show --name webSpec --resource-group templateSpecRG --version "1.0.0.0"
```

---

## <a name="deploy-template-spec"></a>Déployer une spec de modèle

Vous pouvez maintenant déployer la spec de modèle. Le déploiement de la spec de modèle est identique à celui du modèle que celle-ci contient, à l’exception du fait que vous transmettez l’ID de ressource de la spec de modèle. Vous utilisez les mêmes commandes de déploiement et, si nécessaire, transmettez les valeurs de paramètres pour la spec de modèle.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroup `
  -Name webRG `
  -Location westus2

$id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name webSpec -Version "1.0.0.0").Versions.Id

New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName webRG
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az group create \
  --name webRG \
  --location westus2

id = $(az template-specs show --name webSpec --resource-group templateSpecRG --version "1.0.0.0" --query "id")

az deployment group create \
  --resource-group webRG \
  --template-spec $id
```

> [!NOTE]
> Il existe un problème connu lié à l’obtention de l’ID de la spec de modèle et à son affectation à une variable dans Windows PowerShell.

---

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir le déploiement d’une spec de modèle en tant que modèle lié, consultez [Tutoriel : Déployer une spec de modèle en tant que modèle lié](template-specs-deploy-linked-template.md).
