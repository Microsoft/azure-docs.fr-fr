---
title: Répertorier les attributions de rôle Azure à l’aide de l’API REST – Azure RBAC
description: Découvrez comment déterminer les ressources, utilisateurs, groupes, principaux de service ou identités managées qui ont accès l’API REST ou au contrôle d’accès en fonction du rôle Azure (Azure RBAC).
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
ms.date: 05/06/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 634e1111c9374a1749e7dbb0666740ce2833a688
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84790974"
---
# <a name="list-azure-role-assignments-using-the-rest-api"></a>Répertorier les attributions de rôle Azure à l’aide de l’API REST

[!INCLUDE [Azure RBAC definition list access](../../includes/role-based-access-control-definition-list.md)] Cet article explique comment lister les attributions de rôles à l’aide de l’API REST.

> [!NOTE]
> Si votre organisation possède des fonctions de gestion externalisées pour un fournisseur de services qui utilise la [gestion des ressources déléguées Azure](../lighthouse/concepts/azure-delegated-resource-management.md), les attributions de rôles autorisées par ce fournisseur de services ne seront pas affichées ici.

## <a name="list-role-assignments"></a>Répertorier les attributions de rôles

Dans Azure RBAC, vous répertoriez les attributions de rôles pour répertorier les accès. Pour répertorier les attributions de rôles, utilisez l’une des API REST de la [liste d’attributions de rôles](/rest/api/authorization/roleassignments/list). Pour affiner vos résultats, vous spécifiez une étendue et un filtre facultatif.

1. Commencez par la requête suivante :

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter={filter}
    ```

1. Dans l’URI, remplacez *{scope}* par l’étendue dont vous souhaitez lister les attributions de rôle.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `providers/Microsoft.Management/managementGroups/{groupId1}` | Groupe d’administration |
    > | `subscriptions/{subscriptionId1}` | Abonnement |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1` | Resource group |
    > | `subscriptions/{subscriptionId1}/resourceGroups/myresourcegroup1/providers/Microsoft.Web/sites/mysite1` | Ressource |

    Dans l’exemple précédent, microsoft.web est un fournisseur de ressources qui fait référence à une instance App Service. De la même façon, vous pouvez utiliser tout autre fournisseur de ressources et spécifier l’étendue. Pour plus d’informations, consultez [Fournisseurs et types de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md) et [Opérations du fournisseur de ressources Azure Resource Manager](resource-provider-operations.md) prises en charge.  
     
1. Remplacez *{filter}* par la condition que vous voulez appliquer pour filtrer la liste des attributions de rôle.

    > [!div class="mx-tableFixed"]
    > | Filtrer | Description |
    > | --- | --- |
    > | `$filter=atScope()` | Répertorie les attributions de rôles pour l’étendue spécifiée seulement, sans y inclure les attributions de rôles à des étendues secondaires. |
    > | `$filter=assignedTo('{objectId}')` | Répertorie les attributions de rôles pour un utilisateur ou un principal de service spécifié.<br/>Si l’utilisateur est membre d’un groupe auquel un rôle a été attribué, cette attribution de rôle est également répertoriée. Ce filtre est transitif pour les groupes, ce qui signifie que si l’utilisateur est membre d’un groupe et que ce groupe est membre d’un autre groupe auquel un rôle a été attribué, cette attribution de rôle est également répertoriée.<br/>Ce filtre accepte uniquement un ID d’objet pour un utilisateur ou principal de service. Vous ne pouvez pas transmettre un ID d’objet pour un groupe. |
    > | `$filter=atScope()+and+assignedTo('{objectId}')` | Liste les attributions de rôles pour l’utilisateur ou le principal de service spécifié, et au niveau de l’étendue spécifiée. |
    > | `$filter=principalId+eq+'{objectId}'` | Répertorie les attributions de rôles pour un utilisateur, un groupe ou un principal de service spécifié. |

La requête suivante répertorie toutes les attributions de rôles pour l’utilisateur spécifié au niveau de l’étendue de l’abonnement :

```http
GET https://management.azure.com/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleAssignments?api-version=2015-07-01&$filter=atScope()+and+assignedTo('{objectId1}')
```

Voici un exemple de sortie :

```json
{
    "value": [
        {
            "properties": {
                "roleDefinitionId": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleDefinitions/2a2b9908-6ea1-4ae2-8e65-a410df84e7d1",
                "principalId": "{objectId1}",
                "scope": "/subscriptions/{subscriptionId1}",
                "createdOn": "2019-01-15T21:08:45.4904312Z",
                "updatedOn": "2019-01-15T21:08:45.4904312Z",
                "createdBy": "{createdByObjectId1}",
                "updatedBy": "{updatedByObjectId1}"
            },
            "id": "/subscriptions/{subscriptionId1}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId1}",
            "type": "Microsoft.Authorization/roleAssignments",
            "name": "{roleAssignmentId1}"
        }
    ]
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter ou supprimer des attributions de rôle Azure à l’aide de l’API REST](role-assignments-rest.md)
- [Référence de l'API REST Azure](/rest/api/azure/)
