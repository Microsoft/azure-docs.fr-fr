---
title: Conception de réseaux virtuels avec des ressources de passerelle NAT
titleSuffix: Azure Virtual Network NAT
description: Découvrez comment concevoir des réseaux virtuels avec des ressources de passerelle NAT.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: nat
Customer intent: As an IT administrator, I want to learn more about how to design virtual networks with NAT gateway resources.
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: allensu
ms.openlocfilehash: 62c1b323899f03a043904f4b10d5fe3bb551e0f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91441760"
---
# <a name="designing-virtual-networks-with-nat-gateway-resources"></a>Conception de réseaux virtuels avec des ressources de passerelle NAT

Les ressources de passerelle NAT font partie du [service NAT de Réseau virtuel](nat-overview.md) et offrent une connectivité Internet sortante à un ou plusieurs sous-réseaux d’un réseau virtuel. Le sous-réseau du réseau virtuel indique quelle passerelle NAT est utilisée. Le service NAT fournit la traduction d’adresses réseau sources (SNAT) à un sous-réseau.  Les ressources de passerelle NAT spécifient les adresses IP statiques utilisées par les machines virtuelles lors de la création de flux sortants. Les adresses IP statiques proviennent de ressources d’adresses IP publiques, de ressources de préfixes d’adresses IP publiques ou des deux. Si une ressource de préfixe d’adresse IP publique est utilisée, toutes les adresses IP de l’ensemble de la ressource de préfixe d’adresse IP publique sont consommées par une ressource de passerelle NAT. Une ressource de passerelle NAT peut utiliser jusqu’à 16 adresses IP statiques.

<p align="center">
  <img src="media/nat-overview/flow-direction1.svg" alt="Figure depicts a NAT gateway resource that consumes all IP addresses for a public IP prefix and directs that traffic to and from two subnets of virtual machines and a virtual machine scale set." width="256" title="Service NAT de Réseau virtuel pour le trafic sortant vers Internet">
</p>

*Figure : Service NAT de Réseau virtuel pour le trafic sortant vers Internet*

## <a name="how-to-deploy-nat"></a>Comment déployer le service NAT

La configuration et l’utilisation de la passerelle NAT sont volontairement simples :  

