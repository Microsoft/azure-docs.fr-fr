---
title: Tutoriel concernant l’utilisation de références Key Vault avec Azure App Configuration dans une application ASP.NET Core | Microsoft Docs
description: Dans ce tutoriel, vous apprenez à utiliser les références Key Vault avec Azure App Configuration depuis une application ASP.NET Core
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 04/08/2020
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 94bab9506d2bf7c29f997bcbfd400a412d5ac041
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932232"
---
# <a name="tutorial-use-key-vault-references-in-an-aspnet-core-app"></a>Tutoriel : Utiliser des références Key Vault dans une application ASP.NET Core

Dans ce tutoriel, vous apprenez à utiliser le service Azure App Configuration avec Azure Key Vault. App Configuration et Key Vault sont des services complémentaires utilisés côte à côte dans la plupart des déploiements d’applications.

App Configuration vous aide à utiliser ces services ensemble en créant des clés qui référencent des valeurs stockées dans Key Vault. Lorsque le service App Configuration crée de telles clés, il stocke les URI des valeurs Key Vault plutôt que les valeurs elles-mêmes.

Votre application utilise le fournisseur client de App Configuration pour récupérer les références Key Vault, tout comme pour toutes les autres clés stockées dans App Configuration. Dans ce cas, les valeurs stockées dans App Configuration sont des URI qui référencent les valeurs dans le coffre de clés. Elles ne sont pas des valeurs de coffre de clés ou des informations d’identification. Étant donné que le fournisseur client reconnaît les clés comme des références Key Vault, il utilise Key Vault pour récupérer leurs valeurs.

Votre application est chargée de s’authentifier correctement auprès d’App Configuration et auprès de Key Vault. Les deux services ne communiquent pas directement.

Ce tutoriel vous montre comment implémenter des références Key Vault dans votre code. Il s’appuie sur l’application web introduite dans les guides de démarrage rapide. Avant de continuer, terminez d’abord l’étape [Créer une application ASP.NET Core avec App Configuration](./quickstart-aspnet-core-app.md).

