---
title: Ajouter ou mettre à jour les informations de profil utilisateur - Azure AD
description: Ces instructions vous expliquent comment ajouter des informations à un profil utilisateur dans Azure Active Directory, y compris une image et des informations sur son poste.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 04/11/2019
ms.author: ajburnle
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 27896bac6f5436fb6717e4bc1ab0a1f9381904ad
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370914"
---
# <a name="add-or-update-a-users-profile-information-using-azure-active-directory"></a>Ajouter ou mettre à jour les informations du profil utilisateur avec Azure Active Directory
Ajoutez des informations à un profil utilisateur, dont une image de profil, des informations sur l’emploi, et quelques réglages avec Azure Active Directory (Azure AD). Pour en savoir plus sur l’ajout de nouveaux utilisateurs, consultez [Comment ajouter ou supprimer des utilisateurs dans Azure Active Directory](add-users-azure-active-directory.md).

## <a name="add-or-change-profile-information"></a>Ajouter ou modifier les informations de profil
Vous trouverez bien plus d’informations disponibles dans un profil utilisateur qu’il est possible d’en ajouter lors de la création de l’utilisateur. Toutes ces informations supplémentaires sont facultatives et peuvent être ajoutées si besoin par votre organisation.

## <a name="to-add-or-change-profile-information"></a>Ajouter ou modifier les informations de profil
1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur d’utilisateurs de l’organisation.

2. Sélectionnez **Azure Active Directory** , **Utilisateurs** , puis un utilisateur. Par exemple, _Alain Charon_ .

    La page **Alain Charon - Profil** s’affiche.

    ![Page du profil utilisateur, avec les informations modifiables](media/active-directory-users-profile-azure-portal/user-profile-all-blade.png)

3. Sélectionnez **Modifier** pour ajouter ou mettre à jour les informations incluses dans chaque section disponible (facultatif).

    ![Page du profil utilisateur, montrant les zones modifiables](media/active-directory-users-profile-azure-portal/user-profile-edit.png)

    - **Image du profil** . Sélectionnez une image miniature pour le compte de l’utilisateur. Cette image s’affiche dans Azure Active Directory et sur la page personnelle de l’utilisateur, comme la page myapps.microsoft.com.

    - **Identité** . Ajoutez ou mettez à jour une valeur d’identité supplémentaire pour l’utilisateur, comme le nom de femme mariée. Vous pouvez définir ce nom indépendamment des valeurs du prénom et du nom. Par exemple, vous pouvez l’utiliser pour ajouter des initiales, un nom de société ou pour modifier la séquence des noms indiqués. Dans un autre exemple, pour deux utilisateurs dont les noms sont « Alain Davignon » vous pouvez utiliser la chaîne d’identité pour définir leurs noms comme « Alain B Davignon » et « Alain F Davignon (Contoso). »

    - **Informations sur l’emploi** . Ajoutez toutes les informations relatives à l’emploi, telles que le poste de l’utilisateur, le département ou son responsable.

    - **Réglages** . Décidez si l’utilisateur peut se connecter au locataire Azure Active Directory. Vous pouvez aussi spécifier l’emplacement global de l’utilisateur.

    - **Informations de contact** . Ajoutez toutes les informations de contact pertinentes pour l’utilisateur, à l’exception des coordonnées de téléphone ou de contact mobile de l’utilisateur (seul un administrateur général peut effectuer la mise à jour pour des utilisateurs dotés d’un rôle d’administrateur).

    - **Informations de contact d’authentification** . Vérifiez ces informations pour vous assurer que le numéro de téléphone et l’adresse e-mail de l’utilisateur sont valides. Azure Active Directory utilise ces informations pour s’assurer que l’utilisateur est véritablement un utilisateur lors de la connexion. Les informations de contact d’authentification ne peuvent être mises à jour que par l’administrateur général.

4. Sélectionnez **Enregistrer** .

    Toutes vos modifications sont enregistrées pour l’utilisateur.

    >[!Note]
    >Vous devez utiliser Windows Server Active Directory pour mettre à jour l’identité, les informations de contact ou sur l’emploi des utilisateurs dont la source d’autorité est Windows Server Active Directory. Une fois la mise à jour terminée, vous devez attendre la fin du prochain cycle de synchronisation pour constater les modifications.

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez mis à jour les profils de vos utilisateurs, vous pouvez exécuter les procédures de base suivantes :

- [Ajouter ou supprimer des utilisateurs](add-users-azure-active-directory.md)

- [Attribuer des rôles aux utilisateurs](active-directory-users-assign-role-azure-portal.md)

- [Créer un groupe de base et ajouter des membres](active-directory-groups-create-azure-portal.md)

Vous pouvez également réaliser d’autres tâches de gestion d’utilisateur, comme l’affectation de délégués, l’utilisation de stratégies et le partage de comptes d’utilisateur. Pour en savoir plus sur les autres actions disponibles, consultez la [documentation Gestion des utilisateurs Azure Active Directory](../enterprise-users/index.yml).
