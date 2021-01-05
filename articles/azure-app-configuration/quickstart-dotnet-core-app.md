---
title: Démarrage rapide pour Azure App Configuration avec .NET Core | Microsoft Docs
description: Dans ce démarrage rapide, créez une application .NET Core avec Azure App Configuration pour centraliser le stockage et la gestion des paramètres d’application en dehors de votre code.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 09/28/2020
ms.author: alkemper
ms.openlocfilehash: 1a5e756e05543d9f19775f4e2d1abcee82794f9e
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932062"
---
# <a name="quickstart-create-a-net-core-app-with-app-configuration"></a>Démarrage rapide : Créer une application .NET Core avec App Configuration

Dans ce guide de démarrage rapide, vous intégrez Azure App Configuration dans une application console .NET Core pour centraliser le stockage et la gestion des paramètres d’application en dehors de votre code.

## <a name="prerequisites"></a>Prérequis

- Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/dotnet)
- [Kit SDK .NET Core](https://dotnet.microsoft.com/download) : également disponible dans [Azure Cloud Shell](https://shell.azure.com).

## <a name="create-an-app-configuration-store"></a>Créer un magasin App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Sélectionnez **Explorateur de configurations** > **Créer** > **Clé-valeur** pour ajouter les paires clé-valeur suivantes :

    | Clé | Valeur |
    |---|---|
    | TestApp:Settings:Message | Data from Azure App Configuration |

    Laissez **Étiquette** et **Type de contenu** vides pour l’instant.

8. Sélectionnez **Appliquer**.

## <a name="create-a-net-core-console-app"></a>Créer une application console .NET Core

Vous utilisez l’[interface de ligne de commande (CLI) .NET Core](/dotnet/core/tools/) pour créer un projet d’application console .NET Core. Par rapport à Visual Studio, l’interface CLI .NET Core offre l’avantage d’être disponible sur les plateformes Windows, macOS et Linux.  Vous pouvez également utiliser les outils préinstallés qui sont disponibles dans [Azure Cloud Shell](https://shell.azure.com).

1. Créez un nouveau dossier pour votre projet.

2. Dans le nouveau dossier, exécutez la commande suivante pour créer un projet d’application console .NET Core :

    ```dotnetcli
    dotnet new console
    ```

## <a name="connect-to-an-app-configuration-store"></a>Se connecter à un magasin App Configuration

1. Ajoutez une référence au package NuGet `Microsoft.Extensions.Configuration.AzureAppConfiguration` en exécutant la commande suivante :

    ```dotnetcli
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

2. Exécutez la commande suivante pour restaurer les packages de votre projet :

    ```dotnetcli
    dotnet restore
    ```

3. Ouvrez *Program.cs*, puis ajoutez une référence au fournisseur App Configuration .NET Core.

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

4. Mettez à jour la méthode `Main` pour utiliser App Configuration en appelant la méthode `builder.AddAzureAppConfiguration()`.

    ```csharp
    static void Main(string[] args)
    {
        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(Environment.GetEnvironmentVariable("ConnectionString"));

        var config = builder.Build();
        Console.WriteLine(config["TestApp:Settings:Message"] ?? "Hello world!");
    }
    ```

## <a name="build-and-run-the-app-locally"></a>Générer et exécuter l’application localement

1. Définissez une variable d’environnement nommée **ConnectionString** et affectez-lui la valeur de la clé d’accès à votre magasin App Configuration. Sur la ligne de commande, exécutez la commande suivante :

    ```cmd
    setx ConnectionString "connection-string-of-your-app-configuration-store"
    ```

    Si vous utilisez Windows PowerShell, exécutez la commande suivante :

    ```azurepowershell
    $Env:ConnectionString = "connection-string-of-your-app-configuration-store"
    ```

    Si vous utilisez macOS ou Linux, exécutez la commande suivante :

    ```console
    export ConnectionString='connection-string-of-your-app-configuration-store'
    ```

    Redémarrez l’invite de commandes pour que la modification soit prise en compte. Imprimez la valeur de la variable d’environnement pour confirmer qu’elle est correctement définie.

2. Exécutez la commande suivante pour générer l’application console :

    ```dotnetcli
    dotnet build
    ```

3. La génération terminée correctement, lancez la commande suivante pour exécuter l’application localement :

    ```dotnetcli
    dotnet run
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

En suivant ce guide de démarrage rapide, vous avez créé un magasin App Configuration et l’avez utilisé avec une application console .NET Core par l’intermédiaire du [fournisseur App Configuration](/dotnet/api/Microsoft.Extensions.Configuration.AzureAppConfiguration). Pour savoir comment configurer votre application .NET Core afin d’actualiser dynamiquement les paramètres de configuration, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Activer la configuration dynamique](./enable-dynamic-configuration-dotnet-core.md)