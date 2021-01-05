---
title: Éléments internes Azure SignalR Service
description: Découvrez les éléments internes Azure SignalR Service, l’architecture, les connexions et la façon dont les données sont transmises.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.custom: devx-track-dotnet
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 3fc6971c66d06ae9f25584f5be28b051075bfa49
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88921968"
---
# <a name="azure-signalr-service-internals"></a>Éléments internes Azure SignalR Service

Azure SignalR Service repose sur le framework ASP.NET Core SignalR. Il prend également en charge ASP.NET SignalR en réimplémentant le protocole de données d’ASP.NET SignalR sur l’infrastructure ASP.NET Core.

Vous pouvez facilement migrer une application ASP.NET Core SignalR locale ou une application ASP.NET SignalR pour qu’elle fonctionne avec SignalR Service en changeant simplement quelques lignes de code.

Le diagramme ci-dessous décrit l’architecture type quand vous utilisez SignalR Service avec votre serveur d’applications.

Les différences par rapport à l’application ASP.NET Core SignalR auto-hébergée sont également abordées.

![Architecture](./media/signalr-concept-internals/arch.png)

## <a name="server-connections"></a>Connexions serveur

Le serveur d’applications ASP.NET Core SignalR auto-hébergé écoute et connecte les clients directement.

Avec SignalR Service, le serveur d’applications n’accepte plus de connexions clientes persistantes. Au lieu de cela :

1. Un point de terminaison `negotiate` est exposé par le SDK Azure SignalR Service pour chaque hub.
1. Ce point de terminaison répond aux requêtes de négociation du client et redirige les clients vers SignalR Service.
1. Au final, les clients sont connectés à SignalR Service.

Pour plus d’informations, consultez [Connexions clientes](#client-connections).

Une fois le serveur d’applications démarré : 
- Pour ASP.NET Core SignalR, le SDK Azure SignalR Service ouvre 5 connexions WebSocket par hub à SignalR Service. 
- Pour ASP.NET SignalR, le SDK Azure SignalR Service ouvre 5 connexions WebSocket par hub à SignalR Service et une connexion WebSocket par application.

Le nombre par défaut de connexions WebSocket que vous pouvez changer dans la [configuration](https://github.com/Azure/azure-signalr/blob/dev/docs/use-signalr-service.md#connectioncount) est de 5.

Les messages en provenance et à destination des clients sont multiplexés dans ces connexions.

Ces connexions sont connectées en permanence à SignalR Service. Si une connexion serveur est déconnectée en raison d’un problème réseau :
- Tous les clients desservis par cette connexion serveur sont déconnectés (pour plus d’informations à ce sujet, consultez [Transmission de données entre le client et le serveur](#data-transmit-between-client-and-server)).
- La connexion serveur commence à se reconnecter automatiquement.

## <a name="client-connections"></a>Connexions clientes

Quand vous utilisez SignalR Service, les clients se connectent à SignalR Service et non au serveur d’applications.
L’établissement de connexions persistantes entre le client et SignalR Service passe par deux étapes.

1. Le client envoie une demande de négociation au serveur d’applications. Avec le SDK Azure SignalR Service, le serveur d’applications retourne une réponse de redirection avec l’URL et le jeton d’accès de SignalR Service.

- Pour ASP.NET Core SignalR, une réponse de redirection standard ressemble à ceci :
    ```
    {
        "url":"https://test.service.signalr.net/client/?hub=chat&...",
        "accessToken":"<a typical JWT token>"
    }
    ```
- Pour ASP.NET SignalR, une réponse de redirection standard ressemble à ceci :
    ```
    {
        "ProtocolVersion":"2.0",
        "RedirectUrl":"https://test.service.signalr.net/aspnetclient",
        "AccessToken":"<a typical JWT token>"
    }
    ```

1. Après avoir reçu la réponse de redirection, le client utilise la nouvelle URL et le nouveau jeton d’accès pour démarrer le processus normal de connexion à SignalR Service.

Découvrez-en plus sur les [protocoles de transport](https://github.com/aspnet/SignalR/blob/release/2.2/specs/TransportProtocols.md) d’ASP.NET Core SignalR.

## <a name="data-transmit-between-client-and-server"></a>Transmission de données entre le client et le serveur

Quand un client est connecté à SignalR Service, le runtime du service trouve une connexion serveur pour desservir ce client.
- Cette étape ne se produit qu’une seule fois et constitue un mappage un-à-un entre les connexions client et serveur.
- Le mappage est conservé dans SignalR Service jusqu’à ce que le client ou le serveur se déconnecte.

À ce stade, le serveur d’applications reçoit un événement avec des informations du nouveau client. Une connexion logique au client est créée dans le serveur d’applications. Le canal de données est établi du client au serveur d’applications par le biais de SignalR Service.

SignalR Service transmet les données du client au serveur d’applications d’appairage. Par ailleurs, les données du serveur d’applications sont envoyées aux clients mappés.

SignalR Service n’enregistre ni ne stocke les données client ; toutes les données client reçues sont transmises au serveur cible ou aux clients en temps réel.

Comme vous pouvez le voir, Azure SignalR Service est essentiellement une couche de transport logique entre les clients et le serveur d’applications. Toutes les connexions persistantes sont déchargées sur SignalR Service.
Le serveur d’applications doit uniquement gérer la logique métier dans la classe hub, sans se soucier des connexions clientes.