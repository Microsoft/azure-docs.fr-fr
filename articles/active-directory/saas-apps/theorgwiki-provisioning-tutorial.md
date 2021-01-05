---
title: 'Tutoriel : Configurer TheOrgWiki pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et désapprovisionner automatiquement des comptes d’utilisateur dans TheOrgWiki.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/26/2019
ms.author: zhchia
ms.openlocfilehash: 8238b9902aafcabc079c551a0eabc7170042209a
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357621"
---
# <a name="tutorial-configure-theorgwiki-for-automatic-user-provisioning"></a>Tutoriel : Configurer TheOrgWiki pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans TheOrgWiki et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le retrait automatiques d’utilisateurs et/ou de groupes sur TheOrgWiki.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire OrgWiki](https://www.theorgwiki.com/welcome/).
* Un compte d’utilisateur dans TheOrgWiki avec des autorisations d’administrateur.

## <a name="assign-users-to-theorgwiki"></a>Affecter des utilisateurs à TheOrgWiki

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à TheOrgWiki. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs ou groupes à TheOrgWiki en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-theorgwiki"></a>Conseils importants pour l’affectation d’utilisateurs à TheOrgWiki

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à TheOrgWiki afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à TheOrgWiki, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-theorgwiki-for-provisioning"></a>Configurer TheOrgWiki pour l’approvisionnement

Avant de configurer TheOrgWiki pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer l’approvisionnement SCIM sur TheOrgWiki.

1. Connectez-vous à votre [console d’administration TheOrgWiki](https://www.theorgwiki.com/login/). Cliquez sur **Console d’administration**.

    ![Capture d’écran du site d’OrgWiki avec l’avatar de l’utilisateur et la console d’administration en évidence.](media/theorgwiki-provisioning-tutorial/login.png)

2. Dans la console d’administration, cliquez sur **l’onglet Paramètres**. 

    ![Capture d’écran de la console d’administration d’OrgWiki avec l’onglet Paramètres en évidence.](media/theorgwiki-provisioning-tutorial/settings.png)
    
3. Accédez à **Comptes de service**.

    ![Capture d’écran de la page Comptes de service dans la console d’administration d’OrgWiki.](media/theorgwiki-provisioning-tutorial/serviceaccount.png)

4. Cliquez sur **+ Compte de service**. Sous **Type de compte de service** , sélectionnez **Basé sur un jeton**. Cliquez sur **Enregistrer**.

    ![Capture d’écran de la boîte de dialogue Nouveau compte de service avec les options Type de compte de service, Basé sur un jeton et Enregistrer en évidence.](media/theorgwiki-provisioning-tutorial/auth.png)

5.  Copiez les **Jetons actifs**. Vous devrez entrer cette valeur dans le champ Jeton secret dans l’onglet Approvisionnement de votre application TheOrgWiki dans le portail Microsoft Azure.
     
    ![Capture d’écran de la boîte de dialogue Gérer les jetons pour l’approvisionnement SCIM.](media/theorgwiki-provisioning-tutorial/token.png)

## <a name="add-theorgwiki-from-the-gallery"></a>Ajouter TheOrgWiki à partir de la galerie

Avant de configurer TheOrgWiki pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter TheOrgWiki à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **TheOrgWiki** , puis sélectionnez **TheOrgWiki** dans le volet de résultats. 

    ![TheOrgWiki dans la liste des résultats](common/search-new-app.png)

5. Sélectionnez le bouton **S’inscrire à TheOrgWiki** qui vous redirigera vers la page de connexion de TheOrgWiki. 

    ![Capture d’écran de la page de connexion d’OrgWiki avec l’URL en évidence.](media/theorgwiki-provisioning-tutorial/image00.png)

6.  Dans le coin supérieur droit, sélectionnez **Connexion**.

    ![Capture d’écran de l’angle supérieur droit de la page de connexion avec l’option Connexion en évidence.](media/theorgwiki-provisioning-tutorial/image02.png)

7. Comme TheOrgWiki est une application OpenIDConnect, choisissez de vous connecter à OrgWiki à l’aide de votre compte professionnel Microsoft.

    ![Capture d’écran de la page de connexion d’OrgWiki avec l’option Se connecter avec Microsoft en évidence.](media/theorgwiki-provisioning-tutorial/image03.png)
    
8. Après une authentification réussie, l’application est automatiquement ajoutée à votre locataire et vous êtes redirigé vers votre compte TheOrgWiki.

    ![Ajouter SCIM OrgWiki](media/theorgwiki-provisioning-tutorial/image04.png)

## <a name="configure-automatic-user-provisioning-to-theorgwiki"></a>Configurer l'approvisionnement automatique d'utilisateurs sur TheOrgWiki 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs ou des groupes dans TheOrgWiki en fonction des attributions d’utilisateurs ou de groupes dans Azure AD.


### <a name="to-configure-automatic-user-provisioning-for-theorgwiki-in-azure-ad"></a>Pour configurer l’attribution automatique d’utilisateurs pour TheOrgWiki dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **TheOrgWiki**.

    ![Lien OrgWiki dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez `https://<TheOrgWiki Subdomain        value>.theorgwiki.com/api/v2/scim/v2/` dans **URL de locataire**. 

    Exemple : `https://test1.theorgwiki.com/api/v2/scim/v2/`

> [!NOTE]
> La **Valeur de sous-domaine** ne peut être définie qu’au cours du processus d’inscription initial à TheOrgWiki.
 
6. Entrez la valeur de jeton dans le champ **Jeton secret** , récupérée précédemment sur TheOrgWiki. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à TheOrgWiki. Si la connexion échoue, vérifiez que votre compte TheOrgWiki dispose des autorisations d’administrateur, puis réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

7. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

8. Cliquez sur **Enregistrer**.

9. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur TheOrgWiki**.

    ![Mappages utilisateur TheOrgWiki](media/theorgwiki-provisioning-tutorial/usermapping.png)

10. Dans la section **Mappage d’attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et TheOrgWiki. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans TheOrgWiki pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur TheOrgWiki](media/theorgwiki-provisioning-tutorial/userattribute.png).

11. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Pour activer le service d’approvisionnement Azure AD pour TheOrgWiki, affectez au paramètre **Statut d’approvisionnement** la valeur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

13. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur OrgWiki en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

14. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur la durée d’approvisionnement des utilisateurs et/ou des groupes, voir [Combien de temps faut-il pour approvisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD sur TheOrgWiki. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md).
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md).