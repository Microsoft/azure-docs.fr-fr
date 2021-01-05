---
title: Surveiller et gérer les travaux Azure Stream Analytics par programme
description: Cet article décrit comment surveiller par programme les travaux Stream Analytics créés via des API REST, le Kit de développement logiciel (SDK) Microsoft Azure ou PowerShell.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 04/20/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 9c746e409964b4625a4c9921d9f02ca0c9730a35
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123444"
---
# <a name="programmatically-create-a-stream-analytics-job-monitor"></a>Créer la surveillance des tâches Stream Analytics par programmation

Cet article explique comment activer la surveillance d'une tâche Stream Analytics. Par défaut, la surveillance n'est pas activée pour les travaux Stream Analytics créés par le biais des API REST, du kit de développement logiciel (SDK) Azure ou de PowerShell. Vous pouvez l’activer manuellement sur le portail Azure. Pour cela, accédez à la page Surveiller du travail et cliquez sur le bouton Activer. Vous pouvez également automatiser ce processus en suivant les étapes décrites dans cet article. Les données de surveillance sont affichées dans la zone Métriques du portail Azure pour le travail Stream Analytics.

## <a name="prerequisites"></a>Prérequis

Avant de commencer à lire ce processus, vous devez disposer des éléments suivants :

* Visual Studio 2019 ou 2015
* [kit de développement logiciel (SDK) Azure .NET](https://azure.microsoft.com/downloads/) téléchargé et installé
* Un travail Stream Analytics existant pour lequel la surveillance doit être activée

## <a name="create-a-project"></a>Création d’un projet

1. Créez une application console Visual Studio .NET C#.
2. Dans la console du Gestionnaire de package, exécutez les commandes suivantes pour installer les packages NuGet. Le premier est le Kit de développement logiciel (SDK) .NET de gestion Azure Stream Analytics. Le second est le SDK Azure Monitor qui sera utilisé pour activer la surveillance. Le dernier est le client Azure Active Directory utilisé pour l'authentification.
   
   ```powershell
   Install-Package Microsoft.Azure.Management.StreamAnalytics
   Install-Package Microsoft.Azure.Insights -Pre
   Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
   ```
3. Ajoutez la section appSettings suivante au fichier App.config.
   
   ```csharp
   <appSettings>
     <!--CSM Prod related values-->
     <add key="ResourceGroupName" value="RESOURCE GROUP NAME" />
     <add key="JobName" value="YOUR JOB NAME" />
     <add key="StorageAccountName" value="YOUR STORAGE ACCOUNT"/>
     <add key="ActiveDirectoryEndpoint" value="https://login.microsoftonline.com/" />
     <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
     <add key="WindowsManagementUri" value="https://management.core.windows.net/" />
     <add key="AsaClientId" value="1950a258-227b-4e31-a9cf-717495945fc2" />
     <add key="RedirectUri" value="urn:ietf:wg:oauth:2.0:oob" />
     <add key="SubscriptionId" value="YOUR AZURE SUBSCRIPTION ID" />
     <add key="ActiveDirectoryTenantId" value="YOUR TENANT ID" />
   </appSettings>
   ```
   Remplacez les valeurs de *SubscriptionId* et *ActiveDirectoryTenantId* par votre ID d’abonnement et votre ID de locataire Azure. Vous pouvez obtenir ces valeurs en exécutant l'applet de commande PowerShell suivante :
   
   ```powershell
   Get-AzureAccount
   ```
4. Ajoutez les instructions using suivantes au fichier source (Program.cs) dans le projet.
   
   ```csharp
     using System;
     using System.Configuration;
     using System.Threading;
     using Microsoft.Azure;
     using Microsoft.Azure.Management.Insights;
     using Microsoft.Azure.Management.Insights.Models;
     using Microsoft.Azure.Management.StreamAnalytics;
     using Microsoft.Azure.Management.StreamAnalytics.Models;
     using Microsoft.IdentityModel.Clients.ActiveDirectory;
   ```
5. Ajoutez une méthode d'aide pour l'authentification.

   ```csharp   
   public static string GetAuthorizationHeader()
   {
      AuthenticationResult result = null;
      var thread = new Thread(() =>
      {
         try
         {
             var context = new AuthenticationContext(
                ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] +
                ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
             result = context.AcquireToken(
                 resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
                 clientId: ConfigurationManager.AppSettings["AsaClientId"],
                 redirectUri: new Uri(ConfigurationManager.AppSettings["RedirectUri"]),
                 promptBehavior: PromptBehavior.Always);
         }
         catch (Exception threadEx)
         {
             Console.WriteLine(threadEx.Message);
         }
     });

     thread.SetApartmentState(ApartmentState.STA);
     thread.Name = "AcquireTokenThread";
     thread.Start();
     thread.Join();
   
     if (result != null)
     {
         return result.AccessToken;
     }
         throw new InvalidOperationException("Failed to acquire token");
   }
   ```

## <a name="create-management-clients"></a>Création de clients de gestion

Le code suivant définit les variables nécessaires et les clients de gestion.

   ```csharp
    string resourceGroupName = "<YOUR AZURE RESOURCE GROUP NAME>";
    string streamAnalyticsJobName = "<YOUR STREAM ANALYTICS JOB NAME>";

    // Get authentication token
    TokenCloudCredentials aadTokenCredentials =
        new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader());

    Uri resourceManagerUri = new
    Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    // Create Stream Analytics and Insights management client
    StreamAnalyticsManagementClient streamAnalyticsClient = new
    StreamAnalyticsManagementClient(aadTokenCredentials, resourceManagerUri);
    InsightsManagementClient insightsClient = new
    InsightsManagementClient(aadTokenCredentials, resourceManagerUri);
   ```

## <a name="enable-monitoring-for-an-existing-stream-analytics-job"></a>Activation de la surveillance pour un travail Stream Analytics existant

Le code suivant permet d'activer la surveillance pour un travail Stream Analytics **existant**. La première partie du code exécute une requête GET sur le service Stream Analytics pour récupérer des informations sur le travail Stream Analytics spécifique. Elle utilise la propriété *ID* (récupérée à partir de la requête GET) en tant que paramètre pour la méthode Put dans la seconde moitié du code qui envoie une requête PUT au service Insights afin d'activer la surveillance du travail Stream Analytics.

> [!WARNING]
> Si vous avez déjà activé la surveillance pour un travail Stream Analytics différente par le passé, via le portail Azure ou par programmation via le code ci-dessous, **nous vous recommandons de fournir le même nom de compte de stockage que celui utilisé lors de l’activation de la surveillance.**
> 
> Le compte de stockage est lié à la région dans laquelle vous avez créé la tâche Stream Analytics, pas spécifiquement à la tâche en soi.
> 
> Tous les travaux Stream Analytics (et l’ensemble des autres ressources Microsoft Azure) d’une même région partagent ce compte de stockage afin d’y consigner les données de surveillance. Si vous communiquez un compte de stockage différent, cela peut entraîner des effets collatéraux imprévus sur la surveillance de vos autres travaux Stream Analytics ou autres ressources Microsoft Azure.
> 
> Le compte de stockage utilisé pour remplacer `<YOUR STORAGE ACCOUNT NAME>` dans le code ci-dessous doit être un compte de stockage qui se trouve dans l’abonnement du travail Stream Analytics pour lequel vous avez activé la surveillance.
> 
> 
>    ```csharp
>    // Get an existing Stream Analytics job
>     JobGetParameters jobGetParameters = new JobGetParameters()
>     {
>         PropertiesToExpand = "inputs,transformation,outputs"
>     };
>     JobGetResponse jobGetResponse = streamAnalyticsClient.StreamingJobs.Get(resourceGroupName, streamAnalyticsJobName, jobGetParameters);
>
>    // Enable monitoring
>    ServiceDiagnosticSettingsPutParameters insightPutParameters = new ServiceDiagnosticSettingsPutParameters()
>    {
>            Properties = new ServiceDiagnosticSettings()
>           {
>               StorageAccountName = "<YOUR STORAGE ACCOUNT NAME>"
>           }
>    };
>   insightsClient.ServiceDiagnosticSettingsOperations.Put(jobGetResponse.Job.Id, insightPutParameters);
>   ```


## <a name="get-support"></a>Obtenir de l’aide

Pour obtenir de l’aide supplémentaire, consultez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)