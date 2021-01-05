---
title: Forum aux questions sur Azure Application Gateway
description: Trouvez des réponses aux questions les plus fréquentes sur Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 05/26/2020
ms.author: victorh
ms.custom: references_regions
ms.openlocfilehash: d46c1d8efb5234d47c80ca4256c2f56d56ccf805
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862171"
---
# <a name="frequently-asked-questions-about-application-gateway"></a>Forum aux questions sur Application Gateway

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vous trouverez ci-dessous les questions fréquentes sur Azure Application Gateway.

## <a name="general"></a>Général

### <a name="what-is-application-gateway"></a>Présentation de Application Gateway

Le service Azure Application Gateway fournit un contrôleur de livraison d'applications (ADC) sous forme de service. Il offre diverses fonctionnalités d’équilibrage de charge de couche 7 pour vos applications. Ce service est hautement disponible, évolutif et complètement managé par Azure.

### <a name="what-features-does-application-gateway-support"></a>Quelles sont les fonctionnalités prises en charge par Application Gateway ?

Application Gateway prend en charge la mise à l’échelle automatique, le déchargement TLS et TLS de bout en bout, un pare-feu d’application web (WAF), l’affinité de session basée sur les cookies, le routage basé sur le chemin d’accès de l’URL, l’hébergement de plusieurs sites, et bien plus encore. Pour obtenir une liste complète des fonctionnalités prises en charge, voir [Vue d’ensemble d’Application Gateway](./overview.md).

### <a name="how-do-application-gateway-and-azure-load-balancer-differ"></a>En quoi Application Gateway et Azure Load Balancer diffère-t-il ?

Application Gateway est un équilibreur de charge de couche 7, ce qui signifie qu’il fonctionne uniquement avec le trafic web (HTTP/HTTPS/WebSocket etHTTP/2). Il prend en charge des fonctionnalités telles que la terminaison TLS, l’affinité de session basée sur les cookies et le tourniquet (round robin) pour le trafic d’équilibrage de charge. Load Balancer équilibre la charge du trafic au niveau de la couche 4 (TCP ou UDP).

### <a name="what-protocols-does-application-gateway-support"></a>Quels sont les protocoles pris en charge par Application Gateway ?

Application Gateway prend en charge les protocoles HTTP, HTTPS, HTTP/2 et WebSocket.

### <a name="how-does-application-gateway-support-http2"></a>Application Gateway prend-il en charge le protocole HTTP/2 ?

