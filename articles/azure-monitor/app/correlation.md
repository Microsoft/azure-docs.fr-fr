---
title: Corrélation de télémétrie dans Azure Application Insights | Microsoft Docs
description: Corrélation de télémétrie dans Application Insights
ms.topic: conceptual
author: lgayhardt
ms.author: lagayhar
ms.date: 06/07/2019
ms.reviewer: sergkanz
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 20e9ed7e83ff3359651acebc11a939a998f2889d
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607913"
---
# <a name="telemetry-correlation-in-application-insights"></a>Corrélation de télémétrie dans Application Insights

Dans l’environnement des microservices, les tâches associées à chaque opération logique doivent être exécutées dans différents composants du service. Vous pouvez superviser séparément chacun de ces composants en utilisant [Application Insights](../../azure-monitor/app/app-insights-overview.md). Application Insights prend en charge la corrélation de télémétrie distribuée, qui vous permet de détecter le composant responsable des défaillances ou de la dégradation des performances.

Cet article décrit le modèle de données utilisé par Application Insights pour mettre en corrélation les données de télémétrie envoyées par plusieurs composants. Il couvre les protocoles et les techniques de propagation de contexte. Il traite également de l’implémentation des tactiques de corrélation sur différentes plateformes et dans différents langages.

## <a name="data-model-for-telemetry-correlation"></a>Modèle de données pour la corrélation de télémétrie

Application Insights définit le [modèle de données](../../azure-monitor/app/data-model.md) pour la corrélation de télémétrie distribuée. Pour associer la télémétrie à une opération logique, chaque élément de télémétrie comporte un champ de contexte appelé `operation_Id`. Cet identificateur est partagé par chaque élément de télémétrie dans la trace distribuée. Ainsi, même si vous perdez la télémétrie d’une seule couche, vous pouvez toujours associer la télémétrie communiquée par d’autres composants.

Une opération logique distribuée se compose généralement d’un ensemble d’opérations plus petites qui sont des requêtes traitées par un des composants. Ces opérations sont définies par la [télémétrie des requêtes](../../azure-monitor/app/data-model-request-telemetry.md). Chaque élément de télémétrie d’une requête a son propre `id`, qui l’identifie de façon univoque et globale. Par ailleurs, tous les éléments de télémétrie (comme les traces et les exceptions) associés à la requête doivent définir le `operation_parentId` sur la valeur de l’`id` de la requête.

Chaque opération sortante, telle qu’un appel HTTP vers un autre composant, est représentée par la [télémétrie des dépendances](../../azure-monitor/app/data-model-dependency-telemetry.md). La télémétrie des dépendances définit également ses propres `id` qui sont globalement uniques. La télémétrie des requêtes, lancée par cet appel de dépendances, utilise cet `id` comme `operation_parentId`.

Vous pouvez générer une vue de l’opération logique distribuée en utilisant `operation_Id`, `operation_parentId` et `request.id` avec `dependency.id`. Ces champs définissent également l’ordre de causalité des appels de télémétrie.

Dans un environnement de microservices, les traces des composants peuvent se diriger vers différents éléments de stockage. Chaque composant peut avoir sa propre clé d’instrumentation dans Application Insights. Pour obtenir la télémétrie pour l’opération logique, Application Insights interroge les données de chaque élément de stockage. Quand le nombre d’éléments de stockage est important, vous avez besoin d’une indication quant à l’endroit à regarder ensuite. Le modèle de données Application Insights définit deux champs pour résoudre ce problème : `request.source` et `dependency.target`. Le premier champ identifie le composant qui a lancé la demande de dépendance. Le deuxième champ identifie le composant qui a retourné la réponse de l’appel de dépendance.

## <a name="example"></a>Exemple

Intéressons-nous à un exemple. Une application appelée Stock Prices affiche le cours actuel sur le marché d’une action en utilisant une API externe nommée Stock. L’application Stock Prices a une page nommée Stock que le navigateur web client ouvre en utilisant `GET /Home/Stock`. L’application interroge l’API Stock en utilisant l’appel HTTP `GET /api/stock/value`.

