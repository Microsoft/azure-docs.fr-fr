---
title: Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver | Microsoft Docs
description: Dans cet article, nous allons aborder la haute disponibilité des machines virtuelles Azure pour SAP NetWeaver.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 1cfcc14a-6795-4cfd-a740-aa09d6d2b817
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4b329cce5b20d69377fc5ec14a1fafd8d222b2c8
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94958555"
---
# <a name="azure-virtual-machines-high-availability-for-sap-netweaver"></a>Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-file-share]:sap-high-availability-installation-wsfc-file-share.md

[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-suse-ascs-ha-setting-ha-nfs]:high-availability-guide-suse.md#setting-up-a-highly-available-nfs-server
[sap-suse-ascs-ha-setting-ha-anf]:high-availability-guide-suse-netapp-files.md#setting-up-the-azure-netapp-files-infrastructure
[sap-suse-ascs-ha-sap-installation]:high-availability-guide-suse.md#prepare-for-sap-netweaver-installation
[sap-suse-ascs-ha-sap-installation-anf]:high-availability-guide-suse-netapp-files.md
[sap-rhel-ascs-ha]:high-availability-guide-rhel.md
[sap-rhel-ascs-ha-sap-installation-anf]:high-availability-guide-rhel-netapp-files.md
[high-availability-guide-rhel-glusterfs]:high-availability-guide-rhel-glusterfs.md
[high-availability-guide-rhel-pacemaker]:high-availability-guide-rhel-pacemaker.md

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

[deployment-guide]:deployment-guide.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md

[ha-guide]:sap-high-availability-guide.md

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

