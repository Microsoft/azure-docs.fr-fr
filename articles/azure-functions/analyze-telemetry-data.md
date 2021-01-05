---
title: Analyser la télémétrie d’Azure Functions dans Application Insights
description: Découvrez comment afficher et interroger les données de télémétrie Azure Functions collectées par la fonctionnalité Azure Application Insights et stockées dans celle-ci.
ms.topic: how-to
ms.date: 10/14/2020
ms.custom: contperf-fy21q2
ms.openlocfilehash: 1d8d9cc9a7a4111e98b1d9141957769d6f157d45
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97027728"
---
# <a name="analyze-azure-functions-telemetry-in-application-insights"></a>Analyser la télémétrie d’Azure Functions dans Application Insights 

Azure Functions s’intègre à Application Insights pour vous permettre de mieux analyser vos applications de fonction. Application Insights collecte les données de télémétrie générées par votre application de fonction, y compris les informations que votre application écrit dans les journaux. L’intégration d’Application Insights est généralement activée lors de la création de votre application de fonction. Si votre application de fonction n’a pas de clé d’instrumentation définie, vous devez d’abord [activer l’intégration d’Application Insights](configure-monitoring.md#enable-application-insights-integration). 

Par défaut, les données collectées à partir de votre application de fonction sont stockées dans Application Insights. Dans le [portail Azure](https://portal.azure.com), Application Insights fournit un ensemble complet de visualisations de vos données de télémétrie. Vous pouvez parcourir les journaux d’erreurs et interroger les événements et les métriques. Cet article fournit des exemples de base montrant comment afficher et interroger les données que vous avez collectées. Pour plus d’informations sur l’exploration des données de votre application de fonction dans Azure Application Insights, consultez [Présentation d’Application Insights](../azure-monitor/app/app-insights-overview.md). 

Pour en savoir plus sur la conservation des données et les coûts de stockage potentiels, consultez [Collecte, rétention et stockage des données dans Application Insights](../azure-monitor/app/data-retention-privacy.md).   

## <a name="viewing-telemetry-in-monitor-tab"></a>Affichage de la télémétrie dans l’onglet Surveiller

Une fois l'[intégration d’Application Insights activée](configure-monitoring.md#enable-application-insights-integration), vous pouvez afficher des données de télémétrie dans l'onglet **Surveiller**.

1. Dans la page d’application de fonction, sélectionnez une fonction exécutée au moins une fois après la configuration d’Application Insights. Ensuite, sélectionnez **Surveiller** dans le volet gauche. Sélectionnez **Actualiser** régulièrement jusqu’à ce que la liste d’appels de fonction s’affiche.

   ![Liste d’appels](media/functions-monitoring/monitor-tab-ai-invocations.png)

    > [!NOTE]
    > Cette opération peut prendre jusqu’à cinq minutes, le temps que le client de télémétrie organise les données par lots pour les transmettre au serveur. Ce délai ne s’applique pas au [Flux de métriques temps réel](../azure-monitor/app/live-stream.md). Le service se connecte à l’hôte Functions lorsque vous chargez la page, de sorte que les journaux d’activité sont transmis directement sur la page.

1. Pour afficher les journaux d’activité liés à un appel de fonction spécifique, sélectionnez le lien de la colonne **Date (UTC)** correspondant à cet appel. La sortie de journalisation pour cet appel s’affiche dans une nouvelle page.

   ![Détails des appels](media/functions-monitoring/invocation-details-ai.png)

1. Choisissez **Exécuter dans Application Insights** pour afficher la source de la requête qui récupère les données de journal Azure Monitor dans Azure Log Analytics. Si c’est la première fois que vous utilisez Azure Log Analytics dans votre abonnement, vous êtes invité à l’activer.

1. Une fois que vous avez activé Log Analytics, la requête suivante s’affiche. Vous pouvez voir que les résultats de la requête sont limités aux 30 derniers jours (`where timestamp > ago(30d)`) et qu’ils n’affichent pas plus de 20 lignes (`take 20`). Par contre, la liste des détails des appels pour votre fonction porte sur les 30 derniers jours, sans limite.

   ![Liste d’appels Application Insights Analytics](media/functions-monitoring/ai-analytics-invocation-list.png)

Pour plus d’informations, consultez la section [Interroger les données de télémétrie](#query-telemetry-data) dans la suite de cet article.

## <a name="view-telemetry-in-application-insights"></a>Afficher les données de télémétrie dans Application Insights

Pour ouvrir Application Insights à partir d’une application de fonction dans le [portail Azure](https://portal.azure.com) :

1. Accédez à votre application de fonction dans le portail.

1. Sélectionnez **Application Insights** sous **Paramètres** dans la page de gauche. 

1. Si c’est la première fois que vous utilisez Application Insights avec votre abonnement, vous êtes invité à l’activer. Pour ce faire, sélectionnez **Activer Application Insights**, puis **Appliquer** sur la page suivante.

![Ouvrir Application Insights à partir de la page Vue d'ensemble de l'application de fonction](media/functions-monitoring/ai-link.png)

Pour plus d’informations sur l’utilisation d’Application Insights, consultez la [documentation d’Application Insights](/azure/application-insights/). Cette section présente des exemples montrant comment afficher les données dans Application Insights. Si vous êtes déjà familiarisé avec Application Insights, vous pouvez passer directement aux [sections sur la configuration et la personnalisation des données de télémétrie](configure-monitoring.md#configure-log-levels).

![Onglet Vue d'ensemble d’Application Insights](media/functions-monitoring/metrics-explorer.png)

Les domaines d’Application Insights suivants peuvent s'avérer utiles lors de l'évaluation du comportement, du niveau de performance et des erreurs de vos fonctions :

| Examiner | Description |
| ---- | ----------- |
| **[Échecs](../azure-monitor/app/asp-net-exceptions.md)** |  Créez des graphiques et des alertes basés sur les échecs de fonction et les exceptions de serveur. Le **Nom de l’opération** est le nom de la fonction. Les échecs de dépendances ne sont pas affichés, sauf si vous implémentez des données de télémétrie personnalisées pour les dépendances. |
| **[Niveau de performance](../azure-monitor/app/performance-counters.md)** | Analysez les problèmes de performances en consultant l’utilisation des ressources et le débit par **instances de rôle cloud**. Ces données de performances peuvent être utiles pour déboguer les scénarios où les fonctions ralentissent vos ressources sous-jacentes. |
| **[Métriques](../azure-monitor/platform/metrics-charts.md)** | Créez des graphiques et des alertes basés sur des métriques. Les métriques incluent le nombre d’appels de fonction, le délai d’exécution ainsi que les taux de réussite. |
| **[Métriques temps réel](../azure-monitor/app/live-stream.md)** | Visualisez les données des métriques au fil de leur création en quasi-temps réel. |

## <a name="query-telemetry-data"></a>Interroger les données de télémétrie

[Application Insights Analytics](../azure-monitor/log-query/log-query-overview.md) vous donne accès à toutes les données de télémétrie sous forme de tables de base de données. Analytics fournit un langage de requête pour l’extraction, la manipulation et la visualisation des données. 

Choisissez **Journaux** pour explorer ou rechercher les événements journalisés.

![Exemple Analytics](media/functions-monitoring/analytics-traces.png)

Voici un exemple de requête qui montre la distribution des demandes par nœud Worker au cours des 30 dernières minutes.

<pre>
requests
| where timestamp > ago(30m) 
| summarize count() by cloud_RoleInstance, bin(timestamp, 1m)
| render timechart
</pre>

Les tables disponibles sont affichées sous l’onglet **Schéma** situé à gauche. Les données générées par les appels de fonction sont disponibles dans les tables suivantes :

| Table de charge de travail | Description |
| ----- | ----------- |
| **traces** | Journaux créés par le runtime et traces de votre code de fonction. |
| **requests** | Une demande par appel de fonction. |
| **exceptions** | Toutes les exceptions levées par le runtime. |
| **customMetrics** | Nombre d’appels ayant réussi ou échoué, taux de réussite, durée. |
| **customEvents** | Événements suivis par le runtime, par exemple : requêtes HTTP qui déclenchent une fonction. |
| **performanceCounters** | Informations sur le niveau de performance des serveurs sur lesquels les fonctions sont en cours d’exécution. |

Les autres tables concernent les tests de disponibilité et les données de télémétrie client et navigateur. Vous pouvez implémenter une télémétrie personnalisée en vue d’y ajouter des données.

Dans chaque table, un champ `customDimensions` contient certaines des données spécifiques à Azure Functions.  Par exemple, la requête suivante récupère toutes les traces dont le niveau de journal est `Error`.

<pre>
traces 
| where customDimensions.LogLevel == "Error"
</pre>

Le runtime fournit les champs `customDimensions.LogLevel` et `customDimensions.Category`. Vous pouvez fournir des champs supplémentaires dans les journaux d’activité que vous écrivez dans votre code de fonction. Pour obtenir un exemple en C#, consultez [Journalisation structurée](functions-dotnet-class-library.md#structured-logging) dans le guide du développeur de la bibliothèque de classes .NET.

## <a name="consumption-plan-specific-metrics"></a>Mesures spécifiques au plan Consommation

Dans le cadre d’un [plan Consommation](functions-scale.md#consumption-plan), le *coût* d’exécution d’une fonction unique se mesure en *Go-secondes*. Le coût d’exécution est calculé en combinant l’utilisation de sa mémoire et sa durée d’exécution. Pour en savoir plus, consultez [Estimation des coûts d’un plan Consommation](functions-consumption-costs.md).

Les requêtes de télémétrie suivantes sont spécifiques aux métriques qui ont un impact sur le coût d’exécution des fonctions dans le plan Consommation.

[!INCLUDE [functions-consumption-metrics-queries](../../includes/functions-consumption-metrics-queries.md)]

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la supervision dans Azure Functions :

+ [Superviser Azure Functions](functions-monitoring.md)
+ [Comment configurer l’analyse pour Azure Functions](configure-monitoring.md)

