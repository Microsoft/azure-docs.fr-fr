---
title: 'Tutoriel : Configurer Harness pour le provisionnement automatique des utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur sur Harness.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/29/2019
ms.author: Zhchia
ms.openlocfilehash: 13ae960f5d259314f00f8f09b2999a36c0919bc5
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94353711"
---
# <a name="tutorial-configure-harness-for-automatic-user-provisioning"></a>Tutoriel : Configurer Harness pour le provisionnement automatique d’utilisateurs

Dans cet article, vous allez apprendre à configurer Azure Active Directory (Azure AD) pour provisionner et déprovisionner automatiquement des utilisateurs ou des groupes dans Harness.

> [!NOTE]
> Cet article décrit un connecteur qui s’appuie sur le service d’attribution d’utilisateurs Azure AD. Pour obtenir des informations importantes sur ce service, ainsi que des réponses aux questions fréquentes, consultez [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans cet article suppose que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Harness](https://harness.io/pricing/)
* Un compte d’utilisateur dans Harness avec des autorisations d’ *administrateur*

## <a name="assign-users-to-harness"></a>Attribuer des utilisateurs à Harness

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, identifiez les utilisateurs ou groupes dans Azure AD qui doivent avoir accès à Harness. Vous pouvez alors attribuer ces utilisateurs ou groupes à Harness en suivant les instructions fournies dans [Attribuer un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-harness"></a>Conseils importants pour l’affectation d’utilisateurs à Harness

* Il est recommandé d’attribuer un seul utilisateur Azure AD à Harness afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs ou groupes pourront être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Harness, vous devez sélectionner un rôle propre à l’application valide (si disponible) dans la boîte de dialogue **Attribution**. Les utilisateurs dont le rôle est *Accès par défaut* sont exclus de l’approvisionnement.

## <a name="set-up-harness-for-provisioning"></a>Configurer Harness pour le provisionnement

1. Connectez-vous à votre [console d’administration Harness](https://app.harness.io/#/login), puis accédez à **Continuous Security** (Sécurité continue) > **Access Management** (Gestion de l’accès).

    ![Console d’administration Harness](media/harness-provisioning-tutorial/admin.png)

1. Sélectionnez **API Keys** (Clés API).

    ![Lien API Keys (Clés API) Harness](media/harness-provisioning-tutorial/apikeys.png)

1. Sélectionnez **Add API Key** (Ajouter une clé API). 

    ![Lien Add API Key (Ajouter une clé API)](media/harness-provisioning-tutorial/addkey.png)

1. Dans le volet **Add Api Key** (Ajouter une clé API), effectuez les étapes suivantes :

    ![Volet Add Api Key (Ajouter une clé API)](media/harness-provisioning-tutorial/title.png)
   
   a. Dans la zone **Name** (Name), indiquez un nom pour la clé.  
   b. Dans la liste déroulante **Permissions Inherited from** (Autorisations héritées de), sélectionnez une option. 
   
1. Sélectionnez **Envoyer**.

1. Copiez la valeur **Key** (Clé) pour une utilisation ultérieure dans ce tutoriel.

    ![Harness - Créer un jeton](media/harness-provisioning-tutorial/token.png)

## <a name="add-harness-from-the-gallery"></a>Ajouter Harness à partir de la galerie

Avant de configurer Harness pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez ajouter Harness à votre liste d’applications SaaS managées à partir de la galerie d’applications Azure AD.

1. Dans le volet gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Le bouton « Azure Active Directory »](common/select-azuread.png)

1. Sélectionnez **Applications d’entreprise** > **Toutes les applications**.

    ![Lien Toutes les applications](common/enterprise-applications.png)

1. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

1. Dans la zone de recherche, entrez **Harness** , sélectionnez **Harness** dans la liste des résultats, puis sélectionnez le bouton **Ajouter** pour ajouter l’application.

    ![Harness dans la liste des résultats](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-harness"></a>Configurer l’attribution automatique d’utilisateurs à Harness 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement Azure AD pour créer, mettre à jour et désactiver des utilisateurs ou des groupes dans Harness en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour Harness en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Harness](./harness-tutorial.md). Vous pouvez configurer l’authentification unique indépendamment de l’attribution automatique d’utilisateurs, même si ces deux fonctionnalités se complètent.

> [!NOTE]
> Pour en savoir plus sur le point de terminaison SCIM Harness, consultez l’article sur les [clés API](https://docs.harness.io/article/smloyragsm-api-keys) Harness.

Pour configurer l’attribution automatique d’utilisateurs pour Harness dans Azure AD, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Applications d’entreprise** > **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

1. Dans la liste des applications, sélectionnez **Harness**.

    ![Lien Harness dans la liste des applications](common/all-applications.png)

1. Sélectionnez **Provisionnement**.

    ![Bouton Provisionnement](common/provisioning.png)

1. Dans la liste déroulante **Mode d’approvisionnement** , sélectionnez **Automatique**.

    ![Liste déroulante « Mode de provisionnement »](common/provisioning-automatic.png)

1. Sous **Informations d’identification de l’administrateur** , procédez comme suit :

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)
 
   a. Dans la zone **URL de locataire** , entrez **`https://app.harness.io/gateway/api/scim/account/<your_harness_account_ID>`** . Vous pouvez obtenir votre ID de compte Harness à partir de l’URL dans votre navigateur lorsque vous êtes connecté à Harness.
   b. Dans la zone **Jeton secret** , entrez la valeur du jeton d’authentification SCIM que vous avez enregistré à l’étape 6 de la section « Configurer Harness pour le provisionnement ».  
   c. Sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Harness. Si la connexion échoue, vérifiez que votre compte Harness dispose d’autorisations d’ *administrateur* , puis réessayez.

1. Dans la zone **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur de provisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![Zone « E-mail de notification »](common/provisioning-notification-email.png)

1. Sélectionnez **Enregistrer**.

1. Sous **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Harness**.

    ![Lien Harness « Synchroniser les utilisateurs Azure Active Directory avec Harness »](media/harness-provisioning-tutorial/usermappings.png)

1. Sous **Mappages d’attributs** , passez en revue les attributs utilisateur synchronisés entre Azure AD et Harness. Les attributs sélectionnés en tant que *Correspondance* sont utilisés pour faire correspondre les comptes d’utilisateurs dans Harness pour les opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    ![Volet « Mappages d’attributs » des utilisateurs Harness](media/harness-provisioning-tutorial/userattributes.png)

1. Sous **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Harness**.

    ![Lien Harness « Synchroniser les groupes Azure Active Directory avec Harness »](media/harness-provisioning-tutorial/groupmappings.png)

1. Sous **Mappages d’attributs** , passez en revue les attributs de groupe synchronisés entre Azure AD et Harness. Les attributs sélectionnés comme propriétés de *Correspondance* sont utilisés pour la mise en correspondre des groupes dans Harness dans le cadre des opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications.

    ![Volet « Mappages d’attributs » des groupes Harness](media/harness-provisioning-tutorial/groupattributes.png)

1. Pour configurer des filtres d’étendue, consultez [Provisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

1. Sous **Paramètres** , pour activer le service de provisionnement Azure AD pour Harness, basculez le commutateur **État du provisionnement** sur **Activé**.

    ![Commutateur d’état du provisionnement basculé sur « Activé »](common/provisioning-toggle-on.png)

1. Sous **Paramètres** , dans la liste déroulante **Étendue** , sélectionnez la façon dont vous voulez synchroniser les utilisateurs ou groupes que vous provisionnez dans Harness.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

1. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Bouton Enregistrer du provisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale des utilisateurs ou des groupes que vous provisionnez. La synchronisation initiale prend plus de temps que celles ultérieures. Les synchronisations se produisent toutes les 40 minutes environ, tant que le service de provisionnement Azure AD est en cours d’exécution. Pour superviser la progression, accédez à la section **Détails de la synchronisation**. Vous pouvez également suivre les liens fournis pour accéder à un rapport d’activité de provisionnement. Ce rapport décrit toutes les actions effectuées par le service de provisionnement Azure AD dans Harness.

Pour plus d’informations sur la façon de lire les journaux de provisionnement Azure AD, consultez [Générer un rapport sur le provisionnement automatique de comptes d’utilisateurs](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)