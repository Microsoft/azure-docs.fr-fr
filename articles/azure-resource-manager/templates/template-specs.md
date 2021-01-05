---
title: Présentation des specs de modèle
description: Décrit comment créer des specs de modèle et les partager avec d’autres utilisateurs de votre organisation.
ms.topic: conceptual
ms.date: 11/25/2020
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: dd7616bdc07aee86ac56a2f2fe0ac282838c58b4
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518853"
---
# <a name="azure-resource-manager-template-specs-preview"></a>Specs de modèle Azure Resource Manager (préversion)

Une spec de modèle est un type de ressource permettant de stocker un modèle Resource Manager dans Azure en vue d’un déploiement ultérieur. Ce type de ressource vous permet de partager des modèles Ressource Manager avec d’autres utilisateurs de votre organisation. Comme toute autre ressource Azure, vous pouvez utiliser le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour partager la spec de modèle.

**Microsoft.Resources/templateSpecs** est le type de ressource pour les specs de modèle. Il se compose d’un modèle principal et d’un nombre quelconque de modèles liés. Azure stocke en toute sécurité les specs de modèle dans des groupes de ressources. Specs de modèle prend en charge le [contrôle de version](#versioning).

Pour déployer la spec de modèle, vous utilisez des outils Azure standard tels que PowerShell, Azure CLI, Portail Azure, REST et d’autres Kits de développement logiciel (SDK) et clients pris en charge. Vous utilisez les mêmes commandes que pour le modèle.

> [!NOTE]
> La fonctionnalité Specs de modèle est actuellement en préversion. Pour l’utiliser avec Azure PowerShell, vous devez installer la [version 5.0.0 ou ultérieure](/powershell/azure/install-az-ps). Pour l’utiliser avec Azure CLI, utilisez la [version 2.14.2 ou ultérieure](/cli/azure/install-azure-cli).

## <a name="why-use-template-specs"></a>Pourquoi utiliser des spécifications de modèle ?

Si vous avez actuellement vos modèles dans un GitHub référentiel ou un compte de stockage, vous rencontrez plusieurs défis quand vous essayez de partager et d’utiliser les modèles. Pour qu’un utilisateur le déploie, le modèle doit être local ou l’URL du modèle doit être accessible publiquement. Pour contourner cette limitation, vous pouvez partager des copies du modèle avec les utilisateurs qui ont besoin de le déployer, ou ouvrir l’accès au référentiel ou au compte de stockage. Lorsque les utilisateurs possèdent des copies locales d’un modèle, ces copies peuvent éventuellement séparer le modèle d’origine. Quand vous rendez un référentiel ou un compte de stockage accessible publiquement, des utilisateurs indésirables pourraient accéder au modèle.

L’avantage de l’utilisation de spécifications de modèle est que vous pouvez créer des modèles canoniques et les partager avec des équipes de votre organisation. Les specs de modèle sont sécurisées, car elles sont disponibles pour Azure Resource Manager pour le déploiement, mais ne sont pas accessibles aux utilisateurs sans autorisation Azure RBAC. Les utilisateurs ont uniquement besoin d’un accès en lecture à la spec de modèle pour en déployer le modèle, de sorte que vous pouvez partager le modèle sans permettre à d’autres utilisateurs de le modifier.

Les modèles que vous incluez dans une spec de modèle doivent être vérifiés par les administrateurs de votre organisation afin de respecter les exigences et les directives de l’organisation.

## <a name="create-template-spec"></a>Créer une spec de modèle

L’exemple suivant montre un modèle simple pour la création d’un compte de stockage dans Azure.

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
      ]
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-06-01",
      "name": "[concat('store', uniquestring(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "kind": "StorageV2",
      "sku": {
        "name": "[parameters('storageAccountType')]"
      }
    }
  ]
}
```

Lorsque vous créez la spec de modèle, les commandes PowerShell ou CLI sont transmises au fichier de modèle principal. Si le modèle principal fait référence à des modèles liés, les commandes les recherchent et les empaquettent pour créer la spec de modèle. Pour plus d’informations, consultez [Créer une spec de modèle avec des modèles liés](#create-a-template-spec-with-linked-templates).

Créez une spec de modèle en utilisant :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzTemplateSpec -Name storageSpec -Version 1.0a -ResourceGroupName templateSpecsRg -Location westus2 -TemplateFile ./mainTemplate.json
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az ts create \
  --name storageSpec \
  --version "1.0a" \
  --resource-group templateSpecRG \
  --location "westus2" \
  --template-file "./mainTemplate.json"
```

