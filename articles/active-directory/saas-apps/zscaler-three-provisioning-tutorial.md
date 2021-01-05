---
title: 'Tutoriel : Configurer Zscaler Three pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Ce tutoriel montre comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur dans Zscaler Three.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 8526556c00327df7e933130e7a54d23eade19f8a
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92519705"
---
# <a name="tutorial-configure-zscaler-three-for-automatic-user-provisioning"></a>Tutoriel : Configurer Zscaler Three pour le provisionnement automatique d’utilisateurs

Ce tutoriel montre comment configurer Azure Active Directory (Azure AD) pour provisionner et déprovisionner automatiquement des utilisateurs et/ou des groupes dans Zscaler Three.

> [!NOTE]
> Ce tutoriel décrit un connecteur reposant sur le service de provisionnement d’utilisateurs Azure AD. Pour obtenir des informations importantes sur l’utilisation et le fonctionnement de ce service, ainsi que des réponses aux questions fréquentes, consultez [Automatiser le provisionnement et le déprovisionnement d’utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes décrites dans ce tutoriel, vous avez besoin des éléments suivants :

* Un locataire Azure AD.
* Un locataire Zscaler Three.
* Un compte d’utilisateur Zscaler Three ayant des autorisations d’administrateur.

> [!NOTE]
> L’intégration du provisionnement Azure AD repose sur l’API SCIM Zscaler ZSCloud, qui est disponible pour les comptes entreprise.

## <a name="adding-zscaler-three-from-the-gallery"></a>Ajouter Zscaler Three à partir de la galerie

Avant de configurer Zscaler Three pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Zscaler Three à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

Dans le [portail Azure](https://portal.azure.com), dans le volet de gauche, sélectionnez **Azure Active Directory**  :

![Sélectionnez Azure Active Directory](common/select-azuread.png)

Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**  :

![Applications d’entreprise](common/enterprise-applications.png)

Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la fenêtre :

![Sélectionner la nouvelle application](common/add-new-app.png)

Dans la zone de recherche, entrez **Zscaler Three** . Sélectionnez **Zscaler Three** dans les résultats, puis sélectionnez **Ajouter** .

![Liste des résultats](common/search-new-app.png)

## <a name="assign-users-to-zscaler-three"></a>Attribuer des utilisateurs à Zscaler Three

Les utilisateurs Azure AD doivent être autorisés à accéder aux applications sélectionnées pour pouvoir les utiliser. Dans le cadre du provisionnement automatique d’utilisateurs, seuls les utilisateurs ou groupes attribués à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Zscaler Three. Après cela, vous pouvez attribuer ces utilisateurs et groupes à l’application Zscaler Three en suivant les instructions fournies dans [Attribuer un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-zscaler-three"></a>Conseils importants pour l’attribution d’utilisateurs à Zscaler Three

* Nous avons recommandé d’attribuer au début un seul utilisateur Azure AD à Zscaler Three afin de tester la configuration du provisionnement automatique d’utilisateurs. Vous pourrez attribuer des utilisateurs et groupes supplémentaires ultérieurement.

* Quand vous attribuez un utilisateur à Zscaler Three, vous devez sélectionner un rôle d’application valide (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-automatic-user-provisioning"></a>Configurer le provisionnement d’utilisateurs automatique

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et des groupes dans Zscaler Three en fonction des attributions d’utilisateurs et de groupes définies dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Zscaler Three. Le cas échéant, suivez les instructions fournies dans le [tutoriel sur la l’authentification unique pour Zscaler Three](zscaler-three-tutorial.md). L’authentification unique peut être configurée indépendamment du provisionnement automatique d’utilisateurs, mais ces deux fonctionnalités sont complémentaires.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Zscaler Three**  :

    ![Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Zscaler Three**  :

    ![Liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Provisioning** (Provisionnement) :

    ![Configuration de Zscaler Three](./media/zscaler-three-provisioning-tutorial/provisioning-tab.png)

4. Définissez **Provisioning Mode** (Mode de provisionnement) sur **Automatic** (Automatique) :

    ![Définir le mode de provisionnement](./media/zscaler-three-provisioning-tutorial/provisioning-credentials.png)

5. Dans la section **Admin Credentials** (Informations d’identification d’administrateur), entrez les valeurs **Tenant URL** (URL de locataire) et **Secret Token** (Jeton secret) de votre compte Zscaler Three, comme cela est décrit dans l’étape suivante.

6. Pour obtenir les valeurs **Tenant URL** (URL de locataire) et **Secret Token** (Jeton secret), accédez à **Administration** > **Authentication Settings** (Paramètres d’authentification) dans le portail Zscaler Three, puis sélectionnez **SAML** sous **Authentication Type** (Type d’authentification) :

    ![Paramètres d’authentification de Zscaler Three](./media/zscaler-three-provisioning-tutorial/secret-token-1.png)

    Sélectionnez **Configure SAML** (Configurer SAML) pour ouvrir la fenêtre **Configure SAML**  :

    ![Fenêtre Configure SAML (Configurer SAML)](./media/zscaler-three-provisioning-tutorial/secret-token-2.png)

    Sélectionnez **Enable SCIM-Based Provisioning** (Activer le provisionnement SCIM), copiez les valeurs **Base URL** (URL de base) et **Bearer Token** (Jeton du porteur), puis enregistrez les paramètres. Dans le portail Azure, collez les valeurs **URL de base** dans le champ **URL de locataire** et la valeur **Jeton du porteur** dans le champ **Jeton secret** .

7. Après avoir entré les valeurs dans les champs **URL de locataire** et **Jeton secret** , sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Zscaler Three. Si la connexion échoue, vérifiez que votre compte Zscaler Three a les autorisations d’administrateur et réessayez.

    ![Tester la connexion](./media/zscaler-three-provisioning-tutorial/test-connection.png)

8. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur de provisionnement. Sélectionnez **Envoyer une notification par e-mail en cas de défaillance**  :

    ![Configurer l’e-mail de notification](./media/zscaler-three-provisioning-tutorial/notification.png)

9. Sélectionnez **Enregistrer** .

10. Dans la section **Mappages** , sélectionnez **Synchronize Azure Active Directory Users to Zscaler Three** (Synchroniser les utilisateurs Azure Active Directory avec Zscaler Three) :

    ![Synchroniser les utilisateurs Azure AD](./media/zscaler-three-provisioning-tutorial/user-mappings.png)

11. Dans la section **Mappages des attributs** , passez en revue les attributs d’utilisateur qui sont synchronisés entre Azure AD et Zscaler Three. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondance des comptes d’utilisateur dans Zscaler Three dans le cadre des opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappages des attributs avec sept mappages affichés.](./media/zscaler-three-provisioning-tutorial/user-attribute-mappings.png)

12. Dans la section **Mappages** , sélectionnez **Synchronize Azure Active Directory Groups to Zscaler Three** (Synchroniser les groupes Azure Active Directory avec Zscaler Three) :

    ![Synchroniser les groupes Azure AD](./media/zscaler-three-provisioning-tutorial/group-mappings.png)

13. Dans la section **Mappages des attributs** , passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Zscaler Three. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondance des groupes dans Zscaler Three dans le cadre des opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappages des attributs avec trois mappages affichés.](./media/zscaler-three-provisioning-tutorial/group-attribute-mappings.png)

14. Pour configurer des filtres d’étendue, reportez-vous aux instructions fournies dans le [tutoriel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Pour activer le service de provisionnement Azure AD pour Zscaler Three, définissez le paramètre **État du provisionnement** sur **Activé** dans la section **Paramètres**  :

    ![État de l’approvisionnement](./media/zscaler-three-provisioning-tutorial/provisioning-status.png)

16. Définissez quels utilisateurs et/ou groupes vous voulez provisionner dans Zscaler Three en choisissant les valeurs appropriées sous **Étendue** dans la section **Paramètres**  :

    ![Valeurs d’étendue](./media/zscaler-three-provisioning-tutorial/scoping.png)

17. Quand vous êtes prêt à effectuer le provisionnement, sélectionnez **Enregistrer**  :

    ![Sélectionner Enregistrer](./media/zscaler-three-provisioning-tutorial/save-provisioning.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et groupes définis sous **Étendue** dans la section **Paramètres** . La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui ont lieu toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez surveiller la progression dans la section **Détails de la synchronisation** . Vous pouvez également suivre les liens fournis pour accéder à un rapport d’activité de provisionnement. Ce rapport décrit toutes les actions effectuées par le service de provisionnement Azure AD dans Zscaler Three.

Pour avoir des informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-03.png