---
title: Exploration des journaux des traces Java dans Azure Application Insights
description: Recherche de suivi Log4J ou Logback dans Application Insights
ms.topic: conceptual
ms.date: 05/18/2019
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.openlocfilehash: fb91662dfcdeb404f51f91c1fef893dc72dcac73
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601048"
---
# <a name="explore-java-trace-logs-in-application-insights"></a>Exploration du suivi des journaux d’activité Java dans Application Insights

> [!IMPORTANT]
> L’approche recommandée pour surveiller des applications Java consiste à utiliser l’instrumentation automatique sans modifier le code. Suivez les instructions pour l’[agent Application Insights agent Java 3.0](./java-in-process-agent.md).

Si vous utilisez Logback ou Log4J (v1.2 ou v2.0) pour le suivi, vous pouvez faire en sorte que vos journaux d’activité de suivi soient envoyés automatiquement à Application Insights, où vous pouvez les explorer et effectuer des recherches.

> [!TIP]
> Vous ne devez définir votre clé d’instrumentation Application Insights qu’une seule fois pour votre application. Si vous utilisez une infrastructure comme Java Spring, vous avez peut-être déjà inscrit la clé ailleurs dans la configuration de votre application.

## <a name="using-the-application-insights-java-agent"></a>Utilisation de l’agent Java Application Insights

Par défaut, l’agent Java Application Insights capture automatiquement la journalisation effectuée au niveau `WARN` et au-dessus.

Vous pouvez changer le seuil de journalisation capturé à l’aide du fichier `AI-Agent.xml` :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsightsAgent>
   <Instrumentation>
      <BuiltIn>
         <Logging threshold="info"/>
      </BuiltIn>
   </Instrumentation>
</ApplicationInsightsAgent>
```

Vous pouvez désactiver la capture de la journalisation par l’agent Java à l’aide du fichier `AI-Agent.xml` :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationInsightsAgent>
   <Instrumentation>
      <BuiltIn>
         <Logging enabled="false"/>
      </BuiltIn>
   </Instrumentation>
</ApplicationInsightsAgent>
```

## <a name="alternatively-as-opposed-to-using-the-java-agent-you-can-follow-the-instructions-below"></a>Sinon (par opposition à l’utilisation de l’agent Java), vous pouvez suivre les instructions ci-dessous.

### <a name="install-the-java-sdk"></a>Installer le Kit de développement logiciel (SDK) Java

Suivez les instructions pour installer le [kit de développement logiciel (SDK) Application Insights pour Java][java], si ce n’est pas déjà fait.

### <a name="add-logging-libraries-to-your-project"></a>Ajouter des bibliothèques de journalisation à votre projet
*Choisissez la méthode adaptée à votre projet.*

#### <a name="if-youre-using-maven"></a>Si vous utilisez Maven...
Si votre projet est déjà configuré pour être assemblé avec Maven, fusionnez les extraits de code suivants dans votre fichier pom.xml.

Actualisez ensuite les dépendances du projet pour télécharger les fichiers binaires.

*Logback*

```XML

    <dependencies>
       <dependency>
          <groupId>com.microsoft.azure</groupId>
          <artifactId>applicationinsights-logging-logback</artifactId>
          <version>[2.0,)</version>
       </dependency>
    </dependencies>
```

*Log4J v2.0*

```XML

    <dependencies>
       <dependency>
          <groupId>com.microsoft.azure</groupId>
          <artifactId>applicationinsights-logging-log4j2</artifactId>
          <version>[2.0,)</version>
       </dependency>
    </dependencies>
```

*Log4J v1.2*

```XML

    <dependencies>
       <dependency>
          <groupId>com.microsoft.azure</groupId>
          <artifactId>applicationinsights-logging-log4j1_2</artifactId>
          <version>[2.0,)</version>
       </dependency>
    </dependencies>
```

#### <a name="if-youre-using-gradle"></a>Si vous utilisez Gradle...
Si votre projet est déjà configuré pour utiliser Gradle, ajoutez une des lignes suivantes au groupe `dependencies` dans votre fichier build.gradle :

