---
title: Intégration à un réseau virtuel de services Azure pour l’isolement réseau
titlesuffix: Azure Virtual Network
description: Cet article décrit différentes méthodes d’intégration d’un service Azure à un réseau virtuel qui vous permet d’accéder en toute sécurité au service Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 12/01/2020
ms.author: kumud
ms.openlocfilehash: 814158ad68e63760c1f2a1bce3eeff812de3c13e
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938730"
---
# <a name="integrate-azure-services-with-virtual-networks-for-network-isolation"></a>Intégrer des services Azure à des réseaux virtuels pour l’isolement réseau

L’intégration d’un service Azure à un réseau virtuel vous permet de verrouiller l’accès au service à votre infrastructure de réseau virtuel uniquement. L’infrastructure de réseau virtuel comprend également des réseaux virtuels appairés et des réseaux locaux.

L’intégration au réseau virtuel offre aux services Azure les avantages de l’isolement réseau et peut être accomplie à l’aide d’une ou plusieurs des méthodes suivantes :
- [Déploiement d’instances dédiées du service dans un réseau virtuel](virtual-network-service-endpoints-overview.md). Les services sont alors accessibles de manière privée dans le réseau virtuel, et à partir des réseaux locaux.
- Utilisation d’un [point de terminaison privé](../private-link/private-endpoint-overview.md) qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur [Azure Private Link](../private-link/private-link-overview.md). Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel.
- Accès au service à l’aide de points de terminaison publics en étendant un réseau virtuel au service par le biais de [points de terminaison de service](virtual-network-service-endpoints-overview.md). Les points de terminaison de service permettent de sécuriser les ressources de service au sein du réseau virtuel.
- Utilisation d’[étiquettes de service](service-tags-overview.md) pour autoriser ou refuser le trafic vers vos ressources Azure vers et depuis des points de terminaison IP publics.

## <a name="deploy-dedicated-azure-services-into-virtual-networks"></a>Déployer des services Azure dédiés dans des réseaux virtuels

Quand vous déployez des services Azure dédiés sur un réseau virtuel, vous pouvez communiquer avec les ressources de service de manière privée, par le biais d’adresses IP privées.

![Déployer des services Azure dédiés dans des réseaux virtuels](./media/virtual-network-for-azure-services/deploy-service-into-vnet.png)

Le déploiement d’un service Azure dédié dans votre réseau virtuel offre les capacités suivantes :
- Les ressources situées sur le réseau virtuel peuvent communiquer entre elles de manière privée, par le biais d’adresses IP privées. Exemple : le transfert direct des données entre HDInsight et SQL Server sur une machine virtuelle, au sein d’un réseau virtuel.
- Les ressources locales peuvent accéder aux ressources d’un réseau virtuel à l’aide d’adresses IP privées par le biais d’une connexion VPN site à site (passerelle VPN) ou ExpressRoute.
- Les réseaux virtuels peuvent être appairés pour permettre aux ressources situées sur les réseaux virtuels de communiquer entre elles, par le biais d’adresses IP privées.
- Les instances de service dans un réseau virtuel sont généralement entièrement managées par le service Azure. Cela inclut la surveillance de l’intégrité des ressources et la mise à l’échelle avec une charge.
- Les instances de service sont déployées dans un sous-réseau d’un réseau virtuel. L’accès au réseau entrant ou sortant pour le sous-réseau doit être ouvert par le biais de groupes de sécurité réseau, selon les recommandations fournies par le service.
- Certains services imposent également des restrictions sur le sous-réseau dans lequel ils sont déployés, en limitant l’application des stratégies, des itinéraires ou en combinant des ressources des machines virtuelles et de service au sein du même sous-réseau. Vérifiez sur chaque service les restrictions spécifiques, car elles peuvent changer au fil du temps. Azure NetApp Files, Dedicated HSM, Azure Container Instances, App Service sont des exemples de ces services.
- Les services peuvent éventuellement nécessiter un sous-réseau délégué comme identificateur explicit qu’un sous-réseau peut héberger un service particulier. Grâce à la délégation, les services obtiennent des autorisations explicites pour créer pour créer des ressources propres au service dans le sous-réseau délégué.
- Consultez un exemple de réponse de l’API REST sur un réseau virtuel avec un sous-réseau délégué. Une liste complète de services qui utilisent le modèle de sous-réseau délégué peut être obtenue via l’API Available Delegations (Délégations disponibles).

