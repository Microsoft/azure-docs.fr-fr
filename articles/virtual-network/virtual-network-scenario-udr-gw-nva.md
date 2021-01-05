---
title: Connexion hybride avec une application à 2 niveaux | Microsoft Docs
description: Découvrez comment déployer des appliances virtuelles et un UDR pour créer un environnement d’application multiniveau dans Azure
services: virtual-network
documentationcenter: na
author: KumudD
manager: carmonm
ms.assetid: 1f509bec-bdd1-470d-8aa4-3cf2bb7f6134
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/05/2016
ms.author: kumud
ms.openlocfilehash: 1d2dde4e77a39b114f721cd6d2be250141984e7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86231707"
---
# <a name="virtual-appliance-scenario"></a>Scénario d’appliance virtuelle
Pour les clients Azure volumineux, il faut souvent fournir une application à deux niveaux exposée à Internet, tout en autorisant l’accès au niveau d’arrière-plan à partir d’un centre de données local. Ce document vous guide dans un scénario utilisant des itinéraires définis par l’utilisateur (UDR), une passerelle VPN et des appliances virtuelles de réseau pour déployer un environnement à deux niveaux conforme aux exigences suivantes :

* L’application web ne doit être accessible qu’à partir de l’Internet public.
* Le serveur web hébergeant l’application doit pouvoir accéder à un serveur d’application principal.
* Tout le trafic transitant entre Internet et l’application web doit passer par une appliance virtuelle de pare-feu. Cette appliance virtuelle ne sera utilisée que pour le trafic Internet.
* Tout le trafic envoyé au serveur d’applications doit transiter par une appliance virtuelle de pare-feu. Cette appliance virtuelle sera utilisée pour accéder au serveur principal depuis le réseau local via une passerelle VPN.
* Les administrateurs doivent pouvoir gérer les appliances virtuelles de pare-feu à partir de leurs ordinateurs locaux, en utilisant une troisième appliance virtuelle de pare-feu exclusivement à des fins de gestion.

Il s’agit d’un scénario de réseau de périmètre standard (également nommé DMZ) avec une zone DMZ et un réseau protégé. Ce scénario peut être mis en œuvre dans Azure à l’aide de groupes de sécurité réseau, d’appliances virtuelles de pare-feu ou d’une combinaison des deux. Le tableau ci-dessous présente certains avantages et inconvénients des groupes de sécurité réseau et appliances virtuelles de pare-feu.

|  | Avantages | Inconvénients |
| --- | --- | --- |
| **Groupe de sécurité réseau** |Aucun coût. <br/>Intégré dans Azure RBAC. <br/>Il est possible de créer des règles dans les modèles Azure Resource Manager. |Complexité variable dans les environnements de grande taille. |
| **Pare-feu** |Contrôle total sur le plan des données. <br/>Gestion centralisée via la console du pare-feu. |Coût de l’appliance de pare-feu. <br/>Non intégré dans Azure RBAC. |

La solution ci-dessous utilise des appliances virtuelles de pare-feu pour implémenter un scénario de réseau de périmètre (DMZ)/réseau protégé.

## <a name="considerations"></a>Considérations
Vous pouvez déployer l’environnement décrit ci-dessus dans Azure à l’aide de différentes fonctionnalités disponibles aujourd’hui, comme suit.

