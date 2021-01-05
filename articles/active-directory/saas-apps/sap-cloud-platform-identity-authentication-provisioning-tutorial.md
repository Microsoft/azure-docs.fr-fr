---
title: 'Tutoriel : Configurer SAP Cloud Platform Identity Authentication pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur sur SAP Cloud Platform Identity Authentication.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/19/2019
ms.author: Zhchia
ms.openlocfilehash: 419f25ee3df471bc2fc4526254f5677b8bd71856
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96342729"
---
# <a name="tutorial-configure-sap-cloud-platform-identity-authentication-for-automatic-user-provisioning"></a>Tutoriel : Configurer SAP Cloud Platform Identity Authentication pour le provisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans SAP Cloud Platform Identity Authentication et Azure Active Directory (Azure AD) pour configurer Azure AD pour le provisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur SAP Cloud Platform Identity Authentication.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire SAP Cloud Platform Identity Authentication](https://cloudplatform.sap.com/pricing.html)
* Un compte d’utilisateur dans SAP Cloud Platform Identity Authentication disposant d’autorisations d’administrateur

## <a name="assigning-users-to-sap-cloud-platform-identity-authentication"></a>Affectation d’utilisateurs à SAP Cloud Platform Identity Authentication

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à SAP Cloud Platform Identity Authentication. Une fois que vous avez choisi, vous pouvez affecter ces utilisateurs et/ou groupes à SAP Cloud Platform Identity Authentication en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-sap-cloud-platform-identity-authentication"></a>Conseils importants pour l’affectation d’utilisateurs à SAP Cloud Platform Identity Authentication

* Il est recommandé de n’affecter qu’un seul utilisateur Azure AD à SAP Cloud Platform Identity Authentication afin de tester la configuration du provisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous affectez un utilisateur à SAP Cloud Platform Identity Authentication, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-sap-cloud-platform-identity-authentication-for-provisioning"></a>Configurer SAP Cloud Platform Identity Authentication pour le provisionnement

1. Connectez-vous à votre [console d’administration SAP Cloud Platform Identity Authentication](https://sapmsftintegration.accounts.ondemand.com/admin). Accédez à **Users & Authorizations > Administrators** (Utilisateurs et autorisations > Administrateurs).

    ![Console d’administration SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/adminconsole.png)

2.  Appuyez sur le bouton **+Add** (+Ajouter) dans le panneau de gauche afin d’ajouter un nouvel administrateur à la liste. Choisissez **Add System** (Ajouter un système) et entrez le nom du système.   

> [!NOTE]
> L’utilisateur administrateur dans SAP Cloud Platform Identity Authentication doit être de type **système**. La création d’un utilisateur administrateur standard peut entraîner des erreurs *non autorisé* pendant le provisionnement.   

3.  Sous Configure Authorizations (Configurer les autorisations), activez le bouton bascule en regard de **Manage Users** (Gérer les utilisateurs) et **Manage Groups** (Gérer les groupes).

    ![SAP Cloud Platform Identity Authentication - Ajout de SCIM](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/configurationauth.png)

4. Vous recevrez un e-mail pour activer votre compte et définir un mot de passe pour le **service SAP Cloud Platform Identity Authentication**.

4.  Copiez l’ID utilisateur (**User ID**) et le mot de passe (**Password**). Ces valeurs seront entrées respectivement dans les champs Nom d’utilisateur de l’administrateur et Mot de passe de l’administrateur, sous l’onglet Provisionnement de votre application SAP Cloud Platform Identity Authentication dans le portail Azure.

## <a name="add-sap-cloud-platform-identity-authentication-from-the-gallery"></a>Ajouter SAP Cloud Platform Identity Authentication à partir de la galerie

Avant de configurer SAP Cloud Platform Identity Authentication pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter SAP Cloud Platform Identity Authentication à votre liste d’applications SaaS managées à partir de la galerie d’applications Azure AD.

**Pour ajouter SAP Cloud Platform Identity Authentication à partir de la galerie d’applications Azure AD, effectuez les étapes ci-dessous :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **SAP Cloud Platform Identity Authentication**, sélectionnez **SAP Cloud Platform Identity Authentication** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![SAP Cloud Platform Identity Authentication dans la liste de résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-sap-cloud-platform-identity-authentication"></a>Configuration du provisionnement automatique d’utilisateurs pour SAP Cloud Platform Identity Authentication 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans SAP Cloud Platform Identity Authentication en fonction des affectations d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour SAP Cloud Platform Identity Authentication en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour SAP Cloud Platform Identity Authentication](./sap-hana-cloud-platform-identity-authentication-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent

### <a name="to-configure-automatic-user-provisioning-for-sap-cloud-platform-identity-authentication-in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs pour SAP Cloud Platform Identity Authentication dans Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **SAP Cloud Platform Identity Authentication**.

    ![Lien SAP Cloud Platform Identity Authentication dans la liste Applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://<tenantID>.accounts.ondemand.com/service/scim ` dans **URL de locataire**. Entrez les valeurs **User ID** et **Password** récupérées plus tôt dans les champs **Nom d’utilisateur de l’administrateur** et **Mot de passe de l’administrateur**, respectivement. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à SAP Cloud Platform Identity Authentication. Si la connexion échoue, vérifiez que votre compte SAP Cloud Platform Identity Authentication dispose d’autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/testconnection.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec SAP Cloud Platform Identity Authentication**.

    ![Mappage d’utilisateurs de SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/mapping.png)

9. Passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers SAP Cloud Platform Identity Authentication dans la section **Mappages d’attributs**. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondance des comptes d’utilisateur dans SAP Cloud Platform Identity Authentication dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur de SAP Cloud Platform Identity Authentication](media/sap-cloud-platform-identity-authentication-provisioning-tutorial/userattributes.png)

10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service de provisionnement Azure AD pour SAP Cloud Platform Identity Authentication, affectez la valeur **Activé** au paramètre **État du provisionnement** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou groupes que vous souhaitez provisionner dans SAP Cloud Platform Identity Authentication en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour analyser la progression et suivre les liens vers les rapports d’activité de provisionnement ; elle décrit toutes les actions effectuées par le service de provisionnement Azure AD sur SAP Cloud Platform Identity Authentication.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* Le point de terminaison SCIM de l’authentification d’identité SAP Cloud Platform demande un format particulier pour certains attributs. Vous pouvez en savoir plus sur ces attributs et leur format spécifique [ici](https://help.sap.com/viewer/6d6d63354d1242d185ab4830fc04feb1/Cloud/en-US/b10fc6a9a37c488a82ce7489b1fab64c.html#).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)