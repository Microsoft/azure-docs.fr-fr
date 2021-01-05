---
title: Composants et limites du système d’informations Azure
description: Cet article fournit une description générale de l’architecture de Microsoft Azure et des opérations de gestion associées.
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
ms.openlocfilehash: 68535f70507e7a81d217f4148314a3d76ec832ea
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68727210"
---
# <a name="azure-information-system-components-and-boundaries"></a>Composants et limites du système d’informations Azure
Cet article offre une description générale de l’architecture et de la gestion Azure. L’environnement du système Azure se compose des réseaux suivants :

- Réseau de production Microsoft Azure (réseau Azure)
- Réseau d’entreprise Microsoft (réseau Corpnet)

Des équipes informatiques distinctes sont responsables des opérations et de la maintenance de ces réseaux.

## <a name="azure-architecture"></a>Architecture Azure
Azure est une plateforme et une infrastructure de cloud computing qui permet de générer, de déployer et de gérer des applications et des services à travers un réseau de centres de données. Microsoft gère ces centres de données. En fonction du nombre de ressources que vous spécifiez, Azure crée des machines virtuelles basées sur vos besoins en ressources. Ces machines virtuelles s’exécutent sur un hyperviseur Azure conçu pour une utilisation dans le cloud et inaccessible au public.

Chaque nœud serveur physique Azure est doté d’un hyperviseur qui s’exécute directement sur le matériel. L’hyperviseur divise un nœud en un nombre variable de machines virtuelles invitées. Chaque nœud dispose également d’une machine virtuelle racine, qui exécute le système d’exploitation hôte. Le Pare-feu Windows est activé sur chaque machine virtuelle. Vous définissez quels ports sont adressables en configurant le fichier de définition de service. Ces ports sont les seuls ouverts et adressables, en interne et en externe. L’ensemble du trafic et des accès au disque et au réseau est régi par l’hyperviseur et le système d’exploitation racine.

Au niveau de la couche hôte, les machines virtuelles Azure exécutent la dernière version de Windows Server, personnalisée et renforcée. Azure utilise une version de Windows Server qui inclut uniquement les composants nécessaires à l’hébergement de machines virtuelles. Cela permet d’améliorer les performances et de réduire la surface d’attaque. Les limites des machines sont appliquées par l’hyperviseur qui ne dépend pas de la sécurité du système d’exploitation.

### <a name="azure-management-by-fabric-controllers"></a>Gestion d’Azure par les contrôleurs de structure

Dans Azure, un « cluster » regroupe environ 1 000 machines virtuelles s’exécutant sur des serveurs physiques (panneaux/nœuds). Les machines virtuelles sont gérées indépendamment par un composant logiciel de plateforme, mis à l’échelle horizontalement et redondant, appelé « contrôleur de structure » (FC, Fabric Controller).

Chaque FC gère le cycle de vie des applications qui s’exécutent dans son cluster, et provisionne et surveille l’intégrité du matériel sous son contrôle. Il exécute des opérations autonomes, ce qui lui permet par exemple de réincarner des instances de machine virtuelle sur des serveurs sains quand il détermine qu’un serveur a échoué. Le FC effectue également des opérations de gestion des applications comme le déploiement, la mise à jour et la mise à l’échelle horizontale d’applications.

Le centre de données est divisé en clusters. Les clusters isolent les erreurs au niveau du FC et empêchent certaines classes d’erreurs d’affecter des serveurs au-delà du cluster dans lequel elles se manifestent. Les FC qui servent un cluster Azure particulier sont regroupés dans un cluster FC.

### <a name="hardware-inventory"></a>Inventaire matériel

Le FC prépare l’inventaire des composants matériel et réseau Azure pendant le processus de configuration du démarrage. Tout nouveau composant matériel et réseau entrant dans l’environnement de production Azure doivent suivre le processus de configuration du démarrage. Le FC est responsable de la gestion de tout l’inventaire répertorié dans le fichier de configuration datacenter.xml.

### <a name="fc-managed-operating-system-images"></a>Images du système d'exploitation géré par FC

L’équipe du système d’exploitation fournit des images sous la forme de disques durs virtuels qui sont déployés sur toutes les machines virtuelles hôtes et invitées dans l’environnement de production Azure. L’équipe construit ces images de base par le biais d’un processus de génération hors connexion automatisé. L’image de base est une version du système d’exploitation dans laquelle le noyau et d’autres composants de base ont été modifiés et optimisés pour prendre en charge l’environnement Azure.

Il existe trois types d’images de système d’exploitation gérées par Fabric :

- Hôte : Système d’exploitation personnalisé qui s’exécute sur des machines virtuelles hôtes.
- Natif : Système d’exploitation natif qui s’exécute sur les locataires (par exemple, le stockage Azure). Ce système d’exploitation n’a pas d’hyperviseur.
- Invité : Système d’exploitation invité qui s’exécute sur des machines virtuelles invitées.

