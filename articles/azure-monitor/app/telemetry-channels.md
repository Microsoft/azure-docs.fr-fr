---
title: Canaux de télémétrie dans Azure Application Insights | Microsoft Docs
description: Découvrez comment personnaliser les canaux de télémétrie dans les SDK Azure Application Insights pour .NET et .NET Core.
ms.topic: conceptual
ms.date: 05/14/2019
ms.custom: devx-track-csharp
ms.reviewer: mbullwin
ms.openlocfilehash: fec7bfc16e2cc36d19c84b93b5b93c3c1365b166
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90564013"
---
# <a name="telemetry-channels-in-application-insights"></a>Canaux de télémétrie dans Application Insights

Les canaux de télémétrie font partie intégrante des [SDK Azure Application Insights](./app-insights-overview.md). Ils gèrent la mise en mémoire tampon des données de télémétrie et leur transmission au service Application Insights. Les versions .NET et .NET Core des SDK intègrent deux canaux de télémétrie : `InMemoryChannel` et `ServerTelemetryChannel`. Cet article décrit chaque canal en détail, notamment comment personnaliser leur comportement.

## <a name="what-are-telemetry-channels"></a>Que sont les canaux de télémétrie ?

Les canaux de télémétrie gèrent la mise en mémoire tampon des éléments de télémétrie et la transmission de ces éléments au service Application Insights, où ils sont stockés pour être interrogés et analysés. Un canal de télémétrie est une classe qui implémente l’interface [`Microsoft.ApplicationInsights.ITelemetryChannel`](/dotnet/api/microsoft.applicationinsights.channel.itelemetrychannel?view=azure-dotnet).

La méthode `Send(ITelemetry item)` d’un canal de télémétrie est appelée une fois que tous les initialiseurs et processeurs de télémétrie ont été appelés. Par conséquent, tous les éléments supprimés par un processeur de télémétrie n’atteignent pas le canal. En règle générale, `Send()` n’envoie pas instantanément les éléments au back-end. Il les place généralement en mémoire tampon et les envoie par lots pour optimiser leur transmission.

Le [flux de métriques temps réel](live-stream.md) a également un canal personnalisé pour le streaming en direct des données de télémétrie. Le présent document ne s’applique pas à ce canal, qui est indépendant du canal de télémétrie standard.

## <a name="built-in-telemetry-channels"></a>Canaux de télémétrie intégrés

Les kits SDK .NET et .NET Core d’Application Insights intègrent deux canaux :

* `InMemoryChannel`: canal léger qui met en mémoire tampon les éléments en attendant qu’ils soient envoyés. Les éléments sont mis en mémoire tampon et vidés toutes les 30 secondes, ou dès que la mémoire tampon atteint 500 éléments. Ce canal offre des garanties de fiabilité minimales, car il ne retente pas l’envoi des éléments de télémétrie après un échec. Par ailleurs, comme il ne conserve pas les éléments sur le disque, tous les éléments non envoyés sont définitivement perdus à l’arrêt de l’application (arrêt approprié ou non). Il implémente une méthode `Flush()` qui permet de forcer le vidage synchrone des éléments de télémétrie de la mémoire. Ce canal convient bien pour les applications d’exécution de courte durée où un vidage synchrone est la meilleure option.

    Ce canal fait partie du package NuGet Microsoft.ApplicationInsights plus grand et constitue le canal par défaut utilisé par le SDK si aucune autre configuration n’a été spécifiée.

