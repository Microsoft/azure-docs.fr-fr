---
title: 'Voir les routes effectives d’un hub virtuel : Azure Virtual WAN | Microsoft Docs'
description: Comment afficher les itinéraires effectifs d’un hub virtuel dans Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: ad487aa7492abcf02988c78ccfa2ba7cd4798249
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90983644"
---
# <a name="view-virtual-hub-effective-routes"></a>Afficher les routes effectives d’un hub virtuel

Vous pouvez afficher tous les itinéraires de votre hub WAN virtuel dans le portail Azure. Cet article détaille les étapes permettant d’afficher les itinéraires effectifs. Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="select-connections-or-route-tables"></a><a name="routing"></a>Sélectionner des connexions ou des tables de routage

1. Accédez à votre hub virtuel, puis sélectionnez **Routage**. Sur la page de Routage, sélectionnez **Itinéraires effectifs**.
1. Dans la liste déroulante, vous pouvez sélectionner **Table de routage**. Si vous ne voyez pas d’option de table de routage, cela signifie que vous n’avez pas de table de routage personnalisée ou par défaut configurée dans ce hub virtuel.

## <a name="view-output"></a><a name="output"></a>Afficher la sortie

La sortie de la page affiche les champs suivants :

* **Préfixe** : préfixe d'adresse connu de l'entité actuelle (transmis par le routeur du hub virtuel)
* **Type de tronçon suivant** : Il peut s’agir d’une connexion de réseau virtuel, d’une VPN_S2S_Gateway, d’une ExpressRouteGateway, d’un hub distant ou d’un pare-feu Azure.
* **Tronçon suivant** : il s'agit du lien d'accès à l'ID de ressource du tronçon suivant, ou affiche simplement le libellé On-link pour impliquer le hub actuel.
* **Origine** : ID de ressource de la source de routage.
* **Chemin AS** : Le chemin d’accès AS (système autonome) de l’attribut BGP répertorie tous les numéros AS qui doivent être parcourus pour atteindre l’emplacement à partir duquel le chemin d’accès est attaché.

### <a name="example"></a><a name="example"></a>Exemple

Les valeurs dans l’exemple de tableau suivant impliquent que la connexion de hub virtuel ou la table de routage ont appris l’itinéraire de 10.2.0.0/24 (un préfixe de branche). Elles ont appris l’itinéraire grâce au **type de VPN du tronçon suivant** VPN_S2S_Gateway avec l’ID de ressource de passerelle VPN du **tronçon suivant**. **L’origine de l’itinéraire** pointe vers l’ID de ressource de la passerelle VPN/table de routage/connexion d’origine. **Chemin AS** indique le chemin AS pour la branche.

Utilisez la barre de défilement au bas de la table pour voir le « Chemin AS ».

| **Préfixe** |  **Type de tronçon suivant** | **Tronçon suivant** |  **Origine de la route** |**Chemin AS** |
| ---        | ---                | ---          | ---               | ---         |
| 10.2.0.0/24| VPN_S2S_Gateway |/subscriptions/`<sub id>`/resourceGroups/`<resource group name>`/providers/Microsoft.Network/vpnGateways/vpngw|/subscriptions/`<sub id>`/resourceGroups/`<resource group name>`/providers/Microsoft.Network/vpnGateways/vpngw| 20000|

**Considérations :**

* Si vous voyez 0.0.0.0/0 dans la sortie **Obtenir des itinéraires effectifs**, cela signifie que l’itinéraire existe dans l’une des tables de routage. Toutefois, si cet itinéraire a été configuré pour Internet, un indicateur supplémentaire **« enableInternetSecurity » : true** est requis sur la connexion. L’itinéraire effectif sur la carte réseau de la machine virtuelle n’affiche pas l’itinéraire si l’indicateur « enableInternetSecurity » sur la connexion est « false ».

* Le champ **Propager l’itinéraire par défaut** s’affiche dans le portail Azure Virtual WAN lorsque vous modifiez une connexion de réseau virtuel, une connexion VPN ou une connexion ExpressRoute. Ce champ affiche l’indicateur **enableInternetSecurity**, qui est toujours défini par défaut sur « false » pour les connexions ExpressRoute et VPN, mais sur « true » pour les connexions de réseau virtuel.

* Lorsque vous consultez les itinéraires effectifs sur la carte réseau d'une machine virtuelle, si le tronçon suivant correspond à « Virtual Network Gateway », cela implique le routeur du hub virtuel si la machine virtuelle se trouve dans un réseau en étoile connecté à un hub Virtual WAN.

* Le champ Afficher les itinéraires effectifs pour une table de routage de hub virtuel est renseigné uniquement si le hub virtuel est connecté à au moins un type de connexion (VPN/ER/VNET).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Azure Virtual WAN, consultez l’article [Vue d’ensemble d’Azure Virtual WAN](virtual-wan-about.md).
* Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).
