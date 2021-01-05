---
title: Forum aux questions sur Azure SignalR Service
description: Obtenez des réponses aux questions fréquemment posées sur Azure SignalR Service, notamment la résolution des problèmes et les scénarios d’utilisation classiques.
author: sffamily
ms.service: signalr
ms.topic: overview
ms.custom: devx-track-dotnet
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 910de9efbd132fb98a0c4bd596867800f65f5ad5
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150958"
---
# <a name="azure-signalr-service-faq"></a>FAQ sur Azure SignalR Service

## <a name="is-azure-signalr-service-ready-for-production-use"></a>Azure SignalR Service est-il prêt pour une utilisation en production ?

Oui, [ASP.NET Core SignalR](https://dotnet.microsoft.com/apps/aspnet/signalr) et [ASP.NET SignalR](/aspnet/signalr/overview/getting-started/introduction-to-signalr) sont tous deux en disponibilité générale.

## <a name="when-there-are-multiple-application-servers-are-client-messages-sent-to-all-servers-or-just-one-of-them"></a>Si plusieurs serveurs d’applications sont en place, les messages d’un client sont-ils envoyés à tous les serveurs ou à un seul d’entre eux ?

Un client et un serveur d’applications sont liés par un mappage un-à-un. Les messages d’un client sont donc toujours envoyés au même serveur d’applications.

Le mappage est conservé jusqu’à ce que le client ou le serveur d’applications se déconnecte.

## <a name="if-one-of-my-application-servers-is-down-how-can-i-find-it-and-get-notified"></a>Si l’un de mes serveurs d’applications est en panne, comment puis-je le trouver et recevoir une notification ?

Azure SignalR Service supervise les pulsations des serveurs d’applications.
Si aucune pulsation n’est reçue pendant une période spécifiée, le serveur d’applications est considéré comme étant hors connexion. Toutes les connexions clientes mappées à ce serveur d’applications sont alors déconnectées.

## <a name="why-does-my-custom-iuseridprovider-throw-an-exception-when-im-switching-from-aspnet-core-signalr-sdk-to-azure-signalr-service-sdk"></a>Pourquoi mon `IUserIdProvider` personnalisé lève une exception quand je passe du SDK ASP.NET Core SignalR au SDK Azure SignalR Service ?

Le paramètre `HubConnectionContext context` varie entre le SDK ASP.NET Core SignalR et le SDK Azure SignalR Service quand `IUserIdProvider` est appelé.

Dans ASP.NET Core SignalR, `HubConnectionContext context` est le contexte de la connexion cliente physique avec des valeurs valides pour toutes les propriétés.

Dans le SDK Azure SignalR Service, `HubConnectionContext context` est le contexte de la connexion cliente logique. Le client physique étant connecté à l’instance Azure SignalR Service, un nombre limité de propriétés sont fournies.

Pour l’instant, seules les propriétés `HubConnectionContext.GetHttpContext()` et `HubConnectionContext.User` sont accessibles.
Vous pouvez [examiner le code source](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/HubHost/ServiceHubConnectionContext.cs).

## <a name="can-i-configure-the-transports-available-in-azure-signalr-service-on-the-server-side-with-aspnet-core-signalr-for-example-can-i-disable-websocket-transport"></a>Puis-je configurer les transports disponibles dans Azure SignalR Service côté serveur avec ASP.NET Core SignalR ? Est-ce que je peux par exemple désactiver le transport WebSocket ?

Non.

Azure SignalR Service fournit les trois transports pris en charge par ASP.NET Core SignalR par défaut. Ce n’est pas configurable. Azure SignalR Service gère les connexions et les transports pour toutes les connexions clientes.

Vous pouvez configurer des transports côté client comme indiqué dans [Configuration d’ASP.NET Core SignalR](/aspnet/core/signalr/configuration#configure-allowed-transports-1).

## <a name="what-is-the-meaning-of-metrics-like-message-count-or-connection-count-shown-in-the-azure-portal-which-kind-of-aggregation-type-should-i-choose"></a>Que signifient les métriques, telles que le nombre de messages ou de connexions affiché dans le portail Azure ? Quel type d’agrégation dois-je choisir ?

Vous trouverez plus d’informations sur le calcul de ces métriques dans [Messages et connexions dans Azure SignalR Service](signalr-concept-messages-and-connections.md).

Dans le volet Vue d’ensemble des ressources Azure SignalR Service, nous avons déjà choisi le type d’agrégation adapté à vos besoins. Si vous accédez au volet Métriques, vous pouvez utiliser le type d’agrégation pour [Messages et connexions dans Azure SignalR Service](../azure-monitor/platform/metrics-supported.md#microsoftsignalrservicesignalr) en tant que référence.

## <a name="what-is-the-meaning-of-the-default-serverless-and-classic-service-modes-how-can-i-choose"></a>Quelle est la signification des modes de service `Default`, `Serverless` et `Classic` ? Comment faire mon choix ?

Pour les nouvelles applications, seuls les modes par défaut et serverless peuvent être utilisés. La principale différence est liée à la présence ou non de serveurs d’applications qui établissent des connexions serveur au service (vous utilisez alors `AddAzureSignalR()` pour vous connecter au service). En présence de tels serveurs, utilisez le mode par défaut ; sinon, choisissez le mode serverless.

Le mode classique vise à assurer la compatibilité descendante des applications existantes. Il ne doit donc pas être utilisé pour les nouvelles applications.

Pour plus d’informations sur le mode de service, consultez [cette documentation](concept-service-mode.md).

## <a name="can-i-send-message-from-client-in-serverless-mode"></a>Puis-je envoyer un message du client en mode serverless ?

Vous pouvez envoyer un message du client si vous définissez la configuration en amont dans votre instance SignalR. La configuration en amont est un ensemble de points de terminaison qui peuvent recevoir des messages et des événements de connexion du service SignalR. Si aucune configuration en amont n’est définie, les messages provenant du client sont ignorés.

Pour plus d’informations sur la configuration en amont, consultez [cette documentation](concept-upstream.md).

La configuration en amont (Upstream) est actuellement en préversion publique.

## <a name="are-there-any-feature-differences-in-using-azure-signalr-service-with-aspnet-signalr"></a>Existe-t-il des différences de fonctionnalités lors de l’utilisation d’Azure SignalR Service avec ASP.NET SignalR ?

Lorsque vous utilisez Azure SignalR Service, certaines API et fonctionnalités d’ASP.NET SignalR ne sont pas prises en charge :
- Il n’est pas possible de faire passer l’état arbitraire entre les clients et le hub (souvent appelé `HubState`).
- La classe `PersistentConnection` n’est pas prise en charge.
- Le *transport Forever Frame* n’est pas pris en charge.
- Azure SignalR Service ne relit plus les messages envoyés au client lorsque le client est hors connexion.
- Lorsque vous utilisez Azure SignalR Service, le trafic pour une connexion cliente est toujours routé (également appelé *rémanent*) vers une instance de serveur d’applications pendant la durée de la connexion.

Comme la prise en charge d’ASP.NET SignalR se concentre sur la compatibilité, les nouvelles fonctionnalités d’ASP.NET Core SignalR ne sont pas toutes prises en charge. Par exemple, *MessagePack* et *Streaming* sont disponibles uniquement pour les applications ASP.NET Core SignalR.

Vous pouvez configurer Azure SignalR Service pour différents modes de service : `Classic`, `Default` et `Serverless`. Le mode `Serverless` n’est pas pris en charge pour ASP.NET. Le plan de données API REST n’est pas non plus pris en charge.

## <a name="where-does-my-data-reside"></a>Où mes données résident-elles ?

Azure SignalR Service fonctionne comme un service de processeur de données. Il ne stocke pas de contenu client et la résidence des données est incluse par défaut. Si vous utilisez Azure SignalR Service avec d’autres services Azure, comme le Stockage Azure pour les diagnostics, consultez [ce livre blanc](https://azure.microsoft.com/resources/achieving-compliant-data-residency-and-security-with-azure/) pour obtenir des conseils sur la façon de maintenir la résidence des données dans les régions Azure.