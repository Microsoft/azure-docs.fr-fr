---
title: Garantir la haute disponibilité des applications lors de leur exécution dans VMware sur Azure
description: Décrit les fonctionnalités de haute disponibilité CloudSimple qui permettent de résoudre les scénarios de défaillance courants pour les applications exécutées dans un cloud privé CloudSimple
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: a3eed033ba6a1a6f9237116a53ec7751ae906fe4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77025348"
---
# <a name="ensure-application-high-availability-when-running-in-vmware-on-azure"></a>Garantir la haute disponibilité des applications lors de leur exécution dans VMware sur Azure

La solution CloudSimple fournit des fonctionnalités de haute disponibilité pour les applications exécutées sur VMware dans l'environnement Azure. Le tableau suivant répertorie les scénarios de défaillance et les fonctionnalités de haute disponibilité (HA) associées.

| Scénario de défaillance | Application protégée ? | Fonctionnalité HA plateforme | Fonctionnalité HA VMware | Fonctionnalité HA Azure |
------------ | ------------- | ------------ | ------------ | ------------- |
| Défaillance disque | YES | Remplacement rapide du nœud défaillant | [À propos de la stratégie de stockage par défaut du réseau SAN virtuel](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.virtualsan.doc/GUID-C228168F-6807-4C2A-9D74-E584CAF49A2A.html) |
| Défaillance ventilateur | YES | Ventilateurs redondants, remplacement rapide du nœud défaillant |  |  |
| Défaillance carte réseau | YES | Carte réseau redondante, remplacement rapide du nœud défaillant
| Panne d'alimentation de l'hôte | YES | Alimentation redondante |  |  |
| Défaillance hôte ESXi | YES | Remplacement rapide du nœud défaillant | [Haute disponibilité VMware vSphere](https://www.vmware.com/products/vsphere/high-availability.html) |  |  |
| Défaillance machines virtuelles | YES | [Équilibreurs de charge](load-balancers.md)  | [Haute disponibilité VMware vSphere](https://www.vmware.com/products/vsphere/high-availability.html) | Azure Load Balancer pour les machines virtuelles VMware sans état |
| Défaillance port commuté de nœud terminal | YES | Carte réseau redondante |  |  |
| Défaillance commutateur de nœud terminal | YES | Commutateurs de nœud terminal redondants |  |  |
| Défaillance rack | YES | Groupes de placement |  |  |
| Connectivité de réseau pour un contrôleur de domaine local | YES  | Services de mise en réseau redondants |  | Circuits ER redondants |
| Connectivité réseau à Azure | YES | |  | Circuits ER redondants |
| Défaillance centre de données | YES |  |  | Zones de disponibilité |
| Défaillance régionale | YES  |  |  | Régions Azure |

Azure VMware Solution by CloudSimple fournit les fonctionnalités de haute disponibilité suivantes.

## <a name="fast-replacement-of-failed-node"></a>Remplacement rapide du nœud défaillant

Le logiciel du plan de contrôle de CloudSimple surveille en permanence l'intégrité des clusters VMware et détecte toute défaillance sur un nœud ESXi. Il ajoute ensuite automatiquement un nouvel hôte ESXi au cluster VMware concerné à partir de son pool de nœuds disponibles et retire le nœud défaillant du cluster. Cette fonctionnalité permet de restaurer rapidement la capacité disponible dans le cluster VMware afin de rétablir la résilience fournie par le réseau SAN virtuel et la haute disponibilité VMware.

## <a name="placement-groups"></a>Groupes de placement

Un utilisateur qui crée un cloud privé peut sélectionner une région Azure et un groupe de placement dans la région sélectionnée. Un groupe de placement est un ensemble de nœuds répartis sur plusieurs racks au sein du même segment de réseau. Les nœuds d'un même groupe de placement peuvent se joindre avec un maximum de deux tronçons de commutation supplémentaires. Un groupe de placement se trouve toujours dans la même zone de disponibilité Azure et s'étend sur plusieurs racks. Le plan de contrôle CloudSimple répartit les nœuds d'un cloud privé sur plusieurs racks en fonction de l'effort optimal. Les nœuds de différents groupes de placement sont placés dans des racks différents.

## <a name="availability-zones"></a>Zones de disponibilité

Les zones de disponibilité constituent une offre à haute disponibilité qui protège vos applications et données des pannes des centres de données. Les zones de disponibilité sont des emplacements physiques spéciaux au sein d'une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un système de refroidissement et d’un réseau indépendants. Chaque région dispose d'une zone de disponibilité. Pour plus d’informations, consultez [Que sont les zones de disponibilité dans Azure ?](../availability-zones/az-overview.md).

## <a name="redundant-azure-expressroute-circuits"></a>Circuits Azure ExpressRoute redondants

La connectivité du centre de données à Azure vNet via ExpressRoute comporte des circuits redondants pour fournir un lien de connectivité réseau à haut niveau de disponibilité.

## <a name="redundant-networking-services"></a>Services de mise en réseau redondants

Tous les services de mise en réseau CloudSimple du cloud privé (VLAN, pare-feu, adresses IP publiques, Internet, VPN, etc.) sont conçus pour être hautement disponibles et capables de prendre en charge le contrat de niveau de service (SLA).

## <a name="azure-layer-7-load-balancer-for-stateless-vmware-vms"></a>Azure Load Balancer Layer 7 pour les machines virtuelles VMware sans état

Les utilisateurs peuvent placer un équilibreur de charge Azure Load Balancer Layer 7 devant les machines virtuelles sans état du niveau Web exécutées dans l'environnement VMware afin d'obtenir une haute disponibilité pour le niveau Web.

## <a name="azure-regions"></a>Régions Azure

Une région Azure est un ensemble de centres de données déployés dans un périmètre avec latence définie et connectés via un réseau régional dédié à faible latence. Pour plus d'informations, consultez [Régions Azure](https://azure.microsoft.com/global-infrastructure/regions).
