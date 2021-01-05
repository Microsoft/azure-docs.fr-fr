---
title: Mode de service dans Azure SignalR Service
description: Une vue d’ensemble des différents modes de service dans Azure SignalR Service qui explique leurs différences et les scénarios utilisateur applicables.
author: chenkennt
ms.service: signalr
ms.topic: conceptual
ms.date: 08/19/2020
ms.author: kenchen
ms.openlocfilehash: 60f1ab0440120cb9a96e6c05a4fc1987ead29188
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143256"
---
# <a name="service-mode-in-azure-signalr-service"></a>Mode de service dans Azure SignalR Service

Le mode de service est un concept important dans Azure SignalR Service. Lorsque vous créez une ressource SignalR, vous êtes invité à spécifier un mode de service :

:::image type="content" source="media/concept-service-mode/create.png" alt-text="Choisir le mode de service au moment de la création":::

Vous pouvez également le modifier ultérieurement dans le menu des paramètres :

:::image type="content" source="media/concept-service-mode/update.png" alt-text="Choisir le mode de service au moment de la création":::

Actuellement, Azure SignalR Service prend en charge trois modes de service : **par défaut**, **serverless** et **classique**. Votre ressource SignalR se comportera différemment selon le mode. Dans cet article, vous allez découvrir ces différences et la façon de choisir le mode de service approprié en fonction de votre scénario.

## <a name="default-mode"></a>Mode par défaut

Le mode par défaut est la valeur par défaut pour le mode de service lorsque vous créez une ressource SignalR. Dans ce mode, votre application fonctionne comme une application SignalR ASP.NET Core (ou ASP.NET) typique, où vous disposez d’un serveur web qui héberge un hub (appelé « serveur hub » ci-dessous) et où les clients peuvent avoir une communication duplex en temps réel avec le serveur hub. La seule différence est qu’au lieu de se connecter directement, le client et le serveur se connectent tous deux au service SignalR et utilisent le service comme un proxy. Vous trouverez ci-dessous un diagramme illustrant la structure d’application typique en mode par défaut :

:::image type="content" source="media/concept-service-mode/default.png" alt-text="Choisir le mode de service au moment de la création":::

Par conséquent, si vous avez une application SignalR et que vous souhaitez l’intégrer au service SignalR, le mode par défaut devrait être le bon choix dans la plupart des cas.

### <a name="connection-routing-in-default-mode"></a>Routage de connexion en mode par défaut

En mode par défaut, il y aura des connexions WebSocket entre le serveur hub et le service SignalR (appelées connexions serveur). Ces connexions sont utilisées pour transférer des messages entre le serveur et le client. Lorsqu’un nouveau client est connecté, le service SignalR achemine le client vers un serveur hub (en supposant que vous ayez plus d’un serveur) via des connexions serveur existantes. Ensuite, la connexion cliente est maintenue sur le même serveur hub pendant toute sa durée de vie. Lorsque le client envoie des messages, il accède toujours au même serveur hub. Grâce à ce comportement, vous pouvez gérer en toute sécurité certains états pour les connexions individuelles sur votre serveur hub. Par exemple, si vous souhaitez diffuser un contenu entre le serveur et le client, vous n’avez pas besoin de considérer le cas où les paquets de données sont dirigés vers des serveurs différents.

> [!IMPORTANT]
> Cela signifie également qu’en mode par défaut, le client ne peut pas se connecter sans que le serveur ne soit connecté au préalable. Si tous vos serveurs hubs sont déconnectés en raison d’une interruption du réseau ou du redémarrage du serveur, la connexion de votre client recevra un message d’erreur vous informant qu’aucun serveur n’est connecté. C’est pourquoi il est de votre responsabilité de vous assurer à tout moment qu’au moins un serveur hub est connecté au service SignalR (par exemple, en disposant de plusieurs serveurs hubs et en vous assurant qu’ils ne se déconnecteront pas en même temps pour des opérations telles que la maintenance).

Ce modèle de routage signifie également que, lors de la mise hors connexion d’un serveur hub, les connexions acheminées vers ce serveur seront interrompues. Par conséquent, vous devez vous attendre à une perte de connexion lorsque votre serveur hub est hors connexion pour des raisons de maintenance et gérer correctement la reconnexion afin que cela ne nuise pas à votre application.

## <a name="serverless-mode"></a>Mode serverless

Le mode serverless, comme son nom l’indique, est un mode dans lequel vous ne pouvez pas avoir de serveur hub. Par rapport au mode par défaut, dans ce mode, le client n’a pas besoin d’un serveur hub pour se connecter. Toutes les connexions sont connectées au service en mode « serverless » et le service est chargé de maintenir les connexions des clients, par exemple la gestion des tests Ping des clients (en mode par défaut, cela est géré par les serveurs hubs).

En outre, il n’existe pas de connexion au serveur dans ce mode : si vous essayez d’utiliser le Kit de développement logiciel (SDK) du service pour établir une connexion au serveur, vous obtiendrez une erreur. Par conséquent, il n’y a pas non plus de routage de connexion ni d’adhérence entre le client et le serveur (comme décrit dans la section relative au mode par défaut). Toutefois, vous pouvez toujours avoir une application côté serveur pour envoyer (push) des messages aux clients. Pour ce faire, vous pouvez utiliser des [API REST](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md) pour un envoi unique ou une connexion WebSocket afin de pouvoir envoyer plus efficacement plusieurs messages. (Notez que cette connexion WebSocket est différente de la connexion au serveur.)

