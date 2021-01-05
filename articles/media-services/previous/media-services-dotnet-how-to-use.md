---
title: Configuration de l’ordinateur pour le développement Media Services avec .NET
description: Familiarisez-vous avec les conditions préalables pour Media Services à l’aide du Kit de développement logiciel (SDK) Media Services pour .NET. Apprenez également à créer une application Visual Studio.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: ec2804c7-c656-4fbf-b3e4-3f0f78599a7f
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 8eb45397b1941b9a6955066c22d6d9d00eeef4c3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89262088"
---
# <a name="media-services-development-with-net"></a>Développement Media Services avec .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Cet article explique comment commencer à développer des applications Media Services à l’aide de .NET.

La bibliothèque du **Kit de développement logiciel (SDK) Azure Media Services pour .NET** permet de programmer pour Media Services à l'aide de .NET. Pour que le développement avec .NET soit encore plus simple, la bibliothèque des **extensions du Kit de développement logiciel (SDK) Azure Media Services pour .NET** est fournie. Cette bibliothèque contient un ensemble de méthodes d’extension et de fonctions d’assistance qui simplifient votre code .NET. Les deux bibliothèques sont disponibles par le biais de **NuGet** et de **GitHub**.

## <a name="prerequisites"></a>Prérequis
* Un compte Media Services dans un abonnement Azure nouveau ou existant. Consultez l’article [Création d’un compte Media Services](media-services-portal-create-account.md).
* Systèmes d'exploitation : Windows 10, Windows 7, Windows 2008 R2 ou Windows 8.
* .NET Framework 4.5 ou ultérieur.
* Visual Studio.

## <a name="create-and-configure-a-visual-studio-project"></a>Créer et configurer un projet Visual Studio
Cette section vous montre comment créer un projet dans Visual Studio et le configurer pour le développement Media Services.  Dans ce cas, le projet est une application console C# Windows, mais les étapes de configuration présentées ici s’appliquent aux autres types de projets que vous pouvez créer pour les applications Media Services (par exemple, une application Windows Forms ou Web ASP.NET).

Cette section montre comment utiliser **NuGet** pour ajouter des extensions de Kit de développement logiciel (SDK) Media Services pour .NET et d’autres bibliothèques dépendantes.

Vous pouvez également obtenir les dernières informations relatives au Kit de développement logiciel (SDK) Media Services pour .NET à partir de GitHub ([github.com/Azure/azure-sdk-for-media-services](https://github.com/Azure/azure-sdk-for-media-services) ou [github.com/Azure/azure-sdk-for-media-services-extensions](https://github.com/Azure/azure-sdk-for-media-services-extensions)), générer la solution et ajouter les références au projet client. Toutes les dépendances nécessaires sont téléchargées et extraites automatiquement.

1. Créez une application console C# dans Visual Studio. Entrez le **nom**, l’**emplacement** et le **nom de solution**, puis cliquez sur OK.
2. Générez la solution.
3. Utilisez **NuGet** pour installer et ajouter les **extensions du Kit de développement logiciel (SDK) Azure Media Services pour .NET** (**windowsazure.mediaservices.extensions**). L'installation de ce package installe également le **Kit de développement logiciel (SDK) Media Services pour .NET** et ajoute toutes les autres dépendances requises.
   
    Assurez-vous que la version la plus récente de NuGet est installée. Pour obtenir des informations supplémentaires et des instructions relatives à l'installation, consultez la page [NuGet](https://nuget.codeplex.com/).

    1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le nom du projet, puis sélectionnez **Gérer les packages NuGet**.

    2. La boîte de dialogue Gérer les packages NuGet apparaît.

    3. Dans la galerie en ligne, recherchez Extensions Azure MediaServices, choisissez **Azure Media Services .NET SDK Extensions** (**windowsazure.mediaservices.extensions**), puis cliquez sur le bouton **Installer**.
   
    4. Le projet est modifié et des références aux extensions du SDK Media Services pour .NET, au SDK Media Services pour .NET et à d’autres assemblys dépendants sont ajoutées.
4. Pour promouvoir un environnement de développement plus propre, envisagez d’activer la restauration du package NuGet. Pour plus d'informations, consultez le document [Restauration du package NuGet](https://docs.nuget.org/consume/package-restore).
5. Ajoutez une référence à l'assembly **System.Configuration** . Cet assembly contient la classe System.Configuration.**ConfigurationManager** qui est utilisée pour accéder aux fichiers de configuration (par exemple, App.config).
   
    1. Pour ajouter des références à l’aide de la boîte de dialogue Gérer les références, cliquez avec le bouton droit sur le nom du projet dans l’Explorateur de solutions. Ensuite, cliquez sur **Ajouter** puis sur **Référence**.
   
    2. La boîte de dialogue Gérer les références s’affiche.
    3. Sous Assemblys .NET Framework, recherchez et sélectionnez l’assembly System.Configuration et appuyez sur **OK**.
6. Ouvrez le fichier App.config et ajoutez-y une section **appSettings**. Définissez les valeurs nécessaires pour se connecter à l’API Media Services. Pour plus d’informations, consultez l’article [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). 

    Définissez les valeurs nécessaires pour se connecter en utilisant la méthode d’authentification **Principal du service**.

    ```xml
    <configuration>
    ...
        <appSettings>
            <add key="AMSAADTenantDomain" value="tenant"/>
            <add key="AMSRESTAPIEndpoint" value="endpoint"/>
            <add key="AMSClientId" value="id"/>
            <add key="AMSClientSecret" value="secret"/>
        </appSettings>
    </configuration>
    ```

7. Ajoutez la référence **System.Configuration** à votre projet.
8. Remplacez les instructions **using** existantes au début du fichier Program.cs par le code suivant :

    ```csharp      
    using System;
    using System.Configuration;
    using System.IO;
    using Microsoft.WindowsAzure.MediaServices.Client;
    using System.Threading;
    using System.Collections.Generic;
    using System.Linq;
    ```

    À ce stade, vous êtes prêt à commencer le développement d’une application Media Services.    

## <a name="example"></a>Exemple

Ce court exemple permet de se connecter à l’API AMS et de répertorier tous les processeurs multimédias disponibles.

```csharp
class Program
{
    // Read values from the App.config file.

    private static readonly string _AADTenantDomain =
        ConfigurationManager.AppSettings["AMSAADTenantDomain"];
    private static readonly string _RESTAPIEndpoint =
        ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
    private static readonly string _AMSClientId =
        ConfigurationManager.AppSettings["AMSClientId"];
    private static readonly string _AMSClientSecret =
        ConfigurationManager.AppSettings["AMSClientSecret"];
        
    private static CloudMediaContext _context = null;
    static void Main(string[] args)
    {
        AzureAdTokenCredentials tokenCredentials = 
            new AzureAdTokenCredentials(_AADTenantDomain,
                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                AzureEnvironments.AzureCloudEnvironment);

        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
        
        // List all available Media Processors
        foreach (var mp in _context.MediaProcessors)
        {
            Console.WriteLine(mp.Name);
        }
        
    }
 ```

## <a name="next-steps"></a>Étapes suivantes

[Vous pouvez désormais vous connecter à l’API AMS](media-services-use-aad-auth-to-access-ams-api.md) et commencer à [développer](media-services-dotnet-get-started.md).


## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
