---
title: Consignes pour planifier un réseau Azure NetApp Files | Microsoft Docs
description: Fournit des instructions qui peuvent vous aider à concevoir une architecture réseau efficace à l’aide d’Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: ram-kakani
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: ramakk
ms.openlocfilehash: 96d8ba058a33d408ec2ee2a1adfba9011f393da9
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184482"
---
# <a name="guidelines-for-azure-netapp-files-network-planning"></a>Consignes pour planifier un réseau Azure NetApp Files

La planification de l’architecture réseau est un élément clé dans la conception d’infrastructure d’application. Cet article vous aide à concevoir une architecture réseau efficace pour que vos charges de travail tirent parti des fonctionnalités enrichies d’Azure NetApp Files.

Les volumes Azure NetApp Files sont conçus pour être contenus dans un sous-réseau à but précis appelé [sous-réseau délégué](../virtual-network/virtual-network-manage-subnet.md), au sein de votre réseau virtuel Azure. Par conséquent, vous pouvez accéder aux volumes directement depuis votre réseau virtuel, depuis des réseaux virtuels homologués dans la même région ou en local via une passerelle de réseau virtuel (ExpressRoute ou passerelle VPN) en fonction des besoins. Le sous-réseau est dédié à Azure NetApp Files et il n’y a aucune connectivité aux autres services Azure ni à internet.

## <a name="considerations"></a>Considérations  

Vous devez comprendre quelques considérations lorsque vous prévoyez d’utiliser un réseau Azure NetApp Files.

### <a name="constraints"></a>Contraintes

Les fonctionnalités ci-dessous ne sont actuellement pas prises en charge pour Azure NetApp Files : 

* Groupes de sécurité réseau (NSG) appliqués au sous-réseau délégué
* Itinéraires définis par l’utilisateur (UDR) appliqués au sous-réseau délégué
* Stratégies Azure (par exemple, des stratégies d’affectation de noms personnalisés) sur l’interface Azure NetApp Files
* Équilibreurs de charge pour le trafic d’Azure NetApp Files
* WAN virtuel Azure 
* Passerelles de réseau virtuel redondantes dans une zone (références SKU de passerelle avec AZ) 
* Actives/Passerelles de réseau virtuel actives 
* Réseau virtuel à double pile (IPv4 et IPv6)

Les restrictions suivantes s’appliquent à Azure NetApp Files :

* Le nombre d’adresses IP en cours d’utilisation dans un réseau virtuel avec Azure NetApp Files (y compris les réseaux virtuels homologués) ne peut pas dépasser 1 000. Nous travaillons à l’augmentation de cette limite pour répondre aux demandes de mise à l’échelle du client. 
* Dans chaque réseau virtuel Azure, un seul sous-réseau peut être délégué à Azure NetApp Files.


### <a name="supported-network-topologies"></a>Topologies de réseau prises en charge

Le tableau suivant décrit les topologies de réseau prises en charge par Azure NetApp Files.  Il décrit également les solutions de contournement pour les topologies non prises en charge. 

|    Topologies    |    est pris en charge    |     Solution de contournement    |
|-------------------------------------------------------------------------------------------------------------------------------|--------------------|-----------------------------------------------------------------------------|
|    Connectivité à un volume dans un réseau local virtuel    |    Oui    |         |
|    Connectivité à un volume dans un réseau virtuel homologué (même région)    |    Oui    |         |
|    Connectivité à un volume dans un réseau virtuel homologué (entre région ou peering mondial)    |    Non    |    None    |
|    Connectivité à un volume via la passerelle ExpressRoute    |    Oui    |         |
|    Connectivité locale à un volume dans un réseau virtuel spoke via la passerelle ExpressRoute et le peering du réseau avec transit de passerelle    |    Oui    |        |
|    Connectivité locale à un volume dans un réseau virtuel spoke via la passerelle VPN    |    Oui    |         |
|    Connectivité locale à un volume dans un réseau virtuel spoke via la passerelle VPN et le peering du réseau avec transit de passerelle    |    Oui    |         |


## <a name="virtual-network-for-azure-netapp-files-volumes"></a>Réseau virtuel pour les volumes Azure NetApp Files

