---
title: Kit de développement logiciel (SDK) Management .NET pour Azure Stream Analytics
description: Familiarisez-vous avec le SDK .NET Stream Analytics Management. Découvrez comment configurer et exécuter des travaux d’analyse. Créez un projet, ainsi que des entrées, des sorties et des transformations.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/06/2018
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 93e10baa5281481b57e945f93b7375060b574de5
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93126853"
---
# <a name="management-net-sdk-set-up-and-run-analytics-jobs-using-the-azure-stream-analytics-api-for-net"></a>Kit de développement logiciel (SDK) .NET de gestion : Configurer et exécuter des tâches analytics à l’aide de l’API Azure Stream Analytics pour .NET
Découvrez comment configurer et exécuter des travaux d’analyse à l’aide de l’API Stream Analytics pour .NET avec le Kit de développement logiciel (SDK) Management .NET. configurer un projet, créer des sources d’entrée et de sortie, des transformations, et démarrer et arrêter des tâches. Pour vos tâches d’analyse, vous pouvez diffuser des données à partir du stockage d’objets blob ou d’un hub d’événements.

Consultez la [documentation de référence de gestion de l’API Stream Analytics pour .NET](/previous-versions/azure/dn889315(v=azure.100)).

Azure Stream Analytics est un service entièrement géré permettant de traiter des événements avec une latence faible, une haute disponibilité et de façon évolutive via des données de diffusion dans le cloud. Stream Analytics permet aux clients de configurer des tâches de diffusion en continu pour analyser des flux de données et de mener à bien des analyses en quasi-temps réel.  

> [!NOTE]
> Nous avons mis à jour l’exemple de code de cet article vers la version v2.x du Kit de développement logiciel (SDK) Azure Stream Analytics Management .NET. Pour visualiser un exemple de code utilisant la version héritée (1.x) du Kit de développement logiciel (SDK), consultez [Use the Management .NET SDK v1.x for Stream Analytics (Utiliser le Kit de développement logiciel [SDK] Management .NET v1.x. pour Stream Analytics)]().

## <a name="prerequisites"></a>Prérequis
Avant de commencer cet article, vous devez disposer de la configuration requise suivante :

* Installez Visual Studio 2019 ou 2015.
* Téléchargez et installez le [Kit SDK Azure .NET](https://azure.microsoft.com/downloads/).
* Créez un groupe de ressources Azure dans votre abonnement. Voici un exemple de script Azure PowerShell. Pour obtenir des informations sur Azure PowerShell, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).  

   ```powershell
   # Log in to your Azure account
   Add-AzureAccount
   
   # Select the Azure subscription you want to use to create the resource group
   Select-AzureSubscription -SubscriptionName <subscription name>
   
   # If Stream Analytics has not been registered to the subscription, remove the remark    symbol (#) to run the Register-AzProvider cmdlet to register the provider namespace
   #Register-AzProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
   
   # Create an Azure resource group
   New-AzureResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
   ```

* Configurez une source d’entrée et une cible de sortie auxquelles le travail se connecte.

## <a name="set-up-a-project"></a>Configuration d’un projet
Pour créer une tâche d’analyse à l’aide de l’API Stream Analytics pour .NET, commencez par configurer votre projet.

1. Créez une application console Visual Studio .NET C#.
2. Dans la console du Gestionnaire de package, exécutez les commandes suivantes pour installer les packages NuGet. Le premier est le Kit de développement logiciel (SDK) .NET de gestion Azure Stream Analytics. La seconde concerne l’authentification du client Azure.

   ```powershell   
   Install-Package Microsoft.Azure.Management.StreamAnalytics -Version 2.0.0
   Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.3.1
   ```

3. Ajoutez la section **appSettings** suivante au fichier App.config :
   
   ```powershell
   <appSettings>
       <add key="ClientId" value="1950a258-227b-4e31-a9cf-717495945fc2" />
       <add key="RedirectUri" value="urn:ietf:wg:oauth:2.0:oob" />
       <add key="SubscriptionId" value="YOUR SUBSCRIPTION ID" />
       <add key="ActiveDirectoryTenantId" value="YOUR TENANT ID" />
   </appSettings>
   ```

    Remplacez les valeurs de **SubscriptionId** et **ActiveDirectoryTenantId** par votre ID d’abonnement et votre ID de locataire Azure. Vous pouvez obtenir ces valeurs en exécutant l’applet de commande Azure PowerShell suivante :

   ```powershell
      Get-AzureAccount
   ```

