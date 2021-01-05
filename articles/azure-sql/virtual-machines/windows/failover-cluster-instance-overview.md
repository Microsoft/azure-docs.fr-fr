---
title: Instances de cluster de basculement
description: En savoir plus sur les instances de cluster de basculement (FCI) avec SQL Server sur des machines virtuelles Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: overview
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.openlocfilehash: 33be57832d9364b859042cd38349c2437bcfcb18
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358144"
---
# <a name="failover-cluster-instances-with-sql-server-on-azure-virtual-machines"></a>Instances de cluster de basculement avec SQL Server sur des machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article présente les différences de fonctionnalités lorsque vous utilisez des instances de cluster de basculement (FCI) pour SQL Server sur des machines virtuelles Azure. 

## <a name="overview"></a>Vue d’ensemble

SQL Server sur les machines virtuelles Azure utilise la fonctionnalité WSFC (clustering de basculement Windows Server) pour fournir une haute disponibilité locale grâce à la redondance au niveau de l'instance de serveur : une instance de cluster de basculement. Une instance FCI est une instance unique de SQL Server installée sur les nœuds WSFC (ou tout simplement du cluster) et, éventuellement, sur plusieurs sous-réseaux. Sur le réseau, une instance FCI semble être une instance de SQL Server s’exécutant sur un seul ordinateur. Toutefois, l’instance FCI permet le basculement d’un nœud WSFC vers un autre si le nœud actuel devient indisponible.

Le reste de cet article se concentre sur les différences entre les instances de cluster de basculement lorsqu’elles sont utilisées avec SQL Server sur des machines virtuelles Azure. Pour en savoir plus sur la technologie de clustering de basculement, consultez : 

- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)
- [Instances de cluster de basculement SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)

## <a name="quorum"></a>Quorum

Les instances de cluster de basculement avec SQL Server sur des machines virtuelles Azure prennent en charge l’utilisation d’un témoin de disque, d’un témoin de cloud ou d’un témoin de partage de fichiers pour le quorum du cluster.

