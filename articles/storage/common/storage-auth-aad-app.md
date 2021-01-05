---
title: Obtenir un jeton à partir d’Azure AD pour autoriser les requêtes à partir d’une application cliente
titleSuffix: Azure Storage
description: Utilisez Azure Active Directory pour vous authentifier à partir d’une application client, acquérir un jeton OAuth 2.0 et autoriser les requêtes de Stockage Blob et File d’attente Azure.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/07/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-csharp
ms.openlocfilehash: 6d6a152096ce4e16849542c26d1c7a675a972b89
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779071"
---
# <a name="acquire-a-token-from-azure-ad-for-authorizing-requests-from-a-client-application"></a>Obtenir un jeton à partir d’Azure AD pour autoriser les requêtes à partir d’une application cliente

Le principal avantage d’utiliser Azure Active Directory (Azure AD) avec le Stockage Blob ou File d’attente Azure est que vos informations d’identification n’ont plus besoin d’être stockées dans votre code. Au lieu de cela, vous pouvez demander un jeton d’accès OAuth 2.0 auprès de la plateforme d’identité Microsoft. Azure AD authentifie le principal de sécurité (un utilisateur, un groupe ou un principal de service) qui exécute l’application. Si l’authentification réussit, Azure AD retourne le jeton d’accès à l’application et l’application peut ensuite l’utiliser pour autoriser les demandes vers le Stockage Blob ou File d’attente Azure.

Cet article explique comment configurer l’authentification avec la Plateforme d’identités Microsoft sur une application native ou une application web à l’aide d’un exemple d’application disponible au téléchargement. Ce dernier utilise .NET, mais l’approche est similaire avec d’autres langages. Pour plus d’informations sur la Plateforme d’identités Microsoft, consultez [Vue d’ensemble de la Plateforme d’identités Microsoft](../../active-directory/develop/v2-overview.md).

Pour avoir une vue d’ensemble du flux d’octroi de code OAuth 2.0, consultez [Autoriser l’accès aux applications web Azure Active Directory à l’aide du flux d’octroi de code OAuth 2.0](../../active-directory/develop/v2-oauth2-auth-code-flow.md).

## <a name="about-the-sample-application"></a>À propos de l’exemple d’application

