---
title: Créer ou mettre à jour des rôles personnalisés Azure à l’aide de l’API REST - RBAC Azure
description: Découvrez comment lister, créer, mettre à jour ou supprimer des rôles personnalisés Azure à l’aide de l’API REST et du contrôle d’accès en fonction du rôle Azure (RBAC Azure).
services: active-directory
documentationcenter: na
author: rolyon
manager: mtillman
editor: ''
ms.assetid: 1f90228a-7aac-4ea7-ad82-b57d222ab128
ms.service: role-based-access-control
ms.workload: multiple
ms.tgt_pltfrm: rest-api
ms.devlang: na
ms.topic: how-to
ms.date: 03/19/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: b459f44308827308c28687db3c3fc33df470ea8d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84790185"
---
# <a name="create-or-update-azure-custom-roles-using-the-rest-api"></a>Créer ou mettre à jour des rôles personnalisés Azure à l’aide de l’API REST

> [!IMPORTANT]
> L’ajout d’un groupe d’administration à `AssignableScopes` est actuellement en préversion.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si les [rôles intégrés Azure](built-in-roles.md) ne répondent pas aux besoins spécifiques de votre organisation, vous pouvez créer vos propres rôles personnalisés. Cet article explique comment lister, créer, mettre à jour ou supprimer des rôles personnalisés à l’aide de l’API REST.

## <a name="list-custom-roles"></a>Répertorier les rôles personnalisés

Pour répertorier tous les rôles personnalisés dans un répertoire, utilisez l’API REST [Définitions de rôles - Liste](/rest/api/authorization/roledefinitions/list).

1. Commencez par la requête suivante :

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. Remplacez *{filter}* par le type de rôle.

    > [!div class="mx-tableFixed"]
    > | Filtrer | Description |
    > | --- | --- |
    > | `$filter=type+eq+'CustomRole'` | Filtre basé sur le type CustomRole |

## <a name="list-custom-roles-at-a-scope"></a>Répertorier les rôles personnalisés dans une étendue

Pour répertorier les rôles personnalisés dans une étendue, utilisez l’API REST [Définitions de rôles - Liste](/rest/api/authorization/roledefinitions/list).

1. Commencez par la requête suivante :

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. Dans l’URI, remplacez *{scope}* par l’étendue dont vous souhaitez lister les rôles.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Abonnement |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Resource group |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}/providers/Microsoft.Web/sites/{site1}` | Ressource |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Groupe d’administration |

1. Remplacez *{filter}* par le type de rôle.

    > [!div class="mx-tableFixed"]
    > | Filtrer | Description |
    > | --- | --- |
    > | `$filter=type+eq+'CustomRole'` | Filtre basé sur le type CustomRole |

## <a name="list-a-custom-role-definition-by-name"></a>Lister une définition de rôle personnalisé par nom

Pour obtenir des informations sur le rôle personnalisé en fonction de son nom d’affichage, utilisez l’API REST [Définitions de rôles – Obtenir](/rest/api/authorization/roledefinitions/get).

1. Commencez par la requête suivante :

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01&$filter={filter}
    ```

1. Dans l’URI, remplacez *{scope}* par l’étendue dont vous souhaitez lister les rôles.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Abonnement |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Resource group |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}/providers/Microsoft.Web/sites/{site1}` | Ressource |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Groupe d’administration |

1. Remplacez *{filter}* par le nom d’affichage du rôle.

    > [!div class="mx-tableFixed"]
    > | Filtrer | Description |
    > | --- | --- |
    > | `$filter=roleName+eq+'{roleDisplayName}'` | Utilisez la forme codée de l’URL du nom d’affichage exact du rôle. Par exemple, `$filter=roleName+eq+'Virtual%20Machine%20Contributor'`. |

## <a name="list-a-custom-role-definition-by-id"></a>Lister une définition de rôle personnalisé par ID

Pour obtenir des informations sur un rôle personnalisé en fonction de son identificateur unique, utilisez l’API REST [Définitions de rôles – Obtenir](/rest/api/authorization/roledefinitions/get).

1. Utilisez l’API REST [Définitions de rôles – Lister](/rest/api/authorization/roledefinitions/list) pour obtenir l’identificateur GUID du rôle.

1. Commencez par la requête suivante :

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. Dans l’URI, remplacez *{scope}* par l’étendue dont vous souhaitez lister les rôles.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Abonnement |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Resource group |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}/providers/Microsoft.Web/sites/{site1}` | Ressource |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Groupe d’administration |

