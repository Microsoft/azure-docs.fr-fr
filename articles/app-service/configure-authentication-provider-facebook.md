---
title: Configurer l’authentification Facebook
description: Découvrez comment configurer l’authentification Facebook pour en faire un fournisseur d’identité pour votre application App Service ou Azure Functions.
ms.assetid: b6b4f062-fcb4-47b3-b75a-ec4cb51a62fd
ms.topic: article
ms.date: 06/06/2019
ms.custom:
- seodec18
- fasttrack-edit
ms.openlocfilehash: b6aad323c0d6fa8f59c9fad203640c477b162503
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80519954"
---
# <a name="configure-your-app-service-or-azure-functions-app-to-use-facebook-login"></a>Configurer votre application App Service ou Azure Functions pour utiliser une connexion Facebook

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

Cet article vous montre comment configurer Azure App Service ou Azure Functions pour utiliser Facebook comme fournisseur d’authentification.

Pour effectuer la procédure de cet article, vous avez besoin d’un compte Facebook avec une adresse e-mail vérifiée et un numéro de téléphone mobile. Pour créer un compte Facebook, allez sur [facebook.com].

## <a name="register-your-application-with-facebook"></a><a name="register"> </a>Inscrire votre application auprès de Facebook

1. Accédez au site web [Développeurs Facebook] et connectez-vous à l’aide des informations d’identification de votre compte Facebook.

   Si vous n’avez pas de compte Facebook pour les développeurs, sélectionnez **Prise en main** et suivez les étapes d’inscription.
1. Sélectionnez **Mes applications** > **Ajouter une nouvelle application**.
1. Dans le champ **Nom d’affichage** :
   1. Tapez un nom unique pour votre application.
   1. Indiquez votre **E-mail du contact**.
   1. Sélectionnez **Créer un ID d'application**.
   1. Effectuez la vérification de sécurité.

   Le tableau de bord du développeur pour votre nouvelle application Facebook s’ouvre.
1. Sélectionnez **Tableau de bord** > **Connexion Facebook** > **Configurer** > **Web**.
1. Dans le volet de navigation de gauche, sous **Connexion Facebook**, sélectionnez **Paramètres**.
1. Dans le champ **URI de redirection OAuth Valides**, entrez `https://<app-name>.azurewebsites.net/.auth/login/facebook/callback`. N’oubliez pas de remplacer `<app-name>` par le nom de votre application Azure App Service.
1. Sélectionnez **Enregistrer les modifications**.
1. Dans le volet gauche, sélectionnez **Paramètres** > **De base**. 
1. Dans le champ **Secret d’application**, sélectionnez **Afficher**. Copiez les valeurs **ID d’application** et **Secret d’application**. Vous les utiliserez plus tard pour configurer votre application App Service dans Azure.

   > [!IMPORTANT]
   > La clé secrète de l'application est une information d'identification de sécurité importante. Ne partagez cette clé secrète avec personne et ne la distribuez pas dans une application cliente.
   >

1. Le compte Facebook que vous avez utilisé pour inscrire l'application est un administrateur de l'application. À ce stade, seuls les administrateurs peuvent se connecter à cette application.

   Pour authentifier d’autres comptes Facebook, sélectionnez **Révision de l’application** et activez **Rendre \<your-app-name> public** pour activer l’accès public général à l’application à l’aide de l’authentification Facebook.

## <a name="add-facebook-information-to-your-application"></a><a name="secrets"> </a>Ajouter des informations Facebook à votre application

1. Connectez-vous au [Azure portal]et accédez à votre application App Service.
1. Sélectionnez **Paramètres** > **Authentification/Autorisation**, puis vérifiez que **l’authentification App Service** est activée, sur **On**.
1. Sélectionnez **Facebook**, puis collez-y les valeurs d’ID d’application et de secret d’application que vous avez obtenues précédemment. Activez toutes les étendues nécessaires à votre application.
1. Sélectionnez **OK**.

   ![Capture d’écran des paramètres Facebook d’application mobile][0]

    Par défaut, App Service fournit une authentification, mais ne restreint pas l’accès autorisé au contenu et aux API de votre site. Vous avez besoin d’autoriser les utilisateurs dans votre code d’application.
1. (Facultatif) Pour restreindre l’accès aux seuls utilisateurs authentifiés par Facebook, définissez **Action à exécuter quand une demande n’est pas authentifiée** sur **Facebook**. Quand vous définissez cette fonctionnalité, votre application exige que toutes les demandes soient authentifiées. Elle redirige également toutes les demandes non authentifiées vers Facebook à des fins d’authentification.

   > [!CAUTION]
   > Cette manière de restreindre l’accès s’applique à tous les appels à votre application qui peuvent ne pas être souhaitables pour les applications qui ont une page d’accès publique disponible, comme dans de nombreuses applications monopages. Pour de telles applications, préférez **Autoriser les requêtes anonymes (aucune action)** . L’application démarre alors elle-même manuellement l’authentification. Pour plus d’informations, consultez [Flux d’authentification](overview-authentication-authorization.md#authentication-flow).

1. Sélectionnez **Enregistrer**.

Vous êtes maintenant prêt à utiliser Facebook pour l’authentification dans votre application.

## <a name="next-steps"></a><a name="related-content"> </a>Étapes suivantes

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- Images. -->
[0]: ./media/app-service-mobile-how-to-configure-facebook-authentication/mobile-app-facebook-settings.png

<!-- URLs. -->
[Développeurs Facebook]: https://go.microsoft.com/fwlink/p/?LinkId=268286
[facebook.com]: https://go.microsoft.com/fwlink/p/?LinkId=268285
[Get started with authentication]: /en-us/develop/mobile/tutorials/get-started-with-users-dotnet/
[Azure portal]: https://portal.azure.com/