4. Ajoutez la référence suivante dans votre fichier .csproj :

   ```csharp
   <Reference Include="System.Configuration" />
   ```

5. Ajoutez les instructions **using** suivantes au fichier source (Program.cs) dans le projet :
   
   ```csharp
   using System;
   using System.Collections.Generic;
   using System.Configuration;
   using System.Threading;
   using System.Threading.Tasks;
   
   using Microsoft.Azure.Management.StreamAnalytics;
   using Microsoft.Azure.Management.StreamAnalytics.Models;
   using Microsoft.Rest.Azure.Authentication;
   using Microsoft.Rest;
   ```

6. Ajoutez une méthode d’aide pour l’authentification :

   ```csharp
   private static async Task<ServiceClientCredentials> GetCredentials()
   {
       var activeDirectoryClientSettings = ActiveDirectoryClientSettings.UsePromptOnly(ConfigurationManager.AppSettings["ClientId"], new Uri("urn:ietf:wg:oauth:2.0:oob"));
       ServiceClientCredentials credentials = await UserTokenProvider.LoginWithPromptAsync(ConfigurationManager.AppSettings["ActiveDirectoryTenantId"], activeDirectoryClientSettings);
       
       return credentials;
    }
   ```

## <a name="create-a-stream-analytics-management-client"></a>Création d’un client de gestion Stream Analytics
Un objet **StreamAnalyticsManagementClient** vous permet de gérer le travail et les composants de travail, comme l’entrée, la sortie et la transformation.

Ajoutez le code suivant au début de la méthode **Main** :

   ```csharp
    string resourceGroupName = "<YOUR AZURE RESOURCE GROUP NAME>";
    string streamingJobName = "<YOUR STREAMING JOB NAME>";
    string inputName = "<YOUR JOB INPUT NAME>";
    string transformationName = "<YOUR JOB TRANSFORMATION NAME>";
    string outputName = "<YOUR JOB OUTPUT NAME>";
    
    SynchronizationContext.SetSynchronizationContext(new SynchronizationContext());
    
    // Get credentials
    ServiceClientCredentials credentials = GetCredentials().Result;
    
    // Create Stream Analytics management client
    StreamAnalyticsManagementClient streamAnalyticsManagementClient = new StreamAnalyticsManagementClient(credentials)
    {
        SubscriptionId = ConfigurationManager.AppSettings["SubscriptionId"]
    };
   ```

La valeur de la variable **resourceGroupName** doit être la même que le nom du groupe de ressources que vous avez créé ou sélectionné lors des étapes préalables.

Pour automatiser l’aspect de présentation des informations d’identification pendant la création d’un travail, consultez [Authentification d’un principal du service à l’aide d’Azure Resource Manager](../active-directory/develop/howto-authenticate-service-principal-powershell.md).

Les sections suivantes de cet article supposent que ce code se trouve au début de la méthode **Main** .

## <a name="create-a-stream-analytics-job"></a>Création d’un travail Stream Analytics
Le code suivant crée un travail Stream Analytics sous le groupe de ressources que vous avez défini. Vous ajouterez ultérieurement une entrée, une sortie et une transformation au travail.

   ```csharp
   // Create a streaming job
   StreamingJob streamingJob = new StreamingJob()
   {
       Tags = new Dictionary<string, string>()
       {
           { "Origin", ".NET SDK" },
           { "ReasonCreated", "Getting started tutorial" }
       },
       Location = "West US",
       EventsOutOfOrderPolicy = EventsOutOfOrderPolicy.Drop,
       EventsOutOfOrderMaxDelayInSeconds = 5,
       EventsLateArrivalMaxDelayInSeconds = 16,
       OutputErrorPolicy = OutputErrorPolicy.Drop,
       DataLocale = "en-US",
       CompatibilityLevel = CompatibilityLevel.OneFullStopZero,
       Sku = new Sku()
       {
           Name = SkuName.Standard
       }
   };
   StreamingJob createStreamingJobResult = streamAnalyticsManagementClient.StreamingJobs.CreateOrReplace(streamingJob, resourceGroupName, streamingJobName);
   ```

