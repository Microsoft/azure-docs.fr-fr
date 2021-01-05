---
title: Authentification et autorisation
description: Découvrez les différentes façons par lesquelles une application ou un service peut s’authentifier auprès d’Azure Spatial Anchors, ainsi que les niveaux de contrôle dont vous disposez pour réguler l’accès à Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: 0166a3b6031f9e1d364a37db99be5bc5a65267df
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95484608"
---
# <a name="authentication-and-authorization-to-azure-spatial-anchors"></a>Autorisation et authentification auprès d’Azure Spatial Anchors

Dans cet article, vous allez découvrir les différentes façons dont vous pouvez vous authentifier auprès des Azure Spatial Anchors à partir de votre application ou service web. Vous allez également découvrir comment utiliser le contrôle d’accès en fonction du rôle Azure (Azure RBAC) dans Azure Active Directory (Azure AD) pour contrôler l’accès à vos comptes Spatial Anchors.

## <a name="overview"></a>Vue d’ensemble

![Diagramme montrant une vue d’ensemble de l’authentification auprès d’Azure Spatial Anchors.](./media/spatial-anchors-authentication-overview.png)

Pour accéder à un compte Azure Spatial Anchors donné, les clients doivent d’abord obtenir un jeton d’accès à partir du service d’émission de jeton de sécurité (STS, Security Token Service) Azure Mixed Reality. Les jetons obtenus de STS ont une durée de vie de 24 heures. Ils contiennent des informations permettant aux services Spatial Anchors de prendre des décisions d’autorisation sur le compte. Ils garantissent que seuls les principaux autorisés peuvent accéder à ce compte.

Les jetons d’accès peuvent être obtenus en échange d’une clé de compte ou de jetons émis par AD Azure.

Les clés de compte vous permettent de commencer rapidement à utiliser le service Azure Spatial Anchors. Mais avant de déployer votre application en production, nous vous recommandons de mettre à jour votre application pour utiliser l’authentification Azure AD.

Vous pouvez obtenir des jetons d’authentification Azure AD de deux manières :

- Si vous créez une application d’entreprise et que votre entreprise utilise Azure AD comme système d’identité, vous pouvez utiliser l’authentification Azure AD basée sur l’utilisateur dans votre application. Vous pouvez ensuite accorder l’accès à vos comptes Spatial Anchors à l’aide de vos groupes de sécurité Azure AD existants. Vous pouvez également accorder l’accès directement aux utilisateurs de votre organisation.
- Sinon, nous vous recommandons d’obtenir des jetons Azure AD à partir d’un service web prenant en charge votre application. Nous vous recommandons d’utiliser cette méthode pour les applications de production, car cela vous permet d’éviter d’incorporer les informations d’identification pour l’accès à Azure Spatial Anchors dans votre application cliente.

## <a name="account-keys"></a>Clés de compte

Le recours à des clés de compte pour l’accès à votre compte Azure Spatial Anchors est la façon la plus simple de bien démarrer. Vous trouverez vos clés de compte dans le portail Azure. Accédez à votre compte, puis sélectionnez l’onglet **Clés** :

![Capture d’écran montrant l’onglet « Clés » avec le bouton « Copier » de la « Clé primaire » mis en évidence.](../../../includes/media/spatial-anchors-get-started-create-resource/view-account-key.png)

Deux clés sont disponibles. Les deux sont valides simultanément pour l’accès au compte Spatial Anchors. Nous vous recommandons de mettre à jour régulièrement la clé que vous utilisez pour accéder au compte. Le fait d’avoir deux clés valides distinctes active ces mises à jour sans temps d’arrêt. Il vous suffit de mettre à jour la clé primaire et la clé secondaire.

Le kit de développement logiciel (SDK) offre une prise en charge intégrée de l’authentification via les clés de compte. Vous devez simplement définir la propriété `AccountKey` sur votre objet `cloudSession` :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AccountKey = @"MyAccountKey";
```

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
_cloudSession.configuration.accountKey = @"MyAccountKey";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.accountKey = "MyAccountKey"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAccountKey("MyAccountKey");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AccountKey(R"(MyAccountKey)");
```

