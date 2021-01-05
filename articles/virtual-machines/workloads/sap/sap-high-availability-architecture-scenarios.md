---
title: Scénarios et architecture de haute disponibilité des machines virtuelles Azure pour SAP NetWeaver
description: Scénarios et architecture de haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 887caaec-02ba-4711-bd4d-204a7d16b32b
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 02/26/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b409254abbde1c1f6156052c49a07e6cc09a4dfd
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94958776"
---
# <a name="high-availability-architecture-and-scenarios-for-sap-netweaver"></a>Scénarios et architecture de haute disponibilité pour SAP NetWeaver

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png


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
[sap-ascs-ha-multi-sid-wsfc-file-share]:sap-ascs-ha-multi-sid-wsfc-file-share.md
[sap-ascs-ha-multi-sid-wsfc-shared-disk]:sap-ascs-ha-multi-sid-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-suse-ascs-ha-anf]:high-availability-guide-suse-netapp-files.md
[sap-higher-availability]:sap-higher-availability-architecture-scenarios.md

[planning-guide]:planning-guide.md  
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77
[planning-guide-3.3.2]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d
[planning-guide-7.1]:planning-guide.md#3e9c3690-da67-421a-bc3f-12c520d99a30
[planning-guide-7]:planning-guide.md#96a77628-a05e-475d-9df3-fb82217e8f14
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3
[planning-guide-azure-premium-storage]:planning-guide.md#ff5ad0f9-f7f4-4022-9102-af07aef3bc92

[virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]:../../windows/sql/virtual-machines-windows-portal-sql-alwayson-availability-groups-manual.md
[virtual-machines-windows-portal-sql-alwayson-int-listener]:../../../azure-sql/virtual-machines/windows/availability-group-load-balancer-portal-configure.md

[sap-ha-bc-virtual-env-hyperv-vmware-white-paper]:https://scn.sap.com/docs/DOC-44415
[sap-ha-partner-information]:https://scn.sap.com/docs/DOC-8541
[azure-sla]:https://azure.microsoft.com/support/legal/sla/
[azure-virtual-machines-manage-availability]:../../windows/manage-availability.md
[azure-storage-redundancy]:https://azure.microsoft.com/documentation/articles/storage-redundancy/
[azure-storage-managed-disks-overview]:https://docs.microsoft.com/azure/storage/storage-managed-disks-overview

[planning-guide-figure-100]:media/virtual-machines-shared-sap-planning-guide/100-single-vm-in-azure.png
[planning-guide-figure-1300]:media/virtual-machines-shared-sap-planning-guide/1300-ref-config-iaas-for-sap.png
[planning-guide-figure-1400]:media/virtual-machines-shared-sap-planning-guide/1400-attach-detach-disks.png
[planning-guide-figure-1600]:media/virtual-machines-shared-sap-planning-guide/1600-firewall-port-rule.png
[planning-guide-figure-1700]:media/virtual-machines-shared-sap-planning-guide/1700-single-vm-demo.png
[planning-guide-figure-1900]:media/virtual-machines-shared-sap-planning-guide/1900-vm-set-vnet.png
[planning-guide-figure-200]:media/virtual-machines-shared-sap-planning-guide/200-multiple-vms-in-azure.png
[planning-guide-figure-2100]:media/virtual-machines-shared-sap-planning-guide/2100-s2s.png
[planning-guide-figure-2200]:media/virtual-machines-shared-sap-planning-guide/2200-network-printing.png
[planning-guide-figure-2300]:media/virtual-machines-shared-sap-planning-guide/2300-sapgui-stms.png
[planning-guide-figure-2400]:media/virtual-machines-shared-sap-planning-guide/2400-vm-extension-overview.png
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/planning-monitoring-overview-2502.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-2901]:media/virtual-machines-shared-sap-planning-guide/2901-azure-ha-sap-ha-md.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-3201]:media/virtual-machines-shared-sap-planning-guide/3201-sap-ha-with-sql-md.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

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

[virtual-machines-manage-availability]:../../manage-availability.md


## <a name="terminology-definitions"></a>Définitions de terminologie

**Haute disponibilité** : Fait référence à un ensemble de technologies qui réduisent les interruptions des services informatiques en assurant la continuité de leur activité par le biais de composants redondants, tolérants aux pannes ou protégés par basculement au sein du *même* centre de données. Dans notre cas, le centre de données se trouve dans une région Azure.