* **Réseau virtuel**. Un réseau virtuel Azure se comporte de manière similaire à un réseau local et peut se décomposer en un ou plusieurs sous-réseaux pour isoler le trafic et les problèmes.
* **Appliance virtuelle**. Plusieurs partenaires fournissent des appliances virtuelles dans Azure Marketplace, utilisables pour les trois pare-feu décrits ci-dessus. 
* **Itinéraires définis par l’utilisateur**. Les tables peuvent contenir des itinéraires définis par l’utilisateur, utilisés par la mise en réseau Azure pour contrôler le flux de paquets dans un réseau virtuel. Ces tables d’itinéraires peuvent être appliquées à des sous-réseaux. L’une des fonctionnalités les plus récentes d’Azure consiste à appliquer une table d’itinéraires au sous-réseau GatewaySubnet, pour transférer tout le trafic entrant dans le réseau virtuel Azure entre la connexion hybride et une appliance virtuelle.
* **Transfert IP**. Par défaut, le moteur de mise en réseau Azure ne transfère les paquets aux cartes réseau que si l’adresse IP de destination des paquets correspond à l’adresse IP de la carte réseau. Par conséquent, si un itinéraire défini par l’utilisateur détermine qu’un paquet doit être envoyé à une appliance virtuelle donnée, le moteur de mise en réseau Azure supprime ce paquet. Pour vérifier que le paquet est envoyé à une machine virtuelle (dans ce cas, une appliance virtuelle) qui n’est pas la destination réelle du paquet, vous devez activer le transfert IP pour l’appliance virtuelle.
* **Groupes de sécurité réseau (NSG)** . L’exemple ci-dessous ne fait pas l’utilisation de groupes de sécurité réseau, mais vous pouvez utiliser des NSG appliqués aux sous-réseaux ou des cartes réseau dans cette solution pour filtrer le trafic vers et depuis ces sous-réseaux et cartes réseau.

![Connectivité IPv6](./media/virtual-network-scenario-udr-gw-nva/figure01.png)

Dans cet exemple, il existe un abonnement qui contient les éléments suivants :

* 2 groupes de ressources, non représentés sur le diagramme. 
  * **ONPREMRG**. Contient toutes les ressources nécessaires pour simuler un réseau local.
  * **AZURERG**. Contient toutes les ressources nécessaires pour l’environnement du réseau virtuel Azure. 
* Un réseau virtuel nommé **onpremvnet** utilisé pour imiter un centre de données local segmenté comme indiqué ci-dessous.
  * **onpremsn1**. Sous-réseau contenant une machine virtuelle exécutant Ubuntu pour imiter un serveur local.
  * **onpremsn2**. Sous-réseau contenant une machine virtuelle exécutant Ubuntu pour simuler un ordinateur local utilisé par un administrateur.
* Une appliance virtuelle de pare-feu nommée **OPFW** sur **onpremvnet** utilisée pour conserver un tunnel vers **azurevnet**.
* Un réseau virtuel nommé **azurevnet** segmenté comme indiqué ci-dessous.
  * **azsn1**. Sous-réseau utilisé exclusivement pour le pare-feu externe. Tout le trafic Internet entrera par ce sous-réseau. Ce sous-réseau ne contient qu’une carte réseau liée au pare-feu externe.
  * **azsn2**. Sous-réseau frontal hébergeant une machine virtuelle exécutée en tant que serveur web accessible à partir d’Internet.
  * **azsn3**. Sous-réseau principal hébergeant une machine virtuelle exécutant un serveur d’applications principal sollicité par le serveur web frontal.
  * **azsn4**. Sous-réseau utilisé exclusivement pour fournir un accès de gestion à toutes les appliances virtuelles de pare-feu. Ce sous-réseau ne contient qu’une carte réseau pour chaque appliance virtuelle de pare-feu utilisée dans la solution.
  * **GatewaySubnet**. Sous-réseau de connexion hybride Azure requis pour une route ExpressRoute et une passerelle VPN assurant la connectivité entre des réseaux virtuels Azure et d’autres réseaux. 
* Le réseau **azurevnet** contient 3 appliances virtuelle de pare-feu. 
  * **AZF1**. Pare-feu externe exposé à l’Internet public par l’utilisation d’une ressource ayant une adresse IP publique dans Azure. Vous devez vous procurer un modèle auprès de Marketplace ou directement auprès de votre fournisseur d’appliances, qui approvisionne une appliance virtuelle à 3 cartes réseau.
  * **AZF2**. Pare-feu interne utilisé pour contrôler le trafic entre **azsn2** et **azsn3**. C’est également une appliance virtuelle à 3 cartes réseau.
  * **AZF3**. Pare-feu de gestion accessible aux administrateurs à partir du centre de données local, et connecté à un sous-réseau servant à gérer toutes les appliances de pare-feu. Vous pouvez trouver des modèles d’appliance virtuelle à 2 cartes réseau dans Marketplace ou en demander un à votre fournisseur d’appliances.

