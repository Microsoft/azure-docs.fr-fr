---
title: Utilisation de Micrometer avec le Kit de développement logiciel (SDK) Java Azure Application Insights
description: Guide pas à pas sur l’utilisation de Micrometer avec vos applications Application Insights Spring Boot et autres que Spring Boot.
ms.topic: conceptual
author: MS-jgol
ms.custom: devx-track-java
ms.author: jgol
ms.date: 11/01/2018
ms.openlocfilehash: bb5caafea944d21547a904b99f9043aef63a6ffa
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656464"
---
# <a name="how-to-use-micrometer-with-azure-application-insights-java-sdk-not-recommended"></a>Utilisation de Micrometer avec le Kit de développement logiciel (SDK) Java Azure Application Insights (non recommandé)

> [!IMPORTANT]
> L’approche recommandée pour surveiller des applications Java consiste à utiliser l’instrumentation automatique sans modifier le code. Les données de télémétrie Micrometer sont automatiquement collectée avec l’agent Application Insights Java 3.0 : suivez les instructions relatives à [l’agent Application Insights Java 3.0](./java-in-process-agent.md).

> [!NOTE]
> Le Kit de développement logiciel (SDK) Application Insights pour Java ne prend pas en charge Spring Webflux ; utilisez plutôt [l’agent Application Insights pour Java 3.0](./java-in-process-agent.md). 
>
> WebFlux et Micrometer sont pris en charge dans [l’agent Application Insights agent Java 3.0](./java-on-premises.md) sans nécessiter aucune instrumentation. 

La supervision d’application Micrometer mesure les métriques pour le code d’application basé sur la JVM et vous permet d’exporter les données vers vos systèmes de supervision préférés. Cet article vous montrera comment utiliser Micrometer avec Application Insights pour les applications Spring Boot et les autres.

