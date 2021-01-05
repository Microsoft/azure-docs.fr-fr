---
title: 'Démarrage rapide : Créer un blueprint avec PowerShell'
description: Dans ce guide de démarrage rapide, vous allez utiliser Azure Blueprints pour créer, définir et déployer des artefacts avec PowerShell.
ms.date: 08/27/2020
ms.topic: quickstart
ms.openlocfilehash: 339cd4628cda5f469a783db02c10f86259c93941
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89051489"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-powershell"></a>Démarrage rapide : Définir et affecter un blueprint Azure avec PowerShell

Un utilisateur qui sait comment créer et affecter des blueprints peut définir des modèles courants et développer des configurations réutilisables et rapides à déployer en fonction de modèles Azure Resource Manager (modèles ARM), de stratégies, d’exigences en matière de sécurité, etc. Dans ce tutoriel, vous allez découvrir comment utiliser Azure Blueprint pour effectuer des tâches courantes liées à la création, à la publication et à l’affectation d’un blueprint dans votre organisation, notamment :

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

Si le module n’est pas encore installé, suivez les instructions fournies dans [Ajouter le module Az.Blueprint](./how-to/manage-assignments-ps.md#add-the-azblueprint-module) pour installer et valider le module **AZ.Blueprint** à partir de PowerShell Gallery.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-a-blueprint"></a>Créer un blueprint

La première étape de la définition d’un modèle standard à des fins de conformité est de composer un blueprint à partir des ressources disponibles. Nous allons créer un blueprint nommé « MyBlueprint » pour configurer les attributions de rôle et de stratégie pour l’abonnement. Ensuite, nous allons ajouter un groupe de ressources, un modèle Resource Manager et une attribution de rôle sur le groupe de ressources.

> [!NOTE]
> Quand PowerShell est utilisé, l’objet _blueprint_ est créé en premier. Pour chaque _artefact_ à ajouter contenant des paramètres, vous devez définir ces paramètres à l’avance sur le _blueprint_ initial.

1. Créez l’objet _blueprint_ initial. Le paramètre **BlueprintFile** prend un fichier JSON qui comprend les propriétés du blueprint, les groupes de ressources à créer et tous les paramètres au niveau du blueprint. Les paramètres sont définis durant l’affectation et utilisés par les artefacts ajoutés aux étapes ultérieures.

   - Fichier JSON – blueprint.json

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
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
                         "displayName": "storage account type.",
                         "description": null
                     }
                 },
                 "tagName": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The name of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "tagValue": {
                     "type": "string",
                     "metadata": {
                         "displayName": "The value of the tag to provide the policy assignment.",
                         "description": null
                     }
                 },
                 "contributors": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription",
                         "strongType": "PrincipalId"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group",
                         "strongType": "PrincipalId"
                     }
                 }
             },
             "resourceGroups": {
                 "storageRG": {
                     "description": "Contains the resource template deployment and a role assignment."
                 }
             }
         }
     }
     ```

   - Commande PowerShell

     ```azurepowershell-interactive
     # Login first with Connect-AzAccount if not using Cloud Shell

     # Get a reference to the new blueprint object, we'll use it in subsequent steps
     $blueprint = New-AzBlueprint -Name 'MyBlueprint' -BlueprintFile .\blueprint.json
     ```

     > [!NOTE]
     > Utilisez le nom de fichier _blueprint.json_ quand vous créez vos définitions de blueprint programmatiquement.
     > Ce nom de fichier est utilisé au moment d’appeler [Import-AzBlueprintWithArtifact](/powershell/module/az.blueprint/import-azblueprintwithartifact).

     Par défaut, l’objet blueprint est créé dans l’abonnement par défaut. Pour spécifier le groupe d’administration, utilisez le paramètre **ManagementGroupId**. Pour spécifier l’abonnement, utilisez le paramètre **SubscriptionId**.

1. Ajoutez une attribution de rôle au niveau de l’abonnement. **ArtifactFile** définit le _genre_ d’artefact, les propriétés varient en fonction de l’identificateur de définition de rôle et les identités de principal sont passées sous forme de tableau de valeurs. Dans l’exemple ci-dessous, les identités de principal ayant reçu le rôle spécifié sont configurées avec un paramètre qui est défini durant l’affectation du blueprint. Cet exemple utilise le rôle intégré _Contributeur_ avec le GUID `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   - Fichier JSON – \artifacts\roleContributor.json

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

   - Commande PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'roleContributor' -ArtifactFile .\artifacts\roleContributor.json
     ```

1. Ajoutez une affectation de stratégie au niveau de l’abonnement. **ArtifactFile** définit le _genre_ d’artefact, les propriétés qui varient en fonction de la définition d’une stratégie ou d’une initiative, et configure l’affectation de stratégie pour utiliser les paramètres de blueprint définis (à configurer durant l’affectation du blueprint). Cet exemple utilise la stratégie intégrée _Appliquer l’étiquette et sa valeur par défaut aux groupes de ressources_ avec le GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Fichier JSON – \artifacts\policyTags.json

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "displayName": "Apply tag and its default value to resource groups",
             "description": "Apply tag and its default value to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "[parameters('tagName')]"
                 },
                 "tagValue": {
                     "value": "[parameters('tagValue')]"
                 }
             }
         }
     }
     ```

   - Commande PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'policyTags' -ArtifactFile .\artifacts\policyTags.json
     ```

1. Ajoutez une autre affectation de stratégie pour l’étiquette Storage (réutilisation du paramètre _storageAccountType_) au niveau de l’abonnement. Cet artefact d’affectation de stratégie supplémentaire montre qu’un paramètre défini sur le blueprint peut être utilisé par plusieurs artefacts. Dans l’exemple, l’élément **storageAccountType** sert à définir une étiquette sur le groupe de ressources. Cette valeur fournit des informations sur le compte de stockage créé à l’étape suivante. Cet exemple utilise la stratégie intégrée _Appliquer l’étiquette et sa valeur par défaut aux groupes de ressources_ avec le GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Fichier JSON – \artifacts\policyStorageTags.json

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
             "displayName": "Apply storage tag to resource group",
             "description": "Apply storage tag and the parameter also used by the template to resource groups",
             "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71",
             "parameters": {
                 "tagName": {
                     "value": "StorageType"
                 },
                 "tagValue": {
                     "value": "[parameters('storageAccountType')]"
                 }
             }
         }
     }
     ```

   - Commande PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'policyStorageTags' -ArtifactFile .\artifacts\policyStorageTags.json
     ```

1. Ajoutez le modèle sous le groupe de ressources. Le **TemplateFile** d’un modèle Resource Manager comprend le composant JSON normal du modèle. Le modèle réutilise également les paramètres de blueprint **storageAccountType**, **tagName** et **tagValue** en transmettant chacun au modèle. Les paramètres de blueprint sont accessibles au modèle en utilisant le paramètre **TemplateParameterFile** et dans le fichier JSON, cette paire clé/valeur est utilisée pour injecter la valeur. Les noms de paramètres de blueprint et de modèle peuvent être les mêmes.

   - Fichier de modèle JSON Resource Manager – \artifacts\templateStorage.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "type": "string",
                 "metadata": {
                     "description": "Storage Account type"
                 }
             },
             "tagNameFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag name from blueprint"
                 }
             },
             "tagValueFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag value from blueprint"
                 }
             }
         },
         "variables": {
             "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
         },
         "resources": [{
             "type": "Microsoft.Storage/storageAccounts",
             "name": "[variables('storageAccountName')]",
             "apiVersion": "2016-01-01",
             "tags": {
                 "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
             },
             "location": "[resourceGroup().location]",
             "sku": {
                 "name": "[parameters('storageAccountTypeFromBP')]"
             },
             "kind": "Storage",
             "properties": {}
         }],
         "outputs": {
             "storageAccountSku": {
                 "type": "string",
                 "value": "[variables('storageAccountName')]"
             }
         }
     }
     ```

   - Fichier de paramètres de modèle JSON Resource Manager – \artifacts\templateStorageParams.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "value": "[parameters('storageAccountType')]"
             },
             "tagNameFromBP": {
                 "value": "[parameters('tagName')]"
             },
             "tagValueFromBP": {
                 "value": "[parameters('tagValue')]"
             }
         }
     }
     ```

   - Commande PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Type TemplateArtifact -Name 'templateStorage' -TemplateFile .\artifacts\templateStorage.json -TemplateParameterFile .\artifacts\templateStorageParams.json -ResourceGroupName storageRG
     ```

