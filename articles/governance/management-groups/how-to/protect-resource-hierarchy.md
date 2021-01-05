---
title: Guide pratique pour protéger votre hiérarchie de ressources – Gouvernance Azure
description: Découvrez comment protéger votre hiérarchie de ressources avec des paramètres de hiérarchie qui incluent la définition du groupe d’administration par défaut.
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: cfaf7a732b3233536006009613cecd5aa1649dee
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91951891"
---
# <a name="how-to-protect-your-resource-hierarchy"></a>Comment protéger votre hiérarchie de ressources

Votre hiérarchie de ressources est un ensemble constitué de vos ressources, groupes de ressources, abonnements, groupes d’administration et autre locataire. Les paramètres au niveau du groupe d’administration racine, tels que les rôles Azure personnalisés ou les affectations de stratégie Azure Policy, peuvent avoir un impact sur chaque ressource de votre hiérarchie de ressources. Il est important de protéger la hiérarchie de ressources contre les modifications susceptibles d’avoir des répercussions négatives sur toutes les ressources.

Les groupes d’administration ont désormais des paramètres de hiérarchie qui permettent à l’administrateur du locataire de contrôler ces comportements. Cet article décrit chacun des paramètres de hiérarchie disponibles et explique comment les définir.

## <a name="azure-rbac-permissions-for-hierarchy-settings"></a>Autorisations Azure RBAC pour les paramètres de hiérarchie

La configuration des paramètres de hiérarchie exige l’utilisation des deux opérations de fournisseur de ressources suivantes au niveau du groupe d’administration racine :

- `Microsoft.Management/managementgroups/settings/write`
- `Microsoft.Management/managementgroups/settings/read`

Ces opérations permettent uniquement à un utilisateur de lire et mettre à jour les paramètres de hiérarchie. Elles ne fournissent aucun autre accès à la hiérarchie du groupe d’administration ou aux ressources de la hiérarchie. Ces deux opérations sont disponibles dans le rôle intégré Azure **Administrateur des paramètres de hiérarchie**.

## <a name="setting---default-management-group"></a>Paramètre – Groupe d’administration par défaut

Par défaut, un nouvel abonnement ajouté au sein d’un locataire est ajouté en tant que membre du groupe d’administration racine. Si les affectations de stratégie, le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et d’autres concepts de gouvernance sont affectés au groupe d’administration racine, ils ont un effet immédiat sur ces nouveaux abonnements. C’est pour cette raison que de nombreuses organisations n’appliquent pas ces concepts au niveau du groupe d’administration racine, même s’il s’agit de leur emplacement d’affectation idéal. Dans d’autres cas, un ensemble de contrôles plus restrictif est souhaité pour les nouveaux abonnements, mais il n’est pas recommandé de les affecter à tous les abonnements. Ce paramètre prend en charge les deux cas d’usage.

En permettant la définition du groupe d’administration par défaut des nouveaux abonnements, les concepts de gouvernance à l’échelle de l’organisation peuvent être appliqués au niveau du groupe d’administration racine, et il est possible de définir un groupe d’administration distinct avec des affectations de stratégie ou des attributions de rôle Azure plus adaptées à un nouvel abonnement.

### <a name="set-default-management-group-in-portal"></a>Définir un groupe d'administration par défaut dans le portail

Pour configurer ce paramètre dans le portail Azure, procédez comme suit :

1. Utilisez la barre de recherche pour rechercher et sélectionner « Groupes d’administration ».

1. Dans le groupe d’administration racine, sélectionnez **Détails** en regard du nom du groupe d’administration.

1. Dans **Paramètres**, sélectionnez **Paramètres de hiérarchie**.

1. Sélectionnez le bouton **Modifier le groupe d'administration par défaut**.

   > [!NOTE]
   > Si le bouton **Modifier le groupe d’administration par défaut** est désactivé, soit le groupe d’administration affiché n’est pas le groupe d’administration racine, soit votre principal de sécurité ne dispose pas des autorisations nécessaires pour modifier les paramètres de la hiérarchie.

