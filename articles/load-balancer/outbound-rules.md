---
title: Règles de trafic sortant Azure Load Balancer
description: Cet article explique comment configurer des règles de trafic sortant pour contrôler la sortie du trafic Internet avec Azure Load Balancer.
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: conceptual
ms.custom: contperf-fy21q1
ms.date: 10/13/2020
ms.author: allensu
ms.openlocfilehash: 6b73eb51831238f23400ef60d0a6162bca38ea85
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033151"
---
# <a name="outbound-rules-azure-load-balancer"></a><a name="outboundrules"></a>Règles de trafic sortant Azure Load Balancer

Les règles de trafic sortant vous permettent de définir explicitement la traduction d’adresses réseau sources (source network address translation, SNAT) pour un équilibreur de charge standard public. Cette configuration vous permet d’utiliser les adresses IP publiques de votre équilibreur de charge pour fournir une connectivité Internet sortante pour vos instances de serveur principal.

Cette configuration permet :

* L’usurpation d’adresse IP
* La simplification de vos listes d’autorisation.
* La réduction du nombre de ressources IP publiques pour le déploiement.

Avec les règles de trafic sortant, vous disposez d’un contrôle déclaratif complet sur la connectivité Internet sortante. Les règles de trafic sortant vous permettent de mettre à l’échelle et de régler cette capacité en fonction de vos besoins spécifiques. 

Les règles de trafic sortant sont uniquement suivies si la machine virtuelle principale n’a pas d’adresse IP publique au niveau de l’instance (ILPIP).

![Règles de trafic sortant dans Load Balancer](media/load-balancer-outbound-rules-overview/load-balancer-outbound-rules.png)

Avec les règle de trafic sortant, vous pouvez définir explicitement le comportement **SNAT** sortant.

Les règles de trafic sortant vous permettent de déterminer :

* **Quelles machines virtuelles doivent être traduites vers quelles adresses IP publiques.**
     * Deux règles : le pool principal A utilise l’adresse IP A et B, le pool principal B utilise l’adresse IP C et D.
* **Comment les ports SNAT de trafic sortant sont alloués.**
     * Le pool principal B est le seul pool effectuant des connexions sortantes, attribuez au pool principal B tous les ports SNAT et aucun au pool principal A.
* **Les protocoles qui fournissent la traduction sortante.**
     * Le pool principal B a besoin de ports UDP pour le trafic sortant. Le pool principal A a besoin de TCP. Donnez les ports TCP à A et les ports UDP à B.
* **La durée à utiliser comme délai d’inactivité des connexions sortantes ( 4 à 120 minutes).**
     * S’il existe des connexions de longue durée avec les KeepAlive, réservez les ports inactifs pour les connexions de longue durée pendant jusqu’à 120 minutes. Supposer que les connexions obsolètes sont abandonnées et les mettre en production les ports en 4 minutes pour les nouvelles connexions 
* **La réinitialisation TCP au terme du délai d’inactivité.**
     * Lors de l’expiration des connexions inactives, est-ce que nous envoyons un RST TCP au client et au serveur afin qu’ils sachent que le flux est abandonné ?

## <a name="outbound-rule-definition"></a>Définition de règle de trafic sortant

Les règles de trafic sortant suivent la syntaxe habituelle des règles d’équilibrage de charge et des règles NAT de trafic entrant : **serveur frontal** + **paramètres** + **pool principal**. 

Une règle de trafic sortant configure la NAT de trafic sortant pour que _toutes les machines virtuelles identifiées par le pool backend_ soient traduites sur le _frontend_.  

Les _paramètres_ permettent de contrôler avec plus de précision l’algorithme de la NAT de trafic sortant.

## <a name="scale-outbound-nat-with-multiple-ip-addresses"></a><a name="scale"></a> Mettre à l’échelle la NAT de trafic sortant avec plusieurs adresses IP

Chaque adresse IP supplémentaire fournie par un serveur frontal met à disposition 64 000 ports éphémères que l’équilibreur de charge peut utiliser en tant que ports SNAT. 

