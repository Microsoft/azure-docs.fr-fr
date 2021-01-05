---
title: Prise en main du Kit de développement logiciel (SDK) WebJobs
description: Introduction au Kit de développement logiciel (SDK) WebJobs pour le traitement en arrière-plan basé sur les événements. Découvrez comment accéder aux données des services Azure et des services tiers.
author: ggailey777
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 02/18/2019
ms.author: glenga
ms.openlocfilehash: 4a22602dd9638b981cfe3d8bae9b5cdaacbf90dc
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91652038"
---
# <a name="get-started-with-the-azure-webjobs-sdk-for-event-driven-background-processing"></a>Prise en main du Kit de développement logiciel (SDK) Azure WebJobs pour le traitement en arrière-plan basé sur les événements

Cet article explique comment utiliser Visual Studio 2019 pour créer un projet de Kit de développement logiciel (SDK) Azure WebJobs, l’exécuter localement et le déployer vers [Azure App Service](overview.md). La version 3.x du kit de développement logiciel (SDK) WebJobs prend en charge les applications console .NET Core et .NET Framework. Pour en savoir plus sur l’utilisation du kit de développement logiciel (SDK) WebJobs, consultez [Comment utiliser le kit de développement logiciel (SDK) Azure WebJobs pour le traitement en arrière-plan basé sur les événements](webjobs-sdk-how-to.md).