Vous pouvez analyser la télémétrie obtenue en exécutant une requête :

```kusto
(requests | union dependencies | union pageViews)
| where operation_Id == "STYz"
| project timestamp, itemType, name, id, operation_ParentId, operation_Id
```

Dans les résultats, notez que tous les éléments de télémétrie partagent la racine `operation_Id`. Quand un appel Ajax est effectué à partir de la page, un nouvel ID unique (`qJSXU`) est affecté à la télémétrie des dépendances, et l’ID de la pageView est utilisé en tant que `operation_ParentId`. La requête de serveur utilise ensuite l’ID Ajax en tant que `operation_ParentId`.

| itemType   | name                      | id           | operation_ParentId | operation_Id |
|------------|---------------------------|--------------|--------------------|--------------|
| pageView   | Stock page                |              | STYz               | STYz         |
| dependency | GET /Home/Stock           | qJSXU        | STYz               | STYz         |
| request    | GET Home/Stock            | KqKwlrSt9PA= | qJSXU              | STYz         |
| dependency | GET /api/stock/value      | bBrf2L7mm2g= | KqKwlrSt9PA=       | STYz         |

Quand l’appel `GET /api/stock/value` est effectué vers un service externe, vous devez connaître l’identité de ce serveur afin de pouvoir définir le champ `dependency.target` de façon appropriée. Quand le service externe ne prend pas en charge la supervision, `target` prend comme valeur le nom d’hôte du service (par exemple `stock-prices-api.com`). Cependant, si le service s’identifie lui-même en retournant un en-tête HTTP prédéfini, `target` contient l’identité du service qui permet à Application Insights de générer une trace distribuée en interrogeant la télémétrie provenant de ce service.

## <a name="correlation-headers-using-w3c-tracecontext"></a>En-têtes de corrélation à l’aide de W3C TraceContext

