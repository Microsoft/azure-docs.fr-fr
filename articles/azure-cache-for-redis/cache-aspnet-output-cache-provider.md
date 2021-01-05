---
title: Fournisseur de caches de sortie ASP.NET pour le Cache Azure pour Redis
description: Découvrez comment mettre en cache une sortie de pages ASP.NET à l’aide d’Azure Cache pour Redis. Le fournisseur de caches de sortie Redis est un mécanisme de stockage hors processus pour les données de cache de sortie.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.date: 04/22/2018
ms.openlocfilehash: 6d711b07a10e04dcdf31259f3e53c9687af28e28
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993393"
---
# <a name="aspnet-output-cache-provider-for-azure-cache-for-redis"></a>Fournisseur de caches de sortie ASP.NET pour le Cache Azure pour Redis

Le fournisseur de caches de sortie Redis est un mécanisme de stockage hors processus pour les données de cache de sortie. Ces données concernent spécialement les réponses HTTP complètes (mise en cache de la sortie de pages). Le fournisseur se connecte au nouveau point d'extension du fournisseur de caches de sortie introduit dans ASP.NET 4. Pour des informations sur les applications ASP.NET Core, consultez [Mise en cache des réponses dans ASP.NET Core](/aspnet/core/performance/caching/response). 

Pour utiliser le fournisseur de caches de sortie Redis, configurez d’abord votre cache, puis configurez votre application ASP.NET en utilisant le package NuGet du fournisseur de caches de sortie Redis. Cette rubrique fournit des conseils sur la configuration de votre application pour utiliser le fournisseur de caches de sortie Redis. Pour plus d’informations sur la création et la configuration d’une instance de Cache Azure pour Redis, voir [Créer un cache](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).

## <a name="store-aspnet-page-output-in-the-cache"></a>Stockage de la sortie de pages ASP.NET dans le cache

Pour configurer une application cliente dans Visual Studio avec le package NuGet de l’État de session du Cache Azure pour Redis, cliquez sur **Gestionnaire de package NuGet**, **Console du Gestionnaire de package** dans le menu **Outils**.

Exécutez la commande suivante depuis la fenêtre `Package Manager Console`.

```
Install-Package Microsoft.Web.RedisOutputCacheProvider
```

