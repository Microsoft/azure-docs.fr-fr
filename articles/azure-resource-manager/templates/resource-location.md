---
title: Emplacement de la ressource de modèle
description: Décrit comment définir un emplacement de ressource dans un modèle Azure Resource Manager (modèle ARM).
ms.topic: conceptual
ms.date: 09/04/2019
ms.custom: ''
ms.openlocfilehash: 84a818109e6681b8d0e18de4d2d7969310582818
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922401"
---
# <a name="set-resource-location-in-arm-template"></a>Définir l'emplacement des ressources dans un modèle Resource Manager

Quand vous déployez un modèle Azure Resource Manager (modèle ARM), vous devez fournir un emplacement pour chaque ressource. L’emplacement n’a pas à être le même que celui du groupe de ressources.

## <a name="get-available-locations"></a>Obtenir les emplacements disponibles

Différents types de ressources sont pris en charge à différents emplacements. Pour obtenir les emplacements pris en charge pour un type de ressource, utilisez Azure PowerShell ou Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
((Get-AzResourceProvider -ProviderNamespace Microsoft.Batch).ResourceTypes `
  | Where-Object ResourceTypeName -eq batchAccounts).Locations
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az provider show \
  --namespace Microsoft.Batch \
  --query "resourceTypes[?resourceType=='batchAccounts'].locations | [0]" \
  --out table
```

---

## <a name="use-location-parameter"></a>Utiliser le paramètre d’emplacement

Pour plus de souplesse lors du déploiement de votre modèle, utilisez un paramètre pour spécifier l’emplacement des ressources. Définissez la valeur par défaut du paramètre sur `resourceGroup().location`.

L’exemple suivant illustre le déploiement d’un compte de stockage dans un emplacement spécifié en tant que paramètre :

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
    "storageAccountName": "[concat('storage', uniquestring(resourceGroup().id))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2018-07-01",
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

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir la liste complète des fonctions de modèle, consultez [Fonctions des modèles ARM](template-functions.md).
* Pour plus d'informations sur les fichiers de modèle, consultez [Comprendre la structure et la syntaxe des modèles Resource Manager](template-syntax.md).
