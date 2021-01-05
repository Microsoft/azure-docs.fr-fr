---
title: Composants VMware de cloud privé
titleSuffix: Azure VMware Solution by CloudSimple
description: Découvrez comment le service CloudSimple vous permet de déployer VMware en mode natif dans des emplacements Azure. Les clouds privés sont intégrés au reste d’Azure Cloud.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: ff9aec1904be754990958869666e9d67038e4fb3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88142500"
---
# <a name="private-cloud-vmware-components"></a>Composants VMware de cloud privé

Un cloud privé est un environnement isolé de la pile VMware (hôtes ESXi, vCenter, vSAN et NSX) géré par un serveur vCenter dans un domaine de gestion.  Le service CloudSimple permet de déployer VMware en mode natif sur une infrastructure nue Azure dans des emplacements Azure.  Les clouds privés sont intégrés au reste d’Azure Cloud.  Ils sont déployés avec plusieurs composants de la pile VMware :

* **VMware ESXi –** Hyperviseur sur des nœuds dédiés Azure
* **VMware vCenter –** Appliance de gestion centralisée d’un environnement vSphere de cloud privé
* **VMware vSAN –** Solution d’infrastructure hyperconvergée
* **Centre de données VMware NSX –** Logiciel de sécurité et de virtualisation de réseau  

## <a name="vmware-component-versions"></a>Versions des composants VMware

Une pile VMware de cloud privé se déploie avec la version logicielle suivante.

| Composant | Version | Version sous licence |
|-----------|---------|------------------|
| ESXi | 6.7U2 | Enterprise Plus |
| vCenter | 6.7U2 | vCenter Standard |
| vSAN | 6.7 | Entreprise |
| Centre de données NSX | 2.4.1 | Avancé |

## <a name="esxi"></a>ESXi

VMware ESXi est installé sur les nœuds CloudSimple approvisionnés lors de la création d’un cloud privé.  ESXi fournit l’hyperviseur permettant de déployer des machines virtuelles de charge de travail.  Les nœuds offrent une infrastructure hyperconvergée (calcul et stockage) sur le cloud privé.  Ils font partie du cluster vSphere sur le cloud privé.  Chaque nœud possède quatre interfaces de réseaux physiques connectées au réseau sous-jacent.  Deux sont utilisées pour créer un **commutateur réparti vSphere (VDS)** sur vCenter et deux pour créer un **commutateur réparti virtuel géré par NSX (N-VDS)**.  Les interfaces réseau sont configurées en mode actif-actif pour la haute disponibilité.

En savoir plus sur VMware ESXi

## <a name="vcenter-server-appliance"></a>Appliance serveur vCenter

L’appliance serveur vCenter (VCSA) fournit les fonctions d’authentification, de gestion et d’orchestration de VMware Solution by CloudSimple. L’appliance VCSA avec le Platform Services Controller (PSC) intégré est déployée lors de la création du cloud privé,  sur le cluster vSphere créé au moment du déploiement du cloud privé.  Chaque cloud privé a sa propre appliance VCSA.  L’expansion d’un cloud privé ajoute les nœuds à l’appliance VCSA sur le cloud privé.

### <a name="vcenter-single-sign-on"></a>Authentification unique vCenter

Le Platform Services Controller intégré sur VCSA est associé à un **domaine d’authentification unique vCenter**  dont le nom est **cloudsimple.local**.  Un utilisateur par défaut **CloudOwner@cloudsimple.com** est créé pour vous permettre d’accéder à vCenter.  Vous pouvez ajouter des [sources d’identité pour vCenter](set-vcenter-identity.md) locales/Azure Active Directory.

## <a name="vsan-storage"></a>Stockage vSAN

Les clouds privés sont créés avec un stockage vSAN 100 % flash entièrement configuré, en local pour le cluster.  Il faut au minimum trois nœuds de la même référence SKU pour créer un cluster vSphere avec magasin de données vSAN.  La déduplication et la compression sont activées par défaut dans la banque de données vSAN.  Deux groupes de disques sont créés sur chaque nœud du cluster vSphere. Chacun contient un disque de cache et trois disques de capacité.

Une stratégie de stockage vSAN par défaut est créée sur le cluster vSphere et appliquée au magasin de données vSAN.  Elle détermine la manière dont les objets de stockage de machine virtuelle sont approvisionnés et alloués dans le magasin de données pour garantir le niveau de service requis.  La stratégie de stockage définit le **nombre de pannes tolérées** et la **méthode de tolérance aux pannes**.  Il est possible de créer des stratégies de stockage et de les appliquer aux machines virtuelles. Pour remplir le contrat SLA, une capacité supplémentaire de 25 % est nécessaire sur le magasin de données vSAN.  

