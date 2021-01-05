---
title: Réinitialisation TCP de l’équilibreur de charge et délai d’inactivité dans Azure
titleSuffix: Azure Load Balancer
description: Dans cet article, découvrez Azure Load Balancer avec des paquets TCP RST bidirectionnels pendant le délai d’inactivité.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/07/2020
ms.author: allensu
ms.openlocfilehash: 0d02b46345af13770f77a7dac452127a665e01fd
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696742"
---
# <a name="load-balancer-tcp-reset-and-idle-timeout"></a>Réinitialisation TCP de l’équilibreur de charge et délai d’inactivité

Vous pouvez utiliser [Standard Load Balancer](./load-balancer-overview.md) pour créer un comportement d’application plus prévisible pour vos scénarios en activant Réinitialisation TCP pendant le délai d'inactivité pour une règle donnée. Le comportement par défaut de Load Balancer consiste à supprimer silencieusement des flux lorsque le délai d’inactivité d’un flux est atteint.  Lorsque cette fonctionnalité est activée, Load Balancer envoie des réinitialisations TCP bidirectionnelles (paquet TCP RST) sur un délai d'inactivité.  Les points de terminaison de votre application sont ainsi informés que la connexion a expiré et n'est plus utilisable.  Les points de terminaison peuvent immédiatement établir une nouvelle connexion, si besoin.

![Réinitialisation TCP de Load Balancer](media/load-balancer-tcp-reset/load-balancer-tcp-reset.png)
 
## <a name="tcp-reset"></a>Réinitialisation du protocole TCP

Vous modifiez ce comportement par défaut et activez l’envoi des réinitialisations TCP pendant un délai d’inactivité, sur des règles NAT entrantes, des règles d’équilibrage de charge et des [règles de trafic sortant](./load-balancer-outbound-connections.md#outboundrules).  Lorsqu’il est activé par règle, Load Balancer envoie une réinitialisation TCP bidirectionnelle (paquets TCP RST) aux points de terminaison client et serveur pendant un délai d’inactivité de tous les flux correspondants.

Les points de terminaison recevant les paquets TCP RST ferment aussitôt le socket correspondant. Une notification immédiate est fournie aux points de terminaison et informe que la libération de la connexion s’est produite et que toute communication ultérieure sur cette même connexion TCP est vouée à l’échec.  Les applications peuvent purger les connexions lorsque le socket se ferme, et les rétablir en fonction des besoins sans attendre que la connexion TCP finisse par expirer.

Pour de nombreux scénarios, le besoin d’envoyer des conservations de connexion active TCP (ou de la couche Application) pour actualiser le délai d’inactivité d’un flux peut s’en trouver réduit. 

Si vos durées d’inactivité sont supérieures à celles autorisées par la configuration, ou si votre application présente un comportement indésirable lorsque les réinitialisations TCP sont activées, vous devrez peut-être continuer à utiliser des conservations de connexion active TCP (ou des conservations de connexion active de la couche Application) pour surveiller l’activité des connexions TCP.  De plus, les conservations de connexion active peuvent également se révéler utiles lorsque la connexion est transmise par proxy à un endroit quelconque sur le chemin, en particulier les conservations de connexion active de la couche Application.  

Examinez attentivement le scénario de bout en bout dans son intégralité pour décider si vous tirez parti de l’activation des réinitialisations TCP, de l’ajustement du délai d’inactivité, et si des étapes supplémentaires peuvent être nécessaires pour garantir le comportement de l’application souhaitée.

## <a name="configurable-tcp-idle-timeout"></a>Délai d’inactivité TCP configurable

Azure Load Balancer a la plage de délai d’inactivité suivante :
-  4 à 100 minutes pour les règles de trafic sortant
-  4 à 30 minutes pour les règles d’équilibrage de charge et les règles NAT de trafic entrant

Par défaut, il est défini sur 4 minutes. Si une période d’inactivité est supérieure à la valeur de délai d’expiration, il n’est pas garanti que la session TCP ou HTTP est maintenue entre le client et votre service cloud.

Lorsque la connexion est fermée, votre application cliente peut recevoir le message d’erreur suivant : « Le serveur a clos la connexion sous-jacente : une connexion qui devait être tenue active a été fermée par le serveur. »

Une pratique courante consiste à utiliser TCP keep-alive. Cela permet de maintenir la connexion active pendant une période plus longue. Pour plus d’informations, consultez ces [exemples .NET](/dotnet/api/system.net.servicepoint.settcpkeepalive). avec keep-alive activé, les paquets sont envoyés au cours des périodes d’inactivité sur la connexion. Les paquets keep-alive garantissent que la valeur de délai d’inactivité n’est pas atteinte et que la connexion est maintenue pendant une longue période.

Le paramètre fonctionne uniquement pour les connexions entrantes. Pour éviter la perte de la connexion, configurez TCP keep-alive sur un intervalle inférieur au paramètre de délai d’inactivité ou augmentez la valeur du délai d’inactivité. Pour prendre en charge ces scénarios, la prise en charge d’un délai d’inactivité configurable a été ajoutée.

TCP keep-alive convient aux scénarios où l’autonomie de la batterie n’est pas une contrainte. Il n’est pas recommandé de l’utiliser pour les applications mobiles. L’utilisation de TCP keep-alive depuis une application mobile peut décharger la batterie de l’appareil plus rapidement.


## <a name="limitations"></a>Limites

- Réinitialisation TCP envoyée uniquement au cours d’une connexion TCP dont l’état est ESTABLISHED.
- La réinitialisation TCP n’est pas envoyée pour les équilibreurs de charge internes avec des ports haute disponibilité configurés.
- Le délai d’inactivité TCP n’affecte pas les règles d’équilibrage de charge sur le protocole UDP.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Standard Load Balancer](./load-balancer-overview.md).
- En savoir plus sur les [règles de trafic sortant](./load-balancer-outbound-connections.md#outboundrules).
- [Configurer le délai d’inactivité TCP RST](load-balancer-tcp-idle-timeout.md)