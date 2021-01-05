---
title: Automatiser l’ajout d’un utilisateur de labo dans Azure DevTest Labs | Microsoft Docs
description: Cet article explique comment automatiser l’ajout d’un utilisateur à un laboratoire dans Azure DevTest Labs à l’aide de modèles Azure Resource Manager, de PowerShell et de CLI.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 61853efacc5974b81d46b2b8cca0f2796672d72d
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92327958"
---
# <a name="automate-adding-a-lab-user-to-a-lab-in-azure-devtest-labs"></a>Automatiser l’ajout d’un utilisateur de labo à un labo dans Azure DevTest Labs
Azure DevTest Labs vous permet de créer rapidement des environnements de développement/test en libre-service à l’aide du Portail Azure. Toutefois, si vous disposez de plusieurs équipes et de plusieurs instances DevTest Labs, l’automatisation du processus de création peut vous faire gagner du temps. Les [modèles Azure Resource Manager](https://github.com/Azure/azure-devtestlab/tree/master/Environments) vous permettent de créer des labos, des machines virtuelles de labo, des images personnalisées et des formules, et d’ajouter des utilisateurs de manière automatisée. Cet article concerne spécifiquement l’ajout d’utilisateurs à une instance DevTest Labs.

Pour ajouter un utilisateur à un labo, vous ajoutez cet utilisateur au rôle **Utilisateur de DevTest Labs** pour le labo. Cet article vous indique comment automatiser l’ajout d’un utilisateur à un labo à l’aide de l’une des méthodes suivantes :

- Modèles Microsoft Azure Resource Manager
- Applets de commande Azure PowerShell 
- Azure CLI.

## <a name="use-azure-resource-manager-templates"></a>Utiliser les modèles Azure Resource Manager
L’exemple de modèle Resource Manager ci-après spécifie l’ajout d’un utilisateur au rôle **Utilisateur de DevTest Labs** d’un labo. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "type": "string",
      "metadata": {
        "description": "The objectId of the user, group, or service principal for the role."
      }
    },
    "labName": {
      "type": "string",
      "metadata": {
        "description": "The name of the lab instance to be created."
      }
    },
    "roleAssignmentGuid": {
      "type": "string",
      "metadata": {
        "description": "Guid to use as the name for the role assignment."
      }
    }
  },
  "variables": {
    "devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
    "fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]",
    "roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
  },
  "resources": [
    {
      "apiVersion": "2016-05-15",
      "type": "Microsoft.DevTestLab/labs",
      "name": "[parameters('labName')]",
      "location": "[resourceGroup().location]"
    },
    {
      "apiVersion": "2016-07-01",
      "type": "Microsoft.DevTestLab/labs/providers/roleAssignments",
      "name": "[variables('fullDevTestLabUserRoleName')]",
      "properties": {
        "roleDefinitionId": "[variables('devTestLabUserRoleId')]",
        "principalId": "[parameters('principalId')]",
        "scope": "[variables('roleScope')]"
      },
      "dependsOn": [
        "[resourceId('Microsoft.DevTestLab/labs', parameters('labName'))]"
      ]
    }
  ]
}

