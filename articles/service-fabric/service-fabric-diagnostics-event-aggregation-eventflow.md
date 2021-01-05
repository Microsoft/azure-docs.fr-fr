---
title: Agrégation d’événements Azure Service Fabric avec EventFlow
description: Découvrez l’agrégation et la collecte d’événements à l’aide d’EventFlow pour la surveillance et le diagnostic de clusters Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 2/25/2019
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: a80eea5a50aa7b1e441049eeb2cae381994cd3ec
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006334"
---
# <a name="event-aggregation-and-collection-using-eventflow"></a>Agrégation et collection d’événements avec EventFlow

[Microsoft Diagnostics EventFlow](https://github.com/Azure/diagnostics-eventflow) peut acheminer les événements d’un nœud vers une ou plusieurs destinations de surveillance. Étant donné que cet outil est inclus en tant que package NuGet dans votre projet de service, le code et la configuration d’EventFlow accompagnent le service, éliminant ainsi le problème de configuration par nœud mentionné plus haut à propos des diagnostics Azure. EventFlow s’exécute au sein de votre processus de service et se connecte directement aux sorties configurées. En raison de cette connexion directe, EventFlow fonctionne pour les déploiements d’un service sur Azure, dans un conteneur et en local. Faites preuve de prudence si vous exécutez EventFlow dans des scénarios de haute densité, par exemple dans un conteneur, car chaque pipeline EventFlow établit une connexion externe. Par conséquent, si vous hébergez plusieurs processus, cela donne lieu à plusieurs connexions sortantes. Ce n’est pas vraiment une préoccupation pour les applications Service Fabric, car tous les réplicas d’un événement `ServiceType` s’exécutent dans le même processus, ce qui limite le nombre de connexions sortantes. EventFlow propose également le filtrage des événements. Ainsi, seuls les événements correspondant au filtre spécifié sont envoyés.

## <a name="set-up-eventflow"></a>Configurer EventFlow

Les fichiers binaires EventFlow sont disponibles sous la forme d’un ensemble de packages NuGet. Pour ajouter EventFlow à un projet de service Service Fabric, cliquez sur le projet dans l’Explorateur de solutions et sélectionnez Gérer les packages NuGet. Basculez vers l’onglet Parcourir et recherchez « `Diagnostics.EventFlow` » :

![Packages NuGet EventFlow dans l’interface utilisateur du gestionnaire de package NuGet Visual Studio](./media/service-fabric-diagnostics-event-aggregation-eventflow/eventflow-nuget.png)

Une liste de différents packages s’affiche, étiquetés avec « Entrées » et « Sorties ». EventFlow prend en charge différents fournisseurs de journalisation et analyseurs. Le service qui héberge EventFlow doit inclure les packages appropriés en fonction de la source et de la destination des journaux d’activité d’application. Outre le package ServiceFabric principal, au moins un package Entrée et un package Sortie doivent également être configurés. Par exemple, vous pouvez ajouter les packages suivants pour envoyer des événements EventSource à Application Insights :

* `Microsoft.Diagnostics.EventFlow.Inputs.EventSource` (pour capturer les données à partir de la classe EventSource du service et d’EventSources standard tels que *Microsoft-ServiceFabric-Services* et *Microsoft-ServiceFabric-Actors*)
* `Microsoft.Diagnostics.EventFlow.Outputs.ApplicationInsights` (nous allons envoyer les journaux d’activité à une ressource Azure Application Insights)
* `Microsoft.Diagnostics.EventFlow.ServiceFabric` (permet d’initialiser le pipeline EventFlow à partir de la configuration du service Service Fabric et signale tout problème lié à l’envoi des données de diagnostic sous forme de rapports d’intégrité Service Fabric)

>[!NOTE]
>Le package `Microsoft.Diagnostics.EventFlow.Inputs.EventSource` nécessite le ciblage de .NET Framework 4.6 ou version ultérieure par le projet de service. Veillez à définir l’infrastructure cible appropriée dans les propriétés du projet avant d’installer ce package.

Une fois tous les packages installés, l’étape suivante consiste à configurer et à activer EventFlow dans le service.

## <a name="configure-and-enable-log-collection"></a>Configurer et activer la collecte de journaux
Le pipeline EventFlow chargé d’envoyer les journaux d’activité est créé à partir d’une spécification stockée dans un fichier de configuration. Le package `Microsoft.Diagnostics.EventFlow.ServiceFabric` installe un fichier de configuration EventFlow de démarrage dans le dossier de solution `PackageRoot\Config`, nommé `eventFlowConfig.json`. Ce fichier de configuration doit être modifié pour capturer les données à partir de la classe `EventSource` du service par défaut et toutes les autres entrées que vous voulez configurer, puis envoyer les données à l’emplacement approprié.

>[!NOTE]
>Si votre fichier de projet est au format Visual Studio 2017, le fichier `eventFlowConfig.json` n’est pas ajouté automatiquement. Pour corriger cela, créez le fichier dans le dossier `Config` et définissez l’action de génération `Copy if newer`. 

Voici un exemple d’*eventFlowConfig.json* basé sur les packages NuGet mentionnés ci-dessus :
```json
{
  "inputs": [
    {
      "type": "EventSource",
      "sources": [
        { "providerName": "Microsoft-ServiceFabric-Services" },
        { "providerName": "Microsoft-ServiceFabric-Actors" },
        // (replace the following value with your service's ServiceEventSource name)
        { "providerName": "your-service-EventSource-name" }
      ]
    }
  ],
  "filters": [
    {
      "type": "drop",
      "include": "Level == Verbose"
    }
  ],
  "outputs": [
    {
      "type": "ApplicationInsights",
      // (replace the following value with your AI resource's instrumentation key)
      "instrumentationKey": "00000000-0000-0000-0000-000000000000"
    }
  ],
  "schemaVersion": "2016-08-11"
}
```

Le nom de l’élément ServiceEventSource du service correspond à la valeur de la propriété Name de l’élément `EventSourceAttribute` appliqué à la classe ServiceEventSource. Tout cela est spécifié dans le fichier `ServiceEventSource.cs`, qui est inclus dans le code du service. Par exemple, dans l’extrait de code suivant, le nom de l’élément ServiceEventSource est *MyCompany-Application1-Stateless1* :

```csharp
[EventSource(Name = "MyCompany-Application1-Stateless1")]
internal sealed class ServiceEventSource : EventSource
{
    // (rest of ServiceEventSource implementation)
}
```

Notez que le fichier `eventFlowConfig.json` fait partie du package de configuration du service. Les modifications apportées à ce fichier peuvent être incluses dans les mises à niveau portant sur l’intégralité du service ou sur sa configuration uniquement, qui sont soumises aux contrôles d’intégrité et à la restauration automatique de Service Fabric en cas d’échec de la mise à niveau. Pour plus d’informations, consultez [Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md).

La section *filters* de la configuration vous permet de personnaliser davantage les informations qui vont passer à travers le pipeline EventFlow vers les sorties, ce qui vous permet de supprimer ou d’inclure certaines informations, ou de modifier la structure des données d’événement. Pour plus d’informations sur le filtrage, consultez les [Filtres EventFlow](https://github.com/Azure/diagnostics-eventflow#filters).

La dernière étape consiste à instancier le pipeline EventFlow dans le code de démarrage de votre service, qui se trouve dans le fichier `Program.cs` :

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using Microsoft.ServiceFabric;
using Microsoft.ServiceFabric.Services.Runtime;

// **** EventFlow namespace
using Microsoft.Diagnostics.EventFlow.ServiceFabric;

namespace Stateless1
{
    internal static class Program
    {
        /// <summary>
        /// This is the entry point of the service host process.
        /// </summary>
        private static void Main()
        {
            try
            {
                // **** Instantiate log collection via EventFlow
                using (var diagnosticsPipeline = ServiceFabricDiagnosticPipelineFactory.CreatePipeline("MyApplication-MyService-DiagnosticsPipeline"))
                {

                    ServiceRuntime.RegisterServiceAsync("Stateless1Type",
                    context => new Stateless1(context)).GetAwaiter().GetResult();

                    ServiceEventSource.Current.ServiceTypeRegistered(Process.GetCurrentProcess().Id, typeof(Stateless1).Name);

                    Thread.Sleep(Timeout.Infinite);
                }
            }
            catch (Exception e)
            {
                ServiceEventSource.Current.ServiceHostInitializationFailed(e.ToString());
                throw;
            }
        }
    }
}
```

Le nom transmis comme paramètre de la méthode `CreatePipeline` de l’élément `ServiceFabricDiagnosticsPipelineFactory` correspond au nom de l’*entité d’intégrité* représentant le pipeline de collection des journaux EventFlow. Ce nom est utilisé si EventFlow rencontre une erreur et la signale via le sous-système d’intégrité de Service Fabric.

### <a name="use-service-fabric-settings-and-application-parameters-in-eventflowconfig"></a>Utiliser les paramètres Service Fabric et les paramètres d’application dans eventFlowConfig

EventFlow prend en charge l’utilisation de paramètres Service Fabric et de paramètres d’application pour configurer ses paramètres. Vous pouvez faire référence aux paramètres Service Fabric à l’aide de la syntaxe spéciale suivante pour les valeurs :

```json
servicefabric:/<section-name>/<setting-name>
```

`<section-name>` est nom de la section de configuration de Service Fabric, et `<setting-name>` est le paramètre de configuration fournissant la valeur qui sera utilisée pour configurer un paramètre EventFlow. Pour plus d’informations sur la procédure à suivre, accédez à la rubrique [Prise en charge des paramètres Service Fabric et des paramètres d’application](https://github.com/Azure/diagnostics-eventflow#support-for-service-fabric-settings-and-application-parameters).

## <a name="verification"></a>Vérification

Démarrez votre service et observez la fenêtre de sortie Débogage de Visual Studio. Une fois le service démarré, vous devez commencer à voir des preuves qu’il envoie des enregistrements à la sortie que vous avez configurée. Accédez à votre plateforme d’analyse et de visualisation des événements et vérifiez que les journaux d’activité ont commencé à s’afficher (cela peut prendre quelques minutes).

## <a name="next-steps"></a>Étapes suivantes

* [Analyse et visualisation d’événements avec Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Analyse et visualisation d’événements avec les journaux d’activité Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
* [Documentation relative à EventFlow](https://github.com/Azure/diagnostics-eventflow)
