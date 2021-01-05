---
title: Azure Monitor Application Insights Java
description: APM pour les applications Java s’exécutant dans n’importe quel environnement sans modification du code. Suivi distribué et mise en correspondance d’applications.
ms.topic: conceptual
ms.date: 03/29/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 3cab22c2271fd5874b4b094be65c36f5b5f3a22d
ms.sourcegitcommit: 287c20509c4cf21d20eea4619bbef0746a5cd46e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97371881"
---
# <a name="java-codeless-application-monitoring-azure-monitor-application-insights"></a>Supervision des applications Java sans code avec Azure Monitor Application Insights

La supervision des applications Java sans code se caractérise par sa simplicité : le code n’est pas modifié et l’agent Java peut être activé avec quelques changements de configuration.

 L’agent Java fonctionne dans n’importe quel environnement et vous permet de superviser toutes vos applications Java. En d’autres termes, que vous exécutiez vos applications Java sur des machines virtuelles, locales, dans AKS, sur Windows, sur Linux, entre autres, l’agent Java 3.0 les supervise.

L’ajout du Kit de développement logiciel (SDK) Java Application Insights à votre application n’est plus nécessaire, car l’agent 3.0 collecte automatiquement les demandes, les dépendances et les journaux.

Vous pouvez toujours envoyer des données de télémétrie personnalisées à partir de votre application. L’agent 3.0 en effectue le suivi et la corrélation avec toutes les données de télémétrie collectées automatiquement.

L’agent 3.0 prend en charge Java 8 et versions ultérieures.

## <a name="quickstart"></a>Démarrage rapide

**1. Télécharger l’agent**

> [!WARNING]
> **Si vous effectuez une mise à niveau à partir de la préversion 3.0**
>
> Examinez soigneusement toutes les [options de configuration](./java-standalone-config.md) ci-dessous, car la structure JSON a complètement changé, en plus du nom de fichier désormais en minuscules.

Téléchargez [applicationinsights-agent-3.0.0.jar](https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.0.0/applicationinsights-agent-3.0.0.jar).

**2. Pointer la machine virtuelle Java (JVM) vers l’agent**

Ajoutez `-javaagent:path/to/applicationinsights-agent-3.0.0.jar` aux arguments JVM de votre application

Les arguments JVM standard incluent `-Xmx512m` et `-XX:+UseG1GC`. Par conséquent, si vous savez où ajouter ces derniers, vous savez déjà où ajouter celui-ci.

Pour obtenir une aide supplémentaire sur la configuration des arguments JVM de votre application, consultez [Conseils pour la mise à jour de vos arguments JVM](./java-standalone-arguments.md).

**3. Pointer l’agent vers votre ressource Application Insights**

Si vous n’avez pas encore de ressource Application Insights, vous pouvez en créer une en suivant les étapes décrites dans le [guide de création de ressources](./create-new-resource.md).

Pointez l’agent sur votre ressource Application Insights, soit en définissant une variable d’environnement :

```
APPLICATIONINSIGHTS_CONNECTION_STRING=InstrumentationKey=...
```

Soit en créant un fichier config nommé `applicationinsights.json` et en le plaçant dans le même répertoire que `applicationinsights-agent-3.0.0.jar`, avec le contenu suivant :

```json
{
  "connectionString": "InstrumentationKey=..."
}
```

Vous pouvez trouver votre chaîne de connexion dans votre ressource Application Insights :

:::image type="content" source="media/java-ipa/connection-string.png" alt-text="Chaîne de connexion Application Insights":::

**4. Et voilà !**

Maintenant, démarrez l’application et accédez à votre ressource Application Insights dans le portail Azure pour voir vos données de supervision.

> [!NOTE]
> L’affichage des données de supervision dans le portail peut prendre quelques minutes.


## <a name="configuration-options"></a>Options de configuration

Dans le fichier `applicationinsights.json`, vous pouvez également configurer :

* Nom du rôle cloud
* Instance de rôle cloud
* échantillonnage
* Métriques JMX
* Dimensions personnalisées
* Processeurs de télémétrie (préversion)
* Journalisation collectée automatiquement
* Métriques de Micrometer collectées automatiquement (y compris les métriques de Spring Boot Actuator)
* Heartbeat
* Proxy HTTP
* Autodiagnostics

Pour plus d’informations, consultez [Options de configuration](./java-standalone-config.md).

## <a name="auto-collected-requests-dependencies-logs-and-metrics"></a>Demandes, dépendances, journaux et métriques collectés automatiquement

### <a name="requests"></a>Demandes

* Consommateurs JMS
* Consommateurs Kafka
* Netty/WebFlux
* Servlets
* Spring Scheduling

### <a name="dependencies-with-distributed-trace-propagation"></a>Dépendances avec propagation de suivi distribué

* Apache HttpClient et HttpAsyncClient
* gRPC
* java.net.HttpURLConnection
* JMS
* Kafka
* Client Netty
* OkHttp

### <a name="other-dependencies"></a>Autres dépendances

* Cassandra
* JDBC
* MongoDB (asynchrone et synchrone)
* Redis (Lettuce et Jedis)

### <a name="logs"></a>Journaux d’activité

* java.util.logging
* Log4j (y compris les propriétés MDC)
* SLF4J/Logback (y compris les propriétés MDC)

### <a name="metrics"></a>Mesures

* Micrometer (y compris les métriques de Spring Boot Actuator)
* Métriques JMX

## <a name="send-custom-telemetry-from-your-application"></a>Envoyer des données de télémétrie personnalisées à partir de votre application