[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md

[suse-hana-ha-guide]:https://www.suse.com/docrep/documents/ir8w88iwu7/suse_linux_enterprise_server_for_sap_applications_12_sp1.pdf
[suse-drbd-guide]:https://www.suse.com/documentation/sle-ha-12/singlehtml/book_sleha_techguides/book_sleha_techguides.html

[template-multisid-xscs]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[template-converged]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-converged-md%2Fazuredeploy.json
[template-file-server]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-file-server-md%2Fazuredeploy.json

[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

Machines virtuelles Azure est la solution idéale pour les organisations qui ont besoin de ressources de calcul, de stockage et réseau rapidement, sans longs cycles d’acquisition. Vous pouvez utiliser des machines virtuelles Azure pour déployer des applications classiques comme ABAP basé sur SAP NetWeaver, Java et ABAP+Java Stack. Élargissez la fiabilité et la disponibilité sans ressources locales supplémentaires. Comme le service Machines virtuelles Azure prend en charge la connectivité intersite, vous pouvez intégrer les machines virtuelles Azure dans les domaines locaux, les clouds privés et le paysage SAP de votre organisation.

Cette série d’articles couvre les thèmes suivants :

* Architecture et scénarios
* Préparation de l’infrastructure
* Étapes d’installation de SAP pour le déploiement des systèmes SAP à haute disponibilité dans Azure en utilisant le modèle de déploiement Azure Resource Manager

    > [!IMPORTANT]
    > Nous vous recommandons vivement d’utiliser le modèle de déploiement Azure Resource Manager pour vos installations SAP. Celui-ci offre de nombreux avantages par rapport au modèle de déploiement Classic. En savoir plus sur les [modèles de déploiement][virtual-machines-azure-resource-manager-architecture-benefits-arm] Azure.   
    >
* Haute disponibilité SAP sur :
  * ![Logo Windows.][Logo_Windows]  **Windows** avec **cluster de basculement Windows Server (WSFC)**
  * ![Logo Linux.][Logo_Linux] **Linux** avec **Linux Cluster Framework**

Ces articles vous apprendront à protéger les composants constituant un point de défaillance unique comme les services centraux SAP (ASCS/SCS) et les systèmes de gestion de base de données (SGBD). Ils vous permettront également d’en savoir plus sur les composants redondants dans Azure comme le serveur d’applications SAP.

## <a name="high-availability-architecture-and-scenarios-for-sap-netweaver"></a>Scénarios et architecture de haute disponibilité pour SAP NetWeaver

**Résumé :** dans cet article, nous présentons l’architecture haute disponibilité d’un système SAP dans Azure. Nous expliquons comment assurer la haute disponibilité des composants redondants et constituant un point de défaillance unique et abordons les spécificités de la haute disponibilité de l’infrastructure Azure. Nous expliquons également comment ces éléments sont liés aux composants du système SAP. La discussion aborde les spécificités propres à Windows et Linux. Différents scénarios de haute disponibilité SAP sont également abordés.

**Date de mise à jour :** Octobre 2017

* [Scénarios et architecture de haute disponibilité de machines virtuelles Azure pour SAP NetWeaver][sap-high-availability-architecture-scenarios]

L’article aborde les deux ![logos Windows.][Logo_Windows] **Logo Windows** et ![Linux.][Logo_Linux] **Linux**.


## <a name="azure-infrastructure-preparation-for-sap-netweaver-high-availability-deployment"></a>Préparation de l’infrastructure Azure pour le déploiement de la haute disponibilité de SAP NetWeaver

**Résumé :** dans les articles répertoriés ici, nous expliquons les étapes que vous pouvez suivre pour déployer l’infrastructure Azure en vue de l’installation de SAP. Pour simplifier le déploiement de l’infrastructure Azure, les modèles SAP d’Azure Resource Manager sont utilisés pour automatiser l’ensemble du processus.

**Date de mise à jour :** Mars 2019

* ![Logo Windows.][Logo_Windows] [Préparer une infrastructure Azure pour la haute disponibilité SAP à l’aide de cluster de basculement Windows et de **disque partagé** pour une instance SAP (A)SCS][sap-high-availability-infrastructure-wsfc-shared-disk]

* ![Logo Windows.][Logo_Windows] [Préparer une infrastructure Azure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows et un **partage de fichiers** pour une instance SAP (A)SCS][sap-high-availability-infrastructure-wsfc-file-share]

* ![Logo Linux.][Logo_Linux] [Préparer l’infrastructure Azure pour la haute disponibilité SAP en utilisant l’infrastructure de cluster SUSE Linux Enterprise Server pour des instances de SAP (A)SCS][sap-suse-ascs-ha-setting-ha-nfs]

* ![Logo Linux.][Logo_Linux] [Préparer une infrastructure Azure pour la haute disponibilité SAP à l’aide d’un framework de cluster SUSE Linux Enterprise Server pour des instances SAP ASCS/SCS avec Azure NetApp Files][sap-suse-ascs-ha-setting-ha-anf]

* ![RHEL][Logo_Linux] [Préparer l’infrastructure Azure pour la haute disponibilité de SAP ASCS/SCS - Configuration de GlusterFS sur RHEL][high-availability-guide-rhel-glusterfs]

* ![RHEL][Logo_Linux] [Préparer l’infrastructure Azure pour la haute disponibilité de SAP ASCS/SCS - Configuration de Pacemaker sur RHEL][high-availability-guide-rhel-pacemaker]

## <a name="installation-of-an-sap-netweaver-high-availability-system-in-azure"></a>Installation d’un système haute disponibilité SAP NetWeaver dans Azure

**Résumé :** les articles répertoriés ici présentent des exemples détaillés d’installation et de configuration d’un système SAP à haute disponibilité dans un cluster de basculement Windows Server et Linux Cluster Framework dans Azure.

**Date de mise à jour :** Mars 2019

* ![Logo Windows.][Logo_Windows] [Installer la haute disponibilité SAP NetWeaver à l’aide d’un cluster de basculement Windows et d’un **disque partagé** pour une instance SAP (A)SCS][sap-high-availability-installation-wsfc-shared-disk]

* ![Logo Windows.][Logo_Windows] [Installation de la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un **partage de fichiers** pour une instance SAP (A)SCS sur Azure][sap-high-availability-installation-wsfc-file-share]

* ![Logo Linux.][Logo_Linux] [Installation de la haute disponibilité SAP NetWeaver avec l’infrastructure de cluster SUSE Linux Enterprise Server pour une instance SAP (A)SCS][sap-suse-ascs-ha-sap-installation]

* ![Logo Linux.][Logo_Linux] [Installer la haute disponibilité SAP NetWeaver à l’aide d’un framework de cluster SUSE Linux Enterprise Server pour des instances SAP ASCS/SCS avec Azure NetApp Files][sap-suse-ascs-ha-sap-installation-anf]

* ![RHEL][Logo_Linux] [Installer SAP NetWeaver ASCS/SCS dans une configuration de haute disponibilité sur RHEL][sap-rhel-ascs-ha]

* ![RHEL][Logo_Linux] [Installer SAP NetWeaver ASCS/SCS dans une configuration de haute disponibilité sur RHEL avec Azure NetApp Files][sap-rhel-ascs-ha-sap-installation-anf]

