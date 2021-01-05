---
title: 'Tutoriel : Approvisionnement d’utilisateurs pour Pingboard – Azure AD'
description: Découvrez comment configurer Azure Active Directory pour approvisionner et retirer automatiquement des comptes utilisateur sur Pingboard.
services: active-directory
author: ArvindHarinder1
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: arvinh
ms.openlocfilehash: ac36f5d6d1f57fd8453c54bcc8cf19dd964f47f6
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94357893"
---
# <a name="tutorial-configure-pingboard-for-automatic-user-provisioning"></a>Tutoriel : Configurer Pingboard pour le provisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de vous montrer les étapes à suivre pour activer et désactiver le provisionnement automatique de comptes d’utilisateur à partir d’Azure Active Directory (Azure AD) vers Pingboard.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

* un locataire Azure AD ;
* Un [compte professionnel](https://pingboard.com/pricing) de locataire Pingboard
* Un compte utilisateur dans Pingboard avec des autorisations d’administrateur

> [!NOTE]
> L’intégration du provisionnement Azure AD repose sur [l’API Pingboard](https://pingboard.docs.apiary.io/#), disponible sur votre compte.

## <a name="assign-users-to-pingboard"></a>Affectation d’utilisateurs à Pingboard

Azure AD utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre du provisionnement automatique des comptes utilisateur, seuls les utilisateurs affectés à une application dans Azure AD sont synchronisés. 

Avant de configurer et d’activer le service de provisionnement, vous devez déterminer quels utilisateurs dans Azure AD ont besoin d’accéder à votre application Pingboard. Vous pouvez ensuite affecter ces utilisateurs à votre application Pingboard en suivant les instructions fournies ici :

[Affecter un utilisateur à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-pingboard"></a>Conseils importants pour l’affectation d’utilisateurs à Pingboard

Nous vous recommandons d’affecter un seul utilisateur Azure AD à Pingboard pour tester la configuration du provisionnement. D’autres utilisateurs pourront être affectés ultérieurement.

## <a name="configure-user-provisioning-to-pingboard"></a>Configuration du provisionnement des utilisateurs sur Pingboard 

Cette section vous guide tout au long de la connexion d’Azure AD à l’API de provisionnement des comptes utilisateur de Pingboard. Vous configurez aussi le service de provisionnement pour créer, mettre à jour et désactiver les comptes d’utilisateur attribués dans Pingboard en fonction des attributions d’utilisateurs dans Azure AD.

> [!TIP]
> Pour activer l’authentification unique basée sur SAML pour Pingboard, suivez les instructions fournies sur le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’attribution automatique, bien que ces deux fonctionnalités se complètent.

### <a name="to-configure-automatic-user-account-provisioning-to-pingboard-in-azure-ad"></a>Pour configurer le provisionnement automatique de comptes utilisateur sur Pingboard dans Azure AD

1. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory** > **Applications d’entreprise** > **Toutes les applications**.

1. Si vous avez déjà configuré Pingboard pour l’authentification unique, recherchez votre instance Pingboard à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **Pingboard** dans la galerie d’applications. Sélectionnez **Pingboard** dans les résultats de recherche et ajoutez-le à votre liste d’applications.

1. Sélectionnez votre instance Pingboard, puis sélectionnez l’onglet **Provisionnement**.

1. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Approvisionnement de Pingboard](./media/pingboard-provisioning-tutorial/pingboardazureprovisioning.png)

1. Dans la section **Informations d’identification de l’administrateur** , utilisez les étapes suivantes :

    a. Dans **URL de locataire** , entrez `https://your_domain.pingboard.com/scim/v2` et remplacez « your_domain » par votre domaine réel.

    b. Connectez-vous à [Pingboard](https://pingboard.com/) à l’aide de votre compte d’administrateur.

    c. Sélectionnez **Modules complémentaires** > **Intégrations** > **Azure Active Directory**.

    d. Accédez à l’onglet **Configurer** et sélectionnez **Enable user provisioning from Azure** (Activer le provisionnement d’utilisateurs à partir d’Azure).

    e. Copiez le jeton dans **Jeton du porteur OAuth** et entrez-le dans **Jeton secret**.

1. Dans le portail Azure, sélectionnez **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à votre application Pingboard. Si la connexion échoue, vérifiez que votre compte Pingboard dispose des autorisations d’administrateur et recommencez l’étape **Tester la connexion**.

1. Entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur de provisionnement dans **E-mail de notification**. Cochez la case située dessous.

1. Sélectionnez **Enregistrer**.

1. Dans la section **Mappages** , sélectionnez **Synchronize Azure Active Directory Users to Pingboard** (Synchroniser les utilisateurs Azure Active Directory avec Pingboard).

1. Dans la section **Mappages des attributs** , passez en revue les attributs utilisateur à synchroniser entre Azure AD et Pingboard. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes utilisateur dans Pingboard pour les opérations de mise à jour. Sélectionnez **Enregistrer** pour valider les modifications. Pour plus d’informations, consultez [Personnalisation des mappages d’attributs de provisionnement d’utilisateurs](../app-provisioning/customize-application-attributes.md).

1. Pour activer le service de provisionnement Azure AD pour Pingboard, définissez le paramètre **État de provisionnement** sur **Activé** dans la section **Paramètres**.

1. Cliquez sur **Enregistrer** pour lancer la synchronisation initiale des utilisateurs affectés à Pingboard.

La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Utilisez la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement. Les journaux d’activité décrivent toutes les actions effectuées par le service de provisionnement sur votre application Pingboard.

Pour savoir plus en détail comment lire les journaux d’activité d’attribution Azure AD, consultez la section [Générer un état sur l’attribution automatique de comptes d’utilisateurs](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion du provisionnement de comptes d’utilisateur pour les applications d’entreprise](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](pingboard-tutorial.md)