## <a name="create-a-stream-analytics-input-source"></a>Création d’une source d’entrée Stream Analytics
Le code suivant crée une source d’entrée Stream Analytics avec objet blob comme type de source d’entrée et sérialisation CSV. Pour créer une source d’entrée de hub d’événements, utilisez **EventHubStreamInputDataSource** au lieu de **BlobStreamInputDataSource**. De même, vous pouvez personnaliser le type de sérialisation de la source d’entrée.

   ```csharp
   // Create an input
   StorageAccount storageAccount = new StorageAccount()
   {
       AccountName = "<YOUR STORAGE ACCOUNT NAME>",
       AccountKey = "<YOUR STORAGE ACCOUNT KEY>"
   };
   Input input = new Input()
   {
       Properties = new StreamInputProperties()
       {
           Serialization = new CsvSerialization()
           {
               FieldDelimiter = ",",
               Encoding = Encoding.UTF8
           },
           Datasource = new BlobStreamInputDataSource()
           {
               StorageAccounts = new[] { storageAccount },
               Container = "<YOUR STORAGE BLOB CONTAINER>",
               PathPattern = "{date}/{time}",
               DateFormat = "yyyy/MM/dd",
               TimeFormat = "HH",
               SourcePartitionCount = 16
           }
       }
   };
   Input createInputResult = streamAnalyticsManagementClient.Inputs.CreateOrReplace(input, resourceGroupName, streamingJobName, inputName);
   ```

Les sources d’entrée, qu’elles proviennent d’un stockage d’objets blob ou d’un hub d’événements, sont liées à une tâche spécifique. Pour utiliser la même source d’entrée pour différents travaux, vous devez appeler la méthode à nouveau et spécifier un nom de travail différent.

## <a name="test-a-stream-analytics-input-source"></a>Test d’une source d’entrée Stream Analytics
La méthode **TestConnection** teste si le travail Stream Analytics est en mesure de se connecter à la source d’entrée, ainsi que d’autres aspects spécifiques au type de source d’entrée. Par exemple, pour la source d’entrée d’objet blob que vous avez créée à l’étape précédente, la méthode vérifie que le nom et la paire de clés du compte de stockage peuvent être utilisés pour se connecter au compte de stockage et que le conteneur spécifié existe.

   ```csharp
   // Test the connection to the input
   ResourceTestStatus testInputResult = streamAnalyticsManagementClient.Inputs.Test(resourceGroupName, streamingJobName, inputName);
   ```

## <a name="create-a-stream-analytics-output-target"></a>Création d’une cible de sortie Stream Analytics
La création d’une cible de sortie est similaire à la création d’une source d’entrée Stream Analytics. Tout comme les sources d’entrée, les cibles de sortie sont liées à une tâche spécifique. Pour utiliser la même cible de sortie pour différents travaux, vous devez appeler la méthode à nouveau et spécifier un nom de travail différent.

Le code suivant crée une cible de sortie (Azure SQL Database). Vous pouvez personnaliser le type de données de la cible de sortie et/ou le type de sérialisation.

   ```csharp
   // Create an output
   Output output = new Output()
   {
       Datasource = new AzureSqlDatabaseOutputDataSource()
       {
           Server = "<YOUR DATABASE SERVER NAME>",
           Database = "<YOUR DATABASE NAME>",
           User = "<YOUR DATABASE LOGIN>",
           Password = "<YOUR DATABASE LOGIN PASSWORD>",
           Table = "<YOUR DATABASE TABLE NAME>"
       }
   };
   Output createOutputResult = streamAnalyticsManagementClient.Outputs.CreateOrReplace(output, resourceGroupName, streamingJobName, outputName);
   ```

