---
title: Consentement administrateur pour les connexions au compte LinkedIn – Azure AD | Microsoft Docs
description: Explique comment activer et désactiver les connexions de compte d’intégration LinkedIn dans les applications Microsoft dans Azure Active Directory
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: beengen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5ffa937f91af7078b28331a655f99397d3821def
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96545891"
---
# <a name="integrate-linkedin-account-connections-in-azure-active-directory"></a>Intégrer des connexions de compte LinkedIn dans Azure Active Directory

Vous pouvez autoriser des utilisateurs de votre organisation à accéder à leurs connexions LinkedIn dans certaines applications Microsoft. Aucune donnée n’est partagée tant que l’utilisateur n’a pas consenti à la connexion à son compte. Vous pouvez intégrer votre organisation dans le [centre d’administration](https://aad.portal.azure.com) Azure Active Directory (Azure AD).

> [!IMPORTANT]
> Le paramètre Connexions au compte LinkedIn est en cours de déploiement vers les organisations Azure AD. Une fois déployé dans votre organisation, il est activé par défaut.
>
> Exceptions :
>
> * Le paramètre n’est pas disponible pour les clients qui utilisent Microsoft Cloud for US Government, Microsoft Cloud Germany ou Azure et Microsoft 365 gérés par 21Vianet en Chine.
> * Le paramètre est désactivé par défaut pour les organisations Azure AD provisionnées en Allemagne. Notez que le paramètre n’est pas disponible pour les clients qui utilisent Microsoft Cloud Germany.
> * Le paramètre est désactivé par défaut pour les organisations provisionnées en France.
>
> Une fois les connexions de compte LinkedIn activées pour votre organisation, elles fonctionnent après que les utilisateurs ont consenti à ce que des applications accèdent aux données de l’entreprise en leur nom. Pour plus d’informations sur le paramètre de consentement de l’utilisateur, voir [Comment supprimer l’accès d’un utilisateur à une application](../manage-apps/methods-for-removing-user-access.md).

## <a name="enable-linkedin-account-connections-in-the-azure-portal"></a>Activer les connexions au compte LinkedIn dans le portail Azure

Vous pouvez activer les connexions au compte LinkedIn pour les utilisateurs de votre choix, qu’il s’agisse de l’ensemble de votre organisation aux de certains utilisateurs sélectionnés au sein de celle-ci.

1. Connectez-vous au [centre d’administration d’Azure AD](https://aad.portal.azure.com/) avec un compte d’administrateur général pour l’organisation Azure AD.
1. Sélectionnez **Utilisateurs**.
1. Dans la page **Utilisateurs**, sélectionnez **Paramètres utilisateur**.
1. Sous **Connexions au compte LinkedIn**, autorisez des utilisateurs à se connecter à leur compte pour accéder à leur connexions LinkedIn dans certaines applications Microsoft. Aucune donnée n’est partagée tant que l’utilisateur n’a pas consenti à la connexion à son compte.

    * Sélectionnez **Oui** pour activer le service pour tous les utilisateurs de votre organisation
    * Choisissez **Groupe sélectionné** pour activer le service uniquement pour un groupe d’utilisateurs sélectionnés au sein de votre organisation
    * Sélectionnez **Non** pour retirer le consentement de tous les utilisateurs au sein de votre organisation

    ![Intégrer les connexions au compte LinkedIn dans l’organisation](./media/linkedin-integration/linkedin-integration.png)

1. Lorsque vous avez terminé, sélectionnez **Enregistrer** pour enregistrer vos paramètres.

> [!Important]
> L’intégration de LinkedIn n’est pas entièrement activée pour vos utilisateurs tant que ceux-ci n’ont pas consenti à la connexion à leur compte. Aucune donnée n’est partagée lorsque vous activez les connexions au compte pour vos utilisateurs.

### <a name="assign-selected-users-with-a-group"></a>Affecter des utilisateurs sélectionnés avec un groupe

Nous avons remplacé l’option « Sélectionnés » qui spécifie une liste d’utilisateurs par la possibilité de sélectionner un groupe d’utilisateurs, de sorte que vous puissiez activer la possibilité de se connecter à des comptes LinkedIn et Microsoft pour un groupe plutôt que pour plusieurs utilisateurs individuels. Si vous n’avez pas de connexions au compte LinkedIn activées pour des utilisateurs sélectionnés, vous n’avez pas besoin de faire quoi que ce soit. Si vous avez précédemment activé les connexions au compte LinkedIn pour des utilisateurs sélectionnés, vous devez :

1. Obtenir la liste actuelle des utilisateurs
1. Déplacer des utilisateurs actuellement activés vers un groupe
1. Utilisez le groupe précédent comme groupe sélectionné dans le paramètre Connexions au compte LinkedIn dans le centre d’administration Azure AD.

> [!NOTE]
> Même si vous ne déplacez pas les utilisateurs actuellement sélectionnés vers un groupe, ils peuvent toujours voir les informations LinkedIn dans des applications Microsoft.

### <a name="move-currently-selected-users-to-a-group"></a>Déplacer des utilisateurs sélectionnés vers un groupe

1. Créez un fichier CSV des utilisateurs sélectionnés pour les connexions de compte LinkedIn.
1. Connectez-vous à Microsoft 365 avec votre compte d’administrateur.
1. Lancez PowerShell.
1. Installez le module Azure AD en exécutant `Install-Module AzureAD`.
1. Exécutez le script suivant :

  ``` PowerShell
  $groupId = "GUID of the target group"
  
  $users = Get-Content 
  Path to the CSV file
  
  $i = 1
  foreach($user in $users} { Add-AzureADGroupMember -ObjectId $groupId -RefObjectId $user ; Write-Host $i Added $user ; $i++ ; Start-Sleep -Milliseconds 10 }
  ```

Pour utiliser le groupe l’étape deux en tant que groupe sélectionné dans le paramètre Connexions au compte LinkedIn dans le centre d’administration Azure AD, voir [Activer les connexions au compte LinkedIn dans le portail Azure](#enable-linkedin-account-connections-in-the-azure-portal).

## <a name="use-group-policy-to-enable-linkedin-account-connections"></a>Utiliser une stratégie de groupe pour activer les connexions au compte LinkedIn

1. Télécharger les [fichiers modèles d’administration Office 2016 (ADMX/ADML)](https://www.microsoft.com/download/details.aspx?id=49030)
1. Extrayez les fichiers **ADMX** et copiez-les dans votre magasin central.
1. Ouvrez la gestion des stratégies de groupe.
1. Créez un objet de stratégie de groupe avec le paramètre suivant : **Configuration utilisateur** > **Modèles d’administration** > **Microsoft Office 2016** > **Divers** > **Afficher les fonctionnalités LinkedIn dans les applications Office**.
1. Sélectionnez **Activé** ou **Désactivé**.
  
   State | Résultat
   ------ | ------
   **Activé** | Le paramètre **Afficher les fonctionnalités LinkedIn dans les applications Office** dans les options d’Office 2016 est activé. Les utilisateurs de votre organisation peuvent utiliser les fonctionnalités LinkedIn dans leurs applications Office 2016.
   **Désactivé** | Le paramètre **Afficher les fonctionnalités LinkedIn dans les applications Office** dans les options d’Office 2016 est désactivé et les utilisateurs finaux ne peuvent pas le modifier. Les utilisateurs de votre organisation ne peuvent pas utiliser les fonctionnalités LinkedIn dans leurs applications Office 2016.

Cette stratégie de groupe affecte uniquement les applications Office 2016 installées sur un ordinateur local. Si les utilisateurs désactivent LinkedIn dans leurs applications Office 2016, ils peuvent toujours voir les fonctionnalités LinkedIn dans Microsoft 365.

## <a name="next-steps"></a>Étapes suivantes

* [Consentement de l’utilisateur et partage de données pour LinkedIn](linkedin-user-consent.md)

* [Informations et fonctionnalités LinkedIn dans les applications Microsoft](https://go.microsoft.com/fwlink/?linkid=850740)

* [Centre d’aide LinkedIn](https://www.linkedin.com/help/linkedin)

* [Afficher vos paramètres d’intégration LinkedIn actuels dans le portail Azure](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/UserSettings)