Le package NuGet du fournisseur de caches de sortie Redis a une dépendance sur le package StackExchange.Redis.StrongName. Le package StackExchange.Redis.StrongName est automatiquement installé s’il ne figure pas déjà dans votre projet. Pour plus d’informations sur le package NuGet du fournisseur de caches de sortie Redis, consultez la page NuGet [RedisOutputCacheProvider](https://www.nuget.org/packages/Microsoft.Web.RedisOutputCacheProvider/).

>[!NOTE]
>Il existe, en plus du package StackExchange.Redis.StrongName avec nom fort, une version de StackExchange.Redis sans nom fort. Si votre projet utilise la version StackExchange.Redis sans nom fort, vous devez la désinstaller, sans quoi vous rencontrerez des conflits de noms dans votre projet. Pour plus d’informations sur ces packages, consultez la section [Configuration des clients de cache .NET](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).

Le package NuGet télécharge et ajoute les références d’assembly nécessaires et ajoute la section suivante dans votre fichier web.config. Cette section contient la configuration requise pour que votre application ASP.NET utilise le fournisseur de cache de sortie Redis.

```xml
<caching>
  <outputCache defaultProvider="MyRedisOutputCache">
    <providers>
      <add name="MyRedisOutputCache" type="Microsoft.Web.Redis.RedisOutputCacheProvider"
           host=""
           accessKey=""
           ssl="true" />
    </providers>
  </outputCache>
</caching>
```

Configurez les attributs avec les valeurs du panneau de votre cache sur le portail Microsoft Azure et configurez les autres valeurs selon votre choix. Pour obtenir des instructions sur l’accès aux propriétés de votre cache, voir [Configurer les paramètres du Cache Azure pour Redis](cache-configure.md#configure-azure-cache-for-redis-settings).

| Attribut | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| *host* | string | "localhost" | Le nom hôte ou l’adresse IP du serveur Redis |
| *port* | entier positif | 6379 (non TLS/SSL)<br/>6380 (TLS/SSL) | Port du serveur Redis |
| *accessKey* | string | "" | Mot de passe Redis lorsque l’autorisation Redis est activée. La valeur est une chaîne vide par défaut, ce qui signifie que le fournisseur d’état de session ne peut pas utiliser n’importe quel mot de passe pour la connexion au serveur Redis. **Si votre serveur Redis se trouve sur un réseau accessible publiquement comme le Cache Redis Azure, activez l’autorisation Redis pour améliorer la sécurité et fournir un mot de passe sécurisé.** |
| *ssl* | boolean | **false** | Indique s’il faut ou non se connecter au serveur Redis via TLS. La valeur est **false** par défaut, car Redis ne prend pas en charge TLS sans configuration préalable. **Si vous utilisez le Cache Redis Azure avec SSL sans configuration préalable, sélectionnez la valeur true pour améliorer la sécurité.**<br/><br/>Le port non TLS est désactivé par défaut pour les nouveaux caches. Spécifiez **true** pour ce paramètre afin d’utiliser le port non TLS. Pour plus d’informations sur l’activation du port non TLS, consultez la section relative aux [ports d’accès](cache-configure.md#access-ports) dans la rubrique [Configuration d’un cache](cache-configure.md). |
| *databaseIdNumber* | entier positif | 0 | *Cet attribut peut uniquement être spécifié par le biais de web.config ou AppSettings.*<br/><br/>Spécifie la base de données Redis à utiliser. |
| *connectionTimeoutInMilliseconds* | entier positif | Fourni par StackExchange.Redis | Permet de définir *ConnectTimeout* lors de la création de StackExchange.Redis.ConnectionMultiplexer. |
| *operationTimeoutInMilliseconds* | entier positif | Fourni par StackExchange.Redis | Permet de définir *SyncTimeout* lors de la création de StackExchange.Redis.ConnectionMultiplexer. |
| *connectionString* (chaîne de connexion StackExchange.Redis valide) | string | *n/a* | Référence de paramètre à AppSettings ou web.config, ou une chaîne de connexion StackExchange.Redis valide. Cet attribut peut fournir des valeurs pour l’*hôte*, la *port*, *accessKey*, *ssl* et d’autres attributs de StackExchange.Redis. Pour plus de détail sur *connectionString*, consultez [Paramétrage de connectionString](#setting-connectionstring) dans la section [Remarques sur les attributs](#attribute-notes). |
| *settingsClassName*<br/>*settingsMethodName* | string<br/>string | *n/a* | *Ces attributs peuvent uniquement être spécifiés par le biais de web.config ou AppSettings.*<br/><br/>Utilisez ces attributs pour fournir une chaîne de connexion. *settingsClassName* doit être un nom de classe d’assembly qualifié qui contient la méthode spécifiée par *settingsMethodName*.<br/><br/>La méthode spécifiée par *settingsMethodName* doit être publique, statique et vide (sans aucun paramètre), avec un type de retour **chaîne**. Cette méthode retourne la chaîne de connexion réelle. |
| *loggingClassName*<br/>*loggingMethodName* | string<br/>string | *n/a* | *Ces attributs peuvent uniquement être spécifiés par le biais de web.config ou AppSettings.*<br/><br/>Utilisez ces attributs pour déboguer votre application en fournissant des journaux à partir du Cache de l’état de session/sortie, ainsi que des journaux à partir de StackExchange.Redis. *loggingClassName* doit être un nom de classe d’assembly qualifié qui contient la méthode spécifiée par *loggingMethodName*.<br/><br/>La méthode spécifiée par *loggingMethodName* doit être publique, statique et vide (sans aucun paramètre), avec un type de retour **System.IO.TextWriter**. |
| *applicationName* | string | Le nom du module du processus en cours ou « / » | *SessionStateProvider uniquement*<br/>*Cet attribut peut uniquement être spécifié par le biais de web.config ou AppSettings.*<br/><br/>Le préfixe de nom d’application à utiliser dans le cache Redis. Le client peut utiliser le même cache Redis à des fins différentes. Pour s’assurer que les clés de session ne sont pas en conflit, il est possible de les préfixer avec le nom de l’application. |
| *throwOnError* | boolean | true | *SessionStateProvider uniquement*<br/>*Cet attribut peut uniquement être spécifié par le biais de web.config ou AppSettings.*<br/><br/>Indique s’il faut ou non lever une exception lorsqu’une erreur se produit.<br/><br/>Pour plus d’informations sur *throwOnError*, consultez [Remarques sur *throwOnError*](#notes-on-throwonerror) dans la section [Remarques sur les attributs](#attribute-notes). |>*Microsoft.Web.Redis.RedisSessionStateProvider.LastException*. |
| *retryTimeoutInMilliseconds* | entier positif | 5 000 | *SessionStateProvider uniquement*<br/>*Cet attribut peut uniquement être spécifié par le biais de web.config ou AppSettings.*<br/><br/>Combien de nouvelles tentatives en cas d’échec d’une opération. Si cette valeur est inférieure à *operationTimeoutInMilliseconds*, le fournisseur ne fera pas de nouvelles tentatives.<br/><br/>Pour plus d’informations sur *retryTimeoutInMilliseconds*, consultez [Remarques sur *retryTimeoutInMilliseconds*](#notes-on-retrytimeoutinmilliseconds) dans la section [Remarques sur les attributs](#attribute-notes). |
| *redisSerializerType* | string | *n/a* | Spécifie le nom de type d’assembly qualifié d’une classe qui implémente Microsoft.Web.Redis. ISerializer et qui contient la logique personnalisée pour sérialiser et désérialiser les valeurs. Pour plus d’informations, consultez [À propos de *redisSerializerType*](#about-redisserializertype) dans la section [Remarques sur les attributs](#attribute-notes). |

## <a name="attribute-notes"></a>Remarques sur les attributs

### <a name="setting-connectionstring"></a>Paramétrage de *connectionString*

La valeur de *connectionString* est utilisée comme clé pour extraire la chaîne de connexion réelle d’AppSettings, si cette chaîne existe dans AppSettings. Si cette valeur ne se trouve pas dans AppSettings, la valeur de *connectionString* sera utilisée comme clé pour extraire la chaîne de connexion réelle de la section **ConnectionString** de web.config, si cette section existe. Si la chaîne de connexion n’existe pas dans AppSettings ou dans la section web.config **ConnectionString**, la valeur littérale de *connectionString* sera utilisée comme la chaîne de connexion lors de la création de StackExchange.Redis.ConnectionMultiplexer.

Les exemples suivants illustrent comment *connectionString* est utilisé.

#### <a name="example-1"></a>Exemple 1

```xml
<connectionStrings>
    <add name="MyRedisConnectionString" connectionString="mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False" />
</connectionStrings>
```

Dans `web.config`, utilisez la clé ci-dessus comme valeur de paramètre au lieu de la valeur réelle.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "MyRedisConnectionString"/>
    </providers>
</sessionState>
```

#### <a name="example-2"></a>Exemple 2

```xml
<appSettings>
    <add key="MyRedisConnectionString" value="mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False" />
</appSettings>
```

Dans `web.config`, utilisez la clé ci-dessus comme valeur de paramètre au lieu de la valeur réelle.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "MyRedisConnectionString"/>
    </providers>
</sessionState>
```

#### <a name="example-3"></a>Exemple 3

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False"/>
    </providers>
</sessionState>
```

### <a name="notes-on-throwonerror"></a>Remarques sur *throwOnError*

Actuellement, si une erreur se produit pendant une opération de session, le fournisseur d’état de session lève une exception. Ceci ferme l’application.

Ce comportement a été modifié afin de répondre aux attentes des utilisateurs du fournisseur état de session ASP.NET existants, tout en offrant également la possibilité d’agir sur les exceptions si besoin. Le comportement par défaut lève toujours une exception lorsqu’une erreur se produit, de façon cohérente avec les autres fournisseurs d’état de session ASP.NET. Les codes d’erreur existants doivent rester les mêmes.

Si vous définissez *throwOnError* sur la valeur **false**, au lieu de lever une exception lorsqu’une erreur se produit, il échouera en mode silencieux. Pour voir si une erreur s’est produite et, dans ce cas, découvrir l’exception qui l’a provoquée, vérifiez la propriété statique *Microsoft.Web.Redis.RedisSessionStateProvider.LastException*.

### <a name="notes-on-retrytimeoutinmilliseconds"></a>Remarques sur *retryTimeoutInMilliseconds*

Cet élément fournit une logique de nouvelle tentative pour simplifier les cas dans lesquels une opération de session doit réessayer en cas d’échec en raison par exemple d’un problème réseau, tout en vous permettant de contrôler le délai d’attente de nouvelle tentative ou de refuser toute nouvelle tentative.

Si vous définissez *retryTimeoutInMilliseconds* sur un nombre, par exemple 2000, si une opération de session échoue, il réessaiera pendant 2000 millisecondes avant de traiter le problème comme une erreur. Par conséquent, pour que le fournisseur d’état de session applique cette logique de nouvelle tentative, il suffit de configurer le délai d’attente. La première nouvelle tentative aura lieu après 20 millisecondes, ce qui est suffisant dans la plupart des cas quand un problème réseau se produit. Après cela, il réessaiera toutes les secondes jusqu’à expiration. Juste après le l’expiration, il réessaiera encore une fois pour s’assurer qu’il ne réduit pas le délai d’attente d’une seconde (au plus).

Si vous ne pensez pas avoir besoin de nouvelles tentatives (par exemple, lorsque vous exécutez le serveur Redis sur le même ordinateur que votre application) ou si vous souhaitez gérer vous-même la logique de nouvelle tentative, définissez *retryTimeoutInMilliseconds* sur 0.

### <a name="about-redisserializertype"></a>À propos de *redisSerializerType*

Par défaut, la sérialisation pour stocker les valeurs sur Redis est effectuée dans un format binaire fourni par la classe **BinaryFormatter**. Utilisez *redisSerializerType* pour spécifier le nom de type d’assembly qualifié d’une classe qui implémente **Microsoft.Web.Redis.ISerializer** et possède la logique personnalisée pour sérialiser et désérialiser les valeurs. Par exemple, voici une classe de sérialiseur Json qui utilise JSON.NET :

```cs
namespace MyCompany.Redis
{
    public class JsonSerializer : ISerializer
    {
        private static JsonSerializerSettings _settings = new JsonSerializerSettings() { TypeNameHandling = TypeNameHandling.All };

        public byte[] Serialize(object data)
        {
            return Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(data, _settings));
        }

        public object Deserialize(byte[] data)
        {
            if (data == null)
            {
                return null;
            }
            return JsonConvert.DeserializeObject(Encoding.UTF8.GetString(data), _settings);
        }
    }
}
```

En supposant que cette classe est définie dans un assembly avec pour nom **MyCompanyDll**, vous pouvez définir le paramètre *redisSerializerType* à utiliser :

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvider"
             name = "MySessionStateStore"
             redisSerializerType = "MyCompany.Redis.JsonSerializer,MyCompanyDll"
             ... />
    </providers>
</sessionState>
```

## <a name="output-cache-directive"></a>Directive de cache de sortie

Ajoutez une directive OutputCache à chaque page pour laquelle vous voulez mettre en cache la sortie.

```xml
<%@ OutputCache Duration="60" VaryByParam="*" %>
```

Dans l’exemple précédent, les données de page mises en cache resteront dans le cache pendant 60 secondes et une version différente de la page est mise en cache pour chaque combinaison de paramètres. Pour plus d’informations sur la directive OutputCache, consultez [@OutputCache](/previous-versions/dotnet/netframework-4.0/hdxfb6cy(v=vs.100)).

Une fois ces étapes effectuées, votre application est configurée pour utiliser le fournisseur de caches de sortie Redis.

## <a name="third-party-output-cache-providers"></a>Fournisseur de caches de sortie tiers

* [NCache](https://www.alachisoft.com/blogs/how-to-use-a-distributed-cache-for-asp-net-output-cache/)
* [Apache Ignite](https://apacheignite-net.readme.io/docs/aspnet-output-caching)


## <a name="next-steps"></a>Étapes suivantes

Voir [Fournisseur d’États de session ASP.NET pour le Cache Azure pour Redis](cache-aspnet-session-state-provider.md).