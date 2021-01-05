---
title: Ajouter des attributions de rôle Azure avec des modèles Azure Resource Manager - Azure RBAC
description: Découvrez comment accorder l’accès aux ressources Azure pour les utilisateurs, groupes, principaux de service ou identités managées à l’aide des modèles Azure Resource Manager et du contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/13/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 9bdd70baa906d9dc03a37eecb0388eee5638f153
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184278"
---
# <a name="add-azure-role-assignments-using-azure-resource-manager-templates"></a>Ajouter des attributions de rôle Azure à l’aide de modèles Azure Resource Manager

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control-definition-grant.md)] En plus d’utiliser Azure PowerShell ou l’interface Azure CLI, vous pouvez attribuer des rôles à l’aide des [modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md). Les modèles peuvent être utiles si vous devez déployer les ressources de manière cohérente et répétée. Cet article explique comment attribuer des rôles à l’aide de modèles.

## <a name="get-object-ids"></a>Récupérer des ID d’objet

Pour attribuer un rôle, vous devez spécifier l’ID de l’utilisateur, du groupe ou de l’application auxquels vous souhaitez attribuer le rôle. L’ID a le format : `11111111-1111-1111-1111-111111111111`. Vous pouvez récupérer l’ID à l’aide du Portail Azure, d’Azure PowerShell ou d’Azure CLI.

### <a name="user"></a>Utilisateur

Pour récupérer l’ID d’un utilisateur, vous pouvez utiliser les commandes [Get-AzADUser](/powershell/module/az.resources/get-azaduser) ou [az ad user show](/cli/azure/ad/user#az-ad-user-show).

```azurepowershell
$objectid = (Get-AzADUser -DisplayName "{name}").id
```

```azurecli
objectid=$(az ad user show --id "{email}" --query objectId --output tsv)
```

### <a name="group"></a>Groupe

Pour récupérer l’ID d’un groupe, vous pouvez utiliser les commandes [Get-AzADGroup](/powershell/module/az.resources/get-azadgroup) ou [az ad group show](/cli/azure/ad/group#az-ad-group-show).

```azurepowershell
$objectid = (Get-AzADGroup -DisplayName "{name}").id
```

```azurecli
objectid=$(az ad group show --group "{name}" --query objectId --output tsv)
```

### <a name="managed-identities"></a>Identités managées

Pour obtenir l’ID d’une identité managée, vous pouvez utiliser les commandes [Get-AzAdServiceprincipal](/powershell/module/az.resources/get-azadserviceprincipal) ou [az ad sp](/cli/azure/ad/sp).

```azurepowershell
$objectid = (Get-AzADServicePrincipal -DisplayName <Azure resource name>).id
```

```azurecli
objectid=$(az ad sp list --display-name <Azure resource name> --query [].objectId --output tsv)
```

### <a name="application"></a>Application

Pour récupérer l’ID d’un principal de service (identité utilisée par une application), vous pouvez utiliser les commandes [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal) ou [az ad sp list](/cli/azure/ad/sp#az-ad-sp-list). Pour un principal de service, utilisez l’ID d’objet et **non** l’ID d’application.

```azurepowershell
$objectid = (Get-AzADServicePrincipal -DisplayName "{name}").id
```

```azurecli
objectid=$(az ad sp list --display-name "{name}" --query [].objectId --output tsv)
```

## <a name="add-a-role-assignment"></a>Ajouter une attribution de rôle

Dans Azure RBAC, vous ajoutez une attribution de rôle pour accorder l’accès.

### <a name="resource-group-scope-without-parameters"></a>Étendue du groupe de ressources (sans paramètres)

Le modèle suivant montre comment ajouter de façon très simple une attribution de rôle. Certaines valeurs sont spécifiées dans le modèle. Le modèle suivant montre comment :

-  Attribuer un rôle [lecteur](built-in-roles.md#reader) à un utilisateur, un groupe ou une application dans une étendue de groupe de ressources

Pour utiliser le modèle, vous devez effectuer les opérations suivantes :

- Créez un fichier JSON et copiez le modèle.
- Remplacez `<your-principal-id>` par l’ID d’un utilisateur, d’un groupe, d’une identité managée ou d’une application auxquels attribuer le rôle

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[guid(resourceGroup().id)]",
            "properties": {
                "roleDefinitionId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]",
                "principalId": "<your-principal-id>"
            }
        }
    ]
}
```

Les exemples suivants, avec les commandes [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) et [az group deployment create](/cli/azure/group/deployment#az-group-deployment-create), montrent comment démarrer le déploiement dans un groupe de ressources nommé ExampleGroup.

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile rbac-test.json
```

```azurecli
az group deployment create --resource-group ExampleGroup --template-file rbac-test.json
```

L’exemple suivant illustre l’attribution du rôle lecteur à un utilisateur pour un groupe de ressources après le déploiement du modèle.

![Attribution de rôle dans une étendue de groupe de ressources](./media/role-assignments-template/role-assignment-template.png)

### <a name="resource-group-or-subscription-scope"></a>Étendue du groupe de ressources ou de l’abonnement

Le modèle précédent n’est pas très flexible. Le modèle suivant utilise des paramètres et peut s’appliquer à différentes étendues. Le modèle suivant montre comment :

- Attribuer un rôle à un utilisateur, un groupe ou une application dans une étendue de groupe de ressources ou d’abonnement
- Spécifier les rôles Propriétaire, Collaborateur et Lecteur comme paramètre

Pour utiliser le modèle, vous devez spécifier les entrées suivantes :

- L’ID d’un utilisateur, d’un groupe, d’une identité managée ou d’une application auxquels attribuer le rôle
- Un identificateur unique qui sera utilisé pour l’attribution de rôle, ou vous pouvez utiliser l’ID par défaut

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "principalId": {
            "type": "string",
            "metadata": {
                "description": "The principal to assign the role to"
            }
        },
        "builtInRoleType": {
            "type": "string",
            "allowedValues": [
                "Owner",
                "Contributor",
                "Reader"
            ],
            "metadata": {
                "description": "Built-in role to assign"
            }
        },
        "roleNameGuid": {
            "type": "string",
            "defaultValue": "[newGuid()]",
            "metadata": {
                "description": "A new GUID used to identify the role assignment"
            }
        }
    },
    "variables": {
        "Owner": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', '8e3af657-a8ff-443c-a75c-2fe8c4bcb635')]",
        "Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]",
        "Reader": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]"
    },
    "resources": [
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[parameters('roleNameGuid')]",
            "properties": {
                "roleDefinitionId": "[variables(parameters('builtInRoleType'))]",
                "principalId": "[parameters('principalId')]"
            }
        }
    ]
}
```

> [!NOTE]
> Ce modèle n’est pas idempotent, à moins que la même valeur de `roleNameGuid` soit fournie comme paramètre pour chaque déploiement du modèle. Si aucune valeur de `roleNameGuid` n’est fournie, par défaut, un nouveau GUID est généré à chaque déploiement et les déploiements suivants échouent avec une erreur `Conflict: RoleAssignmentExists`.

L’étendue de l’attribution de rôle est déterminée à partir du niveau du déploiement. Les exemples suivants, avec les commandes [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) et [az group deployment create](/cli/azure/group/deployment#az-group-deployment-create), montrent comment démarrer le déploiement dans une étendue de groupe de ressources.

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile rbac-test.json -principalId $objectid -builtInRoleType Reader
```