Actualisez ensuite les dépendances du projet pour télécharger les fichiers binaires.

**Logback**

```

    compile group: 'com.microsoft.azure', name: 'applicationinsights-logging-logback', version: '2.0.+'
```

**Log4J v2.0**

```
    compile group: 'com.microsoft.azure', name: 'applicationinsights-logging-log4j2', version: '2.0.+'
```

**Log4J v1.2**

```
    compile group: 'com.microsoft.azure', name: 'applicationinsights-logging-log4j1_2', version: '2.0.+'
```

#### <a name="otherwise-"></a>Sinon...
Suivez les instructions pour installer manuellement le kit de développement logiciel (SDK) Application Insights pour Java, téléchargez le fichier jar (une fois sur la page principale Maven, cliquez sur le lien « jar » dans la section de téléchargement) de l'appender approprié, puis ajoutez le fichier jar de l'appender téléchargé au projet.

| Enregistreur | Téléchargement | Bibliothèque |
| --- | --- | --- |
| Logback |[Jar de l’appender Logback](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22applicationinsights-logging-logback%22) |applicationinsights-logging-logback |
| Log4J v2.0 |[Jar de l’appender Log4J v2](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22applicationinsights-logging-log4j2%22) |applicationinsights-logging-log4j2 |
| Log4J v1.2 |[Jar de l’appender Log4J v1.2](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22applicationinsights-logging-log4j1_2%22) |applicationinsights-logging-log4j1_2 |


### <a name="add-the-appender-to-your-logging-framework"></a>Ajouter l’appender à votre infrastructure de journalisation
Pour recevoir le suivi, fusionnez l’extrait de code approprié dans le fichier de configuration Log4J ou Logback : 

*Logback*

```XML

    <appender name="aiAppender" 
      class="com.microsoft.applicationinsights.logback.ApplicationInsightsAppender">
        <instrumentationKey>[APPLICATION_INSIGHTS_KEY]</instrumentationKey>
    </appender>
    <root level="trace">
      <appender-ref ref="aiAppender" />
    </root>
```

*Log4J v2.0*

```XML

    <Configuration packages="com.microsoft.applicationinsights.log4j.v2">
      <Appenders>
        <ApplicationInsightsAppender name="aiAppender" instrumentationKey="[APPLICATION_INSIGHTS_KEY]" />
      </Appenders>
      <Loggers>
        <Root level="trace">
          <AppenderRef ref="aiAppender"/>
        </Root>
      </Loggers>
    </Configuration>
```

*Log4J v1.2*

```XML

    <appender name="aiAppender" 
         class="com.microsoft.applicationinsights.log4j.v1_2.ApplicationInsightsAppender">
        <param name="instrumentationKey" value="[APPLICATION_INSIGHTS_KEY]" />
    </appender>
    <root>
      <priority value ="trace" />
      <appender-ref ref="aiAppender" />
    </root>
```

Les appenders Application Insights peuvent être référencés par n’importe quel enregistreur configuré et pas nécessairement par l’enregistreur racine (comme indiqué dans les exemples de code ci-dessus).

## <a name="explore-your-traces-in-the-application-insights-portal"></a>Explorer le suivi dans le portail Application Insights
Maintenant que vous avez configuré votre projet pour qu’il envoie le suivi à Application Insights, vous pouvez rechercher et consulter ce suivi dans le portail Application Insights, dans le panneau [Recherche][diagnostic].

Les exceptions envoyées via les enregistreurs d'événements s'afficheront sur le portail en tant que données de télémétrie d'exception.

![Dans Application Insights, ouvrez Recherche](./media/java-trace-logs/01-diagnostics.png)

## <a name="next-steps"></a>Étapes suivantes
[Recherche de diagnostic][diagnostic]

<!--Link references-->

[diagnostic]: ./diagnostic-search.md
[java]: java-get-started.md

