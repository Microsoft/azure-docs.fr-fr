---
title: 'Tutoriel : Configurer DocuSign pour l’approvisionnement automatique d’utilisateurs avec Azure Active Directory | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et DocuSign.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/21/2020
ms.author: jeedes
ms.openlocfilehash: dc3f307a21b746981a84b1c0747c4b22c448541f
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349903"
---
# <a name="tutorial-configure-docusign-for-automatic-user-provisioning"></a>Tutoriel : Configurer DocuSign pour l’approvisionnement automatique d’utilisateurs

L’objectif de ce didacticiel est de vous montrer les étapes à effectuer dans DocuSign et Azure AD pour approvisionner et retirer automatiquement des comptes utilisateur d’Azure AD vers DocuSign.

## <a name="prerequisites"></a>Prérequis

Le scénario décrit dans ce didacticiel part du principe que vous disposez des éléments suivants :

*   Un locataire Azure Active Directory.
*   Un abonnement DocuSign pour lequel l’authentification unique est activée.
*   Un compte d’utilisateur dans DocuSign avec les autorisations d’administrateur d’équipe.

## <a name="assigning-users-to-docusign"></a>Affectation d’utilisateurs à DocuSign

Azure Active Directory utilise un concept appelé « affectations » pour déterminer les utilisateurs devant recevoir l’accès aux applications sélectionnées. Dans le cadre de l’approvisionnement automatique de comptes utilisateur, seuls les utilisateurs et les groupes qui ont été « assignés » à une application dans Azure AD sont synchronisés.

Avant de configurer et d’activer le service d’approvisionnement, vous devez déterminer quels utilisateurs et/ou groupes dans Azure AD représentent les utilisateurs qui ont besoin d’accéder à votre application DocuSign. Une fois que vous avez choisi, vous pouvez assigner ces utilisateurs à votre application DocuSign en suivant les instructions fournies ici :

