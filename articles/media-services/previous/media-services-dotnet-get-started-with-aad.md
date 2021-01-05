---
title: Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec .NET | Microsoft Docs
description: Cette rubrique aborde l’utilisation de l’authentification Azure Active Directory (Azure AD) en vue d’accéder à l’API Azure Media Services (AMS) avec .NET.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: cca17aacc914412d34f613adfeba31617c60c455
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652940"
---
# <a name="use-azure-ad-authentication-to-access-azure-media-services-api-with-net"></a>Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

À partir de windowsazure.mediaservices 4.0.0.4, Azure Media Services prend en charge l’authentification basée sur Azure Active Directory (Azure AD). Cette rubrique aborde l’utilisation de l’authentification Azure AD en vue d’accéder à l’API Azure Media Services avec Microsoft .NET.

## <a name="prerequisites"></a>Conditions préalables requises

- Un compte Azure. Pour plus d’informations, consultez la page [Version d’évaluation gratuite d’Azure](https://azure.microsoft.com/pricing/free-trial/).
- Un compte Media Services. Pour plus d’informations, voir [Création d’un compte Azure Media Services à l’aide du portail Azure](media-services-portal-create-account.md).
- La dernière version du package [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices).
- Pensez à consulter [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Lorsque vous utilisez l’authentification Azure AD avec Azure Media Services, vous pouvez vous authentifier de deux manières :

- L’**authentification utilisateur** authentifie une personne qui utilise l’application pour interagir avec les ressources Azure Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Par exemple, une application de console de gestion peut être utilisée par les utilisateurs autorisés pour contrôler les travaux d’encodage ou de streaming en direct.
- L’**authentification de principal de service** authentifie un service. Les applications qui utilisent généralement cette méthode d’authentification sont des applications qui exécutent des services démon, des services de niveau intermédiaire ou des travaux planifiés (par exemple, applications web, applications de fonction, applications logiques, API ou microservices).

>[!IMPORTANT]
>Azure Media Services prend actuellement en charge un modèle d’authentification Azure Access Control Service. Toutefois, l’autorisation Access Control sera déconseillée à compter du 22 juin 222018. Nous vous recommandons de migrer vers un modèle d’authentification Azure Active Directory dès que possible.

## <a name="get-an-azure-ad-access-token"></a>Obtenir un jeton d’accès Azure AD

Pour que vous puissiez vous connecter à l’API Azure Media Services avec l’authentification Azure AD, l’application cliente doit demander un jeton d’accès Azure AD. Lorsque vous utilisez le kit de développement logiciel (SDK) client Media Services pour .NET, beaucoup de détails sur la façon d’acquérir un jeton d’accès Azure AD sont encapsulées et simplifiés dans les classes [AzureAdTokenProvider](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenProvider.cs) et [AzureAdTokenCredentials](https://github.com/Azure/azure-sdk-for-media-services/blob/dev/src/net/Client/Common/Common.Authentication/AzureAdTokenCredentials.cs).

Par exemple, vous n’avez pas besoin d’indiquer l’autorité Azure AD, l’URI de ressource Media Services, ni les détails de l’application Azure AD native. Il s’agit de valeurs connues qui sont déjà configurées par la classe de fournisseur de jetons d’accès Azure AD.

Si vous n’utilisez pas le kit de développement logiciel (SDK) Azure Media Services pour .NET, nous vous recommandons d’utiliser la [bibliothèque d’authentification Azure AD](../../active-directory/azuread-dev/active-directory-authentication-libraries.md). Pour obtenir les valeurs des paramètres à utiliser avec la bibliothèque d’authentification Azure AD, voir [Use the Azure portal to access Azure AD authentication settings](media-services-portal-get-started-with-aad.md) (Utiliser le portail Azure pour accéder aux paramètres d’authentification Azure AD).

Vous avez également la possibilité de remplacer l’implémentation par défaut de **AzureAdTokenProvider** par votre propre implémentation.

## <a name="install-and-configure-azure-media-services-net-sdk"></a>Installer et configurer le kit de développement logiciel (SDK) Azure Media Services pour .NET

>[!NOTE]
>Pour utiliser l’authentification Azure AD avec le kit de développement logiciel (SDK) Media Services pour .NET, vous devez disposer de la dernière version du package [NuGet](https://www.nuget.org/packages/windowsazure.mediaservices). En outre, ajoutez une référence à l’assembly **Microsoft.IdentityModel.Clients.ActiveDirectory**. Si vous utilisez une application existante, incluez l’assembly **Microsoft.WindowsAzure.MediaServices.Client.Common.Authentication.dll**.

1. Créez une application console C# dans Visual Studio.
2. Utilisez le package NuGet [windowsazure.mediaservices](https://www.nuget.org/packages/windowsazure.mediaservices) pour installer le **kit de développement logiciel (SDK) Azure Media Services pour .NET**.

    Pour ajouter des références à l’aide de NuGet, procédez comme suit : dans l’**Explorateur de solutions**, cliquez avec le bouton droit de la souris sur le nom du projet, puis sélectionnez **Gérer les packages NuGet**. Ensuite, recherchez **windowsazure.mediaservices** et cliquez sur **Installer**.

    -ou-

    Dans la **Console du Gestionnaire de package** de Visual Studio, exécutez la commande suivante.

    ```console
    Install-Package windowsazure.mediaservices -Version 4.0.0.4
    ```

3. Ajoutez **using** à votre code source.

    ```csharp
    using Microsoft.WindowsAzure.MediaServices.Client;
    ```

## <a name="use-user-authentication"></a>Utiliser une authentification utilisateur

Pour vous connecter à l’API Azure Media Services avec l’option d’authentification utilisateur, l’application cliente doit demander un jeton Azure AD à l’aide des paramètres suivants :

- Point de terminaison de locataire Azure AD. Les informations relatives au locataire peuvent être récupérées à partir du portail Azure. Pointez sur l’utilisateur connecté dans le coin supérieur droit.
- URI de ressource Media Services.
- ID client d’application Media Services (natif).
- URI de redirection d’application Media Services (natif).

Les valeurs de ces paramètres sont accessibles dans **AzureEnvironments.AzureCloudEnvironment**. La constante **AzureEnvironments.AzureCloudEnvironment** est une application auxiliaire du kit de développement logiciel (SDK) pour .NET qui permet d’obtenir les paramètres de variable d’environnement corrects pour un centre de données Azure public.

Elle contient des paramètres d’environnement prédéfinis pour accéder à Media Services dans les centres de données publics uniquement. Pour les régions de cloud souverain ou de cloud pour le secteur public, vous pouvez utiliser respectivement **AzureChinaCloudEnvironment**, **AzureUsGovernmentEnvironment** ou **AzureGermanCloudEnvironment**.

L’exemple de code suivant permet de créer un jeton :

```csharp
var tokenCredentials = new AzureAdTokenCredentials("microsoft.onmicrosoft.com", AzureEnvironments.AzureCloudEnvironment);
var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
```

Pour commencer à programmer sur Media Services, vous devez créer une instance **CloudMediaContext** qui représente le contexte du serveur. **CloudMediaContext** comprend des références à des collections importantes comme les travaux, les éléments multimédias, les fichiers, les stratégies d'accès et les localisateurs.

Vous devez également transmettre l’**URI de ressource des services REST Media** au constructeur **CloudMediaContext**. Pour obtenir l’URI de ressource des services REST Media, connectez-vous au portail Azure, indiquez votre compte Azure Media Services, sélectionnez **Accès API**, puis choisissez l’option **Se connecter à l'API Azure Media Services avec l'authentification utilisateur**.

L’exemple de code suivant permet de créer une instance **CloudMediaContext** :

```csharp
CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
```

L’exemple suivant montre comment créer le jeton Azure AD et le contexte :

```csharp
namespace AzureADAuthSample
{
    class Program
    {
        static void Main(string[] args)
        {
            // Specify your Azure AD tenant domain, for example "microsoft.onmicrosoft.com".
            var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}", AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".
            CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);

            var assets = context.Assets;
            foreach (var a in assets)
            {
                Console.WriteLine(a.Name);
            }
        }
    }
}
```

>[!NOTE]
>Si vous obtenez une exception indiquant que le serveur distant a renvoyé une erreur 401 « Non autorisé », consultez la section sur le [contrôle d’accès](media-services-use-aad-auth-to-access-ams-api.md#access-control) dans la vue d’ensemble de l’accès à l’API Azure Media Services avec l’authentification Azure AD.

## <a name="use-service-principal-authentication"></a>Utiliser une authentification de principal de service

Pour vous connecter à l’API Azure Media Services à l’aide de l’option de principal de service, votre application de niveau intermédiaire (API web ou application web) doit demander un jeton Azure AD avec les paramètres suivants :

- Point de terminaison de locataire Azure AD. Les informations relatives au locataire peuvent être récupérées à partir du portail Azure. Pointez sur l’utilisateur connecté dans le coin supérieur droit.
- URI de ressource Media Services.
- Valeurs de l’application Azure AD : **ID client** et **clé secrète client**.

Les valeurs pour les paramètres d’**ID client** et de **clé secrète client** se trouvent dans le portail Azure. Pour plus d’informations, voir [Prise en main de l’authentification Azure AD à l’aide du portail Azure](media-services-portal-get-started-with-aad.md).

L’exemple de code suivant permet de créer un jeton à l’aide du constructeur **AzureAdTokenCredentials**, qui accepte **AzureAdClientSymmetricKey** en tant que paramètre :

```csharp
var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                        new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"),
                        AzureEnvironments.AzureCloudEnvironment);

var tokenProvider = new AzureAdTokenProvider(tokenCredentials);
```

Vous pouvez également spécifier le constructeur **AzureAdTokenCredentials**, qui accepte **AzureAdClientCertificate** en tant que paramètre.

Pour obtenir des instructions sur la création et la configuration d’un certificat dans un formulaire qui peut être utilisé par Azure AD, consultez les étapes de configuration manuelle dans [Authenticating to Azure AD in daemon apps with certificates](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) (Authentification auprès d’Azure AD dans des applications démons avec des certificats).

```csharp
var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                        new AzureAdClientCertificate("{YOUR CLIENT ID HERE}", "{YOUR CLIENT CERTIFICATE THUMBPRINT}"),
                        AzureEnvironments.AzureCloudEnvironment);
```

Pour commencer à programmer sur Media Services, vous devez créer une instance **CloudMediaContext** qui représente le contexte du serveur. Vous devez également transmettre l’**URI de ressource des services REST Media** au constructeur **CloudMediaContext**. Vous pouvez également obtenir la valeur de l’**URI de ressource des services REST Media** à partir du portail Azure.

L’exemple de code suivant permet de créer une instance **CloudMediaContext** :

```csharp
CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);
```

L’exemple suivant montre comment créer le jeton Azure AD et le contexte :

```csharp
namespace AzureADAuthSample
{
    class Program
    {
        static void Main(string[] args)
        {
            var tokenCredentials = new AzureAdTokenCredentials("{YOUR Azure AD TENANT DOMAIN HERE}",
                                        new AzureAdClientSymmetricKey("{YOUR CLIENT ID HERE}", "{YOUR CLIENT SECRET}"),
                                        AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Specify your REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".
            CloudMediaContext context = new CloudMediaContext(new Uri("YOUR REST API ENDPOINT HERE"), tokenProvider);

            var assets = context.Assets;
            foreach (var a in assets)
            {
                Console.WriteLine(a.Name);
            }

            Console.ReadLine();
        }
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

Commencez le [chargement de fichiers sur votre compte](media-services-dotnet-upload-files.md).
