---
title: FAQ Pare-feu Azure
description: FAQ sur Pare-feu Azure. Service de sécurité réseau cloud managé qui protège vos ressources Réseau virtuel Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 08/13/2020
ms.author: victorh
ms.openlocfilehash: 3e6ea6692a81a06bbf3180904dfb465a88b105d1
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94653417"
---
# <a name="azure-firewall-faq"></a>FAQ Pare-feu Azure

## <a name="what-is-azure-firewall"></a>Qu’est-ce qu’un pare-feu Azure ?

Pare-feu Azure est un service de sécurité réseau informatique géré qui protège vos ressources Réseau virtuel Azure. Il s’agit d’un service de pare-feu entièrement avec état, pourvu d’une haute disponibilité intégrée et de l’extensibilité sans limites du cloud. Vous pouvez créer, appliquer et consigner des stratégies de connectivité réseau et d’application de façon centralisée entre les abonnements et les réseaux virtuels.

## <a name="what-capabilities-are-supported-in-azure-firewall"></a>Quelles fonctionnalités sont prises en charge dans Pare-feu Azure ?

Pour découvrir les fonctionnalités du Pare-feu Azure, consultez [Fonctionnalités du Pare-feu Azure](features.md).

## <a name="what-is-the-typical-deployment-model-for-azure-firewall"></a>Qu’est le modèle de déploiement type de Pare-feu Azure ?

Vous pouvez déployer Pare-feu Azure sur n’importe quel réseau virtuel, mais en général, les clients le déploient sur un réseau virtuel central et appairent les autres réseaux virtuels à celui-ci dans un modèle hub-and-spoke. Vous pouvez ensuite définir l’itinéraire par défaut des réseaux virtuels appairés pour qu’il pointe vers ce réseau virtuel de pare-feu central. Global VNET Peering est pris en charge, mais n’est pas recommandé en raison de problèmes potentiels de performances et de latence sur les régions. Pour de meilleures performances, déployez un pare-feu par région.

L'avantage de ce modèle est qu'il permet d'exercer un contrôle centralisé sur plusieurs réseaux virtuels spoke pour différents abonnements. Il permet également de réaliser des économies de coûts puisqu'il n'est pas nécessaire de déployer un pare-feu au sein de chaque réseau virtuel. Les économies doivent être mesurées par rapport aux coûts de peering associés en fonction des modèles de trafic du client.

## <a name="how-can-i-install-the-azure-firewall"></a>Comment installer Pare-feu Azure ?

Vous pouvez installer Pare-feu Azure via le Portail Microsoft Azure, PowerShell, les API REST ou à l’aide de modèles. Voir le [tutoriel : Déployer et configurer Pare-feu Azure avec le portail Azure](tutorial-firewall-deploy-portal.md) pour obtenir des instructions pas à pas.

## <a name="what-are-some-azure-firewall-concepts"></a>Quels sont les concepts de Pare-feu Azure ?

Pare-feu Azure prend en charge les règles et les regroupements de règles. Un regroupement de règles est un ensemble de règles qui partagent le même ordre et la même priorité. Les regroupements de règles sont exécutés dans l’ordre de leur priorité. Ainsi, les regroupements de règles Réseau sont prioritaires pas rapport aux regroupements de règles Application, et toutes les règles se terminent.

Il existe trois types de regroupement de règles :

* *Règles d’application* : Configurez des noms de domaine pleinement qualifiés (FQDN) qui sont accessibles depuis un sous-réseau.
* *Règles de réseau* : Configurez des règles contenant les adresses sources, les protocoles, les ports de destination et les adresses de destination.
* *Règles NAT* : Configurez des règles DNAT pour autoriser les connexions Internet entrantes.

## <a name="does-azure-firewall-support-inbound-traffic-filtering"></a>Pare-feu Azure prend-il en charge le filtrage du trafic entrant ?

