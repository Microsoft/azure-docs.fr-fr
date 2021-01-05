---
title: Restreindre l’accès des utilisateurs aux opérations de données uniquement avec Azure Cosmos DB
description: Découvrez comment restreindre l’accès aux opérations de données uniquement avec Azure Cosmos DB
author: voellm
ms.service: cosmos-db
ms.topic: how-to
ms.date: 12/9/2019
ms.author: tvoellm
ms.openlocfilehash: 61e8ee2cbcd1252704b2d1e220fcfaaff17c2f28
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93085987"
---
# <a name="restrict-user-access-to-data-operations-in-azure-cosmos-db"></a>Restreindre l’accès des utilisateurs aux opérations de données dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Dans Azure Cosmos DB, il existe deux façons d’authentifier vos interactions auprès du service de base de données :

- En utilisant votre identité Azure Active Directory lors de l’interaction avec le portail Azure
- En utilisant des [jetons de ressources](secure-access-to-data.md#resource-tokens) ou des [clés](database-security.md#primary-keys) Azure Cosmos DB lors de l’émission d’appels à partir d’API et de kits SDK

Chaque méthode d’authentification donne accès à différents ensembles d’opérations, qui se chevauchent pour certaines :

:::image type="content" source="./media/how-to-restrict-user-data/operations.png" alt-text="Répartition des opérations par type d’authentification" border="false":::

Dans certains scénarios, vous souhaiterez peut-être limiter le champ d’actions de certains utilisateurs aux opérations de données (c’est-à-dire les requêtes et les demandes CRUD). C’est généralement le cas des développeurs qui n’ont pas besoin de créer ou de supprimer de ressources ou de changer le débit provisionné des conteneurs sur lesquels ils travaillent.

Vous pouvez restreindre l’accès en appliquant les étapes suivantes :
1. Création d’un rôle Azure Active Directory personnalisé pour les utilisateurs dont vous souhaitez restreindre l’accès. Le rôle Active Directory personnalisé doit avoir un niveau d’accès affiné aux opérations qui utilisent des [actions précises](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb) d’Azure Cosmos DB.
1. Interdiction de l’exécution d’opérations non liées à des données avec des clés. Pour ce faire, vous pouvez limiter ces opérations aux appels d’Azure Resource Manager.

Les sections suivantes de cet article expliquent comment effectuer ces étapes.

> [!NOTE]
> Pour exécuter les commandes indiquées dans les sections suivantes, vous devez installer le module Azure PowerShell version 3.0.0 ou ultérieure ainsi que le [rôle Propriétaire Azure](../role-based-access-control/built-in-roles.md#owner) sur l’abonnement que vous essayez de modifier.

Dans les scripts PowerShell des sections suivantes, remplacez les espaces réservés ci-après par des valeurs propres à votre environnement :
- `$MySubscriptionId` : ID d’abonnement qui contient le compte Azure Cosmos dans lequel vous souhaitez limiter les autorisations. Par exemple : `e5c8766a-eeb0-40e8-af56-0eb142ebf78e`.
- `$MyResourceGroupName` : groupe de ressources contenant le compte Azure Cosmos. Par exemple : `myresourcegroup`.
- `$MyAzureCosmosDBAccountName` : nom de votre compte Azure Cosmos. Par exemple : `mycosmosdbsaccount`.
- `$MyUserName` : identifiant (username@domain) de l’utilisateur pour lequel vous souhaitez limiter l’accès. Par exemple : `cosmosdbuser@contoso.com`.

## <a name="select-your-azure-subscription"></a>Sélectionner votre abonnement Azure

Pour exécuter les commandes Azure PowerShell, vous devez vous connecter et sélectionner l’abonnement :

```azurepowershell
Login-AzAccount
Select-AzSubscription $MySubscriptionId
```

## <a name="create-the-custom-azure-active-directory-role"></a>Créer le rôle Azure Active Directory personnalisé

Le script suivant crée une attribution de rôle Azure Active Directory avec un accès « Clé uniquement » pour les comptes Azure Cosmos. Le rôle est basé sur les [rôles personnalisés Azure](../role-based-access-control/custom-roles.md) et les [actions précises pour Azure Cosmos DB](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb). Ces rôles et actions font partie de l’espace de noms Azure Active Directory `Microsoft.DocumentDB`.

1. Tout d’abord, créez un document JSON nommé `AzureCosmosKeyOnlyAccess.json` avec le contenu suivant :

    ```
    {
        "Name": "Azure Cosmos DB Key Only Access Custom Role",
        "Id": "00000000-0000-0000-0000-0000000000",
        "IsCustom": true,
        "Description": "This role restricts the user to read the account keys only.",
        "Actions":
        [
            "Microsoft.DocumentDB/databaseAccounts/listKeys/action"
        ],
        "NotActions": [],
        "DataActions": [],
        "NotDataActions": [],
        "AssignableScopes":
        [
            "/subscriptions/$MySubscriptionId"
        ]
    }
    ```

1. Exécutez les commandes suivantes pour créer l’attribution de rôle et l’attribuer à l’utilisateur :

    ```azurepowershell
    New-AzRoleDefinition -InputFile "AzureCosmosKeyOnlyAccess.json"
    New-AzRoleAssignment -SignInName $MyUserName -RoleDefinitionName "Azure Cosmos DB Key Only Access Custom Role" -ResourceGroupName $MyResourceGroupName -ResourceName $MyAzureCosmosDBAccountName -ResourceType "Microsoft.DocumentDb/databaseAccounts"
    ```

## <a name="disallow-the-execution-of-non-data-operations"></a>Interdire l’exécution d’opérations non liées à des données

Les commandes suivantes suppriment la possibilité d’utiliser des clés pour :
- créer, modifier ou supprimer des ressources
- mettre à jour les paramètres de conteneur (notamment les stratégies d’indexation, le débit, etc.).

```azurepowershell
$cdba = Get-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" -ApiVersion "2015-04-08" -ResourceGroupName $MyResourceGroupName -ResourceName $MyAzureCosmosDBAccountName
$cdba.Properties.disableKeyBasedMetadataWriteAccess="True"
$cdba | Set-AzResource -Force
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez le [contrôle d’accès en fonction du rôle de Cosmos DB](role-based-access-control.md).
- Obtenez une vue d’ensemble de l’[accès sécurisé aux données dans Cosmos DB](secure-access-to-data.md).
