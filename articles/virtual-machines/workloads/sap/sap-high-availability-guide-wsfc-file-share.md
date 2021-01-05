---
title: Mettre en cluster SAP ASCS/SCS sur WSFC à l’aide d’un partage de fichiers dans Azure | Microsoft Docs
description: Découvrez comment mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide du partage de fichiers dans Azure.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 5e514964-c907-4324-b659-16dd825f6f87
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 07/24/2019
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 469f6a1021fde661c4eae7951b86c9bb500c7050
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012491"
---
# <a name="cluster-an-sap-ascsscs-instance-on-a-windows-failover-cluster-by-using-a-file-share-in-azure"></a>Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide du partage de fichiers dans Azure

> ![Logo Windows][Logo_Windows] Windows
>

Le clustering de basculement Windows Server constitue la base d’une installation de SGBD et de SAP ASCS/SCS à haute disponibilité dans Windows.

Un cluster de basculement est un groupe de 1 + n serveurs indépendants (nœuds) qui fonctionnent ensemble pour accroître la disponibilité des applications et des services. En cas d’échec d’un nœud, le clustering de basculement Windows Server calcule le nombre d’échecs qui peuvent se produire sans que le cluster ne perde son intégrité, de sorte que les applications et les services puissent être fournis. Différents modes de quorum sont disponibles pour obtenir un clustering de basculement.

## <a name="prerequisites"></a>Prérequis
Avant d’aborder les tâches décrites dans cet article, consultez l’article suivant :

* [Scénarios et architecture de haute disponibilité de machines virtuelles Azure pour SAP NetWeaver][sap-high-availability-architecture-scenarios]

> [!IMPORTANT]
> Le clustering d’instances SAP ASCS/SCS avec le partage de fichiers est pris en charge pour SAP NetWeaver 7.40 (et versions ultérieures), avec SAP Kernel 7.49 (et versions ultérieures).
>


## <a name="windows-server-failover-clustering-in-azure"></a>Clustering de basculement Windows Server dans Azure

Par rapport aux déploiements complets ou de cloud privé, le service Machines virtuelles Azure requiert des étapes supplémentaires pour configurer le clustering de basculement Windows Server. Quand vous créez un cluster, vous devez définir plusieurs adresses IP et noms d’hôtes virtuels pour l’instance SAP ASCS/SCS.

### <a name="name-resolution-in-azure-and-the-cluster-virtual-host-name"></a>Résolution de noms dans Azure et nom d’hôte virtuel du cluster

La plateforme cloud Azure ne permet pas de configurer des adresses IP virtuelles telles que des adresses IP flottantes. Vous avez besoin d’une autre solution pour configurer une adresse IP virtuelle afin d’atteindre la ressource de cluster dans le cloud. 

Le service Azure Load Balancer fournit un *équilibreur de charge interne* pour Azure. Avec l’équilibrage de charge interne, les clients atteignent le cluster via son adresse IP virtuelle. 

Déployez l’équilibreur de charge interne dans le groupe de ressources qui contient les nœuds de cluster. Ensuite, configurez toutes les règles de réacheminement de port nécessaires en utilisant les ports de sondage de l’équilibreur de charge interne. Les clients peuvent se connecter avec le nom d’hôte virtuel. Le serveur DNS résout l’adresse IP du cluster. L’équilibreur de charge interne gère le réacheminement de port vers le nœud actif du cluster.

![Figure 1 : Configuration du clustering de basculement Windows Server dans Azure sans disque partagé][sap-ha-guide-figure-1001]

_**Figure 1 :** Configuration du clustering de basculement Windows Server dans Azure sans disque partagé_

## <a name="sap-ascsscs-ha-with-file-share"></a>Haute disponibilité SAP ASCS/SCS avec partage de fichiers

SAP a développé une nouvelle approche et une alternative aux disques partagés pour le clustering d’une instance SAP ASCS/SCS sur un cluster de basculement Windows. Au lieu d’utiliser des disques partagés de cluster, vous pouvez utiliser un partage de fichiers SMB pour déployer des fichiers d’hôte global SAP.

