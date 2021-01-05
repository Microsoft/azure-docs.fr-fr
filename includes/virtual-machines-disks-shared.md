---
title: Fichier include
description: Fichier include
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 07/14/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 74c77356df4f35461a8b9f1459712cdcf7f77cbf
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95563638"
---
La fonctionnalité Disques partagés Azure est une nouvelle fonctionnalité pour disques managés Azure qui vous permet de connecter simultanément un disque managé à plusieurs machines virtuelles. Le fait d’attacher un disque managé à plusieurs machines virtuelles vous permet de déployer de nouvelles applications en cluster ou de migrer des applications en cluster existantes vers Azure.

## <a name="how-it-works"></a>Fonctionnement

Les machines virtuelles du cluster peuvent lire ou écrire sur le disque que vous avez attaché en fonction de la réservation choisie par l’application en cluster via les [réservations persistantes SCSI](https://www.t10.org/members/w_spc3.htm) (SCSI PR). SCSI PR est un standard du secteur dont tirent parti les applications qui s’exécutent sur un réseau de zone de stockage (SAN) local. L’activation des réservations persistantes SCSI sur un disque managé vous permet de migrer ces applications vers Azure en l’état.

Le partage de disques managés offre un stockage en bloc partagé accessible à partir de plusieurs machines virtuelles, qui sont exposées sous forme de numéros d’unité logique. Les numéros d’unité logique sont ensuite présentés à un initiateur (machine virtuelle) à partir d’une cible (disque). Pour la machine virtuelle, ces numéros d’unité logiques ressemblent à un stockage en attachement direct (DAS) ou à un lecteur local.

Les disques managés partagés n’offrent pas en mode natif un système de fichiers entièrement géré accessible via SMB/NFS. Vous devez utiliser un gestionnaire de cluster, comme le cluster de basculement Windows Server (WSFC) ou Pacemaker, qui gère la communication des nœuds de cluster ainsi que le verrouillage en écriture.

## <a name="limitations"></a>Limites

[!INCLUDE [virtual-machines-disks-shared-limitations](virtual-machines-disks-shared-limitations.md)]

### <a name="operating-system-requirements"></a>Système d'exploitation requis

Les disques partagés prennent en charge plusieurs systèmes d’exploitation. Consultez les sections [Windows](#windows) ou [Linux](#linux) pour les systèmes d’exploitation pris en charge.

## <a name="disk-sizes"></a>Tailles du disque

[!INCLUDE [virtual-machines-disks-shared-sizes](virtual-machines-disks-shared-sizes.md)]

## <a name="sample-workloads"></a>Exemples de charges de travail

### <a name="windows"></a>Windows

Les disques partagés Azure sont pris en charge sur Windows Server 2008 et versions ultérieures. Le clustering basé sur Windows repose essentiellement sur WSFC, qui gère toute l’infrastructure principale pour la communication des nœuds de cluster, ce qui permet à vos applications de tirer parti des modèles d’accès parallèle. WSFC active les options CSV et non CSV en fonction de votre version de Windows Server. Pour plus d’informations, consultez [Créer un cluster de basculement](/windows-server/failover-clustering/create-failover-cluster).

Voici quelques exemples d’applications bien connues qui s’exécutent sur WSFC :

- [Création d’une instance FCI avec des disques partagés Azure (SQL Server sur les machines virtuelles Azure)](../articles/azure-sql/virtual-machines/windows/failover-cluster-instance-azure-shared-disks-manually-configure.md)
- Serveur de fichiers Scale-out (SoFS) [modèle] (https://aka.ms/azure-shared-disk-sofs-template)
- SAP ASCS/SCS [modèle] (https://aka.ms/azure-shared-disk-sapacs-template)
- Serveur de fichiers pour une utilisation générale (charge de travail IW)
- Disque de profil utilisateur de serveur Bureau à distance (RDS UPD)

### <a name="linux"></a>Linux

Les disques partagés Azure sont pris en charge sur :
- [SUSE SLE pour SAP et SUSE SLE HA 15 SP1 et versions ultérieures](https://www.suse.com/c/azure-shared-disks-excercise-w-sles-for-sap-or-sle-ha/)
- [Ubuntu 18.04 et ultérieures](https://discourse.ubuntu.com/t/ubuntu-high-availability-corosync-pacemaker-shared-disk-environments/14874)
- [Version préliminaire pour les développeurs RHEL sur n'importe quelle version de RHEL 8](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/deploying_red_hat_enterprise_linux_8_on_public_cloud_platforms/index?lb_target=production#azure-configuring-shared-block-storage_configuring-rhel-high-availability-on-azure)
- [Oracle Enterprise Linux](https://docs.oracle.com/en/operating-systems/oracle-linux/8/availability/hacluster-1.html)

Les clusters Linux peuvent tirer parti de gestionnaires de cluster comme [Pacemaker](https://wiki.clusterlabs.org/wiki/Pacemaker). Pacemaker repose sur [Corosync](http://corosync.github.io/corosync/), ce qui autorise la communication de cluster pour les applications déployées dans des environnements hautement disponibles. Parmi les systèmes de fichiers en cluster courants figurent [ocfs2](https://oss.oracle.com/projects/ocfs2/) et [gfs2](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/global_file_system_2/ch-overview-gfs2). Vous pouvez utiliser les modèles de clustering basés sur la réservation persistante SCSI (SCSI PR) et/ou le dispositif de blocage STONITH (SBD) pour arbitrer l'accès au disque. Vous pouvez manipuler les réservations et les inscriptions à l’aide des utilitaires que sont [fence_scsi](http://manpages.ubuntu.com/manpages/eoan/man8/fence_scsi.8.html) et [sg_persist](https://linux.die.net/man/8/sg_persist) lors de l’utilisation de SCSI PR.

## <a name="persistent-reservation-flow"></a>Flux de réservation persistante

Le diagramme suivant illustre un exemple d’application de base de données en cluster à deux nœuds qui se sert de SCSI PR pour activer le basculement d’un nœud vers l’autre.

![Cluster à deux nœuds. Une application s’exécutant sur le cluster gère l’accès au disque](media/virtual-machines-disks-shared-disks/shared-disk-updated-two-node-cluster-diagram.png)

Voici comment se déroule l’opération :

1. L’application en cluster s’exécutant sur Azure VM1 et Azure VM2 inscrit son intention de lire ou d’écrire sur le disque.
1. L’instance d’application sur VM1 prend ensuite une réservation exclusive pour écrire sur le disque.
1. Cette réservation est appliquée sur votre disque Azure et la base de données peut dès lors écrire exclusivement sur le disque. Les écritures éventuelles provenant de l’instance d’application sur VM2 échouent.
1. Si l’instance d’application sur VM1 cesse de fonctionner, l’instance sur VM2 peut alors lancer un basculement de base de données et prendre le contrôle du disque.
1. Cette réservation est dès lors appliquée sur le disque Azure et celui-ci n’accepte plus les écritures en provenance de VM1. Il accepte uniquement les écritures de VM2.
1. L’application en cluster peut mener à bien le basculement de base de données et traiter les demandes de VM2.

Le diagramme suivant illustre une autre charge de travail en cluster courante dans laquelle plusieurs nœuds lisent les données du disque de façon pour des processus parallèles en cours d’exécution, comme l’entraînement de modèles Machine Learning.

![Cluster de machines virtuelles à quatre nœuds ; chaque nœud inscrit une intention d’écrire ; l’application prend une réservation exclusive pour gérer correctement les résultats d’écriture](media/virtual-machines-disks-shared-disks/shared-disk-updated-machine-learning-trainer-model.png)

Voici comment se déroule l’opération :

1. L’application en cluster s’exécutant sur toutes les machines virtuelles inscrit l’intention de lire ou d’écrire sur le disque.
1. L’instance d’application sur VM1 prend une réservation exclusive pour écrire sur le disque tout en permettant aux autres machines virtuelles de lire sur le disque.
1. Cette réservation est appliquée sur votre disque Azure.
1. Tous les nœuds du cluster peuvent dès lors lire sur le disque. Un seul nœud écrit en retour les résultats sur le disque pour le compte de tous les nœuds du cluster.

### <a name="ultra-disks-reservation-flow"></a>Flux de réservation de disques Ultra

Les disques Ultra offrent une limitation supplémentaire, soit deux limitations en tout. Pour cette raison, le flux de réservation des disques Ultra peut fonctionner comme décrit dans la section précédente, ou il peut limiter et distribuer les performances de manière plus précise.

:::image type="content" source="media/virtual-machines-disks-shared-disks/ultra-reservation-table.png" alt-text="Image d’une table représentant l’accès en lecture seule ou en lecture/écriture pour le détenteur de la réservation, l’utilisateur inscrit et autres.":::

## <a name="performance-throttles"></a>Limitation des performances

### <a name="premium-ssd-performance-throttles"></a>Limitations des performances des disques SSD Premium

Sur un disque SSD Premium, les IOPS et le débit sont fixes. Par exemple, un P30 présente 5000 IOPS. Cette valeur reste la même que le disque soit partagé entre deux ou cinq machines virtuelles. Les limites de disque peuvent être atteintes à partir d’une seule machine virtuelle ou réparties sur plusieurs machines virtuelles. 

### <a name="ultra-disk-performance-throttles"></a>Limitations des performances des disques Ultra

Les disques Ultra ont la capacité unique de vous permettre de définir vos performances en exposant des attributs modifiables et en vous permettant de les modifier. Par défaut, il n’y a que deux attributs modifiables, mais les disques Ultra partagés ont deux attributs supplémentaires.


|Attribut  |Description  |
|---------|---------|
|DiskIOPSReadWrite     |Nombre total d’opérations d’IOPS autorisées sur toutes les machines virtuelles montant le disque partagé avec accès en écriture.         |
|DiskMBpsReadWrite     |Débit total (Mo/s) autorisé sur toutes les machines virtuelles montant le disque partagé avec accès en écriture.         |
|DiskIOPSReadOnly*     |Nombre total d’opérations d’IOPS autorisées sur toutes les machines virtuelles montant le disque partagé `ReadOnly`.         |
|DiskMBpsReadOnly*     |Débit total (Mo/s) autorisé sur toutes les machines virtuelles montant le disque partagé `ReadOnly`.         |

\* S’applique uniquement aux disques Ultra partagés

Les formules suivantes expliquent comment définir les attributs de performance, car ils peuvent être modifiés par l’utilisateur :

- DiskIOPSReadWrite/DiskIOPSReadOnly : 
    - Limites d’IOPS de 300 IOPS/Gio, avec un maximum de 160 000 IOPS par disque
    - Minimum de 100 IOPS
    - DiskIOPSReadWrite + DiskIOPSReadOnly est au moins 2 IOPS/Gio
- DiskMBpsRead    Write/DiskMBpsReadOnly :
    - La limite de débit d’un seul disque est de 256 Kio/s pour chaque IOPS provisionnée, avec un maximum de 2000 Mbits/s par disque.
    - Le débit minimal garanti par disque est de 4 Kio/s pour chaque IOPS provisionnée avec une base de référence globale minimale de 1 Mbits/s.

#### <a name="examples"></a>Exemples

Les exemples suivants illustrent quelques scénarios qui montrent comment la limitation peut fonctionner notamment avec des disques Ultra partagés.

##### <a name="two-nodes-cluster-using-cluster-shared-volumes"></a>Cluster à deux nœuds utilisant des volumes partagés de cluster

Voici un exemple de cluster WSFC à 2 nœuds utilisant des volumes partagés en cluster. Dans cette configuration, les deux machines virtuelles disposent d’un accès en écriture simultané au disque, ce qui entraîne la répartition de la limitation `ReadWrite` entre les deux machines virtuelles et la non-utilisation de la limitation `ReadOnly`.

:::image type="content" source="media/virtual-machines-disks-shared-disks/ultra-two-node-example.png" alt-text="Exemple de disque Ultra à deux nœuds utilisant des volumes partagés de cluster":::

##### <a name="two-node-cluster-without-cluster-share-volumes"></a>Cluster à deux nœuds sans volumes de partage de cluster

Voici un exemple de cluster WSFC à 2 nœuds qui n’utilise pas de volumes partagés en cluster. Dans cette configuration, une seule machine virtuelle dispose d’un accès en écriture au disque. Cela entraîne l’utilisation exclusive de la limitation `ReadWrite` pour la machine virtuelle principale et l’utilisation de la limitation `ReadOnly` uniquement par la machine virtuelle secondaire.

:::image type="content" source="media/virtual-machines-disks-shared-disks/ultra-two-node-no-csv.png" alt-text="Exemple de disque Ultra à deux nœuds sans volumes partagés de cluster":::

##### <a name="four-node-linux-cluster"></a>Cluster Linux à quatre nœuds

Voici un exemple de cluster Linux à quatre nœuds avec un seul enregistreur et trois lecteurs en parallèle. Dans cette configuration, une seule machine virtuelle dispose d’un accès en écriture au disque. Cela entraîne `ReadWrite` l’utilisation exclusive de la limitation pour la machine virtuelle principale et l’utilisation de la limitation `ReadOnly` uniquement par la machine virtuelle secondaire.

:::image type="content" source="media/virtual-machines-disks-shared-disks/ultra-four-node-example.png" alt-text="Exemple de limitation dans le cas d’un disque Ultra à quatre nœuds":::

#### <a name="ultra-pricing"></a>Tarifs Ultra

Les disques partagés Ultra sont facturés en fonction de la capacité approvisionnée, du nombre total d’IOPS approvisionnées (diskIOPSReadWrite + diskIOPSReadOnly) et du débit total approvisionné Mbits/s (diskMBpsReadWrite + diskMBpsReadOnly). Il n’y a pas de frais additionnels pour chaque montage de machine virtuelle supplémentaire. Par exemple, un disque ultra partagé avec la configuration suivante (diskSizeGB : 1 024, DiskIOPSReadWrite : 10 000, DiskMBpsReadWrite : 600, DiskIOPSReadOnly : 100, DiskMBpsReadOnly : 1) est facturé avec 1 024 Gio, 10 100 IOPS et 601 Mbits/s, qu’il soit monté sur deux machines virtuelles ou sur cinq machines virtuelles.