Consultez [HTTP/2 support](./configuration-listeners.md#http2-support).

### <a name="what-resources-are-supported-as-part-of-a-backend-pool"></a>Quelles sont les ressources prises en charge dans un pool backend ?

Consultez [Ressources principales prises en charge](./application-gateway-components.md#backend-pools).

### <a name="in-what-regions-is-application-gateway-available"></a>Dans quelles régions Application Gateway est-il disponible ?

Application Gateway v1 (Standard et WAF) est disponible dans toutes les régions d’Azure global. Il est également disponible dans [Azure - Chine 21Vianet](https://www.azure.cn/) et [Azure Government](https://azure.microsoft.com/overview/clouds/government/).

Pour la disponibilité d’Application Gateway v2 (Standard_v2 et WAF_v2), consultez [Régions prises en charge pour Application Gateway v2](./application-gateway-autoscaling-zone-redundant.md#supported-regions)

### <a name="is-this-deployment-dedicated-for-my-subscription-or-is-it-shared-across-customers"></a>S’agit-il d’un déploiement dédié à mon abonnement ou est-il partagé entre les clients ?

Application Gateway est un déploiement dédié dans votre réseau virtuel.

### <a name="does-application-gateway-support-http-to-https-redirection"></a>Application Gateway prend-il en charge la redirection HTTP vers HTTPS ?

La redirection est prise en charge. Consultez [Vue d’ensemble de la redirection Application Gateway](./redirect-overview.md).

### <a name="in-what-order-are-listeners-processed"></a>Dans quel ordre les écouteurs sont-ils traités ?

Consultez l'[ordre de traitement des écouteurs](./configuration-listeners.md#order-of-processing-listeners).

### <a name="where-do-i-find-the-application-gateway-ip-and-dns"></a>Où puis-je trouver l’adresse IP et le DNS d’Application Gateway ?

Si vous utilisez une adresse IP publique en tant que point de terminaison, vous trouverez les informations d’adresse IP et de DNS dans la ressource d’adresse IP publique. Elles sont également disponibles sur la page Vue d'ensemble de la passerelle d’application. Si vous utilisez des adresses IP internes, ces informations se trouvent dans la page Vue d'ensemble.

Pour la référence SKU v2, ouvrez la ressource IP publique et sélectionnez **Configuration**. Le champ **Étiquette du nom DNS (facultatif)** est disponible pour configurer le nom DNS.

### <a name="what-are-the-settings-for-keep-alive-timeout-and-tcp-idle-timeout"></a>Quels sont les paramètres du délai de maintien de connexion et du délai d’inactivité TCP ?

Le *délai d’expiration Keep-Alive* régit la durée pendant laquelle Application Gateway attend qu’un client envoie une autre requête HTTP sur une connexion permanente avant de la réutiliser ou de la fermer. Le *délai d’inactivité TCP* régit la durée pendant laquelle une connexion TCP est maintenue ouverte en cas d’absence d’activité. 

Le *délai d’expiration Keep-Alive* est de 120 secondes dans la référence SKU Application Gateway v1 et de 75 secondes dans la référence SKU v2. Par défaut, le *délai d’inactivité TCP* est de 4 minutes sur l’adresse IP virtuelle du serveur frontal des SKU v1 et v2 d’Application Gateway. Vous pouvez configurer la valeur du délai d’inactivité TCP sur les passerelles d’applications v1 et v2 pour qu’elle soit comprise entre 4 et 30 minutes. Pour les passerelles d’application v1 et v2, vous devrez accéder à l’IP publique de la passerelle d’application et modifier le délai d’inactivité TCP sous le panneau « Configuration » de l’IP publique sur le portail. Vous pouvez définir la valeur du délai d’inactivité TCP de l’IP publique via PowerShell en exécutant les commandes suivantes : 

```azurepowershell-interactive
$publicIP = Get-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyResourceGroup
$publicIP.IdleTimeoutInMinutes = "15"
Set-AzPublicIpAddress -PublicIpAddress $publicIP
```

### <a name="does-the-ip-or-dns-name-change-over-the-lifetime-of-the-application-gateway"></a>L’adresse IP ou le nom DNS changent-ils pendant la durée de vie d’Application Gateway ?

Dans la référence SKU Application Gateway V1, l’adresse IP virtuelle peut changer si vous arrêtez et démarrez la passerelle d’application. Le nom DNS associé à la passerelle d’application, quant à lui, ne change pas pendant toute la durée de vie de la passerelle. Le nom DNS ne changeant pas, vous devez utiliser un alias CNAME et le faire pointer vers l’adresse DNS de la passerelle d’application. Dans la référence SKU Application Gateway V2, vous pouvez définir l’adresse IP comme étant statique, de sorte que l’adresse IP et le nom DNS ne changent pas pendant la durée de vie de la passerelle d’application. 

### <a name="does-application-gateway-support-static-ip"></a>Application Gateway prend-il en charge les adresses IP statiques ?

Oui. La référence SKU v2 d’Application Gateway prend en charge les adresses IP publiques statiques. La référence SKU v1 quant à elle prend en charge les adresses IP internes statiques.

### <a name="does-application-gateway-support-multiple-public-ips-on-the-gateway"></a>Application Gateway prend-il en charge plusieurs adresses IP publiques sur la passerelle ?

Une seule adresse IP publique est prise en charge sur une passerelle d’application.

### <a name="how-large-should-i-make-my-subnet-for-application-gateway"></a>Quelle taille mon sous-réseau pour Application Gateway doit-il avoir ?

Consultez [Considérations relatives à la taille du sous-réseau Application Gateway](./configuration-infrastructure.md#size-of-the-subnet).

### <a name="can-i-deploy-more-than-one-application-gateway-resource-to-a-single-subnet"></a>Puis-je déployer plusieurs ressources Application Gateway dans un seul sous-réseau ?

Oui. En plus de plusieurs instances d’un déploiement Application Gateway donné, vous pouvez configurer une autre ressource Application Gateway unique dans un sous-réseau existant qui contient une autre ressource Application Gateway.

Un seul sous-réseau ne peut pas prendre en charge les références (SKU) v2 et v1 d’Application Gateway.

### <a name="does-application-gateway-v2-support-user-defined-routes-udr"></a>Application Gateway v2 prend-il en charge les itinéraires définis par l’utilisateur (UDR) ?

Oui, mais uniquement dans des scénarios spécifiques. Pour plus d’informations, consultez [Configuration de l’infrastructure Application Gateway](configuration-infrastructure.md#supported-user-defined-routes).

### <a name="does-application-gateway-support-x-forwarded-for-headers"></a>Application Gateway prend-il en charge les en-têtes x-forwarded-for ?

Oui. Consultez [Modifications apportées à une requête](./how-application-gateway-works.md#modifications-to-the-request).

### <a name="how-long-does-it-take-to-deploy-an-application-gateway-will-my-application-gateway-work-while-its-being-updated"></a>Combien de temps faut-il pour déployer une instance Application Gateway ? Mon instance Application Gateway continue-t-elle de fonctionner lorsqu’elle est mise à jour ?

La configuration des déploiements de nouvelles références SKU v1 d’Application Gateway peut prendre jusqu’à 20 minutes. Les modifications apportées à la taille ou au nombre des instances n’entraînent pas d’interruption, et la passerelle reste active pendant ce temps.

Pour la plupart des déploiements qui utilisent la référence SKU v2, le provisionnement prend environ 6 minutes. Toutefois, cette opération peut prendre plus de temps en fonction du type de déploiement. Par exemple, les déploiements sur plusieurs zones de disponibilité avec de nombreuses instances peuvent prendre plus de 6 minutes. 

### <a name="can-i-use-exchange-server-as-a-backend-with-application-gateway"></a>Puis-je utiliser Exchange Server en tant que serveur principal avec Application Gateway ?

Non. Application Gateway ne prend pas en charge les protocoles de messagerie tels que SMTP, IMAP et POP3.

### <a name="is-there-guidance-available-to-migrate-from-the-v1-sku-to-the-v2-sku"></a>Existe-t-il des instructions pour migrer de la référence SKU v1 vers la référence SKU v2 ?

Oui. Pour plus d'informations, consultez [Migrer Azure Application Gateway et le pare-feu d’applications web de v1 à v2](migrate-v1-v2.md).

### <a name="will-the-application-gateway-v1-sku-continue-to-be-supported"></a>Le support de la référence SKU Application Gateway v1 va-t-il continuer ?

Oui. Le support de la référence SKU Application Gateway v1 continue d’être assuré. Cependant, il est fortement recommandé de passer à v2 pour tirer parti des mises à jour des fonctionnalités de cette référence SKU. Pour plus d’informations, consultez [Application Gateway v2 avec mise à l’échelle automatique et redondance interzone](application-gateway-autoscaling-zone-redundant.md).

### <a name="does-application-gateway-v2-support-proxying-requests-with-ntlm-authentication"></a>Application Gateway v2 prend-il en charge les demandes de proxy avec l’authentification NTLM ?

Non. Application Gateway v2 ne prend pas en charge les demandes de proxy avec l’authentification NTLM.

### <a name="does-application-gateway-affinity-cookie-support-samesite-attribute"></a>Le cookie d’affinité d’Application Gateway prend-il en charge l’attribut SameSite ?
Oui, la [mise à jour v80](https://chromiumdash.appspot.com/schedule) du [navigateur Chromium](https://www.chromium.org/Home) a permis que les cookies HTTP sans attribut SameSite soient traités comme SameSite=Lax. Cela signifie que le cookie d’affinité d’Application Gateway ne sera pas envoyé par le navigateur dans un contexte tiers. 

Pour prendre en charge ce scénario, Application Gateway injecte un autre cookie appelé *ApplicationGatewayAffinityCORS* en plus du cookie *ApplicationGatewayAffinity* existant.  Ces cookies sont similaires, mais le cookie *ApplicationGatewayAffinityCORS* a deux attributs supplémentaires qui lui sont ajoutés : *SameSite=None; Secure*. Ces attributs maintiennent les sessions rémanentes même pour les requêtes cross-origin. Pour plus d’informations, consultez la [section Affinité basée sur les cookies](configuration-http-settings.md#cookie-based-affinity).

## <a name="performance"></a>Performances

### <a name="how-does-application-gateway-support-high-availability-and-scalability"></a>Comment Application Gateway prend-il en charge la haute disponibilité et la scalabilité ?

La référence SKU v1 d’Application Gateway prend en charge les scénarios de haute disponibilité lorsque vous avez déployé deux instances ou plus. Azure distribue ces instances entre les domaines de mise à jour et d’erreur pour garantir qu'elles n’échouent pas toutes en même temps. La référence SKU v1 prend en charge la scalabilité en ajoutant plusieurs instances de la même passerelle pour partager la charge.

La référence SKU v2 garantit automatiquement que les nouvelles instances sont réparties sur les domaines d’erreur et les domaines de mise à jour. Si vous choisissez une redondance de zone, les instances les plus récentes sont également réparties sur les zones de disponibilité pour offrir une résilience zonale en cas d’échec.

### <a name="how-do-i-achieve-a-dr-scenario-across-datacenters-by-using-application-gateway"></a>Comment puis-je obtenir un scénario DR dans les centres de données à l'aide d'Application Gateway ?

Utilisez Traffic Manager pour répartir le trafic entre plusieurs instances Application Gateway dans différents centres de données.

### <a name="does-application-gateway-support-autoscaling"></a>Application Gateway prend-il en charge la mise à l'échelle automatique ?

Oui. La référence SKU v2 d’Application Gateway prend en charge la mise à l’échelle automatique. Pour plus d’informations, consultez [Application Gateway avec mise à l’échelle automatique et redondance interzone](application-gateway-autoscaling-zone-redundant.md).

### <a name="does-manual-or-automatic-scale-up-or-scale-down-cause-downtime"></a>L’opération de scale-up ou scale-down manuelle ou automatique entraîne-t-elle un temps d’arrêt ?

Non. Aucun temps d’arrêt n’a lieu, les instances sont réparties entre les domaines de mise à niveau et les domaines d’erreur.

### <a name="does-application-gateway-support-connection-draining"></a>Application Gateway prend-il en charge le drainage de connexion ?

Oui. Vous pouvez configurer le drainage de connexion afin de modifier des membres au sein d’un pool principal sans interrompre le service. Pour plus d’informations, consultez la [section relative au drainage des connexions d’Application Gateway](features.md#connection-draining).

### <a name="can-i-change-instance-size-from-medium-to-large-without-disruption"></a>Puis-je passer d’une taille moyenne à une grande taille d’instance sans interruption de service ?

Oui.

## <a name="configuration"></a>Configuration

### <a name="is-application-gateway-always-deployed-in-a-virtual-network"></a>Application Gateway est-il toujours déployé dans un réseau virtuel ?

Oui. Application Gateway est toujours déployé dans un sous-réseau de réseau virtuel. Ce sous-réseau ne peut contenir que des instances Application Gateway. Pour plus d’informations, consultez [Exigences de réseau virtuel et de sous-réseau](./configuration-infrastructure.md#virtual-network-and-dedicated-subnet).

### <a name="can-application-gateway-communicate-with-instances-outside-of-its-virtual-network-or-outside-of-its-subscription"></a>Application Gateway peut-il communiquer avec des instances en dehors de son réseau virtuel ou de son abonnement ?

Sous réserve de disposer d'une connectivité IP, Application Gateway peut communiquer avec des instances en dehors du réseau virtuel dans lequel il se trouve. Application Gateway peut également communiquer avec des instances extérieures à l'abonnement dans lequel il se trouve. Si vous envisagez d'utiliser des adresses IP internes en tant que membres de pool de back-ends, utilisez le [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md) ou la [passerelle VPN Azure](../vpn-gateway/vpn-gateway-about-vpngateways.md).

### <a name="can-i-deploy-anything-else-in-the-application-gateway-subnet"></a>Puis-je déployer autre chose dans le sous-réseau de la passerelle d’application ?

Non. Mais vous pouvez déployer d’autres passerelles d’application dans le sous-réseau.

### <a name="are-network-security-groups-supported-on-the-application-gateway-subnet"></a>Les groupes de sécurité réseau sont-ils pris en charge dans le sous-réseau de la passerelle d’application ?

Consultez [Groupes de sécurité réseau dans le sous-réseau Application Gateway](./configuration-infrastructure.md#network-security-groups).

### <a name="does-the-application-gateway-subnet-support-user-defined-routes"></a>Le sous-réseau de la passerelle d'application prend-il en charge les routes définies par l’utilisateur ?

Consultez [Les routes définies par l’utilisateur prises en charge dans le sous-réseau Application Gateway](./configuration-infrastructure.md#supported-user-defined-routes).

### <a name="are-service-endpoint-policies-supported-in-the-application-gateway-subnet"></a>Les stratégies de point de terminaison de service sont-elles prises en charge dans le sous-réseau Application Gateway ?

Non. Les [stratégies de point de terminaison de service](../virtual-network/virtual-network-service-endpoint-policies-overview.md) pour les comptes de stockage ne sont pas prises en charge dans un sous-réseau Application Gateway sous-réseau, et leur configuration bloque le trafic d’infrastructure Azure.

### <a name="what-are-the-limits-on-application-gateway-can-i-increase-these-limits"></a>Quelles sont les limites d’Application Gateway ? Puis-je augmenter ces limites ?

Consultez [Limites d’Application Gateway](../azure-resource-manager/management/azure-subscription-service-limits.md#application-gateway-limits).

### <a name="can-i-simultaneously-use-application-gateway-for-both-external-and-internal-traffic"></a>Puis-je utiliser simultanément Application Gateway pour le trafic externe et interne ?

Oui. Application Gateway peut avoir une adresse IP interne et une adresse IP externe par passerelle d’application.

### <a name="does-application-gateway-support-virtual-network-peering"></a>Application Gateway prend-il en charge le peering de réseaux virtuels ?

Oui. Le peering de réseaux virtuels permet d'équilibrer la charge du trafic dans d'autres réseaux virtuels.

### <a name="can-i-talk-to-on-premises-servers-when-theyre-connected-by-expressroute-or-vpn-tunnels"></a>Puis-je communiquer avec les serveurs locaux lorsqu’ils sont connectés via ExpressRoute ou des tunnels VPN ?

Oui, tant que le trafic est autorisé.

### <a name="can-one-backend-pool-serve-many-applications-on-different-ports"></a>Un pool back-end peut-il servir plusieurs applications sur des ports différents ?

L’architecture orientée microservices est prise en charge. Pour sonder différents ports, vous devez configurer plusieurs paramètres HTTP.

### <a name="do-custom-probes-support-wildcards-or-regex-on-response-data"></a>Les sondes personnalisées prennent-elles en charge les caractères génériques ou les expressions régulières sur les données de réponse ?

Non. 

### <a name="how-are-routing-rules-processed-in-application-gateway"></a>Comment les règles d'acheminement sont-elles traitées dans Application Gateway ?

Consultez [Ordre de traitement des règles](./configuration-request-routing-rules.md#order-of-processing-rules).

### <a name="for-custom-probes-what-does-the-host-field-signify"></a>À quoi correspond le champ Hôte pour les sondes personnalisées ?

Le champ hôte Spécifie le nom auquel envoyer la sonde lorsque vous avez configuré plusieurs sites sur Application Gateway. Sinon, utilisez « 127.0.0.1 ». Cette valeur est différente du nom d’hôte de la machine virtuelle. Son format est \<protocol\>://\<host\>:\<port\>\<path\>.

### <a name="can-i-allow-application-gateway-access-to-only-a-few-source-ip-addresses"></a>Puis-je autoriser l’accès d’Application Gateway à quelques adresses IP sources uniquement ?

Oui. Consultez [Restreindre l’accès à des adresses IP sources spécifiques](./configuration-infrastructure.md#allow-access-to-a-few-source-ips).

### <a name="can-i-use-the-same-port-for-both-public-facing-and-private-facing-listeners"></a>Puis-je utiliser le même port pour les écouteurs publics et privés ?

Non.

### <a name="does-application-gateway-support-ipv6"></a>Application Gateway prend-il en charge IPv6 ?

Application Gateway v2 ne prend pas en charge IPv6 pour le moment. Il peut fonctionner dans un réseau virtuel à double pile à l’aide de IPv4 uniquement, mais le sous-réseau de passerelle doit être uniquement en IPv4. Application Gateway v1 ne prend pas en charge la double pile Réseaux virtuels. 

### <a name="how-do-i-use-application-gateway-v2-with-only-private-frontend-ip-address"></a>Comment utiliser Application Gateway v2 avec uniquement une adresse IP frontend privée ?

Application Gateway v2 ne prend actuellement pas en charge le mode IP privé. Cette méthode prend en charge les combinaisons suivantes :
* adresse IP privée et adresse IP publique
* adresse IP publique uniquement

Toutefois, si vous souhaitez utiliser Application Gateway v2 avec uniquement une adresse IP privée, vous pouvez suivre le processus ci-dessous :
1. Créer une Application Gateway avec une adresse IP frontend publique et privée
2. Ne créez aucun écouteur pour l’adresse IP de serveur frontend public. Application Gateway n’écoute aucun trafic sur l’adresse IP publique si aucun écouteur n’est créé pour celui-ci.
3. Créez et associez un [groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md) pour le sous-réseau Application Gateway avec la configuration suivante dans l’ordre de priorité :
    
    a. Autorisez le trafic de la source en tant que balise de service **GatewayManager** et de la destination en tant que **N’importe laquelle** et du port de destination en tant que **65200-65535**. Cette plage de ports est nécessaire pour la communication avec l’infrastructure Azure. Ces ports sont protégés (verrouillés) par l’authentification par certificat. Les entités externes, y compris les administrateurs d’utilisateurs de la passerelle, ne peuvent pas initier de modifications sur ces points de terminaison sans que les certificats appropriés soient en place.
    
    b. Autorisez le trafic de la source en tant que balise de service **AzureLoadBalancer** et de la destination en tant que **N’importe laquelle**.
    
    c. Refusez tout trafic entrant à partir de la source en tant que balise de service **Internet** et la destination et port de destination en tant que **N’importe laquelle**. Donnez à cette règle la *priorité la plus faible* dans les règles de trafic entrant
    
    d. Conserver les règles par défaut, comme autoriser les VirtualNetwork entrants, afin que l’accès à l’adresse IP privée ne soit pas bloqué
    
    e. La connectivité Internet sortante ne peut pas être bloquée. Dans le cas contraire, vous rencontrerez des problèmes avec la journalisation, les mesures, etc.

Exemple de configuration de groupe de sécurité réseau pour un accès d’adresse IP privée uniquement : ![Configuration Application Gateway v2 NSG pour l’accès IP privé uniquement](./media/application-gateway-faq/appgw-privip-nsg.png)

## <a name="configuration---tls"></a>Configuration - TLS

### <a name="what-certificates-does-application-gateway-support"></a>Quels sont les certificats pris en charge par Application Gateway ?

Application Gateway prend en charge les certificats auto-signés, les certificats d’autorité de certification (CA), les certificats de validation étendue (EV), les certificats à plusieurs domaines (SAN) et les certificats génériques.

### <a name="what-cipher-suites-does-application-gateway-support"></a>Quelles sont les suites de chiffrement prises en charge par Application Gateway ?

Application Gateway prend en charge les suites de chiffrement ci-dessous. 

- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
- TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_DHE_RSA_WITH_AES_256_CBC_SHA
- TLS_DHE_RSA_WITH_AES_128_CBC_SHA
- TLS_RSA_WITH_AES_256_GCM_SHA384
- TLS_RSA_WITH_AES_128_GCM_SHA256
- TLS_RSA_WITH_AES_256_CBC_SHA256
- TLS_RSA_WITH_AES_128_CBC_SHA256
- TLS_RSA_WITH_AES_256_CBC_SHA
- TLS_RSA_WITH_AES_128_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
- TLS_DHE_DSS_WITH_AES_256_CBC_SHA256
- TLS_DHE_DSS_WITH_AES_128_CBC_SHA256
- TLS_DHE_DSS_WITH_AES_256_CBC_SHA
- TLS_DHE_DSS_WITH_AES_128_CBC_SHA
- TLS_RSA_WITH_3DES_EDE_CBC_SHA
- TLS_DHE_DSS_WITH_3DES_EDE_CBC_SHA

Pour savoir comment personnaliser les options TLS, consultez [Configurer les versions de stratégie TLS et les suites de chiffrement sur Application Gateway](application-gateway-configure-ssl-policy-powershell.md).

### <a name="does-application-gateway-support-reencryption-of-traffic-to-the-backend"></a>Application Gateway prend-il en charge le nouveau chiffrement du trafic sur le back-end ?

Oui. Application Gateway prend en charge le déchargement TLS et TLS de bout en bout, qui chiffre à nouveau le trafic vers le serveur principal.

### <a name="can-i-configure-tls-policy-to-control-tls-protocol-versions"></a>Puis-je configurer la stratégie TLS de manière à contrôler les versions du protocole TLS ?

Oui. Vous pouvez configurer Application Gateway pour refuser TLS1.0, TLS1.1 et TLS1.2. Par défaut, SSL 2.0 et 3.0 sont déjà désactivés et ne sont pas configurables.

### <a name="can-i-configure-cipher-suites-and-policy-order"></a>Puis-je configurer les suites de chiffrement et l’ordre de la stratégie ?

Oui. Dans Application Gateway, vous pouvez [configurer des suites de chiffrement](application-gateway-ssl-policy-overview.md). Pour définir une stratégie personnalisée, activez au moins une des suites de chiffrement suivantes. 

* TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 
* TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
* TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
* TLS_RSA_WITH_AES_128_GCM_SHA256
* TLS_RSA_WITH_AES_256_CBC_SHA256
* TLS_RSA_WITH_AES_128_CBC_SHA256

Application Gateway utilise SHA256 pour la gestion des serveurs principaux.

### <a name="how-many-tlsssl-certificates-does-application-gateway-support"></a>Combien de certificats TLS/SSL Application Gateway prend-il en charge ?

Application Gateway prend en charge jusqu'à 100 certificats TLS/SSL.

### <a name="how-many-authentication-certificates-for-backend-reencryption-does-application-gateway-support"></a>Combien de certificats d’authentification pour le nouveau chiffrement du back-end Application Gateway prend-il en charge ?

Application Gateway prend en charge jusqu’à 100 certificats d’authentification.

### <a name="does-application-gateway-natively-integrate-with-azure-key-vault"></a>Application Gateway s’intègre-t-il en mode natif à Azure Key Vault ?

La référence SKU v2 d’Application Gateway prend en charge Key Vault. Pour plus d'informations, consultez [Arrêt de TLS avec des certificats Key Vault](key-vault-certs.md).

### <a name="how-do-i-configure-https-listeners-for-com-and-net-sites"></a>Comment configurer des écouteurs HTTPS pour les sites .com et .net ? 

Pour un acheminement (basé sur l'hôte) sur plusieurs domaines, vous pouvez créer des écouteurs multisites, configurer des écouteurs utilisant le protocole HTTPS et associer les écouteurs aux règles d'acheminement. Pour plus d’informations, consultez [Hébergement de plusieurs sites à l'aide d'Application Gateway](./multiple-site-overview.md).

### <a name="can-i-use-special-characters-in-my-pfx-file-password"></a>Puis-je utiliser des caractères spéciaux dans le mot de passe de mon fichier. pfx ?

Non, utilisez uniquement des caractères alphanumériques dans le mot de passe de votre fichier. pfx.

### <a name="my-ev-certificate-is-issued-by-digicert-and-my-intermediate-certificate-has-been-revoked-how-do-i-renew-my-certificate-on-application-gateway"></a>Mon certificat EV est émis par DigiCert et mon certificat intermédiaire a été révoqué. Comment faire renouveler mon certificat sur Application Gateway ?

Les membres de l’organisation Certification Authority Browser Forum ont récemment publié des rapports détaillant les différents certificats émis par les fournisseurs d’autorités de certification qui sont utilisés par nos clients, Microsoft et la communauté technologique dans son ensemble et qui n’étaient plus conformes aux normes du secteur pour les autorités de certification publiquement fiables. Les rapports concernant les autorités de certification non conformes sont accessibles ici :  

* [Bogue 1649951](https://bugzilla.mozilla.org/show_bug.cgi?id=1649951)
* [Bogue 1650910](https://bugzilla.mozilla.org/show_bug.cgi?id=1650910)

Conformément aux exigences du secteur en matière de conformité, les fournisseurs d’autorités de certification commençaient à révoquer les autorités de certification non conformes et émettaient des autorités de certification conformes, ce qui oblige les clients à émettre à nouveau leurs certificats. Microsoft travaille en étroite collaboration avec ces fournisseurs pour réduire l’impact potentiel sur les services Azure. **Toutefois, les certificats auto-émis ou ceux utilisés dans des scénarios BYOC (Bring Your Own Certificate) sont toujours susceptibles d’être révoqués de manière inattendue**.

Pour vérifier si les certificats utilisés par votre application ont été révoqués, reportez-vous à l’[annonce de DigiCert](https://knowledge.digicert.com/alerts/DigiCert-ICA-Replacement) et au [suivi des révocations de certificats](https://misissued.com/#revoked). Si vos certificats ont été révoqués ou le seront, vous devrez demander de nouveaux certificats auprès du fournisseur d’autorité de certification utilisé dans vos applications. Pour éviter que la disponibilité de votre application ne soit interrompue en raison d’une révocation inattendue de certificats, ou pour mettre à jour un certificat qui a été révoqué, reportez-vous à notre billet relatifs aux mises à jour Azure afin d’accéder aux liens de correction des différents services Azure qui prennent en charge BYOC : https://azure.microsoft.com/updates/certificateauthorityrevocation/.

Pour des informations spécifiques à Application Gateway, voir ci-dessous :

Si vous utilisez un certificat émis par l’une des autorités de certification révoquées, la disponibilité de votre application peut être interrompue et, en fonction de votre application, vous pouvez recevoir différents messages d’erreur, notamment : 

1.  Certificat non valide/certificat révoqué
2.  Délai de connexion dépassé
3.  HTTP 502

Pour éviter toute interruption de votre application en raison de ce problème, ou pour réémettre une autorité de certification qui a été révoquée, vous devez procéder comme suit : 

1.  Contactez votre fournisseur de certificats pour savoir comment émettre à nouveau vos certificats.
2.  Une fois vos certificats réémis, mettez-les à jour sur Azure Application Gateway/WAF avec la [chaîne de confiance](/windows/win32/seccrypto/certificate-chains) complète (certificats feuille, intermédiaire, racine). En fonction de l’emplacement où vous utilisez votre certificat, soit sur l’écouteur, soit sur les paramètres HTTP de la passerelle d’application, suivez les étapes ci-dessous pour mettre à jour les certificats et consultez les liens de documentation mentionnés pour plus d’informations.
3.  Mettez à jour vos serveurs d’applications principaux pour utiliser le certificat réémis. Selon le serveur principal que vous utilisez, les étapes de mise à jour de votre certificat peuvent varier. Consultez la documentation de votre fournisseur.

Pour mettre à jour le certificat dans votre écouteur :

1.  Dans le [portail Azure](https://portal.azure.com/), ouvrez votre ressource Application Gateway.
2.  Ouvrez les paramètres d’écouteur associés à votre certificat.
3.  Cliquez sur  Renouveler ou modifier le certificat sélectionné ».
4.  Chargez votre nouveau certificat PFX avec le mot de passe et cliquez sur Enregistrer.
5.  Accédez au site web et vérifiez que le site fonctionne comme prévu. Pour plus d’informations, consultez la documentation [ici](./renew-certificates.md).

Si vous référencez des certificats d’Azure Key Vault dans votre écouteur de passerelle d’application, nous vous recommandons d’utiliser les étapes suivantes pour une modification rapide :

1.  Dans le [portail Azure](https://portal.azure.com/), accédez aux paramètres Azure Key Vault qui ont été associés à la passerelle d’application.
2.  Ajoutez/importez le certificat réémis dans votre magasin. Pour plus d’informations sur les procédures, consultez la documentation [ici](../key-vault/certificates/quick-create-portal.md).
3.  Une fois le certificat importé, accédez aux paramètres d’écouteur de votre passerelle d’application puis, sous « Choisir un certificat à partir de Key Vault », cliquez sur la liste déroulante « Certificat » et choisissez le certificat récemment ajouté.
4.  Cliquez sur Enregistrer. Pour plus d’informations relatives à la terminaison TLS sur Application Gateway avec des certificats Key Vault, consultez la documentation [ici](./key-vault-certs.md).


Pour mettre à jour le certificat dans vos paramètres HTTP :

Si vous utilisez la SKU v1 du service Application Gateway/WAF, vous devez charger le nouveau certificat en tant que certificat d’authentification principal.
1.  Dans le [portail Azure](https://portal.azure.com/), ouvrez votre ressource Application Gateway.
2.  Ouvrez les paramètres HTTP associés à votre certificat.
3.  Cliquez sur « Ajouter un certificat » et chargez le certificat réémis, puis cliquez sur Enregistrer.
4.  Vous pouvez supprimer l’ancien certificat ultérieurement en cliquant sur le bouton d’options « … » à côté de l’ancien certificat et en sélectionnant Supprimer, puis cliquez sur Enregistrer.
Pour plus d’informations, consultez la documentation [ici](./end-to-end-ssl-portal.md#add-authenticationtrusted-root-certificates-of-back-end-servers).

Si vous utilisez la SKU v2 du service Application Gateway/WAF, vous n’êtes pas obligé de charger le nouveau certificat dans les paramètres HTTP, car la SKU v2 utilise des « certificat racine approuv » et aucune action n’est nécessaire ici.

## <a name="configuration---ingress-controller-for-aks"></a>Configuration - Contrôleur d’entrée pour AKS

### <a name="what-is-an-ingress-controller"></a>Qu’est-ce qu’un contrôleur d’entrée ?

Kubernetes permet de créer des ressources `deployment` et `service` pour exposer en interne un groupe de pods dans le cluster. Pour exposer le même service en externe, une ressource [`Ingress`](https://kubernetes.io/docs/concepts/services-networking/ingress/) est définie, ce qui permet l’équilibrage de charge, l’arrêt TLS et l’hébergement virtuel basé sur le nom.
Pour satisfaire cette ressource `Ingress`, un contrôleur d’entrée est nécessaire, qui écoute les modifications apportées aux ressources `Ingress` et configure les stratégies d’équilibreur de charge.

Le contrôleur d’entrée Application Gateway (AGIC) permet l’utilisation d’[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) en tant qu’entrée pour [Azure Kubernetes Service](https://azure.microsoft.com/services/kubernetes-service/), également appelé cluster AKS.

### <a name="can-a-single-ingress-controller-instance-manage-multiple-application-gateways"></a>Une seule instance de contrôleur d’entrée peut-elle gérer plusieurs Application Gateways ?

Actuellement, une instance de contrôleur d’entrée ne peut être associée qu’à une seule Application Gateway.

### <a name="why-is-my-aks-cluster-with-kubenet-not-working-with-agic"></a>Pourquoi mon cluster AKS avec kubenet ne fonctionne-t-il pas avec l’AGIC ?

L’AGIC essaie d’associer automatiquement la ressource de la table de routage au sous-réseau de la passerelle d’application, mais peut ne pas y parvenir en raison du manque d’autorisations de l’AGIC. Si l’AGIC n’est pas en mesure d’associer la table de routage au sous-réseau de la passerelle d’application, une erreur se produira dans les journaux de l’AGIC. Dans ce cas, vous devrez associer manuellement la table de routage créée par le cluster AKS au sous-réseau de la passerelle d’application. Pour plus d’informations, consultez [Routes définies par l’utilisateur prises en charge](configuration-infrastructure.md#supported-user-defined-routes).

### <a name="can-i-connect-my-aks-cluster-and-application-gateway-in-separate-virtual-networks"></a>Puis-je connecter mon cluster AKS et Application Gateway dans des réseaux virtuels distincts ? 

Oui, tant que les réseaux virtuels font l’objet d’un peering et qu’ils n’ont pas d’espaces d’adressage qui se chevauchent. Si vous exécutez AKS avec kubenet, veillez à associer la table de routage générée par AKS au sous-réseau de la passerelle d’application. 

### <a name="what-features-are-not-supported-on-the-agic-add-on"></a>Quelles fonctionnalités ne sont pas prises en charge sur le module complémentaire AGIC ? 

Consultez [ici](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on) les différences entre l’AGIC déployé via Helm et celui déployé en tant que module complémentaire AKS.

### <a name="when-should-i-use-the-add-on-versus-the-helm-deployment"></a>Quand dois-je utiliser le module complémentaire plutôt que le déploiement Helm ? 

Consultez [ici](ingress-controller-overview.md#difference-between-helm-deployment-and-aks-add-on) les différences entre l’AGIC déployé via Helm et celui déployé en tant que module complémentaire AKS, en particulier les tables qui documentent les scénarios pris en charge par l’AGIC déployé via Helm, par opposition à un module complémentaire AKS. En général, le déploiement via Helm vous permet de tester les fonctionnalités bêta et les versions finales (RC) avant leur publication officielle. 

### <a name="can-i-control-which-version-of-agic-will-be-deployed-with-the-add-on"></a>Puis-je contrôler quelle version de l’AGIC sera déployée avec le module complémentaire ?

Non, le module complémentaire AGIC est un service géré, ce qui signifie que Microsoft met automatiquement à jour le module complémentaire vers la version stable la plus récente. 

## <a name="diagnostics-and-logging"></a>Diagnostics et journalisation

### <a name="what-types-of-logs-does-application-gateway-provide"></a>Quels types de journaux Application Gateway fournit-il ?

Application Gateway fournit trois types de journaux : 

* **ApplicationGatewayAccessLog** : le journal d’accès contient toutes les requêtes envoyées au serveur frontal de la passerelle d’application. Les données incluent l’adresse IP de l’appelant, l’URL demandée, la latence de réponse, le code de retour, et les octets d’entrée et de sortie. Il contient un enregistrement par instance Application Gateway.
* **ApplicationGatewayPerformanceLog** : Le journal des performances capture les informations de performances de chaque instance d'Application Gateway. Parmi les informations consignées figurent notamment le débit en octets, le nombre total de requêtes traitées, le nombre de requêtes ayant échoué, ainsi que le nombre d'instances de serveur principal saines ou non saines.
* **ApplicationGatewayFirewallLog** : Pour les passerelles d’application que vous configurez avec le pare-feu d'applications web, le journal du pare-feu contient les requêtes consignées via le mode de détection ou le mode de prévention.

Tous les journaux sont collectés toutes les 60 secondes. Pour plus d’informations, consultez [Intégrité du serveur principal, journaux de diagnostics et métriques pour Application Gateway](application-gateway-diagnostics.md).

### <a name="how-do-i-know-if-my-backend-pool-members-are-healthy"></a>Comment savoir si les membres de mon pool back-end sont intègres ?

Pour vérifier leur intégrité, utilisez la cmdlet PowerShell `Get-AzApplicationGatewayBackendHealth` ou le portail. Pour plus d’informations, consultez [Diagnostics Application Gateway](application-gateway-diagnostics.md).

### <a name="whats-the-retention-policy-for-the-diagnostic-logs"></a>Quelle est la stratégie de rétention des journaux de diagnostic ?

Les journaux de diagnostic sont envoyés au compte de stockage du client. Les clients peuvent définir la stratégie de rétention en fonction de leurs préférences. Les journaux de diagnostic peuvent également être envoyés à un Event Hub ou à des journaux d’activité Azure Monitor. Pour plus d’informations, consultez [Diagnostics Application Gateway](application-gateway-diagnostics.md).

### <a name="how-do-i-get-audit-logs-for-application-gateway"></a>Comment puis-je obtenir des journaux d’audit pour Application Gateway ?

Dans le portail, dans le panneau du menu de la passerelle d’application, sélectionnez **Journal d’activité** pour accéder au journal d’audit. 

### <a name="can-i-set-alerts-with-application-gateway"></a>Puis-je définir des alertes avec Application Gateway ?

Oui. Dans Application Gateway, les alertes sont configurées sur les métriques. Pour plus d’informations, consultez [Métriques Application Gateway](./application-gateway-metrics.md) et [Recevoir des notifications d’alerte](../azure-monitor/platform/alerts-overview.md).

### <a name="how-do-i-analyze-traffic-statistics-for-application-gateway"></a>Comment analyser les statistiques de trafic pour Application Gateway ?

Vous pouvez afficher et analyser les journaux d’accès de plusieurs façons. Utilisez les journaux d'activité Azure Monitor, Excel, Power BI, etc.

Vous pouvez également utiliser un modèle Resource Manager qui installe et exécute le célèbre analyseur de journal d’activité [GoAccess](https://goaccess.io/) pour les journaux d’accès Application Gateway. GoAccess fournit des statistiques de trafic HTTP précieuses telles que les visiteurs uniques, les fichiers demandés, les hôtes, les systèmes d’exploitation, les navigateurs ou les codes d’état HTTP. Pour plus d’informations, consultez le [fichier Lisez-moi dans le dossier de modèles Resource Manager dans GitHub](https://aka.ms/appgwgoaccessreadme).

### <a name="what-could-cause-backend-health-to-return-an-unknown-status"></a>Dans quelles circonstances, l'intégrité d'un serveur principal peut-elle indiquer un état inconnu ?

En règle générale, vous constatez un état inconnu lorsque l'accès au serveur principal est bloqué par un groupe de sécurité réseau (NSG), un DNS personnalisé ou un routage défini par l'utilisateur (UDR) sur le sous-réseau d'Application Gateway. Pour plus d’informations, consultez [Intégrité du serveur principal, journalisation des diagnostics et métriques pour Application Gateway](application-gateway-diagnostics.md).

### <a name="are-nsg-flow-logs-supported-on-nsgs-associated-to-application-gateway-v2-subnet"></a>Les journaux de flux NSG sont-ils pris en charge sur les NSG associés au sous-réseau d’Application Gateway v2 ?

En raison des limitations actuelles de la plateforme, si vous avez un groupe de sécurité réseau sur le sous-réseau d’Application Gateway v2 (Standard_v2, WAF_v2) et si vous avez activé les journaux de flux NSG sur celui-ci, vous verrez un comportement non déterministe et ce scénario n’est actuellement pas pris en charge.

### <a name="does-application-gateway-store-customer-data"></a>Application Gateway stocke-t-il des données client ?

Non, Application Gateway ne stocke pas de données client.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Application Gateway, consultez [Qu’est-ce qu’Azure Application Gateway ?](overview.md)
