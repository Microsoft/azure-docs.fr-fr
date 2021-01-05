---
title: 'Tutoriel : Créer un rôle personnalisé Azure avec Azure CLI - Azure RBAC'
description: Découvrez dans ce tutoriel comment créer un rôle personnalisé Azure à l’aide d’Azure CLI et du contrôle d’accès en fonction du rôle Azure (RBAC Azure).
services: active-directory
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 02/20/2019
ms.author: rolyon
ms.custom: devx-track-azurecli
ms.openlocfilehash: 319bca74c8e781e5dc5022e9fb901b2edca24a80
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87485641"
---
# <a name="tutorial-create-an-azure-custom-role-using-azure-cli"></a>Tutoriel : Créer un rôle personnalisé Azure à l’aide d’Azure CLI

Si les [rôles intégrés Azure](built-in-roles.md) ne répondent pas aux besoins spécifiques de votre organisation, vous pouvez créer vos propres rôles personnalisés. Pour ce tutoriel, vous allez créer un rôle personnalisé nommé Reader Support Tickets à l’aide d’Azure CLI. Le rôle personnalisé permet à l’utilisateur de consulter tous les éléments du plan de gestion d’un abonnement et d’ouvrir des tickets de support.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un rôle personnalisé
> * Répertorier les rôles personnalisés
> * Mettre à jour un rôle personnalisé
> * Supprimer un rôle personnalisé

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour exécuter ce didacticiel, les éléments suivants sont nécessaires :

