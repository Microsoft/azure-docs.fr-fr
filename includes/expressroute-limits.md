---
title: Fichier Include
description: Fichier Include
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: include
ms.date: 07/25/2019
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: d1402bcc0c46003429e1809e8d09e9662218cd82
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89147780"
---
| Ressource | Limite |
| --- | --- |
| Circuits ExpressRoute par abonnement |10 |
| Circuits ExpressRoute par région par abonnement, avec Azure Resource Manager |10 |
| Nombre maximal d’itinéraires publiés vers le peering privé Azure avec ExpressRoute Standard |4 000 |
| Nombre maximal d’itinéraires publiés vers le peering privé Azure avec le module complémentaire ExpressRoute Premium |10 000 |
| Nombre maximal d'itinéraires publiés depuis le peering privé Azure à partir de l’espace d’adressage de réseau virtuel pour une connexion ExpressRoute |200 |
| Nombre maximal d’itinéraires publiés vers le peering Microsoft avec ExpressRoute Standard |200 |
| Nombre maximal d’itinéraires publiés vers le peering Microsoft avec le module complémentaire ExpressRoute Premium |200 |
| Nombre maximal de circuits ExpressRoute liés au même réseau virtuel dans le même emplacement de peering |4 |
| Nombre maximal de circuits ExpressRoute liés au même réseau virtuel dans différents emplacements de peering |4 |
| Nombre de liens de réseau virtuel autorisés par circuit ExpressRoute |Consultez le tableau [Nombre de réseaux virtuels par circuit ExpressRoute](#vnetpercircuit).  |

#### <a name="number-of-virtual-networks-per-expressroute-circuit"></a><a name="vnetpercircuit"></a> Nombre de réseaux virtuels par circuit ExpressRoute
| **Taille du circuit** | **Nombre de liens de réseau virtuel pour la version standard** | **Nombre de liens de réseau virtuel avec le module complémentaire Premium** |
| --- | --- | --- |
| 50 Mbits/s |10 |20 |
| 100 Mbits/s |10 |25 |
| 200 Mbits/s |10 |25 |
| 500 Mbits/s |10 |40 |
| 1 Gbit/s |10 |50 |
| 2 Gbit/s |10 |60 |
| 5 Gbit/s |10 |75 |
| 10 Gbits/s |10 |100 |
| 40 Gbits/s* |10 |100 |
| 100 Gbits/s* |10 |100 |

**ExpressRoute Direct 100 Gbits/s uniquement*

> [!NOTE]
> Les connexions Global Reach sont décomptées par rapport à la limite de connexions de réseau virtuel par circuit ExpressRoute. Par exemple, un circuit Premium de 10 Gbits/s autorise 5 connexions Global Reach et 95 connexions aux passerelles ExpressRoute ou 95 connexions Global Reach et 5 connexions aux passerelles ExpressRoute, ou toute autre combinaison jusqu’à la limite de 100 connexions pour le circuit.
