---
title: Lister les affectations de refus Azure avec l’API REST - Azure RBAC
description: Apprenez à dresser la liste des affectations de refus Azure pour les utilisateurs, groupes et applications avec l’API REST et le contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: active-directory
documentationcenter: na
author: rolyon
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: role-based-access-control
ms.workload: multiple
ms.tgt_pltfrm: rest-api
ms.devlang: na
ms.topic: how-to
ms.date: 03/19/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 2f835c270930734bf9963a7c7c3168b873eddaf6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84791909"
---
# <a name="list-azure-deny-assignments-using-the-rest-api"></a>Lister les affectations de refus Azure avec l’API REST

Les [affectations de refus Azure](deny-assignments.md) empêchent les utilisateurs d’effectuer des actions particulières sur les ressources Azure, même si une attribution de rôle leur confère un accès. Cet article décrit comment lister les affectations de refus existantes à l’aide de l’API REST.

> [!NOTE]
> Vous ne pouvez pas directement créer vos propres affectations de refus. Pour obtenir des informations sur la création des affectations de refus, consultez [Affectations de refus Azure](deny-assignments.md).

## <a name="prerequisites"></a>Prérequis

Pour obtenir des informations sur une affectation de refus, vous devez disposer de :

- l’autorisation `Microsoft.Authorization/denyAssignments/read`, qui est incluse dans la plupart des [rôles intégrés Azure](built-in-roles.md).

## <a name="list-a-single-deny-assignment"></a>Lister une seule affectation de refus

1. Commencez par la requête suivante :

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments/{deny-assignment-id}?api-version=2018-07-01-preview
    ```

1. Dans l’URI, remplacez *{scope}* par l’étendue dont vous souhaitez lister les affectations de refus.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId}` | Abonnement |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Resource group |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/providers/Microsoft.Web/sites/mysite1` | Ressource |

1. Remplacez *{deny-assignment-id}* par l’identificateur de l’affectation de refus à récupérer.

## <a name="list-multiple-deny-assignments"></a>Lister plusieurs affectations de refus

1. Commencez par l’une des requêtes suivantes :

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview
    ```

    Avec des paramètres facultatifs :

    ```http
    GET https://management.azure.com/{scope}/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter={filter}
    ```

1. Dans l’URI, remplacez *{scope}* par l’étendue dont vous souhaitez lister les affectations de refus.

    > [!div class="mx-tableFixed"]
    > | Étendue | Type |
    > | --- | --- |
    > | `subscriptions/{subscriptionId}` | Abonnement |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1` | Resource group |
    > | `subscriptions/{subscriptionId}/resourceGroups/myresourcegroup1/providers/Microsoft.Web/sites/mysite1` | Ressource |

1. Remplacez *{filter}* par la condition que vous voulez appliquer pour filtrer la liste des affectations de refus.

    > [!div class="mx-tableFixed"]
    > | Filtrer | Description |
    > | --- | --- |
    > | (aucun filtre) | Répertorie toutes les affectations de refus dans l’étendue spécifiée, mais aussi dans les étendues au-dessus et en dessous. |
    > | `$filter=atScope()` | Répertorie les affectations de refus uniquement dans la portée spécifiée et dans les étendues au-dessus. N’inclut pas les affectations de refus dans les étendues en dessous. |
    > | `$filter=assignedTo('{objectId}')` | Répertorie les affectations de refus pour l’utilisateur ou le principal de service spécifié.<br/>Si l’utilisateur est membre d’un groupe auquel un rôle a été attribué, cette affectation de refus est également répertoriée. Ce filtre est transitif pour les groupes, ce qui signifie que si l’utilisateur est membre d’un groupe et que ce groupe est membre d’un autre groupe auquel un refus a été affecté, cette affectation de refus est également répertoriée.<br/>Ce filtre accepte uniquement un ID d’objet pour un utilisateur ou principal de service. Vous ne pouvez pas transmettre un ID d’objet pour un groupe. |
    > | `$filter=atScope()+and+assignedTo('{objectId}')` | Répertorie les affectations de refus pour l’utilisateur ou le principal de service spécifié, et au niveau de l’étendue spécifiée. |
    > | `$filter=denyAssignmentName+eq+'{deny-assignment-name}'` | Répertorie les affectations de refus avec le nom spécifié. |
    > | `$filter=principalId+eq+'{objectId}'` | Répertorie les affectations de refus pour l’utilisateur, le groupe ou le principal de service spécifié. |

## <a name="list-deny-assignments-at-the-root-scope-"></a>Lister les affectations de refus dans l’étendue racine (/)

1. Élevez votre accès comme cela est décrit dans [Élever l’accès pour gérer tous les abonnements et groupes d’administration Azure](elevate-access-global-admin.md).

1. Utilisez la requête suivante :

    ```http
    GET https://management.azure.com/providers/Microsoft.Authorization/denyAssignments?api-version=2018-07-01-preview&$filter={filter}
    ```

1. Remplacez *{filter}* par la condition que vous voulez appliquer pour filtrer la liste des affectations de refus. Un filtre est obligatoire.

    > [!div class="mx-tableFixed"]
    > | Filtrer | Description |
    > | --- | --- |
    > | `$filter=atScope()` | Liste les affectations de refus dans l’étendue racine uniquement. N’inclut pas les affectations de refus dans les étendues en dessous. |
    > | `$filter=denyAssignmentName+eq+'{deny-assignment-name}'` | Liste les affectations de refus avec le nom spécifié. |

1. Supprimez l’accès avec élévation des privilèges.

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre les affectations de refus Azure](deny-assignments.md)
- [Élever l’accès pour gérer tous les abonnements et groupes d’administration Azure](elevate-access-global-admin.md)
- [Référence de l'API REST Azure](/rest/api/azure/)