Application Insights effectue la transition vers [W3C Trace-Context](https://w3c.github.io/trace-context/), qui définit :

- `traceparent`: représente l’ID d’opération globalement unique et l’identificateur unique de l’appel.
- `tracestate`: Représente le contexte de traçage spécifique au système.

La dernière version du SDK Application Insights prend en charge le protocole Trace-Context, mais il peut être nécessaire d’y adhérer. (La compatibilité descendante avec le protocole de corrélation précédent pris en charge par le SDK Application Insights sera conservée.)

Le [protocole HTTP de corrélation, également appelé Request-Id](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/HttpCorrelationProtocol.md) est déprécié. Ce protocole définit deux en-têtes :

- `Request-Id`: représente le GUID de l’appel.
- `Correlation-Context`: représente la collection de paires nom-valeur des propriétés de trace distribuée.

Application Insights définit également l’[extension](https://github.com/lmolkova/correlation/blob/master/http_protocol_proposal_v2.md) pour le protocole HTTP de corrélation. Il utilise des paires nom-valeur `Request-Context` pour propager la collection de propriétés utilisée par l’appelant ou l’appelé. Le SDK Application Insights utilise cet en-tête pour définir les champs `dependency.target` et `request.source`.

Les modèles de données [W3C Trace-Context](https://w3c.github.io/trace-context/) et Application Insights correspondent de la façon suivante :

| Application Insights                   | W3C TraceContext                                      |
|------------------------------------    |-------------------------------------------------|
| `Id` de `Request` et `Dependency`     | [parent-id](https://w3c.github.io/trace-context/#parent-id)                                     |
| `Operation_Id`                         | [trace-id](https://w3c.github.io/trace-context/#trace-id)                                           |
| `Operation_ParentId`                   | [parent-id](https://w3c.github.io/trace-context/#parent-id) de l’étendue parente de cette étendue. S’il s’agit d’une étendue racine, ce champ doit être vide.     |

Pour plus d’informations, consultez le [Modèle de données de télémétrie d’Application Insights](../../azure-monitor/app/data-model.md).

### <a name="enable-w3c-distributed-tracing-support-for-net-apps"></a>Activer la prise en charge du suivi distribué W3C pour les applications .NET

Le suivi distribué basé sur W3C TraceContext est activé par défaut dans tous les kits de développement logiciel (SDK) .NET Framework/.NET Core récents, ainsi que la compatibilité descendante avec le protocole request-id hérité.

### <a name="enable-w3c-distributed-tracing-support-for-java-apps"></a>Activer la prise en charge du traçage distribué W3C pour les applications Java

#### <a name="java-30-agent"></a>Agent Java 3.0

  L’agent Java 3.0 prend en charge W3C prêt à l'emploi et aucune configuration supplémentaire n’est requise. 

#### <a name="java-sdk"></a>Kit de développement logiciel (SDK) Java
- **Configuration d’entrée**

  - Pour les applications Java EE, ajoutez ce qui suit à la balise `<TelemetryModules>` dans le fichier ApplicationInsights.xml :

    ```xml
    <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule>
       <Param name = "W3CEnabled" value ="true"/>
       <Param name ="enableW3CBackCompat" value = "true" />
    </Add>
    ```

  - Pour les applications Spring Boot, ajoutez ces propriétés :

    - `azure.application-insights.web.enable-W3C=true`
    - `azure.application-insights.web.enable-W3C-backcompat-mode=true`

- **Configuration de sortie**

  Ajoutez le code suivant au fichier AI-Agent.xml :

  ```xml
  <Instrumentation>
    <BuiltIn enabled="true">
      <HTTP enabled="true" W3C="true" enableW3CBackCompat="true"/>
    </BuiltIn>
  </Instrumentation>
  ```

  > [!NOTE]
  > Le mode de compatibilité descendante est activé par défaut et le paramètre `enableW3CBackCompat` est facultatif. Utilisez-le uniquement quand vous souhaitez désactiver la compatibilité descendante.
  >
  > Dans l’idéal, vous la désactiveriez quand tous vos services auraient été mis à jour vers une version plus récente des SDK prenant en charge le protocole W3C. Nous vous recommandons vivement de déplacer ces SDK plus récents dès que possible.

> [!IMPORTANT]
> Vérifiez que les configurations entrante et sortante sont rigoureusement identiques.

### <a name="enable-w3c-distributed-tracing-support-for-web-apps"></a>Activer la prise en charge du suivi distribué W3C pour les applications web

Cette fonctionnalité se trouve dans `Microsoft.ApplicationInsights.JavaScript`. Elle est désactivée par défaut. Pour l’activer, utilisez la configuration `distributedTracingMode`. AI_AND_W3C est fourni à des fins de compatibilité descendante avec tous les services instrumentés par Application Insights hérités.

- **[Configuration basée sur npm](./javascript.md#npm-based-setup)**

Ajouter la configuration suivante :
  ```JavaScript
    distributedTracingMode: DistributedTracingModes.W3C
  ```

- **[Configuration basée sur un extrait](./javascript.md#snippet-based-setup)**

Ajouter la configuration suivante :
  ```
      distributedTracingMode: 2 // DistributedTracingModes.W3C
  ```
> [!IMPORTANT] 
> Pour afficher toutes les configurations requises pour activer la corrélation, consultez la [documentation relative à la corrélation JavaScript](./javascript.md#enable-correlation).

## <a name="telemetry-correlation-in-opencensus-python"></a>Corrélation de télémétrie dans OpenCensus Python

OpenCensus Python prend en charge [W3 Trace-Context](https://w3c.github.io/trace-context/) sans nécessiter de configuration supplémentaire.

À titre de référence, le modèle de données OpenCensus se trouve [ici](https://github.com/census-instrumentation/opencensus-specs/tree/master/trace).

### <a name="incoming-request-correlation"></a>Corrélation de requêtes entrantes

OpenCensus Python met en corrélation les en-têtes Trace-Context W3C des requêtes entrantes avec les intervalles générés à partir des requêtes elles-mêmes. OpenCensus effectue cette opération automatiquement en s’intégrant à ces frameworks d’application web populaires : Flask, Django et Pyramid. Vous devez seulement renseigner les en-têtes Trace-Context de W3C avec le [format correct](https://www.w3.org/TR/trace-context/#trace-context-http-headers-format), puis les envoyer avec la requête. Voici un exemple d’application Flask qui illustre ceci :

```python
from flask import Flask
from opencensus.ext.azure.trace_exporter import AzureExporter
from opencensus.ext.flask.flask_middleware import FlaskMiddleware
from opencensus.trace.samplers import ProbabilitySampler

app = Flask(__name__)
middleware = FlaskMiddleware(
    app,
    exporter=AzureExporter(),
    sampler=ProbabilitySampler(rate=1.0),
)

@app.route('/')
def hello():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='localhost', port=8080, threaded=True)
```

Ce code exécute un exemple d’application Flask sur votre machine locale, en écoutant le port `8080`. Pour corréler le contexte de trace, vous envoyez une requête au point de terminaison. Dans cet exemple, vous pouvez utiliser une commande `curl` :
```
curl --header "traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01" localhost:8080
```
En examinant le [format de l’en-tête Trace-Context](https://www.w3.org/TR/trace-context/#trace-context-http-headers-format), vous pouvez déduire les informations suivantes :

`version`: `00`

`trace-id`: `4bf92f3577b34da6a3ce929d0e0e4736`

`parent-id/span-id`: `00f067aa0ba902b7`

`trace-flags`: `01`

Si vous examinez l’entrée de la requête envoyée à Azure Monitor, vous pouvez voir des champs renseignés avec les informations d’en-tête de trace. Vous trouverez ces données sous Journaux (Analytics) dans la ressource Azure Monitor Application Insights.

![Télémétrie des requêtes dans les journaux (Analytics)](./media/opencensus-python/0011-correlation.png)

Le champ `id` est au format `<trace-id>.<span-id>`, où `trace-id` provient de l’en-tête de trace passé dans la requête, et où `span-id` est un tableau de 8 octets généré pour cette étendue.

Le champ `operation_ParentId` est au format `<trace-id>.<parent-id>`, où `trace-id` et `parent-id` sont extraits de l’en-tête de trace passé dans la requête.

### <a name="log-correlation"></a>Corrélation des journaux

OpenCensus Python vous permet de mettre en corrélation des journaux en ajoutant un ID de trace, un ID d’étendue et un indicateur d’échantillonnage aux enregistrements de journal. Vous ajoutez ces attributs en installant l’[intégration de la journalisation](https://pypi.org/project/opencensus-ext-logging/) d’OpenCensus. Les attributs suivants vont être ajoutés aux objets `LogRecord` de Python : `traceId`, `spanId` et `traceSampled`. Notez que ceci prend effet uniquement pour les journaliseurs créés après l’intégration.

Voici un exemple d’application qui illustre cela :

```python
import logging

from opencensus.trace import config_integration
from opencensus.trace.samplers import AlwaysOnSampler
from opencensus.trace.tracer import Tracer

config_integration.trace_integrations(['logging'])
logging.basicConfig(format='%(asctime)s traceId=%(traceId)s spanId=%(spanId)s %(message)s')
tracer = Tracer(sampler=AlwaysOnSampler())

logger = logging.getLogger(__name__)
logger.warning('Before the span')
with tracer.span(name='hello'):
    logger.warning('In the span')
logger.warning('After the span')
```
Quand ce code s’exécute, voici ce qui s’affiche dans la console :
```
2019-10-17 11:25:59,382 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=0000000000000000 Before the span
2019-10-17 11:25:59,384 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=70da28f5a4831014 In the span
2019-10-17 11:25:59,385 traceId=c54cb1d4bbbec5864bf0917c64aeacdc spanId=0000000000000000 After the span
```
Notez qu’un `spanId` est présent pour le message du journal qui se trouve dans l’étendue. Il s’agit du même `spanId` que celui qui appartient à l’étendue nommée `hello`.

Vous pouvez exporter les données du journal avec `AzureLogHandler`. Pour plus d’informations, consultez [cet article](./opencensus-python.md#logs).

## <a name="telemetry-correlation-in-net"></a>Corrélation de télémétrie dans .NET

Le runtime .NET prend en charge la distribution avec l’aide de [Activity](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/ActivityUserGuide.md) et de [DiagnosticSource](https://github.com/dotnet/runtime/blob/master/src/libraries/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md)

Le kit de développement logiciel (SDK) .NET d’Application Insights utilise `DiagnosticSource` et `Activity` pour collecter et mettre en corrélation les données de télémétrie.

<a name="java-correlation"></a>
## <a name="telemetry-correlation-in-java"></a>Corrélation de télémétrie dans Java

L’[agent Java](./java-in-process-agent.md) ainsi que le [Kit de développement logiciel (SDK) Java](../../azure-monitor/app/java-get-started.md) version 2.0.0 ou ultérieure prennent en charge la corrélation automatique de télémétrie. Il renseigne automatiquement `operation_id` pour toutes les données de télémétrie (comme les traces, les exceptions et les événements personnalisés) émises dans l’étendue d’une requête. Il propage aussi les en-têtes de corrélation (décrits plus haut) pour les appels entre les services via le protocole HTTP si l’[agent du SDK Java](../../azure-monitor/app/java-agent.md) est configuré.

> [!NOTE]
> L’agent Java Application Insights collecte automatiquement les demandes et les dépendances pour JMS, Kafka, net/webflux et bien plus. Pour le Kit de développement logiciel (SDK) Java, seuls les appels effectués via Apache HttpClient sont pris en charge pour la fonctionnalité de corrélation. La propagation automatique de contexte entre les technologies de messagerie (comme Kafka, RabbitMQ et Azure Service Bus) n’est pas prise en charge dans le Kit de développement logiciel (SDK). 

> [!NOTE]
> Pour collecter des données de télémétrie personnalisées, vous devez instrumenter l’application à l'aide du kit de développement logiciel (SDK) Java 2.6. 

### <a name="role-names"></a>Nom des rôles

Vous pouvez personnaliser la façon dont les noms des composants sont affichés dans la [cartographie d’application](../../azure-monitor/app/app-map.md). Pour cela, vous pouvez définir manuellement `cloud_RoleName` en effectuant une des opérations suivantes :

- Pour l'agent Java 3.0 Application Insights, définissez le nom du rôle cloud comme suit :

    ```json
    {
      "role": {
        "name": "my cloud role name"
      }
    }
    ```
    Vous pouvez également définir le nom de rôle cloud à l'aide de la variable d’environnement `APPLICATIONINSIGHTS_ROLE_NAME`.

- Avec Application Insights Java SDK 2.5.0 et ultérieur, vous pouvez spécifier `cloud_RoleName` en ajoutant `<RoleName>` à votre fichier ApplicationInsights.xml :

  ```XML
  <?xml version="1.0" encoding="utf-8"?>
  <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">
     <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>
     <RoleName>** Your role name **</RoleName>
     ...
  </ApplicationInsights>
  ```

- Si vous utilisez Spring Boot avec Application Insights Spring Boot Starter, vous devez seulement définir votre nom personnalisé pour l’application dans le fichier application.properties :

  `spring.application.name=<name-of-app>`

  Spring Boot Starter attribue automatiquement `cloudRoleName` à la valeur que vous entrez pour la propriété `spring.application.name`.

## <a name="next-steps"></a>Étapes suivantes

- Écrivez des [données de télémétrie personnalisées](../../azure-monitor/app/api-custom-events-metrics.md).
- Pour des scénarios de corrélation avancés dans ASP.NET Core et ASP.NET, consultez [Effectuer le suivi des opérations personnalisées](custom-operations-tracking.md).
- Découvrez plus en détail la [définition de cloud_RoleName](./app-map.md#set-or-override-cloud-role-name) pour d’autres kits SDK.
- Intégrez tous les composants de votre microservice sur Application Insights. Consultez les [plateformes prises en charge](./platforms.md).
- Pour connaître les types Application Insights, consultez [Modèle de données](./data-model.md).
- Découvrez comment [étendre et filtrer la télémétrie](./api-filtering-sampling.md).
- Consultez les [informations de référence sur la configuration d’Application Insights](configuration-with-applicationinsights-config.md).