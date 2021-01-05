---
title: "Tutoriel : Configurer StarLeaf pour l'approvisionnement automatique d'utilisateurs avec Azure Active Directory | Microsoft Docs"
description: Apprenez à configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes utilisateur sur StarLeaf.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/19/2019
ms.author: zhchia
ms.openlocfilehash: 07c476c0de644ac63c577d466f4691b5cf415334
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357930"
---
# <a name="tutorial-configure-starleaf-for-automatic-user-provisioning"></a>Tutoriel : Configurer StarLeaf pour l'approvisionnement automatique d'utilisateurs

L'objectif de ce didacticiel est de présenter les étapes à suivre dans StarLeaf et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l'approvisionnement et le retrait automatiques d'utilisateurs et/ou de groupes sur StarLeaf.

> [!NOTE]
>  Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est en préversion. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* Un locataire Azure AD.
* [Un locataire StarLeaf](https://starleaf.com/).
* Un compte d'utilisateur StarLeaf doté d'autorisations d'administrateur.

## <a name="assign-users-to-starleaf"></a>Attribuer des utilisateurs à StarLeaf
Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d'activer l'approvisionnement automatique d'utilisateurs, vous devez déterminer quels utilisateurs et groupes d'Azure AD auront besoin d'accéder à StarLeaf. Vous pourrez ensuite attribuer les utilisateurs et les groupes à StarLeaf en suivant [ces instructions](../manage-apps/assign-user-or-group-access-portal.md).

## <a name="important-tips-for-assigning-users-to-starleaf"></a>Conseils importants pour l'attribution d'utilisateurs à StarLeaf

* Il est recommandé de n'attribuer qu'un seul utilisateur Azure AD à StarLeaf afin de tester la configuration de l'approvisionnement automatique d'utilisateurs. Les autres utilisateurs et groupes pourront être attribués ultérieurement.

* Lorsque vous attribuez un utilisateur à StarLeaf, vous devez sélectionner un rôle valide spécifique à l'application (si disponible) dans la boîte de dialogue d'attribution. Les utilisateurs dont le rôle est Accès par défaut sont exclus de l’approvisionnement.

## <a name="set-up-starleaf-for-provisioning"></a>Configurer StarLeaf pour l'approvisionnement

Avant de configurer StarLeaf pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez configurer l'approvisionnement SCIM dans StarLeaf :

1. Connectez-vous à votre [Console d'administration StarLeaf](https://portal.starleaf.com/#page=login). Accédez à **Intégrations** > **Ajouter une intégration**.

    ![Capture d’écran de la console d’administration StarLeaf avec les options Intégrations et Ajouter une intégration en évidence.](media/starleaf-provisioning-tutorial/image00.png)

2. Pour le **Type** , sélectionnez Microsoft Azure Active Directory. Entrez un nom approprié dans le champ **Nom**. Cliquez sur **Appliquer**.

    ![Capture d’écran de la boîte de dialogue Ajouter une intégration avec les zones de texte Type et Nom en évidence.](media/starleaf-provisioning-tutorial/image01.png)

3.  Les valeurs **URL de base SCIM** et **Jeton d'accès** s'affichent. Ces valeurs doivent être entrées dans les champs **URL de locataire** et **Jeton secret** de l'onglet Approvisionnement de votre application StarLeaf sur le Portail Microsoft Azure. 

    ![Capture d’écran de la boîte de dialogue Modifier l’intégration avec les zones de texte Type, Nom et URL de base SCIM en évidence.](media/starleaf-provisioning-tutorial/image02.png)

## <a name="add-starleaf-from-the-gallery"></a>Ajouter StarLeaf à partir de la galerie

Avant de configurer StarLeaf pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez ajouter StarLeaf à votre liste d'applications SaaS gérées à partir de la galerie d'applications Azure AD.

**Pour ajouter StarLeaf à partir de la galerie d'applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **StarLeaf** , puis sélectionnez **StarLeaf** dans le volet de résultats.
    ![StarLeaf dans la liste des résultats](common/search-new-app.png)

## <a name="configure-automatic-user-provisioning-to-starleaf"></a>Configurer l'approvisionnement automatique d'utilisateurs sur StarLeaf

Cette section vous guide tout au long des étapes de configuration du service d'approvisionnement d'Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans StarLeaf en fonction des attributions d'utilisateurs et/ou de groupes dans Azure AD.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** , puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **StarLeaf**.

    ![Lien StarLeaf dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section Informations d'identification de l'administrateur, entrez respectivement les valeurs de l' **URL de base SCIM** et du **Jeton d'accès** récupérées précédemment dans les champs **URL de locataire** et **Jeton secret**. Cliquez sur **Tester la connexion** pour vérifier qu'Azure AD peut se connecter à StarLeaf. Si la connexion échoue, vérifiez que votre compte StarLeaf dispose d'autorisations d'administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification** , entrez l'adresse e-mail de la personne ou du groupe qui doit recevoir les notifications d'erreur d'approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec StarLeaf**.

    ![Capture d’écran de la section Mappages montrant l’option Synchroniser les utilisateurs Azure Active Directory avec StarLeaf.](media/starleaf-provisioning-tutorial/usermapping.png)

9. Dans la section **Mappages des attributs** , passez en revue les attributs d'utilisateurs synchronisés entre Azure AD et StarLeaf. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d'utilisateur dans StarLeaf pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Capture d’écran de la section Mappages des attributs montrant neuf mappages affichés.](media/starleaf-provisioning-tutorial/userattribute.png)


10. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).


11. Pour activer le service d'approvisionnement Azure AD pour StarLeaf, définissez le paramètre **État d'approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

12. Définissez les utilisateurs et/ou les groupes que vous souhaitez approvisionner sur StarLeaf en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

13. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour analyser la progression et suivre les liens vers les rapports d'activité d'approvisionnement, qui décrit toutes les actions effectuées par le service d'approvisionnement Azure AD sur StarLeaf.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md)

## <a name="connector-limitations"></a>Limitations du connecteur

* StarLeaf ne prend actuellement pas en charge l'approvisionnement des groupes. 
* Dans StarLeaf, **email** et **userName** doivent avoir la même valeur source.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [consulter les journaux d'activité et obtenir des rapports sur l'activité d'approvisionnement](../app-provisioning/check-status-user-account-provisioning.md).
