---
title: Options de configuration – Azure Monitor Application Insights pour Java
description: Comment configurer Azure Monitor Application Insights pour Java
ms.topic: conceptual
ms.date: 11/04/2020
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 73e3d8834fcb41f863fe2e3d686b812bb6174e70
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507657"
---
# <a name="configuration-options---azure-monitor-application-insights-for-java"></a>Options de configuration – Azure Monitor Application Insights pour Java

> [!WARNING]
> **Si vous effectuez une mise à niveau à partir de la préversion 3.0**
>
> Examinez soigneusement toutes les options de configuration ci-dessous, car la structure json a complètement changé, en plus du nom de fichier désormais en minuscules.

## <a name="connection-string-and-role-name"></a>Chaîne de connexion et nom de rôle

La chaîne de connexion et le nom de rôle sont les paramètres les plus courants nécessaires pour commencer :

```json
{
  "connectionString": "InstrumentationKey=...",
  "role": {
    "name": "my cloud role name"
  }
}
```

La chaîne de connexion est requise, et le nom de rôle est important chaque fois que vous envoyez des données à partir de différentes applications vers la même ressource Application Insights.

Vous trouverez plus de détails, ainsi que des options de configuration supplémentaires ci-dessous.

## <a name="configuration-file-path"></a>Chemin d'accès au fichier de configuration

Par défaut, Application Insights Java 3.0 s’attend à ce que le fichier de configuration soit nommé `applicationinsights.json` et se trouve dans le même répertoire que `applicationinsights-agent-3.0.0.jar`.

Vous pouvez spécifier votre propre chemin d’accès au fichier de configuration à l’aide d'un des éléments suivants :

* variable d’environnement `APPLICATIONINSIGHTS_CONFIGURATION_FILE`, ou
* propriété système Java `applicationinsights.configuration.file`

Si vous spécifiez un chemin d’accès relatif, il sera résolu par rapport au répertoire où se trouve `applicationinsights-agent-3.0.0.jar`.

## <a name="connection-string"></a>Chaîne de connexion

La chaîne de connexion est obligatoire. Vous pouvez trouver votre chaîne de connexion dans votre ressource Application Insights :

:::image type="content" source="media/java-ipa/connection-string.png" alt-text="Chaîne de connexion Application Insights":::


```json
{
  "connectionString": "InstrumentationKey=..."
}
```

Vous pouvez également définir la chaîne de connexion à l'aide de la variable d’environnement `APPLICATIONINSIGHTS_CONNECTION_STRING`.

Si vous ne définissez pas la chaîne de connexion, l’agent Java sera désactivé.

## <a name="cloud-role-name"></a>Nom du rôle cloud

Le nom du rôle cloud est utilisé pour étiqueter le composant sur la cartographie d’application.

Si vous souhaitez définir le nom du rôle cloud :

```json
{
  "role": {   
    "name": "my cloud role name"
  }
}
```

Si le nom du rôle cloud n’est pas défini, le nom de la ressource Application Insights sera utilisé pour étiqueter le composant sur la cartographie d’application.

Vous pouvez également définir le nom de rôle cloud à l'aide de la variable d’environnement `APPLICATIONINSIGHTS_ROLE_NAME`.

## <a name="cloud-role-instance"></a>Instance de rôle cloud

Par défaut, l’instance de rôle cloud correspond au nom de la machine.

Si vous souhaitez définir l'instance de rôle cloud sur autre chose que le nom de la machine :

```json
{
  "role": {
    "name": "my cloud role name",
    "instance": "my cloud role instance"
  }
}
```

Vous pouvez également définir l'instance de rôle cloud à l'aide de la variable d’environnement `APPLICATIONINSIGHTS_ROLE_INSTANCE`.

## <a name="sampling"></a>échantillonnage

L’échantillonnage s'avère utile s'il vous faut réduire les coûts.
L’échantillonnage s'effectue sous forme de fonction sur l’ID d’opération (également appelé ID de trace) pour permettre au même ID d’opération de générer la même décision d’échantillonnage. Ainsi, vous n'obtenez pas de parties de transaction distribuée échantillonnées quand d'autres ne le sont pas.

Par exemple, si vous définissez l’échantillonnage sur 10 %, vous ne verrez que 10 % de vos transactions, mais chacun de ces 10 % présentera les détails des transactions de bout en bout.

Voici un exemple montrant comment définir l’échantillonnage de façon à capturer environ **1/3 de toutes les transactions**. Vérifiez que le taux d’échantillonnage est correct pour votre cas d’utilisation :

```json
{
  "sampling": {
    "percentage": 33.333
  }
}
```

Vous pouvez également définir le pourcentage d’échantillonnage à l’aide de la variable d’environnement `APPLICATIONINSIGHTS_SAMPLING_PERCENTAGE`.

