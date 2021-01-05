---
title: Assigner ou supprimer des licences – Azure Active Directory | Microsoft Docs
description: Ces instructions expliquent comment assigner des licences Azure Active Directory à vos utilisateurs ou groupes et les supprimer.
services: active-directory
author: ajburnle
manager: daveba
ms.assetid: f8b932bc-8b4f-42b5-a2d3-f2c076234a78
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
ms.date: 12/14/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: b6900647acf7182529f34c8cc065dbb039de38be
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97504404"
---
# <a name="assign-or-remove-licenses-in-the-azure-active-directory-portal"></a>Assigner ou supprimer des licences dans le portail Azure Active Directory

De nombreux services Azure Active Directory (Azure AD) nécessitent que vous assigniez une licence à chacun de vos utilisateurs ou groupes (et à leurs membres associés) pour ce service. Seuls les utilisateurs disposant de licences actives seront en mesure d’accéder aux services Azure AD sous licence pour lesquels cela est vrai, et de les utiliser . Les licences sont appliquées par locataire et ne sont pas transférées à d’autres locataires. 

## <a name="available-license-plans"></a>Plans de licence disponibles

Plusieurs plans de licence sont disponibles pour le service Azure AD, notamment :

- Azure AD Gratuit

- Azure AD Premium P1

- Azure AD Premium P2

Pour obtenir des informations spécifiques sur chaque plan de licence et sur les licences associées, consultez [De quelle licence ai-je besoin ?](https://azure.microsoft.com/pricing/details/active-directory/). Pour vous inscrire aux plans de licence Premium Azure AD des plans, voir [ici](./active-directory-get-started-premium.md).

Certains services Microsoft ne sont pas disponibles dans tous les emplacements. Avant de pouvoir attribuer une licence à un groupe, vous devez spécifier l’**Emplacement d’utilisation** pour tous les membres. Vous pouvez définir cette valeur dans la zone **Azure Active Directory &gt; Utilisateurs &gt; Profil &gt; Paramètres** dans Azure AD. Tout utilisateur dont l’emplacement d’utilisation n’est pas spécifié hérite de l’emplacement de l’organisation Azure AD.

## <a name="view-license-plans-and-plan-details"></a>Afficher les plans de licence et les détails du plan

Vous pouvez afficher les plans de service disponibles, y compris les licences individuelles, vérifier les dates d’expiration en attente et vérifier le nombre d’affectations disponibles.

### <a name="to-find-your-service-plan-and-plan-details"></a>Pour rechercher votre plan de service et les détails du plan

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) à l’aide d’un compte Administrateur de licence dans votre organisation Azure AD.

1. Sélectionnez **Azure Active Directory**, puis **Licences**.

    :::image type="content" source="media/license-users-groups/license-details-blade.png" alt-text="Page Licences, indiquant le nombre de services achetés et de licences affectées":::

1. Sélectionnez **Tous les produits** pour afficher la page Tous les produits et les valeurs **Total**, **Assigné**, **Disponible** et **Arrive bientôt à expiration** pour vos plans de licence.

    :::image type="content" source="media/license-users-groups/license-products-blade-with-products.png" alt-text="Page de services, avec les plans de licence de service, et les informations de licence associées":::

    > [!NOTE]
    > Les nombres sont définis comme suit : 
    > - Total : nombre total de licences achetées
    > - Assigné : nombre de licences attribuées aux utilisateurs
    > - Disponible : nombre de licences attribuables, y compris celles arrivant bientôt à expiration
    > - Arrive bientôt à expiration Nombre de licences arrivant bientôt à expiration

1. Sélectionnez un nom de plan pour afficher ses utilisateurs et groupes sous licence.

## <a name="assign-licenses-to-users-or-groups"></a>Attribuer des licences aux utilisateurs ou aux groupes

Assurez-vous que toutes les personnes devant utiliser un service Azure AD sous licence disposent de la licence appropriée. Vous pouvez ajouter les droits de licence aux utilisateurs ou à un groupe entier.

### <a name="to-assign-a-license-to-a-user"></a>Pour affecter une licence à un utilisateur

1. Dans la page **Produits**, sélectionnez le nom du plan de licence que vous souhaitez affecter à l’utilisateur.

    ![Page de services, avec un plan de licence de service en surbrillance](media/license-users-groups/license-products-blade-with-product-highlight.png)

1. Dans la page de vue d’ensemble du plan de licence, sélectionnez **Affecter**.

    ![Page de services, avec l’option Affecter en surbrillance](media/license-users-groups/license-products-blade-with-assign-option-highlight.png)

1. Sur la page **Affecter**, sélectionnez **Utilisateurs et groupes**, puis recherchez et sélectionnez l’utilisateur auquel vous souhaitez affecter la licence.

    ![Page Affecter une licence, avec les options Sélectionner et Rechercher en surbrillance](media/license-users-groups/assign-license-blade-with-highlight.png)

