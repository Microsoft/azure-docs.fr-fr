---
title: Fichier include
description: Fichier include
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 09/02/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c476fe2af1fb4bddff4799609b164aab14f57b00
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187394"
---
### <a name="is-azure-virtual-wan-in-ga"></a>Azure Virtual WAN est-il en disponibilité générale ?

Oui, Azure Virtual WAN est en disponibilité générale. Toutefois, le service Virtual WAN est constitué de plusieurs fonctionnalités et scénarios. Il existe des fonctionnalités ou scénarios dans le service Virtual WAN où Microsoft applique la balise Aperçu. Dans ce cas, la fonctionnalité spécifique, ou le scénario lui-même, sont en préversion. Si vous n’utilisez pas de fonctionnalité d’aperçu spécifique, la prise en charge standard en disponibilité générale s’applique. Pour plus d’informations sur la prise en charge de la préversion, consultez [Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="does-the-user-need-to-have-hub-and-spoke-with-sd-wanvpn-devices-to-use-azure-virtual-wan"></a>L’utilisateur doit-il disposer d’une architecture hub-and-spoke avec des appareils SD-WAN/VPN pour utiliser Azure Virtual WAN ?

Virtual WAN propose de nombreuses fonctionnalités regroupées dans un seul écran, notamment : connectivité VPN site/site à site, connectivité utilisateur/P2S, connectivité ExpressRoute, connectivité Réseau virtuel, interconnectivité VPN ExpressRoute, connectivité transitive de réseau virtuel à réseau virtuel, routage centralisé, sécurité avec le Pare-feu Azure et le gestionnaire de pare-feu, supervision, chiffrement ExpressRoute, etc. Il n’est pas nécessaire d’avoir tous ces cas d’usage pour commencer à utiliser Virtual WAN. Vous pouvez commencer avec un seul cas d’usage. Virtual WAN repose sur une architecture hub-and-spoke avec des fonctionnalités de mise à l’échelle et de performances intégrées. Les branches (appareils VPN/SD-WAN), les utilisateurs (clients Azure VPN, openVPN ou IKEv2), les circuits ExpressRoute et les réseaux virtuels sont les spokes (« rayons ») du ou des hubs (« moyeux ») virtuels. Tous les hubs sont connectés selon un maillage complet dans un réseau étendu virtuel standard, ce qui permet à l’utilisateur d’utiliser facilement le réseau principal Microsoft pour se connecter à n’importe quel spoke. Pour l’architecture hub-and-spoke avec des appareils SD-WAN/VPN, les utilisateurs peuvent soit la configurer manuellement dans le portail Azure Virtual WAN, soit utiliser le Virtual WAN Partner CPE (SD-WAN/VPN) pour configurer la connectivité à Azure. Les partenaires Virtual WAN assurent l’automatisation de la connectivité, c’est-à-dire la capacité d’exporter les informations sur l’appareil dans Azure, de télécharger la configuration Azure et d’établir la connectivité avec le hub Azure Virtual WAN. Pour la connectivité VPN point à site/utilisateur, nous prenons en charge le [client Azure VPN](https://go.microsoft.com/fwlink/?linkid=2117554), le client OpenVPN ou le client IKEv2. 

### <a name="can-you-disable-fully-meshed-hubs-in-a-virtual-wan"></a>Pouvez-vous désactiver des hubs entièrement maillés dans un réseau Virtual WAN ?

Il existe deux types de réseaux Virtual WAN : De base et Standard. Dans le réseau Virtual WAN de base, les hubs ne sont pas maillés. Dans un réseau Virtual WAN standard, les hubs sont maillés et automatiquement connectés lorsque le WAN virtuel est configuré pour la première fois. L’utilisateur n’a besoin d’effectuer aucune action spécifique. L’utilisateur n’a pas non plus à désactiver ou activer la fonctionnalité pour obtenir des hubs maillés. Le réseau Virtual WAN vous offre de nombreuses options de routage pour diriger le trafic entre n’importe quels spokes (VNet, VPN ou ExpressRoute). Il assure la facilité d’utilisation des hubs entièrement maillés, ainsi que la flexibilité du routage du trafic selon vos besoins. 

### <a name="how-are-availability-zones-and-resiliency-handled-in-virtual-wan"></a>Comment les zones de disponibilité et la résilience sont-elles gérées dans Virtual WAN ?

Virtual WAN est une collection de hubs et de services mis à votre disposition dans le hub. L’utilisateur peut avoir autant de WAN virtuels qu’il en a besoin. Dans un hub Virtual WAN, il existe plusieurs services tels que VPN, ExpressRoute, etc. Chacun de ces services (hormis le Pare-feu Azure) est déployé dans une région de zones de disponibilité, si cette région prend en charge les zones de disponibilité. Si une région devient une zone de disponibilité après le déploiement initial dans le hub, l’utilisateur peut recréer les passerelles, ce qui déclenche le déploiement d’une zone de disponibilité. Toutes les passerelles sont provisionnées dans un hub sous la forme active-active, ce qui implique une résilience intégrée au sein d’un hub. Les utilisateurs peuvent se connecter à plusieurs hubs s’ils souhaitent une résilience entre les régions. Alors que le concept de Virtual WAN est global, la ressource Virtual WAN réelle est basée sur Resource Manager et déployée de manière régionale. Si la région du WAN virtuel présente elle-même un problème, tous les hubs de ce WAN virtuel continueront à fonctionner en l’état, mais l’utilisateur ne pourra pas créer de nouveaux hubs tant que la région du WAN virtuel ne sera pas disponible.

### <a name="what-client-does-the-azure-virtual-wan-user-vpn-point-to-site-support"></a>Quel client le VPN utilisateur Virtual WAN Azure (point à site) prend en charge ?

Virtual WAN prend en charge le [client Azure VPN](https://go.microsoft.com/fwlink/?linkid=2117554), le client OpenVPN ou tout client IKEv2. L’authentification Azure AD est prise en charge avec Azure VPN Client. Le système d’exploitation client doit être au minimum Windows 10 version 17763.0.  Un ou plusieurs clients OpenVPN peuvent prendre en charge l’authentification basée sur les certificats. Une fois l’authentification basée sur les certificats sélectionnée dans la passerelle, vous voyez le fichier *.ovpn* à télécharger sur votre appareil. IKEv2 prend en charge l’authentification par certificat et RADIUS. 

### <a name="for-user-vpn-point-to-site--why-is-the-p2s-client-pool-split-into-two-routes"></a>Pour un VPN utilisateur (point à site), pourquoi le pool de clients P2S est divisé en deux routes ?

Chaque passerelle ayant deux instances, la division se produit de telle sorte que chaque instance de passerelle puisse allouer indépendamment les adresses IP des clients pour les clients connectés et que le trafic provenant du réseau virtuel soit redirigé vers l’instance de passerelle appropriée afin d’éviter un saut d’instance inter-passerelle.

### <a name="how-do-i-add-dns-servers-for-p2s-clients"></a>Comment ajouter des serveurs DNS pour les clients P2S ?

Il existe deux options pour ajouter des serveurs DNS pour les clients P2S. La première méthode est préférable car elle ajoute les serveurs DNS personnalisés à la passerelle à la place du client.

1. Utilisez le script PowerShell suivant pour ajouter les serveurs DNS personnalisés. Remplacez les valeurs de votre environnement.

   ```powershell
   // Define variables
   $rgName = "testRG1"
   $virtualHubName = "virtualHub1"
   $P2SvpnGatewayName = "testP2SVpnGateway1"
   $vpnClientAddressSpaces = 
   $vpnServerConfiguration1Name = "vpnServerConfig1"
   $vpnClientAddressSpaces = New-Object string[] 2
   $vpnClientAddressSpaces[0] = "192.168.2.0/24"
   $vpnClientAddressSpaces[1] = "192.168.3.0/24"
   $customDnsServers = New-Object string[] 2
   $customDnsServers[0] = "7.7.7.7"
   $customDnsServers[1] = "8.8.8.8"
   $virtualHub = $virtualHub = Get-AzVirtualHub -ResourceGroupName $rgName -Name $virtualHubName
   $vpnServerConfig1 = Get-AzVpnServerConfiguration -ResourceGroupName $rgName -Name $vpnServerConfiguration1Name

   // Specify custom dns servers for P2SVpnGateway VirtualHub while creating gateway
   createdP2SVpnGateway = New-AzP2sVpnGateway -ResourceGroupName $rgname -Name $P2SvpnGatewayName -VirtualHub $virtualHub -VpnGatewayScaleUnit 1 -VpnClientAddressPool $vpnClientAddressSpaces -VpnServerConfiguration $vpnServerConfig1 -CustomDnsServer $customDnsServers

   // Specify custom dns servers for P2SVpnGateway VirtualHub while updating existing gateway
   $P2SVpnGateway = Get-AzP2sVpnGateway -ResourceGroupName $rgName -Name $P2SvpnGatewayName
   $updatedP2SVpnGateway = Update-AzP2sVpnGateway -ResourceGroupName $rgName -Name $P2SvpnGatewayName  -CustomDnsServer $customDnsServers 

   // Re-generate Vpn profile either from PS/Portal for Vpn clients to have the specified dns servers
   ```
2. Ou, si vous utilisez Azure VPN Client pour Windows 10, vous pouvez modifier le fichier XML du profil téléchargé et ajouter les balises **\<dnsservers>\<dnsserver> \</dnsserver>\</dnsservers>** avant de l’importer.

   ```powershell
      <azvpnprofile>
      <clientconfig>

          <dnsservers>
              <dnsserver>x.x.x.x</dnsserver>
              <dnsserver>y.y.y.y</dnsserver>
          </dnsservers>
    
      </clientconfig>
      </azvpnprofile>
   ```

### <a name="for-user-vpn-point-to-site--how-many-clients-are-supported"></a>Pour un VPN utilisateur (point à site), combien de clients sont pris en charge ?

Chaque passerelle P2S VPN utilisateur compte deux instances, chacune d’elles prenant en charge un certain nombre d’utilisateurs à mesure que l’unité d’échelle change. L’unité d’échelle 1-3 prend en charge 500 connexions, l’unité d’échelle 4-6 1 000 connexions, l’unité d’échelle 7-12 5 000 connexions et l’unité d’échelle 13-18 jusqu’à 10 000 connexions. 

Supposons que l’utilisateur choisisse l’unité d’échelle 1. Chaque unité d’échelle implique le déploiement d’une passerelle active-active et chacune des instances (dans ce cas, 2) prend en charge jusqu’à 500 connexions. Comme vous pouvez obtenir 500 connexions * 2 par passerelle, cela ne signifie pas que vous planifiez 1 000 au lieu de 500 pour cette unité d’échelle. Il peut être nécessaire de traiter les instances. Au cours de cette opération, la connectivité pour les 500 supplémentaires peut être interrompue si vous avez dépassé le nombre de connexions recommandé. En outre, veillez à planifier les temps d’arrêt au cas où vous décideriez d’effectuer un scale-up ou un scale-down dans l’unité d’échelle, ou de modifier la configuration de point à site sur la passerelle VPN.

### <a name="what-is-the-difference-between-an-azure-virtual-network-gateway-vpn-gateway-and-an-azure-virtual-wan-vpn-gateway"></a>Quelle est la différence entre une passerelle de réseau virtuel Azure (passerelle VPN) et une passerelle VPN Azure Virtual WAN ?

WAN virtuel fournit une connectivité de site à site à grande échelle et est conçu pour le débit, l’évolutivité et la facilité d’utilisation. Lorsque vous connectez un site à une passerelle VPN Virtual WAN, il est différent d’une passerelle de réseau virtuel normale qui utilise le type de passerelle « VPN ». De même, lorsque vous connectez un circuit ExpressRoute à un hub Virtual WAN, il utilise une ressource différente pour la passerelle ExpressRoute que la passerelle de réseau virtuel normale utilisant le type de passerelle « ExpressRoute ». 

Virtual WAN prend en charge un débit agrégé pouvant atteindre 20 Gbits/s pour VPN et ExpressRoute. Virtual WAN dispose également d’une automatisation pour la connectivité avec un écosystème de partenaires d’appareils de branche CPE. Les appareils de branche CPE intègrent une automatisation qui provisionne et se connecte automatiquement à Azure Virtual WAN. Ces périphériques sont disponibles à partir d’un écosystème croissant des partenaires SD-WAN et VPN. Consultez la [liste des partenaires préférés](../articles/virtual-wan/virtual-wan-locations-partners.md).

### <a name="how-is-virtual-wan-different-from-an-azure-virtual-network-gateway"></a>En quoi Virtual WAN est-il différent d’une passerelle de réseau virtuel Azure ?

Le VPN d’une passerelle de réseau virtuel est limité à 30 tunnels. Pour les connexions, vous devez utiliser un WAN virtuel pour le VPN à grande échelle. Vous pouvez connecter jusqu’à 1 000 connexions de branche par région (hub virtuel) avec un agrégat de 20 Gbits/s par hub. Une connexion est un tunnel actif-actif entre l’appareil VPN local et le hub virtuel. Vous pouvez disposer d’un hub par région, ce qui signifie que vous pouvez connecter plus de 1000 branches parmi tous les hubs.

### <a name="what-is-a-virtual-wan-gateway-scale-unit"></a>Qu’est-ce qu’une unité d’échelle de passerelle Virtual WAN ?

Une unité d’échelle est une unité définie pour choisir un débit agrégé d’une passerelle dans le hub virtuel. 1 unité d’échelle de VPN = 500 Mbit/s 1 unité d’échelle d’ExpressRoute = 2 Gbits/s Exemple : 10 unités d’échelle de VPN représente un débit de 500 Mbits/s * 10 = 5 Gbits/s

### <a name="which-device-providers-virtual-wan-partners-are-supported"></a>Quels sont les fournisseurs d’appareils (partenaires Virtual WAN) pris en charge ?

À ce stade, de nombreux partenaires prennent en charge l’expérience Virtual WAN entièrement automatisée. Pour plus d’informations, consultez [Partenaires WAN virtuel](../articles/virtual-wan/virtual-wan-locations-partners.md).

### <a name="what-are-the-virtual-wan-partner-automation-steps"></a>Quelles sont les étapes d’automatisation des partenaires Virtual WAN ?

Pour découvrir les étapes d’automatisation des partenaires, consultez l’article [Partenaires Virtual WAN](../articles/virtual-wan/virtual-wan-configure-automation-providers.md).

### <a name="am-i-required-to-use-a-preferred-partner-device"></a>Est-il nécessaire d’utiliser un périphérique de partenaire préféré ?

Non. Vous pouvez utiliser n’importe quel appareil prenant en charge les VPN et qui respecte les exigences Azure pour la prise en charge de IPsec IKEv2/IKEv1. Virtual WAN possède également des solutions de partenaires CPE qui automatisent la connectivité à Azure Virtual WAN, ce qui facilite la configuration des connexions VPN IPsec à grande échelle.

### <a name="how-do-virtual-wan-partners-automate-connectivity-with-azure-virtual-wan"></a>Comment les partenaires WAN virtuel automatisent-ils la connectivité avec le WAN virtuel Azure ?

Les solutions de connectivité à définition logicielle gèrent généralement leurs appareils de branche à l’aide d’un contrôleur ou un centre de provisionnement des appareils. Le contrôleur peut utiliser des API Azure pour automatiser la connectivité au WAN virtuel Azure. L’automatisation comprend le chargement des informations de branche, le téléchargement de la configuration Azure, la configuration de tunnels IPSec dans des hubs virtuels Azure et la configuration automatique de la connectivité à l’appareil de branche à Azure Virtual WAN. Lorsque vous avez des centaines de branches, la connexion à l’aide de partenaires CPE Virtual WAN est simple, car l’expérience d’intégration n’a plus besoin de configurer et de gérer une connectivité IPsec à grande échelle. Pour plus d’informations, consultez [Virtual WAN partner automation](../articles/virtual-wan/virtual-wan-configure-automation-providers.md) (Automation de partenaire WAN virtuel).

### <a name="what-if-a-device-i-am-using-is-not-in-the-virtual-wan-partner-list-can-i-still-use-it-to-connect-to-azure-virtual-wan-vpn"></a>Que se passe-t-il si j’utilise un appareil qui n’est pas dans la liste des partenaires Virtual WAN ? Puis-je toujours l’utiliser pour me connecter au VPN Azure Virtual WAN ?

Oui, si l’appareil prend en charge IPsec IKEv1 ou IKEv2. Les partenaires Virtual WAN automatisent la connectivité de l’appareil aux points de terminaison du VPN Azure. Cela implique l’automatisation d’étapes telles que « chargement des informations de branche », « IPsec et configuration » et « connectivité ». Comme votre appareil ne vient pas d’un écosystème de partenaires Virtual WAN, vous devez vous-même récupérer la configuration d’Azure et mettre à jour votre appareil pour configurer la connectivité IPsec.

### <a name="how-do-new-partners-that-are-not-listed-in-your-launch-partner-list-get-onboarded"></a>Comment les nouveaux partenaires qui ne figurent pas dans votre liste de partenaires de lancement sont-ils intégrés ?

Toutes les API de réseau étendu virtuel sont des API ouvertes. Vous pouvez consulter la documentation [Automatisation des partenaires Virtual WAN](../articles/virtual-wan/virtual-wan-configure-automation-providers.md) pour évaluer la faisabilité technique. Un partenaire idéal dispose d’un appareil qui peut être approvisionné pour une connectivité IPsec IKEv1 ou IKEv2. Une fois que l’entreprise a terminé le travail d’automatisation pour son appareil CPE en fonction des instructions d’automatisation fournies ci-dessus, vous pouvez accéder à azurevirtualwan@microsoft.com pour être listé ici [Connectivité via les partenaires]( ../articles/virtual-wan/virtual-wan-locations-partners.md#partners). Si, comme client, vous souhaitez qu’une certaine solution d’entreprise soit listée comme partenaire Virtual WAN, demandez à l’entreprise de contacter le réseau Virtual WAN en envoyant un e-mail à azurevirtualwan@microsoft.com.

### <a name="how-is-virtual-wan-supporting-sd-wan-devices"></a>Comment Virtual WAN prend en charge les appareils SD-WAN ?

Les partenaires Virtual WAN automatisent la connectivité IPsec vers les points de terminaison du VPN Azure. Si le partenaire Virtual WAN est un fournisseur de SD-WAN, il est nécessaire que le contrôleur SD-WAN gère l’automatisation et la connectivité IPsec vers les points de terminaison de VPN Azure. Si l’appareil SD-WAN nécessite son propre point de terminaison au lieu du VPN Azure pour toutes les fonctionnalités SD-WAN propriétaires, vous pouvez déployer le point de terminaison SD-WAN dans un réseau virtuel Azure et le faire coexister avec Azure Virtual WAN.

### <a name="how-many-vpn-devices-can-connect-to-a-single-hub"></a>Combien de périphériques VPN peuvent se connecter à un même hub ?

Chaque hub virtuel prend en charge jusqu’à 1000 connexions. Chaque connexion se compose de quatre liens et chaque connexion de lien prend en charge deux tunnels qui sont dans une configuration actif-actif. Les tunnels se terminent par une passerelle VPN de hub virtuel Azure. Les liens représentent le lien du fournisseur de services Internet physique au niveau de l’appareil de branche/VPN.

### <a name="what-is-a-branch-connection-to-azure-virtual-wan"></a>Qu’est-ce qu’une connexion de branche à Azure Virtual WAN ?

Une connexion à partir d’un appareil de branche ou VPN dans Azure Virtual WAN n’est rien de plus qu’une connexion VPN qui connecte virtuellement le site VPN et la passerelle Azure VPN dans un hub virtuel.

### <a name="can-the-on-premises-vpn-device-connect-to-multiple-hubs"></a>L’appareil VPN local peut-il se connecter à plusieurs hubs ?

Oui. Au départ, le flux du trafic s’effectue de l’appareil local vers le périmètre réseau de Microsoft le plus proche, puis vers le hub virtuel.

### <a name="are-there-new-resource-manager-resources-available-for-virtual-wan"></a>Existe-t-il des nouvelles ressources de Resource Manager disponibles pour le WAN virtuel ?
  
Oui, Virtual WAN possède de nouvelles ressources Resource Manager. Pour plus d’informations, consultez la [Vue d’ensemble](../articles/virtual-wan/virtual-wan-about.md).

### <a name="can-i-deploy-and-use-my-favorite-network-virtual-appliance-in-an-nva-vnet-with-azure-virtual-wan"></a>Puis-je déployer et utiliser l’appliance virtuelle de mon réseau favori (dans un réseau virtuel NVA) avec le WAN virtuel Azure ?

Oui, vous pouvez connecter le réseau virtuel VNet de l’appliance virtuelle (NVA) au WAN virtuel Azure.

### <a name="can-i-create-a-network-virtual-appliance-inside-the-virtual-hub"></a>Puis-je créer une appliance virtuelle réseau à l’intérieur du hub virtuel ?

Une appliance virtuelle réseau (NVA, pour Network Virtual Appliance) ne peut pas être déployée à l’intérieur d’un hub virtuel. Toutefois, vous pouvez la créer dans un réseau virtuel spoke connecté au hub virtuel et activer le routage approprié pour diriger le trafic en fonction de vos besoins.

### <a name="can-a-spoke-vnet-have-a-virtual-network-gateway"></a>Un réseau virtuel spoke peut-il avoir une passerelle de réseau virtuel ?

Non. Le réseau virtuel spoke ne peut pas avoir de passerelle de réseau virtuel s’il est connecté au hub virtuel.

### <a name="is-there-support-for-bgp-in-vpn-connectivity"></a>La prise en charge de BGP est-elle assurée dans la connectivité VPN ?

Oui, BGP est pris en charge. Quand vous créez un site VPN, vous pouvez fournir les paramètres BGP qu’il contient. Cela implique que toutes les connexions créées dans Azure pour ce site sont activées pour BGP.

### <a name="is-there-any-licensing-or-pricing-information-for-virtual-wan"></a>Y a-t-il des informations de licence ou de tarification pour le WAN virtuel ?

Oui. Consultez la page [Tarification](https://azure.microsoft.com/pricing/details/virtual-wan/).

### <a name="is-it-possible-to-construct-azure-virtual-wan-with-a-resource-manager-template"></a>Est-il possible de construire le WAN virtuel Azure avec un modèle Resource Manager ?

La configuration simple d’un Virtual WAN avec un hub et un site VPN peut être créée en utilisant un [modèle de démarrage rapide](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network). Virtual WAN est principalement un service piloté par REST ou le portail.

### <a name="can-spoke-vnets-connected-to-a-virtual-hub-communicate-with-each-other-v2v-transit"></a>Est-ce que les réseaux virtuels spoke qui sont connectés à un hub virtuel peuvent communiquer entre eux (transit V2V) ?

Oui. Le réseau Virtual WAN standard prend en charge la connectivité transitive de réseau virtuel à réseau virtuel via le hub Virtual WAN auquel les réseaux virtuels sont connectés. Dans la terminologie des réseaux étendus virtuels, nous faisons référence à ces chemins en tant que « transit de réseau virtuel Virtual WAN local » pour les réseaux virtuels connectés à un hub Virtual WAN dans une seule région, et en tant que « transit de réseau virtuel Virtual WAN global » pour les réseaux virtuels connectés via plusieurs hubs Virtual WAN dans deux régions ou plus. Dans certains scénarios, les réseaux virtuels spoke peuvent également être directement appairés les uns aux autres via l’[appairage de réseaux virtuels](../articles/virtual-network/virtual-network-peering-overview.md), en plus du transit de réseau virtuel Virtual WAN local ou global. Dans ce cas, l’appairage de réseaux virtuels est prioritaire sur la connexion transitive via le hub Virtual WAN.

### <a name="is-branch-to-branch-connectivity-allowed-in-virtual-wan"></a>La connectivité de branche à branche est-elle autorisée dans le WAN virtuel ?

Oui, la connectivité de branche à branche est autorisée dans le WAN virtuel. Une branche est conceptuellement applicable au site VPN, aux circuits ExpressRoute ou aux utilisateurs de VPN point à site/utilisateur. La connectivité de branche à branche est activée par défaut et peut être localisée dans les paramètres de configuration WAN. Cela permet aux branches/utilisateurs VPN de se connecter à d’autres branches VPN, et la connectivité de transit est activée entre les utilisateurs VPN et ExpressRoute.

### <a name="does-branch-to-branch-traffic-traverse-through-the-azure-virtual-wan"></a>Le trafic branche à branche traverse-t-il Azure Virtual WAN ?

Oui.

### <a name="does-virtual-wan-require-expressroute-from-each-site"></a>Virtual WAN nécessite-t-il ExpressRoute à partir de chaque site ?

Non. Virtual WAN ne nécessite pas ExpressRoute à partir de chaque site. Vos sites peuvent être connectés à un réseau de fournisseur à l’aide d’un circuit ExpressRoute. Pour les sites qui sont connectés via ExpressRoute à un hub virtuel ainsi que via un VPN IPsec au même hub, le hub virtuel fournit une connectivité de transit entre l’utilisateur VPN et ExpressRoute.

### <a name="is-there-a-network-throughput-or-connection-limit-when-using-azure-virtual-wan"></a>Existe-t-il une limite de débit réseau ou de connexions lors de l’utilisation d’Azure Virtual WAN ?

Le débit réseau s’exprime par service dans un hub Virtual WAN. Vous pouvez avoir autant de réseaux WAN virtuels que vous le souhaitez, mais chaque WAN virtuel autorise 1 hub par région. Dans chaque hub, le débit agrégé VPN peut atteindre jusqu’à 20 Gbit/s, le débit agrégé ExpressRoute jusqu’à 20 Gbit/s et le débit agrégé VPN utilisateur/VPN de point à site jusqu’à 20 Gbit/s. Le routeur dans le hub virtuel prend en charge jusqu’à 50 Gbit/s pour les flux de trafic de réseau virtuel à réseau virtuel, et suppose une charge de travail totale de 2000 machines virtuelles parmi tous les réseaux virtuels connectés à un même hub virtuel.

Lorsque des sites VPN se connectent à un hub, ils le font à l’aide de connexions. Virtual WAN prend en charge jusqu’à 1 000 connexions ou 2 000 tunnels IPsec par hub virtuel. Lorsque les utilisateurs distants se connectent au hub virtuel, ils se connectent à la passerelle VPN P2S, qui prend en charge jusqu’à 10 000 utilisateurs en fonction de l’unité d’échelle (bande passante) choisie pour la passerelle VPN P2S dans le hub virtuel.

### <a name="what-is-the-total-vpn-throughput-of-a-vpn-tunnel-and-a-connection"></a>Quel est le débit VPN total d’un tunnel VPN et d’une connexion ?

Le débit VPN total maximal d’un hub est de 20 Gbit/s, en fonction de l’unité d’échelle choisie pour la passerelle VPN. Le débit est partagé par toutes les connexions existantes. Chaque tunnel dans une connexion peut prendre en charge jusqu’à 1 Gbit/s.

### <a name="i-dont-see-the-20-gbps-setting-for-the-virtual-hub-in-portal-how-do-i-configure-that"></a>Je ne vois pas le paramètre 20 Gbit/s pour le hub virtuel dans le portail. Comment le configurer ?

Accédez à la passerelle VPN à l’intérieur d’un hub sur le portail, puis cliquez sur l’unité d’échelle pour la remplacer par le paramètre approprié.

### <a name="does-virtual-wan-allow-the-on-premises-device-to-utilize-multiple-isps-in-parallel-or-is-it-always-a-single-vpn-tunnel"></a>Virtual WAN autorise-t-il l’appareil en local à utiliser plusieurs ISP en parallèle, ou faut-il toujours un tunnel VPN unique ?

Les solutions d’appareils locaux peuvent appliquer des stratégies de trafic pour diriger le trafic entre plusieurs tunnels dans le hub Azure Virtual WAN (passerelle VPN dans le hub virtuel).

### <a name="what-is-global-transit-architecture"></a>Qu’est-ce que l’architecture de transit global ?

Pour plus d’informations sur l’architecture de transit global, consultez [Architecture du réseau de transit global et Virtual WAN](../articles/virtual-wan/virtual-wan-global-transit-network-architecture.md).

### <a name="how-is-traffic-routed-on-the-azure-backbone"></a>Comment le trafic est-il acheminé sur la dorsale principale d’Azure ?

Le trafic suit le modèle suivant : appareil de branche -> ISP -> périmètre réseau de Microsoft - > Microsoft DC (réseau virtuel du hub) -> périmètre réseau de Microsoft -> ISP -> appareil de branche

### <a name="in-this-model-what-do-you-need-at-each-site-just-an-internet-connection"></a>Dans ce modèle, de quoi avez-vous besoin sur chaque site ? Seulement d’une connexion Internet ?

Oui. Une connexion Internet et un appareil physique qui prend en charge IPsec, de préférence parmi nos [partenaires Virtual WAN](../articles/virtual-wan/virtual-wan-locations-partners.md) intégrés. Si vous le souhaitez, vous pouvez gérer manuellement la configuration et la connectivité à Azure à partir de l’appareil de votre choix.

### <a name="how-do-i-enable-default-route-00000-in-a-connection-vpn-expressroute-or-virtual-network"></a>Comment activer la route par défaut (0.0.0.0/0) dans une connexion (VPN, ExpressRoute ou Réseau virtuel) ?

Un hub virtuel peut propager un itinéraire par défaut appris à une connexion de réseau virtuel/VPN site à site/ExpressRoute si l’indicateur est « Activé » sur la connexion. Cet indicateur est visible lorsque l’utilisateur modifie une connexion de réseau virtuel, une connexion VPN ou une connexion ExpressRoute. Par défaut, cet indicateur est désactivé lorsqu’un site ou un circuit ExpressRoute est connecté à un hub. Il est activé par défaut lorsqu’une connexion de réseau virtuel est ajoutée pour connecter un réseau virtuel à un hub virtuel. L’itinéraire par défaut ne provient pas du hub Virtual WAN ; il est propagé s’il est déjà appris par le hub Virtual WAN suite au déploiement d’un pare-feu dans le hub, ou si le tunneling forcé est activé sur un autre site connecté.

### <a name="how-does-the-virtual-hub-in-a-virtual-wan-select-the-best-path-for-a-route-from-multiple-hubs"></a>Comment le hub virtuel d’un réseau étendu virtuel sélectionne le meilleur chemin pour une route à partir de plusieurs hubs ?

Si un hub virtuel apprend la même route à partir de plusieurs hubs distants, l’ordre dans lequel il prend sa décision est le suivant :

1. Correspondance de préfixe la plus longue.
2. Itinéraires locaux sur interHub (le hub virtuel affecte 65520-65520 pour AS interHub)
3. Routes statiques par rapport à BGP : Cela figure dans le contexte de la décision prise par le routeur de hub virtuel. Toutefois, si le décideur est la passerelle VPN dans laquelle un site publie des routes via le protocole BGP ou fournit des préfixes d’adresses statiques, les routes statiques peuvent être préférées aux routes BGP.
4. ExpressRoute (ER) par rapport à VPN : ER est préféré à VPN lorsque le contexte est un hub local. La connectivité de transit entre les circuits ExpressRoute est disponible uniquement par le biais de Global Reach. Par conséquent, dans les scénarios où le circuit ExpressRoute est connecté à un hub et qu’un autre circuit ExpressRoute est connecté à un hub différent avec une connexion VPN, le VPN peut être préféré pour les scénarios entre hubs.
5. Longueur des chemins entre les systèmes autonomes.

### <a name="does-virtual-wan-hub-allow-connectivity-between-expressroute-circuits"></a>Le hub Virtual WAN autorise-t-il la connectivité entre les circuits ExpressRoute.

Le transit entre ER et ER s’effectue toujours via Global Reach. Les passerelles de hub virtuel sont déployées dans les régions DC ou Azure. Lorsque deux circuits ExpressRoute se connectent via Global Reach, il n’est pas nécessaire que le trafic fasse tout le chemin à partir des routeurs de périphérie vers le contrôleur de domaine du hub virtuel.

### <a name="is-there-a-concept-of-weight-in-azure-virtual-wan-expressroute-circuits-or-vpn-connections"></a>Existe-il un concept de poids dans les circuits ExpressRoute Azure Virtual WAN ou les connexions VPN ?

Lorsque plusieurs circuits ExpressRoute sont connectés à un hub virtuel, le poids du routage sur la connexion fournit un mécanisme permettant à ExpressRoute dans le hub virtuel de préférer un circuit à l’autre. Il n’existe aucun mécanisme permettant de définir un poids sur une connexion VPN. Azure préfère toujours une connexion ExpressRoute à une connexion VPN au sein d’un hub individuel.

### <a name="does-virtual-wan-prefer-expressroute-over-vpn-for-traffic-egressing-azure"></a>Un réseau étendu virtuel préfère-t-il ExpressRoute à un VPN pour le trafic sortant d’Azure ?

Oui.

### <a name="when-a-virtual-wan-hub-has-an-expressroute-circuit-and-a-vpn-site-connected-to-it-what-would-cause-a-vpn-connection-route-to-be-preferred-over-expressroute"></a>Lorsqu’un hub Virtual WAN dispose d’un circuit ExpressRoute et d’un site VPN qui lui est connecté, qu’est-ce qui ferait qu’une route de connexion VPN serait préférable à ExpressRoute ?

Quand un circuit ExpressRoute est connecté à un hub virtuel, les routeurs de périphérie Microsoft constituent le premier nœud pour la communication entre l’environnement local et Azure. Ces routeurs de périphérie communiquent avec les passerelles ExpressRoute du WAN virtuel qui, à leur tour, apprennent les routes auprès du routeur de hub virtuel qui contrôle toutes les routes entre les diverses passerelles dans Virtual WAN. Les routeurs de périphérie Microsoft traitent les routes ExpressRoute de hub virtuel avec une préférence plus élevée que les routes apprises à partir de l’environnement local. Quelle qu’en soit la raison, si la connexion VPN devient le support principal à partir duquel le hub virtuel apprend les routes (par exemple dans les scénarios de basculement entre ExpressRoute et VPN), à moins que le site VPN n’ait un chemin AS plus long, le hub virtuel continuera à partager les routes apprises via le VPN avec la passerelle ExpressRoute, ce qui fait que les routeurs de périphérie Microsoft préfèreront les routes VPN aux routes locales.

### <a name="when-two-hubs-hub-1-and-2-are-connected-and-there-is-an-expressroute-circuit-connected-as-a-bow-tie-to-both-the-hubs-what-is-the-path-for-a-vnet-connected-to-hub-1-to-reach-a-vnet-connected-in-hub-2"></a>Lorsque deux hubs (hub 1 et 2) sont connectés et qu’un circuit ExpressRoute est connecté comme un nœud papillon aux deux hubs, quel est le chemin d’un réseau virtuel connecté au hub 1 lui permettant d’atteindre un réseau virtuel connecté dans le hub 2 ?

Le comportement actuel consiste à préférer le chemin du circuit ExpressRoute à la connectivité de hub à hub pour la connectivité de réseau virtuel à réseau virtuel. Toutefois, cela n’est pas recommandé dans une configuration de réseau WAN virtuel. L’équipe Virtual WAN travaille sur un correctif pour activer la préférence pour la connectivité de hub à hub par rapport au chemin ExpressRoute. La recommandation est de faire en sorte que plusieurs circuits ExpressRoute (différents fournisseurs) se connectent à un seul hub et utilisent la connectivité de hub à hub fournie par Virtual WAN pour les flux de trafic inter-régionaux.

### <a name="can-hubs-be-created-in-different-resource-group-in-virtual-wan"></a>Des hubs peuvent-ils être créés dans différents groupes de ressources dans Virtual WAN ?
Oui. Cette option est actuellement disponible avec PowerShell uniquement. Le portail Virtual WAN impose que les hubs soient dans le même groupe de ressources que la ressource Virtual WAN proprement dite.

### <a name="is-there-support-for-ipv6-in-virtual-wan"></a>IPv6 est-il pris en charge dans Virtual WAN ?

IPv6 n’est pas pris en charge dans le hub Virtual WAN et ses passerelles. Si vous disposez d’un réseau virtuel qui prend en charge IPv4 et IPv6, et que vous voulez le connecter à Virtual WAN, ce scénario n’est pas pris en charge actuellement. 

Pour le scénario VPN point à site (utilisateur) avec « breakout » Internet via le Pare-feu Azure, vous devrez probablement désactiver la connectivité IPv6 sur votre appareil client pour forcer le trafic vers le hub Virtual WAN. Cela est dû au fait que les appareils modernes utilisent par défaut des adresses IPv6 par défaut.

### <a name="what-is-the-recommended-api-version-to-be-used-by-scripts-automating-various-virtual-wan-functionalities"></a>Quelle est la version d’API recommandée pour une utilisation par les scripts qui automatisent différentes fonctionnalités de Virtual WAN ?

Cela nécessite au minimum la version 05-01-2020 (1er mai 2020). 

### <a name="are-there-any-virtual-wan-limits"></a>Existe-t-il des limites pour Virtual WAN ?

Consultez la section [Limites applicables à Virtual WAN](../articles/azure-resource-manager/management/azure-subscription-service-limits.md#virtual-wan-limits) dans la page sur les limites d’abonnement et de service.

### <a name="what-are-the-differences-between-the-virtual-wan-types-basic-and-standard"></a>Quelles sont les différences entre les types de Virtual WAN (de base et standard) ?

Consultez [Réseaux Virtual WAN de base et standard](../articles/virtual-wan/virtual-wan-about.md#basicstandard). Pour connaître les tarifs, consultez la page [Tarification](https://azure.microsoft.com/pricing/details/virtual-wan/).

### <a name="does-virtual-wan-store-customer-data"></a>Est-ce que Virtual WAN stocke des données client ? 

Non. Virtual WAN ne stocke pas de données client.