Cette section explique les concepts qui vous aident à la planification du réseau virtuel.

### <a name="azure-virtual-networks"></a>Réseaux virtuels Azure

Avant d’approvisionner un volume Azure NetApp Files, vous devez créer un réseau virtuel Azure (VNet) ou en utiliser un qui existe déjà dans votre abonnement. Le réseau virtuel définit la limite réseau du volume.  Pour plus d’informations sur la création de réseaux virtuels, voir la [documentation du réseau virtuel Azure](../virtual-network/virtual-networks-overview.md).

### <a name="subnets"></a>Sous-réseaux

Les sous-réseaux segmentent le réseau virtuel en espaces d’adressage distincts utilisables par les ressources Azure que vous placez dedans.  Les volumes Azure NetApp Files sont contenus dans un sous-réseau à but précis appelé [sous-réseau délégué](../virtual-network/virtual-network-manage-subnet.md). 

La délégation de sous-réseau donne des autorisations explicites au service Azure NetApp Files pour créer des ressources propres au service dans le sous-réseau.  Il utilise un identificateur unique dans le déploiement du service. Dans ce cas, une interface réseau est créée pour activer la connectivité à Azure NetApp Files.

Si vous utilisez un réseau virtuel, vous pouvez créer un sous-réseau et déléguer le sous-réseau à Azure NetApp Files en suivant les instructions dans [Déléguer un sous-réseau à Azure NetApp Files](azure-netapp-files-delegate-subnet.md). Vous pouvez également déléguer un sous-réseau vide existant qui n’est pas déjà délégué à d’autres services.

Si le réseau virtuel est appairé à un autre réseau virtuel, vous ne pouvez pas développer l’espace d’adressage du réseau virtuel. Pour cette raison, le nouveau sous-réseau délégué doit être créé au sein de l’espace d’adressage du réseau virtuel. Si vous avez besoin d’étendre l’espace d’adressage, vous devez d’abord supprimer le peering.

### <a name="udrs-and-nsgs"></a>Itinéraires définis par l’utilisateur et groupes de sécurité réseau

Les groupes de sécurité réseau (NSG) et les itinéraires définis par l’utilisateur (UDR) ne sont pas pris en charge sur les sous-réseaux délégués pour Azure NetApp Files. Toutefois, vous pouvez appliquer des itinéraires définis par l’utilisateur et des groupes de sécurité réseau à d’autres sous-réseaux, même au sein du même réseau virtuel que le sous-réseau délégué à Azure NetApp Files.

* Les itinéraires définis par l’utilisateur définissent ensuite les flux de trafic des autres sous-réseaux vers le sous-réseau délégué Azure NetApp Files. Cela permet de s’assurer qu’il est aligné sur le flux de trafic de Azure NetApp Files vers les autres sous-réseaux à l’aide des itinéraires système.  
* Les groupes de sécurité réseau autorisent ou refusent le trafic vers et depuis le sous-réseau délégué Azure NetApp Files. 

## <a name="azure-native-environments"></a>Environnements natifs Azure

Le diagramme suivant illustre un environnement natif Azure :

![Environnement de mise en réseau Azure natif](../media/azure-netapp-files/azure-netapp-files-network-azure-native-environment.png)

### <a name="local-vnet"></a>Réseau virtuel local

Un scénario de base consiste à créer ou à se connecter à un volume Azure NetApp Files à partir d’une machine virtuelle (VM) dans le même réseau virtuel. Pour le réseau virtuel 2 dans le diagramme ci-dessus, Volume 1 est créé dans un sous-réseau délégué et peut être monté sur la machine virtuelle 1 dans le sous-réseau par défaut.

### <a name="vnet-peering"></a>Peering de réseaux virtuels

Si vous avez d’autres réseaux virtuels dans la même région qui ont besoin d’accéder aux ressources d’un autre, les réseaux virtuels peuvent être connectés à l’aide du [peering de réseau virtuel](../virtual-network/virtual-network-peering-overview.md) pour activer la connectivité sécurisée via l’infrastructure Azure. 

