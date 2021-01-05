---
title: SNAT pour les connexions sortantes
description: Décrit comment utiliser Azure Load Balancer pour effectuer la SNAT pour la connectivité Internet sortante
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: conceptual
ms.custom: contperf-fy21q1
ms.date: 10/13/2020
ms.author: allensu
ms.openlocfilehash: d30998e0d99ba7b1eeb55bdc47cd5cc54690156f
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032913"
---
# <a name="using-snat-for-outbound-connections"></a>Utilisation de la SNAT pour les connexions sortantes

Les adresses IP front-end d’un équilibreur de charge public Azure peuvent être utilisées pour fournir une connectivité sortante à Internet pour des instances back-end. Cette configuration utilise une **SNAT (« Source Network Address Translation », ou « traduction d’adresses réseau sources »)** . La SNAT remplace l’adresse IP du back-end par l’adresse IP publique de votre équilibreur de charge. 

La SNAT permet le **masquage d’IP** de l’instance back-end. Ce masquage permet d’empêcher des sources externes d’avoir une adresse directe vers les instances back-end. Le partage d’une adresse IP entre des instances back-end réduit le coût des adresses IP publiques statiques et prend en charge des scénarios tels que la simplification des listes d’adresses IP autorisées avec un trafic provenant d’adresses IP publiques connues. 

>[!Note]
> Pour les applications qui demandent un grand nombre de connexions sortantes ou les clients d’entreprise qui nécessitent un seul ensemble d’IP à utiliser à partir d’un réseau virtuel donné, le service [NAT de réseau virtuel](../virtual-network/nat-overview.md) est la solution recommandée. Son allocation dynamique permet une configuration simple et l’utilisation la plus efficace des ports SNAT à partir de chaque adresse IP. Ce service permet aussi à toutes les ressources du réseau virtuel de partager un ensemble d’adresses IP sans avoir besoin de partager un équilibreur de charge.

>[!Important]
> Même sans SNAT sortante configurée, les comptes de stockage Azure de la même région restent accessibles et les ressources back-end ont toujours accès aux services Microsoft comme les mises à jour de Windows.

>[!NOTE] 
>Cet article traite des déploiements Azure Resource Manager uniquement. Consultez [Connexions sortantes dans Azure (Classic)](/previous-versions/azure/load-balancer/load-balancer-outbound-connections-classic) pour découvrir l’ensemble des scénarios de déploiement Classic dans Azure.

## <a name="sharing-frontend-ip-address-across-backend-resources"></a><a name ="snat"></a> Partage de l’adresse IP front-end entre les ressources back-end

Si les ressources back-end d’un équilibreur de charge n’ont pas d’adresses IP publiques au niveau de l’instance (ILPIP), elles établissent une connectivité sortante via l’adresse IP front-end de l’équilibreur de charge public. Les ports sont utilisés pour générer des identificateurs uniques, eux-mêmes utilisés pour gérer des flux distincts. Internet utilise cinq tuples pour fournir cette distinction.

Ces cinq tuples sont constitués des éléments suivants :

* Adresse IP de destination
* Port de destination
* IP Source
* Le port et le protocole source pour fournir cette distinction.

Si un port est utilisé pour des connexions entrantes, il aura un **écouteur** pour ses requêtes de connexions entrantes et ne pourra pas être utilisé pour les connexions sortantes. Pour établir une connexion sortante, vous devez utiliser un **port éphémère** pour fournir un port à la destination, afin de communiquer et de maintenir un flux de trafic distinct. Quand ces ports éphémères sont utilisés pour effectuer la SNAT, ils sont appelés **ports SNAT**. 

Par définition, chaque adresse IP a 65 535 ports. Chaque port peut être utilisé pour les connexions entrantes ou sortantes pour les protocoles TCP (Transmission Control Protocol) et UDP (User Datagram Protocol). Quand une IP publique est ajoutée en tant qu’IP front-end à un équilibreur de charge, Azure accorde 64 000 ports SNAT éligibles. 

