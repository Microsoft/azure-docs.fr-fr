---
title: Tutoriel – Créer un package d’accès – Gestion des droits d’utilisation Azure AD
description: Ce tutoriel pas à pas vous explique comment créer votre premier package d’accès dans la gestion des droits d’utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.subservice: compliance
ms.date: 09/30/2020
ms.author: barclayn
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2e8d68643b5cc0002467f738dd60343fb2c42dc6
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97631281"
---
# <a name="tutorial-create-your-first-access-package-in-azure-ad-entitlement-management"></a>Tutoriel : Créer votre premier package d’accès dans la gestion des droits d’utilisation Azure AD

La gestion de l’accès à toutes les ressources dont les employés ont besoin, comme les groupes, les applications et les sites, est une fonction importante pour les organisations. Vous devez accorder aux employés le niveau d’accès dont ils ont besoin pour être productifs et leur retirer cet accès dès qu’il n’est plus nécessaire.

Dans ce tutoriel, vous travaillez pour Woodgrove Bank en tant qu’administrateur informatique. Vous êtes chargé de créer un package de ressources pour une campagne marketing en veillant à ce que les utilisateurs internes puissent demander l’accès à ces ressources en libre-service. Les demandes ne doivent pas être approuvées, et l’accès de l’utilisateur est valide 30 jours. Dans ce tutoriel, les ressources de la campagne marketing représentent simplement une appartenance à un groupe unique, mais vous pouvez avoir un ensemble de groupes, d’applications ou de sites SharePoint Online.

