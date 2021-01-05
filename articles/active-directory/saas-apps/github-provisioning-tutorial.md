---
title: 'Tutoriel : Approvisionnement d’utilisateurs pour GitHub - Azure AD'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner automatiquement des comptes utilisateur sur GitHub.
services: active-directory
author: Zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/21/2020
ms.author: Zhchia
ms.openlocfilehash: f1600dfc5705ca97f16e8966a796b54fc556d216
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359260"
---
# <a name="tutorial-configure-github-for-automatic-user-provisioning"></a>Tutoriel : Configurer GitHub pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de vous indiquer les étapes à suivre dans GitHub et Azure AD pour approvisionner et déprovisionner automatiquement des comptes utilisateur d’Azure AD vers GitHub.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un locataire Azure Active Directory
* Une organisation GitHub créée dans [GitHub Enterprise Cloud](https://help.github.com/articles/github-s-products/#github-enterprise), ce qui nécessite le [plan de facturation GitHub Enterprise](https://help.github.com/articles/github-s-billing-plans/#billing-plans-for-organizations)
* Un compte d’utilisateur dans GitHub avec des autorisations d’administrateur vers l’organisation
* [SAML configuré pour l’organisation GitHub Enterprise Cloud](./github-tutorial.md)
* Assurez-vous que l’accès OAuth a été fourni pour votre organisation, comme décrit [ici](https://help.github.com/en/github/setting-up-and-managing-organizations-and-teams/approving-oauth-apps-for-your-organization)
* Le provisionnement SCIM pour une seule organisation est pris en charge uniquement lorsque l’authentification unique est activée au niveau de l’organisation

> [!NOTE]
> L’intégration de l’approvisionnement Azure AD s’appuie sur l’[API GitHub SCIM](https://developer.github.com/v3/scim/) disponible pour les clients [GitHub Enterprise Cloud](https://help.github.com/articles/github-s-products/#github-enterprise) dans le [plan de facturation GitHub Enterprise](https://help.github.com/articles/github-s-billing-plans/#billing-plans-for-organizations).

## <a name="assigning-users-to-github"></a>Attribution d’utilisateurs à GitHub

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique des comptes d’utilisateur, seuls les utilisateurs et les groupes qui ont été « affectés » à une application dans Azure AD sont synchronisés. 

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD représentent les utilisateurs qui ont besoin d’accéder à votre application GitHub. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs à votre application GitHub en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-github"></a>Conseils importants pour l’attribution d’utilisateurs à GitHub

* Nous vous recommandons de n’assigner qu’un seul utilisateur Azure AD à GitHub afin de tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à GitHub, vous devez sélectionner le rôle **Utilisateur** ou un autre rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Le rôle **Accès par défaut** ne fonctionne pas pour l’approvisionnement et ces utilisateurs sont ignorés.

## <a name="configuring-user-provisioning-to-github"></a>Configuration de l'approvisionnement des utilisateurs sur GitHub

Cette section explique comment connecter votre Azure AD à l’API d’approvisionnement de comptes d’utilisateur de GitHub pour créer, mettre à jour et désactiver les comptes d’utilisateur attribués dans GitHub en fonction des attributions d’utilisateurs et de groupes dans Azure AD.

### <a name="configure-automatic-user-account-provisioning-to-github-in-azure-ad"></a>Configurer l’approvisionnement automatique de comptes utilisateurs vers GitHub dans Azure AD

1. Dans le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

2. Si vous avez déjà configuré GitHub pour l’authentification unique, recherchez votre instance de GitHub à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **GitHub** dans la galerie d’applications. Sélectionnez GitHub dans les résultats de recherche et ajoutez-le à votre liste d’applications.

3. Sélectionnez votre instance de GitHub, puis sélectionnez l’onglet **Approvisionnement**.

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Approvisionnement GitHub](./media/github-provisioning-tutorial/GitHub1.png)

5. Sous la section **informations d’identification de l’administrateur** , cliquez sur **Autoriser**. Cette opération ouvre une boîte de dialogue d’autorisation GitHub dans une nouvelle fenêtre du navigateur. Notez que vous devez vous assurer d’être approuvé pour autoriser l’accès. Suivez les instructions décrites [ici](https://help.github.com/github/setting-up-and-managing-organizations-and-teams/approving-oauth-apps-for-your-organization).

6. Dans la nouvelle fenêtre, connectez-vous à GitHub à l’aide de votre compte d’administrateur. Dans la boîte de dialogue d’autorisation qui s’affiche, sélectionnez l’équipe GitHub pour laquelle vous souhaitez activer l’approvisionnement, puis sélectionnez **Autoriser**. Une fois cela terminé, revenez au portail Azure pour terminer la configuration de l’approvisionnement.

    ![Capture d’écran représentant la page de connexion pour GitHub.](./media/github-provisioning-tutorial/GitHub2.png)

7. Dans le portail Azure, entrez **URL cliente** et cliquez sur **Connexion test** pour vous assurer qu’Azure AD peut se connecter à votre application GitHub. En cas d’échec de connexion, vérifiez que votre compte GitHub dispose des autorisations Administrateur et que **l’URL de locataire** est correctement entrée. Ensuite, recommencez l’étape « Autoriser » (vous pouvez constituer **l’URL de locataire** par règle : `https://api.github.com/scim/v2/organizations/<Organization_name>`, et rechercher vos organisations sous votre compte GitHub : **Paramètres** > **Organisations** ).

    ![Capture d’écran représentant la page Organisations dans GitHub.](./media/github-provisioning-tutorial/GitHub3.png)

8. Entrez l’adresse de courrier d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement dans le champ **E-mail de notification** , puis cochez la case « Envoyer une notification par e-mail en cas de défaillance ».

9. Cliquez sur **Enregistrer**.

10. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec GitHub**.

11. Dans la section **Mappages des attributs** , passez en revue les attributs d’utilisateur qui sont synchronisés d’Azure AD vers GitHub. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans GitHub pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

12. Pour activer le service d’approvisionnement Azure AD pour GitHub, définissez **l’état d’approvisionnement** sur **Activé** dans la section **Paramètres**

13. Cliquez sur **Enregistrer**.

Cette opération démarre la synchronisation initiale des utilisateurs et/ou des groupes attribués à GitHub dans la section Utilisateurs et Groupes. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)