>[!NOTE]
> Chaque port utilisé pour une règle NAT de trafic entrant ou d’équilibrage de charge utilise une plage de huit ports dans ces 64 000 ports, ce qui réduit le nombre de ports éligibles à la SNAT. Si une règle NAT ou d’équilibrage de charge se trouve dans la même plage de huit qu’une autre règle, elle ne consomme pas de ports supplémentaires. 

Avec les [règles de trafic sortant](./outbound-rules.md) et les règles d’équilibrage de charge, ces ports SNAT peuvent être distribués aux instances back-end pour leur permettre de partager les IP publiques de l’équilibreur de charge pour les connexions sortantes.

Quand vous configurez le [scénario 2](#scenario2) ci-dessous, l’hôte de chaque instance back-end effectue la SNAT sur des paquets qui font partie d’une connexion sortante. Quand l’hôte effectue la SNAT sur une connexion sortante à partir d’une instance back-end, il réécrit l’IP source sur l’une des IP front-end. Pour conserver des flux uniques, l’hôte réécrit le port source de chaque paquet sortant sur l’un des ports SNAT alloués à l’instance back-end.

## <a name="outbound-connection-behavior-for-different-scenarios"></a>Comportement de la connexion sortante dans différents scénarios
  * Machine virtuelle avec IP publique.
  * Machine virtuelle sans IP publique.
  * Machine virtuelle sans IP publique et sans équilibreur de charge standard.
        

 ### <a name="scenario-1-virtual-machine-with-public-ip"></a><a name="scenario1"></a> Scénario 1 : Machine virtuelle avec IP publique


 | Associations | Méthode | Protocoles IP |
 | ---------- | ------ | ------------ |
 | Équilibreur de charge public ou autonome | [SNAT (Source Network Address Translation, traduction d’adresses réseau sources)](#snat) </br> non utilisée | Protocole TCP (Transmission Control Protocol) </br> Protocole UDP (User Datagram Protocol) </br> Protocole ICMP (Internet Control Message Protocol) </br> Protection ESP (Encapsulating Security Payload) |


 #### <a name="description"></a>Description


 Azure utilise l’IP publique attribuée à la configuration IP de la carte d’interface réseau de l’instance pour tous les flux sortants. L’instance a tous les ports éphémères disponibles. Peu importe si la machine virtuelle est équilibrée en charge ou non. Ce scénario prend le pas sur les autres. 


 Une adresse IP publique affectée à une machine virtuelle constitue une relation 1:1 (non pas une relation 1-à-plusieurs) ; elle est implémentée comme un NAT 1:1 sans état.


 ### <a name="scenario-2-virtual-machine-without-public-ip-and-behind-standard-public-load-balancer"></a><a name="scenario2"></a>Scénario 2 : Machine virtuelle sans IP publique et derrière un équilibreur de charge public standard


 | Associations | Méthode | Protocoles IP |
 | ------------ | ------ | ------------ |
 | Équilibreur de charge public | Utilisation des IP front-end de l’équilibreur de charge pour la [SNAT](#snat).| TCP </br> UDP |


 #### <a name="description"></a>Description


 La ressource de l’équilibreur de charge est configurée avec une règle de trafic sortant ou une règle d’équilibrage de charge qui permet la SNAT par défaut. Cette règle est utilisée pour créer un lien entre le front-end de l’IP publique et le pool de back-ends. 


 Si vous n’effectuez pas cette configuration de règle, le comportement est celui décrit dans le scénario n 3. 


 Une règle avec écouteur n’est pas nécessaire pour que la sonde d’intégrité aboutisse.


 Quand une machine virtuelle crée un flux sortant, Azure convertit l’adresse IP source en adresse IP publique du front-end de l’équilibreur de charge public. Cette traduction s’effectue par le biais de la [SNAT](#snat). 


 Les ports éphémères de l’adresse IP publique du front-end de l’équilibreur de charge servent à faire la distinction entre les différents flux en provenance de la machine virtuelle. La traduction d’adresses réseau utilise dynamiquement des [ports éphémères préaffectés](#preallocatedports) lors de la création de flux sortants. 


 Dans ce contexte, les ports éphémères utilisés pour la traduction d’adresses réseau sources sont appelés ports SNAT. Il est fortement recommandé de configurer explicitement une [règle de trafic sortant](./outbound-rules.md). Si vous utilisez la SNAT par défaut par le biais d’une règle d’équilibrage de charge, les ports SNAT sont pré-alloués comme décrit dans le [tableau d’allocation des ports SNAT par défaut](#snatporttable).


 ### <a name="scenario-3-virtual-machine-without-public-ip-and-behind-basic-load-balancer"></a><a name="scenario3"></a>Scénario 3 : Machine virtuelle sans adresse IP publique et derrière un équilibreur de charge de base


 | Associations | Méthode | Protocoles IP |
 | ------------ | ------ | ------------ |
 |Aucun </br> Équilibreur de charge de base | [SNAT](#snat) avec adresse IP dynamique au niveau de l’instance| TCP </br> UDP | 

 #### <a name="description"></a>Description


 Quand la machine virtuelle crée un flux sortant, Azure convertit l’adresse IP source en adresse IP source publique allouée dynamiquement. Cette adresse IP publique **n’est pas configurable** et ne peut pas être réservée. Cette adresse n’est pas comptabilisée dans la limite des ressources IP publiques de l’abonnement. 


 L’adresse IP publique est publiée et une nouvelle adresse IP publique est demandée si vous redéployez ce qui suit : 


 * Machine virtuelle
 * Groupe à haute disponibilité
 * Jeu de mise à l’échelle de machine virtuelle 


 N’utilisez pas ce scénario pour ajouter des IP à une liste d’autorisation. Utilisez le scénario 1 ou 2 dans lequel vous déclarez explicitement le comportement du trafic sortant. Les ports [SNAT](#snat) sont préalloués, comme décrit dans le [tableau d’allocation des ports SNAT par défaut](#snatporttable).


## <a name="exhausting-ports"></a><a name="scenarios"></a> Épuisement des ports

Chaque connexion à une même adresse IP de destination et au même port de destination utilisera un port SNAT. Cette connexion gère un **flux de trafic** distinct depuis l’instance ou le **client** back-end vers un **serveur**. Ce processus fournit au serveur un port distinct vers lequel envoyer le trafic. Sans ce processus, l’ordinateur client ignore dans quel flux trouver tel ou tel paquet.

Imaginez que plusieurs navigateurs vont vers https://www.microsoft.com, à savoir :

* IP de destination = 23.53.254.142
* Port de destination = 443
* Protocole = TCP

Sans ports de destination différents pour le trafic de retour (le port SNAT utilisé pour établir la connexion), le client n’a aucun moyen de séparer le résultat d’une requête d’un autre.

Le nombre de connexions sortantes peut augmenter rapidement. L’instance back-end risque de se retrouver en manque de ports. Si la **réutilisation des connexions** n’est pas activée, le risque d’**épuisement des ports** SNAT augmente.

Un épuisement des ports provoque l’échec des nouvelles connexions sortantes vers une adresse IP de destination. Les connexions sont correctement établies lorsqu’un port est disponible. Cet épuisement se produit lorsque les 64 000 ports à partir d’une adresse IP sont répartis sur de nombreuses instances back-end. Pour obtenir des conseils sur l’atténuation de l’épuisement des ports SNAT, consultez ce [guide de résolution des problèmes](./troubleshoot-outbound-connection.md).  

Pour les connexions TCP, l’équilibreur de charge utilise un seul port SNAT pour chaque adresse IP et port de destination. Ce multiusage permet plusieurs connexions à la même adresse IP de destination avec le même port SNAT. Ce multiusage est limité si la connexion ne dirige pas vers différents ports de destination.

Pour les connexions UDP, l’équilibreur de charge utilise un algorithme nommé « **port-restricted cone NAT** » (ou « NAT à cône restrictif sur les ports »), qui consomme un port SNAT par adresse IP de destination, quel que soit le port de destination. 

Cela permet de réutiliser un port pour un nombre illimité de connexions. Le port est réutilisé uniquement si l’adresse IP ou le port de destination est différent.

## <a name="default-port-allocation"></a><a name="preallocatedports"></a> Allocation des ports par défaut

Chaque adresse IP publique allouée en tant qu’adresse IP front-end de votre équilibreur de charge reçoit 64 000 ports SNAT pour ses membres de pool back-end. Les ports ne peuvent pas être partagés avec des membres du pool back-end. Une plage de ports SNAT ne peut être utilisée que par une seule instance back-end pour garantir le routage correct des paquets de retour. 

Nous vous recommandons d’utiliser une règle de trafic sortant explicite pour configurer l’allocation de port SNAT. Cette règle permet de maximiser le nombre de ports SNAT dont dispose chaque instance de back-end pour les connexions sortantes. 

Si vous utilisez l’allocation automatique de port SNAT sortant par le biais d’une règle d’équilibrage de charge, la table d’allocation définira votre allocation de port.

Le tableau suivant <a name="snatporttable"></a>présente les préaffectations de ports SNAT pour les différents niveaux de tailles de pool back-end :

| Taille du pool (instances de machine virtuelle) | Ports de traduction d’adresses réseau sources préaffectés par configuration IP |
| --- | --- |
| 1-50 | 1 024 |
| 51-100 | 512 |
| 101-200 | 256 |
| 201-400 | 128 |
| 401-800 | 64 |
| 801-1 000 | 32 | 

>[!NOTE]
> Si la taille maximale de votre pool back-end est de 10, chaque instance peut avoir 64 000 / 10 = 6 400 ports si vous définissez une règle de trafic sortant explicite. Selon le tableau ci-dessus, chaque instance n’aura que 1 024 ports si vous choisissez l’allocation automatique.

## <a name="outbound-rules-and-virtual-network-nat"></a><a name="outboundrules"></a> Règles de trafic sortant et NAT de réseau virtuel

Les règles de trafic sortant de l’équilibreur de charge Azure et le service NAT de réseau virtuel sont des options disponibles pour la sortie à partir d’un réseau virtuel.

Pour plus d’informations sur les règles de trafic sortant, consultez [Règles de trafic sortant](outbound-rules.md).

Pour plus d’informations sur le service NAT de réseau virtuel Azure, consultez [Qu’est-ce que le service NAT de réseau virtuel ?](../virtual-network/nat-overview.md)

## <a name="constraints"></a>Contraintes

*   Lorsqu’une connexion est inactive et qu’aucun nouveau paquet n’est envoyé, les ports sont libérés après un délai allant de 4 à 120 minutes.
  * Ce délai peut être configuré à l’aide des règles de trafic sortant.
*   Chaque adresse IP fournit 64 000 ports qui peuvent être utilisés pour la SNAT.
*   Chaque port peut être utilisé pour les connexions TCP et UDP à une adresse IP de destination
  * Un port UDP SNAT est nécessaire si le port de destination est unique ou non. Pour chaque connexion UDP à une adresse IP de destination, un port UDP SNAT est utilisé.
  * Un port TCP SNAT peut être utilisé pour plusieurs connexions à la même adresse IP de destination, à condition que les ports de destination soient différents.
*   L’épuisement des ports SNAT se produit lorsqu’une instance back-end n’a pas assez de ports SNAT. Un équilibreur de charge peut toujours avoir des ports SNAT inutilisés. Si une instance back-end utilise plus de ports SNAT que ceux qui lui sont alloués, elle ne pourra pas établir de nouvelles connexions sortantes.

## <a name="next-steps"></a>Étapes suivantes

*   [Résoudre les échecs de connexion sortante provoqués par l’épuisement des ports SNAT](./troubleshoot-outbound-connection.md)
*   [Passez en revue les métriques SNAT](./load-balancer-standard-diagnostics.md#how-do-i-check-my-snat-port-usage-and-allocation) et familiarisez-vous avec la méthode appropriée pour les filtrer, les fractionner et les afficher.
