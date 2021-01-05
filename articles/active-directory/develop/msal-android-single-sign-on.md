---
title: Comment activer l’authentification unique entre applications sur Android à l’aide de MSAL | Azure
titleSuffix: Microsoft identity platform
description: Comment utiliser la bibliothèque d’authentification Microsoft (MSAL) pour Android pour activer l’authentification unique sur vos applications.
services: active-directory
author: hamiltonha
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: android
ms.devlang: java
ms.topic: how-to
ms.date: 10/15/2020
ms.author: hahamil
ms.reviewer: marsma
ms.openlocfilehash: 3f5791bfcf6547b7fc4e84bee3d4c1c49453af9c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96169492"
---
# <a name="how-to-enable-cross-app-sso-on-android-using-msal"></a>Procédure : Activer l’authentification unique entre applications sur Android à l’aide de MSAL

L’authentification unique (SSO) permet aux utilisateurs de ne saisir leurs informations d’identification qu’une seule fois et de faire en sorte que ces informations d’identification fonctionnent automatiquement entre les applications.

La [Microsoft Identity Platform](./index.yml) et la bibliothèque d’authentification Microsoft (MSAL) vous aident à activer l’authentification unique dans votre propre suite d’applications. Grâce aux applications de service Broker et Authenticator, vous pouvez étendre l’authentification unique sur l’ensemble de l’appareil.

Dans cette procédure, vous allez apprendre à configurer les kits de développement logiciel (SDK) utilisés par votre application pour fournir une authentification unique à vos clients.

## <a name="prerequisites"></a>Prérequis

Cette procédure suppose que vous savez :

