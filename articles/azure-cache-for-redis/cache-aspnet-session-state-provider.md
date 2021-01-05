---
title: Fournisseur d’état de session ASP.NET du cache
description: Découvrez comment stocker l’état de session ASP.NET en mémoire avec Azure Cache for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-dotnet
ms.date: 05/01/2017
ms.openlocfilehash: ce77f5074d707da5cfb251a103653b96e4644b5f
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92544526"
---
# <a name="aspnet-session-state-provider-for-azure-cache-for-redis"></a>Fournisseur d’États de session ASP.NET pour le Cache Azure pour Redis

Le Cache Azure pour Redis intègre un fournisseur d’État de session qui vous permet de stocker l’État de votre session en mémoire avec le Cache Azure pour Redis plutôt qu’avec une base de données SQL Server. Pour utiliser le fournisseur d’État de session de la mise en cache, configurez d’abord votre cache, puis configurez votre application ASP.NET pour la mise en cache à l’aide du package NuGet de l’État de session du Cache Azure pour Redis. Pour les applications ASP.NET Core, consultez [Gestion de session et d’état dans ASP.NET Core](/aspnet/core/fundamentals/app-state).

Dans une application cloud réelle, il n’est souvent pas pratique d’éviter de stocker une forme d’état de session utilisateur, mais certaines approches ont davantage d’incidence que d’autres sur les performances et l'extensibilité. Si vous devez stocker un état, la meilleure solution consiste à veiller à ce qu’il reste de petite taille et à le stocker dans des cookies. Si cette approche est impossible, le mieux est d’utiliser l’état de session ASP.NET avec un fournisseur de cache distribué en mémoire. La pire solution du point de vue des performances et de l’extensibilité consiste à utiliser un fournisseur d’état de session s'appuyant sur une base de données. Cette rubrique comporte des conseils sur l’utilisation du fournisseur d’État de session ASP.NET concernant le Cache Azure pour Redis. Pour plus d’informations sur les autres options d’état de session, consultez [Options d’état de session ASP.NET](#aspnet-session-state-options).

## <a name="store-aspnet-session-state-in-the-cache"></a>Stockage de l'état de session ASP.NET dans le cache

Pour configurer une application cliente dans Visual Studio avec le package NuGet de l’État de session du Cache Azure pour Redis, cliquez sur **Gestionnaire de package NuGet** , **Console du Gestionnaire de package** dans le menu **Outils** .

Exécutez la commande suivante depuis la fenêtre `Package Manager Console`.
    

```powershell
Install-Package Microsoft.Web.RedisSessionStateProvider
```

> [!IMPORTANT]
> Si vous utilisez la fonction de clustering du niveau Premium, vous devez utiliser [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) version 2.0.1 ou ultérieure, sans quoi une exception est levée. Passer à la version 2.0.1 ou ultérieure est une modification avec rupture. Pour plus d’informations, consultez [Détails de la modification avec rupture pour la version 2.0.0](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details). Au moment de la mise à jour de cet article, la version actuelle de ce package est 2.2.3.
> 
> 

Le package NuGet du fournisseur d’état de session Redis a une dépendance sur le package StackExchange.Redis.StrongName. Le package StackExchange.Redis.StrongName est automatiquement installé s’il ne figure pas déjà dans votre projet.

>[!NOTE]
>Il existe, en plus du package StackExchange.Redis.StrongName avec nom fort, une version de StackExchange.Redis sans nom fort. Si votre projet utilise la version StackExchange.Redis sans nom fort, vous devez la désinstaller, sans quoi vous aurez des conflits de noms dans votre projet. Pour plus d’informations sur ces packages, consultez la section [Configuration des clients de cache .NET](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).
>
>

Le package NuGet télécharge et ajoute les références d’assembly nécessaires et ajoute la section suivante dans votre fichier web.config. Cette section contient la configuration requise pour que votre application ASP.NET utilise le fournisseur d’État de session du Cache Azure pour Redis.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
  <providers>
    <!-- Either use 'connectionString' OR 'settingsClassName' and 'settingsMethodName' OR use 'host','port','accessKey','ssl','connectionTimeoutInMilliseconds' and 'operationTimeoutInMilliseconds'. -->
    <!-- 'throwOnError','retryTimeoutInMilliseconds','databaseId' and 'applicationName' can be used with both options. -->
    <!--
      <add name="MySessionStateStore" 
        host = "127.0.0.1" [String]
        port = "" [number]
        accessKey = "" [String]
        ssl = "false" [true|false]
        throwOnError = "true" [true|false]
        retryTimeoutInMilliseconds = "5000" [number]
        databaseId = "0" [number]
        applicationName = "" [String]
        connectionTimeoutInMilliseconds = "5000" [number]
        operationTimeoutInMilliseconds = "1000" [number]
        connectionString = "<Valid StackExchange.Redis connection string>" [String]
        settingsClassName = "<Assembly qualified class name that contains settings method specified below. Which basically return 'connectionString' value>" [String]
        settingsMethodName = "<Settings method should be defined in settingsClass. It should be public, static, does not take any parameters and should have a return type of 'String', which is basically 'connectionString' value.>" [String]
        loggingClassName = "<Assembly qualified class name that contains logging method specified below>" [String]
        loggingMethodName = "<Logging method should be defined in loggingClass. It should be public, static, does not take any parameters and should have a return type of System.IO.TextWriter.>" [String]
        redisSerializerType = "<Assembly qualified class name that implements Microsoft.Web.Redis.ISerializer>" [String]
      />
    -->
    <add name="MySessionStateStore" type="Microsoft.Web.Redis.RedisSessionStateProvider"
         host=""
         accessKey=""
         ssl="true" />
  </providers>
</sessionState>
```

La section commentée fournit un exemple d’attributs et de paramétrage pour chacun de ces attributs.

Configurez les attributs avec les valeurs du panneau de votre cache sur le portail Microsoft Azure et configurez les autres valeurs selon votre choix. Pour obtenir des instructions sur l’accès aux propriétés de votre cache, voir [Configurer les paramètres du Cache Azure pour Redis](cache-configure.md#configure-azure-cache-for-redis-settings).

* **host** : spécifiez le point de terminaison de votre cache.
* **port**  : utilisez votre port non TLS/SSL ou votre port TLS/SSL, selon les paramètres TLS.
* **accessKey** : utilisez la clé primaire ou secondaire pour votre cache.
* **ssl**  : choisissez « true » si vous souhaitez sécuriser les communications cache/client avec TLS ; sinon, choisissez « false ». Veillez à spécifier le port approprié.
  * Le port non TLS est désactivé par défaut pour les nouveaux caches. Spécifiez True pour ce paramètre afin d’utiliser le port TLS. Pour plus d’informations sur l’activation du port non TLS, consultez la section relative aux [ports d’accès](cache-configure.md#access-ports) dans la rubrique [Configuration d’un cache](cache-configure.md).
* **throwOnError** : true si vous voulez lever une exception en cas d’échec, ou false si vous souhaitez que l’opération échoue en silence. Pour contrôler un échec, vous pouvez vérifier la propriété statique Microsoft.Web.Redis.RedisSessionStateProvider.LastException. La valeur par défaut est true.
* **retryTimeoutInMilliseconds**  : intervalle, en millisecondes, au cours duquel interviennent les nouvelles tentatives d’exécution des opérations ayant échoué. La première nouvelle tentative intervient après 20 millisecondes. Les tentatives suivantes se produisent à chaque seconde jusqu’à l’expiration de l’intervalle retryTimeoutInMilliseconds. Une dernière tentative d’exécution est effectuée immédiatement après cet intervalle. Si le problème persiste, l’exception est renvoyée à l’appelant, en fonction du paramètre throwOnError. La valeur 0 par défaut signifie qu’aucune nouvelle tentative n’est effectuée.
* **databaseId** : spécifie la base de données à utiliser pour les données de sortie du cache. Si ce champ n’est pas spécifié, la valeur 0 sera utilisée par défaut.
* **applicationName`{<Application Name>_<Session ID>}_Data` : les clés sont stockées dans redis sous** . Ce schéma d’affectation permet à plusieurs applications de partager la même instance Redis. Ce paramètre est facultatif. Si vous n’indiquez aucune valeur, une valeur par défaut sera utilisée.
* **connectionTimeoutInMilliseconds** : ce paramètre vous permet de remplacer le paramètre connectTimeout dans le client StackExchange.Redis. S’il n’est pas spécifié, le paramètre par défaut connectTimeout 5000 est utilisé. Pour plus d’informations, consultez le [modèle de configuration StackExchange.Redis](https://go.microsoft.com/fwlink/?LinkId=398705).
* **operationTimeoutInMilliseconds** : ce paramètre vous permet de remplacer le paramètre syncTimeout dans le client StackExchange.Redis. S’il n’est pas spécifié, le paramètre par défaut syncTimeout 1000 est utilisé. Pour plus d’informations, consultez le [modèle de configuration StackExchange.Redis](https://go.microsoft.com/fwlink/?LinkId=398705).
* **redisSerializerType**  : ce paramètre vous permet de spécifier une sérialisation personnalisée du contenu de session envoyé à Redis. Le type spécifié doit implémenter `Microsoft.Web.Redis.ISerializer` et déclarer un constructeur public sans paramètre. Par défaut, `System.Runtime.Serialization.Formatters.Binary.BinaryFormatter` est utilisé.

Pour plus d’informations sur ces propriétés, consultez la publication du blog d’origine [Announcing ASP.NET Session State Provider for Redis](https://devblogs.microsoft.com/aspnet/announcing-asp-net-session-state-provider-for-redis-preview-release/)(en anglais).

N’oubliez pas de commenter la section relative au fournisseur d’état de session standard InProc dans votre fichier web.config.

```xml
<!-- <sessionState mode="InProc"
     customProvider="DefaultSessionProvider">
     <providers>
        <add name="DefaultSessionProvider"
              type="System.Web.Providers.DefaultSessionStateProvider,
                    System.Web.Providers, Version=1.0.0.0, Culture=neutral,
                    PublicKeyToken=31bf3856ad364e35"
              connectionStringName="DefaultConnection" />
      </providers>
</sessionState> -->
```

Une fois ces étapes effectuées, votre application est configurée pour utiliser le fournisseur d’État de session du Cache Azure pour Redis. Lorsque vous utilisez l’État de session dans votre application, il est stocké dans une instance de Cache Azure pour Redis.

> [!IMPORTANT]
> Les données stockées dans le cache doivent être sérialisables, contrairement aux données qui peuvent être stockées dans le fournisseur d’état de session en mémoire ASP.NET par défaut. Lorsque vous utilisez le fournisseur d’état de session pour Redis, assurez-vous que les types de données qui sont stockés dans l’état de session sont sérialisables.
> 
> 

## <a name="aspnet-session-state-options"></a>Options d’état de session ASP.NET

* Fournisseur d’état de session en mémoire : ce fournisseur stocke l’état de session dans la mémoire. Ce fournisseur présente l’avantage d’être à la fois simple et rapide. Vous ne pourrez cependant pas faire évoluer vos applications Web si vous utilisez le fournisseur en mémoire dans la mesure où il n’est pas distribué.
* Fournisseur d’état de session dans le serveur SQL : ce fournisseur stocke l’état de session dans le serveur SQL. Utilisez ce fournisseur si vous souhaitez stocker l’état de session dans un stockage permanent. Vous pouvez faire évoluer votre application Web, mais l’utilisation du serveur SQL pour la session a un impact sur les performances de votre application Web. Vous pouvez également utiliser ce fournisseur avec une [configuration OLTP en mémoire](/archive/blogs/sqlserverstorageengine/asp-net-session-state-with-sql-server-in-memory-oltp) pour aider à améliorer les performances.
* Fournisseur d’État de session distribué en mémoire (par exemple, fournisseur d’État de session du Cache Azure pour Redis) : ce fournisseur associe tous les avantages. Votre application Web peut bénéficier d’un fournisseur d’état de session à la fois simple, rapide et évolutif. Étant donné que ce fournisseur stocke l’état de session dans un cache et que votre application doit prendre en considération toutes les caractéristiques associées lorsqu’elle communique avec un cache distribué en mémoire (par exemple, les défaillances réseau temporaires). Pour obtenir des recommandations sur l’utilisation du Cache, consultez la page [Caching Guidance](/azure/architecture/best-practices/caching) dans le [Guide de conception et d’implémentation des applications Azure Cloud](https://github.com/mspnp/azure-guidance) de Microsoft Patterns & Practices.

Pour plus d'informations sur l’état de session et obtenir d’autres meilleures pratiques, consultez la page [Web Development Best Practices (Building Real-World Cloud Apps with Azure)](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/web-development-best-practices).

## <a name="third-party-session-state-providers"></a>Fournisseurs d’état de session tiers

* [NCache](https://www.alachisoft.com/ncache/session-index.html)
* [Apache Ignite](https://apacheignite-net.readme.io/docs/aspnet-session-state-caching)

## <a name="next-steps"></a>Étapes suivantes

Consultez l’article [Fournisseur de caches de sortie ASP.NET pour le Cache Azure pour Redis](cache-aspnet-output-cache-provider.md).