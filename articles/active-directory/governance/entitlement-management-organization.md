---
title: Ajouter une organisation connectée dans la gestion des droits d’utilisation Azure AD – Azure Active Directory
description: Découvrez comment autoriser les personnes extérieures à votre organisation à demander des packages d’accès afin de pouvoir collaborer sur des projets.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/11/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6299f9ad660252ae39903401672d493b1c1ad52c
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97347487"
---
# <a name="add-a-connected-organization-in-azure-ad-entitlement-management"></a>Ajouter une organisation connectée dans la gestion des droits d’utilisation Azure AD

Avec la gestion des droits d’utilisation Azure Active Directory (Azure AD), vous pouvez collaborer avec des personnes extérieures à votre organisation. Si vous collaborez fréquemment avec des utilisateurs dans un annuaire ou domaine Azure AD externe, vous pouvez les ajouter en tant qu’organisation connectée. Cet article explique comment ajouter une organisation connectée afin de permettre aux utilisateurs extérieurs à votre organisation de demander des ressources dans votre annuaire.

## <a name="what-is-a-connected-organization"></a>Qu’est-ce qu’une organisation connectée ?

Une organisation connectée est une autre organisation avec laquelle vous avez une relation.  Pour que les utilisateurs de cette organisation puissent accéder à vos ressources, telles que vos sites SharePoint Online ou vos applications, vous avez besoin d’une représentation des utilisateurs de cette organisation dans ce répertoire.  Étant donné que, dans la plupart des cas, les utilisateurs de cette organisation ne sont pas encore dans votre répertoire Azure AD, vous pouvez utiliser la gestion des droits d’utilisation pour les placer dans votre répertoire Azure AD en fonction des besoins.  

La gestion des droits d’utilisation vous permet de spécifier les utilisateurs qui forment une organisation connectée de trois façons.  Il peut s’agir :

* d’utilisateurs d’un autre répertoire Azure AD ;
* d’utilisateurs d’un autre répertoire non-Azure AD qui a été configuré pour la fédération directe ;
* d’utilisateurs d’un autre répertoire non-Azure AD dont les adresses e-mail ont toutes le même nom de domaine en commun.

Par exemple, supposons que vous travaillez chez Woodgrove Bank et que vous souhaitez collaborer avec deux organisations externes. Ces deux organisations ont des configurations différentes :

- Graphic Design Institute utilise Azure AD, et ses utilisateurs ont un nom d’utilisateur principal qui se termine par *graphicdesigninstitute.com*.
- Contoso n’utilise pas encore Azure AD. Les utilisateurs Contoso ont un nom d’utilisateur principal qui se termine par *contoso.com*.

Dans ce cas, vous pouvez configurer deux organisations connectées. Vous devez créer une organisation connectée pour Graphic Design Institute et une autre pour Contoso. Si vous ajoutez ensuite ces deux organisations connectées à une stratégie, les utilisateurs de chaque organisation qui ont un nom d’utilisateur principal correspondant à la stratégie peuvent demander des packages d’accès. Les utilisateurs dont le nom d’utilisateur principal comprend le domaine *graphicdesigninstitute.com* sont ceux de l’organisation connectée à Graphics Institute et sont autorisés à envoyer des demandes. Les utilisateurs dont le nom d’utilisateur principal comprend le domaine *contoso.com* sont ceux de l’organisation connectée à Contoso et sont également autorisés à demander des packages. De plus, étant donné que Graphic Design Institute utilise Azure AD, tous les utilisateurs dont le nom principal correspond à un [domaine vérifié](../fundamentals/add-custom-domain.md#verify-your-custom-domain-name) suivi de leur locataire (comme dans *graphicdesigninstitute.example*), peuvent également demander des packages d’accès à l’aide de la même stratégie.

![Exemple d’organisation connectée](./media/entitlement-management-organization/connected-organization-example.png)

La manière dont les utilisateurs de l’annuaire ou du domaine Azure AD s’authentifient dépend du type d’authentification. Les types d’authentification pour les organisations connectées sont :

- Azure AD
- [Fédération directe](../external-identities/direct-federation.md)
- [Code secret à usage unique](../external-identities/one-time-passcode.md) (domaine)

Pour une démonstration de l’ajout d’une organisation connectée, regardez la vidéo suivante :

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4dskS]

## <a name="add-a-connected-organization"></a>Ajouter une organisation connectée

Pour ajouter un annuaire ou un domaine Azure AD externe en tant qu’organisation connectée, suivez les instructions de cette section.

**Rôle prérequis** : *Administrateur général* ou *Administrateur d’utilisateurs*

1. Dans le portail Azure, sélectionnez **Azure Active Directory**, puis **Identity Governance**.