1. Ajoutez une attribution de rôle sous le groupe de ressources. Comme pour l’entrée d’attribution de rôle précédente, l’exemple ci-dessous utilise l’identificateur de définition pour le rôle **Propriétaire** et lui fournit un paramètre différent à partir du blueprint. Cet exemple utilise le rôle intégré _Propriétaire_ avec le GUID `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   - Fichier JSON – \artifacts\roleOwner.json

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "resourceGroup": "storageRG",
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635",
             "principalIds": "[parameters('owners')]"
         }
     }
     ```

   - Commande PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintArtifact -Blueprint $blueprint -Name 'roleOwner' -ArtifactFile .\artifacts\roleOwner.json
     ```

## <a name="publish-a-blueprint"></a>Publier un blueprint

Maintenant que les artefacts ont été ajoutés au blueprint, il est temps de le publier. Une fois un blueprint publié, il peut être affecté à un abonnement.

```azurepowershell-interactive
# Use the reference to the new blueprint object from the previous steps
Publish-AzBlueprint -Blueprint $blueprint -Version '{BlueprintVersion}'
```

La valeur de `{BlueprintVersion}` est une chaîne de lettres, de chiffres et de traits d’union qui ne doit pas dépasser 20 caractères et qui ne doit pas contenir d’espaces ou d’autres caractères spéciaux. Utilisez un nom unique et informatif tel que **v20180622-135541**.

## <a name="assign-a-blueprint"></a>Affecter un blueprint

Une fois qu’un blueprint a été publié à l’aide de PowerShell, il peut être affecté à un abonnement. Affectez le blueprint que vous avez créé à l’un des abonnements sous votre hiérarchie de groupes d’administration. Si le blueprint est enregistré dans un abonnement, il ne peut être attribué qu’à cet abonnement. Le paramètre **Blueprint** spécifie le blueprint à affecter. Pour fournir les paramètres name, location, identity, lock et blueprint, utilisez les paramètres PowerShell correspondants dans l’applet de commande `New-AzBlueprintAssignment` ou fournissez-les dans le fichier JSON de paramètres **AssignmentFile**.

1. Exécutez le déploiement du blueprint en l’affectant à un abonnement. Les paramètres **contributors** et **owners** nécessitant l’octroi de l’attribution de rôle à un tableau d’ID d’objet des principaux, utilisez l’[API Graph d’Azure Active Directory](../../active-directory/develop/active-directory-graph-api.md) pour collecter les ID d’objet à utiliser dans le fichier **AssignmentFile** pour vos propres utilisateurs, groupes ou principaux de service.

   - Fichier JSON – blueprintAssignment.json

     ```json
     {
         "properties": {
             "blueprintId": "/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint",
             "resourceGroups": {
                 "storageRG": {
                     "name": "StorageAccount",
                     "location": "eastus2"
                 }
             },
             "parameters": {
                 "storageAccountType": {
                     "value": "Standard_GRS"
                 },
                 "tagName": {
                     "value": "CostCenter"
                 },
                 "tagValue": {
                     "value": "ContosoIT"
                 },
                 "contributors": {
                     "value": [
                         "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                         "38833b56-194d-420b-90ce-cff578296714"
                     ]
                 },
                 "owners": {
                     "value": [
                         "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                         "316deb5f-7187-4512-9dd4-21e7798b0ef9"
                     ]
                 }
             }
         },
         "identity": {
             "type": "systemAssigned"
         },
         "location": "westus"
     }
     ```

   - Commande PowerShell

     ```azurepowershell-interactive
     # Use the reference to the new blueprint object from the previous steps
     New-AzBlueprintAssignment -Blueprint $blueprint -Name 'assignMyBlueprint' -AssignmentFile .\blueprintAssignment.json
     ```

   - Identité managée affectée par l’utilisateur

     Une affectation de blueprint peut également utiliser une [identité managée affectée par l’utilisateur](../../active-directory/managed-identities-azure-resources/overview.md).
     Dans ce cas, la partie **identity** du fichier d’affectation JSON change de la façon suivante. Remplacez `{tenantId}`, `{subscriptionId}``{yourRG}` et `{userIdentity}` par vos ID de locataire, ID d’abonnement et nom de groupe de ressources et par le nom de votre identité managée affectée par l’utilisateur, respectivement.

     ```json
     "identity": {
         "type": "userAssigned",
         "tenantId": "{tenantId}",
         "userAssignedIdentities": {
             "/subscriptions/{subscriptionId}/resourceGroups/{yourRG}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userIdentity}": {}
         }
     },
     ```

     L’**identité managée affectée par l’utilisateur** peut être dans n’importe quels abonnement et groupe de ressources pour lesquels l’utilisateur qui affecte le blueprint a les autorisations nécessaires.

     > [!IMPORTANT]
     > Azure Blueprints ne gère pas l’identité managée affectée par l’utilisateur. Les utilisateurs doivent attribuer des rôles et autorisations suffisants. À défaut, l’affectation du blueprint échouera.

## <a name="clean-up-resources"></a>Nettoyer les ressources

### <a name="unassign-a-blueprint"></a>Annuler l’affectation d’un blueprint

Vous pouvez supprimer un blueprint d’un abonnement. La suppression est souvent effectuée lorsque les ressources d’artefact ne sont plus nécessaires. Quand un blueprint est supprimé, les artefacts affectés dans le cadre de ce blueprint sont abandonnés. Pour supprimer une affectation de blueprint, utilisez l’applet de commande `Remove-AzBlueprintAssignment` :

assignMyBlueprint

```azurepowershell-interactive
Remove-AzBlueprintAssignment -Name 'assignMyBlueprint'
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé, affecté et supprimé un blueprint à l’aide de PowerShell. Pour plus d’informations sur Azure Blueprints, consultez l’article concernant le cycle de vie des blueprints.

> [!div class="nextstepaction"]
> [En savoir plus sur le cycle de vie des blueprints](./concepts/lifecycle.md)