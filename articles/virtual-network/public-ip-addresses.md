---
title: Adresses IP publiques dans Azure
titlesuffix: Azure Virtual Network
description: En savoir plus sur les adresses IP publiques dans Azure.
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
ms.openlocfilehash: ef79844cf2f90ce97ea30a1948a441f909255f98
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96169931"
---
# <a name="public-ip-addresses"></a>Adresses IP publiques

Les adresses IP publiques permettent aux ressources Internet de communiquer avec les ressources Azure (communication entrante). Les adresses IP publiques permettent aux ressources Azure de communiquer avec Internet et les services Azure publics. L’adresse est dédiée à la ressource jusqu’à ce que vous annuliez son attribution. Une ressource sans adresse IP publique attribuée peut établir une communication sortante. Azure attribue dynamiquement une adresse IP disponible qui n’est pas dédiée à la ressource. Pour plus d’informations sur les connexions sortantes dans Azure, consultez [Comprendre les connexions sortantes dans Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Dans Azure Resource Manager, une [adresse IP publique](virtual-network-public-ip-address.md) est une ressource ayant ses propres propriétés. Voici quelques-unes des ressources auxquelles vous pouvez associer une ressource d’adresse IP publique :

* Interfaces réseau de machine virtuelle
* Équilibreurs de charge accessibles sur Internet
* Passerelles VPN
* Passerelles d’application
* Pare-feu Azure

## <a name="ip-address-version"></a>Version de l’adresse IP

Les adresses IP publiques sont créées avec une adresse IPv4 et IPv6. 

## <a name="sku"></a>SKU

Pour en savoir plus sur la mise à niveau du SKU, consultez [Mise à niveau de l’adresse IP publique](../virtual-network/virtual-network-public-ip-address-upgrade.md).

Les adresses IP publiques sont créées avec l’une des références SKU suivantes :

>[!IMPORTANT]
> Les références SKU qui correspondent doivent être utilisées pour les ressources de Load Balancer et d’adresse IP publique. Vous ne pouvez pas avoir à la fois des ressources de référence SKU De base et de référence SKU Standard. Vous ne pouvez pas joindre des machines virtuelles autonomes, des machines virtuelles dans une ressource de groupe à haute disponibilité ou des ressources de groupe de machines virtuelles identiques aux deux références SKU simultanément.  De nouvelles conceptions doivent être envisagées à l’aide des ressources de référence SKU Standard.  Veuillez consulter [Load Balancer Standard](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour plus d’informations.

### <a name="standard"></a>Standard

Les adresses IP publiques de référence SKU standard :

- Utilisent toujours la méthode d’allocation statique.
- Dotées d’un délai d’inactivité du flux entrant réglable de 4 à 30 minutes, avec une valeur par défaut de 4 minutes et d’un délai d’inactivité du flux sortant fixe de 4 minutes.
- Sont sécurisées par défaut et fermées au trafic entrant. Autorisez la liste du trafic entrant avec un [groupe de sécurité réseau](security-overview.md#network-security-groups).
- Sont assignées à des interfaces réseau, des équilibreurs de charge publics standard ou des passerelles d’application. Pour plus d’informations sur Standard Load Balancer, consultez [Azure Standard Load Balancer](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Peuvent être redondantes dans une zone (publiées depuis les 3 zones) ou zonales (peuvent être créées pour une zone et garanties dans une zone de disponibilité spécifique). Pour en savoir plus sur les zones de disponibilité, consultez [Vue d’ensemble des zones de disponibilité](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Équilibreur de charge standard et zones de disponibilité](../load-balancer/load-balancer-standard-availability-zones.md?toc=%2fazure%2fvirtual-network%2ftoc.json). **Les IP redondantes dans une zone peuvent être créées seulement dans des [régions où 3 zones de disponibilité](https://docs.microsoft.com/azure/availability-zones/az-region) sont actives.** Les IP créées avant que les zones soient actives ne sont pas redondantes dans une zone.
 
> [!NOTE]
> La communication entrante avec la ressource de référence SKU standard est possible uniquement si vous créez et associez un [groupe de sécurité réseau](security-overview.md#network-security-groups) et que vous autorisez explicitement le trafic entrant prévu.

> [!NOTE]
> Seules les adresses IP publiques avec la référence SKU De base sont disponibles lorsque vous utilisez [le service de métadonnées d’instance (IMDS, instance metadata service)](../virtual-machines/windows/instance-metadata-service.md). La référence SKU Standard n’est pas prise en charge.

### <a name="basic"></a>De base

Toutes les adresses IP publiques créées avant l’introduction de références SKU sont des adresses IP publiques de référence SKU de base. 

Depuis l’introduction des références SKU, spécifiez quelle référence SKU attribuer à l’adresse IP publique. 

Les adresses de référence SKU de base :

- Sont assignées à l’aide de la méthode d’allocation statique ou dynamique.
- Dotées d’un délai d’inactivité du flux entrant réglable de 4 à 30 minutes, avec une valeur par défaut de 4 minutes et d’un délai d’inactivité du flux sortant fixe de 4 minutes.
- Ouvertes par défaut.  Il est recommandé d’utiliser des groupes de sécurité réseau, mais cela est facultatif pour restreindre le trafic entrant ou sortant.
- Affectées à n’importe quelle ressource Azure à laquelle une adresse IP publique peut être affectée, par exemple :
    * Interfaces réseau
    * Passerelles VPN
    * Passerelles d’application
    * Équilibreurs de charge publics
- Ne gèrent pas les scénarios de zone de disponibilité. Utilisent l’adresse IP publique de la référence SKU Standard pour les scénarios de zone de disponibilité. Pour en savoir plus sur les zones de disponibilité, consultez [Vue d’ensemble des zones de disponibilité](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et [Équilibreur de charge standard et zones de disponibilité](../load-balancer/load-balancer-standard-availability-zones.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="allocation-method"></a>Méthode d’allocation

Les adresses IP publiques de base et standard prennent en charge l’attribution **statique**.  Une adresse IP est attribuée à la ressource au moment de sa création. L’adresse IP est libérée lorsque la ressource est supprimée.

Les adresses IP publiques avec une référence de base prennent en charge une attribution **dynamique**. La méthode d’attribution par défaut est dynamique. Si vous sélectionnez la méthode dynamique, l’adresse IP **n’est pas** attribuée à la ressource au moment de sa création.

L’adresse IP est attribuée lorsque vous associez la ressource d’adresse IP publique à l’élément suivant :

* Machine virtuelle 
* La première machine virtuelle est associée au pool principal d’un équilibreur de charge.

L’adresse IP est libérée lorsque vous arrêtez (ou supprimez) la ressource.  

Par exemple, une ressource IP publique est libérée à partir d’une ressource nommée **Ressource A**. **Ressource A** reçoit une adresse IP différente au démarrage si la ressource IP publique est réaffectée.

L’adresse IP est libérée lorsque la méthode d’allocation passe de **statique** à **dynamique**. Pour vous assurer que l’adresse IP de la ressource associée ne change pas, définissez explicitement la méthode d’allocation sur **statique**. Une adresse IP statique est affectée immédiatement.

> [!NOTE]
> Même lorsque vous définissez la méthode d’allocation sur **statique**, vous ne pouvez pas spécifier l’adresse IP réelle affectée à la ressource IP publique. Azure assigne l’adresse IP à partir d’un pool d’adresses IP disponibles dans l’emplacement Azure dans lequel la ressource est créée.
>

Des adresses IP publiques statiques sont fréquemment utilisées dans les cas suivants :

* Lorsque vous devez mettre à jour les règles de pare-feu pour communiquer avec vos ressources Azure.
* La résolution de noms DNS est telle qu’une modification de l’adresse IP nécessiterait une mise à jour des enregistrements A.
* Vos ressources Azure communiquent avec d’autres applications ou services qui utilisent un modèle de sécurité basé sur une adresse IP.
* Vous utilisez des certificats TLS/SSL liés à une adresse IP.

> [!NOTE]
> Azure alloue des adresses IP publiques d’une plage unique à chaque région dans chaque Cloud Azure. Vous pouvez télécharger la liste des plages (préfixes) pour les clouds Azure [Public](https://www.microsoft.com/download/details.aspx?id=56519), [Gouvernement américain](https://www.microsoft.com/download/details.aspx?id=57063), [Chine](https://www.microsoft.com/download/details.aspx?id=57062), et [Allemagne](https://www.microsoft.com/download/details.aspx?id=57064).
>

## <a name="dns-hostname-resolution"></a>Résolution de nom d’hôte DNS

Sélectionnez l’option permettant de spécifier une étiquette de nom de domaine DNS pour une ressource d’adresse IP publique. 

Cette sélection crée un mappage pour **domainnamelabel**.**location**.cloudapp.azure.com sur l’adresse IP publique dans le DNS géré par Azure. 

Par exemple, la création d’une adresse IP publique avec :

* **contoso** comme **domainnamelabel**
* **West US** Azure **location**

Le nom de domaine complet (FQDN) **contoso.westus.cloudapp.azure.com** correspond à l’adresse IP publique de la ressource.

> [!IMPORTANT]
> Chaque étiquette de nom de domaine créée doit être unique dans son emplacement Azure.  
>

## <a name="dns-recommendations"></a>Recommandations DNS

Si un déplacement de région est nécessaire, vous ne pouvez pas migrer le nom de domaine complet de votre adresse IP publique. Utilisez le nom de domaine complet pour créer un enregistrement CNAME personnalisé qui pointe vers l’adresse IP publique. 

Si un déplacement vers une autre adresse IP publique est nécessaire, mettez à jour l’enregistrement CNAME au lieu de mettre à jour le nom de domaine complet.

Vous pouvez utiliser [Azure DNS](../dns/dns-custom-domain.md?toc=%2fazure%2fvirtual-network%2ftoc.json#public-ip-address) ou un fournisseur DNS externe pour votre enregistrement DNS. 

## <a name="virtual-machines"></a>Machines virtuelles

Vous pouvez associer une adresse IP publique à une machine virtuelle [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) en l’affectant à son **interface réseau**. 

Choisissez **dynamique** ou **statique** pour l’adresse IP publique. Apprenez-en davantage sur l’[assignation d’adresses IP à des interfaces réseau](virtual-network-network-interface-addresses.md).

## <a name="internet-facing-load-balancers"></a>Équilibreurs de charge accessibles sur Internet

Vous pouvez associer une adresse IP publique créée avec l’une des deux [références SKU](#sku) à un [équilibreur de charge Azure](../load-balancer/load-balancer-overview.md) en l’assignant à la configuration **frontale** de l’équilibreur de charge. L’adresse IP publique sert d’adresse IP virtuelle à équilibrage de charge. 

Vous pouvez affecter une adresse IP publique dynamique ou statique à un équilibreur de charge frontal. Vous pouvez affecter plusieurs adresses IP publiques à un équilibreur de charge frontal. Cette configuration permet des scénarios [à plusieurs adresses IP virtuelles](../load-balancer/load-balancer-multivip-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) comme un environnement mutualisé avec des sites Web basés sur TLS. 

Pour plus d’informations sur les références SKU de l’équilibreur de charge Azure, consultez [Référence SKU standard de l’équilibreur de charge Azure](../load-balancer/load-balancer-standard-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="vpn-gateways"></a>Passerelles VPN

La [passerelle VPN Azure](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) connecte un réseau virtuel Azure aux éléments suivants :

* Réseaux virtuels Azure
* Réseaux locaux. 

Une adresse IP publique est assignée à une passerelle VPN pour lui permettre de communiquer avec le réseau distant. Vous pouvez uniquement affecter une adresse IP publique de base *dynamique* à une passerelle VPN.

## <a name="application-gateways"></a>Passerelles d’application

Vous pouvez associer une adresse IP publique à une [Application Gateway](../application-gateway/application-gateway-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)Azure en l’affectant à la configuration **frontale** de la passerelle. 

* Affectez une adresse IP publique de base **dynamique** à une configuration frontale de passerelle d’application V1. 
* Affectez une adresse de référence SKU **statique** à une configuration frontale V2.

## <a name="azure-firewall"></a>Pare-feu Azure

[Pare-feu Azure](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) vous permet de créer, d’appliquer et de consigner des stratégies de connectivité réseau et d’application entre les abonnements et les réseaux virtuels.

Vous pouvez uniquement associer des IP publiques standard **statiques** à un pare-feu. Les pare-feu externes peuvent ainsi identifier le trafic provenant de votre réseau virtuel. 


## <a name="at-a-glance"></a>Aperçu

Le tableau ci-dessous présente la propriété par le biais de laquelle une adresse IP publique peut être associée à une ressource de niveau supérieur, ainsi que les méthodes d’allocation possibles.

| Ressources de niveau supérieur | Association d’adresse IP | Dynamique | statique |
| --- | --- | --- | --- |
| Machine virtuelle |interface réseau |Oui |Oui |
| Équilibreur de charge accessible sur Internet |Configuration frontale |Oui |Oui |
| passerelle VPN |Configuration IP de la passerelle |Oui |Non |
| passerelle d’application |Configuration frontale |Oui (V1 uniquement) |Oui (V2 uniquement) |
| Pare-feu Azure | Configuration frontale | Non | Oui|

## <a name="limits"></a>limites

Les limites imposées pour l’adressage IP sont indiquées dans l’ensemble des [limites pour la mise en réseau](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits) dans Azure. 

Ces limites sont exprimées par région et par abonnement. [Contactez le support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour augmenter les limites par défaut jusqu’aux limites maximum en fonction des besoins de votre entreprise.

## <a name="pricing"></a>Tarifs

Les adresses IP publiques peuvent avoir un coût nominal. Pour en savoir plus sur la tarification des adresses IP dans Azure, lisez la page [Tarification des adresses IP](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="next-steps"></a>Étapes suivantes
* Découvrir les [adresses IP privées dans Azure](private-ip-addresses.md)
* [Déployer une machine virtuelle avec une adresse IP publique statique à l’aide du portail Azure](virtual-network-deploy-static-pip-arm-portal.md)

