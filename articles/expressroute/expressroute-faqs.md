---
title: FAQ - Azure ExpressRoute | Microsoft Docs
description: Le Forum aux questions ExpressRoute contient des informations sur les services Azure pris en charge, le coût, les données et connexions, le contrat de niveau de service, les fournisseurs et les emplacements, la bande passante et les détails techniques.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: duau
ms.openlocfilehash: ef5b065425fa05d016c1b1c1688cc28508f32d30
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96462048"
---
# <a name="expressroute-faq"></a>Forum Aux Questions ExpressRoute

## <a name="what-is-expressroute"></a>Présentation d’ExpressRoute

ExpressRoute est un service Azure qui vous permet de créer des connexions privées entre les centres de données Microsoft et une infrastructure locale ou une installation de colocalisation. Les connexions ExpressRoute ne sont pas établies via le réseau public Internet et offrent plus de sécurité, de fiabilité et de rapidité, ainsi que moins de latences que les connexions classiques sur Internet.

### <a name="what-are-the-benefits-of-using-expressroute-and-private-network-connections"></a>Quels avantages présentent l’utilisation d’ExpressRoute et les connexions de réseau privé ?

Les connexions ExpressRoute ne sont pas établies par le biais de l'Internet public. Elles offrent plus de sécurité, de fiabilité et de rapidité, ainsi que des latences inférieures et cohérentes par rapport aux connexions classiques sur Internet. Dans certains cas, l’utilisation de connexions ExpressRoute pour transférer des données entre les appareils locaux et Azure peut générer des économies significatives.

### <a name="where-is-the-service-available"></a>Où le service est-il disponible ?

Consultez cette page pour connaître l’emplacement et la disponibilité du service : [Partenaires ExpressRoute et emplacements](expressroute-locations.md).

### <a name="how-can-i-use-expressroute-to-connect-to-microsoft-if-i-dont-have-partnerships-with-one-of-the-expressroute-carrier-partners"></a>Comment puis-je utiliser ExpressRoute pour me connecter à Microsoft si je n’ai pas conclu de partenariat avec l’un des partenaires opérateurs d’ExpressRoute ?

Vous pouvez sélectionner un opérateur régional et accéder à des connexions Ethernet établies avec l’un des emplacements de fournisseur Exchange pris en charge. Vous pouvez ensuite vous apparier avec Microsoft à l’emplacement du fournisseur. Vérifiez la dernière section de la rubrique [Partenaires et emplacements ExpressRoute](expressroute-locations.md) pour voir si votre fournisseur de services est présent dans l’un des emplacements Exchange. Vous pouvez ensuite commander un circuit ExpressRoute via le fournisseur de services pour vous connecter à Azure.

### <a name="how-much-does-expressroute-cost"></a>Combien coûte ExpressRoute ?

Pour plus d'informations sur la tarification, consultez la page [Tarification](https://azure.microsoft.com/pricing/details/expressroute/) .

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-do-i-have-this-bandwidth-allocated-for-ingress-and-egress-traffic-separately"></a>Si je paie pour un circuit ExpressRoute d’une bande passante donnée, cette bande passante est-elle allouée séparément pour le trafic d’entrée et de sortie ?

Oui, la bande passante du circuit ExpressRoute utilise un canal duplex. Par exemple, si vous achetez un circuit ExpressRoute de 200 Mbits/s, vous obtenez 200 Mbits/s pour le trafic d’entrée et 200 Mbits/s pour le trafic de sortie.

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-does-the-vpn-connection-i-purchase-from-my-network-service-provider-have-to-be-the-same-speed"></a>Si j’achète un circuit ExpressRoute d’une bande passante donnée, la connexion VPN que j’achète auprès de mon fournisseur de services réseau doit-elle être de la même vitesse ?

Non. Vous pouvez acheter une connexion VPN de n’importe quelle vitesse chez votre fournisseur de services. Toutefois, votre connexion à Azure est limitée à la bande passante du circuit ExpressRoute que vous achetez.

### <a name="if-i-pay-for-an-expressroute-circuit-of-a-given-bandwidth-do-i-have-the-ability-to-use-more-than-my-procured-bandwidth"></a>Si j’achète un circuit ExpressRoute d’une bande passante donnée, puis-je utiliser plus que la bande passante fournie ?