Pour obtenir la liste des services qui peuvent être déployés dans un réseau virtuel, consultez [Déployer des services Azure dédiés dans des réseaux virtuels](virtual-network-for-azure-services.md).

## <a name="private-link-and-private-endpoints"></a>Liaison privée et points de terminaison privés

Vous pouvez utiliser des points de terminaison privés pour autoriser l’entrée sécurisée d’événements directement de votre réseau virtuel à vos ressources Azure via une liaison privée, sans passer par l’Internet public. Un point de terminaison privé est une interface réseau spéciale pour un service Azure dans votre réseau virtuel. Quand vous créez un point de terminaison privé pour votre ressource Azure, il offre une connectivité sécurisée entre les clients de votre réseau virtuel et votre ressource Azure. Une adresse IP est attribuée au point de terminaison privé à partir de la plage d’adresses IP de votre réseau virtuel. La connexion entre le point de terminaison privé et le service Azure utilise une liaison privée sécurisée.

L’exemple suivant montre l’accès privé d’un point de terminaison privé de la ressource Event Grid qui fournit une connectivité sécurisée entre les clients sur un réseau virtuel et la ressource Event Grid.

![Accès privé de la ressource SQL DB à l’aide d’un point de terminaison privé](./media/network-isolation/architecture-diagram.png)

Pour plus d’informations sur Private Link et pour obtenir la liste des services Azure pris en charge, consultez [Qu’est-ce que Liaison privée ?](../private-link/private-link-overview.md)

## <a name="service-endpoints"></a>Points de terminaison de service
Un point de terminaison de service de réseau virtuel fournit une connectivité sécurisée et directe aux services Azure via un itinéraire optimisé du réseau principal Azure. Les points de terminaison permettent de sécuriser vos ressources critiques du service Azure pour vos réseaux virtuels uniquement. Les points de terminaison de service permettent aux adresses IP privées du réseau virtuel d'atteindre le point de terminaison d'un service Azure sans qu'une adresse IP publique soit nécessaire sur le réseau virtuel.

![Sécurisation des services Azure pour des réseaux virtuels](./media/virtual-network-service-endpoints-overview/VNet_Service_Endpoints_Overview.png)

Pour plus d’informations, consultez [Points de terminaison de service de réseau virtuel](virtual-network-service-endpoints-overview.md).

## <a name="service-tags"></a>Balises de service

Une balise de service représente un groupe de préfixes d’adresses IP d’un service Azure donné. À l’aide d’étiquettes de service, vous pouvez définir des contrôles d’accès réseau sur des [groupes de sécurité réseau](https://docs.microsoft.com/azure/virtual-network/security-overview#security-rules) ou sur [Pare-feu Azure](https://docs.microsoft.com/azure/firewall/service-tags). En spécifiant le nom de l’étiquette de service (par exemple, AzureEventGrid) dans le champ Source ou Destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant.

![Autoriser ou refuser le trafic à l’aide d’étiquettes de service](./media/network-isolation/service-tags.png)

Vous pouvez utiliser des étiquettes de service pour isoler le réseau et protéger vos ressources Azure de l’accès Internet général tout en accédant aux services Azure qui ont des points de terminaison publics. Créez des règles de groupe de sécurité de réseau entrant/sortant pour refuser le trafic vers/depuis **Internet** et autoriser le trafic vers/depuis **AzureCloud** ou d’autres [balises de service disponibles](service-tags-overview.md#available-service-tags) de services Azure spécifiques.

Pour plus d’informations sur les étiquettes de service et les services Azure qui les prennent en charge, consultez [Vue d’ensemble des étiquettes de service](service-tags-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [intégrer votre application à un réseau Azure](../app-service/web-sites-integrate-with-vnet.md).
- Découvrez comment [restreindre l’accès aux ressources à l’aide d’étiquettes de service](tutorial-restrict-network-access-to-resources.md).
- Découvrez comment [vous connecter en privé à un compte Azure Cosmos à l’aide d’Azure Private Link](../private-link/create-private-endpoint-cosmosdb-portal.md).