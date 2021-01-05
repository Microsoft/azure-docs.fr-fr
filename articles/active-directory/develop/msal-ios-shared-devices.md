---
title: Mode d’appareil partagé pour les appareils iOS
titleSuffix: Microsoft identity platform | Azure
description: Apprenez à activer le mode Appareil partagé pour permettre aux employés de terrain de partager un appareil iOS
services: active-directory
author: brandwe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/31/2020
ms.author: brandwe
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: a97e14bcb68629f5f175a4913146187949af08be
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94561061"
---
# <a name="shared-device-mode-for-ios-devices"></a>Mode d’appareil partagé pour les appareils iOS

>[!IMPORTANT]
> Cette fonctionnalité [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

Les employés de terrain, comme les représentants, les membres d’équipage et les techniciens d’intervention, utilisent souvent un appareil mobile partagé pour effectuer leur travail. Ces appareils partagés peuvent présenter des risques de sécurité si vos utilisateurs partagent leur mot de passe ou leurs codes secrets, intentionnellement ou non, pour accéder aux données client et métier sur l’appareil partagé.

Le mode Appareil partagé vous permet de configurer un appareil iOS 13 ou ultérieur pour qu’il soit partagé plus facilement et en toute sécurité par les employés. Les employés peuvent se connecter et accéder rapidement aux informations client. Quand ils ont terminé leur journée ou leur tâche, ils peuvent se déconnecter de l’appareil et ce dernier est immédiatement prêt pour être utilisé par l’employé suivant.

Le mode Appareil partagé permet également de gérer l’appareil à l’aide des identités Microsoft.

Cette fonctionnalité utilise l’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) pour gérer les utilisateurs sur l’appareil et distribuer le [plug-in Microsoft Enterprise Single Sign-On pour les appareils Apple](apple-sso-plugin.md).

## <a name="create-a-shared-device-mode-app"></a>Créer une application en mode Appareil partagé

Pour créer une application en mode d’appareil partagé, les développeurs et les administrateurs d’appareils cloud travaillent ensemble :

1. Les **Développeurs d’applications** écrivent une application monocompte (les applications multicomptes ne sont pas prises en charge en mode Appareil partagé) et du code pour traiter des opérations comme la déconnexion des appareils partagés.

1. Les **Administrateurs d’appareils** préparent l’appareil pour qu’il soit partagé à l’aide d’un fournisseur de gestion des périphériques mobiles (GPM) tel que Microsoft Intune pour gérer les appareils de leur organisation. Le GPM envoie (push) l’application Microsoft Authenticator sur les appareils et active le « mode partagé » pour chaque appareil via une mise à jour de profil sur l’appareil. Ce paramètre Mode partagé est ce qui modifie le comportement des applications prises en charge sur l’appareil. Cette configuration à partir du fournisseur GPM définit le mode Appareil partagé pour l’appareil et active le [plug-in Microsoft Enterprise Single Sign-On pour les appareils Apple](apple-sso-plugin.md), qui est requis pour le mode Appareil partagé.