1. Remplacez *{roleDefinitionId}* par l’identificateur GUID de la définition du rôle.

## <a name="create-a-custom-role"></a>Créer un rôle personnalisé

Pour créer un rôle personnalisé, utilisez l’API REST [Définitions de rôles – Créer ou mettre à jour](/rest/api/authorization/roledefinitions/createorupdate). Pour appeler cette API, vous devez être connecté avec un rôle utilisateur qui dispose de l’autorisation `Microsoft.Authorization/roleDefinitions/write` sur tous les `assignableScopes`. Parmi les rôles prédéfinis, seuls les rôles [Propriétaire](built-in-roles.md#owner) et [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) disposent de cette autorisation.

1. Consultez la liste des [opérations de fournisseur de ressources](resource-provider-operations.md) disponibles pour créer les autorisations de votre rôle personnalisé.

1. Utilisez un outil GUID pour générer un identificateur unique qui servira d’identificateur de rôle personnalisé. Cet identificateur est au format : `00000000-0000-0000-0000-000000000000`.

1. Commencez par la requête et le corps suivants :

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId1}",
          "/subscriptions/{subscriptionId2}",
          "/subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}",
          "/subscriptions/{subscriptionId2}/resourceGroups/{resourceGroup2}",
          "/providers/Microsoft.Management/managementGroups/{groupId1}"
        ]
      }
    }
    ```

1. Dans l’URI, remplacez *{scope}* par le premier élément `assignableScopes` du rôle personnalisé.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Abonnement |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Resource group |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Groupe d’administration |

1. Remplacez *{roleDefinitionId}* par l’identificateur GUID du rôle personnalisé.

1. Dans le corps de la demande, remplacez *{roleDefinitionId}* par l’identificateur GUID.

1. Si `assignableScopes` est un abonnement ou un groupe de ressources, remplacez les instances *{subscriptionId}* ou *{resourceGroup}* par vos identificateurs.

1. Si `assignableScopes` est un groupe d’administration, remplacez l’instance *{groupId}* par l’identificateur de votre groupe d’administration. L’ajout d’un groupe d’administration à `assignableScopes` est actuellement en préversion.

1. Dans la propriété `actions`, ajoutez les opérations autorisées par le rôle.

1. Dans la propriété `notActions`, ajoutez les opérations exclues des `actions` autorisées.

1. Dans les propriétés `roleName` et `description`, spécifiez un nom de rôle unique et une description. Pour plus d’informations sur les propriétés, consultez [Rôles personnalisés Azure](custom-roles.md).

    Voici un exemple de corps de demande :

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
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
              "Microsoft.Support/*"
            ],
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000",
          "/providers/Microsoft.Management/managementGroups/marketing-group"
        ]
      }
    }
    ```

## <a name="update-a-custom-role"></a>Mettre à jour un rôle personnalisé

