---
title: Fonctionnalités de sécurité Azure SQL Database
description: Cet article fournit une description générale de la façon dont Azure SQL Database protège les données des clients dans Azure.
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
ms.date: 03/09/2020
ms.author: terrylan
ms.openlocfilehash: e0e7089e7c674f324c2c3d293661c518b41731b9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84021855"
---
# <a name="azure-sql-database-security-features"></a>Fonctionnalités de sécurité Azure SQL Database    
Azure SQL Database fournit un service de base de données relationnelle dans Azure. Pour protéger les données des clients et fournir des fonctionnalités de sécurité renforcée que les clients attendent d’un service de base de données relationnelle, SQL Database a ses propres jeux de fonctionnalités de sécurité. Ces fonctionnalités reposent sur les contrôles hérités d’Azure.

## <a name="security-capabilities"></a>Fonctionnalités de sécurité

### <a name="usage-of-the-tds-protocol"></a>Utilisation du protocole TDS
Azure SQL Database prend uniquement en charge le protocole TDS (tabular data stream), qui exige que la base de données soit accessible uniquement sur le port par défaut TCP/1433.

### <a name="azure-sql-database-firewall"></a>Pare-feu Azure SQL Database
Pour protéger les données des clients, Azure SQL Database comprend une fonctionnalité de pare-feu qui, par défaut, empêche tout accès à SQL Database, comme indiqué ci-dessous.

![Pare-feu Azure SQL Database](./media/infrastructure-sql/sql-database-firewall.png)

Le pare-feu de la passerelle permet de limiter les adresses, les clients peuvent ainsi bénéficier d’un contrôle précis en spécifiant des plages d’adresses IP acceptables. Le pare-feu accorde l’accès en fonction de l’adresse IP d’origine de chaque demande.

Vous pouvez procéder à la configuration du pare-feu à l’aide d’un portail de gestion ou par programmation avec l’API REST Azure SQL Database Management. Par défaut, le pare-feu de la passerelle Azure SQL Database empêche tout accès TDS client à Azure SQL Database. L’accès doit être configuré à l’aide de listes de contrôle d’accès ACL pour autoriser les connexions à Azure SQL Database en fonction des adresses Internet sources et cibles, des protocoles et des numéros de port.

### <a name="dosguard"></a>DoSGuard
Les attaques par déni de service (DoS) sont réduites par un service de passerelle SQL Database appelé DoSGuard. DoSGuard suit activement les échecs de connexion des adresses IP. Si une adresse IP spécifique présente plusieurs échecs de connexion dans une période donnée, elle est bloquée et ne peut plus accéder aux ressources du service pour une durée prédéfinie.

En outre, la passerelle Azure SQL Database effectue :

- Des négociations de fonctionnalité de canal sécurisé pour implémenter des connexions TDS FIPS 140-2 chiffrées validées lors de la connexion aux serveurs de base de données.
- Une inspection des paquets TDS avec état lorsqu’il accepte les connexions des clients. La passerelle valide les informations de connexion et passe les paquets TDS au serveur physique approprié en se basant sur le nom de la base de données spécifié dans la chaîne de connexion.

Le principe dominant de la sécurité réseau du produit Azure SQL Database consiste à autoriser uniquement les connexions et les communications qui sont nécessaires au fonctionnement du service. Tous les autres ports, protocoles et connexions sont bloqués par défaut. Les réseaux locaux virtuels (VLAN) et les listes ACL sont utilisés pour limiter les communications réseau en fonction des réseaux sources et cibles, des protocoles et des numéros de port.

Les mécanismes approuvés pour implémenter des listes ACL basées sur le réseau comprennent les listes ACL sur les routeurs et les équilibreurs de charge. Ils sont gérés par le réseau Azure, le pare-feu de machine virtuelle invitée et les règles de pare-feu de la passerelle Azure SQL Database, qui sont configurées par le client.

## <a name="data-segregation-and-customer-isolation"></a>Séparation des données et isolation des clients
Le réseau de production Azure est structuré de sorte que les composants système accessibles à tous soient séparés des ressources internes. Des limites physiques et logiques existent entre les serveurs web qui fournissent un accès au portail Azure public et à l’infrastructure virtuelle Azure sous-jacente, où se trouvent les données et les instances d’application des clients.

Toutes les informations accessibles au public sont gérées au sein du réseau de production Azure. Le réseau de production est soumis aux mécanismes de l’authentification à deux facteurs et de la protection des limites, utilise le jeu de fonctionnalités de pare-feu et de sécurité décrit dans la section précédente et utilise les fonctions d’isolation des données comme indiqué ci-dessous.