> [!NOTE]
> Ces deux méthodes, par API REST et par WebSocket, sont prises en charge par le [Kit de développement logiciel (SDK) de gestion](https://github.com/Azure/azure-signalr/blob/dev/docs/management-sdk-guide.md) du service SignalR. Si vous utilisez un langage autre que .NET, vous pouvez également appeler manuellement les API REST en suivant ce [spec](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md).
>
> Si vous utilisez Azure Functions, vous pouvez utiliser des [liaisons de service SignalR pour Azure Functions](../azure-functions/functions-bindings-signalr-service.md) (ci-après appelées liaison de fonction) pour envoyer des messages en tant que liaison de sortie.

Il est également possible pour votre application serveur de recevoir des messages et des événements de connexion des clients. Le service remet les messages et les événements de connexion à des points de terminaison préconfigurés (appelés amont) à l’aide de webhooks. Par rapport au mode par défaut, il n’y a aucune garantie d’adhérence et les requêtes HTTP peuvent être moins efficaces que les connexions WebSocket.

Pour plus d’informations sur la configuration en amont, consultez ce [document](./concept-upstream.md).

Vous trouverez ci-dessous un diagramme illustrant le fonctionnement du mode serverless :

:::image type="content" source="media/concept-service-mode/serverless.png" alt-text="Choisir le mode de service au moment de la création":::

> [!NOTE]
> Notez que dans le mode par défaut, vous pouvez également utiliser l’API REST, le Kit de développement logiciel (SDK) de gestion ou la liaison de fonction pour envoyer directement des messages au client si vous ne souhaitez pas passer par le serveur hub. Toutefois, en mode par défaut, les connexions clientes sont toujours gérées par les serveurs hubs et l’amont ne fonctionne pas dans ce mode.

## <a name="classic-mode"></a>Mode classique

Le mode classique est un mixte entre les modes par défaut et serverless. Dans ce mode, le mode de connexion est déterminé par la présence ou non d’un serveur hub connecté lorsque la connexion cliente est établie. S’il y a un serveur hub, la connexion cliente est acheminée vers le serveur hub. Dans le cas contraire, la connexion se fera en mode serverless où le message client à serveur ne peut pas être remis au serveur hub. Cela entraînera certaines divergences. Par exemple, si tous les serveurs hubs sont indisponibles pendant une brève période, toutes les connexions clientes créées pendant cette période seront en mode serverless et ne pourront pas envoyer de messages au serveur hub.

> [!NOTE]
> Le mode classique est principalement destiné à la compatibilité descendante des applications créées avant les modes par défaut et serverless. Il est fortement recommandé de ne pas utiliser ce mode. Pour les nouvelles applications, choisissez le mode par défaut ou le mode serverless en fonction de votre scénario. Pour les applications existantes, il est également recommandé de passer en revue vos cas d’usage et de choisir le mode de service approprié.

Le mode classique ne prend pas non plus en charge certaines nouvelles fonctionnalités telles que l’amont en mode serverless.

## <a name="choose-the-right-service-mode"></a>Choisir le mode de service approprié

À présent, vous devriez comprendre les différences entre les modes de service et savoir comment choisir entre eux. Comme vous l’avez déjà appris dans la section précédente, le mode classique n’est pas recommandé et vous devez uniquement choisir entre le mode par défaut et le mode serverless. Voici d’autres conseils qui peuvent vous aider à faire le bon choix pour les nouvelles applications et à mettre hors service le mode classique pour les applications existantes.

* Si vous êtes déjà familiarisé avec le fonctionnement de la bibliothèque SignalR et que vous souhaitez passer d’un service SignalR auto-hébergé à l’utilisation d’Azure SignalR Service, choisissez le mode par défaut. Le mode par défaut fonctionne exactement de la même façon que le service SignalR auto-hébergé (et vous pouvez utiliser le même modèle de programmation dans la bibliothèque SignalR) : le service SignalR agit simplement comme un proxy entre les clients et les serveurs hubs.

* Si vous créez une application et que vous ne souhaitez pas conserver de serveur hub ni de connexions au serveur, choisissez le mode serverless. Ce mode fonctionne généralement avec Azure Functions, vous n’avez donc pas besoin de gérer un serveur. Vous pouvez toujours avoir des communications duplex (avec l’API REST, le SDK de gestion ou la liaison de fonction + amont), mais le modèle de programmation sera différent de la bibliothèque SignalR.

* Si vous avez des serveurs hubs pour servir des connexions clientes et une application principale pour envoyer (push) directement des messages aux clients (par exemple, via l’API REST), vous devez toujours choisir le mode par défaut. Gardez à l’esprit que la principale différence entre le mode par défaut et le mode serverless est le fait d’avoir ou non des serveurs hubs et la façon dont les connexions clientes sont acheminées. L’API REST, le SDK de gestion et la liaison de fonction peuvent être utilisés dans les deux modes.

* Si vous avez un scénario mixte, par exemple, si vous avez deux hubs différents sur la même ressource SignalR, l’un utilisé comme hub SignalR traditionnel et l’autre utilisé avec Azure Functions et n’ayant pas de serveur hub, nous vous recommandons fortement de les séparer en deux ressources SignalR, l’un en mode par défaut et l’autre en mode serverless.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’utilisation du mode par défaut et du mode serverless, lisez les articles suivants :

* [Éléments internes Azure SignalR Service](signalr-concept-internals.md)

* [Développement et configuration Azure Functions avec Azure SignalR Service](signalr-concept-serverless-development-config.md)