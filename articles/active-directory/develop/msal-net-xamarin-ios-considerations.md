---
title: Considérations relatives à Xamarin iOS (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les points à prendre en considération lors de l’utilisation de Xamarin iOS avec la Bibliothèque d’authentification Microsoft pour .NET (MSAL.NET).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/09/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 70ab4a151fe73b59663fd8fa16170b2e507c2511
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91258057"
---
# <a name="considerations-for-using-xamarin-ios-with-msalnet"></a>Considérations relatives à l’utilisation de Xamarin iOS avec MSAL.NET

Lorsque vous utilisez la Bibliothèque d’authentification Microsoft pour .NET (MSAL.NET) sur Xamarin iOS, vous devez :

- Écraser et implémenter la fonction `OpenUrl` dans `AppDelegate`.
- Activer les groupes de trousseaux.
- Activer le partage du cache de jeton.
- Activer l’accès au trousseau.
- Comprendre les problèmes connus avec iOS 12, iOS 13 et l’authentification.

## <a name="implement-openurl"></a>Implémenter OpenUrl

Remplacez la méthode `OpenUrl` de la classe dérivée `FormsApplicationDelegate` et appelez `AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs`. Voici un exemple :

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

Effectuez également les tâches suivantes :

* Définissez un schéma d’URI de redirection.
* Exigez des autorisations pour permettre à votre application d’appeler une autre application.
* Prévoyez un formulaire spécifique pour l’URI de redirection.
* [Inscrivez un URI de redirection](quickstart-register-app.md#add-a-redirect-uri) dans le portail Azure.

### <a name="enable-keychain-access"></a>Activer l’accès au trousseau

Pour activer l’accès au trousseau, assurez-vous que votre application a un groupe d’accès au trousseau. Vous pouvez définir le groupe d’accès au trousseau lorsque vous créez votre application à l’aide de l’API `WithIosKeychainSecurityGroup()`.

Pour tirer parti du cache et de l’authentification unique, attribuez la même valeur au groupe d’accès au trousseau dans toutes vos applications.

Cet exemple de configuration utilise MSAL 4.x :

```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.adalcache")
     .Build();
```

Activez également l’accès au trousseau dans le fichier `Entitlements.plist`. Utilisez le groupe d’accès suivant ou votre propre groupe d’accès.

```xml
<dict>
  <key>keychain-access-groups</key>
  <array>
    <string>$(AppIdentifierPrefix)com.microsoft.adalcache</string>
  </array>
</dict>
```

Lorsque vous utilisez l’API `WithIosKeychainSecurityGroup()`, MSAL ajoute automatiquement votre groupe de sécurité à la fin de l’*ID d’équipe* de l’application (`AppIdentifierPrefix`). MSAL ajoute votre groupe de sécurité parce que, lorsque vous générez votre application dans Xcode, elle en fait de même. C’est la raison pour laquelle les droits d’utilisation dans le fichier `Entitlements.plist` doivent inclure `$(AppIdentifierPrefix)` devant le groupe d’accès au trousseau.

Pour plus d’informations, voir la [documentation sur les droits d’utilisation iOS](https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps).

### <a name="enable-token-cache-sharing-across-ios-applications"></a>Activer le partage du cache de jeton entre des applications iOS

Depuis MSAL 2.x, vous pouvez spécifier un groupe d’accès au trousseau pour conserver le cache de jeton dans plusieurs applications. Ce paramètre vous permet de partager le cache de jeton entre plusieurs applications qui ont le même groupe d’accès au trousseau. Vous pouvez partager le cache de jeton entre les applications [ADAL.NET](https://aka.ms/adal-net), les applications MSAL.NET Xamarin.iOS et les applications iOS natives qui ont été développées dans [ADAL.objc](https://github.com/AzureAD/azure-activedirectory-library-for-objc) ou [MSAL.objc](https://github.com/AzureAD/microsoft-authentication-library-for-objc).

Le partage du cache de jeton permet une authentification unique entre toutes les applications qui utilisent le même groupe d’accès au trousseau.

Pour activer ce partage du cache, utilisez la méthode `WithIosKeychainSecurityGroup()` pour définir le groupe d’accès au trousseau sur la même valeur dans toutes les applications qui partagent le cache. Le premier exemple de code de cet article montre comment utiliser la méthode.

Plus haut dans cet article, vous avez appris que MSAL ajoute `$(AppIdentifierPrefix)` chaque fois que vous utilisez l’API `WithIosKeychainSecurityGroup()`. MSAL ajoute cet élément parce que l’ID d’équipe `AppIdentifierPrefix` garantit que seules des applications créées par le même serveur de publication peuvent partager un accès au trousseau.

> [!NOTE]
> La propriété `KeychainSecurityGroup` est déconseillée. Utilisez plutôt la propriété `iOSKeychainSecurityGroup`. Le préfixe `TeamId` n’est pas requis lorsque vous utilisez `iOSKeychainSecurityGroup`.

### <a name="use-microsoft-authenticator"></a>Utiliser Microsoft Authenticator

Votre application peut utiliser Microsoft Authenticator en tant que répartiteur pour activer les fonctionnalités suivantes :

- **SSO** : Lorsque vous activez la SSO, vos utilisateurs n’ont pas besoin de se connecter à chaque application.
- **Identification de périphérique** : Utilisez l’identification de périphérique pour effectuer l’authentification en accédant au certificat de l’appareil. Ce certificat est créé sur l’appareil quand celui-ci est joint à l’espace de travail. Votre application sera prête si les administrateurs clients activent l’accès conditionnel lié aux appareils.
- **Vérification de l’identification de l’application** : quand une application appelle le répartiteur, elle transmet son URL de redirection. Le répartiteur vérifie l’URL de redirection.

Pour plus de détails sur l’activation d’un répartiteur, voir [Utiliser Microsoft Authenticator ou le portail d’entreprise Microsoft Intune sur des applications Xamarin iOS et Android](msal-net-use-brokers-with-xamarin-apps.md).

## <a name="known-issues-with-ios-12-and-authentication"></a>Problèmes connus avec iOS 12 et l’authentification

Microsoft a publié un [avis de sécurité](https://github.com/aspnet/AspNetCore/issues/4647) à propos d’une incompatibilité entre iOS 12 et certains types d’authentifications. L’incompatibilité interrompt les connexions aux réseaux sociaux, WSFed et OIDC. L’avis de sécurité vous aide à comprendre comment supprimer les restrictions de sécurité ASP.NET de vos applications pour les rendre compatibles avec iOS 12.

Lors du développement d’applications MSAL.NET sur Xamarin iOS, vous pouvez rencontrer une boucle infinie lorsque vous tentez de vous connecter à des sites web à partir d’iOS 12. Ce comportement est similaire au problème ADAL suivant sur GitHub : [Boucle infinie lors d’une tentative de connexion au site web à partir d’iOS 12 #1329](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/issues/1329).

Vous pouvez également constater une interruption de l’authentification ASP.NET Core OIDC avec iOS 12 Safari. Pour plus d’informations, voir ce [problème WebKit](https://bugs.webkit.org/show_bug.cgi?id=188165).

## <a name="known-issues-with-ios-13-and-authentication"></a>Problèmes connus avec iOS 13 et l’authentification

Si votre application nécessite un accès conditionnel ou une prise en charge de l’authentification par certificat, permettez à votre application de communiquer avec l’application de répartiteur Microsoft Authenticator. MSAL est ensuite chargé de gérer les demandes et les réponses échangées entre votre application et Microsoft Authenticator.

Sur iOS 13, Apple a apporté un changement cassant d’API en supprimant la capacité de l’application à lire l’application source lors de la réception d’une réponse d’une application externe via des modèles d’URL personnalisés.

Documentation d’Apple pour les états [UIApplicationOpenURLOptionsSourceApplicationKey](https://developer.apple.com/documentation/uikit/uiapplicationopenurloptionssourceapplicationkey?language=objc) :

> *Si la demande provient d’une autre application appartenant à votre équipe, UIKit définit la valeur de cette clé sur l’ID de cette application. Si l’identificateur d’équipe de l’application d’origine diffère de l’identificateur de l’application actuelle, la valeur de la clé est nil.*

Ce changement est cassant pour MSAL, car il s’appuie sur `UIApplication.SharedApplication.OpenUrl` pour vérifier la communication entre MSAL et l’application Microsoft Authenticator.

En outre, sur iOS 13, le développeur doit fournir un contrôleur de présentation lors de l’utilisation d’`ASWebAuthenticationSession`.

Votre application est affectée si vous la générez à l’aide de Xcode 11 et utilisez un répartiteur iOS ou `ASWebAuthenticationSession`.

Dans ce cas, utilisez [MSAL.NET 4.4.0+](https://www.nuget.org/packages/Microsoft.Identity.Client/) pour permettre une authentification réussie.

### <a name="additional-requirements"></a>Autres conditions requises

- Lorsque vous utilisez les bibliothèques MSAL les plus récentes, assurez-vous que **Microsoft Authenticator version 6.3.19+** est installé sur l’appareil.
- Lors de la mise à jour vers MSAL.NET 4.4.0+, mettez à jour `LSApplicationQueriesSchemes` dans le fichier *Info.plist*, et ajoutez `msauthv3` :

    ```xml
    <key>LSApplicationQueriesSchemes</key>
    <array>
         <string>msauthv2</string>
         <string>msauthv3</string>
    </array>
    ```

    L’ajout de `msauthv3` au fichier *Info.plist* est nécessaire pour détecter la présence de la dernière application Microsoft Authenticator sur l’appareil, qui prend en charge iOS 13.

## <a name="report-an-issue"></a>Signaler un problème

Si vous avez des questions ou si vous souhaitez signaler un problème que vous avez détecté dans MSAL.NET, ouvrez un problème dans le dépôt [AzureAD/microsoft-authentication-library-for-dotnet](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues) sur GitHub.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les propriétés pour Xamarin iOS, voir le paragraphe [Considérations spécifiques d’iOs](https://github.com/Azure-Samples/active-directory-xamarin-native-v2/tree/master/1-Basic#ios-specific-considerations) dans le fichier README.md de l’exemple suivant :

Exemple | Plateforme | Description
------ | -------- | -----------
[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin.iOS, Android, plateforme Windows universelle (UWP) | Application Xamarin.Forms simple qui montre comment utiliser MSAL pour authentifier les comptes personnels Microsoft et Azure AD via le point de terminaison Azure AD 2.0. L’application montre également comment utiliser le jeton obtenu pour accéder à Microsoft Graph.

<!--- https://github.com/Azure-Samples/active-directory-xamarin-native-v2/blob/master/ReadmeFiles/Topology.png -->