Ressource de passerelle NAT :
- Créez une ressource de passerelle NAT régionale ou zonale (isolé dans une zone).
- Affectez des adresses IP.
- Si nécessaire, modifiez le délai d’inactivité TCP (facultatif).  Vérifiez les [minuteurs](#timers) <ins>avant</ins> de changer la valeur par défaut.

Réseau virtuel :
- Configurez un sous-réseau de réseau virtuel pour utiliser une passerelle NAT.

Les routes définies par l’utilisateur ne sont pas nécessaires.

## <a name="resource"></a>Ressource

La ressource est conçue pour être simple, comme vous pouvez le voir dans l’exemple Azure Resource Manager suivant défini dans un format de type modèle.  Ce format de type modèle est présenté ici pour illustrer les concepts et la structure.  Modifiez l’exemple selon vos besoins.  Ce document n’a pas vocation à service de tutoriel.

Le diagramme suivant montre les références accessibles en écriture entre les différentes ressources Azure Resource Manager.  La flèche indique la direction de la référence, depuis l’emplacement où elle est accessible en écriture. Révision 

<p align="center">
  <img src="media/nat-overview/flow-map.svg" alt="Figure depicts a NAT receiving traffic from internal subnets and directing it to a public IP and an IP prefix." width="256" title="Modèle objet NAT de Réseau virtuel">
</p>

*Figure : Modèle objet NAT de Réseau virtuel*

La traduction d’adresses réseau (NAT) est recommandée pour la plupart des charges de travail, sauf si vous avez une dépendance spécifique vis-à-vis d’une [connectivité sortante Load Balancer basée sur un pool](../load-balancer/load-balancer-outbound-connections.md).  

Vous pouvez migrer à partir de scénarios d’équilibreur de charge standard, [règles de trafic sortant](../load-balancer/load-balancer-outbound-rules-overview.md) comprises, vers une passerelle NAT. Pour migrer, déplacez les ressources d’adresses IP publiques et les ressources de préfixes d’adresses IP publiques des front-ends de l’équilibreur de charge vers la passerelle NAT. Les nouvelles adresses IP de la passerelle NAT ne sont pas nécessaires. Les ressources d’adresses IP publiques et ressources de préfixes d’adresses IP publiques standard peuvent être réutilisées tant que le total ne dépasse pas 16 adresses IP. Planifiez la migration sans oublier l’interruption de service pendant la transition.  Vous pouvez réduire cette interruption au minimum en automatisant le processus. Testez d’abord la migration dans un environnement intermédiaire.  Pendant la transition, les flux entrants ne sont pas affectés.


L’exemple suivant est un extrait de code tiré d’un modèle Azure Resource Manager.  Ce modèle déploie plusieurs ressources, y compris une passerelle NAT.  Dans cet exemple, le modèle a les paramètres suivants :

- **natgatewayname** : nom de la passerelle NAT.
- **location** : région Azure où se trouve la ressource.
- **publicipname** : nom de l’adresse IP publique sortante associée à la passerelle NAT.
- **vnetname** : nom du réseau virtuel.
- **subnetname** : nom du sous-réseau associé à la passerelle NAT.

Le nombre total d’adresses IP fournies par toutes les ressources d’adresse IP et de préfixe ne peut pas dépasser 16 adresses IP. Tout nombre d’adresses IP compris entre 1 et 16 est autorisé.

:::code language="json" source="~/quickstart-templates/101-nat-gateway-vnet/azuredeploy.json" range="81-96":::

Quand la ressource de passerelle NAT est créée, elle peut être utilisée sur un ou plusieurs sous-réseaux d’un réseau virtuel. Spécifiez les sous-réseaux qui utilisent cette ressource de passerelle NAT. Une passerelle NAT ne peut pas s’étendre sur plusieurs réseaux virtuels. Il n’est pas nécessaire d’attribuer la même passerelle NAT à tous les sous-réseaux d’un réseau virtuel. Les sous-réseaux individuels peuvent être configurés avec différentes ressources de passerelle NAT.

Les scénarios qui n’utilisent pas de zones de disponibilité sont régionaux (aucune zone spécifiée). Si vous utilisez des zones de disponibilité, vous pouvez spécifier une zone pour isoler la traduction d’adresses réseau dans une zone spécifique. La redondance dans une zone n’est pas prise en charge. Examinez les [zones de disponibilité](#availability-zones) NAT.

:::code language="json" source="~/quickstart-templates/101-nat-gateway-vnet/azuredeploy.json" range="1-146" highlight="81-96":::

Les passerelles NAT sont définies avec une propriété sur un sous-réseau au sein d’un réseau virtuel. Les flux créés par des machines virtuelles sur le sous-réseau **subnetname** du réseau virtuel **vnetname** utilisent la passerelle NAT. Toute la connectivité sortante utilise les adresses IP associées à **natgatewayname** comme adresse IP source.

Pour plus d’informations sur le modèle Azure Resource Manager utilisé dans cet exemple, consultez :

- [Démarrage rapide : Créer une passerelle NAT - Modèle Resource Manager](quickstart-create-nat-gateway-template.md)
- [Service NAT de réseau virtuel](https://azure.microsoft.com/resources/templates/101-nat-gateway-1-vm/)

## <a name="design-guidance"></a>Conseils pour la conception

Passez en revue cette section pour vous familiariser avec les considérations relatives à la conception des réseaux virtuels avec le service NAT.  

1. [Optimisation des coûts](#cost-optimization)
1. [Coexistence des trafics entrant et sortant](#coexistence-of-inbound-and-outbound)
2. [Gestion des ressources de base](#managing-basic-resources)
3. [Zones de disponibilité](#availability-zones)

### <a name="cost-optimization"></a>Optimisation des coûts

Les [points de terminaison de service](virtual-network-service-endpoints-overview.md) et la [liaison privée](../private-link/private-link-overview.md) sont des options à prendre en compte pour optimiser les coûts. NAT n’est pas nécessaire pour ces services. Le trafic dirigé vers des points de terminaison de service ou une liaison privée n’est pas traité par le service NAT de réseau virtuel.  

Les points de terminaison de service lient les ressources de service Azure à votre réseau virtuel et contrôlent l’accès à vos ressources de service Azure. Par exemple, lorsque vous accédez au stockage Azure, utilisez un point de terminaison de service pour le stockage afin d’éviter des frais de traitement de données par le service NAT. Les points de terminaison de service sont gratuits.

Un lien privé expose le service PaaS Azure (ou d’autres services hébergés avec un lien privé) en tant que point de terminaison privé au sein d’un réseau virtuel.  Un lien privé est facturé selon la durée et les données traitées.

Déterminez si l’une ou l’autre de ces approches convient à votre scénario et utilisez-la si nécessaire.

### <a name="coexistence-of-inbound-and-outbound"></a>Coexistence des trafics entrant et sortant

La passerelle NAT est compatible avec :

 - Équilibreur de charge standard
 - Adresse IP publique standard
 - Préfixe d’adresse IP publique standard

Quand vous développez un nouveau déploiement, commencez avec des références SKU standard.

<p align="center">
  <img src="media/nat-overview/flow-direction1.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network." width="256" title="Service NAT de Réseau virtuel pour le trafic sortant vers Internet">
</p>

*Figure : Service NAT de Réseau virtuel pour le trafic sortant vers Internet*

Le scénario Internet sortant uniquement fourni par la passerelle NAT peut être étendu avec les fonctionnalités de trafic entrant à partir d’Internet. Chaque ressource est consciente de la direction d’origine d’un flux. Sur un sous-réseau doté d’une passerelle NAT, tous les scénarios de trafic sortant vers Internet sont remplacés par la passerelle NAT. Les scénarios de trafic entrant à partir d’Internet sont fournis par la ressource respective.

#### <a name="nat-and-vm-with-instance-level-public-ip"></a>Service NAT et machine virtuelle avec une adresse IP publique au niveau de l’instance

<p align="center">
  <img src="media/nat-overview/flow-direction2.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network and inbound traffic with an instance-level public IP." width="300" title="Service NAT de Réseau virtuel et machine virtuelle avec une adresse IP publique au niveau de l’instance">
</p>

*Figure : Service NAT de Réseau virtuel et machine virtuelle avec une adresse IP publique au niveau de l’instance*

| Sens | Ressource |
|:---:|:---:|
| Trafic entrant | Machine virtuelle avec une adresse IP publique au niveau de l’instance |
| Règle de trafic sortant | Passerelle NAT |

La machine virtuelle utilise la passerelle NAT pour le trafic sortant.  Le trafic entrant n’est pas affecté.

#### <a name="nat-and-vm-with-public-load-balancer"></a>Service NAT et machine virtuelle avec équilibreur de charge public

<p align="center">
  <img src="media/nat-overview/flow-direction3.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network and inbound traffic with a public load balancer." width="350" title="Service NAT de Réseau virtuel et machine virtuelle avec équilibreur de charge public">
</p>

*Figure : Service NAT de Réseau virtuel et machine virtuelle avec équilibreur de charge public*

| Sens | Ressource |
|:---:|:---:|
| Trafic entrant | Équilibreur de charge public |
| Règle de trafic sortant | Passerelle NAT |

Toute configuration de trafic sortant à partir d’une règle d’équilibrage de charge ou de règles de trafic sortant est remplacée par la passerelle NAT.  Le trafic entrant n’est pas affecté.

#### <a name="nat-and-vm-with-instance-level-public-ip-and-public-load-balancer"></a>Service NAT et machine virtuelle avec une adresse IP publique au niveau de l’instance et un équilibreur de charge public

<p align="center">
  <img src="media/nat-overview/flow-direction4.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network and inbound traffic with an instance-level public IP and a public load balancer." width="425" title="Service NAT de Réseau virtuel et machine virtuelle avec une adresse IP publique au niveau de l’instance et un équilibreur de charge public">
</p>

*Figure : Service NAT de Réseau virtuel et machine virtuelle avec une adresse IP publique au niveau de l’instance et un équilibreur de charge public*

| Sens | Ressource |
|:---:|:---:|
| Trafic entrant | Machine virtuelle avec une adresse IP publique au niveau de l’instance et un équilibreur de charge public |
| Règle de trafic sortant | Passerelle NAT |

Toute configuration de trafic sortant à partir d’une règle d’équilibrage de charge ou de règles de trafic sortant est remplacée par la passerelle NAT.  La machine virtuelle utilise également la passerelle NAT pour le trafic sortant.  Le trafic entrant n’est pas affecté.

### <a name="managing-basic-resources"></a>Gestion des ressources de base

L’équilibreur de charge standard, l’adresse IP publique et le préfixe d’adresse IP publique sont compatibles avec la passerelle NAT. Les passerelles NAT fonctionnent dans l’étendue d’un sous-réseau. La référence SKU de base de ces services doit être déployée sur un sous-réseau sans passerelle NAT. Cette séparation permet aux deux variantes de référence SKU de coexister sur le même réseau virtuel.

Les passerelles NAT ont la priorité sur les scénarios de trafic sortant du sous-réseau. L’équilibreur de charge de base ou l’adresse IP publique (et tout service géré créé avec eux) ne peuvent pas s’ajuster aux traductions appropriées. La passerelle NAT prend le contrôle du trafic sortant vers Internet sur un sous-réseau. Le trafic entrant vers l’équilibreur de charge de base et l’adresse IP publique n’est pas disponible. Le trafic entrant vers un équilibreur de charge de base et/ou une adresse IP publique configurée sur une machine virtuelle n’est pas disponible.

### <a name="availability-zones"></a>Zones de disponibilité

#### <a name="zone-isolation-with-zonal-stacks"></a>Isolation de zone avec des piles zonales

<p align="center">
  <img src="media/nat-overview/az-directions.svg" alt="Figure depicts three zonal stacks, each of which contains a NAT gateway and a subnet." width="425" title="NAT de réseau virtuel avec isolation de zone, créant plusieurs « piles zonales » "zonal stacks"">
</p>

*Figure : NAT de réseau virtuel avec isolation de zone, créant plusieurs « piles zonales »*

Même sans zones de disponibilité, le service NAT est résilient et peut survivre à plusieurs défaillances de composants d’infrastructure.  Les zones de disponibilité reposent sur cette résilience avec des scénarios d’isolation de zone pour la traduction d’adresses réseau (NAT).

Les réseaux virtuels et leurs sous-réseaux sont des constructions régionales.  Les sous-réseaux ne sont pas limités à une zone.

Il existe une promesse zonale pour l’isolation de zone quand une instance de machine virtuelle utilisant une ressource de passerelle NAT se trouve dans la même zone que la ressource de passerelle NAT et ses adresses IP publiques. Le modèle que vous voulez utiliser pour l’isolation de zone crée une « pile zonale » par zone de disponibilité.  Cette « pile zonale » comprend des instances de machine virtuelle, des ressources de passerelle NAT, des ressources d’adresse IP publique et/ou de préfixe sur un sous-réseau qui est supposé traiter uniquement la même zone.   Les opérations du plan de contrôle et le plan de données sont alignés sur la zone spécifiée et y sont limités. 

Une défaillance dans une zone autre que celle où se trouve votre scénario est supposée n’exercer aucun impact sur le service NAT. Le trafic sortant des machines virtuelles dans la même zone échoue en raison de l’isolement de la zone.  

#### <a name="integrating-inbound-endpoints"></a>Intégration de points de terminaison entrants

Si votre scénario nécessite des points de terminaison entrants, deux options s’offrent à vous :

| Option | Modèle | Exemple | Avantage | Inconvénient |
|---|---|---|---|---|
| (1) | **Aligner** les points de terminaison entrants sur les **piles zonales** respectives que vous créez pour le trafic sortant. | Créez un équilibreur de charge standard avec un front-end zonal. | Même modèle d’intégrité et même mode d’échec pour le trafic entrant et sortant. Plus simple à utiliser. | Il peut être nécessaire de masquer les adresses IP individuelles par zone à l’aide d’un nom DNS commun. |
| (2) | **Superposer** les piles zonales avec un point de terminaison entrant **entre les zones**. | Créez un équilibreur de charge standard avec un front-end redondant interzone. | Adresse IP unique pour un point de terminaison entrant. | Modèle d’intégrité et modes d’échec variables pour le trafic entrant et sortant.  Plus complexe à utiliser. |

>[!NOTE]
> Une passerelle NAT isolée dans une zone exige que les adresses IP correspondent à la zone de la passerelle NAT. Les ressources de passerelle NAT avec des adresses IP d’une autre zone ou sans zone ne sont pas autorisées.

#### <a name="cross-zone-outbound-scenarios-not-supported"></a>Scénarios de trafic sortant entre les zones non pris en charge

<p align="center">
  <img src="media/nat-overview/az-directions2.svg" alt="Figure depicts three zonal stacks, each of which contains a NAT gateway and a subnet, with the connections between to of the gateways and their subnets broken." width="425" title="NAT de réseau virtuel non compatible avec un sous-réseau couvrant des zones">
</p>

*Figure : NAT de réseau virtuel non compatible avec un sous-réseau couvrant des zones*

Vous ne pouvez pas tenir une promesse zonale avec des ressources de passerelle NAT quand des instances de machine virtuelle sont déployées dans plusieurs zones du même sous-réseau.   Et même plusieurs passerelles NAT zonales étaient attachées à un sous-réseau, l’instance de machine virtuelle ne saurait pas quelle ressource de passerelle NAT sélectionner.

Une promesse zonale n’existe quand a) la zone d’une instance de machine virtuelle et la zone d’une passerelle NAT zonale ne sont pas alignées, ou b) une ressource de passerelle NAT régionale est utilisée avec des instances de machine virtuelle zonales.

Même si le scénario semble fonctionner, son modèle d’intégrité et son mode d’échec ne sont pas définis du point de vue d’une zone de disponibilité. Envisagez plutôt d’utiliser des piles zonales ou de toutes les zones géographiques.

>[!NOTE]
>La propriété de zones d’une ressource de passerelle NAT n’est pas mutable.  Redéployez la ressource de passerelle NAT avec la préférence régionale ou de zone souhaitée.

>[!NOTE] 
>Les adresses IP ne sont pas par elles-mêmes redondantes dans une zone, si aucune zone n’est spécifiée.  Le front-end d’un service [Standard Load Balancer est redondant dans une zone](../load-balancer/load-balancer-standard-availability-zones.md#frontend) si aucune adresse IP n’est créée dans une zone spécifique.  Cela ne s’applique pas au service NAT.  Seul l’isolement régional ou de zone est pris en charge.

## <a name="performance"></a>Performances

Chaque ressource de passerelle NAT peut fournir un débit maximal de 50 Gbits/s. Vous pouvez diviser vos déploiements entre plusieurs sous-réseaux et affecter à chaque sous-réseau ou groupe de sous-réseaux une passerelle NAT pour un scale-out.

Chaque passerelle NAT peut prendre en charge 64 000 flux respectivement pour TCP et UDP par adresse IP sortante attribuée.  Consultez la section suivante sur la traduction d’adresses réseau sources (SNAT, Source Network Address Translation) pour plus de détails, ainsi que l’[article dédié à la résolution des problèmes](https://docs.microsoft.com/azure/virtual-network/troubleshoot-nat) pour obtenir des conseils de résolution des problèmes spécifiques.

## <a name="source-network-address-translation"></a>Traduction d’adresses réseau sources

La traduction d’adresses réseau sources (SNAT) réécrit la source d’un flux pour qu’il provienne d’une autre adresse IP.  Les ressources de passerelle NAT utilisent une variante de SNAT communément appelée traduction d’adresses de port (PAT). PAT réécrit l’adresse source et le port source. Avec SNAT, il n’existe aucune relation fixe entre le nombre d’adresses privées et leurs adresses publiques traduites.  

### <a name="fundamentals"></a>Notions de base

Examinons un exemple de quatre flux pour expliquer le concept de base.  La passerelle NAT utilise la ressource d’adresse IP publique 65.52.1.1 et la machine virtuelle établit des connexions à l’adresse 65.52.0.1.

| Flux | Tuple source | Tuple de destination |
|:---:|:---:|:---:|
| 1 | 192.168.0.16:4283 | 65.52.0.1:80 |
| 2 | 192.168.0.16:4284 | 65.52.0.1:80 |
| 3 | 192.168.0.17.5768 | 65.52.0.1:80 |

Ces flux peuvent ressembler à ceci après l’application de PAT :

| Flux | Tuple source | Tuple source après SNAT | Tuple de destination | 
|:---:|:---:|:---:|:---:|
| 1 | 192.168.0.16:4283 | **65.52.1.1:1234** | 65.52.0.1:80 |
| 2 | 192.168.0.16:4284 | **65.52.1.1:1235** | 65.52.0.1:80 |
| 3 | 192.168.0.17.5768 | **65.52.1.1:1236** | 65.52.0.1:80 |

La destination voit que la source du flux est 65.52.0.1 (Tuple source SNAT) avec le port attribué indiqué.  PAT, comme indiqué dans le tableau précédent, est également appelé SNAT de déguisement de port.  Plusieurs sources privées se font passer pour une adresse IP et un port.  

#### <a name="source-snat-port-reuse"></a>réutilisation des ports source (SNAT)

Les passerelles NAT réutilisent de façon opportuniste les ports sources (SNAT).  L’exemple suivant illustre ce concept comme un flux supplémentaire pour l’ensemble de flux précédent.  La machine virtuelle dans l’exemple est un flux vers l’adresse 65.52.0.2.

| Flux | Tuple source | Tuple de destination |
|:---:|:---:|:---:|
| 4 | 192.168.0.16:4285 | 65.52.0.2:80 |

Une passerelle NAT translatera probablement le flux 4 vers un port également utilisable pour d’autres destinations.  Pour plus d’informations sur le dimensionnement correct de l’approvisionnement de votre adresse IP, consultez [Mise à l’échelle](https://docs.microsoft.com/azure/virtual-network/nat-gateway-resource#scaling).

| Flux | Tuple source | Tuple source après SNAT | Tuple de destination | 
|:---:|:---:|:---:|:---:|
| 4 | 192.168.0.16:4285 | 65.52.1.1:**1234** | 65.52.0.2:80 |

Ne prenez pas de dépendance vis-à-vis de la façon dont les ports sources sont affectés dans l’exemple ci-dessus.  L’illustration précédente montre uniquement le concept de base.

La traduction d’adresses réseau sources (SNAT) fournie par le service NAT diffèrent de [Load Balancer](../load-balancer/load-balancer-outbound-connections.md) sous plusieurs aspects.

### <a name="on-demand"></a>À la demande

NAT fournit des ports SNAT à la demande pour les nouveaux flux de trafic sortant. Tous les ports SNAT disponibles dans l’inventaire sont utilisés par toute machine virtuelle sur les sous-réseaux configurés avec NAT. 

<p align="center">
  <img src="media/nat-overview/lb-vnnat-chart.svg" alt="Figure depicts inventory of all available SNAT ports used by any virtual machine on subnets configured with N A T." width="550" title="SNAT de trafic sortant à la demande du service NAT de Réseau virtuel">
</p>

*Figure : SNAT de trafic sortant à la demande du service NAT de Réseau virtuel*

Toute configuration IP d’une machine virtuelle peut créer des flux sortants à la demande si besoin.  Aucune préallocation, par planification d’instance incluant le surprovisionnement du pire cas par instance, n’est nécessaire.  

<p align="center">
  <img src="media/nat-overview/exhaustion-threshold.svg" alt="Figure depicts inventory of all available SNAT ports used by any virtual machine on subnets configured with N A T with exhaustion threshold." width="550" title="Différences dans les scénarios d’épuisement">
</p>

*Figure : Différences dans les scénarios d’épuisement*

Une fois qu’un port SNAT est libéré, il devient disponible pour être utilisé par toute machine virtuelle sur des sous-réseaux configurés avec NAT.  L’allocation à la demande permet aux charges de travail dynamiques et divergentes sur des sous-réseaux d’utiliser des ports SNAT selon leurs besoins.  Tant qu’un inventaire des ports SNAT est disponible, les flux SNAT aboutissent. Les zones réactives de port SNAT bénéficient plutôt de l’inventaire plus grand. Les ports SNAT ne sont pas laissés inutilisés pour les machines virtuelles qui n’en ont pas activement besoin.

### <a name="scaling"></a>Mise à l'échelle

La mise à l’échelle NAT est principalement une fonction de gestion de l’inventaire des ports SNAT partagés et disponibles. NAT a besoin d’un inventaire de ports SNAT suffisant pour traiter les flux sortant maximaux attendus pour tous les sous-réseaux attachés à une ressource de passerelle NAT.  Vous pouvez utiliser des ressources d’adresses publiques IP, des ressources de préfixes d’adresses IP publiques, ou des deux, pour créer un inventaire des ports SNAT.  

>[!NOTE]
>Si vous attribuez une ressource de préfixe d’adresse IP publique, l’ensemble du préfixe d’adresse IP publique entier est utilisé.  Vous ne pouvez pas attribuer une ressource de préfixe d’adresse IP publique puis séparer des adresses IP individuelles pour les attribuer à d’autres ressources.  Si vous voulez attribuer des adresses IP individuelles à partir d’un préfixe d’adresse IP publique à plusieurs ressources, vous devez créer des adresses IP publiques individuelles à partir de la ressource de préfixe d’adresse IP publique, puis les attribuer en fonction des besoins plutôt que d’attribuer la ressource de préfixe d’adresse IP publique elle-même.

SNAT mappe des adresses privées à une ou plusieurs adresses IP publiques, en réécrivant l’adresse source et le port source dans les processus. Une ressource de passerelle NAT utilise 64 000 ports (ports SNAT) par adresse IP publique configurée pour cette traduction. Les ressources de passerelle NAT peuvent comprendre jusqu’à 16 adresses IP et 1 million de ports SNAT. Si une ressource de préfixe d’adresse IP publique est fournie, chaque adresse IP présente dans le préfixe fournit un inventaire des ports SNAT. L’ajout d’autres adresses IP publiques augmente les ports SNAT disponibles dans l’inventaire. Les protocoles TCP et UDP sont des inventaires de ports SNAT distincts qui ne sont pas liés.

Les ressources de passerelle NAT réutilisent de façon opportuniste les ports sources (SNAT). Au titre de conseil de conception à des fins de mise à l’échelle, vous devez partir du principe que chaque flux nécessite un nouveau port SNAT et mettre à l’échelle le nombre total d’adresses IP disponibles pour le trafic sortant.  Vous devez réfléchir soigneusement à la mise à l’échelle que vous concevez, et approvisionner les quantités d’adresses IP en conséquence.

Les ports SNAT vers différentes destinations sont les plus susceptibles d’être réutilisés quand c’est possible. Et lorsque l’épuisement des ports SNAT approche, les flux peuvent échouer.  

Pour un exemple, consultez [Notions de base de SNAT](https://docs.microsoft.com/azure/virtual-network/nat-gateway-resource#source-network-address-translation).


### <a name="protocols"></a>Protocoles

Les ressources de passerelle NAT interagissent avec les en-têtes IP et de transport IP des flux UDP et TCP. Elles sont indépendantes des charges utiles de la couche Application.  Les autres protocoles IP ne sont pas pris en charge.

### <a name="timers"></a>Minuteurs

>[!IMPORTANT]
>Le minuteur spécifiant une longue période d’inactivité peut augmenter inutilement la probabilité d’épuisement des ressources SNAT. Plus la période que vous spécifiez pour le minuteur est longue, plus la durée pendant laquelle la traduction d’adresses réseau (NAT) accapare les ports SNAT est longue, jusqu’à ce que le délai d’inactivité finisse par être atteint. Si le délai d’inactivité de vos flux est dépassé, ces derniers finissent par échouer quand même et consomment inutilement l’inventaire des ports SNAT.  Les flux qui échouent au bout de 2 heures auraient également échoué au bout de 4 minutes (valeur par défaut). L’augmentation du délai d’inactivité est une option de dernier recours qui doit être utilisée avec modération. Si un flux ne devient jamais inactif, il n’est pas affecté par le minuteur d’inactivité.

Le délai d’inactivité TCP peut varier de 4 minutes (valeur par défaut) à 120 minutes (2 heures) pour tous les flux.  De plus, vous pouvez réinitialiser le minuteur d’inactivité avec du trafic en cours de flux.  Un modèle recommandé pour l’actualisation des connexions inactives longues et la détection d’activité sur les points de terminaison consiste à utiliser des conservations de connexion active TCP.  Les conservations de connexion active TCP apparaissent comme des accusés de réception en double sur les points de terminaison, leur surcharge est faible et elles sont invisibles pour la couche Application.

Les minuteurs suivants sont utilisés pour la libération des ports SNAT :

| Minuterie | Valeur |
|---|---|
| TCP FIN | 60 secondes |
| TCP RST | 10 secondes |
| TCP demi-ouvert | 30 secondes |

Un port SNAT peut être réutilisé pour la même adresse IP de destination et le même port de destination au bout de 5 secondes.

>[!NOTE] 
>Ces paramètres de minuteur sont susceptibles d’être modifiés. Les valeurs sont fournies pour faciliter la résolution des problèmes et vous ne devez pas dépendre de minuteurs spécifiques pour le moment.

## <a name="limitations"></a>Limites

- NAT est compatible avec des ressources d’adresses IP publiques, de préfixes d’adresses IP publiques et d’équilibreur de charge de la référence SKU standard.   Les ressources de base (par exemple, un équilibreur de charge de base) et tous les produits dérivés de celles-ci ne sont pas compatibles avec NAT.  Les ressources de base doivent être placées sur un sous-réseau non configuré avec NAT.
- La famille d’adresses IPv4 est prise en charge.  NAT n’interagit pas avec la famille d’adresses IPv6.  NAT ne peut pas être déployé sur un sous-réseau avec un préfixe IPv6.
- NAT ne peut pas s’étendre sur plusieurs réseaux virtuels.

## <a name="suggestions"></a>Suggestions

Nous aimerions savoir comment nous pouvons améliorer le service. Il vous manque une fonctionnalité ? Envoyez-nous vos suggestions sur [UserVoice for NAT](https://aka.ms/natuservoice).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez le [service NAT de réseau virtuel](nat-overview.md).
* Découvrez les [métriques et les alertes pour les ressources de passerelle NAT](nat-metrics.md).
* Découvrez la [résolution des problèmes liés aux ressources de passerelle NAT](troubleshoot-nat.md).
* Tutoriel de validation de NAT Gateway
  - [Azure CLI](tutorial-create-validate-nat-gateway-cli.md)
  - [PowerShell](tutorial-create-validate-nat-gateway-powershell.md)
  - [Portail](tutorial-create-validate-nat-gateway-portal.md)
* Démarrage rapide du déploiement d’une ressource de passerelle NAT
  - [Azure CLI](./quickstart-create-nat-gateway-cli.md)
  - [PowerShell](./quickstart-create-nat-gateway-powershell.md)
  - [Portail](./quickstart-create-nat-gateway-portal.md)
  - [Modèle](./quickstart-create-nat-gateway-template.md)
* Découvrir l’API de ressource de passerelle NAT
  - [REST API](https://docs.microsoft.com/rest/api/virtualnetwork/natgateways)
  - [Azure CLI](https://docs.microsoft.com/cli/azure/network/nat/gateway)
  - [PowerShell](https://docs.microsoft.com/powershell/module/az.network/new-aznatgateway)
* Découvrez les [zones de disponibilité](../availability-zones/az-overview.md).
* Découvrez l’[équilibreur de charge standard](../load-balancer/load-balancer-standard-overview.md).
* Découvrez les [zones de disponibilité et l’équilibreur de charge standard](../load-balancer/load-balancer-standard-availability-zones.md).
* [Utilisez UserVoice pour nous faire part des prochains développements dont vous aimeriez bénéficier concernant le service NAT de réseau virtuel](https://aka.ms/natuservoice).
