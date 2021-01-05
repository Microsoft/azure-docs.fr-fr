---
title: Questions fréquentes (FAQ) sur le réseau virtuel Azure
titlesuffix: Azure Virtual Network
description: Réponses aux questions les plus fréquemment posées sur les réseaux virtuels Microsoft Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
ms.service: virtual-network
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/26/2020
ms.author: kumud
ms.openlocfilehash: 3ee9e165ce9c24968b072d19367e0285f5438259
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938798"
---
# <a name="azure-virtual-network-frequently-asked-questions-faq"></a>FAQ sur les réseaux virtuels Azure

## <a name="virtual-network-basics"></a>Concepts de base du réseau virtuel

### <a name="what-is-an-azure-virtual-network-vnet"></a>Qu’est un réseau virtuel (VNet) Azure ?
Un réseau virtuel (VNet) Azure est une représentation de votre propre réseau dans le cloud. Il s’agit d’un isolement logique du cloud Azure dédié à votre abonnement. Vous pouvez utiliser des réseaux virtuels pour configurer et gérer des réseaux privés virtuels (VPN) dans Azure et, éventuellement, lier les réseaux virtuels avec les autres réseaux virtuels dans Azure ou avec votre infrastructure informatique locale pour créer des solutions hybrides ou intersite. Chaque réseau virtuel que vous créez dispose de son propre bloc CIDR et peut être lié à d’autres réseaux virtuels et locaux, dans la mesure où les blocs CIDR ne se chevauchent pas. Vous pouvez également contrôler les paramètres de serveur DNS pour les réseaux virtuels et la segmentation du réseau virtuel en sous-réseaux.

Utilisez les réseaux virtuels pour effectuer les actions suivantes :

* Créez un réseau virtuel dédié uniquement au cloud privé. Vous n’avez pas toujours besoin d’une configuration intersite pour votre solution. Lorsque vous créez un réseau virtuel, vos services et les machines virtuelles au sein de votre réseau virtuel peuvent communiquer directement et en toute sécurité dans le cloud. Vous pouvez encore configurer des connexions au point de terminaison pour les machines virtuelles et les services qui requièrent une communication Internet dans le cadre de votre solution.

* Étendez votre centre de données en toute sécurité. Avec les réseaux virtuels, vous pouvez créer des VPN site à site (S2S) traditionnels pour faire évoluer en toute sécurité la capacité de votre centre de données. Les VPN S2S utilisent le protocole IPSEC pour fournir une connexion sécurisée entre votre passerelle VPN d’entreprise et Azure.

* Activez les scénarios de cloud hybride. Les réseaux virtuels vous donnent la possibilité de prendre en charge une variété de scénarios de nuage hybride. Vous pouvez connecter en toute sécurité des applications informatiques à n’importe quel type de système local, comme les ordinateurs centraux et les systèmes Unix.

