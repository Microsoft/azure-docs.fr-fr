---
title: Inscrire une application web qui connecte des utilisateurs – Plateforme d’identités Microsoft | Azure
description: Découvrir comment inscrire une application web qui connecte des utilisateurs
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 30d646ff7d4f97289ff238211418ac57cd8167a4
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107683"
---
# <a name="web-app-that-signs-in-users-app-registration"></a>Application web qui connecte les utilisateurs : Inscription d'application

Cette article explique les spécificités de l’inscription d’application pour une application web qui connecte des utilisateurs.

Pour inscrire votre application, vous pouvez utiliser les méthodes suivantes :

- [Démarrages rapides d’application web](#register-an-app-by-using-the-quickstarts). En plus d’être une excellente première expérience pour la création d’une application, les démarrages rapides du Portail Azure contiennent un bouton nommé **Apporter cette modification pour moi**. Vous pouvez utiliser ce bouton pour définir les propriétés dont vous avez besoin, même pour une application existante. Vous devrez adapter les valeurs de ces propriétés à votre propre cas. En particulier, l’URL de l’API web pour votre application sera probablement différente de celle proposée par défaut, ce qui influera également sur l’URI de déconnexion.
- Portail Azure pour [inscrire votre application manuellement](#register-an-app-by-using-the-azure-portal).
- PowerShell et outils en ligne de commande

## <a name="register-an-app-by-using-the-quickstarts"></a>Inscrire une application en utilisant les démarrages rapides

Vous pouvez utiliser ces liens pour démarrer la création de votre application web :

- [ASP.NET Core](https://aka.ms/aspnetcore2-1-aad-quickstart-v2)
- [ASP.NET](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetWebAppQuickstartPage/sourceType/docs)

## <a name="register-an-app-by-using-the-azure-portal"></a>Inscrire une application à l’aide du Portail Azure

> [!NOTE]
> Le portail à utiliser est différent selon que votre application s’exécute dans le cloud public Microsoft Azure ou dans un cloud national ou souverain. Pour plus d’informations, voir [Clouds nationaux](./authentication-national-cloud.md#app-registration-endpoints)


1. Connectez-vous au [portail Azure](https://portal.azure.com). 
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

1. Lorsque la page **Inscrire une application** s’affiche, saisissez les informations d’inscription de votre application :
   1. Entrez un **nom** pour votre application (par exemple, `AspNetCore-WebApp`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
   1. Sélectionnez les types de comptes pris en charge par votre application. (Consultez [Types de comptes pris en charge](./v2-supported-account-types.md).)
   1. Dans **URI de redirection**, ajoutez le type d’application et la destination d’URI qui acceptera les réponses de jeton retournées après une authentification réussie. Par exemple, entrez `https://localhost:44321`.
   1. Sélectionnez **Inscription**.
1. Sous **Gérer**, sélectionnez **Authentification**, puis ajoutez les informations suivantes :
   1. Dans la section **Web**, ajoutez `https://localhost:44321/signin-oidc` comme **URI de redirection**.
   1. Ajoutez `https://localhost:44321/signout-oidc` comme **URL de déconnexion**.
   1. Sous **Octroi implicite**, sélectionnez **Jetons d’ID**.
   1. Sélectionnez **Enregistrer**.
   
# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

1. Lorsque la page **Inscrire une application** s’affiche, saisissez les informations d’inscription de votre application :
   1. Entrez un **nom** pour votre application (par exemple, `MailApp-openidconnect-v2`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
   1. Sélectionnez les types de comptes pris en charge par votre application. (Consultez [Types de comptes pris en charge](./v2-supported-account-types.md).)
   1. Dans la section **URI de redirection (facultatif)** , sélectionnez **Web** dans la zone de liste modifiable et entrez les URI de redirection suivants : **https://localhost:44326/** .
   1. Sélectionnez **Inscrire** pour créer l’application.
1. Sous **Gérer**, sélectionnez **Authentification**.
1. Dans la section **Octroi implicite**, sélectionnez **Jetons d’ID**. Ce démarrage rapide requiert l’activation du [flux d’octroi implicite](v2-oauth2-implicit-grant-flow.md) pour la connexion de l’utilisateur.
1. Sélectionnez **Enregistrer**.

# <a name="java"></a>[Java](#tab/java)

1. Lorsque la page **Inscrire une application** s’affiche, saisissez les informations d’inscription de votre application : 
    1. Entrez un **nom** pour votre application (par exemple, `java-webapp`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement. 
    1. Sélectionnez **Comptes dans un annuaire organisationnel et comptes personnels Microsoft (par exemple, Skype, Xbox, Outlook.com)** .
    1. Sélectionnez **Inscrire** pour inscrire l’application.
1. Sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme**.
1. Sélectionnez **Web**.
1. Pour **URI de redirection**, entrez le même hôte et le même numéro de port, suivis de `/msal4jsample/secure/aad` pour la page de connexion. 
1. Sélectionnez **Configurer**.
1. Dans la section **Web**, utilisez l’hôte et le numéro de port, suivis de **/msal4jsample/graph/me** comme **URI de redirection** pour la page des informations utilisateur.
Par défaut, l’exemple utilise :
   - **http://localhost:8080/msal4jsample/secure/aad**
   - **http://localhost:8080/msal4jsample/graph/me**

1. Sélectionnez **Enregistrer**.
1. Sous **Gérer**, sélectionnez **Certificats et secrets**.
1. Dans la section **Secrets client**, sélectionnez **Nouveau secret client**, puis :

   1. Entrez une description de clé.
   1. Sélectionnez la durée de clé **Dans 1 an**.
   1. Sélectionnez **Ajouter**.
   1. Lorsque la valeur de clé apparaît, copiez-la pour l’utiliser ultérieurement. Cette valeur ne sera pas réaffichée ou ne pourra pas être récupérée par d’autres moyens.

# <a name="python"></a>[Python](#tab/python)

1. Lorsque la page **Inscrire une application** s’affiche, saisissez les informations d’inscription de votre application :
   1. Entrez un **nom** pour votre application (par exemple, `python-webapp`). Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
   1. Affectez à **Types de comptes pris en charge** la valeur **Comptes dans un annuaire organisationnel et comptes personnels Microsoft (par exemple, Skype, Xbox, Outlook.com)** .
   1. Dans la section **URI de redirection (facultatif)** , sélectionnez **Web** dans la zone de liste modifiable et entrez les URI de redirection suivants : **http://localhost:5000/getAToken** .
   1. Sélectionnez **Inscrire** pour créer l’application.
1. Sur la page **Vue d'ensemble** de l'application, recherchez la valeur de l'**ID d'application (client)** et notez-la. Vous en aurez besoin pour configurer le fichier de configuration Visual Studio pour ce projet.
1. Sous **Gérer**, sélectionnez **Certificats et secrets**.
1. Dans la section **Secrets client**, sélectionnez **Nouveau secret client**, puis :
   1. Entrez une description de clé.
   1. Sélectionnez la durée de clé **Dans 1 an**.
   1. Sélectionnez **Ajouter**.
   1. Lorsque la valeur de clé apparaît, copiez-la. Vous en aurez besoin ultérieurement.
---

## <a name="register-an-app-by-using-powershell"></a>Inscrire une application à l’aide de PowerShell

> [!NOTE]
> Actuellement, Azure AD PowerShell crée des applications avec les types de compte pris en charge suivants uniquement :
>
> - MyOrg (comptes dans cet annuaire organisationnel uniquement)
> - AnyOrg (comptes dans un annuaire organisationnel)
>
> Vous pouvez créer une application qui connecte les utilisateurs avec leurs comptes Microsoft personnels (par exemple, Skype, Xbox ou Outlook.com). Tout d’abord, créez application mutualisée. Les types de comptes pris en charge sont des comptes dans un annuaire organisationnel. Ensuite, définissez la propriété [`accessTokenAcceptedVersion`](./reference-app-manifest.md#accesstokenacceptedversion-attribute) sur **2** et la propriété [`signInAudience`](./reference-app-manifest.md#signinaudience-attribute) sur `AzureADandPersonalMicrosoftAccount` dans le [manifeste de l’application](./reference-app-manifest.md) dans le portail Azure. Pour plus d’informations, consultez l’[étape 1.3](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-3-AnyOrgOrPersonal#step-1-register-the-sample-with-your-azure-ad-tenant) dans le didacticiel ASP.NET Core. Vous pouvez généraliser cette étape aux applications web dans n’importe quel langage.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Configuration du code de l’application](scenario-web-app-sign-user-app-configuration.md).
