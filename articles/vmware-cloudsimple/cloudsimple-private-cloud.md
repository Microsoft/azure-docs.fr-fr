---
title: Azure VMware Solution by CloudSimple – Clouds privés
description: Découvrez les concepts et les avantages de CloudSimple, y compris la continuité opérationnelle complète de VMware, la compatibilité avec les outils, les compétences et les processus existants.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: e2096e8c0ebfb233c7449510bf0bc8e80b56231e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88140715"
---
# <a name="cloudsimple-private-cloud-overview"></a>Vue d’ensemble des clouds privés CloudSimple

CloudSimple transforme et étend les charges de travail VMware aux clouds publics en quelques minutes. Le service CloudSimple vous permet de déployer VMware en mode natif sur une infrastructure nue Azure. Votre déploiement réside aux emplacements Azure et s’intègre complètement au reste du cloud Azure.

La solution CloudSimple assure une continuité opérationnelle VMware complète. Cette solution vous offre les avantages du cloud :

* Élasticité
* Innovation
* Efficacité

Avec CloudSimple, vous bénéficiez d’un modèle de consommation cloud qui réduit votre coût total de possession. Cette solution offre également des fonctionnalités d’approvisionnement à la demande, de paiement en fonction de votre croissance et d’optimisation de la capacité.

CloudSimple est totalement compatible avec les éléments suivants :

* Outils existants
* Compétences
* Processus

Cette compatibilité permet à vos équipes de gérer les charges de travail sur le cloud Azure, sans interrompre ces types de stratégies :

* Réseau
* Sécurité  
* Protection de données  
* Audit

CloudSimple gère l’infrastructure, ainsi que tous les services de mise en réseau et de gestion nécessaires. Le service CloudSimple permet à vos équipes de se concentrer sur les aspects suivants :

* Valeur métier
* Approvisionnement d’applications
* Continuité de l’activité
* Support
* Application de stratégies

## <a name="private-cloud-environment-overview"></a>Vue d’ensemble d’un environnement de cloud privé

Un cloud privé est une pile VMware isolée qui prend en charge :

* Hôtes ESXi
* vCenter
* vSAN
* NSX

Les clouds privés sont managés via le portail CloudSimple. Ils ont leur serveur vCenter situé dans son propre domaine de gestion.

La pile s’exécute sur les éléments suivants :

* Nœuds dédiés
* Nœuds matériels nus isolés

Les utilisateurs consomment la pile par le biais des outils VMware natifs, tels que :

* vCenter
* NSX Manager

Vous pouvez déployer des nœuds dédiés aux emplacements Azure. Vous pouvez ensuite les gérer avec Azure et CloudSimple. Un cloud privé se compose d’un ou de plusieurs clusters vSphere, qui contiennent chacun entre 3 et 16 nœuds.

Vous pouvez créer un cloud privé à l’aide de nœuds achetés, avec paiement à l’utilisation ou réservés dédiés.

Vous pouvez connecter le cloud privé à votre environnement local, et le réseau Azure utilise les connexions suivantes :

* Sécuriser
* VPN privé
* Azure ExpressRoute

L’environnement de cloud privé est conçu pour éliminer les points de défaillance uniques :

* Les clusters ESXi sont configurés avec la haute disponibilité vSphere et sont dimensionnés de manière à disposer d’au moins un nœud de réserve à des fins de résilience.
* vSAN fournit le stockage principal redondant. vSAN nécessite au moins trois nœuds afin d’offrir une protection contre une défaillance unique. Vous pouvez configurer vSAN pour qu’il fournisse une résilience accrue pour les clusters de grande taille.
* Vous pouvez configurer les machines virtuelles vCenter, PSC et NSX Manager avec la stratégie de stockage RAID 10 pour les protéger contre les défaillances de stockage. La haute disponibilité vSphere protège contre les défaillances de nœud et de réseau.

