---
title: 'Azure ExpressRoute : Azure pour les ingénieurs réseau'
description: Cette page explique aux ingénieurs réseau traditionnels comment fonctionnent les réseaux dans Azure.
documentationcenter: na
services: networking
author: osamaz
manager: tracsman
ms.service: virtual-network
ms.topic: article
ms.workload: infrastructure-services
ms.date: 06/25/2020
ms.author: osamaz
ms.openlocfilehash: 4f513da4e7883cd273098039c9c4a4645d849f0f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85515914"
---
# <a name="azure-for-network-engineers"></a>Azure pour les ingénieurs réseau
En tant qu’ingénieur réseau classique, vous avez eu affaire à des biens matériels tels que des routeurs, des commutateurs, des câbles et des pare-feu pour créer une infrastructure. Dans une couche logique, vous avez configuré le réseau local virtuel (VLAN), le protocole STP (Spanning Tree Protocol) et des protocoles de routage (RIP, OSPF, BGP). Vous avez géré votre réseau à l’aide d’outils de gestion et de l’interface CLI. La mise en réseau dans le cloud est différente lorsque les points de terminaison réseau sont logiques et que l’utilisation de protocoles de routage est minimale. Vous utiliserez l’API Azure Resource Manager, Azure CLI et PowerShell pour configurer et gérer des ressources dans Azure. Vous allez commencer votre parcours réseau dans le cloud en comprenant les tenants fondamentaux du réseau Azure. 
## <a name="virtual-network"></a>Réseau virtuel
Lorsque vous concevez un réseau de bas en haut, vous rassemblez quelques informations de base. Ces informations peuvent être le nombre d’hôtes, les périphériques réseau, le nombre de sous-réseaux, le routage entre les sous-réseaux et les domaines d’isolement tels que les VLAN. Ces informations permettent de dimensionner le réseau et les appareils de sécurité, ainsi que de créer l’architecture nécessaire à la prise en charge des applications et des services.

Lorsque vous envisagez de déployer vos applications et services dans Azure, vous allez commencer par créer une limite logique dans Azure, appelée réseau virtuel. Ce réseau virtuel s’apparente à une limite de réseau physique. Comme il s’agit d’un réseau virtuel, vous n’avez pas besoin d’équipement physique, mais vous devez tout de même prévoir les entités logiques telles que les adresses IP, les sous-réseaux IP, le routage et les stratégies.

Lorsque vous créez un réseau virtuel dans Azure, il est préconfiguré avec une plage d’adresses IP (10.0.0.0/16). Cette plage n’est pas fixe, vous pouvez définir votre propre plage d’adresses IP. Vous pouvez définir des plages d’adresses IPv4 et IPv6. Les plages d’adresses IP définies pour le réseau virtuel ne sont pas communiquées à l’Internet. Vous pouvez créer plusieurs sous-réseaux à partir de votre plage d’adresses IP. Ces sous-réseaux seront utilisés pour attribuer des adresses IP aux interfaces réseau virtuelles (cartes réseau virtuelles). Les quatre premières adresses IP de chaque sous-réseau sont réservées et ne peuvent pas être utilisées pour l’allocation d’adresses IP. Il n’existe aucun concept de VLAN dans un cloud public. Toutefois, vous pouvez créer un isolement au sein d’un réseau virtuel en fonction de vos sous-réseaux définis.

Vous pouvez créer un grand sous-réseau englobant tout l’espace d’adressage du réseau virtuel ou choisir de créer plusieurs sous-réseaux. Toutefois, si vous utilisez une passerelle de réseau virtuel, Azure vous oblige à créer un sous-réseau portant le nom « gateway subnet ». Azure utilisera ce sous-réseau pour attribuer des adresses IP aux passerelles de réseau virtuel. 

## <a name="ip-allocation"></a>Allocation d’adresses IP

Lorsque vous attribuez une adresse IP à un hôte, vous attribuez l’adresse IP à une carte réseau. Vous pouvez attribuer deux types d’adresses IP à une carte réseau dans Azure :

