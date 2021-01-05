---
title: Résoudre les problèmes liés à Azure RBAC
description: Résoudre des problèmes liés au contrôle d’accès en fonction du rôle Azure (Azure RBAC).
services: azure-portal
documentationcenter: na
author: rolyon
manager: mtillman
ms.assetid: df42cca2-02d6-4f3c-9d56-260e1eb7dc44
ms.service: role-based-access-control
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 11/10/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.custom: seohack1, devx-track-azurecli
ms.openlocfilehash: e30af9522d7c8fa81c4d93e11d252aefc4426586
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184261"
---
# <a name="troubleshoot-azure-rbac"></a>Résoudre les problèmes liés à Azure RBAC

Cet article répond à certaines questions fréquentes sur le contrôle d’accès en fonction du rôle Azure (Azure RBAC), afin que vous sachiez à quoi vous attendre lorsque vous utilisez les rôles et que vous puissiez résoudre les problèmes d’accès.

## <a name="azure-role-assignments-limit"></a>Limite des attributions de rôle Azure

Azure prend en charge jusqu’à **2 000** attributions de rôle par abonnement. Cette limite comprend les attributions de rôles au niveau de l’abonnement, du groupe de ressources et des étendues de ressources. Si vous obtenez le message d’erreur « Plus aucune attribution de rôle ne peut être créée (code : RoleAssignmentLimitExceeded) » lorsque vous tentez d’attribuer un rôle, essayez de réduire le nombre d’attributions de rôle dans l’abonnement.

> [!NOTE]
> Cette limite d’attribution de rôle de **2 000** par abonnement est fixe et ne peut pas être augmentée.

Si vous vous approchez de cette limite, voici quelques façons de réduire le nombre d’attributions de rôle :

- Ajoutez des utilisateurs aux groupes et attribuez des rôles aux groupes à la place. 
- Combinez plusieurs rôles intégrés avec un rôle personnalisé. 
- Associez les attributions de rôle communes à une étendue plus élevée, comme l’abonnement ou le groupe d’administration.
- Si vous avez Azure AD Premium P2, rendez les attributions de rôle éligibles dans [Azure AD Privileged Identity Management](../active-directory/privileged-identity-management/pim-configure.md) au lieu d’être attribuées de manière permanente. 
- Ajoutez un abonnement supplémentaire. 

