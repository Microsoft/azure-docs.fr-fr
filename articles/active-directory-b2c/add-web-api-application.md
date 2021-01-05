---
title: Ajouter une application API web - Azure Active Directory B2C | Microsoft Docs
description: Apprenez à ajouter une application API web à votre locataire Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 04/16/2019
ms.custom: mvc
ms.topic: how-to
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 0eeff0c8c338b09fbe375587db2b955a143138c2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94949817"
---
# <a name="add-a-web-api-application-to-your-azure-active-directory-b2c-tenant"></a>Ajouter une application d'API web à votre locataire Azure Active Directory B2C

 Inscrivez les ressources des API web auprès de votre locataire afin qu’elles puissent accepter les demandes des applications clientes qui présentent un jeton d’accès et y répondre. Cet article explique comment inscrire une API web dans Azure Active Directory B2C (Azure AD B2C).

Pour inscrire une application dans votre locataire Azure AD B2C, vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d’applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, puis **Nouvelle inscription**.
1. Entrez un **Nom** pour l’application. Par exemple, *webapi1*.
1. Sous **URI de redirection**, sélectionnez **Web**, puis entrez un point de terminaison où Azure AD B2C doit retourner les jetons demandés par votre application. Dans une application de production, vous pouvez définir l’URI de redirection sur un point de terminaison de type `https://localhost:5000`. Lors du développement ou du test, vous pouvez le définir sur `https://jwt.ms`, une application Web Microsoft qui affiche le contenu décodé d’un jeton (le contenu du jeton ne quitte jamais votre navigateur). Vous pouvez ajouter des URI de redirection à vos applications inscrites à tout moment et les modifier.
1. Sélectionnez **Inscription**.
1. Enregistrez l’**ID d’application (client)** pour l’utiliser dans votre code d’API web.

Si vous avez une application qui implémente le flux d’octroi implicite, par exemple une [application à page unique (SPA) basée sur JavaScript](tutorial-register-spa.md), vous pouvez activer le flux en procédant comme suit :

1. Sous **Gérer**, sélectionnez **Authentification**.
1. Sous **Octroi implicite**, cochez les deux cases **Jetons d’accès** et **Jetons d’ID**.
1. Sélectionnez **Enregistrer**.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Veillez à bien utiliser l’annuaire qui contient votre locataire Azure AD B2C. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur et choisissez l’annuaire qui contient votre locataire.
3. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
4. Sélectionnez **Applications (héritées)** , puis **Ajouter**.
5. Entrez un nom pour l’application. Par exemple, *webapi1*.
6. Pour **inclure l’application web/l’API web** et **autoriser un flux implicite**, sélectionnez **Oui**.
7. Pour l’**URL de réponse**, entrez un point de terminaison où Azure AD B2C doit retourner les jetons demandés par votre application. Dans votre application de production, vous pouvez définir l’URL de réponse sur une valeur comme `https://localhost:44332`. À des fins de test, définissez cette URL sur `https://jwt.ms`.
8. Pour l’**L’URI de l’ID d’application**, entrez l’identificateur utilisé pour votre API web. L’identificateur complet URI, y compris le domaine, est généré pour vous. Par exemple : `https://contosotenant.onmicrosoft.com/api`.
9. Cliquez sur **Créer**.
10. Sur la page des propriétés, enregistrez l'ID d'application que vous utiliserez pour configurer l'application web.

* * *

## <a name="configure-scopes"></a>Configurer des étendues

Les étendues permettent de gérer l'accès aux ressources protégées. Elles sont utilisées par l’API web pour implémenter le contrôle d’accès basé sur les étendues. Par exemple, les utilisateurs de l’API web peuvent avoir un accès en lecture et en écriture, ou les utilisateurs de l’API web peuvent avoir l’accès en lecture uniquement. Dans ce didacticiel, vous allez utiliser des étendues pour définir des autorisations d’accès en lecture et en écriture pour l’API web.

[!INCLUDE [active-directory-b2c-scopes](../../includes/active-directory-b2c-scopes.md)]

## <a name="grant-permissions"></a>Accorder des autorisations

Pour appeler une API web protégée à partir d'une application, vous devez accorder à cette application les autorisations d'accès à l'API. Par exemple, dans [Tutoriel : Inscrire une application dans Azure Active Directory B2C](tutorial-register-applications.md), une application web est enregistrée dans Azure AD B2C nommée *webapp1*. Vous pouvez utiliser cette application pour appeler l'API web.

[!INCLUDE [active-directory-b2c-permissions-api](../../includes/active-directory-b2c-permissions-api.md)]

Votre application est inscrite pour appeler l'API web protégée. Un utilisateur s'authentifie auprès d'Azure AD B2C pour utiliser l'application. L'application obtient d'Azure AD B2C l'autorisation d'accéder à l'API web protégée.