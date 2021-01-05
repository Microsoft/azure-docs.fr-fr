---
title: Déployer des ressources sur le locataire
description: Décrit comment déployer des ressources au niveau du locataire dans un modèle Azure Resource Manager.
ms.topic: conceptual
ms.date: 11/24/2020
ms.openlocfilehash: 5733c5d6eb6cbd86207589244c22badc17fe7073
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95807634"
---
# <a name="tenant-deployments-with-arm-templates"></a>Déploiements de locataires avec des modèles Resource Manager

À mesure que votre organisation évolue, vous pouvez être amené à définir et attribuer des [stratégies](../../governance/policy/overview.md) ou des [contrôles d’accès basés sur les rôles Azure (Azure RBAC)](../../role-based-access-control/overview.md) dans votre locataire Azure AD. Avec les modèles au niveau du locataire, vous pouvez appliquer de façon déclarative des stratégies et attribuer des rôles à un niveau global.

## <a name="supported-resources"></a>Ressources prises en charge

Tous les types de ressources ne peuvent pas être déployés au niveau du locataire. Cette section répertorie les types de ressources pris en charge.

Pour les stratégies Azure, utilisez :

* [policyAssignments](/azure/templates/microsoft.authorization/policyassignments)
* [policyDefinitions](/azure/templates/microsoft.authorization/policydefinitions)
* [policySetDefinitions](/azure/templates/microsoft.authorization/policysetdefinitions)

Pour le contrôle d’accès en fonction du rôle Azure (RBAC Azure), utilisez :

* [roleAssignments](/azure/templates/microsoft.authorization/roleassignments)

Pour les modèles imbriqués qui sont déployés sur des groupes d’administration, des abonnements ou des groupes de ressources, utilisez :

* [deployments](/azure/templates/microsoft.resources/deployments)

Pour créer des groupes d’administration, utilisez :

* [managementGroups](/azure/templates/microsoft.management/managementgroups)

Pour créer des abonnements, utilisez :

* [aliases](/azure/templates/microsoft.subscription/aliases)

Pour la gestion des coûts, utilisez :

* [billingProfiles](/azure/templates/microsoft.billing/billingaccounts/billingprofiles)
* [instructions](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/instructions)
* [invoiceSections](/azure/templates/microsoft.billing/billingaccounts/billingprofiles/invoicesections)

Pour la configuration du portail, utilisez :

* [tenantConfigurations](/azure/templates/microsoft.portal/tenantconfigurations)

## <a name="schema"></a>schéma

Le schéma que vous utilisez pour les déploiements au niveau du locataire est différent de celui utilisé pour les déploiements de groupes de ressources.

Pour les modèles, utilisez :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-08-01/tenantDeploymentTemplate.json#",
    ...
}
```

Le schéma d’un fichier de paramètres est le même pour toutes les étendues de déploiement. Fichiers de fichiers de paramètres, utilisez :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    ...
}
```

## <a name="required-access"></a>Accès requis

Le principal déployant le modèle doit être autorisé à créer des ressources au niveau du locataire. Le principal doit être autorisé à exécuter les actions de déploiement (`Microsoft.Resources/deployments/*`) et à créer les ressources définies dans le modèle. Par exemple, pour créer un groupe d’administration, le principal doit disposer de l’autorisation Contributeur au niveau du locataire. Pour créer des attributions de rôles, le principal doit disposer de l’autorisation Propriétaire.

L’administrateur général d'Azure Active Directory n'est pas automatiquement autorisé à attribuer des rôles. Pour activer les déploiements de modèles au niveau du locataire, l’Administrateur général doit procéder comme suit :

1. Élever l’accès au compte de manière à permettre à l'Administrateur général d'attribuer des rôles. Pour plus d’informations, consultez [Élever l’accès pour gérer tous les abonnements et groupes d’administration Azure](../../role-based-access-control/elevate-access-global-admin.md).

1. Attribuer le rôle Propriétaire ou Contributeur au principal devant déployer les modèles.

   ```azurepowershell-interactive
   New-AzRoleAssignment -SignInName "[userId]" -Scope "/" -RoleDefinitionName "Owner"
   ```

   ```azurecli-interactive
   az role assignment create --assignee "[userId]" --scope "/" --role "Owner"
   ```

Le principal dispose désormais des autorisations requises pour déployer le modèle.

## <a name="deployment-commands"></a>Commandes de déploiement