1. Dans le volet de gauche, sélectionnez **Organisations connectées**, puis **Ajouter une organisation connectée**.

    ![Bouton « Ajouter une organisation connectée »](./media/entitlement-management-organization/connected-organization.png)

1. Sélectionnez l’onglet **De base**, puis entrez un nom d’affichage et une description pour l’organisation.

    ![Bouton Ajouter une organisation connectée - Onglet De base](./media/entitlement-management-organization/organization-basics.png)

1. L’état est automatiquement défini sur **Configuré** lorsque vous créez une organisation connectée. Pour plus d’informations sur les propriétés d’état, consultez [Propriétés d’état des organisations connectées](#state-properties-of-connected-organizations).

1. Sélectionnez l’onglet **Annuaire + domaine**, puis sélectionnez **Ajouter un annuaire + domaine**.

    Le volet **Sélectionner des annuaires et des domaines** s’ouvre.

1. Dans la zone de recherche, entrez un nom de domaine pour rechercher l’annuaire ou le domaine Azure AD. Veillez à entrer le nom de domaine complet.

1. Vérifiez que le nom de l’organisation et le type d’authentification sont corrects. La manière dont les utilisateurs se connectent dépend du type d’authentification.

    ![Volet « Sélectionner des annuaires et des domaines »](./media/entitlement-management-organization/organization-select-directories-domains.png)

1. Sélectionnez **Ajouter** pour ajouter l’annuaire ou le domaine Azure AD. Vous ne pouvez ajouter qu’un seul annuaire ou domaine Azure AD par organisation connectée.

    > [!NOTE]
    > Tous les utilisateurs de l’annuaire ou du domaine Azure AD seront en mesure de demander ce package d’accès. Cela comprend les utilisateurs Azure AD de tous les sous-domaines associés à l’annuaire, à moins que ces domaines ne soient bloqués par la liste d’autorisation ou de refus d’Azure AD B2B. Pour plus d’informations, consultez [Autoriser ou bloquer des invitations aux utilisateurs B2B à partir d’organisations spécifiques](../external-identities/allow-deny-list.md).

1. Une fois que vous avez ajouté l’annuaire ou le domaine Azure AD, sélectionnez **Sélectionner**.

    L’organisation apparaît dans la liste.

    ![Volet « Annuaire + domaine »](./media/entitlement-management-organization/organization-directory-domain.png)

1. Sélectionnez l’onglet **Commanditaires**, puis ajoutez des commanditaires facultatifs pour cette organisation connectée.

    Les commanditaires sont des utilisateurs internes ou externes qui se trouvent déjà dans votre annuaire et qui constituent le point de contact pour la relation avec cette organisation connectée. Les commanditaires internes sont des utilisateurs membres de votre annuaire. Les commanditaires externes sont des utilisateurs invités de l’organisation connectée qui ont été invités précédemment et qui se trouvent déjà dans votre annuaire. Les commanditaires peuvent être utilisés en tant qu’approbateurs lorsque des utilisateurs de cette organisation connectée demandent l’accès à ce package d’accès. Pour plus d’informations sur l’invitation d’un utilisateur invité à votre annuaire, consultez [Ajout d'utilisateurs de collaboration Azure Active Directory B2B dans le portail Azure](../external-identities/add-users-administrator.md).

    Lorsque vous sélectionnez **Ajouter/Supprimer**, un volet s’ouvre dans lequel vous pouvez choisir des commanditaires internes ou externes. Le volet affiche une liste non filtrée d’utilisateurs et de groupes dans votre annuaire.

    ![Volet Commanditaires](./media/entitlement-management-organization/organization-sponsors.png)

1. Sélectionnez l’onglet **Vérifier + créer**, vérifiez les paramètres de votre organisation, puis sélectionnez **Créer**.

    ![Volet « Vérifier + créer »](./media/entitlement-management-organization/organization-review-create.png)

## <a name="update-a-connected-organization"></a>Mettre à jour une organisation connectée 

Si l’organisation connectée change de domaine, si le nom de l’organisation change ou si vous souhaitez modifier les commanditaires, vous pouvez mettre à jour l’organisation connectée en suivant les instructions de cette section.

**Rôle prérequis** : *Administrateur général* ou *Administrateur d’utilisateurs*

1. Dans le portail Azure, sélectionnez **Azure Active Directory**, puis **Identity Governance**.

1. Dans le volet de gauche, sélectionnez **Organisations connectées**, puis sélectionnez l’organisation connectée pour l’ouvrir.

1. Dans la vue d’ensemble de l’organisation connectée, sélectionnez **Modifier** pour modifier le nom, la description ou l’état de l’organisation.  

1. Dans la page **Annuaire + domaine**, sélectionnez **Mettre à jour l’annuaire + le domaine** pour basculer vers un autre annuaire ou domaine.

1. Dans la page **Commanditaires**, sélectionnez **Ajouter des sponsors internes** ou **Ajouter des sponsors externes** pour ajouter un utilisateur en tant que sponsor. Pour supprimer un commanditaire, sélectionnez-le, puis, dans le volet droit, sélectionnez **Supprimer**.


## <a name="delete-a-connected-organization"></a>Supprimer une organisation connectée

Si vous n’avez plus de relation avec un annuaire ou un domaine Azure AD externe, vous pouvez supprimer l’organisation connectée.

**Rôle prérequis** : *Administrateur général* ou *Administrateur d’utilisateurs*

1. Dans le portail Azure, sélectionnez **Azure Active Directory**, puis **Identity Governance**.

1. Dans le volet de gauche, sélectionnez **Organisations connectées**, puis sélectionnez l’organisation connectée pour l’ouvrir.

1. Dans la vue d’ensemble de l’organisation connectée, sélectionnez **Supprimer** pour la supprimer.

    Vous pouvez supprimer une organisation connectée seulement si aucun utilisateur n’est connecté.

    ![Bouton « Supprimer une organisation connectée »](./media/entitlement-management-organization/organization-delete.png)

## <a name="managing-a-connected-organization-programmatically"></a>Gestion d’une organisation connectée par programmation

Vous pouvez également créer, répertorier, mettre à jour et supprimer des organisations connectées à l’aide de Microsoft Graph. Un utilisateur doté d’un rôle approprié avec une application disposant de l’autorisation déléguée `EntitlementManagement.ReadWrite.All` peut appeler l’API pour gérer les objets [connectedOrganization](/graph/api/resources/connectedorganization?view=graph-rest-beta) et définir des commanditaires pour eux.

## <a name="state-properties-of-connected-organizations"></a>Propriétés d’état des organisations connectées

Il existe actuellement deux types de propriétés d’état pour les organisations connectées dans la gestion des droits d’utilisation Azure AD, à savoir configurée et proposée : 

- Une organisation connectée configurée est une organisation connectée entièrement fonctionnelle qui permet à ses utilisateurs d’accéder aux packages d’accès. Quand un administrateur crée une organisation connectée dans le portail Azure, celle-ci est en état **configuré** par défaut, car l’administrateur souhaite utiliser cette organisation connectée. En outre, quand une organisation connectée est créée par programme via l’API, l’état par défaut doit être **configuré** sauf si un autre état est explicitement défini. 

    Les organisations connectées configurées s’afficheront dans les sélecteurs pour les organisations connectées, et s’inscriront dans l’étendue de toutes les stratégies ciblant « toutes » les organisations connectées.

- Une organisation connectée proposée est une organisation connectée qui a été créée automatiquement, sans qu’un administrateur la crée ou l’approuve. Quand un utilisateur s’inscrit pour un package d’accès en dehors d’une organisation connectée configurée, toutes les organisations connectées créées automatiquement se trouvent dans l’état **proposé**, car aucun administrateur dans le locataire n’a configuré ce partenariat. 
    
    Les organisations connectées proposées ne s’inscrivent pas dans l’étendue du paramètre « toutes les organisations connectées configurées » d’aucune stratégie, mais peuvent être utilisées uniquement pour les stratégies ciblant des organisations spécifiques. 

Seuls des utilisateurs d’organisations connectées configurées peuvent demander des packages d’accès disponibles pour les utilisateurs de toutes les organisations configurées. Les utilisateurs d’organisations connectées proposées ont une expérience comme s’il n’y avait pas d’organisation connectée pour ce domaine ; ils peuvent uniquement voir et demander des packages d’accès limités à leur organisation spécifique ou à n’importe quel utilisateur.

> [!NOTE]
> Dans le cadre du déploiement de cette nouvelle fonctionnalité, toutes les organisations connectées créées avant 09/09/20 ont été considérées comme **configurées**. Si vous aviez un package d’accès permettant aux utilisateurs de toute organisation de s’inscrire, vous devriez examiner votre liste d’organisations connectées créées avant cette date pour vous assurer qu’aucune n’est erronément catégorisée comme **configurée**.  Un administrateur peut mettre à jour la propriété **État** si nécessaire. Pour obtenir de l’aide, consultez [Mettre à jour une organisation connectée](#update-a-connected-organization).

## <a name="next-steps"></a>Étapes suivantes

- [Régir l’accès des utilisateurs externes](./entitlement-management-external-users.md)
- [Régir l’accès des utilisateurs qui ne font pas partie de votre annuaire](entitlement-management-access-package-request-policy.md#for-users-not-in-your-directory)
