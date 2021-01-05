---
title: "Tutoriel : Configurer Reward Gateway pour l'approvisionnement automatique d'utilisateurs avec Azure Active Directory | Microsoft Docs"
description: Apprenez à configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d'utilisateur sur Reward Gateway.
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
ms.openlocfilehash: 2d51903aff6f3fd1cd53d85a980f1b5dc2a893e9
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94354265"
---
# <a name="tutorial-configure-reward-gateway-for-automatic-user-provisioning"></a>Tutoriel : Configurer Reward Gateway pour l'approvisionnement automatique d'utilisateurs

L'objectif de ce didacticiel est de présenter les étapes à suivre dans Reward Gateway et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l'approvisionnement et le retrait automatiques d'utilisateurs et/ou de groupes sur Reward Gateway.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* Un [locataire Reward Gateway](https://www.rewardgateway.com/).
* Un compte d'utilisateur Reward Gateway doté d'autorisations d'administrateur.

## <a name="assigning-users-to-reward-gateway"></a>Attribuer des utilisateurs à Reward Gateway 

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d'activer l'approvisionnement automatique d'utilisateurs, vous devez déterminer quels utilisateurs et/ou groupes d'Azure AD auront besoin d'accéder à Reward Gateway. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Reward Gateway en suivant les instructions fournies dans [Attribuer un utilisateur ou un groupe à une application d'entreprise](../manage-apps/assign-user-or-group-access-portal.md).


## <a name="important-tips-for-assigning-users-to-reward-gateway"></a>Conseils importants pour l'attribution d'utilisateurs à Reward Gateway 

* Il est recommandé de n'attribuer qu'un seul utilisateur Azure AD à Reward Gateway afin de tester la configuration de l'approvisionnement automatique d'utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous attribuez un utilisateur à Reward Gateway, vous devez sélectionner un rôle valide spécifique à l'application (si disponible) dans la boîte de dialogue d'attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-reward-gateway--for-provisioning"></a>Configurer Reward Gateway pour l'approvisionnement
Avant de configurer Reward Gateway pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez activer l'approvisionnement SCIM sur Reward Gateway.

1. Connectez-vous à votre [Console d'administration Reward Gateway](https://rewardgateway.photoshelter.com/login/). Cliquez sur **Integrations**.

    ![Capture d’écran de la console d’administration Reward Gateway avec l’option Intégrations en évidence.](media/reward-gateway-provisioning-tutorial/image00.png)

2.  Sélectionnez **Mon intégration**.

    ![Capture d’écran des deux options d’intégration avec l’option Mes intégrations en évidence.](media/reward-gateway-provisioning-tutorial/image001.png)

3.  Copiez les valeurs des champs **URL SCIM (v2)** et **Jeton du porteur OAuth**. Ces valeurs doivent être entrées dans les champs URL de locataire et Jeton secret de l'onglet Approvisionnement de votre application Reward Gateway sur le Portail Azure.

    ![Capture d’écran du panneau Mes intégrations avec la zone de texte Jeton du porteur OAuth en évidence.](media/reward-gateway-provisioning-tutorial/image03.png)

## <a name="add-reward-gateway-from-the-gallery"></a>Ajouter Reward Gateway à partir de la galerie

Avant de configurer Reward Gateway pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez ajouter Reward Gateway à votre liste d'applications SaaS managées à partir de la galerie d'applications Azure AD.

**Pour ajouter Reward Gateway à partir de la galerie d'applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Reward Gateway** , sélectionnez **Reward Gateway** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l'application.

    ![Reward Gateway dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-reward-gateway"></a>Configuration de l'approvisionnement automatique d'utilisateurs sur Reward Gateway  

Cette section vous guide tout au long des étapes de configuration du service d'approvisionnement d'Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Reward Gateway en fonction des attributions d'utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d'activer l'authentification unique basée sur SAML pour Reward Gateway en suivant les instructions fournies dans le [didacticiel sur l'authentification unique Reward Gateway](reward-gateway-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-reward-gateway-in-azure-ad"></a>Pour configurer l'approvisionnement automatique d'utilisateurs pour Reward Gateway dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Reward Gateway**.

    ![Lien Reward Gateway dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **Informations d'identification de l'administrateur** , entrez respectivement les valeurs de l' **URL SCIM (v2)** et du **Jeton du porteur OAuth** récupérées précédemment dans les champs **URL de locataire** et **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu'Azure AD peut se connecter à Reward Gateway. Si la connexion échoue, vérifiez que votre compte Reward Gateway dispose des autorisations d'administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Reward Gateway**.

    ![Capture d’écran de la section Mappages avec l’option Synchroniser les utilisateurs Azure Active Directory avec Reward Gateway en évidence.](media/reward-gateway-provisioning-tutorial/user-mappings.png)

9. Dans la section **Mappages des attributs** , passez en revue les attributs d'utilisateurs synchronisés entre Azure AD et Reward Gateway. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d'utilisateur dans Reward Gateway pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappages d’attributs avec six mappages affichés.](media/reward-gateway-provisioning-tutorial/user-attributes.png)

10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service d'approvisionnement Azure AD pour Reward Gateway, définissez le paramètre **État d'approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou groupes à approvisionner sur Reward Gateway en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d'activité d'approvisionnement, qui décrivent toutes les actions effectuées par le service d'approvisionnement Azure AD sur Reward Gateway.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

Reward Gateway ne prend actuellement pas en charge l'approvisionnement de groupe.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

[Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)