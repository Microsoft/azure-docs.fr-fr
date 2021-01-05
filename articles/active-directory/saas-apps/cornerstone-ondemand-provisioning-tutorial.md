---
title: 'Didacticiel : Configurer Cornerstone OnDemand pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour provisionner et déprovisionner automatiquement des comptes d’utilisateur sur Cornerstone OnDemand.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 59c599167089d222324ed880c18e68d763f5e468
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358454"
---
# <a name="tutorial-configure-cornerstone-ondemand-for-automatic-user-provisioning"></a>Didacticiel : Configurer Cornerstone OnDemand pour l’approvisionnement automatique d’utilisateurs

Ce tutoriel présente les étapes à effectuer dans Cornerstone OnDemand et Azure Active Directory (Azure AD) afin de configurer Azure AD pour provisionner et déprovisionner automatiquement des utilisateurs ou des groupes sur Cornerstone OnDemand.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’approvisionnement d’utilisateurs Azure AD. Pour plus d’informations sur l’objet et le fonctionnement de ce service et pour accéder au forum aux questions, consultez [Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Conditions préalables requises

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un locataire Azure AD.
* Un locataire Cornerstone OnDemand.
* Un compte d’utilisateur dans Cornerstone OnDemand avec des autorisations d’administrateur.

> [!NOTE]
> L’intégration du provisionnement Azure AD repose sur le [service web Cornerstone OnDemand](https://www.cornerstoneondemand.com/). Ce service est disponible pour les équipes Cornerstone OnDemand.

## <a name="add-cornerstone-ondemand-from-the-azure-marketplace"></a>Ajouter Cornerstone OnDemand à partir de la Place de marché Azure

Avant de configurer Cornerstone OnDemand pour le provisionnement automatique d’utilisateurs avec Azure AD, ajoutez Cornerstone OnDemand à partir de la Place de marché à votre liste d’applications SaaS managées.

Pour ajouter Cornerstone OnDemand à partir de la Place de marché, procédez comme suit.

1. Dans le volet de navigation de gauche du [portail Azure](https://portal.azure.com), sélectionnez **Azure Active Directory**.

    ![Icône Azure Active Directory](common/select-azuread.png)

2. Accédez à **Applications d’entreprise** , puis sélectionnez **Toutes les applications**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

3. Pour ajouter une application, sélectionnez **Nouvelle application** en haut de la boîte de dialogue.

    ![Bouton Nouvelle application](common/add-new-app.png)

4. Dans la zone de recherche, entrez **Cornerstone OnDemand** et sélectionnez **Cornerstone OnDemand** dans le panneau de résultats. Pour ajouter l’application, sélectionnez **Ajouter**.

    ![Cornerstone OnDemand dans la liste des résultats](common/search-new-app.png)

## <a name="assign-users-to-cornerstone-ondemand"></a>Affecter des utilisateurs à Cornerstone OnDemand

Azure Active Directory utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique d’utilisateurs, seuls les utilisateurs ou groupes affectés à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le provisionnement automatique d’utilisateurs, identifiez les utilisateurs ou groupes dans Azure AD qui doivent accéder à Cornerstone OnDemand. Pour affecter ces utilisateurs ou groupes à Cornerstone OnDemand, suivez les instructions dans [Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-cornerstone-ondemand"></a>Conseils importants pour l’assignation d’utilisateurs à Cornerstone OnDemand

* Nous vous recommandons d’affecter un seul utilisateur Azure AD à Cornerstone OnDemand afin de tester la configuration du provisionnement automatique d’utilisateurs. Vous pouvez affecter des groupes ou des utilisateurs supplémentaires ultérieurement.

* Quand vous affectez un utilisateur à Cornerstone OnDemand, sélectionnez un rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Les utilisateurs dont le rôle est **Accès par défaut** sont exclus de l’approvisionnement.

## <a name="configure-automatic-user-provisioning-to-cornerstone-ondemand"></a>Configurer le provisionnement automatique d’utilisateurs sur Cornerstone OnDemand

Cette section vous guide dans la procédure de configuration du service d’approvisionnement Azure AD. Utilisez-le pour créer, mettre à jour et désactiver des utilisateurs ou des groupes dans Cornerstone OnDemand en fonction des affectations d’utilisateurs ou de groupes dans Azure AD.

Pour configurer le provisionnement automatique d’utilisateurs pour Cornerstone OnDemand dans Azure AD, procédez comme suit.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Applications d’entreprise** > **Toutes les applications** > **Cornerstone OnDemand**.

    ![Panneau Applications d’entreprise](common/enterprise-applications.png)

2. Dans la liste des applications, sélectionnez **Cornerstone OnDemand**.

    ![Lien Cornerstone OnDemand dans la liste des applications](common/all-applications.png)

3. Sélectionnez l’onglet **Approvisionnement**.

    ![Approvisionnement Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningTab.png)

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Mode de provisionnement Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningCredentials.png)

5. Sous la section **Informations d’identification de l’administrateur** , entrez le nom d’utilisateur de l’administrateur, le mot de passe d’administrateur et le domaine de votre compte Cornerstone OnDemand :

    * Dans la zone **Nom d’utilisateur administrateur** , indiquez le domaine ou le nom d’utilisateur du compte Administrateur sur votre locataire Cornerstone OnDemand. Exemple : contoso\admin.

    * Dans la zone **Mot de passe d’administrateur** , indiquez le mot de passe correspondant au nom d’utilisateur de l’administrateur.

    * Dans la zone **Domaine** , indiquez l’URL du service web du locataire Cornerstone OnDemand. Par exemple, le service se trouve à l’adresse `https://ws-[corpname].csod.com/feed30/clientdataservice.asmx`. Pour Contoso, le domaine est `https://ws-contoso.csod.com/feed30/clientdataservice.asmx`. Pour plus d’informations sur la façon de récupérer l’URL du service web, consultez [ce fichier pdf](https://help.csod.com/help/csod_0/Content/Resources/Documents/WebServices/CSOD_Web_Services_-_User-OU_Technical_Specification_v20160222.pdf).

6. Une fois que vous avez renseigné les zones affichées à l’étape 5, sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à Cornerstone OnDemand. Si la connexion échoue, vérifiez que votre compte Cornerstone OnDemand dispose des autorisations d’administrateur et réessayez.

    ![Tester la connexion de Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/TestConnection.png)

7. Dans la zone **E-mail de notification** , entrez l’adresse e-mail de la personne ou du groupe devant recevoir les notifications d’erreur d’approvisionnement. Cochez la case **Envoyer une notification par e-mail en cas de défaillance**.

    ![E-mail de notification Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/EmailNotification.png)

8. Sélectionnez **Enregistrer**.

9. Dans la section **Mappages** , sélectionnez **Synchronize Azure Active Directory Users to Cornerstone OnDemand** (Synchroniser des utilisateurs Azure Active Directory avec Cornerstone OnDemand).

    ![Synchronisation Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserMapping.png)

10. Passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers Cornerstone OnDemand dans la section **Mappages d’attributs**. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans Cornerstone OnDemand pour les opérations de mise à jour. Pour enregistrer les modifications, sélectionnez **Enregistrer**.

    ![Mappages d’attributs Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserMappingAttributes.png)

11. Pour configurer des filtres d’étendue, suivez les instructions fournies dans le [didacticiel sur les filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Pour activer le service de provisionnement Azure AD pour Cornerstone OnDemand, définissez le paramètre **État de provisionnement** sur **Activé** dans la section **Paramètres**.

    ![État du provisionnement Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningStatus.png)

13. Définissez les utilisateurs ou groupes que vous voulez provisionner sur Cornerstone OnDemand. Dans la section **Paramètres** , sélectionnez les valeurs souhaitées sous **Étendue**.

    ![Étendue Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/SyncScope.png)

14. Quand vous êtes prêt à effectuer l’approvisionnement, sélectionnez **Enregistrer**.

    ![Cornerstone OnDemand, Enregistrer](./media/cornerstone-ondemand-provisioning-tutorial/Save.png)

Cette opération démarre la synchronisation initiale de tous les utilisateurs ou groupes définis sous **Étendue** dans la section **Paramètres**. La synchronisation initiale prend plus de temps que les synchronisations ultérieures. Elles se produisent toutes les 40 minutes environ, tant que le service d’approvisionnement Azure AD s’exécute. 

Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers le rapport d’activité d’approvisionnement. Ce rapport décrit toutes les actions effectuées par le service de provisionnement Azure AD sur Cornerstone OnDemand.

Pour avoir des informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Limitations du connecteur

L’attribut **Position** Cornerstone OnDemand attend une valeur correspondant aux rôles du portail Cornerstone OnDemand. Pour obtenir la liste des valeurs **Position** valides, accédez à **Modifier un enregistrement utilisateur > Structure de l’organisation > Position** dans le portail Cornerstone OnDemand.

![Provisionnement Cornerstone OnDemand, Modifier un enregistrement utilisateur](./media/cornerstone-ondemand-provisioning-tutorial/UserEdit.png)

![Provisionnement Cornerstone OnDemand, Position](./media/cornerstone-ondemand-provisioning-tutorial/UserPosition.png)

![Provisionnement Cornerstone OnDemand, Liste des positions](./media/cornerstone-ondemand-provisioning-tutorial/PostionId.png)

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_03.png
