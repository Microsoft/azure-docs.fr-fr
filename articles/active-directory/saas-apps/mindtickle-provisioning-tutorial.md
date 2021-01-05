---
title: "Tutoriel : Configurer MindTickle pour l'approvisionnement automatique d'utilisateurs avec Azure Active Directory | Microsoft Docs"
description: Découvrez comment configurer Azure Active Directory pour approvisionner et supprimer automatiquement des comptes d’utilisateur dans MindTickle.
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
ms.openlocfilehash: c158358b1c4fcd72d9189d7a991645cb65a4dc83
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348124"
---
# <a name="tutorial-configure-mindtickle-for-automatic-user-provisioning"></a>Tutoriel : Configurer MindTickle pour l'approvisionnement automatique d'utilisateurs

L'objectif de ce didacticiel est de présenter les étapes à suivre dans MindTickle et Azure Active Directory (Azure AD) afin de configurer Azure AD pour l'approvisionnement et le retrait automatiques d'utilisateurs et/ou de groupes sur MindTickle.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Ce connecteur est actuellement en préversion publique. Pour plus d’informations sur les conditions d’utilisation Microsoft Azure générales relatives aux fonctionnalités d’évaluation, consultez [Conditions d’utilisation supplémentaires des préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire MindTickle](https://www.mindtickle.com/)
* Un compte d'utilisateur MindTickle doté d'autorisations d'administrateur.

## <a name="assigning-users-to-mindtickle"></a>Affectation d’utilisateurs à MindTickle

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes Azure AD ont besoin d’accéder à MindTickle. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à MindTickle en suivant les instructions fournies ici :
* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-mindtickle"></a>Conseils importants pour l’attribution d’utilisateurs dans MindTickle

* Il est recommandé de n'attribuer qu'un seul utilisateur Azure AD à MindTickle afin de tester la configuration de l'approvisionnement automatique d'utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous attribuez un utilisateur à MindTickle, vous devez sélectionner un rôle valide spécifique à l'application (si disponible) dans la boîte de dialogue d'attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="setup-mindtickle-for-provisioning"></a>Configurer MindTickle pour l’approvisionnement

Avant de configurer MindTickle pour l’approvisionnement automatique d’utilisateurs avec Azure AD, vous devez activer le l’approvisionnement SCIM sur MindTickle.


1.  Contactez [l’équipe de support de MindTickle](mailto:help@mindtickle.com) pour obtenir le jeton JWT nécessaire à la configuration de l’approvisionnement SCIM.


## <a name="add-mindtickle-from-the-gallery"></a>Ajout de MindTickle à partir de la galerie

Avant de configurer MindTickle pour l'approvisionnement automatique d'utilisateurs avec Azure AD, vous devez ajouter MindTickle à votre liste d'applications SaaS gérées à partir de la galerie d'applications Azure AD.

**Pour ajouter MindTickle à partir de la galerie d'applications Azure AD, procédez comme suit :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **MindTickle**, sélectionnez **MindTickle** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l'application.

    ![MindTickle dans la liste des résultats](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-mindtickle"></a>Configuration de l’approvisionnement automatique d’utilisateurs dans MindTickle 

Cette section vous guide tout au long des étapes de configuration du service d'approvisionnement d'Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans MindTickle en fonction des attributions d'utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez aussi choisir d’activer l’authentification unique basée sur SAML pour MindTickle en suivant les instructions fournies dans le [tutoriel sur l’authentification unique pour MindTickle](mindtickle-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent

### <a name="to-configure-automatic-user-provisioning-for-mindtickle-in-azure-ad"></a>Pour configurer l'approvisionnement automatique d'utilisateurs pour MindTickle dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **MindTickle**.

    ![Lien MindTickle dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Dans la section **Informations d’identification de l’administrateur**, entrez `https://admin.mindtickle.com/scim` dans **URL de locataire**. Entrez la valeur de **Jeton JWT** extraite précédemment dans la zone de texte Jeton secret, puis entrez la valeur du **Jeton JWT** fournie par l’équipe de support MindTickle. Cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à myPolicies. Si la connexion échoue, vérifiez que votre compte MindTickle dispose d'autorisations d'administrateur et réessayez.

    ![URL de locataire + Jeton](common/provisioning-testconnection-tenanturltoken.png)

6. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

7. Cliquez sur **Enregistrer**.

8. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur MindTickle**.

    :::image type="content" source="media/mindtickle-provisioning-tutorial/usermapping.png" alt-text="Capture d’écran de la section Mappages. Sous Nom, Synchroniser les utilisateurs Azure Active Directory avec MindTickle est visible." border="false":::

9. Dans la section **Mappage des attributs**, passez en revue les attributs d'utilisateurs synchronisés entre Azure AD et MindTickle. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d'utilisateur dans MindTickle pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    :::image type="content" source="media/mindtickle-provisioning-tutorial/userattribute.png" alt-text="Capture d’écran de la page Mappages d’attributs. Un tableau liste les attributs Azure Active Directory et MindTickle et la priorité de correspondance." border="false":::

12. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Pour activer le service d'approvisionnement Azure AD pour MindTickle, définissez le paramètre **État d'approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

14. Définissez les utilisateurs et/ou groupes à approvisionner sur MindTickle en choisissant les valeurs souhaitées dans le champ **Étendue** de la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

15. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Pour plus d’informations sur la durée d’approvisionnement des utilisateurs et/ou des groupes, voir [Combien de temps faut-il pour approvisionner des utilisateurs ?](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users). 

Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement Azure AD sur MindTickle. Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Pour la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