Prenez pour exemple les réseaux virtuels 2 et 3 dans le diagramme ci-dessus. Si la machine virtuelle 1 doit se connecter à la machine virtuelle 2 ou au volume 2, ou si la machine virtuelle 2 doit se connecter à la machine virtuelle 1 ou au volume 1, vous devez activer le peering de réseaux virtuels entre les réseaux virtuels 2 et 3. 

De plus, prenez pour exemple un scénario où le réseau virtuel 1 est appairé au réseau virtuel 2, et où le réseau virtuel 2 est appairé au réseau virtuel 3 dans la même région. Les ressources du réseau virtuel 1 peuvent se connecter aux ressources du réseau virtuel 2, mais pas aux ressources du réseau virtuel 3, sauf si les réseaux virtuels 1 et 3 sont appairés. 

Dans le diagramme ci-dessus, la machine virtuelle 3 peut se connecter au Volume 1, mais la machine virtuelle 4 ne peut pas se connecter au Volume 2.  Il en est ainsi car les réseaux virtuels spoke ne sont pas appairés, et _le routage de transit n’est pas pris en charge via le peering de réseau virtuel_.

## <a name="hybrid-environments"></a>Environnements hybrides

Le diagramme suivant illustre un environnement hybride : 

![Environnement de mise en réseau hybride](../media/azure-netapp-files/azure-netapp-files-network-hybrid-environment.png)

Dans le scénario hybride, les applications de centres de données locaux ont besoin d’accéder aux ressources dans Azure.  C’est le cas que vous souhaitiez étendre votre centre de données à Azure, ou que vous souhaitiez utiliser les services Azure natifs pour la reprise d’activité. Consultez [Options de planification de la passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json#planningtable) pour plus d’informations sur la façon de connecter plusieurs ressources locales à des ressources dans Azure via un réseau privé virtuel de site à site ou un circuit Express Route.

Dans une topologie hub-and-spoke hybride, le hub du réseau virtuel dans Azure centralise la connectivité à votre réseau local. Les spokes sont des réseaux virtuels appairés avec le hub et qui peuvent être utilisés pour isoler les charges de travail.

Selon la configuration, vous pouvez connecter des ressources locales à des ressources dans le hub et les spokes.

Dans la topologie illustrée ci-dessus, le réseau local est connecté à un hub de réseau virtuel dans Azure et il y a 2 réseaux virtuels spoke dans la même région appairés au hub du réseau virtuel.  Dans ce scénario, les options de connectivité prises en charge pour les volumes Azure NetApp Files sont les suivantes :

* Les ressources locales des machines virtuelles 1 et 2 peuvent se connecter au volume 1 dans le hub via un réseau privé virtuel de site à site ou un circuit Express Route. 
* Les ressources locales des machines virtuelles 1 et 2 peuvent se connecter au volume 2 ou 3 via une passerelle VPN site à site et un peering du réseau virtuel régional.
* La machine virtuelle 3 dans le réseau virtuel du hub peut se connecter au volume 2 dans le réseau virtuel 1 du spoke et au volume 3 dans le réseau virtuel 2 du spoke.
* La machine virtuelle 4 dans le réseau virtuel 1 du spoke et la machine virtuelle 5 dans le réseau virtuel 2 du spoke peuvent se connecter au volume 1 dans le réseau virtuel du hub.
* La machine virtuelle 4 dans le réseau virtuel 1 du spoke ne peut pas se connecter au volume 3 du réseau virtuel 2 du spoke. Aussi, la machine virtuelle 5 dans le réseau virtuel 2 du spoke ne peut pas se connecter au volume 2 du réseau virtuel 1 du spoke. Il en est ainsi car les réseaux virtuels spoke ne sont pas appairés, et _le routage de transit n’est pas pris en charge via le peering de réseau virtuel_.
* Dans l’architecture ci-dessus, s’il existe également une passerelle dans le réseau virtuel spoke, la connectivité au volume ANF à partir d’un réseau local se connectant via la passerelle dans le hub sera perdue. Par défaut, la préférence est donnée à la passerelle dans le réseau virtuel spoke et, ainsi, seules les machines qui se connectent via cette passerelle peuvent se connecter au volume ANF.

## <a name="next-steps"></a>Étapes suivantes

[Déléguer un sous-réseau à Azure NetApp Files](azure-netapp-files-delegate-subnet.md)