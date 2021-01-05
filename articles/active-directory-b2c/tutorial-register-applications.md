---
title: 'Tutoriel : Inscrire une application'
titleSuffix: Azure AD B2C
description: Ce tutoriel explique comment inscrire une application web dans Azure Active Directory B2C à l’aide du portail Azure.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 04/10/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 84a3ef7b41b17e85c594213246211d45911ac56a
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953047"
---
# <a name="tutorial-register-a-web-application-in-azure-active-directory-b2c"></a>Tutoriel : Inscrire une application web dans Azure Active Directory B2C

Pour que vos [applications](application-types.md) puissent interagir avec Azure Active Directory B2C (Azure AD B2C), elles doivent être inscrites dans un locataire que vous gérez. Ce tutoriel vous montre comment inscrire une application web à l’aide du portail Azure. 

Une « application web » fait référence à une application web classique exécutant la majeure partie de la logique d’application sur le serveur. Ces applications peuvent être créées au moyen de frameworks, tels que ASP.NET Core, Maven (Java), Flask (Python) et Express (Node.js).

> [!IMPORTANT]
> Si vous utilisez une application monopage (« SPA ») à la place (par exemple, en utilisant Angular, Vue ou React), consultez le [Guide pratique pour inscrire une application monopage](tutorial-register-spa.md).
> 
> Si vous utilisez une application native (par exemple, iOS, Android, mobile et de bureau), découvrez [comment inscrire une application cliente native](add-native-application.md).

## <a name="prerequisites"></a>Prérequis
Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Si vous n’avez pas encore créé votre propre [locataire Azure AD B2C](tutorial-create-tenant.md), créez-en un maintenant. Vous pouvez utiliser un locataire Azure AD B2C existant.

## <a name="register-a-web-application"></a>Inscrire une application web

Pour inscrire une application web dans votre locataire Azure AD B2C, vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d’applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le portail Azure, recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d’applications**, puis **Nouvelle inscription**.
1. Entrez un **Nom** pour l’application. Par exemple, *webapp1*.
1. Sous **Types de comptes pris en charge**, sélectionnez **Comptes dans un fournisseur d’identité ou annuaire organisationnel (pour authentifier les utilisateurs avec des flux d’utilisateurs)** .
1. Sous **URI de redirection**, sélectionnez **Web**, puis entrez `https://jwt.ms` dans la zone de texte de l’URL.

    L’URI de redirection est le point de terminaison auquel l’utilisateur est envoyé par le serveur d’autorisation (dans ce cas, Azure AD B2C) après avoir terminé son interaction avec l’utilisateur, et auquel un jeton d’accès ou un code d’autorisation est envoyé en cas d’autorisation réussie. Dans une application de production, il s’agit généralement d’un point de terminaison accessible publiquement dans lequel votre application s’exécute, comme `https://contoso.com/auth-response`. À des fins de test comme dans ce didacticiel, vous pouvez le définir sur `https://jwt.ms`, une application web Microsoft qui affiche le contenu décodé d’un jeton (le contenu du jeton ne quitte jamais votre navigateur). Pendant le développement d’une application, vous pouvez ajouter le point de terminaison sur lequel votre application écoute localement, comme `https://localhost:5000`. Vous pouvez ajouter des URI de redirection à vos applications inscrites à tout moment et les modifier.

    Les restrictions suivantes s’appliquent aux URI de redirection :

    * L’URL de réponse doit commencer par le schéma `https`.
    * L’URL de réponse respecte la casse. Sa casse doit correspondre à celle du chemin d’URL de votre application en cours d’exécution. Par exemple, si votre application comprend `.../abc/response-oidc` dans son chemin, ne spécifiez pas `.../ABC/response-oidc` dans l’URL de réponse. Étant donné que le navigateur web considère que les chemins respectent la casse, les cookies associés à `.../abc/response-oidc` peuvent être exclus s’ils sont redirigés vers l’URL `.../ABC/response-oidc` qui ne correspond pas à la casse.

1. Sous **Autorisations**, activez la case à cocher *Accorder le consentement administrateur aux autorisations openid et offline_access*.
1. Sélectionnez **Inscription**.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le portail Azure, recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Applications (héritées)** , puis **Ajouter**.
1. Entrez un nom pour l’application. Par exemple, *webapp1*.
1. Pour **Inclure une application/API web**, sélectionnez **Oui**.
1. Pour l’**URL de réponse**, entrez un point de terminaison où Azure AD B2C doit retourner les jetons demandés par votre application. Par exemple, vous pouvez le définir pour qu’il écoute localement sur `http://localhost:5000`. Vous pouvez ajouter des URI de redirection à vos applications inscrites à tout moment et les modifier.

    Les restrictions suivantes s’appliquent aux URI de redirection :

    * L’URL de réponse doit commencer par le schéma `https`, sauf en cas d’utilisation de `localhost`.
    * L’URL de réponse respecte la casse. Sa casse doit correspondre à celle du chemin d’URL de votre application en cours d’exécution. Par exemple, si votre application comprend `.../abc/response-oidc` dans son chemin, ne spécifiez pas `.../ABC/response-oidc` dans l’URL de réponse. Étant donné que le navigateur web considère que les chemins respectent la casse, les cookies associés à `.../abc/response-oidc` peuvent être exclus s’ils sont redirigés vers l’URL `.../ABC/response-oidc` qui ne correspond pas à la casse.

1. Sélectionnez **Créer** pour terminer l’inscription de l’application.

* * *

## <a name="create-a-client-secret"></a>Créer une clé secrète client

Pour une application web, vous devez créer un secret d’application. Ce secret sera utilisé par votre application afin d’échanger un code d’autorisation pour un jeton d’accès.

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Dans la page **Azure AD B2C – Inscriptions d’applications**, sélectionnez l’application que vous avez créée, par exemple *webapp1*.
1. Dans le menu de gauche, sous **Gérer**, sélectionnez **Certificats et secrets**.
1. Sélectionnez **Nouveau secret client**.
1. Entrez une description pour la clé secrète client dans la zone **Description**. Par exemple, *clientsecret1*.
1. Sous **Expire**, sélectionnez une durée pendant laquelle le secret est valide, puis sélectionnez **Ajouter**.
1. Enregistrez la **Valeur** du secret. Vous utiliserez cette valeur comme secret d’application dans le code de votre application.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Dans la page **Azure AD B2C – Applications**, sélectionnez l’application que vous avez créée, par exemple *webapp1*.
1. Sélectionnez **Clés**, puis **Générer la clé**.
1. Sélectionnez **Enregistrer** pour afficher la clé. Prenez note de la valeur **Clé d’application** . Vous utiliserez cette valeur comme secret d’application dans le code de votre application.

* * *

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Inscrire une application web
> * Créer une clé secrète client

Ensuite, découvrez comment créer des flux utilisateur pour permettre à vos utilisateurs de s’inscrire, se connecter et gérer leur profil.

> [!div class="nextstepaction"]
> [Créer des flux utilisateur dans Azure Active Directory B2C >](tutorial-create-user-flows.md)