- autorisations nécessaires pour créer des rôles personnalisés, par exemple, [Propriétaire](built-in-roles.md#owner) ou [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) ;
- [Azure Cloud Shell](../cloud-shell/overview.md) ou [Azure CLI](/cli/azure/install-azure-cli)

## <a name="sign-in-to-azure-cli"></a>Connectez-vous à Azure CLI

Connectez-vous à [Azure CLI](/cli/azure/authenticate-azure-cli).

## <a name="create-a-custom-role"></a>Créer un rôle personnalisé

Pour créer un rôle personnalisé, le plus simple consiste à débuter avec un modèle JSON, à ajouter vos modifications, puis à créer un nouveau rôle.

1. Passez en revue la liste des opérations pour le [fournisseur de ressources Microsoft.Support](resource-provider-operations.md#microsoftsupport). Il est utile de connaître les opérations qui sont disponibles pour créer vos autorisations.

    | Opération | Description |
    | --- | --- |
    | Microsoft.Support/register/action | S’inscrit auprès du fournisseur de ressources de support. |
    | Microsoft.Support/supportTickets/read | Récupère les détails du ticket de support (notamment l’état, la gravité, les détails du contact et les communications) ou la liste des tickets de support des abonnements. |
    | Microsoft.Support/supportTickets/write | Crée ou met à jour un ticket de support. Vous pouvez créer un ticket de support pour les problèmes techniques ou les problèmes liés à la facturation, aux quotas ou à la gestion des abonnements. Vous pouvez mettre à jour la gravité des problèmes, les détails du contact et les communications pour les tickets de support existants. |
    
1. Créez un fichier nommé **ReaderSupportRole.json**.

1. Ouvrez ReaderSupportRole.json dans un éditeur et ajoutez le code JSON suivant.

    Pour plus d’informations sur les différentes propriétés, consultez [Rôles personnalisés Azure](custom-roles.md).

    ```json
    {
      "Name": "",
      "IsCustom": true,
      "Description": "",
      "Actions": [],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/{subscriptionId1}"
      ]
    }
    ```
    
1. Ajoutez les opérations suivantes à la propriété `Actions`. Ces actions permettent à l’utilisateur de consulter tous les éléments de l’abonnement et de créer des tickets de support.

    ```
    "*/read",
    "Microsoft.Support/*"
    ```

1. Obtenez l’ID de votre abonnement à l’aide de la commande [az account list](/cli/azure/account#az-account-list).

    ```azurecli
    az account list --output table
    ```

1. Dans `AssignableScopes`, remplacez `{subscriptionId1}` par votre ID d’abonnement.

    Vous devez ajouter des ID d’abonnements explicites, sinon vous ne serez pas autorisé à importer le rôle dans votre abonnement.

1. Remplacez les valeurs de propriétés `Name` et `Description` par « Reader Support Tickets » et « View everything in the subscription and also open support tickets ».

    Votre fichier JSON doit ressembler à ceci :

    ```json
    {
      "Name": "Reader Support Tickets",
      "IsCustom": true,
      "Description": "View everything in the subscription and also open support tickets.",
      "Actions": [
        "*/read",
        "Microsoft.Support/*"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
    
1. Pour créer le rôle personnalisé, utilisez la commande [az role definition create](/cli/azure/role/definition#az-role-definition-create) et spécifiez le fichier de définition de rôle JSON.

    ```azurecli
    az role definition create --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

    Le nouveau rôle personnalisé est maintenant disponible et peut être affecté à des utilisateurs, des groupes ou des principaux de service tout comme des rôles intégrés.

## <a name="list-custom-roles"></a>Répertorier les rôles personnalisés

- Pour répertorier tous vos rôles personnalisés, utilisez la commande [az role definition list](/cli/azure/role/definition#az-role-definition-list) avec le paramètre `--custom-role-only`.

    ```azurecli
    az role definition list --custom-role-only true
    ```
    
    ```Output
    [
      {
        "additionalProperties": {},
        "assignableScopes": [
          "/subscriptions/00000000-0000-0000-0000-000000000000"
        ],
        "description": "View everything in the subscription and also open support tickets.",
        "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
        "name": "22222222-2222-2222-2222-222222222222",
        "permissions": [
          {
            "actions": [
              "*/read",
              "Microsoft.Support/*",
              "Microsoft.Resources/deployments/*",
              "Microsoft.Insights/diagnosticSettings/*/read"
            ],
            "additionalProperties": {},
            "dataActions": [],
            "notActions": [],
            "notDataActions": []
          }
        ],
        "roleName": "Reader Support Tickets",
        "roleType": "CustomRole",
        "type": "Microsoft.Authorization/roleDefinitions"
      }
    ]
    ```
    
    Vous pouvez également voir le rôle personnalisé dans le portail Azure.

    ![capture d’écran d’un rôle personnalisé importé dans le portail Azure](./media/tutorial-custom-role-cli/custom-role-reader-support-tickets.png)

## <a name="update-a-custom-role"></a>Mettre à jour un rôle personnalisé

Pour mettre à jour le rôle personnalisé, mettez à jour le fichier JSON puis mettez à jour le rôle personnalisé.

1. Ouvrez le fichier ReaderSupportRole.json.

1. Dans `Actions`, ajoutez l’opération pour créer et gérer les déploiements de groupes de ressources `"Microsoft.Resources/deployments/*"`. Veillez à inclure une virgule après l’opération précédente.

    Votre fichier JSON mis à jour doit ressembler à ceci :

    ```json
    {
      "Name": "Reader Support Tickets",
      "IsCustom": true,
      "Description": "View everything in the subscription and also open support tickets.",
      "Actions": [
        "*/read",
        "Microsoft.Support/*",
        "Microsoft.Resources/deployments/*"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ]
    }
    ```
        
1. Pour mettre à jour le rôle personnalisé, utilisez la commande [az role definition update](/cli/azure/role/definition#az-role-definition-update) et spécifiez le fichier JSON mis à jour.

    ```azurecli
    az role definition update --role-definition "~/CustomRoles/ReaderSupportRole.json"
    ```

    ```Output
    {
      "additionalProperties": {},
      "assignableScopes": [
        "/subscriptions/00000000-0000-0000-0000-000000000000"
      ],
      "description": "View everything in the subscription and also open support tickets.",
      "id": "/subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleDefinitions/22222222-2222-2222-2222-222222222222",
      "name": "22222222-2222-2222-2222-222222222222",
      "permissions": [
        {
          "actions": [
            "*/read",
            "Microsoft.Support/*",
            "Microsoft.Resources/deployments/*"
          ],
          "additionalProperties": {},
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": "Reader Support Tickets",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```
    
## <a name="delete-a-custom-role"></a>Supprimer un rôle personnalisé

- Utilisez la commande [az role definition delete](/cli/azure/role/definition#az-role-definition-delete) et spécifiez le nom de rôle ou l’ID de rôle pour supprimer le rôle personnalisé.

    ```azurecli
    az role definition delete --name "Reader Support Tickets"
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer ou mettre à jour des rôles personnalisés Azure à l’aide d’Azure CLI](custom-roles-cli.md)
