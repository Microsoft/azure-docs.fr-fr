---
title: Surveillance au niveau de l’application Azure Service Fabric
description: Découvrez les événements et journaux d’activité de niveau application et service utilisés pour surveiller et diagnostiquer les clusters Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 11/21/2018
ms.author: srrengar
ms.openlocfilehash: e1871df962a26def8c12000f8b8bc0cf31bae9a0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86247657"
---
# <a name="application-logging"></a>Journalisation des applications

L’instrumentation de votre code est un moyen d’obtenir des informations sur vos utilisateurs, mais également la seule façon de savoir si quelque chose est incorrect dans votre application et de diagnostiquer ce qui doit être corrigé. Bien qu’il soit techniquement possible de connecter un débogueur à un service de production, cette pratique n’est pas courante. Par conséquent, il est important de disposer de données d’instrumentation détaillées.

Certains produits instrumentent automatiquement votre code. Ces solutions peuvent s’avérer efficaces, mais une instrumentation manuelle doit presque toujours être spécifique à votre logique métier. Au final, vous devez disposer d’assez d’informations pour déboguer l’application de manière approfondie. Les applications Service Fabric peuvent être instrumentées avec n’importe quel framework de journalisation. Ce document décrit quelques approches différentes d’instrumentation de votre code et explique quand privilégier telle ou telle approche. 

Pour obtenir des exemples d’utilisation de ces suggestions, consultez [Ajouter une connexion à votre application Service Fabric](service-fabric-how-to-diagnostics-log.md).

## <a name="application-insights-sdk"></a>Kit de développement logiciel (SDK) Application Insights

Application Insights bénéficie d’une intégration riche et prête à l’emploi avec Service Fabric. Les utilisateurs peuvent ajouter les packages NuGet AI Service Fabric et ainsi recevoir des données et journaux d’activité créés et collectés tel qu’affichés dans le Portail Azure. En outre, ils sont invités à ajouter leurs propres données de télémétrie pour diagnostiquer et déboguer leurs applications, mais aussi pour effectuer le suivi des services et parties de leur application les plus utilisés. Dans le SDK, la classe [TelemetryClient](/dotnet/api/microsoft.applicationinsights.telemetryclient?view=azure-dotnet) offre de nombreuses façons d’effectuer le suivi des données de télémétrie dans vos applications. Regardez un exemple dans notre didacticiel montrant comment utiliser et ajouter Application Insights à votre application pour [surveiller et diagnostiquer une application .NET](service-fabric-tutorial-monitoring-aspnet.md).

## <a name="eventsource"></a>EventSource

Lorsque vous créez une solution Service Fabric à partir d’un modèle dans Visual Studio, une classe dérivée **EventSource** (**ServiceEventSource** ou **ActorEventSource**) est générée. Un modèle dans lequel vous pouvez ajouter des événements pour votre application ou votre service est créé. Le nom de la classe dérivée **EventSource** **doit** être unique et créé à partir de la chaîne de modèle par défaut MyCompany-&lt;solution&gt;-&lt;project&gt;. Un même nom attribué à plusieurs définitions **EventSource** peut causer des problèmes lors de l’exécution. Chaque événement défini doit avoir un identificateur unique. Si l’identificateur n’est pas unique, l’exécution échoue. Certaines organisations préaffectent des valeurs aux identificateurs afin d’éviter les conflits entre les équipes de développement. Pour plus d’informations, consultez le [blog de Vance](/archive/blogs/vancem/introduction-tutorial-logging-etw-events-in-c-system-diagnostics-tracing-eventsource) ou la [documentation MSDN](/previous-versions/msp-n-p/dn774985(v=pandp.20)).

## <a name="aspnet-core-logging"></a>Journalisation ASP.NET Core

Il est important de planifier avec soin la manière dont vous allez instrumenter votre code. Avec le plan d’instrumentation adéquat, vous pouvez éviter de déstabiliser votre base de code et d’avoir alors à réinstrumenter le code. Afin de réduire les risques, vous pouvez choisir une bibliothèque d’instrumentation telle que [Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging/), qui fait partie de Microsoft ASP.NET Core. ASP.NET Core présente une interface [ILogger](/dotnet/api/microsoft.extensions.logging.ilogger) que vous pouvez utiliser avec le fournisseur de votre choix, tout en limitant les répercussions sur le code existant. Vous pouvez utiliser le code dans ASP.NET Core sur Windows et Linux, ainsi que dans la version complète de .NET Framework. Par conséquent, votre code d’instrumentation est normalisé.

## <a name="next-steps"></a>Étapes suivantes

Une fois votre fournisseur de journalisation choisi pour l’instrumentation de vos applications et services, vos journaux d’activité et événements doivent être agrégés pour pouvoir être envoyés à une plateforme d’analyse. Pour mieux comprendre certaines des options recommandées d’Azure Monitor, lisez les informations relatives à [Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md) et [EventFlow](service-fabric-diagnostics-event-aggregation-eventflow.md).