Cet article vous explique comment déployer WebJobs en tant qu'application console .NET Core. Pour déployer WebJobs en tant qu'application console .NET Framework, consultez [Tâches web en tant qu’applications console .NET Framework](webjobs-dotnet-deploy-vs.md#webjobs-as-net-framework-console-apps). Si la version 2.x du kit de développement logiciel (SDK) WebJobs, qui prend uniquement en charge .NET Framework, vous intéresse, consultez [Développer et déployer WebJobs à l’aide de Visual Studio - Azure App Service](webjobs-dotnet-deploy-vs.md).

## <a name="prerequisites"></a>Prérequis

* [Installez Visual Studio 2019](/visualstudio/install/) avec la charge de travail de **développement Azure**. Si vous possédez déjà Visual Studio, mais que vous n’avez pas cette charge de travail, ajoutez-la en sélectionnant **Outils > Get Tools and Features** (Obtenir des outils et des fonctionnalités).

* Vous devez disposer d'[un compte Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) pour publier votre projet de kit de développement logiciel (SDK) WebJobs sur Azure.

## <a name="create-a-project"></a>Création d’un projet

1. Dans Visual Studio, sélectionnez **Créer un projet**.

2. Sélectionnez **Application console (.NET Core)** .

3. Nommez le projet *WebJobsSDKSample*, puis sélectionnez **Créer**.

   ![Boîte de dialogue Nouveau projet](./media/webjobs-sdk-get-started/new-project.png)

## <a name="webjobs-nuget-packages"></a>Packages NuGet WebJobs

1. Installez la dernière version 3.x stable du package NuGet [`Microsoft.Azure.WebJobs.Extensions`](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions/), qui inclut `Microsoft.Azure.WebJobs`.

     Voici la commande **Console du Gestionnaire de package** :

     ```powershell
     Install-Package Microsoft.Azure.WebJobs.Extensions -version <3_X_VERSION>
     ```

    Dans cette commande, remplacez `<3_X_VERSION>` par une version prise en charge du package. 

## <a name="create-the-host"></a>Créer l’hôte

L’hôte est le conteneur d’exécution pour les fonctions. Il écoute les déclencheurs et appelle les fonctions. Les étapes suivantes créent un hôte implémentant [`IHost`](/dotnet/api/microsoft.extensions.hosting.ihost), qui est l’hôte générique dans ASP.NET Core.

1. Dans *Program.cs*, ajoutez les instructions `using` suivantes :

    ```cs
    using System.Threading.Tasks;
    using Microsoft.Extensions.Hosting;
    ```

1. Remplacez la méthode `Main` par le code suivant :

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

Dans ASP.NET Core, les configurations de l’hôte sont définies par l’appel de méthodes sur l’instance [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). Pour plus d’informations, consultez [Hôte générique .NET](/aspnet/core/fundamentals/host/generic-host). La méthode d’extension `ConfigureWebJobs` initialise l’hôte WebJobs. Dans `ConfigureWebJobs`, vous initialiez des extensions WebJobs spécifiques et définissez les propriétés de ces extensions.  

## <a name="enable-console-logging"></a>Activer la journalisation de console

Dans cette section, vous allez configurer la journalisation de console qui utilise le [framework de journalisation ASP.NET Core](/aspnet/core/fundamentals/logging).

1. Installez la dernière version stable du package NuGet [`Microsoft.Extensions.Logging.Console`](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/), qui inclut `Microsoft.Extensions.Logging`.

   Voici la commande **Console du Gestionnaire de package** :

   ```powershell
   Install-Package Microsoft.Extensions.Logging.Console -version <3_X_VERSION>
   ```
   Dans cette commande, remplacez `<3_X_VERSION>` par une version 3.x prise en charge du package.

1. Dans *Program.cs*, ajoutez une instruction `using` :

   ```cs
   using Microsoft.Extensions.Logging;
   ```

1. Appelez la méthode [`ConfigureLogging`](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configurelogging) sur [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder). La méthode [`AddConsole`](/dotnet/api/microsoft.extensions.logging.consoleloggerextensions.addconsole) ajoute la journalisation de console à la configuration.

    ```cs
    builder.ConfigureLogging((context, b) =>
    {
        b.AddConsole();
    });
    ```

    La méthode `Main` se présente désormais ainsi :

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

    Cette mise à jour effectue les actions suivantes :

    * Désactive la [journalisation du tableau de bord](https://github.com/Azure/azure-webjobs-sdk/wiki/Queues#logs). Le tableau de bord est un outil de surveillance hérité ; la journalisation du tableau de bord n’est pas recommandée pour les scénarios de production de débit élevé.
    * Ajoute le fournisseur de console avec le [filtrage](webjobs-sdk-how-to.md#log-filtering) par défaut.

Vous pouvez désormais ajouter une fonction qui est déclenchée par des messages arrivant dans une file d’attente du stockage Azure.

## <a name="install-the-storage-binding-extension"></a>Installer l’extension liaison de stockage

À partir de la version 3.x, vous devez installer explicitement l’extension de liaison de stockage nécessaire au kit de développement logiciel (SDK) WebJobs. Dans les versions précédentes, les liaisons de stockage sont incluses dans le kit de développement logiciel (SDK).

1. Installez la dernière version stable du package NuGet [Microsoft.Azure.WebJobs.Extensions.Storage](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.Storage), version 3.x. 

    Voici la commande **Console du Gestionnaire de package** :

    ```powershell
    Install-Package Microsoft.Azure.WebJobs.Extensions.Storage -Version <3_X_VERSION>
    ```
    
    Dans cette commande, remplacez `<3_X_VERSION>` par une version prise en charge du package. 

2. Dans la méthode d’extension `ConfigureWebJobs`, appelez la méthode `AddAzureStorage` sur l’instance [`HostBuilder`](/dotnet/api/microsoft.extensions.hosting.hostbuilder) pour initialiser l’extension de stockage. À ce stade, la méthode `ConfigureWebJobs` est semblable à l’exemple suivant :

    ```cs
    builder.ConfigureWebJobs(b =>
                    {
                        b.AddAzureStorageCoreServices();
                        b.AddAzureStorage();
                    });
    ```

## <a name="create-a-function"></a>Créer une fonction

1. Cliquez avec le bouton droit sur le projet, sélectionnez **Ajouter** > **Nouvel élément...** , choisissez **Classe**, nommez le nouveau fichier de classe C# *Functions.cs*, puis sélectionnez **Ajouter**.

1. Dans Functions.cs, remplacez le modèle généré par le code suivant :
    
    ```cs
    using Microsoft.Azure.WebJobs;
    using Microsoft.Extensions.Logging;
    
    namespace WebJobsSDKSample
    {
        public class Functions
        {
            public static void ProcessQueueMessage([QueueTrigger("queue")] string message, ILogger logger)
            {
                logger.LogInformation(message);
            }
        }
    }
    ```

   L’attribut `QueueTrigger` indique au runtime d’appeler cette fonction lorsqu’un nouveau message est écrit dans une file d’attente de stockage Azure appelée `queue`. Le contenu du message en file d’attente est fourni pour le code de méthode dans le paramètre `message`. Le corps de la méthode correspond à l’endroit où vous traitez les données du déclencheur. Dans cet exemple, le code ne fait qu’enregistrer le message.

   Le paramètre `message` ne doit pas nécessairement être une chaîne. Vous pouvez également le lier à un objet JSON, un tableau d’octets ou un objet [CloudQueueMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage). Consultez la section relative à [l’utilisation des déclencheurs de file d’attente](../azure-functions/functions-bindings-storage-queue-trigger.md?tabs=csharp#usage). Chaque type de liaison (par exemple, des files d’attente, des objets blob ou des tables) dispose d’un ensemble différent de types de paramètre auquel vous pouvez lier des éléments.

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

L’émulateur de Stockage Azure exécuté localement ne possède pas toutes les fonctionnalités nécessaires au Kit de développement logiciel (SDK) WebJobs. Dans cette section, vous allez donc créer un compte de stockage dans Azure et configurer le projet pour l’utiliser. Si vous disposez déjà d'un compte de stockage, passez à l’étape 6.

1. Ouvrez **l’Explorateur de serveurs** dans Visual Studio et connectez-vous à Azure. Cliquez avec le bouton droit sur le nœud **Azure**, puis sélectionner **Se connecter à un abonnement Microsoft Azure**.

   ![Connexion à Azure](./media/webjobs-sdk-get-started/sign-in.png)

1. Sous le nœud **Azure** de **l’Explorateur de serveurs**, cliquez avec le bouton droit sur **Stockage**, puis sélectionnez **Créer un compte de stockage**.

   ![Menu Créer un compte de stockage](./media/webjobs-sdk-get-started/create-storage-account-menu.png)

1. Dans la boîte de dialogue **Créer un compte de stockage**, entrez un nom unique pour le compte de stockage.

1. Choisissez la **région** dans laquelle vous avez créé votre application App Service ou une région qui se trouve à proximité.

1. Sélectionnez **Create** (Créer).

   ![Créer un compte de stockage](./media/webjobs-sdk-get-started/create-storage-account.png)

1. Sous le nœud **Stockage** dans **l’Explorateur de serveurs**, sélectionnez le nouveau compte de stockage. Dans la fenêtre **Propriétés**, sélectionnez les points de suspension ( **...** ) à droite du champ de valeur **Chaîne de connexion**.

   ![Points de suspension du champ Chaîne de connexion](./media/webjobs-sdk-get-started/conn-string-ellipsis.png)

1. Copiez la chaîne de connexion et enregistrez cette valeur là où vous pouvez la copier à nouveau facilement.

   ![Copier la chaîne de connexion](./media/webjobs-sdk-get-started/copy-key.png)

## <a name="configure-storage-to-run-locally"></a>Configurer le stockage pour une exécution locale

Le kit SDK WebJobs recherche la chaîne de connexion de stockage dans les Paramètres de l’application dans Azure. Si vous l’exécutez localement, il recherche cette valeur dans le fichier de configuration local ou dans les variables d’environnement.

1. Cliquez avec le bouton droit sur le projet, sélectionnez **Ajouter** > **Nouvel élément...** , choisissez **Fichier de configuration JSON JavaScript**, nommez le nouveau fichier *appsettings.json*, puis sélectionnez **Ajouter**. 

1. Dans le nouveau fichier, ajoutez un champ `AzureWebJobsStorage`, comme dans l’exemple suivant :

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}"
    }
    ```

1. Remplacez *{chaîne de connexion de stockage}* par la chaîne de connexion que vous avez copiée précédemment.

1. Sélectionnez le fichier *appsettings.json* dans l’Explorateur de solutions et, dans la fenêtre **Propriétés**, définissez **Copier dans le répertoire de sortie** sur **Copier si plus récent**.

Plus tard, vous allez ajouter le même paramètre d’application de chaîne de connexion à votre application dans Azure App Service.

## <a name="test-locally"></a>Tester les topologies localement

Dans cette section, vous allez générer et exécuter le projet localement et déclencher la fonction en créant un message en file d’attente.

1. Appuyez sur **Ctrl+F5** pour exécuter le projet.

   La console indique que le runtime a trouvé votre fonction et attend que les messages en file d’attente le déclenchent. La sortie suivante est générée par l’hôte v3.x :

   ```console
    info: Microsoft.Azure.WebJobs.Hosting.JobHostService[0]
          Starting JobHost
    info: Host.Startup[0]
          Found the following functions:
          WebJobsSDKSample.Functions.ProcessQueueMessage

    info: Host.Startup[0]
          Job host started
    Application started. Press Ctrl+C to shut down.
    Hosting environment: Development
    Content root path: C:\WebJobsSDKSample\WebJobsSDKSample\bin\Debug\netcoreapp2.1\
   ```

1. Fermez la fenêtre de console.

1. Dans **l’Explorateur de serveurs** de Visual Studio, développez le nœud correspondant au nouveau compte de stockage, puis cliquez avec le bouton droit sur **Files d’attente**.

1. Sélectionnez **Créer une file d’attente**.

1. Entrez *file d’attente* comme nom de la file d’attente, puis sélectionnez **OK**.

   ![Capture d’écran montrant où vous créez la file d’attente et la nommez « file d’attente ». ](./media/webjobs-sdk-get-started/create-queue.png)

1. Cliquez avec le bouton droit sur le nœud de la nouvelle file d’attente, puis sélectionnez **Afficher la file d’attente**.

1. Sélectionnez l’icône **Ajouter un message**.

   ![Capture d’écran mettant en évidence l’icône Ajouter un message.](./media/webjobs-sdk-get-started/create-queue-message.png)

1. Dans la boîte de dialogue **Ajouter un message**, entrez *Hello World!* en tant que **Texte du message**, puis sélectionnez **OK**. La file d'attente contient maintenant un message.

   ![Créer la file d’attente](./media/webjobs-sdk-get-started/hello-world-text.png)

1. Réexécutez le projet.

   Comme vous avez utilisé l’attribut `QueueTrigger` dans la fonction `ProcessQueueMessage`, le runtime du Kit de développement logiciel (SDK) WeJobs écoute les messages en file d’attente lors du démarrage. Il recherche un nouveau message dans la file d’attente nommée *file d’attente* et appelle la fonction.

   En raison de la [temporisation exponentielle de l’interrogation de la file d’attente](../azure-functions/functions-bindings-storage-queue-trigger.md?tabs=csharp#polling-algorithm), 2 minutes peuvent être nécessaires au runtime pour trouver le message et appeler la fonction. Ce délai d’attente peut être réduit en lançant l’exécution en [mode de développement](webjobs-sdk-how-to.md#host-development-settings).

   La sortie de console se présente ainsi :

   ```console
    info: Function.ProcessQueueMessage[0]
          Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=2c319369-d381-43f3-aedf-ff538a4209b8)
    info: Function.ProcessQueueMessage[0]
          Trigger Details: MessageId: b00a86dc-298d-4cd2-811f-98ec39545539, DequeueCount: 1, InsertionTime: 1/18/2019 3:28:51 AM +00:00
    info: Function.ProcessQueueMessage.User[0]
          Hello World!
    info: Function.ProcessQueueMessage[0]
          Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=2c319369-d381-43f3-aedf-ff538a4209b8)
   ```

1. Fermez la fenêtre de console. 

1. Revenez à la fenêtre File d'attente et actualisez-la. Le message n'y apparaît plus car il a été traité par votre fonction exécutée localement. 

## <a name="add-application-insights-logging"></a>Ajouter la journalisation dans Application Insights

Lorsque le projet s’exécute dans Azure, vous ne pouvez pas surveiller l’exécution de la fonction en affichant la sortie de la console. Nous vous recommandons d’utiliser [Application Insights](../azure-monitor/app/app-insights-overview.md) comme solution de supervision. Pour plus d’informations, consultez [Surveiller l’exécution des fonctions Azure](../azure-functions/functions-monitoring.md).

Dans cette section, vous allez effectuer les tâches suivantes pour configurer la journalisation dans Application Insights avant de procéder au déploiement sur Azure :

* Vérifiez que vous disposez d’une application App Service et d’une instance Application Insights à utiliser.
* Configurez l’application App Service pour utiliser l’instance Application Insights et le compte de stockage que vous avez créés précédemment.
* Configurez le projet pour la journalisation dans Application Insights.

### <a name="create-app-service-app-and-application-insights-instance"></a>Créer une application App Service et une instance Application Insights

1. Si vous ne disposez pas déjà d’une application App Service que vous pouvez utiliser, [créez-en une](quickstart-dotnet-framework.md). Lorsque vous créez votre application, vous pouvez également créer une ressource Application Insights connectée. Lorsque vous faites cela, `APPINSIGHTS_INSTRUMENTATIONKEY` est défini pour vous dans votre application.

1. Si vous ne disposez pas déjà d’une ressource Application Insights que vous pouvez utiliser, [créez-en une](../azure-monitor/app/create-new-resource.md ). Définissez l’option **Type d’application** sur **Général** et ignorez les sections qui suivent **Copier la clé d’instrumentation**.

1. Si vous disposez déjà d’une ressource Application Insights que vous souhaitez utiliser, [copiez la clé d’instrumentation](../azure-monitor/app/create-new-resource.md#copy-the-instrumentation-key).

### <a name="configure-app-settings"></a>Configuration des paramètres d’application 

1. Dans **l’Explorateur de serveurs** de Visual Studio, développez le nœud **App Service** sous **Azure**.

1. Développez le groupe de ressources dans lequel figure votre application App Service, puis cliquez avec le bouton droit sur votre application App Service.

1. Sélectionnez **Afficher les paramètres**.

1. Dans la zone **Chaînes de connexion**, ajoutez l’entrée suivante.

   |Nom  |Chaîne de connexion  |Type de base de données|
   |---------|---------|------|
   |AzureWebJobsStorage | {chaîne de connexion de stockage que vous avez copiée précédemment}|Custom|

1. Si la zone **Paramètres de l’application** est dépourvue de clé d’instrumentation Application Insights, ajoutez celle que vous avez copiée précédemment. (La clé d’instrumentation figure peut-être déjà dans la zone selon la façon dont vous avez créé l’application App Service.)

   |Nom  |Valeur  |
   |---------|---------|
   |APPINSIGHTS_INSTRUMENTATIONKEY | {clé d’instrumentation} |

1. Remplacez *{clé d’instrumentation}* par la clé d’instrumentation de la ressource Application Insights que vous utilisez.

1. Sélectionnez **Enregistrer**.

1. Ajoutez la connexion Application Insights au projet, afin de pouvoir l’exécuter localement. Dans le fichier *appsettings.json*, ajoutez un champ `APPINSIGHTS_INSTRUMENTATIONKEY`, comme dans l’exemple suivant :

    ```json
    {
        "AzureWebJobsStorage": "{storage connection string}",
        "APPINSIGHTS_INSTRUMENTATIONKEY": "{instrumentation key}"
    }
    ```

    Remplacez *{clé d’instrumentation}* par la clé d’instrumentation de la ressource Application Insights que vous utilisez.

1. Enregistrez vos modifications.

### <a name="add-application-insights-logging-provider"></a>Ajouter le fournisseur de journalisation Application Insights

Pour tirer parti de la journalisation [Application Insights](../azure-monitor/app/app-insights-overview.md), mettez à jour votre code de journalisation pour effectuer les opérations suivantes :

* Ajoutez un fournisseur de journalisation Application Insights avec le [filtrage](webjobs-sdk-how-to.md#log-filtering) par défaut. Lors de l’exécution locale, toutes les informations et les journaux de niveau supérieur sont écrits à la fois dans la console et dans Application Insights.
* Placez l'objet [LoggerFactory](./webjobs-sdk-how-to.md#logging-and-monitoring) dans un bloc `using` pour vous assurer que la sortie du journal est vidée lorsque l’hôte s’arrête.

1. Installez la dernière version 3.x stable du package NuGet [`Microsoft.Azure.WebJobs.Logging.ApplicationInsights`](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Logging.ApplicationInsights/).

   Voici la commande **Console du Gestionnaire de package** :

   ```powershell
   Install-Package Microsoft.Azure.WebJobs.Logging.ApplicationInsights -Version <3_X_VERSION>
   ```
    Dans cette commande, remplacez `<3_X_VERSION>` par une version prise en charge du package.

1. Ouvrez *Program.cs* et remplacez le code existant dans la méthode `Main` par le code suivant :

    ```cs
    static async Task Main()
    {
        var builder = new HostBuilder();
        builder.UseEnvironment(EnvironmentName.Development);
        builder.ConfigureWebJobs(b =>
                {
                    b.AddAzureStorageCoreServices();
                    b.AddAzureStorage();
                });
        builder.ConfigureLogging((context, b) =>
                {
                    b.AddConsole();

                    // If the key exists in settings, use it to enable Application Insights.
                    string instrumentationKey = context.Configuration["APPINSIGHTS_INSTRUMENTATIONKEY"];
                    if (!string.IsNullOrEmpty(instrumentationKey))
                    {
                        b.AddApplicationInsightsWebJobs(o => o.InstrumentationKey = instrumentationKey);
                    }
                });
        var host = builder.Build();
        using (host)
        {
            await host.RunAsync();
        }
    }
    ```

    Le fournisseur Application Insights est ajouté à la journalisation à l'aide de la clé que vous avez précédemment ajoutée aux paramètres de votre application.

## <a name="test-application-insights-logging"></a>Tester la journalisation dans Application Insights

Dans cette section, vous allez effectuer une nouvelle exécution locale pour vérifier que les données de journalisation accèdent maintenant à Application Insights et à la console.

1. Utilisez l'**Explorateur de serveurs** de Visual Studio pour créer un message en file d'attente, comme vous l'avez fait [précédemment](#test-locally), mais cette fois-ci entrez *Hello App Insights !* comme texte du message.

1. Exécutez le projet.

   Le kit de développement logiciel (SDK) WebJobs traite le message en file d'attente, et les journaux d'activité sont disponibles dans la fenêtre de la console.

1. Fermez la fenêtre de console.

1. Ouvrez le [portail Azure](https://portal.azure.com/) pour accéder à la ressource Application Insights. Recherchez et sélectionnez **Application Insights**.

1. Choisissez votre instance d'Application Insights.

1. Sélectionnez **Recherche**.

   ![Sélectionner Recherche](./media/webjobs-sdk-get-started/select-search.png)

1. Si le message *Hello App Insights !* ne s’affiche pas, sélectionnez **Actualiser** à plusieurs reprises pendant plusieurs minutes. (Les journaux d'activité n'apparaissent pas immédiatement, car un certain temps est nécessaire au client Application Insights pour vider les journaux d'activité traités.)

   ![Journaux d’activité dans Application Insights](./media/webjobs-sdk-get-started/logs-in-ai.png)

1. Fermez la fenêtre de console.

## <a name="deploy-to-azure"></a><a name="deploy-as-a-webjob"></a>Déployer sur Azure

Lors du déploiement, vous créez une instance App Service dans laquelle exécuter vos fonctions. Lorsque vous publiez une application console .NET Core vers App Service dans Azure, celle-ci est automatiquement exécutée en tant que tâche web. Pour plus d'informations sur la publication, consultez [Développer et déployer des WebJobs à l’aide de Visual Studio](webjobs-dotnet-deploy-vs.md).

[!INCLUDE [webjobs-publish-net-core](../../includes/webjobs-publish-net-core.md)]

## <a name="trigger-the-function-in-azure"></a>Déclencher la fonction dans Azure

1. Assurez-vous que l’exécution n’est pas locale (fermez la fenêtre de console si elle est toujours ouverte). Dans le cas contraire, l’instance locale risque d’être la première à traiter les messages en file d’attente que vous créez.

1. Dans la page **File d’attente** de Visual Studio, ajoutez un message à la file d’attente, comme vous l’avez déjà fait.

1. Actualisez la page **File d’attente** ; le nouveau message disparaît, car il a été traité par la fonction qui s’exécute dans Azure.

   > [!TIP]
   > Si vous procédez au test dans Azure, utilisez le [mode de développement](webjobs-sdk-how-to.md#host-development-settings) pour vous assurer qu’une fonction de déclenchement de file d’attente est appelée immédiatement et éviter les retards dus à la [temporisation exponentielle de l’interrogation de la file d’attente](../azure-functions/functions-bindings-storage-queue-trigger.md?tabs=csharp#polling-algorithm).

### <a name="view-logs-in-application-insights"></a>Afficher les journaux d’activité dans Application Insights

1. Ouvrez le [portail Azure](https://portal.azure.com/), puis accédez à la ressource Application Insights.

1. Sélectionnez **Recherche**.

1. Si le message *Hello Azure !* ne s’affiche pas, sélectionnez **Actualiser** à plusieurs reprises pendant plusieurs minutes.

   Vous pouvez observer les journaux d’activité de la fonction exécutée dans une tâche web, y compris le texte *Hello Azure !* que vous avez entré dans la section précédente.

## <a name="add-an-input-binding"></a>Ajouter une liaison d’entrée

Les liaisons d’entrée simplifient le code qui lit les données. Pour cet exemple, le message en file d’attente est un nom d’objet blob que vous allez utiliser pour rechercher et lire un objet blob dans Stockage Azure.

1. Dans *Functions.cs*, remplacez la méthode `ProcessQueueMessage` par le code suivant :

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
   }
   ```

   Dans ce code, `queueTrigger` est une [expression de liaison](../azure-functions/functions-bindings-expressions-patterns.md), ce qui signifie qu’elle est résolue en une valeur différente lors de l’exécution.  Lors de l’exécution, son contenu est celui du message en file d’attente.