Les systèmes d’exploitation hôtes et natifs gérés par un contrôleur de structure sont conçus pour une utilisation dans le cloud et ne sont pas accessibles publiquement.

#### <a name="host-and-native-operating-systems"></a>Systèmes d’exploitation hôtes et natifs

Les systèmes d’exploitation hôtes et natifs sont des images de système d’exploitation renforcées qui hébergent les agents Fabric (FA, Fabric Agent) et qui s’exécutent sur un nœud de calcul (comme première machine virtuelle sur le nœud) et les nœuds de stockage. Utiliser des images de base optimisées des systèmes d’exploitation hôtes et natifs offre un avantage : la surface d’exposition pour les API ou les composants non utilisés est réduite. Ces API et composants peuvent présenter de hauts risques de sécurité, et augmenter l’encombrement du système d’exploitation. Ces systèmes d’exploitation à encombrement réduit incluent uniquement les composants nécessaires à Azure.

#### <a name="guest-operating-system"></a>Système d’exploitation invité

Les composants Azure internes, qui s’exécutent sur des machines virtuelles dotées d’un système d’exploitation invité, ne peuvent pas exécuter le protocole RDP (Remote Desktop Protocol). Tout changement apporté aux paramètres de configuration de la ligne de base doit passer par le processus de gestion des changements et des versions.

## <a name="azure-datacenters"></a>Centres de données Azure
L’équipe MCIO (Microsoft Cloud Infrastructure and Operations) gère les installations physiques de Microsoft liées à l’infrastructure et au centre de données pour tous les services en ligne de Microsoft. MCIO est principalement responsable de la gestion des contrôles physiques et environnementaux dans les centres de données, ainsi que de la gestion et de la prise en charge des périphériques réseau du périmètre externe (comme les routeurs de périphérie et routeurs de centre de données). MCIO est également en charge d’installer le matériel minimal nécessaire au fonctionnement des serveurs sur les racks du centre de données. Les clients n’ont aucune interaction directe avec Azure.

## <a name="service-management-and-service-teams"></a>Gestion des services et équipes de service
Le support du service Azure est assuré par plusieurs groupes d’ingénieurs appelés « équipes de service ». Chaque équipe de service est responsable d’un domaine de support pour Azure. Chaque équipe doit désigner un ingénieur, disponible 24h/24 et 7j/7, pour passer en revue et résoudre les défaillances du service. Par défaut, les équipes de service n’ont pas physiquement accès au matériel sur lequel fonctionne Azure.

Les équipes de service sont les suivantes :

- Application Platform
- Azure Active Directory
- Azure Compute
- Azure Net
- Cloud Engineering Services
- ISSD : Sécurité
- Multifactor Authentication
- SQL Database
- Stockage

## <a name="types-of-users"></a>Types d’utilisateurs
Les employés (ou sous-traitants) de Microsoft sont considérés comme des utilisateurs internes. Tous les autres utilisateurs sont considérés comme des utilisateurs externes. Les utilisateurs internes d’Azure ont tous un statut d’employé auquel est affecté un niveau de sensibilité qui définit leur accès ou non aux données des clients. Les privilèges dont disposent les utilisateurs dans Azure (autorisations accordées après l’authentification) sont décrits dans le tableau suivant :

| Role | Interne ou externe | Niveau de sensibilité | Privilèges autorisés et fonctions exécutées | Type d’accès
| --- | --- | --- | --- | --- |
| Ingénieur de centre de données Azure | Interne | Pas d’accès aux données des clients | Gérer la sécurité physique des locaux Effectuer des patrouilles à l’intérieur et à l’extérieur du centre de données, et surveiller tous les points d’entrée Effectuer des services d’escorte dans et hors du centre de données pour certains membres du personnel non autorisés qui fournissent des services généraux (restauration, nettoyage) ou des travaux informatiques dans le centre de données Effectuer la surveillance et la maintenance de routine du matériel réseau Gérer les incidents et réparer le matériel au moyen de divers outils Effectuer la surveillance et la maintenance de routine du matériel physique dans les centres de données Accéder à l’environnement à la demande des propriétaires Capacité à effectuer des investigations, à consigner des rapports d’incident et à exiger une formation et des règles relatives à la sécurité La propriété opérationnelle et la maintenance des outils de sécurité critiques, tels que les scanneurs et la collecte de journaux | Accès persistant à l’environnement |
| Tri des incidents Azure (ingénieurs chargés des interventions rapides) | Interne | Accès aux données des clients | Gérer les communications entre MCIO, le support et les équipes d’ingénierie Tri des incidents de plateforme, des problèmes de déploiement et des demandes de service | Accès juste à temps à l’environnement (avec accès persistant limité aux systèmes non clients) |
| Ingénieurs en charge du déploiement d’Azure | Interne | Accès aux données des clients | Déployer et mettre à niveau les composants de plateforme, les logiciels et les changements de configuration planifiés pour prendre en charge Azure | Accès juste à temps à l’environnement (avec accès persistant limité aux systèmes non clients) |
| Support des pannes des clients Azure (locataire) | Interne | Accès aux données des clients | Déboguer et diagnostiquer les pannes de la plateforme et les erreurs pour les locataires de calcul individuels et les comptes Azure. Analyser les erreurs. Appliquer des correctifs critiques à la plateforme ou au client, et appliquer des améliorations techniques pour le support. | Accès juste à temps à l’environnement (avec accès persistant limité aux systèmes non clients) |
| Ingénieurs chargés du site actif Azure (ingénieurs en supervision) et des incidents | Interne | Accès aux données des clients | Diagnostiquer et résoudre les problèmes d’intégrité de la plateforme à l’aide des outils de diagnostic. Appliquer des correctifs pour les pilotes de volume, réparer des éléments résultant de pannes, et faciliter les actions de restauration des pannes. | Accès juste à temps à l’environnement (avec accès persistant limité aux systèmes non clients) |
|Clients Azure | Externe | N/A | N/A | N/A |

