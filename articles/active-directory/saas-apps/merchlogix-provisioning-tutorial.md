---
title: 'Tutoriel : Configurer MerchLogix pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour attribuer et supprimer automatiquement des comptes d’utilisateur dans MerchLogix.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: zhchia
ms.openlocfilehash: 4d0a52f06a751fba57a00615e2d57485ff740d04
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359695"
---
# <a name="tutorial-configure-merchlogix-for-automatic-user-provisioning"></a>Tutoriel : Configurer MerchLogix pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans MerchLogix et Azure Active Directory (Azure AD) dans le but de configurer Azure AD pour l’attribution et la suppression automatiques d’utilisateurs et/ou de groupes dans MerchLogix.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* Un locataire MerchLogix ;
* Un contact technique chez MerchLogix pouvant fournir l’URL de point de terminaison SCIM et le jeton secret requis pour l’approvisionnement d’utilisateurs

## <a name="adding-merchlogix-from-the-gallery"></a>Ajout de Merchlogix à partir de la galerie

Avant de configurer MerchLogix pour l’attribution automatique d’utilisateurs avec Azure AD, vous devez ajouter MerchLogix à votre liste d’applications SaaS managées, à partir de la galerie d’applications Azure AD.

**Pour ajouter MerchLogix à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**. 

    ![Bouton Azure Active Directory][1]

2. Accédez à **Applications d’entreprise** > **Toutes les applications**.

    ![Section Applications d’entreprise][2]

3. Pour ajouter MerchLogix, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application][3]

4. Dans la zone de recherche, tapez **MerchLogix**.

5. Dans le volet de résultats, sélectionnez **MerchLogix** , puis cliquez sur le bouton **Ajouter** pour ajouter MerchLogix à votre liste d’applications SaaS.

    ![Capture d’écran de la section Ajouter à partir de la galerie avec la zone de texte Entrez un nom en évidence.][4]

## <a name="assigning-users-to-merchlogix"></a>Attribution d’utilisateurs à MerchLogix

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs et/ou les groupes qui ont été « assignés » à une application dans Azure AD sont synchronisés. 

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes Azure AD ont besoin d’accéder à MerchLogix. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application MerchLogix en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-merchlogix"></a>Conseils importants pour l’attribution d’utilisateurs dans MerchLogix

* Il est recommandé d’assigner un seul utilisateur Azure AD à MerchLogix afin de tester la configuration initiale du provisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être assignés ultérieurement, une fois les tests réussis.

* Quand vous attribuez un utilisateur à MerchLogix, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configuring-automatic-user-provisioning-to-merchlogix"></a>Configuration de l’attribution automatique d’utilisateurs dans MerchLogix 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans MerchLogix en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour MerchLogix en suivant les instructions fournies dans le [tutoriel sur l’authentification unique MerchLogix](merchlogix-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-merchlogix-in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs MerchLogix dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

2. Sélectionnez MerchLogix dans votre liste d’applications SaaS.

3. Sélectionnez l’onglet **Approvisionnement**.

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la section MerchLogix – Approvisionnement avec l’option Approvisionnement en évidence, le Mode d’approvisionnement défini sur Automatique et l’option Tester la connexion en évidence.](./media/merchlogix-provisioning-tutorial/Merchlogix1.png)

5. Sous la section **informations d’identification de l’administrateur** :

    * Dans le champ **URL de locataire** , entrez l’URL de point de terminaison SCIM fourni par votre contact technique MerchLogix.

    * Dans le champ **Jeton secret** , entrez le jeton secret fourni par votre contact technique MerchLogix.

6. Après avoir renseigné les champs indiqués à l’étape 5, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à MerchLogix. Si la connexion échoue, vérifiez que votre compte MerchLogix dispose des autorisations d’administrateur, puis réessayez.

7. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

8. Cliquez sur **Enregistrer**.

9. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur MerchLogix**.

10. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et MerchLogix. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour établir une correspondance avec les comptes d’utilisateur MerchLogix en vue de mises à jour ultérieures. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

11. Dans la section **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec MerchLogix**.

12. Dans la section **Mappages des attributs** , passez en revue les attributs groupe qui sont synchronisés entre Azure AD et MerchLogix. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans MerchLogix pour les opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

13. Pour activer le service d’approvisionnement Azure AD pour MerchLogix, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

14. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD dans MerchLogix.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: common/select-azuread.png
[2]: common/enterprise-applications.png
[3]: common/add-new-app.png
[4]: common/search-new-app.png