> [!NOTE]
> Pour le pourcentage d’échantillonnage, choisissez un pourcentage proche de 100/N, où N est un nombre entier. L’échantillonnage ne prend actuellement en charge aucune autre valeur.

## <a name="jmx-metrics"></a>Métriques JMX

Si vous souhaitez collecter des métriques JMX supplémentaires :

```json
{
  "jmxMetrics": [
    {
      "name": "JVM uptime (millis)",
      "objectName": "java.lang:type=Runtime",
      "attribute": "Uptime"
    },
    {
      "name": "MetaSpace Used",
      "objectName": "java.lang:type=MemoryPool,name=Metaspace",
      "attribute": "Usage.used"
    }
  ]
}
```

`name` est le nom qui sera attribué à cette métrique JMX (peut être un nom quelconque).

`objectName` est le [Nom d’objet](https://docs.oracle.com/javase/8/docs/api/javax/management/ObjectName.html) du MBean JMX que vous souhaitez collecter.

`attribute` est le nom d’attribut à l’intérieur du MBean JMX que vous souhaitez collecter.

Les valeurs de métrique JMX numériques et booléennes sont prises en charge. Les métriques JMX booléennes sont mappées à `0` pour false et à `1` pour true.

[//]: # "NOTE: Not documenting APPLICATIONINSIGHTS_JMX_METRICS here"
[//]: # "json embedded in env var is messy, and should be documented only for codeless attach scenario"

## <a name="custom-dimensions"></a>Dimensions personnalisées

Si vous souhaitez ajouter des dimensions personnalisées à toutes vos données de télémétrie :

```json
{
  "customDimensions": {
    "mytag": "my value",
    "anothertag": "${ANOTHER_VALUE}"
  }
}
```

Vous pouvez utiliser `${...}` pour lire la valeur de la variable d’environnement spécifiée au démarrage.

## <a name="telemetry-processors-preview"></a>Processeurs de télémétrie (préversion)

Cette fonctionnalité est en préversion.

Elle vous permet de configurer des règles qui seront appliquées aux données de télémétrie de requêtes, de dépendances et de traces. Par exemple :
 * Masquer des données sensibles
 * Ajouter de manière conditionnelle des dimensions personnalisées
 * Mettre à jour le nom de télémétrie utilisé pour l’agrégation et l’affichage

Pour plus d’informations, consultez la documentation du [processeur de télémétrie](./java-standalone-telemetry-processors.md).

## <a name="auto-collected-logging"></a>Journalisation collectée automatiquement

Les journalisations Log4j, Logback et java.util.logging sont instrumentées automatiquement, et la journalisation effectuée via ces frameworks de journalisation est collectée automatiquement.

La journalisation n’est capturée que si elle respecte, premièrement, le seuil configuré des infrastructures de journalisation, et deuxièmement, le seuil configuré d’Application Insights.

Le seuil d’Application Insights par défaut est `INFO`. Si vous souhaitez modifier ce seuil :

```json
{
  "instrumentation": {
    "logging": {
      "level": "WARN"
    }
  }
}
```

Vous pouvez également définir le seuil à l’aide de la variable d’environnement `APPLICATIONINSIGHTS_INSTRUMENTATION_LOGGING_LEVEL`.

Voici les valeurs de `level` valides que vous pouvez spécifier dans le fichier `applicationinsights.json`, et leurs correspondances avec les niveaux de journalisation dans les différents frameworks de journalisation :

| level             | Log4j  | Logback | JUL     |
|-------------------|--------|---------|---------|
| OFF               | OFF    | OFF     | OFF     |
| FATAL             | FATAL  | ERROR   | SEVERE  |
| ERROR (ou SEVERE) | ERROR  | ERROR   | SEVERE  |
| WARN (ou WARNING) | WARN   | WARN    | WARNING |
| INFO              | INFO   | INFO    | INFO    |
| CONFIG            | DEBUG  | DEBUG   | CONFIG  |
| DEBUG (ou FINE)   | DEBUG  | DEBUG   | FINE    |
| FINER             | DEBUG  | DEBUG   | FINER   |
| TRACE (ou FINEST) | TRACE  | TRACE   | FINEST  |
| ALL               | ALL    | ALL     | ALL     |

## <a name="auto-collected-micrometer-metrics-including-spring-boot-actuator-metrics"></a>Métriques de Micrometer collectées automatiquement (y compris les métriques de Spring Boot Actuator)

Si votre application utilise [Micrometer](https://micrometer.io), les métriques envoyées au registre global de Micrometer sont collectées automatiquement.

Par ailleurs, si votre application utilise [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html), les métriques configurées par Spring Boot Actuator sont également collectées automatiquement.

Pour désactiver la collection automatique des métriques de Micrometer (y compris les métriques de Spring Boot Actuator) :

> [!NOTE]
> Les métriques personnalisées sont facturées séparément et peuvent occasionner des coûts supplémentaires. Veillez à consulter les [informations sur la tarification](https://azure.microsoft.com/pricing/details/monitor/) détaillées. Pour désactiver les métriques de Micrometer et de Spring Actuator, ajoutez la configuration ci-dessous à votre fichier de configuration.

```json
{
  "instrumentation": {
    "micrometer": {
      "enabled": false
    }
  }
}
```

## <a name="heartbeat"></a>Heartbeat

Par défaut, Application Insights Java 3.0 envoie une métrique de pulsation toutes les 15 minutes. Si vous utilisez la métrique de pulsation pour déclencher des alertes, vous pouvez augmenter la fréquence de cette pulsation :

```json
{
  "heartbeat": {
    "intervalSeconds": 60
  }
}
```

> [!NOTE]
> Vous ne pouvez pas réduire la fréquence de la pulsation, car les données de pulsation sont également utilisées pour suivre l’utilisation d’Application Insights.

## <a name="http-proxy"></a>Proxy HTTP

Si votre application se trouve derrière un pare-feu et n’est pas en mesure de se connecter directement à Application Insights (voir [Adresses IP utilisées par Application Insights](./ip-addresses.md)), vous pouvez configurer Application Insights Java 3.0 pour utiliser un proxy HTTP :

```json
{
  "proxy": {
    "host": "myproxy",
    "port": 8080
  }
}
```

[//]: # "NOTEZ que vous ne devez pas annoncer la prise en charge d’OpenTelemetry tant que nous ne prenons pas en charge la version 0.10.0 qui présente des modifications importantes par rapport à la version 0.9.0"

[//]: # "## Prise en charge des versions de l’API OpenTelemetry antérieures à la version 1.0"

[//]: # "La prise en charge des versions antérieures à la version 1.0 de l’API OpenTelemetry est acceptée, parce que l’API n’est pas encore stable"
[//]: # "Par conséquent, chaque version de l’agent ne prend en charge qu’une version spécifique antérieure à la version 1.0 de l’API OpenTelemetry"
[//]: # "(cette limitation cessera de s’appliquer après la mise en production de l’API OpenTelemetry 1.0)."

[//]: # "```json"
[//]: # "{"
[//]: # "  \"preview\": {"
[//]: # "    \"openTelemetryApiSupport\": true"
[//]: # "  }"
[//]: # "}"
[//]: # "```"

## <a name="self-diagnostics"></a>Autodiagnostics

La fonctionnalité « Autodiagnostics » fait référence à la journalisation interne à partir d’Application Insights Java 3.0.

Elle permet de détecter et de diagnostiquer des problèmes liés à Application Insights.

Par défaut, Application Insights Java 3.0 se connecte au niveau `INFO` au fichier `applicationinsights.log` et à la console, ce qui correspond à la configuration suivante :

```json
{
  "selfDiagnostics": {
    "destination": "file+console",
    "level": "INFO",
    "file": {
      "path": "applicationinsights.log",
      "maxSizeMb": 5,
      "maxHistory": 1
    }
  }
}
```

`destination` peut être `file`, `console` ou `file+console`.

`level` peut être `OFF`, `ERROR`, `WARN`, `INFO`, `DEBUG` ou `TRACE`.

`path` peut être un chemin d’accès absolu ou relatif. Les chemins d’accès relatifs sont résolus par rapport au répertoire où se trouve le fichier `applicationinsights-agent-3.0.0.jar`.

`maxSizeMb` est la taille maximale du fichier journal avant son remplacement.

`maxHistory` est le nombre de fichiers journaux remplacés et conservés (en plus du fichier journal actuel).

## <a name="an-example"></a>Exemple

Il s’agit simplement d’un exemple pour montrer à quoi ressemble un fichier de configuration avec plusieurs composants.
Veuillez configurer des options spécifiques en fonction de vos besoins.

```json
{
  "connectionString": "InstrumentationKey=...",
  "role": {
    "name": "my cloud role name"
  },
  "sampling": {
    "percentage": 100
  },
  "jmxMetrics": [
  ],
  "customDimensions": {
  },
  "instrumentation": {
    "logging": {
      "level": "INFO"
    },
    "micrometer": {
      "enabled": true
    }
  },
  "proxy": {
  },
  "preview": {
    "processors": [
    ]
  },
  "selfDiagnostics": {
    "destination": "file+console",
    "level": "INFO",
    "file": {
      "path": "applicationinsights.log",
      "maxSizeMb": 5,
      "maxHistory": 1
    }
  }
}
```