### <a name="unauthorized-systems-and-isolation-of-the-fc"></a>Systèmes non autorisés et isolation du contrôleur de structure
Étant donné que le contrôleur de structure (FC) est l’orchestrateur central de la structure d’Azure, des contrôles importants sont en place pour atténuer les menaces, en particulier des agents étrangers potentiellement compromis dans les applications des clients. Le contrôle de structure ne reconnaît aucun matériel dont les informations d’appareil (par exemple, une adresse MAC) ne sont pas préchargées dans le contrôleur de structure. Les serveurs DHCP sur le contrôleur de structure ont configuré les listes d’adresses MAC des nœuds qu’ils veulent démarrer. Même si les systèmes non autorisés sont connectés, ils ne sont pas incorporés dans l’inventaire de structure et par conséquent ne sont ni connectés ni autorisés à communiquer avec un système de l’inventaire de structure. Cela réduit le risque que les systèmes non autorisés communiquent avec le contrôleur de structure et obtiennent un accès au réseau VLAN et à Azure.

### <a name="vlan-isolation"></a>Isolation du VLAN
Le réseau de production Azure est logiquement séparé en trois grands réseaux VLAN :

- Le VLAN principal : Interconnecte les nœuds des clients non approuvés.
- Le VLAN du contrôleur de structure : Contient les systèmes de support et les contrôleurs de structure approuvés.
- Le VLAN de l’appareil : Contient les réseaux approuvés et les autres infrastructures d’appareil.

### <a name="packet-filtering"></a>Filtrage des paquets
Les pare-feux IPFilter et logiciels implémentés sur le système d’exploitation racine et le système d’exploitation invité des nœuds appliquent des restrictions de connectivité et empêchent le trafic non autorisé entre les machines virtuelles.

### <a name="hypervisor-root-os-and-guest-vms"></a>Hyperviseur, système d’exploitation racine et machines virtuelles invitées
L’isolation du système d’exploitation racine entre des machines virtuelles invitées est gérée par l’hyperviseur et le système d’exploitation racine.

### <a name="types-of-rules-on-firewalls"></a>Types de règles de pare-feu
Une règle est définie comme suit :

{IP Source, Port source, IP de destination, Port de destination, Protocole de destination, Entrée/Sortie, Avec état/Sans état, Expiration du flux avec état}.

Les paquets de caractères synchrones (SYN) sont autorisés en entrée et en sortie seulement si l’une des règles le permet. Pour TCP, Azure utilise des règles sans état dont le principe est d’autoriser uniquement tous les paquets non-SYN entrant ou sortant de la machine virtuelle. Le principe de sécurité est que les piles d’hôtes persistent à ignorer un paquet non-SYN si elles n’ont pas déjà vu de paquet SYN. Le protocole TCP lui-même est avec état, et combiné à la règle basée sur un comportement SYN sans état, il adopte un comportement général d’implémentation avec état.

Pour UDP (User Datagram Protocol), Azure utilise une règle avec état. Chaque fois qu’un paquet UDP correspond à une règle, un flux inverse est créé dans l’autre direction. Ce flux a un délai d’expiration intégré.

Les clients sont responsables de la configuration de leur propre pare-feu en plus de ce que fournit Microsoft Azure. Ici, les clients peuvent définir les règles pour le trafic entrant et sortant.

### <a name="production-configuration-management"></a>Gestion de configuration de production
Les configurations sécurisées standard sont gérées par les équipes des opérations respectives Azure et Azure SQL Database. Tous les changements de configuration des systèmes de production sont documentés et suivis dans un système de suivi centralisé. Les changements logiciels et matériels sont suivis dans le système de suivi centralisé. Les changements de mise en réseau relatifs aux listes ACL sont suivis à l’aide du service de gestion ACL.

Tous les changements de configuration Azure sont développés et testés dans l’environnement de préproduction, puis déployés dans l’environnement de production. Les builds logicielles sont révisées dans le cadre des tests. Les contrôles de sécurité et de confidentialité sont revérifiés dans le cadre des critères de check-list d’entrée. Les changements sont déployés à intervalles planifiés par l’équipe de déploiement respective. Les versions sont révisées et validées par le personnel de l’équipe de déploiement respectif avant de les déployer en production.

Les changements sont vérifiés pour voir s’ils ont été correctement appliqués. Dans un scénario d’échec, le changement est restauré à son état précédent ou un logiciel correctif est déployé pour résoudre le problème avec l’approbation du personnel désigné. Le dépôt source Git, TFS, Master Data Services (MDS), Runners, le monitoring de la sécurité Azure, le contrôleur de structure et la plateforme WinFabric sont utilisés pour centraliser la gestion, l’application et la vérification des paramètres de configuration dans l’environnement virtuel Azure.

De même, les changements matériels et réseau ont des étapes de validation établies pour évaluer leur adhérence aux exigences de build. Les versions sont révisées et autorisées par un comité consultatif sur les modifications coordonnées (CAB) des groupes respectifs de la pile.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Composants et limites du système d’informations Azure](infrastructure-components.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)
