---
title: Octroyer l’accès pour créer des abonnements Azure Enterprise
description: Découvrez comment autoriser un utilisateur ou à un principal de service à créer des abonnements Azure Enterprise par programmation.
author: bandersmsft
ms.service: cost-management-billing
ms.subservice: billing
ms.reviewer: amberb
ms.topic: conceptual
ms.date: 08/26/2020
ms.author: banders
ms.openlocfilehash: c13670df26e5d0f7774b5a2aac81f656de94c960
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844708"
---
# <a name="grant-access-to-create-azure-enterprise-subscriptions-preview"></a>Accorder l’accès pour créer des abonnements Azure Enterprise (préversion)

En tant que client Azure en [Contrat Entreprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/), vous pouvez autoriser un autre utilisateur ou principal de service à créer des abonnements facturés sur votre compte. Dans cet article, vous allez apprendre à utiliser le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md) pour partager la capacité de créer des abonnements et à effectuer un audit des créations d’abonnements. Vous devez disposer du rôle Propriétaire pour le compte que vous souhaitez partager.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="grant-access"></a>Accorder l'accès

Pour [créer des abonnements sous un compte d’inscription](programmatically-create-subscription-enterprise-agreement.md), les utilisateurs doivent avoir le [rôle de propriétaire Azure RBAC](../../role-based-access-control/built-in-roles.md#owner) sur ce compte. Vous pouvez accorder à un utilisateur ou un groupe d’utilisateurs le rôle de propriétaire Azure RBAC sur un compte d’inscription en procédant comme suit :

1. Obtenir l’ID d’objet du compte d’inscription auquel vous souhaitez accorder l’accès

    Pour accorder à d’autres utilisateurs le rôle de propriétaire Azure RBAC sur un compte d’inscription, vous devez être le propriétaire du compte ou un propriétaire Azure RBAC du compte.

    # <a name="rest"></a>[REST](#tab/rest)

    Demande pour obtenir la liste de tous les comptes d’inscription auxquels vous avez accès :

    ```json
    GET https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts?api-version=2018-03-01-preview
    ```

    Azure retourne la liste de tous les comptes d’inscription auxquels vous avez accès :

    ```json
    {
      "value": [
        {
          "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "type": "Microsoft.Billing/enrollmentAccounts",
          "properties": {
            "principalName": "SignUpEngineering@contoso.com"
          }
        },
        {
          "id": "/providers/Microsoft.Billing/enrollmentAccounts/4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
          "type": "Microsoft.Billing/enrollmentAccounts",
          "properties": {
            "principalName": "BillingPlatformTeam@contoso.com"
          }
        }
      ]
    }
    ```

    Utilisez la propriété `principalName` pour identifier le compte auquel vous souhaitez accorder l’accès Azure RBAC. Copiez l’élément `name` de ce compte. Par exemple, si vous souhaitez accorder l’accès propriétaire Azure RBAC au compte d’inscription SignUpEngineering@contoso.com, vous devez copier ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Il s’agit de l’ID d’objet du compte d’inscription. Collez cette valeur quelque part, de façon à pouvoir l’utiliser à l’étape suivante en tant que `enrollmentAccountObjectId`.

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

    Utilisez l’applet de commande [Get-AzEnrollmentAccount](/powershell/module/az.billing/get-azenrollmentaccount) pour lister tous les comptes d’inscription auxquels vous avez accès. Sélectionnez **Essayer** pour ouvrir [Azure Cloud Shell](https://shell.azure.com/). Pour coller le code, cliquez avec le bouton droit sur la fenêtre de l’interpréteur de commandes, puis sélectionnez **Coller**.

    ```azurepowershell-interactive
    Get-AzEnrollmentAccount
    ```

    Azure renvoie la liste de tous les comptes d’inscription auxquels vous avez accès :

    ```azurepowershell
    ObjectId                               | PrincipalName
    747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | SignUpEngineering@contoso.com
    4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx   | BillingPlatformTeam@contoso.com
    ```

    Utilisez la propriété `principalName` pour identifier le compte auquel vous souhaitez accorder l’accès Azure RBAC. Copiez l’élément `ObjectId` de ce compte. Par exemple, si vous souhaitez accorder l’accès propriétaire Azure RBAC au compte d’inscription SignUpEngineering@contoso.com, vous devez copier ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Collez cet ID d’objet quelque part, de façon à pouvoir l’utiliser à l’étape suivante en tant que `enrollmentAccountObjectId`.

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

    Utilisez la commande [az billing enrollment-account list](/cli/azure/billing) pour lister tous les comptes d’inscription auxquels vous avez accès. Sélectionnez **Essayer** pour ouvrir [Azure Cloud Shell](https://shell.azure.com/). Pour coller le code, cliquez avec le bouton droit sur la fenêtre de l’interpréteur de commandes, puis sélectionnez **Coller**.

    ```azurecli-interactive
    az billing enrollment-account list
    ```

    Azure renvoie la liste de tous les comptes d’inscription auxquels vous avez accès :

    ```json
    [
      {
        "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "name": "747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "principalName": "SignUpEngineering@contoso.com",
        "type": "Microsoft.Billing/enrollmentAccounts",
      },
      {
        "id": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "name": "4cd2fcf6-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "principalName": "BillingPlatformTeam@contoso.com",
        "type": "Microsoft.Billing/enrollmentAccounts",
      }
    ]
    ```

    ---

    Utilisez la propriété `principalName` pour identifier le compte auquel vous souhaitez accorder l’accès Azure RBAC. Copiez l’élément `name` de ce compte. Par exemple, si vous souhaitez accorder l’accès propriétaire Azure RBAC au compte d’inscription SignUpEngineering@contoso.com, vous devez copier ```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```. Il s’agit de l’ID d’objet du compte d’inscription. Collez cette valeur quelque part, de façon à pouvoir l’utiliser à l’étape suivante en tant que `enrollmentAccountObjectId`.

1. <a id="userObjectId"></a>Obtenir l’ID d’objet de l’utilisateur ou du groupe auquel vous souhaitez attribuer le rôle de propriétaire Azure RBAC

    1. Dans le portail Azure, recherchez **Azure Active Directory**.
    1. Si vous voulez accorder l’accès à un utilisateur, sélectionnez **Utilisateurs** dans le menu de gauche. Pour donner accès à un groupe, sélectionnez **Groupes**.
    1. Sélectionnez l’utilisateur ou le groupe auquel vous souhaitez attribuer le rôle de propriétaire Azure RBAC.
    1. Si vous avez sélectionné un utilisateur, vous trouverez l’ID d’objet dans la page de profil. Si vous avez sélectionné un groupe, l’ID d’objet sera dans la page Vue d’ensemble. Copiez la valeur de **ObjectID** en sélectionnant l’icône à droite de la zone de texte. Collez-la quelque part de façon à pouvoir l’utiliser à l’étape suivante en tant que `userObjectId`.

1. Accorder à l’utilisateur ou au groupe le rôle de propriétaire Azure RBAC sur le compte d’inscription

    En utilisant les valeurs que vous avez collectées lors des deux premières étapes, accordez à l’utilisateur ou au groupe le rôle de propriétaire Azure RBAC sur le compte d’inscription.

    # <a name="rest"></a>[REST](#tab/rest-2)

    Exécutez la commande suivante, en remplaçant ```<enrollmentAccountObjectId>``` avec la valeur `name` que vous avez copiée lors de la première étape (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Remplacez ```<userObjectId>``` par l’ID d’objet que vous avez copié lors de la deuxième étape.

    ```json
    PUT  https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentGuid>?api-version=2015-07-01

    {
      "properties": {
        "roleDefinitionId": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
        "principalId": "<userObjectId>"
      }
    }
    ```

    Quand le rôle de propriétaire est correctement attribué au niveau de l’étendue du compte d’inscription, Azure retourne les informations de l’attribution de rôle :

    ```json
    {
      "properties": {
        "roleDefinitionId": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
        "principalId": "<userObjectId>",
        "scope": "/providers/Microsoft.Billing/enrollmentAccounts/747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "createdOn": "2018-03-05T08:36:26.4014813Z",
        "updatedOn": "2018-03-05T08:36:26.4014813Z",
        "createdBy": "<assignerObjectId>",
        "updatedBy": "<assignerObjectId>"
      },
      "id": "/providers/Microsoft.Billing/enrollmentAccounts/providers/Microsoft.Authorization/roleDefinitions/<ownerRoleDefinitionId>",
      "type": "Microsoft.Authorization/roleAssignments",
      "name": "<roleAssignmentGuid>"
    }
    ```

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell-2)

    Exécutez la commande [New-AzRoleAssignment](../../role-based-access-control/role-assignments-powershell.md) suivante, en remplaçant ```<enrollmentAccountObjectId>``` par la valeur `ObjectId` récupérée lors de la première étape (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Remplacez ```<userObjectId>``` par l’ID d’objet que vous avez récupéré lors de la deuxième étape.

    ```azurepowershell-interactive
    New-AzRoleAssignment -RoleDefinitionName Owner -ObjectId <userObjectId> -Scope /providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>
    ```

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli-2)

    Exécutez la commande [az role assignment create](../../role-based-access-control/role-assignments-cli.md) suivante, en remplaçant ```<enrollmentAccountObjectId>``` par la valeur `name` copiée lors de la première étape (```747ddfe5-xxxx-xxxx-xxxx-xxxxxxxxxxxx```). Remplacez ```<userObjectId>``` par l’ID d’objet que vous avez récupéré lors de la deuxième étape.

    ```azurecli-interactive
    az role assignment create --role Owner --assignee-object-id <userObjectId> --scope /providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountObjectId>
    ```

    Une fois qu’un utilisateur devient propriétaire Azure RBAC pour votre compte d’inscription, il peut [créer des abonnements par programmation](programmatically-create-subscription-enterprise-agreement.md) sous ce dernier. Un abonnement créé par un utilisateur délégué a toujours le propriétaire de compte d’origine comme administrateur de service, mais il a également l’utilisateur délégué comme propriétaire Azure RBAC par défaut.

    ---

## <a name="audit-who-created-subscriptions-using-activity-logs"></a>Auditer qui a créé des abonnements à l’aide des journaux d’activité

Pour effectuer le suivi des abonnements créés par le biais de cette API, utilisez [l’API des journaux d’activité de locataire](/rest/api/monitor/tenantactivitylogs). Il est impossible d’utiliser le portail Azure, l’interface CLI ou PowerShell pour effectuer le suivi de la création d’abonnement.

1. En tant qu’administrateur locataire du locataire Azure AD, [élevez l’accès](../../role-based-access-control/elevate-access-global-admin.md), puis affectez un rôle de lecteur à l’utilisateur d’audit sur l’étendue `/providers/microsoft.insights/eventtypes/management`. Cet accès est disponible dans le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader), le rôle [Contributeur d’analyse](../../role-based-access-control/built-in-roles.md#monitoring-contributor) ou un [rôle personnalisé](../../role-based-access-control/custom-roles.md).
1. En tant qu’utilisateur d’audit, appelez [l’API des journaux d’activité de locataire](/rest/api/monitor/tenantactivitylogs) pour voir les activités de création d’abonnement. Exemple :

    ```
    GET "/providers/Microsoft.Insights/eventtypes/management/values?api-version=2015-04-01&$filter=eventTimestamp ge '{greaterThanTimeStamp}' and eventTimestamp le '{lessThanTimestamp}' and eventChannels eq 'Operation' and resourceProvider eq 'Microsoft.Subscription'"
    ```

Pour appeler facilement cette API à partir de la ligne de commande, essayez [ARMClient](https://github.com/projectkudu/ARMClient).

## <a name="next-steps"></a>Étapes suivantes

* Maintenant que l’utilisateur ou le principal de service a l’autorisation de créer un abonnement, vous pouvez utiliser cette identité sur [créer des abonnements Azure Enterprise par programmation](programmatically-create-subscription-enterprise-agreement.md).
* Pour obtenir un exemple de création d’abonnements à l’aide de .NET, consultez [l’exemple de code sur GitHub](https://github.com/Azure-Samples/create-azure-subscription-dotnet-core).
* Pour en savoir plus sur Azure Resource Manager et ses API, consultez [Vue d’ensemble d’Azure Resource Manager](../../azure-resource-manager/management/overview.md).
* Pour plus d’informations sur la gestion d’un grand nombre d’abonnements à l’aide de groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../governance/management-groups/overview.md).
* Pour obtenir une aide complète sur les bonnes pratiques de gouvernance d’abonnements dans les grandes entreprises, consultez [Structure d’entreprise Azure : gouvernance normative de l’abonnement](/azure/architecture/cloud-adoption-guide/subscription-governance)