---

Vous pouvez afficher toutes les specs de modèle dans votre abonnement en utilisant :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az ts list
```

---

Vous pouvez afficher les détails d’une spec de modèle, notamment ses versions, avec :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzTemplateSpec -ResourceGroupName templateSpecsRG -Name storageSpec
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az ts show \
    --name storageSpec \
    --resource-group templateSpecRG \
    --version "1.0a"
```

---

## <a name="deploy-template-spec"></a>Déployer une spec de modèle

Une fois que vous avez créé la spec de modèle, les utilisateurs disposant d’un accès **en lecture** à la spec de modèle peuvent la déployer. Pour plus d’informations sur l’octroi de l’accès, consultez [Didacticiel : Accorder à un groupe l’accès aux ressources Azure à l’aide d’Azure PowerShell](../../role-based-access-control/tutorial-role-assignments-group-powershell.md).

Les specs de modèle peuvent être déployées via le portail, PowerShell, Azure CLI, ou en tant que modèle lié dans un déploiement de modèle plus volumineux. Les utilisateurs d’une organisation peuvent déployer une spec de modèle sur n’importe quelle étendue d’Azure (groupe de ressources, abonnement, groupe d’administration ou locataire).

Au lieu de passer par un chemin d’accès ou un URI pour un modèle, vous déployez une spec de modèle en fournissant son ID de ressource. L’ID de ressource possède le format suivant :

**/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Resources/templateSpecs/{template-spec-name}/versions/{template-spec-version}**

Notez que l’ID de la ressource contient un nom de version pour la spec de modèle.

Par exemple, vous déployez une spec de modèle à l’aide de la commande suivante.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0a"

New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
id = "/subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/templateSpecsRG/providers/Microsoft.Resources/templateSpecs/storageSpec/versions/1.0a"

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

En pratique, vous exécutez généralement `Get-AzTemplateSpec` ou `az ts show` pour obtenir l’ID de la spec de modèle à déployer.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
$id = (Get-AzTemplateSpec -Name storageSpec -ResourceGroupName templateSpecsRg -Version 1.0a).Versions.Id

New-AzResourceGroupDeployment `
  -ResourceGroupName demoRG `
  -TemplateSpecId $id
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
id = $(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0a" --query "id")

az deployment group create \
  --resource-group demoRG \
  --template-spec $id
```

---

## <a name="parameters"></a>Paramètres

Passer des paramètres au modèle de spécification revient exactement à passer des paramètres à un modèle ARM. Ajoutez les valeurs de paramètres en ligne ou dans un fichier de paramètres.

Pour passer un paramètre en ligne, utilisez :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -StorageAccountType Standard_GRS
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters storageAccountType='Standard_GRS'
```

---

Pour créer un fichier de paramètres locaux, utilisez :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageAccountType": {
      "value": "Standard_GRS"
    }
  }
}
```

Puis transmettez ce fichier de paramètres avec :

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell
New-AzResourceGroupDeployment `
  -TemplateSpecId $id `
  -ResourceGroupName demoRG `
  -TemplateParameterFile ./mainTemplate.parameters.json
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

```azurecli
az deployment group create \
  --resource-group demoRG \
  --template-spec $id \
  --parameters "./mainTemplate.parameters.json"
```

---

## <a name="create-a-template-spec-with-linked-templates"></a>Créer une spec de modèle avec des modèles liés

