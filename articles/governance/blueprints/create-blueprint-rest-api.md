---
title: "Démarrage rapide : Créer un blueprint avec l'API REST"
description: Dans ce guide de démarrage rapide, vous allez utiliser Azure Blueprints pour créer, définir et déployer des artefacts avec l’API REST.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: aa25a65b20b295045b52a49c8c47fb8849c3cba3
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92056972"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-rest-api"></a>Démarrage rapide : Définir et affecter un blueprint Azure avec l’API REST

Un utilisateur qui sait comment créer et affecter des blueprints peut définir des modèles courants et développer des configurations réutilisables et rapides à déployer en fonction de modèles Azure Resource Manager (modèles ARM), de stratégies, d’exigences en matière de sécurité, etc. Dans ce tutoriel, vous allez découvrir comment utiliser Azure Blueprint pour effectuer des tâches courantes liées à la création, à la publication et à l’affectation d’un blueprint dans votre organisation, notamment :

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.
- Inscrivez le fournisseur de ressources `Microsoft.Blueprint`. Pour obtenir des instructions, consultez [Types et fournisseurs de ressources](../../azure-resource-manager/management/resource-providers-and-types.md).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="getting-started-with-rest-api"></a>Bien démarrer avec l’API REST

Si vous ne connaissez pas bien l’API REST, commencez par passer en revue [Référence de l’API REST Azure](/rest/api/azure/) pour obtenir des informations générales sur l’API REST, en particulier sur l’URI de requête et le corps de la requête. Cet article utilise ces concepts pour fournir des instructions sur l’utilisation des blueprints Azure et suppose une connaissance pratique de ces derniers. Des outils comme [ARMClient](https://github.com/projectkudu/ARMClient) qui peuvent gérer l’autorisation automatiquement sont recommandés pour les débutants.

Pour les caractéristiques Azure Blueprints, consultez [API REST Azure Blueprints](/rest/api/blueprints/).

### <a name="rest-api-and-powershell"></a>API REST et PowerShell

Si vous n’avez pas encore d’outil pour effectuer des appels d’API REST, songez à utiliser PowerShell pour ces instructions. Voici un exemple d’en-tête pour l’authentification auprès d’Azure. Générez un en-tête d’authentification, parfois appelé **jeton du porteur**, puis indiquez l’URI d’API REST auquel se connecter avec un paramètre ou un **corps de demande** :

```azurepowershell-interactive
# Log in first with Connect-AzAccount if not using Cloud Shell

$azContext = Get-AzContext
$azProfile = [Microsoft.Azure.Commands.Common.Authentication.Abstractions.AzureRmProfileProvider]::Instance.Profile
$profileClient = New-Object -TypeName Microsoft.Azure.Commands.ResourceManager.Common.RMProfileClient -ArgumentList ($azProfile)
$token = $profileClient.AcquireAccessToken($azContext.Subscription.TenantId)
$authHeader = @{
    'Content-Type'='application/json'
    'Authorization'='Bearer ' + $token.AccessToken
}

# Invoke the REST API
$restUri = 'https://management.azure.com/subscriptions/{subscriptionId}?api-version=2020-01-01'
$response = Invoke-RestMethod -Uri $restUri -Method Get -Headers $authHeader
```

Remplacez `{subscriptionId}` dans la variable **$restUri** ci-dessus pour obtenir des informations sur votre abonnement. La variable $response contient le résultat de l’applet de commande `Invoke-RestMethod` que vous pouvez analyser avec des applets de commande tels que [ConvertFrom-Json](/powershell/module/microsoft.powershell.utility/convertfrom-json). Si le point de terminaison de service de l’API REST attend un **corps de demande**, fournissez une variable au format JSON au paramètre `-Body` de `Invoke-RestMethod`.

## <a name="create-a-blueprint"></a>Créer un blueprint

La première étape de la définition d’un modèle standard à des fins de conformité est de composer un blueprint à partir des ressources disponibles. Nous allons créer un blueprint nommé « MyBlueprint » pour configurer les attributions de rôle et de stratégie pour l’abonnement. Ensuite, nous allons ajouter un groupe de ressources, un modèle Resource Manager et une attribution de rôle sur le groupe de ressources.

> [!NOTE]
> Quand vous utilisez l’API REST, l’objet _blueprint_ est créé en premier. Pour chaque _artefact_ à ajouter contenant des paramètres, vous devez définir ces paramètres à l’avance sur le _blueprint_ initial.

Dans chaque URI d’API REST, vous devez remplacer les variables utilisées par vos propres valeurs :

- Remplacer `{YourMG}` par l’ID de votre groupe d’administration
- Remplacer `{subscriptionId}` par votre ID d’abonnement

> [!NOTE]
> Les blueprints peuvent aussi être créés au niveau de l’abonnement. Pour plus d’informations, consultez cet [exemple de création de blueprint au niveau de l’abonnement](/rest/api/blueprints/blueprints/createorupdate#subscriptionblueprint).

1. Créez l’objet _blueprint_ initial. Le **corps de la requête** inclut des propriétés du blueprint, les groupes de ressources à créer et tous les paramètres au niveau du blueprint. Les paramètres sont définis durant l’affectation et utilisés par les artefacts ajoutés aux étapes ultérieures.

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2018-11-01-preview
     ```

   - Corps de la requête

     ```json
     {
         "properties": {
             "description": "This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.",
             "targetScope": "subscription",
             "parameters": {
                 "storageAccountType": {
                     "type": "string",
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
                         "description": "List of AAD object IDs that is assigned Contributor role at the subscription"
                     }
                 },
                 "owners": {
                     "type": "array",
                     "metadata": {
                         "description": "List of AAD object IDs that is assigned Owner role at the resource group"
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

1. Ajoutez une attribution de rôle au niveau de l’abonnement. Le **corps de la demande** définit le _genre_ d’artefact, les propriétés sont alignées avec l’identificateur de définition de rôle et les identités de principal sont passées en tant que tableau de valeurs. Dans l’exemple ci-dessous, les identités de principal ayant reçu le rôle spécifié sont configurées avec un paramètre qui est défini durant l’affectation du blueprint. Cet exemple utilise le rôle intégré _Contributeur_ avec le GUID `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleContributor?api-version=2018-11-01-preview
     ```

   - Corps de la requête

     ```json
     {
         "kind": "roleAssignment",
         "properties": {
             "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
             "principalIds": "[parameters('contributors')]"
         }
     }
     ```

1. Ajoutez une affectation de stratégie au niveau de l’abonnement. Le **corps de la requête** définit le _genre_ d’artefact, définit les propriétés qui sont alignées sur une définition de stratégie ou d’initiative, et configure l’affectation de stratégie pour utiliser les paramètres de blueprint définis (à configurer durant l’affectation du blueprint). Cet exemple utilise la stratégie intégrée _Appliquer l’étiquette et sa valeur par défaut aux groupes de ressources_ avec le GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyTags?api-version=2018-11-01-preview
     ```

   - Corps de la requête

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
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

1. Ajoutez une autre affectation de stratégie pour l’étiquette Storage (réutilisation du paramètre _storageAccountType_) au niveau de l’abonnement. Cet artefact d’affectation de stratégie supplémentaire montre qu’un paramètre défini sur le blueprint peut être utilisé par plusieurs artefacts. Dans l’exemple, l’élément **storageAccountType** sert à définir une étiquette sur le groupe de ressources. Cette valeur fournit des informations sur le compte de stockage créé à l’étape suivante. Cet exemple utilise la stratégie intégrée _Appliquer l’étiquette et sa valeur par défaut aux groupes de ressources_ avec le GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/policyStorageTags?api-version=2018-11-01-preview
     ```

   - Corps de la requête

     ```json
     {
         "kind": "policyAssignment",
         "properties": {
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

1. Ajoutez le modèle sous le groupe de ressources. Le **corps de la requête** pour un modèle Resource Manager inclut le composant JSON normal du modèle et définit le groupe de ressources cible avec **properties.resourceGroup**. Le modèle réutilise également les paramètres de blueprint **storageAccountType**, **tagName** et **tagValue** en transmettant chacun au modèle. Pour que les paramètres de blueprint soient accessibles au modèle, définissez **properties.parameters**. Dans le modèle JSON, cette paire clé/valeur est utilisée pour injecter la valeur. Vous pouvez avoir des noms de paramètres de blueprint et de modèle identiques, mais ils sont nommés différemment pour illustrer la manière dont chacun est transmis du blueprint à l’artefact de modèle.

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/templateStorage?api-version=2018-11-01-preview
     ```

   - Corps de la requête

     ```json
     {
         "kind": "template",
         "properties": {
             "template": {
                 "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                 "contentVersion": "1.0.0.0",
                 "parameters": {
                     "storageAccountTypeFromBP": {
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
                     "location": "[resourceGroups('storageRG').location]",
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
             },
             "resourceGroup": "storageRG",
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
     }
     ```

1. Ajoutez une attribution de rôle sous le groupe de ressources. Comme pour l’entrée d’attribution de rôle précédente, l’exemple ci-dessous utilise l’identificateur de définition pour le rôle **Propriétaire** et lui fournit un paramètre différent à partir du blueprint. Cet exemple utilise le rôle intégré _Propriétaire_ avec le GUID `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/artifacts/roleOwner?api-version=2018-11-01-preview
     ```

   - Corps de la requête

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

## <a name="publish-a-blueprint"></a>Publier un blueprint

Maintenant que les artefacts ont été ajoutés au blueprint, il est temps de le publier. Une fois un blueprint publié, il peut être affecté à un abonnement.

- URI de l’API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint/versions/{BlueprintVersion}?api-version=2018-11-01-preview
  ```

La valeur de `{BlueprintVersion}` est une chaîne de lettres, de chiffres et de traits d’union qui ne doit pas dépasser 20 caractères et qui ne doit pas contenir d’espaces ou d’autres caractères spéciaux. Utilisez un nom unique et informatif tel que **v20180622-135541**.

## <a name="assign-a-blueprint"></a>Affecter un blueprint

Une fois qu’un blueprint a été publié à l’aide de l’API REST, il peut être affecté à un abonnement. Affectez le blueprint que vous avez créé à l’un des abonnements sous votre hiérarchie de groupes d’administration. Si le blueprint est enregistré dans un abonnement, il ne peut être attribué qu’à cet abonnement. Le **corps de la requête** spécifie le blueprint à affecter, fournit le nom et l’emplacement des groupes de ressources dans la définition de blueprint, et fournit tous les paramètres définis sur le blueprint et utilisés par un ou plusieurs artefacts attachés.

Dans chaque URI d’API REST, vous devez remplacer les variables utilisées par vos propres valeurs :

- Remplacer `{tenantId}` par l’ID de votre locataire
- Remplacer `{YourMG}` par l’ID de votre groupe d’administration
- Remplacer `{subscriptionId}` par votre ID d’abonnement

1. Octroyez au principal de service Azure Blueprint le rôle **Propriétaire** sur l’abonnement cible. L’ID d’application est statique (`f71766dc-90d9-4b7d-bd9d-4499c4331c3f`), mais l’ID de principal de service varie en fonction du locataire. Vous pouvez demander les détails de votre locataire à l’aide de l’API REST suivante. Elle utilise [l’API Graph Azure Active Directory](../../active-directory/develop/active-directory-graph-api.md), qui a une autorisation différente.

   - URI de l’API REST

     ```http
     GET https://graph.windows.net/{tenantId}/servicePrincipals?api-version=1.6&$filter=appId eq 'f71766dc-90d9-4b7d-bd9d-4499c4331c3f'
     ```

1. Exécutez le déploiement du blueprint en l’affectant à un abonnement. Du fait que les paramètres **contributors** et **owners** nécessitent l’octroi de l’attribution de rôle à un tableau d’ID d’objet des principaux, utilisez [l’API Graph Azure Active Directory](../../active-directory/develop/active-directory-graph-api.md) pour collecter les ID d’objet à utiliser dans le **corps de la demande** pour vos propres utilisateurs, groupes ou principaux de service.

   - URI de l’API REST

     ```http
     PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2018-11-01-preview
     ```

   - Corps de la requête

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

   - Identité managée affectée par l’utilisateur

     Une affectation de blueprint peut également utiliser une [identité managée affectée par l’utilisateur](../../active-directory/managed-identities-azure-resources/overview.md).
     Dans ce cas, la partie **identité** du corps de la requête change de la façon suivante. Remplacez `{yourRG}` et `{userIdentity}` par le nom de votre groupe de ressources et le nom de votre identité managée affectée par l’utilisateur, respectivement.

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

Vous pouvez supprimer un blueprint d’un abonnement. La suppression est souvent effectuée lorsque les ressources d’artefact ne sont plus nécessaires. Quand un blueprint est supprimé, les artefacts affectés dans le cadre de ce blueprint sont abandonnés. Pour supprimer une affectation de blueprint, utilisez l’opération d’API REST suivante :

- URI de l’API REST

  ```http
  DELETE https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Blueprint/blueprintAssignments/assignMyBlueprint?api-version=2018-11-01-preview
  ```

### <a name="delete-a-blueprint"></a>Supprimer un blueprint

Pour supprimer le blueprint, utilisez l’opération d’API REST suivante :

- URI de l’API REST

  ```http
  DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{YourMG}/providers/Microsoft.Blueprint/blueprints/MyBlueprint?api-version=2018-11-01-preview
  ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé, affecté et supprimé un blueprint à l’aide de l’API REST. Pour plus d’informations sur Azure Blueprints, consultez l’article concernant le cycle de vie des blueprints.

> [!div class="nextstepaction"]
> [En savoir plus sur le cycle de vie des blueprints](./concepts/lifecycle.md)