[Affecter un utilisateur ou un groupe à une application d’entreprise](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-docusign"></a>Conseils importants pour l’affectation d’utilisateurs à DocuSign

*   Il est recommandé de n’assigner qu’un seul utilisateur Azure AD à DocuSign afin de tester la configuration de l’approvisionnement. Les autres utilisateurs peuvent être affectés ultérieurement.

*   Quand vous assignez un utilisateur à DocuSign, vous devez sélectionner un rôle d’utilisateur valide. Le rôle « Accès par défaut » ne fonctionne pas pour l’approvisionnement.

> [!NOTE]
> Azure AD ne prend pas en charge le provisionnement de groupes avec l’application Docusign, seuls les utilisateurs peuvent être provisionnés.

## <a name="enable-user-provisioning"></a>Activer l’approvisionnement des utilisateurs

Cette section explique comment connecter votre annuaire Azure AD à l’API d’approvisionnement de comptes d’utilisateur de DocuSign pour créer, mettre à jour et désactiver les comptes d’utilisateur affectés dans DocuSign en fonction des attributions d’utilisateurs et de groupes dans Azure AD.

> [!Tip]
> Vous pouvez également choisir d’activer l’authentification unique basée sur SAML pour DocuSign en suivant les instructions fournies sur le [portail Azure](https://portal.azure.com). L’authentification unique peut être configurée indépendamment de l’approvisionnement automatique, bien que chacune de ces deux fonctionnalités compléte l’autre.

### <a name="to-configure-user-account-provisioning"></a>Pour configurer l’approvisionnement de comptes utilisateur :

L’objectif de cette section est d’expliquer comment activer l’approvisionnement utilisateur des comptes d’utilisateurs Active Directory sur DocuSign.

1. Sur le [portail Azure](https://portal.azure.com), accédez à la section **Azure Active Directory > Applications d’entreprise > Toutes les applications**.

1. Si vous avez déjà configuré DocuSign pour l’authentification unique, recherchez votre instance de DocuSign à l’aide du champ de recherche. Sinon, sélectionnez **Ajouter** et recherchez **DocuSign** dans la galerie d’applications. Sélectionnez DocuSign dans les résultats de recherche et ajoutez-la à votre liste d’applications.

1. Sélectionnez votre instance de DocuSign, puis sélectionnez l’onglet **Approvisionnement**.

1. Définissez le **Mode d’approvisionnement** sur **Automatique**. 

    ![Capture d’écran de l’onglet Provisionnement pour DocuSign dans le portail Azure. Le mode de provisionnement est défini sur Automatique et les champs Nom d’utilisateur administrateur, Mot de passe et Tester la connexion sont en évidence.](./media/docusign-provisioning-tutorial/provisioning.png)

1. Dans la section **Informations d’identification de l’administrateur**, fournissez les paramètres de configuration suivants :
   
    a. Dans la zone de texte **Nom d’utilisateur Admin**, tapez le nom d’un compte DocuSign auquel le profil **System Administrator** (Administrateur système) est affecté dans DocuSign.com.
   
    b. Dans la zone de texte **Mot de passe d’administrateur**, entrez le mot de passe de ce compte.

> [!NOTE]
> Si l’authentification unique et le provisionnement d’utilisateurs sont configurés, les informations d’identification d’autorisation utilisées pour le provisionnement doivent être configurées pour fonctionner avec l’authentification unique et le nom d’utilisateur/mot de passe.

1. Sur le portail Azure, cliquez sur **Tester la connexion** pour vérifier qu’Azure AD peut se connecter à votre application DocuSign.

1. Dans le champ **E-mail de notification**, entrez l’adresse e-mail d’une personne ou d’un groupe qui doit recevoir les notifications d’erreur d’approvisionnement, puis cochez la case.

1. Cliquez sur **Enregistrer.**

1. Dans la section Mappages, sélectionnez **Synchroniser les utilisateurs Azure Active Directory avec DocuSign**.

1. Dans la section **Mappages des attributs**, passez en revue les attributs utilisateur qui sont synchronisés d’Azure AD vers DocuSign. Les attributs sélectionnés en tant que propriétés de **Correspondance** sont utilisés pour faire correspondre les comptes d’utilisateur dans DocuSign pour les opérations de mise à jour. Cliquez sur le bouton Enregistrer pour valider les modifications.

1. Pour activer le service d’approvisionnement Azure AD pour DocuSign, affectez au paramètre **Statut d’approvisionnement** la valeur **Activé** dans la section Paramètres.

1. Cliquez sur **Enregistrer.**

Cette commande démarre la synchronisation initiale des utilisateurs affectés à DocuSign dans la section Utilisateurs et Groupes. La synchronisation initiale prend plus de temps que les synchronisations suivantes, qui se produisent toutes les 40 minutes environ tant que le service est en cours d’exécution. Vous pouvez utiliser la section **Détails de la synchronisation** pour surveiller la progression et suivre les liens vers les journaux d’activité de provisionnement, qui décrivent toutes les actions effectuées par le service de provisionnement dans votre application DocuSign.

Pour plus d’informations sur la lecture des journaux d’activité d’approvisionnement Azure AD, consultez [Création de rapports sur l’approvisionnement automatique de comptes d’utilisateur](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="troubleshooting-tips"></a>Conseils de dépannage
* L’approvisionnement d’un rôle ou d’un profil d’autorisation pour un utilisateur dans Docusign peut être effectué à l’aide d’une expression dans vos mappages d’attributs à l’aide des fonctions [switch](../app-provisioning/functions-for-customizing-application-data.md#switch) et [singleAppRoleAssignment](../app-provisioning/functions-for-customizing-application-data.md#singleapproleassignment). Par exemple, l’expression ci-dessous approvisionne l’ID « 8032066 » lorsqu’un utilisateur a le rôle « administrateur DS » affecté dans Azure AD. Aucun profil d’autorisation n’est approvisionné si l’utilisateur n’est pas affecté à un rôle dans Azure AD. L’ID peut être récupéré à partir du [portail](https://support.docusign.com/articles/Default-settings-for-out-of-the-box-DocuSign-Permission-Profiles) DocuSign.

Switch(SingleAppRoleAssignment([appRoleAssignments])," ", "8032066", "DS Admin")


## <a name="additional-resources"></a>Ressources supplémentaires

* [Gestion de l’approvisionnement de comptes d’utilisateur pour les applications d’entreprise](tutorial-list.md)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../manage-apps/what-is-single-sign-on.md)
* [Configurer l’authentification unique](docusign-tutorial.md)