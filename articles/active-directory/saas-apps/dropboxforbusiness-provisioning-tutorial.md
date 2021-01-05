---
title: 'Tutoriel : Configurer Dropbox Business pour le provisionnement automatique des utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes utilisateur sur Dropbox Business.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: jeedes
ms.openlocfilehash: 04d17e17ef11696efd52f04ea83639f2a9b81fea
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938747"
---
# <a name="tutorial-configure-dropbox-for-business-for-automatic-user-provisioning"></a>Tutoriel : Configurer Dropbox for Business pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Dropbox Business et Azure Active Directory (Azure AD) afin de configurer Azure AD pour le provisionnement et le déprovisionnement automatiques d’utilisateurs et/ou de groupes sur Dropbox Business.

> [!IMPORTANT]
> Microsoft et Dropbox déconseilleront l’ancienne intégration de Dropbox à partir du 01/04/2021. Pour éviter toute interruption de service, nous recommandons de migrer vers la nouvelle intégration de Dropbox qui prend en charge les groupes. Pour migrer vers la nouvelle intégration de Dropbox, ajoutez et configurez une nouvelle instance de Dropbox pour l’approvisionnement dans votre locataire Azure AD en suivant les étapes ci-dessous. Une fois que vous avez configuré la nouvelle intégration de Dropbox, désactivez l’approvisionnement de l’ancienne intégration de Dropbox pour éviter les conflits d’approvisionnement. Pour obtenir des instructions plus détaillées sur la migration vers la nouvelle intégration de Dropbox, consultez [Comment passer à la dernière version de l’application Dropbox Business utilisant Azure AD](https://help.dropbox.com/installs-integrations/third-party/update-dropbox-azure-ad-connector).

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce tutoriel part du principe que vous disposez des prérequis suivants :

* un locataire Azure AD ;
* [Un locataire Dropbox Business](https://www.dropbox.com/business/pricing)
* Un compte d’utilisateur dans Dropbox Business avec des autorisations d’administrateur

## <a name="add-dropbox-for-business-from-the-gallery"></a>Ajouter Dropbox Business à partir de la galerie

Avant de configurer Dropbox Business pour l’attribution automatique des utilisateurs avec Azure AD, vous devez ajouter Dropbox Business à votre liste d’applications SaaS managées, à partir de la galerie d’applications Azure AD.

**Pour ajouter Dropbox Business à partir de la galerie d’applications Azure AD, effectuez les étapes suivantes :**

1. Dans le panneau de navigation gauche du **[portail Azure](https://portal.azure.com)** , sélectionnez **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une nouvelle application, cliquez sur le bouton **Nouvelle application** en haut du volet.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Dropbox Business**, sélectionnez **Dropbox Business** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Dropbox for Business dans la liste des résultats](common/search-new-app.png)

## <a name="assigning-users-to-dropbox-for-business"></a>Assignation d’utilisateurs à Dropbox for Business

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre d’une attribution automatique d’utilisateurs, seuls les utilisateurs ou les groupes auxquels une application a été attribuée dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’attribution automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes d’Azure AD ont besoin d’accéder à Dropbox Business. Une fois que vous avez choisi, vous pouvez attribuer ces utilisateurs et/ou groupes à votre application Dropbox Business en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-dropbox-for-business"></a>Conseils importants concernant l’assignation d’utilisateurs à Dropbox for Business

* Il est recommandé de n’attribuer qu’un seul utilisateur Azure AD à Dropbox Business afin de tester la configuration de l’attribution d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous attribuez un utilisateur à Dropbox Business, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configuring-automatic-user-provisioning-to-dropbox-for-business"></a>Configuration de Dropbox Business pour l’attribution automatique d’utilisateurs 

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Dropbox Business en fonction des attributions d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Dropbox Business en suivant les instructions fournies dans le [tutoriel sur l’authentification unique Dropbox Business](dropboxforbusiness-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-dropbox-for-business-in-azure-ad"></a>Pour configurer l’attribution automatique d’utilisateurs pour Dropbox Business dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise**, puis **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Dropbox for Business**.

    ![Lien Dropbox for Business dans la liste Applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Capture d’écran des options Gérer avec l’option Provisionnement en évidence.](common/provisioning.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran de la liste déroulante Mode de provisionnement avec l’option Automatique en évidence.](common/provisioning-automatic.png)

5. Sous la section **informations d’identification de l’administrateur**, cliquez sur **Autoriser**. Cela permet d’ouvrir une boîte de dialogue de connexion à Dropbox for Business dans une nouvelle fenêtre de navigateur.

    ![Approvisionnement ](common/provisioning-oauth.png)

6. Dans la boîte de dialogue **Se connecter à Dropbox Business pour établir une connexion à Azure AD**, connectez-vous à votre locataire Dropbox Business, puis confirmez votre identité.

    ![Connexion à Dropbox Business](media/dropboxforbusiness-provisioning-tutorial/dropbox01.png)

7. Lorsque vous aurez effectué les étapes 5 et 6, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à votre application Dropbox Business. Si la connexion échoue, vérifiez que votre compte Dropbox Business dispose d’autorisations d’administrateur, puis réessayez.

    ![par jeton](common/provisioning-testconnection-oauth.png)

8. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification](common/provisioning-notification-email.png)

9. Cliquez sur **Enregistrer**.

10. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Dropbox Business**.

    ![Mappages d’utilisateurs Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-user-mapping.png)

11. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Dropbox. Les attributs sélectionnés en tant que propriétés **correspondantes** sont utilisés pour établir une correspondance avec les comptes d’utilisateur Dropbox en vue de mises à jour ultérieures. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs utilisateur Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-user-attributes.png)

12. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Dropbox**.

    ![Mappages de groupes Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-group-mapping.png)

13. Dans la section **Mappage d’attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Dropbox. Les attributs sélectionnés comme propriétés **correspondantes** sont utilisés pour établir une correspondance avec les groupes Dropbox en vue de mises à jour ultérieures. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Attributs de groupes Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-group-attributes.png)

14. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Pour activer le service de provisionnement Azure AD pour Dropbox, définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement activé](common/provisioning-toggle-on.png)

16. Définissez les utilisateurs et/ou groupes que vous souhaitez attribuer dans Dropbox en choisissant les valeurs souhaitées sous **Étendue**, dans la section **Paramètres**.

    ![Étendue de l’approvisionnement](common/provisioning-scope.png)

17. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Enregistrement de la configuration de l’approvisionnement](common/provisioning-configuration-save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour superviser la progression et les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD dans Dropbox.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur
 
* Dropbox ne prend pas en charge la suspension des utilisateurs invités. Si un utilisateur invité est suspendu, celui-ci est supprimé.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