# <a name="cwinrt"></a>[C++/WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AccountKey(LR"(MyAccountKey)");
```

---

Une fois cette propriété définie, le SDK gère l’échange de la clé de compte contre un jeton d’accès, et la mise en cache nécessaire des jetons pour votre application.

> [!WARNING]
> Nous vous recommandons d’utiliser des clés de compte pour une intégration rapide, mais uniquement pendant le développement/prototypage. Nous vous déconseillons d’envoyer votre application en production avec une clé de compte incorporée. Utilisez plutôt les approches d’authentification Azure AD basées sur l’utilisateur ou sur les services décrites ci-dessous.

## <a name="azure-ad-user-authentication"></a>Authentification utilisateur Azure AD

Pour les applications qui ciblent des utilisateurs d’Azure Active Directory, nous vous recommandons d’utiliser un jeton Azure AD pour l’utilisateur. Vous pouvez obtenir ce jeton à l’aide de la [MSAL](../../active-directory/develop/msal-overview.md). Suivez les étapes décrites dans le [Guide de démarrage rapide pour l’inscription d’une application](../../active-directory/develop/quickstart-register-app.md), qui comprend :

**Dans le portail Azure**
1.    Inscrivez votre application dans Azure AD en tant qu’application native. Dans le cadre de l’inscription, vous devez déterminer si votre application doit être multi-locataire. Vous devez également fournir les URL de redirection autorisées pour votre application.
1.  Accéder à l’onglet **Autorisations de l’API**.
2.  Sélectionnez **Ajouter une autorisation**.
    1.  Sélectionner **Fournisseur de ressources Azure Mixed Reality** dans l’onglet **API utilisées par mon organisation**.
    2.  Sélectionnez **Autorisations déléguées**.
    3.  Sélectionnez **mixedreality.signin** sous **mixedreality**.
    4.  Sélectionnez **Ajouter des autorisations**.
3.  Sélectionner **Accorder le consentement administrateur**.

2. Accordez à votre application ou aux utilisateurs l’accès à votre ressource :
   1.    Accédez à votre ressource Spatial Anchors dans le portail Azure.
   2.    Accédez à l’onglet **Contrôle d’accès (IAM)** .
   3.    Sélectionnez **Ajouter une attribution de rôle**.
   1.    [Sélectionnez un rôle](#azure-role-based-access-control).
   2.    Dans la zone **Sélectionner**, entrez le nom des utilisateurs, groupes et/ou applications auxquels vous souhaitez accorder l’accès.
   3.    Sélectionnez **Enregistrer**.

**Dans votre code**
1.    Veillez à utiliser l’ID d’application et l’URI de redirection de votre propre application Azure AD comme paramètres d’**ID client** et d’**Uri de redirection** dans la bibliothèque MSAL.
2.    Définissez les informations sur le locataire :
        1.    Si votre application prend en charge **Mon organisation uniquement**, remplacez cette valeur par l’**ID de locataire** ou le **Nom du locataire**. Par exemple « contoso.microsoft.com ».
        2.    Si votre application prend en charge les **comptes dans un annuaire organisationnel**, remplacez cette valeur par **Organizations**.
        3.    Si votre application prend en charge **Tous les utilisateurs de compte Microsoft**, remplacez cette valeur par **Common**.
3.    Sur votre demande de jeton, définissez l’**étendue** sur **« `https://sts.<account-domain>//.default` »** , où `<account-domain>` est remplacé par le **domaine de compte** de votre compte Azure Spatial Anchors. **« `https://sts.mixedreality.azure.com//.default` »** est un exemple d’étendue pour un compte Azure Spatial Anchors dans le domaine de compte USA Est 2. Cette étendue indique à Azure AD que votre application demande un jeton pour le service d’émission de jeton de sécurité (STS) Mixed Reality.