Les commandes utilisées pour les déploiements de locataires sont différentes de celles utilisées pour les déploiements de groupes de ressources.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour Azure CLI, utilisez [az deployment tenant create](/cli/azure/deployment/tenant#az-deployment-tenant-create) :

```azurecli-interactive
az deployment tenant create \
  --name demoTenantDeployment \
  --location WestUS \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour Azure PowerShell, utilisez [New-AzTenantDeployment](/powershell/module/az.resources/new-aztenantdeployment).

```azurepowershell-interactive
New-AzTenantDeployment `
  -Name demoTenantDeployment `
  -Location "West US" `
  -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/tenant-deployments/new-mg/azuredeploy.json"
```

---

Pour plus d’informations sur les commandes et options de déploiement de modèles Resource Manager, consultez :

* [Déployer des ressources avec des modèles ARM et le Portail Azure](deploy-portal.md)
* [Déployer des ressources à l’aide de modèles ARM et l’interface CLI Azure](deploy-cli.md)
* [Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell](deploy-powershell.md)
* [Déployer des ressources avec des modèles ARM et l’API REST Azure Resource Manager](deploy-rest.md)
* [Utiliser un bouton de déploiement pour déployer des modèles à partir du référentiel GitHub](deploy-to-azure-button.md)
* [Déployer des modèles ARM à partir de Cloud Shell](deploy-cloud-shell.md)

## <a name="deployment-location-and-name"></a>Emplacement et nom du déploiement

Pour les déploiements au niveau du locataire, vous devez fournir un emplacement de déploiement. L’emplacement du déploiement est distinct de l’emplacement des ressources que vous déployez. L’emplacement de déploiement indique où stocker les données de déploiement. Les déploiements d’[abonnement](deploy-to-subscription.md) et de [groupe d’administration](deploy-to-management-group.md) nécessitent également un emplacement. Pour les déploiements d’un [groupe de ressources](deploy-to-resource-group.md), l’emplacement du groupe de ressources est utilisé pour stocker les données de déploiement.

Vous pouvez fournir un nom de déploiement ou utiliser le nom de déploiement par défaut. Le nom par défaut est le nom du fichier de modèle. Par exemple, le déploiement d’un modèle nommé **azuredeploy.json** crée le nom de déploiement par défaut **azuredeploy**.

Pour chaque nom de déploiement, l’emplacement est immuable. Il n’est pas possible de créer un déploiement dans un emplacement s’il existe un déploiement du même nom dans un autre emplacement. Par exemple, si vous créez un déploiement de locataire avec le nom **deployment1** dans **centralus**, vous ne pouvez pas créer par la suite un autre déploiement avec le nom **deployment1** mais un emplacement **westus**. Si vous obtenez le code d’erreur `InvalidDeploymentLocation`, utilisez un autre nom ou le même emplacement que le déploiement précédent pour ce nom.

## <a name="deployment-scopes"></a>Étendues de déploiement

Lors du déploiement dans un locataire, vous pouvez déployer des ressources vers :

* le locataire
* des groupes d’administration dans le locataire
* subscriptions
* groupes de ressources
* les [ressources d’extension](scope-extension-resources.md) peuvent être appliquées aux ressources

L’utilisateur qui déploie le modèle doit avoir accès à l’étendue spécifiée.

Cette section montre comment spécifier des étendues différentes. Vous pouvez combiner ces différentes étendues dans un seul modèle.

### <a name="scope-to-tenant"></a>Étendue au locataire

Les ressources définies dans la section Ressources du modèle sont appliquées au locataire.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/default-tenant.json" highlight="5":::

### <a name="scope-to-management-group"></a>Étendue au groupe d’administration

Pour cibler un groupe d’administration au sein du locataire, ajoutez un déploiement imbriqué et spécifiez la propriété `scope`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-mg.json" highlight="10,17,18,22":::

### <a name="scope-to-subscription"></a>Étendue à l’abonnement

Vous pouvez également cibler des abonnements dans le locataire. L’utilisateur qui déploie le modèle doit avoir accès à l’étendue spécifiée.

Pour cibler un abonnement au sein du locataire, utilisez un déploiement imbriqué et la propriété `subscriptionId`.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-subscription.json" highlight="9,10,18":::

### <a name="scope-to-resource-group"></a>Étendue au groupe de ressources

Vous pouvez également cibler des groupes de ressources au sein du locataire. L’utilisateur qui déploie le modèle doit avoir accès à l’étendue spécifiée.

Pour cibler un groupe de ressources au sein du locataire, utilisez un déploiement imbriqué. Définissez les propriétés `subscriptionId` et `resourceGroup`. Ne définissez pas un emplacement pour le déploiement imbriqué, car il est déployé à l’emplacement du groupe de ressources.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/scope/tenant-to-rg.json" highlight="9,10,18":::

## <a name="create-management-group"></a>Créer un groupe d’administration

Le modèle suivant crée un groupe d'administration.

:::code language="json" source="~/quickstart-templates/tenant-deployments/new-mg/azuredeploy.json":::

## <a name="assign-role"></a>Affecter le rôle

Le modèle suivant attribue un rôle au niveau du locataire.

:::code language="json" source="~/quickstart-templates/tenant-deployments/tenant-role-assignment/azuredeploy.json":::

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur l’attribution de rôles, consultez [Ajouter des attributions de rôle Azure à l’aide de modèles Resource Manager](../../role-based-access-control/role-assignments-template.md).
* Vous pouvez également déployer des modèles au [niveau de l’abonnement](deploy-to-subscription.md) ou au [niveau du groupe d'administration](deploy-to-management-group.md).
