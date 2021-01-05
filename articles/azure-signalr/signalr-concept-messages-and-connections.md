---
title: Messages et connexions dans Azure SignalR Service
description: Présentation des concepts clés concernant les messages et les connexions dans Azure SignalR Service.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 08/05/2020
ms.author: zhshang
ms.openlocfilehash: d71462bd30dbd4b377353c792a3c579e02949680
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151098"
---
# <a name="messages-and-connections-in-azure-signalr-service"></a>Messages et connexions dans Azure SignalR Service

Le modèle de facturation d’Azure SignalR Service est basé sur le nombre de connexions et le nombre de messages. Cet article explique de quelle manière les messages et les connexions sont définis et comptabilisés en vue de la facturation.


## <a name="message-formats"></a>Formats des messages 

Azure SignalR Service prend en charge les mêmes formats qu’ASP.NET Core SignalR : [JSON](https://www.json.org/) et [MessagePack](/aspnet/core/signalr/messagepackhubprotocol).

## <a name="message-size"></a>Taille des messages

Azure SignalR Service ne limite pas la taille des messages.

Les messages volumineux sont divisés en messages plus petits de 2 Ko maximum chacun et sont transmis séparément. Des SDK gèrent la division et l’assemblage des messages. Aucune implication des développeurs n’est nécessaire.

Les messages volumineux diminuent les performances de messagerie. Utilisez les messages les plus petits possibles et faites des tests afin de déterminer la taille de message optimale pour chaque scénario d’usage.

## <a name="how-messages-are-counted-for-billing"></a>Mode de calcul des messages pour la facturation

Pour la facturation, seuls les messages envoyés à partir d’Azure SignalR Service sont comptabilisés. Les messages Ping transmis entre les clients et les serveurs ne sont pas pris en compte.

Les messages de plus de 2 Ko sont comptabilisés comme plusieurs messages de 2 Ko chacun. Le graphique du nombre de messages dans le portail Azure est mis à jour tous les 100 messages par hub.

Par exemple, imaginez que vous avez un serveur d’applications et trois clients :

Le serveur d’applications diffuse un message de 1 ko à tous les clients connectés : le message du serveur d’applications au service est considéré comme un message entrant gratuit. Seuls les trois messages envoyés à partir du service à chaque client sont facturés en tant que messages sortants.

Le client A envoie un message de 1 ko à un autre client B, sans passer par le serveur d’applications. Le message du client A au service est un message entrant gratuit. Le message du service au client B est facturé comme message sortant.

Si vous avez trois clients et un serveur d’applications. Un client envoie un message de 4 Ko pour permettre au serveur de diffuser vers tous les clients. Il y a huit messages facturés comptabilisés : un message entre le service et le serveur d’applications, et trois messages entre le service et les clients. Chaque message est comptabilisé comme deux messages de 2 Ko chacun.

## <a name="how-connections-are-counted"></a>Mode de calcul des connexions

Il existe des connexions serveur et des connexions client avec Azure SignalR Service. Par défaut, chaque serveur d’applications démarre avec cinq connexions initiales par hub et chaque client dispose d’une connexion client.

Le nombre de connexions indiqué dans le portail Azure inclut les connexions serveur et les connexions clientes.

Par exemple, imaginez que vous utilisez deux serveurs d’applications et que vous définissez cinq hubs dans le code. Le nombre de connexions serveur sera de 50 : 2 serveurs d’applications * 5 hubs * 5 connexions par hub.

ASP.NET SignalR calcule le nombre de connexions serveur de manière différente. Il inclut un hub par défaut en plus des hubs que vous définissez. Par défaut, chaque serveur d’applications a besoin de cinq connexions serveur initiales supplémentaires. Le nombre de connexions initiales pour le hub par défaut reste cohérent avec les autres hubs.

Le service et le serveur d’applications continuent à synchroniser l’état de la connexion et à ajuster les connexions au serveur pour obtenir de meilleures performances et une meilleure stabilité du service.  Par conséquent, le numéro de connexion au serveur peut changer de temps en temps.

## <a name="how-inboundoutbound-traffic-is-counted"></a>Mode de calcul du trafic entrant/sortant

Le message envoyé au service est un message entrant. Le message envoyé à partir du service est un message sortant. Le trafic est calculé en octets.

## <a name="related-resources"></a>Ressources associées

- [Types d’agrégation dans Azure Monitor](../azure-monitor/platform/metrics-supported.md#microsoftsignalrservicesignalr )
- [Configuration d’ASP.NET Core SignalR](/aspnet/core/signalr/configuration)
- [JSON](https://www.json.org/)
- [MessagePack](/aspnet/core/signalr/messagepackhubprotocol)