---
title: 'Tutoriel : Configurer Velpic pour l’attribution automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes utilisateur sur Velpic.
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
ms.openlocfilehash: cdd4fb96a42d154ccd8b508950283978ddf58ef4
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94354901"
---
# <a name="tutorial-configuring-velpic-for-automatic-user-provisioning"></a>Tutoriel : Configuration de Velpic pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de vous montrer les étapes à effectuer dans Velpic et Azure AD pour approvisionner et retirer automatiquement des comptes utilisateur d’Azure AD vers Velpic.

> [!NOTE]
> Ce didacticiel décrit un connecteur reposant sur le service d’attribution d’utilisateurs Azure AD. Pour découvrir les informations importantes sur ce que fait ce service, comment il fonctionne et consulter le forum aux questions, reportez-vous à l’article [Automatiser l’attribution et l’annulation de l’attribution des utilisateurs dans les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* Un client Azure Active Directory
* Un locataire Velpic avec le [plan Entreprise](https://www.velpic.com/pricing.html) ou mieux activé
* Un compte d’utilisateur dans Velpic avec des autorisations d’administration

## <a name="assigning-users-to-velpic"></a>Affectation d’utilisateurs à Velpic

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique de comptes d’utilisateur, les utilisateurs et les groupes qui ont été « affectés » à une application dans Azure AD sont synchronisés. 

Avant de configurer et activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD représentent les utilisateurs qui ont besoin d’accéder à votre application Velpic. Une fois que vous avez choisi, vous pouvez affecter ces utilisateurs à votre application Velpic en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-velpic"></a>Conseils importants pour l’affectation d’utilisateurs à Velpic

* Il est recommandé qu’un seul utilisateur Azure AD soit affecté à Velpic pour tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Quand vous attribuez un utilisateur à Velpic, vous devez sélectionner le rôle **Utilisateur** ou un autre rôle valide propre à l’application (si disponible) dans la boîte de dialogue d’attribution. Notez que le rôle **Accès par défaut** ne fonctionne pas pour l’approvisionnement et ces utilisateurs seront ignorés.

## <a name="configuring-user-provisioning-to-velpic"></a>Configuration de l’approvisionnement des utilisateurs sur Velpic

Cette section vous guide à travers la connexion de votre instance d’Azure AD au compte d’utilisateur Velpic fournissant l’API et la configuration du service de provisionnement pour créer, mettre à jour et désactiver les comptes utilisateur affectés dans Velpic en fonction des attributions d’utilisateurs et de groupes dans Azure AD.

> [!TIP]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Velpic en suivant les instructions disponibles dans le [Portail Microsoft Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="to-configure-automatic-user-account-provisioning-to-velpic-in-azure-ad"></a>Configurer l’attribution automatique des comptes d’utilisateurs sur Velpic dans Azure AD :

1. Dans le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

2. Si vous avez déjà configuré Velpic pour l’authentification unique, recherchez votre instance Velpic à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **Velpic** dans la galerie d’applications. Sélectionnez Velpic dans les résultats de la recherche, puis ajoutez-le à votre liste d’applications.

3. Sélectionnez votre instance Velpic, puis sélectionnez l’onglet **Approvisionnement**.

4. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Configuration de Velpic](./media/velpic-provisioning-tutorial/Velpic1.png)

5. Sous la section **Admin Credentials (Informations d’identification de l’administrateur)** , entrez l’ **URL de locataire et le jeton secret** de Velpic. Vous pouvez trouver ces valeurs sous votre compte Velpic : **Manage (Gérer)**  > **Intégration** > **Plugin (Complément)**  > **SCIM**

    ![Valeurs d’autorisation](./media/velpic-provisioning-tutorial/Velpic2.png)

6. Dans le portail Microsoft Azure, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à votre application Velpic. Si la connexion échoue, vérifiez que votre compte Velpic dispose des autorisations d’administrateur et réessayez l’étape 5.

7. Entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement dans le champ **E-mail de notification** , puis cochez la case se trouvant en dessous.

8. Cliquez sur **Enregistrer**.

9. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur Velpic**.

10. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur qui seront synchronisés d’Azure AD vers Velpic. Notez que les attributs sélectionnés en tant que propriétés de **Correspondance** seront utilisés pour faire correspondre les comptes utilisateur dans Velpic pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

11. Pour activer le service d’approvisionnement Azure AD pour Velpic, définissez le paramètre **État d’approvisionnement** sur **Activé** dans la section **Paramètres**

12. Cliquez sur **Enregistrer**.

Cette commande démarre la synchronisation initiale des utilisateurs et/ou groupes affectés à Velpic dans la section Utilisateurs et Groupes. Notez que la synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les rapports d’activité d’approvisionnement, qui décrivent toutes les actions effectuées par le service d’approvisionnement.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Étapes suivantes

* [Découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md)