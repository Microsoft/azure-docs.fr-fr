---
title: 'Tutoriel : Configurer Storegate pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur sur Storegate.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/15/2019
ms.author: Zhchia
ms.openlocfilehash: c984beff630ef90ea33a13e2fef1bca0189c2314
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357935"
---
# <a name="tutorial-configure-storegate-for-automatic-user-provisioning"></a>Tutoriel : Configurer Storegate pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans Storegate et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l’approvisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur Storegate.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Storegate](https://www.storegate.com)
* Un compte d’utilisateur sur un Storegate avec des autorisations Administrateur.

## <a name="assign-users-to-storegate"></a>Attribuer des utilisateurs à Storegate

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Storegate. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Storegate en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-storegate"></a>Conseils importants pour l’attribution d’utilisateurs à Storegate

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Storegate afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Storegate, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="set-up-storegate-for-provisioning"></a>Configurer Storegate pour l’approvisionnement

Avant de configurer Storegate pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez récupérer certaines informations d’approvisionnement à partir de Storegate.

1. Connectez-vous à votre [Console Administrateur Storegate](https://ws1.storegate.com/identity/core/login?signin=c71fb8fe18243c571da5b333d5437367) et accédez aux paramètres en cliquant sur l’icône utilisateur dans le coin supérieur droit, puis sélectionnez **Paramètres de compte**.

    ![Storegate Ajouter SCIM](media/storegate-provisioning-tutorial/admin.png)

2. Dans Paramètres, accédez à **Équipe > Paramètres** et vérifiez que le bouton bascule est activé dans la section **Authentification unique**.

    ![Paramètres Storegate](media/storegate-provisioning-tutorial/team.png)

    ![Bouton bascule Storegate](media/storegate-provisioning-tutorial/sso.png)

3. Copiez l’ **URL de locataire** et le **Jeton**. Ces valeurs doivent être entrées dans les champs **URL de locataire** et **Jeton secret** respectivement de l’onglet Approvisionnement de votre application Storegate sur le Portail Azure. 

    ![Storegate Créer le jeton](media/storegate-provisioning-tutorial/token.png)

## <a name="add-storegate-from-the-gallery"></a>Ajouter Storegate à partir de la galerie

Avant de configurer Storegate pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Storegate à votre liste d’applications SaaS managées à partir de la galerie d’applications Azure AD.

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Storegate** , puis sélectionnez **Storegate**  dans le volet de résultats. 

    ![Storegate dans la liste des résultats](common/search-new-app.png)

5. Sélectionnez le bouton **S’inscrire à Storegate**  qui vous redirigera vers la page de connexion de Storegate. 

    ![Storegate OIDC Ajouter](media/storegate-provisioning-tutorial/signup.png)

6.  Connectez-vous à votre [Console Administrateur Storegate](https://ws1.storegate.com/identity/core/login?signin=c71fb8fe18243c571da5b333d5437367) et accédez aux paramètres en cliquant sur l’icône utilisateur dans le coin supérieur droit, puis sélectionnez **Paramètres de compte**.

    ![Connexion Storegate](media/storegate-provisioning-tutorial/admin.png)

7. Dans Paramètres, accédez à **Équipe > Paramètres** et cliquez sur le bouton bascule dans la section Authentification unique pour démarrer le flux de consentement. Cliquez sur **Activer**.

    ![Équipe Storegate](media/storegate-provisioning-tutorial/team.png)

    ![Authentification unique Storegate](media/storegate-provisioning-tutorial/sso.png)

    ![Storegate activer](media/storegate-provisioning-tutorial/activate.png)

8. Comme Storegate est une application OpenIDConnect, choisissez de vous connecter à Storegate à l’aide de votre compte professionnel Microsoft.

    ![Storegate OIDC connexion](media/storegate-provisioning-tutorial/login.png)

9. Une fois l’authentification réussie, acceptez l’invite de consentement pour la page de consentement. L’application est alors automatiquement ajoutée à votre locataire et vous serez redirigé vers votre compte Storegate.

    ![Storegate OIDC consentement](media/storegate-provisioning-tutorial/accept.png)

## <a name="configure-automatic-user-provisioning-to-storegate"></a>Configurer l’approvisionnement automatique d’utilisateurs sur Storegate 

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Storegate en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!NOTE]
> Pour en savoir plus sur le point de terminaison SCIM de Storegate, reportez-vous à ce [document](https://en-support.storegate.com/article/step-by-step-instruction-how-to-enable-azure-provisioning-to-your-storegate-team-account/).

### <a name="to-configure-automatic-user-provisioning-for-storegate-in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Storegate dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Storegate**.

    ![Lien Storegate dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez `https://dialpad.com/scim` dans **URL de locataire**. Entrez la valeur que vous avez récupérée et enregistrée précédemment à partir de Storegate dans **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Storegate. Si la connexion échoue, vérifiez que votre compte Storegate dispose d’autorisations Administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Storegate**.

    ![Mappages d’utilisateurs Storegate](media/storegate-provisioning-tutorial/usermappings.png)

9. Dans la section **Mappages des attributs** , passez en revue les attributs d’utilisateurs qui sont synchronisés entre Azure AD et Storegate. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Storegate pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Storegate](media/storegate-provisioning-tutorial/userattributes.png)

10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Pour activer le service d’approvisionnement Azure AD pour Storegate, attribuez la valeur **Activé** au paramètre **État de l’approvisionnement** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Storegate en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour analyser la progression et suivre les liens vers le rapport d’activité d’approvisionnement, qui décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur Storegate.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
