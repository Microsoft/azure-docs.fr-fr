---
title: Remédier aux ressources non conformes
description: Ce guide explique comment corriger les ressources qui ne sont pas conformes aux stratégies dans Azure Policy.
ms.date: 10/05/2020
ms.topic: how-to
ms.openlocfilehash: 76d2e57c1b5df965c81c88506ff2c2f70b2cb1f8
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91876326"
---
# <a name="remediate-non-compliant-resources-with-azure-policy"></a>Corriger les ressources non conformes avec Azure Policy

Les ressources qui ne sont pas conformes à une stratégie **deployIfNotExists** ou **modify** peuvent être placées dans un état conforme par le biais d’une **correction**. Pour effectuer cette correction, vous devez demander à Azure Policy d’exécuter l’effet **deployIfNotExists** ou des **opérations de modification** de la stratégie attribuée à vos ressources existantes, que cette attribution s’adresse à un groupe d’administration, à un abonnement, à un groupe de ressources ou à une ressource individuelle. Cet article explique les étapes nécessaires pour comprendre et exécuter des corrections avec Azure Policy.

## <a name="how-remediation-security-works"></a>Fonctionnement de la sécurité de la correction

Lorsque le logiciel Azure Policy exécute le modèle dans la définition de stratégie **deployIfNotExists**, il utilise une [identité managée](../../../active-directory/managed-identities-azure-resources/overview.md).
Azure Policy crée automatiquement une identité managée pour chaque affectation, mais doit obtenir des informations sur les rôles à accorder à l’identité managée. S’il manque des rôles à l’identité managée, une erreur est affichée durant l’affectation de la stratégie ou d’une initiative. Quand vous utilisez le portail, une fois l’affectation lancée, Azure Policy accorde automatiquement à l’identité managée les rôles listés. Lorsque vous utilisez le kit de développement logiciel (SDK), les rôles doivent être accordés manuellement à l’identité managée. La _localisation_ de l’identité managée n’impacte pas son fonctionnement avec Azure Policy.

:::image type="content" source="../media/remediate-resources/missing-role.png" alt-text="Capture d’écran d’une stratégie deployIfNotExists qui ne dispose pas d’une autorisation définie sur l’identité gérée." border="false":::

