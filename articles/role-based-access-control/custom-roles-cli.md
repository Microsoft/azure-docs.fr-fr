---
title: Créer ou mettre à jour des rôles personnalisés Azure à l’aide d’Azure CLI - RBAC Azure
description: Découvrez comment lister, créer, mettre à jour ou supprimer des rôles personnalisés Azure à l’aide d’Azure CLI et du contrôle d’accès en fonction du rôle Azure (RBAC Azure).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 3483ee01-8177-49e7-b337-4d5cb14f5e32
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/17/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 31dabcf77f0db76047919fa76d00f1c5ed3c96d6
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97369138"
---
# <a name="create-or-update-azure-custom-roles-using-azure-cli"></a>Créer ou mettre à jour des rôles personnalisés Azure à l’aide d’Azure CLI

> [!IMPORTANT]
> L’ajout d’un groupe d’administration à `AssignableScopes` est actuellement en préversion.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si les [rôles intégrés Azure](built-in-roles.md) ne répondent pas aux besoins spécifiques de votre organisation, vous pouvez créer vos propres rôles personnalisés. Cet article explique comment lister, créer, mettre à jour ou supprimer des rôles personnalisés à l’aide d’Azure CLI.

Pour obtenir un tutoriel pas à pas sur la création d’un rôle personnalisé, consultez [Tutoriel : Créer un rôle personnalisé Azure à l’aide d’Azure CLI](tutorial-custom-role-cli.md).

## <a name="prerequisites"></a>Prérequis

Pour créer des rôles personnalisés, vous avez besoin des éléments suivants :

- autorisations nécessaires pour créer des rôles personnalisés, par exemple, [Propriétaire](built-in-roles.md#owner) ou [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) ;
- [Azure Cloud Shell](../cloud-shell/overview.md) ou [Azure CLI](/cli/azure/install-azure-cli)

## <a name="list-custom-roles"></a>Répertorier les rôles personnalisés

Pour lister les rôles personnalisés pouvant être affectés, utilisez [az role definition list](/cli/azure/role/definition#az-role-definition-list). L’exemple suivant liste tous les rôles personnalisés de l’abonnement actuel.

```azurecli
az role definition list --custom-role-only true --output json --query '[].{roleName:roleName, roleType:roleType}'
```

```json
[
  {
    "roleName": "My Management Contributor",
    "type": "CustomRole"
  },
  {
    "roleName": "My Service Reader Role",
    "type": "CustomRole"
  },
  {
    "roleName": "Virtual Machine Operator",
    "type": "CustomRole"
  }
]
```

## <a name="list-a-custom-role-definition"></a>Lister une définition de rôle personnalisé

Pour lister une définition de rôle personnalisé, utilisez [az role definition list](/cli/azure/role/definition#az-role-definition-list). Il s’agit de la même commande que vous utiliseriez pour un rôle intégré.

```azurecli
az role definition list --name {roleName}
```

L’exemple suivant liste la définition de rôle *Opérateur de machine virtuelle* :

```azurecli
az role definition list --name "Virtual Machine Operator"
```

```json
[
  {
    "assignableScopes": [
      "/subscriptions/{subscriptionId}"
    ],
    "description": "Can monitor and restart virtual machines.",
    "id": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/00000000-0000-0000-0000-000000000000",
    "name": "00000000-0000-0000-0000-000000000000",
    "permissions": [
      {
        "actions": [
          "Microsoft.Storage/*/read",
          "Microsoft.Network/*/read",
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/start/action",
          "Microsoft.Compute/virtualMachines/restart/action",
          "Microsoft.Authorization/*/read",
          "Microsoft.ResourceHealth/availabilityStatuses/read",
          "Microsoft.Resources/subscriptions/resourceGroups/read",
          "Microsoft.Insights/alertRules/*",
          "Microsoft.Insights/diagnosticSettings/*",
          "Microsoft.Support/*"
        ],
        "dataActions": [],
        "notActions": [],
        "notDataActions": []
      }
    ],
    "roleName": "Virtual Machine Operator",
    "roleType": "CustomRole",
    "type": "Microsoft.Authorization/roleDefinitions"
  }
]
```

L’exemple suivant liste seulement les actions du rôle *Opérateur de machine virtuelle* :

```azurecli
az role definition list --name "Virtual Machine Operator" --output json --query '[].permissions[0].actions'
```

```json
[
  [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ]
]
```

## <a name="create-a-custom-role"></a>Créer un rôle personnalisé

Pour créer un rôle personnalisé, utilisez [az role definition create](/cli/azure/role/definition#az-role-definition-create). La définition de rôle peut être une description JSON ou le chemin d’un fichier contenant une description JSON.

```azurecli
az role definition create --role-definition {roleDefinition}
```

L’exemple suivant crée un rôle personnalisé appelé *Virtual Machine Operator*. Ce rôle personnalisé attribue l’accès à toutes les opérations de lecture (« read ») des fournisseurs de ressources *Microsoft.Compute*, *Microsoft.Storage* et *Microsoft.Network* et attribue l’accès pour démarrer (« start »), redémarrer (« restart ») et surveiller (« monitor ») les machines virtuelles. Ce rôle personnalisé peut être utilisé dans deux abonnements. Cet exemple utilise un fichier JSON en tant qu’entrée.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}"
  ]
}
```

```azurecli
az role definition create --role-definition ~/roles/vmoperator.json
```

## <a name="update-a-custom-role"></a>Mettre à jour un rôle personnalisé

Pour mettre à jour un rôle personnalisé, utilisez d’abord [az role definition list](/cli/azure/role/definition#az-role-definition-list) pour récupérer la définition de rôle. Apportez ensuite les modifications souhaitées à la définition de rôle. Enfin, utilisez [az role definition update](/cli/azure/role/definition#az-role-definition-update) pour enregistrer la définition de rôle mise à jour.

```azurecli
az role definition update --role-definition {roleDefinition}
```

L’exemple suivant ajoute l’opération *Microsoft.Insights/diagnosticSettings/* à `Actions` et ajoute un groupe d’administration à `AssignableScopes` pour le rôle personnalisé *Opérateur de machine virtuelle*. L’ajout d’un groupe d’administration à `AssignableScopes` est actuellement en préversion.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.ResourceHealth/availabilityStatuses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId1}",
    "/subscriptions/{subscriptionId2}",
    "/providers/Microsoft.Management/managementGroups/marketing-group"
  ]
}
```

```azurecli
az role definition update --role-definition ~/roles/vmoperator.json
```

## <a name="delete-a-custom-role"></a>Supprimer un rôle personnalisé

Pour supprimer un rôle personnalisé, utilisez [az role definition delete](/cli/azure/role/definition#az-role-definition-delete). Pour spécifier le rôle à supprimer, utilisez le nom ou l’ID du rôle. Pour déterminer l’ID du rôle, utilisez [az role definition list](/cli/azure/role/definition#az-role-definition-list).

```azurecli
az role definition delete --name {roleNameOrId}
```

L’exemple suivant supprime le rôle personnalisé *Virtual Machine Operator*.

```azurecli
az role definition delete --name "Virtual Machine Operator"
```

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Créer un rôle personnalisé Azure à l’aide d’Azure CLI](tutorial-custom-role-cli.md)
- [Rôle personnalisés Azure](custom-roles.md)
- [Opérations de fournisseur de ressources Azure](resource-provider-operations.md)