Vous pouvez utiliser l’éditeur de code de votre choix pour exécuter les étapes de ce tutoriel. Par exemple, [Visual Studio Code](https://code.visualstudio.com/) est un éditeur de code multiplateforme qui est disponible pour les systèmes d’exploitation Windows, macOS et Linux.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une clé App Configuration qui référence une valeur stockée dans Key Vault.
> * Accéder à la valeur de cette clé à partir d’une application web ASP.NET Core.

## <a name="prerequisites"></a>Prérequis

Avant d’effectuer ce tutoriel, installez le [kit SDK .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-vault"></a>Création d'un coffre

1. Sélectionnez l’option **Créer une ressource** dans le coin supérieur gauche du Portail Azure :

    ![La capture d’écran montre l’option Créer une ressource dans le portail Azure.](./media/quickstarts/search-services.png)
1. Dans la zone de recherche, entrez **Key Vault**.
1. Dans la liste des résultats, sélectionnez **Coffres de clés** sur la gauche.
1. Dans **Coffres de clés**, sélectionnez **Ajouter**.
1. Sur la droite, dans **Créer un coffre de clés**, renseignez les informations suivantes :
    - Sélectionnez **Abonnement** pour choisir un abonnement.
    - Dans **Groupe de ressources**, sélectionnez **Créer un nouveau** et entrez un nom de groupe de ressources.
    - Dans **Nom du coffre de clés**, un nom unique est requis. Pour ce tutoriel, entrez **Contoso-vault2**.
    - Dans la liste déroulante **Région**, choisissez un emplacement.
1. Conservez les valeurs par défaut des autres options **Création d’un coffre de clés**.
1. Sélectionnez **Create** (Créer).

À ce stade, votre compte Azure est le seul autorisé à accéder à ce nouveau coffre.

![La capture d’écran montre votre coffre de clés.](./media/quickstarts/vault-properties.png)

## <a name="add-a-secret-to-key-vault"></a>Ajouter un secret au coffre de clés

Pour ajouter un secret au coffre, vous n’avez qu’à effectuer deux autres étapes. Dans ce cas, ajoutez un message que vous pouvez utiliser pour tester la récupération de Key Vault. Le message est appelé **Message** et vous stockez la valeur « Hello from Key Vault » dedans.

1. Depuis les pages des propriétés Key Vault, sélectionnez **Secrets**.
1. Sélectionnez **Générer/Importer**.
1. Dans le volet **Créer un secret**, saisissez les valeurs suivantes :
    - **Options de chargement** : Entrez **Manuel**.
    - **Name** : Entrez **Message**.
    - **Valeur** : Entrez **Hello from Key Vault**.
1. Conservez les valeurs par défaut des autres propriétés **Créer un secret**.
1. Sélectionnez **Create** (Créer).

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Ajouter une référence Key Vault à App Configuration

1. Connectez-vous au [portail Azure](https://portal.azure.com). Sélectionnez **Toutes les ressources**, puis sélectionnez l’instance du magasin App Configuration que vous avez créée dans le guide de démarrage rapide.

1. Sélectionnez **Explorateur de configuration**.

1. Sélectionnez **+ Créer** > **Référence Key Vault**, puis choisissez les valeurs suivantes :
    - **Clé** : Sélectionnez **TestApp:Settings:KeyVaultMessage**.
    - **Étiquette** : Laissez cette valeur vide.
    - **Abonnement**, **Groupe de ressources** et **Key Vault** : Entrez les valeurs correspondant au coffre de clés que vous avez créé à la section précédente.
    - **Secret** : Sélectionnez le secret nommé **Message** que vous avez créé dans la section précédente.

## <a name="connect-to-key-vault"></a>Se connecter à Key Vault

1. Dans ce tutoriel, vous utilisez un principal de service pour l’authentification auprès de KeyVault. Pour créer ce principal de service, utilisez la commande Azure CLI [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) :

    ```azurecli
    az ad sp create-for-rbac -n "http://mySP" --sdk-auth
    ```

    Cette opération retourne une série de paires clé/valeur :

    ```console
    {
    "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
    "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
    "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
    "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
    }
    ```

1. Exécutez la commande suivante pour autoriser le principal du service à accéder à votre coffre de clés :

    ```cmd
    az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --secret-permissions delete get list set --key-permissions create decrypt delete encrypt get list unwrapKey wrapKey
    ```

1. Ajoutez des variables d’environnement pour stocker les valeurs de *clientId*, *clientSecret* et *tenantId*.

    #### <a name="windows-command-prompt"></a>[Invite de commandes Windows](#tab/cmd)

    ```cmd
    setx AZURE_CLIENT_ID <clientId-of-your-service-principal>
    setx AZURE_CLIENT_SECRET <clientSecret-of-your-service-principal>
    setx AZURE_TENANT_ID <tenantId-of-your-service-principal>
    ```

    #### <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```PowerShell
    $Env:AZURE_CLIENT_ID = <clientId-of-your-service-principal>
    $Env:AZURE_CLIENT_SECRET = <clientSecret-of-your-service-principal>
    $Env:AZURE_TENANT_ID = <tenantId-of-your-service-principal>
    ```

    #### <a name="bash"></a>[Bash](#tab/bash)

    ```bash
    export AZURE_CLIENT_ID = <clientId-of-your-service-principal>
    export AZURE_CLIENT_SECRET = <clientSecret-of-your-service-principal>
    export AZURE_TENANT_ID = <tenantId-of-your-service-principal>
    ```

    ---

    > [!NOTE]
    > Ces informations d’identification Key Vault sont utilisées uniquement dans votre application. Votre application s’authentifie directement auprès de Key Vault à l’aide de ces informations d’identification. Elles ne sont jamais transmises au service App Configuration.

1. Redémarrez votre terminal pour charger ces nouvelles variables d’environnement.

## <a name="update-your-code-to-use-a-key-vault-reference"></a>Mettre à jour votre code pour utiliser une référence Key Vault

1. Ajoutez une référence aux packages NuGet requis en exécutant la commande suivante :

    ```dotnetcli
    dotnet add package Azure.Identity
    ```

1. Ouvrez *Program.cs* et ajoutez des références aux packages requis suivants :

    ```csharp
    using Azure.Identity;
    ```

1. Mettez à jour la méthode `CreateWebHostBuilder` pour utiliser App Configuration en appelant la méthode `config.AddAzureAppConfiguration`. Incluez l’option `ConfigureKeyVault` et transmettez les informations d’identification correctes à votre coffre de clés.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                            .ConfigureKeyVault(kv =>
                            {
                                kv.SetCredential(new DefaultAzureCredential());
                            });
                });
            })
            .UseStartup<Startup>();
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                            .ConfigureKeyVault(kv =>
                            {
                                kv.SetCredential(new DefaultAzureCredential());
                            });
                });
            })
            .UseStartup<Startup>());
    ```

1. Quand vous avez initialisé la connexion à App Configuration, vous avez configuré la connexion à Key Vault en appelant la méthode `ConfigureKeyVault`. Après l’initialisation, vous pouvez accéder aux valeurs des références Key Vault de la même façon que vous accédez aux valeurs des clés App Configuration normales.

    Pour voir ce processus en action, ouvrez *Index.cshtml* dans **Affichages** > **Dossier de base**. Remplacez son contenu par le code ci-dessous :

    ```html
    @using Microsoft.Extensions.Configuration
    @inject IConfiguration Configuration

    <style>
        body {
            background-color: @Configuration["TestApp:Settings:BackgroundColor"]
        }
        h1 {
            color: @Configuration["TestApp:Settings:FontColor"];
            font-size: @Configuration["TestApp:Settings:FontSize"]px;
        }
    </style>

    <h1>@Configuration["TestApp:Settings:Message"]
        and @Configuration["TestApp:Settings:KeyVaultMessage"]</h1>
    ```

    Vous accédez à la valeur de la référence Key Vault **TestApp:Settings:KeyVaultMessage** de la même façon que vous accédez à la valeur de configuration **TestApp:Settings:Message**.

## <a name="build-and-run-the-app-locally"></a>Générer et exécuter l’application localement

1. Pour générer l’application à l’aide de l’interface CLI .NET Core, exécutez la commande suivante dans l’interpréteur de commandes :

    ```dotnetcli
    dotnet build
    ```

1. Une fois la génération terminée, utilisez la commande suivante pour exécuter l’application web localement :

    ```dotnetcli
    dotnet run
    ```

1. Ouvrez une fenêtre de navigateur, puis accédez à `http://localhost:5000`, qui est l’URL par défaut de l’application web hébergée localement.

    ![Démarrage rapide du lancement d’application locale](./media/key-vault-reference-launch-local.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous créez une clé App Configuration qui référence une valeur stockée dans Key Vault. Pour savoir comment ajouter une identité de service gérée par Azure qui simplifie l’accès à App Configuration et à Key Vault, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Intégration des identités managées](./howto-integrate-azure-managed-service-identity.md)
