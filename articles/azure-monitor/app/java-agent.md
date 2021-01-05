---
title: Supervision des performances des applications web Java - Azure Application Insights
description: Surveillance étendue des performances et de l’utilisation de votre site web Java avec Application Insights.
ms.topic: conceptual
ms.date: 01/10/2019
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: 299e9010b74c8363cacd1c20044d183dc1def6a6
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601286"
---
# <a name="monitor-dependencies-caught-exceptions-and-method-execution-times-in-java-web-apps"></a>Surveiller les dépendances, les exceptions interceptées et les temps d’exécution des méthodes dans les applications web Java

> [!IMPORTANT]
> L’approche recommandée pour surveiller des applications Java consiste à utiliser l’instrumentation automatique sans modifier le code. Suivez les instructions pour l’[agent Application Insights agent Java 3.0](./java-in-process-agent.md).

Si vous avez [instrumenté votre application web Java avec le Kit de développement logiciel (SDK) Application Insights][java], vous pouvez utiliser l’agent Java pour obtenir des informations plus détaillées, sans pour autant modifier le code :

* **Dépendances :** données sur les appels passés par votre application à destination d’autres composants, dont :
  * Les **appels HTTP sortants** effectués via Apache HttpClient, OkHttp et `java.net.HttpURLConnection` sont capturés.
  * Les **appels Redis** passés via le client Jedis sont capturés.
  * **Requêtes JDBC** : pour MySQL et PostgreSQL, si l’appel prend plus de 10 s, l’agent signale le plan de requête.

* **Journalisation des applications :** capturez et mettez en corrélation les journaux de vos applications avec des requêtes HTTP et d’autres données de télémétrie
  * **Log4j 1.2**
  * **Log4j2**
  * **Logback**

* **Meilleure appellation des opérations** : (utilisée pour l’agrégation des demandes dans le portail)
  * **Spring** - basé sur `@RequestMapping`.
  * **JAX-RS** - basé sur `@Path`. 

Pour utiliser l’agent Java, installez-le sur votre serveur. Vos applications web doivent être instrumentées à l’aide du [Kit de développement logiciel (SDK) Java Application Insights][java]. 

## <a name="install-the-application-insights-agent-for-java"></a>Installer l’agent Application Insights pour Java
1. [Téléchargez l'agent](https://github.com/Microsoft/ApplicationInsights-Java/releases/latest) sur la machine exécutant votre serveur Java. Veillez à télécharger la même version de Java Agent en tant que packages de noyau et web de SDK Java Application Insights.
2. Modifiez le script de démarrage du serveur d’applications, puis ajoutez l’argument JVM suivant :
   
    `-javaagent:<full path to the agent JAR file>`
   
    Par exemple, dans Tomcat sur une machine Linux :
   
    `export JAVA_OPTS="$JAVA_OPTS -javaagent:<full path to agent JAR file>"`
3. Redémarrez votre serveur d’applications.

## <a name="configure-the-agent"></a>Configurer l’agent
Créez un fichier nommé `AI-Agent.xml` et placez-le dans le même dossier que le fichier JAR de l’agent.

Définissez le contenu du fichier xml. Modifiez l’exemple suivant pour inclure ou omettre les fonctionnalités souhaitées.

```XML
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsightsAgent>
   <Instrumentation>
      <BuiltIn enabled="true">

         <!-- capture logging via Log4j 1.2, Log4j2, and Logback, default is true -->
         <Logging enabled="true" />

         <!-- capture outgoing HTTP calls performed through Apache HttpClient, OkHttp,
              and java.net.HttpURLConnection, default is true -->
         <HTTP enabled="true" />

         <!-- capture JDBC queries, default is true -->
         <JDBC enabled="true" />

         <!-- capture Redis calls, default is true -->
         <Jedis enabled="true" />

         <!-- capture query plans for JDBC queries that exceed this value (MySQL, PostgreSQL),
              default is 10000 milliseconds -->
         <MaxStatementQueryLimitInMS>1000</MaxStatementQueryLimitInMS>

      </BuiltIn>
   </Instrumentation>
</ApplicationInsightsAgent>
```

## <a name="additional-config-spring-boot"></a>Configuration supplémentaire (Spring Boot)

`java -javaagent:/path/to/agent.jar -jar path/to/TestApp.jar`

Pour Azure App Services, procédez comme suit :

* Cliquez sur Paramètres > Paramètres de l’application
* Sous Paramètres de l’application, ajoutez une nouvelle paire clé/valeur :

Clé : Valeur `JAVA_OPTS` : `-javaagent:D:/home/site/wwwroot/applicationinsights-agent-2.5.0.jar`

Pour connaître la dernière version de l’agent Java, consultez les versions [ici](https://github.com/Microsoft/ApplicationInsights-Java/releases
). 

L’agent doit être empaqueté sous forme de ressource dans votre projet de sorte qu’il se retrouve dans le répertoire D:/home/site/wwwroot/. Vous pouvez vérifier que votre agent se trouve dans le répertoire App Service en accédant à **Outils de développement** > **Outils avancés** > **Console de débogage**, puis en examinant le contenu du répertoire du site.    

* Enregistrez les paramètres, puis redémarrez votre application. (Ces étapes s’appliquent uniquement à App Services s’exécutant sur Windows.)

> [!NOTE]
> Le fichier AI-agent.XML et le fichier jar de l’agent doivent se trouver dans le même dossier. Ils sont souvent placés ensemble dans le dossier `/resources` du projet.  

#### <a name="enable-w3c-distributed-tracing"></a>Activer le traçage distribué W3C

Ajoutez le code suivant au fichier AI-Agent.xml :

```xml
<Instrumentation>
   <BuiltIn enabled="true">
      <HTTP enabled="true" W3C="true" enableW3CBackCompat="true"/>
   </BuiltIn>
</Instrumentation>
```

> [!NOTE]
> Le mode de compatibilité descendante est activé par défaut ; le paramètre enableW3CBackCompat est facultatif et ne doit être utilisé que si vous souhaitez désactiver ce mode. 

Idéalement, ce serait le cas si tous vos services avaient été mis à jour vers une version plus récente des SDK prenant en charge le protocole W3C. Il est vivement recommandé de migrer au plus vite vers une version plus récente des SDK prenant en charge le protocole W3C.

Assurez-vous que les **configurations [entrante](correlation.md#enable-w3c-distributed-tracing-support-for-java-apps) et sortante (agent)** sont totalement identiques.

## <a name="view-the-data"></a>Visualiser les données
Dans la ressource Application Insights, les temps d’exécution cumulés de la méthode et de la dépendance distante apparaissent [dans la vignette Performances][metrics].

Pour rechercher des instances de dépendance et d’exception ainsi que des rapports de méthode, ouvrez [Rechercher][diagnostic].

[En savoir plus sur le diagnostic des problèmes de dépendance](./asp-net-dependencies.md#diagnosis).

## <a name="questions-problems"></a>Des questions ? Des problèmes ?
* Pas de données ? [Définir les exceptions de pare-feu](./ip-addresses.md)
* [Résolution des problèmes Java](java-troubleshoot.md)

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiexceptions]: ./api-custom-events-metrics.md#track-exception
[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[eclipse]: app-insights-java-eclipse.md
[java]: java-get-started.md
[javalogs]: java-trace-logs.md
[metrics]: ../platform/metrics-charts.md

