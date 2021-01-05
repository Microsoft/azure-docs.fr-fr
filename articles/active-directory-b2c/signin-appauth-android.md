---
title: Acquérir un jeton dans une application Android
titleSuffix: Azure AD B2C
description: Guide pratique pour créer une application Android qui utilise AppAuth avec Azure Active Directory B2C pour gérer les identités utilisateur et authentifier les utilisateurs.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/12/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: acbd2918bd311cec1c27018763ad10771d779d85
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953319"
---
# <a name="sign-in-using-an-android-application-in-azure-active-directory-b2c"></a>Se connecter à l'aide d'une application Android dans Azure Active Directory B2C

La plateforme d’identité Microsoft utilise des normes ouvertes telles que OAuth2 et OpenID Connect. Ces normes vous permettent de tirer parti de toutes les bibliothèques que vous souhaitez intégrer à Azure Active Directory B2C. Pour vous aider à utiliser d’autres bibliothèques, vous pouvez utiliser une procédure pas à pas comme celle-ci afin d’expliquer la configuration des bibliothèques tierces pour se connecter à la plateforme d’identité Microsoft. La plupart des bibliothèques qui implémentent la [spécification RFC6749 OAuth2](https://tools.ietf.org/html/rfc6749) peuvent se connecter à la plateforme d’identité Microsoft.

> [!WARNING]
> Microsoft ne fournit pas de correctifs pour les bibliothèques tierces et ne les a pas vérifiées. Cet exemple utilise une bibliothèque tierce appelée AppAuth dont la compatibilité a été testée dans des scénarios de base avec Azure AD B2C. Les problèmes et les demandes de fonctionnalités doivent être soumis au projet open source de la bibliothèque. Pour plus d’informations, consultez [cet article](../active-directory/develop/reference-v2-libraries.md).
>
>

Si vous découvrez OAuth2 ou OpenID Connect, cet exemple de configuration n’est peut-être pas très parlant pour vous. Nous vous recommandons de consulter une brève [vue d’ensemble du protocole que nous avons décrit ici](protocols-overview.md).

## <a name="get-an-azure-ad-b2c-directory"></a>Obtention d'un répertoire Azure AD B2C

Avant de pouvoir utiliser Azure AD B2C, vous devez créer un répertoire ou un client. Un répertoire est un conteneur destiné à recevoir tous vos utilisateurs, applications, groupes, etc. Si vous n’en possédez pas déjà un, [créez un répertoire B2C](tutorial-create-tenant.md) avant de continuer.

## <a name="create-an-application"></a>Créer une application

Inscrivez ensuite une application dans votre locataire Azure AD B2C. Vous fournissez ainsi à Azure AD les informations nécessaires pour communiquer avec votre application en toute sécurité.

[!INCLUDE [active-directory-b2c-appreg-native](../../includes/active-directory-b2c-appreg-native.md)]

Enregistrez l’**ID d’application (client)** pour l’utiliser dans une étape ultérieure.

Enregistrez également votre URI de redirection personnalisé pour l’utiliser dans une étape ultérieure. Par exemple : `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`.

## <a name="create-your-user-flows"></a>Créer vos flux utilisateur

Dans Azure AD B2C, chaque expérience utilisateur est définie par un [flux utilisateur](user-flow-overview.md), c’est-à-dire un ensemble de stratégies qui contrôlent le comportement d’Azure AD. Cette application requiert un flux utilisateur de connexion et d'inscription. Lorsque vous créez le flux utilisateur, veillez à effectuer les actions suivantes :

* Choisir le **nom d’affichage** et un attribut d’inscription dans votre flux utilisateur.
* Choisir **le nom d’affichage** et **l’ID objet** comme revendications d’application pour chaque flux utilisateur. Vous pouvez aussi choisir d'autres revendications.
* Copier le **nom** de chaque flux utilisateur après sa création. Il doit porter le préfixe `b2c_1_`.  Vous aurez besoin du nom du flux utilisateur ultérieurement.

Une fois vos flux d’utilisateurs créés, vous pouvez générer votre application.

## <a name="download-the-sample-code"></a>Télécharger l’exemple de code

Nous avons fourni un exemple fonctionnel qui utilise AppAuth avec Azure AD B2C [sur GitHub](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c). Vous pouvez télécharger le code et l’exécuter. Vous pouvez prendre rapidement en main votre application à l’aide de votre propre configuration Azure AD B2C en suivant les instructions du fichier [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md).

L’exemple est une modification de l’exemple fourni par [AppAuth](https://openid.github.io/AppAuth-Android/). Consultez la page correspondante pour en savoir plus sur AppAuth et ses fonctionnalités.

## <a name="modifying-your-app-to-use-azure-ad-b2c-with-appauth"></a>Modification de votre application pour utiliser Azure AD B2C avec AppAuth

> [!NOTE]
> AppAuth prend en charge Android API 16 (Jellybean) et versions ultérieures. Nous recommandons d’utiliser API 23 et versions ultérieures.
>

### <a name="configuration"></a>Configuration

Vous pouvez configurer la communication avec Azure AD B2C en spécifiant l’URI de détection ou en spécifiant les URI du point de terminaison d’autorisation et du point de terminaison de jeton. Dans les deux cas, vous aurez besoin des informations suivantes :

* ID client (par ex., contoso.onmicrosoft.com)
* Nom du flux utilisateur (par ex., B2C\_1\_SignUpIn)

Si vous choisissez de détecter automatiquement les URI du point de terminaison d’autorisation et de jeton URI, vous devrez extraire des informations de l’URI de détection. L’URI de détection peut être généré en remplaçant le`<tenant-id>` et le `<policy-name>` dans l’URL suivante :

```java
String mDiscoveryURI = "https://<tenant-name>.b2clogin.com/<tenant-id>/<policy-name>/v2.0/.well-known/openid-configuration";
```

Vous pouvez alors obtenir les URI des points de terminaison d’autorisation et de jeton et créer un objet AuthorizationServiceConfiguration en exécutant la commande suivante :

```java
final Uri issuerUri = Uri.parse(mDiscoveryURI);
AuthorizationServiceConfiguration config;

AuthorizationServiceConfiguration.fetchFromIssuer(
    issuerUri,
    new RetrieveConfigurationCallback() {
      @Override public void onFetchConfigurationCompleted(
          @Nullable AuthorizationServiceConfiguration serviceConfiguration,
          @Nullable AuthorizationException ex) {
        if (ex != null) {
            Log.w(TAG, "Failed to retrieve configuration for " + issuerUri, ex);
        } else {
            // service configuration retrieved, proceed to authorization...
        }
      }
  });
```

Plutôt que d’utiliser la détection pour obtenir les URI des points de terminaison d’autorisation et de jeton, vous pouvez également les spécifier explicitement en remplaçant le `<tenant-id>` et le `<policy-name>`dans l’URL ci-dessous :

```java
String mAuthEndpoint = "https://<tenant-name>.b2clogin.com/<tenant-id>/<policy-name>/oauth2/v2.0/authorize";

String mTokenEndpoint = "https://<tenant-name>.b2clogin.com/<tenant-id>/<policy-name>/oauth2/v2.0/token";
```

Exécutez le code suivant pour créer votre objet AuthorizationServiceConfiguration :

```java
AuthorizationServiceConfiguration config =
        new AuthorizationServiceConfiguration(name, mAuthEndpoint, mTokenEndpoint);

// perform the auth request...
```

### <a name="authorizing"></a>Autorisation

Après la configuration ou la récupération d’une configuration de service d’autorisation, une demande d’autorisation peut être créée. Pour créer la demande, vous aurez besoin des informations suivantes :

* ID client (ID d’APPLICATION) que vous avez enregistré précédemment. Par exemple : `00000000-0000-0000-0000-000000000000`.
* URI de redirection personnalisé que vous avez enregistré précédemment. Par exemple : `com.onmicrosoft.contosob2c.exampleapp://oauth/redirect`.

Les deux éléments doivent avoir été enregistrés là où vous avez [inscrit votre application](#create-an-application).

```java
AuthorizationRequest req = new AuthorizationRequest.Builder(
    config,
    clientId,
    ResponseTypeValues.CODE,
    redirectUri)
    .build();
```

Veuillez vous reporter au [guide d’AppAuth](https://openid.github.io/AppAuth-Android/) pour le reste du processus. Si vous avez besoin de prendre rapidement en main une application, consultez [notre exemple](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c). Suivez les étapes du fichier [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md) pour entrer votre propre configuration Azure AD B2C.