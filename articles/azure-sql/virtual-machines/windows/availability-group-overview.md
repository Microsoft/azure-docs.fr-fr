---
title: Vue d’ensemble des groupes de disponibilité Always On SQL Server
description: Cet article présente les groupes de disponibilité Always On de SQL Server sur les machines virtuelles Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.assetid: 601eebb1-fc2c-4f5b-9c05-0e6ffd0e5334
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: overview
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/07/2020
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: f39380e253d3fa9e86bfea3a8c436862738ff8e3
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359929"
---
# <a name="always-on-availability-group-on-sql-server-on-azure-vms"></a>Groupes de disponibilité Always On sur SQL Server sur les machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article présente les groupes de disponibilité Always On sur SQL Server sur les machines virtuelles Azure. 

## <a name="overview"></a>Vue d’ensemble

Les groupes de disponibilité Always On sur les machines virtuelles Azure sont semblables aux [groupes de disponibilité Always On locaux](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server). Toutefois, étant donné que les machines virtuelles sont hébergées dans Azure, il existe également quelques points supplémentaires à prendre en considération, comme la redondance des machines virtuelles et le routage du trafic sur le réseau Azure. 

Le diagramme suivant illustre un groupe de disponibilité pour SQL Server sur les machines virtuelles Azure :

![Groupe de disponibilité](./media/availability-group-overview/00-EndstateSampleNoELB.png)


## <a name="vm-redundancy"></a>Redondance des machines virtuelles 