> [!NOTE]
> Le partage de fichiers SMB est une alternative aux disques partagés de cluster pour le clustering d’instances SAP ASCS/SCS.  
>

Voici les spécificités de cette architecture :

* Les services centraux SAP (avec une structure de fichiers et des processus de messages et d’empilement propres) sont séparés des fichiers d’hôte global SAP.
* Les services centraux SAP s’exécutent sous une instance SAP ASCS/SCS.
* L’instance SAP ASCS/SCS est en cluster et accessible à l’aide du nom d’hôte virtuel \<ASCS/SCS virtual host name\>.
* Les fichiers globaux SAP sont placés sur le partage de fichiers SMB et accessibles à l’aide du nom d’hôte \<SAP global host\> : \\\\&lt;Hôte global SAP&gt;\sapmnt\\&lt;SID&gt;\SYS\..
* L’instance SAP ASCS/SCS est installée sur un disque local sur les deux nœuds de cluster
* Le nom réseau \<ASCS/SCS virtual host name\> est différent de &lt;SAP global host&gt;.

![Figure 2 : Architecture à haute disponibilité SAP ASCS/SCS avec partage de fichiers SMB][sap-ha-guide-figure-8004]

_**Figure 2 :** Nouvelle architecture à haute disponibilité SAP ASCS/SCS avec partage de fichiers SMB_

Conditions préalables pour un partage de fichiers SMB :

* Protocole SMB 3.0 (ou version ultérieure)
* Possibilité de définir les listes de contrôle d’accès (ACL, access control list) Active Directory pour les groupes d’utilisateurs Active Directory et l’objet Ordinateur `computer$`
* Haute disponibilité activée pour le partage de fichiers :
    * Les disques utilisés pour stocker des fichiers ne doivent pas constituer un point de défaillance unique.
    * Les temps d’arrêt de serveur ou de machine virtuelle n’entraînent pas de temps d’arrêt du partage de fichiers.

Le rôle de cluster SAP \<SID\> ne contient aucun disque partagé de cluster ni aucune ressource de cluster générique de partage de fichiers.


![Figure 3 : Ressources du rôle de cluster SAP \<SID\> pour l’utilisation d’un partage de fichiers][sap-ha-guide-figure-8005]

_**Figure 3 :** Ressources du rôle de cluster SAP &lt;SID&gt; pour l’utilisation d’un partage de fichiers_


## <a name="scale-out-file-shares-with-storage-spaces-direct-in-azure-as-an-sapmnt-file-share"></a>Partages de fichiers avec montée en puissance parallèle avec les espaces de stockage direct dans Azure en tant que partage de fichiers SAPMNT

Vous pouvez utiliser un partage de fichiers avec montée en puissance parallèle pour héberger et protéger des fichiers d’hôte global SAP. Un partage de fichiers avec montée en puissance parallèle offre également un service de partage de fichiers SAPMNT hautement disponible.

![Figure 4 : Partage de fichiers avec scale-out utilisé pour protéger les fichiers d’hôte global SAP][sap-ha-guide-figure-8006]

_**Figure 4 :** Partage de fichiers avec scale-out utilisé pour protéger les fichiers d’hôte global SAP_

> [!IMPORTANT]
> Les partages de fichiers avec montée en puissance parallèle sont entièrement pris en charge dans le cloud Microsoft Azure et dans les environnements locaux.
>

Un partage de fichiers avec montée en puissance parallèle offre un partage de fichiers SAPMNT hautement disponible et scalable horizontalement.

Les espaces de stockage direct sont utilisés en tant que disque partagé pour un partage de fichiers avec montée en puissance parallèle. Vous pouvez utiliser les espaces de stockage direct pour générer un stockage hautement disponible et évolutif à l’aide de serveurs à stockage local. Le stockage partagé utilisé pour un partage de fichiers avec montée en puissance parallèle, comme pour les fichiers d’hôte global SAP, ne constitue pas un point de défaillance unique.

