---
title: Infrastructure Azure pour SAP ASCS/SCS avec WSFC et un disque partagé | Microsoft Docs
description: Découvrez comment préparer l’infrastructure Azure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows et d’un disque partagé pour une instance SAP ASCS/SCS.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: ec976257-396b-42a0-8ea1-01c97f820fa6
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 00d0aa10f2beda3d7b8508e5ca823e020dc79d95
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489137"
---
# <a name="prepare-the-azure-infrastructure-for-sap-ha-by-using-a-windows-failover-cluster-and-shared-disk-for-sap-ascsscs"></a>Préparer l’infrastructure Azure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows et d’un disque partagé pour SAP ASCS/SCS

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides
[tuning-failover-cluster-network-thresholds]:https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834

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
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-ha-guide-9.1.1]:high-availability-guide.md#a97ad604-9094-44fe-a364-f89cb39bf097
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f


[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-shared-disk-vpn]:sap-high-availability-infrastructure-wsfc-shared-disk.md#c87a8d3f-b1dc-4d2f-b23c-da4b72977489
[sap-high-availability-infrastructure-wsfc-shared-disk-change-def-ilb]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-wsfc]:sap-high-availability-infrastructure-wsfc-shared-disk.md#0d67f090-7928-43e0-8772-5ccbf8f59aab
[sap-high-availability-infrastructure-wsfc-shared-disk-collect-cluster-config]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5eecb071-c703-4ccc-ba6d-fe9c6ded9d79
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868
[sap-high-availability-infrastructure-wsfc-shared-disk-add-dot-net]:sap-high-availability-infrastructure-wsfc-shared-disk.md#1c2788c3-3648-4e82-9e0d-e058e475e2a3
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios-both-nodes]:sap-high-availability-infrastructure-wsfc-shared-disk.md#dd41d5a2-8083-415b-9878-839652812102
[sap-high-availability-infrastructure-wsfc-shared-disk-setup-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#d9c1fc8e-8710-4dff-bec2-1f535db7b006

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

[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md


> ![Système d’exploitation Windows][Logo_Windows] Windows


Cet article décrit les étapes à suivre pour préparer l’infrastructure Azure à l’installation et à la configuration d’une instance SAP ASCS/SCS haute disponibilité sur un cluster de basculement Windows en utilisant un *disque partagé de cluster* en tant qu’option pour le clustering d’une instance SAP ASCS.
Deux alternatives au *disque partagé de cluster* sont présentées dans la documentation :

- [disques partagés Azure](../../disks-shared.md) ;
- utilisation de [SIOS DataKeeper Cluster Edition](https://us.sios.com/products/datakeeper-cluster/) pour créer un stockage en miroir qui simule un disque partagé en cluster. 

La configuration présentée repose sur des [groupes de placement de proximité (PPG)](./sap-proximity-placement-scenarios.md) pour atteindre une latence réseau optimale pour les charges de travail SAP. La documentation ne couvre pas la couche de base de données.  

> [!NOTE]
> Les groupes de placement de proximité Azure sont une condition préalable à l’utilisation de disques partagés Azure.
 

## <a name="prerequisites"></a>Prérequis

Avant de commencer l’installation, consultez cet article :

* [Guide d'architecture : Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster][sap-high-availability-guide-wsfc-shared-disk].

## <a name="create-the-ascs-vms"></a>Créer les machines virtuelles ASCS

Pour le cluster SAP ASCS/SCS, déployez deux machines virtuelles dans un groupe à haute disponibilité Azure. Déployez les machines virtuelles dans le même groupe de placement de proximité. Une fois les machines virtuelles déployées :  
- Créez un équilibreur de charge interne Azure Internal pour l’instance SAP ASCS/SCS. 
- Ajoutez des machines virtuelles Windows au domaine AD.

Les noms d’hôte et les adresses IP pour le scénario présenté sont les suivants :

| Rôle du nom d’hôte | Nom de l’hôte | Adresse IP statique | Groupe à haute disponibilité | Groupe de placement de proximité |
| --- | --- | --- |---| ---|
| 1er nœud de cluster du cluster ASCS/SCS |pr1-ascs-10 |10.0.0.4 |pr1-ascs-avset |PR1PPG |
| 2e nœud de cluster du cluster ASCS/SCS |pr1-ascs-11 |10.0.0.5 |pr1-ascs-avset |PR1PPG |
| Nom réseau du cluster | pr1clust |10.0.0.42 (**uniquement** pour le cluster Win 2016) | n/a | n/a |
| Nom réseau du cluster ASCS | pr1-ascscl |10.0.0.43 | n/a | n/a |
| Nom réseau du cluster ERS (**uniquement** pour ERS2) | pr1-erscl |10.0.0.44 | n/a | n/a |


## <a name="create-azure-internal-load-balancer"></a><a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> Créer un équilibreur de charge interne Azure

SAP ASCS, SAP SCS et le nouveau SAP ERS2 utilisent un nom d’hôte virtuel et des adresses IP virtuelles. Sur Azure, un [équilibreur de charge](../../../load-balancer/load-balancer-overview.md) est nécessaire pour utiliser une adresse IP virtuelle. Nous vous recommandons vivement d’utiliser un [équilibreur de charge standard](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). 

> [!IMPORTANT]
> Une adresse IP flottante n’est pas prise en charge sur une configuration IP secondaire de carte réseau pour des scénarios d’équilibrage de charge. Pour plus d’informations, consultez [Limitations d’équilibreur de charge Azure](../../../load-balancer/load-balancer-multivip-overview.md#limitations). Si vous avez besoin d’une adresse IP supplémentaire pour la machine virtuelle, déployez une deuxième carte réseau.    


La liste suivante présente la configuration de l’équilibreur de charge d’(A)SCS/ERS. La configuration de SAP ASCS et ERS2 est effectuée dans le même équilibreur de charge Azure.  

**(A)SCS**
- Configuration du frontend
    - Adresse IP ASCS/SCS statique **10.0.0.43**
- Configuration du backend  
    Ajoutez toutes les machines virtuelles qui doivent faire partie du cluster (A)SCS/ERS. Dans cet exemple, il s’agit des machines virtuelles **pr1-ascs-10** et **pr1-ascs-11**.
- Port de la sonde
    - Port 620 **nr** Conservez l’option par défaut pour Protocole (TCP), Intervalle (5) et Seuil de défaillance sur le plan de l’intégrité (2)
- Règles de l’équilibrage de charge
    - Si vous utilisez Standard Load Balancer, sélectionnez Ports haute disponibilité
    - Si vous utilisez Basic Load Balancer, créez des règles d’équilibrage de charge pour les ports suivants
        - TCP 32 **nr**
        - TCP 36 **nr**
        - TCP 39 **nr**
        - TCP 81 **nr**
        - TCP 5 **nr** 13
        - TCP 5 **nr** 14
        - TCP 5 **nr** 16

    - Assurez-vous que le délai d’inactivité (en minutes) est défini sur la valeur maximale (30), et que l’option Adresse IP flottante (retour direct du serveur) est activée.

**ERS2**

Le serveur ERS2 (Enqueue Replication Server 2) étant également en cluster, son adresse IP virtuelle doit aussi être configurée sur Azure ILB en plus de l’adresse IP SAP ASCS/SCS ci-dessus. Cette section ne s’applique qu’en cas d’utilisation de l’architecture ERS2.  
- Configuration du deuxième frontend
    - Adresse IP statique de SAP ERS2 **10.0.0.44**

- Configuration du backend  
  Les machines virtuelles ont déjà été ajoutées au pool principal ILB.  

- Port de la deuxième sonde
    - Port 621 **nr**  
    Conservez l’option par défaut pour Protocole (TCP), Intervalle (5) et Seuil de défaillance sur le plan de l’intégrité (2)

- Règles du deuxième équilibrage de charge
    - Si vous utilisez Standard Load Balancer, sélectionnez Ports haute disponibilité
    - Si vous utilisez Basic Load Balancer, créez des règles d’équilibrage de charge pour les ports suivants
        - TCP 32 **nr**
        - TCP 33 **nr**
        - TCP 5 **nr** 13
        - TCP 5 **nr** 14
        - TCP 5 **nr** 16

    - Assurez-vous que le délai d’inactivité (en minutes) est défini sur la valeur maximale (30), et que l’option Adresse IP flottante (retour direct du serveur) est activée.


> [!TIP]
> Le [modèle Azure Resource Manager pour WSFC pour instance SAP ASCS/SCS avec disque partagé Azure](https://github.com/robotechredmond/301-shared-disk-sap) vous permet d’automatiser la préparation de l’infrastructure à l’aide d’un disque partagé Azure pour un SID SAP avec ERS1.  
> Le modèle ARM Azure crée deux machines virtuelles Windows 2019 ou 2016, crée un disque partagé Azure et l’attache aux machines virtuelles. Un équilibreur de charge interne Azure est également créé et configuré. Pour plus d’informations, consultez le modèle ARM. 

## <a name="add-registry-entries-on-both-cluster-nodes-of-the-ascsscs-instance"></a><a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> Ajouter des entrées de registre aux deux nœuds de cluster de l’instance SAP ASCS/SCS

Azure Load Balancer peut fermer des connexions inactives pendant une période dépassant le délai d’inactivité. Les processus de travail SAP ouvrent des connexions au processus de mise en file d’attente SAP dès que la première demande de mise en file d’attente/enlèvement de la file d’attente doit être envoyée. Pour éviter d’interrompre ces connexions, modifiez les valeurs KeepAliveTime et KeepAliveInterval de TCP/IP sur les deux nœuds de cluster. Si vous utilisez ERS1, il est également nécessaire d’ajouter des paramètres de profil SAP, comme décrit plus loin dans cet article.
Les entrées de registre suivantes doivent être modifiées sur les deux nœuds de cluster :

- KeepAliveTime
- KeepAliveInterval

| Path| Nom de la variable | Type de variable  | Valeur | Documentation |
| --- | --- | --- |---| ---|
| HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |KeepAliveTime |REG_DWORD (décimal) |120 000 |[KeepAliveTime](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)) |
| HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |KeepAliveInterval |REG_DWORD (décimal) |120 000 |[KeepAliveInterval](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)) |


Pour appliquer les modifications, redémarrez les deux nœuds de cluster.
  
## <a name="add-the-windows-vms-to-the-domain"></a><a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> Ajouter les machines virtuelles Windows au domaine
Après avoir affecté les adresses IP statiques aux machines virtuelles, ajoutez les machines virtuelles au domaine. 

## <a name="install-and-configure--windows-failover-cluster"></a><a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> Installer et configurer un cluster de basculement Windows 

### <a name="install-the-windows-failover-cluster-feature"></a>Installer la fonctionnalité de cluster de basculement Windows

Exécutez la commande suivante sur l’un des nœuds du cluster :

   ```powershell
    # Hostnames of the Win cluster for SAP ASCS/SCS
    $SAPSID = "PR1"
    $ClusterNodes = ("pr1-ascs-10","pr1-ascs-11")
    $ClusterName = $SAPSID.ToLower() + "clust"
    
    # Install Windows features.
    # After the feature installs, manually reboot both nodes
    Invoke-Command $ClusterNodes {Install-WindowsFeature Failover-Clustering, FS-FileServer -IncludeAllSubFeature -IncludeManagementTools }
   ```

Une fois l’installation de la fonctionnalité terminée, redémarrez les deux nœuds de cluster.  

### <a name="test-and-configure-windows-failover-cluster"></a>Tester et configurer le cluster de basculement Windows 

Sur Windows 2019, le cluster reconnaît automatiquement qu’il s’exécute dans Azure et, comme option par défaut pour l’adresse IP de gestion du cluster, il utilise le nom de réseau distribué. Il utilise donc l’une des adresses IP locales de nœuds de cluster. Par conséquent, il n’est pas nécessaire d’avoir un nom de réseau (virtuel) dédié pour le cluster, ou de configurer cette adresse IP sur un équilibreur de charge interne Azure.

Pour plus d’informations, consultez la page consacrée aux [nouvelles fonctionnalités de clustering de basculement Windows Server 2019](https://techcommunity.microsoft.com/t5/failover-clustering/windows-server-2019-failover-clustering-new-features/ba-p/544029). Exécutez la commande suivante sur l’un des nœuds de cluster :

   ```powershell
    # Hostnames of the Win cluster for SAP ASCS/SCS
    $SAPSID = "PR1"
    $ClusterNodes = ("pr1-ascs-10","pr1-ascs-11")
    $ClusterName = $SAPSID.ToLower() + "clust"
    
    # IP adress for cluster network name is needed ONLY on Windows Server 2016 cluster
    $ClusterStaticIPAddress = "10.0.0.42"
        
    # Test cluster
    Test-Cluster –Node $ClusterNodes -Verbose
    
    $ComputerInfo = Get-ComputerInfo
    
    $WindowsVersion = $ComputerInfo.WindowsProductName
    
    if($WindowsVersion -eq "Windows Server 2019 Datacenter"){
        write-host "Configuring Windows Failover Cluster on Windows Server 2019 Datacenter..."
        New-Cluster –Name $ClusterName –Node  $ClusterNodes -Verbose
    }elseif($WindowsVersion -eq "Windows Server 2016 Datacenter"){
        write-host "Configuring Windows Failover Cluster on Windows Server 2016 Datacenter..."
        New-Cluster –Name $ClusterName –Node  $ClusterNodes –StaticAddress $ClusterStaticIPAddress -Verbose 
    }else{
        Write-Error "Not supported Windows version!"
    }
   ```

### <a name="configure-cluster-cloud-quorum"></a>Configurer un quorum cloud de cluster
Lorsque vous utilisez Windows Server 2016 ou 2019, nous vous recommandons de configurer un [témoin cloud Azure](/windows-server/failover-clustering/deploy-cloud-witness), en tant que quorum de cluster.

Exécutez la commande suivante sur l’un des nœuds du cluster :

   ```powershell
    $AzureStorageAccountName = "cloudquorumwitness"
    Set-ClusterQuorum –CloudWitness –AccountName $AzureStorageAccountName -AccessKey <YourAzureStorageAccessKey> -Verbose
   ```

### <a name="tuning-the-windows-failover-cluster-thresholds"></a>Réglage des seuils du cluster de basculement Windows
 
Une fois que vous avez correctement installé le cluster de basculement Windows, vous devez ajuster certains seuils pour qu’ils soient adaptés aux clusters déployés dans Azure. Les paramètres à changer sont documentés dans : [Tuning Failover Cluster Network Thresholds (Réglage des seuils réseau de cluster de basculement)](https://techcommunity.microsoft.com/t5/Failover-Clustering/Tuning-Failover-Cluster-Network-Thresholds/ba-p/371834). En supposant que les deux machines virtuelles qui constituent la configuration du cluster Windows pour ASCS/SCS se trouvent dans le même sous-réseau, modifiez les paramètres suivants avec ces valeurs :
- SameSubNetDelay = 2000
- SameSubNetThreshold = 15
- RoutingHistoryLength = 30

Ces paramètres ont été testés avec des clients et offrent un bon compromis. Ils sont suffisamment résilients, mais assurent également un basculement assez rapide dans des conditions d’erreur réelles de charges de travail SAP ou de défaillance de machine virtuelle.  

## <a name="configure-azure-shared-disk"></a>Configurer un disque partagé Azure
Cette section s’applique uniquement si vous utilisez un disque partagé Azure. 

### <a name="create-and-attach-azure-shared-disk-with-powershell"></a>Créer et attacher un disque partagé Azure avec PowerShell
Exécutez la commande suivante sur l’un des nœuds du cluster. Vous devez ajuster les valeurs de votre groupe de ressources, de votre région Azure, du SID SAP, etc.  

   ```powershell
    #############################
    # Create Azure Shared Disk
    #############################
    
    $ResourceGroupName = "MyResourceGroup"
    $location = "MyAzureRegion"
    $SAPSID = "PR1"
    
    $DiskSizeInGB = 512
    $DiskName = "$($SAPSID)ASCSSharedDisk"
    
    # With parameter '-MaxSharesCount', we define the maximum number of cluster nodes to attach the shared disk
    $NumberOfWindowsClusterNodes = 2
            
    $diskConfig = New-AzDiskConfig -Location $location -SkuName Premium_LRS  -CreateOption Empty  -DiskSizeGB $DiskSizeInGB -MaxSharesCount $NumberOfWindowsClusterNodes
    $dataDisk = New-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName -Disk $diskConfig
    
    ##################################
    ## Attach the disk to cluster VMs
    ##################################
    # ASCS Cluster VM1
    $ASCSClusterVM1 = "$SAPSID-ascs-10"
    
    # ASCS Cluster VM2
    $ASCSClusterVM2 = "$SAPSID-ascs-11"
    
    # Add the Azure Shared Disk to Cluster Node 1
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM1 
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
    
    # Add the Azure Shared Disk to Cluster Node 2
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM2
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
   ```

### <a name="format-the-shared-disk-with-powershell"></a>Formater le disque partagé avec PowerShell
1. Procurez-vous le numéro du disque. Exécutez les commandes PowerShell suivantes sur l’un des nœuds du cluster :

   ```powershell
    Get-Disk | Where-Object PartitionStyle -Eq "RAW"  | Format-Table -AutoSize 
    # Example output
    # Number Friendly Name     Serial Number HealthStatus OperationalStatus Total Size Partition Style
    # ------ -------------     ------------- ------------ ----------------- ---------- ---------------
    # 2      Msft Virtual Disk               Healthy      Online                512 GB RAW            

   ```
2. Formatez le disque. Dans cet exemple, il s’agit du disque numéro 2. 

   ```powershell
    # Format SAP ASCS Disk number '2', with drive letter 'S'
    $SAPSID = "PR1"
    $DiskNumber = 2
    $DriveLetter = "S"
    $DiskLabel = "$SAPSID" + "SAP"
    
    Get-Disk -Number $DiskNumber | Where-Object PartitionStyle -Eq "RAW" | Initialize-Disk -PartitionStyle GPT -PassThru |  New-Partition -DriveLetter $DriveLetter -UseMaximumSize | Format-Volume  -FileSystem ReFS -NewFileSystemLabel $DiskLabel -Force -Verbose
    # Example outout
    # DriveLetter FileSystemLabel FileSystem DriveType HealthStatus OperationalStatus SizeRemaining      Size
    # ----------- --------------- ---------- --------- ------------ ----------------- -------------      ----
    # S           PR1SAP          ReFS       Fixed     Healthy      OK                    504.98 GB 511.81 GB
   ```

3. Vérifiez que le disque est maintenant visible en tant que disque de cluster.  
   ```powershell
    # List all disks
    Get-ClusterAvailableDisk -All
    # Example output
    # Cluster    : pr1clust
    # Id         : 88ff1d94-0cf1-4c70-89ae-cbbb2826a484
    # Name       : Cluster Disk 1
    # Number     : 2
    # Size       : 549755813888
    # Partitions : {\\?\GLOBALROOT\Device\Harddisk2\Partition2\}
   ```
4. Enregistrez le disque dans le cluster.  
   ```powershell
    # Add the disk to cluster 
    Get-ClusterAvailableDisk -All | Add-ClusterDisk
    # Example output     
    # Name           State  OwnerGroup        ResourceType 
    # ----           -----  ----------        ------------ 
    # Cluster Disk 1 Online Available Storage Physical Disk
   ```

## <a name="sios-datakeeper-cluster-edition-for-the-sap-ascsscs-cluster-share-disk"></a><a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> SIOS DataKeeper Cluster Edition pour le disque de partage de cluster SAP ASCS/SCS
Cette section s’applique uniquement si vous utilisez le logiciel tiers SIOS DataKeeper Cluster Edition pour créer un stockage en miroir qui simule le disque partagé de cluster.  

Vous disposez maintenant d’une configuration fonctionnelle de clustering de basculement Windows Server dans Azure. Pour installer une instance SAP ASCS/SCS, vous avez besoin d’une ressource de disque partagé. L’une des options consister à utiliser SIOS DataKeeper Cluster Edition, une solution tierce que vous pouvez utiliser pour créer des ressources de disque partagé.  

L’installation de SIOS DataKeeper Cluster Edition pour le disque de partage en cluster SAP ASCS/SCS implique les tâches suivantes :
- Ajouter Microsoft .NET Framework si nécessaire. Consultez la [documentation de la version SIÔS] (https://us.sios.com/products/datakeeper-cluster/) pour connaître les exigences de .NET Framework les plus récentes. 
- Installer SIOS DataKeeper.
- Configurer SIOS DataKeeper.

### <a name="install-sios-datakeeper"></a> Installer SIOS DataKeeper
Installez SIOS DataKeeper Cluster Edition sur chaque nœud du cluster. Pour créer un stockage partagé virtuel avec SIOS DataKeeper, créez un miroir synchronisé, puis simulez le stockage partagé en cluster.

Avant d’installer le logiciel SIOS, créez l’utilisateur de domaine DataKeeperSvc.

> [!NOTE]
> Ajoutez l’utilisateur de domaine DataKeeperSvc au groupe Administrateur local sur les deux nœuds de cluster.
>

1. Installez le logiciel SIOS sur les deux nœuds de cluster.

   ![Programme d’installation de SIOS][sap-ha-guide-figure-3030]

   ![Figure 31 : Première page de l’installation de SIOS DataKeeper][sap-ha-guide-figure-3031]

   _Première page de l’installation de SIOS DataKeeper_

2. Dans la boîte de dialogue, sélectionnez **Oui**.

   ![Figure 32 : DataKeeper vous indique qu’un service va être désactivé][sap-ha-guide-figure-3032]

   _DataKeeper vous informe qu’un service va être désactivé_

3. Dans la boîte de dialogue, nous vous recommandons de sélectionner **Compte de domaine ou de serveur**.

   ![Figure 33 : Sélection de l’utilisateur de SIOS DataKeeper][sap-ha-guide-figure-3033]

   _Sélection de l’utilisateur de SIOS DataKeeper_

4. Entrez le nom et le mot de passe de l’utilisateur de compte de domaine que vous avez créés pour SIOS DataKeeper.

   ![Figure 34 : Entrer le nom et le mot de passe de l’utilisateur du domaine pour l’installation de SIOS DataKeeper][sap-ha-guide-figure-3034]

   _Entrer le nom et le mot de passe de l’utilisateur du domaine pour l’installation de SIOS DataKeeper_

5. Installez la clé de licence de votre instance SIOS DataKeeper, comme indiqué dans la figure 35.

   ![Figure 35 : Entrer votre clé de licence SIOS DataKeeper][sap-ha-guide-figure-3035]

   _Entrer votre clé de licence SIOS DataKeeper_

6. À l’invite, redémarrez la machine virtuelle.

### <a name="configure-sios-datakeeper"></a>Configurer SIOS DataKeeper.
Après avoir installé SIOS DataKeeper sur les deux nœuds, commencez la configuration. L’objectif de la configuration est de mettre en place une réplication synchrone des données entre les disques supplémentaires attachés à chacune des machines virtuelles.

1. Démarrez l’outil de configuration et de gestion de DataKeeper, puis sélectionnez **Connect Server** (Connexion au serveur)

   ![Figure 36 : Outil de configuration et de gestion de SIOS DataKeeper][sap-ha-guide-figure-3036]

   _Outil de configuration et de gestion de SIOS DataKeeper_

2. Insérez le nom ou l’adresse TCP/IP du premier nœud auquel l’outil de gestion et de configuration doit se connecter, puis du second nœud.

   ![Figure 37 : Insérer le nom ou l’adresse TCP/IP du premier nœud auquel l’outil de gestion et de configuration doit se connecter, puis du deuxième nœud][sap-ha-guide-figure-3037]

   _Insérer le nom ou l’adresse TCP/IP du premier nœud auquel l’outil de gestion et de configuration doit se connecter, puis du deuxième nœud_

3. Créez le travail de réplication entre les deux nœuds.

   ![Figure 38 : Créer un travail de réplication][sap-ha-guide-figure-3038]

   _Créer un travail de réplication_

   Un assistant vous guide à travers le processus de création d’un travail de réplication.

4. Définissez le nom du travail de réplication.

   ![Figure 39 : Définir le nom du travail de réplication][sap-ha-guide-figure-3039]

   _Définir le nom du travail de réplication_

   ![Figure 40 : Définir les données de base du nœud qui doit être le nœud source actuel][sap-ha-guide-figure-3040]

   _Définir les données de base du nœud qui doit être le nœud source actuel_

5. Définissez le nom, l’adresse TCP/IP et le volume de disque du nœud cible.

   ![Figure 41 : Définir le nom, l'adresse TCP/IP et le volume de disque du nœud cible actuel][sap-ha-guide-figure-3041]

   _Définir le nom, l'adresse TCP/IP et le volume de disque du nœud cible actuel_

6. Définissez les algorithmes de compression. Dans notre exemple, nous vous recommandons de compresser le flux de réplication. Dans les situations de resynchronisation notamment, la compression du flux de réplication accélère considérablement la resynchronisation. La compression utilise les ressources de processeur et de RAM d’une machine virtuelle. Plus le taux de compression augmente, plus le volume de ressources de processeur utilisées est important. Vous pouvez ajuster ce paramètre ultérieurement.

7. Vous devez également vérifier si la réplication se fait de façon asynchrone ou synchrone. Lorsque vous protégez des configurations SAP ASCS/SCS, vous devez utiliser la réplication synchrone.  

   ![Figure 42 : Définir les détails de la réplication][sap-ha-guide-figure-3042]

   _Définir les détails de la réplication_

8. Définissez si le volume répliqué par le travail de réplication doit être représenté auprès d’une configuration de cluster de basculement Windows Server en tant que disque partagé. Pour la configuration SAP ASCS/SCS, sélectionnez **Oui** pour que le cluster Windows voie le volume répliqué en tant que disque partagé utilisable comme volume de cluster.

   ![Figure 43 : Sélectionner Oui pour définir le volume répliqué comme volume de cluster][sap-ha-guide-figure-3043]

   _Sélectionner **Oui** pour définir le volume répliqué comme volume de cluster_

   Une fois le volume créé, l’outil de configuration et de gestion de DataKeeper indique que le travail de réplication est actif.

   ![Figure 44 : La mise en miroir synchrone de DataKeeper pour le disque de partage SAP ASCS/SCS est active][sap-ha-guide-figure-3044]

   _La mise en miroir synchrone de DataKeeper pour le disque de partage SAP ASCS/SCS est active_

   Le Gestionnaire du cluster de basculement affiche maintenant le disque en tant que disque DataKeeper, comme indiqué dans la figure 45 :

   ![Figure 45 : Le Gestionnaire du cluster de basculement montre le disque répliqué par DataKeeper][sap-ha-guide-figure-3045]

   _Le Gestionnaire du cluster de basculement montre le disque répliqué par DataKeeper_


## <a name="next-steps"></a>Étapes suivantes

* [Installer la haute disponibilité SAP NetWeaver à l’aide d’un cluster de basculement Windows et d’un disque partagé pour une instance SAP ASCS/SCS][sap-high-availability-installation-wsfc-shared-disk]