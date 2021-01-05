---
title: Meilleures pratiques en matière de configuration de cluster
description: En savoir plus sur les configurations de cluster prises en charge lorsque vous configurez la haute disponibilité et la récupération d’urgence (HADR) pour SQL Server sur des machines virtuelles Azure, telles que des options de prise en charge de quorums ou de routage de connexion.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: conceptual
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.openlocfilehash: 5a2540aeb36cfcb2048ec994bbb486badc8a68d1
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358807"
---
# <a name="cluster-configuration-best-practices-sql-server-on-azure-vms"></a>Meilleures pratiques en matière de configuration de cluster (SQL Server sur des machines virtuelles Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Un cluster est utilisé pour la haute disponibilité et la récupération d’urgence (HADR) avec SQL Server sur des machines virtuelles Azure. 

Cet article présente les meilleures pratiques en matière de configuration de cluster pour les [instances (FCI) de cluster de basculement](failover-cluster-instance-overview.md) et les [groupes de disponibilité](availability-group-overview.md) lorsque vous les utilisez avec SQL Server sur des machines virtuelles Azure. 


## <a name="networking"></a>Mise en réseau

Utilisez une seule carte réseau (NIC) par serveur (nœud de cluster) et un seul sous-réseau. Les réseaux Azure intègrent une redondance physique, ce qui rend inutiles les cartes réseau et les sous-réseaux supplémentaires sur un cluster invité de machine virtuelle Azure. Le rapport de validation du cluster vous avertit que les nœuds sont accessibles uniquement sur un seul réseau. Vous pouvez ignorer cet avertissement sur les clusters de basculement invités de machines virtuelles Azure.

### <a name="tuning-failover-cluster-network-thresholds"></a>Réglage des seuils réseau de cluster de basculement

Quand vous exécutez des nœuds de cluster de basculement Windows dans des machines virtuelles Azure avec SQL Server AlwaysOn, il est recommandé de remplacer le paramètre de cluster par un état d’analyse plus souple.  Cela rend le cluster bien plus stable et fiable.  Pour plus de détails, consultez [IaaS avec SQL AlwaysOn – Réglage des seuils réseau de cluster de basculement](/windows-server/troubleshoot/iaas-sql-failover-cluster).

## <a name="quorum"></a>Quorum

Bien qu’un cluster à deux nœuds puisse fonctionner sans [ressource de quorum](/windows-server/storage/storage-spaces/understand-quorum), les clients sont strictement tenus d’utiliser une ressource de quorum pour prendre en charge la production. La validation du cluster ne transmet pas de cluster sans ressource de quorum. 

Techniquement, un cluster à trois nœuds peut survivre à la perte d’un seul nœud (jusqu’à deux nœuds) sans ressource de quorum. Toutefois, une fois que le cluster est réduit à deux nœuds, il existe un risque que les ressources en cluster soient déconnectées en cas de perte d’un nœud ou d’échec de communication pour éviter un scénario Split-Brain.

La configuration d’une ressource de quorum permet au cluster de continuer en ligne avec un seul nœud en ligne.

Le tableau suivant répertorie les options de quorum disponibles dans l’ordre recommandé pour une machine virtuelle Azure, le témoin de disque étant le choix privilégié : 


||[Témoin de disque](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum)  |[Témoin cloud](/windows-server/failover-clustering/deploy-cloud-witness)  |[Témoin de partage de fichiers](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum)  |
|---------|---------|---------|---------|
|**Systèmes d’exploitation pris en charge**| Tous |Windows Server 2016+| Tous|


### <a name="disk-witness"></a>Témoin de disque

Un témoin de disque est un petit disque en cluster qui se trouve dans le groupe de stockage disponible du cluster. Ce disque est hautement disponible et peut basculer d’un nœud vers un autre. Il contient une copie de la base de données du cluster, dont la taille par défaut est généralement inférieure à 1 Go. Le témoin de disque est l’option de quorum par défaut pour tous les clusters qui utilisent des disques partagés Azure (ou toute solution de disque partagé comme SCSI, iSCSI ou SAN Fiber Channel partagé).  Un volume partagé en cluster ne peut pas être utilisé comme témoin de disque.

Configurer un disque partagé Azure comme témoin de disque. 

Pour commencer, consultez [Configurer un témoin de disque](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum).


**Systèmes d’exploitation pris en charge** : Tous   


### <a name="cloud-witness"></a>Témoin de cloud

