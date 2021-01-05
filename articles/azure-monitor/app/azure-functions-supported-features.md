---
title: Azure Application Insights - Fonctionnalités prises en charge pour Azure Functions
description: Fonctionnalités Application Insights prises en charge pour Azure Functions
ms.topic: reference
author: TimothyMothra
ms.author: tilee
ms.date: 4/23/2019
ms.reviewer: mbullwin
ms.openlocfilehash: b44279f31aea8fc02130f1c3d7520f42c648bd4c
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607947"
---
# <a name="application-insights-for-azure-functions-supported-features"></a>Fonctionnalités Application Insights prises en charge pour Azure Functions

Azure Functions offre une [intégration prédéfinie](../../azure-functions/functions-monitoring.md) à Application Insights, elle est disponible via l’interface ILogger. La liste des fonctionnalités actuellement prises en charge est indiquée ci-dessous. Servez-vous du guide d’Azure Functions pour [Bien démarrer](../../azure-functions/configure-monitoring.md#enable-application-insights-integration).

Pour plus d’informations sur les versions du runtime Functions, voir [ici](../../azure-functions/functions-versions.md).

Pour plus d’informations sur les versions compatibles d’Application Insights, voir [Dépendances](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Logging.ApplicationInsights/).

## <a name="supported-features"></a>Fonctionnalités prises en charge

| Azure Functions                   | V1            | V2 et V3 | 
|-----------------------------------|---------------|------------------|
| | | | 
| **Collecte automatique de**        |               |                  |
| &bull; Requêtes                     | Oui           | Oui              |
| &bull; Exceptions                   | Oui           | Oui              |
| &bull; Compteurs de performances         | Oui           | Oui              |
| &bull; Dépendances                 |               |                  |
| &nbsp;&nbsp;&nbsp;&mdash; HTTP      |               | Oui              |
| &nbsp;&nbsp;&nbsp;&mdash; ServiceBus|               | Oui              |
| &nbsp;&nbsp;&nbsp;&mdash; EventHub  |               | Oui              |
| &nbsp;&nbsp;&nbsp;&mdash; SQL       |               | Oui              |
| | | | 
| **Fonctionnalités prises en charge**              |               |                  |
| &bull; Pulsation rapide/Métriques temps réel       | Oui           | Oui              | 
| &nbsp;&nbsp;&nbsp;&mdash; Canal de contrôle sécurisé |               | Oui | 
| &bull; Échantillonnage                     | Oui           | Oui              | 
| &bull; Pulsations                   | | Oui              | 
| | | |
| **Corrélation**                    |               |                  |
| &bull; ServiceBus                  |               | Oui              |
| &bull; EventHub                    |               | Oui              |
| | | | 
| **Configurable**                  |               |                  |           
| &bull; Entièrement configurable.<br/>Consultez [Azure Functions](https://github.com/Microsoft/ApplicationInsights-aspnetcore/issues/759#issuecomment-426687852) pour obtenir des instructions.<br/>Consultez [ASP.NET Core](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Custom-Configuration) pour connaître toutes les options.           |               | Oui                 | 

## <a name="performance-counters"></a>Compteurs de performance

La collecte automatique des compteurs de performances fonctionne uniquement sur les machines Windows.

## <a name="live-metrics--secure-control-channel"></a>Métriques en temps réel et canal de contrôle sécurisé

Les critères de filtres personnalisés que vous spécifiez sont renvoyés au composant de métriques temps réel dans le Kit de développement logiciel (SDK) Application Insights. Les filtres peuvent potentiellement contenir des informations sensibles telles que des ID clients. Vous pouvez sécuriser le canal avec une clé API secrète. Pour obtenir des instructions, consultez [Sécuriser le canal de contrôle](./live-stream.md#secure-the-control-channel).

## <a name="sampling"></a>échantillonnage

Azure Functions permet l’échantillonnage par défaut dans sa configuration. Pour plus d’informations, consultez [Configurer l’échantillonnage](../../azure-functions/configure-monitoring.md#configure-sampling).

Si votre projet prend une dépendance sur le kit de développement logiciel (SDK) Application Insights pour effectuer un suivi télémétrique manuel, vous constaterez peut-être un comportement inhabituel en présence d'une configuration d’échantillonnage différente de la configuration d’échantillonnage de Functions. 

Nous vous recommandons d’utiliser la même configuration que Functions. Avec **Functions v2**, vous pouvez obtenir la même configuration à l’aide de l’injection de dépendances dans votre constructeur :

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;

public class Function1 
{

    private readonly TelemetryClient telemetryClient;

    public Function1(TelemetryConfiguration configuration)
    {
        this.telemetryClient = new TelemetryClient(configuration);
    }

    [FunctionName("Function1")]
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req, ILogger logger)
    {
        this.telemetryClient.TrackTrace("C# HTTP trigger function processed a request.");
    }
}
```