L’exemple d’application offre une expérience de bout en bout montrant comment configurer l’authentification avec Azure AD sur une application web dans un environnement de développement local. Pour voir et exécuter l’exemple d’application, commencez par le cloner ou le télécharger sur [GitHub](https://github.com/Azure-Samples/storage-dotnet-azure-ad-msal). Suivez ensuite la procédure décrite dans l’article pour configurer une inscription d’application Azure et mettre à jour l’application en fonction de votre environnement.

## <a name="assign-a-role-to-an-azure-ad-security-principal"></a>Attribuer un rôle à un principal de sécurité Azure AD

Pour authentifier un principal de sécurité à partir de votre application Stockage Azure, commencez par configurer les paramètres de contrôle d’accès en fonction du rôle Azure (Azure RBAC) de ce principal de sécurité. Le Stockage Azure définit des rôles intégrés qui englobent les autorisations pour les conteneurs et les files d’attente. Quand le rôle Azure est attribué à un principal de sécurité, ce dernier obtient l’accès à cette ressource. Pour plus d’informations, consulter [Gestion des droits d’accès aux données Blob et File d’attente Azure avec Azure RBAC](./storage-auth-aad-rbac-portal.md).

## <a name="register-your-application-with-an-azure-ad-tenant"></a>Inscrire votre application à un locataire Azure AD

La première étape d’utilisation d’Azure AD pour autoriser l’accès aux ressources de stockage est d’inscrire votre application cliente avec un locataire Azure AD à partir du [Portail Microsoft Azure](https://portal.azure.com). Lorsque vous inscrivez votre application cliente, vous fournissez des informations sur l’application à Azure AD. Azure AD fournit ensuite un ID de client (appelé aussi *ID d’application*) que vous utilisez pour associer votre application à Azure AD au moment de l’exécution. Pour en savoir plus sur l’ID de client, consultez [Objets application et principal du service dans Azure Active Directory](../../active-directory/develop/app-objects-and-service-principals.md). Pour inscrire votre application de Stockage Microsoft Azure, veuillez suivre les étapes indiquées dans le [Guide de démarrage rapide : Inscrire une application avec la plateforme des identités Microsoft](../../active-directory/develop/quickstart-configure-app-access-web-apis.md). 

L’illustration suivante montre les paramètres courants d’inscription d’une application web. À savoir : dans cet exemple, l’URI de redirection est défini sur `http://localhost:5000/signin-oidc` pour tester l’exemple d’application dans l’environnement de développement. Vous pourrez par la suite modifier ce paramètre sous le paramètre **Authentification** de votre application inscrite sur le Portail Azure :

:::image type="content" source="media/storage-auth-aad-app/app-registration.png" alt-text="Capture d’écran montrant comment inscrire votre application de stockage dans Azure AD":::

> [!NOTE]
> Si vous inscrivez votre application comme une application native, vous pouvez spécifier n’importe quel URI valide pour **l’URI de redirection**. Pour les applications natives, cette valeur ne devra pas être une URL réelle. Pour les applications web, l’URI de redirection doit être un URI valide, car il spécifie l’URL à laquelle les jetons sont fournis.

Une fois votre application inscrite, l’ID d’application (ou ID de client) se trouve sous **Paramètres** :

:::image type="content" source="media/storage-auth-aad-app/app-registration-client-id.png" alt-text="Capture d’écran montrant l’ID de client":::

Pour plus d’informations sur l’inscription d’une application dans Azure AD, consultez [Intégration d’applications à Azure Active Directory](../../active-directory/develop/quickstart-register-app.md).

### <a name="grant-your-registered-app-permissions-to-azure-storage"></a>Accorder à votre application inscrite des autorisations sur le Stockage Azure

Ensuite, accordez à votre application l’autorisation d’appeler les API de Stockage Azure. Cette étape permet à votre application d’autoriser les requêtes vers le Stockage Azure avec Azure AD.

1. Sur la page **Autorisations d’API** de votre application inscrite, sélectionnez **Ajouter une autorisation**.
1. Sous l’onglet **API Microsoft**, sélectionnez **Stockage Azure**.
1. Comme vous pouvez le constater sous **De quel type d’autorisations votre application a-t-elle besoin ?** dans le volet **Demander des autorisations d’API**, le type d’autorisation disponible est **Autorisations déléguées**. Cette option est sélectionnée pour vous par défaut.
1. Sous **Autorisations**, cochez la case à côté de **user_impersonation**, puis sélectionnez le bouton **Ajouter des autorisations**.

    :::image type="content" source="media/storage-auth-aad-app/registered-app-permissions-1.png" alt-text="Capture d’écran montrant les autorisations relatives à l’API de stockage":::

1. Accordez ensuite le consentement administrateur pour ces autorisations en cliquant sur **Accorder le consentement administrateur pour le répertoire par défaut**.

Le volet **Autorisations d’API** indique à présent que votre application Azure AD inscrite a accès aux API Microsoft Graph et Stockage Azure. Par ailleurs, le consentement est accordé pour le répertoire par défaut. Les autorisations sont accordées à Microsoft Graph automatiquement quand vous inscrivez votre application pour la première fois dans Azure AD.

:::image type="content" source="media/storage-auth-aad-app/registered-app-permissions-2.png" alt-text="Capture d’écran montrant les autorisations d’API de l’application inscrite":::

### <a name="create-a-client-secret"></a>Créer une clé secrète client

L’application a besoin d’une clé secrète client pour prouver son identité lors de la requête de jeton. Pour ajouter une clé secrète client, procédez comme suit :

1. Accédez à votre inscription d’application dans le Portail Microsoft Azure.
1. Sélectionnez le paramètre **Certificats et clés secrètes**.
1. Sous **Clés secrètes client**, cliquez sur **Nouvelle clé secrète client** pour créer une clé secrète.
1. Fournissez une description pour la clé secrète, puis choisissez une intervalle d’expiration.
1. Copiez immédiatement la valeur de la nouvelle clé secrète dans un emplacement sécurisé. La valeur complète s’affiche pour vous une seule fois.

    ![Capture d’écran montrant la clé secrète client](media/storage-auth-aad-app/client-secret.png)

### <a name="enable-implicit-grant-flow"></a>Activer le flux d’octroi implicite

Configurez maintenant le processus d’octroi implicite de votre application. Procédez comme suit :

1. Accédez à votre inscription d’application dans le Portail Microsoft Azure.
1. Dans la section **Gérer**, sélectionnez le paramètre **authentification**.
1. Dans la section **Octroi implicite**, activez la case des jetons d’ID, comme illustré dans l’image suivante :

    :::image type="content" source="media/storage-auth-aad-app/enable-implicit-grant-flow.png" alt-text="Capture d’écran montrant comment activer les paramètres pour le flux d’octroi implicite":::

## <a name="client-libraries-for-token-acquisition"></a>Bibliothèques clientes pour l’acquisition de jeton

Lorsque vous avez inscrit votre application et les autorisations pour accéder aux données dans le Stockage Blob ou File d’attente Azure, vous pouvez ajouter le code à votre application pour authentifier un principal de sécurité et acquérir un jeton OAuth 2.0. Pour authentifier et acquérir le jeton, vous pouvez utiliser l’une des [Bibliothèques d’authentification de plateforme d’identité Microsoft](../../active-directory/develop/reference-v2-libraries.md) ou une autre bibliothèque Open Source prenant en charge OpenID Connect 1.0. Votre application peut alors utiliser le jeton d’accès pour autoriser une requête sur le Stockage Blob ou File d’attente Azure.

Pour connaître la liste de scénarios dans lesquels l’acquisition de jetons est prise en charge, consultez la section [Flux d’authentification](../../active-directory/develop/msal-authentication-flows.md) de la documentation [Bibliothèque d’authentification Microsoft (MSAL)](../../active-directory/develop/msal-overview.md).

## <a name="well-known-values-for-authentication-with-azure-ad"></a>Valeurs connues pour l’authentification avec Azure AD

Pour authentifier un principal de sécurité avec Azure AD, vous devez inclure des valeurs connues dans votre code.

### <a name="azure-ad-authority"></a>Autorité Azure AD

Pour le cloud public Microsoft, l’autorité Azure AD de base est comme suit, où *tenant-id* est votre ID de locataire Active Directory (ou ID de répertoire) :

`https://login.microsoftonline.com/<tenant-id>/`

L’ID client identifie le client Azure AD pour l’authentification. Il est également appelé ID de répertoire. Pour récupérer l’ID de locataire, accédez à la page **Vue d’ensemble** de votre inscription d’application dans le Portail Microsoft Azure et copiez la valeur à partir de là.

### <a name="azure-storage-resource-id"></a>ID de ressource de stockage Azure

[!INCLUDE [storage-resource-id-include](../../../includes/storage-resource-id-include.md)]

## <a name="net-code-example-create-a-block-blob"></a>Exemple de code .NET : Créer un objet blob de blocs

L’exemple de code montre comment obtenir un jeton d’accès d’Azure AD. Le jeton d’accès est utilisé pour authentifier l’utilisateur spécifié et autoriser une demande de création d’objet blob de blocs. Pour que cet exemple fonctionne, suivez d’abord les étapes décrites dans les sections précédentes.

Pour envoyer une requête de jeton, vous aurez besoin des valeurs suivantes lors de l’enregistrement de votre application :

- Le nom de votre client Azure AD. Récupérez cette valeur à partir de la page **Vue d’ensemble** de votre instance Azure Active Directory.
- L’ID du locataire (ou répertoire). Récupérez cette valeur à partir de la page **Vue d’ensemble** de votre inscription d’application.
- L’ID client (ou d’application). Récupérez cette valeur à partir de la page **Vue d’ensemble** de votre inscription d’application.
- L’URI de redirection du client. Récupérez cette valeur à partir du paramètre **Authentification** pour l’inscription de votre application.
- La valeur de la clé secrète client. Récupérez cette valeur à partir de l’emplacement où vous l’avez précédemment copiée.

### <a name="create-a-storage-account-and-container"></a>Créer un compte de stockage et un conteneur

Pour exécuter l’exemple de code, créez un compte de stockage dans le même abonnement que votre instance Azure Active Directory. Puis créez un conteneur dans ce compte de stockage. L’exemple de code crée un objet blob de blocs dans ce conteneur.

Ensuite, affectez explicitement le rôle **Contributeur aux données Blob du stockage** au compte d’utilisateur sous lequel vous allez exécuter l’exemple de code. Pour connaître les instructions à suivre pour attribuer ce rôle sur le Portail Azure, consulter [Attribution d’un rôle Azure pour l’accès à des données Blob et File d’attente avec le Portail Azure](storage-auth-aad-rbac-portal.md).

> [!NOTE]
> Lorsque vous créez un compte de stockage Azure, aucune autorisation d’accès aux données ne vous est automatiquement attribuée via Azure AD. Vous devez vous attribuer explicitement un rôle Azure pour le Stockage Azure. Vous pouvez l’attribuer au niveau de votre abonnement, groupe de ressources, compte de stockage, conteneur ou file d’attente.
>
> Avant de vous attribuer un rôle pour l’accès aux données, vous pouvez accéder aux données de votre compte de stockage via le portail Azure, car ce dernier peut également utiliser la clé de compte pour l’accès aux données. Pour plus d’informations, consultez [Choisir comment autoriser l’accès à des données blobs dans le portail Azure](../blobs/authorize-data-operations-portal.md).

### <a name="create-a-web-application-that-authorizes-access-to-blob-storage-with-azure-ad"></a>Créer une application web qui autorise l’accès au Stockage Blob avec Azure AD

Lorsque votre application accède au stockage Azure, elle le fait pour le compte de l’utilisateur, ce qui signifie que les ressources blob ou de file d’attente sont accessibles en utilisant les autorisations de l’utilisateur qui s’est connecté. Pour tester cet exemple de code, vous avez besoin d’une application web qui invite l’utilisateur à se connecter à l’aide d’une identité Azure AD. Vous pouvez créer la vôtre, ou utiliser l’exemple d’application fourni par Microsoft.

Un exemple d’application web complet qui acquiert un jeton et l’utilise pour créer un objet blob dans le Stockage Azure est disponible sur [GitHub](https://aka.ms/aadstorage). Nous vous conseillons d’examiner et d’exécuter l’exemple complet pour mieux comprendre les exemples de code. Pour obtenir des instructions sur la façon d’exécuter l’exemple complet, veuillez consultez la section intitulée [Afficher et exécuter l’exemple complet](#view-and-run-the-completed-sample).

#### <a name="add-references-and-using-statements"></a>Ajouter des références et des instructions using  

Dans Visual Studio, installez la bibliothèque de client du Stockage Azure. Dans le menu **Outils**, sélectionnez **Gestionnaire de package NuGet**, puis **Console du gestionnaire de package**. Tapez les commandes suivantes dans la fenêtre de la console pour installer les packages nécessaires à partir de la bibliothèque de client du Stockage Azure pour .NET :

# <a name="net-v12-sdk"></a>[Kit de développement logiciel (SDK) .NET v12](#tab/dotnet)

```console
Install-Package Azure.Storage.Blobs
Install-Package Microsoft.Identity.Web -Version 0.4.0-preview
```

Ensuite, ajoutez le code suivant à l’aide des instructions sur le fichier HomeController.cs :

```csharp
using Microsoft.Identity.Web; //MSAL library for getting the access token
using Azure.Storage.Blobs;
```

# <a name="net-v11-sdk"></a>[Kit de développement logiciel (SDK) .NET v11](#tab/dotnet11)

```console
Install-Package Microsoft.Azure.Storage.Blob
Install-Package Microsoft.Identity.Web -Version 0.4.0-preview //or a later version
```

Ensuite, ajoutez le code suivant à l’aide des instructions sur le fichier HomeController.cs :

```csharp
using Microsoft.Identity.Client; //MSAL library for getting the access token
using Microsoft.Azure.Storage.Auth;
using Microsoft.Azure.Storage.Blob;
```

---

#### <a name="create-a-block-blob"></a>Créer un objet blob de blocs

Ajoutez l’extrait de code suivant pour créer un objet blob de blocs. N’oubliez pas de remplacer les valeurs entre crochets par vos propres valeurs :

# <a name="net-v12-sdk"></a>[Kit de développement logiciel (SDK) .NET v12](#tab/dotnet)

```csharp
private static async Task<string> CreateBlob(TokenAcquisitionTokenCredential tokenCredential)
{
    Uri blobUri = new Uri("https://<storage-account>.blob.core.windows.net/<container>/Blob1.txt");
    BlobClient blobClient = new BlobClient(blobUri, tokenCredential);

    string blobContents = "Blob created by Azure AD authenticated user.";
    byte[] byteArray = Encoding.ASCII.GetBytes(blobContents);

    using (MemoryStream stream = new MemoryStream(byteArray))
    {
        await blobClient.UploadAsync(stream);
    }
    return "Blob successfully created";
}
```

# <a name="net-v11-sdk"></a>[Kit de développement logiciel (SDK) .NET v11](#tab/dotnet11)

```csharp
private static async Task<string> CreateBlob(string accessToken)
{
    // Create a blob on behalf of the user.
    TokenCredential tokenCredential = new TokenCredential(accessToken);
    StorageCredentials storageCredentials = new StorageCredentials(tokenCredential);

    // Replace the URL below with the URL to your blob.
    Uri blobUri = new Uri("https://<storage-account>.blob.core.windows.net/<container>/Blob1.txt");
    CloudBlockBlob blob = new CloudBlockBlob(blobUri, storageCredentials);
    await blob.UploadTextAsync("Blob created by Azure AD authenticated user.");
    return "Blob successfully created";
}
```

---

> [!NOTE]
> Pour autoriser les opérations d’objet blob et de file d’attente avec un jeton OAuth 2.0, vous devez utiliser HTTPS.

Dans l’exemple ci-dessus, la bibliothèque cliente .NET gère l’autorisation de la demande de création de l’objet blob de blocs. D’autres bibliothèques clientes du Stockage Azure pour d’autres langages gèrent également l’autorisation de la requête pour vous. Toutefois, si vous appelez une opération Stockage Azure avec un jeton OAuth à l’aide de l’API REST, vous devez construire l’en-tête **Autorisation** à l’aide du jeton OAuth.

Pour appeler les opérations des services Blob et de File d’attente à l’aide des jetons d’accès OAuth, passez le jeton d’accès dans l’en-tête **Authorization** en utilisant le schéma **Bearer** et spécifiez une version de service 2017-11-09 ou ultérieure, comme indiqué dans l’exemple suivant :

```https
GET /container/file.txt HTTP/1.1
Host: mystorageaccount.blob.core.windows.net
x-ms-version: 2017-11-09
Authorization: Bearer eyJ0eXAiOnJKV1...Xd6j
```

#### <a name="get-an-access-token-from-azure-ad"></a>Obtenir un jeton d’accès à partir d’Azure AD

Ensuite, ajoutez une méthode qui demande un jeton à Azure AD pour le compte de l’utilisateur. Cette méthode définit l’étendue pour laquelle les autorisations doivent être accordées. Vous trouverez plus d’informations sur les autorisations et les étendues dans [Autorisations et consentement dans le point de terminaison de la plateforme d’identités Microsoft](../../active-directory/develop/v2-permissions-and-consent.md).

Utilisez l’ID de ressource pour construire l’étendue pour laquelle acquérir le jeton. L’exemple construit l’étendue à l’aide de l’ID de ressource avec l’étendue `user_impersonation` intégrée, ce qui indique que le jeton est demandé pour le compte de l’utilisateur.

Gardez à l’esprit que vous devrez peut-être présenter à l’utilisateur une interface lui permettant de donner son consentement pour la demande de jeton en son nom :

```csharp
[AuthorizeForScopes(Scopes = new string[] { "https://storage.azure.com/user_impersonation" })]
public async Task<IActionResult> Blob()
{
    string message = await CreateBlob(new TokenAcquisitionTokenCredential(_tokenAcquisition));
    ViewData["Message"] = message;
    return View();
}
```

Le consentement est le processus via lequel un utilisateur autorise une application à accéder à des ressources protégées en son nom. La Plateforme d’identités Microsoft prend en charge le consentement incrémentiel. En d’autres termes, un principal de sécurité peut commencer par demander un ensemble d’autorisations minimal, puis ajouter des autorisations au fil du temps en fonction des besoins. Lorsque votre code demande un jeton d’accès, spécifiez l’étendue des autorisations dont votre application a besoin. Pour plus d’informations sur le consentement incrémentiel, consultez [Consentement incrémentiel et dynamique](../../active-directory/azuread-dev/azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent).

## <a name="view-and-run-the-completed-sample"></a>Afficher et exécuter l’exemple complet

Pour exécuter l’exemple d’application, commencez par le cloner ou le télécharger depuis [GitHub](https://github.com/Azure-Samples/storage-dotnet-azure-ad-msal). Puis mettez à jour l’application comme décrit dans les sections suivantes.

### <a name="provide-values-in-the-settings-file"></a>Fournir des valeurs dans le fichier de paramètres

Mettez à jour le fichier *appsettings.json* avec vos propres valeurs :

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "<azure-ad-domain-name>.onmicrosoft.com",
    "TenantId": "<tenant-id>",
    "ClientId": "<client-id>",
    "ClientSecret": "<client-secret>"
    "ClientCertificates": [
    ],
    "CallbackPath": "/signin-oidc"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

### <a name="update-the-storage-account-and-container-name"></a>Mettre à jour le conteneur et le nom du compte de stockage

Dans le fichier *HomeController.cs*, mettez à jour l’URI qui fait référence à l’objet blob de blocs pour utiliser votre conteneur et le nom de votre compte de stockage, en remplaçant les valeurs entre crochets par vos propres valeurs :

```html
https://<storage-account>.blob.core.windows.net/<container>/Blob1.txt
```

## <a name="next-steps"></a>Étapes suivantes

- [Plateforme d’identité Microsoft](../../active-directory/develop/index.yml)
- [Gestion des droits d’accès aux données de stockage avec Azure RBAC](./storage-auth-aad-rbac-portal.md)
- [Authenticate access to blobs and queues with Azure Active Directory and managed identities for Azure Resources](storage-auth-aad-msi.md) (Authentifier l’accès aux objets blob et files d’attente avec Azure Active Directory et les identités managées pour les ressources Azure)