## <a name="user-defined-routing-udr"></a>Itinéraire défini par l’utilisateur (UDR)
Chaque sous-réseau dans Azure peut être lié à une table d’UDR utilisée pour définir le mode d’acheminement du trafic dans ce sous-réseau. Si aucun UDR n’est défini, Azure utilise les itinéraires par défaut pour autoriser le trafic d’un sous-réseau à un autre. Pour mieux comprendre les itinéraires définis par l’utilisateur, consultez [Présentation des itinéraires définis par l’utilisateur et du transfert IP](virtual-networks-udr-overview.md).

Pour assurer la communication transite par l’appliance de pare-feu appropriée, conformément à la dernière exigence ci-dessus, vous devez créer la table suivante contenant les UDR de **azurevnet**.

### <a name="azgwudr"></a>azgwudr
Dans ce scénario, seul le trafic local dirigé vers Azure servira à gérer le pare-feu en se connectant à **AZF3**. De plus, le trafic doit transiter par le pare-feu interne **AZF2**. Par conséquent, un seul itinéraire est nécessaire dans **GatewaySubnet** comme indiqué ci-dessous.

| Destination | Tronçon suivant | Explication |
| --- | --- | --- |
| 10.0.4.0/24 |10.0.3.11 |Autorise le trafic local à atteindre le pare-feu de gestion **AZF3** |

### <a name="azsn2udr"></a>azsn2udr
| Destination | Tronçon suivant | Explication |
| --- | --- | --- |
| 10.0.3.0/24 |10.0.2.11 |Autorise le trafic vers le sous-réseau principal qui héberge le serveur d’applications, à transiter par **AZF2** |
| 0.0.0.0/0 |10.0.2.10 |Autorise tout autre trafic à transiter par **AZF1** |

### <a name="azsn3udr"></a>azsn3udr
| Destination | Tronçon suivant | Explication |
| --- | --- | --- |
| 10.0.2.0/24 |10.0.3.10 |Autorise le trafic destiné à **azsn2** à circuler du serveur d’applications vers le serveur web via **AZF2**. |

Vous devez également créer des tables d’itinéraires pour les sous-réseaux dans **onpremvnet** afin d’imiter le centre de données local.

### <a name="onpremsn1udr"></a>onpremsn1udr
| Destination | Tronçon suivant | Explication |
| --- | --- | --- |
| 192.168.2.0/24 |192.168.1.4 |Autorise le trafic destiné à **onpremsn2** à transiter par **OPFW**. |

### <a name="onpremsn2udr"></a>onpremsn2udr
| Destination | Tronçon suivant | Explication |
| --- | --- | --- |
| 10.0.3.0/24 |192.168.2.4 |Autorise le trafic vers le sous-réseau sauvegardé dans Azure à transiter par **OPFW** |
| 192.168.1.0/24 |192.168.2.4 |Autorise le trafic destiné à  **onpremsn1** à transiter par **OPFW**. |

## <a name="ip-forwarding"></a>transfert IP
L’UDR et le transfert IP sont des fonctionnalités que vous pouvez combiner pour permettre à des appliances virtuelles de contrôler le trafic dans un réseau virtuel Azure.  Une appliance virtuelle n’est rien de plus qu’une machine virtuelle qui exécute une application permettant de gérer le trafic réseau d’une certaine façon, comme un pare-feu ou un périphérique NAT.

La machine virtuelle d’appliance virtuelle doit être capable de recevoir le trafic entrant qui ne lui est pas adressé. Pour permettre à une machine virtuelle de recevoir le trafic adressé à d’autres destinations, vous devez activer le transfert IP pour la machine virtuelle. Il s’agit d’un paramètre Azure, pas d’un paramètre du système d’exploitation invité. Votre appliance virtuelle doit toujours exécuter un type d’application pour gérer le trafic entrant et l’acheminer correctement.

Pour plus d’informations sur le transfert IP, consultez la [Présentation des itinéraires définis par l’utilisateur et du transfert IP](virtual-networks-udr-overview.md).

Par exemple, imaginez un réseau virtuel Azure avec la configuration suivante :

