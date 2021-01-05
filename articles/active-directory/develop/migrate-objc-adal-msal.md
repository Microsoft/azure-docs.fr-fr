---
title: Guide de migration ADAL vers MSAL (MSAL iOS/macOS) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les différences entre MSAL pour iOS/macOS et la bibliothèque Azure AD Authentication pour ObjectiveC (ADAL.ObjC), ainsi que la façon d’effectuer une migration vers MSAL pour iOS/macOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev
ms.openlocfilehash: fb66d8a4bf97a6f8a10534c9c4459123ad6a2654
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107917"
---
# <a name="migrate-applications-to-msal-for-ios-and-macos"></a>Migrer des applications vers MSAL pour iOS et macOS

Azure Active Directory Authentication Library ([ADAL Objective-C](https://github.com/AzureAD/azure-activedirectory-library-for-objc)) a été créée pour fonctionner avec les comptes Azure Active Directory par le biais du point de terminaison v1.0.

Microsoft Authentication Library pour iOS et macOS (MSAL) est conçue pour fonctionner avec toutes les identités Microsoft, telles que les comptes Azure Active Directory (Azure AD), les comptes Microsoft personnels et les comptes Azure AD B2C par le biais de la plateforme d’identités Microsoft (point de terminaison Azure AD v2.0).

La plateforme d’identités Microsoft présente quelques différences clés avec Azure Active Directory v1.0. Cet article met en évidence ces différences et fournit des conseils pour migrer une application d’ADAL vers MSAL.

## <a name="adal-and-msal-app-capability-differences"></a>Différences entre les fonctionnalités des applications ADAL et MSAL

### <a name="who-can-sign-in"></a>Qui peut se connecter

* ADAL prend uniquement en charge les comptes professionnels et scolaires, également appelés comptes Azure AD.
* MSAL prend en charge les comptes Microsoft personnels (comptes MSA) tels que Hotmail.com, Outlook.com et Live.com.
* MSAL prend en charge les comptes professionnels et scolaires et les comptes Azure AD B2C.

### <a name="standards-compliance"></a>Conformité aux normes

* Le point de terminaison de plateforme d’identités Microsoft suit les normes OAuth 2.0 et OpenId Connect.

### <a name="incremental-and-dynamic-consent"></a>Consentement incrémentiel et dynamique

* Le point de terminaison Azure Active Directory v1.0 exige que toutes les autorisations soient déclarées à l’avance pendant l’inscription de l’application. Cela signifie que ces autorisations sont statiques.
* La plateforme d’identités Microsoft vous permet de demander des autorisations de manière dynamique. Les applications peuvent demander des autorisations uniquement en fonction des besoins, et en demander plus à mesure qu’elles en ont besoin.

Pour plus d’informations sur les différences entre Azure Active Directory v1.0 et la plateforme d’identités Microsoft, consultez [Pourquoi mettre à jour à la plateforme d’identités Microsoft (v2.0) ?](../azuread-dev/azure-ad-endpoint-comparison.md).

## <a name="adal-and-msal-library-differences"></a>Différences entre les bibliothèques ADAL et MSAL

L’API publique MSAL reflète quelques différences clés entre Azure AD v1.0 et la plateforme d’identités Microsoft.

### <a name="msalpublicclientapplication-instead-of-adauthenticationcontext"></a>MSALPublicClientApplication au lieu de ADAuthenticationContext

`ADAuthenticationContext` est le premier objet créé par une application ADAL. Il représente une instanciation d’ADAL. Les applications créent une nouvelle instance de `ADAuthenticationContext` pour chaque combinaison cloud/locataire (autorité) Azure Active Directory. Le même `ADAuthenticationContext` peut être utilisé pour obtenir des jetons pour plusieurs applications clientes publiques.

Dans MSAL, l’interaction principale s’effectue par le biais d’un objet `MSALPublicClientApplication`, qui est modélisé d’après le [client public OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-2.1). Une instance de `MSALPublicClientApplication` peut être utilisée pour interagir avec plusieurs clouds et locataires AAD sans qu’il soit nécessaire de créer une instance pour chaque autorité. Pour la plupart des applications, une seule instance de `MSALPublicClientApplication` suffit.

### <a name="scopes-instead-of-resources"></a>Étendues au lieu de ressources

Dans ADAL, une application devait fournir un identificateur de *ressource* tel que `https://graph.microsoft.com` pour acquérir des jetons à partir du point de terminaison Azure Active Directory v1.0. Une ressource peut définir plusieurs étendues, ou oAuth2Permissions dans le manifeste de l’application, qu’elle comprend. Cela permettait aux applications clientes de demander des jetons à partir de cette ressource pour un certain ensemble d’étendues prédéfini lors de l’inscription de l’application.

Dans MSAL, au lieu d’un identificateur de ressource unique, les applications fournissent un ensemble d’étendues par demande. Une étendue est un identificateur de ressource suivi d’un nom d’autorisation sous la forme ressource/autorisation. Par exemple : `https://graph.microsoft.com/user.read`

Il existe deux façons de fournir des étendues dans MSAL :

* Fournissez une liste de toutes les autorisations dont votre application a besoin. Par exemple : 

    `@[@"https://graph.microsoft.com/directory.read", @"https://graph.microsoft.com/directory.write"]`

    Dans ce cas, l’application demande les autorisations `directory.read` et `directory.write`. L’utilisateur sera invité à donner son consentement pour ces autorisations s’il ne l’a pas encore fait pour cette application. L’application peut également recevoir des autorisations supplémentaires pour lesquelles l’utilisateur lui a déjà accordé son consentement. L’utilisateur sera invité à donner son consentement uniquement pour les nouvelles autorisations ou celles qui n’ont pas été accordées.

* L’étendue `/.default`.

Il s’agit de l’étendue intégrée pour chaque application. Elle fait référence à la liste statique des autorisations configurées lors de l’inscription de l’application. Son comportement est semblable à celui de `resource`. Cela peut être utile lors de la migration, pour s’assurer qu’un ensemble similaire d’étendues et d’expérience utilisateur est conservé.

Pour utiliser l’étendue `/.default`, ajoutez `/.default` à l’identificateur de ressource. Par exemple : `https://graph.microsoft.com/.default`. Si votre ressource se termine par une barre oblique (`/`), vous devez quand même ajouter `/.default`, y compris la barre oblique de début. Vous obtiendrez ainsi une étendue avec une double barre oblique (`//`).

Pour en savoir plus sur l’utilisation de l’étendue « /.default », consultez [cet article](./v2-permissions-and-consent.md#the-default-scope).

### <a name="supporting-different-webview-types--browsers"></a>Prise en charge de différents types WebView & navigateurs

ADAL prend uniquement en charge UIWebView/WKWebView pour iOS et WebView pour macOS. MSAL pour iOS prend en charge d’autres options pour afficher le contenu web lors de la demande d’un code d’autorisation et ne prend plus en charge `UIWebView`, ce qui peut améliorer l’expérience utilisateur et la sécurité.

Par défaut, MSAL sur iOS utilise [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession?language=objc), qui est le composant web recommandé par Apple pour l’authentification sur les appareils iOS 12+. Il procure les avantages offerts par l’authentification unique par le biais du partage des cookies entre les applications et le navigateur Safari.

Vous pouvez choisir d’utiliser un composant web différent en fonction des spécifications de l’application et de l’expérience de l’utilisateur final souhaitée. Pour plus d’options, consultez [Types d’affichage web pris en charge](customize-webviews.md).

Lors de la migration d’ADAL vers MSAL, `WKWebView` fournit l’expérience utilisateur qui ressemble le plus à ADAL sur iOS et macOS. Nous vous encourageons à migrer vers `ASWebAuthenticationSession` sur iOS dans la mesure du possible. Pour macOS, nous vous encourageons à utiliser `WKWebView`.

### <a name="account-management-api-differences"></a>Différences entre les API de gestion des comptes

Quand vous appelez la méthode ADAL `acquireToken()` ou `acquireTokenSilent()`, vous recevez un objet `ADUserInformation` contenant une liste de revendications de l’`id_token` qui représente le compte en cours d’authentification. De plus, `ADUserInformation` retourne un `userId` basé sur la revendication `upn`. Après l’acquisition de jeton interactive initiale, ADAL s’attend à ce que le développeur fournisse `userId` dans tous les appels silencieux.

La bibliothèque ADAL ne fournit pas d’API pour récupérer les identités d’utilisateur connues. Elle se repose sur l’application pour enregistrer et gérer ces comptes.

MSAL fournit un ensemble d’API permettant de lister tous les comptes connus de MSAL sans avoir à acquérir un jeton.

Comme ADAL, MSAL retourne des informations de compte qui contiennent une liste de revendications à partir de l’`id_token`. Elle fait partie de l’objet `MSALAccount` à l’intérieur de l’objet `MSALResult`.

MSAL fournit un ensemble d’API pour supprimer des comptes, ce qui rend les comptes supprimés inaccessibles à l’application. Une fois le compte supprimé, les appels d’acquisition de jeton ultérieurs invitent l’utilisateur à effectuer une acquisition de jeton interactive. La suppression de compte s’applique uniquement à l’application cliente qui l’a démarrée, et ne supprime pas le compte des autres applications en cours d’exécution sur l’appareil ou à partir du navigateur système. Cela permet de s’assurer que l’utilisateur continue à avoir une expérience d’authentification unique sur l’appareil même après s’être déconnecté d’une application.

MSAL retourne également un identificateur de compte qui peut être utilisé pour demander un jeton en mode silencieux ultérieurement. Toutefois, l’identificateur de compte (accessible par le biais de la propriété `identifier` dans l’objet `MSALAccount`) n’est pas affichable ; vous ne pouvez pas anticiper son format et ne devez pas essayer de l’interpréter ou de l’analyser.

### <a name="migrating-the-account-cache"></a>Migration du cache de compte

Lors de la migration à partir d’ADAL, les applications stockent normalement le `userId` ADAL, qui n’a pas l’`identifier` requis par MSAL. En guise d’étape de migration ponctuelle, une application peut interroger un compte MSAL à l’aide du userId ADAL avec l’API suivante :

`- (nullable MSALAccount *)accountForUsername:(nonnull NSString *)username error:(NSError * _Nullable __autoreleasing * _Nullable)error;`

Cette API lit le cache de MSAL et d’ADAL pour trouver le compte en fonction du userId ADAL (UPN).

Si le compte est trouvé, le développeur doit l’utiliser pour effectuer une acquisition de jeton en mode silencieux. La première acquisition de jeton en mode silencieux mettra à niveau le compte, et le développeur obtiendra un identificateur de compte compatible MSAL dans le résultat MSAL (`identifier`). Après cela, seul `identifier` doit être utilisé pour les recherches de compte à l’aide de l’API suivante :

`- (nullable MSALAccount *)accountForIdentifier:(nonnull NSString *)identifier error:(NSError * _Nullable __autoreleasing * _Nullable)error;`

Bien qu’il soit possible de continuer à utiliser le `userId` d’ADAL pour toutes les opérations dans MSAL, étant donné que `userId` est basé sur UPN, il est soumis à plusieurs limitations qui entraînent une expérience utilisateur de piètre qualité. Par exemple, si l’UPN change, l’utilisateur doit se reconnecter. Nous recommandons que toutes les applications utilisent le compte non affichable `identifier` pour toutes les opérations.

Pour en savoir plus sur la migration de l’état de cache, consultez [cet article](sso-between-adal-msal-apps-macos-ios.md).

### <a name="token-acquisition-changes"></a>Changements relatifs à l’acquisition de jeton

MSAL introduit certains changements pour les appels d’acquisition de jeton :

* Comme ADAL, `acquireTokenSilent` génère toujours une demande silencieuse.
* Contrairement à ADAL, `acquireToken` produit toujours une interface utilisateur actionnable par l’utilisateur par le biais de l’affichage web ou de l’application Microsoft Authenticator. En fonction de l’état de l’authentification unique dans WebView/Microsoft Authenticator, l’utilisateur peut être invité à entrer ses informations d’identification.
* Dans ADAL, `acquireToken` avec `AD_PROMPT_AUTO` tente d’abord l’acquisition du jeton en mode silencieux, et affiche uniquement l’interface utilisateur en cas d’échec de la requête silencieuse. Dans MSAL, cette logique peut être obtenue en appelant d’abord `acquireTokenSilent` et en appelant `acquireToken` uniquement si l’acquisition en mode silencieux échoue. Cela permet aux développeurs de personnaliser l’expérience utilisateur avant de démarrer l’acquisition interactive de jetons.

### <a name="error-handling-differences"></a>Différences relatives à la gestion des erreurs

MSAL fournit plus de clarté entre les erreurs qui peuvent être gérées par votre application et celles qui nécessitent une intervention de l’utilisateur. Le développeur doit gérer un nombre limité d’erreurs :

* `MSALErrorInteractionRequired`: l’utilisateur doit effectuer une requête interactive. Cela peut être dû à différentes raisons, par exemple la session d’authentification a expiré, une stratégie d’accès conditionnel a changé, un jeton d’actualisation a expiré ou a été révoqué, il n’existe aucun jeton valide dans le cache, et ainsi de suite.
* `MSALErrorServerDeclinedScopes`: la requête n’a pas été complètement effectuée et certaines étendues n’ont pas obtenu l’autorisation d’accès. Cela peut être dû au fait qu’un utilisateur a refusé le consentement à une ou plusieurs étendues.

La gestion de toutes les autres erreurs dans la [liste `MSALError`](https://github.com/AzureAD/microsoft-authentication-library-for-objc/blob/master/MSAL/src/public/MSALError.h#L128) est facultative. Vous pouvez utiliser les informations contenues dans ces erreurs pour améliorer l’expérience utilisateur.

Pour plus d’informations sur la gestion des erreurs MSAL, consultez [Gestion des exceptions et erreurs à l’aide de MSAL](msal-error-handling-ios.md).

### <a name="broker-support"></a>Prise en charge de service réparti

MSAL, à partir de la version 0.3.0, prend en charge l’authentification répartie à l’aide de l’application Microsoft Authenticator. Microsoft Authenticator autorise également la prise en charge des scénarios d’accès conditionnel. Parmi les scénarios d’accès conditionnel figurent ceux dans lesquels les stratégies de conformité des appareils obligent l’utilisateur à inscrire l’appareil par le biais d’Intune ou à s’inscrire auprès d’AAD pour obtenir un jeton, ainsi que les stratégies d’accès conditionnel de gestion des applications mobiles (GAM), qui nécessitent une preuve de conformité avant que votre application puisse obtenir un jeton.

Pour activer le répartiteur pour votre application

1. Inscrivez un format d’URI de redirection compatible avec le répartiteur pour l’application. Le format d’URI de redirection compatible avec le répartiteur est `msauth.<app.bundle.id>://auth`. Remplacez `<app.bundle.id>` par l’ID de bundle de votre application. Si vous effectuez une migration à partir d’ADAL et que votre application était déjà compatible avec un répartiteur, vous n’avez rien de plus à faire. Votre URI de redirection précédent est entièrement compatible avec MSAL. Vous pouvez donc passer à l’étape 3.

2. Ajoutez le schéma d’URI de redirection de votre application à votre fichier info.plist. Pour l’URI de redirection MSAL par défaut, le format est `msauth.<app.bundle.id>`. Par exemple :

    ```xml
    <key>CFBundleURLSchemes</key>
    <array>
        <string>msauth.<app.bundle.id></string>
    </array>
    ```

3. Ajoutez les schémas suivants au fichier Info.plist de votre application sous LSApplicationQueriesSchemes :

    ```xml
    <key>LSApplicationQueriesSchemes</key>
    <array>
         <string>msauthv2</string>
         <string>msauthv3</string>
    </array>
    ```

4. Ajoutez le code suivant à votre fichier AppDelegate.m pour gérer les rappels : Objective-C :
    
    ```objc
    - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options`
    {
        return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```
    
    Swift :
    
    ```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
    }
    ```

### <a name="business-to-business-b2b"></a>Business to business (B2B)

Dans ADAL, vous créez des instances distinctes de `ADAuthenticationContext` pour chaque locataire pour lequel l’application demande des jetons. Il ne s’agit plus d’une exigence dans MSAL. Dans MSAL, vous pouvez créer une instance unique de `MSALPublicClientApplication` et l’utiliser pour n’importe quel organisation et cloud AAD en spécifiant une autorité différente pour les appels acquireToken et acquireTokenSilent.

## <a name="sso-in-partnership-with-other-sdks"></a>Authentification unique en partenariat avec d’autres SDK

MSAL pour iOS peut implémenter l’authentification unique par le biais d’un cache unifié avec les SDK suivants :

- ADAL Objective-C 2.7.x+
- MSAL.NET pour Xamarin 2.4.x+
- ADAL.NET pour Xamarin 4.4.x+

L’authentification unique est obtenue par le biais du partage de trousseau iOS, et est disponible uniquement entre les applications publiées à partir du même compte de développeur Apple.

L’authentification unique par le biais du partage de trousseau iOS est le seul type d’authentification unique silencieux.

Sur macOS, MSAL peut implémenter l’authentification unique avec d’autres applications MSAL pour iOS et macOS et d’autres applications ADAL Objective-C.

MSAL sur iOS prend également en charge deux autres types d’authentification unique :

* Authentification unique par le biais du navigateur web. MSAL pour iOS prend en charge `ASWebAuthenticationSession`, qui fournit l’authentification unique par le biais de cookies partagés entre d’autres applications sur l’appareil et spécifiquement le navigateur Safari.
* Authentification unique par le biais d’un répartiteur d’authentification. Sur un appareil iOS, Microsoft Authenticator agit en tant que répartiteur d’authentification. Il peut suivre des stratégies d’accès conditionnel, par exemple exiger un appareil conforme, et fournit l’authentification unique pour les appareils inscrits. Les SDK MSAL à compter de la version 0.3.0 prennent en charge un répartiteur par défaut.

## <a name="intune-mam-sdk"></a>SDK GAM Intune

Le [SDK Intune GAM](/intune/app-sdk-get-started) prend en charge MSAL pour iOS à compter de la version [11.1.2](https://github.com/msintuneappsdk/ms-intune-app-sdk-ios/releases/tag/11.1.2).

## <a name="msal-and-adal-in-the-same-app"></a>MSAL et ADAL dans la même application

ADAL version 2.7.0 et ultérieures ne peuvent pas coexister avec MSAL dans la même application. Cela est principalement dû au code commun partagé du sous-module. Objective-C ne prenant pas en charge les espaces de noms, si vous ajoutez les frameworks ADAL et MSAL à votre application, il y aura deux instances de la même classe. Il n’y a aucune garantie quant à celui qui sera choisi au moment de l’exécution. Si les deux SDK utilisent la même version de la classe en conflit, votre application peut continuer à fonctionner. En revanche, s’il s’agit d’une version différente, il se peut que votre application rencontre des plantages inattendus difficiles à diagnostiquer.

L’exécution d’ADAL et de MSAL dans la même application de production n’est pas prise en charge. Toutefois, si vous ne faites que tester et migrer vos utilisateurs d’ADAL Objective-C vers MSAL pour iOS et macOS, vous pouvez continuer à utiliser [ADAL Objective-C 2.6.10](https://github.com/AzureAD/azure-activedirectory-library-for-objc/releases/tag/2.6.10). Il s’agit de la seule version qui fonctionne avec MSAL dans la même application. Il n’y aura pas de nouvelles mises à jour de fonctionnalités pour cette version d’ADAL. Vous devez donc l’utiliser uniquement à des fins de migration et de test. Votre application ne doit pas reposer sur la coexistence à long terme d’ADAL et de MSAL.

La coexistence d’ADAL et de MSAL dans la même application n’est pas prise en charge.
La coexistence d’ADAL et de MSAL entre plusieurs applications est entièrement prise en charge.

## <a name="practical-migration-steps"></a>Étapes de migration pratiques

### <a name="app-registration-migration"></a>Migration d’inscription d’application

Vous n’avez pas besoin de modifier votre application AAD existante pour basculer vers MSAL et activer les comptes AAD. Toutefois, si votre application basée sur ADAL ne prend pas en charge l’authentification répartie, vous devez inscrire un nouvel URI de redirection pour l’application avant de basculer vers MSAL.

L’URI de redirection doit être au format suivant : `msauth.<app.bundle.id>://auth`. Remplacez `<app.bundle.id>` par l’ID de bundle de votre application. Spécifiez l’URI de redirection dans le [portail Azure](https://aka.ms/MobileAppReg).

Pour iOS uniquement, afin de prendre en charge l’authentification basée sur le certificat, vous devez inscrire un URI de redirection supplémentaire dans votre application et dans le portail Azure au format suivant : `msauth://code/<broker-redirect-uri-in-url-encoded-form>`. Par exemple : `msauth://code/msauth.com.microsoft.mybundleId%3A%2F%2Fauth`

Nous recommandons que toutes les applications inscrivent les deux URI de redirection.

Si vous souhaitez ajouter la prise en charge du consentement incrémentiel, sélectionnez les API et les autorisations auxquelles votre application est configurée pour demander l’accès dans votre inscription d’application sous l’onglet **Autorisations de l’API**.

Si vous effectuez une migration à partir d’ADAL et que vous souhaitez prendre en charge les comptes AAD et MSA, vous devez mettre à jour votre inscription d’application existante pour qu’elle prenne les deux en charge. Nous vous déconseillons de mettre à jour votre application de production existante pour qu’elle prenne en charge AAD et MSA immédiatement. Au lieu de cela, créez un autre ID client qui prend en charge AAD et MSA à des fins de test. Ensuite, une fois que vous avez vérifié que tous les scénarios fonctionnent, mettez à jour l’application existante.

### <a name="add-msal-to-your-app"></a>Ajouter MSAL à votre application

Vous pouvez ajouter le SDK MSAL à votre application à l’aide de votre outil de gestion de packages préféré. Pour obtenir des instructions détaillées, consultez [cet article](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki/Installation).

### <a name="update-your-apps-infoplist-file"></a>Mettre à jour le fichier Info.plist de votre application

Pour iOS uniquement, ajoutez le schéma d’URI de redirection de votre application à votre fichier info.plist. Pour les applications compatibles avec le répartiteur ADAL, il doit déjà y figurer. Le schéma d’URI de redirection MSAL par défaut aura le format suivant : `msauth.<app.bundle.id>`.  

```xml
<key>CFBundleURLSchemes</key>
<array>
    <string>msauth.<app.bundle.id></string>
</array>
```

Ajoutez les schémas suivants au fichier Info.plist de votre application sous `LSApplicationQueriesSchemes`.

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
     <string>msauthv2</string>
     <string>msauthv3</string>
</array>
```

### <a name="update-your-appdelegate-code"></a>Mettre à jour votre code AppDelegate

Pour iOS uniquement, ajoutez le code suivant à votre fichier AppDelegate.m :

Objective-C :

```objc
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options`
{
    return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
}
```

Swift :

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
}
```

**Si vous utilisez Xcode 11**, vous devez placer le rappel MSAL dans le fichier `SceneDelegate`.
Si vous prenez en charge à la fois UISceneDelegate et UIApplicationDelegate pour assurer la compatibilité avec une version plus ancienne d’iOS, le rappel MSAL doit être placé dans les deux fichiers.

Objective-C :

```objc
 - (void)scene:(UIScene *)scene openURLContexts:(NSSet<UIOpenURLContext *> *)URLContexts
 {
     UIOpenURLContext *context = URLContexts.anyObject;
     NSURL *url = context.URL;
     NSString *sourceApplication = context.options.sourceApplication;
     
     [MSALPublicClientApplication handleMSALResponse:url sourceApplication:sourceApplication];
 }
```

Swift :

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        
        guard let urlContext = URLContexts.first else {
            return
        }
        
        let url = urlContext.url
        let sourceApp = urlContext.options.sourceApplication
        
        MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApp)
    }
```

Cela permet à MSAL de gérer les réponses du répartiteur et du composant web.
Cela n’était pas nécessaire dans ADAL, car les méthodes déléguées d’application étaient combinées automatiquement. L’ajout manuel est moins sujet aux erreurs et procure davantage de contrôle à l’application.

### <a name="enable-token-caching"></a>Activer la mise en cache des jetons

Par défaut, MSAL met en cache les jetons de votre application dans le trousseau iOS ou macOS. 

Pour activer la mise en cache des jetons
1. Vérifiez que votre application est signée correctement.
2. Accédez à vos paramètres de projet Xcode > **onglet Fonctionnalités** > **Activer le partage de trousseau**.
3. Cliquez sur **+** et fournissez l’une des entrées **Groupes de trousseaux** suivantes : 3.a. Pour iOS, entrez `com.microsoft.adalcache` 3.b. Pour macOS, entrez `com.microsoft.identity.universalstorage`.

### <a name="create-msalpublicclientapplication-and-switch-to-its-acquiretoken-and-acquiretokesilent-calls"></a>Créer MSALPublicClientApplication et basculer vers ses appels acquireToken et acquireTokeSilent

Vous pouvez créer `MSALPublicClientApplication` à l’aide du code suivant :

Objective-C :

```objc
NSError *error = nil;
MSALPublicClientApplicationConfig *configuration = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];
    
MSALPublicClientApplication *application =
[[MSALPublicClientApplication alloc] initWithConfiguration:configuration
                                                     error:&error];
```

Swift :

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
do {
  let application = try MSALPublicClientApplication(configuration: config)
  // continue on with application
            
} catch let error as NSError {
  // handle error here
}
```

Ensuite, appelez l’API de gestion des comptes pour voir s’il existe des comptes dans le cache :

Objective-C :

```objc
NSString *accountIdentifier = nil /*previously saved MSAL account identifier */;
NSError *error = nil;
MSALAccount *account = [application accountForIdentifier:accountIdentifier error:&error];
```

Swift :

```swift
// definitions that need to be initialized
let application: MSALPublicClientApplication!
let accountIdentifier: String! /*previously saved MSAL account identifier */

do {
  let account = try application.account(forIdentifier: accountIdentifier)
  // continue with account usage
} catch let error as NSError {
  // handle error here
}
```



ou lisez tous les comptes :

Objective-C :

```objc
NSError *error = nil;
NSArray<MSALAccount *> *accounts = [application allAccounts:&error];
```

Swift :

```swift
let application: MSALPublicClientApplication!
do {
  let accounts = try application.allAccounts()
  // continue with account usage
} catch let error as NSError {
  // handle error here
}
```



Si un compte est trouvé, appelez l’API `acquireTokenSilent` MSAL :

Objective-C :

```objc
MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"<your-resource-here>/.default"] account:account];
    
[application acquireTokenSilentWithParameters:silentParameters
                              completionBlock:^(MSALResult *result, NSError *error)
{
    if (result)
    {
        NSString *accessToken = result.accessToken;
        // Use your token
    }
    else
    {
        // Check the error
        if ([error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
        {
            // Interactive auth will be required
        }
            
        // Other errors may require trying again later, or reporting authentication problems to the user
    }
}];
```

Swift :

```swift
let application: MSALPublicClientApplication!
let account: MSALAccount!
        
let silentParameters = MSALSilentTokenParameters(scopes: ["<your-resource-here>/.default"], 
                                                 account: account)
application.acquireTokenSilent(with: silentParameters) {
  (result: MSALResult?, error: Error?) in
  if let accessToken = result?.accessToken {
     // use accessToken
  }
  else {
    // Check the error
    guard let error = error else {
      assert(true, "callback should contain a valid result or error")
      return
    }
    
    let nsError = error as NSError
    if (nsError.domain == MSALErrorDomain
        && nsError.code == MSALError.interactionRequired.rawValue) {
      // Interactive auth will be required
    }
                
    // Other errors may require trying again later, or reporting authentication problems to the user
  }
}
```



## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md)