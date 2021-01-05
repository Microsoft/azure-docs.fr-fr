---
title: Migrer des applications Xamarin Android à l’aide de répartiteurs vers MSAL.NET
titleSuffix: Microsoft identity platform
description: Découvrez comment migrer d’ADAL.NET vers MSAL.NET des applications Xamarin Android qui utilisent Microsoft Authenticator ou Portail d’entreprise Intune.
author: aiwang
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/31/2020
ms.author: aiwang
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 01af0e620ecb100839f7b1101e5ff9fcfc874eea
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206667"
---
# <a name="migrate-android-applications-that-use-a-broker-from-adalnet-to-msalnet"></a>Migrer des applications Android qui utilisent un répartiteur d’ADAL.NET vers MSAL.NET

Si vous avez une application Xamarin Android qui utilise actuellement Bibliothèque d’authentification Azure Active Directory pour .NET (ADAL.NET) et un [répartiteur d’authentification](msal-android-single-sign-on.md), il est temps de migrer vers le [bibliothèque d’authentification Microsoft pour .NET](msal-overview.md) (MSAL.NET).

## <a name="prerequisites"></a>Prérequis

* Une application Xamarin Android déjà intégrée à un répartiteur ([Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) ou [Portail d’entreprise Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal)) et ADAL.NET que vous devez migrer vers MSAL.NET.

## <a name="step-1-enable-the-broker"></a>Étape 1 : Activer le répartiteur

<table>
<tr><td>Code ADAL actuel :</td><td>Contrepartie MSAL :</td></tr>
<tr><td>
Dans ADAL.NET, la prise en charge du répartiteur est activée pour chaque contexte d’authentification.

Pour appeler le répartiteur, vous deviez définir un indicateur `useBroker` sur *true* dans le constructeur `PlatformParameters` :

```CSharp
public PlatformParameters(
        Activity callerActivity,
        bool useBroker)
```

Dans le code de rendu de page spécifique à la plateforme pour Android, vous définissez l’indicateur `useBroker` sur true :

```CSharp
page.BrokerParameters = new PlatformParameters(
        this,
        true,
        PromptBehavior.SelectAccount);
```

Ensuite, incluez les paramètres dans l’appel d’acquisition de jeton :

```CSharp
AuthenticationResult result =
        await
            AuthContext.AcquireTokenAsync(
                Resource,
                ClientId,
                new Uri(RedirectURI),
                platformParameters)
                .ConfigureAwait(false);
```

</td><td>
Dans MSAL.NET, la prise en charge du répartiteur est activée pour chaque application cliente publique (PublicClientApplication).

Utilisez le paramètre `WithBroker()` (défini sur true par défaut) pour appeler le répartiteur :

```CSharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithRedirectUri(redirectUriOnAndroid)
                .Build();
```

Ensuite, dans l’appel AcquireToken :

```CSharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

## <a name="step-2-set-an-activity"></a>Étape 2 : Définir une activité

Dans ADAL.NET, vous avez passé une activité (généralement MainActivity) dans le cadre de PlatformParameters, comme indiqué dans [Étape 1 : Activer le répartiteur](#step-1-enable-the-broker).

MSAL.NET utilise également une activité, mais elle n’est pas requise dans le cas d’une utilisation Android normale sans répartiteur. Pour pouvoir utiliser le répartiteur, définissez l’activité pour qu’elle envoie et reçoive des réponses du répartiteur.

<table>
<tr><td>Code ADAL actuel :</td><td>Contrepartie MSAL :</td></tr>
<tr><td>
L’activité est transmise dans l’élément PlatformParameters de la plateforme spécifique à Android.

```CSharp
page.BrokerParameters = new PlatformParameters(
          this,
          true,
          PromptBehavior.SelectAccount);
```
</td><td>

Dans MSAL.NET, effectuez deux opérations pour définir l’activité pour Android :

1. Dans `MainActivity.cs`, définissez `App.RootViewController` sur `MainActivity` pour vous assurer qu’il existe une activité avec l’appel du répartiteur.

    S’il n’est pas défini correctement, vous pouvez recevoir cette erreur : `"Activity_required_for_android_broker":"Activity is null, so MSAL.NET cannot invoke the Android broker. See https://aka.ms/Brokered-Authentication-for-Android"`.

1. Sur l’appel AcquireTokenInteractive, utilisez `.WithParentActivityOrWindow(App.RootViewController)` et transmettez la référence à l’activité que vous allez utiliser. Cet exemple utilise MainActivity.

**Par exemple :**

Dans *App.cs* :

```CSharp
   public static object RootViewController { get; set; }
```

Dans *MainActivity.cs* :

```CSharp
   LoadApplication(new App());
   App.RootViewController = this;
```

Dans l’appel AcquireToken :

```CSharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les considérations spécifiques à Android lors de l’utilisation de MSAL.NET avec Xamarin, consultez [Configuration requise et conseils de dépannage pour Xamarin Android avec MSAL.NET](msal-net-xamarin-android-considerations.md).