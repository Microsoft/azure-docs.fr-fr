---
title: 'Tutoriel : Configurer Netskope User Authentication pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d’utilisateur sur Netskope User Authentication.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/07/2019
ms.author: Zhchia
ms.openlocfilehash: 46766a7439185714648572f3f1b9d51ef96abba6
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357468"
---
# <a name="tutorial-configure-netskope-user-authentication-for-automatic-user-provisioning"></a>Tutoriel : Configurer Netskope User Authentication pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans Netskope User Authentication et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le retrait automatiques d’utilisateurs et/ou de groupes sur Netskope User Authentication.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Netskope User Authentication](https://www.netskope.com/)
* un compte d’utilisateur dans Netskope User Authentication avec les autorisations d’administrateur.

## <a name="assigning-users-to-netskope-user-authentication"></a>Attribution d’utilisateurs à Netskope User Authentication

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Netskope User Authentication. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Netskope User Authentication en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-netskope-user-authentication"></a>Conseils importants pour l’attribution d’utilisateurs à Netskope User Authentication

* Il est recommandé de n’attribuer qu’un utilisateur Azure AD à Netskope User Authentication afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous assignez un utilisateur à Netskope User Authentication, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’assignation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-netskope-user-authentication-for-provisioning"></a>Configurer Netskope User Authentication pour l’approvisionnement

1. Connectez-vous à votre [console d’administration Netskope User Authentication](https://netskope.goskope.com/). Accédez à **Accueil > Paramètres**.

    ![Console d’administration Netskope User Authentication](media/netskope-administrator-console-provisioning-tutorial/admin.png)

2.  Accédez à **Outils**. Dans le menu **Outils** , accédez à **Outils Active Directory > INTÉGRATION SCIM**.

    ![Outils Netskope User Authentication](media/netskope-administrator-console-provisioning-tutorial/tools.png)

    ![Netskope User Authentication - Ajouter SCIM](media/netskope-administrator-console-provisioning-tutorial/directory.png)

3. Défilez vers le bas, puis cliquez sur le bouton **Ajouter un jeton**. Dans la boîte de dialogue **Ajouter un nom de client OAuth** , indiquez un **NOM DE CLIENT** , puis cliquez sur le bouton **Enregistrer**.

    ![Netskope User Authentication - Ajouter un jeton](media/netskope-administrator-console-provisioning-tutorial/add.png)

    ![Nom du client Netskope User Authentication](media/netskope-administrator-console-provisioning-tutorial/clientname.png)

3.  Copiez l’ **URL du serveur SCIM** et le **JETON**. Ces valeurs doivent être entrées dans les champs URL de locataire et Jeton secret respectivement de l’onglet Approvisionnement de votre application Netskope User Authentication application sur le portail Azure.

    ![Netskope User Authentication - Créer un jeton](media/netskope-administrator-console-provisioning-tutorial/token.png)

## <a name="add-netskope-user-authentication-from-the-gallery"></a>Ajout de Netskope User Authentication à partir de la galerie

Avant de configurer Netskope User Authentication pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Netskope User Authentication à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Netskope User Authentication à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Netskope User Authentication** , sélectionnez **Netskope User Authentication** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Netskope User Authentication dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-netskope-user-authentication"></a>Configuration de l’approvisionnement automatique d’utilisateurs sur Netskope User Authentication 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Netskope User Authentication en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique SAML pour Netskope User Authentication en suivant les instructions fournies dans le [Tutoriel sur l’authentification unique Netskope User Authentication](./netskope-cloud-security-tutorial.md). La configuration de l’authentification unique est indépendante de celle de l’approvisionnement automatique d’utilisateurs, même si ces deux fonctionnalités se complètent.

> [!NOTE]
> Pour en savoir plus sur le point de terminaison SCIM de Netskope User Authentication, cliquez [ici](https://docs.google.com/document/d/1n9P_TL98_kd1sx5PAvZL2HS6MQAqkQqd-OSkWAAU6ck/edit#heading=h.prxq74iwdpon).

### <a name="to-configure-automatic-user-provisioning-for-netskope-user-authentication-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Netskope User Authentication dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Netskope User Authentication**.

    ![Lien Netskope User Authentication dans la liste Applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification Administrateur** , entrez la valeur **URL du serveur SCIM** récupérée précédemment dans **URL de locataire**. Saisissez la valeur de **TOKEN** récupérée précédemment dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Netskope User Authentication. Si la connexion échoue, vérifiez que votre compte Netskope User Authentication dispose d’autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Netskope User Authentication**.

    ![Mappages d’utilisateurs Netskope User Authentication](media/netskope-administrator-console-provisioning-tutorial/usermappings.png)

9. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Netskope User Authentication. Les attributs sélectionnés en tant que propriétés de **Correspondance** servent à faire correspondre les comptes utilisateur dans Netskope User Authentication, en vue d’opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Netskope User Authentication](media/netskope-administrator-console-provisioning-tutorial/userattributes.png)

10. Dans la section **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Netskope User Authentication**.

    ![Mappages de groupes Netskope User Authentication](media/netskope-administrator-console-provisioning-tutorial/groupmappings.png)

11. Dans la section **Mappages des attributs** , passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Netskope User Authentication. Les attributs sélectionnés en tant que propriétés de **Correspondance** servent à faire correspondre les groupes dans Netskope User Authentication, en vue d’opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs des groupes Netskope User Authentication](media/netskope-administrator-console-provisioning-tutorial/groupattributes.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour Netskope User Authentication, affectez la valeur **Activé** au paramètre **État de l’approvisionnement** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Netskope User Authentication en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD sur Netskope User Authentication.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)