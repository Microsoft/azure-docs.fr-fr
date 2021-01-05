---
title: Configurer l’analyse pour Azure Functions
description: Découvrez comment connecter votre application de fonction à Application Insights à des fins de surveillance et de configuration de la collecte de données.
ms.date: 8/31/2020
ms.topic: how-to
ms.custom: contperf-fy21q2
ms.openlocfilehash: 77cc87e44990f471ef38871c0225835660779a3c
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825750"
---
# <a name="how-to-configure-monitoring-for-azure-functions"></a>Comment configurer l’analyse pour Azure Functions

Azure Functions s’intègre à Application Insights pour vous permettre de mieux analyser vos applications de fonction. Application Insights, fonctionnalité d’Azure Monitor, est un service extensible de gestion des performances des applications (APM) qui collecte les données générées par votre application de fonction, notamment les informations que votre application écrit dans les journaux. L’intégration d’Application Insights est généralement activée lors de la création de votre application de fonction. Si votre application n’a pas de clé d’instrumentation définie, vous devez d’abord [activer l’intégration d’Application Insights](#enable-application-insights-integration). 

Vous pouvez utiliser Application Insights sans en personnaliser la configuration. La configuration par défaut risque d’engendrer d'importants volumes de données. Si vous utilisez un abonnement Azure Visual Studio, vous risquez d’atteindre votre plafond de données pour Application Insights. Pour en savoir plus sur les coûts liés à Application Insights, consultez [Gérer l’utilisation et les coûts d’Application Insights](../azure-monitor/app/pricing.md).

Plus loin dans cet article, vous découvrirez comment configurer et personnaliser les données que vos fonctions envoient à Application Insights. La journalisation d'une application de fonction est configurée dans le fichier [host.json]. 

> [!NOTE]
> Vous pouvez utiliser des paramètres d’application spécialement configurés pour représenter des paramètres spécifiques dans un fichier host.json pour un environnement spécifique. Cela vous permet de modifier efficacement les paramètres host.json sans avoir à republier le fichier host.json dans votre projet. Pour plus d’informations, consultez [Substituer les valeurs de host.json](functions-host-json.md#override-hostjson-values).

## <a name="configure-categories"></a>Configurer des catégories

L’enregistreur d’événements d’Azure Functions inclut une *catégorie* par journal. La catégorie indique quelle partie du code d’exécution ou de votre code de fonction a écrit le journal. Les catégories diffèrent entre la version 1.x et les versions ultérieures. Le tableau suivant décrit les principales catégories de journaux d’activité générées par le runtime. 

# <a name="v2x"></a>[v2.x+](#tab/v2)

| Category | Table de charge de travail | Description |
| ----- | ----- | ----- |
| **`Function.<YOUR_FUNCTION_NAME>`** | **dependencies**| Les données de dépendance sont collectées automatiquement pour certains services. Pour les exécutions ayant réussi, ces journaux d’activité sont de niveau `Information`. Pour plus d'informations, consultez [Dépendances](functions-monitoring.md#dependencies). Les exceptions sont journalisées au niveau `Error`. Le runtime crée également des journaux de niveau `Warning`, par exemple lorsque des messages de la file d’attente sont envoyés à la file d’attente de [messages incohérents](functions-bindings-storage-queue-trigger.md#poison-messages). | 
| **`Function.<YOUR_FUNCTION_NAME>`** | **customMetrics**<br/>**customEvents** | Les SDK C# et JavaScript vous permettent de collecter des métriques personnalisées et de journaliser des événements personnalisés. Pour plus d’informations, consultez [Données de télémétrie personnalisées](functions-monitoring.md#custom-telemetry-data).|
| **`Function.<YOUR_FUNCTION_NAME>`** | **traces**| Inclut les journaux des fonctions démarrées et terminées pour des exécutions de fonctions spécifiques. Pour les exécutions ayant réussi, ces journaux d’activité sont de niveau `Information`. Les exceptions sont journalisées au niveau `Error`. Le runtime crée également des journaux de niveau `Warning`, par exemple lorsque des messages de la file d’attente sont envoyés à la file d’attente de [messages incohérents](functions-bindings-storage-queue-trigger.md#poison-messages). | 
| **`Function.<YOUR_FUNCTION_NAME>.User`** | **traces**| Les journaux générés par l’utilisateur, qui peuvent être de n’importe quel niveau de journalisation. Pour en savoir plus sur l’écriture dans les journaux à partir de vos fonctions, consultez la page sur les [écritures dans des journaux](functions-monitoring.md#writing-to-logs). | 
| **`Host.Aggregator`** | **customMetrics** | Ces journaux d’activité générés par le runtime fournissent des nombres et des moyennes d’appels de fonction sur une période de temps [configurable](#configure-the-aggregator). La période par défaut est 30 secondes ou 1 000 résultats, selon la première de ces éventualités. Le nombre d’exécutions, le taux de réussite et la durée en sont des exemples. Tous ces journaux sont écrits au niveau `Information`. Si vous filtrez sur le niveau `Warning` ou supérieur, vous ne pouvez pas visualiser ces données. |
| **`Host.Results`** | **requests** | Ces journaux générés par le runtime indiquent la réussite ou l’échec d’une fonction. Tous ces journaux sont écrits au niveau `Information`. Si vous filtrez sur le niveau `Warning` ou supérieur, vous ne pouvez pas visualiser ces données. |
| **`Microsoft`** | **traces** | Catégorie de journal complète qui reflète un composant du runtime .NET appelé par l’hôte.  |
| **`Worker`** | **traces** | Journaux générés par le processus de traitement des langages pour les langages autres que .NET. Les journaux de processus de traitement des langages peuvent également être journalisés dans une catégorie `Microsoft.*`, telle que `Microsoft.Azure.WebJobs.Script.Workers.Rpc.RpcFunctionInvocationDispatcher`. Ces journaux sont écrits au niveau `Information`.|

> [!NOTE]
> Pour les fonctions de la bibliothèque de classes .NET, ces catégories partent du principe que vous utilisez `ILogger` et non `ILogger<T>`. Pour plus d’informations, consultez la [documentation relative à Functions ILogger](functions-dotnet-class-library.md#ilogger). 

# <a name="v1x"></a>[v1.x](#tab/v1)

| Category | Table de charge de travail | Description |
| ----- | ----- | ----- |
| **`Function`** | **traces**| Les journaux générés par l’utilisateur, qui peuvent être de n’importe quel niveau de journalisation. Pour en savoir plus sur l’écriture dans les journaux à partir de vos fonctions, consultez la page sur les [écritures dans des journaux](functions-monitoring.md#writing-to-logs). | 
| **`Host.Aggregator`** | **customMetrics** | Ces journaux d’activité générés par le runtime fournissent des nombres et des moyennes d’appels de fonction sur une période de temps [configurable](#configure-the-aggregator). La période par défaut est 30 secondes ou 1 000 résultats, selon la première de ces éventualités. Le nombre d’exécutions, le taux de réussite et la durée en sont des exemples. Tous ces journaux sont écrits au niveau `Information`. Si vous filtrez sur le niveau `Warning` ou supérieur, vous ne pouvez pas visualiser ces données. |
| **`Host.Executor`** | **traces** | Inclut les journaux des **fonctions démarrées** et **terminées** pour des exécutions de fonctions spécifiques. Pour les exécutions ayant réussi, ces journaux d’activité sont de niveau `Information`. Les exceptions sont journalisées au niveau `Error`. Le runtime crée également des journaux de niveau `Warning`, par exemple lorsque des messages de la file d’attente sont envoyés à la file d’attente de [messages incohérents](functions-bindings-storage-queue-trigger.md#poison-messages).  |
| **`Host.Results`** | **requests** | Ces journaux générés par le runtime indiquent la réussite ou l’échec d’une fonction. Tous ces journaux sont écrits au niveau `Information`. Si vous filtrez sur le niveau `Warning` ou supérieur, vous ne pouvez pas visualiser ces données. |

---

La colonne **Table** indique dans quelle table Application Insights le journal est écrit. 

## <a name="configure-log-levels"></a>Configurer des niveaux de journaux

[!INCLUDE [functions-log-levels](../../includes/functions-log-levels.md)]

Pour chaque catégorie, vous indiquez le niveau de journal minimal à envoyer. Les paramètres host.json varient en fonction de la [version du runtime Functions](functions-versions.md). 

L’exemple ci-dessous définit la journalisation en fonction des règles suivantes :

+ Pour les journaux de `Host.Results` ou de `Function`, seuls les événements sont journalisés dans `Error` ou à un niveau supérieur. 
+ Pour les journaux de `Host.Aggregator`, journalisez toutes les métriques générées (`Trace`).
+ Pour tous les autres journaux, notamment les journaux des utilisateurs, journalisez uniquement les événements de niveau `Information` et supérieur.

# <a name="v2x"></a>[v2.x+](#tab/v2)

```json
{
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error",
      "Function": "Error",
      "Host.Aggregator": "Trace"
    }
  }
}
```

# <a name="v1x"></a>[v1.x](#tab/v1) 

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host.Results": "Error",
        "Function": "Error",
        "Host.Aggregator": "Trace"
      }
    }
  }
}
```

---

Si [host.json] inclut plusieurs journaux qui commencent par la même chaîne, les journaux les plus définis sont traités en priorité. Prenons l’exemple suivant qui journalise tout dans le runtime, à l’exception de `Host.Aggregator`, au niveau de `Error` :

# <a name="v2x"></a>[v2.x+](#tab/v2)

```json
{
  "logging": {
    "fileLoggingMode": "always",
    "logLevel": {
      "default": "Information",
      "Host": "Error",
      "Function": "Error",
      "Host.Aggregator": "Information"
    }
  }
}
```

# <a name="v1x"></a>[v1.x](#tab/v1) 

```json
{
  "logger": {
    "categoryFilter": {
      "defaultLevel": "Information",
      "categoryLevels": {
        "Host": "Error",
        "Function": "Error",
        "Host.Aggregator": "Information"
      }
    }
  }
}
```

---

Vous pouvez utiliser un paramètre de niveau de journalisation `None` pour empêcher l’écriture de journaux pour une catégorie. 

## <a name="configure-the-aggregator"></a>Configurer le paramètre d’agrégation

Comme indiqué dans la section précédente, le runtime agrège les données sur les exécutions de fonction sur une période de temps. La période par défaut est 30 secondes ou 1 000 exécutions, selon la première de ces éventualités. Vous pouvez configurer ce paramètre dans le fichier [host.json].  Voici un exemple :

```json
{
    "aggregator": {
      "batchSize": 1000,
      "flushTimeout": "00:00:30"
    }
}
```

## <a name="configure-sampling"></a>Configurer l’échantillonnage

Application Insights présente une fonctionnalité [d’échantillonnage](../azure-monitor/app/sampling.md) qui peut vous éviter de produire une quantité excessive de données de télémétrie portant sur les exécutions terminées aux heures de forte activité. Lorsque le taux d'exécutions entrantes dépasse un seuil spécifié, Application Insights commence à ignorer de manière aléatoire certaines exécutions entrantes. Par défaut, le nombre maximal d’exécutions par seconde est fixé à 20 (cinq dans la version 1.x). Vous pouvez configurer l’échantillonnage dans [host.json](./functions-host-json.md#applicationinsights).  Voici un exemple :

# <a name="v2x"></a>[v2.x+](#tab/v2)

```json
{
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond" : 20,
        "excludedTypes": "Request;Exception"
      }
    }
  }
}
```


Vous pouvez exclure certains types de données de télémétrie de l’échantillonnage. Dans cet exemple, les données de type `Request` et `Exception` sont exclues de l’échantillonnage. Cela garantit la journalisation de *toutes* les exécutions de fonction (requêtes) et exceptions, tandis que d’autres types de données de télémétrie restent soumis à l’échantillonnage. 

# <a name="v1x"></a>[v1.x](#tab/v1)  

```json
{
  "applicationInsights": {
    "sampling": {
      "isEnabled": true,
      "maxTelemetryItemsPerSecond" : 5
    }
  }
}
```
---

Pour plus d’informations, consultez [Échantillonnage dans Application Insights](../azure-monitor/app/sampling.md).

## <a name="configure-scale-controller-logs"></a>Configurer les journaux de contrôleur d’échelle

_Cette fonctionnalité est en préversion._ 

Vous pouvez faire en sorte que le [contrôleur d’échelle Azure Functions](./functions-scale.md#runtime-scaling) émette des journaux vers Application Insights ou vers le stockage Blob pour mieux comprendre les décisions prises par ce contrôleur pour votre application de fonction.

Pour activer cette fonctionnalité, vous ajoutez un paramètre d’application nommé `SCALE_CONTROLLER_LOGGING_ENABLED` à vos paramètres d’application de fonction. La valeur de ce paramètre doit être au format `<DESTINATION>:<VERBOSITY>`, en fonction des éléments suivants :

[!INCLUDE [functions-scale-controller-logging](../../includes/functions-scale-controller-logging.md)]

Par exemple, la commande Azure CLI suivante active la journalisation détaillée à partir du contrôleur de mise à l’échelle pour Application Insights :

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings SCALE_CONTROLLER_LOGGING_ENABLED=AppInsights:Verbose
```

Dans cet exemple, remplacez `<FUNCTION_APP_NAME>` et `<RESOURCE_GROUP_NAME>` par le nom de votre application de fonction et le nom du groupe de ressources, respectivement. 

La commande Azure CLI suivante désactive la journalisation en définissant le niveau de détail sur `None` :

```azurecli-interactive
az functionapp config appsettings set --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--settings SCALE_CONTROLLER_LOGGING_ENABLED=AppInsights:None
```

Vous pouvez également désactiver la journalisation en supprimant le paramètre `SCALE_CONTROLLER_LOGGING_ENABLED` à l’aide de la commande Azure CLI suivante :

```azurecli-interactive
az functionapp config appsettings delete --name <FUNCTION_APP_NAME> \
--resource-group <RESOURCE_GROUP_NAME> \
--setting-names SCALE_CONTROLLER_LOGGING_ENABLED
```

## <a name="enable-application-insights-integration"></a>Activer l’intégration à Application Insights

Pour qu’une application de fonction envoie des données à Application Insights, elle a besoin de connaître la clé d’instrumentation d’une ressource Application Insights. Cette clé doit se trouver dans un paramètre d’application nommé **APPINSIGHTS_INSTRUMENTATIONKEY**.

Lorsque vous créez votre application de fonction dans le [portail Azure](functions-create-first-azure-function.md), à partir de la ligne de commande avec [Azure Functions Core Tools](./create-first-function-cli-csharp.md), ou à l’aide de [Visual Studio Code](./create-first-function-vs-code-csharp.md), l’intégration d’Application Insights est activée par défaut. La ressource Application Insights, qui porte le même nom que votre application de fonction est créée dans la même région ou dans la région la plus proche.

### <a name="new-function-app-in-the-portal"></a>Nouvelle application de fonction dans le portail

Pour examiner la ressource Application Insights en cours de création, sélectionnez-la de manière à développer la fenêtre **Application Insights**. Vous pouvez modifier le **Nouveau nom de ressource** ou choisir un autre **Emplacement** dans une [Zone géographique Azure](https://azure.microsoft.com/global-infrastructure/geographies/) où vous souhaitez stocker vos données.

![Activer Application Insights pendant la création d’une application de fonction](media/functions-monitoring/enable-ai-new-function-app.png)

Lorsque vous sélectionnez **Créer**, une ressource Application Insights est créée avec votre application de fonction, et `APPINSIGHTS_INSTRUMENTATIONKEY` est défini dans les paramètres de l’application. Tout est prêt.

<a id="manually-connect-an-app-insights-resource"></a>
### <a name="add-to-an-existing-function-app"></a>Ajouter à une application de fonction existante 

Si une ressource Application Insights n’a pas été créée avec votre application de fonction, procédez comme suit pour créer la ressource. Vous pouvez ensuite ajouter la clé d’instrumentation de cette ressource en tant que [paramètre d’application](functions-how-to-use-azure-function-app-settings.md#settings) dans votre application de fonction.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Application de fonction**, puis choisissez votre application de fonction. 

1. Sélectionnez la bannière **Application Insights n'est pas configuré** en haut de la fenêtre. Si vous ne voyez pas cette bannière, cela signifie que votre application a probablement déjà Application Insights activé.

    :::image type="content" source="media/configure-monitoring/enable-application-insights.png" alt-text="Activer Application Insights à partir du portail":::

1. Développez **Changer votre ressource** et créez une ressource Application Insights en utilisant les paramètres spécifiés dans le tableau suivant.  

    | Paramètre      | Valeur suggérée  | Description                                        |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nouveau nom de la ressource** | Nom d’application unique | Il est plus facile d’utiliser le même nom que celui de votre application de fonction, qui doit être unique dans votre abonnement. | 
    | **Lieu** | Europe Ouest | Si possible, utilisez la même [région](https://azure.microsoft.com/regions/) que celle de votre application de fonction ou une région proche. |

    :::image type="content" source="media/configure-monitoring/ai-general.png" alt-text="Création d’une ressource Application Insights":::

1. Sélectionnez **Appliquer**. 

   La ressource Application Insights est créée dans le même groupe de ressources et le même abonnement que votre application de fonction. Une fois la ressource créée, fermez la fenêtre Application Insights.

1. Dans votre application de fonction, sélectionnez **Configuration** sous **Paramètres**, puis sélectionnez **Paramètres d'application**. Si vous voyez un paramètre nommé `APPINSIGHTS_INSTRUMENTATIONKEY`, cela signifie que l’intégration d’Application Insights est activée pour votre application de fonction s’exécutant dans Azure. Si, pour une raison quelconque, ce paramètre n’existe pas, ajoutez-le en utilisant votre clé d’instrumentation Application Insights comme valeur.

> [!NOTE]
> Les premières versions de Functions utilisaient la surveillance intégrée, ce qui n’est plus recommandé. Lorsque vous activez l’intégration d’Application Insights pour un telle application de fonction, vous devez également [désactiver la journalisation intégrée](#disable-built-in-logging).  

## <a name="disable-built-in-logging"></a>Désactiver la journalisation intégrée

Lorsque vous activez Application Insights, désactivez la journalisation intégrée qui utilise le Stockage Azure. La journalisation intégrée permet de tester des charges de travail légères, mais n'a pas vocation à être utilisée en production avec des charges importantes. Nous recommandons Application Insights à des fins de surveillance de la production. Si la journalisation intégrée est utilisée en production, l’enregistrement de journal peut être incomplet en raison d’une limitation du Stockage Azure.

Pour désactiver la journalisation intégrée, supprimez le paramètre d’application `AzureWebJobsDashboard`. Pour plus d’informations sur la suppression de paramètres d’application dans le portail Azure, consultez la section **Paramètres de l’application** dans [Comment gérer une application de fonction dans le portail Azure](functions-how-to-use-azure-function-app-settings.md#settings). Avant de supprimer le paramètre d’application, assurez-vous qu’aucune fonction existante dans la même application de fonction ne l’utilise pour les déclencheurs ou les liaisons du Stockage Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la supervision, consultez :

+ [Superviser Azure Functions](functions-monitoring.md)
+ [Analyser les données de télémétrie Azure Functions dans Application Insights](analyze-telemetry-data.md)
+ [Application Insights](/azure/application-insights/)


[host.json]: functions-host-json.md