**Reprise d’activité** : Fait également référence à la baisse des interruptions des services informatiques et à leur récupération, mais sur *divers* centres de données pouvant se trouver à des centaines de kilomètres les uns des autres. Dans notre cas, les centres de données peuvent se trouver dans diverses régions Azure d’une même région géopolitique ou dans des emplacements définis par vous en tant que client.


## <a name="overview-of-high-availability"></a>Vue d’ensemble de la haute disponibilité
La haute disponibilité SAP dans Azure peut être divisée en trois types :

* **Haute disponibilité de l’infrastructure Azure** : 

    La haute disponibilité peut inclure, par exemple, le calcul (machines virtuelles), le réseau ou le stockage et ses avantages en termes d’augmentation de la disponibilité des applications SAP.

* **Utilisation du redémarrage de machine virtuelle de l’infrastructure Azure pour permettre une *plus haute disponibilité* des applications SAP** : 

    Si vous décidez de ne pas utiliser des fonctionnalités telles que le Clustering de basculement Windows Server (WSFC) ou Pacemaker sur Linux, le redémarrage de machine virtuelle Azure est utilisé. Il protège les systèmes SAP contre les temps d’arrêt planifiés ou non de l’infrastructure de serveur physique Azure et de la plateforme Azure sous-jacente globale.

* **Haute disponibilité de l’application SAP** : 

    Pour obtenir une haute disponibilité du système SAP complet, vous devez protéger tous les composants critiques du système SAP. Par exemple :
    * Serveurs d’applications SAP redondants.
    * Composants uniques. Un composant à point de défaillance unique (SPOF) tel qu’une instance SAP ASCS/SCS ou un système de gestion de base de données (SGBD) en est un exemple.

La haute disponibilité SAP dans Azure est différente de la haute disponibilité SAP dans un environnement physique local ou virtuel. Le document suivant, [Haute disponibilité et continuité des activités de SAP NetWeaver dans les environnements virtuels avec VMware et Hyper-V sur Microsoft Windows][sap-ha-bc-virtual-env-hyperv-vmware-white-paper], décrit les configurations à haute disponibilité SAP standard dans des environnements virtualisés sur Windows.

Il n’existe aucune configuration à haute disponibilité SAP intégrée pour Linux comme celle de Windows. Pour plus d’informations sur la haute disponibilité SAP locale pour Linux, consultez [Informations des partenaires pour la haute disponibilité][sap-ha-partner-information].

## <a name="azure-infrastructure-high-availability"></a>Haute disponibilité de l’infrastructure Azure

### <a name="sla-for-single-instance-virtual-machines"></a>SLA pour les machines virtuelles à instance unique

Il existe actuellement un contrat SLA de 99,9 % pour machine virtuelle unique avec le stockage Premium. Pour avoir une idée de ce à quoi peut ressembler la disponibilité d’une machine virtuelle unique, vous pouvez calculer le produit des divers [Contrats de niveau de service Azure][azure-sla] disponibles.

La base de calcul est de 30 jours par mois, ou 43 200 minutes. Par exemple, un temps d’arrêt de 0,05 % correspond à 21,6 minutes. Comme d’habitude, la disponibilité des divers services est calculée comme suit :

