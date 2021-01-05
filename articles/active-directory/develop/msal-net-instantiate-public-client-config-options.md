---
title: Instancier une application cliente publique (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Découvrez comment instancier une application cliente publique avec des options de configuration à l’aide de Microsoft Authentication Library for .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 04/30/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 349ec8c99bc9e6768e5ec2ded346893da95c1636
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88166142"
---
# <a name="instantiate-a-public-client-application-with-configuration-options-using-msalnet"></a>Instancier une application cliente publique avec des options de configuration à l’aide de MSAL.NET

Cet article décrit comment instancier une [application cliente publique](msal-client-applications.md) à l’aide de Microsoft Authentication Library for .NET (MSAL.NET).  L’application est instanciée avec les options de configuration définies dans un fichier de paramètres.

Avant d’initialiser une application, vous devez d’abord [l’inscrire](quickstart-register-app.md) afin que votre application puisse être intégrée à la plateforme d’identités Microsoft. Après l’inscription, vous aurez peut-être besoin des informations suivantes (que vous trouverez sur le Portail Azure) :

- L’ID client (une chaîne représentant un GUID)
- L’URL du fournisseur d’identité (l’instance) et l’audience de connexion pour votre application. Ces deux paramètres représentent collectivement l’autorité.
- L’ID locataire si vous écrivez une application métier uniquement pour votre organisation (également nommée application à locataire unique).
- Pour les applications web et parfois pour les applications clientes publiques (en particulier lorsque votre application doit utiliser un répartiteur), vous devez également avoir défini l’URI de redirection auquel le fournisseur d’identité recontactera votre application avec les jetons de sécurité.


Une application de console .NET Core peut avoir le fichier de configuration *appsettings.json* suivant :

```json
{
  "Authentication": {
    "AzureCloudInstance": "AzurePublic",
    "AadAuthorityAudience": "AzureAdMultipleOrgs",
    "ClientId": "ebe2ab4d-12b3-4446-8480-5c3828d04c50"
  },

  "WebAPI": {
    "MicrosoftGraphBaseEndpoint": "https://graph.microsoft.com"
  }
}
```

Le code suivant lit ce fichier à l’aide de l’infrastructure de configuration .NET :

```csharp
public class SampleConfiguration
{
    /// <summary>
    /// Authentication options
    /// </summary>
    public PublicClientApplicationOptions PublicClientApplicationOptions { get; set; }

    /// <summary>
    /// Base URL for Microsoft Graph (it varies depending on whether the application is ran
    /// in Microsoft Azure public clouds or national / sovereign clouds
    /// </summary>
    public string MicrosoftGraphBaseEndpoint { get; set; }

    /// <summary>
    /// Reads the configuration from a json file
    /// </summary>
    /// <param name="path">Path to the configuration json file</param>
    /// <returns>SampleConfiguration as read from the json file</returns>
    public static SampleConfiguration ReadFromJsonFile(string path)
    {
        // .NET configuration
        IConfigurationRoot Configuration;
        var builder = new ConfigurationBuilder()
          .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile(path);
        Configuration = builder.Build();

        // Read the auth and graph endpoint config
        SampleConfiguration config = new SampleConfiguration()
        {
            PublicClientApplicationOptions = new PublicClientApplicationOptions()
        };
        Configuration.Bind("Authentication", config.PublicClientApplicationOptions);
        config.MicrosoftGraphBaseEndpoint = Configuration.GetValue<string>("WebAPI:MicrosoftGraphBaseEndpoint");
        return config;
    }
}
```

Le code suivant crée votre application à l’aide de la configuration du fichier de paramètres :

```csharp
SampleConfiguration config = SampleConfiguration.ReadFromJsonFile("appsettings.json");
var app = PublicClientApplicationBuilder.CreateWithApplicationOptions(config.PublicClientApplicationOptions)
           .Build();
```

