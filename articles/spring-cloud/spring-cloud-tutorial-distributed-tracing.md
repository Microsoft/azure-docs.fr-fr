---
title: Utiliser le suivi distribué avec Azure Spring Cloud
description: Découvrez comment utiliser le suivi distribué de Spring Cloud par le biais d’Azure Application Insights
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 10/06/2019
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: a78aec8c18f3b89629bbf696de3a097397ac59bc
ms.sourcegitcommit: 2a8a53e5438596f99537f7279619258e9ecb357a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "94337914"
---
# <a name="use-distributed-tracing-with-azure-spring-cloud"></a>Utiliser le suivi distribué avec Azure Spring Cloud

Avec les outils de suivi distribué d’Azure Spring Cloud, vous pouvez facilement déboguer et superviser les problèmes complexes. Azure Spring Cloud intègre Azure [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) au service [Application Insights](../azure-monitor/app/app-insights-overview.md) d’Azure. Cette intégration fournit une fonctionnalité puissante de suivi distribué qui est disponible dans le portail Azure.

::: zone pivot="programming-language-csharp"
Dans cet article, vous allez apprendre à activer une application .NET Core Steeltoe pour utiliser le traçage distribué.

## <a name="prerequisites"></a>Prérequis

Pour suivre ces procédures, vous avez besoin d’une application Steeltoe déjà [préparée pour le déploiement sur Azure Spring Cloud](spring-cloud-tutorial-prepare-app-deployment.md).

## <a name="dependencies"></a>Dépendances

Pour Steeltoe 2.4.4, ajoutez les packages NuGet suivants :