1. [**Requis en Préversion publique uniquement**] Un utilisateur doté du rôle [Administrateur d’appareil cloud](../roles/permissions-reference.md#cloud-device-administrator) doit alors lancer l’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) et joindre son appareil à l’organisation.

    Pour configurer l’appartenance de vos rôles organisationnels dans le Portail Azure : **Azure Active Directory** > **Rôles et administrateurs** > **Administrateur d’appareil cloud**

Les sections suivantes vous aident à mettre à jour votre application pour prendre en charge le mode Appareil partagé.

## <a name="use-intune-to-enable-shared-device-mode--sso-extension"></a>Utiliser Intune pour activer le mode Appareil partagé et l’extension SSO

> [!NOTE]
> L’étape suivante est requise uniquement dans la préversion publique.

Votre appareil doit être configuré pour prendre en charge le mode Appareil partagé. La version 13 ou ultérieure d’iOS doit être installée et inscrite sur GPM. La configuration GPM doit également activer le [plug-in Microsoft Enterprise Single Sign-On pour les appareils Apple](apple-sso-plugin.md). Pour en savoir plus sur les extensions SSO, regardez la [vidéo Apple](https://developer.apple.com/videos/play/tech-talks/301/).

1. Dans le portail de configuration Intune, indiquez à l’appareil d’activer le [plug-in Microsoft Enterprise Single Sign-On pour les appareils Apple](apple-sso-plugin.md) avec la configuration suivante :

    - **Type** : Rediriger
    - **ID d’extension** : com.microsoft.azureauthenticator.ssoextension
    - **ID d'équipe** : (ce champ n'est pas nécessaire pour iOS)
    - **URL** :   
        - `https://login.microsoftonline.com`
        - `https://login.microsoft.com`
        - `https://sts.windows.net`
        - `https://login.partner.microsoftonline.cn`
        - `https://login.chinacloudapi.cn`
        - `https://login.microsoftonline.de`
        - `https://login.microsoftonline.us`
        - `https://login.usgovcloudapi.net`
        - `https://login-us.microsoftonline.com`
    - **Données supplémentaires à configurer** :
      - Clé : sharedDeviceMode
      - Tapez : Boolean
      - Valeur : true

    Pour plus d’informations sur la configuration avec Intune, consultez la [documentation sur la configuration d’Intune](/intune/configuration/ios-device-features-settings).

1. Ensuite, configurez votre GPM pour envoyer (push) l’application Microsoft Authenticator à votre appareil via un profil GPM.

    Définissez les options de configuration suivantes pour activer le mode Appareil partagé :

    - Configuration 1 :
      - Clé : sharedDeviceMode
      - Tapez : Boolean
      - Valeur : true

## <a name="modify-your-ios-application-to-support-shared-device-mode"></a>Modifier votre application iOS pour prendre en charge le mode Appareil partagé

Vos utilisateurs dépendent de vous et vous devez leur garantir que leurs données ne seront pas divulguées à un autre utilisateur. Les sections suivantes fournissent des signaux utiles pour indiquer à votre application qu’une modification a eu lieu et doit être traitée.

Il vous incombe de vérifier l’état de l’utilisateur sur l’appareil chaque fois que votre application est utilisée, puis d’effacer les données de l’utilisateur précédent. Cela inclut le rechargement en arrière-plan en mode multitâche.

En cas de changement d’utilisateur, vous devez vous assurer que les données de l’utilisateur précédent sont effacées et que toutes les données en cache affichées dans votre application sont supprimées. Nous vous recommandons vivement, à vous et à votre entreprise, de procéder à un examen de sécurité après avoir mis à jour votre application pour qu’elle prenne en charge le mode Appareil partagé.

### <a name="detect-shared-device-mode"></a>Détecter le mode Appareil partagé

La détection du mode Appareil partagé est importante pour votre application. De nombreuses applications nécessitent une modification de leur expérience utilisateur (UX) quand l’application est utilisée sur un appareil partagé. Par exemple, votre application peut avoir une fonctionnalité « Inscription », ce qui n’est pas approprié pour un employé de terrain, car il a probablement déjà un compte. Vous pouvez également ajouter une sécurité supplémentaire au traitement des données de votre application si celle-ci est en mode Appareil partagé.

Utilisez l’API `getDeviceInformationWithParameters:completionBlock:` dans `MSALPublicClientApplication` pour déterminer si une application est en cours d’exécution sur un appareil en mode Appareil partagé.

Les extraits de code suivants présentent des exemples d’utilisation de l’API `getDeviceInformationWithParameters:completionBlock:`.

#### <a name="swift"></a>Swift

```swift
application.getDeviceInformation(with: nil, completionBlock: { (deviceInformation, error) in

    guard let deviceInfo = deviceInformation else {
        return
    }

    let isSharedDevice = deviceInfo.deviceMode == .shared
    // Change your app UX if needed
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
[application getDeviceInformationWithParameters:nil
                                completionBlock:^(MSALDeviceInformation * _Nullable deviceInformation, NSError * _Nullable error)
{
    if (!deviceInformation)
    {
        return;
    }

    BOOL isSharedDevice = deviceInformation.deviceMode == MSALDeviceModeShared;
    // Change your app UX if needed
}];
```

### <a name="get-the-signed-in-user-and-determine-if-a-user-has-changed-on-the-device"></a>Obtenir l’utilisateur connecté et déterminer si l’utilisateur de l’appareil a changé

Une autre partie importante de la prise en charge du mode Appareil partagé consiste à déterminer l’état de l’utilisateur sur l’appareil et à effacer les données d’application si l’utilisateur de l’appareil a changé ou si aucun utilisateur n’est enregistré sur l’appareil. Il vous incombe de veiller à ce que les données ne soient pas divulguées à un autre utilisateur.

Vous pouvez utiliser l’API `getCurrentAccountWithParameters:completionBlock:` pour interroger le compte actuellement connecté sur l’appareil.

#### <a name="swift"></a>Swift

```swift
let msalParameters = MSALParameters()
msalParameters.completionBlockQueue = DispatchQueue.main

application.getCurrentAccount(with: msalParameters, completionBlock: { (currentAccount, previousAccount, error) in

    // currentAccount is the currently signed in account
    // previousAccount is the previously signed in account if any
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALParameters *parameters = [MSALParameters new];
parameters.completionBlockQueue = dispatch_get_main_queue();

[application getCurrentAccountWithParameters:parameters
                             completionBlock:^(MSALAccount * _Nullable account, MSALAccount * _Nullable previousAccount, NSError * _Nullable error)
{
    // currentAccount is the currently signed in account
    // previousAccount is the previously signed in account if any
}];
```

### <a name="globally-sign-in-a-user"></a>Connecter globalement un utilisateur

Quand un appareil est configuré en tant qu’appareil partagé, votre application peut appeler l’API `acquireTokenWithParameters:completionBlock:` pour se connecter au compte. Le compte sera disponible dans le monde entier pour toutes les applications admissibles sur l’appareil après la première connexion de l’application sur le compte.

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALInteractiveTokenParameters *parameters = [[MSALInteractiveTokenParameters alloc] initWithScopes:@[@"api://myapi/scope"] webviewParameters:[self msalTestWebViewParameters]];

parameters.loginHint = self.loginHintTextField.text;

[application acquireTokenWithParameters:parameters completionBlock:completionBlock];
```

### <a name="globally-sign-out-a-user"></a>Déconnecter globalement un utilisateur

Le code suivant supprime le compte connecté et efface les jetons mis en cache de l’application, mais également de l’appareil en mode Appareil partagé. Toutefois, il n’efface pas les *données* de votre application. Vous devez effacer les données de votre application, ainsi que toutes les données en cache que votre application peut afficher à l’utilisateur.

#### <a name="clear-browser-state"></a>Effacer l’état du navigateur

> [!NOTE]
> L’étape suivante est requise uniquement dans la préversion publique.

Dans cette version préliminaire publique, le [plug-in Microsoft Enterprise Single Sign-On pour les appareils Apple](apple-sso-plugin.md) efface l’état des applications uniquement. Il n’efface pas l’état du navigateur Safari. Nous vous recommandons d’effacer manuellement la session de navigateur pour vous assurer qu’aucune trace de l’état utilisateur n’est conservée. Vous pouvez utiliser la propriété facultative `signoutFromBrowser` présentée ci-dessous pour effacer les cookies. Le navigateur est alors lancé brièvement sur l’appareil.

#### <a name="swift"></a>Swift

```swift
let account = .... /* account retrieved above */

let signoutParameters = MSALSignoutParameters(webviewParameters: self.webViewParamaters!)
signoutParameters.signoutFromBrowser = true // Only needed for Public Preview.

application.signout(with: account, signoutParameters: signoutParameters, completionBlock: {(success, error) in

    if let error = error {
        // Signout failed
        return
    }

    // Sign out completed successfully
})
```

#### <a name="objective-c"></a>Objective-C

```objective-c
MSALAccount *account = ... /* account retrieved above */;

MSALSignoutParameters *signoutParameters = [[MSALSignoutParameters alloc] initWithWebviewParameters:webViewParameters];
signoutParameters.signoutFromBrowser = YES; // Only needed for Public Preview.

[application signoutWithAccount:account signoutParameters:signoutParameters completionBlock:^(BOOL success, NSError * _Nullable error)
{
    if (!success)
    {
        // Signout failed
        return;
    }

    // Sign out completed successfully
}];
```

## <a name="next-steps"></a>Étapes suivantes

Pour voir le mode Appareil partagé en action, l’exemple de code suivant sur GitHub fournit un exemple d’exécution d’une application FirstLine Worker sur un appareil iOS en mode Appareil partagé :

[Exemple d’API Microsoft Graph MSAL iOS Swift](https://github.com/Azure-Samples/ms-identity-mobile-apple-swift-objc)