(Service de disponibilité #1/100) * (Service de disponibilité #2/100) * (Service de disponibilité #3/100) \*…

Par exemple :

(99,95/100) * (99,9/100) * (99,9/100) = 0,9975, soit une disponibilité globale de 99,75 %.

### <a name="multiple-instances-of-virtual-machines-in-the-same-availability-set"></a>Plusieurs instances de machines virtuelles dans le même groupe à haute disponibilité
Pour toutes les machines virtuelles comportant au moins deux instances déployées dans le même *groupe à haute disponibilité*, nous garantissons une connectivité à une instance minimum pendant au moins 99,95 % du temps.

Lorsque deux machines virtuelles ou plus font partie du même groupe à haute disponibilité, chaque machine virtuelle du groupe à haute disponibilité se voit attribuer un *domaine de mise à jour* et un *domaine d’erreur* par la plateforme Azure sous-jacente.

* Les **domaines de mise à jour** garantissent que les diverses machines virtuelles ne sont pas redémarrées en même temps lors de la maintenance planifiée d’une infrastructure Azure. Une seule machine virtuelle est redémarrée à la fois.

* Les **domaines d’erreur** garantissent que les machines virtuelles sont déployées sur des composants matériels qui ne partagent pas une source d’alimentation électrique ni un commutateur réseau communs. Lorsqu’un temps d’arrêt non planifié des serveurs, d’un commutateur réseau ou d’une source d’alimentation électrique se produit, une seule machine virtuelle est affectée.

Pour plus d’informations, consultez [Gérer la disponibilité des machines virtuelles Windows dans Azure][azure-virtual-machines-manage-availability].

Un groupe à haute disponibilité est utilisé pour la haute disponibilité des :

* Serveurs d’applications SAP redondants.  
* Clusters à deux nœuds ou plus (machines virtuelles, par exemple) qui protègent les SPOF comme une instance SAP ASCS/SCS ou un SGBD.


### <a name="azure-availability-zones"></a>Zones de disponibilité Azure
Azure est en train de déployer un concept de [zones de disponibilité Azure](../../../availability-zones/az-overview.md) sur différentes [régions Azure](https://azure.microsoft.com/global-infrastructure/regions/). Les régions Azure où sont proposées les zones de disponibilité ont plusieurs centres de données, chacun avec sa source d’alimentation, son système de refroidissement et son réseau. Nous offrons différentes zones au sein d’une même région Azure pour vous permettre de déployer des applications sur deux ou trois zones de disponibilité proposées. En imaginant que les problèmes d’alimentation et/ou de réseau n’affectent qu’une seule infrastructure de zone de disponibilité, le déploiement de votre application au sein d’une région Azure devrait toujours entièrement opérationnel. Avec peut-être quelques diminutions des fonctionnalités, car certaines machines virtuelles d’une zone pourraient être perdues. Mais celles des deux autres zones seraient toujours opérationnelles. Les régions Azure qui offrent des zones sont listées dans [Zones de disponibilité Azure](../../../availability-zones/az-overview.md).

Vous devez tenir compte de certains éléments quand vous utilisez des zones de disponibilité. Notamment :

- Vous ne pouvez pas déployer de groupes à haute disponibilité Azure dans une zone de disponibilité. Vous devez choisir une zone de disponibilité ou un groupe à haute disponibilité comme cadre de déploiement pour une machine virtuelle.
- Vous ne pouvez pas utiliser l’[équilibreur de charge de base](../../../load-balancer/load-balancer-overview.md) pour créer des solutions de cluster de basculement basées sur les services de cluster de basculement Windows ou Linux Pacemaker. À la place, vous devez utiliser la [référence SKU Standard Load Balancer d’Azure](../../../load-balancer/load-balancer-standard-availability-zones.md)
- Les zones de disponibilité Azure ne donnent aucune garantie d’une certaine distance entre les différentes zones dans une région
- La latence du réseau entre les différentes zones de disponibilité Azure dans les différentes régions Azure peut varier selon la région. Il est tout à fait possible que vous, en tant que client, puissiez raisonnablement exécuter la couche Application SAP déployée sur différentes zones parce que la latence du réseau d’une zone vers la machine virtuelle SGBD active est encore acceptable du point de vue de l’impact sur les processus métier. En revanche, il est possible aussi que la latence entre la machine virtuelle SGBD active dans une zone et une instance d’application SAP sur une machine virtuelle dans une autre zone soit trop intrusive et inacceptable pour les processus métier SAP. Par conséquent, les architectures de déploiement doivent être différentes avec une architecture active/active pour l’application ou une architecture active/passive si la latence est trop forte.
- L’utilisation de [disques managés Azure](https://azure.microsoft.com/services/managed-disks/) est obligatoire pour le déploiement dans les zones de disponibilité Azure 


### <a name="planned-and-unplanned-maintenance-of-virtual-machines"></a>Maintenance planifiée et non planifiée des machines virtuelles

Deux types d’événements de la plateforme Azure peuvent affecter la disponibilité de vos machines virtuelles :

* Les événements de **maintenance planifiée** sont des mises à jour périodiques appliquées par Microsoft à la plateforme Azure sous-jacente. Les mises à jour améliorent la fiabilité, les performances et la sécurité globales de l’infrastructure de plateforme sur laquelle vos machines virtuelles sont exécutées.

* Les événements de **maintenance non planifiée** ont lieu lorsque l’infrastructure physique ou matérielle sous-jacente de votre machine virtuelle connaît une défaillance. Cela peut comprendre les défaillances du réseau local, du disque local ou au niveau du rack. Lorsqu’une défaillance de ce type est détectée, la plateforme Azure migre automatiquement votre machine virtuelle du serveur physique défectueux qui l’héberge vers un serveur physique sain. De tels événements sont rares, mais ils peuvent entraîner un redémarrage de votre machine virtuelle.

Pour plus d’informations, consultez [Gérer la disponibilité des machines virtuelles Windows dans Azure][azure-virtual-machines-manage-availability].

### <a name="azure-storage-redundancy"></a>Redondance de Stockage Azure
Les données dans votre compte de stockage sont toujours répliquées pour garantir une durabilité, ainsi qu’une haute disponibilité, et répondre ainsi au contrat de niveau de service (SLA) Stockage Azure, y compris face aux défaillances matérielles temporaires.

Stockage Azure conservant trois images des données par défaut, l’utilisation de RAID 5 ou RAID 1 sur plusieurs disques Azure n’est pas nécessaire.

Pour plus d’informations, consultez l’article [Réplication de Stockage Azure][azure-storage-redundancy].

### <a name="azure-managed-disks"></a>Azure Disques managés
Les disques managés sont des ressources dans Azure Resource Manager qui peuvent être utilisées à la place des disques durs virtuels (VHD) stockés dans les comptes de stockage Azure. Les disques managés sont automatiquement alignés sur un groupe à haute disponibilité de la machine virtuelle à laquelle ils sont attachés. Ils améliorent la disponibilité de votre machine virtuelle et des services qui sont exécutés sur celle-ci.

Pour plus d’informations, consultez [Vue d’ensemble des disques managés Azure][azure-storage-managed-disks-overview].

Nous vous recommandons d’utiliser des disques managés, car ils simplifient le déploiement et la gestion de vos machines virtuelles.



## <a name="utilizing-azure-infrastructure-high-availability-to-achieve-higher-availability-of-sap-applications"></a>Utilisation de la haute disponibilité de l’infrastructure Azure pour permettre une *plus haute disponibilité* des applications SAP

Si vous décidez de ne pas utiliser des fonctionnalités telles que WSFC ou Pacemaker sur Linux (actuellement prises en charge pour SUSE Linux Enterprise Server [SLES] 12 et versions ultérieures uniquement), le redémarrage de la machine virtuelle Azure est utilisé. Il protège les systèmes SAP contre les temps d’arrêt planifiés ou non de l’infrastructure de serveur physique Azure et de la plateforme Azure sous-jacente globale.

Pour plus d’informations sur cette approche, consultez [Utiliser le redémarrage des machines virtuelles de l’infrastructure Azure pour une plus haute disponibilité du système SAP][sap-higher-availability].

## <a name="high-availability-of-sap-applications-on-azure-iaas"></a><a name="baed0eb3-c662-4405-b114-24c10a62954e"></a> Haute disponibilité des applications SAP sur Azure IaaS

Pour obtenir une haute disponibilité du système SAP complet, vous devez protéger tous les composants critiques du système SAP. Par exemple :
  * Serveurs d’applications SAP redondants.
  * Composants uniques. Un composant à point de défaillance unique (SPOF) tel qu’une instance SAP ASCS/SCS ou un système de gestion de base de données (SGBD) en est un exemple.

Les sections suivantes expliquent comment obtenir la haute disponibilité pour les trois composants critiques du système SAP.

### <a name="high-availability-architecture-for-sap-application-servers"></a>Architecture de haute disponibilité pour des serveurs d’applications SAP

> Cette section s’applique à :
>
> ![Logo Windows.][Logo_Windows] Windows et ![Logo Linux.][Logo_Linux] Linux
>

En règle générale, vous n’avez pas besoin d’une solution à haute disponibilité pour le serveur d’applications et les instances de dialogue SAP. La haute disponibilité s’obtient via la redondance, et vous configurez plusieurs instances de dialogue sur diverses instances de machines virtuelles Azure. Vous devez avoir au moins deux instances d’applications SAP installées dans deux instances de machines virtuelles Azure.

![Figure 1 : Serveur d’applications SAP à haute disponibilité][sap-ha-guide-figure-2000]

_**Figure 1 :** Serveur d’applications SAP à haute disponibilité_

Vous devez placer toutes les machines virtuelles qui hébergent des instances de serveur d’applications SAP dans le même groupe à haute disponibilité Azure. Un groupe à haute disponibilité Azure garantit que :

* Toutes les machines virtuelles ne font pas partie du même domaine de mise à jour.  
    Un domaine de mise à jour permet de s’assurer que les machines virtuelles ne soient pas mises à jour en même temps lors d’un temps d’arrêt de maintenance planifiée.

    La fonctionnalité de base, qui repose sur des domaines de mise à jour et d’erreur différents au sein d’une unité d’échelle Azure a déjà été présentée dans la section [Domaines de mise à jour][planning-guide-3.2.2].

* Toutes les machines virtuelles ne font pas partie du même domaine domaine d'erreur.  
    Un domaine d’erreur permet de s’assurer que les machines virtuelles soient déployées de sorte qu’aucun point de défaillance unique n’affecte la disponibilité de l’ensemble des machines virtuelles.

Le nombre de domaines de mise à jour et d’erreur pouvant être utilisé par un groupe à haute disponibilité Azure au sein d’une unité d’échelle Azure est limité. Si vous continuez d’ajouter des machines virtuelles à un groupe à haute disponibilité, deux machines virtuelles ou plus pourront se retrouver dans le même domaine d’erreur ou de mise à jour.

Si vous déployez plusieurs instances de serveurs d’applications SAP dans leurs machines virtuelles dédiées, en supposant que nous avons cinq domaines de mise à jour, nous obtenons l’image suivante. Le nombre maximum réel de domaines de mise à jour et d’erreur au sein d’un groupe à haute disponibilité peut changer à l’avenir :

![Figure 2 : Haute disponibilité des serveurs d’applications SAP dans un groupe à haute disponibilité Azure][planning-guide-figure-3000]
 _**Figure 2 :** Haute disponibilité des serveurs d’applications SAP dans un groupe à haute disponibilité Azure_

Pour plus d’informations, consultez [Gérer la disponibilité des machines virtuelles Windows dans Azure][azure-virtual-machines-manage-availability].

Pour plus d’informations, consultez la section [Groupes à haute disponibilité Azure][planning-guide-3.2.3] du document Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver.

**Disques non managés uniquement :** Comme le compte de stockage Azure constitue un point de défaillance unique potentiel, il est important de disposer d’au moins deux comptes de stockage Azure sur lesquels au moins deux machines virtuelles sont distribuées. Dans une configuration idéale, les disques de chaque machine virtuelle exécutant une instance de dialogue SAP sont déployés dans un compte de stockage différent.

> [!IMPORTANT]
> Nous vous recommandons vivement d’utiliser des disques managés Azure pour vos installations à haute disponibilité SAP. Étant donné que les disques managés s’alignent automatiquement avec le groupe à haute disponibilité de la machine virtuelle à laquelle ils sont joints, ils augmentent la disponibilité de votre machine virtuelle et des services exécutés sur celle-ci.  
>

### <a name="high-availability-architecture-for-an-sap-ascsscs-instance-on-windows"></a>Architecture de haute disponibilité pour une instance SAP ASCS/SCS sur Windows

> ![Logo Windows.][Logo_Windows] Windows
>

Vous pouvez utiliser une solution WSFC pour protéger l’instance SAP ASCS/SCS. La solution comporte deux variantes :

* **Clustering de l’instance SAP ASCS/SCS à l’aide de disques partagés en cluster** : Pour plus d’informations sur cette architecture, consultez [Clustering d’une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster][sap-high-availability-guide-wsfc-shared-disk].   

* **Clustering de l’instance SAP ASCS/SCS à l’aide d’un partage de fichiers** : Pour plus d’informations sur cette architecture, consultez [Clustering d’une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un partage de fichiers][sap-high-availability-guide-wsfc-file-share].

* **Clustering de l’instance SAP ASCS/SCS à l’aide d’un partage ANF SMB** : Pour plus d’informations sur cette architecture, consultez [Clustering d’une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un partage de fichiers ANF SMB](./high-availability-guide-windows-netapp-files-smb.md).

### <a name="high-availability-architecture-for-an-sap-ascsscs-instance-on-linux"></a>Architecture de haute disponibilité pour une instance SAP ASCS/SCS sur Linux

> ![Logo Linux.][Logo_Linux] Linux
> 
> Pour plus d’informations sur le clustering de l’instance SAP ASCS/SCS à l’aide de l’infrastructure de cluster SLES, consultez [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server pour les applications SAP][sap-suse-ascs-ha]. Pour connaître une autre architecture HA sur SLES ne nécessitant pas de système NFS à haute disponibilité, consultez le [Guide de haute disponibilité pour SAP NetWeaver sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP][sap-suse-ascs-ha-anf].

Pour plus d’informations sur le clustering de l’instance SAP ASCS/SCS à l’aide du framework de cluster Red Hat, consultez [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux](./high-availability-guide-rhel.md)


### <a name="sap-netweaver-multi-sid-configuration-for-a-clustered-sap-ascsscs-instance"></a>Configuration multi-SID de SAP NetWeaver pour une instance SAP ASCS/SCS en cluster

> ![Logo Windows.][Logo_Windows] Windows
> 
> Le multi-SID est pris en charge avec WSFC, à l'aide de partages de fichiers et disques partagés.
> 
> Pour plus d’informations sur l’architecture de haute disponibilité multi-SID sur Windows, consultez :

* [Haute disponibilité multi-SID de l’instance SAP ASCS/SCS pour le clustering de basculement Windows Server et le partage de fichiers][sap-ascs-ha-multi-sid-wsfc-file-share]

* [Haute disponibilité multi-SID de l’instance SAP ASCS/SCS pour le clustering de basculement Windows Server et un disque partagé][sap-ascs-ha-multi-sid-wsfc-shared-disk]

> ![Logo Linux.][Logo_Linux] Linux
> 
> Le clustering multi-SID est pris en charge sur les clusters Linux Pacemaker pour SAP ASCS/ERS, limité à **cinq** SID SAP sur le même cluster.
> Pour plus d’informations sur l’architecture de haute disponibilité multi-SID sur Linux, consultez :

* [Guide de haute disponibilité pour SAP NW sur les machines virtuelles Azure sur SLES pour les applications SAP multi-SID](./high-availability-guide-suse-multi-sid.md)
* [Guide de haute disponibilité pour SAP NW sur les machines virtuelles Azure sur RHEL pour les applications SAP multi-SID](./high-availability-guide-rhel-multi-sid.md)

### <a name="high-availability-dbms-instance"></a>Instance de SGBD à haute disponibilité

Le SGBD constitue également un point de défaillance unique d’un système SAP. Vous devez le protéger à l’aide d’une solution à haute disponibilité. La figure suivante montre une solution à haute disponibilité SQL Server AlwaysOn dans Azure, avec le clustering de basculement Windows Server et l’équilibreur de charge interne Azure. SQL Server AlwaysOn réplique les fichiers journaux et les données SGBD à l’aide de sa propre réplication de SGBD. Dans ce cas, vous n’avez pas besoin d’un disque partagé en cluster, ce qui simplifie l’ensemble de la configuration.

![Figure 3 : Exemple de SGBD SAP à haute disponibilité avec SQL Server Always On][sap-ha-guide-figure-2003]

_**Figure 3 :** Exemple de SGBD SAP à haute disponibilité avec SQL Server Always On_

Pour plus d’informations sur le clustering de SGBD SQL Server dans Azure à l’aide du modèle de déploiement Azure Resource Manager, consultez ces articles :

* [Configurer un groupe de disponibilité AlwaysOn dans des machines virtuelles Azure manuellement à l’aide de Resource Manager][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]

* [Configurer un équilibreur de charge interne Azure pour un groupe de disponibilité AlwaysOn dans Azure][virtual-machines-windows-portal-sql-alwayson-int-listener]

Pour plus d’informations sur le clustering du SGBD SAP HANA dans Azure à l’aide du modèle de déploiement Azure Resource Manager, consultez [Haute disponibilité de SAP HANA sur des machines virtuelles Azure][sap-hana-ha].