Azure utilise des identificateurs uniques pour authentifier les utilisateurs et clients organisationnels (ou les processus agissant au nom d’utilisateurs organisationnels). Cela s’applique à l’ensemble des ressources et appareils qui font partie de l’environnement Azure.

### <a name="azure-internal-authentication"></a>Authentification interne dans Azure

Les communications entre les composants internes Azure sont protégées avec le chiffrement TLS. Dans la plupart des cas, les certificats X.509 sont auto-signés. Des exceptions sont faites pour les certificats avec des connexions accessibles à partir de l’extérieur du réseau Azure, par exemple les certificats pour les FC. Ces derniers ont des certificats émis par une autorité de certification (AC) Microsoft soutenue par une AC racine de confiance. Les clés publiques des FC peuvent ainsi être facilement reconduites. En outre, les outils de développement Microsoft utilisent des clés publiques des FC. Quand les développeurs soumettent de nouvelles images d’application, celles-ci sont donc chiffrées avec une clé publique de FC pour protéger les secrets incorporés.

### <a name="azure-hardware-device-authentication"></a>Authentification des périphériques matériels Azure

Le FC gère un ensemble d’informations d’identification (clés et/ou mots de passe) permettant son authentification auprès de divers périphériques matériels sous son contrôle. Microsoft utilise un système pour empêcher l’accès à ces informations d’identification. Tout particulièrement, le transport, la persistance et l’utilisation de ces informations d’identification sont conçus pour empêcher les développeurs Azure, les administrateurs et les services/personnels de remplacement d’avoir accès aux informations privées, confidentielles ou sensibles.

Microsoft utilise le chiffrement basé sur la clé publique de l’identité maître du FC. Ce chiffrement est utilisé dans les phases d’installation et de reconfiguration du FC pour transférer les informations d’identification permettant d’accéder aux périphériques matériels réseau. Lorsque le FC a besoin des informations d’identification, il les récupère et les déchiffre.

### <a name="network-devices"></a>Périphériques réseau

Les comptes de service réseau sont configurés par l’équipe en charge de la mise en réseau d’Azure pour permettre à un client Azure de s’authentifier auprès de périphériques réseau (routeurs, commutateurs et équilibreurs de charge).

## <a name="secure-service-administration"></a>Administration sécurisée des services
Le personnel d’exploitation Azure doit utiliser des stations de travail administrateur sécurisées (SAW). Les clients peuvent implémenter des contrôles similaires à l’aide de stations de travail à accès privilégié. Avec les stations de travail SAW, le personnel d’administration utilise un compte d’administration affecté à titre individuel et distinct du compte standard de l’utilisateur. SAW s’appuie sur cette pratique de séparation des comptes en fournissant une station de travail digne de confiance pour ces comptes sensibles.

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur ce que Microsoft fait pour sécuriser l’infrastructure Azure, consultez :

- [Sécurité locale et physique des centres de données Azure](physical-security.md)
- [Disponibilité de l’infrastructure Azure](infrastructure-availability.md)
- [Architecture réseau Azure](infrastructure-network.md)
- [Réseau de production Azure](production-network.md)
- [Fonctionnalités de sécurité d’Azure SQL Database](infrastructure-sql.md)
- [Opérations de production et administration Azure](infrastructure-operations.md)
- [Surveillance de l’infrastructure Azure](infrastructure-monitoring.md)
- [Intégrité de l’infrastructure Azure](infrastructure-integrity.md)
- [Protection des données client Azure](protection-customer-data.md)
