---
title: Présentation de la gestion réseau Azure Service Fabric
description: Apprenez-en plus sur les réseaux, les passerelles et le routage de trafic intelligent dans Service Fabric Mesh.
author: georgewallace
ms.topic: conceptual
ms.date: 11/26/2018
ms.author: gwallace
ms.custom: mvc, devcenter
ms.openlocfilehash: 372e8a46bfe588aa22b0f3be0ef8ae442fa342e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842697"
---
# <a name="introduction-to-networking-in-service-fabric-mesh-applications"></a>Présentation de la gestion réseau dans les applications Service Fabric Mesh
Cet article décrit les différents types d’équilibreurs de charge, comment les passerelles connectent le réseau de vos applications à d’autres réseaux, et comment le trafic est acheminé entre les services dans vos applications.

## <a name="layer-4-vs-layer-7-load-balancers"></a>Équilibreurs de charge de couche 7 et de couche 4
L’équilibrage de charge peut être effectué au niveau de différentes couches dans le [modèle OSI](https://en.wikipedia.org/wiki/OSI_model) de mise en réseau, souvent dans la couche 4 (L4) et la couche 7 (L7).  En règle générale, il existe deux types d’équilibreurs de charge :

- Un équilibreur de charge L4 fonctionne au niveau de la couche de transport réseau, qui gère la remise de paquets indépendamment de leur contenu. Seuls les en-têtes de paquets sont inspectés par l’équilibreur de charge. Par conséquent, les critères d’équilibrage se limitent aux adresses IP et aux ports. Par exemple, un client établit une connexion TCP à l’équilibreur de charge. L’équilibreur de charge met fin à la connexion (en répondant directement au SYN), sélectionne un serveur principal et établit une nouvelle connexion TCP à ce dernier (envoie un nouveau SYN). Généralement, un équilibreur de charge L4 opère uniquement au niveau de la connexion TCP/UDP ou session de couche 4. Par conséquent, l’équilibreur de charge redirige les octets autour et permet de s’assurer que les octets de la même session se retrouvent sur le même serveur principal. L’équilibreur de charge L4 n’a pas connaissance des détails sur l’application des octets qu’il déplace. Les octets peuvent correspondre à n’importe quel protocole d’application.

- Un équilibreur de charge L7 fonctionne au niveau de la couche Application, qui gère le contenu de chaque paquet. Il inspecte le contenu du paquet, car il comprend les protocoles tels que HTTP, HTTPS ou WebSockets. Cela permet à l’équilibreur de charge d’effectuer un routage avancé. Par exemple, un client établit une connexion TCP HTTP/2 unique à l’équilibreur de charge. L’équilibreur de charge établit ensuite deux connexions au serveur principal. Lorsque le client envoie deux flux HTTP/2 à l’équilibreur de charge, le premier est envoyé au premier serveur principal et le second au deuxième serveur principal. Par conséquent, même les clients de multiplexage aux charges de requêtes très différentes seront équilibrés efficacement entre les serveurs principaux. 

## <a name="networks-and-gateways"></a>Réseaux et passerelles
Dans le [modèle de ressource Service Fabric](service-fabric-mesh-service-fabric-resources.md), une ressource réseau est une ressource déployable individuellement, indépendante de toute ressource d’application ou de service qui pourrait y faire référence en tant que dépendance. Elle permet de créer un réseau pour vos applications, ouvert à Internet. Plusieurs services de différentes applications peuvent faire partie d’un même réseau. Ce réseau privé est créé et géré par Service Fabric et n’est pas un réseau virtuel Azure. Les applications peuvent être ajoutées et supprimées dynamiquement de la ressource réseau pour activer et désactiver la connectivité de réseau virtuel. 

Une passerelle est utilisée pour relier deux réseaux. La ressource de passerelle déploie un [proxy Envoy](https://www.envoyproxy.io/) qui fournit un routage L4 pour tous les protocoles et un routage L7 pour le routage des applications HTTP(S) avancées. La passerelle achemine le trafic dans votre réseau à partir d’un réseau externe et détermine le service vers lequel acheminer le trafic.  Le réseau externe peut être un réseau ouvert (essentiellement l’Internet public) ou un réseau virtuel Azure, qui vous permet de vous connecter avec vos autres applications et ressources Azure. 

![Réseau et passerelle][Image1]

Lorsque la ressource réseau est créée avec `ingressConfig`, une adresse IP publique lui est affectée. L’adresse IP publique sera liée à la durée de vie de la ressource réseau.

Lorsqu’une application Mesh est créée, elle doit faire référence à une ressource réseau existante. De nouveaux ports publics peuvent être ajoutés ou des ports existants supprimés de la configuration d’entrée. La suppression d’une ressource réseau échoue si une ressource d’application y fait référence. Lorsque l’application est supprimée, la ressource réseau l’est également.

## <a name="next-steps"></a>Étapes suivantes 
Pour en savoir plus sur Service Fabric mesh, voir la
- [Vue d’ensemble de Service Fabric mesh](service-fabric-mesh-overview.md)

[Image1]: media/service-fabric-mesh-networks-and-gateways/NetworkAndGateway.png