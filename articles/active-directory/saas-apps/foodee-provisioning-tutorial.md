---
title: 'Tutoriel : Configurer Foodee pour le provisionnement automatique d’utilisateurs à l’aide d’Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur sur Foodee.
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
ms.openlocfilehash: 8b4bfa7e9bf457d79c6c4a0b5255bce4fe36dff4
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358675"
---
# <a name="tutorial-configure-foodee-for-automatic-user-provisioning"></a>Tutoriel : Configurer Foodee pour le provisionnement automatique d’utilisateurs

Cet article vous montre comment configurer Azure Active Directory (Azure AD) dans Foodee et Azure AD pour provisionner et déprovisionner automatiquement des utilisateurs ou des groupes dans Foodee.

> [!NOTE]
> Cet article décrit un connecteur reposant sur le service de provisionnement d’utilisateurs Azure AD. Pour plus d’informations sur le fonctionnement de ce service et obtenir des réponses aux questions fréquentes, consultez [Automatiser le provisionnement et le déprovisionnement d’utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion. Pour plus d’informations sur la fonctionnalité Conditions d'utilisation Azure relatives aux fonctionnalités d’évaluation, consultez [Avenant aux conditions d'utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Foodee](https://www.food.ee/about/).
* Un compte d’utilisateur dans Foodee avec des autorisations d’administration

## <a name="assign-users-to-foodee"></a>Affecter des utilisateurs à Foodee 

Azure AD utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d'activer le provisionnement automatique d'utilisateurs, vous devez déterminer quels utilisateurs ou groupes d'Azure AD auront besoin d'accéder à Foodee. Une fois cette sélection effectuée, vous pouvez affecter ces utilisateurs ou groupes à Foodee en suivant les instructions fournies dans [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-foodee"></a>Conseils importants pour l’affectation d’utilisateurs à Foodee 

Lorsque vous affectez des utilisateurs, gardez à l'esprit les conseils suivants :

* Nous vous recommandons d’attribuer un seul utilisateur Azure AD à Foodee afin de tester la configuration du provisionnement automatique d’utilisateurs. Vous pouvez affecter des groupes ou des utilisateurs supplémentaires ultérieurement.

* Quand vous affectez un utilisateur à Foodee, sélectionnez un rôle valide propre à l’application (si disponible) dans le volet des **affectations**. Les utilisateurs dont le rôle est *Accès par défaut* sont exclus du provisionnement.

## <a name="set-up-foodee-for-provisioning"></a>Configurer Foodee pour le provisionnement

Avant de configurer Foodee pour le provisionnement automatique des utilisateurs avec Azure AD, vous devez activer le provisionnement SCIM (System for Cross-domain Identity Management) dans Foodee.

1. Connectez-vous à [Foodee](https://www.food.ee/login/), puis sélectionnez votre ID de locataire.

    :::image type="content" source="media/Foodee-provisioning-tutorial/tenant.png" alt-text="Capture d’écran du menu principal du portail d’entreprise Foodee. Un espace réservé d’ID de locataire est visible dans le menu." border="false":::

1. Sous **Enterprise Portal** > sélectionnez **Single Sign On (Authentification unique)** .

    ![Le menu du volet gauche de Foodee Enterprise Portal](media/Foodee-provisioning-tutorial/scim.png)

1. Copiez la valeur dans la zone **API Token (Jeton d’API)** pour une utilisation ultérieure. Vous devrez entrer cette valeur dans la zone **Secret Token (Jeton secret)** sous l’onglet **Provisioning (Provisionnement)** de votre application Foodee sur le portail Azure.

    :::image type="content" source="media/Foodee-provisioning-tutorial/token.png" alt-text="Capture d’écran d’une page du portail d’entreprise Foodee. Une valeur Jeton d’API est mise en évidence." border="false":::

## <a name="add-foodee-from-the-gallery"></a>Ajouter Foodee à partir de la galerie

Pour configurer Foodee pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Foodee à votre liste d’applications SaaS managées à partir de la galerie d’applications Azure AD.

Pour ajouter Foodee à partir de la galerie d’applications Azure AD, procédez comme suit :

1. Dans le volet gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![La commande Azure Active Directory](common/select-azuread.png)

1. Sélectionnez **Applications d’entreprise** > **Toutes les applications**.

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, sélectionnez **Nouvelle application** dans la partie supérieure du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, entrez **Foodee** , sélectionnez **Foodee** dans le volet de résultats, puis **Ajouter** pour ajouter l’application.

    ![Foodee dans la liste de résultats](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-foodee"></a>Configurer le provisionnement automatique d’utilisateurs sur Foodee 

Dans cette section, vous configurez le service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs ou des groupes dans Foodee en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également activer l’authentification unique basée sur SAML pour Foodee en suivant les instructions du [tutoriel sur l’authentification unique Foodee](Foodee-tutorial.md). Vous pouvez configurer l’authentification unique indépendamment de l’approvisionnement d’utilisateurs automatique, même si ces deux fonctionnalités se complètent.

Configurez le provisionnement automatique d’utilisateurs pour Foodee dans Azure AD en procédant comme suit :

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Applications d’entreprise** > **Toutes les applications**.

    ![Volet Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des **applications** , sélectionnez **Foodee**.

    ![Lien Foodee dans la liste des applications](common/all-applications.png)

1. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

1. Dans la liste déroulante **Mode d’approvisionnement** , sélectionnez **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

1. Sous **Informations d’identification de l’administrateur** , procédez comme suit :

   a. Dans la zone **URL de locataire** , entrez la valeur **https:\//concierge.food.ee/scim/v2** que vous avez récupérée précédemment.

   b. Dans la zone **Secret Token (Jeton secret)** , entrez la valeur **API Token (Jeton d’API)** que vous avez récupérée précédemment.
   
   c. Pour vérifier qu’Azure AD peut se connecter à Foodee, sélectionnez **Tester la connexion**. Si la connexion échoue, vérifiez que votre compte Foodee dispose des autorisations d’administrateur, puis réessayez.

    ![Lien Tester la connexion](common/provisioning-testconnection-tenanturltoken.png)

1. Dans la zone **E-mail de notification** , entrez l'adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d'erreur d'approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![Zone de texte E-mail de notification](common/provisioning-notification-email.png)

1. Sélectionnez **Enregistrer**.

1. Sous **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Foodee**.

    :::image type="content" source="media/Foodee-provisioning-tutorial/usermapping.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les utilisateurs Azure Active Directory avec Foodee est mis en évidence." border="false":::

1. Sous **Mappages des attributs** , passez en revue les attributs utilisateur synchronisés entre Azure AD et Foodee. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les *comptes d’utilisateurs* dans Foodee pour les opérations de mise à jour. 

    :::image type="content" source="media/Foodee-provisioning-tutorial/userattribute.png" alt-text="Capture d’écran de la page Mappages d’attributs. Un tableau liste les attributs Azure Active Directory et Foodee et la priorité de correspondance." border="false":::

1. Pour valider vos modifications, sélectionnez **Enregistrer**.
1. Sous **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Foodee**.

    :::image type="content" source="media/Foodee-provisioning-tutorial/groupmapping.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les groupes Azure Active Directory avec Foodee est mis en évidence." border="false":::

1. Sous **Mappages des attributs** , passez en revue les attributs utilisateur synchronisés entre Azure AD et Foodee. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les *comptes de groupes* dans Foodee pour les opérations de mise à jour.

    :::image type="content" source="media/Foodee-provisioning-tutorial/groupattribute.png" alt-text="Capture d’écran de la page Mappages d’attributs. Un tableau liste les attributs Azure Active Directory, les attributs Foodee et la priorité de correspondance." border="false":::

1. Pour valider vos modifications, sélectionnez **Enregistrer**.
1. Configurez les filtres d’étendue. Pour cette opération, reportez-vous aux instructions fournies dans le [tutoriel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Pour activer le service d’approvisionnement d’Azure AD pour Foodee, dans la section **Paramètres** , définissez le paramètre **État d’approvisionnement** sur **Activé**.

    ![Commutateur d’état d’approvisionnement](common/provisioning-toggle-on.png)

1. Sous **Paramètres** , dans la liste déroulante **Étendue** , définissez les utilisateurs ou groupes que vous souhaitez provisionner dans Foodee.

    ![Liste déroulante Étendue de l’approvisionnement](common/provisioning-scope.png)

1. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Bouton Enregistrer pour la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

L'opération précédente lance la synchronisation initiale des utilisateurs ou groupes que vous avez définis dans la liste déroulante **Étendue**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d'informations, voir [Combien de temps faut-il pour approvisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Vous pouvez utiliser la section **État actuel** pour superviser la progression et suivre les liens vers votre rapport d’activité d’approvisionnement. Ce rapport décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur Foodee. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