Pare-feu Azure prend en charge le filtrage du trafic entrant et sortant. La protection du trafic entrant est généralement utilisée pour les protocoles non HTTP/S. Par exemple, les protocoles RDP, SSH et FTP. Pour une protection HTTP/S optimale du trafic entrant, utilisez un pare-feu d’applications web comme le [pare-feu d’applications web Azure (WAF)](../web-application-firewall/overview.md).

## <a name="which-logging-and-analytics-services-are-supported-by-the-azure-firewall"></a>Quels sont les services de journalisation et d’analyse pris en charge par Pare-feu d’Azure ?

Pare-feu Azure est intégré à Azure Monitor pour la consultation et l’analyse des journaux d’activité de Pare-feu. Les journaux d’activité peuvent être envoyés à Log Analytics, Stockage Azure ou Event Hubs. Ils peuvent être analysés dans Log Analytics ou par d’autres outils comme Excel et Power BI. Pour plus d’informations, consultez [Didacticiel : Surveiller les journaux d’activité de pare-feu Azure](./firewall-diagnostics.md).

## <a name="how-does-azure-firewall-work-differently-from-existing-services-such-as-nvas-in-the-marketplace"></a>En quoi Pare-feu Azure fonctionne-t-il différemment des services existants comme les appliances virtuelles réseau sur la Place de marché ?

Pare-feu Azure est un service de pare-feu de base qui peut résoudre les problèmes que rencontrent les clients dans certains cas. Il est probable que vous obteniez un mélange d’appliances virtuelles réseau et de Pare-feu Azure tiers. Mieux travailler ensemble est une priorité.

## <a name="what-is-the-difference-between-application-gateway-waf-and-azure-firewall"></a>Quelle est la différence entre Application Gateway WAF et Pare-feu Azure ?

Le pare-feu d’applications web (WAF, Web Application Firewall) est une fonctionnalité d’Application Gateway qui protège en entrée vos applications web de manière centralisée contre les vulnérabilités et les exploits courants. Pare-feu Azure assure une protection en entrée pour les protocoles non-HTTP/S (par exemple, RDP, SSH et FTP), une protection en sortie au niveau du réseau pour tous les ports et protocoles, ainsi qu’une protection au niveau de l’application pour le trafic HTTP/S sortant.

## <a name="what-is-the-difference-between-network-security-groups-nsgs-and-azure-firewall"></a>Quelle est la différence entre les groupes de sécurité réseau (NSG) et Pare-feu Azure ?

Le service Pare-feu Azure complète les fonctionnalités de groupe de sécurité réseau. Ensemble, ils fournissent une meilleure sécurité réseau grâce à une défense approfondie. Les groupes de sécurité réseau assurent un filtrage du trafic distribué au niveau de la couche réseau pour limiter le trafic aux ressources au sein de réseaux virtuels de chaque abonnement. Pare-feu Azure est un service de pare-feu entièrement centralisé et avec état qui offre une protection au niveau du réseau et de l’application entre les différents abonnements et réseaux virtuels.

## <a name="are-network-security-groups-nsgs-supported-on-the-azurefirewallsubnet"></a>Les groupes de sécurité réseau sont-ils pris en charge dans AzureFirewallSubnet ?

Pare-feu Azure est un service géré avec plusieurs couches de protection, y compris la protection de plateforme avec des groupes de sécurité réseau au niveau de la carte réseau (non visible).  Les groupes de sécurité réseau au niveau du sous-réseau ne sont pas nécessaires sur AzureFirewallSubnet, et sont désactivés pour empêcher toute interruption du service.

## <a name="how-do-i-set-up-azure-firewall-with-my-service-endpoints"></a>Comment faire pour configurer Pare-feu Azure avec mes points de terminaison de service ?

Pour bénéficier d’un accès sécurisé aux services PaaS, nous vous recommandons des points de terminaison de service. Vous pouvez choisir d’activer des points de terminaison de service dans le sous-réseau de Pare-feu Azure et les désactiver dans les réseaux virtuels spoke connectés. De cette façon, vous bénéficiez des deux fonctionnalités : sécurité de point de terminaison de service et journalisation centralisée pour tout le trafic.