* `ServerTelemetryChannel`: canal plus avancé qui offre des stratégies de nouvelles tentatives et la capacité de stocker des données sur un disque local. Ce canal retente d’envoyer les données de télémétrie si des erreurs temporaires se produisent. Il utilise également le stockage sur disque local pour conserver les éléments sur le disque durant les pannes réseau ou la génération de gros volumes de données de télémétrie. Grâce aux stratégies de nouvelles tentatives et au stockage sur disque local, ce canal est considéré comme plus fiable et est recommandé pour tous les scénarios de production. Ce canal est utilisé par défaut pour les applications [ASP.NET](./asp-net.md) et [ASP.NET Core](./asp-net-core.md) qui sont configurées conformément à la documentation officielle. Il est optimisé pour les scénarios de serveur exécutant des processus de longue durée. La méthode [`Flush()`](#which-channel-should-i-use) implémentée par ce canal n’est pas synchrone.

    Ce canal est fourni dans le package NuGet Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel et est activé automatiquement quand vous utilisez le package NuGet Microsoft.ApplicationInsights.Web ou Microsoft.ApplicationInsights.AspNetCore.

## <a name="configure-a-telemetry-channel"></a>Configurer un canal de télémétrie

Vous configurez un canal de télémétrie en l’ajoutant à la configuration de télémétrie active. Pour les applications ASP.NET, la configuration implique de définir l’instance du canal de télémétrie sur `TelemetryConfiguration.Active`, ou de modifier `ApplicationInsights.config`. Pour les applications ASP.NET Core, la configuration implique d’ajouter le canal au conteneur d’injection de dépendance.

Les sections suivantes présentent des exemples de configuration du paramètre `StorageFolder` pour le canal dans différents types d’applications. `StorageFolder` est juste l’un des paramètres configurables. Pour obtenir la liste complète des paramètres de configuration, consultez la [section des paramètres](#configurable-settings-in-channels) plus loin dans cet article.

### <a name="configuration-by-using-applicationinsightsconfig-for-aspnet-applications"></a>Configuration avec ApplicationInsights.config pour les applications ASP.NET

L’extrait suivant du fichier [ApplicationInsights.config](configuration-with-applicationinsights-config.md) montre le canal `ServerTelemetryChannel` dont le paramètre `StorageFolder` est défini à un emplacement personnalisé :

```xml
    <TelemetrySinks>
        <Add Name="default">
            <TelemetryProcessors>
                <!-- Telemetry processors omitted for brevity  -->
            </TelemetryProcessors>
            <TelemetryChannel Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel, Microsoft.AI.ServerTelemetryChannel">
                <StorageFolder>d:\temp\applicationinsights</StorageFolder>
            </TelemetryChannel>
        </Add>
    </TelemetrySinks>
```

### <a name="configuration-in-code-for-aspnet-applications"></a>Configuration dans le code pour les applications ASP.NET

Le code suivant configure une instance « ServerTelemetryChannel » avec le paramètre `StorageFolder` défini à un emplacement personnalisé. Ajoutez ce code au début de l’application, généralement dans la méthode `Application_Start()` dans Global.aspx.cs.

```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
protected void Application_Start()
{
    var serverTelemetryChannel = new ServerTelemetryChannel();
    serverTelemetryChannel.StorageFolder = @"d:\temp\applicationinsights";
    serverTelemetryChannel.Initialize(TelemetryConfiguration.Active);
    TelemetryConfiguration.Active.TelemetryChannel = serverTelemetryChannel;
}
```

### <a name="configuration-in-code-for-aspnet-core-applications"></a>Configuration dans le code pour les applications ASP.NET Core

Modifiez la méthode `ConfigureServices` de la classe `Startup.cs` comme indiqué ici :

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

public void ConfigureServices(IServiceCollection services)
{
    // This sets up ServerTelemetryChannel with StorageFolder set to a custom location.
    services.AddSingleton(typeof(ITelemetryChannel), new ServerTelemetryChannel() {StorageFolder = @"d:\temp\applicationinsights" });

    services.AddApplicationInsightsTelemetry();
}

```

> [!IMPORTANT]
> La configuration du canal avec `TelemetryConfiguration.Active` n’est pas recommandée pour les applications ASP.NET Core.

### <a name="configuration-in-code-for-netnet-core-console-applications"></a>Configuration dans le code pour les applications console .NET/.NET Core

Pour les applications console, le code est identique pour .NET et .NET Core :

```csharp
var serverTelemetryChannel = new ServerTelemetryChannel();
serverTelemetryChannel.StorageFolder = @"d:\temp\applicationinsights";
serverTelemetryChannel.Initialize(TelemetryConfiguration.Active);
TelemetryConfiguration.Active.TelemetryChannel = serverTelemetryChannel;
```

## <a name="operational-details-of-servertelemetrychannel"></a>Détails du fonctionnement de ServerTelemetryChannel

`ServerTelemetryChannel` stocke les éléments entrants dans la mémoire tampon. Les éléments sont sérialisés, compressés et stockés dans une instance `Transmission` toutes les 30 secondes, ou dès que la mémoire tampon atteint 500 éléments. Une seule instance `Transmission` peut contenir jusqu’à 500 éléments de télémétrie, constituant un lot qui est envoyé au service Application Insights par le biais d’un appel HTTPS unique.

Par défaut, un maximum de 10 instances `Transmission` peuvent être envoyées en parallèle. Si les données de télémétrie arrivent à un rythme plus rapide, ou si le réseau ou le serveur back-end Application Insights est lent, les instances `Transmission` sont mises en mémoire tampon. La capacité par défaut de cette mémoire tampon `Transmission` est de 5 Mo. En cas de dépassement de la capacité mémoire, les instances `Transmission` sont stockées sur le disque local à hauteur de 50 Mo. Les instances `Transmission` sont également stockées sur le disque local quand des problèmes réseau se produisent. Seuls les éléments stockés sur un disque local sont conservés après un plantage de l’application. Ils sont envoyés au redémarrage de l’application.

## <a name="configurable-settings-in-channels"></a>Paramètres configurables dans les canaux

Pour obtenir la liste complète des paramètres configurables de chaque canal, consultez :

* [InMemoryChannel](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/BASE/src/Microsoft.ApplicationInsights/Channel/InMemoryChannel.cs)

* [ServerTelemetryChannel](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/BASE/src/ServerTelemetryChannel/ServerTelemetryChannel.cs)

Voici les paramètres les plus couramment utilisés pour `ServerTelemetryChannel` :

1. `MaxTransmissionBufferCapacity`: quantité de mémoire maximale, en octets, utilisée par le canal pour mettre en mémoire tampon les données avant leur transmission. Quand cette limite de capacité est atteinte, les nouveaux éléments sont stockés directement sur le disque local. La valeur par défaut est 5 Mo. Si vous définissez une valeur plus élevée pour réduire l’utilisation du disque, n’oubliez pas que les éléments en mémoire seront perdus en cas de plantage de l’application.

1. `MaxTransmissionSenderCapacity`: nombre maximal d’instances `Transmission` envoyées simultanément à Application Insights. La valeur par défaut est 10. Ce paramètre peut être défini à un nombre plus élevé, ce qui est d’ailleurs recommandé si un gros volume de données de télémétrie doit être généré. Cette situation se produit habituellement pendant les tests de charge ou quand l’échantillonnage est désactivé.

1. `StorageFolder`: dossier utilisé par le canal pour stocker les éléments sur le disque en fonction des besoins. Dans Windows, le dossier %LOCALAPPDATA% ou %TEMP% est utilisé si aucun autre chemin n’a été spécifié explicitement. Dans les autres environnements que Windows, vous devez spécifier un emplacement valide, car sinon les données de télémétrie ne seront pas stockées sur le disque local.

## <a name="which-channel-should-i-use"></a>Quel canal dois-je utiliser ?

Le canal `ServerTelemetryChannel` est conseillé dans la plupart des scénarios de production impliquant des applications d’exécution de longue durée. Outre que la méthode `Flush()` implémentée par `ServerTelemetryChannel` n’est pas synchrone, elle ne garantit pas l’envoi de tous les éléments qui sont en attente dans la mémoire ou sur le disque. Si vous utilisez ce canal dans des scénarios où l’application est sur le point de s’arrêter, nous vous conseillons d’introduire un délai après l’appel de `Flush()`. Le délai exact à prévoir n’est pas prévisible. Il dépend de différents facteurs tels que le nombre d’instances `Transmission` ou d’éléments mis en mémoire, stockés sur le disque et transmis au serveur back-end, et si le canal est situé au milieu de scénarios de back-off exponentiel.

Si vous devez effectuer un vidage synchrone, nous vous conseillons d’utiliser `InMemoryChannel`.

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="does-the-application-insights-channel-guarantee-telemetry-delivery-if-not-what-are-the-scenarios-in-which-telemetry-can-be-lost"></a>Le canal Application Insights garantit-il toujours la remise des données de télémétrie ? Si ce n’est pas le cas, quels sont les scénarios dans lesquels les données de télémétrie peuvent être perdues ?

En bref, la réponse est qu’aucun des canaux intégrés n’offre une garantie de type transactionnel de la remise des données de télémétrie au serveur back-end. `ServerTelemetryChannel` offre une meilleure fiabilité de remise par rapport à `InMemoryChannel`, mais il utilise aussi la méthode de la meilleure tentative possible pour envoyer les données de télémétrie. Le risque de perte de données de télémétrie demeure dans plusieurs situations, y compris dans ces scénarios courants :

1. Les éléments en mémoire sont perdus quand l’application plante.

1. Les données de télémétrie sont perdues si les problèmes de réseau rencontrés durent trop longtemps. Elles sont stockées sur le disque local durant les pannes réseau ou en cas de problèmes avec le serveur back-end d’Application Insights. Toutefois, les éléments datant de plus de 48 heures sont supprimés.

1. Dans Windows, les emplacements par défaut sur le disque où sont stockées les données de télémétrie sont les dossiers %LOCALAPPDATA% ou %TEMP%. Ces emplacements se trouvent généralement sur la machine locale. Si l’application est migrée physiquement d’un emplacement à un autre, les données de télémétrie stockées à l’emplacement d’origine sont perdues.

1. Dans Web Apps sur Windows, l’emplacement de stockage sur disque par défaut est D:\local\LocalAppData. Cet emplacement n’est pas persistant. Il est supprimé après le redémarrage de l’application, sa mise à l’échelle ou autre opération de ce type, et toutes les données de télémétrie stockées à cet emplacement sont perdues. Vous pouvez remplacer la valeur par défaut et spécifier un emplacement de stockage persistant, comme D:\home. Notez toutefois que ces emplacements persistants sont gérés par le stockage distant et qu’ils peuvent donc être lents.

### <a name="does-servertelemetrychannel-work-on-systems-other-than-windows"></a>Le canal ServerTelemetryChannel fonctionne-t-il sur d’autres systèmes que Windows ?

Bien que le nom du package et de l’espace de noms de ce canal mentionne « WindowsServer », ce canal est pris en charge sur d’autres systèmes que Windows, avec l’exception suivante. Sur les autres systèmes, le canal ne crée pas de dossier de stockage local par défaut. Vous devez créer manuellement un dossier de stockage local et configurer le canal pour qu’il l’utilise. Une fois que le stockage local a été configuré, le canal fonctionne de la même façon sur tous les systèmes.

> [!NOTE]
> Grâce à la version 2.15.0-beta3 et celles ultérieures, un stockage local est désormais automatiquement créé pour Linux, Mac et Windows. Pour les systèmes non Windows, le Kit de développement logiciel (SDK) crée automatiquement un dossier de stockage local selon la logique suivante :
> - `${TMPDIR}` : Si la variable d’environnement `${TMPDIR}` est définie, cet emplacement est utilisé.
> - `/var/tmp` : Si l’emplacement précédent n’existe pas, nous essayons `/var/tmp`.
> - `/tmp` : Si les deux emplacements précédents n’existent pas, nous essayons `tmp`. 
> - Si aucun de ces emplacements n’existe, le stockage local n’est pas créé et la configuration manuelle est toujours requise. [En savoir plus sur l’implémentation](https://github.com/microsoft/ApplicationInsights-dotnet/pull/1860).

### <a name="does-the-sdk-create-temporary-local-storage-is-the-data-encrypted-at-storage"></a>Le kit de développement logiciel (SDK) crée-t-il un stockage local temporaire ? Les données sont-elles chiffrées dans le stockage ?

Le SDK stocke les données de télémétrie dans le stockage local en cas de problèmes de réseau ou de limitation. Ces données ne sont pas chiffrées localement.

Sur les systèmes Windows, le SDK crée automatiquement un dossier local temporaire dans le répertoire %TEMP% ou %LOCALAPPDATA%, auquel seuls les administrateurs et l’utilisateur actuel ont accès.

Sur les systèmes autres que Windows, aucun stockage local n’est créé automatiquement par le SDK et, par conséquent, aucune donnée n’est stockée localement par défaut.

> [!NOTE]
> Grâce à la version 2.15.0-beta3 et celles ultérieures, un stockage local est désormais automatiquement créé pour Linux, Mac et Windows. 

 Vous pouvez créer manuellement un répertoire de stockage et configurer le canal pour qu’il l’utilise. Dans ce cas, vous devez vous assurer que le répertoire est sécurisé.
Découvrez-en plus sur [la protection et la confidentialité des données](data-retention-privacy.md#does-the-sdk-create-temporary-local-storage).

## <a name="open-source-sdk"></a>Kit de développement logiciel (SDK) open source
Comme chaque SDK pour Application Insights, les canaux sont open source. Lisez et contribuez au code, ou signalez les problèmes rencontrés dans le [dépôt GitHub officiel](https://github.com/Microsoft/ApplicationInsights-dotnet).

## <a name="next-steps"></a>Étapes suivantes

* [Échantillonnage](./sampling.md)
* [Résolution des problèmes avec les SDK](./asp-net-troubleshoot-no-data.md)

