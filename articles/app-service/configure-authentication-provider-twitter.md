---
title: Configurer l’authentification Twitter
description: Découvrez comment configurer l’authentification Twitter pour en faire un fournisseur d’identité pour votre application App Service ou Azure Functions.
ms.assetid: c6dc91d7-30f6-448c-9f2d-8e91104cde73
ms.topic: article
ms.date: 02/28/2020
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: 11c913b12b4dcb7d2a5ffa532064b347b82904ef
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80519907"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-twitter-login"></a>Configurer votre application App Service ou Azure Functions pour utiliser une connexion Twitter

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Cet article vous montre comment configurer Azure App Service ou Azure Functions pour utiliser Twitter comme fournisseur d’authentification.

Pour effectuer la procédure décrite dans cet article, vous avez besoin d’un compte Twitter avec une adresse e-mail vérifiée et un numéro de téléphone. Pour créer un compte Twitter, consultez la page [twitter.com].

## <a name="register-your-application-with-twitter"></a><a name="register"></a>Inscription de votre application avec Twitter

1. Connectez-vous au [Azure portal] et accédez à votre application. Copiez votre **URL**. Vous allez l’utiliser pour configurer votre application Twitter.
1. Accédez au site web [Twitter Developers], connectez-vous avec vos informations d’identification Twitter, puis sélectionnez **Create an app** (Créer une application).
1. Entrez le **App name** (Nom de l’application) et l’**Application description** (Description de l’application) de votre nouvelle application. Collez l’**URL** de votre application dans le champ **Website URL** (URL du site). Dans la section **Callback URLs** (URL de rappel), entrez l’URL HTTPS de votre application App Service et ajoutez le chemin d’accès `/.auth/login/twitter/callback`. Par exemple : `https://contoso.azurewebsites.net/.auth/login/twitter/callback`.
1. En bas de la page, saisissez au moins 100 caractères dans **Tell us how this app will be used** (Indiquez-nous comment cette application sera utilisée), puis sélectionnez **Create** (Créer). Cliquez à nouveau sur **Create** dans la fenêtre contextuelle. Les détails de l’application s’affichent.
1. Sélectionnez l’onglet **Keys and Access Tokens** .

   Prenez note de ces valeurs :
   - Clé API
   - Clé secrète API

   > [!NOTE]
   > La clé secrète API est un élément important des informations d’identification en matière de sécurité. Ne partagez pas cette clé secrète avec quiconque et ne la distribuez pas avec votre application.

## <a name="add-twitter-information-to-your-application"></a><a name="secrets"></a>Ajout des informations Twitter à votre application

1. Accédez à votre application dans le [Azure portal].
1. Sélectionnez **Paramètres** > **Authentification/Autorisation**, puis vérifiez que **l’authentification App Service** est activée, sur **On**.
1. Sélectionnez **Twitter**.
1. Collez les valeurs `API key` et `API secret key` que vous avez obtenues précédemment.
1. Sélectionnez **OK**.

   ![Capture d’écran des paramètres Twitter de l’application mobile][1]

   Par défaut, App Service fournit une authentification, mais ne restreint pas l’accès autorisé au contenu et aux API de votre site. Vous devez autoriser les utilisateurs dans votre code d'application.

1. (Facultatif) Pour restreindre l’accès à votre site aux seuls utilisateurs authentifiés par Twitter, définissez **Action à exécuter quand une demande n’est pas authentifiée** sur **Twitter**. Quand vous définissez cette fonctionnalité, votre application exige que toutes les demandes soient authentifiées. Elle redirige également toutes les demandes non authentifiées vers Twitter à des fins d’authentification.

   > [!CAUTION]
   > Cette manière de restreindre l’accès s’applique à tous les appels à votre application qui peuvent ne pas être souhaitables pour les applications qui ont une page d’accès publique disponible, comme dans de nombreuses applications monopages. Pour de telles applications, préférez **Autoriser les requêtes anonymes (aucune action)** . L’application démarre alors elle-même manuellement l’authentification. Pour plus d’informations, consultez [Flux d’authentification](overview-authentication-authorization.md#authentication-flow).

1. Sélectionnez **Enregistrer**.

Vous êtes maintenant prêt à utiliser Twitter pour l'authentification dans votre application.

## <a name="next-steps"></a><a name="related-content"> </a>Étapes suivantes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-twitter-authentication/app-service-twitter-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-twitter-authentication/mobile-app-twitter-settings.png

<!-- URLs. -->

[Twitter Developers]: https://go.microsoft.com/fwlink/p/?LinkId=268300
[twitter.com]: https://go.microsoft.com/fwlink/p/?LinkID=268287
[Azure portal]: https://portal.azure.com/
[xamarin]: ../app-services-mobile-app-xamarin-ios-get-started-users.md
