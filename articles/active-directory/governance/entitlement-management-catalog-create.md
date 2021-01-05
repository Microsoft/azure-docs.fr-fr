---
title: Créer et gérer un catalogue de ressources dans la gestion des droits d'utilisation - Azure AD
description: Découvrez comment créer un conteneur de ressources et de packages d’accès dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: HANKI
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: hanki
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4b87fdd7c73d089c46d456491c02d49cb6f8883d
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97745961"
---
# <a name="create-and-manage-a-catalog-of-resources-in-azure-ad-entitlement-management"></a>Créer et gérer un catalogue de ressources dans la gestion des droits d’utilisation Azure AD

## <a name="create-a-catalog"></a>Créer un catalogue

Un catalogue est un conteneur de ressources et de packages d’accès. Vous créez un catalogue lorsque vous souhaitez regrouper des ressources et packages d’accès liés. La personne qui crée le catalogue en devient le premier propriétaire. Un propriétaire de catalogue peut ajouter d’autres propriétaires de catalogue.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou créateur de catalogue

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**.

    ![Catalogues de gestion des droits d’utilisation dans le Portail Azure](./media/entitlement-management-catalog-create/catalogs.png)

1. Cliquez sur **Nouveau catalogue**.

1. Entrez un nom unique pour le catalogue et fournissez une description.

    Les utilisateurs verront ces informations dans les détails d’un package d’accès.

1. Si vous souhaitez que les packages d’accès dans ce catalogue soient accessibles aux utilisateurs qui les demandent dès leur création, définissez **Activé** sur **Oui**.

1. Si vous souhaitez que les utilisateurs de répertoires externes sélectionnés puissent demander des packages d’accès dans ce catalogue, définissez **Activé pour les utilisateurs externes** sur **Oui**.

    ![Volet Nouveau catalogue](./media/entitlement-management-shared/new-catalog.png)

1. Cliquez sur **Créer** pour créer le catalogue.

### <a name="creating-a-catalog-programmatically"></a>Créer un catalogue par programmation

Vous pouvez également créer un catalogue à l'aide de Microsoft Graph.  Un utilisateur doté d'un rôle approprié avec une application disposant de l'autorisation déléguée `EntitlementManagement.ReadWrite.All` peut appeler l'API pour [créer un catalogue accessPackageCatalog](/graph/api/accesspackagecatalog-post?view=graph-rest-beta).

## <a name="add-resources-to-a-catalog"></a>Ajouter des ressources à un catalogue

Pour inclure des ressources dans un package d’accès, les ressources doivent exister dans un catalogue. Les types de ressources que vous pouvez ajouter sont des groupes, des applications et des sites SharePoint Online. Pour les groupes, il peut s’agir de groupes Microsoft 365 ou de groupes de sécurité Azure AD créés sur le cloud. Pour les applications, il peut s’agir d’applications d’entreprise Azure AD, ce qui inclut les applications SaaS et vos propres applications fédérées avec Azure AD. Pour les sites, il peut s’agir de sites ou de collection de sites SharePoint Online.

