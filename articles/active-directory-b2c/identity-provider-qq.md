---
title: Configurer l’inscription et la connexion avec un compte QQ à l’aide d’Azure Active Directory B2C
description: Proposez l’inscription et la connexion aux clients disposant de comptes QQ dans vos applications à l’aide d’Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 08/08/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: d230bc8a1e9bf388e1cca4e3a3a691223146d734
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85387981"
---
# <a name="set-up-sign-up-and-sign-in-with-a-qq-account-using-azure-active-directory-b2c"></a>Configurer l’inscription et la connexion avec un compte QQ à l’aide d’Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-public-preview](../../includes/active-directory-b2c-public-preview.md)]

## <a name="create-a-qq-application"></a>Créer une application QQ

Pour utiliser un compte QQ en tant que fournisseur d’identité dans Azure Active Directory B2C (Azure AD B2C), vous devez créer dans votre locataire une application qui le représente. Si vous n’avez pas encore de compte QQ, vous pouvez en souscrire un à l’adresse [https://ssl.zc.qq.com/en/index.html?type=1&ptlang=1033](https://ssl.zc.qq.com/en/index.html?type=1&ptlang=1033).

### <a name="register-for-the-qq-developer-program"></a>Inscrivez-vous au programme de développement QQ

1. Connectez-vous au [portail des développeurs QQ](http://open.qq.com) avec les informations d’identification de votre compte QQ.
1. Une fois connecté, accédez à [https://open.qq.com/reg](https://open.qq.com/reg) vous inscrire en tant que développeur.
1. Sélectionnez **个人** (développeur individuel).
1. Entrez les informations requises et sélectionnez **下一步** (étape suivante).
1. Finalisez le processus de vérification d’e-mail. Une fois que vous êtes inscrit en tant que développeur, vous devez attendre votre approbation pendant quelques jours.

### <a name="register-a-qq-application"></a>Inscrire une application QQ

1. Accédez à [https://connect.qq.com/index.html](https://connect.qq.com/index.html).
1. Sélectionnez **应用管理** (gestion des applications).
1. Sélectionnez**创建应用**(créer une application) et entrez les informations requises.
1. Dans le champ **授权回调域** (URL de rappel), entrez `https://your-tenant-name.b2clogin.com/your-tenant-name}.onmicrosoft.com/oauth2/authresp`. Par exemple, si votre `tenant_name` est contoso, définissez l’URL sur `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`.
1. Sélectionnez **创建应用** (créer une application).
1. Dans la page de confirmation, sélectionnez **应用管理** (gestion des applications) pour revenir à la page de gestion des applications.
1. Sélectionnez **查看** (afficher) en regard de l’application que vous avez créée.
1. Sélectionnez**修改**(modifier).
1. Copiez l’**ID de l’application** et la **clé de l’application**. Vous avez besoin de ces deux valeurs pour ajouter le fournisseur d’identité à votre locataire.

## <a name="configure-qq-as-an-identity-provider"></a>Configuration de QQ en tant que fournisseur d'identité

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez le répertoire qui contient votre locataire Azure AD B2C.
1. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.
1. Cliquez sur **Fournisseurs d’identité**, puis sélectionnez **QQ (Préversion)** .
1. Saisissez un **Nom**. Par exemple, *QQ*.
1. Dans **ID client**, entrez l’ID d’application de l’application QQ que vous avez créée précédemment.
1. Dans **Clé secrète client**, entrez la clé d’application que vous avez enregistrée.
1. Sélectionnez **Enregistrer**.
