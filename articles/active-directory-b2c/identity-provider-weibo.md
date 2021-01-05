---
title: Configurer l’inscription et la connexion avec un compte Weibo
titleSuffix: Azure AD B2C
description: Proposez l’inscription et la connexion aux clients disposant de comptes Weibo dans vos applications à l’aide d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/08/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 08aa7e4af6dc5d5e5bff470bc4c5d023e25b3014
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85387879"
---
# <a name="set-up-sign-up-and-sign-in-with-a-weibo-account-using-azure-active-directory-b2c"></a>Configurer l’inscription et la connexion avec un compte Weibo à l’aide d’Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-public-preview](../../includes/active-directory-b2c-public-preview.md)]

## <a name="create-a-weibo-application"></a>Créer une application Weibo

Pour utiliser un compte Weibo en tant que fournisseur d’identité dans Azure Active Directory B2C (Azure AD B2C), vous devez créer dans votre locataire une application qui le représente. Si vous n’avez pas encore de compte Weibo, vous pouvez vous inscrire à l’adresse [https://weibo.com/signup/signup.php?lang=en-us](https://weibo.com/signup/signup.php?lang=en-us).

1. Connectez-vous au [portail des développeurs Weibo](https://open.weibo.com/) avec les informations d’identification de votre compte Weibo.
1. Une fois que vous êtes connecté, sélectionnez votre nom d’affichage dans le coin supérieur droit.
1. Dans la liste déroulante, sélectionnez **编辑开发者信息** (Modifier les informations développeur).
1. Entrez les informations requises et sélectionnez **提交** (Envoyer).
1. Finalisez le processus de vérification d’e-mail.
1. Accédez à la page de [vérification d’identité](https://open.weibo.com/developers/identity/edit).
1. Entrez les informations requises et sélectionnez **提交** (Envoyer).

### <a name="register-a-weibo-application"></a>Inscrire une application Weibo

1. Accédez à la [Page d’inscription de l’application Weibo](https://open.weibo.com/apps/new).
1. Entrez les informations nécessaires relatives à l’application.
1. Sélectionnez **创建** (Créer).
1. Copiez les valeurs **Clé d’application** et **Secret d’application**. Vous avez besoin des deux pour ajouter le fournisseur d’identité à votre locataire.
1. Téléchargez les photos requises et entrez les informations nécessaires.
1. Sélectionnez **保存以上信息** (Enregistrer).
1. Sélectionnez **高级信息** (Informations avancées).
1. Sélectionnez **编辑** (Modifier) en regard du champ pour OAuth2.0 **授权设置**(URL de redirection).
1. Entrez `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp` pour OAuth2.0 **授权设置**(URL de redirection). Par exemple, si le nom de votre locataire est contoso, définissez l’URL sur `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`.
1. Sélectionnez **提交** (Envoyer).

## <a name="configure-a-weibo-account-as-an-identity-provider"></a>Configuration d’un compte Weibo en tant que fournisseur d’identité

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur général de votre locataire Azure AD B2C.
1. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.
1. Choisissez **Tous les services** dans le coin supérieur gauche du Portail Azure, recherchez et sélectionnez **Azure Active Directory B2C**.
1. Cliquez sur **Fournisseurs d’identité**, puis sélectionnez **Weibo (Préversion)** .
1. Saisissez un **Nom**. Par exemple, *Weibo*.
1. Dans **ID client**, entrez la clé d’application de l’application Weibo que vous avez créée précédemment.
1. Dans **Clé secrète client**, entrez la clé secrète d’application que vous avez consignée.
1. Sélectionnez **Enregistrer**.