## <a name="using-spring-boot-15x"></a>Utilisation de Spring Boot 1.5x
Ajoutez les dépendances suivantes à votre fichier pom.xml ou build.gradle : 
* [Application Insights spring-boot-starter](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter) version 2.5.0 ou ultérieure
* Micrometer Azure Registry version 1.1.0 ou ultérieure
* [Micrometer Spring Legacy](https://micrometer.io/docs/ref/spring/1.5) version 1.1.0 ou ultérieure (ce qui revient à porter le code autoconfig dans le framework Spring).
* [Ressource ApplicationInsights](./create-new-resource.md)

Étapes

1. Mettez à jour le fichier pom.xml de votre application Spring Boot et ajoutez-y les dépendances suivantes :

    ```XML
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>applicationinsights-spring-boot-starter</artifactId>
        <version>2.5.0</version>
    </dependency>

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-spring-legacy</artifactId>
        <version>1.1.0</version>
    </dependency>

    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-azure-monitor</artifactId>
        <version>1.1.0</version>
    </dependency>

    ```
2. Mettez à jour le fichier application.properties ou yml avec la clé d’instrumentation Application Insights en utilisant la propriété suivante :

     `azure.application-insights.instrumentation-key=<your-instrumentation-key-here>`
1. Créer votre application et l’exécuter
2. L’approche ci-dessus devrait vous permettre d’être opérationnel avec des métriques préagrégées collectées automatiquement pour Azure Monitor. Pour plus de détails sur la façon d’ajuster Application Insights Spring Boot Starter, reportez-vous au [fichier Lisez-moi sur GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/spring/azure-spring-boot-starter/README.md).

## <a name="using-spring-2x"></a>Utilisation de Spring 2.x

Ajoutez les dépendances suivantes à votre fichier pom.xml ou build.gradle :

* Application Insights Spring-boot-starter version 2.1.2 ou ultérieure
* Azure-spring-boot-metrics-starters version 2.0.7 ou ultérieure
* [Ressource Application Insights](./create-new-resource.md)

Étapes :

1. Mettez à jour le fichier pom.xml de votre application Spring Boot et ajoutez-y la dépendance suivante :

    ```XML
    <dependency> 
          <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-spring-boot-metrics-starter</artifactId>
          <version>2.0.7</version>
    </dependency>
    ```
1. Mettez à jour le fichier application.properties ou yml avec la clé d’instrumentation Application Insights en utilisant la propriété suivante :

     `azure.application-insights.instrumentation-key=<your-instrumentation-key-here>`
3. Créer votre application et l’exécuter
4. L’approche ci-dessus devrait vous permettre d’être opérationnel avec des métriques préagrégées collectées automatiquement pour Azure Monitor. Pour plus de détails sur la façon d’ajuster Application Insights Spring Boot Starter, reportez-vous au [fichier Lisez-moi sur GitHub](https://github.com/Microsoft/azure-spring-boot/releases/latest).

Métriques par défaut :

*    Métriques configurées automatiquement pour Tomcat, machine virtuelle Java, métriques Logback, métriques Log4J, métriques de disponibilité, métriques de processeur, FileDescriptorMetrics.
*    Par exemple, si l’API Netflix Hystrix est présente dans le chemin d’accès des classes, nous obtenons également ces métriques. 
*    Les métriques suivantes peuvent être obtenues par l’ajout de leurs beans respectifs. 
        - CacheMetrics (CaffeineCache, EhCache2, GuavaCache, HazelcastCache, JCache)
        - DataBaseTableMetrics 
        - HibernateMetrics 
        - JettyMetrics 
        - Mesures OKHttp3 
        - Mesures Kafka 



Comment désactiver la collecte automatique de métriques : 

- Métriques de JVM : 
    - management.metrics.binders.jvm.enabled=false 
- Mesures Logback : 
    - management.metrics.binders.logback.enabled=false
- Mesures sur le fonctionnement : 
    - management.metrics.binders.uptime.enabled=false 
- Mesures sur le processeur :
    -  management.metrics.binders.processor.enabled=false 
- FileDescriptorMetrics :
    - management.metrics.binders.files.enabled=false 
- Métriques Hystrix si la bibliothèque est enregistrée dans le chemin de classes : 
    - management.metrics.binders.hystrix.enabled=false 
- Métriques AspectJ si la bibliothèque est enregistrée dans le chemin de classes : 
    - Spring.AOP.Enabled=false 

> [!NOTE]
> Spécifiez les propriétés ci-dessus dans le fichier application.properties ou application.yml de votre application Spring Boot

## <a name="use-micrometer-with-non-spring-boot-web-applications"></a>Utilisez Micrometer avec les applications web non Spring Boot

Ajoutez les dépendances suivantes à votre fichier pom.xml ou build.gradle :

* Application Insights Web Auto version 2.5.0 ou ultérieure
* Micrometer Azure Registry version 1.1.0 ou ultérieure
* [Ressource Application Insights](./create-new-resource.md)

Étapes :

1. Ajoutez les dépendances suivantes à votre fichier pom.xml ou build.gradle :

    ```XML
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-azure-monitor</artifactId>
            <version>1.1.0</version>
        </dependency>

        <dependency>
            <groupId>com.microsoft.azure</groupId>
            <artifactId>applicationinsights-web-auto</artifactId>
            <version>2.5.0</version>
        </dependency>
     ```

2. Placez le fichier `ApplicationInsights.xml` dans le dossier des ressources :

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <ApplicationInsights xmlns="http://schemas.microsoft.com/ApplicationInsights/2013/Settings" schemaVersion="2014-05-30">

       <!-- The key from the portal: -->
       <InstrumentationKey>** Your instrumentation key **</InstrumentationKey>

       <!-- HTTP request component (not required for bare API) -->
       <TelemetryModules>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebRequestTrackingTelemetryModule"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebSessionTrackingTelemetryModule"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.modules.WebUserTrackingTelemetryModule"/>
       </TelemetryModules>

       <!-- Events correlation (not required for bare API) -->
       <!-- These initializers add context data to each event -->
       <TelemetryInitializers>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationIdTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebOperationNameTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebSessionTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserTelemetryInitializer"/>
          <Add type="com.microsoft.applicationinsights.web.extensibility.initializers.WebUserAgentTelemetryInitializer"/>
       </TelemetryInitializers>

    </ApplicationInsights>
    ```

3. Exemple de classe de servlet (émet une métrique de minuteur) :

    ```Java
        @WebServlet("/hello")
        public class TimedDemo extends HttpServlet {

          private static final long serialVersionUID = -4751096228274971485L;

          @Override
          @Timed(value = "hello.world")
          protected void doGet(HttpServletRequest request, HttpServletResponse response)
              throws ServletException, IOException {

            response.getWriter().println("Hello World!");
            MeterRegistry registry = (MeterRegistry) getServletContext().getAttribute("AzureMonitorMeterRegistry");

        //create new Timer metric
            Timer sampleTimer = registry.timer("timer");
            Stream<Integer> infiniteStream = Stream.iterate(0, i -> i+1);
            infiniteStream.limit(10).forEach(integer -> {
              try {
                Thread.sleep(1000);
                sampleTimer.record(integer, TimeUnit.MILLISECONDS);
              } catch (Exception e) {}
               });
          }
          @Override
          public void init() throws ServletException {
            System.out.println("Servlet " + this.getServletName() + " has started");
          }
          @Override
          public void destroy() {
            System.out.println("Servlet " + this.getServletName() + " has stopped");
          }

        }

    ```

4. Exemple de classe de configuration :

    ```Java
         @WebListener
         public class MeterRegistryConfiguration implements ServletContextListener {

           @Override
           public void contextInitialized(ServletContextEvent servletContextEvent) {

         // Create AzureMonitorMeterRegistry
           private final AzureMonitorConfig config = new AzureMonitorConfig() {
             @Override
             public String get(String key) {
                 return null;
             }
            @Override
               public Duration step() {
                 return Duration.ofSeconds(60);}

             @Override
             public boolean enabled() {
                 return false;
             }
         };

      MeterRegistry azureMeterRegistry = AzureMonitorMeterRegistry.builder(config);

             //set the config to be used elsewhere
             servletContextEvent.getServletContext().setAttribute("AzureMonitorMeterRegistry", azureMeterRegistry);

           }

           @Override
           public void contextDestroyed(ServletContextEvent servletContextEvent) {

           }
         }
    ```

Pour en savoir plus sur les métriques, consultez la [Documentation Micrometer](https://micrometer.io/docs/).

D’autres exemples de code sur la création de différents types de métriques sont disponibles dans le [référentiel GitHub officiel de Micrometer](https://github.com/micrometer-metrics/micrometer/tree/master/samples/micrometer-samples-core/src/main/java/io/micrometer/core/samples).

## <a name="how-to-bind-additional-metrics-collection"></a>Comment lier la collecte de métriques supplémentaires

### <a name="springbootspring"></a>SpringBoot/Spring

Créez un bean de la catégorie de métrique appropriée. Par exemple, supposons que nous ayons besoin de métriques du cache Guava :

```Java
    @Bean
    GuavaCacheMetrics guavaCacheMetrics() {
        Return new GuavaCacheMetrics();
    }
```
Il existe plusieurs métriques qui ne sont pas activées par défaut mais qui peuvent être liées de la manière décrite ci-dessus. Pour obtenir la liste complète, reportez-vous au [référentiel GitHub officiel de Micrometer](https://github.com/micrometer-metrics/micrometer/tree/master/micrometer-core/src/main/java/io/micrometer/core/instrument/binder ).

### <a name="non-spring-apps"></a>Applications non Spring
Ajoutez le code de liaison suivant au fichier de configuration :
```Java 
    New GuavaCacheMetrics().bind(registry);
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur Micrometer, consultez la [documentation officielle Micrometer](https://micrometer.io/docs).
* Pour en savoir plus sur Spring sur Azure, consultez la [documentation officielle Spring sur Azure](/java/azure/spring-framework/?view=azure-java-stable).