## <a name="test-a-stream-analytics-output-target"></a>Test d’une cible de sortie Stream Analytics
Une cible de sortie Stream Analytics dispose également de la méthode **TestConnection** pour tester les connexions.

   ```csharp
   // Test the connection to the output
   ResourceTestStatus testOutputResult = streamAnalyticsManagementClient.Outputs.Test(resourceGroupName, streamingJobName, outputName);
   ```

## <a name="create-a-stream-analytics-transformation"></a>Création d’une transformation Stream Analytics
Le code suivant crée une transformation Stream Analytics avec la requête « select * from Input » et spécifie l’allocation d’une unité de diffusion en continu pour le travail Stream Analytics. Pour plus d’informations sur le réglage des unités de diffusion en continu, consultez [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md).

   ```csharp
   // Create a transformation
   Transformation transformation = new Transformation()
   {
       Query = "Select Id, Name from <your input name>", // '<your input name>' should be replaced with the value you put for the 'inputName' variable above or in a previous step
       StreamingUnits = 1
   };
   Transformation createTransformationResult = streamAnalyticsManagementClient.Transformations.CreateOrReplace(transformation, resourceGroupName, streamingJobName, transformationName);
   ```

Tout comme les entrées et sorties, une transformation est liée au travail Stream Analytics spécifique dans lequel elle a été créée.

## <a name="start-a-stream-analytics-job"></a>Démarrage d’un travail Stream Analytics
Après avoir créé un travail Stream Analytics et ses entrées, sorties et transformations, vous pouvez démarrer le travail en appelant la méthode **Start** .

L’exemple de code suivant démarre un travail Stream Analytics avec une heure de début de sortie personnalisée, le 12 décembre 2012, 12:12:12 UTC :

   ```csharp
   // Start a streaming job
   StartStreamingJobParameters startStreamingJobParameters = new StartStreamingJobParameters()
   {
       OutputStartMode = OutputStartMode.CustomTime,
       OutputStartTime = new DateTime(2012, 12, 12, 12, 12, 12, DateTimeKind.Utc)
   };
   streamAnalyticsManagementClient.StreamingJobs.Start(resourceGroupName, streamingJobName, startStreamingJobParameters);
   ```

## <a name="stop-a-stream-analytics-job"></a>Arrêt d’un travail Stream Analytics
Vous pouvez arrêter un travail Stream Analytics en cours d’exécution en appelant la méthode **Stop** .

   ```csharp
   // Stop a streaming job
   streamAnalyticsManagementClient.StreamingJobs.Stop(resourceGroupName, streamingJobName);
   ```

## <a name="delete-a-stream-analytics-job"></a>Suppression d’un travail Stream Analytics
La méthode **Delete** supprime le travail, ainsi que les ressources sous-jacentes, y compris les entrées, les sorties et la transformation du travail.

   ```csharp
   // Delete a streaming job
   streamAnalyticsManagementClient.StreamingJobs.Delete(resourceGroupName, streamingJobName);
   ```

## <a name="get-support"></a>Obtenir de l’aide
Pour obtenir de l’aide supplémentaire, consultez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes
Vous avez découvert les principes de base de l’utilisation d’un Kit de développement logiciel (SDK) .NET pour créer et exécuter des travaux d’analyse. Pour en savoir plus, consultez les articles suivants :

* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Kit de développement logiciel (SDK) .NET Azure Stream Analytics Management](/previous-versions/azure/dn889315(v=azure.100))
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)

<!--Image references-->
[5]: ./media/markdown-template-for-new-articles/octocats.png
[6]: ./media/markdown-template-for-new-articles/pretty49.png
[7]: ./media/markdown-template-for-new-articles/channel-9.png


<!--Link references-->
[azure.blob.storage]: https://azure.microsoft.com/documentation/services/storage/
[azure.blob.storage.use]: ../storage/blobs/storage-quickstart-blobs-dotnet.md

[azure.event.hubs]: https://azure.microsoft.com/services/event-hubs/
[azure.event.hubs.developer.guide]: /previous-versions/azure/dn789972(v=azure.100)

[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.forum]: https://go.microsoft.com/fwlink/?LinkId=512151

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.developer.guide]: stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/