![Diagramme illustrant la vue d’ensemble du scénario.](./media/entitlement-management-access-package-first/elm-scenario-overview.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un package d’accès avec un groupe comme ressource
> * Autoriser un utilisateur dans votre annuaire à demander l’accès
> * Montrer comment un utilisateur interne peut demander le package d’accès

Pour obtenir une démonstration pas à pas du processus de déploiement de la gestion des droits d'utilisation d’Azure Active Directory, notamment la création de votre premier package d’accès, regardez la vidéo suivante :

>[!VIDEO https://www.youtube.com/embed/zaaKvaaYwI4]

Vous pouvez également créer un package d’accès par programme à l’aide de Microsoft Graph. Pour obtenir un tutoriel qui montre comment créer un package d’accès par programme, consultez [API de gestion des droits d’utilisation](/graph/tutorial-access-package-api?view=graph-rest-beta).

## <a name="prerequisites"></a>Prérequis

Pour utiliser la gestion des droits d’utilisation Azure AD, vous devez disposer d’une des licences suivantes :

- Azure AD Premium P2
- Licence Enterprise Mobility + Security (EMS) E5

Pour plus d’informations, consultez [Exigences des licences](entitlement-management-overview.md#license-requirements).

## <a name="step-1-set-up-users-and-group"></a>Étape 1 : Configurer des utilisateurs et un groupe

Un répertoire de ressources comprend une ou plusieurs ressources à partager. Dans cette étape, vous créez un groupe nommé **Marketing resources** dans l’annuaire Woodgrove Bank qui constitue la ressource cible pour la gestion des droits d’utilisation. Vous configurez également un demandeur interne.

**Rôle prérequis :** Administrateur général ou Administrateur d’utilisateurs

![Créer des utilisateurs et des groupes](./media/entitlement-management-access-package-first/elm-users-groups.png)

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’Administrateur général ou Administrateur d’utilisateurs.  

1. Dans le volet de navigation à gauche, cliquez sur **Azure Active Directory**.

1. Créez ou configurez les deux utilisateurs suivants. Vous pouvez utiliser ces noms ou des noms différents. **Admin1** peut être l’utilisateur avec lequel vous êtes actuellement connecté.

    | Nom | Rôle d’annuaire |
    | --- | --- |
    | **Admin1** | Administrateur général<br/>-ou-<br/>Administrateur d’utilisateurs |
    | **Requestor1** | Utilisateur |

1. Créez un groupe de sécurité Azure AD nommé **Marketing resources** avec le type d’appartenance **Affecté**.

    Ce groupe sera la ressource cible pour la gestion des droits d’utilisation. Le groupe ne doit contenir aucun membre pour commencer.

## <a name="step-2-create-an-access-package"></a>Étape 2 : Créer un package d’accès

Un *package d’accès* est un bundle de ressources dont une équipe ou un projet a besoin et qui est régi par des stratégies. Les packages d’accès sont définis dans des conteneurs appelés *catalogues*. Dans cette étape, vous allez créer un package d’accès **Marketing Campaign** dans le catalogue **Général**.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

![Créer un package d’accès](./media/entitlement-management-access-package-first/elm-access-package.png)

1. Dans le portail Azure, dans le volet de navigation gauche, cliquez sur **Azure Active Directory**.

2. Dans le menu gauche, cliquez sur **Identity Governance**.

3. Dans le menu gauche, cliquez sur **Packages d’accès**.  Si vous voyez **Accès refusé**, vérifiez qu’une licence Azure AD Premium P2 est présente dans votre annuaire.

4. Cliquez sur **Nouveau package d’accès**.

    ![Gestion des droits d’utilisation dans le portail Azure](./media/entitlement-management-shared/access-packages-list.png)

5. Sous l’onglet **Bases**, tapez le nom de package d’accès **Marketing Campaign** (Campagne marketing) et la description **Access to resources for the campaign** (Accès aux ressources de la campagne).

6. Dans la liste déroulante **Catalogue**, conservez la sélection **Général**.

    ![Nouveau package d’accès - Onglet De base](./media/entitlement-management-access-package-first/basics.png)

7. Cliquez sur **Suivant** pour ouvrir l’onglet **Rôles des ressources**.

    Sous cet onglet, sélectionnez les ressources et le rôle de ressource à inclure dans le package d’accès.

8. Cliquez sur **Groupes et équipes**.

9. Dans le volet Sélectionner les groupes, recherchez le groupe **Marketing resources** créé précédemment et sélectionnez-le.

     Par défaut, des groupes sont présents à l’intérieur du catalogue Général. Quand vous sélectionnez un groupe à l’extérieur du catalogue Général (vous le voyez si vous cochez la case de l’option **Tout afficher**), il est ajouté au catalogue Général.

    ![Capture d’écran montrant l’onglet « Nouveau package d’accès - Rôles des ressources » et la fenêtre « Sélectionner des groupes ».](./media/entitlement-management-access-package-first/resource-roles-select-groups.png)

10. Cliquez sur **Sélectionner** pour ajouter le groupe à la liste.

11. Dans la liste déroulante **Rôle**, sélectionnez **Membre**.

    ![Nouveau package d’accès - Onglet Rôles des ressources](./media/entitlement-management-access-package-first/resource-roles.png)

    >[!IMPORTANT]
    >Les groupes avec attribution de rôle ajoutés à un package d’accès sont indiqués à l’aide du sous-type **Attribuables aux rôles**. Pour plus d’informations sur les groupes attribuables aux rôles Azure AD, reportez-vous à [Créer un groupe avec attribution de rôle](../roles/groups-create-eligible.md) dans Azure Active Directory. Si vous ne voyez aucun groupe avec attribution de rôle à ajouter ou si vous ne pouvez pas l’ajouter, vérifiez que vous possédez le rôle d’annuaire Azure AD et le rôle de gestion des droits d’utilisation requis. Vous devrez peut-être demander à un utilisateur doté des rôles requis d’ajouter la ressource à votre catalogue. Pour plus d’informations, consultez [Rôles requis pour ajouter des ressources à un catalogue](entitlement-management-delegate.md#required-roles-to-add-resources-to-a-catalog).

    >[!NOTE]
    > Quand vous utilisez des [groupes dynamiques](../enterprise-users/groups-create-rule.md), vous ne voyez pas d’autres rôles disponibles en plus du propriétaire. C'est la procédure normale.
    > ![Vue d’ensemble du scénario](./media/entitlement-management-access-package-first/dynamic-group-warning.png)
    


12. Cliquez sur **Suivant** pour ouvrir l’onglet **Requêtes**.

    Sous cet onglet, vous créez une stratégie de demande. Une *stratégie* définit les règles ou barrières mises en place pour accéder à un package d’accès. Vous créez une stratégie qui permet à un utilisateur spécifique de l’annuaire de ressources de demander ce package d’accès.

13. Dans la section **Utilisateurs qui peuvent demander l’accès**, cliquez sur **Pour les utilisateurs de votre annuaire**, puis cliquez sur **Utilisateurs et groupes spécifiques**.

    ![Nouveau package d’accès - Onglet Demandes](./media/entitlement-management-access-package-first/requests.png)

14. Cliquez sur **Ajouter des utilisateurs et des groupes**.

15. Dans le volet Sélectionner des utilisateurs et des groupes, sélectionnez l’utilisateur **Requestor1** que vous avez créé.

    ![Nouveau package d’accès - Onglet Demandes - Sélectionner des utilisateurs et des groupes](./media/entitlement-management-access-package-first/requests-select-users-groups.png)

16. Cliquez sur **Sélectionner**.

17. Faites défiler jusqu’aux sections **Approbation** et **Activer les demandes**.

18. Laissez l’option **Exiger l’approbation** définie sur **Non**.

19. Pour **Activer les demandes**, cliquez sur **Oui** afin que ce package d’accès puisse être demandé dès qu’il est créé.

    ![Nouveau package d’accès - Onglet Demandes - Approbation et Activer les demandes](./media/entitlement-management-access-package-first/requests-approval-enable.png)

20. Cliquez sur **Suivant** pour ouvrir l’onglet **Cycle de vie**.

21. Dans la section **Expiration**, sélectionnez **Nombre de jours** en regard de **Les attributions de package d’accès expirent**.

22. Définissez **Les attributions expirent après** sur **30** jours.

    ![Nouveau package d’accès - Onglet Cycle de vie](./media/entitlement-management-access-package-first/lifecycle.png)

23. Cliquez sur **Suivant** pour ouvrir l’onglet **Vérifier + créer**.

    ![Nouveau package d’accès - Onglet Vérifier + créer](./media/entitlement-management-access-package-first/review-create.png)

    Après quelques instants, vous devez obtenir une notification indiquant que le package d’accès a été créé.

24. Dans le menu gauche du package de campagne marketing, cliquez sur **Vue d’ensemble**.

25. Copiez le lien sous **Lien du portail Mon Accès**.

    Vous en aurez besoin à l’étape suivante.

    ![Vue d’ensemble du package d’accès - Lien du portail Mon Accès](./media/entitlement-management-shared/my-access-portal-link.png)

## <a name="step-3-request-access"></a>Étape 3 : Demander l'accès

Dans cette étape, vous demandez l’accès au package d’accès en tant que **demandeur interne**. Les demandeurs envoient leurs demandes à l’aide d’un site appelé « portail Mon Accès ». Le portail Mon Accès permet aux demandeurs d’envoyer des demandes de packages d’accès, de voir ceux auxquels ils ont déjà accès et de consulter l’historique de leurs demandes.

**Rôle prérequis :** Demandeur interne

1. Déconnectez-vous du portail Azure.

1. Dans une nouvelle fenêtre de navigateur, accédez au lien du portail Mon Accès que vous avez copié à l’étape précédente.

1. Connectez-vous au portail Mon Accès en tant que **Requestor1**.

    Vous devez voir le package d’accès **Marketing Campaign**.

1. Si nécessaire, dans la colonne **Description**, cliquez sur la flèche pour afficher des détails sur le package d’accès.

    ![Portail Mon Accès - Packages d’accès](./media/entitlement-management-shared/my-access-access-packages.png)

1. Cochez la case pour sélectionner le package.

1. Cliquez sur **Demander l’accès** pour ouvrir le volet Demander l’accès.

    ![Portail Mon Accès - Bouton Demander l’accès](./media/entitlement-management-access-package-first/my-access-request-access-button.png)

1. Dans la zone **Justification professionnelle**, tapez la justification **I am working on the new marketing campaign** (Je participe à la nouvelle campagne marketing).

    ![Portail Mon Accès - Demander l’accès](./media/entitlement-management-shared/my-access-request-access.png)

1. Cliquez sur **Envoyer**.

1. Dans le menu gauche, cliquez sur **Historique des demandes** pour vérifier que votre demande a été envoyée.

## <a name="step-4-validate-that-access-has-been-assigned"></a>Étape 4 : Valider que l’accès a été affecté

Dans cette étape, vous allez vérifier que le package d’accès a été affecté au **demandeur interne** et qu’il est désormais membre du groupe **Marketing resources**.

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Déconnectez-vous du portail Mon Accès.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’**Admin1**.

1. Cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu gauche, cliquez sur **Packages d’accès**.

1. Recherchez le package d’accès **Marketing Campaign** et cliquez dessus.

1. Dans le menu gauche, cliquez sur **Demandes**.

    Vous devriez voir Requestor1 et la stratégie initiale avec l’état **Remis**.

1. Cliquez sur la demande pour l’examiner en détail.

    ![Package d’accès - Détails de la demande](./media/entitlement-management-access-package-first/request-details.png)

1. Dans le volet de navigation à gauche, cliquez sur **Azure Active Directory**.

1. Cliquez sur **Groupes** et ouvrez le groupe **Marketing resources**.

1. Cliquez sur **Membres**.

    **Requestor1** doit être listé comme membre.

    ![Membre du groupe Marketing resources](./media/entitlement-management-access-package-first/group-members.png)

## <a name="step-5-clean-up-resources"></a>Étape 5 : Nettoyer les ressources

Dans cette étape, vous supprimez les modifications apportées et le package d’accès **Marketing Campaign**.

**Rôle prérequis :**  Administrateur général ou Administrateur d’utilisateurs

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Ouvrez le package d’accès **Marketing Campaign**.

1. Cliquez sur **Affectations**.

1. Pour **Requestor1**, cliquez sur le bouton de sélection ( **...** ), puis sur **Supprimer l’accès**. Dans le message qui s’affiche, cliquez sur **Oui**.

    Après quelques instants, l’état passe de Remis à Expiré.

1. Cliquez sur **Rôles des ressources**.

1. Pour **Marketing resources**, cliquez sur le bouton de sélection ( **...** ), puis sur **Supprimer le rôle de ressource**. Dans le message qui s’affiche, cliquez sur **Oui**.

1. Ouvrez la liste des packages d’accès.

1. Pour **Marketing Campaign**, cliquez sur le bouton de sélection ( **...** ), puis sur **Supprimer**. Dans le message qui s’affiche, cliquez sur **Oui**.

1. Dans Azure Active Directory, supprimez tous les utilisateurs que vous avez créés comme **Requestor1** et **Admin1**.

1. Supprimez le groupe **Marketing resources**.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour en savoir plus sur les étapes de scénarios courants dans la gestion des droits d’utilisation.
> [!div class="nextstepaction"]
> [Scénarios courants](entitlement-management-scenarios.md)