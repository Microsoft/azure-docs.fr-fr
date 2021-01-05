---
title: 'Tutoriel : Configurer Signagelive pour le provisionnement d’utilisateurs automatique avec Azure Active Directory | Microsoft Docs'
description: Apprenez à configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d'utilisateur sur Signagelive.
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
ms.openlocfilehash: 10ad06041e8136b5661b1b1ff487cd4d3b0f5153
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358403"
---
# <a name="tutorial-configure-signagelive--for-automatic-user-provisioning"></a>Tutoriel : Configurer Signagelive pour l’approvisionnement automatique d’utilisateurs

L'objectif de ce didacticiel est de présenter les étapes à suivre dans Signagelive et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l'approvisionnement et le retrait automatiques d'utilisateurs et/ou de groupes sur Signagelive.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire Signagelive](https://signagelive.com/pricing/)
* Un compte d’utilisateur dans Signagelive avec des autorisations d’administration.

## <a name="assigning-users-to-signagelive"></a>Affectation d’utilisateurs à Signagelive   

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d'activer l'approvisionnement automatique d'utilisateurs, vous devez déterminer quels utilisateurs et/ou groupes d'Azure AD auront besoin d'accéder à Signagelive. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à Signagelive en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-signagelive"></a>Conseils importants pour l'attribution d'utilisateurs à Signagelive   

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Signagelive afin de tester la configuration de l’approvisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous assignez un utilisateur à Signagelive, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-signagelive--for-provisioning"></a>Configurer Signagelive pour l’approvisionnement

Avant de configurer Signagelive pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer l’approvisionnement SCIM sur Signagelive.

  Contactez [Signagelive](mailto:development@signagelive.com) pour obtenir le jeton secret nécessaire à la configuration de l’approvisionnement SCIM.

## <a name="add-signagelive-from-the-gallery"></a>Ajouter Signagelive à partir de la galerie

Avant de configurer Signagelive pour le provisionnement d’utilisateurs automatique avec Azure AD, vous avez besoin d’ajouter Signagelive depuis la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Signagelive depuis la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Signagelive** , sélectionnez **Signagelive** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Signagelive    dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-signagelive"></a>Configurer l'approvisionnement automatique d'utilisateurs sur Signagelive    

Cette section vous guide tout au long des étapes de configuration du service d’approvisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Signagelive en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
>  Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Signagelive en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Signagelive](Signagelive-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-signagelive--in-azure-ad"></a>Pour configurer l’approvisionnement automatique d’utilisateurs pour Signagelive dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Signagelive**.

    ![Lien Signagelive dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section Informations d’identification de l’administrateur, entrez ` https://samlapi.signagelive.com/scim/v2` dans **URL de locataire**. Dans le champ **Jeton secret** , entrez la valeur du **Jeton du porteur** fournie par l’équipe de développement. Cliquez sur **Tester la connexion** pour vérifier qu'Azure AD peut se connecter à Signagelive. Si la connexion échoue, vérifiez que votre compte Signagelive dispose des autorisations d’administrateur et réessayez.
    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Signagelive**.

    ![Capture d’écran de la section Mappages avec l’option Synchroniser les utilisateurs Azure Active Directory avec Signagelive en évidence.](media/signagelive-provisioning-tutorial/usermapping.png)

9. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Signagelive. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes utilisateur dans Signagelive pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappages des attributs avec sept mappages affichés.](media/signagelive-provisioning-tutorial/userattribute.png)

10. Dans la section **Mappages** , sélectionnez **Synchroniser le groupe Azure Active Directory sur Signagelive**.

    ![Capture d’écran de la section Mappages avec l’option Synchroniser le Groupe Azure Active Directory avec Signagelive en évidence.](media/signagelive-provisioning-tutorial/groupmapping.png)

11. Dans la section **Mappages des attributs** , passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Signagelive. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes de groupe dans Signagelive pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappages des attributs avec trois mappages affichés.](media/signagelive-provisioning-tutorial/groupattribute.png)

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d’approvisionnement Azure AD pour Signagelive, définissez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur Signagelive en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur la durée d’approvisionnement des utilisateurs et/ou des groupes, voir [Combien de temps faut-il pour approvisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users). 

Vous pouvez utiliser la section **État actuel** pour superviser la progression et suivre les liens vers vos rapports d’activités de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD sur Signagelive. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