Pour plus d’informations, consultez [Meilleures pratiques de quorum avec des machines virtuelles SQL Server dans Azure](hadr-cluster-best-practices.md#quorum). 


## <a name="storage"></a>Stockage

Dans les environnements en cluster locaux traditionnels, un cluster de basculement Windows utilise un réseau de zone de stockage (SAN) qui est accessible par les deux nœuds en tant que stockage partagé. Les fichiers SQL Server sont hébergés sur le stockage partagé, et seul le nœud actif peut accéder simultanément aux fichiers. 

SQL Server sur les machines virtuelles Azure offre différentes options en tant que solution de stockage partagé pour un déploiement d’instances SQL Server de cluster de basculement : 

||[Disques partagés Azure](../../../virtual-machines/disks-shared.md)|[Partages de fichiers Premium](../../../storage/files/storage-how-to-create-premium-fileshare.md) |[Espaces de stockage direct (S2D)](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)|
|---------|---------|---------|---------|
|**Version de système d’exploitation minimale**| Tous |Windows Server 2012|Windows Server 2016|
|**Version minimale de SQL Server**|Tous|SQL Server 2012|SQL Server 2016|
|**Disponibilité des machines virtuelles prises en charge** |Groupes à haute disponibilité avec groupes de placement de proximité (pour SSD Premium) </br> Même zone de disponibilité (pour SSD Ultra) |Groupes à haute disponibilité et zones de disponibilité|Groupes à haute disponibilité |
|**Prend en charge FileStream**|Oui|Non|Oui |
|**Cache d'objets blob Azure**|Non|Non|Oui|

Le reste de cette section répertorie les avantages et les limitations de chaque option de stockage disponible pour SQL Server sur les machines virtuelles Azure. 

### <a name="azure-shared-disks"></a>Disques partagés Azure

[Les disques partagés Azure](../../../virtual-machines/disks-shared.md) sont une fonctionnalité des [disques managés Azure](../../../virtual-machines/managed-disks-overview.md). Le clustering de basculement Windows Server prend en charge l’utilisation de disques partagés Azure avec une instance de cluster de basculement. 

**Systèmes d’exploitation pris en charge** : Tous   
**Version de SQL pris en charge** : Tous     

**Avantages** : 
- Utile pour les applications qui cherchent à migrer vers Azure tout en conservant leur architecture de haute disponibilité et de récupération d’urgence (HADR) telle quelle. 
- Peut migrer des applications en cluster vers Azure en raison de la prise en charge des réservations persistantes SCSI (SCSI PR). 
- Prend en charge le stockage SSD Premium Azure partagées et Disque Ultra Azure.
- Peut utiliser un seul disque partagé ou entrelacer plusieurs disques partagés pour créer un pool de stockage partagé. 
- Prend en charge Filestream.
- Les disques SSD Premium prennent en charge les groupes à haute disponibilité. 


**Limitations** : 
- Il est recommandé de placer les machines virtuelles dans le même groupe à haute disponibilité et le même groupe de placement de proximité.
- Les disques Ultra ne prennent pas en charge les groupes à haute disponibilité. 
- Les zones de disponibilité sont prises en charge pour les disques Ultra, mais les machines virtuelles doivent être dans la même zone de disponibilité, ce qui réduit la disponibilité de la machine virtuelle. 
- Quelle que soit la solution de disponibilité matérielle choisie, la disponibilité du cluster de basculement est toujours de 99,9 % lorsque vous utilisez des disques partagés Azure. 
- La mise en cache des disques SSD Premium n’est pas prise en charge.

 
Pour commencer, consultez [Instance de cluster de basculement SQL Server avec disques partagés Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md). 

### <a name="storage-spaces-direct"></a>Espaces de stockage direct

[Espaces de stockage direct](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) est une fonctionnalité Windows Server prise en charge avec le clustering de basculement sur les machines virtuelles Azure. Il fournit un réseau SAN virtuel basé sur logiciel.

**Systèmes d’exploitation pris en charge** : Windows Server 2016 et versions ultérieures   
**Version de SQL pris en charge** : SQL Server 2016 et versions ultérieures   


**Avantages :** 
- une bande passante réseau suffisante offre une solution de stockage partagé robuste et hautement performante. 
- Prend en charge le cache d’objets blob Azure pour que les lectures puissent être effectuées localement à partir du cache. (Les mises à jour sont répliquées simultanément sur les deux nœuds.) 
- Prend en charge FileStream. 

**Limitations :**
- Disponible uniquement pour Windows Server 2016 et versions ultérieures. 
- Les zones de disponibilité ne sont pas prises en charge.
- Requiert la même capacité de disque attachée aux deux machines virtuelles. 
- Une bande passante réseau élevée est nécessaire pour obtenir des performances élevées en raison d’une réplication de disque continue. 
- Requiert une plus grande taille de machine virtuelle et le double paiement du stockage, car ce dernier est attaché à chaque machine virtuelle. 

Pour commencer, consultez [Instance de cluster de basculement SQL Server avec espaces de stockage direct](failover-cluster-instance-storage-spaces-direct-manually-configure.md). 

### <a name="premium-file-share"></a>Partage de fichiers Premium

[Les partages de fichiers Premium](../../../storage/files/storage-how-to-create-premium-fileshare.md) sont une fonctionnalité d’[Azure Files](../../../storage/files/index.yml). Les partages de fichiers Premium sont des disques SSD sauvegardés et ont une faible latence. Ils sont entièrement pris en charge pour une utilisation avec les instances de cluster de basculement pour SQL Server 2012 ou version ultérieure sur Windows Server 2012 ou version ultérieure. Les partages de fichiers Premium vous offrent une plus grande flexibilité, car vous pouvez redimensionner et mettre à l’échelle un partage de fichiers sans temps d’arrêt.

**Systèmes d’exploitation pris en charge** : Windows Server 2012 et ultérieur   
**Version de SQL pris en charge** : SQL Server 2012 et versions ultérieures   

**Avantages :** 
- Seule une solution de stockage partagé pour les machines virtuelles est répartie sur plusieurs zones de disponibilité. 
- Système de fichiers complètement managé avec des latences à un chiffre et des performances d’e/s expansibles. 

**Limitations :**
- Disponible uniquement pour Windows Server 2012 et versions ultérieures. 
- FileStream n’est pas pris en charge. 


Pour commencer, consultez [Instance de cluster de basculement SQL Server avec partage de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md). 

### <a name="partner"></a>Partenaire

Il existe des solutions de clustering de partenaires avec stockage pris en charge. 

**Systèmes d’exploitation pris en charge** : Tous   
**Version de SQL pris en charge** : Tous   

Un exemple utilise le stockage DataKeeper SIOS. Pour plus d’informations, consultez l’entrée de blog [Cluster de basculement et DataKeeper SIOS](https://azure.microsoft.com/blog/high-availability-for-a-file-share-using-wsfc-ilb-and-3rd-party-software-sios-datakeeper/).

### <a name="iscsi-and-expressroute"></a>iSCSI et ExpressRoute

Vous pouvez également exposer un stockage de bloc partagé cible iSCSI via Azure ExpressRoute. 

**Systèmes d’exploitation pris en charge** : Tous   
**Version de SQL pris en charge** : Tous   

Par exemple, NPS (NetApp Private Storage) expose une cible iSCSI via ExpressRoute avec Equinix dans les machines virtuelles Azure.

Pour les solutions de stockage partagé et de réplication de données proposées par des partenaires Microsoft, contactez le fournisseur pour tout problème lié à l’accès aux données lors du basculement.

## <a name="connectivity"></a>Connectivité

Les instances de cluster de basculement avec SQL Server sur des machines virtuelles Azure utilisent un [nom de réseau distribué (DNN)](failover-cluster-instance-distributed-network-name-dnn-configure.md) ou un [nom de réseau virtuel (VNN) avec Azure Load Balancer](failover-cluster-instance-vnn-azure-load-balancer-configure.md) pour acheminer le trafic vers l’instance SQL Server, quel que soit le nœud actuellement propriétaire des ressources en cluster. Des considérations supplémentaires sont à prendre en compte lors de l’utilisation de certaines fonctionnalités et de DNN avec une instance FCI de SQL Server. Pour en savoir plus, consultez [Interopérabilité DNN avec l’instance FCI de SQL Server](failover-cluster-instance-dnn-interoperability.md). 

Pour plus d’informations sur les options de connectivité des clusters, consultez [Acheminer les connexions HADR vers SQL Server sur des machines virtuelles Azure](hadr-cluster-best-practices.md#connectivity). 

## <a name="limitations"></a>Limites

Tenez compte des limitations suivantes pour les instances de cluster de basculement avec SQL Server sur des machines virtuelles Azure. 

### <a name="lightweight-extension-support"></a>Prise en charge d’extension légère   

Pour le moment, les instances de cluster de basculement SQL Server sur des machines virtuelles Azure sont prises en charge uniquement avec le [mode de gestion léger](sql-server-iaas-agent-extension-automate-management.md#management-modes) de l’extension SQL Server IaaS Agent. Pour passer du mode d’extension complet au mode d’extension léger, supprimez la ressource **Machine virtuelle SQL** pour les machines virtuelles correspondantes, puis inscrivez-les auprès de l’extension SQL IaaS Agent en mode léger. Quand vous supprimez la ressource **Machine virtuelle SQL** à partir du portail Azure, décochez la case à côté de la machine virtuelle appropriée pour éviter de supprimer la machine virtuelle. 

L’extension complète prend en charge des fonctionnalités telles que la sauvegarde et la mise à jour corrective automatisées et la gestion avancée du portail. Ces fonctionnalités ne marchent pas pour les machines virtuelles SQL Server inscrites en mode de gestion léger.

### <a name="msdtc"></a>MSDTC 

La solution Machines virtuelles Azure prend en charge Microsoft Distributed Transaction Coordinator (MSDTC) sur Windows Server 2019, avec un stockage sur les volumes partagés en cluster (CSV) et [Azure Standard Load Balancer](../../../load-balancer/load-balancer-overview.md) ou sur des machines virtuelles SQL Server qui utilisent des disques partagés Azure. 

Concernant Machines virtuelles Azure, MSDTC n’est pas pris en charge pour Windows Server 2016 ou versions antérieures avec des volumes partagés en cluster pour la raison suivante :

- La ressource MSDTC en cluster n’est pas configurable pour utiliser le stockage partagé. Sur Windows Server 2016, si vous créez une ressource MSDTC, celle-ci n’affiche pas le stockage partagé qui est disponible pour l’utilisation, et cela même si le stockage est disponible. Ce problème a été résolu dans Windows Server 2019.
- L’équilibreur de charge de base ne gère pas les ports RPC.


## <a name="next-steps"></a>Étapes suivantes

Passez en revue les [meilleures pratiques pour les configurations de cluster](hadr-cluster-best-practices.md), puis vous pouvez [préparer votre machine virtuelle SQL Server pour FCI](failover-cluster-instance-prepare-vm.md). 

Pour plus d'informations, consultez les pages suivantes : 

- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instances de cluster de basculement SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)