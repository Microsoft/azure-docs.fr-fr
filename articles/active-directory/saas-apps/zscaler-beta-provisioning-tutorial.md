---
title: 'Tutoriel : Configurer Zscaler Beta pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et retirer automatiquement des comptes d’utilisateur sur Zscaler Beta.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 59784b6e618e875a6eedc947ce75afdf3bd1df74
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327387"
---
# <a name="tutorial-configure-zscaler-beta-for-automatic-user-provisioning"></a>Tutoriel : Configurer Zscaler Beta pour le provisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de présenter les étapes à effectuer dans Zscaler Beta et Azure Active Directory (Azure AD) afin de configurer Azure AD pour le provisionnement et le retrait automatiques d’utilisateurs et/ou de groupes sur Zscaler Beta.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).
>


## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* un locataire Azure AD ;
* Un abonné Zscaler Beta
* Un compte d’utilisateur Zscaler Beta ayant des autorisations d’administrateur

> [!NOTE]
> L’intégration de l’approvisionnement Azure AD s’appuie sur l’API SCIM Zscaler Beta, qui est disponible pour les développeurs Zscaler Beta disposant de comptes pourvus du package Enterprise.

## <a name="adding-zscaler-beta-from-the-gallery"></a>Ajouter Zscaler Beta à partir de la galerie

Avant de configurer Zscaler Beta pour le provisionnement automatique d’utilisateurs avec Azure AD, vous devez ajouter Zscaler Beta à partir de la galerie d’applications Azure AD à votre liste d’applications SaaS managées.

**Pour ajouter Zscaler Beta à partir de la galerie d’applications Azure AD, procédez comme suit :**

