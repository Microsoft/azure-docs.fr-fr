---
title: Créer un package d’accès dans la gestion des droits d’utilisation – Azure AD
description: Découvrez comment créer un package d’accès de ressources que vous souhaitez partager dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: b2ce3b362d02e7acb0a11e6d93b8e94ca8e4d04e
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903528"
---
# <a name="create-a-new-access-package-in-azure-ad-entitlement-management"></a>Créer un package d’accès dans la gestion des droits d’utilisation Azure AD

Un package d’accès vous permet d’effectuer une configuration unique des ressources et des stratégies qui gèrent automatiquement l’accès pendant toute la durée de vie du package d’accès. Cet article décrit comment créer un package d’accès.

## <a name="overview"></a>Vue d’ensemble

Tous les packages d’accès doivent être placés dans un conteneur appelé catalogue. Un catalogue définit les ressources que vous pouvez ajouter à votre package d’accès. Si vous ne spécifiez aucun catalogue, votre package d’accès est placé dans le catalogue Général. Actuellement, vous ne pouvez pas déplacer un package d’accès existant vers un autre catalogue.

Si vous êtes un gestionnaire de package Access, vous ne pouvez pas ajouter des ressources que vous possédez à un catalogue. Vous ne pouvez utiliser que les ressources disponibles dans le catalogue. Pour ajouter des ressources à un catalogue, vous pouvez contacter le propriétaire du catalogue.

Tous les packages d’accès doivent avoir au moins une stratégie. Les stratégies spécifient qui peut demander le package d’accès, ainsi que les paramètres d’approbation et d’expiration. Lorsque vous créez un nouveau package d’accès, vous pouvez créer une stratégie initiale pour les utilisateurs présents dans votre répertoire, pour les utilisateurs non présents dans votre répertoire, pour les attributions directes d’administrateur uniquement, ou vous pouvez choisir de créer la stratégie ultérieurement.

![Créer un package d’accès](./media/entitlement-management-access-package-create/access-package-create.png)

Voici les principales étapes à suivre pour créer un package d’accès.

1. Dans Identity Governance, démarrez le processus de création d’un nouveau package d’accès.

1. Sélectionnez le catalogue dans lequel vous souhaitez créer le package d’accès.

1. Ajoutez des ressources du catalogue à votre package d’accès.

1. Assignez des rôles de ressource pour chaque ressource.

1. Spécifiez les utilisateurs qui peuvent demander l’accès.

1. Spécifiez les paramètres d’approbation.

1. Changez les paramètres de cycle de vie.

## <a name="start-new-access-package"></a>Démarrer le nouveau package d’accès

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu gauche, cliquez sur **Packages d’accès**.

1. Cliquez sur **Nouveau package d’accès**.
   
    ![Gestion des droits d’utilisation dans le portail Azure](./media/entitlement-management-shared/access-packages-list.png)

## <a name="basics"></a>Concepts de base

Sur l’onglet **Concepts de base**, vous donnez un nom au package d’accès et spécifiez le catalogue dans lequel vous le créez.

1. Entrez un nom d’affichage et une description pour le package d’accès. Les utilisateurs voient ces informations lorsqu’ils envoient une demande pour le package d’accès.

1. Dans la liste déroulante **Catalogue**, sélectionnez le catalogue dans lequel vous souhaitez créer le package d’accès. Par exemple, vous pouvez avoir un propriétaire du catalogue qui gère toutes les ressources marketing qui peuvent être demandées. Dans ce cas, vous pouvez sélectionner le catalogue marketing.

    Vous verrez seulement les catalogues dans lesquels vous êtes autorisé à créer des packages d’accès. Pour créer un package d’accès dans un catalogue existant, vous devez être au moins un Administrateur général, un Administrateur d’utilisateurs, le propriétaire du catalogue ou le gestionnaire de package d’accès dans ce catalogue.

    ![Package d’accès - Concepts de base](./media/entitlement-management-access-package-create/basics.png)

    Si vous êtes un Administrateur général, un Administrateur d’utilisateurs ou un créateur de catalogue, et si vous souhaitez créer votre package d’accès dans un nouveau catalogue qui ne figure pas dans la liste, cliquez sur **Créer un catalogue**. Entrez le nom du catalogue et sa description, puis cliquez sur **Créer**.

    Le package d’accès que vous créez et toutes les ressources incluses sont ajoutés dans le nouveau catalogue. Vous pouvez également ajouter d’autres propriétaires de catalogue plus tard.

1. Cliquez sur **Suivant**.

## <a name="resource-roles"></a>Rôles de ressources

Sous l’onglet **Rôles de ressources**, sélectionnez les ressources à inclure dans le package d’accès. Les utilisateurs qui demandent et reçoivent le package d’accès recevront tous les rôles de ressource dans le package d’accès.

1. Cliquez sur le type de ressource que vous souhaitez ajouter (**Groupes et équipes**, **Applications** ou **Sites SharePoint**).