Pour accroître la redondance et la haute disponibilité, les machines virtuelles SQL Server doivent se trouver dans le même [groupe à haute disponibilité](../../../virtual-machines/windows/tutorial-availability-sets.md#availability-set-overview) ou dans des [zones de disponibilité](../../../availability-zones/az-overview.md) différentes.

Le fait de placer un ensemble de machines virtuelles dans un même groupe à haute disponibilité protège contre les pannes qui se produisent au sein d’un centre de données en raison d’un équipement défaillant (les machines virtuelles au sein d’un groupe à haute disponibilité ne partagent pas les ressources) ou contre les mises à jour (les machines virtuelles au sein d’un groupe à haute disponibilité ne sont pas mises à jour en même temps). Les zones de disponibilité protègent contre la défaillance d’un centre de données dans son intégralité, chaque zone représentant un ensemble de centres de centres au sein d’une région.  En faisant en sorte de placer les ressources dans différentes zones de disponibilité, aucune panne au niveau d’un centre de données ne peut mettre toutes vos machines virtuelles hors connexion.

Pendant la création de machines virtuelles Azure, vous devez choisir entre configurer des groupes à haute disponibilité ou des zones de disponibilité.  Une machine virtuelle Azure ne peut pas faire partie des deux.


## <a name="connectivity"></a>Connectivité 

Dans un déploiement local traditionnel, les clients se connectent à l’écouteur de groupe de disponibilité à l’aide du nom de réseau virtuel (VNN), et l’écouteur route le trafic vers le réplica SQL Server approprié dans le groupe de disponibilité. Il existe toutefois une exigence supplémentaire pour router le trafic sur le réseau Azure. 

Avec SQL Server sur les machines virtuelles Azure, configurez un [équilibreur de charge](availability-group-vnn-azure-load-balancer-configure.md) pour router le trafic vers votre écouteur de groupe de disponibilité ou, si vous êtes sur SQL Server 2019 CU8 et une version ultérieure, vous pouvez configurer un [écouteur de nom de réseau distribué (DNN)](availability-group-distributed-network-name-dnn-listener-configure.md) pour remplacer l’écouteur de groupe de disponibilité VNN (nom de réseau virtuel) classique. 


### <a name="vnn-listener"></a>Écouteur de VNN 

Utilisez un [équilibreur de charge Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) pour router le trafic depuis le client vers l’écouteur de nom de réseau virtuel (VNN) du groupe de disponibilité classique sur le réseau Azure. 

Cet équilibreur de charge contient les adresses IP de l’écouteur de VNN. Si vous avez plusieurs groupes de disponibilité, chacun nécessite un écouteur VNN. Un équilibrage de charge prend en charge plusieurs écouteurs.

Pour commencer, consultez [Configurer un équilibreur de charge](availability-group-vnn-azure-load-balancer-configure.md). 

### <a name="dnn-listener"></a>Écouteur de DNN

SQL Server 2019 CU8 introduit la prise en charge de l’écouteur de nom de réseau distribué (DNN, Distributed Network Name). L’écouteur DNN remplace l’écouteur de groupe de disponibilité traditionnel, annulant ainsi complètement la nécessité d’un équilibreur de charge Azure Load Balancer pour router le trafic sur le réseau Azure. 

L’écouteur DNN est la solution de connectivité HADR recommandée dans Azure, car elle simplifie le déploiement, réduit la maintenance et les coûts, et diminue le temps de basculement en cas de défaillance. 

Utilisez l’écouteur DNN pour remplacer un écouteur VNN existant, ou utilisez-le conjointement avec un écouteur VNN existant afin que votre groupe de disponibilité dispose de deux points de connexion distincts : l’un utilisant le nom de l’écouteur VNN (et le port s’il n’est pas défini par défaut) et l’autre utilisant le nom et le port de l’écouteur DNN. Cela peut être utile pour les clients qui veulent éviter la latence du basculement de l’équilibreur de charge, mais en continuant à tirer parti des fonctionnalités SQL Server qui dépendent de l’écouteur VNN, comme les groupes de disponibilité distribués, Service Broker ou un flux de fichier. Pour plus d’informations, consultez [Interopérabilité des fonctionnalités SQL Server et de l’écouteur DNN](availability-group-dnn-interoperability.md)

Pour commencer, consultez [Configurer un écouteur DNN](availability-group-distributed-network-name-dnn-listener-configure.md).


## <a name="deployment"></a>Déploiement 

Il existe plusieurs options pour déployer un groupe de disponibilité sur SQL Server sur des machines virtuelles Azure, certaines comportant davantage d’automatisation que d’autres. 

Le tableau suivant fournit une comparaison des options disponibles :

| | Portail Azure | Azure CLI/PowerShell | Modèles de démarrage rapide | Manuel |
|---------|---------|---------|---------|---------|
|**Version SQL Server** |2016 + |2016 +|2016 +|2012 +|
|**Édition de SQL Server** |Entreprise |Entreprise |Entreprise |Entreprise, Standard|
|**Version de Windows Server**| 2016 + | 2016 + | 2016 + | Tous|
|**Crée le cluster à votre place**|Oui|Oui | Oui |Non|
|**Crée le groupe de disponibilité à votre place** |Oui |Non|Non|Non|
|**Crée un écouteur et un équilibreur de charge de manière indépendante** |Non|Non|Non|Oui|
|**Possibilité de créer un écouteur DNN à l’aide de cette méthode ?**|Non|Non|Non|Oui|
|**Configuration de quorum WSFC**|Témoin de cloud|Témoin de cloud|Témoin de cloud|Tous|
|**Reprise d’activité avec plusieurs régions** |Non|Non|Non|Oui|
|**Prise en charge de plusieurs sous-réseaux** |Oui|Oui|Oui|Oui|
|**Prise en charge d'un domaine d'application existant**|Oui|Oui|Oui|Oui|
|**Reprise d’activité avec plusieurs zones dans la même région**|Oui|Oui|Oui|Oui|
|**Groupe de disponibilité distribué sans domaine d’application**|Non|Non|Non|Oui|
|**Groupe de disponibilité distribué sans cluster** |Non|Non|Non|Oui|

Pour plus d’informations, consultez [Portail Azure](availability-group-azure-portal-configure.md), [Azure CLI/PowerShell](./availability-group-az-commandline-configure.md), [Modèles de démarrage rapide](availability-group-quickstart-template-configure.md) et [Manuel](availability-group-manually-configure-prerequisites-tutorial.md).

## <a name="considerations"></a>Considérations 

Sur un cluster de basculement invité de machine virtuelle IaaS Azure, nous recommandons une seule carte réseau par serveur (nœud de cluster) et un seul sous-réseau. Les réseaux Azure intègrent une redondance physique, ce qui rend inutiles les cartes réseau et les sous-réseaux supplémentaires sur un cluster invité de machine virtuelle IaaS Azure. Même si le rapport de validation de cluster émet un avertissement stipulant que les nœuds sont uniquement accessibles sur un seul réseau, vous pouvez ignorer ce dernier en toute sécurité sur les clusters de basculement invités de machine virtuelle IaaS Azure. 

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les [bonnes pratiques pour HADR](hadr-cluster-best-practices.md), puis commencez à déployer votre groupe de disponibilité avec le [portail Azure](availability-group-azure-portal-configure.md), [Azure CLI/PowerShell](./availability-group-az-commandline-configure.md), les [modèles de démarrage rapide](availability-group-quickstart-template-configure.md) ou [manuellement](availability-group-manually-configure-prerequisites-tutorial.md).

Vous pouvez également déployer un [groupe de disponibilité sans cluster](availability-group-clusterless-workgroup-configure.md) ou un groupe de disponibilité dans [plusieurs régions](availability-group-manually-configure-multiple-regions.md).
