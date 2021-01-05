---
title: "Tutoriel : Configurer Salesforce Sandbox pour l'approvisionnement automatique d'utilisateurs avec Azure Active Directory | Microsoft Docs"
description: Découvrez les étapes que vous devez effectuer dans Salesforce Sandbox et Azure AD pour approvisionner et déprovisionner automatiquement des comptes d’utilisateur d’Azure AD vers Salesforce Sandbox.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/26/2018
ms.author: jeedes
ms.openlocfilehash: 7e3f8e5e975468b468712ae8907cdca0e80a5f9f
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94352606"
---
# <a name="tutorial-configure-salesforce-sandbox-for-automatic-user-provisioning"></a>Tutoriel : Configurer Salesforce Sandbox pour l'approvisionnement automatique d'utilisateurs

L’objectif de ce didacticiel est de présenter les étapes à effectuer dans Salesforce Sandbox et Azure AD permettant d’approvisionner et retirer automatiquement des comptes utilisateur d’Azure AD vers Salesforce Sandbox.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

*   Un locataire Azure Active Directory.
*   Un abonné valide pour les éditions Salesforce Sandbox for Work ou Salesforce Sandbox for Education. Vous pouvez utiliser un compte d’essai gratuit pour chaque service.
*   Un compte d’utilisateur dans Salesforce Sandbox avec les autorisations d’administrateur d’équipe.

