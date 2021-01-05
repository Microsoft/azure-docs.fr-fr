---
title: Créer et exécuter des tests de disponibilité personnalisés avec Azure Functions
description: Cette documentation explique comment créer une fonction Azure avec TrackAvailability() pour qu’elle s’exécute régulièrement selon la configuration spécifiée dans la fonction TimerTrigger. Les résultats de ce test sont envoyés à votre ressource Application Insights, où vous pouvez rechercher et signaler des données sur les résultats de disponibilité. Les tests personnalisés vous permettent d’écrire des tests de disponibilité plus complexes qu’avec l’IU du portail, de superviser une application dans votre réseau virtuel Azure, de changer l’adresse du point de terminaison ou de créer un test de disponibilité, si ce dernier n’est pas disponible dans votre région.
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: 3553b212d1b63d4bd239893ba90aa3465d98df60
ms.sourcegitcommit: ba7fafe5b3f84b053ecbeeddfb0d3ff07e509e40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91945649"
---
# <a name="create-and-run-custom-availability-tests-using-azure-functions"></a>Créer et exécuter des tests de disponibilité personnalisés avec Azure Functions

Cet article explique comment créer une fonction Azure avec TrackAvailability() pour qu’elle s’exécute régulièrement selon la configuration spécifiée dans la fonction TimerTrigger avec votre propre logique métier. Les résultats de ce test sont envoyés à votre ressource Application Insights, où vous pouvez rechercher et signaler des données sur les résultats de disponibilité. Cela vous permet de créer des tests personnalisés similaires à ceux que vous pouvez effectuer via la [supervision de la disponibilité](./monitor-web-app-availability.md) dans le portail. Les tests personnalisés vous permettent d’écrire des tests de disponibilité plus complexes qu’avec l’IU du portail, de superviser une application dans votre réseau virtuel Azure, de changer l’adresse du point de terminaison ou de créer un test de disponibilité même si cette fonctionnalité n’est pas disponible dans votre région.

> [!NOTE]
> Cet exemple est conçu uniquement pour vous montrer comment fonctionne l’appel d’API TrackAvailability() au sein d’une fonction Azure. Notez comment écrire le code de test HTTP sous-jacent/la logique métier nécessaires pour convertir ce code en test de disponibilité entièrement fonctionnel. Par défaut, si vous passez en revue cet exemple, vous allez créer un test de disponibilité qui génère toujours un échec.

## <a name="create-timer-triggered-function"></a>Créer une fonction déclenchée par un minuteur

- Si vous avez une ressource Application Insights :
    - Par défaut, Azure Functions crée une ressource Application Insights, mais si vous souhaitez utiliser l’une de vos ressources déjà créées, vous devez le spécifier durant la création.
    - Suivez les instructions permettant de [créer une ressource Azure Functions et une fonction déclenchée par un minuteur](../../azure-functions/functions-create-scheduled-function.md) (arrêter avant le nettoyage) avec les choix suivants.
        -  Sélectionnez l’onglet **Supervision** en haut.

            ![ Créer une application Azure Functions avec votre propre ressource App Insights](media/availability-azure-functions/create-function-app.png)

        - Sélectionnez la zone de liste déroulante Application Insights et tapez ou sélectionnez le nom de votre ressource.

            ![Sélection d’une ressource Application Insights existante](media/availability-azure-functions/app-insights-resource.png)

        - Sélectionner **Vérifier + créer**
- Si vous n’avez pas encore créé de ressource Application Insights pour votre fonction déclenchée par un minuteur :
    - Par défaut, quand vous créez votre application Azure Functions, une ressource Application Insights est créée automatiquement.
    - Suivez les instructions permettant de [créer une ressource Azure Functions et une fonction déclenchée par un minuteur](../../azure-functions/functions-create-scheduled-function.md) (arrêter avant le nettoyage).

## <a name="sample-code"></a>Exemple de code

Copiez le code ci-dessous dans le fichier run.csx (cette opération remplace le code préexistant). Pour ce faire, accédez à votre application Azure Functions, puis sélectionnez votre fonction avec déclenchement par minuteur sur la gauche.

>[!div class="mx-imgBorder"]
>![Fichier run.csx de la fonction Azure dans le portail Azure](media/availability-azure-functions/runcsx.png)