## <a name="what-is-the-pricing-for-azure-firewall"></a>Quels sont les tarifs de Pare-feu Azure ?

Consultez [Tarification de Pare-feu Azure](https://azure.microsoft.com/pricing/details/azure-firewall/).

## <a name="how-can-i-stop-and-start-azure-firewall"></a>Comment arrêter et démarrer le Pare-feu Azure ?

Vous pouvez utiliser les méthodes *désallouer* et *allouer* d’Azure PowerShell.

Par exemple :

```azurepowershell
# Stop an existing firewall

$azfw = Get-AzFirewall -Name "FW Name" -ResourceGroupName "RG Name"
$azfw.Deallocate()
Set-AzFirewall -AzureFirewall $azfw
```

```azurepowershell
# Start a firewall

$azfw = Get-AzFirewall -Name "FW Name" -ResourceGroupName "RG Name"
$vnet = Get-AzVirtualNetwork -ResourceGroupName "RG Name" -Name "VNet Name"
$publicip1 = Get-AzPublicIpAddress -Name "Public IP1 Name" -ResourceGroupName "RG Name"
$publicip2 = Get-AzPublicIpAddress -Name "Public IP2 Name" -ResourceGroupName "RG Name"
$azfw.Allocate($vnet,@($publicip1,$publicip2))

Set-AzFirewall -AzureFirewall $azfw
```

> [!NOTE]
> Vous devez réaffecter un pare-feu et une adresse IP publique au groupe de ressources et à l’abonnement d’origine.

## <a name="what-are-the-known-service-limits"></a>Quelles sont les limites connues du service ?

Pour connaître les limites du service Pare-feu Azure, voir [Limites, quotas et contraintes applicables aux services et abonnements Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-firewall-limits).

## <a name="can-azure-firewall-in-a-hub-virtual-network-forward-and-filter-network-traffic-between-two-spoke-virtual-networks"></a>Dans un réseau virtuel de hub, Pare-feu Azure peut-il envoyer et filtrer le trafic réseau entre deux réseaux virtuels spoke ?

Oui, vous pouvez utiliser Pare-feu Azure dans un réseau virtuel de hub pour acheminer et filtrer le trafic entre deux réseaux virtuels spoke. Afin de fonctionner correctement, les sous-réseaux de chaque réseau virtuel spoke doivent disposer d’un UDR pointant vers Pare-feu Azure en tant que passerelle par défaut pour ce scénario.

## <a name="can-azure-firewall-forward-and-filter-network-traffic-between-subnets-in-the-same-virtual-network-or-peered-virtual-networks"></a>Pare-feu Azure peut-il envoyer et filtrer le trafic réseau entre des sous-réseaux d’un même réseau virtuel ou réseau virtuel appairé ?

Oui. Toutefois, la configuration des UDR pour rediriger le trafic entre les sous-réseaux d'un même réseau virtuel nécessite une attention supplémentaire. Bien que l'utilisation de la plage d'adresses du réseau virtuel comme préfixe cible pour l'UDR soit suffisante, celle-ci achemine également tout le trafic d'une machine vers une autre au sein du même sous-réseau via l'instance du Pare-feu Azure. Pour éviter cela, incluez l'itinéraire du sous-réseau dans l'UDR, avec **VNET** comme type de tronçon suivant. La gestion de ces itinéraires peut être lourde et sujette à erreur. La méthode recommandée pour la segmentation interne du réseau consiste à utiliser des groupes de sécurité réseau, qui ne requièrent pas d’UDR.

## <a name="does-azure-firewall-outbound-snat-between-private-networks"></a>La sortie de Pare-feu Azure traduit-elle l’adresse réseau source (SNAT) entre réseaux privés ?

Pare-feu Azure ne traduit pas l’adresse réseau source lorsque l’adresse IP de destination est une plage d’adresses IP privées selon la norme [IANA RFC 1918](https://tools.ietf.org/html/rfc1918). Si votre organisation utilise une plage d’adresses IP publiques pour les réseaux privés, Pare-feu Azure traduit l’adresse réseau source du trafic en une des adresses IP privées du pare-feu dans AzureFirewallSubnet. Vous pouvez configurer le pare-feu Azure pour ne **pas** traduire l’adresse réseau source (SNAT) de votre plage d’adresses IP publiques. Pour plus d’informations, consultez [Plages d’adresses IP privées SNAT de pare-feu Azure](snat-private-range.md).

## <a name="is-forced-tunnelingchaining-to-a-network-virtual-appliance-supported"></a>Le tunneling/chaînage forcé à une appliance virtuelle réseau est-il pris en charge ?

Le tunneling forcé est pris en charge lorsque vous créez un nouveau pare-feu. Vous ne pouvez pas configurer un pare-feu existant pour le tunneling forcé. Pour plus d’informations, consultez la page [Tunneling forcé du Pare-feu Azure](forced-tunneling.md).

Le Pare-feu Azure doit avoir une connectivité Internet directe. Si votre AzureFirewallSubnet prend connaissance d’un itinéraire par défaut pour votre réseau local via le protocole BGP, vous devez le remplacer par un UDR 0.0.0.0/0 avec la valeur **NextHopType** définie sur **Internet** pour garantir une connectivité Internet directe.

Si votre configuration nécessite un tunneling forcé vers un réseau local et que vous pouvez déterminer les préfixes IP cibles pour vos destinations Internet, vous pouvez configurer ces plages en faisant du réseau local le tronçon suivant avec une route définie par l’utilisateur sur le sous-réseau AzureFirewallSubnet. Vous pouvez aussi utiliser le protocole BGP pour définir ces routes.

## <a name="are-there-any-firewall-resource-group-restrictions"></a>Les groupes de ressources de pare-feu font-ils l’objet de restrictions ?

Oui. Le pare-feu, le réseau virtuel et l’IP publique doivent tous être dans le même groupe de ressources.

## <a name="when-configuring-dnat-for-inbound-internet-network-traffic-do-i-also-need-to-configure-a-corresponding-network-rule-to-allow-that-traffic"></a>Lorsque je configure DNAT pour le trafic Internet entrant, dois-je également configurer une règle de réseau correspondante pour autoriser ce trafic ?

Non. Les règles NAT ajoutent implicitement une règle de réseau correspondante pour autoriser le trafic traduit. Vous pouvez remplacer ce comportement en ajoutant explicitement une collection de règles de réseau avec des règles de refus correspondant au trafic traduit. Pour plus d’informations sur la logique de traitement des règles de Pare-feu Azure, consultez l’article [Logique de traitement des règles du service Pare-feu Azure](rule-processing.md).

## <a name="how-do-wildcards-work-in-an-application-rule-target-fqdn"></a>Comment fonctionnent les caractères génériques dans un nom de domaine complet cible d’une règle d’application ?

Les caractères génériques ne peuvent être utilisés que sur le côté gauche du FQDN. Par exemple, **_.contoso.com_* et **_contoso.com_*.

Si vous configurez **_.contoso.com_*, cela autorise *toutevaleur*.contoso.com, mais pas contoso.com (l’apex de domaine). Si vous souhaitez autoriser le domaine apex, vous devez explicitement le configurer en tant que nom de domaine complet cible.

## <a name="what-does-provisioning-state-failed-mean"></a>Que signifie l’*état d’approvisionnement : Échec* ?

Chaque fois qu’une modification de configuration est appliquée, Pare-feu Azure tente de mettre à jour toutes ses instances de serveur principal sous-jacentes. Dans de rares cas, il se peut qu’une de ces instances de serveur principal ne soit pas mise à jour avec la nouvelle configuration et le processus de mise à jour s’arrête avec un état d’approvisionnement ayant échoué. Votre Pare-feu Azure est toujours opérationnel, mais la configuration appliquée peut se trouver dans un état incohérent, où certaines instances ont toujours la configuration précédente et d’autres ont l’ensemble de règles mis à jour. Si cela se produit, essayez de mettre à jour votre configuration une fois de plus jusqu’à la réussite de l’opération que votre Pare-feu aie l’état d’approvisionnement *Réussite*.

## <a name="how-does-azure-firewall-handle-planned-maintenance-and-unplanned-failures"></a>Comment le pare-feu Azure gère la maintenance planifiée et les défaillances non planifiées ?
Le pare-feu Azure est constitué de plusieurs nœuds back-end dans une configuration active-active.  Pour toute activité de maintenance planifiée, notre logique de drainage des connexions permet de mettre à jour les nœuds de manière appropriée.  Pour limiter le risque d’interruption, les mises à jour sont planifiées en dehors des heures d’ouverture, et ce pour chaque région Azure.  Pour les problèmes non planifiés, nous instancions un nouveau nœud pour remplacer le nœud défaillant.  La connectivité avec le nouveau nœud est généralement rétablie dans les 10 secondes suivant la défaillance.

## <a name="how-does-connection-draining-work"></a>Comment fonctionne ce drainage des connexions ?

Pour toute activité de maintenance planifiée, la logique de drainage des connexions met à jour les nœuds principaux de manière appropriée. Pare-feu Azure attend 90 secondes que les connexions existantes se ferment. Si nécessaire, les clients peuvent automatiquement rétablir la connectivité à un autre nœud principal.

## <a name="is-there-a-character-limit-for-a-firewall-name"></a>Une limite de caractères s'applique-t-elle aux noms des pare-feu ?

Oui. Le nom d’un pare-feu est limité à 50 caractères.

## <a name="why-does-azure-firewall-need-a-26-subnet-size"></a>Pourquoi le service Pare-feu Azure impose-t-il une taille de sous-réseau de /26 ?

Le Pare-feu Azure doit approvisionner davantage de machines virtuelles au fil de sa mise à l'échelle. Avec un espace d'adressage de /26, le pare-feu dispose de suffisamment d'adresses IP pour prendre en charge la mise à l'échelle.

## <a name="does-the-firewall-subnet-size-need-to-change-as-the-service-scales"></a>La taille du sous-réseau du pare-feu doit-elle changer au fil de la mise à l'échelle du service ?

Non. Un sous-réseau de /26 suffit au Pare-feu Azure.

## <a name="how-can-i-increase-my-firewall-throughput"></a>Comment puis-je augmenter le débit de mon pare-feu ?

La capacité de débit initiale de Pare-feu Azure est de 2,5 à 3 Gbits/s et monte en charge jusqu’à 30 Gbits/s. Elle s’adapte automatiquement en fonction de l’utilisation de l’UC et du débit.

## <a name="how-long-does-it-take-for-azure-firewall-to-scale-out"></a>Combien de temps le scale-out du Pare-feu Azure prend-il ?

Le Pare-feu Azure s'adapte progressivement lorsque le débit moyen ou la consommation du processeur atteint 60 %. Un débit maximal de déploiement par défaut est d’environ 2,5 à 3 Gbits/s et son scale-out commence lorsqu’il atteint 60 % de cette valeur. Cette opération prend de cinq à sept minutes. 

Lors des tests de performances, veillez à tester pendant au moins 10 à 15 minutes et à initier de nouvelles connexions pour profiter des nœuds de pare-feu nouvellement créés.

## <a name="does-azure-firewall-allow-access-to-active-directory-by-default"></a>Le Pare-feu Azure autorise-t-il l’accès à Active Directory par défaut ?

Non. Le Pare-feu Azure bloque l’accès à Active Directory par défaut. Pour autoriser l’accès, configurez l’étiquette du service Azure Active Directory. Pour plus d’informations, consultez [Étiquettes du service Pare-feu Azure](service-tags.md).

## <a name="can-i-exclude-a-fqdn-or-an-ip-address-from-azure-firewall-threat-intelligence-based-filtering"></a>Puis-je exclure un nom de domaine complet (FSDN) ou une adresse IP du filtrage basé sur la Threat Intelligence du Pare-feu Azure ?

Oui, vous pouvez utiliser Azure PowerShell pour le faire :

```azurepowershell
# Add a Threat Intelligence allow list to an Existing Azure Firewall

## Create the allow list with both FQDN and IPAddresses

$fw = Get-AzFirewall -Name "Name_of_Firewall" -ResourceGroupName "Name_of_ResourceGroup"
$fw.ThreatIntelWhitelist = New-AzFirewallThreatIntelWhitelist `
   -FQDN @("fqdn1", "fqdn2", …) -IpAddress @("ip1", "ip2", …)

## Or Update FQDNs and IpAddresses separately

$fw = Get-AzFirewall -Name $firewallname -ResourceGroupName $RG
$fw.ThreatIntelWhitelist.IpAddresses = @($fw.ThreatIntelWhitelist.IpAddresses + $ipaddresses)
$fw.ThreatIntelWhitelist.fqdns = @($fw.ThreatIntelWhitelist.fqdns + $fqdns)


Set-AzFirewall -AzureFirewall $fw
```

## <a name="why-can-a-tcp-ping-and-similar-tools-successfully-connect-to-a-target-fqdn-even-when-no-rule-on-azure-firewall-allows-that-traffic"></a>Pourquoi un test Ping TCP et des outils similaires peuvent-ils se connecter à un nom de domaine complet cible même si aucune règle n’autorise ce trafic sur le Pare-feu Azure ?

Un test Ping TCP ne se connecte pas réellement au nom de domaine complet cible. En effet, le proxy transparent du Pare-feu Azure écoute le trafic sortant sur le port 80/443. Le test Ping TCP établit une connexion avec le pare-feu, qui supprime ensuite le paquet. Ce comportement n’a aucun impact sur la sécurité. Toutefois, pour éviter toute confusion, nous étudions la possibilité de modifier ce comportement.

## <a name="are-there-limits-for-the-number-of-ip-addresses-supported-by-ip-groups"></a>Le nombre d’adresses IP prises en charge par les groupes IP est-il limité ?

Oui. Pour plus d’informations, consultez [Abonnement Azure et limites, quotas et contraintes de service](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-firewall-limits)

## <a name="can-i-move-an-ip-group-to-another-resource-group"></a>Puis-je déplacer un groupe IP vers un autre groupe de ressources ?

Non, le déplacement d’un groupe IP vers un autre groupe de ressources n’est pas pris en charge actuellement.

## <a name="what-is-the-tcp-idle-timeout-for-azure-firewall"></a>Quel est le délai d’inactivité TCP pour le pare-feu Azure ?

Le comportement standard d’un pare-feu réseau consiste à garantir que les connexions TCP restent actives et à les fermer rapidement en l’absence d’activité. Le délai d’inactivité TCP du pare-feu Azure est de quatre minutes. Ce paramètre n’est pas configurable. Si une période d’inactivité est supérieure à la valeur de délai d’expiration, le maintien de la session TCP ou HTTP n’est pas garanti. Une pratique courante consiste à utiliser TCP keep-alive. Cela permet de maintenir la connexion active pendant une période plus longue. Pour plus d’informations, consultez ces [exemples .NET](/dotnet/api/system.net.servicepoint.settcpkeepalive?view=netcore-3.1#System_Net_ServicePoint_SetTcpKeepAlive_System_Boolean_System_Int32_System_Int32_).

## <a name="can-i-deploy-azure-firewall-without-a-public-ip-address"></a>Puis-je déployer un pare-feu Azure sans adresse IP publique ?

Non. Actuellement, vous devez déployer le pare-feu Azure avec une adresse IP publique.

## <a name="where-does-azure-firewall-store-customer-data"></a>Où le Pare-feu Azure stocke-t-il les données client ?

Le Pare-feu Azure ne déplace pas ni ne stocke les données client en dehors de la région dans laquelle il est déployé.