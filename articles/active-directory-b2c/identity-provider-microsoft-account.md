---
title: Configurer l’inscription et la connexion avec un compte Microsoft
titleSuffix: Azure AD B2C
description: Proposez l’inscription et la connexion aux clients disposant de comptes Microsoft dans vos applications à l’aide d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/12/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: ad4b08a12e63b4ae3eed0eb09e295d9730de97bb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85387998"
---
# <a name="set-up-sign-up-and-sign-in-with-a-microsoft-account-using-azure-active-directory-b2c"></a>Configurer l’inscription et la connexion avec un compte Microsoft à l’aide d’Azure Active Directory B2C

## <a name="create-a-microsoft-account-application"></a>Créer une application de compte Microsoft

Pour utiliser un compte Microsoft en tant que [fournisseur d’identité](openid-connect.md) dans Azure Active Directory B2C (Azure AD B2C), vous devez créer une application dans le locataire Azure AD. Le locataire Azure AD n’est pas identique à votre locataire Azure AD B2C. Si vous n’avez pas encore de compte Microsoft, vous pouvez en créer un sur [https://www.live.com/](https://www.live.com/).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire Azure AD.
1. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Inscriptions d’applications**.
1. Sélectionnez **Nouvelle inscription**.
1. Entrez un **nom** pour votre application. Par exemple, *MSAapp1*.
1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un annuaire organisationnel (tout annuaire Azure AD mutualisé) et comptes Microsoft personnels (par exemple, Skype, Xbox)** .

   Pour plus d’informations sur les différents types de compte disponibles, consultez [Démarrage rapide : Inscrire une application avec la plateforme des identités Microsoft](../active-directory/develop/quickstart-register-app.md).
1. Sous **URI de redirection (facultatif)** , sélectionnez **Web**, puis entrez `https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/oauth2/authresp` dans la zone de texte. Remplacez `<tenant-name>` par le nom de votre locataire Azure AD B2C.
1. Sélectionnez **Inscrire**.
1. Enregistrez l’**ID d’application (client)** indiqué dans la page Vue d’ensemble de l’application. Vous en aurez besoin au moment de configurer le fournisseur d’identité dans la section suivante.
1. Sélectionnez **Certificats et secrets**.
1. Cliquez sur **Nouveau secret client**.
1. Entrez une **Description** pour le secret, par exemple *Mot de passe 1 de l’application*, puis cliquez sur **Ajouter**.
1. Enregistrez le mot de passe de l’application affiché dans la colonne **Valeur**. Vous en aurez besoin au moment de configurer le fournisseur d’identité dans la section suivante.

## <a name="configure-a-microsoft-account-as-an-identity-provider"></a>Configuration d’un compte Microsoft en tant que fournisseur d’identité

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur général de votre locataire Azure AD B2C.
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.
1. Choisissez **Tous les services** dans le coin supérieur gauche du Portail Azure, recherchez et sélectionnez **Azure Active Directory B2C**.
1. Sélectionnez **Fournisseurs d’identité**, puis **Compte Microsoft**.
1. Saisissez un **Nom**. Par exemple, *MSA*.
1. Dans **ID client**, entrez l’ID (client) de l’application Azure AD que vous avez créée précédemment.
1. Dans **Clé secrète client**, entrez la clé secrète client que vous avez enregistrée.
1. Sélectionnez **Enregistrer**.
