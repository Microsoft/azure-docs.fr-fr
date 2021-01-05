---
title: Mettre en cluster une instance SAP ASCS/SCS sur WSFC à l'aide d'un disque partagé dans Azure | Microsoft Docs
description: Découvrez comment mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: f6fb85f8-c77a-4af1-bde8-1de7e4425d2e
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: e46aa79bc84f3eba218932c1e8a463584de3f1bb
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488950"
---
# <a name="cluster-an-sap-ascsscs-instance-on-a-windows-failover-cluster-by-using-a-cluster-shared-disk-in-azure"></a>Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster dans Azure

> ![Système d’exploitation Windows][Logo_Windows] Windows
>

Le clustering de basculement Windows Server constitue la base d’une installation de SGBD et de SAP ASCS/SCS à haute disponibilité dans Windows.

Un cluster de basculement est un groupe de 1 + n serveurs indépendants (nœuds) qui fonctionnent ensemble pour accroître la disponibilité des applications et des services. En cas d’échec d’un nœud, le clustering de basculement Windows Server calcule le nombre d’échecs qui peuvent se produire sans que le cluster ne perde son intégrité, de sorte que les applications et les services puissent être fournis. Différents modes de quorum sont disponibles pour obtenir un clustering de basculement.

## <a name="prerequisites"></a>Prérequis
Avant d’aborder les tâches décrites dans cet article, consultez l’article suivant :

* [Scénarios et architecture de haute disponibilité de machines virtuelles Azure pour SAP NetWeaver][sap-high-availability-architecture-scenarios]


## <a name="windows-server-failover-clustering-in-azure"></a>Clustering de basculement Windows Server dans Azure

Le clustering de basculement Windows Server avec Machines virtuelles Azure implique des étapes de configuration supplémentaires. Quand vous créez un cluster, vous devez définir plusieurs adresses IP et noms d’hôtes virtuels pour l’instance SAP ASCS/SCS.

### <a name="name-resolution-in-azure-and-the-cluster-virtual-host-name"></a>Résolution de noms dans Azure et nom d’hôte virtuel du cluster

La plateforme cloud Azure ne permet pas de configurer des adresses IP virtuelles telles que des adresses IP flottantes. Vous avez besoin d’une autre solution pour configurer une adresse IP virtuelle afin d’atteindre la ressource de cluster dans le cloud. 

Le service Azure Load Balancer fournit un *équilibreur de charge interne* pour Azure. Avec l’équilibrage de charge interne, les clients atteignent le cluster via son adresse IP virtuelle. 

Déployez l’équilibreur de charge interne dans le groupe de ressources qui contient les nœuds de cluster. Ensuite, configurez toutes les règles de réacheminement de port nécessaires en utilisant les ports de sondage de l’équilibreur de charge interne. Les clients peuvent se connecter avec le nom d’hôte virtuel. Le serveur DNS résout l’adresse IP du cluster et l’équilibrage de charge interne gère le réacheminement de port vers le nœud actif du cluster.

> [!IMPORTANT]
> Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.  

![Figure 1 : Configuration du clustering de basculement Windows dans Azure sans disque partagé][sap-ha-guide-figure-1001]

_Configuration du clustering de basculement Windows Server dans Azure sans disque partagé_

### <a name="sap-ascsscs-ha-with-cluster-shared-disks"></a>Haute disponibilité SAP ASCS/SCS avec des disques partagés de cluster
Dans Windows, une instance SAP ASCS/SCS contient des services SAP centraux, le serveur de messages SAP, des processus de serveur de mise en file d’attente et des fichiers d’hôte global SAP. Les fichiers d’hôte global SAP stockent les fichiers centraux pour l’ensemble du système SAP.

Une instance SAP ASCS/SCS inclut les composants suivants :

* Services centraux SAP :
    * Deux processus, un serveur de messages et de mise en file d’attente, et un \<ASCS/SCS virtual host name> utilisé pour accéder à ces deux processus.
    * Structure de fichiers : S:\usr\sap\\&lt;SID&gt;\ASCS/SCS\<instance number\>


* Fichiers d’hôte global SAP :
  * Structure de fichiers : S:\usr\sap\\&lt;SID&gt;\SYS\..
  * Partage de fichiers sapmnt, qui autorise l’accès à ces fichiers S:\usr\sap\\&lt;SID&gt;\SYS\... à l’aide du chemin UNC suivant :

    \\\\<Nom d’hôte virtuel ASCS/SCS\>\sapmnt\\&lt;SID&gt;\SYS\..