```

Si vous attribuez le rôle dans le même modèle que celui qui crée le labo, pensez à ajouter une dépendance entre la ressource d’attribution de rôle et le labo. Pour plus d’informations, consultez l’article [Définition de dépendances dans les modèles Azure Resource Manager](../azure-resource-manager/templates/define-resource-dependency.md).

### <a name="role-assignment-resource-information"></a>Informations de ressource d’attribution de rôle
La ressource d’attribution de rôle doit spécifier le type et le nom.

En premier lieu, notez que la ressource ne présente pas le type `Microsoft.Authorization/roleAssignments`, comme elle le ferait pour un groupe de ressources.  À la place, le type de ressource suit le modèle `{provider-namespace}/{resource-type}/providers/roleAssignments`. Dans ce cas précis, le type de ressource est `Microsoft.DevTestLab/labs/providers/roleAssignments`.

Le nom d’attribution de rôle proprement dit doit être globalement unique.  Le nom de l’attribution utilise le modèle `{labName}/Microsoft.Authorization/{newGuid}`. L’élément `newGuid` est une valeur de paramètre pour le modèle. Il garantit l’unicité du nom d’attribution de rôle. Étant donné qu’il n’existe aucune fonction de modèle pour la création des GUID, vous devez générer un GUID vous-même à l’aide de n’importe quel outil de génération de GUID.  

Dans le modèle, le nom de l’attribution de rôle est défini par la variable `fullDevTestLabUserRoleName`. La ligne exacte figurant dans le modèle est la suivante :

```json
"fullDevTestLabUserRoleName": "[concat(parameters('labName'), '/Microsoft.Authorization/', parameters('roleAssignmentGuid'))]"
```


### <a name="role-assignment-resource-properties"></a>Propriétés de ressource d’attribution de rôle
Une attribution de rôle définit trois propriétés : `roleDefinitionId`, `principalId` et `scope`.

### <a name="role-definition"></a>Définition de rôle
L’ID de définition de rôle est l’identificateur de chaîne de la définition de rôle existante. L’ID de rôle présente le format `/subscriptions/{subscription-id}/providers/Microsoft.Authorization/roleDefinitions/{role-definition-id}`. 

L’ID d’abonnement est obtenu à l’aide de la fonction de modèle `subscription().subscriptionId`.  

Vous devez obtenir la définition de rôle du rôle intégré `DevTest Labs User`. Pour obtenir le GUID du rôle [Utilisateur de DevTest Labs](../role-based-access-control/built-in-roles.md#devtest-labs-user), vous pouvez utiliser [l’API REST Attributions de rôles](/rest/api/authorization/roleassignments) ou la cmdlet [Get-AzRoleDefinition](/powershell/module/az.resources/get-azroledefinition?view=azps-1.8.0).

```powershell
$dtlUserRoleDefId = (Get-AzRoleDefinition -Name "DevTest Labs User").Id
```

L’ID de rôle est défini dans la section des variables et porte le nom `devTestLabUserRoleId`. Dans le modèle, l’ID de rôle est défini sur : 111111111-0000-0000-11111111111111111. 

```json
"devTestLabUserRoleId": "[concat('/subscriptions/', subscription().subscriptionId, '/providers/Microsoft.Authorization/roleDefinitions/111111111-0000-0000-11111111111111111')]",
```

### <a name="principal-id"></a>ID du principal
L’ID du principal est l’ID d’objet de l’utilisateur, du groupe ou du principal de service Active Directory que vous souhaitez ajouter en tant qu’utilisateur de labo au labo. Le modèle utilise l’élément `ObjectId` en tant que paramètre.

Vous pouvez obtenir l’élément ObjectId à l’aide des cmdlets PowerShell [Get-AzureRMADUser](/powershell/module/azurerm.resources/get-azurermaduser?view=azurermps-6.13.0), [Get-AzureRMADGroup ou [Get-AzureRMADServicePrincipal](/powershell/module/azurerm.resources/get-azurermadserviceprincipal?view=azurermps-6.13.0). Ces cmdlets renvoient un seul objet ou une liste d’objets Active Directory dotés d’une propriété d’ID, qui constitue l’ID d’objet dont vous avez besoin. L’exemple ci-après vous indique comment obtenir l’ID d’objet d’un utilisateur unique dans une entreprise.

```powershell
$userObjectId = (Get-AzureRmADUser -UserPrincipalName ‘email@company.com').Id
```

Vous pouvez également utiliser les cmdlets PowerShell Azure Active Directory qui comprennent [Get-MsolUser](/powershell/module/msonline/get-msoluser?view=azureadps-1.0), [Get-MsolGroup](/powershell/module/msonline/get-msolgroup?view=azureadps-1.0) et [Get-MsolServicePrincipal](/powershell/module/msonline/get-msolserviceprincipal?view=azureadps-1.0).

### <a name="scope"></a>Étendue
L’étendue spécifie la ressource ou le groupe de ressources auxquels l’attribution de rôle doit s’appliquer. Pour les ressources, l’étendue présente le format suivant : `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/{provider-namespace}/{resource-type}/{resource-name}`. Le modèle utilise la fonction `subscription().subscriptionId` pour renseigner la partie `subscription-id`, et la fonction de modèle `resourceGroup().name` pour renseigner la partie `resource-group-name`. L’utilisation de ces fonctions signifie que le labo auquel vous attribuez un rôle doit exister dans l’abonnement actuel, ainsi que dans le groupe de ressources sur lequel vous déployez le modèle. La dernière partie, `resource-name`, indique le nom du labo. Dans cet exemple, cette valeur est obtenue par le biais du paramètre de modèle. 

Voici l’étendue du rôle dans le modèle : 

```json
"roleScope": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.DevTestLab/labs/', parameters('labName'))]"
```

### <a name="deploying-the-template"></a>Déploiement du modèle
Commencez par créer un fichier de paramètres (par exemple, azuredeploy.parameters.json) chargé de transmettre les valeurs des paramètres au modèle Resource Manager. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "principalId": {
      "value": "11111111-1111-1111-1111-111111111111"
    },
    "labName": {
      "value": "MyLab"
    },
    "roleAssignmentGuid": {
      "value": "22222222-2222-2222-2222-222222222222"
    }
  }
}
```

Ensuite, déployez le modèle Resource Manager à l’aide de la cmdlet PowerShell [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment?view=azurermps-6.13.0). L’exemple de commande ci-après attribue une personne, un groupe ou un principal de service au rôle Utilisateur de DevTest Labs pour un labo.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateParameterFile .\azuredeploy.parameters.json -TemplateFile .\azuredeploy.json
```

Il est important de noter que le nom du déploiement de groupe et le GUID d’attribution de rôle doivent être uniques. Si vous essayez de déployer une attribution de ressources en spécifiant un GUID non unique, vous obtiendrez une erreur `RoleAssignmentUpdateNotPermitted`.

Si vous prévoyez d’utiliser le modèle à plusieurs reprises pour ajouter plusieurs objets Active Directory au rôle Utilisateur de DevTest Labs pour votre labo, envisagez d’utiliser des objets dynamiques dans votre commande PowerShell. L’exemple ci-après utilise la cmdlet [New-Guid](/powershell/module/Microsoft.PowerShell.Utility/New-Guid?view=powershell-5.0) pour spécifier de manière dynamique le nom du déploiement de groupe de ressources et le GUID d’attribution de rôle.

```powershell
New-AzureRmResourceGroupDeployment -Name "MyLabResourceGroup-$(New-Guid)" -ResourceGroupName 'MyLabResourceGroup' -TemplateFile .\azuredeploy.json -roleAssignmentGuid "$(New-Guid)" -labName "MyLab" -principalId "11111111-1111-1111-1111-111111111111"
```

## <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell
Comme indiqué dans l’introduction, vous allez créer une attribution de rôle Azure afin d’ajouter un utilisateur au rôle **Utilisateur de DevTest Labs** pour le labo. Dans PowerShell, vous effectuez cette opération à l’aide de la cmdlet [New-AzureRMRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment?view=azurermps-6.13.0). Cette cmdlet dispose de nombreux paramètres facultatifs pour plus de flexibilité. Vous pouvez spécifier l’élément `ObjectId`, `SigninName` ou `ServicePrincipalName` en tant qu’objet auquel accorder des autorisations.  

Voici un exemple de commande Azure PowerShell qui ajoute un utilisateur au rôle Utilisateur de DevTest Labs dans le labo spécifié.

```powershell
New-AzureRmRoleAssignment -UserPrincipalName <email@company.com> -RoleDefinitionName 'DevTest Labs User' -ResourceName '<Lab Name>' -ResourceGroupName '<Resource Group Name>' -ResourceType 'Microsoft.DevTestLab/labs'
```

Pour spécifier la ressource à laquelle accorder les autorisations, combinez les éléments `ResourceName`, `ResourceType` et `ResourceGroup` ou utilisez le paramètre `scope`. Quelle que soit la combinaison de paramètres utilisée, fournissez suffisamment d’informations à la cmdlet pour identifier de manière unique l’objet Active Directory (utilisateur, groupe ou principal de service), l’étendue (groupe de ressources ou ressource) et la définition de rôle.

## <a name="use-azure-command-line-interface-cli"></a>Utiliser l’interface de ligne de commande Azure (CLI)
Dans Azure CLI, l’ajout d’un utilisateur de labo à un labo s’effectue à l’aide de la commande `az role assignment create`. Pour plus d’informations sur les applets de commande Azure CLI, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide de Azure CLI](../role-based-access-control/role-assignments-cli.md).

L’objet auquel accorder l’accès peut être spécifié par les paramètres `objectId`, `signInName` et `spn`. Le labo auquel l’objet est autorisé à accéder est identifiable par l’URL `scope` ou par une combinaison des paramètres `resource-name`, `resource-type` et `resource-group`.

L’exemple Azure CLI ci-après vous indique comment ajouter une personne au rôle Utilisateur de DevTest Labs pour le labo spécifié.  

```azurecli
az role assignment create --roleName "DevTest Labs User" --signInName <email@company.com> -–resource-name "<Lab Name>" --resource-type “Microsoft.DevTestLab/labs" --resource-group "<Resource Group Name>"
```

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- [Créer et gérer des machines virtuelles avec DevTest Labs à l’aide de l’interface de ligne de commande Azure](devtest-lab-vmcli.md)
- [Créer une machine virtuelle avec DevTest Labs en utilisant Azure PowerShell](devtest-lab-vm-powershell.md)
- [Utiliser des outils de ligne de commande pour démarrer et arrêter des machines virtuelles Azure DevTest Labs](use-command-line-start-stop-virtual-machines.md)

