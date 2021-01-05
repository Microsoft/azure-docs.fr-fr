---
title: 'Tutoriel : Configurer Zendesk pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d’utilisateur sur Zendesk.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/06/2019
ms.author: jeedes
ms.openlocfilehash: 620dd8fd586352ebeaf097a8f870a606f8e06c01
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359712"
---
# <a name="tutorial-configure-zendesk-for-automatic-user-provisioning"></a>Tutoriel : Configurer Zendesk pour le provisionnement automatique d’utilisateurs

Ce didacticiel montre comment procéder dans Zendesk et Azure Active Directory (Azure AD) afin de configurer Azure AD pour approvisionner et déprovisionner automatiquement des utilisateurs et des groupes sur Zendesk.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’approvisionnement d’utilisateurs Azure AD. Pour plus d’informations sur l’objet et le fonctionnement de ce service et pour accéder au forum aux questions, consultez [Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un locataire Azure AD.
* Un locataire Zendesk avec le plan Professionnel ou mieux activé
* Un compte d’utilisateur dans Zendesk avec des autorisations d’administrateur.

## <a name="add-zendesk-from-the-azure-marketplace"></a>Ajouter Zendesk à partir de la Place de marché Azure

Avant de configurer Zendesk pour l’approvisionnement automatique d’utilisateurs avec Azure AD, ajoutez Zendesk à votre liste d’applications SaaS managées à partir de la Place de marché Azure.

Pour ajouter Zendesk à partir de la Place de marché, procédez comme suit.

1. Dans le volet de navigation de gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Icône Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Zendesk** , puis, dans le panneau des résultats, sélectionnez **Zendesk**. Pour ajouter l’application, sélectionnez **Ajouter**.

    ![Zendesk dans la liste des résultats](common/search-new-app.png)

## <a name="assign-users-to-zendesk"></a>Affecter des utilisateurs à Zendesk

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs ou groupes affectés à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer l’approvisionnement automatique d’utilisateurs, identifiez les utilisateurs ou groupes dans Azure AD qui doivent accéder à Zendesk. Pour affecter ces utilisateurs ou groupes à Zendesk, suivez les instructions données dans [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-zendesk"></a>Conseils importants pour l’affectation d’utilisateurs à Zendesk

* Les rôles Zendesk sont désormais indiqués de manière dynamique et automatique dans l’interface utilisateur du portail Azure. Avant d’assigner des rôles Zendesk aux utilisateurs, assurez-vous que la synchronisation initiale est effectuée par rapport à Zendesk afin de récupérer les rôles les plus récents dans votre locataire Zendesk.

* Nous vous recommandons d’affecter un seul utilisateur Azure AD à Zendesk afin de tester votre configuration initiale de l’approvisionnement automatique d’utilisateurs. Vous pourrez affecter des groupes ou des utilisateurs supplémentaires par la suite, une fois les tests concluants.

* Quand vous affectez un utilisateur à Zendesk, sélectionnez un rôle valide spécifique de l’application, si disponible, dans la boîte de dialogue des affectations. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configure-automatic-user-provisioning-to-zendesk"></a>Configurer l’approvisionnement automatique d’utilisateurs sur Zendesk 

Cette section vous guide dans la procédure de configuration du service d’approvisionnement Azure AD. Celui-ci permet de créer, mettre à jour et désactiver des utilisateurs ou des groupes dans Zendesk en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également activer l’authentification unique SAML pour Zendesk. Suivez les instructions fournies dans le [didacticiel sur la l’authentification unique pour Zendesk](zendesk-tutorial.md). La configuration de l’authentification unique est indépendante de celle de l’approvisionnement automatique d’utilisateurs, même si ces deux fonctionnalités se complètent.

### <a name="configure-automatic-user-provisioning-for-zendesk-in-azure-ad"></a>Configurer l’approvisionnement automatique d’utilisateurs pour Zendesk dans Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Zendesk**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Zendesk**.

    ![Lien Zendesk dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Provisionnement de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk16.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Mode d’approvisionnement de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk1.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez le nom d’utilisateur de l’administrateur, le jeton secret et le domaine de votre compte Zendesk. Voici des exemples de valeurs :

   * Dans la zone **Nom d’utilisateur de l’administrateur** , entrez le nom d’utilisateur du compte administrateur sur votre locataire Zendesk. par exemple admin@contoso.com.

   * Dans la zone **Jeton secret** , indiquez le jeton secret comme décrit à l’étape 6.

   * Dans la zone **Domaine** , indiquez le sous-domaine de votre locataire Zendesk. Par exemple, pour un compte avec l’URL de locataire `https://my-tenant.zendesk.com`, votre sous-domaine est **my-tenant**.

6. Le jeton secret pour votre compte Zendesk se trouve dans **Admin** > **API** > **Paramètres**. Assurez-vous que **Jeton d’accès** est défini sur **Activé**.

    ![Paramètres d’administration de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk4.png)

    ![Jeton secret Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk2.png)

7. Après avoir complété les zones présentées à l’étape 5, sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Zendesk. Si la connexion échoue, vérifiez que votre compte Zendesk dispose des autorisations d’administrateur et réessayez.

    ![Tester la connexion Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk19.png)

8. Dans la zone **E-mail de notification** , entrez l’adresse e-mail de la personne ou du groupe devant recevoir les notifications d’erreur d’approvisionnement. Cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk9.png)

9. Sélectionnez **Enregistrer**.

10. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec Zendesk**.

    ![Synchronisation des utilisateurs de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk10.png)

11. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui sont synchronisés entre Azure AD et ZenDesk. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Zendesk pour les opérations de mise à jour. Pour enregistrer les modifications, sélectionnez **Enregistrer**.

    ![Zendesk : mise en correspondance des attributs utilisateur](./media/zendesk-provisioning-tutorial/ZenDesk11.png)

12. Dans la section **Mappages** , sélectionnez **Synchroniser les groupes Azure Active Directory avec Zendesk**.

    ![Synchronisation des groupes Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk12.png)

13. Dans la section **Mappages des attributs** , passez en revue les attributs de groupe qui sont synchronisés entre Azure AD et Zendesk. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les groupes dans Zendesk pour les opérations de mise à jour. Pour enregistrer les modifications, sélectionnez **Enregistrer**.

    ![Zendesk : mise en correspondance des attributs de groupe](./media/zendesk-provisioning-tutorial/ZenDesk13.png)

14. Pour configurer des filtres d’étendue, suivez les instructions fournies dans le [didacticiel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Pour activer le service d’approvisionnement Azure AD pour Zendesk, définissez **État d’attribution** sur **Activé** dans la section **Paramètres**.

    ![État d’approvisionnement de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk14.png)

16. Définissez les utilisateurs ou groupes que vous souhaitez approvisionner dans Zendesk. Dans la section **Paramètres** , sélectionnez les valeurs souhaitées sous **Étendue**.

    ![Étendue de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk15.png)

17. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Enregistrement de Zendesk](./media/zendesk-provisioning-tutorial/ZenDesk18.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs ou groupes définis sous **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Elles se produisent toutes les 40 minutes environ, tant que le service d’approvisionnement Azure AD s’exécute. 

Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers le rapport d’activité d’approvisionnement. Ce rapport décrit toutes les actions effectuées par le service d’approvisionnement Azure AD sur Zendesk.

Pour avoir des informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

* Zendesk prend en charge l’utilisation de groupes pour les utilisateurs avec les rôles **Agent** uniquement. Pour plus d’informations, consultez la [documentation Zendesk](https://support.zendesk.com/hc/en-us/articles/203661966-Creating-managing-and-using-groups).

* Quand un rôle personnalisé est affecté à un utilisateur ou à un groupe, le service d’approvisionnement automatique d’utilisateurs Azure AD affecte également le rôle par défaut **Agent**. Seuls les agents peuvent recevoir un rôle personnalisé. Pour plus d’informations, consultez la [documentation de l’API Zendesk](https://developer.zendesk.com/rest_api/docs/support/users#json-format-for-agent-or-admin-requests). 

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zendesk-tutorial/tutorial_general_01.png
[2]: ./media/zendesk-tutorial/tutorial_general_02.png
[3]: ./media/zendesk-tutorial/tutorial_general_03.png