1. Sélectionnez **Options d’affectation**. Assurez-vous que les options de licence appropriées sont activées, puis sélectionnez **OK**.

    ![Page d’options de licence, avec toutes les options disponibles dans le plan de licence](media/license-users-groups/license-option-blade-assignments.png)

    La page **Affecter une licence** s’actualise pour indiquer qu’un utilisateur est sélectionné et que les attributions sont configurées.

    > [!NOTE]
    > Certains services Microsoft ne sont pas disponibles dans tous les emplacements. Avant de pouvoir attribuer une licence à un utilisateur, vous devez spécifier l’**Emplacement d’utilisation**. Vous pouvez définir cette valeur dans la zone **Azure Active Directory &gt; Utilisateurs &gt; Profil &gt; Paramètres** dans Azure AD. Tout utilisateur dont l’emplacement d’utilisation n’est pas spécifié hérite de l’emplacement de l’organisation Azure AD.

1. Sélectionnez **Attribuer**.

    L’utilisateur est ajouté à la liste des utilisateurs sous licence et a accès aux services Azure AD inclus.
    > [!NOTE]
    > Les licences peuvent également être attribuées directement à un utilisateur à partir de la page **Licences** de l’utilisateur. Si un utilisateur dispose d’une licence attribuée par le biais d’une appartenance à un groupe et que vous souhaitez attribuer la même licence à l’utilisateur directement, vous ne pouvez le faire qu’à partir de la page **Produits** mentionnée à l’étape 1 uniquement.

### <a name="to-assign-a-license-to-a-group"></a>Pour affecter une licence à un groupe

1. Dans la page **Produits**, sélectionnez le nom du plan de licence que vous souhaitez affecter à l’utilisateur.

    ![Panneau Produits, avec le plan de licence de produit en surbrillance](media/license-users-groups/license-products-blade-with-product-highlight.png)

1. Sur la page **Offre 2 d’Azure Active Directory Premium**, sélectionnez **Affecter**.

    ![Page Produits, avec l’option Affecter en surbrillance](media/license-users-groups/license-products-blade-with-assign-option-highlight.png)

1. Sur la page **Affecter**, sélectionnez **Utilisateurs et groupes**, puis recherchez et sélectionnez le groupe auquel vous souhaitez affecter la licence.

    ![Page Affecter une licence, avec les options Sélectionner et Rechercher mises en surbrillance 2](media/license-users-groups/assign-group-license-blade-with-highlight.png)

1. Sélectionnez **Options d’affectation**. Assurez-vous que les options de licence appropriées sont activées, puis sélectionnez **OK**.

    ![Page d’options de licence, avec toutes les options disponibles dans le plan de licence 2](media/license-users-groups/license-option-blade-group-assignments.png)

    La page **Affecter une licence** s’actualise pour indiquer qu’un utilisateur est sélectionné et que les attributions sont configurées.

1. Sélectionnez **Attribuer**.

    Le groupe est ajouté à la liste des groupes sous licence et tous les membres ont accès aux services Azure AD inclus.

## <a name="remove-a-license"></a>Supprimer une licence

Vous pouvez supprimer une licence de la page d’utilisateur Azure AD d’un utilisateur, à partir de la page de vue d’ensemble du groupe pour une affectation de groupe, ou à partir de la page **Licences** Azure AD pour afficher les utilisateurs et les groupes d’une licence.

### <a name="to-remove-a-license-from-a-user"></a>Pour supprimer une licence d’un utilisateur

1. Dans la page **Utilisateurs sous licence** du plan de service, sélectionnez l’utilisateur dont vous souhaitez supprimer la licence. Par exemple, _Alain Charon_.

1. Sélectionnez **Supprimer la licence**.

    ![Page Utilisateurs sous licence, avec l’option Supprimer une licence en surbrillance](media/license-users-groups/license-products-user-blade-with-remove-option-highlight.png)

> [!IMPORTANT]
> Les licences dont un utilisateur hérite à partir d’un groupe ne peuvent pas être supprimées directement. Au lieu de cela, vous devez supprimer l’utilisateur du groupe à partir duquel il a hérité de la licence.

### <a name="to-remove-a-license-from-a-group"></a>Pour supprimer une licence d’un groupe

1. Dans la page **Groupes sous licence** du plan de licence, sélectionnez le groupe dont vous souhaitez supprimer la licence.

1. Sélectionnez **Supprimer la licence**.

    ![Page Groupes sous licence avec l’option Supprimer une licence mise en surbrillance 2](media/license-users-groups/license-products-group-blade-with-remove-option-highlight.png)
    
    > [!NOTE]
    > Lorsqu’un compte d’utilisateur local synchronisé avec Azure AD est hors de portée pour la synchronisation ou lorsque la synchronisation est supprimée, l’utilisateur est supprimé de manière réversible dans Azure AD. Lorsque cela se produit, les licences attribuées directement à l’utilisateur ou via des licences basées sur des groupes sont marquées comme **suspendues** plutôt que **supprimées**.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez assigné vos licences, vous pouvez exécuter les procédures suivantes :

- [Identifier et résoudre les problèmes d’affectation de licence](../enterprise-users/licensing-groups-resolve-problems.md)

- [Ajouter des utilisateurs sous licence à un groupe pour la gestion des licences](../enterprise-users/licensing-groups-migrate-users.md)

- [Scénarios, limitations et problèmes connus liés à l’utilisation de groupes dans le cadre de la gestion des licences dans Azure Active Directory](../enterprise-users/licensing-group-advanced.md)

- [Ajouter ou modifier les informations de profil](active-directory-users-profile-azure-portal.md)