**Rôle prérequis :** Voir [Rôles requis pour ajouter des ressources à un catalogue](entitlement-management-delegate.md#required-roles-to-add-resources-to-a-catalog)

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**, puis ouvrez le catalogue auquel vous souhaitez ajouter des ressources.

1. Dans le menu de gauche, cliquez sur **Ressources**.

1. Cliquez sur **Ajouter des ressources**.

1. Cliquez sur un type de ressources : **Groupes et équipes**, **Applications** ou **Sites SharePoint**.

    Si vous ne voyez aucune ressource que vous souhaitez ajouter ou si vous ne pouvez pas ajouter de ressource, vérifiez que vous possédez le rôle d’annuaire Azure AD et le rôle de gestion des droits d’utilisation requis. Vous devrez peut-être demander à quelqu’un avec les rôles requis d’ajouter la ressource à votre catalogue. Pour plus d’informations, consultez [Rôles requis pour ajouter des ressources à un catalogue](entitlement-management-delegate.md#required-roles-to-add-resources-to-a-catalog).

1. Sélectionnez une ou plusieurs ressources du type que vous souhaitez ajouter au catalogue.

    ![Ajouter des ressources à un catalogue](./media/entitlement-management-catalog-create/catalog-add-resources.png)

1. Lorsque vous avez terminé, cliquez sur **Ajouter**.

    Ces ressources peuvent désormais être incluses dans des packages d’accès du catalogue.

### <a name="adding-a-resource-to-a-catalog-programmatically"></a>Ajouter une ressource à un catalogue par programmation

Vous pouvez également ajouter une ressource à un catalogue à l'aide de Microsoft Graph.  Un utilisateur doté d'un rôle approprié, ou un propriétaire de catalogue et de ressource, avec une application disposant de l'autorisation déléguée `EntitlementManagement.ReadWrite.All` peut appeler l'API pour [créer une demande accessPackageResourceRequest](/graph/api/accesspackageresourcerequest-post?view=graph-rest-beta).

## <a name="remove-resources-from-a-catalog"></a>Supprimer des ressources d’un catalogue

Vous pouvez supprimer des ressources d’un catalogue. Une ressource ne peut être supprimée d’un catalogue que si elle n’est pas utilisée dans un des packages d’accès du catalogue.

**Rôle prérequis :** Voir [Rôles requis pour ajouter des ressources à un catalogue](entitlement-management-delegate.md#required-roles-to-add-resources-to-a-catalog)

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**, puis ouvrez le catalogue duquel vous souhaitez supprimer des ressources.

1. Dans le menu de gauche, cliquez sur **Ressources**.

1. Sélectionnez les ressources que vous souhaitez supprimer.

1. Cliquez sur **Supprimer** (ou cliquez sur le bouton de sélection ( **...** ), puis sur **Supprimer la ressource**).


## <a name="add-additional-catalog-owners"></a>Ajouter d’autres propriétaires de catalogue

L’utilisateur qui a créé un catalogue devient le premier propriétaire de catalogue. Pour déléguer la gestion d’un catalogue, vous ajoutez des utilisateurs au rôle de propriétaire de catalogue. Ainsi, les responsabilités de la gestion du catalogue sont mieux partagées. 

Suivez ces étapes pour affecter un utilisateur au rôle de propriétaire de catalogue :

**Rôle prérequis :** administrateur général, administrateur d’utilisateurs ou propriétaire de catalogue.

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**, puis ouvrez le catalogue auquel vous souhaitez ajouter des administrateurs.

1. Dans le menu de gauche, cliquez sur **Rôles et administrateurs**.

    ![Rôles et administrateurs de catalogues](./media/entitlement-management-shared/catalog-roles-administrators.png)

1. Cliquez sur **Ajouter des propriétaires** pour sélectionner les membres de ces rôles.

1. Cliquez sur **Sélectionner** pour ajouter ces membres.

## <a name="edit-a-catalog"></a>Modifier un catalogue

Vous pouvez modifier le nom et la description d’un catalogue. Les utilisateurs verront ces informations dans les détails d’un package d’accès.

**Rôle prérequis :** administrateur général, administrateur d’utilisateurs ou propriétaire de catalogue.

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**, puis ouvrez le catalogue que vous souhaitez modifier.

1. Dans la page **Vue d’ensemble** du catalogue, cliquez sur **Modifier**.

1. Modifiez le nom, la description ou les paramètres activés du catalogue.

    ![Modifier les paramètres du catalogue](./media/entitlement-management-shared/catalog-edit.png)

1. Cliquez sur **Enregistrer**.

## <a name="delete-a-catalog"></a>Supprimer un catalogue

Vous pouvez supprimer un catalogue, mais seulement s’il ne contient pas de packages d’accès.

**Rôle prérequis :** administrateur général, administrateur d’utilisateurs ou propriétaire de catalogue.

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Catalogues**, puis ouvrez le catalogue que vous souhaitez supprimer.

1. Dans la **Vue d’ensemble** du catalogue, cliquez sur **Supprimer**.

1. Dans la boîte de message qui s’affiche, cliquez sur **Oui**.

### <a name="deleting-a-catalog-programmatically"></a>Supprimer un catalogue par programmation

Vous pouvez également supprimer un catalogue à l'aide de Microsoft Graph.  Un utilisateur doté d'un rôle approprié avec une application disposant de l'autorisation déléguée `EntitlementManagement.ReadWrite.All` peut appeler l'API pour [supprimer un catalogue accessPackageCatalog](/graph/api/accesspackagecatalog-delete?view=graph-rest-beta).

## <a name="next-steps"></a>Étapes suivantes

- [Déléguer la gouvernance des accès aux gestionnaires de package d’accès](entitlement-management-delegate-managers.md)