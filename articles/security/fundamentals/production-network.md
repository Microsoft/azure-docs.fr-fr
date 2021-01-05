---
title: Réseau de production Azure
description: Découvrez le réseau de production Azure. Consultez les méthodes d’accès à la sécurité et les mécanismes de protection pour établir une connexion au réseau.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/28/2018
ms.author: terrylan
ms.openlocfilehash: 724c2ee2d74ca9cab11280e92faac71c2cb2218e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87543807"
---
# <a name="the-azure-production-network"></a>Réseau de production Azure
Les utilisateurs du réseau de production Azure peuvent être des clients externes qui accèdent à leurs propres applications Azure ou des membres du personnel interne de support Azure qui gèrent le réseau de production. Cet article décrit les méthodes d’accès de sécurité et les mécanismes de protection pour l’établissement de connexions au réseau de production Azure.

## <a name="internet-routing-and-fault-tolerance"></a>Routage Internet et tolérance de panne
Une infrastructure Azure DNS interne et externe globalement redondante, combinée avec plusieurs clusters de serveurs DNS principaux et secondaires fournit une tolérance de panne. En même temps, des contrôles de sécurité réseau Azure supplémentaires, tels que NetScaler, sont utilisés pour empêcher les attaques par déni de service (DDoS) distribuées et protéger l’intégrité des services Azure DNS.

Les serveurs Azure DNS sont situés dans plusieurs installations de centres de données. L’implémentation de Azure DNS intègre une hiérarchie de serveurs DNS secondaires/primaires pour résoudre publiquement les noms de domaine client Azure. Les noms de domaine se résolvent généralement en une adresse CloudApp.net, qui encapsule l’adresse IP virtuelle (VIP) pour le service du client. Propre à Azure, l’adresse IP virtuelle correspondant à l’adresse IP dédiée (DIP) interne de la traduction du client est établie par les équilibreurs de charge Microsoft responsables de cette adresse IP virtuelle.

Azure est hébergé dans des centres de données Azure répartis géographiquement dans tous les États-Unis, et repose sur des plateformes de routage de pointe qui implémentent des normes architecturales robustes et évolutives. Voici certaines des fonctionnalités notables :

- Ingénierie de trafic basée sur MPLS assurant une utilisation efficace des liens et une dégradation progressive du service en cas de panne.
- Les réseaux sont implémentés avec des architectures à redondance « need plus one » (N + 1) ou mieux.
- En externe, les centres de données sont desservis par des circuits réseau dédiés haut débit dédiés qui connectent de manière redondante des propriétés à plus de 1 200 fournisseurs de services Internet dans le monde entier en plusieurs points de peering. Cette connexion fournit plus de 2 000 Go par seconde (Gbits/s) de la capacité de périphérie.

Comme Microsoft possède ses propres circuits réseau entre centres de données, ces attributs permettent à l’offre Azure d’atteindre une disponibilité de réseau supérieure à 99,9 % sans nécessité de faire appel à des fournisseurs de services Internet tiers traditionnels.

## <a name="connection-to-production-network-and-associated-firewalls"></a>Connexion au réseau de production et pare-feux associés
La stratégie de flux de trafic Internet du réseau Azure dirige le trafic vers le réseau de production Azure situé dans le centre de données régional le plus proche aux États-Unis. Dans la mesure où les centres de données de production Azure maintiennent une architecture et un matériel réseau cohérents, la description du flux de trafic ci-dessous s’applique systématiquement à tous les centres de données.

Une fois le trafic Internet pour Azure acheminé vers le centre de données le plus proche, une connexion est établie pour les routeurs d’accès. Ces routeurs d’accès servent à isoler le trafic entre les nœuds Azure et les machines virtuelles instanciées par le client. Les périphériques d’infrastructure réseau aux emplacements d’accès et de périphérie sont les points limites où s’appliquent les filtres d’entrée et de sortie. Ces routeurs sont configurés par une liste de contrôle d’accès (ACL) à plusieurs niveaux afin de filtrer le trafic indésirable et d’appliquer des limites de débit de trafic, si nécessaire. Le trafic autorisé par la liste de contrôle d’accès est acheminé vers les équilibreurs de charge. Les routeurs de distribution sont conçus pour autoriser uniquement les adresses IP approuvées par Microsoft, assurer la détection d’usurpation, et établir des connexions TCP qui utilisent des listes de contrôle d’accès.

Les appareils d’équilibrage de charge externe sont situés derrière les routeurs d’accès pour effectuer la traduction d’adresses réseau (NAT) d’adresses IP routables sur Internet en adresses IP internes Azure. Ils renvoient également des paquets vers des ports et adresses IP internes de production valides, et servent de mécanisme de protection pour limiter l’exposition de l’espace d’adressage du réseau de production interne.

Par défaut, Microsoft met en œuvre le protocole sécurisé HTTPS (Hypertext Transfer Protocol Secure) pour tout le trafic transmis aux navigateurs web du client, y compris la connexion et tout le trafic qui s’ensuit. L’utilisation de TLS v1.2 crée un tunnel sécurisé pour la circulation du trafic. Les listes de contrôle d’accès sur les routeurs d’accès et de base garantissent que la source du trafic est cohérente avec ce qui est attendu.

Une différence importante de cette architecture par rapport à une architecture de sécurité traditionnelle est qu’il n’y a aucun dispositif dédié de pare-feu matériel, d’appareils de détection ou de prévention d’intrusion spécialisés, ou d’autres appliances de sécurité normalement attendus avant l’établissement de connexions à l’environnement de production Azure. Les clients s’attendent en général à trouver ces périphériques de pare-feu matériel dans le réseau Azure. Pourtant, aucun d’eux n’est utilisé dans Azure. Ces fonctionnalités de sécurité sont intégrées presque exclusivement dans la couche logicielle exécutant l’environnement Azure afin de fournir des mécanismes de sécurité multicouche robustes, dont les fonctionnalités de pare-feu. En outre, l’étendue de la limite et la prolifération associée de périphériques de sécurité critiques sont plus faciles à gérer et à inventorier, comme illustré ci-dessus, car elles sont est gérées par le logiciel exécutant Azure.