```azurecli
az group deployment create --resource-group ExampleGroup --template-file rbac-test.json --parameters principalId=$objectid builtInRoleType=Reader
```

Les exemples suivants, avec les commandes [New-AzDeployment](/powershell/module/az.resources/new-azdeployment) et [az deployment create](/cli/azure/deployment#az-deployment-create), montrent comment démarrer le déploiement dans une étendue d’abonnement et spécifier l’emplacement.

```azurepowershell
New-AzDeployment -Location centralus -TemplateFile rbac-test.json -principalId $objectid -builtInRoleType Reader
```

```azurecli
az deployment create --location centralus --template-file rbac-test.json --parameters principalId=$objectid builtInRoleType=Reader
```

### <a name="resource-scope"></a>Étendue des ressources

Si vous avez besoin d’ajouter une attribution de rôle au niveau d’une ressource, définissez la propriété `scope` de l’attribution de rôle à l’aide du nom de la ressource.

Le modèle suivant montre comment :

- Création d’un nouveau compte de stockage
- Attribuer un rôle à un utilisateur, un groupe ou une application dans l’étendue d’un compte de stockage
- Spécifier les rôles Propriétaire, Collaborateur et Lecteur comme paramètre

Pour utiliser le modèle, vous devez spécifier les entrées suivantes :

- L’ID d’un utilisateur, d’un groupe, d’une identité managée ou d’une application auxquels attribuer le rôle

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "principalId": {
            "type": "string",
            "metadata": {
                "description": "The principal to assign the role to"
            }
        },
        "builtInRoleType": {
            "type": "string",
            "allowedValues": [
                "Owner",
                "Contributor",
                "Reader"
            ],
            "metadata": {
                "description": "Built-in role to assign"
            }
        },
        "roleNameGuid": {
            "type": "string",
            "defaultValue": "[newGuid()]",
            "metadata": {
                "description": "A new GUID used to identify the role assignment"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {
        "Owner": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', '8e3af657-a8ff-443c-a75c-2fe8c4bcb635')]",
        "Contributor": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]",
        "Reader": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'acdd72a7-3385-48ef-bd42-f606fba81ae7')]",
        "storageName": "[concat('storage', uniqueString(resourceGroup().id))]"
    },
    "resources": [
        {
            "apiVersion": "2019-04-01",
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {}
        },
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2020-04-01-preview",
            "name": "[parameters('roleNameGuid')]",
            "scope": "[concat('Microsoft.Storage/storageAccounts', '/', variables('storageName'))]",
            "dependsOn": [
                "[variables('storageName')]"
            ],
            "properties": {
                "roleDefinitionId": "[variables(parameters('builtInRoleType'))]",
                "principalId": "[parameters('principalId')]"
            }
        }
    ]
}
```

Pour déployer le modèle précédent, vous devez utiliser les commandes de groupe de ressources. Les exemples suivants, avec les commandes [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) et [az group deployment create](/cli/azure/group/deployment#az-group-deployment-create), montrent comment démarrer le déploiement dans une étendue de ressource.

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile rbac-test.json -principalId $objectid -builtInRoleType Contributor
```

