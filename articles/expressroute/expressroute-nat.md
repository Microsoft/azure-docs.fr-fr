---
title: 'Azure ExpressRoute : Configuration NAT requise pour les circuits'
description: Cette page détaille les conditions nécessaires à la configuration et à la gestion des NAT pour les circuits ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/18/2019
ms.author: duau
ms.openlocfilehash: a2c322c765d39a3afe4974ed88bf4dc18fd467a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89569643"
---
# <a name="expressroute-nat-requirements"></a>Configuration NAT requise pour ExpressRoute
Pour vous connecter aux services de cloud Microsoft à l'aide d'ExpressRoute, vous devez configurer et gérer les NAT. Certains fournisseurs de connectivité proposent la configuration et la gestion NAT comme un service géré. Vérifiez auprès de votre fournisseur de connectivité s’il offre un tel service. Si ce n'est pas le cas, vous devez respecter les conditions décrites ci-dessous. 

Examinez la page [Circuits ExpressRoute et domaines de routage](expressroute-circuit-peerings.md) pour obtenir une vue d'ensemble des différents domaines de routage. Pour répondre aux exigences en matière d'adresses IP publiques pour les peerings publics Azure et Microsoft, nous vous recommandons de configurer un NAT entre votre réseau et Microsoft. Cette section fournit une description détaillée de l'infrastructure NAT que vous devez configurer.

## <a name="nat-requirements-for-microsoft-peering"></a>Configuration NAT requise pour le peering Microsoft
Le chemin de peering Microsoft vous permet de vous connecter aux services de cloud Microsoft non pris en charge via le chemin de peering public Azure. La liste des services inclut les services Microsoft 365, notamment Exchange Online, SharePoint Online et Skype Entreprise. Microsoft prévoit la prise en charge de la connectivité bidirectionnelle sur le peering Microsoft. Le trafic destiné aux services de cloud Microsoft doit être configuré en SNAT avec des adresses IPv4 publiques valides avant leur entrée sur le réseau Microsoft. Le trafic provenant des services de cloud Microsoft doit être configuré en SNAT dans votre session Internet pour éviter un [routage asymétrique](expressroute-asymmetric-routing.md). L’illustration suivante indique de façon sommaire comment configurer un NAT pour le peering Microsoft.

![Diagramme de haut niveau de la façon dont la traduction d’adresses réseau doit être configurée pour Microsoft Peering.](./media/expressroute-nat/expressroute-nat-microsoft.png) 

### <a name="traffic-originating-from-your-network-destined-to-microsoft"></a>Trafic en provenance de votre réseau et destiné à Microsoft
* Vous devez vous assurer que le trafic qui entre via le chemin d'accès de peering Microsoft utilise une adresse IPv4 publique valide. Microsoft doit être en mesure de valider le propriétaire du pool d'adresses NAT IPv4 par rapport au Registre Internet de routage régional (RIR) ou à un Registre de routage Internet (IRR). Une vérification sera effectuée en fonction du numéro AS en cours d’homologation et des adresses IP utilisées pour le NAT. Reportez-vous à la page [Configuration requise pour le routage ExpressRoute](expressroute-routing.md) pour plus d'informations sur les registres de routage.
* Les adresses IP utilisées pour la configuration du peering public Azure et d’autres circuits ExpressRoute ne doivent pas être proposées à Microsoft via la session BGP. Il n'existe aucune restriction concernant la longueur du préfixe IP NAT publié via ce peering.
  
  > [!IMPORTANT]
  > Le pool IP NAT proposé à Microsoft ne doit pas être publié sur Internet. Cela interromprait la connectivité avec d'autres services Microsoft.
  > 
  > 

