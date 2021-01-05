---
title: Plug-in Microsoft Enterprise SSO pour les appareils Apple
titleSuffix: Microsoft identity platform | Azure
description: Découvrez le plug-in d’authentification unique de Microsoft Azure Active Directory pour les appareils iOS et macOS.
services: active-directory
author: brandwe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/15/2020
ms.author: brandwe
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: b7ec6ab8b52d9d43d898f481a2f36310e5c0897d
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94561078"
---
# <a name="microsoft-enterprise-sso-plug-in-for-apple-devices-preview"></a>Plug-in Microsoft Enterprise SSO pour les appareils Apple (préversion)

>[!IMPORTANT]
> Cette fonctionnalité [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

Le *plug-in Microsoft Enterprise SSO pour les appareils Apple* fournit une authentification unique (SSO) pour les comptes Azure Active Directory (Azure AD) dans toutes les applications qui prennent en charge la fonctionnalité [Enterprise Single Sign-on](https://developer.apple.com/documentation/authenticationservices) d’Apple. Microsoft a travaillé en étroite collaboration avec Apple pour développer ce plug-in et ainsi rendre plus facile l’utilisation de votre application tout en offrant la meilleure protection que peuvent offrir Apple et Microsoft.

Dans cette préversion publique, le plug-in Enterprise SSO est disponible uniquement pour les appareils iOS et est distribué dans certaines applications Microsoft.

## <a name="features"></a>Fonctionnalités

Le plug-in Microsoft Enterprise SSO pour les appareils Apple offre les avantages suivants :

- Il fournit l’authentification unique pour les comptes Azure AD dans toutes les applications qui prennent en charge la fonctionnalité Enterprise Single Sign-On d’Apple.
- Il est fourni automatiquement dans Microsoft Authenticator et peut être activé par toute solution de gestion des appareils mobiles (MDM).

## <a name="requirements"></a>Spécifications

Pour utiliser le plug-in Microsoft Enterprise SSO pour les appareils Apple :

- iOS version 13.0 ou ultérieure doit être installé sur l’appareil.
- Une application Microsoft qui fournit le plug-in Microsoft Enterprise SSO pour les appareils Apple doit être installée sur l’appareil. Pour la préversion publique, ces applications incluent l’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md).
- L’appareil doit être inscrit à MDM (par exemple, avec Microsoft Intune).
- La configuration doit être envoyée (push) à l’appareil pour activer le plug-in Microsoft Enterprise SSO pour les appareils Apple sur l’appareil. Cette contrainte de sécurité est requise par Apple.

## <a name="enable-the-sso-plug-in-with-mobile-device-management-mdm"></a>Activer le plug-in SSO avec la gestion des appareils mobiles (MDM)

Pour activer le plug-in Microsoft Enterprise SSO pour les appareils Apple, vos appareils doivent recevoir un signal via un service MDM. Comme Microsoft inclut le plug-in Enterprise SSO dans l’[application Microsoft Authenticator](..//user-help/user-help-auth-app-overview.md), utilisez votre MDM pour configurer l’application afin d’activer le plug-in Microsoft Enterprise SSO.

Utilisez les paramètres suivants pour configurer le plug-in Microsoft Enterprise SSO pour les appareils Apple :

- **Type** : Redirect
- **ID d’extension** : `com.microsoft.azureauthenticator.ssoextension`
- **ID d’équipe** : (ce champ n’est pas nécessaire pour iOS)
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
  
### <a name="additional-configuration-options"></a>Options de configuration supplémentaires
Des options de configuration supplémentaires peuvent être ajoutées pour étendre la fonctionnalité SSO à d’autres applications.

#### <a name="enable-sso-for-apps-that-dont-use-msal"></a>Activer l’authentification unique pour les applications qui n’utilisent pas MSAL

Le plug-in SSO permet à n’importe quelle application de participer à l’authentification unique, même si elle n’a pas été développée à l’aide d’un SDK Microsoft tel que la bibliothèque d’authentification Microsoft (MSAL).

Le plug-in SSO est installé automatiquement par les appareils qui ont téléchargé l’application Microsoft Authenticator et inscrit les appareils auprès de votre organisation. Votre organisation utilise probablement l’application Authenticator pour des scénarios tels que l’authentification multifacteur, l’authentification sans mot de passe et l’accès conditionnel. Elle peut être activée pour vos applications à l’aide de n’importe quel fournisseur MDM, bien que Microsoft facilite sa configuration dans Microsoft Endpoint Manager d’Intune. Une liste d’autorisation est utilisée pour configurer ces applications afin qu’elles utilisent le plug-in SSO installé par l’application Authenticator.

Seules les applications qui utilisent des technologies réseau Apple natives ou des vues WebView sont prises en charge. Si une application est livrée avec sa propre implémentation de couche réseau, le plug-in plug-in Microsoft Enterprise SSO n’est pas pris en charge.  

Utilisez les paramètres suivants pour configurer le plug-in Microsoft Enterprise SSO pour les applications qui n’utilisent pas MSAL :

- **Clé** : `AppAllowList`
- **Type** : `String`
- **Valeur** : Liste délimitée par des virgules des ID d’ensemble d’applications pour les applications qui sont autorisées à participer à l’authentification unique
- **Exemple** : `com.contoso.workapp, com.contoso.travelapp`

[Les applications avec consentement](./application-consent-experience.md) autorisées par l’administrateur MDM à participer à l’authentification unique peuvent obtenir en mode silencieux un jeton pour l’utilisateur final. Par conséquent, il est important d’ajouter uniquement des applications approuvées à la liste d’autorisation. 

Vous n’avez pas besoin d’ajouter les applications qui utilisent MSAL ou ASWebAuthenticationSession à cette liste. Elles sont activées par défaut. 

#### <a name="allow-creating-sso-session-from-any-application"></a>Autoriser la création d’une session d’authentification unique à partir de n’importe quelle application

Par défaut, le plug-in Microsoft Enterprise SSO fournit l’authentification unique pour les applications autorisées uniquement lorsque le plug-in SSO dispose déjà d’informations d’identification partagées. Le plug-in plug-in Microsoft Enterprise SSO peut acquérir des informations d’identification partagées lorsqu’il est appelé par une autre application ADAL ou MSAL lors de l’acquisition de jetons. La plupart des applications Microsoft utilisent déjà Microsoft Authenticator ou le plug-in SSO. Cela signifie que par défaut, l’authentification unique en dehors des flux d’application natifs est recommandée.  

L’activation de l’indicateur `browser_sso_interaction_enabled` permet aux applications non MSAL et au navigateur Safari d’effectuer l’amorçage initial et d’obtenir des informations d’identification partagées. Si le plug-in plug-in Microsoft Enterprise SSO ne dispose pas encore d’informations d’identification partagées, il essaiera d’en obtenir chaque fois qu’une connexion est demandée à partir d’une URL Azure AD dans le navigateur Safari, ASWebAuthenticationSession, SafariViewController ou une autre application native autorisée.  

- **Clé** : `browser_sso_interaction_enabled`
- **Type** : `Integer`
- **Valeur** : 1 ou 0

Nous vous recommandons d’activer cet indicateur pour obtenir une expérience plus cohérente sur toutes les applications. Il est désactivé par défaut. 

#### <a name="disable-oauth2-application-prompts"></a>Désactiver les invites d’application OAuth2

Le plug-in plug-in Microsoft Enterprise SSO fournit l’authentification unique en ajoutant les informations d’identification partagées aux demandes réseau provenant des applications autorisées. Certaines applications OAuth2 peuvent appliquer l’invite de l’utilisateur final sur la couche de protocole. Les informations d’identification partagées sont ignorées pour ces applications.  

L’activation de l’indicateur `disable_explicit_app_prompt` restreint la capacité des applications natives et des applications web à forcer une invite de l’utilisateur final sur la couche de protocole et à contourner l’authentification unique.

- **Clé** : `disable_explicit_app_prompt`
- **Type** : `Integer`
- **Valeur** : 1 ou 0

Nous vous recommandons d’activer cet indicateur pour obtenir une expérience plus cohérente sur toutes les applications. Il est désactivé par défaut. 

#### <a name="use-intune-for-simplified-configuration"></a>Utiliser Intune pour une configuration simplifiée

Vous pouvez utiliser Microsoft Intune comme service MDM pour faciliter la configuration du plug-in Microsoft Enterprise SSO. Pour plus d’informations, consultez la [documentation sur la configuration d’Intune](/intune/configuration/ios-device-features-settings).

## <a name="using-the-sso-plug-in-in-your-application"></a>Utilisation du plug-in SSO dans votre application

La [bibliothèque d’authentification Microsoft (MSAL) pour les appareils Apple](https://github.com/AzureAD/microsoft-authentication-library-for-objc) de version 1.1.0 ou ultérieure prend en charge le plug-in Microsoft Enterprise SSO pour les appareils Apple.

Si vous créez une application pour des scénarios de travail Frontline, consultez [Mode d’appareil partagé pour les appareils iOS](msal-ios-shared-devices.md) pour une configuration supplémentaire de cette fonctionnalité.

## <a name="how-the-sso-plug-in-works"></a>Fonctionnement du plug-in SSO

Le plug-in plug-in Microsoft Enterprise SSO s’appuie sur l’[infrastructure d’authentification unique d’Apple](https://developer.apple.com/documentation/authenticationservices/asauthorizationsinglesignonprovider?language=objc). Les fournisseurs d’identité intégrés à l’infrastructure peuvent intercepter le trafic réseau pour leurs domaines et améliorer ou modifier le mode de traitement de ces demandes. Par exemple, le plug-in SSO peut afficher une interface utilisateur supplémentaire pour collecter les informations d’identification de l’utilisateur final en toute sécurité, exiger l’authentification MFA ou fournir en mode silencieux des jetons à l’application.

Les applications natives peuvent également implémenter des opérations personnalisées et communiquer directement avec le plug-in SSO.
Vous pouvez en savoir plus sur l’infrastructure d’authentification unique dans cette [vidéo 2019 WWDC d’Apple](https://developer.apple.com/videos/play/tech-talks/301/)

### <a name="applications-that-use-msal"></a>Applications utilisant la MSAL

La [bibliothèque d’authentification Microsoft (MSAL) pour les appareils Apple](https://github.com/AzureAD/microsoft-authentication-library-for-objc) de version 1.1.0 ou ultérieure prend en charge le plug-in Microsoft Enterprise SSO pour les appareils Apple et de façon native pour les comptes scolaires et professionnels. 

Aucune configuration particulière n’est nécessaire si vous avez suivi [toutes les étapes recommandées](./quickstart-v2-ios.md) et utilisé le [format d’URI de redirection](./redirect-uris-ios.md) par défaut. Lorsqu’elle exécutée sur un appareil sur lequel est présent le plug-in SSO, MSAL l’appelle automatiquement pour toutes les demandes de jeton interactives et silencieuses, ainsi que l’énumération de compte et les opérations de suppression de compte. Étant donné que MSAL implémente le protocole de plug-in SSO natif qui repose sur les opérations personnalisées, cette configuration assure l’expérience native la plus fluide pour l’utilisateur final. 

Si le plug-in SSO n’est pas activé par MDM, mais que l’application Microsoft Authenticator est présente sur l’appareil, MSAL utilise l’application Microsoft Authenticator pour toutes les demandes de jeton interactives. Le plug-in SSO partage l’authentification unique avec l’application Microsoft Authenticator.

### <a name="applications-that-dont-use-msal"></a>Applications n’utilisant pas la MSAL

Les applications qui n’utilisent pas MSAL peuvent toujours bénéficier de l’authentification unique si un administrateur les ajoute explicitement à la liste d’autorisation. 

Aucune modification de code n’est nécessaire dans ces applications tant que les conditions suivantes sont satisfaites :

- L’application utilise les infrastructures Apple pour exécuter des requêtes réseau (par exemple, [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview), [NSURLSession](https://developer.apple.com/documentation/foundation/nsurlsession)) 
- L’application utilise des protocoles standard pour communiquer avec Azure AD (par exemple, OAuth2, SAML, WS-Federation)
- L’application ne collecte pas de nom d’utilisateur et de mot de passe en texte clair dans l’interface utilisateur native

Dans ce cas, l’authentification unique est fournie lorsque l’application crée une requête réseau et ouvre un navigateur web pour connecter l’utilisateur. Lorsque l’utilisateur est redirigé vers une URL de connexion Azure AD, le plug-in SSO valide l’URL et vérifie s’il existe des informations d’identification SSO disponibles pour cette URL. S’il en existe, le plug-in SSO transmet les informations d’identification SSO à Azure AD, ce qui permet à l’application de terminer la requête réseau sans demander à l’utilisateur d’entrer ses informations d’identification. En outre, si l’appareil est connu d’Azure AD, le plug-in SSO transmet également le certificat de l’appareil pour répondre à la vérification de l’accès conditionnel en fonction de l’appareil. 

Pour prendre en charge l’authentification unique pour les applications non-MSAL, le plug-in SSO implémente un protocole similaire au plug-in de navigateur Windows décrit dans [Qu’est-ce qu’un jeton d’actualisation principal](../devices/concept-primary-refresh-token.md#browser-sso-using-prt). 

En comparaison avec les applications basées sur MSAL, le plug-in SSO agit plus en toute transparence pour les applications non-MSAL en s’intégrant à l’expérience de connexion de navigateur existante fournie par les applications. L’utilisateur final voit son expérience habituelle, avec l’avantage de ne pas avoir à effectuer d’autres connexions dans chacune des applications. Par exemple, au lieu d’afficher le sélecteur de compte natif, le plug-in SSO ajoute des sessions SSO à l’expérience du sélecteur de compte basé sur le web. 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le mode d’appareil partagé sur iOS, consultez [Mode d’appareil partagé pour les appareils iOS](msal-ios-shared-devices.md).
