---
title: 'Tutoriel : Configurer Visitly pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur sur Visitly.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/30/2019
ms.author: Zhchia
ms.openlocfilehash: ff3f3ab65df2d801b7c962de7cce645e9fc00b30
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358607"
---
# <a name="tutorial-configure-visitly-for-automatic-user-provisioning"></a>Tutoriel : Configurer Visitly pour le provisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Visitly et Azure Active Directory (Azure AD) pour configurer Azure AD en vue de provisionner et déprovisionner des utilisateurs ou des groupes dans Visitly.

> [!NOTE]
> Ce tutoriel décrit un connecteur reposant sur le service de provisionnement d’utilisateurs Azure AD. Pour en savoir plus sur l’objet et le fonctionnement de ce service et pour accéder aux questions fréquentes (FAQ), consultez [Automatiser le provisionnement et le déprovisionnement des utilisateurs pour les applications SaaS (software as a service) avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d'évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Visitly](https://www.visitly.io/pricing/).
* Un compte d’utilisateur Visitly avec des autorisations d’administrateur

## <a name="assign-users-to-visitly"></a>Affecter des utilisateurs à Visitly 

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs ou groupes affectés à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, identifiez les utilisateurs ou groupes dans Azure AD qui ont besoin d’accéder à Visitly. Ensuite, affectez ces utilisateurs ou groupes à Visitly en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-visitly"></a>Conseils importants pour l’affectation d’utilisateurs à Visitly 

* Nous vous recommandons d’affecter un seul utilisateur Azure AD à Visitly afin de tester la configuration du provisionnement d’utilisateurs automatique. Les autres utilisateurs ou groupes pourront être affectés ultérieurement.

* Quand vous affectez un utilisateur à Visitly, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est Accès par défaut sont exclus de l’approvisionnement.

## <a name="set-up-visitly-for-provisioning"></a>Configurer Visitly pour le provisionnement

Avant de configurer Visitly pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez activer le provisionnement SCIM (System for Cross-domain Identity Management) dans Visitly.

1. Connectez-vous à [Visitly](https://app.visitly.io/login). Sélectionnez **Integrations** > **Host Synchronization**.

    ![Synchronisation hôte](media/Visitly-provisioning-tutorial/login.png)

2. Sélectionnez la section **Azure AD**.

    ![Section Azure AD](media/Visitly-provisioning-tutorial/integration.png)

3. Copiez la **Clé d’API**. Ces valeurs sont entrées dans la zone **Jeton secret** sous l’onglet **Provisionnement** de votre application Visitly dans le portail Azure.

    ![Clé de l’API](media/Visitly-provisioning-tutorial/token.png)


## <a name="add-visitly-from-the-gallery"></a>Ajouter Visitly à partir de la galerie

Pour configurer Visitly pour le provisionnement automatique d’utilisateurs avec Azure AD, ajoutez Visitly à votre liste d’applications SaaS managées à partir de la galerie d’applications Azure AD.

Pour ajouter Visitly à partir de la galerie d’applications Azure AD, procédez comme suit.

1. Dans le panneau de navigation gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Visitly** , sélectionnez **Visitly** dans le volet de résultats, puis sélectionnez **Ajouter** pour ajouter l’application.

    ![Visitly dans la liste des résultats](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-visitly"></a>Configurer le provisionnement automatique d’utilisateurs sur Visitly 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs ou des groupes dans Visitly en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

> [!TIP]
> Pour activer l’authentification unique basée sur SAML pour Visitly, suivez les instructions fournies dans le [tutoriel sur l’authentification unique Visitly](Visitly-tutorial.md). La configuration de l’authentification unique est indépendante de celle de l’approvisionnement automatique d’utilisateurs, même si ces deux fonctionnalités se complètent.

### <a name="configure-automatic-user-provisioning-for-visitly-in-azure-ad"></a>Configurer le provisionnement automatique d’utilisateurs pour Visitly dans Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** > **Toutes les applications**.

    ![Toutes les applications](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Visitly**.

    ![Lien Visitly dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Onglet Approvisionnement](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Mode de provisionnement défini sur Automatique](common/provisioning-automatic.png)

5. Sous la section Informations d’identification de l’administrateur, entrez respectivement les valeurs de `https://api.visitly.io/v1/usersync/SCIM` et du **Clé d’API** récupérées précédemment dans les champs **URL de locataire** et **Jeton secret**. Sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Visitly. Si la connexion échoue, vérifiez que votre compte Visitly dispose d’autorisations d’administration, puis réessayez.

    ![URL de locataire + jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans la zone **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur de provisionnement. Cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Sélectionnez **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Visitly**.

    ![Mappages d’utilisateurs Visitly](media/visitly-provisioning-tutorial/usermapping.png)

9. Dans la section **Mappage d’attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Visitly. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Visitly pour les opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Visitly](media/visitly-provisioning-tutorial/userattribute.png)

10. Pour configurer des filtres d’étendue, suivez les instructions fournies dans le [tutoriel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service de provisionnement d’Azure AD pour Visitly, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs ou groupes à provisionner dans Visitly en choisissant les valeurs souhaitées dans **Étendue** , dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Enregistrement de la configuration du provisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs ou groupes définis sous **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur la durée de provisionnement des utilisateurs ou groupes, consultez [Combien de temps faut-il pour provisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users)

Vous pouvez utiliser la section **État actuel** pour superviser la progression et suivre les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement d’Azure AD dans Visitly. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

Visitly ne prend pas en charge les suppressions définitives. Toutes les suppressions sont réversibles uniquement.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