- IP publiques : utilisées pour la communication entrante et sortante (sans traduction d’adresses réseau [NAT]) avec Internet et d’autres ressources Azure non connectées à un réseau virtuel. L’attribution d’une adresse IP publique à une carte d’interface réseau est facultative. Les IP publiques appartiennent à l’espace d’adressage IP Microsoft.
- Adresses IP privées : utilisées pour la communication au sein d’un réseau virtuel, avec votre réseau local et Internet (sans NAT). L’espace d’adressage IP que vous définissez dans le réseau virtuel est considéré comme privé même si vous configurez votre espace d’adressage IP public. Microsoft ne communique pas cet espace à l’Internet. Vous devez attribuer au moins une adresse IP privée à une machine virtuelle.

Comme pour les périphériques ou les hôtes physiques, il existe deux façons d’allouer une adresse IP à une ressource : de manière dynamique ou statique. Dans Azure, la méthode d’allocation par défaut est dynamique, où une adresse IP est allouée lorsque vous créez une machine virtuelle ou démarrez une machine virtuelle arrêtée. L’adresse IP est libérée lorsque vous arrêtez ou supprimez la machine virtuelle. Pour vous assurer que l’adresse IP de la machine virtuelle ne change pas, vous pouvez définir explicitement la méthode d’allocation sur statique. Dans ce cas, une adresse IP est attribuée immédiatement. Elle est libérée uniquement lorsque vous supprimez la machine virtuelle ou lorsque vous modifiez sa méthode d’allocation en dynamique. 

Les adresses IP privées sont allouées à partir des sous-réseaux que vous avez définis dans un réseau virtuel. Pour une machine virtuelle, vous choisissez un sous-réseau pour l’allocation d’adresses IP. Si une machine virtuelle contient plusieurs cartes réseau, vous pouvez choisir un sous-réseau différent pour chaque carte réseau.

## <a name="routing"></a>Routage
Lorsque vous créez un réseau virtuel, Azure crée une table de routage pour votre réseau. Cette table de routage contient les types d’itinéraires suivants.
- Itinéraires système
- Itinéraires par défaut des sous-réseaux
- Itinéraires à partir d’autres réseaux virtuels
- Itinéraires BGP
- Itinéraires de point de terminaison de service
- Itinéraires définis par l’utilisateur (UDR)

Lorsque vous créez un réseau virtuel pour la première fois sans définir de sous-réseaux, Azure crée des entrées de routage dans la table de routage. Ces itinéraires sont appelés itinéraires système. Les itinéraires système sont définis à cet emplacement. Vous ne pouvez pas modifier ces itinéraires. Toutefois, vous pouvez remplacer les itinéraires système en configurant des UDR.

Lorsque vous créez un ou plusieurs sous-réseaux au sein d’un réseau virtuel, Azure crée des entrées par défaut dans la table de routage pour permettre la communication entre ces sous-réseaux au sein du réseau virtuel. Ces itinéraires peuvent être modifiés à l’aide d’itinéraires statiques, qui sont des itinéraires définis par l’utilisateur (UDR) dans Azure.

lorsque vous créez un peering de réseau virtuel entre deux réseaux virtuels, un itinéraire est ajouté pour chaque plage d’adresses dans l’espace d’adressage de chaque réseau virtuel pour laquelle un peering est créé.

Si votre passerelle réseau locale échange des itinéraires Border Gateway Protocol (BGP) avec une passerelle de réseau virtuel Azure, un itinéraire est ajouté pour chaque itinéraire propagé à partir de la passerelle réseau locale. Ces itinéraires apparaissent dans la table de routage en tant qu’itinéraires BGP.

les adresses IP publiques de certains services sont ajoutées à la table de routage par Azure lorsque vous activez un point de terminaison de service pour le service. Des points de terminaison de service sont activés pour des sous-réseaux individuels au sein d’un réseau virtuel. Lorsque vous activez un point de terminaison de service, l’itinéraire est uniquement ajouté à la table de routage du sous-réseau qui appartient à ce service. Azure gère automatiquement les adresses dans la table de routage en cas de changement d’adresses.

