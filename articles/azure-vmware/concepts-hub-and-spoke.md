---
title: 'Concept : intégrer un déploiement Azure VMware Solution dans une architecture hub and spoke'
description: En savoir plus sur l’intégration d’un déploiement Azure VMware Solution dans une architecture hub and spoke sur Azure.
ms.topic: conceptual
ms.date: 10/26/2020
ms.openlocfilehash: 788ef9886e0d102a549e84cd01c658e9e4131c63
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967446"
---
# <a name="integrate-azure-vmware-solution-in-a-hub-and-spoke-architecture"></a>Intégrer Azure VMware Solution dans une architecture hub and spoke

Cet article fournit des recommandations relatives à l’intégration d’un déploiement Azure VMware Solution dans une [architecture hub and spoke](/azure/architecture/reference-architectures/hybrid-networking/#hub-spoke-network-topology) existante ou nouvelle sur Azure. 


Le scénario Hub and Spoke suppose un environnement de cloud hybride avec des charges de travail sur :

* Azure natif utilisant les services IaaS ou PaaS
* Azure VMware Solution 
* vSphere local

## <a name="architecture"></a>Architecture

Le *Hub* est un réseau virtuel Azure qui centralise la connectivité à votre cloud privé local et Azure VMware Solution. Les *Spokes* sont des réseaux virtuels appairés avec le Hub pour permettre la communication entre des réseaux virtuels.

Le trafic entre le centre de données local, le cloud privé Azure VMware Solution et le hub passe par des connexions Azure ExpressRoute. Les réseaux virtuels Spoke contiennent généralement des charges de travail IaaS, mais ils peuvent avoir des services PaaS comme [App Service Environment](../app-service/environment/intro.md), qui a une intégration directe avec le réseau virtuel, ou d’autres services PaaS avec [Azure Private Link](../private-link/index.yml) activé.

>[!IMPORTANT]
>Vous pouvez utiliser une passerelle ExpressRoute existante pour vous connecter à Azure VMware Solution, à condition qu’elle ne dépasse pas la limite de quatre circuits ExpressRoute par réseau virtuel.  Toutefois, pour accéder à Azure VMware Solution à partir d’un site local via ExpressRoute, vous devez disposer de ExpressRoute Global Reach, car la passerelle ExpressRoute ne fournit pas de routage transitif entre ses circuits connectés.

Le diagramme montre un exemple de déploiement Hub and Spoke dans Azure, connecté en local et à Azure VMware Solution via ExpressRoute Global Reach.

:::image type="content" source="./media/hub-spoke/azure-vmware-solution-hub-and-spoke-deployment.png" alt-text="Déploiement de l’intégration hub and spoke Azure VMware Solution" border="false" lightbox="./media/hub-spoke/azure-vmware-solution-hub-and-spoke-deployment.png":::

L’architecture possède les composants majeurs suivants :

- **Site local :** Un ou plusieurs centres de données client locaux connectés à Azure via une connexion ExpressRoute.

- **Cloud privé Azure VMware Solution** SDDC Azure VMware Solution formé par un ou plusieurs clusters vSphere, chacun avec un maximum de 16 hôtes.

- **Passerelle ExpressRoute :** Active la communication entre le cloud privé Azure VMware Solution, les services partagés sur le réseau virtuel du hub et les charges de travail exécutées sur des réseaux virtuels Spoke.

- **ExpressRoute Global Reach :** Active la connectivité entre une installation locale et le cloud privé Azure VMware Solution. La connectivité entre Azure VMware Solution et l’infrastructure Azure s’effectue à l’aide d’ExpressRoute Global Reach uniquement. Vous ne pouvez sélectionner aucune option autre qu’ExpressRoute Fast Path.  ExpressRoute Direct n’est pas pris en charge.


- **Considérations S2S VPN :** Pour les déploiements de production Azure VMware Solution, Azure S2S VPN n’est pas pris en charge en raison de la configuration réseau requise pour VMware HCX. Toutefois, vous pouvez l’utiliser pour un déploiement POC.


- **Réseau virtuel Hub :** Joue le rôle de point central de la connectivité pour votre réseau local cloud privé Azure VMware Solution.

- **Réseau virtuel Spoke**

    - **Spoke IaaS :** Un spoke IaaS héberge des charges de travail IaaS Azure, y compris des groupes à haute disponibilité de machines virtuelles et des groupes de machines virtuelles identiques, ainsi que les composants réseau correspondants.

    - **Spoke PaaS :** Un spoke PaaS héberge des services PaaS Azure à l’aide de l’adressage privé grâce au [Point de terminaison privé](../private-link/private-endpoint-overview.md) et à la [Liaison privée](../private-link/private-link-overview.md).

- **Pare-feu Azure :** Fait office d’élément central permettant de segmenter le trafic entre les spokes et Azure VMware Solution.

- **Application Gateway :** Expose et protège des applications web qui s’exécutent sur des machines virtuelles IaaS/PaaS Azure ou Azure VMware Solution. Il s’intègre à d’autres services tels que la Gestion des API.

## <a name="network-and-security-considerations"></a>Considérations relatives au réseau et à la sécurité

Les connexions ExpressRoute permettent au trafic de circuler entre les sites locaux, Azure VMware Solution et l’infrastructure du réseau Azure. Azure VMware Solution utilise le service [Global Reach d’ExpressRoute](../expressroute/expressroute-global-reach.md) pour implémenter cette connectivité.

Comme une passerelle ExpressRoute ne fournit pas de routage transitif entre ses circuits connectés, la connectivité locale doit également utiliser ExpressRoute Global Reach pour communiquer entre l’environnement vSphere local et Azure VMware Solution. 

* **Flux de trafic local vers Azure VMware Solution**

  :::image type="content" source="./media/hub-spoke/on-premises-azure-vmware-solution-traffic-flow.png" alt-text="Flux de trafic local vers Azure VMware Solution" border="false" lightbox="./media/hub-spoke/on-premises-azure-vmware-solution-traffic-flow.png":::


* **Flux de trafic Azure VMware Solution vers le réseau virtuel hub**

  :::image type="content" source="./media/hub-spoke/azure-vmware-solution-hub-vnet-traffic-flow.png" alt-text="Flux de trafic Azure VMware Solution vers le réseau virtuel hub" border="false" lightbox="./media/hub-spoke/azure-vmware-solution-hub-vnet-traffic-flow.png":::


Pour plus d’informations sur les réseaux Azure VMware Solution et les concepts de connectivité, consultez la [Documentation produit d’Azure VMware Solution](./concepts-networking.md).

### <a name="traffic-segmentation"></a>Segmentation du trafic

[Pare-feu Azure](../firewall/index.yml) est la partie centrale de la topologie hub and spoke, déployée sur le réseau virtuel hub. Utilisez le Pare-feu Azure ou une autre appliance virtuelle réseau prise en charge par Azure pour établir des règles de trafic et segmenter la communication entre les différents spokes et les charges de travail Azure VMware Solution.

Créez des tables de routage pour diriger le trafic vers le Pare-feu Azure.  Pour les réseaux virtuels spoke, créez un itinéraire qui définit l’itinéraire par défaut vers l’interface interne de Pare-feu Azure. Ainsi, lorsqu’une charge de travail dans le réseau virtuel doit atteindre l’espace d’adressage Azure VMware Solution, le pare-feu peut l’évaluer et appliquer la règle de trafic correspondante pour l’autoriser ou la refuser.  

:::image type="content" source="media/hub-spoke/create-route-table-to-direct-traffic.png" alt-text="Créer des tables de routage pour diriger le trafic vers le Pare-feu Azure" lightbox="media/hub-spoke/create-route-table-to-direct-traffic.png":::


> [!IMPORTANT]
> Un itinéraire avec le préfixe d’adresse 0.0.0.0/0 sur le paramètre **GatewaySubnet** n’est pas pris en charge.

Définissez des itinéraires pour des réseaux spécifiques sur la table de routage correspondante. Par exemple, des itinéraires pour atteindre les préfixes d’adresses IP des charges de travail et de gestion Azure VMware Solution depuis des charges de travail de spoke et inversement.

:::image type="content" source="media/hub-spoke/specify-gateway-subnet-for-route-table.png" alt-text="Définir des itinéraires pour des réseaux spécifiques sur la table de routage correspondante" lightbox="media/hub-spoke/specify-gateway-subnet-for-route-table.png":::

Un deuxième niveau de segmentation du trafic utilisant les groupes de sécurité réseau dans les Spokes et le Hub pour créer une stratégie de trafic plus granulaire.

> [!NOTE]
> **Trafic local vers Azure VMware Solution :** Le trafic entre des charges de travail locales (vSphere ou autres) est activé par Global Reach, mais le trafic ne passe pas par le Pare-feu Azure sur le hub. Dans ce scénario, vous devez implémenter des mécanismes de segmentation du trafic, localement ou dans Azure VMware Solution.

### <a name="application-gateway"></a>Application Gateway

L’Application Gateway Azure V1 et V2 ont été testées avec des applications web qui s’exécutent sur des machines virtuelles Azure VMware Solution en tant que pool principal. L’Application Gateway est actuellement la seule méthode prise en charge pour exposer des applications web s’exécutant sur des machines virtuelles Azure VMware Solution à Internet. Elle peut également exposer les applications aux utilisateurs internes en toute sécurité.

Pour plus d’informations et pour connaître les conditions requises, consultez l’article spécifique à Azure VMware Solution sur [Application Gateway](./protect-azure-vmware-solution-with-application-gateway.md).

:::image type="content" source="media/hub-spoke/azure-vmware-solution-second-level-traffic-segmentation.png" alt-text="Deuxième niveau de segmentation du trafic à l’aide des groupes de sécurité réseau" border="false":::


### <a name="jump-box-and-azure-bastion"></a>Serveur jump et Azure Bastion

Accédez à l’environnement Azure VMware Solution à l’aide d’un serveur jump, qui est une machine virtuelle Windows 10 ou Windows Server déployée dans le sous-réseau de service partagé au sein du réseau virtuel hub.

>[!IMPORTANT]
>Azure Bastion est le service recommandé pour se connecter au serveur de rebond afin d’empêcher l’exposition à Internet d’Azure VMware Solution. Vous ne pouvez pas utiliser Azure Bastion pour vous connecter à des machines virtuelles Azure VMware Solution, car il ne s’agit pas d’objets IaaS Azure.  

Pour des raisons de sécurité, il est recommandé de déployer le service [Microsoft Azure Bastion](../bastion/index.yml) au sein du réseau virtuel Hub. Azure Bastion fournit un accès RDP et SSH transparent aux machines virtuelles déployées sur Azure sans avoir à approvisionner des adresses IP publiques pour ces ressources. Une fois que vous avez configuré le service Azure Bastion, vous pouvez accéder à la machine virtuelle sélectionnée à partir du Portail Azure. Après avoir établi la connexion, un nouvel onglet s’ouvre et affiche le bureau du serveur jump et, à partir de ce bureau, vous pouvez accéder au plan de gestion du cloud privé Azure VMware Solution.

> [!IMPORTANT]
> N’attribuez pas d’IP publique à la machine virtuelle du serveur jump ni n’exposez le port 3389/TCP à l’Internet public. 


:::image type="content" source="media/hub-spoke/azure-bastion-hub-vnet.png" alt-text="Réseau virtuel Hub Azure Bastion" border="false":::


## <a name="azure-dns-resolution-considerations"></a>Considérations relatives à la résolution Azure DNS

Pour la résolution Azure DNS, deux options sont disponibles :

-   Utilisez les contrôleurs de domaine Azure Active Directory (Azure AD) déployés sur le Hub (décrits dans [Considérations relatives aux identités](#identity-considerations)) en tant que serveurs de noms.

-   Déployez et configurez une zone privée Azure DNS.

La meilleure approche consiste à combiner les deux pour fournir une résolution de noms fiable pour Azure VMware Solution, sur site et Azure.

En guise de suggestion générale, utilisez l’infrastructure Azure DNS existante (dans ce cas, le DNS intégré à Active Directory) déployée sur au moins deux machines virtuelles Azure déployées dans le réseau virtuel Hub et configurées dans les réseaux virtuels Spoke pour utiliser ces serveurs Azure DNS dans les paramètres DNS.

Vous pouvez utiliser Azure DNS privé, où la zone Azure DNS privé est liée au réseau virtuel.  Les serveurs DNS sont utilisés en tant que solutions de résolution hybrides avec un transfert conditionnel vers des DNS locaux ou Azure VMware Solution tirant parti de l’infrastructure client d’Azure DNS privé. 

Pour gérer automatiquement le cycle de vie des enregistrements DNS pour les machines virtuelles déployées au sein de réseaux virtuels spoke, activez l’inscription automatique. Lorsque l’inscription automatique est activée, le nombre maximal de zones DNS privées est 1. Si l’inscription automatique est désactivée, le nombre maximal est 1 000.

Vous pouvez configurer des serveurs locaux et Azure VMware Solution avec des redirecteurs conditionnels vers des machines virtuelles de résolution dans Azure pour la zone privée Azure DNS.

## <a name="identity-considerations"></a>Identité - Éléments à prendre en compte

À des fins d’identité, la meilleure approche consiste à déployer au moins un contrôleur de domaine AD sur le hub. Utilisez deux sous-réseaux de service partagés dans le mode distribué par zone ou dans un groupe à haute disponibilité de machines virtuelles. Consultez [Centre des architectures Azure](/azure/architecture/reference-architectures/identity/adds-extend-domain) pour étendre votre domaine AD local à Azure.

En outre, déployez un autre contrôleur de domaine sur le côté Azure VMware Solution pour agir en tant qu’identité et source DNS au sein de l’environnement vSphere.

Comme meilleure pratique recommandée, intégrez [domaine AD avec Azure Active Directory](/azure/architecture/reference-architectures/identity/azure-ad).

<!-- LINKS - external -->
[Azure Architecture Center]: /azure/architecture/

[Hub & Spoke topology]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke

[Azure networking documentation]: ../networking/index.yml

<!-- LINKS - internal -->
