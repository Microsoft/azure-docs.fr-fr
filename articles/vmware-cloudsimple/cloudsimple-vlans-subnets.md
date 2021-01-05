---
title: Réseaux locaux virtuels et sous-réseaux dans Azure VMware Solution by CloudSimple
description: En savoir plus sur les réseaux locaux virtuels ainsi que les sous-réseaux dans le cloud privé CloudSimple et sur le réseau fourni par CloudSimple pour le déploiement de votre service CloudSimple.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: e3235b394f739e8af846f30dfe51705d0926c843
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88140664"
---
# <a name="vlans-and-subnets-overview"></a>Vue d’ensemble des réseaux locaux virtuels et des sous-réseaux

CloudSimple fournit un réseau par région où votre service CloudSimple est déployé.  Le réseau est un seul espace d’adressage TCP de couche 3 avec routage activé par défaut.  Tous les clouds privés et sous-réseaux créés dans cette région peuvent communiquer entre eux sans aucune configuration supplémentaire.  Vous pouvez créer des groupes de ports distribués sur le serveur vCenter à l’aide des réseaux locaux virtuels.

![Topologie de réseau CloudSimple](media/cloudsimple-network-topology.png)

## <a name="vlans"></a>Réseaux VLAN

Un réseau local virtuel (réseau de couche 2) est créé pour chaque cloud privé.  Le trafic de la couche 2 reste dans les limites d’un cloud privé, ce qui vous permet d’isoler le trafic local dans le cloud privé.  Un réseau local virtuel créé dans le cloud privé peut être utilisé pour créer des groupes de ports distribués uniquement dans ce cloud privé.  Un réseau local virtuel créé dans un cloud privé est configuré automatiquement sur tous les commutateurs connectés aux hôtes d’un cloud privé.

## <a name="subnets"></a>Sous-réseaux

Vous pouvez créer un sous-réseau quand vous créez un réseau local virtuel, en définissant l’espace d’adressage du sous-réseau. Une adresse IP de l’espace d’adressage est affectée en tant que sous-réseau de passerelle. Un seul espace d’adressage privé de couche 3 est affecté par client et par région. Vous pouvez configurer n’importe quel espace d’adressage ne se chevauchant pas conforme à la RFC 1918, avec votre réseau local ou réseau virtuel Azure, dans la région de votre réseau.

Tous les sous-réseaux peuvent communiquer entre eux par défaut, ce qui réduit le temps de travail nécessaire pour configurer le routage entre des clouds privés. Les données Est-Ouest sur des PC de la même région restent dans le même réseau de couche 3 et sont transférées sur l’infrastructure du réseau local au sein de la région. Aucune sortie n’est requise pour la communication entre les clouds privés d’une même région. Cette approche permet d’éliminer toute altération des performances au niveau du réseau étendu et de la sortie lors du déploiement de différentes charges de travail dans différents clouds privés.

## <a name="vspherevsan-subnets-cidr-range"></a>Plage CIDR de sous-réseaux vSphere/vSAN

Un cloud privé est créé sous la forme d’un environnement isolé de la pile VMware (hôtes ESXi, vCenter, vSAN et NSX) géré par un serveur vCenter.  Les composants de gestion sont déployés sur le réseau sélectionné pour la plage CIDR de sous-réseaux vSphere/vSAN.  La plage CIDR du réseau est divisée en plusieurs sous-réseaux pendant le déploiement.

* Préfixe de plage CIDR de sous-réseaux vSphere/vSAN minimum : **/24**
* Préfixe de plage CIDR de sous-réseaux vSphere/vSAN maximum : **/21**

> [!CAUTION]
> L’utilisation des adresses IP de la plage CIDR vSphere/vSAN est réservée à l’infrastructure du cloud privé.  N’utilisez pas d’adresses IP de cette plage pour les machines virtuelles.

### <a name="vspherevsan-subnets-cidr-range-limits"></a>Limites de la plage CIDR des sous-réseaux vSphere/vSAN

La sélection de la taille de la plage CIDR de sous-réseaux vSphere/vSAN a un impact sur la taille de votre cloud privé.  Le tableau suivant indique le nombre maximal de nœuds que vous pouvez avoir en fonction de la taille de la plage CIDR de sous-réseaux vSphere/vSAN.

| Longueur spécifiée du préfixe CIDR de sous-réseaux vSphere/vSAN | Nombre maximal de nœuds |
|---------------------------------------------------|-------------------------|
| /24 | 26 |
| /23 | 58 |
| /22 | 118 |
| /21 | 220 |

### <a name="management-subnets-created-on-a-private-cloud"></a>Sous-réseaux de gestion créés sur un cloud privé

Les sous-réseaux de gestion suivants sont créés quand vous créez un cloud privé.

* **System management**. Réseau local virtuel et sous-réseau pour le réseau de gestion, le serveur DNS et le serveur vCenter des hôtes ESXi.
* **VMotion**. Réseau local virtuel et sous-réseau pour le réseau vMotion des hôtes ESXi.
* **VSAN**. Réseau local virtuel et sous-réseau pour le réseau vSAN des hôtes ESXi.
* **NsxtEdgeUplink1**. Réseau local virtuel et sous-réseau pour les liaisons montantes de réseau local virtuel à un réseau externe.
* **NsxtEdgeUplink2**. Réseau local virtuel et sous-réseau pour les liaisons montantes de réseau local virtuel à un réseau externe.
* **NsxtEdgeTransport**. Réseau local virtuel et sous-réseau pour les zones de transport contrôlant la portée des réseaux de couche 2 dans NSX-T.
* **NsxtHostTransport**. Réseau local virtuel et sous-réseau pour la zone de transport de l’hôte.

### <a name="management-network-cidr-range-breakdown"></a>Décomposition de la plage CIDR du réseau de gestion

La plage CIDR de sous-réseaux vSphere/vSAN spécifiée est divisée en plusieurs sous-réseaux.  Le tableau suivant montre un exemple de la décomposition pour les préfixes autorisés.  L’exemple utilise 192.168.0.0 comme plage CIDR.

Exemple :

| Plage CIDR de sous-réseaux vSphere/vSAN et préfixe spécifiés | 192.168.0.0/21 | 192.168.0.0/22 | 192.168.0.0/23 | 192.168.0.0/24 |
|---------------------------------|----------------|----------------|----------------|----------------|
| Gestion de systèmes | 192.168.0.0/24 | 192.168.0.0/24 | 192.168.0.0/25 | 192.168.0.0/26 |
| vMotion | 192.168.1.0/24 | 192.168.1.0/25 | 192.168.0.128/26 | 192.168.0.64/27 |
| vSAN | 192.168.2.0/24 | 192.168.1.128/25 | 192.168.0.192/26 | 192.168.0.96/27 |
| NSX-T Host Transport | 192.168.4.0/23 | 192.168.2.0/24 | 192.168.1.0/25 | 192.168.0.128/26 |
| NSX-T Edge Transport | 192.168.7.208/28 | 192.168.3.208/28 | 192.168.1.208/28 | 192.168.0.208/28 |
| NSX-T Edge Uplink1 | 192.168.7.224/28 | 192.168.3.224/28 | 192.168.1.224/28 | 192.168.0.224/28 |
| NSX-T Edge uplink2 | 192.168.7.240/28 | 192.168.3.240/28 | 192.168.1.240/28 | 192.168.0.240/28 |

## <a name="next-steps"></a>Étapes suivantes

* [Créer et gérer des réseaux locaux virtuels et des sous-réseaux](create-vlan-subnet.md)
