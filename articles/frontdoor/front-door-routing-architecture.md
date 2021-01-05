---
title: Azure Front Door - Architecture de routage | Microsoft Docs
description: Cet article vous aide à comprendre l’architecture globale de Front Door.
services: front-door
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: duau
ms.openlocfilehash: 948cf3c65dfdc912f2f807dfac34076985f1bc89
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91449192"
---
# <a name="routing-architecture-overview"></a>Vue d’ensemble de l’architecture de routage

Quand Azure Front Door reçoit vos demandes client, il effectue l’une des deux opérations suivantes. Soit il y répond si vous activez la mise en cache, soit il les transmet au serveur principal d’application approprié en tant que proxy inverse.

## <a name="selecting-the-front-door-environment-for-traffic-routing-anycast"></a><a name = "anycast"></a>Sélection de l’environnement Front Door pour le routage du trafic (Anycast)

Le trafic acheminé vers les environnements Azure Front Door utilise [Anycast](https://en.wikipedia.org/wiki/Anycast) pour le trafic DNS (Domain Name System) et HTTP (Hypertext Transfer Protocol), ce qui permet aux demandes utilisateur d’atteindre l’environnement le plus proche en un minimum de tronçons réseau. Cette architecture offre des durées aller-retour plus brèves pour les utilisateurs finaux en optimisant ainsi les avantages du fractionnement TCP. Front Door organise ses environnements en « anneaux » principaux et de secours. L’anneau externe a des environnements qui sont plus proches des utilisateurs, offrant des latences réduites.  L’anneau interne a des environnements capables de gérer le basculement pour l’environnement d’anneau externe en cas de problèmes. L’anneau externe est la cible par défaut pour tout le trafic, et l’anneau interne doit gérer le dépassement de trafic de l’anneau externe. Chaque hôte ou domaine frontal desservi par Front Door reçoit une adresse IP virtuelle principale qui est annoncée par les environnements tant dans l’anneau interne que dans l’anneau externe. Une adresse IP virtuelle de secours n’est annoncée par les environnements que dans l’anneau interne. 

Cette architecture garantit que les demandes de vos utilisateurs finaux atteignent toujours l’environnement Front Door le plus proche. Même si l’environnement Front Door préféré est défectueux, tout le trafic passe automatiquement à l’environnement le plus proche suivant.

## <a name="connecting-to-front-door-environment-split-tcp"></a><a name = "splittcp"></a>Connexion à l’environnement Front Door (fractionnement TCP)

Le [TCP de fractionnement](https://en.wikipedia.org/wiki/Performance-enhancing_proxy) est une technique permettant de réduire les latences et les problèmes liés aux protocole TCP en fractionnant les connexions dont les durées de parcours circulaire seraient trop élevées. Avec des environnements Front Door plus proches des utilisateurs finaux, les connexions TCP se terminent à l’intérieur de l’environnement Front Door. Une connexion TCP qui a une durée aller-retour (RTT) longue au serveur principal de l’application est fractionnée en deux connexions distinctes. La « connexion courte » entre l’utilisateur final et l’environnement Front Door signifie que la connexion est établie sur trois allers-retours brefs plutôt que sur trois allers-retours longs, ce qui réduit la latence. La « connexion longue » entre l’environnement Front Door et le serveur principal peut être préétablie, puis réutilisée pour d’autres demandes d’utilisateurs finaux, ce qui réduit le durée de connexion. L’effet de fractionnement TCP est multiplié lors de l’établissement d’une connexion SSL/TLS (Transport Layer Security), car davantage d’allers-retours sont nécessaires pour sécuriser une connexion.

## <a name="processing-request-to-match-a-routing-rule"></a>Traitement de la requête pour la mise en correspondance avec une règle de routage
Après l’établissement d’une connexion et l’accomplissement d’une négociation TLS, la première étape après l’atterrissage d’une demande sur un environnement Front Door est de la faire correspondre à la règle de routage. La correspondance est déterminée par les configurations sur Front Door, définissant avec quelle règle de routage spécifique mettre la requête en correspondance. Pour en savoir plus sur la façon dont Front Door effectue la mise en correspondance d’itinéraire, consultez [cet article](front-door-route-matching.md).

## <a name="identifying-available-backends-in-the-backend-pool-for-the-routing-rule"></a>Identification des backends disponibles dans le pool de backends pour la règle de routage
Une fois que Front Door a établi une correspondance avec une règle de routage pour une demande entrante, l’étape suivante consiste à obtenir l’état de la sonde d’intégrité pour le pool principal associé à la règle de routage s’il n’y a pas de mise en cache. Pour en savoir plus sur la façon dont Front Door supervise l’intégrité des backends à l’aide de sondes d’intégrité, consultez [cet article](front-door-health-probes.md).

## <a name="forwarding-the-request-to-your-application-backend"></a>Transfert de la requête à votre backend d’application
Enfin, en supposant que la mise en cache n’est pas configurée, la demande utilisateur est transférée au « meilleur » serveur principal en fonction de la configuration de votre [méthode de routage](front-door-routing-methods.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
