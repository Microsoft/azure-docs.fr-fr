---
title: 'Tutoriel : Configurer Wrike pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur sur Wrike.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/26/2019
ms.author: Zhchia
ms.openlocfilehash: 53b1db1a8c4da59055c0af5f448fa0c8a6933daf
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95988091"
---
# <a name="tutorial-configure-wrike-for-automatic-user-provisioning"></a>Tutoriel : Configurer Wrike pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Wrike et Azure Active Directory (Azure AD) pour configurer Azure AD en vue de provisionner et déprovisionner des utilisateurs ou des groupes dans Wrike.

> [!NOTE]
> Ce tutoriel décrit un connecteur reposant sur le service de provisionnement d’utilisateurs Azure AD. Pour en savoir plus sur l’objet et le fonctionnement de ce service et pour accéder aux questions fréquentes (FAQ), consultez [Automatiser le provisionnement et le déprovisionnement des utilisateurs pour les applications SaaS (software as a service) avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d'évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Wrike](https://www.wrike.com/price/)
* Un compte d’utilisateur dans Wrike avec des autorisations d’administrateur

## <a name="assign-users-to-wrike"></a>Attribuer des utilisateurs à Wrike
Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs ou groupes affectés à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, identifiez les utilisateurs ou groupes dans Azure AD qui doivent accéder à Wrike. Ensuite, affectez ces utilisateurs ou groupes à Wrike en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-wrike"></a>Conseils importants pour l’attribution d’utilisateurs à Wrike

* Nous vous recommandons d’attribuer un seul utilisateur Azure AD à Wrike afin de tester la configuration du provisionnement d’utilisateurs automatique. Les autres utilisateurs ou groupes pourront être affectés ultérieurement.

* Lorsque vous affectez un utilisateur à Wrike, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est Accès par défaut sont exclus de l’approvisionnement.

## <a name="set-up-wrike-for-provisioning"></a>Configurer Wrike pour l’approvisionnement

Avant de configurer Wrike pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez activer le provisionnement SCIM (System for Cross-domain Identity Management) dans Wrike.

1. Connectez-vous à votre [console d’administration Wrike](https://www.Wrike.com/login/). Accédez à votre ID de locataire. Sélectionnez **Applications et intégrations**.

    ![Applications et intégrations](media/Wrike-provisioning-tutorial/admin.png)

2.  Accédez à **Azure AD** et sélectionnez-le.

    ![Azure AD](media/Wrike-provisioning-tutorial/Capture01.png)

3.  Sélectionnez SCIM. Copier l’**URL de base**.

    ![URL de base](media/Wrike-provisioning-tutorial/Wrike-tenanturl.png)

4. Sélectionnez **API** > **Azure SCIM**.

    ![Azure SCIM](media/Wrike-provisioning-tutorial/Wrike-add-scim.png)

5.  Une fenêtre contextuelle s’ouvre. Entrez le même mot de passe que celui que vous avez défini auparavant pour créer un compte.

    ![Création de jeton Wrike](media/Wrike-provisioning-tutorial/password.png)

6.  Copiez le **jeton secret** et collez-le dans Azure AD. Sélectionnez **Enregistrer** pour terminer la configuration du provisionnement sur Wrike.

    ![Jeton d’accès permanent](media/Wrike-provisioning-tutorial/Wrike-create-token.png)


## <a name="add-wrike-from-the-gallery"></a>Ajouter Wrike à partir de la galerie

Avant de configurer Wrike pour le provisionnement automatique d’utilisateurs avec Azure AD, ajoutez Wrike à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

Pour ajouter Wrike à partir de la galerie d’applications Azure AD, procédez comme suit.

1. Dans le panneau de navigation gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Wrike**, sélectionnez **Wrike** dans le volet de résultats, puis sélectionnez **Ajouter** pour ajouter l’application.

    ![Wrike dans la liste des résultats](common/search-new-app.png)


## <a name="configure-automatic-user-provisioning-to-wrike"></a>Configurer le provisionnement d’utilisateurs automatique sur Wrike 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs ou groupes dans Wrike en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

> [!TIP]
> Pour activer l’authentification unique basée sur SAML pour Wrike, suivez les instructions fournies dans le [tutoriel sur l’authentification unique Wrike](wrike-tutorial.md). La configuration de l’authentification unique est indépendante de celle de l’approvisionnement automatique d’utilisateurs, même si ces deux fonctionnalités se complètent.

### <a name="configure-automatic-user-provisioning-for-wrike-in-azure-ad"></a>Configurer le provisionnement automatique d’utilisateurs pour Wrike dans Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** > **Toutes les applications**.

    ![Toutes les applications](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Wrike**.

    ![Lien Wrike dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Onglet Approvisionnement](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Mode de provisionnement défini sur Automatique](common/provisioning-automatic.png)

5. Dans la section Informations d’identification de l’administrateur, entrez respectivement les valeurs de l’**URL de base** et du **jeton d’accès permanent** récupérées précédemment dans les champs **URL de locataire** et **Jeton secret**. Sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Wrike. Si la connexion échoue, vérifiez que votre compte Wrike dispose d’autorisations d’administration, puis réessayez.

    ![URL de locataire + jeton](common/provisioning-testconnection-tenanturltoken.png)

7. Dans la zone **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur de provisionnement. Cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

8. Sélectionnez **Enregistrer**.

9. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur Wrike**.

    ![Mappages d’utilisateurs Wrike](media/Wrike-provisioning-tutorial/Wrike-user-mappings.png)

10. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Wrike. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes utilisateur dans Wrike pour les opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Wrike](media/Wrike-provisioning-tutorial/Wrike-user-attributes.png)

11. Pour configurer des filtres d’étendue, suivez les instructions fournies dans le [tutoriel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Pour activer le service d’approvisionnement Azure AD pour Wrike, définissez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

13. Définissez les utilisateurs ou groupes à provisionner dans Wrike en choisissant les valeurs souhaitées dans **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

14. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Enregistrement de la configuration du provisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs ou groupes définis sous **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur la durée de provisionnement des utilisateurs ou groupes, consultez [Combien de temps faut-il pour provisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users)

Vous pouvez utiliser la section **État actuel** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD sur Wrike. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)