1. Dans le volet de sélection qui s’affiche, sélectionnez une ou plusieurs ressources dans la liste.

    ![Package d’accès - Rôles des ressources](./media/entitlement-management-access-package-create/resource-roles.png)

    Si vous créez le package d’accès dans le catalogue Général ou dans un nouveau catalogue, vous pouvez récupérer n’importe quelle ressource à partir du répertoire que vous possédez. Vous devez être au moins Administrateur général, Administrateur d’utilisateurs ou Créateur de catalogue.

    Si vous créez le package d’accès dans un catalogue existant, vous pouvez sélectionner n’importe quelle ressource déjà présente dans le catalogue sans avoir à en être propriétaire.

    Si vous êtes Administrateur général, Administrateur d’utilisateurs ou Propriétaire de catalogue, vous avez également la possibilité de sélectionner des ressources que vous possédez et qui ne sont pas encore dans le catalogue. Si vous sélectionnez des ressources qui ne sont actuellement pas présentes dans le catalogue sélectionné, ces ressources sont ajoutées au catalogue pour que d’autres administrateurs du catalogue puissent créer des packages d’accès à partir de ces dernières. Pour afficher toutes les ressources qui peuvent être ajoutées au catalogue, cochez la case **Afficher tout** en haut du volet Sélectionner. Si vous souhaitez uniquement sélectionner les ressources qui se trouvent actuellement dans le catalogue sélectionné, laissez la case à cocher **Afficher tout** désactivée (état par défaut).

1. Une fois que vous avez sélectionné les ressources, dans la liste **Rôle**, sélectionnez le rôle à assigner aux utilisateurs pour les ressources.

    ![Package d’accès - Sélection du rôle des ressources](./media/entitlement-management-access-package-create/resource-roles-role.png)

1. Cliquez sur **Suivant**.

>[!NOTE]
>Vous pouvez ajouter des groupes dynamiques à un catalogue et à un package d’accès. Toutefois, si vous gérez une ressource de groupe dynamique dans un package d’accès, vous ne pourrez sélectionner que le rôle Propriétaire.

## <a name="requests"></a>Demandes

Sous l’onglet **Requêtes**, vous créez la première stratégie afin de spécifier qui peut demander le package d’accès, ainsi que les paramètres d’approbation et d’expiration. Plus tard, vous pourrez créer plus de stratégies de requêtes afin d’autoriser des groupes d’utilisateurs supplémentaires à demander le package d’accès avec leurs propres paramètres d’approbation.

![Onglet Requêtes de package d’accès](./media/entitlement-management-access-package-create/requests.png)

Selon les utilisateurs qui pourront demander ce package d’accès, suivez les étapes décrites dans l’une des sections suivantes.

[!INCLUDE [Entitlement management request policy](../../../includes/active-directory-entitlement-management-request-policy.md)]

[!INCLUDE [Entitlement management lifecycle policy](../../../includes/active-directory-entitlement-management-lifecycle-policy.md)]

## <a name="review--create"></a>Vérifier + créer

Sous l’onglet **Vérifier + créer**, vous pouvez consulter vos paramètres et vérifier qu’ils ne contiennent aucune erreur de validation.

1. Vérifier les paramètres du package d’accès

    ![Package d’accès – Activer le paramètre de stratégie](./media/entitlement-management-access-package-create/review-create.png)

1. Cliquez sur **Créer** pour créer le package d’accès.

    Le nouveau package d’accès apparaît dans la liste des packages d’accès.

## <a name="creating-an-access-package-programmatically"></a>Création d’un package par programme

Vous pouvez également créer un package d’accès à l’aide de Microsoft Graph.  Un utilisateur doté d’un rôle approprié avec une application disposant de l’autorisation déléguée `EntitlementManagement.ReadWrite.All` peut appeler l’API pour

1. [Répertorier les ressources accessPackageResources dans le catalogue](/graph/api/accesspackagecatalog-list?tabs=http&view=graph-rest-beta) et [créer une demande accessPackageResourceRequest](/graph/api/accesspackageresourcerequest-post?tabs=http&view=graph-rest-beta) pour toutes les ressources qui ne figurent pas encore dans le catalogue.
1. [Répertorier les rôles accessPackageResourceRoles](/graph/api/accesspackage-list-accesspackageresourcerolescopes?tabs=http&view=graph-rest-beta) de chaque ressources accessPackageResource dans un catalogue accessPackageCatalog. Cette liste de rôles est ensuite utilisée pour sélectionner un rôle lors de la création ultérieure d’une étendue accessPackageResourceRoleScope.
1. [Créer un package accessPackage](/graph/tutorial-access-package-api?view=graph-rest-beta).
1. [Créer une stratégie accessPackageAssignmentPolicy](/graph/api/accesspackageassignmentpolicy-post?tabs=http&view=graph-rest-beta).
1. [Créer une étendue accessPackageResourceRoleScope](/graph/api/accesspackage-post-accesspackageresourcerolescopes?tabs=http&view=graph-rest-beta) pour chaque rôle de ressource nécessaire dans le package d’accès.

## <a name="next-steps"></a>Étapes suivantes

- [Partager un lien pour demander un package d’accès](entitlement-management-access-package-settings.md)
- [Modifier les rôles de ressources d’un package d’accès](entitlement-management-access-package-resources.md)
- [Affecter directement un utilisateur au package d’accès](entitlement-management-access-package-assignments.md)