1. Sélectionnez un groupe d’administration dans votre hiérarchie et utilisez le bouton **Sélectionner**.

### <a name="set-default-management-group-with-rest-api"></a>Définir un groupe d’administration par défaut avec l’API REST

Pour configurer ce paramètre avec l’API REST, le point de terminaison [Paramètres de hiérarchie ](/rest/api/resources/hierarchysettings) est appelé. Pour ce faire, utilisez le format de corps et d’URI d’API REST suivant. Remplacez `{rootMgID}` par l’ID de votre groupe d’administration racine et `{defaultGroupID}` par l’ID du groupe d’administration qui doit devenir le groupe d’administration par défaut :

- URI de l’API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{rootMgID}/settings/default?api-version=2020-02-01
  ```

- Corps de la requête

  ```json
  {
      "properties": {
          "defaultManagementGroup": "/providers/Microsoft.Management/managementGroups/{defaultGroupID}"
      }
  }
  ```

Pour faire à nouveau du groupe d’administration par défaut le groupe d’administration racine, utilisez le même point de terminaison et attribuez à **defaultManagementGroup** la valeur `/providers/Microsoft.Management/managementGroups/{rootMgID}`.

## <a name="setting---require-authorization"></a>Paramètre – Exiger une autorisation

Par défaut, n’importe quel utilisateur peut créer de nouveaux groupes d’administration au sein d’un locataire. Les administrateurs d’un locataire peuvent juger souhaitable de n’attribuer ces autorisations qu’à certains utilisateurs de façon à préserver la cohérence et la conformité dans la hiérarchie du groupe d’administration. S’il y est autorisé, un utilisateur a besoin de l’opération `Microsoft.Management/managementGroups/write` sur le groupe d’administration racine pour créer de nouveaux groupes d’administration enfants.

### <a name="set-require-authorization-in-portal"></a>Définir l’exigence d’une autorisation dans le portail

Pour configurer ce paramètre dans le portail Azure, procédez comme suit :

1. Utilisez la barre de recherche pour rechercher et sélectionner « Groupes d’administration ».

1. Dans le groupe d’administration racine, sélectionnez **Détails** en regard du nom du groupe d’administration.

1. Dans **Paramètres**, sélectionnez **Paramètres de hiérarchie**.

1. Activez ou désactivez **Demander des autorisations pour créer des groupes d’administration.** option sur on.

   > [!NOTE]
   > Si le bouton d’activation ou de désactivation **Demander des autorisations pour créer des groupes d’administration.** est désactivé, soit le groupe d’administration affiché n’est pas le groupe d’administration racine, soit votre principal de sécurité ne dispose pas des autorisations nécessaires pour modifier les paramètres de la hiérarchie.

### <a name="set-require-authorization-with-rest-api"></a>Définir Demander une autorisation avec l’API REST

Pour configurer ce paramètre avec l’API REST, le point de terminaison [Paramètres de hiérarchie ](/rest/api/resources/hierarchysettings) est appelé. Pour ce faire, utilisez le format de corps et d’URI d’API REST suivant. S’agissant d’une valeur _booléenne_, indiquez la valeur **true** ou **false**. La valeur **true** active cette méthode de protection de la hiérarchie de votre groupe d’administration :

- URI de l’API REST

  ```http
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{rootMgID}/settings/default?api-version=2020-02-01
  ```

- Corps de la requête

  ```json
  {
      "properties": {
          "requireAuthorizationForGroupCreation": true
      }
  }
  ```

Pour désactiver le paramètre, utilisez le même point de terminaison et attribuez à **requireAuthorizationForGroupCreation** la valeur **false**.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les groupes d’administration, consultez :

- [Créer des groupes d’administration pour organiser les ressources Azure](../create-management-group-portal.md)
- [Guide pratique pour modifier, supprimer ou gérer vos groupes d’administration](../manage.md)