1. Ajoutez un paramètre `using` :

   ```cs
   using System.IO;
   ```

1. Créez un conteneur d’objets blob dans votre compte de stockage.

   a. Dans **l’Explorateur de serveurs** de Visual Studio, développez le nœud de votre compte de stockage, cliquez avec le bouton droit sur **Objets blob**, puis sélectionnez **Créer un conteneur d’objets blob**.

   b. Dans la boîte de dialogue **Créer un conteneur d’objets blob**, entrez *conteneur* comme nom du conteneur, puis cliquez sur **OK**.

1. Chargez le fichier *Program.cs* dans le conteneur d’objets blob. (Ce fichier est utilisé ici à titre d’exemple. Vous pouvez charger un fichier texte et créer un message en file d’attente portant le nom du fichier.)

   a. Dans **l’Explorateur de serveurs**, double-cliquez sur le nœud correspondant au conteneur que vous avez créé.

   b. Dans la fenêtre **Conteneur**, sélectionnez le bouton **Charger**.

   ![Bouton Charger l’objet blob](./media/webjobs-sdk-get-started/blob-upload-button.png)

   c. Recherchez et sélectionnez *Program.cs*, puis sélectionnez **OK**.

1. Créez un message en file d’attente dans la file d’attente que vous avez créée précédemment, avec *Program.cs* comme texte du message.

   ![Message en file d’attente Program.cs](./media/webjobs-sdk-get-started/queue-msg-program-cs.png)

