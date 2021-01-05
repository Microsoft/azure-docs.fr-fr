---
title: 'Tutoriel : Configurer Flock pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes utilisateur sur Flock.
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
ms.openlocfilehash: 01c3f6429d2a5c8443ac128d763033dc8c53cbc7
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359406"
---
# <a name="tutorial-configure-flock-for-automatic-user-provisioning"></a>Tutoriel : Configurer Flock pour l’attribution automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans Flock et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le retrait automatiques d’utilisateurs et/ou de groupes sur Flock.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire Flock](https://flock.com/pricing/)
* Un compte d’utilisateur dans Flock avec des autorisations d’administrateur.

## <a name="assigning-users-to-flock"></a>Affectation d’utilisateurs à Flock 

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Flock. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Flock en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-flock"></a>Conseils importants pour l’attribution d’utilisateurs à Flock 

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Flock afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Flock, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue Attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-flock--for-provisioning"></a>Configurer Flock pour l’approvisionnement

Avant de configurer Flock pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer l’approvisionnement SCIM sur Flock.

1. Connectez-vous à [Flock](https://web.flock.com/?). Cliquez sur **Icône Paramètres** > **Gérer votre équipe**.

    :::image type="content" source="media/flock-provisioning-tutorial/icon.png" alt-text="Capture d’écran du site web Flock. L’icône des paramètres est mise en surbrillance et son menu contextuel est visible. Dans ce menu, Gérer votre équipe est mis en surbrillance." border="false":::

2. Sélectionnez **Authentification et approvisionnement**.

    :::image type="content" source="media/Flock-provisioning-tutorial/auth.png" alt-text="Capture d’écran d’un menu sur le site web Flock. L’élément Authentification et provisionnement est mis en surbrillance." border="false":::

3. Copiez le jeton d’API ( **API Token** ). Vous devrez entrer ces valeurs dans le champ **Jeton secret** dans l’onglet Approvisionnement de votre application Flock dans le portail Microsoft Azure.

    :::image type="content" source="media/Flock-provisioning-tutorial/provisioning.png" alt-text="Capture d’écran d’un onglet Provisionnement sur le site web Flock. Sous Jeton d’API, une valeur est mise en surbrillance. En regard du jeton se trouve un bouton Copier le jeton." border="false":::


## <a name="add-flock--from-the-gallery"></a>Ajouter Flock à partir de la galerie

Avant de configurer Flock pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Flock à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Flock depuis la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Flock** , sélectionnez **Flock** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Flock dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-flock"></a>Configuration de l’approvisionnement automatique d’utilisateurs sur Flock  

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Flock en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Flock en suivant les instructions fournies dans le [didacticiel sur l’authentification unique pour Flock](Flock-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent

### <a name="to-configure-automatic-user-provisioning-for-flock--in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs pour Flock dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Flock**.

    ![Lien Flock dans la liste Applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section Informations d’identification de l’administrateur, entrez respectivement les valeurs de `https://api.flock-staging.com/v2/scim` et du **Jeton d’API** récupérées précédemment dans les champs **URL de locataire** et **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Flock. Si la connexion échoue, vérifiez que votre compte Flock dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Flock**.

    ![Mappages d’utilisateurs Flock](media/flock-provisioning-tutorial/usermapping.png)

9. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Flock. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateurs dans Flock pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Flock](media/flock-provisioning-tutorial/userattribute.png)

11. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Pour activer le service d’approvisionnement Azure AD pour Flock, modifiez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

13. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Flock en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

14. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur la durée d’approvisionnement des utilisateurs et/ou des groupes, voir [Combien de temps faut-il pour approvisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

Vous pouvez utiliser la section **État actuel** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD sur Flock. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).



## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)