* Le sous-réseau **onpremsn1** contient une machine virtuelle nommée **onpremvm1**.
* Le sous-réseau **onpremsn2** contient une machine virtuelle nommée **onpremvm2**.
* Une appliance virtuelle nommée **OPFW** est connectée à **onpremsn1** et à **onpremsn2**.
* Un itinéraire défini par l’utilisateur et lié à **onpremsn1** spécifie que tout le trafic destiné à **onpremsn2** doit être envoyé à **OPFW**.

À ce point, si **onpremvm1** tente d’établir une connexion à **onpremvm2**, l’UDR sera utilisé et le trafic sera envoyé à **OPFW** en tant tronçon suivant. N’oubliez pas que la destination réelle du paquet n’est pas modifiée ( **onpremvm2** ). 

Si le transfert IP n’est pas activé pour **OPFW**, la logique du réseau virtuel Azure supprimera les paquets, car elle n’autorise l’envoi des paquets qu’à une machine virtuelle dont l’adresse IP correspond à la destination du paquet.

Avec le transfert IP, la logique du réseau virtuel Azure transmet les paquets à OPFW, sans modifier leur adresse de destination d’origine. **OPFW** doit gérer les paquets et déterminer ce qu’il faut en faire.

Pour que ce scénario fonctionne, vous devez activer le transfert IP sur les cartes réseau d’**OPFW**, **AZF1**, **AZF2** et **AZF3** qui sont utilisées pour le routage (toutes les cartes réseau sauf celles liées au sous-réseau de gestion). 

## <a name="firewall-rules"></a>Règles de pare-feu
Comme décrit ci-dessus, le transfert IP ne garantit que l’envoi des paquets à des appliances virtuelles. Votre appliance doit encore décider quoi faire avec les paquets. Dans le scénario ci-dessus, vous devez créer les règles suivantes dans vos appliances :

### <a name="opfw"></a>OPFW
OPFW représente un appareil local contenant les règles suivantes :

* **Itinéraire** : tout le trafic vers 10.0.0.0/16 (**azurevnet**) doit être envoyé via le tunnel **ONPREMAZURE**.
* **Stratégie** : autoriser tout le trafic bidirectionnel entre **port2** et **ONPREMAZURE**.

### <a name="azf1"></a>AZF1
AZF1 représente une appliance virtuelle Azure contenant les règles suivantes :

* **Stratégie** : autoriser tout le trafic bidirectionnel entre **port1** et **port2**.

### <a name="azf2"></a>AZF2
AZF2 représente une appliance virtuelle Azure contenant les règles suivantes :

* **Itinéraire** : tout le trafic vers 10.0.0.0/16 (**onpremvnet**) doit être envoyé à l’adresse IP de la passerelle Azure (par exemple, 10.0.0.1) via **port1**.
* **Stratégie** : autoriser tout le trafic bidirectionnel entre **port1** et **port2**.

## <a name="network-security-groups-nsgs"></a>Groupes de sécurité réseau (NSG)
Dans ce scénario, les groupes de sécurité réseau ne sont pas utilisés. Toutefois, vous pouvez appliquer des groupes de sécurité réseau à chaque sous-réseau pour limiter les trafics entrant et sortant. Par exemple, vous pouvez appliquer les règles de groupe de sécurité réseau suivantes au sous-réseau FW externe.

**Entrant**

* Autoriser tout le trafic TCP provenant d’Internet vers le port 80 sur toutes les machines virtuelles du sous-réseau.
* Refuser tout autre trafic provenant d’Internet.

**Sortant**

* Refuser tout le trafic provenant d’Internet.

## <a name="high-level-steps"></a>Étapes de haut niveau
Pour déployer ce scénario, suivez la procédure générale suivante.

1. Connectez-vous à votre abonnement Azure.
2. Si vous souhaitez déployer un réseau virtuel pour imiter le réseau local, approvisionnez les ressources qui font partie de **ONPREMRG**.
3. Approvisionnez les ressources qui font partie de **AZURERG**.
4. Approvisionnez le tunnel reliant **onpremvnet** à **azurevnet**.
5. Une fois toutes les ressources approvisionnées, connectez-vous sur **onpremvm2** et envoyez la commande ping 10.0.3.101 pour tester la connectivité entre **onpremsn2** et **azsn3**.