> [!NOTE]
> Pour l’adresse du point de terminaison, utilisez `EndpointAddress= https://dc.services.visualstudio.com/v2/track`. Si votre ressource se trouve dans une région comme Azure Government ou Azure Chine, consultez cet article sur le [remplacement des points de terminaison par défaut](./custom-endpoints.md#regions-that-require-endpoint-modification), puis sélectionnez le point de terminaison de canal de télémétrie approprié pour votre région.

```C#
#load "runAvailabilityTest.csx"
 
using System;
using System.Diagnostics;
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;
 
// The Application Insights Instrumentation Key can be changed by going to the overview page of your Function App, selecting configuration, and changing the value of the APPINSIGHTS_INSTRUMENTATIONKEY Application setting.
// DO NOT replace the code below with your instrumentation key, the key's value is pulled from the environment variable/application setting key/value pair.
private static readonly string instrumentationKey = Environment.GetEnvironmentVariable("APPINSIGHTS_INSTRUMENTATIONKEY");
 
//[CONFIGURATION_REQUIRED]
// If your resource is in a region like Azure Government or Azure China, change the endpoint address accordingly.
// Visit https://docs.microsoft.com/azure/azure-monitor/app/custom-endpoints#regions-that-require-endpoint-modification for more details.
private const string EndpointAddress = "https://dc.services.visualstudio.com/v2/track";
 
private static readonly TelemetryConfiguration telemetryConfiguration = new TelemetryConfiguration(instrumentationKey, new InMemoryChannel { EndpointAddress = EndpointAddress });
private static readonly TelemetryClient telemetryClient = new TelemetryClient(telemetryConfiguration);
 
public async static Task Run(TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"Entering Run at: {DateTime.Now}");
 
    if (myTimer.IsPastDue)
    {
        log.LogWarning($"[Warning]: Timer is running late! Last ran at: {myTimer.ScheduleStatus.Last}");
    }
 
    // [CONFIGURATION_REQUIRED] provide {testName} accordingly for your test function
    string testName = "AvailabilityTestFunction";
 
    // REGION_NAME is a default environment variable that comes with App Service
    string location = Environment.GetEnvironmentVariable("REGION_NAME");
 
    log.LogInformation($"Executing availability test run for {testName} at: {DateTime.Now}");
    string operationId = Guid.NewGuid().ToString("N");
 
    var availability = new AvailabilityTelemetry
    {
        Id = operationId,
        Name = testName,
        RunLocation = location,
        Success = false
    };
 
    var stopwatch = new Stopwatch();
    stopwatch.Start();
 
    try
    {
        await RunAvailbiltyTestAsync(log);
        availability.Success = true;
    }
    catch (Exception ex)
    {
        availability.Message = ex.Message;
 
        var exceptionTelemetry = new ExceptionTelemetry(ex);
        exceptionTelemetry.Context.Operation.Id = operationId;
        exceptionTelemetry.Properties.Add("TestName", testName);
        exceptionTelemetry.Properties.Add("TestLocation", location);
        telemetryClient.TrackException(exceptionTelemetry);
    }
    finally
    {
        stopwatch.Stop();
        availability.Duration = stopwatch.Elapsed;
        availability.Timestamp = DateTimeOffset.UtcNow;
 
        telemetryClient.TrackAvailability(availability);
        // call flush to ensure telemetry is sent
        telemetryClient.Flush();
    }
}

```

À droite, sous Afficher les fichiers, sélectionnez **Ajouter**. Appelez le nouveau fichier **function.proj** avec la configuration suivante.

```C#
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights" Version="2.15.0" /> <!-- Ensure you’re using the latest version -->
    </ItemGroup>
</Project>

```

>[!div class="mx-imgBorder"]
>![À droite, sélectionnez Ajouter. Nommer le fichier function.proj](media/availability-azure-functions/addfile.png)

À droite, sous Afficher les fichiers, sélectionnez **Ajouter**. Appelez le nouveau fichier **runAvailabilityTest.csx** avec la configuration suivante.

```C#
public async static Task RunAvailbiltyTestAsync(ILogger log)
{
    // Add your business logic here.
    throw new NotImplementedException();
}

```

## <a name="check-availability"></a>Vérifier la disponibilité

Pour vérifier que tout fonctionne correctement, vous pouvez consulter le graphe sous l’onglet Disponibilité de votre ressource Application Insights.

> [!NOTE]
> Si vous avez implémenté votre propre logique métier dans runAvailabilityTest.csx, vous allez voir des résultats corrects comme dans les captures d’écran ci-dessous. Dans le cas contraire, les résultats sont incorrects. Les tests créés à l’aide de `TrackAvailability()` s’affichent avec **CUSTOM** à côté du nom du test.

>[!div class="mx-imgBorder"]
>![Onglet Disponibilité avec des résultats réussis](media/availability-azure-functions/availability-custom.png)

Pour voir les détails de la transaction de bout en bout, sélectionnez **Réussite** ou **Échec** dans Explorer, puis sélectionnez un exemple. Vous pouvez également accéder aux détails de la transaction de bout en bout en sélectionnant un point de données sur le graphe.

>[!div class="mx-imgBorder"]
>![Sélectionner un exemple de test de disponibilité](media/availability-azure-functions/sample.png)

>[!div class="mx-imgBorder"]
>![Détails de la transaction de bout en bout](media/availability-azure-functions/end-to-end.png)

Si vous avez tout exécuté tel quel (sans ajouter de logique métier), vous allez voir que le test a échoué.

## <a name="query-in-logs-analytics"></a>Interroger les journaux (Analytics)

Vous pouvez utiliser les journaux (Analytics) pour voir les résultats de la disponibilité, les dépendances, etc. Pour en savoir plus sur les journaux, consultez [Vue d’ensemble des requêtes de journal](../log-query/log-query-overview.md).

>[!div class="mx-imgBorder"]
>![Résultats de disponibilité](media/availability-azure-functions/availabilityresults.png)

>[!div class="mx-imgBorder"]
>![Screenshot shows New Query tab with dependencies limited to 50.](media/availability-azure-functions/dependencies.png)

## <a name="next-steps"></a>Étapes suivantes

- [Plan de l’application](./app-map.md)
- [Diagnostics de transaction](./transaction-diagnostics.md)

