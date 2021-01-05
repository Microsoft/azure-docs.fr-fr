---
title: Fichier include
description: Fichier include
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 11/01/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: a665c3c27cca03a084896895ae51c56732ceb7b6
ms.sourcegitcommit: 877491bd46921c11dd478bd25fc718ceee2dcc08
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/02/2020
ms.locfileid: "84237669"
---
Lorsque vous créez une machine virtuelle Azure, vous devez créer un [réseau virtuel](../articles/virtual-network/virtual-networks-overview.md) ou en utiliser un existant. Vous devez également décider de la façon dont vos machines virtuelles doivent accéder au réseau virtuel. Il est essentiel de [planifier les choses avant de créer des ressources](../articles/virtual-network/virtual-network-vnet-plan-design-arm.md) et de s’assurer que vous connaissez les [limites des ressources réseau](../articles/azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

Dans l’illustration suivante, les machines virtuelles sont représentées en tant que serveurs web et serveurs de base de données. Chaque ensemble de machines virtuelles est attribué à différents sous-réseaux du réseau virtuel.

![réseau virtuel Azure](./media/virtual-machines-common-network-overview/vnetoverview.png)

Vous pouvez créer un réseau virtuel avant de créer une machine virtuelle ou bien le faire lors de la création d’une machine virtuelle. Vous créez ces ressources pour prendre en charge la communication avec une machine virtuelle :

- Interfaces réseau
- Adresses IP
- Réseau virtuel et sous-réseaux

En plus de ces ressources de base, vous devez également envisager d’utiliser les ressources facultatives suivantes :

- Groupes de sécurité réseau
- Équilibreurs de charge 

## <a name="network-interfaces"></a>Interfaces réseau

Une [carte d’interface réseau](../articles/virtual-network/virtual-network-network-interface.md) est l’interconnexion entre une machine virtuelle et un réseau virtuel. Une machine virtuelle doit posséder au moins une carte d’interface réseau. Plusieurs cartes d’interface réseau peuvent être nécessaires en fonction de la taille de la machine virtuelle que vous créez. Découvrez combien de cartes réseau chaque taille de machine virtuelle prend en charge pour [Windows](../articles/virtual-machines/windows/sizes.md) ou [Linux](../articles/virtual-machines/linux/sizes.md).

Vous pouvez créer une machine virtuelle avec plusieurs cartes réseau et ajouter ou supprimer des cartes réseau tout au long du cycle de vie d’une machine virtuelle. Une machine virtuelle dotée de plusieurs cartes réseau peut se connecter à différents sous-réseaux et envoyer ou recevoir le trafic sur l’interface la plus appropriée. Les machines virtuelles d’un même groupe à haute disponibilité peuvent comporter autant d’interfaces réseau que le permet leur taille. 

Chaque carte d’interface réseau attachée à une machine virtuelle doit se trouver dans le même emplacement et abonnement que la machine virtuelle. Chaque carte d’interface réseau doit être connectée à un réseau virtuel existant dans les mêmes emplacement et abonnement Azure que la carte d’interface réseau. Après la création d’une carte réseau, vous pouvez modifier le sous-réseau auquel une machine virtuelle est connectée, mais pas le réseau virtuel. Chaque carte d’interface réseau attachée à une machine virtuelle est attribuée à une adresse MAC qui ne change pas jusqu’à ce que la machine virtuelle soit supprimée.

Ce tableau répertorie les méthodes que vous pouvez utiliser pour créer une interface réseau.

| Méthode | Description |
| ------ | ----------- |
| Portail Azure | Lorsque vous créez une machine virtuelle dans le portail Azure, une interface réseau est automatiquement créée pour vous (vous ne pouvez pas utiliser une carte d’interface réseau que vous créez séparément). Le portail crée une machine virtuelle avec une seule carte d’interface réseau. Si vous souhaitez créer une machine virtuelle avec plusieurs cartes d’interface réseau, vous devez la créer en suivant une autre méthode. |
| [Azure PowerShell](../articles/virtual-machines/windows/multiple-nics.md) | Utilisez [New-AzNetworkInterface](https://docs.microsoft.com/powershell/module/az.network/new-aznetworkinterface) avec le paramètre **-PublicIpAddressId** pour fournir l’identificateur de l’adresse IP publique que vous avez créée précédemment. |
| [Azure CLI](../articles/virtual-machines/linux/multiple-nics.md) | Pour fournir l’identificateur de l’adresse IP publique que vous avez créée précédemment, utilisez [az network nic create](https://docs.microsoft.com/cli/azure/network/nic) avec le paramètre **--public-ip-address**. |
| [Modèle](../articles/virtual-network/template-samples.md) | Utilisez [Network Interface in a Virtual Network with Public IP Address](https://github.com/Azure/azure-quickstart-templates/tree/master/101-nic-publicip-dns-vnet) comme guide pour le déploiement d’une interface réseau à l’aide d’un modèle. |

## <a name="ip-addresses"></a>Adresses IP 

Vous pouvez attribuer ces types [d’adresses IP](../articles/virtual-network/public-ip-addresses.md) à une carte d’interface réseau dans Azure :

- **Adresses IP publiques** : utilisées pour la communication entrante et sortante (sans traduction d’adresses réseau) avec Internet et d’autres ressources Azure non connectées à un réseau virtuel. L’attribution d’une adresse IP publique à une carte d’interface réseau est facultative. Les adresses IP publiques ont un coût nominal et il existe une limite maximum d’adresses IP publiques pouvant être utilisées par abonnement.
- **Adresses IP privées** : utilisées pour la communication au sein d’un réseau virtuel, avec votre réseau local et Internet (sans traduction d’adresses réseau). Vous devez attribuer au moins une adresse IP privée à une machine virtuelle. Pour en savoir plus sur la traduction d’adresses réseau dans Azure, consultez [Understanding outbound connections in Azure](../articles/load-balancer/load-balancer-outbound-connections.md) (Comprendre les connexions sortantes dans Azure).

Vous pouvez attribuer des adresses IP publiques à des machines virtuelles ou des équilibrages de charge accessibles sur Internet. Vous pouvez attribuer des adresses IP privées à des machines virtuelles et des équilibrages de charge internes. Vous attribuez des adresses IP à une machine virtuelle à l’aide d’une interface réseau.

L’allocation d’une adresse IP à une ressource est possible à l’aide de deux méthodes : dynamique ou statique. La méthode d’allocation par défaut est dynamique. Dans ce cas, une adresse IP n’est pas allouée au moment de sa création. Au lieu de cela, l’adresse IP est allouée lorsque vous créez une machine virtuelle ou lorsque vous démarrez une machine virtuelle arrêtée. L’adresse IP est libérée lorsque vous arrêtez ou supprimez la machine virtuelle. 

Pour vous assurer que l’adresse IP de la machine virtuelle ne change pas, vous pouvez définir explicitement la méthode d’allocation sur statique. Dans ce cas, une adresse IP est attribuée immédiatement. Elle est libérée uniquement lorsque vous supprimez la machine virtuelle ou lorsque vous modifiez sa méthode d’allocation en dynamique.
    
Ce tableau répertorie les méthodes que vous pouvez utiliser pour créer une adresse IP.

| Méthode | Description |
| ------ | ----------- |
| [Azure portal](../articles/virtual-network/virtual-network-deploy-static-pip-arm-portal.md) | Les adresses IP publiques étant par défaut dynamiques, elles sont susceptibles de changer quand la machine virtuelle est supprimée ou arrêtée. Pour vous assurer que la machine virtuelle utilise toujours la même adresse IP publique, créez une adresse IP publique statique. Par défaut, le portail attribue une adresse IP privée dynamique à une carte d’interface réseau lors de la création d’une machine virtuelle. Vous pouvez modifier cette adresse IP statique après la création de la machine virtuelle.|
| [Azure PowerShell](../articles/virtual-network/virtual-network-deploy-static-pip-arm-ps.md) | Vous utilisez [New-AzPublicIpAddress](https://docs.microsoft.com/powershell/module/az.network/new-azpublicipaddress) avec le paramètre **-AllocationMethod** défini sur la valeur Dynamic ou Static. |
| [Azure CLI](../articles/virtual-network/virtual-network-deploy-static-pip-arm-cli.md) | Vous utilisez [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip) avec le paramètre **--allocation-method** défini sur la valeur Dynamic ou Static. |
| [Modèle](../articles/virtual-network/template-samples.md) | Utilisez [Network Interface in a Virtual Network with Public IP Address](https://github.com/Azure/azure-quickstart-templates/tree/master/101-nic-publicip-dns-vnet) comme guide pour le déploiement d’une adresse IP publique à l’aide d’un modèle. |

Après avoir créé une adresse IP publique, vous pouvez l’associer à une machine virtuelle en l’attribuant à une carte d’interface réseau.

## <a name="virtual-network-and-subnets"></a>Réseau virtuel et sous-réseaux

Un sous-réseau est une plage d’adresses IP dans le réseau virtuel. Vous pouvez diviser un réseau virtuel en plusieurs sous-réseaux pour plus de sécurité et une meilleure organisation. Chaque carte d’interface réseau est connectée à un sous-réseau dans un réseau virtuel. Les cartes d’interface réseau connectées aux sous-réseaux (identiques ou différents) au sein d’un réseau virtuel peuvent communiquer entre elles sans qu’il y ait besoin de configuration supplémentaire.

Lorsque vous configurez un réseau virtuel, vous spécifiez la topologie, y compris les sous-réseaux et les espaces d’adressage disponibles. Si le réseau virtuel doit être connecté à d’autres réseaux virtuels ou réseaux locaux, vous devez sélectionner des plages d’adresses qui ne se chevauchent pas. Les adresses IP sont privées et ne sont pas accessibles à partir d’Internet, ce qui était vrai uniquement pour les adresses IP non routables telles que 10.0.0.0/8, 172.16.0.0/12 ou 192.168.0.0/16. Désormais, Azure considère que toutes les plages d’adresses font partie de l’espace d’adressage IP du réseau virtuel privé qui est uniquement accessible dans le réseau virtuel, les réseaux virtuels interconnectés et depuis votre emplacement local. 

Si vous travaillez au sein d’une organisation dans laquelle une autre personne est responsable des réseaux internes, vous devez discuter avec elle avant de sélectionner votre espace d’adressage. Assurez-vous que les plages d’adresses ne se chevauchent pas et indiquez à cette personne l’espace que vous souhaitez utiliser afin qu’elle n’essaie pas d’utiliser la même plage d’adresses IP que vous. 

Par défaut, il n’existe aucune limite de sécurité entre les sous-réseaux. Ainsi, les machines virtuelles de chaque sous-réseau peuvent communiquer entre elles. Toutefois, vous pouvez configurer des groupes de sécurité réseau, qui vous permettent de contrôler le flux du trafic vers et depuis les sous-réseaux et les machines virtuelles. 

Ce tableau répertorie les méthodes que vous pouvez utiliser pour créer un réseau virtuel et des sous-réseaux.    

| Méthode | Description |
| ------ | ----------- |
| [Azure portal](../articles/virtual-network/quick-create-portal.md) | Si vous autorisez Azure à créer un réseau virtuel lors de la création d’une machine virtuelle, le nom attribué sera une combinaison du nom du groupe de ressources qui contient le réseau virtuel et de **-vnet**. L’espace d’adressage est 10.0.0.0/24, le nom du sous-réseau nécessaire est **default**, et la plage d’adresses de sous-réseau est 10.0.0.0/24. |
| [Azure PowerShell](../articles/virtual-network/quick-create-powershell.md) | Vous utilisez [New-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetworkSubnetConfig) et [New-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/new-azvirtualnetwork) pour créer un sous-réseau et un réseau virtuel. Vous pouvez également utiliser [Add-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/Az.Network/Add-AzVirtualNetworkSubnetConfig) pour ajouter un sous-réseau à un réseau virtuel existant. |
| [Azure CLI](../articles/virtual-network/quick-create-cli.md) | Le sous-réseau et le réseau virtuel sont créés en même temps. Fournissez un paramètre **--subnet-name** à [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet) avec le nom du sous-réseau. |
| Modèle | Le moyen le plus simple de créer un réseau virtuel et des sous-réseaux consiste à télécharger un modèle existant, tel que [Virtual Network with two Subnets](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vnet-two-subnets), et à le modifier en fonction de vos besoins. |

## <a name="network-security-groups"></a>Groupes de sécurité réseau

Un [groupe de sécurité réseau](../articles/virtual-network/virtual-network-vnet-plan-design-arm.md) contient une liste des règles de liste de contrôle d’accès qui autorise ou rejette le trafic réseau vers les sous-réseaux, les cartes d’interface réseau ou les deux. Des groupes de sécurité réseau peuvent être associés à des sous-réseaux ou à des cartes d’interface réseau connectées à un sous-réseau. Lorsqu’un groupe de sécurité réseau est associé à un sous-réseau, les règles de liste de contrôle d’accès s’appliquent à toutes les machines virtuelles présentes dans ce sous-réseau. En outre, le trafic vers une carte d’interface réseau peut être limité par l’association directe d’un groupe de sécurité réseau à une carte d’interface réseau.

Les groupes de sécurité réseau contiennent deux ensembles de règles : les règles de trafic entrant et les règles de trafic sortant. La priorité d’une règle doit être unique dans chaque ensemble. Chaque règle possède des propriétés relatives au protocole, aux plages de ports de destination et source, aux préfixes d’adresse, à la direction du trafic, à la priorité et au type d’accès. 

Tous les groupes de ressources réseau contiennent un ensemble de règles par défaut. Les règles par défaut ne peuvent pas être supprimées, mais comme la priorité la plus basse leur est attribuée, elles peuvent être remplacées par les règles que vous créez. 

 Lorsque vous associez un groupe de sécurité réseau à une carte réseau, les règles d’accès réseau du groupe de sécurité réseau sont appliquées uniquement à cette carte d’interface réseau. Si un groupe de sécurité réseau est appliqué à une seule carte d’interface réseau sur une machine virtuelle comprenant plusieurs cartes d’interface réseau, il n’affecte pas le trafic lié aux autres cartes d’interface réseau. Vous pouvez associer différents groupes de sécurité réseau à une carte d’interface réseau (ou une machine virtuelle, selon le modèle de déploiement) et au sous-réseau auquel une carte d’interface réseau ou une machine virtuelle est liée. La priorité est donnée en fonction de la direction du trafic.

Veillez à [planifier](../articles/virtual-network/virtual-network-vnet-plan-design-arm.md) vos groupes de sécurité réseau lorsque vous planifiez vos machines virtuelles et votre réseau virtuel.

Ce tableau répertorie les méthodes que vous pouvez utiliser pour créer un groupe de sécurité réseau.

| Méthode | Description |
| ------ | ----------- |
| [Azure portal](../articles/virtual-network/tutorial-filter-network-traffic.md) | Lorsque vous créez une machine virtuelle dans le portail Azure, un groupe de sécurité réseau est automatiquement créé et associé à la carte d’interface réseau créée par le portail. Le nom du groupe de sécurité réseau est une combinaison du nom de la machine virtuelle et de **-nsg**. Ce groupe de sécurité réseau contient une règle de trafic entrant avec une priorité de 1000, un service défini sur RDP, le protocole défini sur TCP, le port défini sur 3389 et l’action définie sur Allow. Si vous souhaitez autoriser tout autre trafic entrant vers la machine virtuelle, vous devez ajouter des règles supplémentaires pour le groupe de sécurité réseau. |
| [Azure PowerShell](../articles/virtual-network/tutorial-filter-network-traffic.md) | Utilisez [New-AzNetworkSecurityRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecurityruleconfig) et fournissez les informations de la règle requise. Utilisez [New-AzNetworkSecurityGroup](https://docs.microsoft.com/powershell/module/az.network/new-aznetworksecuritygroup) pour créer le groupe de sécurité réseau. Utilisez [Set-AzVirtualNetworkSubnetConfig](https://docs.microsoft.com/powershell/module/az.network/set-azvirtualnetworksubnetconfig) pour configurer le groupe de sécurité réseau du sous-réseau. Utilisez [Set-AzVirtualNetwork](https://docs.microsoft.com/powershell/module/az.network/set-azvirtualnetwork) pour ajouter le groupe de sécurité réseau au réseau virtuel. |
| [Azure CLI](../articles/virtual-network/tutorial-filter-network-traffic-cli.md) | Utilisez [az network nsg create](https://docs.microsoft.com/cli/azure/network/nsg) pour créer initialement le groupe de sécurité réseau. Utilisez [az network nsg rule create](https://docs.microsoft.com/cli/azure/network/nsg/rule) pour ajouter des règles à un groupe de sécurité réseau. Utilisez [az network vnet subnet update](https://docs.microsoft.com/cli/azure/network/vnet/subnet) pour ajouter le groupe de sécurité réseau au sous-réseau. |
| [Modèle](../articles/virtual-network/template-samples.md) | Utilisez [Create a Network Security Group](https://github.com/Azure/azure-quickstart-templates/tree/master/101-security-group-create) comme guide pour le déploiement d’un groupe de sécurité réseau à l’aide d’un modèle. |

## <a name="load-balancers"></a>Équilibreurs de charge

[Azure Load Balancer](../articles/load-balancer/load-balancer-overview.md) offre une haute disponibilité et des performances réseau élevées pour vos applications. Un équilibrage de charge peut être configuré pour [équilibrer le trafic Internet entrant](../articles/load-balancer/load-balancer-internet-overview.md) vers les machines virtuelles ou pour [équilibrer le trafic entre les machines virtuelles d’un réseau virtuel](../articles/load-balancer/load-balancer-internal-overview.md). Un équilibrage de charge peut également équilibrer le trafic entre les machines virtuelles et les ordinateurs locaux d’un réseau intersite ou transférer le trafic externe vers une machine virtuelle spécifique.

L’équilibrage de charge mappe le trafic entrant et le trafic sortant entre l’adresse IP publique et le port de l’équilibrage de charge, et l’adresse IP privée et le port de la machine virtuelle.

Lorsque vous créez un équilibrage de charge, vous devez également prendre en compte les éléments de configuration suivants :

- **Configuration d’une adresse IP frontale** : un équilibrage de charge peut inclure une ou plusieurs adresses IP frontales. Ces adresses IP servent d'entrée pour le trafic.
- **Pool d’adresses principal** : adresses IP qui sont associées à la carte d’interface réseau vers laquelle la charge est distribuée.
- **[Réacheminement de prot](../articles/load-balancer/tutorial-load-balancer-port-forwarding-portal.md)**  : définissent le trafic entrant qui transite via l’adresse IP frontale et qui est distribué à l’adresse IP principale utilisant des règles NAT de trafic entrant.
- **Règles d’équilibrage de charge** : mappent une combinaison donnée d’adresses IP et de ports frontaux vers un ensemble de combinaisons d’adresses IP et de port principaux. Un même équilibreur de charge peut avoir plusieurs règles d’équilibrage de charge. Chaque règle est une combinaison d’une adresse IP et d’un port frontaux et d’une adresse IP et d’un port principaux associés aux machines virtuelles.
- **[Sondes](../articles/load-balancer/load-balancer-custom-probe-overview.md)**  : surveillent l’intégrité des machines virtuelles. Lorsqu’une sonde ne répond pas, l’équilibrage de charge n’envoie plus de nouvelles connexions aux machines virtuelles défaillantes. Les connexions existantes ne sont pas affectées et les nouvelles connexions sont envoyées aux machines virtuelles saines.
- **[Règles de trafic sortant](../articles/load-balancer/load-balancer-outbound-rules-overview.md)** - Une règle de trafic sortant configure la traduction d’adresses réseau (NAT) du trafic sortant pour que toutes les machines virtuelles ou instances identifiées par le pool de back-ends de votre équilibreur de charge De base soient traduites en front-end.

Ce tableau répertorie les méthodes que vous pouvez utiliser pour créer un équilibrage de charge accessible sur Internet.

| Méthode | Description |
| ------ | ----------- |
| Portail Azure |  Vous pouvez [équilibrer la charge du trafic Internet sur les machines virtuelles avec le portail Azure](../articles/load-balancer/tutorial-load-balancer-standard-manage-portal.md). |
| [Azure PowerShell](/azure/load-balancer/load-balancer-get-started-ilb-arm-ps) | Utilisez [New-AzLoadBalancerFrontendIpConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerfrontendipconfig) avec le paramètre **-PublicIpAddress** pour fournir l’identificateur de l’adresse IP publique que vous avez créée précédemment. Utilisez [New-AzLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) pour créer la configuration du pool d’adresses principal. Utilisez [New-AzLoadBalancerInboundNatRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) pour créer des règles NAT de trafic entrant associées à la configuration IP frontale que vous avez créée. Utilisez [New-AzLoadBalancerProbeConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerprobeconfig) pour créer les sondes dont vous avez besoin. Utilisez [New-AzLoadBalancerRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerruleconfig) pour créer la configuration d’équilibrage de charge. Utilisez [New-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancer) pour créer l’équilibrage de charge.|
| [Azure CLI](../articles/load-balancer/load-balancer-get-started-internet-arm-cli.md) | Utilisez [az network lb create](https://docs.microsoft.com/cli/azure/network/lb) pour créer la configuration d’équilibrage de charge initiale. Utilisez [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip) pour ajouter l’adresse IP publique que vous avez créée précédemment. Utilisez [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool) pour ajouter la configuration du pool d’adresses principal. Utilisez [az network lb inbound-nat-rule create](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-rule) pour ajouter des règles de traduction d’adresses réseau. Utilisez [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule) pour ajouter les règles d’équilibrage de charge. Utilisez [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe) pour ajouter les sondes. |
| [Modèle](../articles/load-balancer/quickstart-load-balancer-standard-public-template.md) | Utilisez [2 VMs in a Load Balancer and configure NAT rules on the LB](https://github.com/Azure/azure-quickstart-templates/tree/master/101-load-balancer-standard-create) comme guide pour le déploiement d’un équilibrage de charge à l’aide d’un modèle. |
    
Ce tableau répertorie les méthodes que vous pouvez utiliser pour créer un équilibrage de charge interne.

| Méthode | Description |
| ------ | ----------- |
| Portail Azure | Vous pouvez [équilibrer la charge du trafic interne avec un équilibreur de charge sur le portail Azure](../articles/load-balancer/tutorial-load-balancer-standard-internal-portal.md). |
| [Azure PowerShell](../articles/load-balancer/load-balancer-get-started-ilb-arm-ps.md) | Pour fournir une adresse IP privée dans le sous-réseau du réseau, utilisez [New-AzLoadBalancerFrontendIpConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerfrontendipconfig) avec le paramètre **-PrivateIpAddress**. Utilisez [New-AzLoadBalancerBackendAddressPoolConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) pour créer la configuration du pool d’adresses principal. Utilisez [New-AzLoadBalancerInboundNatRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) pour créer des règles NAT de trafic entrant associées à la configuration IP frontale que vous avez créée. Utilisez [New-AzLoadBalancerProbeConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerprobeconfig) pour créer les sondes dont vous avez besoin. Utilisez [New-AzLoadBalancerRuleConfig](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancerruleconfig) pour créer la configuration d’équilibrage de charge. Utilisez [New-AzLoadBalancer](https://docs.microsoft.com/powershell/module/az.network/new-azloadbalancer) pour créer l’équilibrage de charge.|
| [Azure CLI](../articles/load-balancer/load-balancer-get-started-ilb-arm-cli.md) | Utilisez la commande [az network lb create](https://docs.microsoft.com/cli/azure/network/lb) pour créer la configuration d’équilibrage de charge initiale. Pour définir l’adresse IP privée, utilisez [az network lb frontend-ip create](https://docs.microsoft.com/cli/azure/network/lb/frontend-ip) avec le paramètre **--private-ip-address**. Utilisez [az network lb address-pool create](https://docs.microsoft.com/cli/azure/network/lb/address-pool) pour ajouter la configuration du pool d’adresses principal. Utilisez [az network lb inbound-nat-rule create](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-rule) pour ajouter des règles de traduction d’adresses réseau. Utilisez [az network lb rule create](https://docs.microsoft.com/cli/azure/network/lb/rule) pour ajouter les règles d’équilibrage de charge. Utilisez [az network lb probe create](https://docs.microsoft.com/cli/azure/network/lb/probe) pour ajouter les sondes.|
| [Modèle](../articles/load-balancer/load-balancer-get-started-ilb-arm-template.md) | Utilisez [2 VMs in a Load Balancer and configure NAT rules on the LB](https://github.com/Azure/azure-quickstart-templates/tree/master/201-2-vms-internal-load-balancer) comme guide pour le déploiement d’un équilibrage de charge à l’aide d’un modèle. |

### <a name="virtual-machine-scale-sets"></a>Groupes identiques de machines virtuelles 

Pour plus d’informations sur l’équilibreur de charge et les groupes de machines virtuelles identiques, consultez [Mise en réseau pour des groupes de machines virtuelles identiques Azure](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-networking).

## <a name="vms"></a>Machines virtuelles

Les machines virtuelles peuvent être créées dans le même réseau virtuel et se connecter les unes aux autres à l’aide d’adresses IP privées. Elles peuvent se connecter même si elles se trouvent dans des sous-réseaux différents, sans que vous ayez besoin de configurer de passerelle, ni d’utiliser des adresses IP publiques. Pour placer des machines virtuelles dans un réseau virtuel, vous créez le réseau virtuel, puis dès que vous créez une machine virtuelle, vous l’attribuez au réseau virtuel et au sous-réseau. Les machines virtuelles acquièrent leurs paramètres réseau lors du déploiement ou du démarrage.  

Les machines virtuelles sont attribuées à une adresse IP lorsqu’elles sont déployées. Si vous déployez plusieurs machines virtuelles dans un réseau virtuel ou un sous-réseau, des adresses IP leur sont attribuées lorsqu’elles démarrent. Vous pouvez également allouer une adresse IP statique à une machine virtuelle. Dans ce cas, utilisez éventuellement un sous-réseau spécifique afin d’éviter de réutiliser accidentellement l’adresse IP statique pour une autre machine virtuelle.  

Si vous créez une machine virtuelle et que vous voulez la migrer plus tard vers un réseau virtuel, il ne s’agit pas d’une modification de configuration simple. Vous devez redéployer la machine virtuelle dans le réseau virtuel. Pour la redéployer, le plus simple consiste à supprimer la machine virtuelle (mais pas les disques attachés à celle-ci), puis à recréer la machine virtuelle à l’aide des disques d’origine dans le réseau virtuel. 

Ce tableau répertorie les méthodes que vous pouvez utiliser pour créer une machine virtuelle dans un réseau virtuel.

| Méthode | Description |
| ------ | ----------- |
| [Azure portal](../articles/virtual-machines/windows/quick-create-portal.md) | Utilise les paramètres de réseau par défaut qui ont été mentionnés précédemment pour créer une machine virtuelle avec une seule carte d’interface réseau. Pour créer une machine virtuelle avec plusieurs cartes d’interface réseau, vous devez utiliser une autre méthode. |
| [Azure PowerShell](../articles/virtual-machines/windows/tutorial-manage-vm.md) | Inclut l’utilisation de [Add-AzVMNetworkInterface](https://docs.microsoft.com/powershell/module/az.compute/add-azvmnetworkinterface) pour ajouter la carte d’interface réseau que vous avez créée précédemment pour la configuration de la machine virtuelle. |
| [Azure CLI](../articles/virtual-machines/linux/create-cli-complete.md) | Créez et connectez une machine virtuelle à un réseau virtuel, un sous-réseau et une carte réseau selon différentes étapes. |
| [Modèle](../articles/virtual-machines/windows/ps-template.md) | Utilisez [Very simple deployment of a Windows VM](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows) comme guide pour le déploiement d’une machine virtuelle à l’aide d’un modèle. |

## <a name="next-steps"></a>Étapes suivantes
Pour connaître les étapes spécifiques aux machines virtuelles concernant la gestion des réseaux virtuels Azure pour les machines virtuelles, consultez les didacticiels [Windows](../articles/virtual-machines/windows/tutorial-virtual-network.md) ou [Linux](../articles/virtual-machines/linux/tutorial-virtual-network.md).

Il existe également des didacticiels sur l’équilibrage de charge des machines virtuelles et la création d’applications à haute disponibilité pour [Windows](../articles/virtual-machines/windows/tutorial-load-balancer.md) ou [Linux](../articles/virtual-machines/linux/tutorial-load-balancer.md).

- Découvrez comment configurer les [itinéraires définis par l’utilisateur et le transfert IP](../articles/virtual-network/virtual-networks-udr-overview.md). 
- Découvrez comment configurer [les connexions de réseau virtuel à réseau virtuel](../articles/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md).
- Découvrez comment [résoudre les problèmes relatifs aux itinéraires](../articles/virtual-network/diagnose-network-routing-problem.md).
- En savoir plus sur la [bande passante réseau des machines virtuelles](../articles/virtual-network/virtual-machine-network-throughput.md)