Utilisez plusieurs adresses IP pour planifier des scénarios à grande échelle. Utilisez des règles de trafic sortant pour atténuer [l’épuisement SNAT](troubleshoot-outbound-connection.md#snatexhaust). 

Vous pouvez utiliser un [préfixe d’IP publique](./load-balancer-outbound-connections.md#outboundrules) directement avec une règle de trafic sortant. 

Un préfixe d’adresse IP publique augmente la mise à l’échelle de votre déploiement. Le préfixe peut être ajouté à la liste d’autorisation des flux provenant de vos ressources Azure. Vous pouvez configurer une configuration IP front-end au sein de l’équilibreur de charge pour référencer un préfixe d’adresse IP publique.  

L’équilibreur de charge contrôle le préfixe d’adresse IP publique. La règle de trafic sortant utilise automatiquement toutes les adresses IP publiques contenues dans le préfixe d’adresse IP publique pour les connexions sortantes. 

Chacune des adresses IP du préfixe d’IP publique fournit 64 000 ports éphémères supplémentaires par adresse IP que l’équilibreur de charge peut utiliser en tant que ports SNAT.

## <a name="outbound-flow-idle-timeout-and-tcp-reset"></a><a name="idletimeout"></a>Délai d’inactivité des flux sortants et réinitialisation TCP

Les règles de trafic sortant ont un paramètre de configuration qui permet de contrôler le délai d’inactivité des flux sortants et de l’ajuster en fonction des besoins de votre application. Le délai d’inactivité des flux sortants est de 4 minutes par défaut. Pour plus d’informations, consultez [Configurer les délais d’inactivité](load-balancer-tcp-idle-timeout.md). 

Le comportement par défaut de l’équilibreur de charge consiste à supprimer silencieusement des flux quand le délai d’inactivité d’un flux sortant est atteint. Le paramètre `enableTCPReset` active un comportement et un contrôle prévisibles de l’application. Le paramètre détermine s’il faut envoyer une réinitialisation TCP bidirectionnelle (RST TCP) au délai d’expiration du délai d’inactivité sortant. 

Pour plus d’informations, y compris sur la disponibilité dans les régions, consultez [Réinitialisation TCP au terme du délai d’inactivité](./load-balancer-tcp-reset.md).

## <a name="securing-and-controlling-outbound-connectivity-explicitly"></a><a name="preventoutbound"></a>Sécurisation et contrôle explicites de la connectivité sortante

Les règles d’équilibrage de charge fournissent une programmation automatique de la NAT de trafic sortant. Dans certains scénarios, il est préférable, ou obligatoire, de désactiver la programmation automatique de la NAT de trafic sortant par la règle d’équilibrage de charge. La désactivation via la règle vous permet de contrôler ou d’adapter vous-même le comportement.  

Vous pouvez utiliser ce paramètre de deux manières :

1. Prévention de l’adresse IP entrante pour la SNAT de trafic sortant. Désactivez la NAT de trafic sortant dans la règle d’équilibrage de charge.
  
2. Paramétrage de la **SNAT** de trafic sortant d’une adresse IP utilisée à la fois pour le trafic entrant et le trafic sortant. La NAT automatique de trafic sortant doit être désactivée pour permettre le contrôle par une règle de trafic sortant. Pour changer l’allocation de ports SNAT d’une adresse qui est également utilisée pour le trafic entrant, le paramètre `disableOutboundSnat` doit être défini sur true. 

L’opération de configuration d’une règle de trafic sortant échoue si vous tentez de redéfinir une adresse IP utilisée pour le trafic entrant.  Désactivez d’abord la NAT de trafic sortant de la règle d’équilibrage de charge.

>[!IMPORTANT]
> Votre machine virtuelle n’aura pas de connectivité sortante si vous définissez ce paramètre sur true et que vous n’avez pas créé de règle de trafic sortant pour définir la connectivité sortante.  Certaines opérations de votre machine virtuelle ou application peuvent avoir besoin d’une connectivité sortante. Assurez-vous de bien comprendre toutes les dépendances de votre scénario, ainsi que les conséquences d’un tel changement.

Dans certains cas, il est préférable pour une machine virtuelle de ne pas créer de flux sortant. Il peut exister une condition qui détermine les destinations qui peuvent recevoir des flux sortants ou celles qui lancent des flux entrants. Utilisez des [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md) pour déterminer les destinations que la machine virtuelle atteint. Utilisez des groupes de sécurité réseau pour déterminer les destinations publiques qui lancent des flux entrants.

Quand vous appliquez un groupe de sécurité réseau à une machine virtuelle à charge équilibrée, vérifiez les [balises de service](../virtual-network/network-security-groups-overview.md#service-tags) et les [règles de sécurité par défaut](../virtual-network/network-security-groups-overview.md#default-security-rules). 

Assurez-vous que la machine virtuelle peut recevoir des demandes de sonde d’intégrité d’Azure Load Balancer.

Si un groupe de sécurité réseau bloque les demandes de sonde d’intégrité depuis la balise par défaut AZURE_LOADBALANCER, votre sonde d’intégrité de la machine virtuelle échoue et la machine virtuelle est marquée comme non disponible. L’équilibreur de charge arrête l’envoi de nouveaux flux vers cette machine virtuelle.

## <a name="scenarios-with-outbound-rules"></a>Scénarios avec règles de trafic sortant
        

### <a name="outbound-rules-scenarios"></a>Scénarios de règles de trafic sortant


* Configurer des connexions sortantes pour un ensemble spécifique d’adresses IP publiques ou un préfixe d’adresse IP publique.
* Modifier l’allocation de ports [SNAT](load-balancer-outbound-connections.md).
* Activer le trafic sortant uniquement.
* NAT de trafic sortant pour les machines virtuelles uniquement (pas pour le trafic entrant).
* NAT de trafic sortant pour un équilibreur de charge standard interne.
* Activer les protocoles TCP et UDP pour la NAT de trafic sortant avec un équilibreur de charge standard public.


### <a name="scenario-1-configure-outbound-connections-to-a-specific-set-of-public-ips-or-prefix"></a><a name="scenario1out"></a>Scénario 1 : Configurer des connexions sortantes pour un ensemble spécifique d’adresses IP publiques ou un préfixe d’adresse IP publique


#### <a name="details"></a>Détails


Utilisez ce scénario pour adapter les connexions sortantes à l’origine d’un ensemble d’adresses IP publiques. Ajoutez des adresses IP publiques ou des préfixes à une liste verte ou rouge en fonction de l’origine.


Cette adresse IP publique ou ce préfixe d’adresse IP publique peuvent être les mêmes que ceux utilisés par une règle d’équilibrage de charge. 


Pour utiliser une adresse IP publique ou un préfixe adresse IP publique différents de ceux utilisés par une règle d’équilibrage de charge : 


1. Créez un préfixe d’adresse IP publique ou une adresse IP publique.
2. Créez un équilibreur de charge standard public. 
3. Créez un serveur frontal référençant le préfixe d’adresse IP publique ou l’adresse IP publique que vous souhaitez utiliser. 
4. Réutilisez ou créez un pool principal, et placez les machines virtuelles dans un pool principal de l’équilibreur de charge public.
5. Configurez une règle de trafic sortant sur l’équilibreur de charge public pour permettre à la NAT de trafic sortant pour les machines virtuelles d’utiliser le serveur frontal. Il n’est pas recommandé d’utiliser une règle d’équilibrage de charge pour le trafic sortant. Désactivez la SNAT de trafic sortant sur la règle d’équilibrage de charge.


### <a name="scenario-2-modify-snatport-allocation"></a><a name="scenario2out"></a>Scénario 2 : Modifier l’allocation de ports [SNAT](load-balancer-outbound-connections.md)


#### <a name="details"></a>Détails


Vous pouvez utiliser des règles de trafic sortant pour paramétrer [l’allocation de ports SNAT automatique en fonction de la taille du pool backend](load-balancer-outbound-connections.md#preallocatedports). 


En cas d’épuisement SNAT, augmentez le nombre de ports [SNAT](load-balancer-outbound-connections.md) par rapport à leur nombre par défaut de 1024. 


Chaque adresse IP publique offre jusqu’à 64 000 ports éphémères. Le nombre de machines virtuelles dans le pool principal détermine le nombre de ports distribués à chaque machine virtuelle. Une machine virtuelle du pool principal a accès au nombre maximal de 64 000 ports. Pour deux machines virtuelles, un maximum de 32 000 ports [SNAT](load-balancer-outbound-connections.md) peuvent être alloués avec une règle de trafic sortant (2 x 32 000 = 64 000). 


Vous pouvez utiliser des règles de trafic sortant pour paramétrer les ports SNAT alloués par défaut. Vous allouez plus ou moins de ports [SNAT](load-balancer-outbound-connections.md) que l’allocation par défaut. Chaque IP publique de serveur frontal d’une règle de trafic sortant offre jusqu’à 64 000 ports éphémères utilisables en tant que ports [SNAT](load-balancer-outbound-connections.md). 


L’équilibreur de charge alloue des ports [SNAT](load-balancer-outbound-connections.md) par multiples de 8. Si vous indiquez une valeur non divisible par huit, l’opération de configuration est rejetée. Chaque règle d’équilibrage de charge et règle NAT de trafic entrant utilise une plage de 8 ports. Si une règle d’équilibrage de charge ou une règle NAT de trafic entrant partage la plage de 8 avec une autre, aucun port supplémentaire n’est utilisé.


Si vous essayez d’allouer plus de ports [SNAT](load-balancer-outbound-connections.md) qu’il en est disponible compte tenu du nombre d’IP publiques, l’opération de configuration est rejetée. Par exemple, si vous allouez 10 000 ports par machine virtuelle et que sept machines virtuelles d’un pool principal partagent une même adresse IP publique, la configuration est rejetée. Le produit de sept fois 10 000 dépasse la limite de 64 000 ports. Pour permettre le scénario, ajoutez des adresses IP publiques au serveur frontal de la règle de trafic sortant. 


Rétablissez l’[allocation de ports par défaut](load-balancer-outbound-connections.md#preallocatedports) en spécifiant 0 comme nombre de ports. Les 50 premières instances de machine virtuelle obtiendront 1 024 ports, et les instances de machine virtuelle 51 à 100 en obtiendront 512 jusqu’au nombre maximal d’instances. Pour plus d’informations sur l’allocation de ports SNAT par défaut, consultez [Table d’allocation de ports SNAT](./load-balancer-outbound-connections.md#preallocatedports).


### <a name="scenario-3-enable-outbound-only"></a><a name="scenario3out"></a>Scénario 3 : Activer pour le trafic sortant uniquement


#### <a name="details"></a>Détails


Utilisez un équilibreur de charge standard public afin de fournir la NAT de trafic sortant pour un groupe de machines virtuelles. Dans ce scénario, utilisez une seule règle de trafic sortant, sans autre règle configurée.


> [!NOTE]
> Une **NAT de réseau virtuel Azure** peut offrir une connectivité sortante pour des machines virtuelles sans nécessiter d’équilibreur de charge. Pour plus d’informations, consultez [Présentation du service NAT de réseau virtuel Azure](../virtual-network/nat-overview.md).

### <a name="scenario-4-outbound-nat-for-vms-only-no-inbound"></a><a name="scenario4out"></a>Scénario 4 : NAT de trafic sortant pour les machines virtuelles uniquement (pas en entrée)


> [!NOTE]
> Une **NAT de réseau virtuel Azure** peut offrir une connectivité sortante pour des machines virtuelles sans nécessiter d’équilibreur de charge. Pour plus d’informations, consultez [Présentation du service NAT de réseau virtuel Azure](../virtual-network/nat-overview.md).

#### <a name="details"></a>Détails


Pour ce scénario : Les règles de trafic sortant de l’équilibreur de charge Azure et le service NAT de réseau virtuel sont des options disponibles pour la sortie à partir d’un réseau virtuel.


1. Créez une adresse IP publique ou un préfixe d’adresse IP publique.
2. Créez un équilibreur de charge standard public. 
3. Créez un serveur frontal associé à l’adresse IP publique ou au préfixe d’adresse IP publique dédié pour le trafic sortant.
4. Créez un pool principal pour les machines virtuelles.
5. Placez les machines virtuelles dans le pool principal.
6. Configurez une règle de trafic sortant pour activer la NAT de trafic sortant.



Utilisez un préfixe ou une IP publique pour mettre à l’échelle les ports [SNAT](load-balancer-outbound-connections.md). Ajoutez la source des connexions sortantes à une liste verte ou rouge.



### <a name="scenario-5-outbound-nat-for-internal-standard-load-balancer"></a><a name="scenario5out"></a>Scénario 5 : NAT de trafic sortant pour équilibreur de charge standard interne


> [!NOTE]
> Une **NAT de réseau virtuel Azure** peut offrir une connectivité sortante pour des machines virtuelles utilisant un équilibreur de charge standard interne. Pour plus d’informations, consultez [Présentation du service NAT de réseau virtuel Azure](../virtual-network/nat-overview.md).

#### <a name="details"></a>Détails


Aucune connectivité sortante n’est disponible pour un équilibreur de charge standard interne tant que celui-ci n’a pas été explicitement déclaré via des adresses IP publiques au niveau de l’instance ou une NAT de réseau virtuel, ou en associant les membres du pool principal à une configuration d’équilibreur de charge en sortie uniquement. 


Pour plus d’informations, consultez [Configuration de l’équilibreur de charge en sortie uniquement](./egress-only.md).




### <a name="scenario-6-enable-both-tcp--udp-protocols-for-outbound-nat-with-a-public-standard-load-balancer"></a><a name="scenario6out"></a>Scénario 6 : Activer les protocoles TCP et UDP pour la NAT de trafic sortant avec un équilibreur de charge standard public


#### <a name="details"></a>Détails


Quand vous utilisez un équilibreur de charge standard public, la NAT de trafic sortant automatique fournie correspond au protocole de transport de la règle d’équilibrage de charge. 


1. Désactivez la [SNAT](load-balancer-outbound-connections.md) de trafic sortant sur la règle d’équilibrage de charge. 
2. Configurez une règle de trafic sortant sur le même équilibreur de charge.
3. Réutilisez le pool backend de vos machines virtuelles. 
4. Spécifiez "protocol": "All" dans la définition de la règle de trafic sortant. 


Si vous utilisez uniquement des règles de NAT de trafic entrant, aucune NAT de trafic sortant n’est fournie. 


1. Placez les machines virtuelles dans un pool backend.
2. Définissez une ou plusieurs configurations d’adresse IP frontale avec une ou plusieurs adresses IP publiques ou un préfixe d’adresse IP publique. 
3. Configurez une règle de trafic sortant sur le même équilibreur de charge. 
4. Spécifiez "protocol": "All" dans la définition de la règle de trafic sortant


## <a name="limitations"></a>Limites

- Le nombre maximal de ports éphémères utilisables par adresse IP frontend est de 64 000.
- Le délai d’inactivité défini pour un flux sortant doit être compris entre 4 et 120 minutes (240 à 7 200 secondes).
- L’équilibreur de charge ne prend pas en charge ICMP pour la NAT de trafic sortant.
- Les règles de trafic sortant peuvent uniquement être appliquées à la configuration d’adresse IP principale d’une carte réseau.  Vous ne pouvez pas créer une règle de trafic sortant pour l’adresse IP secondaire d’une machine virtuelle ou d’une appliance virtuelle réseau. La présence de plusieurs cartes réseau est prise en charge.
- Toutes les machines virtuelles d’un **groupe à haute disponibilité** doivent être ajoutées au pool principal pour la connectivité sortante. 
- Toutes les machines virtuelles d’un **groupe de machines virtuelles identiques** doivent être ajoutées au pool principal pour la connectivité sortante.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Standard Load Balancer](load-balancer-overview.md)
- Consultez notre [foire aux questions relative à Azure Load Balancer](load-balancer-faqs.md)