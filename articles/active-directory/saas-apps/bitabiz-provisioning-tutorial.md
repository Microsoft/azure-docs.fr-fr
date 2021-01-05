---
title: 'Tutoriel : Configurer BitaBIZ pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et dé-approvisionner automatiquement des comptes d’utilisateur dans BitaBIZ.
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
ms.openlocfilehash: 8eccc3be7da201ee1e2af046c6b515871ef05adc
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350770"
---
# <a name="tutorial-configure-bitabiz-for-automatic-user-provisioning"></a>Tutoriel : Configurer BitaBIZ pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans BitaBIZ et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le retrait automatiques d’utilisateurs et/ou de groupes sur BitaBIZ.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire BitaBIZ](https://bitabiz.dk/en/price/).
* Un compte d’utilisateur BitaBIZ avec des autorisations d’administrateur.

## <a name="assigning-users-to-bitabiz"></a>Attribution d’utilisateurs à BitaBIZ

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à BitaBIZ. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs ou groupes à l’application BitaBIZ en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-bitabiz"></a>Conseils importants pour l’attribution d’utilisateurs dans BitaBIZ

* Dans un premier temps, il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à BitaBIZ afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à BitaBIZ, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-bitabiz-for-provisioning"></a>Configurer BitaBIZ pour l’approvisionnement

Avant de configurer BitaBIZ pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer l’approvisionnement SCIM sur BitaBIZ.

1. Connectez-vous à la [Console d’administration BitaBIZ](https://www.bitabiz.com/login?lang=en). Cliquez sur **CONFIGURATION ADMINISTRATEUR**.

    :::image type="content" source="media/bitabiz-provisioning-tutorial/setup-admin.png" alt-text="Capture d’écran de la console d’administration BitaBIZ, avec l’administrateur d’installation mis en surbrillance." border="false":::

2.  Accédez à **INTEGRATION**.

    :::image type="content" source="media/bitabiz-provisioning-tutorial/integration.png" alt-text="Capture d’écran de la console d’administration BitaBIZ, avec Integration mis en surbrillance." border="false":::

2.  Accédez à **Approvisionnement Microsoft Azure AD**.  Sélectionnez **Activé** dans l’approvisionnement automatique d’utilisateurs. Copiez les valeurs de l’**URL du point de terminaison d’approvisionnement SCIM** et du **Jeton du porteur**. Ces valeurs doivent être entrées dans les champs URL de locataire et Jeton secret dans l’onglet Approvisionnement de votre application BitaBIZ dans le Portail Microsoft Azure.

    ![Ajouter un SCIM BitaBIZ](media/bitabiz-provisioning-tutorial/authentication.png)


## <a name="add-bitabiz-from-the-gallery"></a>Ajouter BitaBIZ depuis la galerie

Pour configurer BitaBIZ pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter BitaBIZ à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter BitaBIZ à partir de la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **BitaBIZ**, sélectionnez **BitaBIZ** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![BitaBIZ dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-bitabiz"></a>Configuration de l’approvisionnement automatique d’utilisateurs dans BitaBIZ 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans BitaBIZ en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour BitaBIZ en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour BitaBIZ](BitaBIZ-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent

### <a name="to-configure-automatic-user-provisioning-for-bitabiz-in-azure-ad"></a>Si vous souhaitez configurer l’attribution automatique d’utilisateurs pour BitaBIZ dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **BitaBIZ**.

    ![Lien BitaBIZ dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section Informations d’identification de l’administrateur, entrez les valeurs d’**URL de point de terminaison d’approvisionnement SCIM** et de **Jeton du porteur** récupérées précédemment dans URL de locataire et Jeton secret, respectivement. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à BitaBIZ. Si la connexion échoue, vérifiez que votre compte BitaBIZ dispose des autorisations d’administrateur, puis réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec BitaBIZ**.

    ![Mappages d’utilisateurs BitaBIZ](media/bitabiz-provisioning-tutorial/usermapping.png)

9. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et BitaBIZ. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d’utilisateur BitaBIZ en vue de mises à jour ultérieures. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur BitaBIZ](media/bitabiz-provisioning-tutorial/user-attribute.png)


10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service d’approvisionnement Azure AD pour BitaBIZ, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou groupes que vous souhaitez approvisionner sur BitaBIZ en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD dans BitaBIZ.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* BitaBIZ requiert **userName**, **email**, **firstName** et **lastName** comme attributs obligatoires. 
* BitaBIZ ne prend pas en charge les suppressions matérielles actuellement.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md).
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md).