Une fois ces étapes effectuées, votre application devrait être en mesure d’obtenir un jeton Azure AD à partir de MSAL. Vous pouvez définir cette jeton Azure AD comme `authenticationToken` sur votre objet de configuration de session cloud :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AuthenticationToken = @"MyAuthenticationToken";
```

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
_cloudSession.configuration.authenticationToken = @"MyAuthenticationToken";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.authenticationToken = "MyAuthenticationToken"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAuthenticationToken("MyAuthenticationToken");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AuthenticationToken(R"(MyAuthenticationToken)");
```

# <a name="cwinrt"></a>[C++/WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AuthenticationToken(LR"(MyAuthenticationToken)");
```

---

## <a name="azure-ad-service-authentication"></a>Authentification auprès du service Azure AD

Pour déployer des applications qui utilisent Azure Spatial Anchors en production, nous vous recommandons d’utiliser un service principal qui effectuera les demandes d’authentification. Voici une vue d’ensemble du processus :

![Diagramme fournissant une vue d’ensemble de l’authentification auprès d’Azure Spatial Anchors.](./media/spatial-anchors-aad-authentication.png)

Ici, il est supposé que votre application utilise son propre mécanisme pour s’authentifier auprès de son service principal. (Par exemple, un compte Microsoft, PlayFab, Facebook, un ID Google ou un nom d’utilisateur et un mot de passe personnalisés.)  Une fois que vos utilisateurs sont authentifiés auprès de votre service back-end, celui-ci peut récupérer un jeton Azure AD, l’échanger contre un jeton d’accès pour Azure Spatial Anchors, et le renvoyer à votre application cliente.

Le jeton d’accès Azure AD est récupéré à l’aide de [MSAL](../../active-directory/develop/msal-overview.md). Suivez les étapes indiquées dans le [guide de démarrage rapide Inscrire une application](../../active-directory/develop/quickstart-register-app.md), notamment :

**Dans le portail Azure**
1.    Inscrivez votre application dans Azure AD :
        1.    Dans le portail Azure, sélectionnez **Azure Active Directory**, puis **Inscriptions d’applications**.
        2.    Sélectionnez **Nouvelle inscription**.
        3.    Entrez le nom de votre application, sélectionnez **Application/API web** comme type d’application, puis entrez l’URL d’authentification pour votre service. Sélectionnez **Create** (Créer).
2.    Dans cette application, sélectionnez **Paramètres**, puis l’onglet **Certificats et secrets**. Créez une clé secrète client, sélectionnez une durée, puis **Ajouter**. Veillez à enregistrer la valeur secrète. Vous devez l’inclure dans le code de votre service web.
3.    Accordez à votre application et/ou aux utilisateurs l’accès à votre ressource :
        1.    Accédez à votre ressource Spatial Anchors dans le portail Azure.
        2.    Accédez à l’onglet **Contrôle d’accès (IAM)** .
        3.    Sélectionnez **Ajouter une attribution de rôle**.
        4.    [Sélectionnez un rôle](#azure-role-based-access-control).
        5.    Dans la zone **Sélectionner**, entrez les noms des applications auxquelles vous souhaitez affecter l’accès. Si vous voulez que les utilisateurs de votre application aient différents rôles sur le compte Spatial Anchors, inscrivez plusieurs applications dans Azure AD et attribuez à chacune un rôle distinct. Ensuite, implémentez votre logique d’autorisation afin d’utiliser le rôle approprié pour vos utilisateurs.

              > [!NOTE]
              > Dans le volet **Ajouter une attribution de rôle**, dans **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou principal de service Azure AD**.

        6.    Sélectionnez **Enregistrer**.

**Dans votre code**

>[!NOTE]
> Vous pouvez utiliser l’exemple de service disponible sur GitHub.

1.    Veillez à utiliser l’ID d’application, le secret d’application et l’URI de redirection de votre propre application Azure AD comme paramètres d’**ID client**, de **secret** et d’**URI de redirection** dans la bibliothèque MSAL.
2.    Définissez comme ID de locataire votre propre ID de locataire Azure AD dans le paramètre d’**autorité** dans la bibliothèque MSAL.
3.    Sur votre demande de jeton, définissez l’**étendue** sur **« `https://sts.<account-domain>//.default` »** , où `<account-domain>` est remplacé par le **domaine de compte** de votre compte Azure Spatial Anchors. **« `https://sts.mixedreality.azure.com//.default` »** est un exemple d’étendue pour un compte Azure Spatial Anchors dans le domaine de compte USA Est 2.

