---
title: Fonctions de modèle dans les déploiements délimités
description: Décrit le mode de résolution des fonctions de modèle dans les déploiements délimités. Les étendues peuvent être un locataire, des groupes de ressources, des abonnements et des groupes de ressources.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: cb02a3dc808604a80fd9943138c1cd0d8648904e
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92681279"
---
# <a name="arm-template-functions-in-deployment-scopes"></a>Fonctions de modèle ARM dans les étendues de déploiement

Avec les modèles Azure Resource Manager (modèles ARM), vous pouvez déployer sur des groupes de ressources, des abonnements, des groupes d’administration ou des locataires. En règle générale, les [fonctions de modèle ARM](template-functions.md) fonctionnent de la même manière pour toutes les étendues. Cet article décrit les différences qui existent pour certaines fonctions d’après l’étendue.

## <a name="supported-functions"></a>Fonctions prises en charge

Lors du déploiement sur différentes étendues, il est important de tenir compte des points suivants :

* La fonction [resourceGroup()](template-functions-resource.md#resourcegroup) est **prise en charge** pour les déploiements de groupes de ressources.
* La fonction [subscription()](template-functions-resource.md#subscription) est **prise en charge** pour les déploiements de groupes de ressources et d’abonnements.
* Les fonctions [reference()](template-functions-resource.md#reference) et [list()](template-functions-resource.md#list) sont **prises en charge** pour toutes les étendues.
* Utilisez [resourceId()](template-functions-resource.md#resourceid) pour obtenir l’ID d’une ressource déployée au niveau du groupe de ressources.

  ```json
  "subnet": {
    "id": "[resourceId(parameters('virtualNetworkResourceGroup'), 'Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnet1Name'))]"
  }
  ```

* Vous pouvez également utiliser la fonction [subscriptionResourceId()](template-functions-resource.md#subscriptionresourceid) pour récupérer l’ID d’une ressource déployée au niveau de l’abonnement.

  Par exemple, pour obtenir l’ID de ressource d’une définition de stratégie déployée sur un abonnement, utilisez :

  ```json
  "roleDefinitionId": "[subscriptionResourceId('Microsoft.Authorization/roleDefinitions', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
  ```

* Utilisez la fonction [extensionResourceId()](template-functions-resource.md#extensionresourceid) pour les ressources qui sont implémentées en tant qu’extensions du groupe d’administration. Les définitions de stratégie personnalisée qui sont déployées sur un groupe d’administration sont des extensions de celui-ci.

  Pour obtenir l’ID de ressource d’une définition de stratégie personnalisée au niveau du groupe d’administration, utilisez :
  
  ```json
  "policyDefinitionId": "[extensionResourceId(variables('mgScope'), 'Microsoft.Authorization/policyDefinitions', parameters('policyDefinitionID'))]"
  ```

* Utilisez la fonction [tenantResourceId()](template-functions-resource.md#tenantresourceid) pour obtenir l’ID d’une ressource déployée au niveau du locataire. Les définitions de stratégie intégrées sont des ressources de niveau locataire. Lorsque vous affectez une stratégie intégrée au niveau du groupe d’administration, utilisez la fonction tenantResourceId.

  Pour obtenir l’ID de ressource d’une définition de stratégie intégrée, utilisez :
  
  ```json
  "policyDefinitionId": "[tenantResourceId('Microsoft.Authorization/policyDefinitions', parameters('policyDefinitionID'))]"
  ```

## <a name="function-resolution-in-scopes"></a>Résolution de fonction dans les étendues

Quand vous effectuez un déploiement dans plusieurs étendues, les fonctions [resourceGroup()](template-functions-resource.md#resourcegroup) et [subscription()](template-functions-resource.md#subscription) sont résolues différemment selon la façon dont vous spécifiez le modèle. Quand vous établissez un lien vers un modèle externe, les fonctions sont toujours résolues par rapport à la portée de ce modèle. Quand vous imbriquez un modèle dans un modèle parent, utilisez la propriété `expressionEvaluationOptions` pour indiquer si les fonctions sont résolues par rapport au groupe de ressources et à l’abonnement pour le modèle parent ou le modèle imbriqué. Affectez à la propriété la valeur `inner` pour effectuer la résolution par rapport à l’étendue du modèle imbriqué. Affectez à la propriété la valeur `outer` pour effectuer la résolution par rapport à l’étendue du modèle parent.

Le tableau suivant indique si les fonctions sont résolues par rapport à l’abonnement ou au groupe de ressources parent ou incorporé.

| Type de modèle | Étendue | Résolution |
| ------------- | ----- | ---------- |
| Imbriqué        | Externe (par défaut) | Groupe de ressources parent |
| Imbriqué        | interne | Sous-groupe de ressources |
| Lié        | N/A   | Sous-groupe de ressources |

L’[exemple de modèle](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/crossresourcegroupproperties.json) suivant montre :

* Un modèle imbriqué avec l’étendue par défaut (externe)
* Un modèle imbriqué avec l’étendue interne
* Un modèle lié

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/crossresourcegroupproperties.json":::

Pour tester le modèle précédent et voir les résultats, utilisez PowerShell ou Azure CLI.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
New-AzResourceGroup -Name parentGroup -Location southcentralus
New-AzResourceGroup -Name inlineGroup -Location southcentralus
New-AzResourceGroup -Name linkedGroup -Location southcentralus

New-AzResourceGroupDeployment `
  -ResourceGroupName parentGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

La sortie de l’exemple précédent est :

```output
 Name             Type                       Value
 ===============  =========================  ==========
 parentRG         String                     Parent resource group is parentGroup
 defaultScopeRG   String                     Default scope resource group is parentGroup
 innerScopeRG     String                     Inner scope resource group is inlineGroup
 linkedRG         String                     Linked resource group is linkedgroup
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli-interactive
az group create --name parentGroup --location southcentralus
az group create --name inlineGroup --location southcentralus
az group create --name linkedGroup --location southcentralus

az deployment group create \
  --name ExampleDeployment \
  --resource-group parentGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

La sortie de l’exemple précédent est :

```output
"outputs": {
  "defaultScopeRG": {
    "type": "String",
    "value": "Default scope resource group is parentGroup"
  },
  "innerScopeRG": {
    "type": "String",
    "value": "Inner scope resource group is inlineGroup"
  },
  "linkedRG": {
    "type": "String",
    "value": "Linked resource group is linkedGroup"
  },
  "parentRG": {
    "type": "String",
    "value": "Parent resource group is parentGroup"
  }
},
```

---

## <a name="next-steps"></a>Étapes suivantes

* Pour comprendre comment définir des paramètres dans votre modèle, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](template-syntax.md).
* Pour obtenir des conseils sur la résolution des erreurs courantes de déploiement, consultez la page [Résolution des erreurs courantes de déploiement Azure avec Azure Resource Manager](common-deployment-errors.md).
* Pour plus d’informations sur le déploiement d’un modèle qui nécessite un jeton SAP, consultez [Déploiement d’un modèle privé avec un jeton SAP](secure-template-with-sas-token.md).