> [!IMPORTANT]
> Dans les scénarios suivants, l’autorisation d’accès à l’identité managée de l’affectation doit être [accordée manuellement](#manually-configure-the-managed-identity) sinon le déploiement de la correction échoue :
>
> - Si l’affectation est créée via le kit de développement logiciel (SDK)
> - Si une ressource modifiée par **deployIfNotExists** ou **modify** est en dehors de l’étendue de l’affectation de stratégie
> - Si le modèle accède à des propriétés sur des ressources en dehors de l’étendue de l’affectation de stratégie

## <a name="configure-policy-definition"></a>Configurer une définition de stratégie

La première étape consiste à définir les rôles dont **deployIfNotExists** ou **modify** ont besoin dans la définition de stratégie pour déployer le contenu du modèle inclus. Sous la propriété **details**, ajoutez une propriété **roleDefinitionIds**. Il s’agit d’un tableau de chaînes qui correspondent à des rôles dans votre environnement. Pour obtenir un exemple complet, consultez l’[exemple deployIfNotExists](../concepts/effects.md#deployifnotexists-example) ou les [exemples modify](../concepts/effects.md#modify-examples).

```json
"details": {
    ...
    "roleDefinitionIds": [
        "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleGUID}",
        "/providers/Microsoft.Authorization/roleDefinitions/{builtinroleGUID}"
    ]
}
```

La propriété **roleDefinitionIds** utilise l’identificateur de ressource complet et n’accepte pas le **roleName** court du rôle. Pour obtenir l’ID du rôle « Contributeur » dans votre environnement, utilisez le code suivant :

```azurecli-interactive
az role definition list --name 'Contributor'
```

## <a name="manually-configure-the-managed-identity"></a>Configurer manuellement l’identité managée

Lorsque vous créez une affectation à l’aide du portail, Azure Policy génère l’identité managée et lui accorde les rôles définis dans **roleDefinitionIds**. Dans les conditions suivantes, les étapes permettant de créer l’identité managée et de lui accorder des autorisations doivent être effectuées manuellement :

- Durant l’utilisation du SDK (par exemple, Azure PowerShell)
- Quand une ressource en dehors de l’étendue de l’affectation est modifiée par le modèle
- Quand une ressource en dehors de l’étendue de l’affectation est lue par le modèle

### <a name="create-managed-identity-with-powershell"></a>Créer une identité managée avec PowerShell

Pour créer une identité managée pendant l’affectation de la stratégie, vous devez définir **Location** et utiliser **AssignIdentity**. L’exemple suivant obtient la définition de la stratégie intégrée **Déployer Transparent Data Encryption SQL DB**, définit le groupe de ressources cible, puis crée l’affectation.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Get the built-in "Deploy SQL DB transparent data encryption" policy definition
$policyDef = Get-AzPolicyDefinition -Id '/providers/Microsoft.Authorization/policyDefinitions/86a912f6-9a06-4e26-b447-11b16ba8659f'

# Get the reference to the resource group
$resourceGroup = Get-AzResourceGroup -Name 'MyResourceGroup'

# Create the assignment using the -Location and -AssignIdentity properties
$assignment = New-AzPolicyAssignment -Name 'sqlDbTDE' -DisplayName 'Deploy SQL DB transparent data encryption' -Scope $resourceGroup.ResourceId -PolicyDefinition $policyDef -Location 'westus' -AssignIdentity
```

La `$assignment` variable contient à présent l’ID du principal de l’identité managée, ainsi que les valeurs standard retournées au moment de la création d’une affectation de stratégie. Elle est accessible via `$assignment.Identity.PrincipalId`.

### <a name="grant-defined-roles-with-powershell"></a>Accorder des rôles définis avec PowerShell

La nouvelle identité managée doit effectuer la réplication par le biais d’Azure Active Directory pour pouvoir obtenir les rôles nécessaires. Une fois la réplication terminée, l’exemple suivant parcourt la définition de stratégie dans `$policyDef` à la recherche des ID **roleDefinitionIds** et utilise [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) pour accorder les rôles à la nouvelle identité managée.

```azurepowershell-interactive
# Use the $policyDef to get to the roleDefinitionIds array
$roleDefinitionIds = $policyDef.Properties.policyRule.then.details.roleDefinitionIds

if ($roleDefinitionIds.Count -gt 0)
{
    $roleDefinitionIds | ForEach-Object {
        $roleDefId = $_.Split("/") | Select-Object -Last 1
        New-AzRoleAssignment -Scope $resourceGroup.ResourceId -ObjectId $assignment.Identity.PrincipalId -RoleDefinitionId $roleDefId
    }
}
```

### <a name="grant-defined-roles-through-portal"></a>Accorder des rôles définis par le biais du portail

À l’aide du portail, vous pouvez accorder les rôles définis à l’identité managée d’une affectation de deux façons : en utilisant la **gestion des identités et des accès** ou en modifiant l’affectation de la stratégie ou d’une initiative et en sélectionnant **Enregistrer**.

Pour ajouter un rôle à l’identité managée de l’affectation, effectuez les étapes suivantes :

1. Lancez le service Azure Policy dans le portail Azure en sélectionnant **Tous les services**, puis en recherchant et en cliquant sur **Stratégie**.

1. Sélectionnez **Affectations** du côté gauche de la page Azure Policy.

1. Recherchez l’affectation qui a une identité managée, puis sélectionnez le nom.

1. Rechercher la propriété **ID de l’affectation** dans la page de modification. L’ID d’affectation est semblable à ceci :

   ```output
   /subscriptions/{subscriptionId}/resourceGroups/PolicyTarget/providers/Microsoft.Authorization/policyAssignments/2802056bfc094dfb95d4d7a5
   ```

   Le nom de l’identité managée est la dernière partie de l’ID de ressource d’affectation, c’est-à-dire `2802056bfc094dfb95d4d7a5` dans cet exemple. Copiez cette partie de l’ID de ressource d’affectation.

1. Accédez à la ressource ou au conteneur de ressources parent (groupe de ressources, abonnement, groupe d’administration) auquel la définition de rôle doit être ajoutée manuellement.

1. Sélectionnez le lien **Contrôle d’accès (IAM)** dans la page des ressources, puis sélectionnez **+ Ajouter une attribution de rôle** en haut de la page du contrôle d’accès.

1. Sélectionnez le rôle approprié qui correspond à un **roleDefinitionIds** dans la définition de stratégie.
   Laissez **Attribuer l’accès à** sur la valeur par défaut « Utilisateur, groupe ou application Azure AD ». Dans la zone **Sélectionner**, collez ou tapez la partie de l’ID de ressource d’affectation trouvée plus haut. Une fois la recherche terminée, sélectionnez l’objet portant le même nom pour sélectionner l’ID, puis sélectionnez **Enregistrer**.

## <a name="create-a-remediation-task"></a>Créer une tâche de correction

### <a name="create-a-remediation-task-through-portal"></a>Créer une tâche de correction via le portail

Durant l’évaluation, l’affectation de stratégie avec les effets **deployIfNotExists** ou **modify** détermine s’il existe des ressources non conformes. Quand des ressources non conformes sont trouvées, les détails sont fournis dans la page **Correction**. Outre la liste des stratégies qui ont des ressources non conformes se trouve l’option permettant de déclencher une **tâche de correction**. Cette option crée un déploiement à partir du modèle **deployIfNotExists** ou des opérations **modify**.

Pour créer une **tâche de correction**, effectuez les étapes suivantes :

1. Lancez le service Azure Policy dans le portail Azure en sélectionnant **Tous les services**, puis en recherchant et en cliquant sur **Stratégie**.

   :::image type="content" source="../media/remediate-resources/search-policy.png" alt-text="Capture d’écran d’une stratégie deployIfNotExists qui ne dispose pas d’une autorisation définie sur l’identité gérée." border="false":::

1. Sélectionnez **Correction** sur le côté gauche de la page Azure Policy.

   :::image type="content" source="../media/remediate-resources/select-remediation.png" alt-text="Capture d’écran d’une stratégie deployIfNotExists qui ne dispose pas d’une autorisation définie sur l’identité gérée." border="false":::

1. Toutes les affectations de stratégie **deployIfNotExists** ou **modify** ayant des ressources non conformes sont incluses sous l’onglet **Stratégies à corriger** et dans la table de données. Sélectionnez une stratégie ayant des ressources non conformes. La page **Nouvelle tâche de correction** s’ouvre.

   > [!NOTE]
   > Pour ouvrir la page **Tâche de correction**, vous pouvez également rechercher et sélectionner la stratégie à partir de la page **Conformité**, puis sélectionner le bouton **Créer une tâche de correction**.

1. Dans la page **Nouvelle tâche de correction**, filtrez les ressources à corriger à l’aide des points de suspension de la section **Étendue** pour sélectionner les ressources enfants à partir de l’endroit où la stratégie est affectée (y compris jusqu’aux objets de ressource individuels). En outre, utilisez la liste déroulante **Emplacements** pour filtrer davantage les ressources. Seules les ressources répertoriées dans la table sont corrigées.

   :::image type="content" source="../media/remediate-resources/select-resources.png" alt-text="Capture d’écran d’une stratégie deployIfNotExists qui ne dispose pas d’une autorisation définie sur l’identité gérée." border="false":::

1. Lancez la tâche de correction une fois les ressources filtrées en sélectionnant **Corriger**. La page de conformité à la stratégie s’ouvre sur l’onglet **Tâches de correction**, qui affiche l’état de la progression des tâches. Les déploiements créés par la tâche de correction commencent immédiatement.

   :::image type="content" source="../media/remediate-resources/task-progress.png" alt-text="Capture d’écran d’une stratégie deployIfNotExists qui ne dispose pas d’une autorisation définie sur l’identité gérée." border="false":::

1. Sélectionnez la **tâche de correction** dans la page de conformité à la stratégie pour obtenir plus d’informations sur la progression. Le filtrage utilisé pour la tâche est affiché, ainsi qu’une liste des ressources en cours de correction.

1. Dans la page **Tâche de correction**, cliquez avec le bouton droit sur une ressource pour afficher le déploiement de la tâche de correction ou la ressource. À la fin de la ligne, sélectionnez **Événements associés** pour voir des détails tels qu’un message d’erreur.

   :::image type="content" source="../media/remediate-resources/resource-task-context-menu.png" alt-text="Capture d’écran d’une stratégie deployIfNotExists qui ne dispose pas d’une autorisation définie sur l’identité gérée." border="false":::

Les ressources déployées par le biais d’une **tâche de correction** sont ajoutées à l’onglet **Ressources déployées** sur la page de conformité à la stratégie.

### <a name="create-a-remediation-task-through-azure-cli"></a>Créer une tâche de correction via Azure CLI

Pour créer un **tâche de correction** avec Azure CLI, utilisez les commandes `az policy remediation`. Remplacez `{subscriptionId}` par votre ID d’abonnement et `{myAssignmentId}` par l’ID d’affectation de stratégie **deployIfNotExists** ou **modify**.

```azurecli-interactive
# Login first with az login if not using Cloud Shell

# Create a remediation for a specific assignment
az policy remediation create --name myRemediation --policy-assignment '/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/{myAssignmentId}'
```

Pour accéder à d’autres exemples et commandes de correction, consultez les commandes de [correction de stratégie az](/cli/azure/policy/remediation).

### <a name="create-a-remediation-task-through-azure-powershell"></a>Créer une tâche de correction via Azure PowerShell

Pour créer un **tâche de correction** avec Azure PowerShell, utilisez les commandes `Start-AzPolicyRemediation`. Remplacez `{subscriptionId}` par votre ID d’abonnement et `{myAssignmentId}` par l’ID d’affectation de stratégie **deployIfNotExists** ou **modify**.

```azurepowershell-interactive
# Login first with Connect-AzAccount if not using Cloud Shell

# Create a remediation for a specific assignment
Start-AzPolicyRemediation -Name 'myRemedation' -PolicyAssignmentId '/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyAssignments/{myAssignmentId}'
```

Pour accéder à d’autres cmdlets et exemples de correction, consultez le module [Az.PolicyInsights](/powershell/module/az.policyinsights/#policy_insights).

### <a name="create-a-remediation-task-during-policy-assignment-in-the-azure-portal"></a>Créer une tâche de correction au moment d’affecter une stratégie sur le portail Azure

Une façon simple de créer une tâche de correction est d’utiliser le portail Azure pendant l’affectation d’une stratégie. Si la définition de la stratégie à affecter est un effet **deployIfNotExists** ou **Modify**, l’Assistant situé sous l’onglet **Correction** propose une option _Créer une tâche de correction_. Si cette option est sélectionnée, une tâche de correction est créée en même temps que l’attribution de la stratégie.

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la [Structure de définition Azure Policy](../concepts/definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](../concepts/effects.md).
- Découvrez comment [créer des stratégies par programmation](programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](get-compliance-data.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).
