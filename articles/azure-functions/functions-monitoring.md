---
title: Surveiller l’exécution des fonctions Azure
description: Découvrez comment utiliser Azure Application Insights avec Azure Functions pour surveiller l’exécution des fonctions.
ms.assetid: 501722c3-f2f7-4224-a220-6d59da08a320
ms.topic: conceptual
ms.date: 10/14/2020
ms.custom: devx-track-csharp, fasttrack-edit, contperf-fy21q2, devx-track-js
ms.openlocfilehash: 7dbaa8712e09de9084e2bcb66d43f2181af292a0
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033032"
---
# <a name="monitor-azure-functions"></a>Surveiller l’exécution des fonctions Azure

[Azure Functions](functions-overview.md) est intégré à [Azure Application Insights](../azure-monitor/app/app-insights-overview.md) pour surveiller des fonctions. Cet article fournit une vue d’ensemble des fonctionnalités de surveillance fournies par Azure pour la surveillance d’Azure Functions.

Application Insights collecte des données de journal, de performances et d’erreur. Il détecte automatiquement les anomalies de performances et propose de puissants outils d’analyse, vous pouvez plus facilement diagnostiquer les problèmes et comprendre la manière dont vos fonctions sont utilisées. Ces outils sont conçus pour vous permettre d’améliorer continuellement les performances et la convivialité de vos fonctions. Vous pouvez également utiliser Application Insights lors du développement d'un projet d'application de fonction local. Pour plus d’informations, consultez [Présentation d’Application Insights](../azure-monitor/app/app-insights-overview.md).

L’instrumentation Application Insights étant intégrée à Azure Functions, une clé d'instrumentation valide est nécessaire pour connecter votre application de fonction à une ressource Application Insights. La clé d’instrumentation est ajoutée aux paramètres de votre application lorsque vous créez la ressource de votre application de fonction dans Azure. Si votre application de fonction n’a pas encore cette clé, vous pouvez la [définir manuellement](configure-monitoring.md#enable-application-insights-integration).  

## <a name="application-insights-pricing-and-limits"></a>Tarification et limites d’Application Insights

Vous pouvez tester l’intégration d’Application Insights avec Azure Functions gratuitement, avec une limite quotidienne de la quantité de données traitées.

Si vous activez Application Insights pendant le développement, vous pouvez atteindre cette limite pendant le test. Azure envoie des notifications sur le portail et par e-mail lorsque la limite quotidienne est proche. Si vous manquez ces alertes et atteignez la limite fixée, les nouveaux journaux d’activité n’apparaîtront pas dans les requêtes Application Insights. N’oubliez pas ces limites pour éviter de passer du temps à résoudre des problèmes superflus. Pour plus d’informations, consultez l’article [Gérer la tarification et le volume de données dans Application Insights](../azure-monitor/app/pricing.md).