### <a name="traffic-originating-from-microsoft-destined-to-your-network"></a>Trafic en provenance de Microsoft et destiné à votre réseau
* Certains scénarios exigent que Microsoft initie la connectivité aux points de terminaison de service hébergés au sein de votre réseau. Un exemple typique de ce scénario est la connectivité avec des serveurs ADFS hébergés sur votre réseau à partir de Microsoft 365. Dans ce cas, vous devez transférer les préfixes appropriés de votre réseau vers le peering Microsoft. 
* Vous devez configurer en SNAT le trafic Microsoft dans votre session Internet pour les points de terminaison de service au sein de votre réseau pour éviter un [routage asymétrique](expressroute-asymmetric-routing.md). Les requêtes et **réponses** avec une adresse IP de destination correspondant à un itinéraire reçu via ExpressRoute est toujours envoyé via ExpressRoute. Un routage asymétrique se produit si la requête est reçue via Internet avec la réponse envoyée via ExpressRoute. La configuration en SNAT du trafic Microsoft entrant dans la session Internet force le trafic de réponse vers Internet, résolvant ainsi le problème.

![Routage asymétrique avec ExpressRoute](./media/expressroute-asymmetric-routing/AsymmetricRouting2.png)

## <a name="nat-requirements-for-azure-public-peering"></a>Configuration NAT requise pour le peering public Azure

> [!NOTE]
> Le peering public Azure n’est pas disponible pour les nouveaux circuits.
> 

Le chemin de peering public Azure vous permet de vous connecter à tous les services hébergés dans Azure en utilisant leurs adresses IP publiques. Cela inclut les services répertoriés dans le [FAQ sur ExpressRoute](expressroute-faqs.md) et tous les services hébergés par les éditeurs de logiciels sur Microsoft Azure. 

> [!IMPORTANT]
> La connectivité aux services Microsoft Azure sur le peering public est toujours lancée de votre réseau vers le réseau Microsoft. Par conséquent, les sessions ne peut pas être lancées à partir des services Microsoft Azure vers votre réseau via ExpressRoute. En cas de tentative, les paquets envoyés à ces adresses IP publiées utiliseront Internet au lieu d’ExpressRoute.
> 

Le trafic destiné à Microsoft Azure sur le peering public doit être configuré en SNAT avec des adresses IPv4 publiques valides avant leur entrée sur le réseau Microsoft. L'illustration suivante indique de façon sommaire comment configurer un NAT pour répondre à cette exigence.

![Diagramme de haut niveau de la façon dont la traduction d'adresses réseau peut être configurée en SNAT avec des adresses IPv4 publiques valides avant leur entrée sur le réseau Microsoft.](./media/expressroute-nat/expressroute-nat-azure-public.png) 

### <a name="nat-ip-pool-and-route-advertisements"></a>Pool d’adresses IP NAT et publications de routage
Vous devez vous assurer que le trafic qui entre via le chemin d'accès de peering public Azure utilise une adresse IPv4 publique valide. Microsoft doit être en mesure de valider la propriété du pool d'adresses NAT IPv4 par rapport à un Registre Internet de routage régional (RIR) ou un Registre de routage Internet (IRR). Une vérification sera effectuée en fonction du numéro AS en cours d’homologation et des adresses IP utilisées pour le NAT. Reportez-vous à la page [Configuration requise pour le routage ExpressRoute](expressroute-routing.md) pour plus d'informations sur les registres de routage.

Il n'existe aucune restriction concernant la longueur du préfixe IP NAT publié via ce peering. Vous devez surveiller le pool NAT et vous assurer que vous n’êtes pas à court de sessions NAT.

> [!IMPORTANT]
> Le pool IP NAT proposé à Microsoft ne doit pas être publié sur Internet. Cela interromprait la connectivité avec d'autres services Microsoft.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
* Reportez-vous aux conditions requises pour le [routage](expressroute-routing.md) et la [qualité de service](expressroute-qos.md).
* Pour plus d’informations sur les workflows, consultez [Workflows d’approvisionnement du circuit ExpressRoute et états du circuit](expressroute-workflows.md).
* Configurez votre connexion ExpressRoute.
  
  * [Création d’un circuit ExpressRoute](expressroute-howto-circuit-portal-resource-manager.md)
  * [Configuration du routage](expressroute-howto-routing-portal-resource-manager.md)
  * [Liaison d’un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-portal-resource-manager.md)