Une fois ces étapes effectuées, votre service principal peut récupérer un jeton Azure AD. Il peut ensuite l’échanger contre un jeton MR qu’il renverra au client. L’utilisation d’un jeton Azure AD pour récupérer un jeton MR s’effectue par le biais d’un appel REST. Voici un exemple d’appel :

```
GET https://sts.mixedreality.azure.com/Accounts/35d830cb-f062-4062-9792-d6316039df56/token HTTP/1.1
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni<truncated>FL8Hq5aaOqZQnJr1koaQ
Host: sts.mixedreality.azure.com
Connection: Keep-Alive

HTTP/1.1 200 OK
Date: Sun, 24 Feb 2019 08:00:00 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 1153
Accept: application/json
MS-CV: 05JLqWeKFkWpbdY944yl7A.0
{"AccessToken":"eyJhbGciOiJSUzI1NiIsImtpZCI6IjI2MzYyMTk5ZTI2NjQxOGU4ZjE3MThlM2IyMThjZTIxIiwidHlwIjoiSldUIn0.eyJqdGkiOiJmMGFiNWIyMy0wMmUxLTQ1MTQtOWEzNC0xNzkzMTA1NTc4NzAiLCJjYWkiOiIzNWQ4MzBjYi1mMDYyLTQwNjItOTc5Mi1kNjMxNjAzOWRmNTYiLCJ0aWQiOiIwMDAwMDAwMC0wMDAwLTAwMDAtMDAwMC0wMDAwMDAwMDAwMDAiLCJhaWQiOiIzNWQ4MzBjYi1mMDYyLTQwNjItOTc5Mi1kNjMxNjAzOWRmNTYiLCJhYW8iOi0xLCJhcHIiOiJlYXN0dXMyIiwicmlkIjoiL3N1YnNjcmlwdGlvbnMvNzIzOTdlN2EtNzA4NC00ODJhLTg3MzktNjM5Y2RmNTMxNTI0L3Jlc291cmNlR3JvdXBzL3NhbXBsZV9yZXNvdXJjZV9ncm91cC9wcm92aWRlcnMvTWljcm9zb2Z0Lk1peGVkUmVhbGl0eS9TcGF0aWFsQW5jaG9yc0FjY291bnRzL2RlbW9fYWNjb3VudCIsIm5iZiI6MTU0NDU0NzkwMywiZXhwIjoxNTQ0NjM0MzAzLCJpYXQiOjE1NDQ1NDc5MDMsImlzcyI6Imh0dHBzOi8vbXJjLWF1dGgtcHJvZC50cmFmZmljbWFuYWdlci5uZXQvIiwiYXVkIjoiaHR0cHM6Ly9tcmMtYW5jaG9yLXByb2QudHJhZmZpY21hbmFnZXIubmV0LyJ9.BFdyCX9UJj0i4W3OudmNUiuaGgVrlPasNM-5VqXdNAExD8acFJnHdvSf6uLiVvPiQwY1atYyPbOnLYhEbIcxNX-YAfZ-xyxCKYb3g_dbxU2w8nX3zDz_X3XqLL8Uha-rkapKbnNgxq4GjM-EBMCill2Svluf9crDmO-SmJbxqIaWzLmlUufQMWg_r8JG7RLseK6ntUDRyDgkF4ex515l2RWqQx7cw874raKgUO4qlx0cpBAB8cRtGHC-3fA7rZPM7UQQpm-BC3suXqRgROTzrKqfn_g-qTW4jAKBIXYG7iDefV2rGMRgem06YH_bDnpkgUa1UgJRRTckkBuLkO2FvA"}
```