Pour obtenir le nombre d’attributions de rôle, vous pouvez afficher le [graphique sur la page contrôle d’accès (IAM)](role-assignments-list-portal.md#list-number-of-role-assignments) dans le Portail Azure. Vous pouvez également utiliser les commandes Azure PowerShell suivantes :

```azurepowershell
$scope = "/subscriptions/<subscriptionId>"
$ras = Get-AzRoleAssignment -Scope $scope | Where-Object {$_.scope.StartsWith($scope)}
$ras.Count
```

## <a name="problems-with-azure-role-assignments"></a>Problèmes liés aux attributions de rôle Azure

- Si vous ne pouvez pas ajouter d’attribution de rôle dans le portail Azure sur **Contrôle d’accès (IAM)** car l’option **Ajouter** > **Ajouter une attribution de rôle** est désactivée, ou parce que vous obtenez l’erreur d’autorisations « Le client avec l’ID d’objet n’est pas autorisé à effectuer l’action », vérifiez que vous êtes actuellement connecté avec un utilisateur disposant d’un rôle qui a l’autorisation `Microsoft.Authorization/roleAssignments/write`, comme [Propriétaire](built-in-roles.md#owner) ou [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) dans l’étendue sur laquelle vous essayez d’attribuer le rôle.
- Si vous utilisez un principal de service pour attribuer des rôles, il se peut que le message d’erreur « Privilèges insuffisants pour effectuer l’opération » s’affiche. Supposons, par exemple, que vous avez un principal de service auquel le rôle Propriétaire a été attribué et que vous tentez de créer l’attribution de rôle suivante en tant que principal de service à l’aide d’Azure CLI :

    ```azurecli
    az login --service-principal --username "SPNid" --password "password" --tenant "tenantid"
    az role assignment create --assignee "userupn" --role "Contributor"  --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}"
    ```

    Si le message d’erreur « Privilèges insuffisants pour terminer l’opération » s’affiche, cela est probablement dû au fait qu’Azure CLI tente de rechercher l’identité de la personne responsable dans Azure AD, et que le principal du service ne peut pas lire Azure AD par défaut.

    Il existe deux façons de résoudre cette erreur. La première consiste à attribuer au principal de service le rôle [Lecteurs de répertoire](../active-directory/roles/permissions-reference.md#directory-readers) afin qu’il puisse lire les données dans l’annuaire.

    La deuxième consiste à créer l’attribution de rôle à l’aide du paramètre `--assignee-object-id` au lieu de `--assignee`. En utilisant `--assignee-object-id`, Azure CLI ignorera la recherche Azure AD. Vous devez obtenir l’ID d’objet de l’utilisateur, du groupe ou de l’application auquel vous souhaitez attribuer le rôle. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](role-assignments-cli.md#add-role-assignment-for-a-new-service-principal-at-a-resource-group-scope).

    ```azurecli
    az role assignment create --assignee-object-id 11111111-1111-1111-1111-111111111111  --role "Contributor" --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}"
    ```
- Si vous tentez de supprimer la dernière attribution de rôle Propriétaire pour un abonnement, l’erreur « Impossible de supprimer le dernier RBAC admin assignment » risque de s’afficher. La suppression de la dernière attribution de rôle Propriétaire pour un abonnement n’est pas prise en charge afin d’éviter que l’abonnement ne devienne orphelin. Si vous voulez annuler votre abonnement, consultez [Annulation de votre abonnement Azure](../cost-management-billing/manage/cancel-azure-subscription.md).

## <a name="problems-with-custom-roles"></a>Problèmes liés aux rôles personnalisés

- Si vous avez besoin de connaître les étapes permettant de créer un rôle personnalisé, consultez les tutoriels de rôle personnalisé en utilisant le [Portail Azure](custom-roles-portal.md) (actuellement en préversion), [Azure PowerShell](tutorial-custom-role-powershell.md) ou [Azure CLI](tutorial-custom-role-cli.md).
- Si vous ne parvenez pas à mettre à jour un rôle personnalisé existant, vérifiez que vous êtes actuellement connecté avec un utilisateur disposant d’un rôle qui a l’autorisation `Microsoft.Authorization/roleDefinition/write`, comme [Propriétaire](built-in-roles.md#owner) ou [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator).
- Si vous ne pouvez pas supprimer un rôle personnalisé et que vous obtenez le message d’erreur « Certaines attributions de rôle existantes font référence au rôle (code : RoleDefinitionHasAssignments) », cela signifie que des attributions de rôle utilisent toujours le rôle personnalisé. Supprimez ces attributions de rôle et réessayez de supprimer ce rôle.
- Si vous obtenez le message d’erreur « La limite de définition de rôle a été dépassée. Plus aucune définition de rôle ne peut être créée (code : RoleDefinitionLimitExceeded) » quand vous essayez de créer un rôle personnalisé, supprimez tous les rôles personnalisés qui ne sont pas utilisés. Azure prend en charge jusqu’à **5 000** rôles personnalisés dans un répertoire. (Pour Azure Allemagne et Azure Chine 21Vianet, la limite est de 2 000 rôles personnalisés.)
- Si vous obtenez une erreur similaire au message « Le client a l’autorisation d’effectuer l’action "Microsoft.Authorization/roleDefinitions/write" sur l’étendue "/subscriptions/{subscriptionid}". Cependant, l’abonnement lié est introuvable » quand vous essayez de mettre à jour un rôle personnalisé, vérifiez si une ou plusieurs [étendues attribuables](role-definitions.md#assignablescopes) ont été supprimées du répertoire. Si l’étendue a été supprimée, créez un ticket de support, car il n’existe aucune solution en libre-service disponible pour l’instant.

## <a name="custom-roles-and-management-groups"></a>Rôles personnalisés et groupes d’administration

- Vous ne pouvez définir qu’un seul groupe d’administration dans `AssignableScopes` d’un rôle personnalisé. L’ajout d’un groupe d’administration à `AssignableScopes` est actuellement en préversion.
- Les rôles personnalisés avec `DataActions` ne peuvent pas être attribués dans l’étendue du groupe d’administration.
- Azure Resource Manager ne valide pas le groupe d’administration existant dans l’étendue attribuable de la définition de rôle.
- Pour plus d’informations sur les rôles personnalisés et les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../governance/management-groups/overview.md#azure-custom-role-definition-and-assignment).

## <a name="transferring-a-subscription-to-a-different-directory"></a>Transfert d’un abonnement vers un autre répertoire

- Pour connaître la procédure à suivre pour transférer un abonnement vers une autre instance Azure AD, consultez [Transférer un abonnement Azure vers une autre instance Azure AD Directory](transfer-subscription.md).
- Si vous transférez un abonnement vers un autre répertoire Azure AD, toutes les attributions de rôle définies sont **définitivement** supprimées du répertoire Azure AD source et ne sont pas migrées sur le répertoire Azure AD cible. Vous devez recréer vos attributions de rôle dans le répertoire cible. Vous devez également recréer manuellement les identités managées pour les ressources Azure. Pour plus d’informations, consultez [Questions fréquentes et problèmes connus en lien avec les identités managées](../active-directory/managed-identities-azure-resources/known-issues.md).
- Si vous êtes administrateur général Azure AD et que vous n’avez pas accès à un abonnement après son transfert entre des répertoires, utilisez l’option **Gestion de l’accès pour les ressources Azure** afin d’[élever votre accès](elevate-access-global-admin.md) temporairement et ainsi accéder à l’abonnement.

## <a name="issues-with-service-admins-or-co-admins"></a>Problèmes liés aux coadministrateurs ou administrateurs de service

- Si vous rencontrez des problèmes avec l’Administrateur de service ou les coadministrateurs, consultez [Ajouter ou modifier les administrateurs d’abonnements Azure](../cost-management-billing/manage/add-change-subscription-administrator.md) et [Rôles Administrateur d’abonnement classique, rôles Azure et rôles Azure AD](rbac-and-directory-admin-roles.md).

## <a name="access-denied-or-permission-errors"></a>Accès refusé ou erreurs d’autorisations

- Si vous obtenez l’erreur d’autorisations « Le client avec l’ID d’objet n’est pas autorisé à effectuer l’action sur l’étendue (code : AuthorizationFailed) » lorsque vous tentez de créer une ressource, vérifiez que vous êtes actuellement connecté avec un utilisateur disposant d’un rôle qui a l’autorisation d’écriture sur la ressource au niveau de l’étendue sélectionnée. Par exemple, pour gérer les machines virtuelles dans un groupe de ressources, vous devez disposer du rôle [Contributeur de machines virtuelles](built-in-roles.md#virtual-machine-contributor) sur le groupe de ressources (ou l’étendue parente). Afin d’obtenir la liste des autorisations pour chaque rôle intégré, consultez [Rôles intégrés Azure](built-in-roles.md).
- Si vous obtenez l’erreur d’autorisations « Vous n’êtes pas autorisé à créer une demande de support » quand vous tentez de créer ou de mettre à jour un ticket de support, vérifiez que vous êtes actuellement connecté avec un utilisateur disposant d’un rôle qui a l’autorisation `Microsoft.Support/supportTickets/write`, comme [Collaborateur de la demande de support](built-in-roles.md#support-request-contributor).

## <a name="move-resources-with-role-assignments"></a>Déplacer des ressources avec des attributions de rôles

Si vous déplacez une ressource à laquelle un rôle Azure est affecté directement (ou est affecté à une ressource enfant de cette ressource), l’attribution de rôle n’est pas déplacée et devient orpheline. Après le déplacement, vous devez recréer l’attribution de rôle. Finalement, l’attribution de rôle orpheline sera automatiquement supprimée, mais il est recommandé de supprimer l’attribution de rôle avant de déplacer la ressource.

Pour plus d'informations sur le déplacement des ressources, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).

## <a name="role-assignments-with-identity-not-found"></a>Attributions de rôle avec identité introuvable

Dans la liste des attributions de rôles pour le Portail Azure, vous pouvez remarquer que le principal de sécurité (utilisateur, groupe, principal de service ou identité gérée) est répertorié comme **Identité introuvable** avec un type **Inconnu**.

![Identité introuvable répertoriée dans les attributions de rôle Azure](./media/troubleshooting/unknown-security-principal.png)

L’identité n’a peut-être pas été trouvée pour deux raisons :

- Vous avez récemment invité un utilisateur lors de la création d’une attribution de rôle
- Vous avez supprimé un principal de sécurité qui avait une attribution de rôle

Si vous avez récemment invité un utilisateur lors de la création d’une attribution de rôle, ce principal de sécurité peut encore être dans le processus de réplication entre les régions. Si c’est le cas, patientez quelques instants, puis actualisez la liste des attributions de rôles.

Toutefois, si ce principal de sécurité n’est pas un utilisateur récemment invité, il peut s’agir d’un principal de sécurité supprimé. Si vous attribuez un rôle à un principal de sécurité, puis que vous supprimez ce principal de sécurité sans d’abord supprimer l’attribution de rôle, le principal de sécurité sera répertorié comme **Identité introuvable** avec un type **Inconnu**.

Si vous répertoriez cette attribution de rôle à l’aide d’Azure PowerShell, vous pourrez voir un `DisplayName` vide et un `ObjectType` défini sur **Inconnu**. Par exemple, [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) retourne une attribution de rôle similaire à ce qui suit :

```
RoleAssignmentId   : /subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
Scope              : /subscriptions/11111111-1111-1111-1111-111111111111
DisplayName        :
SignInName         :
RoleDefinitionName : Storage Blob Data Contributor
RoleDefinitionId   : ba92f5b4-2d11-453d-a403-e96b0029c9fe
ObjectId           : 33333333-3333-3333-3333-333333333333
ObjectType         : Unknown
CanDelegate        : False
```

De même, si vous répertoriez cette attribution de rôle à l’aide d’Azure CLI, `principalName` est vide à l’écran. Par exemple, [az role assignment list](/cli/azure/role/assignment#az-role-assignment-list) retourne une attribution de rôle similaire à ce qui suit :

```
{
    "canDelegate": null,
    "id": "/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222",
    "name": "22222222-2222-2222-2222-222222222222",
    "principalId": "33333333-3333-3333-3333-333333333333",
    "principalName": "",
    "roleDefinitionId": "/subscriptions/11111111-1111-1111-1111-111111111111/providers/Microsoft.Authorization/roleDefinitions/ba92f5b4-2d11-453d-a403-e96b0029c9fe",
    "roleDefinitionName": "Storage Blob Data Contributor",
    "scope": "/subscriptions/11111111-1111-1111-1111-111111111111",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

Il n’est pas un problème de conserver ces attributions de rôle pour lesquelles le principal de sécurité a été supprimé. Si vous le souhaitez, vous pouvez supprimer ces attributions de rôle en utilisant les étapes similaires aux autres attributions de rôle. Pour plus d’informations sur la suppression des attributions de rôles, consultez [Portail Azure](role-assignments-portal.md#remove-a-role-assignment), [Azure PowerShell](role-assignments-powershell.md#remove-a-role-assignment) ou [Azure CLI](role-assignments-cli.md#remove-a-role-assignment)

Dans PowerShell, si vous essayez de supprimer les attributions de rôles à l’aide de l’ID d’objet et du nom de définition de rôle alors que plusieurs attributions de rôle correspondent à vos paramètres, le message d’erreur suivant s’affiche : « Les informations fournies ne correspondent pas à une attribution de rôle ». Voici un exemple du message d’erreur :

```
PS C:\> Remove-AzRoleAssignment -ObjectId 33333333-3333-3333-3333-333333333333 -RoleDefinitionName "Storage Blob Data Contributor"

Remove-AzRoleAssignment : The provided information does not map to a role assignment.
At line:1 char:1
+ Remove-AzRoleAssignment -ObjectId 33333333-3333-3333-3333-333333333333 ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : CloseError: (:) [Remove-AzRoleAssignment], KeyNotFoundException
+ FullyQualifiedErrorId : Microsoft.Azure.Commands.Resources.RemoveAzureRoleAssignmentCommand
```

Si ce message d’erreur s’affiche, prenez soin de spécifier également les paramètres `-Scope` ou `-ResourceGroupName`.

```
PS C:\> Remove-AzRoleAssignment -ObjectId 33333333-3333-3333-3333-333333333333 -RoleDefinitionName "Storage Blob Data Contributor" - Scope /subscriptions/11111111-1111-1111-1111-111111111111
```

## <a name="role-assignment-changes-are-not-being-detected"></a>Les changements d’attribution de rôle ne sont pas détectés

Azure Resource Manager met parfois en cache des données et des configurations pour améliorer les performances. Lorsque vous ajoutez ou supprimez des attributions de rôle, un délai de 30 minutes maximum peut être nécessaire avant que les modifications soient prises en compte. Si vous utilisez le portail Azure, Azure PowerShell ou Azure CLI, vous pouvez forcer une actualisation de vos modifications d’attribution de rôle en vous déconnectant et en vous reconnectant. Si vous apportez des modifications d’attribution de rôle à l’aide d’appels d’API REST, vous pouvez forcer une actualisation en actualisant votre jeton d’accès.

Si vous ajoutez ou supprimez une attribution de rôle au niveau de l’étendue du groupe d’administration et que le rôle est doté de `DataActions`, l’accès au plan de données peut ne pas être mis à jour pendant plusieurs heures. Cela s’applique uniquement à l’étendue du groupe d’administration et au plan de données.

## <a name="web-app-features-that-require-write-access"></a>Fonctionnalités d’application web qui nécessitent un accès en écriture

Si vous accordez un accès utilisateur en lecture seule à une seule application web, certaines fonctionnalités sont désactivées, ce que vous n’avez peut-être pas prévu. Les fonctionnalités de gestion suivantes exigent un accès en **écriture** à une application web (Collaborateur ou Propriétaire) et ne sont pas disponibles en lecture seule.

* Commandes (comme start, stop, etc.)
* Modification de paramètres tels que la configuration générale, les paramètres de mise à l’échelle, les paramètres de sauvegarde et les paramètres d’analyse
* Accès aux informations d’identification de publication et autres informations secrètes, telles que les paramètres d’application et les chaînes de connexion
* Diffusion de journaux d’activité
* Configuration des journaux de ressources
* Console (invite de commandes)
* Déploiements actifs et récents (pour le déploiement continu Git local)
* Estimation de dépense
* Tests web
* Réseau virtuel (accessible à un lecteur uniquement si un réseau virtuel a été précédemment configuré par un utilisateur avec accès en écriture).

Si vous ne pouvez accéder à aucune de ces vignettes, vous devez obtenir l’accès Collaborateur à l’application web auprès de votre administrateur.

## <a name="web-app-resources-that-require-write-access"></a>Ressources d’application web qui nécessitent un accès en écriture

Les applications web sont compliqués par la présence de quelques ressources différentes qui interagissent. Voici un groupe de ressources classique avec plusieurs sites web :

![Groupe de ressources d'application web](./media/troubleshooting/website-resource-model.png)

En conséquence, si vous accordez à un utilisateur le seul accès à l’application web, la plupart des fonctionnalités du volet Site web dans le portail Azure sont désactivées.

Les éléments suivants requièrent l’accès **en écriture** au **plan App Service** qui correspond à votre site web :  

* Affichage du niveau tarifaire de l’application web (Gratuit ou Standard)  
* Configuration de mise à l'échelle (le nombre d'instances, la taille de la machine virtuelle, les paramètres de mise à l'échelle automatique)  
* Quotas (stockage, bande passante, UC)  

Les éléments suivants requièrent l’accès **en écriture** à l’ensemble du **Groupe de ressources** qui contient le site web :  

* Certificats et liaisons TLS/SSL (les certificats TLS/SSL peuvent être partagés entre différents sites dans le même groupe de ressources et emplacement)  
* Règles d'alerte  
* Paramètres de mise à l'échelle automatique  
* Composants Application Insights  
* Tests web  

## <a name="virtual-machine-features-that-require-write-access"></a>Fonctionnalités de machine virtuelle qui nécessitent un accès en écriture

Comme pour les applications web, certaines fonctionnalités du volet de la machine virtuelle exigent un accès en écriture à la machine virtuelle ou à d’autres ressources du groupe de ressources.

Les machines virtuelles sont associées aux noms de domaine, réseaux virtuels, comptes de stockage et règles d'alerte.

Les éléments suivants requièrent l’accès **en écriture** à la **machine virtuelle** :

* Points de terminaison  
* Adresses IP  
* Disques  
* Extensions  

Les éléments suivants requièrent l’accès **en écriture** à la **machine virtuelle**, ainsi qu’au **Groupe de ressources** (avec le nom de domaine) auxquels ils appartiennent :  

* Groupe à haute disponibilité  
* Jeu d'équilibrage de la charge  
* Règles d'alerte  

Si vous ne pouvez accéder à aucune de ces vignettes, demandez l’accès Collaborateur au groupe de ressources à votre administrateur.

## <a name="azure-functions-and-write-access"></a>Azure Functions et accès en écriture

Certaines fonctionnalités de [Azure Functions](../azure-functions/functions-overview.md) nécessitent un accès en écriture. Par exemple, si un utilisateur est assigné au rôle [Lecteur](built-in-roles.md#reader), il ne peut pas voir les fonctions au sein d’une application de fonction. Le portail affiche **(aucun accès)** .

![Aucun accès aux applications de fonction](./media/troubleshooting/functionapps-noaccess.png)

Un lecteur peut cliquer sur l’onglet **Fonctionnalités de plateforme**, puis cliquez sur **Tous les paramètres** pour afficher certains paramètres liés à une application de fonction (semblable à une application Web), mais il ne peut pas modifier ces paramètres. Pour accéder à ces fonctionnalités, vous aurez besoin du rôle [Contributeur](built-in-roles.md#contributor).

## <a name="next-steps"></a>Étapes suivantes

- [Résolution des problèmes pour les utilisateurs invités](role-assignments-external-users.md#troubleshoot)
- [Ajouter ou supprimer des attributions de rôles Azure avec le portail Azure](role-assignments-portal.md)
- [Afficher les journaux d’activité pour voir les changements RBAC Azure](change-history-report.md)
