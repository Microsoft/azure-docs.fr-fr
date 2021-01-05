---
title: 'Tutoriel : Approvisionnement d’utilisateurs pour Cerner Central – Azure AD'
description: Découvrez comment configurer Azure Active Directory pour approvisionner automatiquement des utilisateurs dans une liste de Cerner Central.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: arvinh
ms.openlocfilehash: 1f82cab1172e7293e2a5910d35280eefb30ed49e
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357451"
---
# <a name="tutorial-configure-cerner-central-for-automatic-user-provisioning"></a>Tutoriel : Configurer Cerner Central pour l'approvisionnement automatique d'utilisateurs

Ce didacticiel vous montre les étapes à effectuer dans Cerner Central et Azure AD pour approvisionner et retirer automatiquement des comptes d’utilisateur d’Azure AD vers une liste d’utilisateurs dans Cerner Central.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un client Azure Active Directory
* Un locataire Cerner Central

> [!NOTE]
> Azure Active Directory s’intègre dans Cerner Central à l’aide du protocole [SCIM](http://www.simplecloud.info/).

## <a name="assigning-users-to-cerner-central"></a>Assignation d’utilisateurs à Cerner Central

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique de comptes utilisateur, seuls les utilisateurs et les groupes qui ont été « assignés » à une application dans Azure AD sont synchronisés. 

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD représentent les utilisateurs qui ont besoin d’accéder à votre application Cerner Central. Une fois que vous avez choisi, vous pouvez assigner ces utilisateurs à votre application Cerner Central en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-cerner-central"></a>Conseils importants pour l’assignation d’utilisateurs à Cerner Central

* Il est recommandé qu’un seul utilisateur Azure AD soit assigné à Cerner Central pour tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Une fois le test initial terminé pour un seul utilisateur, Cerner Central recommande d’assigner la liste complète des utilisateurs censés pouvoir accéder à toutes les solutions Cerner (et pas seulement Cerner Central) pour un approvisionnement vers la liste d’utilisateurs Cerner.  Les autres solutions Cerner tirent parti de cette liste d’utilisateurs.

* Quand vous assignez un utilisateur à Cerner Central, vous devez sélectionner le rôle **Utilisateur** dans la boîte de dialogue d’assignation. Les utilisateurs dont le rôle est « Accès par défaut » sont exclus de l’approvisionnement.

## <a name="configuring-user-provisioning-to-cerner-central"></a>Configuration de l’approvisionnement des utilisateurs sur Cerner Central

Cette section vous explique comment connecter votre instance d’Azure AD à la liste d’utilisateurs Cerner Central par le biais de l’API d’approvisionnement de comptes d’utilisateur SCIM Cerner, et comment configurer le service d’approvisionnement afin de créer, mettre à jour et désactiver des comptes d’utilisateur assignés dans Cerner Central en fonction des assignations d’utilisateurs et de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Cerner Central en suivant les instructions fournies dans le [Portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que ces deux fonctionnalités se complètent. Pour plus d’informations, consultez le [didacticiel dédié à l’authentification unique dans Cerner Central](cernercentral-tutorial.md).

### <a name="to-configure-automatic-user-account-provisioning-to-cerner-central-in-azure-ad"></a>Pour configurer l’approvisionnement automatique de comptes d’utilisateur sur Cerner Central dans Azure AD :

Pour approvisionner des comptes d’utilisateur sur Cerner Central, vous devez demander la création d’un compte système Cerner Central auprès de Cerner et générer un jeton du porteur OAuth qu’Azure AD utilisera pour se connecter au point de terminaison SCIM de Cerner. Il est également recommandé d’exécuter l’intégration dans un environnement de bac à sable Cerner avant le déploiement en production.

1. La première étape consiste à s’assurer que les personnes gérant l’intégration Cerner et Azure AD disposent d’un compte CernerCare, qui est requis afin d’accéder à la documentation nécessaire pour suivre les instructions. Si besoin est, utilisez les URL ci-dessous pour créer des comptes CernerCare dans chaque environnement applicable.

   * Bac à sable : https://sandboxcernercare.com/accounts/create

   * Production : https://cernercare.com/accounts/create  

2. Vous devez ensuite créer un compte système pour Azure AD. Utilisez les instructions ci-dessous afin de demander un compte système pour vos environnements de bac à sable et de production.

   * Instructions : https://wiki.ucern.com/display/CernerCentral/Requesting+A+System+Account

   * Bac à sable : https://sandboxcernercentral.com/system-accounts/

   * Production : https://cernercentral.com/system-accounts/

3. Générez ensuite un jeton du porteur OAuth pour chacun de vos comptes système. Pour ce faire, procédez comme suit.

   * Instructions : https://wiki.ucern.com/display/public/reference/Accessing+Cerner%27s+Web+Services+Using+A+System+Account+Bearer+Token

   * Bac à sable : https://sandboxcernercentral.com/system-accounts/

   * Production : https://cernercentral.com/system-accounts/

4. Enfin, vous devez acquérir des ID de domaine de liste d’utilisateurs pour le bac à sable et les environnements de production dans Cerner afin de terminer la configuration. Pour plus d’informations sur la procédure d’acquisition, consultez la page : https://wiki.ucern.com/display/public/reference/Publishing+Identity+Data+Using+SCIM. 

5. Vous pouvez maintenant configurer Azure AD afin d’approvisionner des comptes d’utilisateur sur Cerner. Connectez-vous au [portail Azure](https://portal.azure.com), puis accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

6. Si vous avez déjà configuré Cerner Central pour l’authentification unique, recherchez votre instance de Cerner Central à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **Cerner Central** dans la galerie d’applications. Sélectionnez Cerner Central dans les résultats de recherche et ajoutez-le à votre liste d’applications.

7. Sélectionnez votre instance de Cerner Central, puis sélectionnez l’onglet **Approvisionnement**.

8. Définissez le **Mode d’approvisionnement** sur **Automatique**.

   ![Approvisionnement Central Cerner](./media/cernercentral-provisioning-tutorial/Cerner.PNG)

9. Renseignez les champs suivants sous **Informations d’identification de l’administrateur**  :

   * Dans le champ **URL de locataire** , entrez une URL au format ci-dessous, en remplaçant « User-Roster-Realm-ID » par l’ID de domaine que vous avez obtenu à l’étape 4.

    > Bac à sable : https://user-roster-api.sandboxcernercentral.com/scim/v1/Realms/User-Roster-Realm-ID/ 
    > 
    > Production : https://user-roster-api.cernercentral.com/scim/v1/Realms/User-Roster-Realm-ID/ 

   * Dans le champ **Jeton secret** , entrez le jeton du porteur OAuth que vous avez généré à l’étape 3, puis cliquez sur **Tester la connexion**.

   * Une notification de réussite doit s’afficher en haut à droite de votre portail.

1. Entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement dans le champ **E-mail de notification** , puis cochez la case se trouvant en dessous.

1. Cliquez sur **Enregistrer**.

1. Dans la section **Mappages d’attributs** , passez en revue les attributs d’utilisateur et de groupe qui seront synchronisés entre Azure AD et Cerner Central. Les attributs sélectionnés en tant que propriétés **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur et les groupes dans Cerner Central pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

1. Afin d’activer le service d’approvisionnement Azure AD pour Cerner Central, modifiez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**

1. Cliquez sur **Enregistrer**.

Cette commande lance la synchronisation initiale des utilisateurs et/ou groupes assignés à Cerner Central dans la section Utilisateurs et Groupes. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service de provisionnement Azure AD est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement dans votre application Cerner Central.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Cerner Central : Publication de données d'identité à l'aide d'Azure AD](https://wiki.ucern.com/display/public/reference/Publishing+Identity+Data+Using+Azure+AD)
* [Tutoriel : Configuration de Cerner Central pour l'authentification unique avec Azure Active Directory](cernercentral-tutorial.md)
* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md).