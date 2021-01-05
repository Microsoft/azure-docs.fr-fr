---
title: 'Tutoriel : Configurer Priority Matrix pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour attribuer et retirer automatiquement des comptes d’utilisateurs sur Priority Matrix.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/08/2019
ms.author: Zhchia
ms.openlocfilehash: e79f21300325c6b451dd564bf2c69830f003f55c
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357859"
---
# <a name="tutorial-configure-priority-matrix-for-automatic-user-provisioning"></a>Tutoriel : Configurer Priority Matrix pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Priority Matrix et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’attribution et le retrait automatiques d’utilisateurs et/ou de groupes sur Priority Matrix.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Priority Matrix](https://appfluence.com/pricing/)
* Un compte d’utilisateur dans Priority Matrix avec des autorisations d’administrateur.

## <a name="assign-users-to-priority-matrix"></a>Affecter des utilisateurs à Priority Matrix

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Priority Matrix. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application Priority Matrix en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-priority-matrix"></a>Conseils importants pour l’attribution d’utilisateurs à Priority Matrix

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Priority Matrix afin de tester la configuration de l’attribution automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Priority Matrix, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-priority-matrix-for-provisioning"></a>Configurer Priority Matrix pour l’approvisionnement

Avant de configurer Priority Matrix pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez récupérer certaines informations de configuration à partir de Priority Matrix.

1. Connectez-vous à votre console d’administration [Priority Matrix](https://sync.appfluence.com/accounts/login/?next=/accounts/provisioning).

3. Cliquez sur **jeton de connexion OAuth** pour Priority Matrix

    ![Ajouter SCIM - Priority Matrix](media/priority-matrix-provisioning-tutorial/oauthlogin.png)

4. Cliquez sur le bouton **OBTENIR UN NOUVEAU JETON**. Copiez la **Chaîne de jeton**. Vous devrez entrer cette valeur dans le champ **Jeton** secret dans l’onglet Approvisionnement de votre application Priority Matrix dans le portail Microsoft Azure. 

    ![Création de jeton Priority Matrix](media/priority-matrix-provisioning-tutorial/token.png)

## <a name="add-priority-matrix-from-the-gallery"></a>Ajouter Priority Matrix à partir de la Galerie

Avant de configurer Priority Matrix pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Priority Matrix à votre liste d’applications SaaS managées à partir de la galerie d’applications Azure AD.

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Priority Matrix** , puis sélectionnez **Priority Matrix** dans le volet de résultats. 

    ![Priority Matrix dans la liste des résultats](common/search-new-app.png)

5. Sélectionnez le bouton **S’inscrire à Priority Matrix** qui vous redirige vers la page de connexion de Priority Matrix. 

    ![Ajouter OIDC - Priority Matrix](media/priority-matrix-provisioning-tutorial/signup.png)

6. Comme Priority Matrix est une application OpenIDConnect, choisissez de vous connecter à Priority Matrix à l’aide de votre compte professionnel Microsoft.

    ![Connexion OIDC - Priority Matrix](media/priority-matrix-provisioning-tutorial/msftsignin.png)

7. Une fois l’authentification réussie, acceptez l’invite de consentement pour la page de consentement. L’application est alors automatiquement ajoutée à votre locataire et vous serez redirigé vers votre compte Priority Matrix.

    ![Consentement OIDC - Priority Matrix](media/priority-matrix-provisioning-tutorial/consent.png)

## <a name="configure-automatic-user-provisioning-to-priority-matrix"></a>Configurer l’approvisionnement automatique d’utilisateurs sur Priority Matrix 

Cette section vous guide tout au long des étapes de configuration du service d’attribution d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Priority Matrix en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!NOTE]
> Pour en savoir plus sur le point de terminaison SCIM de Priority Matrix, consultez [Approvisionnement d’utilisateurs et Priority Matrix](https://appfluence.com/help/article/user-provisioning/).

### <a name="to-configure-automatic-user-provisioning-for-priority-matrix-in-azure-ad"></a>Pour configurer l’attribution automatique d’utilisateurs pour Priority Matrix dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Priority Matrix**.

    ![Lien de Priority Matrix dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez `https://sync.appfluence.com/scim/v2/` dans **URL de locataire**. Entrez la valeur que vous avez récupérée et enregistrée précédemment de Priority Matrix, dans **Secret Token** (Jeton secret). Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Priority Matrix. Si la connexion échoue, vérifiez que votre compte Priority Matrix dispose des autorisations d’administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchronize Azure Active Directory Users to Priority Matrix** (Synchroniser les utilisateurs Azure Active Directory avec Priority Matrix).

    ![Mappages d’utilisateur Priority Matrix](media/priority-matrix-provisioning-tutorial/usermappings.png)

9. Dans la section **Mappage d’attributs** , passez en revue les attributs d’utilisateur qui sont synchronisés entre Azure AD et Priority Matrix. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes utilisateur dans Priority Matrix pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs d’utilisateur Priority Matrix](media/priority-matrix-provisioning-tutorial/userattributes.png)

10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service d’approvisionnement Azure AD pour Priority Matrix, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou groupes que vous souhaitez attribuer sur Priority Matrix en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’attribution, qui décrivent toutes les actions effectuées par le service d’attribution Azure AD sur Priority Matrix.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)