```azurecli
az group deployment create --resource-group ExampleGroup --template-file rbac-test.json --parameters principalId=$objectid builtInRoleType=Contributor
```

L’exemple suivant illustre l’attribution du rôle contributeur à un utilisateur pour un compte de stockage après le déploiement du modèle.

![Attribution de rôle dans une étendue de ressource](./media/role-assignments-template/role-assignment-template-resource.png)

### <a name="new-service-principal"></a>Nouveau principal de service

Dans certains cas, si vous créez un principal de service et que vous tentez immédiatement de lui attribuer un rôle, cette attribution peut échouer. Par exemple, si vous créez une identité managée et que vous tentez d’attribuer un rôle à ce principal de service dans le même modèle Azure Resource Manager, l’attribution de rôle peut échouer. Cet échec est souvent lié au délai de réplication. Le principal du service est créé dans une région. Toutefois, l’attribution de rôle peut s’effectuer dans une autre région, qui n’a pas encore répliqué le principal de service.

Dans le cadre de ce scénario, vous devez définir la propriété `principalType` sur `ServicePrincipal` lors de la création de l’attribution de rôle. Vous devez également définir `apiVersion` de l’attribution de rôle sur `2018-09-01-preview` ou une version ultérieure.

Le modèle suivant montre comment :

- Créer un principal de service d’identité managée
- Spécifier le `principalType`
- Attribuer le rôle contributeur à ce principal de service dans une étendue de groupe de ressources

Pour utiliser le modèle, vous devez spécifier les entrées suivantes :

- Le nom de base de l’identité managée. Vous pouvez également utiliser la chaîne par défaut.

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "baseName": {
            "type": "string",
            "defaultValue": "msi-test"
        }
    },
    "variables": {
        "identityName": "[concat(parameters('baseName'), '-bootstrap')]",
        "bootstrapRoleAssignmentId": "[guid(concat(resourceGroup().id, 'contributor'))]",
        "contributorRoleDefinitionId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/', 'b24988ac-6180-42a0-ab88-20f7382dd24c')]"
    },
    "resources": [
        {
            "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
            "name": "[variables('identityName')]",
            "apiVersion": "2018-11-30",
            "location": "[resourceGroup().location]"
        },
        {
            "type": "Microsoft.Authorization/roleAssignments",
            "apiVersion": "2018-09-01-preview",
            "name": "[variables('bootstrapRoleAssignmentId')]",
            "dependsOn": [
                "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
            ],
            "properties": {
                "roleDefinitionId": "[variables('contributorRoleDefinitionId')]",
                "principalId": "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName')), '2018-11-30').principalId]",
                "scope": "[resourceGroup().id]",
                "principalType": "ServicePrincipal"
            }
        }
    ]
}
```

Les exemples suivants, avec les commandes [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) et [az group deployment create](/cli/azure/group/deployment#az-group-deployment-create), montrent comment démarrer le déploiement dans une étendue de groupe de ressources.

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup2 -TemplateFile rbac-test.json
```

```azurecli
az group deployment create --resource-group ExampleGroup2 --template-file rbac-test.json
```

L’exemple suivant illustre l’attribution du rôle contributeur à un nouveau principal de service d’identité managée après le déploiement du modèle.

![Attribution de rôle pour un nouveau principal de service d’identité managée](./media/role-assignments-template/role-assignment-template-msi.png)

## <a name="remove-a-role-assignment"></a>Supprimer une attribution de rôle

Dans Azure RBAC, vous devez supprimer l’attribution de rôle pour supprimer l’accès à une ressource Azure. Il n’existe aucun moyen de supprimer une attribution de rôle à l’aide d’un modèle. Pour supprimer une attribution de rôle, vous devez utiliser d’autres outils tels que :

- [Azure portal](role-assignments-portal.md#remove-a-role-assignment)
- [Azure PowerShell](role-assignments-powershell.md#remove-a-role-assignment)
- [Azure CLI](role-assignments-cli.md#remove-a-role-assignment)
- [REST API](role-assignments-rest.md#remove-a-role-assignment)

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide : Créer et déployer des modèles Azure Resource Manager à l’aide du portail Azure](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md)
- [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
- [Créer des groupes de ressources et des ressources au niveau de l’abonnement](../azure-resource-manager/templates/deploy-to-subscription.md)
- [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?term=rbac)