Les itinéraires définis par l’utilisateur sont également appelés itinéraires personnalisés. Vous créez des UDR dans Azure pour remplacer les itinéraires système par défaut d’Azure ou pour ajouter des itinéraires supplémentaires à la table de routage d’un sous-réseau. Dans Azure, vous créez une table de routage, puis associez cette dernière aux sous-réseaux du réseau virtuel.

Lorsque vous avez des entrées en concurrence dans une table de routage, Azure sélectionne le tronçon suivant en fonction de la correspondance du préfixe le plus long, comme pour les routeurs traditionnels. Toutefois, s’il existe plusieurs entrées de tronçon suivant avec le même préfixe d’adresse, Azure sélectionne les itinéraires dans l’ordre suivant.
1. Itinéraires définis par l’utilisateur (UDR)
1. Itinéraires BGP
1. Itinéraires système

## <a name="security"></a>Sécurité

Vous pouvez filtrer le trafic vers et depuis les ressources d’un réseau virtuel à l’aide de groupes de sécurité réseau. Vous pouvez également utiliser des appliances virtuelles réseau (NVA) telles que Pare-feu Azure ou des pare-feux d’autres fournisseurs. Vous pouvez contrôler la façon selon laquelle Azure achemine le trafic à partir des sous-réseaux. Vous pouvez également limiter qui dans votre organisation peut utiliser des ressources dans des réseaux virtuels.

Un groupe de sécurité réseau contient une liste des règles de liste de contrôle d’accès qui autorise ou rejette le trafic vers les sous-réseaux, les cartes réseau ou les deux. Des groupes de sécurité réseau peuvent être associés à des sous-réseaux ou à des cartes d’interface réseau connectées à un sous-réseau. Lorsqu’un groupe de sécurité réseau est associé à un sous-réseau, les règles de liste de contrôle d’accès s’appliquent à toutes les machines virtuelles présentes dans ce sous-réseau. En outre, le trafic vers une carte d’interface réseau peut être limité par l’association directe d’un groupe de sécurité réseau à une carte d’interface réseau.

Les groupes de sécurité réseau contiennent deux ensembles de règles : les règles de trafic entrant et les règles de trafic sortant. La priorité d’une règle doit être unique dans chaque ensemble. Chaque règle possède des propriétés relatives au protocole, aux plages de ports de destination et source, aux préfixes d’adresse, à la direction du trafic, à la priorité et au type d’accès. Tous les groupes de ressources réseau contiennent un ensemble de règles par défaut. Les règles par défaut ne peuvent pas être supprimées, mais comme la priorité la plus basse leur est attribuée, elles peuvent être remplacées par les règles que vous créez.

## <a name="verification"></a>Vérification
### <a name="routes-in-virtual-network"></a>Itinéraires dans le réseau virtuel
Ce mélange d’itinéraires que vous créez, d’itinéraires par défaut d’Azure et d’itinéraires propagés à partir de votre réseau local via une passerelle VPN Azure (si le réseau virtuel est connecté au réseau local) en utilisant Border Gateway Protocol (BGP), constitue les itinéraires effectifs de toutes les interfaces réseau d’un sous-réseau. Vous pouvez voir ces itinéraires efficaces en accédant à la carte réseau via le portail, PowerShell ou l’interface CLI.
### <a name="network-security-groups"></a>Network Security Group
Les règles de sécurité effectives appliquées à une interface réseau sont une agrégation des règles qui existent dans un NSG associé à l’interface réseau, et le sous-réseau sur lequel celle-ci se trouve. Vous pouvez afficher toutes les règles de sécurité en vigueur de vos groupes de sécurité réseau, telles qu’appliquées aux interfaces réseau de votre machine virtuelle en accédant à la carte réseau via le portail, PowerShell ou l’interface CLI.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur [le réseau virtuel][VNet].

En savoir plus sur [le routage de réseau virtuel][vnet-routing].

En savoir plus sur [les groupes de sécurité réseau][network-security].

<!--Link References-->
[VNet]: https://docs.microsoft.com/azure/virtual-network/tutorial-connect-virtual-networks-portal
[vnet-routing]: https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview
[network-security]: https://docs.microsoft.com/azure/virtual-network/security-overview

