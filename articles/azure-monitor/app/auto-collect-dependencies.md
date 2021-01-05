---
title: Azure Application Insights - Collecte automatique de dépendance | Microsoft Docs
description: Application Insights collecte et visualise automatiquement les dépendances
ms.topic: reference
ms.custom: devx-track-dotnet
ms.date: 05/06/2020
ms.openlocfilehash: 8a4d79e52465e93fb4db2625217cb37a06917218
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91930864"
---
# <a name="dependency-auto-collection"></a>Collecte automatique de dépendance

Voici la liste des appels de dépendance actuellement pris en charge qui sont automatiquement détectés comme des dépendances sans devoir apporter des modifications supplémentaires au code de votre application. Ces dépendances sont visualisées dans les vues [Cartographie d’application](./app-map.md) et [Diagnostics des transactions](./transaction-diagnostics.md) d’Application Insights. Si votre dépendance ne figure pas dans la liste ci-dessous, vous pouvez toujours la suivre manuellement avec un [appel de suivi des dépendances](./api-custom-events-metrics.md#trackdependency).

## <a name="net"></a>.NET

| Infrastructures d’application| Versions |
| ------------------------|----------|
| Formulaires web ASP.NET | 4.5+ |
| ASP.NET MVC | 4+ |
| Web API ASP.NET | 4.5+ |
| ASP.NET Core | 1.1+ |
| <b> Bibliothèques de communication</b> |
| [HttpClient](https://www.microsoft.com/net/) | 4.5+, .NET Core 1.1+ |
| [SqlClient](https://www.nuget.org/packages/System.Data.SqlClient) | .NET Core 1.0+, NuGet 4.3.0 |
| [Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient/1.1.2)| 1.1.0 - dernière version stable. (Voir la remarque ci-dessous.)
| [Kit de développement logiciel (SDK) client EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs) | 1.1.0 |
| [Kit de développement logiciel (SDK) client ServiceBus](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus) | 3.0.0 |
| <b>Clients de stockage</b>|  |
| ADO.NET | 4.5+ |

> [!NOTE]
> Il existe un [problème connu](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1347) avec les versions antérieures de Microsoft.Data.SqlClient. Nous vous recommandons d’utiliser la version 1.1.0 ou ultérieure pour atténuer ce problème. Entity Framework Core n’est pas nécessairement fourni avec la dernière version stable de Microsoft.Data.SqlClient. Nous vous recommandons donc de confirmer que vous disposez d’au moins la version 1.1.0 pour éviter ce problème.   


## <a name="java"></a>Java
| Serveurs d’applications | Versions |
|-------------|----------|
| [Tomcat](https://tomcat.apache.org/) | 7, 8 | 
| [EAP JBoss](https://developers.redhat.com/products/eap/download/) | 6, 7 |
| [Jetty](https://www.eclipse.org/jetty/) | 9 |
| <b>Infrastructures d’application </b> |  |
| [Spring](https://spring.io/) | 3.0 |
| [Spring Boot](https://spring.io/projects/spring-boot) | 1.5.9+<sup>*</sup> |
| Servlet Java | 3.1+ |
| <b>Bibliothèques de communication</b> |  |
| [Client Http Apache](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient) | 4.3+<sup>†</sup> |
| <b>Clients de stockage</b> | |
| [SQL Server]( https://mvnrepository.com/artifact/com.microsoft.sqlserver/mssql-jdbc) | 1+<sup>†</sup> |
| [PostgreSQL (prise en charge de la version bêta)](https://github.com/Microsoft/ApplicationInsights-Java/blob/master/CHANGELOG.md#version-240-beta) | |
| [Oracle]( https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html) | 1+<sup>†</sup> |
| [MySql]( https://mvnrepository.com/artifact/mysql/mysql-connector-java) | 1+<sup>†</sup> |
| <b>Bibliothèques de journalisation</b> | |
| [Logback](https://logback.qos.ch/) | 1+ |
| [Log4j](https://logging.apache.org/log4j/) | 1.2+ |
| <b>Bibliothèques de mesures</b> |  |
| JMX | 1.0+ |

> [!NOTE]
> \* Sauf prise en charge de programmation réactive.
> <br>† Installation de [Agent JVM](./java-agent.md#install-the-application-insights-agent-for-java) requise.

## <a name="nodejs"></a>Node.js

| Bibliothèques de communication | Versions |
| ------------------------|----------|
| [HTTP](https://nodejs.org/api/http.html), [HTTPS](https://nodejs.org/api/https.html) | 0.10+ |
| <b>Clients de stockage</b> | |
| [Redis](https://www.npmjs.com/package/redis) | 2.x |
| [MongoDb](https://www.npmjs.com/package/mongodb) ; [MongoDb Core](https://www.npmjs.com/package/mongodb-core) | 2.x - 3.x |
| [MySQL](https://www.npmjs.com/package/mysql) | 2.0.0 - 2.16.x |
| [PostgreSql](https://www.npmjs.com/package/pg) ; | 6.x - 7.x |
| [pg-pool](https://www.npmjs.com/package/pg-pool) | 1.x - 2.x |
| <b>Bibliothèques de journalisation</b> | |
| [console](https://nodejs.org/api/console.html) | 0.10+ |
| [Bunyan](https://www.npmjs.com/package/bunyan) | 1.x |
| [Winston](https://www.npmjs.com/package/winston) | 2.x - 3.x |

## <a name="javascript"></a>JavaScript

| Bibliothèques de communication | Versions |
| ------------------------|----------|
| [XMLHttpRequest](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) | Tous |

## <a name="next-steps"></a>Étapes suivantes

- Configurez le suivi des dépendances personnalisé pour [.NET](./asp-net-dependencies.md).
- Configurez le suivi des dépendances personnalisé pour [Java](./java-agent.md).
- Configurez le suivi personnalisé des dépendances pour [OpenCensus Python](./opencensus-python-dependency.md).
- [Écrire des données de télémétrie des dépendances personnalisées](./api-custom-events-metrics.md#trackdependency)
- Pour connaître les types et les modèles de données Application Insights, consultez [Modèle de données](./data-model.md).
- Découvrez quelles [plateformes](./platforms.md) sont prises en charge par Application Insights.