1. Dans le volet de navigation gauche du **[portail Azure](https://portal.azure.com)** , cliquez sur l’icône **Azure Active Directory**.

    ![Bouton Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise**, puis sélectionnez l’option **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter l’application, cliquez sur le bouton **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, tapez **Zscaler Beta**, sélectionnez **Zscaler Beta** dans le volet de résultats, puis cliquez sur le bouton **Ajouter** pour ajouter l’application.

    ![Zscaler Beta dans la liste des résultats](common/search-new-app.png)

## <a name="assigning-users-to-zscaler-beta"></a>Affectation d’utilisateurs à Zscaler Beta

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs et/ou les groupes qui ont été « assignés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, vous devez décider quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à Zscaler Beta. Une fois que vous avez choisi, vous pouvez assigner ces utilisateurs et/ou groupes à votre application Zscaler Beta en suivant les instructions fournies ici :

* [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-zscaler-beta"></a>Conseils importants pour l’attribution d’utilisateurs à Zscaler Beta

* Il est recommandé de n’affecter qu’un seul utilisateur Azure AD à Zscaler Beta afin de tester la configuration du provisionnement automatique d’utilisateurs. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous assignez un utilisateur à Zscaler Beta, vous devez sélectionner un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’assignation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configuring-automatic-user-provisioning-to-zscaler-beta"></a>Configuration du provisionnement automatique d’utilisateurs sur Zscaler Beta

Cette section vous guide tout au long des étapes de configuration du service de provisionnement d’Azure AD pour créer, mettre à jour et désactiver des utilisateurs et/ou des groupes dans Zscaler Beta en fonction des assignations d’utilisateurs et/ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Zscaler Beta en suivant les instructions fournies dans le [didacticiel sur l’authentification unique Zscaler Beta](zscaler-beta-tutorial.md). L’authentification unique peut être configurée indépendamment de l’attribution automatique d’utilisateurs, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-provisioning-for-zscaler-beta-in-azure-ad"></a>Pour configurer le provisionnement automatique d’utilisateurs pour Zscaler Beta dans Azure AD :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez **Applications d’entreprise**, **Toutes les applications**, puis **Zscaler Beta**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Zscaler Beta**.

    ![Lien Zscaler Beta dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Il existe une liste d’onglets organisés en catégories et intitulés ZScaler Beta - Provisioning / Enterprise Application. L’onglet Provisioning (Provisionnement) de la catégorie Manage (Gérer) est sélectionnée.](./media/zscaler-beta-provisioning-tutorial/provisioning-tab.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Le mode Automatic (Automatique) a été sélectionné dans la liste déroulante Provisioning Mode (Mode de provisionnement). Il existe des champs pour Admin Credentials (Informations d’identification d’administrateur), qui servent à se connecter à l’API Zscaler Beta, ainsi qu’un bouton Test Connection (Tester la connexion).](./media/zscaler-beta-provisioning-tutorial/provisioning-credentials.png)

5. Dans la section **Admin Credentials** (Informations d’identification d’administrateur), entrez les valeurs **Tenant URL** (URL de locataire) et **Secret Token** (Jeton secret) de votre compte Zscaler Beta, comme cela est décrit dans l’étape 6.

6. Pour obtenir les valeurs **Tenant URL** (URL de locataire) et **Secret Token** (Jeton secret), accédez à **Administration > Authentication Settings** (Paramètres d’authentification) dans l’interface utilisateur du portail Zscaler Beta, puis sélectionnez **SAML** sous **Authentication Type** (Type d’authentification).

    ![Dans Authentication Settings (Paramètres d’authentification), dans Authentication Profil (Profil d’authentification), le type de répertoire (Directory Type) défini est Hosted DB (Base de données hébergée) et le type d’authentification (Authentication Type) défini est SAML.](./media/zscaler-beta-provisioning-tutorial/secret-token-1.png)

    Cliquez sur **Configurer SAML** pour ouvrir les options de **Configuration SAML**.

    ![Dans Configure SAML (Configurer SAML), les options Enable SAML Auto-Provisioning (Activer le provisionnement automatique SAML) et Enable SCIM-Based Provisioning (Activer le provisionnement basé sur SCIM) sont sélectionnées. Les cases à cocher Base URL (URL de base) et Bearer Token (Jeton du porteur) sont mises en évidence.](./media/zscaler-beta-provisioning-tutorial/secret-token-2.png)

    Sélectionnez **Enable SCIM-Based Provisioning** (Activer le provisionnement SCIM) pour copier les valeurs **Base URL** (URL de base) et **Bearer Token** (Jeton du porteur), puis enregistrez les paramètres. Copiez **Base URL** (URL de base) dans **Tenant URL** (URL de locataire) et **Bearer Token** (Jeton du porteur) sur **Secret Token** (Jeton secret) dans le portail Azure.

7. Après avoir renseigné les champs indiqués à l’étape 5, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à Zscaler Beta. Si la connexion échoue, vérifiez que votre compte Zscaler Beta dispose des autorisations d’administrateur et réessayez.

    ![Dans Admin Credentials (Informations d’identification d’administrateur), les champs Tenant URL (URL de locataire) et Secret Token (Jeton secret) sont renseignés et le bouton Test Connection (Tester la connexion) est en évidence.](./media/zscaler-beta-provisioning-tutorial/test-connection.png)

8. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![La zone de texte Notification Email (E-mail de notification) est vide et la case Send an email notification when a failure occurs (Envoyer une notification par e-mail lorsqu’une défaillance se produit) est décochée.](./media/zscaler-beta-provisioning-tutorial/notification.png)

9. Cliquez sur **Enregistrer**.

10. Dans la section **Mappages**, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Zscaler Beta**.

    ![Synchronize Azure Active Directory Users to Zscaler (Synchroniser les utilisateurs Azure Active Directory avec Zscaler) est sélectionné et activé.](./media/zscaler-beta-provisioning-tutorial/user-mappings.png)

11. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et Zscaler Beta. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondre des comptes d’utilisateur dans Zscaler Beta dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Dans la section Attribute Mapping (Mappages d’attributs) pour les attributs utilisateur, les attributs Active Directory figurent en regard des attributs Zscalar Beta avec lesquels ils sont synchronisés. Une paire d’attributs est indiquée comme étant correspondante (Matching).](./media/zscaler-beta-provisioning-tutorial/user-attribute-mappings.png)

12. Dans la section **Mappages**, sélectionnez **Synchroniser les groupes Azure Active Directory avec Zscaler Beta**.

    ![Synchronize Azure Active Directory Groups to Zscaler (Synchroniser les groupes Azure Active Directory avec Zscaler) est sélectionné et activé.](./media/zscaler-beta-provisioning-tutorial/group-mappings.png)

13. Dans la section **Mappages des attributs**, passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Zscaler Beta. Les attributs sélectionnés comme propriétés de **Correspondance** sont utilisés pour la mise en correspondre des groupes dans Zscaler Beta dans le cadre des opérations de mise à jour. Cliquez sur le bouton **Enregistrer** pour valider les modifications.

    ![Dans la section Attribute Mapping (Mappages d’attributs) pour les attributs de groupe, les attributs Active Directory figurent à côté des attributs Zscalar Beta avec lesquels ils sont synchronisés. Une paire d’attributs est indiquée comme étant correspondante (Matching).](./media/zscaler-beta-provisioning-tutorial/group-attribute-mappings.png)

14. Pour configurer des filtres d’étendue, reportez-vous aux instructions suivantes fournies dans [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Pour activer le service de provisionnement Azure AD pour Zscaler Beta, définissez le paramètre **État du provisionnement** sur **Activé** dans la section **Paramètres**.

    ![Provisioning Status (État du provisionnement) est présenté et défini sur On (Activé).](./media/zscaler-beta-provisioning-tutorial/provisioning-status.png)

16. Définissez les utilisateurs et/ou groupes que vous souhaitez provisionner sur Zscaler Beta en choisissant les valeurs souhaitées dans **Étendue** dans la section **Paramètres**.

    ![La liste déroulante Scope (Étendue) est présentée, et Sync only assigned users and groups (Synchroniser uniquement les utilisateurs et groupes assignés) est sélectionné. L’autre valeur disponible est Sync all users and groups (Synchroniser tous les utilisateurs et groupes).](./media/zscaler-beta-provisioning-tutorial/scoping.png)

17. Lorsque vous êtes prêt à effectuer l’approvisionnement, cliquez sur **Enregistrer**.

    ![Le bouton Save (Enregistrer) en haut de Zscaler Beta - Provisioning est mis en évidence. Il y a également un bouton Discard (Ignorer).](./media/zscaler-beta-provisioning-tutorial/save-provisioning.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs et/ou groupes définis dans **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de synchronisation** pour surveiller la progression et les liens vers les rapports d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement Azure AD sur Zscaler Beta.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zscaler-beta-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-beta-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-beta-provisioning-tutorial/tutorial-general-03.png