### <a name="default-vsan-storage-policy"></a>Stratégie de stockage vSAN par défaut

Le tableau ci-dessous présente les paramètres de stratégie de stockage vSAN par défaut.

| Nombre de nœuds du cluster vSphere | Nombre de pannes tolérées | Méthode de tolérance aux pannes |
|------------------------------------|-----|--------------------------|
| 3 ou 4 nœuds | 1 | RAID-1 (mise en miroir) : crée deux copies |
| Entre 5 et 16 nœuds | 2 | RAID-1 (mise en miroir) : crée trois copies |

## <a name="nsx-data-center"></a>Centre de données NSX

Le centre de données NSX offre des fonctionnalités de virtualisation de réseau, de microsegmentation et de sécurité réseau sur le cloud privé.  Tous les services pris en charge par le centre de données NSX sont configurables sur le cloud privé avec NSX.  Lors de la création d’un cloud privé, les composants NSX suivants sont installés et configurés.

* NSX Manager
* Zone de transport
* Profil de liaison montante hôte et périphérie
* Commutateur logique pour le transport Edge, Ext1 et Ext2
* Pool d’adresses IP pour le nœud de transport ESXi
* Pool d’adresses IP pour le nœud de transport Edge
* Nœuds de périphérie
* Règle anti-affinité DRS pour le contrôleur et les machines virtuelles Edge
* Routeur de niveau 0
* Protocole BGP sur le routeur de niveau 0

## <a name="vsphere-cluster"></a>Cluster vSphere

Les hôtes ESXi sont configurés en tant que cluster pour garantir la haute disponibilité du cloud privé.  Lors de la création d’un cloud privé, les composants de gestion de vSphere sont déployés sur le premier cluster.  Une liste des ressources est créée pour les composants de gestion, dans laquelle toutes les machines virtuelles de gestion sont déployées. Il n’est pas possible de supprimer le premier cluster pour réduire le cloud privé.  Le cluster vSphere assure la haute disponibilité des machines virtuelles avec **vSphere HA**.  Le nombre de pannes tolérées dépend du nombre de nœuds disponibles dans le cluster.  Vous pouvez utiliser la formule ```Number of nodes = 2N+1```, où ```N``` est le nombre de pannes à tolérer.

### <a name="vsphere-cluster-limits"></a>Limites du cluster vSphere

| Ressource | Limite |
|----------|-------|
| Nombre minimal de nœuds pour la création d’un cloud privé (premier cluster vSphere) | 3 |
| Nombre maximal de nœuds dans un cluster vSphere sur un cloud privé | 16 |
| Nombre maximal de nœuds dans un cloud privé | 64 |
| Nombre maximal de clusters vSphere dans un cloud privé | 21 |
| Nombre minimal de nœuds sur un nouveau cluster vSphere | 3 |

## <a name="vmware-infrastructure-maintenance"></a>Maintenance de l’infrastructure VMware

Il est parfois nécessaire d’apporter des modifications à la configuration de l’infrastructure VMware. La fréquence, actuellement mensuelle ou bimestrielle, devrait diminuer au fil du temps. Ce type de maintenance est généralement possible sans interruption de la consommation des services CloudSimple. Pendant un intervalle de maintenance VMware, les services suivants continuent de fonctionner sans aucun impact :

* Applications et plan de gestion VMware
* Accès vCenter
* Tout ce qui concerne les réseaux et le stockage
* Tout le trafic Azure

## <a name="updates-and-upgrades"></a>Mises à jour et mises à niveau

CloudSimple est responsable de la gestion du cycle de vie des logiciels VMware (ESXi vCenter, PSC et NSX) dans le cloud privé.

Il existe plusieurs mises à jour de logiciels :

* **Correctifs**. Mises à jour de sécurité ou correctifs de bogues publiés par VMware.
* **Mises à jour**. Changement de version mineure d’un composant de la pile VMware.
* **Mises à niveau**. Changement de version majeure d’un composant de la pile VMware.

CloudSimple teste les mises à jour de sécurité critiques dès qu’elles sont mises à disposition par VMware. Conformément au contrat SLA, CloudSimple déploie le correctif de sécurité dans les environnements de cloud privé sous une semaine.

CloudSimple propose des mises à jour de maintenance trimestrielles des composants logiciels VMware. Lorsqu’une nouvelle version majeure d’un logiciel de VMware est disponible, CloudSimple se coordonne avec les clients pour trouver une fenêtre de maintenance adaptée à la mise à niveau.  

## <a name="next-steps"></a>Étapes suivantes

* [Maintenance et mises à jour CloudSimple](cloudsimple-maintenance-updates.md)
