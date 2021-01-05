---
title: 'Azure ExpressRoute : Configuration requise du routage'
description: Cette page détaille les conditions nécessaires à la configuration et à la gestion du routage pour les circuits ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/19/2019
ms.author: duau
ms.openlocfilehash: 436e866969d620389818bcebca3c5c37b8805309
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97629032"
---
# <a name="expressroute-routing-requirements"></a>Configuration requise pour le routage ExpressRoute
Pour vous connecter aux services de cloud Microsoft à l’aide d’ExpressRoute, vous devez configurer et gérer le routage. Certains fournisseurs de connectivité proposent la configuration et la gestion du routage comme un service géré. Vérifiez auprès de votre fournisseur de connectivité s’il offre ce service. Si ce n’est pas le cas, vous devez respecter les conditions suivantes :

Pour obtenir une description des sessions de routage qui doivent être configurées afin d’établir la connectivité, reportez-vous à l’article [Circuits et domaines de routage](expressroute-circuit-peerings.md).

> [!NOTE]
> Microsoft ne prend pas en charge les protocoles de redondance de routeur (HSRP, VRRP, par exemple) pour les configurations à haute disponibilité. Nous nous appuyons sur une paire de sessions BGP par peering pour la haute disponibilité.
> 
> 

## <a name="ip-addresses-used-for-peerings"></a>Adresses IP utilisées pour les peerings
Vous devez réserver quelques blocs d’adresses IP pour configurer le routage entre votre réseau et les routeurs Microsoft Enterprise Edge (MSEE). Cette section fournit une liste des conditions requises et décrit les règles relatives à la façon dont ces adresses IP doivent être acquises et utilisées.

### <a name="ip-addresses-used-for-azure-private-peering"></a>Adresses IP utilisées pour le peering privé Azure
Pour configurer les peerings, vous pouvez utiliser des adresses IP privées ou publiques. La plage d’adresses utilisée pour configurer des routages ne doit pas chevaucher les plages d’adresses utilisées pour créer des réseaux virtuels dans Azure. 

