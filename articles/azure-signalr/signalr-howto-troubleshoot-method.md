---
title: Pratique de dépannage pour Azure SignalR Service
description: Découvrez comment résoudre les problèmes de connectivité et de remise des messages
author: YanJin
ms.service: signalr
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: yajin1
ms.openlocfilehash: 413bb88deac96c1ca12e8a9d25fc9cd16edf4616
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183955"
---
# <a name="how-to-troubleshoot-connectivity-and-message-delivery-issues"></a>Comment résoudre les problèmes de connectivité et de remise de messages

Ce guide présente plusieurs méthodes pour faciliter l’auto-diagnostic afin de trouver la cause racine directement ou de réduire le problème. Le résultat de l’auto-diagnostic est également utile lorsqu’il nous est communiqué pour un examen plus approfondi.

Tout d’abord, vous devez vérifier à partir du portail Azure sur quel [ServiceMode](./concept-service-mode.md) Azure SignalR Service (également appelé **ASRS**) est configuré.

:::image type="content" source="./media/signalr-howto-troubleshoot-method/service-mode.png" alt-text="ServiceMode":::

* Pour le mode `Default`, consultez [Résolution des problèmes en mode par défaut](#default_mode_tsg).

* Pour le mode `Serverless`, consultez [Résolution des problèmes en mode serverless](#serverless_mode_tsg).

* Pour le mode `Classic`, consultez [Résolution des problèmes en mode classique](#classic_mode_tsg).

<a name="default_mode_tsg"></a>

## <a name="default-mode-troubleshooting"></a>Résolution des problèmes en mode par défaut

Lorsqu’**ASRS** est en mode *par défaut*, il existe **trois rôles** : *Client*, *Serveur* et *Service* :

* *Client* : *Client* désigne les clients connectés à **ASRS**. Les connexions persistantes entre le client et **ASRS** sont appelées *Connexions clientes* dans ce guide.

* *Serveur* : *Serveur* désigne le serveur qui sert à la négociation du client et héberge la logique `Hub` de SignalR. Les connexions persistantes entre le *serveur* et **ASRS** sont appelées *Connexions serveur* dans ce guide.

* *Service* : *Service* est le nom abrégé d’**ASRS** dans ce guide.

Pour une présentation détaillée de l’ensemble de l’architecture et du workflow, reportez-vous à [Internals of Azure SignalR Service](https://github.com/Azure/azure-signalr/blob/dev/docs/internal.md).

Il existe plusieurs façons de vous aider à cerner le problème. 

* Si le problème surgit dès le début ou s’il est reproductible, la méthode la plus simple consiste à afficher le trafic en cours. 

* Si le problème est difficile à reproduire, les traces et les journaux peuvent vous aider.

### <a name="how-to-view-the-traffic-and-narrow-down-the-issue"></a>Comment afficher le trafic et cerner le problème

La capture du trafic en cours est la manière la plus simple de cerner le problème. Vous pouvez capturer les [traces du réseau](/aspnet/core/signalr/diagnostics#network-traces) à l’aide des options décrites ci-dessous :

* [Collecter une trace du réseau avec Fiddler](/aspnet/core/signalr/diagnostics#network-traces)

* [Collecter une trace du réseau avec tcpdump](/aspnet/core/signalr/diagnostics#collect-a-network-trace-with-tcpdump-macos-and-linux-only)

* [Collecter une trace du réseau dans le navigateur](/aspnet/core/signalr/diagnostics#collect-a-network-trace-in-the-browser)

<a name="view_traffic_client"></a>

#### <a name="client-requests"></a>Demandes client

Pour une connexion persistante SignalR, il faut d’abord négocier (`/negotiate`) avec votre serveur d’applications hébergé et la rediriger vers Azure SignalR Service, puis établir la connexion persistante réelle à Azure SignalR Service. Reportez-vous à [Internals of Azure SignalR Service](https://github.com/Azure/azure-signalr/blob/dev/docs/internal.md) pour connaître les étapes détaillées.

Avec la trace du réseau côté client en main, vérifiez quelle requête échoue avec quel code d’état et quelle réponse, et cherchez des solutions dans le [guide de résolution des problèmes](./signalr-howto-troubleshoot-guide.md).

#### <a name="server-requests"></a>Requêtes serveur

Le *serveur* SignalR maintient la *connexion serveur* entre le *serveur* et le *service*. Lorsque le serveur d’applications démarre, il établit la connexion **WebSocket** avec Azure SignalR Service. Tous les trafics client sont acheminés via Azure SignalR Service vers ces *connexions serveur*, puis distribués au `Hub`. Quand une *connexion serveur* est interrompue, les clients acheminés vers cette *connexion serveur* sont affectés. Notre Kit de développement logiciel (SDK) Azure SignalR suit une logique « Toujours réessayer » pour reconnecter la *connexion serveur* avec un délai d’une minute au maximum pour réduire l’impact.

Les *connexions serveur* peuvent être interrompues en raison de l’instabilité du réseau ou de la maintenance régulière d’Azure SignalR Service, ou encore de la mise à jour/maintenance de votre serveur d’applications hébergé. Tant que le mécanisme de déconnexion/reconnexion opère côté client, l’impact est minime, comme toute déconnexion/reconnexion intervenant côté client.

Consultez la trace du réseau côté serveur pour connaître le code d’état et les détails de l’erreur pour laquelle la *connexion serveur* est interrompue ou rejetée par le *service*, et recherchez la cause racine dans le [guide de résolution des problèmes](./signalr-howto-troubleshoot-guide.md).


### <a name="how-to-add-logs"></a>Comment ajouter des journaux

Les journaux peuvent être utiles pour diagnostiquer les problèmes et surveiller l’état d’exécution.

<a name="add_logs_client"></a>

#### <a name="how-to-enable-client-side-log"></a>Comment activer le journal côté client

L’expérience de journalisation côté client est exactement la même que lors de l’utilisation de SignalR auto-hébergé.

##### <a name="enable-client-side-logging-for-aspnet-core-signalr"></a>Activer la journalisation côté client pour `ASP.NET Core SignalR`

* [Journalisation du client JavaScript](/aspnet/core/signalr/diagnostics#javascript-client-logging)

* [Journalisation du client .NET](/aspnet/core/signalr/diagnostics#net-client-logging)


##### <a name="enable-client-side-logging-for-aspnet-signalr"></a>Activer la journalisation côté client pour `ASP.NET SignalR`

* [Client .NET](/aspnet/signalr/overview/testing-and-debugging/enabling-signalr-tracing#enabling-tracing-in-the-net-client-windows-desktop-apps)

* [Activation du traçage dans les clients Windows Phone 8](/aspnet/signalr/overview/testing-and-debugging/enabling-signalr-tracing#enabling-tracing-in-windows-phone-8-clients)

* [Activation du traçage dans le client JavaScript](/aspnet/signalr/overview/testing-and-debugging/enabling-signalr-tracing#enabling-tracing-in-the-javascript-client)

<a name="add_logs_server"></a>

#### <a name="how-to-enable-server-side-log"></a>Comment activer le journal côté serveur

##### <a name="enable-server-side-logging-for-aspnet-core-signalr"></a>Activer la journalisation côté serveur pour `ASP.NET Core SignalR`

La journalisation côté serveur pour `ASP.NET Core SignalR` s’intègre à la [journalisation](/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x&view=aspnetcore-2.1) basée sur `ILogger` fournie dans l’infrastructure `ASP.NET Core`. Vous pouvez activer la journalisation côté serveur à l’aide de `ConfigureLogging`, un exemple d’utilisation comme suit :

```cs
.ConfigureLogging((hostingContext, logging) =>
        {
            logging.AddConsole();
            logging.AddDebug();
        })
```

Les catégories d’enregistreur d’événements pour Azure SignalR commencent toujours par `Microsoft.Azure.SignalR`. Pour activer les journaux détaillés d’Azure SignalR, configurez les préfixes précédents sur le niveau `Information` dans votre fichier **appsettings.json** comme ci-dessous :

```JSON
{
    "Logging": {
        "LogLevel": {
            ...
            "Microsoft.Azure.SignalR": "Information",
            ...
        }
    }
}
```

Vérifiez si des journaux d’avertissements et d’erreurs anormaux sont enregistrés. 


##### <a name="enable-server-side-traces-for-aspnet-signalr"></a>Activer les traces côté serveur pour `ASP.NET SignalR`

En cas d’utilisation d’une version du Kit de développement logiciel (SDK) >= `1.0.0`, vous pouvez activer les traces en ajoutant le code suivant au fichier `web.config`: ([Détails](https://github.com/Azure/azure-signalr/issues/452#issuecomment-478858102))

```xml
<system.diagnostics>
    <sources>
      <source name="Microsoft.Azure.SignalR" switchName="SignalRSwitch">
        <listeners>
          <add name="ASRS" />
        </listeners>
      </source>
    </sources>
    <!-- Sets the trace verbosity level -->
    <switches>
      <add name="SignalRSwitch" value="Information" />
    </switches>
    <!-- Specifies the trace writer for output -->
    <sharedListeners>
      <add name="ASRS" type="System.Diagnostics.TextWriterTraceListener" initializeData="asrs.log.txt" />
    </sharedListeners>
    <trace autoflush="true" />
  </system.diagnostics>
```

Vérifiez si des journaux d’avertissements et d’erreurs anormaux sont enregistrés. 


#### <a name="how-to-enable-logs-inside-azure-signalr-service"></a>Comment activer les journaux dans Azure SignalR Service

Vous pouvez également [activer les journaux de diagnostic](./signalr-howto-diagnostic-logs.md) pour Azure SignalR Service. Ces journaux fournissent des informations détaillées sur chaque connexion à Azure SignalR Service.

<a name="serverless_mode_tsg"></a>

## <a name="serverless-mode-troubleshooting"></a>Résolution des problèmes en mode serverless

Quand **ASRS** est en mode *serverless*, seul **ASP.NET Core SignalR** prend en charge le mode `Serverless` ; **ASP.NET SignalR** ne prend **PAS** en charge ce mode.

Pour diagnostiquer les problèmes de connectivité en mode `Serverless`, la méthode la plus simple consiste à [consulter le trafic côté client](#view_traffic_client). Activer les [journaux côté client](#add_logs_client)et les [journaux côté service](#add_logs_server) peut également être utile.

<a name="classic_mode_tsg"></a>

## <a name="classic-mode-troubleshooting"></a>Résolution des problèmes en mode classique

Le mode `Classic` est obsolète et son utilisation est déconseillée. Dans ce mode, Azure SignalR Service utilise les *connexions serveur* établies pour déterminer si le service actuel est en mode `default` ou `serverless`. Cela peut entraîner des problèmes intermédiaires de connectivité client, car, en cas d’interruption soudaine de toutes les *connexions serveur*, par exemple en raison de l’instabilité du réseau, Azure SignalR considère avoir basculé en mode `serverless` et les clients connectés pendant cette période ne seront jamais acheminés vers le serveur d’applications hébergé. Activez les [journaux côté service](#add_logs_server), et vérifiez si des clients sont enregistrés en mode `ServerlessModeEntered` si vous avez un serveur d’applications hébergé, mais que certains clients ne l’atteignent jamais. S’il y en a, [abandonnez ces connexions clientes](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md#API) et laissez les clients redémarrer.

La résolution des problèmes de connectivité et de remise des messages en mode `classic` est similaire à la [résolution des problèmes en mode par défaut](#default_mode_tsg).

## <a name="service-health"></a>État d’intégrité du service

Vous pouvez vérifier l’API d’intégrité pour connaître l’intégrité du service.

* Demande : GET `https://{instance_name}.service.signalr.net/api/v1/health`

* Code d’état de la réponse :
  * 200 : Sain.
  * 503 : Votre service n’est pas sain. Vous pouvez :
    * Patienter quelques minutes pour la récupération automatique ;
    * Vérifier que l’adresse IP est identique à celle du portail : ou
    * Redémarrer l’instance.
    * Si aucune des options ci-dessus ne fonctionne, contactez-nous en ajoutant une nouvelle demande de support dans Portail Azure.

En savoir plus sur la [récupération d’urgence](./signalr-concept-disaster-recovery.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à résoudre les problèmes de connectivité et de remise des messages. Vous pouvez également apprendre à gérer les problèmes courants. 

> [!div class="nextstepaction"]
> [Guide de résolution des problèmes](./signalr-howto-troubleshoot-guide.md)