## <a name="scenarios-for-deploying-a-private-cloud"></a>Scénarios de déploiement d’un cloud privé

Voici quelques exemples de cas d’usage du déploiement de cloud privé.

### <a name="data-center-retirement-or-migration"></a>Mise hors service ou migration des centres de données

* Bénéficiez d’une capacité supplémentaire lorsque vous atteignez les limites de votre centre de données existant ou actualisez le matériel.
* Ajoutez la capacité requise dans le cloud et éliminez les problèmes liés à la gestion des actualisations du matériel.
* Réduisez les risques et les coûts des migrations vers le cloud, comparativement aux longues tâches de conversion ou de remaniement de l’architecture.
* Appuyez-vous sur les compétences et outils VMware familiers pour accélérer les migrations vers le cloud. Dans le cloud, modernisez vos applications à votre rythme à l’aide des services Azure.

### <a name="expand-on-demand"></a>Développer à la demande

* Étendez-vous au cloud pour faire face aux imprévus, tels que les nouveaux environnements de développement ou les pics de capacité saisonniers.
* Créez de la capacité à la demande et ne la conservez que le temps nécessaire.
* Réduisez votre investissement initial, accélérez la vitesse d’approvisionnement et réduisez la complexité en utilisant la même architecture et les mêmes stratégies en local que dans le cloud.

### <a name="disaster-recovery-and-virtual-desktops-in-the-azure-cloud"></a>Récupération d'urgence et bureaux virtuels dans le cloud Azure

* Établissez un accès à distance aux données, aux applications et aux postes de travail dans le cloud Azure. Grâce aux connexions à large bande passante, vous chargez/téléchargez rapidement des données pour accélérer la récupération en cas d’incident. Les réseaux à faible latence offrent la rapidité de réponse que les utilisateurs attendent d’une application de bureau.

* Répliquez toutes vos stratégies et mises en réseau dans le cloud à l’aide du portail CloudSimple et des outils VMware familiers. La réplication réduit les efforts et les risques liés à la création et à la gestion des implémentations de récupération d’urgence et d’infrastructure VDI (Virtual Desktop Infrastructure).

### <a name="high-performance-applications-and-databases"></a>Applications et bases de données hautes performances

* Exécutez vos charges de travail les plus exigeantes avec l’architecture hyperconvergée fournie par CloudSimple.
* Exécutez Oracle, Microsoft SQL Server, les systèmes intergiciels et les bases de données non-SQL hautes performances.
* Testez le cloud comme s’il s’agissait de votre propre centre de données avec des connexions réseau à haut débit de 25 Gbits/s. Les connexions à haut débit vous permettent d’exécuter des applications hybrides qui couvrent les charges de travail locales, les charges de travail VMware sur Azure et les charges de travail privées Azure, sans compromettre les performances.

### <a name="true-hybrid"></a>Expérience véritablement hybride

* Unifiez les opérations DevOps dans l’ensemble des services VMware et Azure.
* Optimisez l’administration VMware des solutions et services Azure qui sont applicables à toutes vos charges de travail.
* Accédez aux services cloud publics sans avoir à étendre votre centre de données ni à remanier l’architecture de vos applications.
* Centralisez les identités, les stratégies de contrôle d’accès, la journalisation et la surveillance des applications VMware sur Azure.

## <a name="limits"></a>limites

Le tableau suivant répertorie les limites de nœud sur les ressources d’un cloud privé.

| Ressource | Limite |
|----------|-------|
| Nombre minimal de nœuds pour la création d'un cloud privé | 3 |
| Nombre maximal de nœuds dans un cluster sur un cloud privé | 16 |
| Nombre maximal de nœuds dans un cloud privé | 64 |
| Nombre minimal de nœuds sur un nouveau cluster | 3 |

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [créer un cloud privé](create-private-cloud.md)
* Découvrez comment [configurer un environnement de cloud privé](quickstart-create-private-cloud.md)
