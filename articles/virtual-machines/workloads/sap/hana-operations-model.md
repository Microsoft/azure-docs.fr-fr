---
title: Modèle d’opération de SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Modèle d’opération de SAP HANA sur Azure (grandes Instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/04/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 73cec422f581daac66f61476ecd4d934b50096eb
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967633"
---
# <a name="operations-model-and-responsibilities"></a>Responsabilités et modèle opérationnel

Le service fourni avec SAP HANA sur Azure (grandes instances) est aligné avec les services Azure IaaS. Vous bénéficiez d’une grande instance HANA avec un système d’exploitation installé optimisé pour SAP HANA. Comme avec les machines virtuelles Azure IaaS, la plupart des tâches de renforcement du SE, d’installation des logiciels supplémentaires, d’installation de HANA, le fonctionnement du SE et de HANA et la mise à jour du SE et de HANA relèvent de votre responsabilité. Microsoft ne vous impose pas de mises à jour du SE, ni de HANA.

![Responsabilités relatives à SAP HANA sur Azure (grandes instances)](./media/hana-overview-architecture/image2-responsibilities.png)

Comme indiqué dans le diagramme, SAP HANA sur Azure (grandes instances) est une offre IaaS mutualisée. Et par conséquent, la répartition des responsabilités se trouve à la limite entre le SE et l’infrastructure. Microsoft se charge de tous les aspects du service sous la ligne du système d’exploitation. Vous êtes responsable de tous les aspects du service au-dessus de la ligne. Le système d’exploitation est sous votre responsabilité. Vous pouvez continuer à utiliser les méthodes locales les plus récentes que vous employez peut-être pour la conformité, la sécurité, la gestion des applications, la base et la gestion du système d’exploitation. Les systèmes sont considérés comme faisant partie de votre réseau, à tous les égards.

Ce service étant optimisé pour SAP HANA, vous devez collaborer avec Microsoft dans certains domaines pour utiliser les capacités de l’infrastructure sous-jacente et obtenir de meilleurs résultats.

La liste suivante fournit plus de détails sur chacune des couches et vos responsabilités :

**Mise en réseau** : tous les réseaux internes pour le tampon de grande Instance exécutant SAP HANA. Votre responsabilité inclut l’accès au stockage, la connectivité entre les instances (pour la montée en puissance et d’autres fonctions), la connectivité avec le paysage et la connectivité à Azure, à l’endroit où la couche Application SAP est hébergée dans les machines virtuelles Azure. Elle inclut également la connectivité WAN entre les centres de données Azure pour la réplication à des fins de récupération d’urgence. Tous les réseaux sont partitionnés par l’abonné et la qualité de service est appliquée.

**Stockage**: le stockage virtualisé partitionné pour tous les volumes requis par les serveurs de SAP HANA, ainsi que pour les captures instantanées. 

**Serveurs** : les serveurs physiques dédiés à l’exécution des bases de données SAP HANA assignées aux abonnés. Les serveurs de classe Type I des références SKU font abstraction du matériel. Avec ces types de serveurs, la configuration du serveur est collectée et gérée dans des profils, qui peuvent être déplacés d’un matériel physique à un autre. Ce déplacement (manuel) d’un profil par des opérations peut être comparé à la réparation des services Azure. Les serveurs des références SKU de classe Type II n’offrent pas ce type de fonction.

**SDDC** : le logiciel de gestion utilisé pour gérer les centres de données comme des entités à définition logicielle. Il permet à Microsoft de regrouper des ressources à des fins de mise à l’échelle, de disponibilité et de performances.

**Système d’exploitation** : le SE que vous choisissez (SUSE Linux ou Red Hat Linux) et qui s’exécute sur les serveurs. Les images du système d’exploitation que vous obtenez sont les images fournies par chaque fournisseur Linux à Microsoft pour l’exécution de SAP HANA. Vous devez être abonné au fournisseur Linux de l’image optimisée pour SAP HANA spécifique. Vous êtes responsable de l’enregistrement des images auprès du fournisseur du SE. 

À partir du point de transfert de Microsoft, vous êtes également responsable des différentes mises à jour correctives du système d’exploitation Linux. Cette mise à jour corrective inclut des packages supplémentaires qui peuvent être nécessaires pour réussir l’installation de SAP HANA et qui ne sont pas inclus par le fournisseur Linux spécifique dans ses images de SE optimisé pour SAP HANA. (Pour plus d’informations, consultez la documentation d’installation HANA de SAP et les Remarques SAP.) 

Vous êtes responsable de la mise à jour corrective du système d’exploitation en raison de mauvais fonctionnement ou d’optimisation du système d’exploitation et de ses pilotes par rapport à du matériel de serveur spécifique. Vous êtes également responsable de la sécurité ou de la mise à jour corrective fonctionnelle du système d’exploitation. 

Vous êtes également responsable de l’analyse et de la planification de la capacité des éléments suivants :

- consommation des ressources d’UC
- consommation de la mémoire
- volumes de disques liés à l’espace libre, à l’IOPS et à la latence
- trafic en volume du réseau entre la grande instance HANA et la couche Application SAP.

L’infrastructure sous-jacente de la grande instance HANA fournit une fonction de sauvegarde et de restauration du volume du SE. L’utilisation de cette fonctionnalité relève également de votre responsabilité.

**Intergiciel** : l’instance SAP HANA principalement. L’administration, les opérations et la surveillance relèvent de votre responsabilité. Vous pouvez utiliser la fonction fournie pour utiliser des captures instantanées de stockage à des fins de récupération d’urgence, de restauration et de sauvegarde. Ces fonctionnalités sont fournies par l’infrastructure. Vos responsabilités incluent également la conception d’une haute disponibilité ou de récupération d’urgence avec ces fonctionnalités, leur utilisation et l’analyse pour déterminer si les captures instantanées de stockage sont correctement exécutées.

**Données** : vos données managées par SAP HANA et d’autres données telles que les fichiers de sauvegarde situés sur des volumes ou des partages de fichiers. Vos responsabilités incluent l’analyse de l’espace libre du disque et la gestion du contenu sur les volumes. Vous êtes également chargé d’analyser de l’exécution réussie des sauvegardes de volumes de disque et des captures instantanées de stockage. L’exécution réussie de la réplication de données pour les sites de récupération d’urgence est sous la responsabilité de Microsoft.

**Applications :** les instances d’application SAP ou, dans le cas d’applications autres que SAP, la couche Application de ces applications. Vos responsabilités incluent le déploiement, l’administration, les opérations et l’analyse de ces applications. Vous êtes responsable de la planification de la capacité de la consommation des ressources de l’UC, la consommation de mémoire, la consommation du stockage Azure et la consommation de la bande passante réseau dans les réseaux virtuels. Vous êtes également chargé de la planification de la capacité pour la consommation des ressources à partir de réseaux virtuels vers SAP HANA sur Azure (grandes instances).

**WAN** : les connexions que vous établissez en local vers des déploiements Azure pour des charges de travail. Tous les clients disposant de la grande instance HANA utilisent Azure ExpressRoute pour la connectivité. Cette connexion ne fait pas partie de la solution SAP HANA sur solution Azure (grandes instances). Vous êtes responsable de l’installation de cette connexion.

**Archive** : vous préférerez peut-être archiver des copies de données à l’aide de vos propres méthodes dans des comptes de stockage. L’archivage inclut de la gestion, de la conformité, des coûts et des opérations. Vous êtes chargé de générer des copies de l’archive et des sauvegardes sur Azure et de les stocker d’une manière conforme.

Consultez le [SLA pour SAP HANA sur des grandes instances Azure](https://azure.microsoft.com/support/legal/sla/sap-hana-large/).

**Étapes suivantes**
- Voir [Architecture SAP HANA (grandes instances) sur Azure](hana-architecture.md)