## <a name="assigning-users-to-salesforce-sandbox"></a>Assigner des utilisateurs à Salesforce Sandbox

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique de comptes utilisateur, seuls les utilisateurs et les groupes qui ont été « assignés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service de provisionnement, vous devez déterminer quels utilisateurs ou groupes dans Azure AD ont besoin d’accéder à votre application Salesforce Sandbox. Une fois cette décision prise, vous pouvez affecter ces utilisateurs à votre application Salesforce Sandbox en suivant les instructions fournies dans [affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-salesforce-sandbox"></a>Conseils importants concernant l’assignation d’utilisateurs à Salesforce Sandbox

* Il est recommandé de n’assigner qu’un seul utilisateur Azure AD à Salesforce Sandbox afin de tester la configuration de l’approvisionnement. Les autres utilisateurs et/ou groupes peuvent être affectés ultérieurement.

* Lorsque vous assignez un utilisateur à Salesforce Sandbox, vous devez sélectionner un rôle d’utilisateur valide. Le rôle « Accès par défaut » ne fonctionne pas pour l’approvisionnement.

> [!NOTE]
> Cette application importe des rôles personnalisés à partir de Salesforce Sandbox dans le cadre du processus d’approvisionnement. Il est possible que le client souhaite sélectionner ce processus lors de l’assignation des utilisateurs.

## <a name="enable-automated-user-provisioning"></a>Activer l’approvisionnement automatique des utilisateurs

Cette section va vous guider afin de connecter votre instance Azure AD à votre compte utilisateur Salesforce Sandbox qui fournit l’API. À l’aide de ce guide, vous pourrez aussi découvrir comment configurer le service d’approvisionnement pour créer, mettre à jour et désactiver des comptes utilisateur assignés dans Salesforce Sandbox, en fonction des attributions d’utilisateur et de groupe dans Azure AD.

>[!Tip]
>Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour Salesforce Sandbox, grâce aux instructions disponibles dans le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="configure-automatic-user-account-provisioning"></a>Configurer l’approvisionnement automatique d’un compte utilisateur

Cette section décrit comment activer l’approvisionnement des utilisateurs des comptes d’utilisateurs Active Directory sur Salesforce Sandbox.

1. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

1. Si vous avez déjà configuré Salesforce Sandbox pour l’authentification unique, recherchez votre instance de Salesforce Sandbox à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et effectuer une recherche pour **Salesforce Sandbox** dans la galerie d’applications. Dans les résultats de la recherche, sélectionnez Salesforce Sandbox, puis ajoutez-la à votre liste d’applications.

1. Sélectionnez votre instance de Salesforce Sandbox, puis sélectionnez l’onglet **Approvisionnement**.

1. Définissez le **Mode d’approvisionnement** sur **Automatique**.

    ![Capture d’écran montrant la page d’approvisionnement de Salesforce Sandbox, avec le Mode d’approvisionnement défini sur Automatique et d’autres valeurs que vous pouvez définir.](./media/salesforce-sandbox-provisioning-tutorial/provisioning.png)

1. Dans la section **Informations d’identification de l’administrateur** , fournissez les paramètres de configuration suivants :
   
    a. Dans la zone de texte **Nom d’utilisateur administrateur Salesforce Sandbox** , entrez le nom d’un compte Salesforce Sandbox auquel le profil **Administrateur système** est assigné dans Salesforce.com.
   
    b. Dans la zone de texte **Mot de passe d’administrateur** , entrez le mot de passe de ce compte.

1. Pour obtenir le jeton de sécurité Salesforce Sandbox, ouvrez un nouvel onglet et connectez-vous au même compte Administrateur Salesforce Sandbox. Dans le coin supérieur droit de la page, cliquez sur votre nom, puis cliquez sur **Paramètres**.

     ![Capture d’écran montrant le lien Paramètres sélectionné.](./media/salesforce-sandbox-provisioning-tutorial/sf-my-settings.png "Activer l'approvisionnement de l'utilisateur automatique.")

1. Dans le volet de navigation gauche, cliquez sur **Mes informations personnelles** pour développer la section associée, puis sur **Réinitialiser mon jeton de sécurité**.
  
    ![Capture d’écran montrant Réinitialiser mon jeton de sécurité sélectionné à partir de Mes informations personnelles.](./media/salesforce-sandbox-provisioning-tutorial/sf-personal-reset.png "Activer l'approvisionnement de l'utilisateur automatique.")

1. Sur la page **Réinitialiser le jeton de sécurité** , cliquez sur le bouton **Réinitialiser le jeton de sécurité**.

    ![Capture d’écran montrant la page Jeton de sécurité REST, avec du texte explicatif et l’option de réinitialisation du jeton de sécurité](./media/salesforce-sandbox-provisioning-tutorial/sf-reset-token.png "Activer l'approvisionnement de l'utilisateur automatique.")

1. Contrôlez la boîte de réception associée à ce compte d’administrateur. Vous allez recevoir un e-mail de la part de Salesforce Sandbox.com dans lequel se trouve le nouveau jeton de sécurité.

1. Copiez le jeton, accédez à votre fenêtre Azure AD et collez-le dans le champ **Jeton secret**.

1. Dans le portail Azure, cliquez sur **Tester la connexion** pour vous assurer qu’Azure AD peut se connecter à votre application Salesforce Sandbox.

1. Dans le champ **E-mail de notification** , entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case.

1. Cliquez sur **Enregistrer.**  
    
1.  Dans la section Mappages, sélectionnez **Synchroniser des utilisateurs Azure Active Directory avec Salesforce Sandbox**.

1. Dans la section **Mappages d’attributs** , passez en revue les attributs utilisateur qui sont synchronisés à partir d’Azure AD vers Salesforce Sandbox. Les attributs sélectionnés en tant que propriétés de **Correspondance** servent à faire correspondre les comptes utilisateur dans Salesforce Sandbox, en vue d’opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

1. Pour activer le service d’approvisionnement Azure AD pour Salesforce Sandbox, accédez à la section Paramètres et définissez l’ **État de l’approvisionnement** sur **Activé**

1. Cliquez sur **Enregistrer.**

Cette commande démarre la synchronisation initiale des utilisateurs et/ou des groupes affectés à Salesforce Sandbox dans la section Utilisateurs et Groupes. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement dans votre application Salesforce Sandbox.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](./salesforce-sandbox-tutorial.md)