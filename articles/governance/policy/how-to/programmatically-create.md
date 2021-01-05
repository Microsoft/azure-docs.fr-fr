---
title: Créer des stratégies par programmation
description: Cet article vous explique comment créer et gérer des stratégies par programmation pour Azure Policy avec Azure CLI, Azure PowerShell et l’API REST.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 86c1b60608780cba4d8ae27e3c82ace458f09a26
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94920013"
---
# <a name="programmatically-create-policies"></a>Créer des stratégies par programmation

Cet article vous explique comment créer et gérer des stratégies par programmation. Les définitions de stratégie Azure appliquent différentes règles et des différents effets sur vos ressources. Cette mise en conformité permet de garantir que les ressources restent conformes à vos normes d’entreprise et contrats de niveau de service.

Pour plus d’informations sur la conformité, consultez [Obtention de données de conformité](get-compliance-data.md).

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que les conditions préalables suivantes sont remplies :

1. Si ce n’est pas déjà fait, installez [ARMClient](https://github.com/projectkudu/ARMClient). C’est un outil qui envoie des requêtes HTTP aux API Azure Resource Manager.

1. Procédez à la mise à jour de votre module Azure PowerShell vers la dernière version. Pour plus d'informations, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Pour plus d’informations sur la version la plus récente, voir [Azure PowerShell](https://github.com/Azure/azure-powershell/releases).

1. Inscrivez le fournisseur de ressources Azure Policy Insights à l’aide d’Azure PowerShell pour vérifier que votre abonnement fonctionne avec ce fournisseur de ressources. Pour inscrire un fournisseur de ressources, vous devez être autorisé à exécuter l’opération d’inscription pour le fournisseur de ressources. Cette opération est incluse dans les rôles de contributeur et de propriétaire. Exécutez la commande suivante pour enregistrer le fournisseur de ressources :

   ```azurepowershell-interactive
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'
   ```

   Pour plus d’informations sur l’inscription et l’affichage des fournisseurs de ressources, consultez [Fournisseurs et types de ressources](../../../azure-resource-manager/management/resource-providers-and-types.md).

1. Si ce n’est déjà fait, installez Azure CLI. Vous pouvez obtenir la dernière version sur la page [Installer Azure CLI sur Windows](/cli/azure/install-azure-cli-windows).

## <a name="create-and-assign-a-policy-definition"></a>Créer et attribuer une définition de stratégie

Pour une meilleure visibilité de vos ressources, la première chose à faire est de créer et d’attribuer des stratégies à vos ressources. L’étape suivante consiste à apprendre à créer et assigner une stratégie par programmation. L’exemple de stratégie proposé réalise l’audit des comptes de stockage ouverts à tous les réseaux publics à l’aide de PowerShell, d’Azure CLI et de requêtes HTTP.

### <a name="create-and-assign-a-policy-definition-with-powershell"></a>Créer et assigner une définition de stratégie avec PowerShell

1. L’extrait de code JSON suivant permet de créer un fichier JSON nommé AuditStorageAccounts.json.

   ```json
   {
       "if": {
           "allOf": [{
                   "field": "type",
                   "equals": "Microsoft.Storage/storageAccounts"
               },
               {
                   "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                   "equals": "Allow"
               }
           ]
       },
       "then": {
           "effect": "audit"
       }
   }
   ```

   Pour plus d’informations sur la création d’une définition de stratégie, consultez la page [Structure de définition Azure Policy](../concepts/definition-structure.md).

1. Exécutez la commande suivante pour créer une définition de stratégie en utilisant le fichier AuditStorageAccounts.json.

   ```azurepowershell-interactive
   New-AzPolicyDefinition -Name 'AuditStorageAccounts' -DisplayName 'Audit Storage Accounts Open to Public Networks' -Policy 'AuditStorageAccounts.json'
   ```

   La commande crée une définition de stratégie nommée _Audit Storage Accounts Open to Public Networks_.
   Pour plus d'informations sur les autres paramètres que vous pouvez utiliser, consultez [New-AzPolicyDefinition](/powershell/module/az.resources/new-azpolicydefinition).

   Lorsqu’il est appelé sans paramètre d’emplacement, par défaut, `New-AzPolicyDefinition` enregistre la définition de stratégie dans l’abonnement sélectionné du contexte de sessions. Pour enregistrer la définition dans un autre emplacement, utilisez les paramètres suivants :

   - **SubscriptionId** : enregistrer dans un autre abonnement. Une valeur _GUID_ est nécessaire.
   - **ManagementGroupName** : enregistrer dans un groupe d’administration. Une valeur de _chaîne_ est nécessaire.

1. Après avoir créé votre définition de stratégie, vous pouvez créer une attribution de stratégie en exécutant les commandes suivantes :

   ```azurepowershell-interactive
   $rg = Get-AzResourceGroup -Name 'ContosoRG'
   $Policy = Get-AzPolicyDefinition -Name 'AuditStorageAccounts'
   New-AzPolicyAssignment -Name 'AuditStorageAccounts' -PolicyDefinition $Policy -Scope $rg.ResourceId
   ```

   Remplacez _ContosoRG_ par le nom de votre groupe de ressources prévu.

   Le paramètre **Scope** sur `New-AzPolicyAssignment` peut être défini pour un groupe d’administration, un abonnement, un groupe de ressources ou une seule ressource. Le paramètre utilise un chemin d’accès de ressource complet, que la propriété **ResourceId** renvoie sur `Get-AzResourceGroup`. Pour chaque conteneur, le modèle **Étendue** est le suivant. Remplacez `{rName}`, `{rgName}`, `{subId}` et `{mgName}` par le nom de la ressource, le nom du groupe de ressources, l’ID de l’abonnement et le nom du groupe d’administration, respectivement.
   `{rType}` est remplacé par le **type de la ressource**, comme `Microsoft.Compute/virtualMachines` pour une machine virtuelle.

   - Ressource : `/subscriptions/{subID}/resourceGroups/{rgName}/providers/{rType}/{rName}`
   - Groupe de ressources : `/subscriptions/{subId}/resourceGroups/{rgName}`
   - Abonnement : `/subscriptions/{subId}`
   - Groupe d'administration : `/providers/Microsoft.Management/managementGroups/{mgName}`

Pour plus d’informations sur la gestion des stratégies de ressources à l’aide du module Resource Manager PowerShell, consultez [Az.Resources](/powershell/module/az.resources/#policies).

### <a name="create-and-assign-a-policy-definition-using-armclient"></a>Créer et assigner une définition de stratégie avec ARMClient

Pour créer une définition de stratégie, procédez comme suit.

1. Copiez l’extrait de code JSON suivant pour créer un fichier JSON. Vous appellerez le fichier à l’étape suivante.

   ```json
   "properties": {
       "displayName": "Audit Storage Accounts Open to Public Networks",
       "policyType": "Custom",
       "mode": "Indexed",
       "description": "This policy ensures that storage accounts with exposure to Public Networks are audited.",
       "parameters": {},
       "policyRule": {
           "if": {
               "allOf": [{
                       "field": "type",
                       "equals": "Microsoft.Storage/storageAccounts"
                   },
                   {
                       "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                       "equals": "Allow"
                   }
               ]
           },
           "then": {
               "effect": "audit"
           }
       }
   }
   ```

1. Créez la définition de stratégie à l’aide de l’un des appels suivants :

   ```console
   # For defining a policy in a subscription
   armclient PUT "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/policyDefinitions/AuditStorageAccounts?api-version=2019-09-01" @<path to policy definition JSON file>

   # For defining a policy in a management group
   armclient PUT "/providers/Microsoft.Management/managementgroups/{managementGroupId}/providers/Microsoft.Authorization/policyDefinitions/AuditStorageAccounts?api-version=2019-09-01" @<path to policy definition JSON file>
   ```

   Remplacez {subscriptionId} précédent par l’ID de votre abonnement ou {managementGroupId} par l’ID de votre [groupe d’administration](../../management-groups/overview.md).

   Pour plus d’informations sur la structure de la requête, consultez [Définitions de stratégie Azure – Créer ou mettre à jour](/rest/api/resources/policydefinitions/createorupdate) et [Définitions de stratégie – Créer ou mettre à jour dans le groupe d’administration](/rest/api/resources/policydefinitions/createorupdateatmanagementgroup)

Utilisez la procédure suivante pour créer une attribution de stratégie et assigner la définition de stratégie au niveau du groupe de ressources.

1. Copiez l’extrait de code JSON suivant pour créer un fichier d’attribution de stratégie JSON. Remplacez les informations de l’exemple entre &lt;&gt; par vos propres valeurs.

   ```json
   {
       "properties": {
           "description": "This policy assignment makes sure that storage accounts with exposure to Public Networks are audited.",
           "displayName": "Audit Storage Accounts Open to Public Networks Assignment",
           "parameters": {},
           "policyDefinitionId": "/subscriptions/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks",
           "scope": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>"
       }
   }
   ```

1. Créez l’attribution de stratégie à l’aide de l’appel suivant :

   ```console
   armclient PUT "/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.Authorization/policyAssignments/Audit Storage Accounts Open to Public Networks?api-version=2019-09-01" @<path to Assignment JSON file>
   ```

   Remplacez les informations de l’exemple entre &lt;&gt; par vos propres valeurs.

   Pour plus d’informations sur les appels HTTP à l’API REST, consultez la page [Ressources API REST Azure](/rest/api/resources/).

### <a name="create-and-assign-a-policy-definition-with-azure-cli"></a>Créer et attribuer une définition de stratégie avec Azure CLI

Pour créer une définition de stratégie, procédez comme suit :

1. Copiez l’extrait de code JSON suivant pour créer un fichier d’attribution de stratégie JSON.

   ```json
   {
       "if": {
           "allOf": [{
                   "field": "type",
                   "equals": "Microsoft.Storage/storageAccounts"
               },
               {
                   "field": "Microsoft.Storage/storageAccounts/networkAcls.defaultAction",
                   "equals": "Allow"
               }
           ]
       },
       "then": {
           "effect": "audit"
       }
   }
   ```

   Pour plus d’informations sur la création d’une définition de stratégie, consultez la page [Structure de définition Azure Policy](../concepts/definition-structure.md).

1. Pour créer une définition de stratégie, exécutez la commande suivante :

   ```azurecli-interactive
   az policy definition create --name 'audit-storage-accounts-open-to-public-networks' --display-name 'Audit Storage Accounts Open to Public Networks' --description 'This policy ensures that storage accounts with exposures to public networks are audited.' --rules '<path to json file>' --mode All
   ```

   La commande crée une définition de stratégie nommée _Audit Storage Accounts Open to Public Networks_.
   Pour plus d’informations sur les autres paramètres utilisables, consultez [az policy definition create](/cli/azure/policy/definition#az_policy_definition_create).

   Lorsqu’il est appelé sans paramètre d’emplacement, par défaut, `az policy definition creation` enregistre la définition de stratégie dans l’abonnement sélectionné du contexte de sessions. Pour enregistrer la définition dans un autre emplacement, utilisez les paramètres suivants :

   - **subscription** : enregistrer dans un autre abonnement. Nécessite une valeur _GUID_ pour l’ID d’abonnement ou une valeur _chaîne_ pour le nom de l’abonnement.
   - **--management-group** : enregistrer dans un groupe d’administration. Une valeur de _chaîne_ est nécessaire.

1. Pour créer une attribution de stratégie, utilisez la commande suivante. Remplacez les informations de l’exemple entre &lt;&gt; par vos propres valeurs.

   ```azurecli-interactive
   az policy assignment create --name '<name>' --scope '<scope>' --policy '<policy definition ID>'
   ```

   Le paramètre **Scope** sur `az policy assignment create` peut être défini pour un groupe d’administration, un abonnement, un groupe de ressources ou une seule ressource. Le paramètre utilise un chemin de ressource complet. Pour chaque conteneur, le modèle pour **scope** est le suivant. Remplacez `{rName}`, `{rgName}`, `{subId}` et `{mgName}` par le nom de la ressource, le nom du groupe de ressources, l’ID de l’abonnement et le nom du groupe d’administration, respectivement. `{rType}` est remplacé par le **type de la ressource**, comme `Microsoft.Compute/virtualMachines` pour une machine virtuelle.

   - Ressource : `/subscriptions/{subID}/resourceGroups/{rgName}/providers/{rType}/{rName}`
   - Groupe de ressources : `/subscriptions/{subID}/resourceGroups/{rgName}`
   - Abonnement : `/subscriptions/{subID}`
   - Groupe d'administration : `/providers/Microsoft.Management/managementGroups/{mgName}`

Vous pouvez obtenir l’ID de définition de stratégie Azure à l’aide de PowerShell avec la commande suivante :

```azurecli-interactive
az policy definition show --name 'Audit Storage Accounts with Open Public Networks'
```

L’ID de définition de stratégie pour la définition de stratégie que vous avez créée doit ressembler à ce qui suit :

```output
"/subscription/<subscriptionId>/providers/Microsoft.Authorization/policyDefinitions/Audit Storage Accounts Open to Public Networks"
```

Pour plus d’informations sur la façon dont vous pouvez gérer les stratégies de ressources avec Azure CLI, consultez [Stratégies de ressources Azure CLI](/cli/azure/policy).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les commandes et les requêtes de cet article, consultez les articles suivants.

- [Ressources API REST Azure](/rest/api/resources/)
- [Modules Azure PowerShell](/powershell/module/az.resources/#policies)
- [Commandes de stratégie Azure CLI](/cli/azure/policy)
- [Informations de référence sur l’API REST du fournisseur de ressources Azure Policy Insights](/rest/api/policy-insights)
- [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).
