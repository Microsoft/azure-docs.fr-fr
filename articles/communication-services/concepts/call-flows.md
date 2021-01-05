---
title: Flux d’appels dans Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Découvrez les flux d’appels dans Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 9fe5cb13ee352b2c49ab6ae57cabd6116cdfa720
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91667671"
---
# <a name="call-flows"></a>Flux d’appels

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

La section ci-dessous offre une vue d’ensemble des flux d’appels dans Azure Communication Services. Les flux multimédias et de signalisation dépendent des types d’appels que vos utilisateurs effectuent. Des exemples de types d’appels incluent VoIP un-à-un, RTPC un-à-un et les appels de groupe combinant des participants connectés par VoIP et RTPC. Passez en revue [Types d’appels](./voice-video-calling/about-call-types.md).

## <a name="about-signaling-and-media-protocols"></a>À propos des protocoles de signalisation et multimédias

Lorsque vous établissez un appel d’égal à égal ou de groupe, deux protocoles sont utilisés en arrière-plan : HTTP (REST) pour la signalisation et SRTP pour les flux multimédias. 

La signalisation entre bibliothèques de client ou entre les bibliothèques de client et les contrôleurs de signalisation des services de communication est gérée avec HTTP REST (TLS). Pour le trafic multimédia en temps réel (RTP), le protocole UDP (User Datagram Protocol) est préféré. Si l’utilisation du protocole UDP est bloquée par votre pare-feu, la bibliothèque de client utilise le protocole TCP (Transmission Control Protocol) pour le contenu multimédia. 

Passons en revue les protocoles de signalisation et multimédias dans différents scénarios. 

## <a name="call-flow-cases"></a>Cas de flux d’appels

### <a name="case-1-voip-where-a-direct-connection-between-two-devices-is-possible"></a>Cas n° 1 : VoIP où une connexion directe entre deux appareils est possible

Dans des appels VoIP un-à-un ou des appels vidéo, le trafic préfère le chemin le plus direct. Le « chemin direct » signifie que si deux bibliothèques de client peuvent s’atteindre mutuellement directement, elles établissent une connexion directe. Cela est généralement possible lorsque deux bibliothèques de client se trouvent dans le même sous-réseau (par exemple, dans un sous-réseau 192.168.1.0/24) ou dans deux sous-réseaux lorsque les appareils résident dans des sous-réseaux qui peuvent se voir mutuellement (les bibliothèques de client dans le sous-réseau 10.10.0.0/16 et le sous-réseau 192.168.1.0/24 peuvent s’atteindre mutuellement).

:::image type="content" source="./media/call-flows/about-voice-case-1.png" alt-text="Schéma montrant un appel VoIP direct entre les utilisateurs et Communication Services.":::

### <a name="case-2-voip-where-a-direct-connection-between-devices-is-not-possible-but-where-connection-between-nat-devices-is-possible"></a>Cas n° 2 : VoIP où une connexion directe entre les appareils n’est pas possible, mais où la connexion entre les périphériques NAT est possible

Si deux appareils se trouvent dans des sous-réseaux qui ne peuvent pas s’atteindre mutuellement (par exemple, Alice travaille dans un café et Bob travaille dans son bureau à domicile), mais que la connexion entre les périphériques NAT est possible, les bibliothèques de client côté client établissent la connexion via les périphériques NAT. 

Pour Alice, il s’agit du traducteur d’adresses réseau (NAT) du café et, pour Bob, du traducteur d’adresses réseau de son bureau à domicile. L’appareil d’Alice envoie l’adresse externe de son traducteur d’adresses réseau et Bob fait de même. Les bibliothèques de client apprennent ces adresses externes à partir d’un service STUN (Session Traversal Utilities for NAT) qu’Azure Communication Services fournit gratuitement. La logique qui gère la négociation entre Alice et Bob est incorporée dans les bibliothèques de client fournies par Azure Communication Services. (Vous n’avez pas besoin d’une configuration supplémentaire)

