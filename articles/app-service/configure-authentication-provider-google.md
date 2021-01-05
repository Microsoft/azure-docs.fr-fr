---
title: Configurer l’authentification Google
description: Découvrez comment configurer l’authentification Google pour en faire un fournisseur d’identité pour votre application App Service ou Azure Functions.
ms.assetid: 2b2f9abf-9120-4aac-ac5b-4a268d9b6e2b
ms.topic: article
ms.date: 09/02/2019
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: e8a9fbe6072f3628d755ad3ad5aa5a623fc3ab23
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80519940"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-google-login"></a>Configurer votre application App Service ou Azure Functions pour utiliser une connexion Google

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Cette rubrique montre comment configurer Azure App Service ou Azure Functions pour utiliser Google comme fournisseur d’authentification.

Pour effectuer la procédure décrite dans cette rubrique, vous devez disposer d'un compte Google avec une adresse électronique vérifiée. Pour créer un compte Google, accédez à [accounts.google.com](https://go.microsoft.com/fwlink/p/?LinkId=268302).

## <a name="register-your-application-with-google"></a><a name="register"> </a>Inscrire votre application auprès de Google

1. Pour créer un ID client et un secret client, suivez les instructions Google de la page [Google Sign-In for server-side apps](https://developers.google.com/identity/sign-in/web/server-side-flow) (Connexion Google pour les applications côté serveur). Il n’est pas nécessaire d’apporter des modifications au code. Utilisez simplement les informations suivantes :
    - Pour **Origines JavaScript autorisées**, utilisez `https://<app-name>.azurewebsites.net` avec le nom de votre application dans *\<app-name>* .
    - Pour **URI de redirection autorisée**, utilisez `https://<app-name>.azurewebsites.net/.auth/login/google/callback`.
1. Copiez les valeurs de l’ID d’application et du secret d’application.

    > [!IMPORTANT]
    > Le secret d’application est une information d’identification de sécurité importante. Ne partagez cette clé secrète avec personne et ne la distribuez pas dans une application cliente.

## <a name="add-google-information-to-your-application"></a><a name="secrets"> </a>Ajouter des informations Google à votre application

1. Dans le [Azure portal], accédez à votre application App Service.
1. Sélectionnez **Paramètres** > **Authentification/Autorisation**, puis vérifiez que **l’authentification App Service** est activée, sur **On**.
1. Sélectionnez **Google**, puis collez les valeurs d’ID d’application et de secret d’application que vous avez obtenues précédemment. Activez toutes les étendues nécessaires à votre application.
1. Sélectionnez **OK**.

   App Service fournit une authentification, mais ne restreint pas l’accès autorisé au contenu et aux API de votre site. Pour plus d’informations, consultez [Autoriser ou refuser des utilisateurs](app-service-authentication-how-to.md#authorize-or-deny-users).

1. (Facultatif) Pour restreindre l’accès à votre site aux seuls utilisateurs authentifiés par Google, définissez **Action à exécuter quand une demande n’est pas authentifiée** sur **Google**. Quand vous définissez cette fonctionnalité, votre application exige que toutes les demandes soient authentifiées. Elle redirige également toutes les demandes non authentifiées vers Google à des fins d’authentification.

    > [!CAUTION]
    > Cette manière de restreindre l’accès s’applique à tous les appels à votre application qui peuvent ne pas être souhaitables pour les applications qui ont une page d’accès publique disponible, comme dans de nombreuses applications monopages. Pour de telles applications, préférez **Autoriser les requêtes anonymes (aucune action)** . L’application démarre alors elle-même manuellement l’authentification. Pour plus d’informations, consultez [Flux d’authentification](overview-authentication-authorization.md#authentication-flow).

1. Sélectionnez **Enregistrer**.

Vous êtes maintenant prêt à utiliser Google pour l'authentification dans votre application.

## <a name="next-steps"></a><a name="related-content"> </a>Étapes suivantes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Anchors. -->

<!-- Images. -->

[0]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-redirect.png
[1]: ./media/app-service-mobile-how-to-configure-google-authentication/mobile-app-google-settings.png

<!-- URLs. -->

[Google apis]: https://go.microsoft.com/fwlink/p/?LinkId=268303

[Azure portal]: https://portal.azure.com/