L’en-tête Authorization est mis en forme comme suit : `Bearer <Azure_AD_token>`

La réponse contient le jeton MR en texte brut.

Ce jeton MR est ensuite renvoyé au client. Votre application cliente peut ensuite le définir comme son jeton d’accès dans la configuration de session cloud :

# <a name="c"></a>[C#](#tab/csharp)

```csharp
this.cloudSession.Configuration.AccessToken = @"MyAccessToken";
```

# <a name="objective-c"></a>[Objective-C](#tab/objc)

```objc
_cloudSession.configuration.accessToken = @"MyAccessToken";
```

# <a name="swift"></a>[Swift](#tab/swift)

```swift
_cloudSession!.configuration.accessToken = "MyAccessToken"
```

# <a name="java"></a>[Java](#tab/java)

```java
mCloudSession.getConfiguration().setAccessToken("MyAccessToken");
```

# <a name="c-ndk"></a>[C++ NDK](#tab/cpp)

```cpp
auto configuration = cloudSession_->Configuration();
configuration->AccessToken(R"(MyAccessToken)");
```

# <a name="cwinrt"></a>[C++/WinRT](#tab/cppwinrt)

```cpp
auto configuration = m_cloudSession.Configuration();
configuration.AccessToken(LR"(MyAccessToken)");
```

---

## <a name="azure-role-based-access-control"></a>Contrôle d'accès en fonction du rôle Azure

Pour vous aider à contrôler le niveau d’accès accordé aux applications, services ou utilisateurs Azure AD de votre service, les rôles suivants ont été créés. Vous pouvez affecter ces rôles préexistants en fonction des besoins sur vos comptes Azure Spatial Anchors :

- **Propriétaire de compte Spatial Anchors**. Les applications ou utilisateurs qui détiennent ce rôle peuvent créer des points d’ancrage spatial, les interroger et les supprimer. Quand vous vous authentifiez auprès de votre compte à l’aide de clés de compte, le rôle Propriétaire de compte Points d’ancrage Spatial est attribué au principal authentifié.
- **Contributeur de compte Spatial Anchors**. Les applications ou utilisateurs qui détiennent ce rôle peuvent créer des points d’ancrage spatial et les interroger, mais ils ne peuvent pas les supprimer.
- **Lecteur de compte Spatial Anchors**. Les applications ou utilisateurs qui détiennent ce rôle peuvent uniquement interroger les points d’ancrage spatial. Ils ne peuvent pas en créer de nouveaux, en supprimer ou mettre à jour leurs métadonnées. Ce rôle est utilisé généralement pour les applications où certains utilisateurs gèrent l’environnement, mais d’autres peuvent uniquement rappeler les points d’ancrage précédemment placés dans l’environnement.

## <a name="next-steps"></a>Étapes suivantes

Créez votre première application avec Azure Spatial Anchors :

> [!div class="nextstepaction"]
> [Unity (HoloLens)](../quickstarts/get-started-unity-hololens.md)

> [!div class="nextstepaction"]
> [Unity (iOS)](../quickstarts/get-started-unity-ios.md)

> [!div class="nextstepaction"]
> [Unity (Android)](../quickstarts/get-started-unity-android.md)

> [!div class="nextstepaction"]
> [iOS](../quickstarts/get-started-ios.md)

> [!div class="nextstepaction"]
> [Android](../quickstarts/get-started-android.md)

> [!div class="nextstepaction"]
> [HoloLens](../quickstarts/get-started-hololens.md)

> [!div class="nextstepaction"]
> [Xamarin (Android)](../quickstarts/get-started-xamarin-android.md)

> [!div class="nextstepaction"]
> [Xamarin (iOS)](../quickstarts/get-started-xamarin-ios.md)