## <a name="core-security-and-firewall-features"></a>Fonctionnalités de sécurité critique et de pare-feu
Azure implémente des fonctionnalités robustes de sécurité logicielle et de pare-feu à différents niveaux, afin d’appliquer des fonctionnalités de sécurité généralement attendues d’un environnement traditionnel pour protéger la limite d’autorisation de sécurité de base.

### <a name="azure-security-features"></a>Fonctionnalités de sécurité Azure
Azure implémente des pare-feu logiciels basés sur l’hôte à l’intérieur du réseau de production. Plusieurs fonctionnalités de sécurité principale et de pare-feu résident dans l’environnement principal Azure. Ces fonctionnalités de sécurité reflètent une stratégie de défense en profondeur dans l’environnement Azure. Les données du client dans Azure sont protégées par les pare-feu suivants :

**Pare-feu hyperviseur (filtre de paquets)**  : Ce pare-feu est implémenté dans l’hyperviseur et configuré par l’agent du contrôleur de structure. Ce pare-feu protège le client en cours d’exécution à l’intérieur de la machine virtuelle contre tout accès non autorisé. Par défaut, quand une machine virtuelle est créée, tout le trafic est bloqué. Ensuite, l’agent du contrôleur de structure ajoute au filtre des règles et exceptions afin de permettre le trafic autorisé.

Deux catégories de règles sont programmées ici :

- **Règles de configuration de la machine ou de l’infrastructure** : Par défaut, toutes les communications sont bloquées. Il existe des exceptions qui permettent à une machine virtuelle d’envoyer et de recevoir des communications DHCP et des informations DNS, ainsi que d’acheminer du trafic vers l’Internet « public », à destination d’autres machines virtuelles dans le cluster du contrôleur de structure et le serveur d’activation du système d’exploitation. Étant donné que la liste autorisée de destinations sortantes des machines virtuelles n’inclut pas les sous-réseaux de routeur Azure et d’autres propriétés de Microsoft, les règles font office de couche défensive pour ceux-ci.
- **Règles du fichier de configuration de rôle** : Ceci définit les listes de contrôle d’accès entrant en fonction du modèle de service des locataires. Par exemple, si un client dispose d’un serveur web frontal sur le port 80 d’une machine virtuelle, le port 80 est ouvert à toutes les adresses IP. Si la machine virtuelle a un rôle de travail en cours d’exécution, celui-ci est ouvert uniquement à la machine virtuelle au sein du même client.

**Pare-feu d’hôte natif** : Azure Service Fabric et le Stockage Azure s’exécutent sur un système d’exploitation natif dépourvu d’hyperviseur. Par conséquent, le Pare-feu Windows est configuré avec les deux ensembles de règles précédents.

**Pare-feu d’hôte** : Le pare-feu d’hôte sert à protéger la partition hôte qui exécute l’hyperviseur. Les règles sont programmées pour autoriser uniquement le contrôleur de structure et les « jump boxes » à communiquer avec la partition hôte sur un port spécifique. Les autres exceptions sont d’autoriser la réponse DHCP et les réponses DNS. Azure utilise un fichier de configuration des machines qui contient le modèle de règles de pare-feu pour la partition hôte. Il existe également une exception de pare-feu d’hôte qui permet aux machines virtuelles de communiquer avec les composants, le serveur de réseau et le serveur de métadonnées de l’hôte, via un protocole et des ports spécifiques.

**Pare-feu invité** : Élément de pare-feu Windows du système d’exploitation invité, configurable par le client sur ses machines virtuelles et son stockage.

Autres fonctionnalités de sécurité intégrées aux fonctionnalités Azure :

- Composants d’infrastructure auxquels sont affectées des adresses IP provenant d’adresses IP dédiées (DIP). Un attaquant sur Internet ne peut pas envoyer de trafic à ces adresses, car ils n’atteindrait pas Microsoft. Les routeurs de la passerelle Internet filtrent les paquets destinés uniquement à des adresses internes, afin qu’ils n’entrent pas dans le réseau de production. Les seuls composants qui acceptent le trafic dirigé vers les adresses IP virtuelles sont les équilibreurs de charge.
- Les pare-feu implémentés sur tous les nœuds internes offrent trois principales considérations d’architecture de sécurité pour tout scénario :

   - Les pare-feu sont placés derrière l’équilibreur de charge (LB) et acceptent des paquets de toute provenance. Ces paquets sont destinés à être exposés en externe et correspondent aux ports ouverts d’un pare-feu de périmètre traditionnel.
   - Les pare-feu acceptent uniquement les paquets provenant d’un ensemble limité d’adresses. Cette considération fait partie de la stratégie de défense approfondie contre les attaques DDoS. Ces connexions sont authentifiées par chiffrement.
   - Les pare-feux sont accessibles uniquement à partir de nœuds internes spécifiques. Ils acceptent uniquement les paquets provenant d’adresses IP source répertoriées sur une liste énumérée, qui sont toutes des adresses IP dynamiques au sein du réseau Azure. Par exemple, une attaque sur le réseau d’entreprise peut envoyer des requêtes à ces adresses, mais elle est bloquée sauf si l’adresse source du paquet figure sur la liste énumérée au sein du réseau Azure.
     - Le routeur d’accès au niveau du périmètre bloque les paquets sortants envoyés à une adresse interne du réseau Azure en raison de ses itinéraires statiques configurés.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)