Lorsque vous choisissez les espaces de stockage direct, tenez compte des cas d’utilisation suivants :

- Les machines virtuelles utilisées pour créer le cluster d’espaces de stockage direct doivent être déployées dans un groupe à haute disponibilité Azure.
- Pour la récupération d’urgence d’un cluster d’espaces de stockage direct, vous pouvez utiliser les services [Azure Site Recovery](../../../site-recovery/azure-to-azure-support-matrix.md#replicated-machines---storage).
- Il n’est pas possible de déployer le cluster d’espaces de stockage direct sur différentes Zones de disponibilité Azure.

### <a name="sap-prerequisites-for-scale-out-file-shares-in-azure"></a>Conditions préalables liées à SAP pour les partages de fichiers avec montée en puissance parallèle dans Azure

Si vous souhaitez utiliser un partage de fichiers avec montée en puissance parallèle, votre système doit répondre aux exigences suivantes :

* Au moins deux nœuds de cluster doivent être disponibles pour un partage de fichiers avec montée en puissance parallèle.
* Chaque nœud doit disposer d’au moins deux disques locaux.
* Pour des raisons de performances, vous devez utiliser la *mise en miroir de la résilience* :
    * Mise en miroir double pour un partage de fichiers avec montée en puissance parallèle avec deux nœuds de cluster
    * Mise en miroir triple pour un partage de fichiers avec montée en puissance parallèle avec trois nœuds de cluster (ou plus)
* Nous vous recommandons de prévoir trois nœuds de cluster (ou plus) pour un partage de fichiers avec montée en puissance parallèle, avec une mise en miroir triple.
    Cette configuration offre une meilleure extensibilité et une meilleure résilience du stockage que la configuration basée sur un partage de fichiers avec montée en puissance parallèle avec deux nœuds de cluster et une mise en miroir double.
* Vous devez utiliser des disques Azure Premium.
* Nous vous recommandons d’utiliser Azure Disques managés.
* Nous vous recommandons de formater les volumes à l’aide du système ReFS (Resilient File System).
    * Pour plus d’informations, consultez le document [SAP Note 1869038 - SAP support for ReFs filesystem][1869038] (Note SAP n° 1869038 - Prise en charge SAP du système de fichiers ReFs) et la section [Choix du système de fichiers][planning-volumes-s2d-choosing-filesystem] de l’article Planification des volumes dans les espaces de stockage direct.
    * Veillez à installer la [mise à jour cumulative Microsoft KB4025334][kb4025334].
* Vous pouvez utiliser les tailles de machines virtuelles Azure séries DS ou séries DSv2.
* Pour obtenir de bonnes performances réseau entre les machines virtuelles (nécessaires pour la synchronisation des disques d’espaces de stockage direct), utilisez un type de machine virtuelle disposant au moins d’une bande passante réseau élevée.
    Pour plus d’informations, consultez les spécifications des [séries DSv2][dv2-series] et des [séries DS][ds-series].
* Nous vous recommandons de réserver une capacité non allouée dans le pool de stockage. Vous laisserez ainsi aux volumes suffisamment d’espace pour effectuer une réparation « sur place » en cas d’échec d’un disque. Cette méthode améliore les performances et la sécurité des données.  Pour plus d’informations, consultez la rubrique [Choix de la taille des volumes][choosing-the-size-of-volumes-s2d].
* Vous n’avez pas besoin de configurer l’équilibreur de charge interne Azure avec le nom réseau du partage de fichiers Scale-out, comme pour \<SAP global host\>. Cette opération s’effectue pour le \<ASCS/SCS virtual host name\> de l’instance SAP ASCS/SCS ou pour le système de gestion de base de données (SGBD). Un partage de fichiers avec montée en puissance parallèle fait monter en charge l’ensemble des nœuds de cluster. \<SAP global host\> utilise l’adresse IP locale pour tous les nœuds de cluster.


> [!IMPORTANT]
> Vous ne pouvez pas renommer le partage de fichiers SAPMNT, qui pointe vers \<SAP global host\>. SAP prend en charge uniquement le nom de partage « sapmnt ».
>
> Pour plus d’informations, reportez-vous au document [SAP Note 2492395 - Can the share name sapmnt be changed?][2492395] (Note SAP n° 2492395 : Le nom de partage sapmnt peut-il être modifié ?).

### <a name="configure-sap-ascsscs-instances-and-a-scale-out-file-share-in-two-clusters"></a>Configurer des instances SAP ASCS/SCS et un partage de fichiers avec montée en puissance parallèle dans deux clusters

Vous pouvez déployer des instances SAP ASCS/SCS dans un cluster avec leur propre rôle de cluster SAP \<SID\>. Dans ce cas, vous devez configurer le partage de fichiers avec montée en puissance parallèle sur un autre cluster, avec un autre rôle de cluster.

> [!IMPORTANT]
>Dans ce scénario, l’instance SAP ASCS/SCS est configurée pour accéder à l’hôte global SAP à l’aide du chemin d’accès UNC \\\\&lt;hôte global SAP&gt;\sapmnt\\&lt;SID&gt;\SYS\..
>

![Figure 5 : Instance SAP ASCS/SCS et partage de fichiers avec scale-out dans deux clusters][sap-ha-guide-figure-8007]

_**Figure 5 :** Instance SAP ASCS/SCS et partage de fichiers avec scale-out dans deux clusters_

> [!IMPORTANT]
> Dans le cloud Azure, chaque cluster utilisé pour les partage de fichiers scale-out et SAP doit être déployé dans son propre groupe à haute disponibilité Azure ou dans les Zones de disponibilité Azure. Ceci garantit une sélection élective distribuée des machines virtuelles du cluster dans toute l’infrastructure Azure sous-jacente. Les déploiements de zones de disponibilité sont pris en charge avec cette technologie.
>

## <a name="generic-file-share-with-sios-datakeeper-as-cluster-shared-disks"></a>Partage de fichiers générique avec SIOS DataKeeper en tant que disques partagés de cluster


Un partage de fichiers générique constitue une alternative au partage de fichiers hautement disponible.

Dans ce cas, vous pouvez utiliser une solution SIOS tierce en tant que disque partagé de cluster.

## <a name="next-steps"></a>Étapes suivantes

* [Préparation d’infrastructure Azure pour la haute disponibilité SAP à l’aide de cluster de basculement Windows et de partage de fichiers pour une instance SAP (A)SCS][sap-high-availability-infrastructure-wsfc-file-share]
* [Installation de la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un disque partagé pour une instance SAP (A)SCS][sap-high-availability-installation-wsfc-shared-disk]
* [Déployer un serveur de fichiers à deux nœuds Storage Spaces Direct réparti pour le stockage UPD dans Azure][deploy-sofs-s2d-in-azure]
* [Storage Spaces Direct dans Windows Server 2016][s2d-in-win-2016]
* [Présentation approfondie : Columes dans les espaces de stockage direct][deep-dive-volumes-in-s2d]

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[kb4025334]:https://support.microsoft.com/help/4025334/windows-10-update-kb4025334

[dv2-series]:../../dv2-dsv2-series.md
[ds-series]:https://docs.microsoft.com/azure/virtual-machines/windows/sizes-general

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-volumes-s2d-choosing-filesystem]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-filesystem
[choosing-the-size-of-volumes-s2d]:https://docs.microsoft.com/windows-server/storage/storage-spaces/plan-volumes#choosing-the-size-of-volumes
[deploy-sofs-s2d-in-azure]:https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-storage-spaces-direct-deployment
[s2d-in-win-2016]:https://docs.microsoft.com/windows-server/storage/storage-spaces/storage-spaces-direct-overview
[deep-dive-volumes-in-s2d]:https://blogs.technet.microsoft.com/filecab/2016/08/29/deep-dive-volumes-in-spaces-direct/

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

[1869038]:https://launchpad.support.sap.com/#/notes/1869038 
