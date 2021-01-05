---
title: 'Tutoriel : Approvisionnement d’utilisateurs pour Asana – Azure AD'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes utilisateur sur Asana.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: arvinh
ms.reviewer: celested
ms.openlocfilehash: 4abc117ae0e983cf684f0e70a363758f9be196aa
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359424"
---
# <a name="tutorial-configure-asana-for-automatic-user-provisioning"></a>Tutoriel : Configurer Asana pour le provisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à suivre dans Asana et Azure Active Directory (Azure AD) pour attribuer et désattribuer automatiquement des comptes d’utilisateurs d’Azure AD vers Asana.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* un locataire Azure AD ;
* un locataire Asana avec un [plan Entreprise](https://www.asana.com/pricing) ou un plan supérieur ;
* un compte d’utilisateur dans Asana avec des autorisations d’administration.

> [!NOTE]
> L’intégration de l’attribution Azure AD repose sur [l’API Asana](https://asana.com/developers/api-reference/users), qui est accessible à Asana.

## <a name="assign-users-to-asana"></a>Affecter des utilisateurs à Asana

Azure AD utilise un concept appelé *affectations* pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre du provisionnement automatique des comptes utilisateur, seuls les utilisateurs affectés à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service d’attribution, vous devez déterminer quels utilisateurs d’Azure AD ont besoin d’accéder à votre application Asana. Vous pouvez affecter ces utilisateurs à votre application Asana en suivant ces instructions :

[Affecter un utilisateur à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-asana"></a>Conseils importants pour l’affectation d’utilisateurs à Asana

Nous vous recommandons d’affecter un seul utilisateur Azure AD à Asana pour tester la configuration de l’attribution. D’autres utilisateurs pourront être affectés ultérieurement.

## <a name="configure-user-provisioning-to-asana"></a>Configurer l’attribution des utilisateurs sur Asana

Cette section explique pas à pas comment connecter Azure AD à l’API d’attribution des comptes d’utilisateurs Asana. Vous configurerez également le service d’attribution de façon à créer, à mettre à jour et à désactiver les comptes d’utilisateurs affectés dans Asana en fonction de l’attribution des utilisateurs dans Azure AD.

> [!TIP]
> Pour activer l’authentification unique SAML pour Asana, suivez les instructions du [Portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’attribution automatique, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-account-provisioning-to-asana-in-azure-ad"></a>Configurer l’attribution automatique des comptes d’utilisateurs sur Asana dans Azure AD

1. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory** > **Applications d’entreprise** > **Toutes les applications**.

1. Si vous avez déjà configuré l’authentification unique sur Asana, recherchez votre instance Asana à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **Asana** dans la galerie d’applications. Sélectionnez **Asana** dans les résultats de la recherche, puis ajoutez-le à votre liste d’applications.

1. Sélectionnez votre instance Asana, puis sélectionnez l’onglet **Attribution**.

1. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Approvisionnement Asana](./media/asana-provisioning-tutorial/asanaazureprovisioning.png)

1. Dans la section **Informations d’identification d’administration** , suivez ces instructions pour générer le jeton et l’entrer dans **Jeton secret** :

    a. Connectez-vous à [Asana](https://app.asana.com) avec votre compte Administrateur.

    b. Dans la barre supérieure, sélectionnez la photo de profil, puis les paramètres actuels de nom de votre organisation.

    c. Accédez à l’onglet **Comptes de service**.

    d. Sélectionnez **Ajouter un compte de service**.

    e. Mettez à jour **Nom** et **À propos de** ainsi que la photo de profil si besoin. Copiez le jeton dans **Jeton** , puis sélectionnez-le dans **Enregistrer les modifications**.

1. Sur le Portail Azure, sélectionnez **Tester la connexion** pour vérifier qu’Azure AD parvient à se connecter à votre application Asana. Si la connexion échoue, vérifiez que votre compte Asana dispose des autorisations d’administration, puis recommencez l’étape **Tester la connexion**.

1. Entrez l’adresse e-mail de la personne ou du groupe qui recevra les notifications d’erreur d’attribution dans **E-mail de notification**. Cochez la case située dessous.

1. Sélectionnez **Enregistrer**.

1. Dans la section **Mappages** , sélectionnez **Synchroniser les utilisateurs Azure Active Directory sur Asana**.

1. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateurs à synchroniser d’Azure AD à Asana. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateurs dans Asana pour les opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications. Pour plus d’informations, consultez la section [Personnaliser les mappages d’attributs de l’attribution des utilisateurs](../app-provisioning/customize-application-attributes.md).

1. Pour activer le service d’attribution Azure AD pour Asana, définissez **État d’attribution** sur **Activé** dans la section **Paramètres**.

1. Sélectionnez **Enregistrer**.

La synchronisation initiale commence pour les utilisateurs affectés à Asana dans la section **Utilisateurs**. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Utilisez la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement. Les journaux d’audit décrivent toutes les actions effectuées par le service de provisionnement dans votre application Asana.

Pour savoir plus en détail comment lire les journaux d’activité d’attribution Azure AD, consultez la section [Générer un état sur l’attribution automatique de comptes d’utilisateurs](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gérer l’attribution de comptes d’utilisateurs pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](asana-tutorial.md)