![Figure 2 : Processus, structure de fichiers et partage de fichiers sapmnt d’hôte global d’une instance SAP ASCS/SCS][sap-ha-guide-figure-8001]

_Processus, structure de fichiers et partage de fichiers sapmnt d’hôte global d’une instance SAP ASCS/SCS_

Dans un paramètre de haute disponibilité, vous mettez en cluster les instances SAP ASCS/SCS. Nous utilisons des *disques partagés de cluster* (lecteur S dans notre exemple) pour placer les fichiers SAP ASCS/SCS et d’hôte global SAP.

![Figure 3 : Architecture de haute disponibilité (HA) SAP ASCS/SCS avec disque partagé][sap-ha-guide-figure-8002]

_Architecture de haute disponibilité (HA) SAP ASCS/SCS avec disque partagé_


Avec l’architecture ERS1 (Enqueue Replication Server 1) :
* Le même \<ASCS/SCS virtual host name> est utilisé pour accéder aux processus de serveur de mise en file d’attente et de messages SAP, ainsi qu’aux fichiers d’hôte global SAP par le biais du partage de fichiers sapmnt.
* Ils partagent le même lecteur de disque partagé de cluster S.  

Avec l’architecture ERS2 (Enqueue Replication Server 2) : 
* Le \<ASCS/SCS virtual host name> utilisé pour accéder au processus de serveur de messages SAP et aux fichiers d’hôte global SAP par le biais du partage de fichiers sapmnt est le même.
* Ils partagent le même lecteur de disque partagé de cluster S.
* Il existe un \<ERS virtual host name> distinct servant à accéder au processus du serveur d’empilement.  

![Figure 4 : Architecture de haute disponibilité (HA) SAP ASCS/SCS avec disque partagé][sap-ha-guide-figure-8003]

_Architecture de haute disponibilité (HA) SAP ASCS/SCS avec disque partagé_

#### <a name="shared-disk-and-enqueue-replication-server"></a>Disque partagé et architecture ERS 

1. Le disque partagé est pris en charge avec l’architecture ERS1, où l’instance ERS respecte les conditions suivantes :   

   - Il ne s’agit pas d’une instance en cluster.
   - L’instance utilise le nom `localhost`.
   - L’instance est déployée sur des disques locaux sur chacun des nœuds de cluster.