Notre objectif dans 3.0+ est de vous permettre d’envoyer vos données de télémétrie personnalisées à l’aide d’API standard.

Nous prenons en charge Micrometer, les frameworks de journalisation populaires, et le kit de développement logiciel (SDK) Application Insights Java 2.x.
Application Insights pour Java 3.0 capture automatiquement les données de télémétrie envoyées via ces API et les met en corrélation avec les données de télémétrie collectées automatiquement.

### <a name="supported-custom-telemetry"></a>Données de télémétrie personnalisées prises en charge

Le tableau ci-dessous représente les types de données de télémétrie personnalisées actuellement pris en charge que vous pouvez activer pour compléter l’agent Java 3.0. Pour résumer, les métriques personnalisées sont prises en charge via Micrometer, les exceptions et les traces personnalisées peuvent être activées via des frameworks de journalisation, et tout type de données de télémétrie personnalisées est pris en charge par le [Kit de développement logiciel (SDK) Application Insights Java 2.x](#send-custom-telemetry-using-application-insights-java-2x-sdk).

|                     | Micrometer | Log4j, logback, JUL | 2.x SDK |
|---------------------|------------|---------------------|---------|
| **Événements personnalisés**   |            |                     |  Oui    |
| **Métriques personnalisées**  |  Oui       |                     |  Oui    |
| **Dépendances**    |            |                     |  Oui    |
| **Exceptions**      |            |  Oui                |  Oui    |
| **Affichages de pages**      |            |                     |  Oui    |
| **Demandes**        |            |                     |  Oui    |
| **Traces**          |            |  Oui                |  Oui    |

Nous n’avons pas l’intention de publier un kit SDK avec Application Insights 3.0 pour le moment.

Application Insights pour Java 3.0 écoute déjà les données de télémétrie envoyées au kit SDK Application Insights pour Java 2.x. Cette fonctionnalité est une partie essentielle de la mise à niveau pour les utilisateurs 2.x existants car elle comble un écart important dans notre prise en charge de la télémétrie personnalisée jusqu’à la disponibilité générale de l’API OpenTelemetry.

### <a name="send-custom-metrics-using-micrometer"></a>Envoyer des métriques personnalisées à l'aide de Micrometer

Ajoutez Micrometer à votre application :

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-core</artifactId>
  <version>1.6.1</version>
</dependency>
```

Utilisez le [registre global](https://micrometer.io/docs/concepts#_global_registry) de Micrometer pour créer un compteur :

```java
static final Counter counter = Metrics.counter("test_counter");
```

et utilisez-le pour enregistrer les métriques :

```java
counter.increment();
```

### <a name="send-custom-traces-and-exceptions-using-your-favorite-logging-framework"></a>Envoyer des traces et des exceptions personnalisées à l'aide de votre framework de journalisation préféré

Les journalisations Log4j, Logback et java.util.logging sont instrumentées automatiquement, et la journalisation effectuée via ces frameworks de journalisation est collectée automatiquement sous forme de données de télémétrie de trace et d'exception.

Par défaut, la journalisation n'est collectée que lorsqu'elle est effectuée au niveau INFO ou à un niveau supérieur.
Consultez les [options de configuration](./java-standalone-config.md#auto-collected-logging) pour savoir comment modifier ce niveau.

Si vous souhaitez joindre des dimensions personnalisées à vos journaux, vous pouvez utiliser [Log4j 1.2 MDC](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/MDC.html), [Log4j 2 MDC](https://logging.apache.org/log4j/2.x/manual/thread-context.html) ou [Logback MDC](http://logback.qos.ch/manual/mdc.html). Application Insights pour Java 3.0 capturera automatiquement ces propriétés MDC sous forme de dimensions personnalisées sur vos données de télémétrie de trace et d’exception.

### <a name="send-custom-telemetry-using-application-insights-java-2x-sdk"></a>Envoyer des données de télémétrie personnalisées à l'aide du kit SDK Application Insights pour Java 2.x

Ajoutez `applicationinsights-core-2.6.2.jar` à votre application (toutes les versions 2.x sont prises en charge par Application Insights pour Java 3.0, mais il est préférable d’utiliser la dernière version si vous avez le choix) :

```xml
<dependency>
  <groupId>com.microsoft.azure</groupId>
  <artifactId>applicationinsights-core</artifactId>
  <version>2.6.2</version>
</dependency>
```

Créez un TelemetryClient :

  ```java
static final TelemetryClient telemetryClient = new TelemetryClient();
```

et utilisez-le pour envoyer des données de télémétrie personnalisées :

##### <a name="events"></a>Événements

```java
telemetryClient.trackEvent("WinGame");
```

##### <a name="metrics"></a>Mesures

```java
telemetryClient.trackMetric("queueLength", 42.0);
```

##### <a name="dependencies"></a>Les dépendances

```java
boolean success = false;
long startTime = System.currentTimeMillis();
try {
    success = dependency.call();
} finally {
    long endTime = System.currentTimeMillis();
    RemoteDependencyTelemetry telemetry = new RemoteDependencyTelemetry();
    telemetry.setTimestamp(new Date(startTime));
    telemetry.setDuration(new Duration(endTime - startTime));
    telemetryClient.trackDependency(telemetry);
}
```

##### <a name="logs"></a>Journaux d’activité

```java
telemetryClient.trackTrace(message, SeverityLevel.Warning, properties);
```

##### <a name="exceptions"></a>Exceptions

```java
try {
    ...
} catch (Exception e) {
    telemetryClient.trackException(e);
}
```