Pour mettre à jour un rôle personnalisé, utilisez l’API REST [Définitions de rôles – Créer ou mettre à jour](/rest/api/authorization/roledefinitions/createorupdate). Pour appeler cette API, vous devez être connecté avec un rôle utilisateur qui dispose de l’autorisation `Microsoft.Authorization/roleDefinitions/write` sur tous les `assignableScopes`. Parmi les rôles prédéfinis, seuls les rôles [Propriétaire](built-in-roles.md#owner) et [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) disposent de cette autorisation.

1. Utilisez l’API REST [Définitions de rôles – Lister](/rest/api/authorization/roledefinitions/list) ou [Définitions de rôles – Obtenir](/rest/api/authorization/roledefinitions/get) pour obtenir des informations sur le rôle personnalisé. Pour plus d’informations, consultez la section précédente [Lister les rôles personnalisés](#list-custom-roles).

1. Commencez par la requête suivante :

    ```http
    PUT https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. Dans l’URI, remplacez *{scope}* par le premier élément `assignableScopes` du rôle personnalisé.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Abonnement |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Resource group |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Groupe d’administration |

1. Remplacez *{roleDefinitionId}* par l’identificateur GUID du rôle personnalisé.

1. En fonction des informations sur le rôle personnalisé, créez un corps de demande au format suivant :

    ```json
    {
      "name": "{roleDefinitionId}",
      "properties": {
        "roleName": "",
        "description": "",
        "type": "CustomRole",
        "permissions": [
          {
            "actions": [
    
            ],
            "notActions": [
    
            ]
          }
        ],
        "assignableScopes": [
          "/subscriptions/{subscriptionId1}",
          "/subscriptions/{subscriptionId2}",
          "/subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}",
          "/subscriptions/{subscriptionId2}/resourceGroups/{resourceGroup2}",
          "/providers/Microsoft.Management/managementGroups/{groupId1}"
        ]
      }
    }
    ```

1. Mettez à jour le corps de la demande avec les modifications que vous souhaitez apporter au rôle personnalisé.

    Voici un exemple de corps de demande avec ajout d’une nouvelle action de paramètres de diagnostic :

    ```json
    {
      "name": "88888888-8888-8888-8888-888888888888",
      "properties": {
        "roleName": "Virtual Machine Operator",
        "description": "Can monitor and restart virtual machines.",
        "type": "CustomRole",
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
            "notActions": []
          }
        ],
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000",
          "/providers/Microsoft.Management/managementGroups/marketing-group"
        ]
      }
    }
    ```

## <a name="delete-a-custom-role"></a>Supprimer un rôle personnalisé

Pour supprimer un rôle personnalisé, utilisez l’API REST [Définitions de rôles – Supprimer](/rest/api/authorization/roledefinitions/delete). Pour appeler cette API, vous devez être connecté avec un rôle utilisateur qui dispose de l’autorisation `Microsoft.Authorization/roleDefinitions/delete` sur tous les `assignableScopes`. Parmi les rôles prédéfinis, seuls les rôles [Propriétaire](built-in-roles.md#owner) et [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) disposent de cette autorisation.

1. Utilisez l’API REST [Définitions de rôles – Lister](/rest/api/authorization/roledefinitions/list) ou [Définitions de rôles – Obtenir](/rest/api/authorization/roledefinitions/get) pour obtenir l’identificateur GUID du rôle personnalisé. Pour plus d’informations, consultez la section précédente [Lister les rôles personnalisés](#list-custom-roles).

1. Commencez par la requête suivante :

    ```http
    DELETE https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}?api-version=2015-07-01
    ```

1. Dans l’URI, remplacez *{scope}* par l’étendue dont vous souhaitez supprimer le rôle personnalisé.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId1}` | Abonnement |
    > | `subscriptions/{subscriptionId1}/resourceGroups/{resourceGroup1}` | Resource group |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Groupe d’administration |

1. Remplacez *{roleDefinitionId}* par l’identificateur GUID du rôle personnalisé.

## <a name="next-steps"></a>Étapes suivantes

- [Rôle personnalisés Azure](custom-roles.md)
- [Ajouter ou supprimer des attributions de rôle Azure à l’aide de l’API REST](role-assignments-rest.md)
- [Référence de l'API REST Azure](/rest/api/azure/)