2. Le disque partagé est également pris en charge avec l’architecture ERS2, où l’instance ERS2 respecte les conditions suivantes :  

   - Il s’agit d’une instance en cluster.
   - L’instance utilise un nom d’hôte virtuel/réseau dédié.
   - L’adresse IP du nom d’hôte virtuel ERS est configurée sur l’équilibreur de charge interne Azure, en plus de l’adresse IP (A)SCS.
   - L’instance est déployée sur des **disques locaux** sur chacun des nœuds de cluster. Il n’est donc pas nécessaire de disposer d’un disque partagé.

   > [!TIP]
   > Vous trouverez ici des informations complémentaires sur les architectures ERS1 et ERS2 :  
   > [Serveur de réplication d’empilement dans un cluster de basculement Microsoft](https://help.sap.com/viewer/3741bfe0345f4892ae190ee7cfc53d4c/CURRENT_VERSION_SWPM20/en-US/8abd4b52902d4b17a105c2fabdf5c0cf.html)  
   > [Nouveau réplicateur d’empilement dans les environnements de cluster de basculement](https://blogs.sap.com/2019/03/19/new-enqueue-replicator-in-failover-cluster-environments/)  

#### <a name="options-for-shared-disk-in-azure-for-sap-workloads"></a>Options de disque partagé dans Azure pour les charges de travail SAP

Il existe deux options pour le disque partagé dans un cluster de basculement Windows dans Azure :

- La fonctionnalité de [disques partagés Azure](../../disks-shared.md), qui permet d’attacher simultanément un disque managé Azure à plusieurs machines virtuelles. 
- Le logiciel tiers [SIOS DataKeeper Cluster Edition](https://us.sios.com/products/datakeeper-cluster) pour créer un stockage en miroir qui simule un stockage partagé de cluster. 

Lorsque vous sélectionnez une technologie de disque partagé, gardez à l’esprit les points suivants :

**Disque partagé Azure pour les charges de travail SAP**
- Cette fonctionnalité permet d’attacher simultanément un disque managé Azure à plusieurs machines virtuelles sans logiciels supplémentaires à gérer ni à utiliser. 
- Cette fonctionnalité implique d’utiliser un seul disque partagé Azure au sein d’un cluster de stockage, ce qui a une incidence sur la fiabilité de votre solution SAP.
- Actuellement, seuls sont pris en charge les déploiements avec disque partagé Premium Azure dans un groupe à haute disponibilité. Le disque partagé Azure n’est pas pris en charge dans le déploiement zonal.     
- Veillez à approvisionner le disque Azure Premium avec une taille de disque minimale (cf. [Plages SSD Premium](../../disks-shared.md#disk-sizes)) pour pouvoir l’attacher simultanément au nombre nécessaire de machines virtuelles (généralement, deux pour le cluster de basculement Windows SAP ASCS). 
- Le disque Ultra partagé Azure n’est pas pris en charge pour les charges de travail SAP, car il ne gère ni le déploiement dans un groupe à haute disponibilité ni le déploiement zonal.  
 
**SIOS**
- La solution SIOS assure la réplication synchrone et en temps réel des données entre deux disques.
- La solution SIOS implique de travailler avec deux disques managés. Or, si vous utilisez des groupes à haute disponibilité ou des zones de disponibilité, ces disques s’afficheront sur des clusters de stockage différents. 
- Le déploiement dans les zones de disponibilité est pris en charge.
- Des logiciels tiers doivent être achetés en plus, installés et utilisés.

### <a name="shared-disk-using-azure-shared-disk"></a>Disque partagé à l’aide d’un disque partagé Azure

Microsoft propose des [disques partagés Azure](../../disks-shared.md), qui peuvent être utilisés pour implémenter la haute disponibilité SAP ASCS/SCS avec une option de disque partagé.

#### <a name="prerequisites-and-limitations"></a>Conditions préalables et limitations

Il est actuellement possible d’utiliser des disques SSD Premium Azure comme disque partagé Azure pour l'instance SAP ASCS/SCS. Quelques limitations s'appliquent pour le moment :

-  Le [disque Ultra Azure](../../disks-types.md#ultra-disk) n'est pas pris en charge comme disque partagé Azure pour les charges de travail SAP. Il est actuellement impossible de placer des machines virtuelles Azure dans un groupe à haute disponibilité à l'aide d'un disque Ultra Azure.
-  Le [disque partagé Azure](../../disks-shared.md) avec disques SSD Premium n'est pris en charge qu'avec les machines virtuelles d'un groupe à haute disponibilité. Il n'est pas pris en charge dans le cadre d'un déploiement de zones de disponibilité. 
-  La valeur [maxShares](../../disks-shared-enable.md?tabs=azure-cli#disk-sizes) du disque partagé Azure détermine combien de nœuds de cluster peuvent utiliser le disque partagé. En règle générale, pour une instance SAP ASCS/SCS, on configure deux nœuds dans le cluster de basculement Windows. Par conséquent, la valeur de `maxShares` doit être définie sur deux.
-  Toutes les machines virtuelles en cluster SAP ASCS/SCS doivent être déployées dans le même [groupe de placement de proximité Azure](../../windows/proximity-placement-groups.md).   
   Bien qu'il soit possible de déployer des machines virtuelles en cluster Windows dans un groupe à haute disponibilité avec un disque partagé Azure sans groupe de placement de proximité, le groupe de placement de proximité garantira une proximité physique étroite des disques partagés Azure et des machines virtuelles en cluster, réduisant ainsi la latence entre les machines virtuelles et la couche de stockage.    

Pour plus d’informations sur les limitations du disque partagé Azure, lisez attentivement la section [Limitations](../../disks-shared.md#limitations) de la documentation consacrée au disque partagé Azure.

> [!IMPORTANT]
> Lors du déploiement d'un cluster de basculement Windows SAP ASCS/SCS avec disque partagé Azure, sachez que votre déploiement fonctionnera avec un seul disque partagé au sein d'un cluster de stockage. Votre instance SAP ASCS/SCS serait affectée en cas de problème au niveau du cluster de stockage dans lequel le disque partagé Azure est déployé.    

> [!TIP]
> Pour connaître les points importants à prendre en compte lors de la planification de votre déploiement SAP, consultez le [Guide de planification SAP NetWeaver sur Azure](./planning-guide.md) et le [Guide de stockage Azure pour les charges de travail SAP](./planning-guide-storage.md).

### <a name="supported-os-versions"></a>Versions du système d'exploitation prises en charge

Windows Server 2016 et 2019 sont tous deux pris en charge (utilisez les dernières images du centre de données).

Nous vous recommandons vivement d'utiliser **Windows Server 2019 Datacenter** pour plusieurs raisons :
- Le service de cluster de basculement Windows 2019 est compatible avec Azure.
- L'intégration et la compatibilité de la maintenance de l'hôte Azure sont renforcées, et l'expérience est améliorée grâce à la supervision des événements de planification Azure.
- Il est possible d'utiliser le nom de réseau distribué (option par défaut). Il n'est donc pas nécessaire de disposer d’une adresse IP dédiée pour le nom réseau du cluster. En outre, il n'y a pas lieu de configurer cette adresse IP sur l'équilibreur de charge interne Azure. 

### <a name="shared-disks-in-azure-with-sios-datakeeper"></a>Disques partagés dans Azure avec SIOS DataKeeper

Autre solution pour le disque partagé, vous pouvez utiliser le logiciel tiers SIOS DataKeeper Cluster Edition pour créer un stockage en miroir qui simule un stockage partagé en cluster. La solution SIOS assure la réplication synchrone des données en temps réel.

Pour créer une ressource de disque partagé pour un cluster :

1. Attachez un disque supplémentaire à chaque machine virtuelle se trouvant dans une configuration de cluster Windows.
2. Exécutez SIOS DataKeeper Cluster Edition sur les deux nœuds de machine virtuelle.
3. Configurez SIOS DataKeeper Cluster Edition de sorte qu’il mette en miroir le contenu du volume attaché au disque supplémentaire de la machine virtuelle source vers le volume attaché au disque supplémentaire de la machine virtuelle cible. SIOS DataKeeper extrait les volumes locaux source et cible, puis les présente au clustering de basculement Windows Server comme un seul disque partagé.

Vous trouverez plus d’informations sur SIOS DataKeeper [ici](https://us.sios.com/products/datakeeper-cluster/).

![Figure 5 : Configuration du clustering de basculement Windows Server dans Azure avec SIOS DataKeeper][sap-ha-guide-figure-1002]

_Configuration du clustering de basculement Windows dans Azure avec SIOS DataKeeper_

> [!NOTE]
> Avec certains SGBD, tels que SQL Server, vous n’avez pas besoin de disques partagés pour atteindre une haute disponibilité. SQL Server AlwaysOn assure la réplication des données et des fichiers de journaux du SGBD à partir du disque local d’un nœud du cluster vers le disque local d’un autre nœud du cluster. Dans ce cas, la configuration du cluster Windows ne nécessite pas de disque partagé.
>

## <a name="next-steps"></a>Étapes suivantes

* [Préparer l’infrastructure Azure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows et d’un disque partagé pour une instance SAP ASCS/SCS][sap-high-availability-infrastructure-wsfc-shared-disk]

* [Installer la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un disque partagé pour une instance SAP ASCS/SCS][sap-high-availability-installation-wsfc-shared-disk]


[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-ha-guide]:sap-high-availability-guide.md
[sap-ha-guide-2]:#42b8f600-7ba3-4606-b8a5-53c4f026da08
[sap-ha-guide-4]:#8ecf3ba0-67c0-4495-9c14-feec1a2255b7
[sap-ha-guide-8]:#78092dbe-165b-454c-92f5-4972bdbef9bf
[sap-ha-guide-8.1]:#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-ha-guide-8.9]:#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-ha-guide-8.12]:#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-ha-guide-8.12.1]:#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-ha-guide-8.12.3]:#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-ha-guide-8.12.3.1]:#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-ha-guide-8.12.3.2]:#dd41d5a2-8083-415b-9878-839652812102
[sap-ha-guide-8.12.3.3]:#d9c1fc8e-8710-4dff-bec2-1f535db7b006
[sap-ha-guide-9]:#a06f0b49-8a7a-42bf-8b0d-c12026c5746b
[sap-ha-guide-9.1]:#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-ha-guide-9.1.1]:#a97ad604-9094-44fe-a364-f89cb39bf097

[sap-ha-multi-sid-guide]:sap-high-availability-multi-sid.md (Configuration de haute disponibilité multi-SID SAP)

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[sap-ha-guide-figure-1000]:./media/virtual-machines-shared-sap-high-availability-guide/1000-wsfc-for-sap-ascs-on-azure.png
[sap-ha-guide-figure-1001]:./media/virtual-machines-shared-sap-high-availability-guide/1001-wsfc-on-azure-ilb.png
[sap-ha-guide-figure-1002]:./media/virtual-machines-shared-sap-high-availability-guide/1002-wsfc-sios-on-azure-ilb.png
[sap-ha-guide-figure-2000]:./media/virtual-machines-shared-sap-high-availability-guide/2000-wsfc-sap-as-ha-on-azure.png
[sap-ha-guide-figure-2001]:./media/virtual-machines-shared-sap-high-availability-guide/2001-wsfc-sap-ascs-ha-on-azure.png
[sap-ha-guide-figure-2003]:./media/virtual-machines-shared-sap-high-availability-guide/2003-wsfc-sap-dbms-ha-on-azure.png
[sap-ha-guide-figure-2004]:./media/virtual-machines-shared-sap-high-availability-guide/2004-wsfc-sap-ha-e2e-archit-template1-on-azure.png
[sap-ha-guide-figure-2005]:./media/virtual-machines-shared-sap-high-availability-guide/2005-wsfc-sap-ha-e2e-arch-template2-on-azure.png

[sap-ha-guide-figure-3000]:./media/virtual-machines-shared-sap-high-availability-guide/3000-template-parameters-sap-ha-arm-on-azure.png
[sap-ha-guide-figure-3001]:./media/virtual-machines-shared-sap-high-availability-guide/3001-configuring-dns-servers-for-Azure-vnet.png
[sap-ha-guide-figure-3002]:./media/virtual-machines-shared-sap-high-availability-guide/3002-configuring-static-IP-address-for-network-card-of-each-vm.png
[sap-ha-guide-figure-3003]:./media/virtual-machines-shared-sap-high-availability-guide/3003-setup-static-ip-address-ilb-for-ascs-instance.png
[sap-ha-guide-figure-3004]:./media/virtual-machines-shared-sap-high-availability-guide/3004-default-ascs-scs-ilb-balancing-rules-for-azure-ilb.png
[sap-ha-guide-figure-3005]:./media/virtual-machines-shared-sap-high-availability-guide/3005-changing-ascs-scs-default-ilb-rules-for-azure-ilb.png
[sap-ha-guide-figure-3006]:./media/virtual-machines-shared-sap-high-availability-guide/3006-adding-vm-to-domain.png
[sap-ha-guide-figure-3007]:./media/virtual-machines-shared-sap-high-availability-guide/3007-config-wsfc-1.png
[sap-ha-guide-figure-3008]:./media/virtual-machines-shared-sap-high-availability-guide/3008-config-wsfc-2.png
[sap-ha-guide-figure-3009]:./media/virtual-machines-shared-sap-high-availability-guide/3009-config-wsfc-3.png
[sap-ha-guide-figure-3010]:./media/virtual-machines-shared-sap-high-availability-guide/3010-config-wsfc-4.png
[sap-ha-guide-figure-3011]:./media/virtual-machines-shared-sap-high-availability-guide/3011-config-wsfc-5.png
[sap-ha-guide-figure-3012]:./media/virtual-machines-shared-sap-high-availability-guide/3012-config-wsfc-6.png
[sap-ha-guide-figure-3013]:./media/virtual-machines-shared-sap-high-availability-guide/3013-config-wsfc-7.png
[sap-ha-guide-figure-3014]:./media/virtual-machines-shared-sap-high-availability-guide/3014-config-wsfc-8.png
[sap-ha-guide-figure-3015]:./media/virtual-machines-shared-sap-high-availability-guide/3015-config-wsfc-9.png
[sap-ha-guide-figure-3016]:./media/virtual-machines-shared-sap-high-availability-guide/3016-config-wsfc-10.png
[sap-ha-guide-figure-3017]:./media/virtual-machines-shared-sap-high-availability-guide/3017-config-wsfc-11.png
[sap-ha-guide-figure-3018]:./media/virtual-machines-shared-sap-high-availability-guide/3018-config-wsfc-12.png
[sap-ha-guide-figure-3019]:./media/virtual-machines-shared-sap-high-availability-guide/3019-assign-permissions-on-share-for-cluster-name-object.png
[sap-ha-guide-figure-3020]:./media/virtual-machines-shared-sap-high-availability-guide/3020-change-object-type-include-computer-objects.png
[sap-ha-guide-figure-3021]:./media/virtual-machines-shared-sap-high-availability-guide/3021-check-box-for-computer-objects.png
[sap-ha-guide-figure-3022]:./media/virtual-machines-shared-sap-high-availability-guide/3022-set-security-attributes-for-cluster-name-object-on-file-share-quorum.png
[sap-ha-guide-figure-3023]:./media/virtual-machines-shared-sap-high-availability-guide/3023-call-configure-cluster-quorum-setting-wizard.png
[sap-ha-guide-figure-3024]:./media/virtual-machines-shared-sap-high-availability-guide/3024-selection-screen-different-quorum-configurations.png
[sap-ha-guide-figure-3025]:./media/virtual-machines-shared-sap-high-availability-guide/3025-selection-screen-file-share-witness.png
[sap-ha-guide-figure-3026]:./media/virtual-machines-shared-sap-high-availability-guide/3026-define-file-share-location-for-witness-share.png
[sap-ha-guide-figure-3027]:./media/virtual-machines-shared-sap-high-availability-guide/3027-successful-reconfiguration-cluster-file-share-witness.png
[sap-ha-guide-figure-3028]:./media/virtual-machines-shared-sap-high-availability-guide/3028-install-dot-net-framework-35.png
[sap-ha-guide-figure-3029]:./media/virtual-machines-shared-sap-high-availability-guide/3029-install-dot-net-framework-35-progress.png
[sap-ha-guide-figure-3030]:./media/virtual-machines-shared-sap-high-availability-guide/3030-sios-installer.png
[sap-ha-guide-figure-3031]:./media/virtual-machines-shared-sap-high-availability-guide/3031-first-screen-sios-data-keeper-installation.png
[sap-ha-guide-figure-3032]:./media/virtual-machines-shared-sap-high-availability-guide/3032-data-keeper-informs-service-be-disabled.png
[sap-ha-guide-figure-3033]:./media/virtual-machines-shared-sap-high-availability-guide/3033-user-selection-sios-data-keeper.png
[sap-ha-guide-figure-3034]:./media/virtual-machines-shared-sap-high-availability-guide/3034-domain-user-sios-data-keeper.png
[sap-ha-guide-figure-3035]:./media/virtual-machines-shared-sap-high-availability-guide/3035-provide-sios-data-keeper-license.png
[sap-ha-guide-figure-3036]:./media/virtual-machines-shared-sap-high-availability-guide/3036-data-keeper-management-config-tool.png
[sap-ha-guide-figure-3037]:./media/virtual-machines-shared-sap-high-availability-guide/3037-tcp-ip-address-first-node-data-keeper.png
[sap-ha-guide-figure-3038]:./media/virtual-machines-shared-sap-high-availability-guide/3038-create-replication-sios-job.png
[sap-ha-guide-figure-3039]:./media/virtual-machines-shared-sap-high-availability-guide/3039-define-sios-replication-job-name.png
[sap-ha-guide-figure-3040]:./media/virtual-machines-shared-sap-high-availability-guide/3040-define-sios-source-node.png
[sap-ha-guide-figure-3041]:./media/virtual-machines-shared-sap-high-availability-guide/3041-define-sios-target-node.png
[sap-ha-guide-figure-3042]:./media/virtual-machines-shared-sap-high-availability-guide/3042-define-sios-synchronous-replication.png
[sap-ha-guide-figure-3043]:./media/virtual-machines-shared-sap-high-availability-guide/3043-enable-sios-replicated-volume-as-cluster-volume.png
[sap-ha-guide-figure-3044]:./media/virtual-machines-shared-sap-high-availability-guide/3044-data-keeper-synchronous-mirroring-for-SAP-gui.png
[sap-ha-guide-figure-3045]:./media/virtual-machines-shared-sap-high-availability-guide/3045-replicated-disk-by-data-keeper-in-wsfc.png
[sap-ha-guide-figure-3046]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ascs-virtual-name-ip.png
[sap-ha-guide-figure-3047]:./media/virtual-machines-shared-sap-high-availability-guide/3047-dns-manager.png
[sap-ha-guide-figure-3048]:./media/virtual-machines-shared-sap-high-availability-guide/3048-default-cluster-probe-port.png
[sap-ha-guide-figure-3049]:./media/virtual-machines-shared-sap-high-availability-guide/3049-cluster-probe-port-after.png
[sap-ha-guide-figure-3050]:./media/virtual-machines-shared-sap-high-availability-guide/3050-service-type-ers-delayed-automatic.png
[sap-ha-guide-figure-5000]:./media/virtual-machines-shared-sap-high-availability-guide/5000-wsfc-sap-sid-node-a.png
[sap-ha-guide-figure-5001]:./media/virtual-machines-shared-sap-high-availability-guide/5001-sios-replicating-local-volume.png
[sap-ha-guide-figure-5002]:./media/virtual-machines-shared-sap-high-availability-guide/5002-wsfc-sap-sid-node-b.png
[sap-ha-guide-figure-5003]:./media/virtual-machines-shared-sap-high-availability-guide/5003-sios-replicating-local-volume-b-to-a.png

[sap-ha-guide-figure-6003]:./media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png


[sap-ha-guide-figure-8001]:./media/virtual-machines-shared-sap-high-availability-guide/8001.png
[sap-ha-guide-figure-8002]:./media/virtual-machines-shared-sap-high-availability-guide/8002.png
[sap-ha-guide-figure-8003]:./media/virtual-machines-shared-sap-high-availability-guide/8003.png
[sap-ha-guide-figure-8004]:./media/virtual-machines-shared-sap-high-availability-guide/8004.png
[sap-ha-guide-figure-8005]:./media/virtual-machines-shared-sap-high-availability-guide/8005.png
[sap-ha-guide-figure-8006]:./media/virtual-machines-shared-sap-high-availability-guide/8006.png
[sap-ha-guide-figure-8007]:./media/virtual-machines-shared-sap-high-availability-guide/8007.png
[sap-ha-guide-figure-8008]:./media/virtual-machines-shared-sap-high-availability-guide/8008.png
[sap-ha-guide-figure-8009]:./media/virtual-machines-shared-sap-high-availability-guide/8009.png
[sap-ha-guide-figure-8010]:./media/virtual-machines-shared-sap-high-availability-guide/8010.png
[sap-ha-guide-figure-8011]:./media/virtual-machines-shared-sap-high-availability-guide/8011.png
[sap-ha-guide-figure-8012]:./media/virtual-machines-shared-sap-high-availability-guide/8012.png
[sap-ha-guide-figure-8013]:./media/virtual-machines-shared-sap-high-availability-guide/8013.png
[sap-ha-guide-figure-8014]:./media/virtual-machines-shared-sap-high-availability-guide/8014.png
[sap-ha-guide-figure-8015]:./media/virtual-machines-shared-sap-high-availability-guide/8015.png
[sap-ha-guide-figure-8016]:./media/virtual-machines-shared-sap-high-availability-guide/8016.png
[sap-ha-guide-figure-8017]:./media/virtual-machines-shared-sap-high-availability-guide/8017.png
[sap-ha-guide-figure-8018]:./media/virtual-machines-shared-sap-high-availability-guide/8018.png
[sap-ha-guide-figure-8019]:./media/virtual-machines-shared-sap-high-availability-guide/8019.png
[sap-ha-guide-figure-8020]:./media/virtual-machines-shared-sap-high-availability-guide/8020.png
[sap-ha-guide-figure-8021]:./media/virtual-machines-shared-sap-high-availability-guide/8021.png
[sap-ha-guide-figure-8022]:./media/virtual-machines-shared-sap-high-availability-guide/8022.png
[sap-ha-guide-figure-8023]:./media/virtual-machines-shared-sap-high-availability-guide/8023.png
[sap-ha-guide-figure-8024]:./media/virtual-machines-shared-sap-high-availability-guide/8024.png
[sap-ha-guide-figure-8025]:./media/virtual-machines-shared-sap-high-availability-guide/8025.png


[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md