:::image type="content" source="./media/call-flows/about-voice-case-2.png" alt-text="Schéma montrant un appel VoIP direct entre les utilisateurs et Communication Services.":::

### <a name="case-3-voip-where-neither-a-direct-nor-nat-connection-is-possible"></a>Cas n° 3 : VoIP où aucune connexion directe ni NAT n’est possible

Si un appareil client ou les deux se trouvent derrière un traducteur d’adresses réseau symétrique, un service cloud distinct servant à relayer le contenu multimédia entre les deux bibliothèques de client est requis. Ce service est appelé TURN (Traversal Using Relays around NAT) et est également fourni par Communication Services. La bibliothèque de client Communication Services Calling utilise automatiquement les services TURN en fonction des conditions de réseau détectées. L’utilisation du service TURN de Microsoft fait l’objet d’une facturation distincte.

:::image type="content" source="./media/call-flows/about-voice-case-3.png" alt-text="Schéma montrant un appel VoIP direct entre les utilisateurs et Communication Services.":::
 
### <a name="case-4-group-calls-with-pstn"></a>Cas 4 : Appels de groupe avec RTPC

Dans le cadre des appels RTPC, la signalisation et le contenu multimédia utilisent tous les deux la ressource de téléphonie d’Azure Communication Services. Cette ressource est interconnectée avec d’autres opérateurs.

Le trafic multimédia RTPC circule via un composant appelé processeur multimédia.

:::image type="content" source="./media/call-flows/about-voice-pstn.png" alt-text="Schéma montrant un appel VoIP direct entre les utilisateurs et Communication Services.":::

> [!NOTE]
> Pour ceux qui connaissent bien le traitement de contenu multimédia, notre processeur multimédia est également un agent utilisateur dos à dos, tel que défini dans la [RFC 3261 Protocole SIP : Protocole d’initiation de session](https://tools.ietf.org/html/rfc3261), ce qui signifie qu’il peut traduire les codecs lors de la gestion des appels entre les réseaux Microsoft et de l’opérateur. Le contrôleur de signalisation d’Azure Communication Services est l’implémentation par Microsoft d’un proxy SIP conformément à la même RFC.

Pour les appels de groupe, le contenu multimédia et la signalisation sont toujours transmis via le backend d’Azure Communication Services. Le contenu audio et/ou vidéo provenant des participants est mélangé dans le composant de processeur multimédia. Tous les membres d’un appel de groupe envoient leurs flux audio et/ou vidéo au processeur multimédia, qui renvoie des flux multimédias mixtes.

Le protocole RTP (Real-Time Protocol) par défaut pour les appels de groupe est le protocole UDP (User Datagram Protocol).

> [!NOTE]
> Le processeur multimédia peut agir comme une unité de contrôle multipoint (MCU) ou une unité de transfert sélectif (SFU)

:::image type="content" source="./media/call-flows/about-voice-group-calls.png" alt-text="Schéma montrant un appel VoIP direct entre les utilisateurs et Communication Services.":::

Si la bibliothèque de client ne peut pas utiliser le protocole UDP pour le contenu multimédia en raison des restrictions du pare-feu, une tentative d’utilisation du protocole TCP (Transmission Control Protocol) est effectuée. Notez que le composant de processeur multimédia nécessite le protocole UDP. Ainsi, lorsque cela se produit, le service TURN de Communication Services est ajouté à l’appel de groupe pour traduire le protocole TCP en protocole UDP. Des frais liés à TURN sont facturés dans ce cas, sauf si les fonctionnalités TURN sont désactivées manuellement.

:::image type="content" source="./media/call-flows/about-voice-group-calls-2.png" alt-text="Schéma montrant un appel VoIP direct entre les utilisateurs et Communication Services.":::

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Prise en main des appels](../quickstarts/voice-video-calling/getting-started-with-calling.md)

Les documents suivants peuvent vous intéresser :

- En savoir plus sur les [types d’appels](../concepts/voice-video-calling/about-call-types.md)
- En savoir plus sur l’[architecture client-serveur](./client-and-server-architecture.md)