### <a name="how-do-i-get-started"></a>Comment faire pour démarrer ?
Consultez la [Documentation Réseau virtuel](https://docs.microsoft.com/azure/virtual-network/) pour commencer. Ce document fournit des informations de présentation et de déploiement pour toutes les fonctionnalités du réseau virtuel.

### <a name="can-i-use-vnets-without-cross-premises-connectivity"></a>Puis-je utiliser des réseaux virtuels sans connectivité intersite ?
Oui. Vous pouvez utiliser un réseau virtuel sans connexion à votre site. Par exemple, vous pouvez exécuter des contrôleurs de domaine Microsoft Windows Server Active Directory et des batteries de serveurs SharePoint dans un réseau virtuel Azure.

### <a name="can-i-perform-wan-optimization-between-vnets-or-a-vnet-and-my-on-premises-data-center"></a>Puis-je exécuter une optimisation WAN entre des réseaux virtuels ou entre un réseau virtuel et mon centre de données local ?
Oui. Vous pouvez déployer une [appliance virtuelle réseau d’optimisation WAN](https://azuremarketplace.microsoft.com/en-us/marketplace/?term=wan%20optimization) à partir de plusieurs fournisseurs via Azure Marketplace.

## <a name="configuration"></a>Configuration

### <a name="what-tools-do-i-use-to-create-a-vnet"></a>Quels outils utiliser pour créer un réseau virtuel ?
Vous pouvez utiliser les outils suivants pour créer ou configurer un réseau virtuel :

* Portail Azure
* PowerShell
* Azure CLI
* Fichier de configuration réseau (netcfg - pour les réseaux virtuels classiques uniquement). Consultez l’article [Configurer un réseau virtuel à l’aide d’un fichier de configuration réseau](virtual-networks-using-network-configuration-file.md).

### <a name="what-address-ranges-can-i-use-in-my-vnets"></a>Quelles plages d’adresses puis-je utiliser dans mes réseaux virtuels ?
Nous vous recommandons d’utiliser les plages d’adresses énumérées dans [RFC 1918](https://tools.ietf.org/html/rfc1918), qui ont été mis de côté par l’IETF pour les espaces d’adressage privés et non routables :
* 10.0.0.0 - 10.255.255.255  (10/8 prefix)
* 172.16.0.0 - 172.31.255.255  (172.16/12 prefix)
* 192.168.0.0 - 192.168.255.255 (192.168/16 prefix)

D’autres espaces d’adressage peuvent fonctionner mais peuvent avoir des effets secondaires indésirables.

En outre, vous ne pouvez pas ajouter les plages d’adresses suivantes :
* 224.0.0.0/4 (multidiffusion)
* 255.255.255.255/32 (diffusion)
* 127.0.0.0/8 (bouclage)
* 169.254.0.0/16 (lien-local)
* 168.63.129.16/32 (DNS interne)

### <a name="can-i-have-public-ip-addresses-in-my-vnets"></a>Puis-je avoir des adresses IP publiques dans mes réseaux virtuels ?
Oui. Pour plus d’informations sur les plages d’adresses IP publiques, consultez [Create a virtual network](manage-virtual-network.md#create-a-virtual-network) (Créer un réseau virtuel). Les adresses IP publiques ne sont pas directement accessibles à partir d’Internet.

### <a name="is-there-a-limit-to-the-number-of-subnets-in-my-vnet"></a>Y a-t-il une limite au nombre de sous-réseaux dans mon réseau virtuel ?
Oui. Pour plus d’informations, consultez [Limites de mise en réseau](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits). Les espaces d’adressage de sous-réseau ne peuvent pas se chevaucher.

### <a name="are-there-any-restrictions-on-using-ip-addresses-within-these-subnets"></a>Existe-t-il des restrictions sur l’utilisation des adresses IP au sein de ces sous-réseaux ?
Oui. Azure réserve 5 adresses IP dans chaque sous-réseau. Il s’agit des adresses x.x.x.0-x.x.x.3 et de la dernière adresse du sous-réseau. Dans chaque sous-réseau, la plage x.x.x.1-x.x.x.3 est réservée aux services Azure.   
- x.x.x.0 : Adresse du réseau
- x.x.x.1 : Réservée par Azure pour la passerelle par défaut
- x.x.x.2, x.x.x.3 : Réservée par Azure pour mapper les adresses IP Azure DNS à l’espace du réseau virtuel
- x.x.x.255 : Adresse de diffusion réseau

### <a name="how-small-and-how-large-can-vnets-and-subnets-be"></a>Quelle taille peuvent avoir les réseaux virtuels et les sous-réseaux ?
Le plus petit sous-réseau IPv4 pris en charge est /29 et le plus grand est /8 (à l’aide des définitions de sous-réseaux CIDR).  La taille des sous-réseaux IPv6 doit être exactement de /64.  

### <a name="can-i-bring-my-vlans-to-azure-using-vnets"></a>Puis-je ajouter mes VLAN à Azure à l’aide de réseaux virtuels ?
Non. Les réseaux virtuels sont des superpositions de couche 3. Azure ne prend en charge aucune sémantique de couche 2.

### <a name="can-i-specify-custom-routing-policies-on-my-vnets-and-subnets"></a>Puis-je spécifier des stratégies de routage personnalisées sur des réseaux virtuels et des sous-réseaux ?
Oui. Vous pouvez créer une table de routage et l’associer à un sous-réseau. Pour plus d’informations sur le routage dans Azure, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md#custom-routes).

### <a name="do-vnets-support-multicast-or-broadcast"></a>Les réseaux virtuels prennent-ils en charge la multidiffusion ou la diffusion ?
Non. La multidiffusion et la diffusion ne sont pas prises en charge.

### <a name="what-protocols-can-i-use-within-vnets"></a>Quels protocoles puis-je utiliser au sein de réseaux virtuels ?
Vous pouvez utiliser les protocoles TCP, UDP et ICMP TCP/IP au sein des réseaux virtuels. La monodiffusion est prise en charge dans les réseaux virtuels, à l’exception du protocole DHCP (Dynamic Host Configuration Protocol) via la monodiffusion (port source UDP/68 / port de destination UDP/67) et du port UDP source 65330 qui est réservé à l’hôte. La multidiffusion, la diffusion, les paquets encapsulés IP dans IP et les paquets Encapsulation générique de routage (GRE, Generic Routing Encapsulation) sont bloqués dans les réseaux virtuels. 

### <a name="can-i-ping-my-default-routers-within-a-vnet"></a>Puis-je effectuer un test Ping sur mes routeurs par défaut au sein d’un réseau virtuel ?
Non.

### <a name="can-i-use-tracert-to-diagnose-connectivity"></a>Puis-je utiliser l’application tracert pour diagnostiquer la connectivité ?
Non.

### <a name="can-i-add-subnets-after-the-vnet-is-created"></a>Puis-je ajouter des sous-réseaux après avoir créé le réseau virtuel ?
Oui. Des sous-réseaux peuvent être ajoutés à des réseaux virtuels à tout moment, tant que la plage d’adresses de sous-réseau ne fait pas partie d’un autre sous-réseau et qu’il reste de l’espace dans la plage d’adresses du réseau virtuel.

### <a name="can-i-modify-the-size-of-my-subnet-after-i-create-it"></a>Puis-je modifier la taille de mon sous-réseau après sa création ?
Oui. Vous pouvez ajouter, supprimer, développer ou réduire un sous-réseau si aucune machine virtuelle ou aucun service n’y est déployé.

### <a name="can-i-modify-vnet-after-i-created-them"></a>Puis-je modifier un Vnet après sa création ?
Oui. Vous pouvez ajouter, supprimer et modifier les blocs CIDR utilisés par un réseau virtuel.

### <a name="if-i-am-running-my-services-in-a-vnet-can-i-connect-to-the-internet"></a>Si j’exécute mes services dans un réseau virtuel, puis-je me connecter à Internet ?
Oui. Tous les services déployés au sein d’un réseau virtuel peuvent être connectés en sortie à Internet. Pour en savoir plus sur les connexions Internet sortantes dans Azure, consultez [Connexions sortantes dans Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si vous souhaitez vous connecter en entrée à une ressource déployée par le biais de Resource Manager, la ressource doit avoir une adresse IP publique qui lui est affectée. Pour en savoir plus sur les adresses IP publiques, consultez [Adresses IP publiques](virtual-network-public-ip-address.md). Chaque service cloud Azure déployé dans Azure dispose d’une adresse IP virtuelle publiquement adressable qui lui est assignée. Vous définissez des points de terminaison d’entrée pour les points de terminaison et les rôles PaaS des machines virtuelles afin de permettre à ces services d’accepter les connexions à partir d’Internet.

### <a name="do-vnets-support-ipv6"></a>Les réseaux virtuels prennent-ils en charge IPv6 ?
Oui, les réseaux virtuels peuvent être IPv4 seulement ou à double pile (IPv4 + IPv6).  Pour plus d’informations, consultez [Aperçu du protocole IPv6 pour des réseaux virtuels Azure](./ipv6-overview.md).

### <a name="can-a-vnet-span-regions"></a>Un réseau virtuel peut-il couvrir plusieurs régions ?
Non. Un réseau virtuel est limité à une seule région. Un réseau virtuel peut toutefois couvrir des zones de disponibilité. Pour en savoir plus sur les zones de disponibilité, consultez [Vue d’ensemble de zones de disponibilité](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Vous pouvez connecter des réseaux virtuels dans différentes régions à l’aide d’un peering de réseaux virtuels. Pour plus d’informations, consultez [Peering de réseaux virtuels](virtual-network-peering-overview.md)

### <a name="can-i-connect-a-vnet-to-another-vnet-in-azure"></a>Puis-je connecter un réseau virtuel à un autre réseau virtuel dans Azure ?
Oui. Vous pouvez connecter un réseau virtuel à un autre réseau virtuel à l’aide des éléments suivants :
- **Peering de réseaux virtuels** : Pour plus d’informations, consultez [Présentation du peering de réseaux virtuels](virtual-network-peering-overview.md)
- **Une passerelle VPN Azure** : Pour en savoir plus, consultez [Configurer une connexion de réseau virtuel à réseau virtuel](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json). 

## <a name="name-resolution-dns"></a>Résolution de noms pour les machines virtuelles et les instances de rôle

### <a name="what-are-my-dns-options-for-vnets"></a>Quelles sont les options DNS pour les réseaux virtuels ?
Utilisez la table des décisions sur la page [Résolution de noms pour les machines virtuelles et les instances de rôle](virtual-networks-name-resolution-for-vms-and-role-instances.md) pour plus informations sur les options DNS disponibles.

### <a name="can-i-specify-dns-servers-for-a-vnet"></a>Puis-je spécifier des serveurs DNS pour un réseau virtuel ?
Oui. Vous pouvez spécifier des adresses IP de serveur DNS dans les paramètres de réseau virtuel. Le paramètre est appliqué en tant que serveur DNS par défaut pour toutes les machines virtuelles du réseau virtuel.

### <a name="how-many-dns-servers-can-i-specify"></a>Combien de serveurs DNS puis-je spécifier ?
Reportez-vous aux [limites de mise en réseau](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits).

### <a name="can-i-modify-my-dns-servers-after-i-have-created-the-network"></a>Puis-je modifier mes serveurs DNS une fois que le réseau créé ?
Oui. Vous pouvez modifier la liste des serveurs DNS de votre réseau virtuel à tout moment. Si vous modifiez votre liste de serveurs DNS, vous devez effectuer un renouvellement de bail DHCP sur toutes les machines virtuelles affectées dans le réseau virtuel, pour que les nouveaux paramètres DNS prennent effet. Pour les machines virtuelles exécutant le système d’exploitation Windows, vous pouvez taper `ipconfig /renew` directement sur la machine virtuelle. Pour les autres types de système d’exploitation, reportez-vous à la documentation de renouvellement du bail DHCP pour le type de système d’exploitation spécifique. 

### <a name="what-is-azure-provided-dns-and-does-it-work-with-vnets"></a>Qu’est ce qu’un serveur DNS fourni par Azure et fonctionne-t-il avec les réseaux virtuels ?
Le serveur DNS fourni par Azure est un serveur DNS mutualisé proposé par Microsoft. Azure enregistre toutes vos machines virtuelles et instances de rôle de service cloud dans ce service. Ce service fournit la résolution de noms par nom d’hôte pour les machines virtuelles et les instances de rôles contenues dans le même service cloud et par nom de domaine complet pour les machines virtuelles et les instances de rôle du même réseau virtuel. Pour en savoir plus sur DNS, consultez [Résolution de noms pour les machines virtuelles et les instances de rôle](virtual-networks-name-resolution-for-vms-and-role-instances.md).

Il existe une limitation aux 100 premiers services cloud du réseau virtuel pour la résolution de nom inter-clients à l’aide du serveur DNS fourni par Azure. Si vous utilisez votre propre serveur DNS, cette restriction ne s’applique pas.

### <a name="can-i-override-my-dns-settings-on-a-per-vm-or-cloud-service-basis"></a>Puis-je remplacer mes paramètres DNS sur une base machine virtuelle/service cloud ?
Oui. Vous pouvez configurer des serveurs DNS par machine virtuelle ou service cloud pour remplacer les paramètres réseau par défaut. Toutefois, il est recommandé d’utiliser autant que possible le DNS à l’échelle du réseau.

### <a name="can-i-bring-my-own-dns-suffix"></a>Puis-je afficher mon propre suffixe DNS ?
Non. Vous ne pouvez pas spécifier un suffixe DNS personnalisé pour vos réseaux virtuels.

## <a name="connecting-virtual-machines"></a>Connexion aux machines virtuelles

### <a name="can-i-deploy-vms-to-a-vnet"></a>Puis-je déployer des machines virtuelles sur un réseau virtuel ?
Oui. Toutes les interfaces réseau (NIC) attachées à une machine virtuelle déployée via le modèle de déploiement Resource Manager doivent être connectées à un réseau virtuel. Les machines virtuelles déployées via le modèle de déploiement classique peuvent éventuellement être connectées à un réseau virtuel.

### <a name="what-are-the-different-types-of-ip-addresses-i-can-assign-to-vms"></a>Quels sont les différents types d’adresses IP que je peux attribuer aux machines virtuelles ?
* **Privé :** attribué à chaque carte réseau sur chaque machine virtuelle. L’adresse est attribuée à l’aide de la méthode statique ou dynamique. Les adresses IP privées sont affectées à partir de la plage que vous avez spécifiée dans les paramètres de sous-réseau de votre réseau virtuel. Les ressources déployées par le biais du modèle de déploiement classique se voient attribuer des adresses IP privées, même si elles ne sont pas connectées à un réseau virtuel. Le comportement de la méthode d’allocation est différent selon qu’une ressource a été déployée avec le modèle de déploiement classique ou Resource Manager : 

  - **Resource Manager** : une adresse IP privée attribuée avec la méthode statique ou dynamique reste associée à une machine virtuelle (Resource Manager) jusqu’à ce que la ressource soit supprimée. La différence est que vous sélectionnez l’adresse à attribuer lors de l’utilisation d’une méthode statique, et Azure choisit quand utiliser la méthode dynamique. 
  - **Classique** : une adresse IP privée attribuée avec la méthode dynamique peut changer lorsqu’une machine virtuelle (classique) est redémarrée après avoir été arrêtée (désallouée). Si vous devez vous assurer que l’adresse IP privée d’une ressource déployée via le modèle de déploiement classique ne change jamais, attribuez une adresse IP privée avec la méthode statique.

* **Public :** attribué (facultatif) aux cartes réseau attachées aux machines virtuelles déployées via le modèle de déploiement Azure Resource Manager. L’adresse peut être attribuée à l’aide de la méthode d’allocation statique ou dynamique. Toutes les machines virtuelles et instances de rôle de services cloud déployées via le modèle de déploiement classique existent au sein d’un service cloud, qui se voit attribuer une adresse IP virtuelle publique *dynamique*. Une adresse IP *statique* publique, appelée [Adresse IP réservée](virtual-networks-reserved-public-ip.md), peut éventuellement être attribuée en tant qu’adresse IP virtuelle. Vous pouvez attribuer des adresses IP publiques à des machines virtuelles ou instances de rôle de services cloud individuelles déployées via le modèle de déploiement classique. Ces adresses sont appelées [Adresses IP publiques de niveau d’instance (ILPIP)](virtual-networks-instance-level-public-ip.md) et elles peuvent être attribuées de manière dynamique.

### <a name="can-i-reserve-a-private-ip-address-for-a-vm-that-i-will-create-at-a-later-time"></a>Puis-je réserver une adresse IP privée pour une machine virtuelle que je créerai ultérieurement ?
Non. Vous ne pouvez pas réserver d’adresse IP privée. Si une adresse IP privée est disponible, elle est affectée à une machine virtuelle ou à une instance de rôle par le serveur DHCP. La machine virtuelle peut être celle à laquelle vous souhaitez attribuer l’adresse IP privée. Vous pouvez toutefois modifier l’adresse IP privée d’une machine virtuelle déjà créée et utiliser n’importe quelle adresse IP privée disponible.

### <a name="do-private-ip-addresses-change-for-vms-in-a-vnet"></a>Les adresses IP privées peuvent-elles changer pour les machines virtuelles d’un réseau virtuel ?
Cela dépend. Si la machine virtuelle a été déployée via Resource Manager, non, et ce, que l’adresse IP ait été attribuée avec la méthode d’allocation statique ou dynamique. Si la machine virtuelle a été déployée par le biais du modèle de déploiement classique, les adresses IP dynamiques peuvent changer quand une machine virtuelle est démarrée après avoir été arrêtée (désallouée). L’adresse est libérée d’une machine virtuelle déployée via un modèle de déploiement lorsque la machine virtuelle est supprimée.

### <a name="can-i-manually-assign-ip-addresses-to-nics-within-the-vm-operating-system"></a>Puis-je attribuer manuellement des adresses IP aux cartes réseau au sein du système d’exploitation de la machine virtuelle ?
Oui, mais cela n’est pas recommandé, sauf si nécessaire, par exemple lors de l’affectation de plusieurs adresses IP à une machine virtuelle. Pour plus d’informations, consultez [Ajouter des adresses IP à une machine virtuelle](virtual-network-multiple-ip-addresses-portal.md#os-config). Si l’adresse IP attribuée à une carte réseau Azure jointe à une machine virtuelle fait l’objet de modifications et si l’adresse IP au sein du système d’exploitation de la machine virtuelle est différente, vous perdez la connectivité à la machine virtuelle.

### <a name="if-i-stop-a-cloud-service-deployment-slot-or-shutdown-a-vm-from-within-the-operating-system-what-happens-to-my-ip-addresses"></a>Si j’arrête un emplacement de déploiement de service cloud ou si j’arrête une machine virtuelle à partir du système d’exploitation, que se passe-t-il pour mes adresses IP ?
Nothing. Les adresses IP (adresse IP virtuelle publique, publique et privée) restent affectées à la machine virtuelle ou à l’emplacement de déploiement de services cloud.

### <a name="can-i-move-vms-from-one-subnet-to-another-subnet-in-a-vnet-without-redeploying"></a>Puis-je déplacer des machines virtuelles d’un sous-réseau à un autre dans un réseau virtuel sans redéploiement ?
Oui. Vous trouverez plus d’informations dans l’article [Déplacement d’une machine virtuelle ou d’une instance de rôle vers un autre sous-réseau](virtual-networks-move-vm-role-to-subnet.md).

### <a name="can-i-configure-a-static-mac-address-for-my-vm"></a>Puis-je configurer une adresse MAC statique pour ma machine virtuelle ?
Non. Une adresse MAC ne peut pas être configurée de manière statique.

### <a name="will-the-mac-address-remain-the-same-for-my-vm-once-its-created"></a>Une fois créée, l’adresse MAC restera-t-elle la même pour ma machine virtuelle ?
Oui, l’adresse MAC reste la même pour une machine virtuelle déployée via les modèles de déploiement Resource Manager et classique jusqu’à sa suppression. Auparavant, l’adresse MAC était libérée si la machine virtuelle était arrêtée (libérée), mais désormais l’adresse MAC est conservée même lorsque la machine virtuelle est dans l’état libéré. L’adresse MAC reste assignée à l’interface réseau jusqu’à ce que l’interface réseau soit supprimée ou que l’adresse IP privée assignée à la configuration IP principale de l’interface réseau principale soit modifiée. 

### <a name="can-i-connect-to-the-internet-from-a-vm-in-a-vnet"></a>Puis-je me connecter à Internet à partir d’une machine virtuelle dans un réseau virtuel ?
Oui. Toutes les machines virtuelles et instances de rôle de services cloud déployés au sein d’un réseau virtuel peuvent se connecter à Internet.

## <a name="azure-services-that-connect-to-vnets"></a>Services Azure qui se connectent à des réseaux virtuels

### <a name="can-i-use-azure-app-service-web-apps-with-a-vnet"></a>Puis-je utiliser Azure App Service Web Apps avec un réseau virtuel ?
Oui. Vous pouvez déployer des applications Web à l’intérieur d’un réseau virtuel à l’aide d’un environnement ASE (App Service Environment). Connectez le serveur principal de vos applications à vos réseaux virtuels avec la fonction d’intégration au réseau virtuel, puis verrouillez le trafic entrant vers votre application avec les points de terminaison de service. Pour plus d’informations, consultez les articles suivants :

* [Fonctionnalités de mise en réseau App Service](../app-service/networking-features.md)
* [Création d'applications web dans un environnement App Service](../app-service/environment/app-service-web-how-to-create-a-web-app-in-an-ase.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
* [Intégrer une application à un réseau virtuel Azure](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
* [App Service access restriction](../app-service/app-service-ip-restrictions.md) (Restrictions d’accès dans App Service)

### <a name="can-i-deploy-cloud-services-with-web-and-worker-roles-paas-in-a-vnet"></a>Puis-je déployer des services cloud avec les rôles web et de travail (PaaS) dans un réseau virtuel ?
Oui. Vous pouvez déployer (facultatif) des instances de rôle de services cloud dans des réseaux virtuels. Pour cela, vous spécifiez le nom de réseau virtuel et les mappages rôle/sous-réseau dans la section de configuration réseau de votre configuration de service. Il est inutile de mettre à jour vos fichiers binaires.

### <a name="can-i-connect-a-virtual-machine-scale-set-to-a-vnet"></a>Puis-je connecter un groupe de machines virtuelles identiques à un réseau virtuel ?
Oui. Vous devez connecter un groupe de machines virtuelles identiques à un réseau virtuel.

### <a name="is-there-a-complete-list-of-azure-services-that-can-i-deploy-resources-from-into-a-vnet"></a>Existe-t-il une liste complète des services Azure à partir desquels je peux déployer des ressources dans un réseau virtuel ?
Oui. Pour plus d’informations, consultez [Intégration d’un réseau virtuel pour les services Azure](virtual-network-for-azure-services.md).

### <a name="how-can-i-restrict-access-to-azure-paas-resources-from-a-vnet"></a>Comment puis-je restreindre l’accès à des ressources PaaS Azure depuis un réseau virtuel ?

Les ressources déployées via certains services PaaS Azure (comme Stockage Azure et Azure SQL Database) peuvent restreindre l’accès réseau à un réseau virtuel via l’utilisation de points de terminaison de service de réseau virtuel ou de Liaison privée Azure. Pour plus d’informations, consultez [Vue d’ensemble des points de terminaison de service de réseau virtuel](virtual-network-service-endpoints-overview.md), [Vue d’ensemble de Liaison privée Azure](../private-link/private-link-overview.md).

### <a name="can-i-move-my-services-in-and-out-of-vnets"></a>Puis-je faire entrer ou sortir mes services dans les réseaux virtuels ?
Non. Impossible de faire entrer ou de faire sortir des services dans les réseaux virtuels. Pour déplacer une ressource vers un autre réseau virtuel, vous devez supprimer et redéployer la ressource.

## <a name="security"></a>Sécurité

### <a name="what-is-the-security-model-for-vnets"></a>Quel est le modèle de sécurité pour les réseaux virtuels ?
Les réseaux virtuels sont isolés les uns des autres, ainsi que des autres services hébergés dans l’infrastructure Azure. Un réseau virtuel est une délimitation d’approbation.

### <a name="can-i-restrict-inbound-or-outbound-traffic-flow-to-vnet-connected-resources"></a>Puis-je limiter le flux de trafic entrant ou sortant aux ressources connectées au réseau virtuel ?
Oui. Vous pouvez appliquer des [groupes de sécurité réseau](security-overview.md) à des sous-réseaux individuels d’un réseau virtuel, à des cartes réseau attachées à un réseau virtuel, ou les deux.

### <a name="can-i-implement-a-firewall-between-vnet-connected-resources"></a>Puis-je mettre en place un pare-feu entre les ressources connectées au réseau virtuel ?
Oui. Vous pouvez déployer une [appliance virtuelle réseau de pare-feu](https://azure.microsoft.com/marketplace/?term=firewall) à partir de plusieurs fournisseurs via Azure Marketplace.

### <a name="is-there-information-available-about-securing-vnets"></a>Existe-t-il des informations sur la sécurisation des réseaux virtuels ?
Oui. Pour plus d’informations, consultez [Présentation de la sécurité réseau Azure](../security/fundamentals/network-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

### <a name="do-virtual-networks-store-customer-data"></a>Les réseaux virtuels stockent-ils des données client ?
Non. Les réseaux virtuels ne stockent aucune donnée client. 

## <a name="apis-schemas-and-tools"></a>API, schémas et outils

### <a name="can-i-manage-vnets-from-code"></a>Puis-je gérer les réseaux virtuels à partir du code ?
Oui. Vous pouvez utiliser des API REST pour les réseaux virtuels dans les modèles de déploiement [Azure Resource Manager](/rest/api/virtual-network) et [classique](https://go.microsoft.com/fwlink/?LinkId=296833).

### <a name="is-there-tooling-support-for-vnets"></a>Existe-t-il une prise en charge des outils pour les réseaux virtuels ?
Oui. En savoir plus sur l’utilisation des éléments suivants :
- Le portail Azure pour déployer des réseaux virtuels via les modèles de déploiement [Azure Resource Manager](manage-virtual-network.md#create-a-virtual-network) et [classique](virtual-networks-create-vnet-classic-pportal.md).
- PowerShell pour gérer les réseaux virtuels déployés via les modèles de déploiement [Resource Manager](/powershell/module/az.network) et [classique](/powershell/module/servicemanagement/azure.service/?view=azuresmps-3.7.0).
- L’interface de ligne de commande Azure pour déployer et gérer les réseaux virtuels déployés via les modèles de déploiement [Resource Manager](/cli/azure/network/vnet) et [classique](../virtual-machines/azure-cli-arm-commands.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-resources).  

## <a name="vnet-peering"></a>Peering de réseaux virtuels

### <a name="what-is-vnet-peering"></a>En quoi consiste le peering VNet ?
Le peering VNet (ou peering de réseau virtuel) permet de connecter des réseaux virtuels. Une connexion de peering VNet entre réseaux virtuels vous permet d’acheminer le trafic entre eux de façon privée par le biais d’adresses IPv4. Les machines virtuelles dans les réseaux virtuels appairés peuvent communiquer entre elles comme si elles se trouvaient dans le même réseau. Ces réseaux virtuels peuvent se situer dans la même région ou dans des régions différentes (Global VNet Peering). Il est également possible de créer des connexions de peering VNet dans les abonnements Azure.

### <a name="can-i-create-a-peering-connection-to-a-vnet-in-a-different-region"></a>Puis-je créer une connexion de peering pour un réseau virtuel dans une autre région ?
Oui. Global VNet Peering vous permet d’homologuer des réseaux virtuels dans différentes régions. Global VNet Peering est disponible dans toutes les régions publiques Azure, dans les régions cloud de Chine et dans les régions cloud Government. Vous ne pouvez pas procéder au peering mondial à partir de régions publiques Azure vers des régions cloud nationales.

### <a name="what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers"></a>Quelles sont les contraintes liées aux équilibreurs de charge et à Global VNet Peering ?
Si les deux réseaux virtuels dans deux régions différentes sont associés sur le Peering de réseaux virtuels globaux, vous ne pouvez pas vous connecter aux ressources qui se trouvent derrière un Load Balancer de base par le biais de l’adresse IP frontale du Load Balancer. Cette restriction n’existe pas pour un Standard Load Balancer.
Les ressources suivantes peuvent utiliser des Load Balancers de base, ce qui signifie que vous ne pouvez pas les atteindre via l’adresse IP frontale du Load Balancer sur le peering de réseaux virtuels globaux. Toutefois, vous pouvez utiliser le peering de réseaux virtuels globaux pour atteindre les ressources directement par le biais de leurs adresses IP de réseau virtuel privé, si cela est autorisé. 
- Machines virtuelles sur lesquelles reposent les équilibreurs de charge de base
- Groupes de machines virtuelles identiques avec des équilibreurs de charge de base 
- Cache Redis 
- Référence SKU Application Gateway (v1)
- Service Fabric
- Gestion des API
- Active Directory Domain Services (ADDS)
- Logic Apps
- HDInsight
-   Azure Batch
- Environnement App Service

Vous pouvez vous connecter à ces ressources via ExpressRoute ou une connexion entre deux réseaux virtuels, par l’intermédiaire de passerelles de réseau virtuel.

### <a name="can-i-enable-vnet-peering-if-my-virtual-networks-belong-to-subscriptions-within-different-azure-active-directory-tenants"></a>Puis-je activer le peering de réseau virtuel si mes réseaux virtuels font partie d’abonnements de différents locataires Azure Active Directory ?
Oui. Il est possible d’établir un peering de réseau virtuel (local ou global) si vos abonnements appartiennent à différents locataires Azure Active Directory. Vous pouvez faire cela via le portail, PowerShell ou Azure CLI.

### <a name="my-vnet-peering-connection-is-in-initiated-state-why-cant-i-connect"></a>Ma connexion de peering de réseau est à l’état *initié*, pourquoi ne puis-je pas me connecter ?
Si votre connexion de peering est dans un état *Initiée*, cela signifie que vous n’avez créé qu’un seul lien. Un lien bidirectionnel doit être créé afin d’établir une connexion réussie. Par exemple, pour homologuer le réseau virtuel A au réseau virtuel B, un lien doit être créé de VNetA à VNetB et de VNetB à VNetA. La création des deux liens modifie l’état à *Connecté*.

### <a name="my-vnet-peering-connection-is-in-disconnected-state-why-cant-i-create-a-peering-connection"></a>Ma connexion de peering VNet se trouve dans l’état *Déconnecté*, pourquoi ne puis-je pas créer une connexion de peering ?
Si votre connexion de peering VNet se trouve dans un état *Déconnecté*, cela signifie qu’un des liens créés a été supprimé. Pour rétablir une connexion de peering, vous devrez supprimer le lien et le créer de nouveau.

### <a name="can-i-peer-my-vnet-with-a-vnet-in-a-different-subscription"></a>Puis-je homologuer mon réseau virtuel avec un réseau virtuel dans un autre abonnement ?
Oui. Vous pouvez homologuer des réseaux virtuels entre des abonnements et régions.

### <a name="can-i-peer-two-vnets-with-matching-or-overlapping-address-ranges"></a>Puis-je homologuer deux réseaux virtuels avec des plages d’adresses correspondantes ou se chevauchant ?
Non. Les espaces d’adresses ne doivent pas se chevaucher pour pouvoir activer le peering de réseau virtuel.

### <a name="can-i-peer-a-vnet-to-two-different-vnets-with-the-the-use-remote-gateway-option-enabled-on-both-the-peerings"></a>Puis-je appairer un réseau virtuel à deux réseaux virtuels différents avec l’option « Utiliser la passerelle à distance » activée sur les deux peerings ?
Non. Vous ne pouvez activer l’option « Utiliser la passerelle à distance » que sur un peering avec l’un des réseaux virtuels.

### <a name="how-much-do-vnet-peering-links-cost"></a>Combien coûtent les liens de peering de réseau virtuel ?
Il n’existe aucun frais pour créer une connexion de peering de réseau virtuel. Le transfert de données entre des connexions de peering est facturé. [Voir ici](https://azure.microsoft.com/pricing/details/virtual-network/).

### <a name="is-vnet-peering-traffic-encrypted"></a>Le trafic de peering de réseau virtuel est-il chiffré ?
Lorsque le trafic Azure se déplace entre des centres de données (hors limites physiques non contrôlées par Microsoft ou pour le compte de Microsoft), [le chiffrement de la couche de liaison de données MACsec](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit) est utilisé sur le matériel réseau sous-jacent.  Cela s’applique au trafic de peering de réseaux virtuels.

### <a name="why-is-my-peering-connection-in-a-disconnected-state"></a>Pourquoi ma connexion de peering est-elle dans un état *Déconnecté* ?
Les connexions de peering de réseau virtuel passent à l’état *Déconnecté* lorsqu’un lien de peering de réseau virtuel est supprimé. Vous devez supprimer les deux liens pour pouvoir rétablir une connexion de peering.

### <a name="if-i-peer-vneta-to-vnetb-and-i-peer-vnetb-to-vnetc-does-that-mean-vneta-and-vnetc-are-peered"></a>Si j’effectue une homologation entre VNetA et VNetB, et que je dois également le faire entre VNetB et VNetC, cela signifie-il que VNetA et VNetC sont homologués ?
Non. Le peering transitif n’est pas pris en charge. Pour qu’ils le soient, vous devez homologuer VNetA et VNetC.

### <a name="are-there-any-bandwidth-limitations-for-peering-connections"></a>Des restrictions de bande passante s’appliquent-elles aux connexions de peering ?
Non. Le peering de réseau virtuel, qu’il soit local ou global, n’impose aucune restriction de bande passante. La bande passante n’est limitée que par les ressources de la machine virtuelle ou de calcul.

### <a name="how-can-i-troubleshoot-vnet-peering-issues"></a>Comment résoudre les problèmes liés à VNet Peering ?
Voici un [guide sur un utilitaire de résolution des problèmes](https://support.microsoft.com/en-us/help/4486956/troubleshooter-for-virtual-network-peering-issues) qui peut vous y aider.

## <a name="virtual-network-tap"></a>TAP de réseau virtuel

### <a name="which-azure-regions-are-available-for-virtual-network-tap"></a>Quelles régions Azure sont disponibles pour le TAP de réseau virtuel ?
La préversion du TAP de réseau virtuel est disponible dans toutes les régions Azure. Les interfaces réseau supervisées, la ressource TAP de réseau virtuel, ainsi que la solution du collecteur ou d’analyse doivent être déployées dans la même région.

### <a name="does-virtual-network-tap-support-any-filtering-capabilities-on-the-mirrored-packets"></a>Le TAP de réseau virtuel prend-il en charge des fonctionnalités de filtrage sur les paquets mis en miroir ?
Les fonctionnalités de filtrage ne sont pas prises en charge avec la préversion du TAP de réseau virtuel. Quand une configuration TAP est ajoutée à une interface réseau, une copie complète de tout le trafic entrant et sortant sur l’interface réseau est diffusée en continu vers la destination TAP.

### <a name="can-multiple-tap-configurations-be-added-to-a-monitored-network-interface"></a>Est-il possible d’ajouter plusieurs configurations TAP à une interface réseau supervisée ?
Une interface réseau supervisée ne peut avoir qu’une seule configuration TAP. Vérifiez auprès de la [solution de partenaire](virtual-network-tap-overview.md#virtual-network-tap-partner-solutions) individuelle si elle dispose de la capacité à diffuser en continu plusieurs copies du trafic TAP vers les outils d’analytique de votre choix.

### <a name="can-the-same-virtual-network-tap-resource-aggregate-traffic-from-monitored-network-interfaces-in-more-than-one-virtual-network"></a>La même ressource TAP de réseau virtuel peut-elle agréger le trafic en provenance d’interfaces réseau supervisées dans plusieurs réseaux virtuels ?
Oui. La même ressource TAP de réseau virtuel peut être utilisée pour agréger le trafic mis en miroir en provenance d’interfaces réseau supervisées dans des réseaux virtuels appairés du même abonnement ou d’un autre abonnement. La ressource TAP de réseau virtuel et l’équilibreur de charge de destination ou l’interface réseau de destination doivent se trouver dans le même abonnement. Tous les abonnements doivent se trouver sous le même locataire Azure Active Directory.

### <a name="are-there-any-performance-considerations-on-production-traffic-if-i-enable-a-virtual-network-tap-configuration-on-a-network-interface"></a>Existe-t-il des considérations relatives aux performances sur le trafic de production si j’active une configuration TAP de réseau virtuel sur une interface réseau ?

Le TAP de réseau virtuel est disponible en préversion. Pendant la période de préversion, il n’existe aucun contrat de niveau de service. La fonctionnalité ne doit pas être utilisée pour des charges de travail de production. Quand une interface réseau de machine virtuelle est activée avec une configuration TAP, les mêmes ressources sur l’hôte Azure allouées à la machine virtuelle pour envoyer le trafic de production sont utilisées pour exécuter la fonction de mise en miroir et envoyer les paquets mis en miroir. Sélectionnez la taille de machine virtuelle [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Windows](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) appropriée pour garantir que suffisamment de ressources sont disponibles pour que la machine virtuelle envoie le trafic de production et le trafic mis en miroir.

### <a name="is-accelerated-networking-for-linux-or-windows-supported-with-virtual-network-tap"></a>La mise en réseau accélérée pour [Linux](create-vm-accelerated-networking-cli.md) ou [Windows](create-vm-accelerated-networking-powershell.md) est-elle prise en charge avec le TAP de réseau virtuel ?

Vous pourrez ajouter une configuration TAP sur une interface réseau attachée à une machine virtuelle activée avec la mise en réseau accélérée. Mais l’ajout de la configuration TAP affecte les performances et la latence sur la machine virtuelle, car le déchargement pour la mise en miroir du trafic n’est actuellement pas prise en charge par la mise en réseau accélérée Azure.

## <a name="virtual-network-service-endpoints"></a>Points de terminaison de service de réseau virtuel

### <a name="what-is-the-right-sequence-of-operations-to-set-up-service-endpoints-to-an-azure-service"></a>Dans quel ordre faut-il effectuer les opérations pour configurer les points de terminaison d’un service Azure ?
La sécurisation d’une ressource de service Azure avec des points de terminaison de service s’effectue en deux étapes :
1. Activez les points de terminaison du service Azure.
2. Configurez les ACL de réseau virtuel sur le service Azure.

La première étape est une opération qui s’effectue côté réseau et la deuxième côté ressources de service. Ces deux étapes peuvent être effectuées par le même administrateur ou par des administrateurs différents en fonction des autorisations Azure RBAC accordées au rôle d’administrateur. Nous vous recommandons d’activer les points de terminaison de service pour votre réseau virtuel avant de configurer les ACL du réseau virtuel côté service Azure. Ces étapes doivent donc être effectuées dans l’ordre indiqué ci-dessus pour configurer les points de terminaison de service de réseau virtuel.

>[!NOTE]
> Vous ne pouvez pas limiter l’accès au service Azure au réseau virtuel et sous-réseau autorisés avant d’avoir effectué les deux opérations ci-dessus. La simple activation des points de terminaison du service Azure côté réseau ne vous permet pas de définir un accès limité. En outre, vous devez également configurer les ACL du réseau virtuel côté service Azure.

Certains services (comme SQL et CosmosDB) autorisent des exceptions à la séquence ci-dessus avec l’indicateur **IgnoreMissingVnetServiceEndpoint**. Une fois l’indicateur défini sur **True**, les ACL du réseau virtuel peuvent être définies côté service Azure avant la configuration des points de terminaison de service côté réseau. Les services Azure fournissent cet indicateur pour aider les clients lorsque des pare-feu IP spécifiques sont configurés sur les services Azure et que l’activation des points de terminaison de service côté réseau peut entraîner une baisse de connectivité, car l’adresse IP source passe d’une adresse IPv4 publique à une adresse privée. La configuration des ACL du réseau virtuel sur le service Azure avant la définition des points de terminaison de service côté réseau peut éviter une baisse de connectivité.

### <a name="do-all-azure-services-reside-in-the-azure-virtual-network-provided-by-the-customer-how-does-vnet-service-endpoint-work-with-azure-services"></a>Tous les services Azure résident-ils sur le réseau virtuel Azure fourni par le client ? Comment le point de terminaison de service du réseau virtuel fonctionne-t-il avec les services Azure ?

Non, tous les services Azure ne résident pas sur le réseau virtuel du client. La majorité des services de données Azure comme le Stockage Azure, Azure SQL et Azure Cosmos DB sont des services mutualisés qui sont accessibles par le biais d’adresses IP publiques. Vous pouvez en savoir plus sur l’intégration d’un réseau virtuel pour les services Azure [ici](virtual-network-for-azure-services.md). 

Lorsque vous utilisez la fonctionnalité de points de terminaison de service de réseau virtuel (activation du point de terminaison de service du réseau virtuel côté réseau et configuration des ACL de réseau virtuel appropriées côté service Azure), l’accès à un service Azure est restreint au réseau virtuel et sous-réseau autorisés.

### <a name="how-does-vnet-service-endpoint-provide-security"></a>Comment le point de terminaison de service de réseau virtuel garantit-il la sécurité ?

La fonctionnalité de point de terminaison de service de réseau virtuel (activation du point de terminaison de service du réseau virtuel côté réseau et configuration des ACL de réseau virtuel appropriées côté service Azure) limite l’accès au service Azure au réseau virtuel et au sous-réseau autorisés, assurant la sécurité et l’isolation du trafic du service Azure au niveau du réseau. Tout le trafic utilisant les points de terminaison de réseau virtuel traverse le segment principal Microsoft, ce qui offre une couche supplémentaire permettant de l’isoler de l’Internet public. En outre, les clients peuvent choisir de supprimer complètement un accès à l’Internet public aux ressources de service Azure et d’autoriser le trafic uniquement à partir de leur réseau virtuel via une combinaison de pare-feu IP et d’ACL de réseau virtuel, protégeant ainsi les ressources de service Azure contre un accès non autorisé.      

### <a name="what-does-the-vnet-service-endpoint-protect---vnet-resources-or-azure-service"></a>Quels éléments le point de terminaison de service du réseau virtuel protège-t-il ? Les ressources du réseau virtuel ou le service Azure ?
Les points de terminaison de service de réseau virtuel permettent de protéger les ressources de service Azure. Les ressources de réseau virtuel sont protégées via les groupes de sécurité réseau (NSG).

### <a name="is-there-any-cost-for-using-vnet-service-endpoints"></a>Quel est le coût d’utilisation des points de terminaison de service de réseau virtuel ?

Aucun. L’utilisation des points de terminaison de service de réseau virtuel n’entraîne aucuns frais supplémentaires.

### <a name="can-i-turn-on-vnet-service-endpoints-and-set-up-vnet-acls-if-the-virtual-network-and-the-azure-service-resources-belong-to-different-subscriptions"></a>Puis-je activer les points de terminaison de service de réseau virtuel et définir des ACL de réseau virtuel si le réseau virtuel et les ressources de service Azure appartiennent à différents abonnements ?

Oui, vous pouvez. Les réseaux virtuels et les ressources du service Azure peuvent être dans des abonnements identiques ou différents. La seule condition est que le réseau virtuel et les ressources de service Azure se trouvent sous le même client Active Directory (AD).

### <a name="can-i-turn-on-vnet-service-endpoints-and-set-up-vnet-acls-if-the-virtual-network-and-the-azure-service-resources-belong-to-different-ad-tenants"></a>Puis-je activer les points de terminaison de service de réseau virtuel et définir des ACL de réseau virtuel si le réseau virtuel et les ressources de service Azure appartiennent à différents clients AD ?
Oui, c’est possible si vous utilisez des points de terminaison de service pour le service Stockage Azure et Azure Key Vault. Pour les autres services, les points de terminaison de service de réseau virtuel et les ACL de réseau virtuel ne sont pas pris en charge sur des clients AD distincts.

### <a name="can-an-on-premises-devices-ip-address-that-is-connected-through-azure-virtual-network-gateway-vpn-or-expressroute-gateway-access-azure-paas-service-over-vnet-service-endpoints"></a>L’adresse IP d’un appareil local connecté par le biais de la passerelle ExpressRoute ou de la passerelle Réseau virtuel Azure (VPN) peut-elle accéder au service Azure PaaS via des points de terminaison de service de réseau virtuel ?
Par défaut, les ressources du service Azure sécurisées pour des réseaux virtuels ne sont pas accessibles à partir des réseaux locaux. Si vous souhaitez autoriser le trafic depuis un réseau local, vous devez également autoriser les adresses IP publiques (généralement NAT) à partir de vos circuits locaux ou ExpressRoute. Ces adresses IP peuvent être ajoutées via la configuration du pare-feu IP des ressources du service Azure.

### <a name="can-i-use-vnet-service-endpoint-feature-to-secure-azure-service-to-multiple-subnets-within-a-virtual-network-or-across-multiple-virtual-networks"></a>Puis-je utiliser la fonctionnalité de point de terminaison de service de réseau virtuel pour sécuriser un service Azure pour plusieurs sous-réseaux au sein d’un réseau virtuel ou sur plusieurs réseaux virtuels ?
Pour sécuriser les services Azure pour plusieurs sous-réseaux au sein d’un réseau virtuel ou sur plusieurs réseaux virtuels, activez les points de terminaison de service côté réseau sur chacun des sous-réseaux indépendamment et sécurisez les ressources de service Azure pour l’ensemble des sous-réseaux en configurant les ACL de réseau virtuel appropriées côté service Azure.
 
### <a name="how-can-i-filter-outbound-traffic-from-a-virtual-network-to-azure-services-and-still-use-service-endpoints"></a>Comment puis-je filtrer le trafic sortant d’un réseau virtuel vers les services Azure en continuant d’utiliser les points de terminaison de service ?
Si vous souhaitez inspecter ou filtrer le trafic destiné à un service Azure à partir d’un réseau virtuel, vous pouvez déployer une appliance virtuelle réseau au sein du réseau virtuel. Vous pouvez ensuite appliquer des points de terminaison de service au sous-réseau sur lequel est déployée l’appliance virtuelle réseau et sécuriser les ressources de service Azure uniquement pour ce sous-réseau via des ACL de réseau virtuel. Ce scénario peut également être utile si vous souhaitez restreindre l’accès aux services Azure à partir de votre réseau virtuel uniquement pour des ressources Azure spécifiques, à l’aide du filtrage de l’appliance virtuelle réseau. Pour plus d’informations, consultez la section relative à la [sortie avec les appliances virtuelles réseau](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha).

### <a name="what-happens-when-you-access-an-azure-service-account-that-has-a-virtual-network-access-control-list-acl-enabled-from-outside-the-vnet"></a>Que se passe-t-il quand vous accédez à un compte de service Azure qui a une ACL de réseau virtuel activée à partir d’un emplacement extérieur au réseau virtuel ?
L’erreur HTTP 403 ou HTTP 404 est retournée.

### <a name="are-subnets-of-a-virtual-network-created-in-different-regions-allowed-to-access-an-azure-service-account-in-another-region"></a>Les sous-réseaux d’un réseau virtuel créés dans des régions différentes sont-ils autorisés à accéder à un compte de service Azure dans une autre région ? 
Oui, pour la plupart des services Azure, les réseaux virtuels créés dans différentes régions peuvent accéder aux services Azure dans une autre région via les points de terminaison de service de réseau virtuel. Par exemple, si un compte Azure Cosmos DB se trouve dans la région USA Ouest ou USA Est et si les réseaux virtuels se trouvent dans plusieurs régions, le réseau virtuel peut accéder à Azure Cosmos DB. Le Stockage et SQL sont des exceptions, car ils sont régionaux par nature. Le réseau virtuel et le service Azure doivent se trouver dans la même région.
  
### <a name="can-an-azure-service-have-both-a-vnet-acl-and-an-ip-firewall"></a>Un service Azure peut-il avoir une ACL de réseau virtuel et un pare-feu IP ?
Oui, une ACL de réseau virtuel et un pare-feu IP peuvent coexister. Ces deux fonctionnalités se complètent pour garantir isolation et sécurité.
 
### <a name="what-happens-if-you-delete-a-virtual-network-or-subnet-that-has-service-endpoint-turned-on-for-azure-service"></a>Que se passe-t-il si l’on supprime un réseau virtuel ou un sous-réseau ayant un point de terminaison de service activé pour le service Azure ?
La suppression des réseaux virtuels et des sous-réseaux est une opération indépendante prise en charge même lorsque des points de terminaison de service sont activés pour les services Azure. Quand les services Azure sont configurés avec des ACL de réseau virtuel, pour les réseaux virtuels et sous-réseaux concernés, les informations des ACL de réseau virtuel associées à ce service Azure sont désactivées en cas de suppression d’un réseau virtuel ou sous-réseau disposant d’un point de terminaison de service de réseau virtuel activé.
 
### <a name="what-happens-if-an-azure-service-account-that-has-a-vnet-service-endpoint-enabled-is-deleted"></a>Que se passe-t-il si l’on supprime un compte de service Azure qui a un point de terminaison de service de réseau virtuel activé ?
La suppression d’un compte de service Azure est une opération indépendante prise en charge même si le point de terminaison de service est activé côté réseau et que des ACL de réseau virtuel sont configurées côté service Azure. 

### <a name="what-happens-to-the-source-ip-address-of-a-resource-like-a-vm-in-a-subnet-that-has-vnet-service-endpoint-enabled"></a>Qu’arrive-t-il à l’adresse IP source d’une ressource (comme une machine virtuelle sur un sous-réseau) ayant un point de terminaison de service de réseau virtuel activé ?
Quand des points de terminaison de service de réseau virtuel sont activés, les adresses IP sources des ressources du sous-réseau de votre réseau virtuel n’utilisent plus des adresses IPV4 publiques mais des adresses IP privées sur le réseau virtuel Azure pour le trafic vers le service Azure. Notez que cela peut entraîner la panne de pare-feux IP spécifiques préalablement configurés sur une adresse IPV4 publique sur les services Azure. 

### <a name="does-the-service-endpoint-route-always-take-precedence"></a>La route du point de terminaison de service est-elle toujours prioritaire ?
Les points de terminaison de service ajoutent une route système prioritaire sur les routes BGP et offrent un routage optimal pour le trafic de point de terminaison de service. Les points de terminaison de service acheminent toujours le trafic de service directement à partir de votre réseau virtuel vers le service sur le réseau principal de Microsoft Azure. Pour en savoir plus sur la façon dont Azure sélectionne un itinéraire, voir [Routage du trafic de réseau virtuel Azure](virtual-networks-udr-overview.md).
 
### <a name="how-does-nsg-on-a-subnet-work-with-service-endpoints"></a>Comment le groupe de sécurité réseau d’un sous-réseau fonctionne-t-il avec les points de terminaison de service ?
Pour atteindre le service Azure, les groupes de sécurité réseau doivent autoriser la connectivité sortante. Si vos groupes de sécurité réseau sont ouverts à tout le trafic Internet sortant, le trafic de point de terminaison de service doit fonctionner. Vous pouvez également limiter le trafic sortant aux adresses IP de service en utilisant uniquement les balises de service.  
 
### <a name="what-permissions-do-i-need-to-set-up-service-endpoints"></a>De quelles autorisations ai-je besoin pour configurer des points de terminaison de service ?
Les points de terminaison de service peuvent être configurés indépendamment sur un réseau virtuel par un utilisateur avec accès en écriture au réseau virtuel. Pour sécuriser les ressources du service Azure pour un réseau virtuel, l’utilisateur doit disposer des autorisations sur **Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action** pour les sous-réseaux à ajouter. Cette autorisation est incluse par défaut dans le rôle d’administrateur de service intégré et peut être modifiée en créant des rôles personnalisés. Apprenez-en davantage sur les rôles intégrés et l’affectation d’autorisations spécifiques aux [rôles personnalisés](https://docs.microsoft.com/azure/role-based-access-control/custom-roles?toc=%2fazure%2fvirtual-network%2ftoc.json).
 

### <a name="can-i-filter-virtual-network-traffic-to-azure-services-allowing-only-specific-azure-service-resources-over-vnet-service-endpoints"></a>Puis-je filtrer le trafic de réseau virtuel vers les services Azure, en autorisant uniquement des ressources de service Azure spécifiques, sur les points de terminaison de service ? 

Les stratégies de points de terminaison de service de réseau virtuel permettent de filtrer le trafic de réseau virtuel vers les services Azure, en autorisant uniquement des ressources de service Azure spécifiques, sur les points de terminaison de service. Les stratégies de points de terminaison fournissent un contrôle d’accès granulaire du trafic de réseau virtuel vers les services Azure. Plus d’informations sur les stratégies de point de terminaison de service sont disponibles [ici](virtual-network-service-endpoint-policies-overview.md).

### <a name="does-azure-active-directory-azure-ad-support-vnet-service-endpoints"></a>Le logiciel Azure Active Directory (Azure AD) prend-il en charge les points de terminaison de service du réseau virtuel ?

Azure Active Directory (Azure AD) ne prend pas en charge les points de terminaison de service en mode natif. La liste complète des services Azure prenant en charge les points de terminaison de service du réseau virtuel est accessible [ici](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview). Notez que la balise « Microsoft.AzureActiveDirectory » indiquée sous les services prenant en charge les points de terminaison de service est utilisée pour gérer ces derniers dans ADLS génération 1. Pour ADLS génération 1, l’intégration au réseau virtuel dans Azure Data Lake Storage Gen1 fait appel à la sécurité des points de terminaison de service de réseau virtuel entre votre réseau virtuel et Azure Active Directory (Azure AD) afin de générer des revendications de sécurité supplémentaires dans le jeton d’accès. Ces revendications permettent ensuite d’authentifier votre réseau virtuel à votre compte Data Lake Storage Gen1 et d’y accéder. Découvrir plus d’informations sur [l’intégration de réseau virtuel dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)

### <a name="are-there-any-limits-on-how-many-vnet-service-endpoints-i-can-set-up-from-my-vnet"></a>Le nombre de points de terminaison de service de réseau virtuel que je peux configurer à partir de mon réseau virtuel est-il limité ?
Il n’existe aucune limite sur le nombre total de points de terminaison de service de réseau virtuel dans un réseau virtuel. Pour une ressource de service Azure (par exemple, un compte de stockage Azure), les services peuvent appliquer des limites sur le nombre de sous-réseaux utilisés pour la sécurisation de la ressource. Le tableau suivant présente des exemples des limites qui s’appliquent : 

|Service Azure| Limites sur les règles de réseau virtuel|
|---|---|
|Stockage Azure| 100|
|Azure SQL| 128|
|Azure Synapse Analytics|   128|
|Azure KeyVault|    127|
|Azure Cosmos DB|   64|
|Azure Event Hub|   128|
|Azure Service Bus| 128|
|Azure Data Lake Store V1|  100|
 
>[!NOTE]
> Les limites sont sujettes à modifications à la discrétion du service Azure. Reportez-vous à la documentation correspondante pour en savoir plus sur chaque service. 




  



