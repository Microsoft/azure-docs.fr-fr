---
title: 'Tutoriel : Configurer Symantec Web Security Service (WSS) pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Apprenez à configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d'utilisateur sur Symantec Web Security Service (WSS).
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/23/2019
ms.author: Zhchia
ms.openlocfilehash: d7e0db1b0bc1e7aef68ee06f3bdd5e5e0f83b73e
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94354697"
---
# <a name="tutorial-configure-symantec-web-security-service-wss-for-automatic-user-provisioning"></a>Tutoriel : Configurer Symantec Web Security Service (WSS) pour l’approvisionnement automatique d’utilisateurs

L'objectif de ce didacticiel est de présenter les étapes à suivre dans Symantec Web Security Service (WSS) et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l'approvisionnement et le retrait automatiques d'utilisateurs et/ou de groupes sur Symantec Web Security Service (WSS).

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Symantec Web Security Service (WSS)](https://www.websecurity.symantec.com/buy-renew?inid=brmenu_nav_brhome)
* Un compte d’utilisateur dans Symantec Web Security Service (WSS) avec des autorisations d’administrateur.

## <a name="assigning-users-to-symantec-web-security-service-wss"></a>Affectation d’utilisateurs à Symantec Web Security Service (WSS)

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d'activer l'approvisionnement automatique d'utilisateurs, vous devez déterminer quels utilisateurs et/ou groupes d'Azure AD auront besoin d'accéder à Symantec Web Security Service (WSS). Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application Symantec Web Security Service (WSS) en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

##  <a name="important-tips-for-assigning-users-to-symantec-web-security-service-wss"></a>Conseils importants pour l’affectation d’utilisateurs à Symantec Web Security Service (WSS)

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Symantec Web Security Service (WSS) afin de tester la configuration de l’attribution d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous attribuez un utilisateur à Symantec Web Security Service (WSS), vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-symantec-web-security-service-wss-for-provisioning"></a>Configurer Symantec Web Security Service (WSS) pour l’approvisionnement

Avant de configurer Symantec Web Security Service (WSS) pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer l’approvisionnement SCIM sur Symantec Web Security Service (WSS).

1. Connectez-vous à la [console d’administration de Symantec Web Security Service](https://portal.threatpulse.com/login.jsp). Accédez à **Solutions** > **Service**.

    ![Symantec Web Security Service (WSS)](media/symantec-web-security-service/service.png)

2. Accédez à **Maintenance du compte** > **Intégrations** > **Nouvelle intégration**.

    ![Symantec Web Security Service (WSS)](media/symantec-web-security-service/acount.png)

3.  Sélectionnez **Synchroniser utilisateurs tiers et groupes**. 

    ![Capture d’écran de l’option Synchroniser utilisateurs tiers et groupes.](media/symantec-web-security-service/third-party-users.png)

4.  Copiez **l’URL SCIM** et le **Jeton**. Ces valeurs doivent être entrées dans les champs **URL de locataire** et **Jeton secret** de l'onglet Approvisionnement de votre application Symantec Web Security Service (WSS)          sur le Portail Azure.

    ![Capture d’écran de la boîte de dialogue Nouvelle intégration avec les zones de texte URL SCIM et Jeton en évidence.](media/symantec-web-security-service/scim.png)

## <a name="add-symantec-web-security-service-wss-from-the-gallery"></a>Ajout de Symantec Web Security Service (WSS) à partir de la galerie

Avant de configurer Symantec Web Security Service (WSS) pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez ajouter Symantec Web Security Service (WSS) à votre liste d'applications SaaS gérées à partir de la galerie d'applications Azure AD.

**Pour ajouter Symantec Web Security Service (WSS) à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Symantec Web Security Service** , sélectionnez **Symantec Web Security Service** dans le volet des résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Symantec Web Security Service (WSS) dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-symantec-web-security-service-wss"></a>Configuration de l’approvisionnement automatique d’utilisateurs dans Symantec Web Security Service (WSS)

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Symantec Web Security Service (WSS) en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Symantec Web Security Service (WSS) en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Symantec Web Security Service (WSS)](symantec-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-symantec-web-security-service-wss-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Symantec Web Security Service (WSS) dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Symantec Web Security Service**.

    ![Lien Symantec Web Security Service (WSS) dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section Informations d'identification de l'administrateur, entrez respectivement les valeurs de **l’URL SCIM** et du **Jeton** récupérées précédemment dans les champs **URL de locataire** et **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Symantec Web Security Service (WSS). Si la connexion échoue, vérifiez que votre compte Symantec Web Security Service (WSS) dispose d’autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Symantec Web Security Service (WSS)** .

    ![Capture d’écran de la section Mappages avec l’option Synchroniser les utilisateurs Azure Active Directory avec Symantec Web Security Service WSS en évidence.](media/symantec-web-security-service/usermapping.png)

9. Dans la section **Mappage des attributs** , passez en revue les attributs d'utilisateurs synchronisés entre Azure AD et Symantec Web Security Service (WSS). Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Symantec Web Security Service (WSS) pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappage des attributs montrant 16 propriétés correspondantes.](media/symantec-web-security-service/userattribute.png)

10. Dans la section **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Symantec Web Security Service**.

    ![Capture d’écran de la section Mappages avec l’option Synchroniser les groupes Azure Active Directory avec Symantec Web Security Service WSS en évidence.](media/symantec-web-security-service/groupmapping.png)

11. Dans la section **Mappages des attributs** , passez en revue les attributs de groupes qui sont synchronisés entre Azure AD et Symantec Web Security Service (WSS). Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Symantec Web Security Service (WSS) pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappage des attributs montrant trois propriétés correspondantes.](media/symantec-web-security-service/groupattribute.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour Symantec Web Security Service, modifiez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes à approvisionner sur Symantec Web Security Service (WSS) en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur la durée d’approvisionnement des utilisateurs et/ou des groupes, voir [Combien de temps faut-il pour approvisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Vous pouvez utiliser la section **État actuel** pour surveiller la progression et suivre les liens vers le rapport d'activité d'approvisionnement, qui décrit toutes les actions effectuées par le service d'approvisionnement Azure AD sur Symantec Web Security Service (WSS). Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
