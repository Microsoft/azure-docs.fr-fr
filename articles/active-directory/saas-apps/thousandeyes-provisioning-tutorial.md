---
title: 'Tutoriel : Approvisionnement d’utilisateurs pour ThousandEyes – Azure AD'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes d’utilisateur sur ThousandEyes.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/28/2019
ms.author: arvinh
ms.openlocfilehash: ff55528013ac89be48454c25e1fc86deac2bca6f
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357230"
---
# <a name="tutorial-configure-thousandeyes-for-automatic-user-provisioning"></a>Tutoriel : Configurer ThousandEyes pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de vous montrer les étapes à effectuer dans ThousandEyes et Azure AD pour approvisionner et retirer automatiquement des comptes d’utilisateur entre Azure AD et ThousandEyes. 

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un locataire Azure Active Directory
* Un locataire ThousandEyes pour lequel au moins le [plan standard](https://www.thousandeyes.com/pricing) est activé 
* Un compte d’utilisateur dans ThousandEyes doté d’autorisations d’administrateur 

> [!NOTE]
> L’intégration de l’approvisionnement d’Azure AD repose sur l’[API ThousandEyes SCIM](https://success.thousandeyes.com/PublicArticlePage?articleIdParam=kA044000000CnWrCAK), mise à la disposition des équipes de ThousandEyes qui relèvent au moins du plan standard.

## <a name="assigning-users-to-thousandeyes"></a>Affectation d’utilisateurs à ThousandEyes

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique des comptes d’utilisateur, seuls les utilisateurs et les groupes qui ont été « affectés » à une application dans Azure AD sont synchronisés. 

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD représentent les utilisateurs qui ont besoin d’accéder à votre application ThousandEyes. Dès que vous avez fait votre choix, vous pouvez assigner ces utilisateurs à votre application ThousandEyes en suivant les instructions disponibles à la page suivante :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-thousandeyes"></a>Conseils importants concernant l’affectation d’utilisateurs à ThousandEyes

* Il est recommandé de n’affecter qu’un seul utilisateur Azure AD à ThousandEyes afin de tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous affectez un utilisateur à ThousandEyes, vous devez sélectionner le rôle **utilisateur** ou un autre rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’affectation. Le rôle **Accès par défaut** ne fonctionne pas pour l’approvisionnement et ces utilisateurs sont ignorés.

## <a name="configuring-user-provisioning-to-thousandeyes"></a>Configuration de l'approvisionnement d’utilisateurs pour ThousandEyes 

Cette section va vous guider afin de connecter votre instance Azure AD à l’API d’approvisionnement de comptes d’utilisateur de ThousandEyes, puis configurer le service d’approvisionnement pour créer, mettre à jour et désactiver des comptes d’utilisateur affectés dans ThousandEyes, en fonction des affectations d’utilisateurs et de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour ThousandEyes, grâce aux instructions disponibles dans le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="configure-automatic-user-account-provisioning-to-thousandeyes-in-azure-ad"></a>Configurer l’approvisionnement automatique de comptes d’utilisateur pour ThousandEyes dans Azure AD

1. Dans le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

2. Si vous avez déjà configuré ThousandEyes pour l’authentification unique, recherchez votre instance de ThousandEyes à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **ThousandEyes** dans la galerie d’applications. Dans les résultats de la recherche, sélectionnez ThousandEyes, puis ajoutez-le à votre liste d’applications.

3. Sélectionnez votre instance de ThousandEyes, puis sélectionnez l’onglet **Approvisionnement**.

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran montrant l’onglet Approvisionnement pour ThousandEyes avec Automatique sélectionné pour le mode d’approvisionnement.](./media/thousandeyes-provisioning-tutorial/ThousandEyes1.png)

5. Dans la section **Informations d’identification de l’administrateur** , entrez le **jeton du porteur OAuth** généré par votre compte ThousandEyes (vous pouvez le trouver ou le générer sous la section **Profil** de votre compte ThousandEyes).

    ![Capture d’écran montrant où trouver le lien des paramètres de compte pour le groupe de comptes actuel.](./media/thousandeyes-provisioning-tutorial/ThousandEyes2.png)

6. Dans le portail Azure, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à votre application ThousandEyes. Si la connexion échoue, vérifiez que votre compte ThousandEyes dispose des autorisations d’administrateur et réessayez l’étape 5.

7. Entrez l’adresse de courrier d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement dans le champ **E-mail de notification** , puis cochez la case « Envoyer une notification par e-mail en cas de défaillance ».

8. Cliquez sur **Enregistrer**.

9. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec ThousandEyes**.

10. Dans la section **Mappages d’attributs** , passez en revue les attributs utilisateur qui sont synchronisés à partir d’Azure AD vers ThousandEyes. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans ThousandEyes pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

11. Pour activer le service d’approvisionnement Azure AD pour ThousandEyes, accédez à la section **Paramètres** et définissez le **Statut de l’approvisionnement** sur **Activé**.

12. Cliquez sur **Enregistrer**.

Cette opération démarre la synchronisation initiale des utilisateurs et/ou des groupes affectés à ThousandEyes dans la section Utilisateurs et groupes. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)