* Vous devez réserver un sous-réseau /29 ou deux sous-réseaux /30 pour les interfaces de routage.
* Les sous-réseaux utilisés pour le routage peuvent être des adresses IP privées ou publiques.
* Les sous-réseaux ne doivent pas entrer en conflit avec la plage réservée par le client pour une utilisation dans le cloud Microsoft.
* Si un sous-réseau /29 est utilisé, il est subdivisé en deux sous-réseaux /30. 
  * Le premier sous-réseau /30 est utilisé pour le lien principal, et le second sous-réseau /30 est utilisé pour le lien secondaire.
  * Pour chacun des sous-réseaux /30, vous devez utiliser la première adresse IP du sous-réseau /30 sur votre routeur. Microsoft utilise la deuxième adresse IP du sous-réseau /30 pour configurer une session BGP.
  * Vous devez configurer les deux sessions BGP pour que notre [contrat SLA de disponibilité](https://azure.microsoft.com/support/legal/sla/) soit valide.  

#### <a name="example-for-private-peering"></a>Exemple pour le peering privé
Si vous choisissez d’utiliser un sous-réseau a.b.c.d/29 pour configurer le peering, il est scindé en deux sous-réseaux /30. Dans l’exemple suivant, notez le mode d’utilisation du sous-réseau a.b.c.d/29 :

* a.b.c.d/29 est scindé en a.b.c.d/30 et a.b.c.d+4/30 puis transmis à Microsoft via les API d’approvisionnement.
  * Vous utilisez a.b.c.d+1 comme l’IP VRF pour le PE principal et Microsoft utilisera a.b.c.d+2 comme IP VRF pour le MSEE principal.
  * Vous utilisez a.b.c.d+5 comme l’IP VRF pour le PE secondaire et Microsoft utilisera a.b.c.d+6 IP VRF pour le MSEE secondaire.

Imaginons que vous sélectionnez 192.168.100.128/29 pour configurer le peering privé. 192.168.100.128/29 inclut les adresses de 192.168.100.128 à 192.168.100.135, parmi lesquelles :

* 192.168.100.128/30 sera attribuée à link1, et le fournisseur utilisera 192.168.100.129 tandis que Microsoft utilisera 192.168.100.130.
* 192.168.100.132/30 sera attribuée à link2, et le fournisseur utilisera 192.168.100.133 tandis que Microsoft utilisera 192.168.100.134.

### <a name="ip-addresses-used-for-microsoft-peering"></a>Adresses IP utilisées pour le peering Microsoft
Vous devez utiliser des adresses IP publiques que vous possédez pour configurer les sessions BGP. Microsoft doit être en mesure de vérifier la propriété des adresses IP via les Registres Internet de routage régional (RIR) et les Registres Internet de routage (IRR).

* Les adresses IP répertoriées dans le portail pour les préfixes publics publiés pour le peering Microsoft créent les listes de contrôle d’accès pour les routeurs principaux Microsoft pour autoriser le trafic entrant à partir de ces adresses IP. 
* Vous devez utiliser un sous-réseau unique /29 (IPv4) ou /125 (IPv6) ou bien deux sous-réseaux /30 (IPv4) ou /126 (IPv6) afin de configurer le peering BGP pour chaque peering par circuit ExpressRoute (si vous en avez plusieurs).
* Si un sous-réseau /29 est utilisé, il est subdivisé en deux sous-réseaux /30.
* Le premier sous-réseau /30 est utilisé pour le lien principal, et le second sous-réseau /30 sera utilisé pour le lien secondaire.
* Pour chacun des sous-réseaux /30, vous devez utiliser la première adresse IP du sous-réseau /30 sur votre routeur. Microsoft utilise la deuxième adresse IP du sous-réseau /30 pour configurer une session BGP.
* Si un sous-réseau /125 est utilisé, il est subdivisé en deux sous-réseaux /126.
* Le premier sous-réseau /126 est utilisé pour le lien principal, et le second sous-réseau /126 sera utilisé pour le lien secondaire.
* Pour chacun des sous-réseaux /126, vous devez utiliser la première adresse IP du sous-réseau /126 sur votre routeur. Microsoft utilise la deuxième adresse IP du sous-réseau /126 pour configurer une session BGP.
* Vous devez configurer les deux sessions BGP pour que notre [contrat SLA de disponibilité](https://azure.microsoft.com/support/legal/sla/) soit valide.

### <a name="ip-addresses-used-for-azure-public-peering"></a>Adresses IP utilisées pour le peering public Azure

> [!NOTE]
> Le peering public Azure n’est pas disponible pour les nouveaux circuits.
> 

Vous devez utiliser des adresses IP publiques que vous possédez pour configurer les sessions BGP. Microsoft doit être en mesure de vérifier la propriété des adresses IP via les Registres Internet de routage régional (RIR) et les Registres Internet de routage (IRR). 

* Vous devez utiliser un sous-réseau unique /29 ou deux sous-réseaux /30 afin de configurer le peering BGP pour chaque peering par circuit ExpressRoute (si vous en avez plusieurs). 
* Si un sous-réseau /29 est utilisé, il est subdivisé en deux sous-réseaux /30. 
  * Le premier sous-réseau /30 est utilisé pour le lien principal, et le second sous-réseau /30 est utilisé pour le lien secondaire.
  * Pour chacun des sous-réseaux /30, vous devez utiliser la première adresse IP du sous-réseau /30 sur votre routeur. Microsoft utilise la deuxième adresse IP du sous-réseau /30 pour configurer une session BGP.
  * Vous devez configurer les deux sessions BGP pour que notre [contrat SLA de disponibilité](https://azure.microsoft.com/support/legal/sla/) soit valide.

## <a name="public-ip-address-requirement"></a>Spécification d’adresse IP publique

### <a name="private-peering"></a>Peering privé
Vous pouvez choisir d’utiliser des adresses IPv4 publiques ou privées pour le peering privé. Nous fournissons une isolation de bout en bout du trafic. Ainsi, le chevauchement des adresses avec d’autres clients n’est pas possible dans le cas d’un peering privé. Ces adresses ne sont pas publiées sur Internet. 

### <a name="microsoft-peering"></a>Peering Microsoft
Le chemin d’accès de peering Microsoft vous permet de vous connecter aux services de cloud computing Microsoft. La liste des services comprend les services Microsoft 365, notamment Exchange Online, SharePoint Online, Skype Entreprise et Microsoft Teams. Microsoft prend en charge la connectivité bidirectionnelle sur le peering Microsoft. Le trafic destiné aux services de cloud Microsoft doit utiliser des adresses IPv4 publiques valides avant leur entrée sur le réseau Microsoft.

Assurez-vous que votre adresse IP et votre numéro AS sont enregistrés à votre nom dans l’un des registres ci-dessous :

* [ARIN](https://www.arin.net/)
* [APNIC](https://www.apnic.net/)
* [AFRINIC](https://www.afrinic.net/)
* [LACNIC](https://www.lacnic.net/)
* [RIPENCC](https://www.ripe.net/)
* [RADB](https://www.radb.net/)
* [ALTDB](https://altdb.net/)

Si vos préfixes et le numéro ASN ne vous sont pas affectés dans les registres précédents, vous devez ouvrir un dossier de support pour valider manuellement vos préfixes et votre numéro ASN. La prise en charge requiert un document, tel qu’une lettre d’autorisation, qui prouve que vous êtes autorisé à utiliser les ressources.

Un numéro ASN privé est autorisé avec le peering Microsoft, mais nécessite également une validation manuelle. Nous supprimons également les numéros AS privés dans le chemin AS PATH pour les préfixes reçus. En conséquence, vous ne pouvez pas ajouter de numéros AS privés au chemin AS PATH pour [influencer le routage pour le peering Microsoft](expressroute-optimize-routing.md). 

> [!IMPORTANT]
> Ne publiez pas le même itinéraire d’adresse IP publique sur l’Internet public et sur ExpressRoute. Pour réduire tout risque de configuration incorrecte pouvant entraîner un routage asymétrique, nous recommandons vivement que les[adresses IP NAT](expressroute-nat.md) transmises à Microsoft via ExpressRoute proviennent d’une plage qui n’est pas du tout publiée sur Internet. Si ce n’est pas possible, il est essentiel de vous assurer d’utiliser une plage plus spécifique via ExpressRoute que celle sur la connexion Internet. En plus de la route publique pour NAT, vous pouvez publier sur ExpressRoute les adresses IP publiques utilisées par les serveurs de votre réseau local, qui communiquent avec des points de terminaison Microsoft 365 dans Microsoft. 
> 
> 

### <a name="public-peering-deprecated---not-available-for-new-circuits"></a>Peering public (déprécié : non disponible pour de nouveaux circuits)
Le chemin de peering public Azure vous permet de vous connecter à tous les services hébergés dans Azure en utilisant leurs adresses IP publiques. Cela inclut les services répertoriés dans le [FAQ sur ExpressRoute](expressroute-faqs.md) et tous les services hébergés par les éditeurs de logiciels sur Microsoft Azure. La connectivité aux services Microsoft Azure sur le peering public est toujours lancée de votre réseau vers le réseau Microsoft. Vous devez utiliser des adresses IP publiques pour le trafic destiné au réseau Microsoft.

> [!IMPORTANT]
> Tous les services Azure PaaS sont accessibles via le peering Microsoft.
>   

Un numéro AS privé est autorisé avec le peering public.

## <a name="dynamic-route-exchange"></a>Échange de routage dynamique
L’échange de routage s’effectuera via le protocole eBGP. Des sessions EBGP sont établies entre les MSEE et les routeurs. L’authentification des sessions BGP n’est pas obligatoire. Si nécessaire, un hachage MD5 peut être configuré. Pour plus d’informations sur la configuration des sessions BGP, consultez [Configuration du routage](how-to-routefilter-portal.md) et [Workflows d’approvisionnement du circuit et états du circuit](expressroute-workflows.md).

## <a name="autonomous-system-numbers"></a>Numéros système autonomes
Microsoft utilise le numéro AS 12076 pour les peerings publics Azure, privés Azure et Microsoft. Nous avons réservé les numéros AS 65515 à 65520 pour un usage interne. Les numéros AS 16 bits et 32 bits sont pris en charge.

Il n’existe aucune exigence concernant une symétrie de transfert des données. Les chemins d’envoi et de réception peuvent transiter par différentes paires de routeurs. Les routages identiques doivent être publiés des deux côtés sur plusieurs paires de circuits vous appartenant. Les métriques de routage n’ont pas besoin d’être identiques.

## <a name="route-aggregation-and-prefix-limits"></a>Agrégation de routages et limites de préfixes
Nous prenons en charge jusqu’à 4 000 préfixes qui nous sont proposés via le peering privé Azure. Ce chiffre peut être augmenté jusqu’à 10 000 préfixes si le module complémentaire ExpressRoute premium est activé. Nous acceptons jusqu’à 200 préfixes par session BGP pour les peerings publics Azure et Microsoft. 

La session BGP s’arrête si le nombre de préfixes dépasse la limite. Nous accepterons les routages par défaut uniquement sur le lien de peering privé. Le fournisseur doit filtrer les adresses IP de routage et privées par défaut (RFC 1918) des chemins de peering public Azure et Microsoft. 

## <a name="transit-routing-and-cross-region-routing"></a>Routage de transit et routage entre régions
ExpressRoute ne peut pas être configuré comme des routeurs de transit. Vous devez vous appuyer sur votre fournisseur de connectivité pour les services de routage de transit.

## <a name="advertising-default-routes"></a>Publication des routages par défaut
Les routages par défaut sont autorisés uniquement sur les sessions de peering privé Azure. Dans ce cas, nous acheminerons tout le trafic des réseaux virtuels associés vers votre réseau. La publication de routages par défaut dans le peering privé entraîne le blocage du chemin Internet à partir d’Azure. Vous devez compter sur votre matériel edge d’entreprise afin d’acheminer le trafic depuis et vers Internet pour les services hébergés dans Azure. 

 Pour activer la connectivité avec d’autres services Azure et services d’infrastructure, vous devez vous assurer qu’un des éléments suivants est en place :

* Le peering public Azure est activée pour acheminer le trafic vers les points de terminaison publics.
* Vous utilisez le routage défini par l’utilisateur pour permettre la connectivité Internet pour chaque sous-réseau nécessitant une connectivité Internet définie par l’utilisateur.

> [!NOTE]
> La publication des routages par défaut arrête Windows et toute autre activation de licence de machine virtuelle. Suivez les instructions [ici](/archive/blogs/mast/use-azure-custom-routes-to-enable-kms-activation-with-forced-tunneling) pour contourner ce problème.
> 
> 

## <a name="support-for-bgp-communities"></a><a name="bgp"></a>Prise en charge des communautés BGP
Cette section fournit une vue d'ensemble de l'utilisation des communautés BGP avec ExpressRoute. Microsoft publiera des routages sur les chemins de peering publics et Microsoft avec des routages marqués à l’aide des valeurs de communauté appropriées. La logique de cette procédure et les détails concernant les valeurs de la communauté sont décrits ci-dessous. Cependant, Microsoft ignorera toutes les valeurs de communauté marquées pour des itinéraires qui lui sont proposés.

Si vous vous connectez à Microsoft via ExpressRoute dans n’importe quel emplacement de peering d’une région géopolitique, vous aurez accès à tous les services de cloud Microsoft de toutes les régions situées dans les limites géopolitiques. 

Par exemple, si vous êtes connecté à Microsoft à Amsterdam via ExpressRoute, vous aurez accès à tous les services de cloud Microsoft hébergés dans les régions Europe Nord et Europe Ouest. 

Reportez-vous à la page [Partenaires ExpressRoute et emplacements de peering](expressroute-locations.md) pour obtenir une liste détaillée des régions géopolitiques, des régions Azure associées et des emplacements de peering ExpressRoute correspondants.

Vous pouvez acheter plusieurs circuits ExpressRoute par région géopolitique. Le fait de disposer de plusieurs connexions vous offre des avantages significatifs en termes de haute disponibilité en raison de la redondance géographique. Si vous avez plusieurs circuits ExpressRoute, vous recevrez le même jeu de préfixes publiés par Microsoft sur les chemins de peering publics et Microsoft. Cela signifie que vous disposez de plusieurs chemins de votre réseau vers Microsoft. Vous risquez ainsi de prendre des décisions de routage non optimales au sein de votre réseau. Et par conséquent, vous risquez de rencontrer des problèmes de connectivité non optimale avec différents services. Vous pouvez compter sur les valeurs fournies par la communauté pour prendre les bonnes décisions en matière de routage et offrir un [routage optimal aux utilisateurs](expressroute-optimize-routing.md).

| **Région Microsoft Azure** | **Communauté BGP régionale** | **Communauté BGP de stockage** | **Communauté BGP SQL** | **Communauté BGP Cosmos DB** | **Communauté BGP de sauvegarde** |
| --- | --- | --- | --- | --- | --- |
| **Amérique du Nord** | |
| USA Est | 12076:51004 | 12076:52004 | 12076:53004 | 12076:54004 | 12076:55004 |
| USA Est 2 | 12076:51005 | 12076:52005 | 12076:53005 | 12076:54005 | 12076:55005 |
| USA Ouest | 12076:51006 | 12076:52006 | 12076:53006 | 12076:54006 | 12076:55006 |
| USA Ouest 2 | 12076:51026 | 12076:52026 | 12076:53026 | 12076:54026 | 12076:55026 |
| Centre-USA Ouest | 12076:51027 | 12076:52027 | 12076:53027 | 12076:54027 | 12076:55027 |
| Centre-Nord des États-Unis | 12076:51007 | 12076:52007 | 12076:53007 | 12076:54007 | 12076:55007 |
| États-Unis - partie centrale méridionale | 12076:51008 | 12076:52008 | 12076:53008 | 12076:54008 | 12076:55008 |
| USA Centre | 12076:51009 | 12076:52009 | 12076:53009 | 12076:54009 | 12076:55009 |
| Centre du Canada | 12076:51020 | 12076:52020 | 12076:53020 | 12076:54020 | 12076:55020 |
| Est du Canada | 12076:51021 | 12076:52021 | 12076:53021 | 12076:54021 | 12076:55021 |
| **Amérique du Sud** | |
| Brésil Sud | 12076:51014 | 12076:52014 | 12076:53014 | 12076:54014 | 12076:55014 |
| **Europe** | |
| Europe Nord | 12076:51003 | 12076:52003 | 12076:53003 | 12076:54003 | 12076:55003 |
| Europe Ouest | 12076:51002 | 12076:52002 | 12076:53002 | 12076:54002 | 12076:55002 |
| Sud du Royaume-Uni | 12076:51024 | 12076:52024 | 12076:53024 | 12076:54024 | 12076:55024 |
| Ouest du Royaume-Uni | 12076:51025 | 12076:52025 | 12076:53025 | 12076:54025 | 12076:55025 |
| France Centre | 12076:51030 | 12076:52030 | 12076:53030 | 12076:54030 | 12076:55030 |
| France Sud | 12076:51031 | 12076:52031 | 12076:53031 | 12076:54031 | 12076:55031 |
| Suisse Nord | 12076:51038 | 12076:52038 | 12076:53038 | 12076:54038 | 12076:55038 |
| Suisse Ouest | 12076:51039 | 12076:52039 | 12076:53039 | 12076:54039 | 12076:55039 | 
| Allemagne Nord | 12076:51040 | 12076:52040 | 12076:53040 | 12076:54040 | 12076:55040 | 
| Allemagne Centre-Ouest | 12076:51041 | 12076:52041 | 12076:53041 | 12076:54041 | 12076:55041 | 
| Norvège Est | 12076:51042 | 12076:52042 | 12076:53042 | 12076:54042 | 12076:55042 | 
| Norvège Ouest | 12076:51043 | 12076:52043 | 12076:53043 | 12076:54043 | 12076:55043 | 
| **Asie-Pacifique** | |
| Asie Est | 12076:51010 | 12076:52010 | 12076:53010 | 12076:54010 | 12076:55010 |
| Asie Sud-Est | 12076:51011 | 12076:52011 | 12076:53011 | 12076:54011 | 12076:55011 |
| **Japon** | |
| Japon Est | 12076:51012 | 12076:52012 | 12076:53012 | 12076:54012 | 12076:55012 |
| OuJapon Est | 12076:51013 | 12076:52013 | 12076:53013 | 12076:54013 | 12076:55013 |
| **Australie** | |
| Australie Est | 12076:51015 | 12076:52015 | 12076:53015 | 12076:54015 | 12076:55015 |
| Sud-Australie Est | 12076:51016 | 12076:52016 | 12076:53016 | 12076:54016 | 12076:55016 |
| **Secteur public australien** | |
| Centre de l’Australie | 12076:51032 | 12076:52032 | 12076:53032 | 12076:54032 | 12076:55032 |
| Centre de l’Australie 2 | 12076:51033 | 12076:52033 | 12076:53033 | 12076:54033 | 12076:55033 |
| **Inde** | |
| Sud de l’Inde | 12076:51019 | 12076:52019 | 12076:53019 | 12076:54019 | 12076:55019 |
| Inde Ouest | 12076:51018 | 12076:52018 | 12076:53018 | 12076:54018 | 12076:55018 |
| Inde Centre | 12076:51017 | 12076:52017 | 12076:53017 | 12076:54017 | 12076:55017 |
| **Corée** | |
| Corée du Sud | 12076:51028 | 12076:52028 | 12076:53028 | 12076:54028 | 12076:55028 |
| Centre de la Corée | 12076:51029 | 12076:52029 | 12076:53029 | 12076:54029 | 12076:55029 |
| **Afrique du Sud**| |
| Afrique du Sud Nord | 12076:51034 | 12076:52034 | 12076:53034 | 12076:54034 | 12076:55034 |
| Afrique du Sud Ouest | 12076:51035 | 12076:52035 | 12076:53035 | 12076:54035 | 12076:55035 |
| **Émirats Arabes Unis**| |
| Émirats arabes unis Nord | 12076:51036 | 12076:52036 | 12076:53036 | 12076:54036 | 12076:55036 |
| Émirats arabes unis Centre | 12076:51037 | 12076:52037 | 12076:53037 | 12076:54037 | 12076:55037 |


Tous les routages publiés par Microsoft seront marqués avec la valeur de communauté appropriée. 

> [!IMPORTANT]
> Les préfixes globaux sont marqués avec une valeur de communauté appropriée.
> 
> 

### <a name="service-to-bgp-community-value"></a>Service associé à la valeur de communauté BGP
Par ailleurs, Microsoft marquera également des préfixes basés sur le service auquel ils appartiennent. Cela s'applique uniquement au peering Microsoft. Le tableau ci-dessous fournit un mappage d’un service à la valeur de communauté BGP. Vous pouvez exécuter l’applet de commande « Get-AzBgpServiceCommunity » pour obtenir la liste complète des valeurs les plus récentes.

| **Service** | **Valeur de communauté BGP** |
| --- | --- |
| Exchange Online\*\* | 12076:5010 |
| SharePoint Online\*\* | 12076:5020 |
| Skype Entreprise Online\*\*/\*\*\* | 12076:5030 |
| CRM Online\*\*\*\* |12076:5040 |
| Services globaux Azure\* | 12076:5050 |
| Azure Active Directory |12076:5060 |
| Azure Resource Manager |12076:5070 |
| Autres services Office 365 Online** | 12076:5100 |

\* Pour le moment, les services globaux Azure incluent uniquement Azure DevOps.

\*\* Autorisation requise par Microsoft. Consultez [Configurer des filtres de routage pour l’homologation Microsoft](how-to-routefilter-portal.md)

\*\*\* Cette communauté publie également les itinéraires nécessaires pour les services Microsoft Team.

\*\*\*\* CRM Online prend en charge Dynamics v8.2 et versions antérieures. Pour les versions ultérieures, sélectionnez la communauté régionale pour vos déploiements Dynamics.

> [!NOTE]
> Microsoft ignore les valeurs de communauté BGP définies sur les itinéraires proposés à Microsoft.
> 
> 

### <a name="bgp-community-support-in-national-clouds"></a>Support de la communauté BGP dans les clouds nationaux

| **Région Azure pour les clouds nationaux**| **Valeur de communauté BGP** |
| --- | --- |
| **Gouvernement américain** |  |
| Gouvernement des États-Unis – Arizona | 12076:51106 |
| US Gov Iowa | 12076:51109 |
| Gouvernement américain - Virginie | 12076:51105 |
| Gouvernement des États-Unis – Texas | 12076:51108 |
| Centre des États-Unis – US DoD | 12076:51209 |
| Est des États-Unis – US DoD | 12076:51205 |


| **Service dans les clouds nationaux** | **Valeur de communauté BGP** |
| --- | --- |
| **Gouvernement américain** |  |
| Exchange Online |12076:5110 |
| SharePoint Online |12076:5120 |
| Skype Entreprise Online |12076:5130 |
| Azure Active Directory |12076:5160 |
| Autres services Office 365 en ligne |12076:5200 |

## <a name="next-steps"></a>Étapes suivantes
* Configurez votre connexion ExpressRoute.
  
  * [Créer et modifier un circuit](expressroute-howto-circuit-arm.md)
  * [Créer et modifier une configuration de peering](expressroute-howto-routing-arm.md)
  * [Liaison d’un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md)