Si le modèle principal de votre spec de modèle fait référence à des modèles liés, les commandes PowerShell et CLI peuvent automatiquement trouver et empaqueter les modèles liés à partir de votre disque local. Vous n’avez pas besoin de configurer manuellement les référentiels ou les comptes de stockage pour héberger les specs de modèle : tout est autonome dans la ressource de spec de modèle.

L’exemple suivant est constitué d’un modèle principal et de deux modèles liés. L’exemple n’est qu’un extrait du modèle. Notez qu’il utilise une propriété nommée `relativePath` pour créer un lien vers les autres modèles. Vous devez utiliser `apiVersion` de `2020-06-01` ou une version ultérieure pour la ressource de déploiement.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/webapp.json"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "relativePath": "artifacts/database.json"
        }
      }
    }
  ],
  "outputs": {}
}
```

Lorsque la commande PowerShell ou CLI permettant de créer la spec de modèle est exécutée pour l’exemple précédent, la commande recherche trois fichiers (le modèle principal, le modèle d’application web [`webapp.json`] et le modèle de base de données [`database.json`]), et les empaquette dans la spec de modèle.

Pour plus d’informations, consultez [Didacticiel : Créer une spec de modèle avec des modèles liés](template-specs-create-linked.md).

## <a name="deploy-template-spec-as-a-linked-template"></a>Déployer la spec de modèle en tant que modèle lié

Une fois que vous avez créé une spec de modèle, il est facile de la réutiliser à partir d’un modèle Resource Manager ou d’une autre spec de modèle. Vous pouvez créer un lien vers une spec de modèle en ajoutant son ID de ressource à votre modèle. La spec de modèle lié est déployée automatiquement lorsque vous déployez le modèle principal. Ce comportement vous permet de développer des specs de modèle modulaires et de les réutiliser en fonction des besoins.

Par exemple, vous pouvez créer une spec de modèle qui déploie des ressources réseau et une autre spec de modèle qui déploie des ressources de stockage. Dans les modèles Resource Manager, vous pouvez établir un lien vers ces deux specs de modèle chaque fois que vous avez besoin de configurer des ressources réseau ou de stockage.

L’exemple suivant est similaire à l’exemple précédent, mais vous utilisez la propriété `id` pour créer un lien vers une spec de modèle plutôt que la propriété `relativePath` pour créer un lien vers un modèle local. Utilisez `2020-06-01` pour la version d’API de la ressource de déploiement. Dans l’exemple, les specs de modèle se trouvent dans un groupe de ressources nommé **templateSpecsRG**.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  ...
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "networkingDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'networkingSpec', '1.0a')]"
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2020-06-01",
      "name": "storageDeployment",
      ...
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "id": "[resourceId('templateSpecsRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '1.0a')]"
        }
      }
    }
  ],
  "outputs": {}
}
```

Pour plus d’informations sur la liaison de specs de modèle, consultez [Didacticiel : Déployer une spec de modèle en tant que modèle lié](template-specs-deploy-linked-template.md).

## <a name="versioning"></a>Gestion de version

Quand vous créez une spec de modèle, vous indiquez son nom de version. Lorsque vous itérez sur le code du modèle, vous pouvez mettre à jour une version existante (pour les correctifs logiciels) ou publier une nouvelle version. La version est une chaîne de texte. Vous êtes libre de choisir le système de contrôle de version, notamment le contrôle de version sémantique. Les utilisateurs de la spec de modèle peuvent fournir le nom de version à utiliser au moment de son déploiement.

## <a name="next-steps"></a>Étapes suivantes

* Pour créer et déployer une spec de modèle, consultez [Démarrage rapide : Créer et déployer une spec de modèle](quickstart-create-template-specs.md).

* Pour plus d’informations sur la liaison de modèles dans les specs de modèle, consultez [Didacticiel : Créer une spec de modèle avec des modèles liés](template-specs-create-linked.md).

* Pour plus d’informations sur le déploiement d’une spec de modèle en tant que modèle lié, consultez [Didacticiel : Déployer une spec de modèle en tant que modèle lié](template-specs-deploy-linked-template.md).
