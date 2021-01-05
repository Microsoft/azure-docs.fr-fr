---
title: 'Tutoriel : Configurer Soloinsight-CloudGate SSO pour l’approvisionnement automatique d’utilisateurs à l’aide d’Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur sur Soloinsight-CloudGate SSO.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/14/2019
ms.author: Zhchia
ms.openlocfilehash: 5c42102c0704d7212411d6b86f4210ad8ecd885c
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96347495"
---
# <a name="tutorial-configure-soloinsight-cloudgate-sso-for-automatic-user-provisioning"></a>Tutoriel : Configurer Soloinsight-CloudGate SSO pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans Soloinsight-CloudGate SSO et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur Soloinsight-CloudGate SSO.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Locataire Soloinsight-CloudGate SSO](https://www.soloinsight.com/)
* Compte d’utilisateur dans Soloinsight-CloudGate SSO ayant des autorisations d’administrateur.

## <a name="assigning-users-to-soloinsight-cloudgate-sso"></a>Affectation d’utilisateurs à Soloinsight-CloudGate SSO

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Soloinsight-CloudGate SSO. Une fois que vous avez choisi, vous pouvez assigner ces utilisateurs et/ou groupes à Soloinsight-CloudGate SSO en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-soloinsight-cloudgate-sso"></a>Conseils importants pour l’affectation d’utilisateurs à Soloinsight-CloudGate SSO

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Soloinsight-CloudGate SSO afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous assignez un utilisateur à Soloinsight-CloudGate SSO, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-soloinsight-cloudgate-sso-for-provisioning"></a>Configurer Soloinsight-CloudGate SSO pour l’approvisionnement

1. Connectez-vous à votre [console d’administration Soloinsight-CloudGate SSO](https://soloinsight.sigateway.com/login). Accédez à **Administration > Paramètres système**.

    ![Console d’administration Soloinsight-CloudGate SSO](media/soloinsight-cloudgate-sso-provisioning-tutorial/admin.png)

2.  Accédez à **Général**.

    ![Soloinsight-CloudGate SSO – Ajouter SCIM](media/soloinsight-cloudgate-sso-provisioning-tutorial/config.png)

3.  Faites défiler jusqu’à la fin de la page pour accéder à **URL du locataire** et **Jeton secret**. Copiez le **Jeton secret**. Vous devrez entrer cette valeur dans le champ Jeton secret dans l’onglet Approvisionnement de votre application Soloinsight-CloudGate SSO dans le Portail Azure.

    ![Soloinsight-CloudGate SSO – Créer un jeton](media/soloinsight-cloudgate-sso-provisioning-tutorial/token.png)

## <a name="add-soloinsight-cloudgate-sso-from-the-gallery"></a>Ajouter Soloinsight-CloudGate SSO à partir de la galerie

Avant de configurer Soloinsight-CloudGate SSO pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Soloinsight-CloudGate SSO à votre liste d’applications SaaS managées à partir de la galerie d’applications Azure AD.

**Pour ajouter Soloinsight-CloudGate SSO à partir de la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Soloinsight-CloudGate SSO**, sélectionnez **Soloinsight-CloudGate SSO** dans le panneau des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Soloinsight-CloudGate SSO dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-soloinsight-cloudgate-sso"></a>Configuration de l’approvisionnement automatique d’utilisateurs sur Soloinsight-CloudGate SSO 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Soloinsight-CloudGate SSO en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour Soloinsight-CloudGate SSO en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour Soloinsight-CloudGate SSO](./soloinsight-cloudgate-sso-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent

### <a name="to-configure-automatic-user-provisioning-for-soloinsight-cloudgate-sso-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Soloinsight-CloudGate SSO dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Soloinsight-CloudGate SSO**.

    ![Lien Soloinsight-CloudGate SSO dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://sigateway.com/scim/v2/sync/serviceproviderconfig` dans **URL de locataire**. Saisissez la valeur de **Jeton d’authentification SCIM** récupérée précédemment dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Soloinsight-CloudGate SSO. Si la connexion échoue, vérifiez que votre compte Soloinsight-CloudGate SSO dispose d’autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Soloinsight-CloudGate SSO**.

    ![Soloinsight-CloudGate SSO – Mappages d’utilisateurs](media/soloinsight-cloudgate-sso-provisioning-tutorial/usermappings.png)

9. Passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers Soloinsight-CloudGate SSO dans la section **Mappage d’attributs**. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Soloinsight-CloudGate SSO pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Soloinsight-CloudGate SSO – Attributs utilisateur](media/soloinsight-cloudgate-sso-provisioning-tutorial/userattributes.png)

10. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Soloinsight-CloudGate SSO**.

    ![Soloinsight-CloudGate SSO – Mappages de groupes](media/soloinsight-cloudgate-sso-provisioning-tutorial/groupmappings.png)

11. Passez en revue les attributs de groupe qui sont synchronisés d’Azure AD vers Soloinsight-CloudGate SSO dans la section **Mappage d’attributs**. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Soloinsight-CloudGate SSO pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Soloinsight-CloudGate SSO – Attributs de groupes](media/soloinsight-cloudgate-sso-provisioning-tutorial/groupattributes.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour Soloinsight-CloudGate SSO, modifiez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Soloinsight-CloudGate SSO en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et suivre les liens vers le rapport d’activité d’approvisionnement, qui décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur Soloinsight-CloudGate SSO.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)