1. Exécutez le projet localement.

   Le message en file d’attente déclenche la fonction, qui lit l’objet blob et consigne sa longueur. La sortie de console se présente ainsi :

   ```console
   Found the following functions:
   ConsoleApp1.Functions.ProcessQueueMessage
   Job host started
   Executing 'Functions.ProcessQueueMessage' (Reason='New queue message detected on 'queue'.', Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   Blob name:Program.cs
   Size: 532 bytes
   Executed 'Functions.ProcessQueueMessage' (Succeeded, Id=5a2ac479-de13-4f41-aae9-1361f291ff88)
   ```

## <a name="add-an-output-binding"></a>Ajouter une liaison de sortie

Les liaisons de sortie simplifient le code qui écrit les données. Cet exemple modifie l’exemple précédent en écrivant une copie de l’objet blob au lieu de consigner sa taille. Les liaisons du stockage d’objets Blob sont incluses dans le package d’extension du stockage Azure que nous avons installé précédemment.

1. Remplacez la méthode `ProcessQueueMessage` par le code suivant :

   ```cs
   public static void ProcessQueueMessage(
       [QueueTrigger("queue")] string message,
       [Blob("container/{queueTrigger}", FileAccess.Read)] Stream myBlob,
       [Blob("container/copy-{queueTrigger}", FileAccess.Write)] Stream outputBlob,
       ILogger logger)
   {
       logger.LogInformation($"Blob name:{message} \n Size: {myBlob.Length} bytes");
       myBlob.CopyTo(outputBlob);
   }
   ```