* [Steeltoe.Management.TracingCore](https://www.nuget.org/packages/Steeltoe.Management.TracingCore/)
* [Steeltoe.Management.ExporterCore](https://www.nuget.org/packages/Microsoft.Azure.SpringCloud.Client/)

Pour Steeltoe 3.0.0, ajoutez le package NuGet suivant :

* [Steeltoe.Management.TracingCore](https://www.nuget.org/packages/Steeltoe.Management.TracingCore/)

## <a name="update-startupcs"></a>Mettre à jour Startup.cs

1. Pour Steeltoe 2.4.4, appelez `AddDistributedTracing` et `AddZipkinExporter` dans la méthode `ConfigureServices`.

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDistributedTracing(Configuration);
       services.AddZipkinExporter(Configuration);
   }
   ```

   Pour Steeltoe 3.0.0, appelez `AddDistributedTracing` dans la méthode `ConfigureServices`.

   ```csharp
   public void ConfigureServices(IServiceCollection services)
   {
       services.AddDistributedTracing(Configuration, builder => builder.UseZipkinWithTraceOptions(services));
   }
   ```

1. Pour Steeltoe 2.4.4, appelez `UseTracingExporter` dans la méthode `Configure`.

   ```csharp
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
        app.UseTracingExporter();
   }
   ```

   Pour Steeltoe 3.0.0, aucune modification n’est requise dans la méthode `Configure`.

## <a name="update-configuration"></a>Mettre à jour la configuration

Ajoutez les paramètres suivants à la source de configuration qui sera utilisée lors de l’exécution de l’application dans Azure Spring Cloud :

1. Définissez `management.tracing.alwaysSample` sur True.

2. Si vous souhaitez afficher les étendues de suivi envoyées entre le serveur Eureka, le serveur de configuration et les applications utilisateur : définissez `management.tracing.egressIgnorePattern` sur /api/v2/spans|/v2/apps/. */permissions|/eureka/.* |/oauth/.*.

Par exemple, *appSettings.json* comprend les propriétés suivantes :
 
```json
"management": {
    "tracing": {
      "alwaysSample": true,
      "egressIgnorePattern": "/api/v2/spans|/v2/apps/.*/permissions|/eureka/.*|/oauth/.*"
    }
  }
```

Pour plus d’informations sur le traçage distribué dans les applications Steeltoe .NET Core, consultez [Traçage distribué](https://steeltoe.io/docs/3/tracing/distributed-tracing) dans la documentation de Steeltoe.
::: zone-end
::: zone pivot="programming-language-java"
Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Activer le suivi distribué dans le portail Azure
> * Ajouter Spring Cloud Sleuth à votre application.
> * Afficher des cartes de dépendances pour vos applications de microservices
> * Lancer des recherches dans des données de suivi avec différents filtres

## <a name="prerequisites"></a>Prérequis

Pour suivre ces procédures, vous avez besoin d’un service Azure Spring Cloud déjà provisionné et en cours d’exécution. Effectuez le démarrage rapide [Déployer votre première application Azure Spring Cloud](spring-cloud-quickstart.md) pour approvisionner et exécuter un service Azure Spring Cloud.

## <a name="add-dependencies"></a>Ajout de dépendances

1. Ajoutez la ligne suivante au fichier application.properties :

   ```xml
   spring.zipkin.sender.type = web
   ```

   Après cette modification, l’expéditeur Zipkin peut envoyer des données vers le Web.

1. Vous pouvez ignorer l’étape suivante si vous avez suivi notre [guide de préparation d’une application Azure Spring Cloud](spring-cloud-tutorial-prepare-app-deployment.md). Sinon, accédez à votre environnement de développement local et modifiez votre fichier pom.xml en y ajoutant la dépendance Spring Cloud Sleuth suivante :

    ```xml
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-sleuth</artifactId>
                <version>${spring-cloud-sleuth.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-sleuth</artifactId>
        </dependency>
    </dependencies>
    ```

1. Générez et déployez à nouveau votre service Azure Spring Cloud pour refléter ces changements.

## <a name="modify-the-sample-rate"></a>Modifier le taux d’échantillonnage

Vous pouvez changer la fréquence de collecte de vos données de télémétrie en modifiant le taux d’échantillonnage. Par exemple, si vous souhaitez échantillonner deux fois moins souvent, ouvrez le fichier application.properties et modifiez la ligne suivante :

```xml
spring.sleuth.sampler.probability=0.5
```

Si vous avez déjà créé et déployé une application, vous pouvez modifier le taux d’échantillonnage. Pour ce faire, ajoutez la ligne précédente en tant que variable d’environnement dans Azure CLI ou dans le portail Azure.
::: zone-end

## <a name="enable-application-insights"></a>Activer Application Insights

1. Accédez à la page du service Azure Spring Cloud dans le portail Azure.
1. Dans la section **Supervision**, sélectionnez **Suivi distribué**.
1. Sélectionnez **Modifier le paramètre** pour modifier ou ajouter un nouveau paramètre.
1. Créez une ressource Application Insights ou sélectionnez-en une existante.
1. Choisissez la catégorie de journalisation à superviser, puis spécifiez la durée de conservation (en jours).
1. Sélectionnez **Appliquer** pour appliquer le nouveau suivi.

## <a name="view-the-application-map"></a>Ouvrir la Cartographie d’application

Revenez à la page **Suivi distribué** et sélectionnez **Afficher la cartographie d’application**. Passez en revue la représentation visuelle de votre application et les paramètres de supervision. Pour savoir comment utiliser la cartographie d’application, consultez [Cartographie d’application : trier des applications distribuées](../azure-monitor/app/app-map.md).

## <a name="use-search"></a>Utiliser la recherche

Utilisez la fonction de recherche pour interroger certaines données de télémétrie. Dans la page **Suivi distribué**, sélectionnez **Rechercher**. Pour plus d’informations sur l’utilisation de la fonction de recherche, consultez [Utilisation de la recherche dans Application Insights](../azure-monitor/app/diagnostic-search.md).

## <a name="use-application-insights"></a>Utiliser Application Insights

En plus des fonctionnalités de cartographie d’application et de recherche, Application Insights fournit des fonctions de supervision. Recherchez le nom de votre application dans le portail Azure, puis ouvrez une page Application Insights pour accéder aux informations de supervision. Pour plus d’informations sur l’utilisation de ces outils, consultez [Requêtes de journal Azure Monitor](/azure/data-explorer/kusto/query/).

## <a name="disable-application-insights"></a>Désactiver Application Insights

1. Accédez à la page du service Azure Spring Cloud dans le portail Azure.
1. Dans la section **Supervision**, sélectionnez **Suivi distribué**.
1. Sélectionnez **Désactiver** pour désactiver Application Insights.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris ce qu’est le suivi distribué dans Azure Spring Cloud et vous avez vu comment l’activer. Pour en savoir plus sur la liaison de services à une application, consultez [Lier une base de données Azure Cosmos DB à une application Azure Spring Cloud](spring-cloud-tutorial-bind-cosmos.md).