Oui, vous pouvez utiliser jusqu’à deux fois la limite de bande passante que vous avez achetée à l’aide de la bande passante disponible sur la connexion secondaire de votre circuit ExpressRoute. La redondance intégrée de votre circuit est configurée à l’aide de connexions principales et secondaires, chacune des bandes passantes fournies, sur deux routeurs Microsoft Enterprise Edge (MSEE). La bande passante disponible par le biais de votre connexion secondaire peut être utilisée pour le trafic supplémentaire, si nécessaire. Étant donné que la connexion secondaire est destinée à être redondante, elle n’est pas garantie et ne doit pas être utilisée pour le trafic supplémentaire pendant une période prolongée. Pour en savoir plus sur l’utilisation des deux connexions pour transmettre le trafic, consultez [Solution : ajoutez le préfixe AS PATH](./expressroute-optimize-routing.md#solution-use-as-path-prepending).

Si vous envisagez d’utiliser uniquement votre connexion principale pour transmettre le trafic, la bande passante de la connexion est fixe et toute tentative de sur-souscription entraînera l’augmentation des abandons de paquets. Si le trafic transite par une passerelle ExpressRoute, la bande passante de la référence (SKU) de passerelle est fixe et non burstable. Pour connaître la bande passante de chaque référence (SKU) de passerelle, consultez [À propos des passerelles de réseau virtuel ExpressRoute](expressroute-about-virtual-network-gateways.md#aggthroughput).

### <a name="can-i-use-the-same-private-network-connection-with-virtual-network-and-other-azure-services-simultaneously"></a>Puis-je utiliser la même connexion réseau privée avec un réseau virtuel et d’autres services Azure simultanément ?

Oui. Un circuit ExpressRoute, une fois configuré, vous permet d’accéder simultanément aux services au sein d’un réseau virtuel et aux autres services Azure. Vous vous connectez aux réseaux virtuels via le chemin d’accès de peering privé, et aux autres services via le chemin d’accès de peering Microsoft.

### <a name="how-are-vnets-advertised-on-expressroute-private-peering"></a>Comment les réseaux virtuels sont-ils publiés pour le peering privé ExpressRoute ?

La passerelle ExpressRoute publie le ou les *espaces d’adressage* du réseau virtuel Azure. Vous ne pouvez pas configurer d’inclusions ou d’exclusions au niveau du sous-réseau. En effet, c’est toujours l’espace d’adressage du réseau virtuel qui est publié. En outre, si le peering des réseaux virtuels est utilisé et si l’option « Utiliser la passerelle distante » est activée pour le réseau virtuel appairé, l’espace d’adressage du réseau virtuel appairé sera également publié.

### <a name="how-many-prefixes-can-be-advertised-from-a-vnet-to-on-premises-on-expressroute-private-peering"></a>Combien de préfixes peuvent être publiés d’un réseau virtuel vers un serveur local sur un Peering privé ExpressRoute ?

Au maximum 200 préfixes publiés sur une connexion ExpressRoute unique, ou via VNET Peering utilisent le transit de passerelle. Par exemple, si vous avez 199 espaces d’adressage sur un seul réseau virtuel connecté à un circuit ExpressRoute, tous ces préfixes sont publiés localement. Ou bien, si vous avez un réseau virtuel activé pour autoriser le transit par passerelle avec 1 espace d’adressage et 150 réseaux virtuels spoke activés à l’aide de l’option « Autoriser la passerelle distante », le réseau virtuel déployé avec la passerelle publie 151 préfixes localement.

### <a name="what-happens-if-i-exceed-the-prefix-limit-on-an-expressroute-connection"></a>Que se passe-t-il si je dépasse la limite de préfixes sur une connexion ExpressRoute ?

La connexion entre le circuit ExpressRoute et la passerelle (et les réseaux virtuels homologués utilisant le transit de la passerelle, le cas échéant) est interrompue. Elle se rétablit lorsque la limite de préfixes n’est plus dépassée.  

### <a name="can-i-filter-routes-coming-from-my-on-premises-network"></a>Puis-je filtrer les routes qui proviennent de mon réseau local ?

La seule façon de filtrer ou d’inclure des routes consiste à utiliser le routeur de périphérie local. Vous pouvez ajouter des routes définies par l’utilisateur au réseau virtuel afin d’affecter un certain routage. Toutefois, cela sera statique et ne fera pas partie de la publication BGP.

### <a name="does-expressroute-offer-a-service-level-agreement-sla"></a>ExpressRoute offre-t-il un contrat de niveau de service (SLA) ?

Pour plus d’informations, consultez la page [SLA ExpressRoute](https://azure.microsoft.com/support/legal/sla/) .

## <a name="supported-services"></a>Services pris en charge

ExpressRoute prend en charge [trois domaines de routage](expressroute-circuit-peerings.md) pour différents types de services : le peering privé, le peering Microsoft et le peering public (déprécié).

### <a name="private-peering"></a>Peering privé

**Pris en charge :**

* Réseaux virtuels, comprenant l’ensemble des machines virtuelles et des services cloud

### <a name="microsoft-peering"></a>Peering Microsoft

Si votre circuit ExpressRoute est activé pour le peering Microsoft Azure, vous pouvez accéder aux [plages d’adresses IP publiques](../virtual-network/public-ip-addresses.md#public-ip-addresses) utilisées dans Azure sur le circuit. Le peering Microsoft Azure permet d’accéder aux services actuellement hébergés sur Azure (avec des restrictions géographiques en fonction de la référence SKU de votre circuit). Pour valider la disponibilité d’un service spécifique, vous pouvez consulter la documentation de ce service pour voir si une plage réservée est publiée pour ce service. Recherchez ensuite les plages d’adresses IP du service cible et comparez-les aux plages listées dans [Plages d’adresses IP Azure et balises de service - Fichier XML cloud public](https://www.microsoft.com/download/details.aspx?id=56519). Vous pouvez également ouvrir un ticket de support pour le service en question afin de clarifier le problème.

**Pris en charge :**

* [Microsoft 365](/microsoft-365/enterprise/azure-expressroute)
* Power BI : disponible via une communauté régionale Azure, voir [ici](/power-bi/service-admin-where-is-my-tenant-located) pour savoir comment déterminer la région de votre locataire Power BI.
* Azure Active Directory
* [Azure DevOps](https://blogs.msdn.microsoft.com/devops/2018/10/23/expressroute-for-azure-devops/) (communauté Services globaux Azure)
* Adresses IP publiques Azure pour IaaS (machines virtuelles, passerelles de réseau virtuel, équilibreurs de charge, etc.)  
* La plupart des autres services Azure sont également pris en charge. Vérifiez directement auprès du service que vous souhaitez utiliser s’il est pris en charge.

**Non pris en charge :**

* CDN
* Azure Front Door
* [Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/)
* Serveur Multi-Factor Authentication (hérité)
* Traffic Manager

### <a name="public-peering"></a>Peering public

Le peering public a été désactivé sur les nouveaux circuits ExpressRoute. Les services Azure sont désormais disponibles sur le peering Microsoft. Si vous avez un circuit qui a été créé avant la dépréciation du peering public, vous pouvez choisir d’utiliser le peering Microsoft ou le peering public, selon les services que vous voulez obtenir.

Pour plus d’informations et pour connaître les étapes de configuration du peering public, consultez [Peering public ExpressRoute](about-public-peering.md).

### <a name="why-i-see-advertised-public-prefixes-status-as-validation-needed-while-configuring-microsoft-peering"></a>Pourquoi l’état « Préfixes publics publiés » est-il « Validation requise » lors de la configuration du peering Microsoft ?

Microsoft vérifie si les « préfixes publics publiés » et « ASN pairs » (ou « ASN client ») spécifiés vous sont attribués dans le registre de routage Internet. Si vous obtenez les préfixes publics d’une autre entité et si l’affectation n’est pas enregistrée avec le registre de routage, la validation automatique ne se termine pas et nécessite une validation manuelle. Si la validation automatique échoue, le message « Validation nécessaire » s’affiche.

Si vous voyez le message « Validation nécessaire », collectez le ou les documents qui affichent les préfixes publics qui sont attribués à votre organisation par l’entité répertoriée comme propriétaire des préfixes dans le registre de routage et soumettez ces documents pour validation manuelle en ouvrant un ticket de support comme indiqué ci-dessous.

![Capture d’écran montrant une nouvelle demande de support (ticket de support) pour « preuve de possession des préfixes publics ».](./media/expressroute-faqs/ticket-portal-msftpeering-prefix-validation.png)

### <a name="is-dynamics-365-supported-on-expressroute"></a>Dynamics 365 est-il pris en charge dans ExpressRoute ?

Les environnements Dynamics 365 et Common Data Service (CDS) sont hébergés dans Azure. Par conséquent, les clients bénéficient de la prise en charge ExpressRoute sous-jacente des ressources Azure. Vous pouvez vous connecter à ses points de terminaison de service si votre filtre de routeur contient les régions Azure où sont hébergés les environnements Dynamics 365/CDS.

> [!NOTE]
> [ExpressRoute Premium](#expressroute-premium) n’est **pas** nécessaire à la connectivité de Dynamics 365 via Azure ExpressRoute si le circuit ExpressRoute est déployé dans la même [région géopolitique](./expressroute-locations-providers.md#expressroute-locations).

## <a name="data-and-connections"></a>Données et connexions

### <a name="are-there-limits-on-the-amount-of-data-that-i-can-transfer-using-expressroute"></a>Existe-t-il des limites sur la quantité de données que je peux transférer avec ExpressRoute ?

Nous ne définissons aucune limite sur la quantité de transfert de données. Pour plus d'informations sur les tarifs de bande passante, consultez la page de [tarification](https://azure.microsoft.com/pricing/details/expressroute/) .

### <a name="what-connection-speeds-are-supported-by-expressroute"></a>Quelles vitesses de connexion sont prises en charge par ExpressRoute ?

Offres relatives à la bande passante prise en charge :

50 Mbit/s, 100 Mbit/s, 200 Mbit/s, 500 Mbit/s, 1 Gbit/s, 2 Gbit/s, 5 Gbit/s, 10 Gbit/s

### <a name="which-service-providers-are-available"></a>Quels fournisseurs de services sont disponibles ?

Pour obtenir la liste des fournisseurs de services et des emplacements, consultez la page [Partenaires et emplacements ExpressRoute](expressroute-locations.md) .

## <a name="technical-details"></a>Détails techniques

### <a name="what-are-the-technical-requirements-for-connecting-my-on-premises-location-to-azure"></a>Quelles sont les exigences techniques pour connecter mon emplacement local à Azure ?

Consultez [la page conditions préalables d’ExpressRoute](expressroute-prerequisites.md) pour la configuration requise.

### <a name="are-connections-to-expressroute-redundant"></a>Les connexions à ExpressRoute sont-elles redondantes ?

Oui. Chaque circuit ExpressRoute dispose d’une paire redondante de connexions croisées configurées pour fournir une haute disponibilité.

### <a name="will-i-lose-connectivity-if-one-of-my-expressroute-links-fail"></a>Vais-je perdre ma connectivité en cas d’échec de l’un de mes liens ExpressRoute ?

Vous ne perdez pas votre connectivité si une des connexions croisées échoue. Une connexion redondante est disponible pour gérer la charge de votre réseau et fournir une haute disponibilité de votre circuit ExpressRoute. Vous pouvez également créer un circuit dans un autre emplacement de peering pour bénéficier de la résilience au niveau du circuit.

### <a name="how-do-i-implement-redundancy-on-private-peering"></a>Comment implémenter la redondance sur le peering privé ?

Plusieurs circuits ExpressRoute provenant de différents emplacements de peering, ou jusqu’à quatre connexions provenant du même emplacement de peering, peuvent être connectés au même réseau virtuel afin de fournir une haute disponibilité en cas d’indisponibilité de l’un des circuits. Vous pouvez ensuite [attribuer une pondération plus élevée](./expressroute-optimize-routing.md#solution-assign-a-high-weight-to-local-connection) à l’une des connexions locales pour favoriser un circuit en particulier. Il est fortement recommandé que les clients configurent au moins deux circuits ExpressRoute pour éviter les points de défaillance uniques. 

Cliquez [ici](./designing-for-high-availability-with-expressroute.md) pour en savoir plus sur la conception pour une haute disponibilité et [ici](./designing-for-disaster-recovery-with-expressroute-privatepeering.md) pour la conception de la récupération d’urgence.  

### <a name="how-i-do-implement-redundancy-on-microsoft-peering"></a>Comment implémenter la redondance sur le peering Microsoft ?

Il est vivement recommandé aux clients qui utilisent un Peering Microsoft pour accéder à des services publics Azure tels que Stockage Azure ou Azure SQL, ainsi qu’aux clients qui utilisent un Peering Microsoft pour Microsoft 365, d’implémenter plusieurs circuits à différents emplacements de Peering pour éviter des points de défaillance uniques. Les clients peuvent soit publier le même préfixe sur les deux circuits et utiliser la [préfixation AS PATH](./expressroute-optimize-routing.md#solution-use-as-path-prepending), soit publier différents préfixes pour déterminer le chemin d’accès à partir du système local.

Cliquez [ici](./designing-for-high-availability-with-expressroute.md) pour en savoir plus sur la conception pour une haute disponibilité.

### <a name="how-do-i-ensure-high-availability-on-a-virtual-network-connected-to-expressroute"></a>Comment garantir une haute disponibilité sur un réseau virtuel connecté à ExpressRoute ?

Vous pouvez obtenir une haute disponibilité en connectant jusqu’à quatre circuits ExpressRoute du même emplacement de peering à votre réseau virtuel, ou en connectant des circuits ExpressRoute à différents emplacements de peering (par exemple, Singapour, Singapour2) à votre réseau virtuel. Si un circuit ExpressRoute tombe en panne, la connectivité bascule vers un autre circuit ExpressRoute. Par défaut, le trafic qui quitte votre réseau virtuel est acheminé selon le principe de routage ECMP (Equal Cost Multi-path Routing). Vous pouvez utiliser le poids de connexion pour choisir un circuit plutôt qu’un autre. Pour plus d’informations, voir [Optimisation du routage ExpressRoute](expressroute-optimize-routing.md).

### <a name="how-do-i-ensure-that-my-traffic-destined-for-azure-public-services-like-azure-storage-and-azure-sql-on-microsoft-peering-or-public-peering-is-preferred-on-the-expressroute-path"></a>Comment puis-je m’assurer que mon trafic destiné aux services publics Azure comme Stockage Azure et Azure SQL sur le peering Microsoft ou le peering public a la préférence sur le chemin ExpressRoute ?

Vous devez implémenter l’attribut *Préférence Locale* sur votre ou vos routeurs pour vous assurer que le chemin d’accès de l’installation à Azure est toujours préféré sur votre ou vos circuits ExpressRoute.

Pour en savoir plus sur la sélection du chemin BGP et les configurations de routeur courantes, veuillez cliquer [ici](./expressroute-optimize-routing.md#path-selection-on-microsoft-and-public-peerings) sur la. 

### <a name="if-im-not-co-located-at-a-cloud-exchange-and-my-service-provider-offers-point-to-point-connection-do-i-need-to-order-two-physical-connections-between-my-on-premises-network-and-microsoft"></a><a name="onep2plink"></a>Si je ne suis pas colocalisé au niveau d’un échange de cloud et que mon fournisseur de services offre une connexion point à point, dois-je commander deux connexions physiques entre mon réseau local et Microsoft ?

Vous n’avez besoin que d’une seule connexion physique si votre fournisseur de services peut établir deux circuits virtuels Ethernet via la connexion physique. La connexion physique (par exemple, une fibre optique) se termine sur un appareil de couche 1 (L1) (voir l’image). Les deux circuits virtuels Ethernet sont marqués avec des ID de VLAN différents, l’un pour le circuit principal et l’autre pour le circuit secondaire. Ces ID de VLAN se trouvent dans l’en-tête Ethernet 802.1Q externe. L’en-tête Ethernet 802.1Q interne (non illustré) est mappé à un [domaine de routage ExpressRoute](expressroute-circuit-peerings.md)spécifique.

![Diagramme mettant en surbrillance les circuits virtuels principaux et secondaires de couche 1 (L1) qui composent la connexion physique entre les commutateurs sur un site client et un emplacement ExpressRoute.](./media/expressroute-faqs/expressroute-p2p-ref-arch.png)

### <a name="can-i-extend-one-of-my-vlans-to-azure-using-expressroute"></a>Puis-je étendre l’un de mes réseaux locaux virtuels vers Azure avec ExpressRoute ?

Non. Nous ne prenons pas en charge les extensions de connectivité de couche 2 dans Azure.

### <a name="can-i-have-more-than-one-expressroute-circuit-in-my-subscription"></a>Puis-je avoir plusieurs circuits ExpressRoute dans mon abonnement ?

Oui. Vous pouvez avoir plusieurs circuits ExpressRoute dans votre abonnement. La limite par défaut est définie sur 10. Vous pouvez contacter le support technique de Microsoft pour augmenter la limite, si nécessaire.

### <a name="can-i-have-expressroute-circuits-from-different-service-providers"></a>Puis-je avoir des circuits ExpressRoute de différents fournisseurs de services ?

Oui. Vous pouvez avoir des circuits ExpressRoute de nombreux fournisseurs de services. Chaque circuit ExpressRoute est associé uniquement à un fournisseur de services. 

### <a name="i-see-two-expressroute-peering-locations-in-the-same-metro-for-example-singapore-and-singapore2-which-peering-location-should-i-choose-to-create-my-expressroute-circuit"></a>Je vois deux emplacements de peering ExpressRoute dans le même métro, par exemple, Singapore et Singapore2. Quel emplacement de peering dois-je choisir pour créer mon circuit ExpressRoute ?
Si votre fournisseur de services propose ExpressRoute sur les deux sites, vous pouvez collaborer avec lui et choisir l’un ou l’autre site pour configurer ExpressRoute. 

### <a name="can-i-have-multiple-expressroute-circuits-in-the-same-metro-can-i-link-them-to-the-same-virtual-network"></a>Puis-je avoir plusieurs circuits ExpressRoute dans le même métro ? Puis-je les lier au même réseau virtuel ?

Oui. Vous pouvez avoir plusieurs circuits ExpressRoute, avec des fournisseurs de services identiques ou différents. Si le métro a plusieurs emplacements de peering ExpressRoute et que les circuits sont créés à différents emplacements de peering, vous pouvez les lier au même réseau virtuel. Si les circuits sont créés au même emplacement de peering, vous pouvez lier jusqu’à quatre circuits au même réseau virtuel.

### <a name="how-do-i-connect-my-virtual-networks-to-an-expressroute-circuit"></a>Comment connecter mes réseaux virtuels à un circuit ExpressRoute ?

Étapes élémentaires :

* Établissez un circuit ExpressRoute et faites-le activer par le fournisseur de services.
* Vous, ou le fournisseur, devez configurer les peerings BGP.
* Liez le réseau virtuel au circuit ExpressRoute.

Pour plus d’informations, consultez [flux de travail ExpressRoute pour la configuration de circuits et les états des circuits](expressroute-workflows.md).

### <a name="are-there-connectivity-boundaries-for-my-expressroute-circuit"></a>Existe-t-il des limites de connectivité pour le circuit ExpressRoute ?

Oui. L’article [partenaires et emplacements ExpressRoute](expressroute-locations.md) offre une vue d’ensemble des limites de connectivité d’un circuit ExpressRoute. La connectivité d’un circuit ExpressRoute est limitée à une seule région géopolitique. La connectivité peut être étendue pour traverser des régions géopolitiques en activant la fonctionnalité Premium d’ExpressRoute.

### <a name="can-i-link-to-more-than-one-virtual-network-to-an-expressroute-circuit"></a>Puis-je lier plusieurs réseaux virtuels à un circuit ExpressRoute ?

Oui. Vous pouvez avoir jusqu’à 10 connexions réseaux virtuels sur un circuit ExpressRoute standard et jusqu’à 100 sur un [circuit ExpressRoute premium](#expressroute-premium). 

### <a name="i-have-multiple-azure-subscriptions-that-contain-virtual-networks-can-i-connect-virtual-networks-that-are-in-separate-subscriptions-to-a-single-expressroute-circuit"></a>Je possède plusieurs abonnements Azure qui contiennent des réseaux virtuels. Puis-je connecter des réseaux virtuels qui figurent dans des abonnements distincts à un circuit ExpressRoute ?

Oui. Vous pouvez lier jusqu’à 10 réseaux virtuels dans le même abonnement que le circuit, ou dans différents abonnements par l’intermédiaire d’un circuit ExpressRoute unique. Cette limite peut être augmentée en activant la fonctionnalité Premium d’ExpressRoute. Notez que les frais de connectivité et de bande passante pour le circuit dédié s’appliquent au propriétaire du circuit ExpressRoute ; tous les réseaux virtuels partagent la même bande passante.

Pour plus d'informations, consultez la page [Partage d'un circuit ExpressRoute entre plusieurs abonnements](expressroute-howto-linkvnet-arm.md).

### <a name="i-have-multiple-azure-subscriptions-associated-to-different-azure-active-directory-tenants-or-enterprise-agreement-enrollments-can-i-connect-virtual-networks-that-are-in-separate-tenants-and-enrollments-to-a-single-expressroute-circuit-not-in-the-same-tenant-or-enrollment"></a>J’ai plusieurs abonnements Azure associés à différents clients Azure Active Directory ou inscriptions d’Accord Entreprise. Puis-je connecter des réseaux virtuels qui se trouvent dans des clients et des inscriptions distincts à un même circuit ExpressRoute qui ne se trouve pas dans le même client ou la même inscription ?

Oui. Les autorisations ExpressRoute peuvent aller jusqu’aux limites de l’abonnement, du client et de l’inscription sans aucune configuration supplémentaire. Notez que les frais de connectivité et de bande passante pour le circuit dédié s’appliquent au propriétaire du circuit ExpressRoute ; tous les réseaux virtuels partagent la même bande passante.

Pour plus d'informations, consultez la page [Partage d'un circuit ExpressRoute entre plusieurs abonnements](expressroute-howto-linkvnet-arm.md).

### <a name="are-virtual-networks-connected-to-the-same-circuit-isolated-from-each-other"></a>Les réseaux virtuels sont-ils connectés à un même circuit en étant isolés les uns des autres ?

Non. Dans une perspective de routage, l’ensemble des réseaux virtuels liés au même circuit ExpressRoute fait partie du même domaine de routage et ne sont pas isolés les un des autres. Si vous devez isoler des itinéraires, vous devez créer un circuit ExpressRoute distinct.

### <a name="can-i-have-one-virtual-network-connected-to-more-than-one-expressroute-circuit"></a>Puis-je avoir un seul réseau virtuel connecté à plusieurs circuits ExpressRoute ?

Oui. Vous pouvez lier un réseau virtuel avec quatre circuits ExpressRoute au maximum, au même emplacement de peering ou à des emplacements différents. 

### <a name="can-i-access-the-internet-from-my-virtual-networks-connected-to-expressroute-circuits"></a>Puis-je accéder à Internet à partir de mes réseaux virtuels connectés à des circuits ExpressRoute ?

Oui. Si vous n’avez pas publié les itinéraires par défaut (0.0.0.0/0) ou les préfixes d’itinéraire Internet via la session BGP, vous pouvez vous connecter à Internet à partir d’un réseau virtuel lié à un circuit ExpressRoute.

### <a name="can-i-block-internet-connectivity-to-virtual-networks-connected-to-expressroute-circuits"></a>Puis-je bloquer connectivité Internet pour les réseaux virtuels connectés à des circuits ExpressRoute ?

Oui. Vous pouvez publier des itinéraires par défaut (0.0.0.0/0) pour bloquer la connectivité Internet de toutes les machines virtuelles qui sont déployées au sein d’un réseau virtuel et qui acheminent tout le trafic sortant via le circuit ExpressRoute.

> [!NOTE]
> Si l’itinéraire publié 0.0.0.0/0 est retiré des itinéraires publiés (par exemple, en raison d’une panne ou d’une configuration incorrecte), Azure fournit un [ltinéraire du système](../virtual-network/virtual-networks-udr-overview.md#system-routes) aux ressources du réseau virtuel connecté pour assurer la connectivité à Internet.  Pour vous assurer que le trafic sortant vers Internet est bloqué, il est recommandé de placer un groupe de sécurité réseau sur tous les sous-réseaux avec une règle de refus sortante pour le trafic Internet.

Si vous publiez des itinéraires par défaut, nous forçons le réacheminement du trafic en direction des services offerts via le peering Microsoft (tels que le Stockage Azure et SQL DB) vers votre environnement local. Vous devez configurer vos routeurs de façon à ce qu’ils retournent le trafic vers Azure via le chemin d’accès de peering Microsoft ou via Internet. Si vous avez activé un point de terminaison de service pour le service, le trafic vers le service n’est pas réacheminé vers votre site. Le trafic reste dans le réseau principal Azure. Découvrez plus d’informations sur les points de terminaison de service dans [Points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md?toc=%2fazure%2fexpressroute%2ftoc.json).

### <a name="can-virtual-networks-linked-to-the-same-expressroute-circuit-talk-to-each-other"></a>Les réseaux virtuels liés à un même circuit ExpressRoute peuvent-ils communiquer entre eux ?

Oui. Les machines virtuelles qui sont déployées dans des réseaux virtuels connectés à un même circuit ExpressRoute peuvent communiquer entre elles.

### <a name="can-i-use-site-to-site-connectivity-for-virtual-networks-in-conjunction-with-expressroute"></a>Puis-je utiliser une connectivité de site à site pour les réseaux virtuels conjointement avec ExpressRoute ?

Oui. ExpressRoute peut coexister avec des réseaux VPN de site à site. Voir [Configurer la coexistence de connexions de site à site et ExpressRoute](expressroute-howto-coexist-resource-manager.md).

### <a name="why-is-there-a-public-ip-address-associated-with-the-expressroute-gateway-on-a-virtual-network"></a>Pourquoi une adresse IP publique est-elle associée à la passerelle ExpressRoute sur un réseau virtuel ?

L’adresse IP publique est utilisée uniquement pour la gestion interne, et ne constitue pas un risque pour la sécurité de votre réseau virtuel.

### <a name="are-there-limits-on-the-number-of-routes-i-can-advertise"></a>Existe-t-il des limites sur le nombre d’itinéraires que je peux publier ?

Oui. Nous acceptons jusqu’à 4 000 préfixes d’itinéraires pour un peering privé, et 200 pour un peering Microsoft. Vous pouvez augmenter ce nombre jusqu’à 10 000 itinéraires par peering privé si vous activez la fonctionnalité Premium d’ExpressRoute.

### <a name="are-there-restrictions-on-ip-ranges-i-can-advertise-over-the-bgp-session"></a>Existe-t-il des restrictions de plages d’adresses IP que je peux publier sur la session BGP ?

Nous n’acceptons pas les préfixes privés (RFC1918) pour la session BGP de peering Microsoft. Nous acceptons n’importe quelle taille de préfixe (jusqu'à /32) sur Microsoft et sur le peering privé.

### <a name="what-happens-if-i-exceed-the-bgp-limits"></a>Que se passe-t-il si je dépasse les limites du protocole BGP ?

Les sessions BGP sont supprimées. Elles sont ensuite réinitialisées lorsque le nombre de préfixes tombe en dessous de la limite.

### <a name="what-is-the-expressroute-bgp-hold-time-can-it-be-adjusted"></a>Quelle est la durée de conservation BGP d'ExpressRoute ? Peut-elle être ajustée ?

La durée de conservation est de 180. Les messages persistants sont envoyés toutes les 60 secondes. Il s'agit de paramètres fixes de Microsoft qui ne peuvent pas être modifiés. Vous pouvez configurer différents minuteurs et les paramètres de la session BGP seront négociés en conséquence.

### <a name="can-i-change-the-bandwidth-of-an-expressroute-circuit"></a>Puis-je modifier la bande passante d’un circuit ExpressRoute ?

Oui, vous pouvez tenter d’augmenter la bande passante de votre circuit ExpressRoute dans le portail Azure, ou à l’aide de PowerShell. Si il y a de la capacité disponible sur le port physique sur lequel votre circuit a été créé, votre modification a réussi. 

Si votre modification échoue, cela signifie peut signifier qu’il ne reste pas suffisamment de capacité sur le port actuel et que vous devez créer un nouveau circuit ExpressRoute avec une plus grande bande passante, ou qu’il n’y a pas de capacité supplémentaire à cet emplacement. Dans ce cas vous ne pourrez pas augmenter la bande passante. 

Vous devez également effectuer un suivi avec votre fournisseur de connectivité pour vous assurer qu’il met à jour les limitations dans ses réseaux pour prendre en charge l’augmentation de la bande passante. Vous ne pouvez toutefois pas réduire la bande passante de votre circuit ExpressRoute. Vous devez créer un nouveau circuit ExpressRoute avec une bande passante inférieure et supprimer l’ancien circuit.

### <a name="how-do-i-change-the-bandwidth-of-an-expressroute-circuit"></a>Comment modifier la bande passante d’un circuit ExpressRoute ?

Vous pouvez mettre à jour de la bande passante du circuit ExpressRoute à l’aide de l’API REST ou l’applet de commande PowerShell.

## <a name="expressroute-premium"></a>ExpressRoute Premium

### <a name="what-is-expressroute-premium"></a>En quoi consiste ExpressRoute Premium ?

ExpressRoute Premium est un ensemble de fonctionnalités répertoriées ci-dessous.

* Augmentation de la limite de la table d’itinéraires de 4 000 à 10 000 itinéraires pour le peering privé.
* Augmentation du nombre de réseaux virtuels et de connexions ExpressRoute Global Reach activables sur un circuit ExpressRoute (la valeur par défaut est 10). Pour plus d’informations, consultez le tableau [Limites d’ExpressRoute](#limits).
* Connectivité à Microsoft 365
* Connectivité globale sur le réseau principal Microsoft. Vous pouvez désormais lier un réseau virtuel dans une région géopolitique à un circuit ExpressRoute d’une autre région.<br>
    **Exemples :**

    *  Vous pouvez lier un réseau virtuel créé en Europe de l’ouest à un circuit créé dans la Silicon Valley. 
    *  Sur le peering Microsoft, les préfixes d’autres régions géopolitiques sont publiés de sorte que vous pouvez vous connecter, par exemple, à SQL Azure en Europe de l’Ouest à partir d’un circuit situé dans la Silicon Valley.


### <a name="how-many-vnets-and-expressroute-global-reach-connections-can-i-enable-on-an-expressroute-circuit-if-i-enabled-expressroute-premium"></a><a name="limits"></a>Combien de réseaux virtuels et de connexions ExpressRoute Global Reach puis-je activer sur un circuit ExpressRoute si j’ai activé ExpressRoute Premium ?

Les tableaux suivants indiquent les limites d’ExpressRoute et le nombre de réseaux virtuels et de connexions ExpressRoute Global Reach par circuit ExpressRoute :

[!INCLUDE [ExpressRoute limits](../../includes/expressroute-limits.md)]

### <a name="how-do-i-enable-expressroute-premium"></a>Comment activer ExpressRoute Premium ?

Les fonctionnalités premium ExpressRoute peuvent être activées lorsque la fonctionnalité est activée et peuvent être arrêtées en mettant à jour l’état du circuit. Vous pouvez activer ExpressRoute Premium quand vous créez le circuit ou vous pouvez appeler l’API REST/l’applet de commande PowerShell pour activer ExpressRoute Premium.

### <a name="how-do-i-disable-expressroute-premium"></a>Comment désactiver ExpressRoute Premium ?

Vous pouvez désactiver ExpressRoute Premium en appelant l’API REST/l’applet de commande PowerShell. Vous devez vous assurer que vous avez redimensionné vos besoins en connectivité afin de respecter les limites par défaut avant de désactiver ExpressRoute Premium. Si votre utilisation met à l’échelle au-delà des limites par défaut, la requête de désactivation d’ExpressRoute Premium échoue.

### <a name="can-i-pick-and-choose-the-features-i-want-from-the-premium-feature-set"></a>Puis-je choisir les fonctionnalités parmi l’ensemble des fonctionnalités Premium ?

Non. Vous ne pouvez pas sélectionner les fonctionnalités. Nous activons toutes les fonctionnalités lorsque vous activez ExpressRoute Premium.

### <a name="how-much-does-expressroute-premium-cost"></a>Combien coûte ExpressRoute Premium ?

Consultez la page de [tarification](https://azure.microsoft.com/pricing/details/expressroute/) pour connaître les coûts.

### <a name="do-i-pay-for-expressroute-premium-in-addition-to-standard-expressroute-charges"></a>Dois-je payer pour ExpressRoute Premium en plus des frais ExpressRoute standard ?

Oui. Les frais d’ExpressRoute Premium s’ajoutent aux frais de circuit ExpressRoute et aux frais du fournisseur de connectivité.

## <a name="expressroute-local"></a>Emplacements ExpressRoute Local
### <a name="what-is-expressroute-local"></a>Présentation d’ExpressRoute Local
ExpressRoute Local est une référence SKU du circuit ExpressRoute, en plus des références SKU Standard et Premium. Une fonctionnalité clé de Local est qu’un circuit Local à un emplacement de peering ExpressRoute vous donne accès uniquement à une ou deux régions Azure au sein ou à proximité du même metro. En revanche, un circuit Standard vous donne accès à toutes les régions Azure dans une zone géopolitique, et un circuit Premium à toutes les régions Azure du monde. 

### <a name="what-are-the-benefits-of-expressroute-local"></a>Quels sont les avantages d’ExpressRoute Local ?
Alors que vous devez payer le transfert de données de sortie pour votre circuit ExpressRoute Standard ou Premium, vous ne payez pas de transfert de données de sortie séparément pour votre circuit ExpressRoute Local. En d’autres termes, le prix d’ExpressRoute Local inclut les frais de transfert de données. ExpressRoute Local est une solution plus économique si vous avez des quantités massives de données à transférer, et vous pouvez faire amener vos données à un emplacement de peering ExpressRoute proche de vos régions Azure souhaitées via une connexion privée. 

### <a name="what-features-are-available-and-what-are-not-on-expressroute-local"></a>Quelles sont les fonctionnalités respectivement disponibles et non disponibles sur ExpressRoute Local ?
Par rapport à un circuit ExpressRoute Standard, un circuit Local a le même ensemble de fonctionnalités à l’exception des suivantes :
* Étendue de l’accès à des régions Azure comme décrit ci-dessus
* ExpressRoute Global Reach n’est pas disponible sur Local

ExpressRoute Local a les mêmes limites de ressources (par exemple, le nombre de réseaux virtuels par circuit) que Standard. 

### <a name="where-is-expressroute-local-available-and-which-azure-regions-is-each-peering-location-mapped-to"></a>Où ExpressRoute Local est-il disponible et à quelles régions Azure chaque emplacement de peering est-il mappé ?
ExpressRoute Local est disponible aux emplacements de peering proches d’une ou de deux régions Azure. Il n’est pas disponible à un emplacement de peering situé dans un état, une province ou un pays/région où il n’y a pas de région Azure. Consultez les mappages exacts à [la page Emplacements](expressroute-locations-providers.md).  

## <a name="expressroute-for-microsoft-365"></a>ExpressRoute pour Microsoft 365

[!INCLUDE [expressroute-office365-include](../../includes/expressroute-office365-include.md)]

### <a name="how-do-i-create-an-expressroute-circuit-to-connect-to-microsoft-365-services"></a>Comment créer un circuit ExpressRoute pour se connecter à des services Microsoft 365 ?

1. Consultez la [page des conditions préalables d’ExpressRoute](expressroute-prerequisites.md) pour vérifier que vous avez respecté les conditions.
2. Pour vous assurer que vos besoins de connectivité sont satisfaisants, examinez la liste des fournisseurs de services et les emplacements dans l’article [Partenaires d’ExpressRoute partenaires et emplacements](expressroute-locations.md).
3. Planifiez vos besoins en capacité en consultant la page [Planification réseau et optimisation des performances pour Microsoft 365](/microsoft-365/enterprise/network-planning-and-performance).
4. Suivez les étapes répertoriées dans les flux de travail pour configurer la connectivité aux [flux de travail ExpressRoute pour l’approvisionnement et les états du circuit](expressroute-workflows.md).

> [!IMPORTANT]
> Assurez-vous d’avoir activé le module complémentaire ExpressRoute lors de la configuration de la connectivité aux services Microsoft 365.
> 
> 

### <a name="can-my-existing-expressroute-circuits-support-connectivity-to-microsoft-365-services"></a>Mes circuits ExpressRoute existants peuvent-ils prendre en charge la connectivité aux services Microsoft 365 ?

Oui. Votre circuit ExpressRoute existant peut être configuré pour prendre en charge la connectivité aux services Microsoft 365. Assurez-vous d’avoir une capacité suffisante pour vous connecter aux services Microsoft 365 et d’avoir activé le module complémentaire Premium. [Planification réseau et optimisation des performances pour Microsoft 365](/microsoft-365/enterprise/network-planning-and-performance) vous aide à prévoir vos besoins en matière de connectivité. Voir également [Création et modification d’un circuit ExpressRoute](expressroute-howto-circuit-classic.md).

### <a name="what-microsoft-365-services-can-be-accessed-over-an-expressroute-connection"></a>Quels services Microsoft 365 sont accessibles via une connexion ExpressRoute ?

Reportez-vous à la page [URL et plages d’adresses IP Microsoft 365](/microsoft-365/enterprise/urls-and-ip-address-ranges) pour obtenir une liste à jour des services pris en charge par le biais d’ExpressRoute.

### <a name="how-much-does-expressroute-for-microsoft-365-services-cost"></a>Combien coûte ExpressRoute pour les services Microsoft 365 ?

Les services Microsoft 365 nécessitent l’activation d’un module complémentaire Premium. Consultez la [page de détails des prix appliqués](https://azure.microsoft.com/pricing/details/expressroute/) pour les coûts.

### <a name="what-regions-is-expressroute-for-microsoft-365-supported-in"></a>Quelles régions sont prises en charge dans ExpressRoute pour Microsoft 365 ?

Consultez [Partenaires et emplacements ExpressRoute ](expressroute-locations.md) pour plus d’informations.

### <a name="can-i-access-microsoft-365-over-the-internet-even-if-expressroute-was-configured-for-my-organization"></a>Puis-je accéder à Microsoft 365 via Internet même si ExpressRoute a été configuré pour mon organisation ?

Oui. Les points de terminaison du service Microsoft 365 sont accessibles via Internet, même si ExpressRoute a été configuré pour votre réseau. Consultez votre équipe réseau pour savoir si le réseau de votre organisation est configuré pour se connecter aux services Microsoft 365 via ExpressRoute.

### <a name="how-can-i-plan-for-high-availability-for-microsoft-365-network-traffic-on-azure-expressroute"></a>Comment planifier la haute disponibilité du trafic réseau Microsoft 365 sur Azure ExpressRoute ?
Consultez la recommandation en matière de [Haute disponibilité et basculement avec Azure ExpressRoute](/microsoft-365/enterprise/network-planning-with-expressroute).

### <a name="can-i-access-office-365-us-government-community-gcc-services-over-an-azure-us-government-expressroute-circuit"></a>Puis-je accéder aux services Office 365 US Government Community (GCC) sur un circuit Azure US Government ExpressRoute ?

Oui. Les points de terminaison du service Office 365 GCC sont accessibles via Azure US Government ExpressRoute. Toutefois, vous devez d’abord ouvrir un ticket de support sur le portail Azure pour fournir à Microsoft les préfixes que vous avez l’intention de publier. La connectivité aux services Office 365 GCC sera établie une fois le ticket de support résolu. 

## <a name="route-filters-for-microsoft-peering"></a>Filtres de routage pour le peering Microsoft

### <a name="i-am-turning-on-microsoft-peering-for-the-first-time-what-routes-will-i-see"></a>J’active le peering Microsoft pour la première fois, quels itinéraires s’afficheront ?

Aucun itinéraire ne s’affichera. Vous devez joindre un filtre de routage à votre circuit pour démarrer des publications de préfixe. Consultez [Configurer des filtres de routage pour le peering Microsoft](how-to-routefilter-powershell.md) pour obtenir des instructions.

### <a name="i-turned-on-microsoft-peering-and-now-i-am-trying-to-select-exchange-online-but-it-is-giving-me-an-error-that-i-am-not-authorized-to-do-it"></a>J’ai activé le peering Microsoft et maintenant j’essaie de sélectionner Exchange Online, mais il affiche une erreur m’indiquant que je ne suis pas autorisé à le faire.

Lorsque vous utilisez des filtres de routage, n’importe quel client peut activer le peering Microsoft. Toutefois, pour utiliser les services Microsoft 365, vous devez toujours obtenir l’autorisation de Microsoft 365.

### <a name="i-enabled-microsoft-peering-prior-to-august-1-2017-how-can-i-take-advantage-of-route-filters"></a>J’ai activé le peering Microsoft avant le 1er août 2017. Comment puis-je tirer parti des filtres de routage ?

Votre circuit existant continuera à publier des préfixes pour Microsoft 365. Si vous souhaitez ajouter des publications de préfixes publics Azure sur le même Peering Microsoft, vous pouvez créer un filtre de routage, sélectionner les services que vous souhaitez publier (notamment les services Microsoft 365 dont vous avez besoin) et joindre le filtre à votre Peering Microsoft. Consultez [Configurer des filtres de routage pour le peering Microsoft](how-to-routefilter-powershell.md) pour obtenir des instructions.

### <a name="i-have-microsoft-peering-at-one-location-now-i-am-trying-to-enable-it-at-another-location-and-i-am-not-seeing-any-prefixes"></a>Je dispose du peering Microsoft à un emplacement, maintenant j’essaie de l’activer à un autre emplacement et aucun préfixe ne s’affiche.

* Le peering Microsoft des circuits ExpressRoute configurés avant le 1er août 2017 entraînera la publication de tous les préfixes de service via le peering Microsoft, même si les filtres de routage ne sont pas définis.

* Le peering Microsoft des circuits ExpressRoute qui sont configurés le 1er août 2017 ou après n’entraînera la publication d’aucun préfixe tant qu’un filtre de routage n’aura pas été attaché au circuit. Aucun préfixe par défaut ne s’affichera.

## <a name="expressroute-direct"></a><a name="expressRouteDirect"></a>ExpressRoute Direct

[!INCLUDE [ExpressRoute Direct](../../includes/expressroute-direct-faq-include.md)]

## <a name="global-reach"></a><a name="globalreach"></a>Global Reach

[!INCLUDE [Global Reach](../../includes/expressroute-global-reach-faq-include.md)]

## <a name="privacy"></a>Confidentialité

### <a name="does-the-expressroute-service-store-customer-data"></a>Le service ExpressRoute stocke-t-il des données client ?

Non.