1. Créez un autre message en file d’attente avec *Program.cs* comme texte du message.

1. Exécutez le projet localement.

   Le message en file d’attente déclenche la fonction, qui lit l’objet blob, consigne sa longueur et crée un objet blob. La sortie de la console est identique, mais lorsque vous accédez à la fenêtre Conteneur d’objets blob et sélectionnez **Actualiser**, vous pouvez observer un nouvel objet blob nommé *copy-Program.cs*.

## <a name="republish-the-updates-to-azure"></a>Republier les mises à jour vers Azure

1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Publier**.

1. Dans la boîte de dialogue **Publier**, vérifiez que le profil actuel est sélectionné, puis choisissez **Publier**. Les résultats de la publication sont détaillés dans la fenêtre **Sortie**.
 
1. Vérifiez la fonction dans Azure en rechargeant un fichier dans le conteneur d'objets blob et en ajoutant un message à la file d’attente correspondant au nom du fichier téléchargé. Comme vous pourrez le constater, le message est supprimé de la file d’attente et une copie du fichier est créée dans le conteneur d’objets blob. 

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment créer, exécuter et déployer un projet de kit de développement logiciel (SDK) WebJobs 3.x.

> [!div class="nextstepaction"]
> [En savoir plus sur le kit SDK WebJobs](webjobs-sdk-how-to.md)