> [!IMPORTANT]
> Application Insights présente une fonctionnalité [d’échantillonnage](../azure-monitor/app/sampling.md) qui peut vous éviter de produire une quantité excessive de données de télémétrie portant sur les exécutions terminées aux heures de forte activité. L’échantillonnage est activé par défaut. Si vous pensez qu’il vous manque des données, il vous faut peut-être simplement adapter les paramètres d’échantillonnage à votre scénario de surveillance. Pour plus d’informations, consultez [Configurer l’échantillonnage](configure-monitoring.md#configure-sampling).

La liste complète des fonctionnalités Application Insights disponibles pour votre application de fonction est détaillée dans [Fonctionnalités Application Insights prises en charge pour Azure Functions](../azure-monitor/app/azure-functions-supported-features.md).

## <a name="application-insights-integration"></a>Intégration d’Application Insights

En général, vous créez une instance d’Application Insights lorsque vous créez votre application de fonction. Dans ce cas, la clé d’instrumentation requise pour l’intégration est déjà définie en tant que paramètre d’application nommé *APPINSIGHTS_INSTRUMENTATIONKEY*. Si, pour une raison quelconque, votre application de fonction n’a pas de clé d’instrumentation définie, vous devez [activer l’intégration d’Application Insights](configure-monitoring.md#enable-application-insights-integration).  

## <a name="collecting-telemetry-data"></a>Collecte des données de télémétrie

Une fois l’intégration d’Application Insights activée, les données de télémétrie sont envoyées à votre instance d’Application Insights connectée. Ces données incluent des journaux générés par l’hôte Functions, des traces écrites à partir de votre code de fonctions et des données de performances. 

>[!NOTE]
>En plus des données de vos fonctions et de l’hôte Functions, vous pouvez également collecter des données à partir du [contrôleur d’échelle Functions](#scale-controller-logs).   

### <a name="log-levels-and-categories"></a>Niveaux et catégories de journalisation

Lorsque vous écrivez des traces à partir de votre code d’application, vous devez leur assigner un niveau de journalisation. Les niveaux de journalisation vous permettent de limiter la quantité de données collectées à partir de vos traces.  

[!INCLUDE [functions-log-levels](../../includes/functions-log-levels.md)]

Pour en savoir plus sur les niveaux de journalisation, consultez [Configurer les niveaux de journalisation](configure-monitoring.md#configure-log-levels).

En affectant des éléments journalisés à une catégorie, vous avez davantage de contrôle sur les données de télémétrie générées à partir de sources spécifiques dans votre application de fonction. Les catégories facilitent l’exécution d’analyses sur les données collectées. Les traces écrites à partir de votre code de fonction sont assignées à des catégories individuelles en fonction du nom de la fonction. Pour en savoir plus sur les catégories, consultez [Configurer des catégories](configure-monitoring.md#configure-categories).

### <a name="custom-telemetry-data"></a>Données de télémétrie personnalisées

En [C#](functions-dotnet-class-library.md#log-custom-telemetry-in-c-functions) et en [JavaScript](functions-reference-node.md#log-custom-telemetry), vous pouvez utiliser un kit de développement logiciel d’Application Insights pour écrire des données de télémétrie personnalisées.

### <a name="dependencies"></a>Dépendances

Depuis la version 2. x de Functions, le runtime collecte automatiquement les données sur les dépendances pour les liaisons qui utilisent certains kits de développement logiciel clients. Application Insights collecte des données sur les dépendances suivantes :

+ Azure Cosmos DB 
+ Azure Event Hubs
+ Azure Service Bus
+ Services de stockage Azure (Blob, file d’attente et table)

Les requêtes HTTP et les appels de base de données à l’aide de `SqlClient` sont également capturés. Pour obtenir la liste complète des dépendances prises en charge par Application Insights, consultez les [dépendances suivies automatiquement](../azure-monitor/app/asp-net-dependencies.md#automatically-tracked-dependencies).

Application Insights génère un _plan de l’application_ des données de dépendance collectées. Voici un exemple de plan d’application d’une fonction de déclencheur HTTP avec une liaison de sortie de stockage File d’attente.  

![Plan d’application avec dépendance](./media/functions-monitoring/app-map.png)

Les dépendances sont écrites au niveau `Information`. Si vous filtrez sur le niveau `Warning` ou supérieur, vous ne pouvez pas visualiser les données de dépendance. En outre, la collecte automatique des dépendances se produit en dehors de l’étendue utilisateur. Pour capturer des données de dépendance, assurez-vous que le niveau est défini sur au moins `Information` en dehors de l’étendue de l’utilisateur (`Function.<YOUR_FUNCTION_NAME>.User`) dans votre hôte.

En plus de la collecte automatique des données de dépendance, vous pouvez également utiliser l’un des kits de développement logiciel d’Application Insights propres aux langages pour écrire des informations de dépendance personnalisées dans les journaux. Pour obtenir un exemple d’écriture de dépendances personnalisées, consultez l’un des exemples de langages suivants :

+ [Journaliser des données de télémétrie personnalisées dans les fonctions C#](functions-dotnet-class-library.md#log-custom-telemetry-in-c-functions)
+ [Journaliser des données de télémétrie personnalisées dans les fonctions JavaScript](functions-reference-node.md#log-custom-telemetry) 

## <a name="writing-to-logs"></a>Écriture dans les journaux 

La façon dont vous écrivez dans les journaux et les API que vous utilisez dépend du langage de votre projet d’application de fonction.   
Pour en savoir plus sur l’écriture des journaux à partir de vos fonctions, consultez le guide du développeur pour votre langage.

+ [C# (Bibliothèque de classes .NET)](functions-dotnet-class-library.md#logging)
+ [Java](functions-reference-java.md#logger)
+ [JavaScript](functions-reference-node.md#write-trace-output-to-logs) 
+ [PowerShell](functions-reference-powershell.md#logging)
+ [Python](functions-reference-python.md#logging)

## <a name="streaming-logs"></a>Journaux d’activité en continu

Quand vous développez une application, vous voulez souvent voir ce qui est écrit dans les journaux en quasi temps réel lors de l’exécution dans Azure.

Il existe deux façons d’afficher un flux des données de journal générées par vos exécutions de fonction.

* **Streaming des journaux intégré** : la plateforme App Service vous permet d’afficher un flux de vos fichiers journaux d’application. Ce flux équivaut à la sortie observée quand vous déboguez vos fonctions au cours du [développement local](functions-develop-local.md) et quand vous utilisez l’onglet **Test** dans le portail. Toutes les informations basées sur les journaux sont affichées. Pour plus d’informations, consultez [Diffuser les journaux d’activité en continu](../app-service/troubleshoot-diagnostic-logs.md#stream-logs). Cette méthode de streaming ne prend en charge qu’une seule instance et ne peut pas être utilisée avec une application exécutée sur Linux dans un plan Consommation.

* **Flux de métriques temps réel** : quand votre application de fonction est [connectée à Application Insights](configure-monitoring.md#enable-application-insights-integration), vous pouvez voir les données des journaux et d’autres métriques en quasi temps réel dans le portail Azure avec [Flux de métriques temps réel](../azure-monitor/app/live-stream.md). Appliquez cette méthode lors de la supervision de fonctions s’exécutant sur plusieurs instances ou sur Linux dans un plan Consommation. Cette méthode utilise des [données échantillonnées](configure-monitoring.md#configure-sampling).

Les flux de journaux peuvent être affichés à la fois dans le portail et dans la plupart des environnements de développement locaux. Pour savoir comment activer les flux de journalisation, consultez [Activer les journaux d’exécution avec diffusion en continu dans Azure Functions](streaming-logs.md).

## <a name="diagnostic-logs"></a>Journaux de diagnostic

_Cette fonctionnalité est en préversion._ 

Application Insights vous permet d’exporter des données de télémétrie vers un stockage à long terme ou d’autres services d’analyse.  

Étant donné que Functions s’intègre également à Azure Monitor, vous pouvez également utiliser des paramètres de diagnostic pour envoyer des données de télémétrie à différentes destinations, y compris des journaux Azure Monitor. Pour en savoir plus, consultez [Monitorage d’Azure Functions avec Azure Monitor Logs](functions-monitor-log-analytics.md).

## <a name="scale-controller-logs"></a>Mettre à l'échelle les journaux de contrôleur

_Cette fonctionnalité est en préversion._ 

Le [contrôleur de mise à l’échelle Azure Functions](./functions-scale.md#runtime-scaling) surveille les instances de l’hôte Azure Functions sur lequel votre application s’exécute. Ce contrôleur prend des décisions concernant l’ajout ou la suppression d’instances en fonction des performances actuelles. Vous pouvez faire en sorte que le contrôleur de mise à l’échelle émette des journaux vers Application Insights pour mieux comprendre les décisions prises par ce contrôleur pour votre application de fonction. Vous pouvez également stocker les journaux générés dans le stockage d’objets blob à des fins d’analyse par un autre service. 

Pour activer cette fonctionnalité, vous ajoutez un paramètre d’application nommé `SCALE_CONTROLLER_LOGGING_ENABLED` à vos paramètres d’application de fonction. Pour en savoir plus, consultez [Configurer les journaux de contrôleur de mise à l’échelle](configure-monitoring.md#configure-scale-controller-logs).

## <a name="report-issues"></a>Signaler des problèmes

Pour signaler un problème avec l’intégration d’Application Insights dans Azure Functions, ou pour faire une suggestion ou une demande, [créez un problème dans GitHub](https://github.com/Azure/Azure-Functions/issues/new).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les ressources suivantes :

* [Application Insights](/azure/application-insights/)
* [Journalisation ASP.NET Core](/aspnet/core/fundamentals/logging/)
