---
title: 'Tutoriel : Configurer NetSuite OneWorld pour le provisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et NetSuite OneWorld.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: a5b995fd273fcfa231e101bc77b11d268be728fb
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359133"
---
# <a name="tutorial-configuring-netsuite-for-automatic-user-provisioning"></a>Tutoriel : Configuration de NetSuite pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce tutoriel est de vous montrer la procédure à suivre dans NetSuite OneWorld et Azure AD pour provisionner et déprovisionner automatiquement des comptes utilisateur d’Azure AD vers NetSuite.

> [!NOTE]
> Actuellement, cette intégration s’authentifie à l’aide de l’authentification de base (nom d’utilisateur et mot de passe). NetSuite a implémenté une exigence d’authentification multifacteur qui empêche les clients d’utiliser cette intégration, sauf exemption. Nous collaborons avec NetSuite afin de mettre à jour cette intégration avec une méthode d’authentification plus récente pour permettre aux clients sans aucune exemption de l’utiliser à nouveau. Nous mettrons à jour ce document avec un ETA lorsque celui-ci sera disponible.

Action recommandée : Veuillez patienter jusqu’à la publication d’une mise à jour du comportement d’authentification de cette intégration ou contactez le support technique NetSuite pour vous renseigner sur une exemption de l’exigence d’authentification multifacteur.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

*   Un locataire Azure Active Directory.
*   Un abonnement NetSuite OneWorld. Notez que le provisionnement automatique d’utilisateurs n’est actuellement pris en charge que par Netsuite OneWorld.
*   Un compte utilisateur dans Netsuite avec des autorisations d’administrateur.
*   Une intégration avec Azure AD requiert une exemption 2FA. Veuillez contacter l’équipe du support technique NetSuite pour demander cette exemption.

## <a name="assigning-users-to-netsuite-oneworld"></a>Affectation d’utilisateurs à NetSuite OneWorld

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique de comptes utilisateur, seuls les utilisateurs et les groupes qui ont été « assignés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD ont besoin d’accéder à votre application NetSuite. Une fois que vous avez effectuer votre choix, vous pouvez affecter ces utilisateurs à votre application NetSuite en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-netsuite-oneworld"></a>Conseils importants pour l’affectation d’utilisateurs à NetSuite OneWorld

*   Il est recommandé de n’assigner qu’un seul utilisateur Azure AD à NetSuite afin de tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

*   Lorsque vous attribuez un utilisateur à NetSuite, vous devez sélectionner un rôle d’utilisateur valide. Le rôle « Accès par défaut » ne fonctionne pas pour l’approvisionnement.

## <a name="enable-user-provisioning"></a>Activer l’approvisionnement des utilisateurs

Cette section explique comment connecter votre Azure AD à l’API d’approvisionnement de comptes utilisateur de NetSuite pour créer, mettre à jour et désactiver les comptes utilisateur affectés dans NetSuite en fonction des attributions d’utilisateurs et de groupes dans Azure AD.

> [!TIP] 
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour NetSuite en suivant les instructions fournies dans le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="to-configure-user-account-provisioning"></a>Pour configurer l’approvisionnement de comptes utilisateur :

L’objectif de cette section est d’expliquer comment activer l’approvisionnement utilisateur des comptes utilisateur Active Directory sur NetSuite.

1. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

1. Si vous avez déjà configuré NetSuite pour l’authentification unique, recherchez votre instance de NetSuite à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **NetSuite** dans la galerie d’applications. Sélectionnez NetSuite dans les résultats de recherche et ajoutez-la à votre liste d’applications.

1. Sélectionnez votre instance de NetSuite, puis sélectionnez l’onglet **Approvisionnement**.

1. Définissez le **Mode d’approvisionnement** sur **Automatique**. 

    ![Capture d’écran montrant la page d’approvisionnement NetSuite, avec le Mode d’approvisionnement défini sur Automatique et d’autres valeurs que vous pouvez définir.](./media/netsuite-provisioning-tutorial/provisioning.png)

1. Dans la section **Informations d’identification de l’administrateur** , fournissez les paramètres de configuration suivants :
   
    a. Dans la zone de texte **Nom d'utilisateur Admin** , tapez le nom d’un compte NetSuite auquel le profil **System Administrator** (Administrateur système) est affecté dans NetSuite.com.
   
    b. Dans la zone de texte **Mot de passe d’administrateur** , entrez le mot de passe de ce compte.
      
1. Dans le portail Azure, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à votre application NetSuite.

1. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case.

1. Cliquez sur **Enregistrer.**

1. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec NetSuite**.

1. Dans la section **Mappages d’attributs** , passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers NetSuite. Remarque : les attributs sélectionnés en tant que propriétés de **Correspondance** servent à faire correspondre les comptes utilisateur dans NetSuite, en vue d’opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

1. Pour activer le service d’approvisionnement Azure AD pour NetSuite, affectez la valeur **Activé** au paramètre **État de l’approvisionnement** dans la section Paramètres.

1. Cliquez sur **Enregistrer.**

Cette commande démarre la synchronisation initiale des utilisateurs et/ou des groupes affectés à NetSuite dans la section Utilisateurs et Groupes. Notez que la synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement dans votre application NetSuite.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](netsuite-tutorial.md)
