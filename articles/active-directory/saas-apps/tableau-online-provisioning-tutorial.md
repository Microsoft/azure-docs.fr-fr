---
title: 'Didacticiel : configurer Tableau Online pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur sur Tableau Online.
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
ms.openlocfilehash: a42790e079985b003776b381c74f837b0ba619b1
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359202"
---
# <a name="tutorial-configure-tableau-online-for-automatic-user-provisioning"></a>Didacticiel : configurer Tableau Online pour l’approvisionnement automatique d’utilisateurs

Ce tutoriel présente les étapes à effectuer dans Tableau Online et Azure Active Directory (Azure AD) afin de configurer Azure AD pour provisionner et déprovisionner automatiquement des utilisateurs et des groupes sur Tableau Online.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’approvisionnement d’utilisateurs Azure AD. Pour plus d’informations sur l’objet et le fonctionnement de ce service et pour accéder au forum aux questions, consultez [Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Conditions préalables requises

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

*   Un locataire Azure AD.
*   Un [locataire Tableau Online](https://www.tableau.com/)
*   Un compte d’utilisateur dans Tableau Online doté d’autorisations d’administrateur

> [!NOTE]
> L’intégration du provisionnement Azure AD repose sur [l’API REST Tableau Online](https://onlinehelp.tableau.com/current/api/rest_api/en-us/help.htm). Cette API est disponible pour les développeurs Tableau Online.

## <a name="add-tableau-online-from-the-azure-marketplace"></a>Ajouter Tableau Online à partir de la Place de marché Azure
Avant de configurer Tableau Online pour l’approvisionnement automatique d’utilisateurs avec Azure AD, ajoutez Tableau Online à partir de la Place de marché Azure à votre liste d’applications SaaS managées.

Pour ajouter Tableau Online à partir de la Place de marché Azure, procédez comme suit.

1. Dans le volet de navigation de gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Icône Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Tableau Online** , puis sélectionnez **Tableau Online** dans le volet de résultats. Pour ajouter l’application, sélectionnez **Ajouter**.

    ![Tableau Online dans la liste des résultats](common/search-new-app.png)

## <a name="assign-users-to-tableau-online"></a>Affectation d’utilisateurs à Tableau Online

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs ou groupes affectés à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, identifiez les utilisateurs ou groupes dans Azure AD qui doivent accéder à Tableau Online. Pour affecter ces utilisateurs ou groupes à Tableau Online, suivez les instructions dans [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-tableau-online"></a>Conseils importants concernant l’affectation d’utilisateurs à Tableau Online

*   Nous vous recommandons d’affecter un seul utilisateur Azure AD à Tableau Online afin de tester la configuration du provisionnement automatique d’utilisateurs. Vous pouvez affecter des groupes ou des utilisateurs supplémentaires ultérieurement.

*   Lorsque vous affectez un utilisateur à Tableau Online, sélectionnez un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configure-automatic-user-provisioning-to-tableau-online"></a>Configurer l’approvisionnement automatique d’utilisateurs dans Tableau Online

Cette section vous guide dans la procédure de configuration du service d’approvisionnement Azure AD. Utilisez-le pour créer, mettre à jour et désactiver des utilisateurs ou des groupes dans Tableau Online en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également activer l’authentification unique SAML pour Tableau Online. Suivez les instructions fournies dans le [tutoriel sur l’authentification unique pour Tableau Online](tableauonline-tutorial.md). La configuration de l’authentification unique est indépendante de celle de l’approvisionnement automatique d’utilisateurs, même si ces deux fonctionnalités se complètent.

### <a name="configure-automatic-user-provisioning-for-tableau-online-in-azure-ad"></a>Configurer l’approvisionnement automatique d’utilisateurs pour Tableau Online dans Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Tableau Online**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Tableau Online**.

    ![Lien Tableau Online dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Approvisionnement de Tableau Online](./media/tableau-online-provisioning-tutorial/ProvisioningTab.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Tableau Online : mode d’approvisionnement](./media/tableau-online-provisioning-tutorial/ProvisioningCredentials.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez le domaine, le nom d’utilisateur de l’administrateur, le mot de passe d’administrateur et l’URL du contenu de votre compte Tableau Online :

   * Dans la zone **Domaine** , indiquez le sous-domaine en fonction de l’étape 6.

   * Dans la zone **Nom d’utilisateur de l’administrateur** , entrez le nom d’utilisateur du compte administrateur sur votre locataire Tableau Online. par exemple admin@contoso.com.

   * Dans la zone **Mot de passe d’administrateur** , entrez le mot de passe du compte administrateur correspondant au nom d’utilisateur de l’administrateur.

   * Dans la zone **URL du contenu** , indiquez le sous-domaine en fonction de l’étape 6.

6. Lorsque vous êtes connecté à votre compte d’administration de Tableau Online, vous pouvez obtenir les valeurs **Domaine** et **URL du contenu** à partir de l’URL de la page d’administration.

    * Le **domaine** de votre compte Tableau Online peut être copié à partir de cette partie de l’URL :

        ![Tableau Online : domaine](./media/tableau-online-provisioning-tutorial/DomainUrlPart.png)

    * **L’URL du contenu** de votre compte Tableau Online peut être copiée à partir de cette section. Cette valeur est définie lors de la configuration du compte. Dans cet exemple, la valeur est « contoso » :

        ![Tableau Online : URL du contenu](./media/tableau-online-provisioning-tutorial/ContentUrlPart.png)

        > [!NOTE]
        > Votre **domaine** peut être différente de celui illustré ici.

7. Une fois que vous avez renseigné les zones présentées à l’étape 5, sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Tableau Online. Si la connexion échoue, vérifiez que votre compte Tableau Online dispose d’autorisations d’administrateur et réessayez.

    ![Tableau Online : test de connexion](./media/tableau-online-provisioning-tutorial/TestConnection.png)

8. Dans la zone **E-mail de notification** , entrez l’adresse e-mail de la personne ou du groupe devant recevoir les notifications d’erreur d’approvisionnement. Cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![Tableau Online : e-mail de notification](./media/tableau-online-provisioning-tutorial/EmailNotification.png)

9. Sélectionnez **Enregistrer**.

10. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur Tableau Online**.

    ![Tableau Online : synchronisation des utilisateurs](./media/tableau-online-provisioning-tutorial/UserMappings.png)

11. Dans la section **Mappages des attributs** , passez en revue les attributs d’utilisateurs qui sont synchronisés entre Azure AD et Tableau Online. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes utilisateur dans Tableau Online pour les opérations de mise à jour. Pour enregistrer les modifications, sélectionnez **Enregistrer**.

    ![Tableau Online : mise en correspondance des attributs utilisateur](./media/tableau-online-provisioning-tutorial/attribute.png)

12. Dans la section **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Tableau Online**.

    ![Tableau Online : synchronisation des groupes](./media/tableau-online-provisioning-tutorial/GroupMappings.png)

13. Dans la section **Mappages des attributs** , passez en revue les attributs de groupes qui sont synchronisés entre Azure AD et Tableau Online. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes utilisateur dans Tableau Online pour les opérations de mise à jour. Pour enregistrer les modifications, sélectionnez **Enregistrer**.

    ![Tableau Online : mise en correspondance des attributs de groupes](./media/tableau-online-provisioning-tutorial/GroupAttributeMapping.png)

14. Pour configurer des filtres d’étendue, suivez les instructions fournies dans le [didacticiel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Pour activer le service d’approvisionnement Azure AD pour Tableau Online, définissez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**.

    ![Tableau Online : état de l’approvisionnement](./media/tableau-online-provisioning-tutorial/ProvisioningStatus.png)

16. Définissez les utilisateurs ou groupes que vous voulez approvisionner dans Tableau Online. Dans la section **Paramètres** , sélectionnez les valeurs souhaitées sous **Étendue**.

    ![Tableau Online : étendue](./media/tableau-online-provisioning-tutorial/ScopeSync.png)

17. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Tableau Online : enregistrement](./media/tableau-online-provisioning-tutorial/SaveProvisioning.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs ou groupes définis sous **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Elles se produisent toutes les 40 minutes environ, tant que le service d’approvisionnement Azure AD s’exécute. 

Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers le rapport d’activité d’approvisionnement. Ce rapport décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur Tableau Online.

Pour avoir des informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="change-log"></a>Journal des modifications
* 30/09/2020 : Ajout de la prise en charge de l’attribut « authSetting » pour les utilisateurs.

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/tableau-online-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/tableau-online-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/tableau-online-provisioning-tutorial/tutorial_general_03.png