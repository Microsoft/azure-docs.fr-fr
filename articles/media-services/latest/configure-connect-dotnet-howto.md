---
title: Se connecter à l’API Azure Media Services v3 - .NET
description: Cet article explique comment se connecter à l’API Azure Media Services v3 avec .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/17/2020
ms.author: inhenkel
ms.custom: has-adal-ref, devx-track-csharp
ms.openlocfilehash: 3d4f232d87209a3a5676cac22e67a38b17af6917
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844316"
---
# <a name="connect-to-media-services-v3-api---net"></a>Se connecter à l’API Media Services v3 - .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cet article explique comment se connecter au kit de développement logiciel (SDK) Node.js Azure Media Services v3 .NET suivant la méthode de connexion au principal du service.

## <a name="prerequisites"></a>Prérequis

- [Créer un compte Media Services](./create-account-howto.md). Veillez à mémoriser le nom du groupe de ressources et le nom du compte Media Services.
- Installez un outil que vous souhaitez utiliser pour le développement .NET. Les étapes décrites dans cet article montrent comment utiliser [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Vous pouvez utiliser Visual Studio Code. Pour cela, consultez [Working with C#](https://code.visualstudio.com/docs/languages/csharp) (Utilisation de C#). Vous pouvez également utiliser un autre éditeur de code.

> [!IMPORTANT]
> Examinez les [conventions d’appellation](media-services-apis-overview.md#naming-conventions).

## <a name="create-a-console-application"></a>Création d’une application console

1. Démarrez Visual Studio. 
1. Dans le menu **Fichier**, cliquez sur **Nouveau** > **Projet**. 
1. Créez une application de console **.NET Core**.

L’exemple d’application de cette rubrique cible `netcoreapp2.0`. Le code utilise « async main », qui est disponible à partir de C# 7.1. Voir le [blog](/archive/blogs/benwilli/async-main-is-available-but-hidden) pour plus de détails.

## <a name="add-required-nuget-packagesassemblies"></a>Ajouter les packages/assemblys NuGet exigés

1. Dans Visual Studio, sélectionnez **Outils** > **Gestionnaire de package NuGet** > **Console du gestionnaire de package NuGet**.
2. Dans la fenêtre de la **console du Gestionnaire de package**, exécutez la commande `Install-Package` pour ajouter les packages NuGet suivants. Par exemple : `Install-Package Microsoft.Azure.Management.Media`.

|Package|Description|
|---|---|
|`Microsoft.Azure.Management.Media`|Kit SDK Azure Media Services. <br/>Pour vérifier que vous utilisez le dernier package Azure Media Services, exécutez [Microsoft.Azure.Management.Media](https://www.nuget.org/packages/Microsoft.Azure.Management.Media).|

### <a name="other-required-assemblies"></a>Autres assemblys requis

- Azure.Storage.Blobs
- Microsoft.Extensions.Configuration
- Microsoft.Extensions.Configuration.EnvironmentVariables
- Microsoft.Extensions.Configuration.Json
- Microsoft.Rest.ClientRuntime.Azure.Authentication

## <a name="create-and-configure-the-app-settings-file"></a>Créer et configurer le fichier des paramètres de l’application

### <a name="create-appsettingsjson"></a>Créer appsettings.json

1. Accédez go **Général** > **Fichier texte**.
1. Nommez-le « appsettings.json ».
1. Définissez la propriété « Copier dans le répertoire de sortie » du fichier .json sur « Copier si plus récent » (pour que l’application puisse y accéder lors de la publication).

### <a name="set-values-in-appsettingsjson"></a>Définir des valeurs dans le fichier appsettings.json

Exécutez la commande `az ams account sp create` comme décrit dans [Accéder aux API](./access-api-howto.md). La commande renvoie le fichier json que vous devez copier dans votre fichier appsettings.json.
 
## <a name="add-configuration-file"></a>Ajouter un fichier de configuration

Pour plus de commodité, ajoutez un fichier de configuration chargé de lire les valeurs à partir du fichier appsettings.json.

1. Ajoutez une nouvelle classe .cs à votre projet. Nommez-le `ConfigWrapper`. 
1. Collez le code suivant dans ce fichier (cet exemple suppose que l’espace de noms est `ConsoleApp1`).

```csharp
using System;

using Microsoft.Extensions.Configuration;

namespace ConsoleApp1
{
    public class ConfigWrapper
    {
        private readonly IConfiguration _config;

        public ConfigWrapper(IConfiguration config)
        {
            _config = config;
        }

        public string SubscriptionId
        {
            get { return _config["SubscriptionId"]; }
        }

        public string ResourceGroup
        {
            get { return _config["ResourceGroup"]; }
        }

        public string AccountName
        {
            get { return _config["AccountName"]; }
        }

        public string AadTenantId
        {
            get { return _config["AadTenantId"]; }
        }

        public string AadClientId
        {
            get { return _config["AadClientId"]; }
        }

        public string AadSecret
        {
            get { return _config["AadSecret"]; }
        }

        public Uri ArmAadAudience
        {
            get { return new Uri(_config["ArmAadAudience"]); }
        }

        public Uri AadEndpoint
        {
            get { return new Uri(_config["AadEndpoint"]); }
        }

        public Uri ArmEndpoint
        {
            get { return new Uri(_config["ArmEndpoint"]); }
        }

        public string Location
        {
            get { return _config["Location"]; }
        }
    }
}
```

## <a name="connect-to-the-net-client"></a>Se connecter au client .NET

Pour commencer à utiliser les API Media Services avec .NET, vous devez créer un objet **AzureMediaServicesClient**. Pour créer l’objet, vous devez fournir les informations d’identification nécessaires pour que le client puisse se connecter à Azure à l’aide d’Azure AD. Dans le code ci-dessous, la fonction GetCredentialsAsync crée l’objet ServiceClientCredentials basé sur les informations d’identification fournies dans le fichier de configuration local.

1. Ouvrez `Program.cs`.
1. Collez le code suivant :

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;

using Microsoft.Azure.Management.Media;
using Microsoft.Azure.Management.Media.Models;
using Microsoft.Extensions.Configuration;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;

namespace ConsoleApp1
{
    class Program
    {
        public static async Task Main(string[] args)
        {
            
            ConfigWrapper config = new ConfigWrapper(new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddEnvironmentVariables()
                .Build());

            try
            {
                IAzureMediaServicesClient client = await CreateMediaServicesClientAsync(config);
                Console.WriteLine("connected");
            }
            catch (Exception exception)
            {
                if (exception.Source.Contains("ActiveDirectory"))
                {
                    Console.Error.WriteLine("TIP: Make sure that you have filled out the appsettings.json file before running this sample.");
                }

                Console.Error.WriteLine($"{exception.Message}");

                ApiErrorException apiException = exception.GetBaseException() as ApiErrorException;
                if (apiException != null)
                {
                    Console.Error.WriteLine(
                        $"ERROR: API call failed with error code '{apiException.Body.Error.Code}' and message '{apiException.Body.Error.Message}'.");
                }
            }

            Console.WriteLine("Press Enter to continue.");
            Console.ReadLine();
        }
 
        private static async Task<ServiceClientCredentials> GetCredentialsAsync(ConfigWrapper config)
        {
            // Use ApplicationTokenProvider.LoginSilentWithCertificateAsync or UserTokenProvider.LoginSilentAsync to get a token using service principal with certificate
            //// ClientAssertionCertificate
            //// ApplicationTokenProvider.LoginSilentWithCertificateAsync

            // Use ApplicationTokenProvider.LoginSilentAsync to get a token using a service principal with symmetric key
            ClientCredential clientCredential = new ClientCredential(config.AadClientId, config.AadSecret);
            return await ApplicationTokenProvider.LoginSilentAsync(config.AadTenantId, clientCredential, ActiveDirectoryServiceSettings.Azure);
        }

        private static async Task<IAzureMediaServicesClient> CreateMediaServicesClientAsync(ConfigWrapper config)
        {
            var credentials = await GetCredentialsAsync(config);

            return new AzureMediaServicesClient(config.ArmEndpoint, credentials)
            {
                SubscriptionId = config.SubscriptionId,
            };
        }

    }
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Charger, encoder et diffuser des vidéos en continu - .NET](stream-files-tutorial-with-api.md) 
- [Tutoriel : Diffuser en direct avec Media Services v3 - .NET](stream-live-tutorial-with-api.md)
- [Tutoriel : Analyser des vidéos avec Media Services v3 - .NET](analyze-videos-tutorial-with-api.md)
- [Créer une entrée de travail à partir d’un fichier local - .NET](job-input-from-local-file-how-to.md)
- [Créer une entrée de travail à partir d’une URL HTTPS - .NET](job-input-from-http-how-to.md)
- [Encoder avec une transformation personnalisée - .NET](customize-encoder-presets-how-to.md)
- [Utiliser le chiffrement dynamique AES-128 et le service de distribution de clés - .NET](protect-with-aes128.md)
- [Utilisation du chiffrement dynamique DRM et du service de remise des licences - .NET](protect-with-drm.md)
- [Obtenir une clé de signature à partir de la stratégie existante - .NET](get-content-key-policy-dotnet-howto.md)
- [Créer des filtres avec Media Services - .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Exemples avancés de vidéo à la demande d’Azure Functions v2 avec Media Services v3](https://aka.ms/ams3functions)

## <a name="see-also"></a>Voir aussi

* [Information de référence sur .NET](/dotnet/api/overview/azure/mediaservices/management?view=azure-dotnet&preserve-view=true)
* Pour obtenir plus d’exemples de code, consultez le dépôt d’[exemples du Kit SDK .NET](https://github.com/Azure-Samples/media-services-v3-dotnet).
