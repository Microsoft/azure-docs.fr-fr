---
title: Adresses IP privées dans Azure
titlesuffix: Azure Virtual Network
description: Découvrez les adresses IP privées dans Azure.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/28/2020
ms.author: allensu
ms.openlocfilehash: 13688b73c0cd73e13e407f1b75beb8dda61a754f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84172254"
---
# <a name="private-ip-addresses"></a>Adresses IP privées
Les adresses IP privées permettent à des ressources de communiquer dans Azure. 

Les ressources peuvent être :
* Des services Azure tels que :
    * Interfaces réseau de machine virtuelle
    * Équilibreurs de charge interne (ILB)
    * Passerelles d’application
* Dans un [réseau virtuel](virtual-networks-overview.md).
* Accessibles sur un réseau local via une passerelle VPN ou un circuit ExpressRoute.

Les adresses IP privées permettent la communication avec ces ressources sans utilisation d’adresse IP publique.

## <a name="allocation-method"></a>Méthode d’allocation

Azure attribue des adresses IP privées aux ressources à partir de la plage d’adresses du sous-réseau de réseau virtuel dans lequel se trouvent les ressources.

Azure réserve les quatre premières adresses dans chaque plage d’adresses de sous-réseau. Ces adresses ne peuvent pas être attribuées à des ressources. Par exemple, si la plage d’adresses du sous-réseau est 10.0.0.0/16, les adresses 10.0.0.0-10.0.0.3 et 10.0.255.255 ne sont pas disponibles. Les adresses IP de la plage d’adresses du sous-réseau peuvent uniquement être attribuées à une ressource à la fois. 

Il existe deux méthodes d’attribution d’adresses IP :

- **Dynamique** : Azure attribue la première adresse IP non attribuée ou non réservée de la plage d’adresses du sous-réseau. Par exemple, Azure attribue l’adresse 10.0.0.10 à une nouvelle ressource si les adresses 10.0.0.4 à 10.0.0.9 sont déjà attribuées à d’autres ressources. 

    La méthode d’allocation par défaut est dynamique. Une fois attribuées, les adresses IP dynamiques sont libérées si :
    
    * Une interface réseau est supprimée
    * Une interface réseau est réaffectée à un autre sous-réseau au sein du même réseau virtuel
    * La méthode d’allocation devient statique et une adresse IP différente est spécifiée 
    
    Par défaut, Azure définit l’adresse statique sur l’adresse dynamique attribuée précédemment quand vous sélectionnez la méthode d’allocation statique à la place de la méthode dynamique.

- **Statique** : vous sélectionnez et attribuez n’importe quelle adresse IP non attribuée ou non réservée de la plage d’adresses du sous-réseau. 

    Par exemple, la plage d’adresses d’un sous-réseau est 10.0.0.0/16 et les adresses 10.0.0.4 à 10.0.0.9 sont attribuées à d’autres ressources. Vous pouvez attribuer n’importe quelle adresse comprise entre 10.0.0.10 et 10.0.255.254. Les adresses statiques ne sont libérées que si l’interface réseau est supprimée. 
    
    Azure attribue l’adresse IP statique en tant qu’adresse IP dynamique quand la méthode d’allocation est changée. La réaffectation se produit même si l’adresse n’est pas la suivante disponible dans le sous-réseau. L’adresse change quand l’interface réseau est attribuée à un autre sous-réseau.
    
    Pour attribuer l’interface réseau à un sous-réseau différent, vous devez remplacer la méthode d’allocation statique par la méthode d’allocation dynamique. Attribuez l’interface réseau à un sous-réseau différent, puis redéfinissez la méthode d’allocation sur statique. Attribuez une adresse IP à partir de la plage d’adresses du nouveau sous-réseau.
    
## <a name="virtual-machines"></a>Machines virtuelles

Une ou plusieurs adresses IP privées sont attribuées à une ou plusieurs **interfaces réseau**. Les interfaces réseau sont attribuées à une machine virtuelle [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Vous pouvez spécifier la méthode d’allocation dynamique ou statique pour chaque adresse IP privée.

### <a name="internal-dns-hostname-resolution-for-virtual-machines"></a>Résolution de nom d’hôte DNS interne (pour les machines virtuelles)

Les machines virtuelles Azure sont configurées avec des [serveurs DNS gérés par Azure](virtual-networks-name-resolution-for-vms-and-role-instances.md#azure-provided-name-resolution) par défaut. Vous pouvez configurer explicitement des serveurs DNS personnalisés. Ces serveurs DNS assurent la résolution de noms interne pour les machines virtuelles qui se trouvent dans le même réseau virtuel.

Un mappage du nom d’hôte à l’adresse IP privée d’une machine virtuelle est ajouté aux serveurs DNS gérés par Azure. 

Un nom d’hôte est mappé à l’adresse IP principale de l’interface réseau principale quand une machine virtuelle a :

* Plusieurs interfaces réseau
* Plusieurs adresses IP
* Les deux

Les machines virtuelles configurées avec un DNS géré par Azure résolvent les noms d’hôte au sein du même réseau virtuel. Utilisez un serveur DNS personnalisé pour résoudre les noms d’hôte de machines virtuelles dans les réseaux virtuels connectés.

## <a name="internal-load-balancers-ilb--application-gateways"></a>Équilibreurs de charge internes (ILB) et Application Gateway

Vous pouvez affecter une adresse IP privée à la configuration **front-end** des éléments suivants :

* [Équilibreur de charge interne Azure](../load-balancer/load-balancer-internal-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
* [Application Gateway Azure](../application-gateway/application-gateway-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json) 

Cette adresse IP privée sert de point de terminaison interne. Le point de terminaison interne est accessible uniquement aux ressources de son réseau virtuel et aux réseaux distants connectés à ce dernier. Une adresse IP dynamique ou statique peut être attribuée.

## <a name="at-a-glance"></a>Aperçu
Le tableau suivant montre la propriété par le biais de laquelle une adresse IP privée peut être associée à une ressource. 

Les méthodes d’allocation utilisables sont également affichées :

* Dynamique
* statique

| Ressources de niveau supérieur | Association d’adresse IP | Dynamique | statique |
| --- | --- | --- | --- |
| Machine virtuelle |interface réseau |Oui |Oui |
| Équilibrage de charge |Configuration frontale |Oui |Oui |
| passerelle d’application |Configuration frontale |Oui |Oui |

## <a name="limits"></a>limites
Les limites pour l’adressage IP figurent dans l’ensemble des [limites pour la mise en réseau](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits) dans Azure. Ces limites sont exprimées par région et par abonnement. [Contactez le support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour augmenter les limites par défaut jusqu’aux limites maximum en fonction de vos besoins métier.

## <a name="next-steps"></a>Étapes suivantes
Découvrir les [adresses IP publiques dans Azure](public-ip-addresses.md)
* [Déployer une machine virtuelle avec une adresse IP privée statique à l’aide du portail Azure](virtual-networks-static-private-ip-arm-pportal.md)