Un témoin de cloud est un type de témoin de quorum de cluster de basculement qui utilise Microsoft Azure pour fournir un vote sur le quorum du cluster. La taille par défaut est d’environ 1 Mo et contient uniquement l’horodatage. Un témoin de cloud est idéal pour les déploiements sur plusieurs sites, dans plusieurs zones et plusieurs régions.

Pour commencer, consultez [Configurer un témoin de cloud](/windows-server/failover-clustering/deploy-cloud-witness#CloudWitnessSetUp).


**Systèmes d’exploitation pris en charge** : Windows Server 2016 et versions ultérieures   


### <a name="file-share-witness"></a>Témoin de partage de fichiers

Un témoin de partage de fichiers est un partage de fichiers SMB qui est généralement configuré sur un serveur de fichiers exécutant Windows Server. Il conserve les informations de clustering dans un fichier witness.log, mais ne stocke pas de copie de la base de données de clusters. Dans Azure, vous pouvez configurer un [partage de fichiers Azure](../../../storage/files/storage-how-to-create-file-share.md) à utiliser comme témoin de partage de fichiers, ou vous pouvez utiliser un partage de fichiers sur une machine virtuelle distincte.

Si vous envisagez d’utiliser un partage de fichiers Azure, vous pouvez le monter en suivant le même processus que pour le [montage du partage de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md#mount-premium-file-share). 

Pour commencer, consultez [Configurer un témoin de partage de fichiers](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum).


**Systèmes d’exploitation pris en charge** : Windows Server 2012 et ultérieur   

## <a name="connectivity"></a>Connectivité

Dans un environnement réseau local traditionnel, une instance de cluster de basculement SQL Server apparaît comme une seule instance de SQL Server s’exécutant sur un seul ordinateur. Étant donné que l’instance de cluster de basculement bascule d’un nœud à l’autre, le nom de réseau virtuel (VNN) de l’instance fournit un point de connexion unifié et permet aux applications de se connecter à l’instance de SQL Server sans savoir quel nœud est actuellement actif. Lorsqu'un basculement se produit, le nom du réseau virtuel est inscrit sur le nouveau nœud actif après son démarrage. Ce processus est transparent pour un client ou une application qui se connecte à SQL Server, ce qui permet de réduire les temps d’arrêts pour le client ou l'application lors d'une défaillance. De même, l’écouteur de groupe de disponibilité utilise un VNN pour acheminer le trafic vers le réplica approprié. 

Utilisez un VNN avec Azure Load Balancer ou un nom de réseau distribué (DNN) pour acheminer le trafic vers le VNN de l’instance de cluster de basculement avec SQL Server sur les machines virtuelles Azure ou pour remplacer l’écouteur de VNN existant dans un groupe de disponibilité (AG). 


Le tableau suivant compare la capacité de prise en charge de la connexion HADR : 

| |**Nom de réseau virtuel (VNN)**  |**Nom de réseau distribué (DNN)**  |
|---------|---------|---------|
|**Version de système d’exploitation minimale**| Tous | Windows Server 2016 |
|**Version minimale de SQL Server** |Tous |SQL Server 2019 CU2 (pour FCI)<br/> SQL Server 2019 CU8 (pour AG)|
|**Solution HADR prise en charge** | Instance de cluster de basculement <br/> Groupe de disponibilité | Instance de cluster de basculement <br/> Groupe de disponibilité|


### <a name="virtual-network-name-vnn"></a>Nom de réseau virtuel

Étant donné que le point d’accès de l’adresse IP virtuelle fonctionne différemment dans Azure, vous devez configurer [Azure Load Balancer](../../../load-balancer/index.yml) pour acheminer le trafic vers l’adresse IP des nœuds FCI ou de l’écouteur de groupe de disponibilité. Dans les machines virtuelles Azure, un équilibreur de charge contient l’adresse IP du VNN sur lequel s’appuient les ressources de SQL Server en cluster. L’équilibreur de charge distribue les flux entrants arrivant au frontal, puis achemine ce trafic vers les instances définies par le pool principal. Vous configurez le flux de trafic à l’aide de règles d’équilibrage de charge et de sondes d’intégrité. Avec l’instance FCI de SQL Server, les instances di pool principal sont les machines virtuelles Azure exécutant SQL Server. 

Il y a un léger délai de basculement lorsque vous utilisez l’équilibreur de charge, car la sonde d’intégrité effectue des vérifications actives toutes les 10 secondes par défaut. 

Pour commencer, découvrez comment configurer Azure Load Balancer pour une [instance de cluster de basculement](failover-cluster-instance-vnn-azure-load-balancer-configure.md) ou un [groupe de disponibilité](availability-group-vnn-azure-load-balancer-configure.md).

**Systèmes d’exploitation pris en charge** : Tous   
**Version de SQL pris en charge** : Tous   
**Solution HADR prise en charge** : Instance de cluster de basculement et groupes de disponibilité   


### <a name="distributed-network-name-dnn"></a>Nom de réseau distribué (DNN)

Le nom de réseau distribué est une nouvelle fonctionnalité Azure pour SQL Server 2019. Le DNN permet aux clients SQL Server de se connecter d’une autre façon à l’instance de cluster de basculement ou au groupe de disponibilité SQL Server sans utiliser d’équilibreur de charge. 

Quand une ressource DNN est créée, le cluster lie le nom DNS à l’adresse IP de tous les nœuds du cluster. Le client SQL essaiera de se connecter à chaque adresse IP de cette liste pour trouver la ressource à laquelle se connecter.  Vous pouvez accélérer ce processus en spécifiant `MultiSubnetFailover=True` dans la chaîne de connexion. Ce paramètre indique au fournisseur d’essayer toutes les adresses IP en parallèle, afin que le client puisse se connecter instantanément à la FCI ou à l’écouteur. 

Un nom de réseau distribué est recommandé sur un équilibreur de charge dans la mesure du possible pour les raisons suivantes : 
- La solution de bout en bout est plus robuste, car vous n’avez plus à gérer la ressource d’équilibreur de charge. 
- L’élimination des sondes de l’équilibreur de charge réduit la durée du basculement. 
- Le DNN simplifie l’approvisionnement et la gestion de l’instance de cluster de basculement ou de l’écouteur de groupe de disponibilité avec SQL Server sur des machines virtuelles Azure. 

La plupart des fonctionnalités SQL Server fonctionnent de manière transparente avec les FCI et les groupes de disponibilité lors de l’utilisation du DNN, mais certaines fonctionnalités peuvent nécessiter une attention particulière. Pour en savoir plus, consultez [Interopérabilité FCI et DNN](failover-cluster-instance-dnn-interoperability.md) et [Interopérabilité AG et DNN](availability-group-dnn-interoperability.md). 

Pour commencer, apprenez à configurer une ressource de nom de réseau distribué pour une [instance de cluster de basculement](failover-cluster-instance-distributed-network-name-dnn-configure.md) ou un [groupe de disponibilité](availability-group-distributed-network-name-dnn-listener-configure.md).

**Systèmes d’exploitation pris en charge** : Windows Server 2016 et versions ultérieures   
**Version de SQL pris en charge** : SQL Server 2019 CU2 (FCI) et SQL Server 2019 CU8 (AG)   
**Solution HADR prise en charge** : Instance de cluster de basculement et groupes de disponibilité   


## <a name="limitations"></a>Limites

Tenez compte des limitations suivantes lorsque vous utilisez l’instance FCI ou des groupes de disponibilité et SQL Server sur des machines virtuelles Azure. 

### <a name="msdtc"></a>MSDTC 

La solution Machines virtuelles Azure prend en charge Microsoft Distributed Transaction Coordinator (MSDTC) sur Windows Server 2019, avec un stockage sur les volumes partagés en cluster (CSV) et [Azure Standard Load Balancer](../../../load-balancer/load-balancer-overview.md) ou sur des machines virtuelles SQL Server qui utilisent des disques partagés Azure. 

Concernant Machines virtuelles Azure, MSDTC n’est pas pris en charge pour Windows Server 2016 ou versions antérieures avec des volumes partagés en cluster pour la raison suivante :

- La ressource MSDTC en cluster n’est pas configurable pour utiliser le stockage partagé. Sur Windows Server 2016, si vous créez une ressource MSDTC, celle-ci n’affiche pas le stockage partagé qui est disponible pour l’utilisation, et cela même si le stockage est disponible. Ce problème a été résolu dans Windows Server 2019.
- L’équilibreur de charge de base ne gère pas les ports RPC.


## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez déterminé les meilleures pratiques appropriées pour votre solution, commencez par [préparer votre machine virtuelle SQL Server pour une FCI](failover-cluster-instance-prepare-vm.md) ou par créer votre groupe de disponibilité à l’aide du [portail Azure](availability-group-azure-portal-configure.md), [d’Azure CLI ou de PowerShell](./availability-group-az-commandline-configure.md) ou de [modèles de démarrage rapide Azure](availability-group-quickstart-template-configure.md).