---
title: Télécharger une liste d'utilisateurs dans le portail Azure Active Directory | Microsoft Docs
description: Téléchargez des enregistrements d’utilisateur en bloc dans le centre d’administration Azure dans Azure Active Directory.
services: active-directory
author: curtand
ms.author: curtand
manager: daveba
ms.date: 12/02/2020
ms.topic: how-to
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.custom: it-pro
ms.reviewer: krbain
ms.collection: M365-identity-device-management
ms.openlocfilehash: 042139a39c28ee4944a7f3f766fc61b163629843
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574392"
---
# <a name="download-a-list-of-users-in-azure-active-directory-portal"></a>Télécharger une liste d’utilisateurs dans le portail Azure Active Directory

Azure Active Directory (Azure AD) prend en charge les opérations d’importation (création) d’utilisateurs en bloc.

## <a name="required-permissions"></a>Autorisations requises

Pour télécharger la liste des utilisateurs à partir du centre d’administration Azure AD, vous devez être connecté avec un ID d’utilisateur associé à un ou plusieurs rôles Administrateur au niveau de l’organisation dans Azure AD (Administrateur d'utilisateurs correspond au rôle minimum requis). Les inviteurs d’invités et les développeurs d’applications ne sont pas considérés comme des rôles Administrateur.

## <a name="to-download-a-list-of-users"></a>Pour télécharger une liste d’utilisateurs

1. [Connectez-vous à votre organisation](https://aad.portal.azure.com) Azure AD avec un compte Administrateur d’utilisateurs de l’organisation.
2. Accédez à Azure Active Directory > Utilisateurs. Sélectionnez ensuite les utilisateurs que vous souhaitez inclure dans le téléchargement en cochant la case dans la colonne de gauche en regard de chaque utilisateur. Remarque : Pour le moment, il n’existe aucun moyen de sélectionner tous les utilisateurs pour l’exportation. Chacun d’eux doit être sélectionné individuellement.
3. Dans Azure AD, sélectionnez **Utilisateurs** > **Télécharger les utilisateurs**.
4. Dans la page **Télécharger les utilisateurs**, sélectionnez **Démarrer** pour recevoir un fichier CSV répertoriant les propriétés de profil utilisateur. Si des erreurs se produisent, vous pouvez télécharger et consulter le fichier de résultats sur la page Résultats de l’opération en bloc. Le fichier contient la raison de chaque erreur.

   ![Sélectionnez l’emplacement où vous souhaitez télécharger la liste des utilisateurs](./media/users-bulk-download/bulk-download.png)

   Le fichier de téléchargement contiendra la liste filtrée des utilisateurs.

   Les attributs utilisateur suivants sont inclus :

   - userPrincipalName
   - displayName
   - surname
   - mail
   - givenName
   - objectId
   - userType
   - jobTitle
   - department
   - accountEnabled
   - usageLocation
   - streetAddress
   - state
   - country
   - physicalDeliveryOfficeName
   - city
   - postalCode
   - telephoneNumber
   - mobile
   - authenticationPhoneNumber
   - authenticationAlternativePhoneNumber
   - authenticationEmail
   - alternateEmailAddress
   - ageGroup
   - consentProvidedForMinor
   - legalAgeGroupClassification

## <a name="check-status"></a>Vérification du statut

Pour connaître l'état de vos demandes d'opérations en bloc en attente, consultez la page **Résultats des opérations en bloc**.

[![Vérifier l'état sur la page Résultats des opérations en bloc.](./media/users-bulk-download/bulk-center.png)](./media/users-bulk-download/bulk-center.png#lightbox)

## <a name="bulk-download-service-limits"></a>Limites du service de téléchargement en bloc

Chaque opération en bloc de création de liste d’utilisateurs peut prendre jusqu’à une heure. Cela permet la création et le téléchargement d’une liste d’au moins 500 000 utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter des utilisateurs en bloc](users-bulk-add.md)
- [Supprimer des utilisateurs en bloc](users-bulk-delete.md)
- [Restaurer des utilisateurs en bloc](users-bulk-restore.md)