- Approvisionner votre application à l’aide du Portail Azure. Pour plus d’informations sur cette rubrique, consultez les instructions relatives à la création d’une application dans [le didacticiel Android](./tutorial-v2-android.md#create-a-project)
- Intégrez votre application à la [bibliothèque d’authentification Microsoft pour Android](https://github.com/AzureAD/microsoft-authentication-library-for-android).

## <a name="methods-for-single-sign-on"></a>Méthodes d’authentification unique

Il existe deux façons pour les applications utilisant MSAL pour Android d’obtenir l’authentification unique :

* Via une [application Broker](#sso-through-brokered-authentication)
* Via le [navigateur système](#sso-through-system-browser)


   Il est recommandé d’utiliser une application Broker pour des avantages tels que l’authentification unique de l’appareil, la gestion des comptes et l’accès conditionnel. Toutefois, les utilisateurs doivent télécharger des applications supplémentaires.

## <a name="sso-through-brokered-authentication"></a>Authentification unique via l’authentification répartie

Nous vous recommandons d’utiliser l’un des répartiteurs d’authentification Microsoft pour participer à l’authentification unique (SSO) des appareils et répondre aux politiques d’accès conditionnel de l’organisation. L’intégration d'un répartiteur vous offre les avantages suivants :

- Authentification unique des appareils
- Accès conditionnel pour :
  - Intune App Protection
  - Inscription des appareils (Workplace Join)
  - Gestion des appareils mobiles
- Gestion des comptes d'appareil
  -  via Android AccountManager et les paramètres de compte
  - « Compte professionnel » - type de compte personnalisé

Sur Android, le répartiteur d’authentification Microsoft est un composant fourni avec l’application [Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) et le [Portail d’entreprise Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal).

Le diagramme suivant illustre la relation entre votre application, la bibliothèque d'authentification Microsoft (MSAL) et les répartiteurs d'authentification de Microsoft.

![Diagramme montrant le lien entre une application et la Bibliothèque d’authentification Microsoft, les applications de répartiteur et le gestionnaire de compte Android.](./media/brokered-auth/brokered-deployment-diagram.png)

### <a name="installing-apps-that-host-a-broker"></a>Installation d’applications hébergeant un répartiteur

Les applications hébergeant un répartiteur peuvent à tout moment être installées par le propriétaire de l’appareil à partir de son magasin d’applications (généralement Google Play Store). Toutefois, certaines API (ressources) sont protégées par des stratégies d’accès conditionnel qui impliquent que les appareils soient :

- Inscrits (rattachés à l’espace de travail) et/ou
- Inscrits dans la Gestion des appareils ou
- Inscrits dans Intune App Protection

Si aucune application de répartiteur n’est installée sur l’appareil, la Bibliothèque d’authentification Microsoft demande à l’utilisateur d’en installer une dès que l’application tente d’obtenir un jeton de manière interactive. L’application doit alors guider l’utilisateur à travers les étapes visant à assurer la conformité de l'appareil avec la stratégie requise.

### <a name="effects-of-installing-and-uninstalling-a-broker"></a>Effets de l’installation et de la désinstallation d’un répartiteur

#### <a name="when-a-broker-is-installed"></a>Lors de l’installation d’un répartiteur

Lorsqu’un répartiteur est installé sur un appareil, toutes les demandes de jetons interactives ultérieures (appels à `acquireToken()`) sont gérées par le répartiteur plutôt que localement par MSAL. Aucun état SSO précédemment disponible pour MSAL n’est disponible pour le répartiteur. Dès lors, l’utilisateur doit s’authentifier à nouveau ou sélectionner un compte dans la liste des comptes existants connus de l’appareil.

L’installation d’un répartiteur n’implique pas que l’utilisateur se connecte à nouveau. Ce n'est que lorsque l'utilisateur doit résoudre une exception `MsalUiRequiredException` que la demande suivante est transmise au répartiteur. `MsalUiRequiredException` peut être levée pour plusieurs raisons et doit être résolue de manière interactive. Par exemple :

- L’utilisateur a modifié le mot de passe associé à son compte.
- Le compte de l’utilisateur ne répond plus à une stratégie d’accès conditionnel.
- L’utilisateur a révoqué son consentement pour l'association de l'application à son compte.

**Répartiteurs multiples** - Si plusieurs répartiteurs sont installés sur un appareil, le répartiteur actif est toujours le premier à avoir été installé. Un seul répartiteur peut être actif sur un appareil.

#### <a name="when-a-broker-is-uninstalled"></a>Lors de la désinstallation d’un répartiteur

En présence d'une seule application hébergeant un répartiteur, si celle-ci est supprimée, l’utilisateur doit se connecter à nouveau. La désinstallation du répartiteur actif supprime le compte et les jetons associés de l’appareil.

Si le Portail d’entreprise Intune est installé et fait office de répartiteur, et que Microsoft Authenticator est également installé, en cas de désinstallation du Portail d’entreprise Intune (répartiteur actif), l’utilisateur doit se connecter à nouveau. Ce faisant, l'application Microsoft Authenticator devient le répartiteur actif.

### <a name="integrating-with-a-broker"></a>Intégration d'un répartiteur

#### <a name="generate-a-redirect-uri-for-a-broker"></a>Générer un URI de redirection pour un service Broker

Vous devez inscrire un URI de redirection compatible avec le répartiteur. L’URI de redirection du répartiteur doit de préférence contenir le nom du package de votre application et la représentation encodée en Base64 de la signature de votre application.

Le format de l’URI de redirection est le suivant : `msauth://<yourpackagename>/<base64urlencodedsignature>`

Vous pouvez utiliser [keytool](https://manpages.debian.org/buster/openjdk-11-jre-headless/keytool.1.en.html) pour générer un hachage de signature encodé en base64 à l’aide des clés de signature de votre application, puis utiliser le Portail Azure pour générer votre URI de redirection à l’aide de ce hachage.

Linux et macOS :

```bash
keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
```

Windows :

```powershell
keytool -exportcert -alias androiddebugkey -keystore %HOMEPATH%\.android\debug.keystore | openssl sha1 -binary | openssl base64
```

Une fois que vous avez généré un hachage de signature avec *keytool*, utilisez le Portail Azure pour générer l’URI de redirection :

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et sélectionnez votre application Android dans **Inscriptions d’applications**.
1. Sélectionnez **Authentification** > **Ajouter une plateforme** > **Android**.
1. Dans le volet **Configurer votre application Android** qui s’ouvre, entrez le **hachage de Signature** que vous avez généré précédemment et un **nom de Package**.
1. Sélectionnez le bouton **Configurer**.

Le Portail Azure génère automatiquement l’URI de redirection et l’affiche dans le champ **URI de redirection** du volet **Configuration Android**.

Pour plus d’informations sur la signature de votre application, consultez [Signature de votre application](https://developer.android.com/studio/publish/app-signing) dans le Guide de l’utilisateur Android Studio.

> [!IMPORTANT]
> Utilisez votre clé de signature de production pour la version de production de votre application.

#### <a name="configure-msal-to-use-a-broker"></a>Configurer MSAL pour utiliser un répartiteur

Pour utiliser un répartiteur dans votre application, vous devez attester que vous avez configuré votre redirection de répartiteur. Par exemple, incluez à la fois votre URI de redirection activé par Broker, et indiquez que vous l’avez enregistré, en incluant les paramètres suivants dans votre fichier de configuration MSAL :

```json
"redirect_uri" : "<yourbrokerredirecturi>",
"broker_redirect_uri_registered": true
```

#### <a name="broker-related-exceptions"></a>Exceptions liées au répartiteur

MSAL communique avec le répartiteur de deux manières :

- Service lié au répartiteur
- Android AccountManager

MSAL utilise d’abord le service lié au répartiteur car l’appel vers ce service ne nécessite aucune autorisation Android. Si la liaison au service lié échoue, MSAL utilise l’API Android AccountManager. MSAL ne le fait que si l’autorisation `"READ_CONTACTS"` est déjà accordée à votre application.

Si vous recevez une exception `MsalClientException` accompagnée du code d’erreur `"BROKER_BIND_FAILURE"`, deux options s'offrent à vous :

- Demandez à l’utilisateur de désactiver l’optimisation de l’alimentation pour l’application Microsoft Authenticator et le Portail d’entreprise Intune.
- Demandez à l’utilisateur d’accorder l’autorisation `"READ_CONTACTS"`.

### <a name="verify-broker-integration"></a>Vérifiez l’intégration du répartiteur

Il n’est peut-être pas immédiatement clair que l’intégration du répartiteur fonctionne, mais vous pouvez suivre les étapes ci-dessous pour opérer une vérification :

1. Sur votre appareil Android, remplissez une demande à l’aide du répartiteur.
1. Dans les paramètres de votre appareil Android, recherchez un compte nouvellement créé correspondant au compte auprès duquel vous vous êtes authentifié. Le compte doit être de type *Compte professionnel*.

Vous pouvez supprimer le compte des paramètres si vous souhaitez répéter le test.

## <a name="sso-through-system-browser"></a>Authentification unique via le navigateur système

Les applications Android ont la possibilité d’utiliser les onglets WebView, explorateur système ou Chrome pour l’expérience utilisateur d’authentification. Si l’application n’utilise pas l’authentification répartie, elle doit utiliser le navigateur système plutôt que la WebView native pour permettre l’authentification unique.

### <a name="authorization-agents"></a>Agents d’autorisation

Le choix d’une stratégie spécifique pour les agents d’autorisation est facultatif et représente des fonctionnalités supplémentaires que les applications peuvent personnaliser. La plupart des applications utiliseront les valeurs par défaut MSAL (consultez [Comprendre le fichier de configuration MSAL Android](msal-configuration.md) pour voir les différentes valeurs par défaut).

MSAL prend en charge l’autorisation à l’aide d’un `WebView` ou du navigateur système.  L’image ci-dessous montre son aspect avec `WebView`, ou le navigateur système avec CustomTabs ou sans CustomTabs :

![Exemples de connexion MSAL](./media/authorization-agents/sign-in-ui.jpg)

### <a name="single-sign-on-implications"></a>Implications de l’authentification unique

Par défaut, les applications intégrées à MSAL utilisent les onglets personnalisés du navigateur système pour l’autorisation. À la différence des vues web, les onglets personnalisés partagent un fichier jar de cookie avec le navigateur système par défaut, ce qui permet de réduire le nombre de connexions avec d’autres applications web ou natives qui ont été intégrées à des onglets personnalisés.

Si l’application utilise une stratégie `WebView` sans intégrer Microsoft Authenticator ou la prise en charge d’un Portail d’entreprise, les utilisateurs ne disposent pas d’une expérience d’authentification unique (SSO) sur l’appareil ou entre des applications natives et des applications Web.

Si l’application utilise MSAL avec un service Broker comme Microsoft Authenticator ou Portail d’entreprise Intune, les utilisateurs peuvent avoir une expérience d’authentification unique entre les applications s’ils disposent d’une connexion active avec l’une des applications.

### <a name="webview"></a>Vue web

Pour utiliser la vue web dans l’application, insérez la ligne suivante dans le fichier JSON de configuration de l’application passé à MSAL :

```json
"authorization_user_agent" : "WEBVIEW"
```

Lorsqu’il utilise la `WebView` dans l’application, l’utilisateur se connecte directement à l’application. Les jetons sont conservés à l’intérieur du bac à sable de l’application et ne sont pas disponibles en dehors du fichier jar du cookie de l’application. Ainsi, l’utilisateur ne peut pas bénéficier d’une expérience SSO entre les applications, sauf si les applications s’intègrent à Authenticator ou au portail d’entreprise.

Toutefois, `WebView` offre la possibilité de personnaliser l’apparence de l’interface utilisateur de connexion. Pour plus d’informations sur la façon de procéder à cette personnalisation, consultez [Vues web Android](https://developer.android.com/reference/android/webkit/WebView).

### <a name="default-browser-plus-custom-tabs"></a>Navigateur par défaut et onglets personnalisés

Par défaut, MSAL utilise le navigateur et une stratégie avec des [onglets personnalisés](https://developer.chrome.com/multidevice/android/customtabs). Vous pouvez indiquer explicitement cette stratégie pour empêcher des changements dans les versions ultérieures de `DEFAULT` en utilisant la configuration JSON suivante dans le fichier de configuration personnalisé :

```json
"authorization_user_agent" : "BROWSER"
```

Utilisez cette approche pour fournir une expérience SSO par le biais du navigateur de l’appareil. MSAL utilise un fichier jar de cookie partagé, qui permet à d’autres applications natives ou web de bénéficier de l’authentification unique sur l’appareil à l’aide du cookie de session persistent défini par MSAL.

### <a name="browser-selection-heuristic"></a>Heuristique de sélection du navigateur

Étant donné qu’il est impossible pour MSAL de spécifier le package de navigateur exact à utiliser sur chaque gamme de téléphones Android, MSAL implémente une heuristique de sélection du navigateur qui tente de fournir la meilleure expérience SSO entre les appareils.

MSAL récupère principalement le navigateur par défaut à partir du gestionnaire de package et vérifie s’il se trouve dans une liste de navigateurs sécurisés testée. Si ce n’est pas le cas, MSAL utilise la Webview au lieu de lancer un autre navigateur non défini par défaut à partir de la liste sécurisée. Le navigateur par défaut est choisi, qu’il prenne en charge les onglets personnalisés ou non. Si le navigateur prend en charge les onglets personnalisés, MSAL lance l’onglet personnalisé. Les onglets personnalisés ont une apparence plus proche de celle d’une `WebView` dans l’application et permettent une personnalisation de l’interface utilisateur de base. Pour plus d’informations, consultez [Onglets personnalisés dans Android](https://developer.chrome.com/multidevice/android/customtabs).

S’il n’existe aucun package de navigateur sur l’appareil, MSAL utilise la `WebView` dans l’application. Si vous ne changez pas le paramètre par défaut de l’appareil, le même navigateur est lancé pour chaque connexion afin de garantir une expérience d’authentification unique (SSO).

#### <a name="tested-browsers"></a>Navigateurs testés

Les navigateurs suivants ont été testés pour voir s’ils redirigent correctement vers le `"redirect_uri"` spécifié dans le fichier de configuration :

| Appareil | Navigateur intégré | Chrome | Opera  | Microsoft Edge | UC Browser | Firefox |
| -- |:-------------:| -----:|-----:|-----:|-----:|-----:|
| Nexus 4 (API 17) | Réussite | Réussite |Non applicable |Non applicable |Non applicable |Non applicable |
| Samsung S7 (API 25) | passer<sup>1</sup> | Réussite | Réussite | Réussite | Échec |Réussite |
| Huawei (API 26) |passer<sup>2</sup> | Réussite | Échec | Réussite | Réussite |Réussite |
| Vivo (API 26) |Réussite|Réussite|Réussite|Réussite|Réussite|Échec|
| Pixel 2 (API 26) |Réussite | Réussite | Réussite | Réussite | Échec |Réussite |
| Oppo | Réussite | non applicable<sup>3</sup>|Non applicable  |Non applicable |Non applicable | Non applicable|
| OnePlus (API 25) |Réussite | Réussite | Réussite | Réussite | Échec |Réussite |
| Nexus (API 28) |Réussite | Réussite | Réussite | Réussite | Échec |Réussite |
|MI | Réussite | Réussite | Réussite | Réussite | Échec |Réussite |

<sup>1</sup>Le navigateur intégré de Samsung est Samsung Internet.<br/>
<sup>2</sup>Le navigateur intégré de Huawei est le navigateur Huawei.<br/>
<sup>3</sup>Le navigateur par défaut ne peut pas être modifié dans le paramètre de l’appareil OPPO.

## <a name="next-steps"></a>Étapes suivantes

Le [mode Appareil partagé pour les appareils Android](msal-android-shared-devices.md) vous permet de configurer un appareil Android de sorte qu’il soit facilement partageable par plusieurs employés.