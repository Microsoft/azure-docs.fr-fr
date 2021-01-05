---
title: 'SAP sur Azure : Guide de planification et d’implémentation'
description: Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver
author: MSSedusch
manager: juergent
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/17/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017, devx-track-azurecli
ms.openlocfilehash: bd45b0e1070efae7ae69a74ad96e1fa94a136006
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96019393"
---
# <a name="azure-virtual-machines-planning-and-implementation-for-sap-netweaver"></a>Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1139904]:https://launchpad.support.sap.com/#/notes/1139904
[1173395]:https://launchpad.support.sap.com/#/notes/1173395
[1245200]:https://launchpad.support.sap.com/#/notes/1245200
[1409604]:https://launchpad.support.sap.com/#/notes/1409604
[1558958]:https://launchpad.support.sap.com/#/notes/1558958
[1585981]:https://launchpad.support.sap.com/#/notes/1585981
[1588316]:https://launchpad.support.sap.com/#/notes/1588316
[1590719]:https://launchpad.support.sap.com/#/notes/1590719
[1597355]:https://launchpad.support.sap.com/#/notes/1597355
[1605680]:https://launchpad.support.sap.com/#/notes/1605680
[1619720]:https://launchpad.support.sap.com/#/notes/1619720
[1619726]:https://launchpad.support.sap.com/#/notes/1619726
[1619967]:https://launchpad.support.sap.com/#/notes/1619967
[1750510]:https://launchpad.support.sap.com/#/notes/1750510
[1752266]:https://launchpad.support.sap.com/#/notes/1752266
[1757924]:https://launchpad.support.sap.com/#/notes/1757924
[1757928]:https://launchpad.support.sap.com/#/notes/1757928
[1758182]:https://launchpad.support.sap.com/#/notes/1758182
[1758496]:https://launchpad.support.sap.com/#/notes/1758496
[1772688]:https://launchpad.support.sap.com/#/notes/1772688
[1814258]:https://launchpad.support.sap.com/#/notes/1814258
[1882376]:https://launchpad.support.sap.com/#/notes/1882376
[1909114]:https://launchpad.support.sap.com/#/notes/1909114
[1922555]:https://launchpad.support.sap.com/#/notes/1922555
[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1941500]:https://launchpad.support.sap.com/#/notes/1941500
[1956005]:https://launchpad.support.sap.com/#/notes/1956005
[1973241]:https://launchpad.support.sap.com/#/notes/1973241
[1984787]:https://launchpad.support.sap.com/#/notes/1984787
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2002167]:https://launchpad.support.sap.com/#/notes/2002167
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2039619]:https://launchpad.support.sap.com/#/notes/2039619
[2069760]:https://launchpad.support.sap.com/#/notes/2069760
[2121797]:https://launchpad.support.sap.com/#/notes/2121797
[2134316]:https://launchpad.support.sap.com/#/notes/2134316
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2191498]:https://launchpad.support.sap.com/#/notes/2191498
[2233094]:https://launchpad.support.sap.com/#/notes/2233094
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[azure-cli]:../../../cli-install-nodejs.md
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azure/
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide.md
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8

[dbms-guide-figure-100]:media/virtual-machines-shared-sap-dbms-guide/100_storage_account_types.png
[dbms-guide-figure-200]:media/virtual-machines-shared-sap-dbms-guide/200-ha-set-for-dbms-ha.png
[dbms-guide-figure-300]:media/virtual-machines-shared-sap-dbms-guide/300-reference-config-iaas.png
[dbms-guide-figure-400]:media/virtual-machines-shared-sap-dbms-guide/400-sql-2012-backup-to-blob-storage.png
[dbms-guide-figure-500]:media/virtual-machines-shared-sap-dbms-guide/500-sql-2012-backup-to-blob-storage-different-containers.png
[dbms-guide-figure-600]:media/virtual-machines-shared-sap-dbms-guide/600-iaas-maxdb.png
[dbms-guide-figure-700]:media/virtual-machines-shared-sap-dbms-guide/700-livecach-prod.png
[dbms-guide-figure-800]:media/virtual-machines-shared-sap-dbms-guide/800-azure-vm-sap-content-server.png
[dbms-guide-figure-900]:media/virtual-machines-shared-sap-dbms-guide/900-sap-cache-server-on-premises.png

[deployment-guide]:deployment-guide.md
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8

[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d
[deployment-guide-figure-100]:media/virtual-machines-shared-sap-deployment-guide/100-deploy-vm-image.png
[deployment-guide-figure-1000]:media/virtual-machines-shared-sap-deployment-guide/1000-service-properties.png
[deployment-guide-figure-11]:deployment-guide.md#figure-11
[deployment-guide-figure-1100]:media/virtual-machines-shared-sap-deployment-guide/1100-azperflib.png
[deployment-guide-figure-1200]:media/virtual-machines-shared-sap-deployment-guide/1200-cmd-test-login.png
[deployment-guide-figure-1300]:media/virtual-machines-shared-sap-deployment-guide/1300-cmd-test-executed.png
[deployment-guide-figure-14]:deployment-guide.md#figure-14
[deployment-guide-figure-1400]:media/virtual-machines-shared-sap-deployment-guide/1400-azperflib-error-servicenotstarted.png
[deployment-guide-figure-300]:media/virtual-machines-shared-sap-deployment-guide/300-deploy-private-image.png
[deployment-guide-figure-400]:media/virtual-machines-shared-sap-deployment-guide/400-deploy-using-disk.png
[deployment-guide-figure-5]:deployment-guide.md#figure-5
[deployment-guide-figure-50]:media/virtual-machines-shared-sap-deployment-guide/50-forced-tunneling-suse.png
[deployment-guide-figure-500]:media/virtual-machines-shared-sap-deployment-guide/500-install-powershell.png
[deployment-guide-figure-6]:deployment-guide.md#figure-6
[deployment-guide-figure-600]:media/virtual-machines-shared-sap-deployment-guide/600-powershell-version.png
[deployment-guide-figure-7]:deployment-guide.md#figure-7
[deployment-guide-figure-700]:media/virtual-machines-shared-sap-deployment-guide/700-install-powershell-installed.png
[deployment-guide-figure-760]:media/virtual-machines-shared-sap-deployment-guide/760-azure-cli-version.png
[deployment-guide-figure-900]:media/virtual-machines-shared-sap-deployment-guide/900-cmd-update-executed.png
[deployment-guide-figure-azure-cli-installed]:deployment-guide.md#402488e5-f9bb-4b29-8063-1c5f52a892d0
[deployment-guide-figure-azure-cli-version]:deployment-guide.md#0ad010e6-f9b5-4c21-9c09-bb2e5efb3fda
[deployment-guide-install-vm-agent-windows]:deployment-guide.md#b2db5c9a-a076-42c6-9835-16945868e866
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b

[deploy-template-cli]:../../../resource-group-template-deploy-cli.md
[deploy-template-portal]:../../../resource-group-template-deploy-portal.md
[deploy-template-powershell]:../../../resource-group-template-deploy.md

[dr-guide-classic]:https://go.microsoft.com/fwlink/?LinkID=521971

[getting-started]:get-started.md
[getting-started-dbms]:get-started.md#1343ffe1-8021-4ce6-a08d-3a1553a4db82
[getting-started-deployment]:get-started.md#6aadadd2-76b5-46d8-8713-e8d63630e955
[getting-started-planning]:get-started.md#3da0389e-708b-4e82-b2a2-e92f132df89c

[getting-started-windows-classic]:../../virtual-machines-windows-classic-sap-get-started.md
[getting-started-windows-classic-dbms]:../../virtual-machines-windows-classic-sap-get-started.md#c5b77a14-f6b4-44e9-acab-4d28ff72a930
[getting-started-windows-classic-deployment]:../../virtual-machines-windows-classic-sap-get-started.md#f84ea6ce-bbb4-41f7-9965-34d31b0098ea
[getting-started-windows-classic-dr]:../../virtual-machines-windows-classic-sap-get-started.md#cff10b4a-01a5-4dc3-94b6-afb8e55757d3
[getting-started-windows-classic-ha-sios]:../../virtual-machines-windows-classic-sap-get-started.md#4bb7512c-0fa0-4227-9853-4004281b1037
[getting-started-windows-classic-planning]:../../virtual-machines-windows-classic-sap-get-started.md#f2a5e9d8-49e4-419e-9900-af783173481c

[ha-guide-classic]:https://go.microsoft.com/fwlink/?LinkId=613056

[install-extension-cli]:virtual-machines-linux-enable-aem.md

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

[msdn-set-Azvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

[planning-guide]:planning-guide.md
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f
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

[powershell-install-configure]:https://docs.microsoft.com/powershell/azure/install-az-ps
[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/management/overview.md
[resource-groups-networking]:../../../networking/networking-overview.md
[sap-pam]:https://support.sap.com/pam
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[storage-azure-cli]:../../../storage/common/storage-azure-cli.md
[storage-azure-cli-copy-blobs]:../../../storage/common/storage-azure-cli.md#copy-blobs
[storage-introduction]:../../../storage/common/storage-introduction.md
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md
[storage-premium-storage-preview-portal]:../../disks-types.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/scalability-targets-standard-accounts.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-Az-versus-azuresm]:virtual-machines-linux-compare-deployment-models.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines/windows/index.yml
[virtual-machines-linux-agent-user-guide]:../../extensions/agent-linux.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../extensions/agent-windows.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-windows-capture-image-resource-manager]:../../windows/capture-image.md
[virtual-machines-windows-capture-image]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-windows-capture-image-prepare-the-vm-for-image-capture]:../../virtual-machines-windows-generalize-vhd.md
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-create-upload-vhd-oracle]:../../linux/oracle-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#format-and-mount-the-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability]:../../linux/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:./../../windows/sql/virtual-machines-windows-sql-high-availability-dr.md
[virtual-machines-sql-server-infrastructure-services]:./../../windows/sql/virtual-machines-windows-sql-server-iaas-overview.md
[virtual-machines-sql-server-performance-best-practices]:./../../windows/sql/virtual-machines-windows-sql-performance.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/documentation/templates/sql-server-2014-alwayson-dsc/
[virtual-network-deploy-multinic-arm-cli]:../../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics-windows]:../../windows/multiple-nics.md
[virtual-networks-multiple-nics-linux]:../../linux/multiple-nics.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-about-vpngateways.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md
[capture-image-linux-step-2-create-vm-image]:../../linux/capture-image.md#step-2-create-vm-image



Microsoft Azure permet aux entreprises d’acquérir des ressources de calcul et de stockage rapidement, sans longs cycles d’acquisition. Le service Machines virtuelles Azure permet aux entreprises de déployer dans Azure des applications classiques, comme les applications basées sur SAP NetWeaver, et d’étendre leur fiabilité et leur disponibilité même sans avoir d’autres ressources disponibles localement. Le service Machines virtuelles Azure prend également en charge la connectivité intersite, ce qui permet aux entreprises d’intégrer de manière active Machines virtuelles Azure dans leurs domaines locaux, leurs clouds privés et leur paysage SAP.
Ce livre blanc décrit les fondamentaux de Machines virtuelles Microsoft Azure et présente les considérations à prendre en compte en matière de planification et d’implémentation pour les installations SAP NetWeaver dans Azure. Il est donc primordial de lire ce document avant de commencer à effectuer des déploiements de SAP NetWeaver dans Azure.
Ce document vient compléter la documentation sur l’installation SAP et les notes SAP, qui représentent les ressources à consulter pour installer et déployer les logiciels SAP sur des plateformes données.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

## <a name="summary"></a>Résumé
Cloud Computing est un terme largement utilisé parce qu’il devient de plus en plus important dans le secteur de l’informatique, que ce soit pour les petites entreprises ou les grandes multinationales.

Microsoft Azure est la plateforme de services cloud de Microsoft, qui offre un large éventail de nouvelles possibilités. Désormais, les clients peuvent approvisionner et déprovisionner rapidement des applications en tant que service dans le cloud, en échappant ainsi aux éventuelles restrictions techniques ou budgétaires. Au lieu de consacrer du temps et de l’argent à l’infrastructure matérielle, les entreprises peuvent se concentrer sur l’application, les processus de travail et leurs avantages pour les clients et les utilisateurs.

Avec le service Machines virtuelles Microsoft Azure, Microsoft propose une infrastructure complète sous forme d’infrastructure en tant que service (IaaS). Les applications basées sur SAP NetWeaver sont prises en charge sur Machines virtuelles Azure (IaaS). Ce livre blanc décrit comment planifier et implémenter des applications basées sur SAP NetWeaver dans Microsoft Azure en tant que plateforme de prédilection.

Ce document met l’accent sur deux aspects principaux :

* La première partie décrit deux modèles de déploiement pris en charge pour les applications basées sur SAP NetWeaver dans Azure. La gestion générale d’Azure avec les déploiements SAP est également abordée.
* La deuxième partie explique en détail l’implémentation des différents scénarios mentionnés dans la première partie.

Pour obtenir des ressources supplémentaires, consultez le chapitre [Ressources][planning-guide-1.2] de ce document.

### <a name="definitions-upfront"></a>Préambule : définitions
Les termes suivants sont utilisés dans le document :

* IaaS : Infrastructure as a Service
* PaaS : Platform as a service
* SaaS : Software as a Service
* Composant SAP : application SAP individuelle telle que ECC, BW, Solution Manager ou S/4HANA.  Les composants SAP peuvent être basés sur des technologies ABAP ou Java traditionnelles ou une application non basée sur NetWeaver telle que Business Objects.
* Environnement SAP : un ou plusieurs composants SAP regroupés de manière logique pour exécuter une fonction métier telle que le développement, l’assurance qualité, la formation, la récupération d’urgence ou la production.
* Paysage SAP : ce terme fait référence à l’ensemble des ressources SAP dans le paysage informatique d’un client. Le paysage SAP comprend tous les environnements de production et les autres types d’environnements.
* Système SAP : ensemble couche SGBD/couche Application, tel que celui d’un système de développement SAP ERP, d’un système de test SAP BW, d’un système de production SAP CRM, etc. Dans les déploiements Azure, il n’est pas possible de répartir ces deux couches entre des sites locaux et Azure. Cela signifie qu’un système SAP est déployé localement ou dans Azure. Toutefois, vous pouvez déployer les différents systèmes d’un paysage SAP dans Azure ou en local. Par exemple, vous pouvez déployer les systèmes de test et de développement SAP CRM dans Azure et le système de production SAP CRM en local.
* Intersite ou hybride : décrit un scénario dans lequel les machines virtuelles sont déployées sur un abonnement Azure qui dispose d’une connectivité de site à site, multisite ou ExpressRoute entre les centres de données locaux et Azure. Dans la documentation Azure courante, ces types de déploiements sont également décrits comme des scénarios intersites ou hybrides. La connexion a pour but d’étendre les domaines locaux, le répertoire Active Directory/OpenLDAP local et le DNS local à Azure. Le paysage local est étendu aux ressources Azure de l’abonnement. Grâce à cette extension, les machines virtuelles peuvent faire partie du domaine local. Les utilisateurs du domaine local peuvent accéder aux serveurs et exécuter des services sur ces machines virtuelles (tels que les services SGBD). La communication et la résolution de noms entre les machines virtuelles déployées en local et les machines virtuelles déployées dans Azure sont possibles. C’est le cas le plus courant, et presque le seul, de déploiement de ressources SAP dans Azure. Pour plus d’informations, consultez [cet article][vpn-gateway-cross-premises-options] et [celui-ci][vpn-gateway-site-to-site-create].
* Extension de la supervision Azure, Supervision améliorée et Extension Azure pour SAP : ces termes désignent tous la même chose. Ils désignent une extension de machine virtuelle que vous devez déployer pour fournir des données sur l’infrastructure Azure à SAP Host Agent. Dans les notes SAP, vous pouvez trouver des références à « Extension de la supervision » ou à « Supervision améliorée ». Dans Azure, nous utilisons le terme « **Extension Azure pour SAP** ».

> [!NOTE]
> Les déploiements intersites ou hybrides de systèmes SAP dans lesquels des machines virtuelles Azure exécutant des systèmes SAP font partie d’un domaine local sont pris en charge pour les systèmes SAP de production. Les configurations hybrides ou intersites sont prises en charge pour le déploiement d’éléments ou de l’intégralité des paysages SAP dans Azure. Ces machines virtuelles doivent faire partie du domaine et de l’annuaire ADS/OpenLDAP locaux même quand l’intégralité du paysage SAP est exécutée dans Azure.
>
>



### <a name="resources"></a><a name="e55d1e22-c2c8-460b-9897-64622a34fdff"></a>Ressources
Le point d’entrée de la charge de travail SAP se trouve [ici](./get-started.md) dans la documentation Azure. De là, vous pouvez consulter un grand nombre d’articles traitant des sujets suivants :

- SAP NetWeaver et Business One sur Azure
- Guides sur le SGBD SAP pour les différents systèmes SGBD dans Azure
- Haute disponibilité et récupération d’urgence pour la charge de travail SAP sur Azure
- Guide relatif à l’exécution de SAP HANA sur Azure
- Guide relatif aux Grandes instances Azure HANA pour le SGBD SAP HANA


> [!IMPORTANT]
> Le cas échéant, un lien pointant vers le guide d’installation de SAP concerné ou une autre documentation SAP est utilisé (référence InstGuide-01 à l’adresse <http://service.sap.com/instguides>). Lorsqu’il s’agit de la configuration requise, du processus d’installation ou des détails d’une fonctionnalité SAP spécifique, la documentation et les guides SAP doivent toujours être lus avec attention, car les documents Microsoft traitent uniquement des tâches spécifiques pour les systèmes SAP installés et utilisés dans une machine virtuelle Microsoft Azure.
>
>

Les notes SAP suivantes sont associées à la rubrique SAP sur Azure :

| Numéro de la note | Intitulé |
| --- | --- |
| [1928533] |Applications SAP sur Azure : Produits et tailles pris en charge |
| [2015553] |SAP sur Microsoft Azure : prérequis pour le support |
| [1999351] |Résolution des problèmes de surveillance Azure améliorée pour SAP |
| [2178632] |Métriques de surveillance clés pour SAP sur Microsoft Azure |
| [1409604] |Virtualisation sur Windows : supervision améliorée |
| [2191498] |SAP sur Linux avec Azure : supervision améliorée |
| [2243692] |Linux sur machine virtuelle Microsoft Azure (IaaS) : problèmes de licence SAP |
| [1984787] |SUSE LINUX Enterprise Server 12 Notes d'installation |
| [2002167] |Red Hat Enterprise Linux 7.x : installation et mise à niveau |
| [2069760] |Installation et mise à niveau SAP pour Oracle Linux 7.x |
| [1597355] |Recommandations relatives à l’espace d’échange pour Linux |

Consultez également le [Wiki SCN](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) qui regroupe toutes les notes SAP pour Linux.

Les limitations générales par défaut et les limitations maximales des abonnements Azure sont exposées dans [cet article][azure-resource-manager/management/azure-subscription-service-limits-subscription].

## <a name="possible-scenarios"></a>Scénarios possibles
SAP est souvent considéré comme une des applications les plus critiques au sein des entreprises. L’architecture et les opérations de ces applications sont généralement complexes, vous devez donc bien respecter les exigences relatives à la disponibilité et aux performances.

Par conséquent, les entreprises doivent choisir avec soin le fournisseur de cloud auquel elles souhaitent confier l’exécution de leurs processus métier critiques. Azure constitue la plateforme de cloud public idéale pour les applications SAP et processus métiers critiques pour l’entreprise. Compte tenu de la grande variété d’infrastructures Azure, la quasi-totalité des systèmes SAP NetWeaver et S/4HANA existants peuvent être hébergés dans Azure aujourd’hui. Azure fournit aux machines virtuelles plusieurs téraoctets de mémoire et plus de 200 UC. En outre, Azure propose [HANA – Grandes instances](./hana-overview-architecture.md), qui permet des déploiements HANA avec scale-up d’un maximum de 24 To et des déploiements SAP HANA avec scale-out d’un maximum de 120 To. Il est possible de dire aujourd’hui que presque tous les scénarios SAP locaux peuvent également être exécutés dans Azure.

Pour obtenir une description approximative des scénarios et quelques scénarios non pris en charge, consultez le document [Charge de travail SAP sur les machines virtuelles Azure – Scénarios pris en charge](./sap-planning-supported-configurations.md).

Vérifiez ces scénarios et certaines des conditions qui ont été désignées comme non prises en charge dans la documentation de référence tout au long de la planification et du développement de l’architecture que vous souhaitez déployer dans Azure.

Dans l’ensemble, le modèle de déploiement le plus courant est un scénario intersite comme celui présenté.

![Réseau privé virtuel avec connectivité de site à site (intersite)][planning-guide-figure-300]

La raison pour laquelle de nombreux clients appliquent un modèle de déploiement intersite est qu’il est plus transparent pour toutes les applications d’étendre l’environnement local à Azure à l’aide d’Azure ExpressRoute et de traiter Azure comme un centre de donnée virtuel. À mesure que de plus en plus de ressources sont déplacées dans Azure, l’infrastructure déployée Azure et l’infrastructure réseau se développeront et les ressources locales diminueront en conséquence. Tout est transparent pour les utilisateurs et les applications.

Pour déployer correctement les systèmes SAP dans Azure IaaS ou IaaS, il est essentiel de comprendre les différences majeures qui existent entre les offres des hébergeurs ou fournisseurs traditionnels et les offres IaaS. Alors que l’hébergeur ou l’infogérant traditionnels proposent d’adapter l’infrastructure (type de serveur, stockage et réseau) à la charge de travail que le client souhaite héberger, il incombe au client ou au partenaire de caractériser la charge de travail et de choisir les composants Azure de machines virtuelles, de stockage et de réseau appropriés pour les déploiements IaaS.

Afin de collecter des données pour la planification de votre déploiement dans Azure, les étapes suivantes sont importantes :

- Évaluer les produits SAP pris en charge dans les machines virtuelles Azure
- Évaluer les versions de système d’exploitation prises en charge avec des machines virtuelles Azure spécifiques pour ces produits SAP
- Évaluer les versions de SGBD prises en charge pour vos produits SAP avec des machines virtuelles Azure spécifiques
- Déterminer si certaines des versions requises du système d’exploitation/SGBD vous obligent à effectuer une mise en production SAP, une mise à niveau du package de support et des mises à niveau du noyau pour obtenir une configuration prise en charge
- Déterminer si vous devez passer à des systèmes d’exploitation différents afin de procéder au déploiement sur Azure

Pour plus d’informations sur les composants SAP pris en charge sur Azure, les unités d’infrastructure Azure prises en charge et les versions de système d’exploitation et du SGBD associées, consultez l’article [Logiciels SAP pris en charge pour les déploiements Azure](./sap-supported-product-on-azure.md). Les résultats obtenus lors de l’évaluation des versions valides de SAP, du système d’exploitation et des versions du SGBD ont un impact important sur les efforts de migration des systèmes SAP vers Azure. Les résultats de cette évaluation vont définir si les efforts de préparation seront importants dans les cas où des mises à niveau de versions SAP ou des changements de systèmes d’exploitation sont nécessaires.


## <a name="azure-regions"></a><a name="be80d1b9-a463-4845-bd35-f4cebdb5424a"></a>Régions Azure
Les services Azure de Microsoft sont regroupés dans des régions Azure. Une région Azure est un centre de données (ou un ensemble de centres de données) contenant le matériel et l'infrastructure permettant d'exécuter et d'héberger les différents services Azure. Cette infrastructure comprend un grand nombre de nœuds qui font office de nœuds de calcul ou de nœuds de stockage, ou qui exécutent des fonctionnalités réseau.

Pour obtenir la liste des différentes régions Azure, consultez l'article [Zones géographiques Azure](https://azure.microsoft.com/global-infrastructure/geographies/). Les services disponibles varient d'une région Azure à une autre. En fonction du produit SAP que vous souhaitez exécuter, ainsi que du système d'exploitation et du SGBD associés, vous pouvez vous retrouver dans une situation où une région donnée n'offre pas les types de machines virtuelles dont vous avez besoin. C'est notamment le cas pour l'exécution de SAP HANA, qui nécessite généralement des machines virtuelles de la série M/Mv2. Ces familles de machines virtuelles ne sont déployées que dans un sous-ensemble de régions. Consultez le site [Produits disponibles par région](https://azure.microsoft.com/global-infrastructure/services/) pour en savoir plus sur les machines virtuelles, types de machines virtuelles, types de stockage Azure et autres services Azure disponibles dans les différentes régions. Lorsque vous entamez votre planification en ayant à l'esprit une région principale et une région secondaire particulières, vous devez d'abord vous assurer que les services nécessaires y sont disponibles.

### <a name="availability-zones"></a>Zones de disponibilité
Différentes régions Azure ont implémenté un concept appelé Zones de disponibilité. Les zones de disponibilité sont des emplacements physiquement séparés au sein d’une région Azure. Chaque zone de disponibilité est composée d’un ou de plusieurs centres de données équipés d’une alimentation, d’un refroidissement et d’un réseau indépendants. Par exemple, le déploiement de deux machines virtuelles dans deux zones de disponibilité Azure et l'implémentation d'une infrastructure haute disponibilité pour votre système de gestion de base de données (SGBD) SAP ou les services centraux SAP vous offrent le meilleur contrat de niveau de service (SLA) d'Azure. Pour plus d'informations sur ce contrat SLA pour machines virtuelles d'Azure, consultez la dernière version de [SLA pour machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/). Compte tenu du développement et de l'extension rapides des régions Azure au cours des dernières années, la topologie des régions Azure, le nombre de centres de données physiques, la distance entre ces centres de données et la distance entre les zones de disponibilité Azure peuvent varier. De même que le temps de réponse du réseau.

Le principe des zones de disponibilité ne s'applique pas au service [HANA - Grandes instances](./hana-overview-architecture.md). Consultez l'article [SLA pour SAP HANA - Grandes instances](https://azure.microsoft.com/support/legal/sla/sap-hana-large/) pour accéder aux contrats de niveau de service relatifs à HANA - Grandes instances.


### <a name="fault-domains"></a><a name="df49dc09-141b-4f34-a4a2-990913b30358"></a>Domaines d’erreur
Les domaines d’erreur représentent une unité physique de défaillance, très étroitement liée à l’infrastructure physique contenue dans les centres de données, et tandis qu’un rack ou une lame physique peut être considéré comme un domaine d’erreur, il n’existe aucun mappage un à un direct entre les deux.

Lorsque vous déployez plusieurs machines virtuelles dans le cadre d'un système SAP dans Microsoft Azure Virtual Machine Services, vous pouvez inciter le contrôleur de structure Azure à déployer votre application dans différents domaines d'erreur, ce qui permet de répondre à des exigences plus élevées en matière de SLA de disponibilité. Toutefois, la répartition des domaines d’erreur dans une unité d’échelle Azure (collection de centaines de nœuds de calcul ou de nœuds de stockage et réseau) ou l’attribution de machines virtuelles à un domaine d’erreur spécifique est un élément sur lequel vous n’avez pas de contrôle direct. Pour gérer le contrôleur de structure Azure afin qu’il déploie un ensemble de machines virtuelles sur différents domaines d’erreur, vous devez attribuer un groupe à haute disponibilité Azure aux machines virtuelles au moment du déploiement. Pour plus d’informations sur les groupes à haute disponibilité Azure, consultez le chapitre [Groupes à haute disponibilité Azure][planning-guide-3.2.3] de ce document.


### <a name="upgrade-domains"></a><a name="fc1ac8b2-e54a-487c-8581-d3cc6625e560"></a>Domaines de mise à niveau
Les domaines de mise à niveau représentent une unité logique qui vous aide à déterminer comment est mise à jour une machine virtuelle au sein d’un système SAP composé d’instances SAP s’exécutant sur plusieurs machines virtuelles. Lorsqu’une mise à niveau a lieu, Microsoft Azure exécute le processus de mise à jour de ces domaines de mise à niveau un par un. En répartissant les machines virtuelles dans différents domaines de mise à niveau au moment du déploiement, vous pouvez protéger, en partie, votre système SAP des interruptions potentielles. Pour forcer Azure à déployer les machines virtuelles d’un système SAP réparties dans différents domaines de mise à niveau, vous devez définir un attribut spécifique au moment du déploiement de chaque machine virtuelle. Comme pour les domaines d’erreur, une unité d’échelle Azure est divisée en plusieurs domaines de mise à niveau. Pour gérer le contrôleur de structure Azure afin qu’il déploie un ensemble de machines virtuelles sur différents domaines de mise à niveau, vous devez attribuer un groupe à haute disponibilité Azure aux machines virtuelles au moment du déploiement. Pour plus d’informations sur les groupes à haute disponibilité Azure, consultez le chapitre [Groupes à haute disponibilité Azure][planning-guide-3.2.3] ci-dessous.


### <a name="azure-availability-sets"></a><a name="18810088-f9be-4c97-958a-27996255c665"></a>Groupes à haute disponibilité Azure
Les machines virtuelles Azure au sein d’un seul groupe à haute disponibilité Azure sont réparties par le contrôleur de structure Azure dans différents domaines d’erreur et de mise à niveau. L’objectif de la répartition dans différents domaines d’erreur et de mise à niveau consiste à empêcher l’arrêt de toutes les machines virtuelles d’un système SAP en cas de maintenance ou de défaillance de l’infrastructure dans un domaine d’erreur. Par défaut, les machines virtuelles ne font pas partie d’un groupe à haute disponibilité. La présence d’une machine virtuelle dans un groupe à haute disponibilité est définie au moment du déploiement ou par la suite dans le cadre d’une reconfiguration et d’un redéploiement de machine virtuelle.

Pour comprendre le concept des groupes à haute disponibilité Azure et leur relation avec les domaines d'erreur et de mise à niveau, consultez [cet article][virtual-machines-manage-availability].

Lorsque vous définissez des groupes à haute disponibilité et essayez de combiner des machines virtuelles de différentes familles au sein d'un même groupe, vous pouvez rencontrer des problèmes qui vous empêchent d'inclure un certain type de machine virtuelle dans ce groupe. Cela est dû au fait que le groupe à haute disponibilité est lié à une unité d'échelle qui contient un certain type d'hôtes de calcul. Et un certain type d'hôte de calcul ne peut exécuter que certains types de familles de machines virtuelles. Par exemple, si vous créez un groupe à haute disponibilité, déployez la première machine virtuelle dans celui-ci, choisissez un type de machine virtuelle de la famille Esv3, puis essayez de déployer une deuxième machine virtuelle appartenant à la famille M, la seconde attribution sera refusée. Cela est dû au fait que les machines virtuelles de la famille Esv3 ne s'exécutent pas sur le même matériel hôte que les machines virtuelles de la famille M. Le même problème peut se produire lorsque vous essayez de redimensionner des machines virtuelles et de déplacer une machine virtuelle de la famille Esv3 vers un type de machine virtuelle de la famille M. Dans le cas d'un redimensionnement vers une famille de machines virtuelles qui ne peut pas être hébergée sur le même matériel hôte, vous devez arrêter toutes les machines virtuelles de votre groupe à haute disponibilité et les redimensionner pour qu'elles puissent s'exécuter sur l'autre type de machine hôte. Pour les SLA des machines virtuelles déployées dans le groupe à haute disponibilité, consultez l'article [SLA pour machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines/).

Le principe de groupe à haute disponibilité et de domaine de mise à jour et d'erreur associé ne s'applique pas au service [HANA - Grandes instances](./hana-overview-architecture.md). Consultez l'article [SLA pour SAP HANA - Grandes instances](https://azure.microsoft.com/support/legal/sla/sap-hana-large/) pour accéder aux contrats de niveau de service relatifs à HANA - Grandes instances.

> [!IMPORTANT]
> Les concepts de Zones de disponibilité Azure et de groupes à haute disponibilité Azure sont incompatibles. Cela signifie que vous pouvez déployer deux ou plusieurs machines virtuelles dans une zone de disponibilité ou dans un groupe à haute disponibilité Azure, mais pas dans les deux.

### <a name="azure-paired-regions"></a>Régions appairées Azure
Azure propose des paires de régions Azure fixes, entre lesquelles la réplication de certaines données est activée. L’appariement des régions est documenté dans l’article [Continuité d’activité et reprise d’activité (BCDR) : régions jumelées d’Azure](../../../best-practices-availability-paired-regions.md). Comme l’explique l’article, la réplication des données est liée aux types de stockage Azure que vous pouvez configurer de sorte qu’ils soient répliqués dans la région appairée. Consultez également l’article [Redondance du stockage dans une région secondaire](../../../storage/common/storage-redundancy.md#redundancy-in-a-secondary-region). Les types de stockage qui autorisent une telle réplication sont ceux qui ne sont pas adaptés à la charge de travail SGBD. Par conséquent, l’utilisation de la réplication du stockage Azure se limite au Stockage Blob Azure (par exemple, à des fins de sauvegarde) ou à d’autres scénarios de stockage à latence élevée. Lorsque vous recherchez des régions appairées ainsi que les services que vous souhaitez utiliser comme région primaire ou secondaire, il peut arriver que les services Azure ou les types de machines virtuelles envisagés pour votre région primaire ne soient pas disponibles dans la région appairée. Vous pouvez également rencontrer une situation dans laquelle la région appairée Azure n’est pas acceptable pour des raisons de conformité des données. Dans ce cas, il est nécessaire d’utiliser une région non appairée comme région de récupération d’urgence/secondaire. Vous devez alors vous occuper de la réplication d’une partie des données, dont Azure se serait chargé. Vous trouverez un exemple de réplication d’une instance Active Directory et d’un nom DNS dans la région de récupération d’urgence dans l’article [Configuration de la récupération d’urgence pour Active Directory et le nom DNS](../../../site-recovery/site-recovery-active-directory.md).


## <a name="azure-virtual-machine-services"></a>Services de machines virtuelles Azure
Azure met à votre disposition une grande variété de machines virtuelles pour vos déploiements. Cela vous permet de réduire vos achats initiaux d'infrastructure et de technologie. L'offre de services de machines virtuelles Azure simplifie la gestion et l'exploitation des applications en fournissant des capacités de calcul et de stockage à la demande pour héberger, mettre à l'échelle et gérer les applications web et les applications connectées. La gestion de l'infrastructure est automatisée grâce à une plateforme conçue pour offrir un haut niveau de disponibilité et une mise à l'échelle dynamique, afin de répondre aux besoins d'utilisation en ayant la possibilité d'utiliser plusieurs modèles de tarification différents.

![Positionnement des services Microsoft Azure Virtual Machine][planning-guide-figure-400]

Grâce aux services de machines virtuelles Azure, Microsoft vous permet de déployer des images serveur personnalisées sur Azure en tant qu'instances IaaS. Différentes images de système d'exploitation sont également à votre disposition dans la bibliothèque d'images Azure.

D’un point de vue opérationnel, Azure Virtual Machine Services offre des expériences similaires à celles des machines virtuelles déployées en local. Vous êtes responsable de l'administration, de l'exploitation et des mises à jour correctives du système d'exploitation exécuté sur une machine virtuelle Azure et des applications de cette machine virtuelle. Microsoft ne fournit aucun service en dehors de l'hébergement de cette machine virtuelle sur son infrastructure Azure (Infrastructure as a Service - IaaS). Pour la charge de travail SAP que vous déployez en tant que client, Microsoft ne propose aucune offre en dehors des offres IaaS.

La plateforme Microsoft Azure est une plateforme multilocataire. En conséquence, les ressources de stockage, de réseau et de calcul qui hébergent les machines virtuelles Azure sont, à quelques exceptions près, partagées entre les locataires. La logique de quota et de limitation intelligente est utilisée pour empêcher un locataire d’affecter les performances d’un autre locataire (voisin bruyant) de manière radicale. En particulier dans le cadre de la certification de la plateforme Azure pour SAP HANA, Microsoft doit prouver l’isolation des ressources dans les cas où plusieurs machines virtuelles peuvent s’exécuter régulièrement sur le même hôte sur SAP. Bien que la logique dans Azure tente de réduire les écarts de bande passante, les plateformes hautement partagées ont tendance à générer des écarts plus importants en termes de disponibilité des ressources/de bande passante que ceux auxquels les clients peuvent être confrontés dans leurs déploiements locaux. La probabilité selon laquelle un système SAP sur Azure puisse connaître des écarts plus importants que ceux d’un système local doit être prise en compte.

### <a name="azure-virtual-machines-for-sap-workload"></a>Machines virtuelles Azure pour la charge de travail SAP

Pour la charge de travail SAP, nous avons restreint la sélection à différentes familles de machines virtuelles adaptées à la charge de travail SAP, et plus particulièrement à la charge de travail SAP HANA. Pour trouver le bon type de machine virtuelle et en savoir plus sur sa capacité à utiliser la charge de travail SAP, consultez le document [Logiciels SAP pris en charge pour les déploiements Azure](./sap-supported-product-on-azure.md).

> [!NOTE]
> Aucun surapprovisionnement en ressources UC et mémoire ne s'applique aux types de machines virtuelles certifiés pour la charge de travail SAP.

Outre la sélection de types de machines virtuelles strictement pris en charge, vous devez également vérifier si ceux-ci sont disponibles dans une région donnée en consultant le site [Produits disponibles par région](https://azure.microsoft.com/global-infrastructure/services/). Mais plus important encore, vous devez déterminer si :

- Les ressources UC et mémoire des différents types de machines virtuelles
- La bande passante IOPS des différents types de machines virtuelles
- Les fonctionnalités réseau des différents types de machines virtuelles
- Le nombre de disques pouvant être connectés
- La capacité à utiliser certains types de stockage Azure

répondent à vos besoins. La plupart de ces données sont accessibles [ici (Linux)][virtual-machines-sizes-linux] et [ici (Windows)][virtual-machines-sizes-windows] pour un type de machine virtuelle particulier.

Concernant le modèle de tarification, vous disposez de différentes options :

- Paiement à l’utilisation
- Réservée(s) pendant un an
- Réservée(s) pendant trois ans
- Tarification Spot

La tarification de chacune des offres, avec différentes offres de services en fonction du système d'exploitation et de la région, est disponible sur le site [Tarification des machines virtuelles Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) et [Tarification des machines virtuelles Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). Pour plus d'informations et pour en savoir plus sur la flexibilité des instances réservées d'un et trois ans, consultez les articles suivants :

- [Qu’est-ce qu’une réservation Azure ?](../../../cost-management-billing/reservations/save-compute-costs-reservations.md)
- [Flexibilité de la taille de machine virtuelle avec des instances de machines virtuelles réservées](../../reserved-vm-instance-size-flexibility.md)
- [Comment la remise de réservation Azure est-elle appliquée aux machines virtuelles ?](../../../cost-management-billing/manage/understand-vm-reservation-charges.md)

Pour plus d'informations sur la tarification Spot, consultez l'article [Machines virtuelles Spot Azure](https://azure.microsoft.com/pricing/spot/). Pour un même type de machine virtuelle, la tarification peut également varier d'une région Azure à une autre. Pour certains clients, il est plus intéressant de procéder à un déploiement dans une région Azure moins chère.

En outre, Azure offre les concepts d'un hôte dédié. Le concept d'hôte dédié vous confère davantage de contrôle sur les cycles de mise à jour corrective d'Azure. Vous pouvez programmer les mises à jour correctives selon vos propres horaires. Cette offre s'adresse spécifiquement aux clients dont la charge de travail peut de ne pas suivre le cycle normal. Pour en savoir plus sur les concepts des offres d'hôte dédié Azure, consultez l'article [Hôte dédié Azure](../../dedicated-hosts.md). Cette offre est prise en charge pour la charge de travail SAP et est utilisée par différents clients SAP qui souhaitent bénéficier d'un contrôle accru sur les mises à jour correctives de l'infrastructure et les éventuels plans de maintenance de Microsoft. Pour plus d'informations sur la façon dont Microsoft assure la maintenance et les mises à jour correctives de l'infrastructure Azure qui héberge les machines virtuelles, consultez l'article [Maintenance des machines virtuelles dans Azure](../../maintenance-and-updates.md).

#### <a name="generation-1-and-generation-2-virtual-machines"></a>Machines virtuelles de générations 1 et 2
L'hyperviseur de Microsoft peut gérer deux générations distinctes de machines virtuelles. Ces formats sont appelés **Génération 1** et **Génération 2**. Le format **Génération 2** a été introduit en 2012 avec l'hyperviseur Windows Server 2012. Azure a commencé par utiliser des machines virtuelles de génération 1. Lorsque vous déployez des machines virtuelles Azure, le format par défaut est toujours Génération 1. Mais vous pouvez également déployer le format de machine virtuelle de génération 2. L'article [Prise en charge des machines virtuelles de génération 2 dans Azure](../../generation-2.md) répertorie les familles de machines virtuelles Azure qui peuvent être déployées en tant que machines virtuelles de génération 2. Cet article présente également les importantes différences fonctionnelles des machines virtuelles de génération 2, qui peuvent s’exécuter sur le cloud privé Hyper-V et Azure. Plus important encore, cet article répertorie les différences fonctionnelles entre les machines virtuelles de génération 1 et les machines virtuelles de génération 2, telles que celles qui sont exécutées dans Azure.

> [!NOTE]
> Il existe des différences fonctionnelles entre les machines virtuelles de génération 1 et de génération 2 exécutées dans Azure. Lisez l'article [Prise en charge des machines virtuelles de génération 2 dans Azure](../../generation-2.md) pour accéder à la liste de ces différences.

Il est impossible de transférer une machine virtuelle existante d'une génération à l'autre. Pour changer de génération, vous devez déployer une nouvelle machine virtuelle de la génération souhaitée et y réinstaller les logiciels que vous exécutez. Cette modification n’affecte que l’image VHD de base de la machine virtuelle et n’a aucun impact sur les disques de données ni les partages NFS et SMB initialement attachés et attribués (par exemple, sur une machine virtuelle de génération 1).

> [!NOTE]
> Le déploiement de machines virtuelles de la famille de machines virtuelles Mv1 en tant que machines virtuelles de 2e génération est possible depuis le début du mois de mai 2020. Avec cela, il est possible d’avoir un upsizing et un downsizing apparent entre les machines virtuelles de la famille Mv1 et MV2.


### <a name="storage-microsoft-azure-storage-and-data-disks"></a><a name="a72afa26-4bf4-4a25-8cf7-855d6032157f"></a>Stockage : Stockage Microsoft Azure et disques de données
Les Microsoft Azure Virtual Machines utilisent différents types de stockage. Pendant l’implémentation SAP sur Azure Virtual Machine Services, il est important de comprendre les différences entre ces deux principaux types de stockage :

* stockage volatil non persistant ;
* stockage persistant.

Les machines virtuelles Azure proposent des disques non persistants suite au déploiement d’une machine virtuelle. Dans le cas du redémarrage d’une machine virtuelle, tout le contenu de ces lecteurs est effacé. Par conséquent, les fichiers de données et les fichiers journaux/de restauration ne doivent en aucun cas se trouver sur ces lecteurs. Il peut exister des exceptions pour certaines des bases de données, où ces lecteurs non persistants peuvent être appropriés pour les espaces de stockage tempdb et temp. Toutefois, évitez d’utiliser ces lecteurs pour les machines virtuelles de série A dans la mesure où ces lecteurs non persistants sont limités en débit avec cette famille de machines virtuelles. Pour plus d’informations, consultez l’article [Understanding the temporary drive on Windows Azure Virtual Machines](/archive/blogs/mast/understanding-the-temporary-drive-on-windows-azure-virtual-machines) (Comprendre le lecteur temporaire sur les machines virtuelles Microsoft Azure)

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Dans une machine virtuelle Azure, le lecteur D:\ est un lecteur non persistant soutenu par des disques locaux présents sur le nœud de calcul Azure. Comme il est non persistant, cela signifie que toutes les modifications apportées au contenu sur le lecteur D:\ sont perdues lors du redémarrage de la machine virtuelle. Par « modifications », nous entendons les fichiers enregistrés, les répertoires créés, les applications installées, etc.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Les machines virtuelles Azure Linux montent automatiquement un lecteur à l’emplacement /mnt/resource. Il s’agit d’un lecteur non persistant soutenu par des disques locaux présents sur le nœud de calcul Azure. Comme il est non persistant, toutes les modifications apportées au contenu de l’emplacement /mnt/resource seront perdues si vous redémarrez la machine virtuelle. Par « modifications », nous entendons les fichiers enregistrés, les répertoires créés, les applications installées, etc.
>
>

#### <a name="azure-storage-accounts"></a>Comptes de stockage Azure

Durant le déploiement de services ou de machines virtuelles dans Azure, le déploiement des VHD et des images de machine virtuelle est organisé dans des unités appelées comptes de stockage Azure. Les [comptes de stockage Azure](../../../storage/common/storage-account-overview.md) présentent des limitations en matière d’IOPS, de débit ou de tailles qu’ils peuvent contenir. Ces limitations sont documentées dans les articles suivants :

- [Objectifs d’extensibilité pour les comptes de stockage standard](../../../storage/common/scalability-targets-standard-account.md)
- [Objectifs de scalabilité pour les comptes de stockage d’objets blob de pages Premium](../../../storage/blobs/scalability-targets-premium-page-blobs.md)

Elles jouaient auparavant un rôle important dans la planification d’un déploiement SAP dans Azure. Il vous appartenait de gérer le nombre de disques persistants dans un compte de stockage. Vous deviez gérer les comptes de stockage et, éventuellement, créer des comptes de stockage pour obtenir des disques plus persistants.

Ces dernières années, l’introduction des [Disques managés Azure](../../managed-disks-overview.md) vous a libéré de ces tâches. Pour les déploiements SAP, il est recommandé d’utiliser des disques managés Azure au lieu de gérer vous-même les comptes de stockage Azure. Les disques managés Azure distribuent les disques sur différents comptes de stockage de façon à ne pas dépasser les limites de chacun des comptes.

Il existe, au sein d’un compte de stockage, un type de concept de dossier appelé « conteneur », qui peut être utilisé pour regrouper certains disques.

Dans Azure, un nom de disque/VHD suit la convention d’affectation des noms suivante, qui implique de donner au VHD un nom unique dans Azure :

`http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>`

La chaîne ci-dessus doit identifier de manière unique le disque/VHD stocké sur le Stockage Azure.


#### <a name="azure-persisted-storage-types"></a>Types de stockage persistants Azure
Azure offre une grande variété d’options de stockage persistantes qui peuvent être utilisées pour la charge de travail SAP et des composants précis de la pile SAP. Pour plus d’informations, consultez le document [Stockage Azure pour les charges de travail SAP](./planning-guide-storage.md).


### <a name="microsoft-azure-networking"></a><a name="61678387-8868-435d-9f8c-450b2424f5bd"></a>Mise en réseau Microsoft Azure

Microsoft Azure fournit une infrastructure réseau qui permet le mappage de tous les scénarios que nous souhaitons réaliser avec le logiciel SAP. Ses fonctionnalités sont les suivantes :

* accès depuis l’extérieur, directement aux machines virtuelles via Windows Terminal Services ou ssh/VNC ;
* accès aux services et ports spécifiques utilisés par les applications dans les machines virtuelles ;
* communication interne et résolution de noms entre un groupe de machines virtuelles déployées en tant que machines virtuelles Azure ;
* connectivité intersite entre un réseau local du client et le réseau Azure
* connectivité entre le centre de données ou la région Azure entre les sites Azure.

Des informations supplémentaires sont disponibles ici : <https://azure.microsoft.com/documentation/services/virtual-network/>

Il existe de nombreuses façons de configurer la résolution de noms et IP dans Azure. Il existe également un service DNS Azure, qui peut être utilisé au lieu de configurer votre propre serveur DNS. Pour plus d’informations, consultez [cet article][virtual-networks-manage-dns-in-vnet] et [cette page](https://azure.microsoft.com/services/dns/).

Pour les scénarios intersites ou hybrides, nous partons du principe que l’annuaire AD/OpenLDAP et le DNS local ont été étendus via une connexion VPN ou privée à Azure. Pour certains scénarios documentés ici, il peut être nécessaire de disposer d’un réplica d’annuaire AD/OpenLDAP installé dans Azure.

La mise en réseau et la résolution de noms étant des aspects essentiels du déploiement de base de données pour un système SAP, ce concept est décrit plus en détail dans le [Guide de déploiement de SGBD][dbms-guide].

##### <a name="azure-virtual-networks"></a>Réseaux virtuels Azure

En créant un réseau virtuel Azure, vous pouvez définir la plage d’adresses des adresses IP privées allouées par la fonctionnalité DHCP Azure. Dans les scénarios intersites, la plage d’adresses IP définie est toujours allouée via la fonctionnalité DHCP par Azure. Toutefois, la résolution de noms de domaine est effectuée en local (en supposant que les machines virtuelles font partie d’un domaine local) et peut donc résoudre les adresses au-delà des différents services cloud Azure.

Chaque machine virtuelle dans Azure doit être connectée à un réseau virtuel.

Pour plus d’informations, consultez [cet article][resource-groups-networking] et [cette page](https://azure.microsoft.com/documentation/services/virtual-network/).


> [!NOTE]
> Par défaut, une fois qu’une machine virtuelle est déployée, vous ne pouvez pas modifier la configuration du réseau virtuel. Les paramètres TCP/IP doivent être conservés sur le serveur DHCP d’Azure. Le comportement par défaut est l’attribution d’adresse IP dynamique.
>
>

L’adresse MAC de la carte de réseau virtuel peut changer (par exemple, après un redimensionnement) et l’OS invité Linux ou Windows sélectionne la nouvelle carte réseau et utilise automatiquement DHCP pour assigner les adresses DNS et IP dans ce cas.

##### <a name="static-ip-assignment"></a>Attribution d’adresse IP statique
Il est possible d’attribuer des adresses IP fixes ou réservées aux machines virtuelles d’un réseau virtuel Azure. Exécuter les machines virtuelles dans un réseau virtuel Azure offre l’opportunité exceptionnelle d’exploiter cette fonctionnalité si cela est nécessaire ou obligatoire pour certains scénarios. L’attribution d’adresse IP reste valide tout au long de l’existence de la machine virtuelle, que celle-ci soit en fonctionnement ou à l’arrêt. Par conséquent, vous devez prendre en compte le nombre total de machines virtuelles (en cours d’exécution et arrêtées) quand vous définissez la plage d’adresses IP du réseau virtuel. L’adresse IP reste attribuée jusqu’à ce que la machine virtuelle et son interface réseau soient supprimées ou jusqu’à ce que l’attribution de l’adresse IP soit à nouveau supprimée. Pour plus d’informations, consultez [cet article][virtual-networks-static-private-ip-arm-pportal].

> [!NOTE]
> Vous devez attribuer des adresses IP statiques aux cartes réseau virtuelles individuelles à l’aide des outils Azure. Vous ne devez pas attribuer d’adresses IP statiques au sein du système d’exploitation invité à une carte réseau virtuelle. Certains services Azure, comme le service de Sauvegarde Azure, s’appuient sur le fait que la carte réseau virtuelle principale est définie sur la DHCP et non sur des adresses IP statiques. Consultez également le document [Dépannage de la sauvegarde de machine virtuelle Azure](../../../backup/backup-azure-vms-troubleshoot.md#networking).
>
>

##### <a name="multiple-nics-per-vm"></a>Plusieurs cartes d’interface réseau par machine virtuelle

Vous pouvez définir plusieurs cartes d’interface de réseau virtuel pour une machine virtuelle Azure. Grâce à plusieurs cartes d’interface de réseau virtuel, vous pouvez commencer à configurer la séparation du trafic réseau où, par exemple, le trafic client est acheminé via une carte d’interface de réseau virtuel et le trafic backend est acheminé via une deuxième carte d’interface de réseau virtuel. En fonction du type de machine virtuelle, il existe différentes limitations quant au nombre de cartes d’interface de réseau virtuel qu’une machine virtuelle peut se voir attribuer. Vous trouverez des informations précises et des renseignements sur les fonctions et les restrictions dans ces articles :

* [Créer une machine virtuelle Windows avec plusieurs cartes d’interface réseau (NIC)][virtual-networks-multiple-nics-windows]
* [Créer une machine virtuelle Linux avec plusieurs cartes réseau][virtual-networks-multiple-nics-linux]
* [Déployer des machines virtuelles avec plusieurs cartes réseau à l’aide d’un modèle][virtual-network-deploy-multinic-arm-template]
* [Déployer des machines virtuelles avec plusieurs cartes réseau à l’aide de PowerShell][virtual-network-deploy-multinic-arm-ps]
* [Déployer des machines virtuelles avec plusieurs cartes réseau à l'aide de l'interface de ligne de commande Azure][virtual-network-deploy-multinic-arm-cli]

#### <a name="site-to-site-connectivity"></a>Connectivité de site à site

La connectivité intersite consiste à lier, via une connexion VPN transparente et permanente, les machines virtuelles Azure et les sites locaux. Cela devrait devenir le modèle de déploiement SAP le plus courant dans Azure. L’hypothèse est que les procédures et les processus opérationnels avec des instances SAP dans Azure doivent fonctionner en toute transparence. Cela signifie que vous devez être en mesure d’imprimer en dehors de ces systèmes, mais également d’utiliser le système de gestion de transport SAP (TMS) pour transporter les changements d’un système de développement dans Azure à un système de test, qui déployé en local. Pour plus d’informations sur la connectivité intersite, consultez [cet article][vpn-gateway-create-site-to-site-rm-powershell].

##### <a name="vpn-tunnel-device"></a>Tunnel VPN

Pour créer une connexion de site à site (centre de données local vers centre de données Azure), vous devez obtenir et configurer un appareil VPN ou utiliser le service Routage et accès distant (RRAS) qui a été introduit en tant que composant logiciel avec Windows Server 2012.

* [Créer un réseau virtuel avec une connexion VPN site à site à l’aide de PowerShell][vpn-gateway-create-site-to-site-rm-powershell]
* [À propos des périphériques VPN pour les connexions de la passerelle VPN de site à site][vpn-gateway-about-vpn-devices]
* [FAQ sur la passerelle VPN][vpn-gateway-vpn-faq]

![Connexion de site à site entre des systèmes locaux et dans Azure][planning-guide-figure-600]

La figure ci-dessus présente deux abonnements Azure qui ont des sous-plages d’adresses IP réservées pour une utilisation dans des réseaux virtuels Azure. La connectivité du réseau local vers Azure est établie via un VPN.

#### <a name="point-to-site-vpn"></a>VPN de point à site

Le VPN de point à site requiert que chaque machine du client se connecte à Azure avec son propre VPN. Pour les scénarios SAP que nous étudions, la connectivité point à site n’est pas pratique. Ainsi, nous n’en parlons pas davantage.

Des informations supplémentaires sont disponibles dans les documents suivants :
* [Configurer une connexion site à site vers un réseau virtuel à l’aide du portail Azure](../../../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md)
* [Configurer une connexion site à site vers un réseau virtuel à l’aide de PowerShell](../../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

#### <a name="multi-site-vpn"></a>VPN multisite

Actuellement, Azure propose aussi de créer une connexion VPN multisite pour un abonnement Azure. Précédemment, un abonnement était limité à une seule connexion VPN de intersite. Cette limitation a disparu et vous pouvez désormais bénéficier de connexions VPN multisites par abonnement. Cela permet d’exploiter plusieurs régions Azure pour un abonnement spécifique par le biais de configurations intersites.

Pour plus d’informations, consultez [cet article][vpn-gateway-create-site-to-site-rm-powershell].

#### <a name="vnet-to-vnet-connection"></a>Connexion de réseau virtuel à réseau virtuel

À l’aide d’un VPN multisite, vous devez configurer un réseau virtuel Azure distinct dans chacune des régions. Toutefois, vous avez souvent besoin que les composants logiciels des différentes régions puissent communiquer entre eux. Dans l’idéal, cette communication ne doit pas être acheminée depuis une région Azure vers un site local, ni depuis ce site local vers l’autre région Azure. En bref, Azure vous offre la possibilité de configurer une connexion depuis un réseau virtuel Azure dans une région vers un autre réseau virtuel Azure hébergé dans l’autre région. Cette fonctionnalité est appelée connexion de réseau virtuel à réseau virtuel. Vous trouverez plus de détails sur cette fonctionnalité ici : <https://azure.microsoft.com/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/>.

#### <a name="private-connection-to-azure-expressroute"></a>Connexion privée à Azure - ExpressRoute

Microsoft Azure ExpressRoute permet de créer des connexions privées entre des centres de données Azure et l’infrastructure locale du client ou un environnement de colocalisation. ExpressRoute est proposé par divers fournisseurs VPN (commutation de paquets) MPLS ou d’autres fournisseurs de services réseau. Les connexions ExpressRoute ne sont pas établies par le biais de l'Internet public. Les connexions ExpressRoute offrent une sécurité accrue, une plus grande fiabilité via plusieurs circuits parallèles, des vitesses plus rapides et des latences moindres par rapport aux connexions classiques sur Internet.

Vous trouverez plus d’informations sur Azure ExpressRoute et les offres ici :

* <https://azure.microsoft.com/documentation/services/expressroute/>
* <https://azure.microsoft.com/pricing/details/expressroute/>
* <https://azure.microsoft.com/documentation/articles/expressroute-faqs/>

ExpressRoute permet d’utiliser plusieurs abonnements Azure via un seul circuit ExpressRoute, comme décrit ici

* <https://azure.microsoft.com/documentation/articles/expressroute-howto-linkvnet-arm/>
* <https://azure.microsoft.com/documentation/articles/expressroute-howto-circuit-arm/>

#### <a name="forced-tunneling-in-case-of-cross-premises"></a>Tunneling forcé en cas de scénario intersite
Pour les machines virtuelles rejoignant les domaines locaux via une connexion site à site, point à site ou ExpressRoute, vous devez vous assurer que les paramètres du proxy Internet sont également déployés pour tous les utilisateurs dans ces machines virtuelles. Par défaut, le logiciel s’exécutant dans ces machines virtuelles ou les utilisateurs utilisant un navigateur pour accéder à Internet ne passeront pas par le proxy d’entreprise, mais se connecteront directement via Azure à Internet. Mais, même le paramétrage du proxy ne constitue pas une solution totalement efficace pour diriger le trafic vers le proxy de l’entreprise, puisque ce sont les services et le logiciel qui sont responsables de vérifier le proxy. Si le logiciel exécuté dans la machine virtuelle n’effectue pas cette vérification ou si un administrateur manipule les paramètres, le trafic vers Internet peut à nouveau être détourné directement vers Internet par le biais d’Azure.

Pour éviter cette connectivité Internet directe, vous pouvez configurer le tunneling forcé avec une connectivité site à site entre le site local et Azure. La description détaillée de la fonctionnalité de tunneling forcé est disponible ici : <https://azure.microsoft.com/documentation/articles/vpn-gateway-forced-tunneling-rm/>

Le tunneling forcé avec ExpressRoute est activé par les clients publiant un itinéraire par défaut via les sessions de peering BGP ExpressRoute.

#### <a name="summary-of-azure-networking"></a>Résumé de la mise en réseau Azure

Ce chapitre abordait plusieurs points importants sur la mise en réseau Azure. Voici un résumé des points principaux :

* Les réseaux virtuels Azure permettent d’ajouter une structure de réseau dans votre déploiement Azure. Les réseaux virtuels peuvent être isolés les uns des autres ou vous pouvez contrôler le trafic entre réseaux virtuels à l’aide de groupes de sécurité réseau.
* Les réseaux virtuels Azure peuvent être exploités pour attribuer des plages d’adresses IP à des machines virtuelles ou pour attribuer des adresses IP fixes à des machines virtuelles.
* Pour configurer une connexion de site à site ou de point à site, vous devez d’abord créer un réseau virtuel Azure.
* Après le déploiement d’une machine virtuelle, il n’est plus possible de modifier le réseau virtuel attribué à la machine virtuelle.

### <a name="quotas-in-azure-virtual-machine-services"></a>Quotas des services de machines virtuelles Azure
Nous devons être clairs sur le fait que l’infrastructure réseau et de stockage est partagée entre les machines virtuelles exécutant une variété de services dans l’infrastructure Azure. Comme dans les propres centres de données du client, il se produit dans une certaine mesure un provisionnement excessif de certaines ressources d’infrastructure. La Plateforme Microsoft Azure utilise le disque, le processeur, le réseau et d’autres quotas pour limiter la consommation de ressources et conserver des niveaux de performance cohérents et déterministes.  Les différents types de machines virtuelles (A5, A6, etc.) possèdent des quotas distincts pour le nombre de disques, le processeur, la RAM et le réseau.

> [!NOTE]
> Les ressources processeur et mémoire des types de machines virtuelles pris en charge par SAP sont pré-allouées sur les nœuds hôtes. Cela signifie qu’une fois la machine virtuelle déployée, les ressources sur l’hôte sont disponibles comme défini par le type de machine virtuelle.
>
>

Quand vous planifiez et redimensionnez des solutions SAP sur Azure, vous devez prendre en compte les quotas de chaque taille de machine virtuelle. Les quotas de machines virtuelles sont décrits [ici (Linux)][virtual-machines-sizes-linux] et [ici (Windows)][virtual-machines-sizes-windows].

Les quotas décrits représentent les valeurs maximum théoriques.  La limite d’IOPS par disque peut être atteinte avec un faible volume d’IOPS (8 Ko), et pas nécessairement avec un volume important d’IOPS (1 Mo).  La limite d’E/S par seconde est appliquée sur la granularité d’un seul disque.

L’arbre de décision approximatif ci-dessous peut servir à déterminer si un système SAP s’adapte à Azure Virtual Machine Services et à ses fonctionnalités ou si un système existant doit être configuré différemment pour déployer le système sur Azure :

![Arbre de décision pour décider de la capacité à déployer SAP sur Azure][planning-guide-figure-700]

1. Les informations les plus importantes à utiliser en premier sont les exigences SAP pour un système SAP donné. La configuration SAP doit être séparée dans la partie SGBD (système de gestion de base de données) et dans les applications SAP, même si le système SAP est déjà déployé localement dans une configuration de niveau 2. Pour les systèmes existants, les SAP liés au matériel souvent utilisés peuvent être déterminés ou estimés en fonction de points de référence SAP existants. Les résultats se trouvent [ici](https://sap.com/about/benchmark.html). Pour les systèmes SAP nouvellement déployés, vous devez avoir effectué un exercice de dimensionnement pour déterminer la configuration SAP du système.
1. Pour les systèmes existants, le volume d’E/S et les opérations d’E/S par seconde sur le serveur SGBD doivent être mesurés. Pour les systèmes récemment planifiés, l’exercice de dimensionnement pour le nouveau système doit également donner une idée approximative de la configuration des E/S côté SGBD. En cas de doute, vous devrez réaliser une preuve de concept.
1. Comparer les exigences SAP pour le serveur SGBD avec les SAP que peuvent fournir les différents types de machine virtuelle Azure. Les informations sur les SAP des différents types de machines virtuelles Azure se trouvent dans la Note de SAP [1928533]. La machine virtuelle du SGBD doit constituer une priorité, car la couche de base de données correspond à la couche d’un système SAP NetWeaver qui n’effectue pas de scale-out dans la plupart des déploiements. En revanche, la taille des instances pour la couche d’application SAP peut être augmentée. Si aucune machine virtuelle Azure prenant en charge SAP ne peut fournir les SAP requis, la charge de travail du système SAP planifié ne peut alors être exécutée dans Azure. Vous devez déployer le système localement ou modifier le volume de charge de travail pour le système.
1. Comme indiqué [ici (Linux)][virtual-machines-sizes-linux] et [ici (Windows)][virtual-machines-sizes-windows], Azure applique un quota d’IOPS par disque, que vous utilisiez un stockage Standard ou Premium. Le nombre de disques de données pouvant être montés varie en fonction du type de machine virtuelle. Par conséquent, vous pouvez calculer le nombre maximal d’E/S par seconde pouvant être obtenues avec chaque type de machine virtuelle. En fonction de la disposition du fichier de base de données, vous pouvez entrelacer les disques pour que ceux-ci ne constituent qu’un seul volume dans le système d’exploitation invité. Toutefois, si le volume d’E/S par seconde actuel d’un système SAP déployé dépasse les limites calculées pour le type de machine virtuelle Azure le plus volumineux, et s’il est impossible de le compenser avec l’ajout de mémoire supplémentaire, la charge de travail du système SAP peut s’en trouver gravement affectée. Dans ce cas, il serait plus judicieux de ne pas déployer le système dans Azure.
1. En particulier dans les systèmes SAP, qui sont déployés localement dans les configurations à deux niveaux, il y a de grandes chances que le système doive être configuré sur trois niveaux dans Azure. Dans cette étape, vous devez vérifier s’il existe un composant dans la couche d’application SAP, dont la taille des instances ne peut être augmentée et qui n’est pas compatible avec les ressources processeur et mémoire fournies par les différents types de machine virtuelle Azure. Si c’est le cas, le système SAP et sa charge de travail ne peuvent pas être déployés vers Azure. Cependant, si vous pouvez effectuer un scale-out des composants d’application SAP dans plusieurs machines virtuelles Azure, le système peut être déployé vers Azure.

Si les composants de la couche Application SAP et SGBD peuvent être exécutés dans des machines virtuelles Azure, la configuration doit être définie en prenant en considération ce qui suit :

* Nombre de machines virtuelles Azure
* Types de machine virtuelle pour les composants individuels
* Nombre de disques durs virtuels dans la machine virtuelle du SGBD (afin de fournir suffisamment d’E/S par seconde).

## <a name="managing-azure-assets"></a>Gestion des ressources Azure

### <a name="azure-portal"></a>Portail Azure

Le portail Azure constitue l’une des trois interfaces destinées à la gestion des déploiements des machines virtuelles Azure. Les tâches de gestion de base, telles que le déploiement des machines virtuelles à partir d’images, peuvent être effectuées via le portail Azure. Par ailleurs, vous pouvez aussi facilement gérer dans le portail Azure des tâches comme la création de comptes de stockage, de réseaux virtuels et d’autres composants Azure. Toutefois, des fonctionnalités telles que le chargement de VHD locaux sur Azure ou la copie d’un VHD au sein d’Azure sont des tâches qui nécessitent des outils tiers ou une administration via PowerShell ou l’interface de lignes de commandes.

![Portail Microsoft Azure - vue d’ensemble de la machine virtuelle][planning-guide-figure-800]


Les tâches d’administration et de configuration de l’instance de la machine virtuelle sont possibles au sein du portail Azure.

Outre redémarrer et arrêter une machine virtuelle, vous pouvez aussi attacher, détacher et créer des disques de données pour l’instance de machine virtuelle, afin de capturer l’instance pour la préparation de l’image, et configurer la taille de l’instance de machine virtuelle.

Le portail Azure fournit des fonctionnalités de base pour déployer et configurer des machines virtuelles et de nombreux autres services Azure. Cependant, toutes les fonctionnalités disponibles ne sont pas couvertes par le portail Azure. Dans le portail Azure, il est impossible d’effectuer des tâches telles que :

* le téléchargement de disques durs virtuels vers Azure ;
* la copie de machines virtuelles.


### <a name="management-via-microsoft-azure-powershell-cmdlets"></a>Gestion par le biais des applets de commande Microsoft Azure PowerShell

Windows PowerShell est une infrastructure puissante et extensible, largement adoptée par les clients qui déploient un nombre plus important de systèmes dans Azure. Après avoir installé des applets de commande PowerShell sur un ordinateur de bureau, un ordinateur portable ou un poste de gestion dédié, les applets de commande PowerShell peuvent être exécutées à distance.

Le processus permettant à un ordinateur local (de bureau ou portable) d’utiliser des cmdlets Azure PowerShell, et la manière de configurer ces dernières pour une utilisation avec des abonnements Azure sont décrits dans [cet article][powershell-install-configure].

Des étapes plus détaillées sur l’installation, la mise à jour et la configuration des cmdlets Azure PowerShell sont également décrites dans ce [chapitre du Guide de déploiement][deployment-guide-4.1].

Le retour de nos clients a permis de déterminer que PowerShell (PS) constitue l’outil le plus puissant pour déployer des machines virtuelles et créer des étapes personnalisées lors de leur déploiement. Tous les clients exécutant des instances SAP dans Azure utilisent des applets de commande PS pour compléter des tâches de gestion qu’ils effectuent dans le portail Azure. Certains utilisent même exclusivement des applets de commande PS pour gérer leurs déploiements dans Azure. Étant donné que les applets de commande dédiées à Azure partagent la même convention d’affectation de noms que les applets de commande Windows (dont le nombre dépasse 2000), les administrateurs Windows peuvent facilement tirer parti de ces applets de commande.

Consultez l’exemple ici : <https://blogs.technet.com/b/keithmayer/archive/2015/07/07/18-steps-for-end-to-end-iaas-provisioning-in-the-cloud-with-azure-resource-manager-arm-powershell-and-desired-state-configuration-dsc.aspx>


Le déploiement de l’extension Azure pour SAP (voir le chapitre [Extension Azure pour SAP][planning-guide-9.1] dans ce document) n’est possible que par le biais de PowerShell ou de l’interface de ligne de commande. Ainsi, il est obligatoire d’installer et de configurer PowerShell ou l’interface de ligne de commandes durant le déploiement ou l’administration d’un système SAP NetWeaver dans Azure.

Étant donné qu’Azure propose davantage de fonctionnalités, de nouvelles applets de commande PS vont être ajoutées ; celles-ci nécessiteront une mise à jour des applets de commande. Il est donc recommandé de consulter le site de téléchargement Azure <https://azure.microsoft.com/downloads/> au moins une fois par mois, pour vérifier si une nouvelle version des applets de commande est disponible. La nouvelle version est installée par-dessus l’ancienne version.

Pour une liste générale des commandes Azure PowerShell, consultez : <https://docs.microsoft.com/powershell/azure/>.

### <a name="management-via-microsoft-azure-cli-commands"></a>Gestion à l’aide des commandes CLI Microsoft Azure

Pour les clients qui utilisent Linux et souhaitent gérer les ressources Azure, PowerShell n'est pas forcément une option. Microsoft propose la CLI Microsoft Azure en guise d’alternative.
La CLI Microsoft Azure offre un ensemble de commandes multiplateforme open source pour l’utilisation de la plateforme Microsoft Azure. La CLI Microsoft Azure offre des fonctionnalités similaires à celles du portail Azure.

Pour plus d’informations sur l’installation, la configuration et l’utilisation des commandes CLI permettant de réaliser des tâches Azure, voir

* [Installer l’interface Azure Classic CLI][xplat-cli]
* [Déploiement et gestion de machines virtuelles à l’aide des modèles Azure Resource Manager et de l’interface de ligne de commande Azure][../../linux/create-ssh-secured-vm-from-template.md]
* [Utiliser Azure Classic CLI pour Mac, Linux et Windows avec Azure Resource Manager][xplat-cli-azure-resource-manager]

Pour savoir comment utiliser l’interface de ligne de commande Azure afin de déployer l’extension Azure pour SAP, lisez le chapitre [Interface de ligne de commande Azure pour machines virtuelles Linux][deployment-guide-4.5.2] du [guide de déploiement][planning-guide].


## <a name="first-steps-planning-a-deployment"></a>Premières étapes de planification d’un déploiement
La première étape de la planification du déploiement consiste à NE PAS rechercher les machines virtuelles disponibles pour l’exécution de SAP. La première étape peut prendre beaucoup de temps, mais plus important, elle consiste à collaborer avec les équipes de conformité et de sécurité de votre entreprise en ce qui concerne les conditions limites pour le déploiement du type de charge de travail SAP ou des processus métier dans le cloud public. Si votre entreprise a auparavant déployé d’autres logiciels dans Azure, le processus peut être facile. Si votre entreprise est au début de son parcours, des discussions plus importantes peuvent être nécessaires pour déterminer les conditions limites, les conditions de sécurité, qui autorisent l’hébergement de certaines données SAP et de certains processus métier SAP dans le cloud public.

Pour vous aider, vous pouvez consulter la liste des offres de conformité proposées par Microsoft en pointant sur [Offres de conformité Microsoft](/microsoft-365/compliance/offering-home).

D’autres aspects tels que le chiffrement des données pour les données au repos ou un autre chiffrement dans le service Azure sont décrits dans [Vue d’ensemble du chiffrement Azure](../../../security/fundamentals/encryption-overview.md).

Ne sous-estimez pas cette phase du projet dans votre planification. Vous ne devez passer à l’étape suivante, qui est la planification de l’architecture réseau que vous déployez dans Azure, que lorsque vous disposez d’un accord et de règles adaptés à cette rubrique.


## <a name="different-ways-to-deploy-vms-for-sap-in-azure"></a>Méthodes différentes pour le déploiement de machines virtuelles pour SAP dans Azure

Dans ce chapitre, vous découvrez les différentes manières de déployer une machine virtuelle dans Azure. Des procédures de préparation supplémentaires, ainsi que la gestion des disques durs virtuels et des machines virtuelles dans Azure, sont présentées dans ce chapitre.

### <a name="deployment-of-vms-for-sap"></a>Déploiement de machines virtuelles pour SAP

Microsoft Azure offre plusieurs modes de déploiement de machines virtuelles et des disques associés. Vous devez donc bien comprendre les différences, car la préparation des machines virtuelles peut différer en fonction du déploiement. Globalement, nous étudions les scénarios suivants :

#### <a name="moving-a-vm-from-on-premises-to-azure-with-a-non-generalized-disk"></a><a name="4d175f1b-7353-4137-9d2f-817683c26e53"></a>Déplacement d’une machine virtuelle locale vers Azure avec un disque non généralisé

Vous envisagez de déplacer un système SAP spécifique local vers Azure. Pour ce faire, vous pouvez charger le VHD, qui contient le système d’exploitation, les fichiers binaires SAP et les fichiers binaires SGBD, ainsi que les VHD contenant les données et les fichiers journaux du SGBD vers Azure. Contrairement au [scénario 2 ci-dessous][planning-guide-5.1.2], vous conservez le nom d’hôte, le SID SAP et les comptes d’utilisateur SAP dans la machine virtuelle Azure tels qu’ils ont été configurés dans l’environnement local. Par conséquent, il n’est pas nécessaire de généraliser l’image. Pour plus d’informations sur les étapes de préparation en local et le chargement des machines virtuelles et VHD non généralisés sur Azure, consultez le chapitre [Préparation du déplacement d’une machine virtuelle locale vers Azure avec un disque non généralisé][planning-guide-5.2.1] dans ce document. Lisez le chapitre [Scénario 3 : Déplacement d’une machine virtuelle locale à l’aide d’un VHD non généralisé Azure avec SAP][deployment-guide-3.4] dans le [Guide de déploiement][deployment-guide] pour obtenir la procédure détaillée du déploiement de ce type d’image dans Azure.

Une autre option que nous n’aborderons pas en détail dans ce guide consiste à utiliser Azure Site Recovery pour répliquer des serveurs d’applications SAP NetWeaver et des services centraux SAP NetWeaver sur Azure. Nous vous déconseillons d’utiliser Azure Site Recovery pour la couche de base de données et de privilégier plutôt des mécanismes de réplication spécifiques à la base de données, par exemple la réplication de système HANA. Pour plus d’informations, consultez le chapitre [Protéger SAP](../../../site-recovery/site-recovery-workload.md#protect-sap) du guide [À propos de la reprise d’activité pour les applications locales](../../../site-recovery/site-recovery-workload.md).

#### <a name="deploying-a-vm-with-a-customer-specific-image"></a><a name="e18f7839-c0e2-4385-b1e6-4538453a285c"></a>Déploiement d’une machine virtuelle avec une image spécifique du client

En raison des exigences spécifiques de correctif par rapport aux versions de votre système d’exploitation ou SGBD, les images fournies provenant de la Place de marché Azure peuvent ne pas répondre à vos besoins. Ainsi, vous devrez peut-être créer une machine virtuelle à l’aide de votre propre image « privée » de machine virtuelle du système d’exploitation/SGBD, qui peut être déployée plusieurs fois par la suite. Pour préparer une image « privée » de ce type à des fins de duplication, les éléments suivants doivent être pris en considération :

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Vous trouverez plus de détails ici : <https://docs.microsoft.com/azure/virtual-machines/windows/upload-generalized-managed> Les paramètres de Windows (comme le nom d’hôte et le SID Windows) doivent être extraits/généralisés sur la machine virtuelle locale via la commande sysprep.
>
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Pour préparer un VHD à un chargement sur Azure, suivez les étapes décrites dans les articles suivants pour [SUSE][virtual-machines-linux-create-upload-vhd-suse], [Red Hat][virtual-machines-linux-redhat-create-upload-vhd] ou [Oracle Linux][virtual-machines-linux-create-upload-vhd-oracle].
>
>

---
Si vous avez déjà installé du contenu SAP sur votre machine virtuelle locale (en particulier pour les systèmes à 2 niveaux), vous pouvez adapter les paramètres du système SAP après le déploiement de la machine virtuelle Azure à l’aide de la procédure de modification du nom d’instance prise en charge par le gestionnaire de déploiement de logiciels SAP (Note de SAP [1619720]). Pour en savoir plus sur les étapes de préparation en local et le chargement d’une machine virtuelle généralisée sur Azure, consultez les chapitres [Préparation du déploiement d’une machine virtuelle avec une image spécifique du client pour SAP][planning-guide-5.2.2] et [Chargement d’un VHD local vers Azure][planning-guide-5.3.2] dans ce document. Lisez le chapitre [Scénario 2 : Déploiement d’une machine virtuelle avec une image personnalisée pour SAP][deployment-guide-3.3] dans le [Guide de déploiement][deployment-guide] pour obtenir la procédure détaillée du déploiement de ce type d’image dans Azure.

#### <a name="deploying-a-vm-out-of-the-azure-marketplace"></a>Déploiement d’une machine virtuelle provenant de la Place de marché Azure

Vous voulez utiliser une image fournie par Microsoft ou par un tiers provenant de la Place de marché Azure pour déployer votre machine virtuelle. Une fois que vous avez déployé votre machine virtuelle dans Azure, vous utilisez les mêmes instructions et outils pour installer les logiciels SAP et/ou SGBD (système de base de données) au sein de votre machine virtuelle comme vous le feriez dans un environnement local. Pour une description plus détaillée du déploiement, consultez le chapitre [Scénario 1 : Déploiement d’une machine virtuelle de la Place de marché Azure pour SAP][deployment-guide-3.2] dans le [Guide de déploiement][deployment-guide].

### <a name="preparing-vms-with-sap-for-azure"></a><a name="6ffb9f41-a292-40bf-9e70-8204448559e7"></a>Préparation de machines virtuelles avec SAP pour Azure

Avant de charger des machines virtuelles dans Azure, vous devez vérifier que les machines virtuelles et les VHD répondent à certaines exigences. Il existe de légères différences selon la méthode de déploiement utilisée.

#### <a name="preparation-for-moving-a-vm-from-on-premises-to-azure-with-a-non-generalized-disk"></a><a name="1b287330-944b-495d-9ea7-94b83aff73ef"></a>Préparation du déplacement d’une machine virtuelle locale vers Azure avec un disque non généralisé

Une méthode de déploiement courante consiste à déplacer une machine virtuelle existante exécutant un système SAP du site local vers Azure. Cette machine virtuelle et le système SAP de la machine virtuelle doivent simplement s’exécuter dans Azure avec le même nom d’hôte et le même SID SAP. Dans ce cas, l’OS invité de la machine virtuelle ne doit pas être généralisé pour plusieurs déploiements. Si le réseau local est étendu à Azure, les comptes de domaine utilisés dans la machine virtuelle peuvent être les mêmes que ceux utilisés jusque-là dans l’environnement local.

Les exigences à respecter pour la préparation de votre propre disque de machine virtuelle Azure sont les suivantes :

* À l’origine, la taille maximale du VHD contenant le système d’exploitation était seulement de 127 Go. Cette limite a été repoussée en mars 2015. Désormais, la taille du VHD contenant le système d’exploitation peut aller jusqu’à 1 To, comme tous les autres VHD hébergés sur le Stockage Azure.
* Il doit être au format VHD fixe. Les disques durs virtuels ou les disques durs virtuels au format VHDx ne sont pas encore pris en charge sur Azure. Les disques durs virtuels dynamiques seront convertis en disques durs virtuels statiques lorsque vous chargez le disque dur virtuel avec les applets de commande PowerShell ou l’interface CLI
* Les VHD montés sur la machine virtuelle et qui doivent être remontés sur celle-ci dans Azure, doivent également être au format VHD fixe. Lisez [cet article](../../managed-disks-overview.md) pour connaître les limites de taille des disques de données. Les disques durs virtuels dynamiques seront convertis en disques durs virtuels statiques lorsque vous chargez le disque dur virtuel avec les applets de commande PowerShell ou l’interface CLI
* Ajoutez un autre compte local doté de privilèges d’administrateur, pouvant être utilisé par le support Microsoft ou être attribué comme contexte d’exécution aux services et applications, jusqu’à ce que la machine virtuelle soit déployée et que des utilisateurs plus appropriés puissent être utilisés.
* Ajoutez d’autres comptes locaux, car ceux-ci peuvent être nécessaires pour le scénario de déploiement spécifique.

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Dans ce scénario, aucune généralisation (sysprep) de la machine virtuelle n’est requise pour charger et déployer la machine virtuelle sur Azure.
> Vérifiez que le lecteur D:\ n’est pas utilisé.
> Définissez le montage automatique de disque pour les disques attachés comme décrit dans le chapitre [Paramétrage du montage automatique pour les disques attachés][planning-guide-5.5.3] dans ce document.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Dans ce scénario, aucune généralisation (waagent -deprovision) de la machine virtuelle n’est requise pour charger et déployer la machine virtuelle sur Azure.
> Assurez-vous que/mnt/resource n’est pas utilisée et que TOUS les disques sont montés via uuid. Pour le disque de système d’exploitation, vérifiez que l’entrée du chargeur de démarrage correspond également au montage uuid.
>
>

---
#### <a name="preparation-for-deploying-a-vm-with-a-customer-specific-image-for-sap"></a><a name="57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3"></a>Préparation du déploiement d’une machine virtuelle avec une image spécifique du client pour SAP

Les fichiers VHD contenant un système d’exploitation généralisé sont stockés dans des conteneurs dans les comptes de stockage Azure ou en tant qu’images de disque managé. Vous pouvez déployer une nouvelle machine virtuelle à partir de ce type d’image en référençant le VHD ou l’image de disque managé comme source dans vos fichiers de modèle de déploiement. Consultez le chapitre [Scénario 2 : Déploiement d’une machine virtuelle avec une image personnalisée pour SAP][deployment-guide-3.3] du [Guide de déploiement][deployment-guide].

Les exigences à respecter pour la préparation de votre propre image de machine virtuelle Azure sont les suivantes :

* À l’origine, la taille maximale du VHD contenant le système d’exploitation était seulement de 127 Go. Cette limite a été repoussée en mars 2015. Désormais, la taille du VHD contenant le système d’exploitation peut aller jusqu’à 1 To, comme tous les autres VHD hébergés sur le Stockage Azure.
* Il doit être au format VHD fixe. Les disques durs virtuels ou les disques durs virtuels au format VHDx ne sont pas encore pris en charge sur Azure. Les disques durs virtuels dynamiques seront convertis en disques durs virtuels statiques lorsque vous chargez le disque dur virtuel avec les applets de commande PowerShell ou l’interface CLI
* Les VHD montés sur la machine virtuelle et qui doivent être remontés sur celle-ci dans Azure, doivent également être au format VHD fixe. Lisez [cet article](../../managed-disks-overview.md) pour connaître les limites de taille des disques de données. Les disques durs virtuels dynamiques seront convertis en disques durs virtuels statiques lorsque vous chargez le disque dur virtuel avec les applets de commande PowerShell ou l’interface CLI
* Ajoutez d’autres comptes locaux, car ceux-ci peuvent être nécessaires pour le scénario de déploiement spécifique.
* Si l’image contient une installation de SAP NetWeaver et si le renommage du nom d’hôte au moment du déploiement Azure est probable, il est recommandé de copier les dernières versions du DVD du gestionnaire de déploiement de logiciels SAP dans le modèle. Cela vous permettra d’utiliser la fonction de renommage fournie par SAP pour adapter le nom d’hôte modifié et/ou changer le SID du système SAP dans l’image de machine virtuelle déployée dès qu’une nouvelle copie est démarrée.

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Assurez-vous que le lecteur D:\ n’est pas un montage automatique de disque défini pour les disques attachés, comme décrit dans le chapitre [Paramétrage du montage automatique pour les disques attachés][planning-guide-5.5.3] dans ce document.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Assurez-vous que/mnt/resource n’est pas utilisée et que TOUS les disques sont montés via uuid. Pour le disque de système d’exploitation, vérifiez que l’entrée du chargeur de démarrage correspond également au montage uuid.
>
>

---
* L’interface utilisateur graphique SAP peut être installée dans un tel modèle (à des fins administratives et de configuration).
* D’autres logiciels nécessaires à l’exécution réussie des machines virtuelles dans les scénarios de coexistence entre sites peuvent être installés tant que ceux-ci fonctionnent avec la machine virtuelle renommée.

Si la préparation de la machine virtuelle permet à cette dernière d’être générique, voire indépendante des comptes/utilisateurs non disponibles pour le scénario de déploiement Azure ciblé, la dernière étape de préparation de la généralisation d’une telle image est effectuée.

##### <a name="generalizing-a-vm"></a>Généralisation d’une machine virtuelle
---
> ![Logo Windows.][Logo_Windows] Windows
>
> La dernière étape consiste à se connecter à une machine virtuelle avec un compte d’administrateur. Ouvrez une fenêtre de commande Windows en tant *qu’administrateur*. Accédez à %windir%\windows\system32\sysprep et exécutez sysprep.exe.
> Une petite fenêtre s’affiche. N’oubliez pas de cocher l’option **Généraliser** (décochée par défaut) et de remplacer la valeur par défaut de l’option Arrêter (Redémarrer) par « Arrêter ». Cette procédure suppose que le processus sysprep est exécuté localement dans le SE invité d’une machine virtuelle.
> Si vous souhaitez exécuter la procédure avec une machine virtuelle en cours d’exécution dans Azure, suivez les étapes décrites dans [cet article](../../windows/capture-image-resource.md).
>
> ![Logo Linux.][Logo_Linux] Linux
>
> [Comment capturer une machine virtuelle Linux à utiliser comme modèle Resource Manager][capture-image-linux-step-2-create-vm-image]
>
>

---
### <a name="transferring-vms-and-vhds-between-on-premises-to-azure"></a>Transfert de machines virtuelles et de disques durs virtuels locaux vers Azure
Étant donné que le chargement d’images de machine virtuelle et de disques sur Azure n’est pas possible via le portail Azure, vous devez utiliser les applets de commande Azure PowerShell ou l’interface de ligne de commande. Une autre possibilité serait d’utiliser l’outil « AzCopy ». Cet outil peut copier les disques durs virtuels locaux vers Azure (et vice-versa). Il peut également copier les disques durs virtuels entre différentes régions Azure. Pour plus d’informations sur le téléchargement et l’utilisation d’AzCopy, consultez [cette documentation][storage-use-azcopy].

Une troisième alternative consisterait à utiliser divers outils tiers orientés GUI. Toutefois, vérifiez que ces outils prennent en charge les objets blob de pages Azure. Dans notre cas, nous devons utiliser le magasin d’objets blob de pages Azure (les différences sont décrites ici : <https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs>). Les outils fournis par Azure sont efficaces pour compresser les machines virtuelles et les VHD à charger. Il s’agit d’un élément important car plus la compression est efficace, plus le temps de chargement est réduit. (Ce temps varie tout de même en fonction du lien de chargement vers Internet depuis l’emplacement local et de la région de déploiement Azure ciblée). Il est logique de supposer que le chargement d’une machine virtuelle ou d’un VHD à partir d’un emplacement européen sur un centre de données Azure se trouvant aux États-Unis prendra plus de temps que le chargement de ces machines virtuelles/VHD vers les centres de données Azure européens.

#### <a name="uploading-a-vhd-from-on-premises-to-azure"></a><a name="a43e40e6-1acc-4633-9816-8f095d5a7b6a"></a>Chargement d’un disque dur virtuel local vers Azure
Pour charger une machine virtuelle ou un disque dur virtuel existants à partir du réseau local, ces derniers doivent répondre aux exigences décrites dans le chapitre [Préparation du déplacement d’une machine virtuelle locale vers Azure avec un disque non généralisé][planning-guide-5.2.1] dans ce document.

Cette machine virtuelle n’a PAS besoin d’être généralisée et peut être chargée dans l’état et la forme dans lesquels elle se trouvait après l’arrêt du côté local. C’est la même chose pour les VHD supplémentaires qui n’ont pas de système d’exploitation.

##### <a name="uploading-a-vhd-and-making-it-an-azure-disk"></a>Chargement d’un disque dur virtuel en vue d’en faire un disque Azure
Dans ce scénario, nous souhaitons charger un disque dur virtuel, avec ou sans système d’exploitation, et le monter sur une machine virtuelle en tant que disque de données ou l’utiliser en tant que disque de système d’exploitation. Ce processus comprend plusieurs étapes

**PowerShell**

* Connectez-vous à votre abonnement avec *Connect-AzAccount*.
* Définissez l’abonnement de votre contexte en entrant *Set-AzContext* et le paramètre SubscriptionId ou SubscriptionName - consultez <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>
* Chargez le disque dur virtuel (VHD) avec *Add-AzVhd* sur un compte de stockage Azure - consultez <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>
* (Facultatif) Créez un disque managé à partir du VHD avec *New-AzDisk* - consultez <https://docs.microsoft.com/powershell/module/az.compute/new-Azdisk>
* Définissez le disque du système d’exploitation d’une nouvelle configuration de machine virtuelle sur le VHD ou le disque managé avec *Set-AzVMOSDisk* - consultez <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>
* Créez une machine virtuelle à partir de la configuration de machine virtuelle avec *New-AzVM* - consultez <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>
* Ajoutez un disque de données à une nouvelle machine virtuelle en entrant *Add-AzVMDataDisk* - consultez <https://docs.microsoft.com/powershell/module/az.compute/add-Azvmdatadisk>

**Azure CLI**

* Connectez-vous à votre abonnement avec *az login*
* Sélectionnez votre abonnement en entrant *az account set --subscription `<subscription name or id`>*
* Chargez le VHD en entrant *az storage blob upload*. Consultez [Utilisation de l’interface de ligne de commande Azure avec Stockage Azure][storage-azure-cli].
* (Facultatif) Créez un disque managé à partir du disque dur virtuel avec *az disk create* - consultez https://docs.microsoft.com/cli/azure/disk
* Créez une machine virtuelle et spécifiez le disque managé ou le VHD chargé en tant que disque du système d’exploitation en entrant *az vm create* et le paramètre *--attach-os-disk*
* Ajoutez un disque de données à une nouvelle machine virtuelle en entrant *az vm disk attach* et le paramètre *--new*

**Modèle**

* Chargez le disque dur virtuel avec PowerShell ou l'interface de ligne de commande Azure (Azure CLI)
* (Facultatif) Créez un disque managé à partir du VHD avec PowerShell, Azure CLI ou le portail Azure
* Déployez la machine virtuelle avec un gabarit JSON en référençant le VHD comme indiqué dans [cet exemple de gabarit JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json)ou en utilisant la fonctionnalité Disques managés comme indiqué dans [cet exemple de gabarit JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json).

#### <a name="deployment-of-a-vm-image"></a>Déploiement d’une image de machine virtuelle
Pour charger une machine virtuelle ou un VHD existants à partir du réseau local afin de les utiliser comme une image de machine virtuelle Azure, cette machine virtuelle ou ce VHD doit respecter les exigences listées dans le chapitre [Préparation du déploiement d’une machine virtuelle avec une image spécifique du client pour SAP][planning-guide-5.2.2] dans ce document.

* Utilisez *sysprep* sur Windows ou *waagent -deprovision* sur Linux afin de généraliser votre machine virtuelle. Voir [Informations techniques de référence de Sysprep](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)) pour Windows ou [Comment capturer une machine virtuelle Linux à utiliser en tant que modèle Resource Manager][capture-image-linux-step-2-create-vm-image] pour Linux.
* Connectez-vous à votre abonnement avec *Connect-AzAccount*.
* Définissez l’abonnement de votre contexte en entrant *Set-AzContext* et le paramètre SubscriptionId ou SubscriptionName - consultez <https://docs.microsoft.com/powershell/module/az.accounts/set-Azcontext>
* Chargez le disque dur virtuel (VHD) avec *Add-AzVhd* sur un compte de stockage Azure - consultez <https://docs.microsoft.com/powershell/module/az.compute/add-Azvhd>
* (Facultatif) Créez une image de disque managé à partir du VHD avec *New-AzImage* - consultez <https://docs.microsoft.com/powershell/module/az.compute/new-Azimage>
* Définissez le disque du système d’exploitation d’une nouvelle configuration de machine virtuelle sur
  * Le VHD avec *Set-AzVMOSDisk -SourceImageUri -CreateOption fromImage* - consultez <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmosdisk>
  * L’image de disque managé avec *Set-AzVMSourceImage* - consultez <https://docs.microsoft.com/powershell/module/az.compute/set-Azvmsourceimage>
* Créez une machine virtuelle à partir de la configuration de machine virtuelle avec *New-AzVM* - consultez <https://docs.microsoft.com/powershell/module/az.compute/new-Azvm>

**Azure CLI**

* Utilisez *sysprep* sur Windows ou *waagent -deprovision* sur Linux afin de généraliser votre machine virtuelle. Voir [Informations techniques de référence de Sysprep](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)) pour Windows ou [Comment capturer une machine virtuelle Linux à utiliser en tant que modèle Resource Manager][capture-image-linux-step-2-create-vm-image] pour Linux.
* Connectez-vous à votre abonnement avec *az login*
* Sélectionnez votre abonnement en entrant *az account set --subscription `<subscription name or id`>*
* Chargez le VHD en entrant *az storage blob upload*. Consultez [Utilisation de l’interface de ligne de commande Azure avec Stockage Azure][storage-azure-cli].
* (Facultatif) Créez une image de disque managé à partir du disque dur virtuel avec *az image create* - consultez https://docs.microsoft.com/cli/azure/image
* Créez une machine virtuelle et spécifiez l’image de disque managé ou le VHD chargé en tant que disque du système d’exploitation en entrant *az vm create* et le paramètre *--image*

**Modèle**

* Utilisez *sysprep* sur Windows ou *waagent -deprovision* sur Linux afin de généraliser votre machine virtuelle. Voir [Informations techniques de référence de Sysprep](/previous-versions/windows/it-pro/windows-vista/cc766049(v=ws.10)) pour Windows ou [Comment capturer une machine virtuelle Linux à utiliser en tant que modèle Resource Manager][capture-image-linux-step-2-create-vm-image] pour Linux.
* Chargez le disque dur virtuel avec PowerShell ou l'interface de ligne de commande Azure (Azure CLI)
* (Facultatif) Créez une image de disque managé à partir du VHD avec PowerShell, Azure CLI ou le portail Azure
* Déployez la machine virtuelle avec un gabarit JSON en référençant le VHD image comme indiqué dans [cet exemple de gabarit JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-specialized-vhd-new-or-existing-vnet/azuredeploy.json)ou en utilisant l’image de disque managé comme indiqué dans [cet exemple de gabarit JSON](https://github.com/Azure/azure-quickstart-templates/blob/master/sap-2-tier-user-image-md/azuredeploy.json).

#### <a name="downloading-vhds-or-managed-disks-to-on-premises"></a>Téléchargement de VHD ou de la fonctionnalité Disques managés au niveau local
IaaS permet bien plus que le chargement de disques durs virtuels et de systèmes SAP. Vous pouvez également déplacer les systèmes SAP de nouveau en local à partir d’Azure.

Les VHD ou les disques managés ne peuvent pas être actifs pendant le téléchargement. La machine virtuelle doit être arrêtée et désallouée, même pendant le téléchargement des disques montés sur la machine virtuelle. Si vous souhaitez seulement télécharger le contenu de la base de données, qui doit ensuite être utilisé pour configurer un nouveau système local, et s'il est acceptable que le système reste opérationnel dans Azure pendant la durée du téléchargement et de l'installation du nouveau système, vous pouvez éviter un long temps d'arrêt en effectuant une sauvegarde de base de données compressée sur un disque et en téléchargeant seulement ce disque au lieu de télécharger aussi la machine virtuelle de base du système d'exploitation.

#### <a name="powershell"></a>PowerShell

* Téléchargement d’un disque managé : vous devez d’abord obtenir l’accès à l’objet blob sous-jacent du disque managé. Vous pouvez ensuite copier l’objet blob sous-jacent dans un nouveau compte de stockage et télécharger l’objet blob à partir de ce compte de stockage.

  ```powershell
  $access = Grant-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name> -Access Read -DurationInSecond 3600
  $key = (Get-AzStorageAccountKey -ResourceGroupName <resource group> -Name <storage account name>)[0].Value
  $destContext = (New-AzStorageContext -StorageAccountName <storage account name -StorageAccountKey $key)
  Start-AzStorageBlobCopy -AbsoluteUri $access.AccessSAS -DestContainer <container name> -DestBlob <blob name> -DestContext $destContext
  # Wait for blob copy to finish
  Get-AzStorageBlobCopyState -Container <container name> -Blob <blob name> -Context $destContext
  Save-AzVhd -SourceUri <blob in new storage account> -LocalFilePath <local file path> -StorageKey $key
  # Wait for download to finish
  Revoke-AzDiskAccess -ResourceGroupName <resource group> -DiskName <disk name>
  ```

* Téléchargement d’un VHD : une fois le système SAP et la machine virtuelle arrêtés, vous pouvez utiliser la cmdlet PowerShell `Save-AzVhd` sur la cible locale pour retélécharger les VHD en local. Pour ce faire, vous avez besoin de l’URL du VHD, qui se trouve dans la section Stockage du portail Azure (accédez au compte de stockage et au conteneur de stockage où le VHD a été créé), et de savoir où le VHD doit être copié.

  Vous pouvez ensuite utiliser la commande en définissant le paramètre SourceUri comme l’URL du VHD à télécharger et LocalFilePath comme l’emplacement physique du VHD (y compris son nom). La commande pourrait ressembler à :

  ```powershell
  Save-AzVhd -ResourceGroupName <resource group name of storage account> -SourceUri http://<storage account name>.blob.core.windows.net/<container name>/sapidedata.vhd -LocalFilePath E:\Azure_downloads\sapidesdata.vhd
  ```

  Pour plus d’informations sur la cmdlet Save-AzVhd, consultez l’article accessible à l’adresse : <https://docs.microsoft.com/powershell/module/az.compute/save-Azvhd>.

#### <a name="azure-cli"></a>Azure CLI
* Téléchargement d’un disque managé : vous devez d’abord obtenir l’accès à l’objet blob sous-jacent du disque managé. Vous pouvez ensuite copier l’objet blob sous-jacent dans un nouveau compte de stockage et télécharger l’objet blob à partir de ce compte de stockage.

  ```azurecli
  az disk grant-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --duration-in-seconds 3600
  az storage blob download --sas-token "<sas token>" --account-name <account name> --container-name <container name> --name <blob name> --file <local file>
  az disk revoke-access --ids "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>"
  ```

* Téléchargement d’un VHD : une fois le système SAP et la machine virtuelle arrêtés, vous pouvez utiliser la commande Azure CLI `_azure storage blob download_` sur la cible locale pour retélécharger les VHD en local. Pour ce faire, vous avez besoin du nom et du conteneur du VHD, qui se trouvent dans la section Stockage du portail Azure (accédez au compte de stockage et au conteneur de stockage où le VHD a été créé), et de savoir où le VHD doit être copié.

  Vous pouvez ensuite utiliser la commande en définissant les paramètres d’objet blob et de conteneur du VHD à télécharger et la destination comme l’emplacement physique cible du VHD (y compris son nom). La commande pourrait ressembler à :

  ```azurecli
  az storage blob download --name <name of the VHD to download> --container-name <container of the VHD to download> --account-name <storage account name of the VHD to download> --account-key <storage account key> --file <destination of the VHD to download>
  ```

### <a name="transferring-vms-and-disks-within-azure"></a>Transfert de machines virtuelles et de disques au sein d’Azure

#### <a name="copying-sap-systems-within-azure"></a>Copie des systèmes SAP au sein d’Azure

Un système SAP ou même un serveur SGBD dédié prenant en charge une couche Application SAP est constitué de plusieurs disques contenant le système d’exploitation avec des fichiers binaires ou des fichiers journaux et de données de la base de données SAP. Les fonctionnalités Azure de copie de disques et d’enregistrement de disques sur un disque local n’ont pas de mécanisme de synchronisation permettant de créer un instantané de plusieurs disques de façon cohérente. Ainsi, l’état des disques copiés ou enregistrés serait différent, même si ceux-ci sont montés sur la même machine virtuelle. Cela signifie que, si les fichiers données et journaux contenus dans les divers disques étaient différents, la base de données serait incohérente.

**Conclusion : Pour copier ou enregistrer des disques dans le cadre de la configuration d’un système SAP, vous devez arrêter celui-ci ainsi que la machine virtuelle déployée. Ce n’est qu’à cette condition que vous pouvez copier ou télécharger l’ensemble de disques afin de créer une copie du système SAP dans Azure ou en local.**

Les disques de données peuvent être stockés en tant que fichiers VHD dans un compte de stockage Azure et peuvent être directement attachés à une machine virtuelle ou utilisés en tant qu’image. Dans ce cas, le VHD est copié vers un autre emplacement avant d’être attaché à la machine virtuelle. Le nom complet du fichier VHD dans Azure doit être unique dans Azure. Comme mentionné plus tôt, le nom est constitué de trois éléments :

`http(s)://<storage account name>.blob.core.windows.net/<container name>/<vhd name>`

Les disques de données peuvent également être des disques managés. Dans ce cas, le disque managé est utilisé pour créer un disque managé avant d’être attaché à la machine virtuelle. Le nom du disque managé doit être unique au sein d’un groupe de ressources.

##### <a name="powershell"></a>PowerShell

Vous pouvez utiliser des cmdlets Azure PowerShell pour copier un disque dur virtuel, comme illustré dans [cet article][storage-powershell-guide-full-copy-vhd]. Pour créer un disque managé, utilisez New-AzDiskConfig et New-AzDisk comme indiqué dans l’exemple suivant.

```powershell
$config = New-AzDiskConfig -CreateOption Copy -SourceUri "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" -Location <location>
New-AzDisk -ResourceGroupName <resource group name> -DiskName <disk name> -Disk $config
```

##### <a name="azure-cli"></a>Azure CLI

Vous pouvez utiliser Azure CLI pour copier un disque dur virtuel. Pour créer un disque managé, utilisez *az disk create* comme indiqué dans l’exemple suivant.

```azurecli
az disk create --source "/subscriptions/<subscription id>/resourceGroups/<resource group>/providers/Microsoft.Compute/disks/<disk name>" --name <disk name> --resource-group <resource group name> --location <location>
```

##### <a name="azure-storage-tools"></a>Outils Azure Storage

* <https://storageexplorer.com/>

Des éditions professionnelles de l’Explorateur de stockage Azure se trouvent ici :

* <https://www.cerebrata.com/>
* <https://clumsyleaf.com/products/cloudxplorer>

La copie d’un VHD se trouvant dans un compte de stockage est un processus qui ne dure que quelques secondes (tout comme le matériel SAN qui crée des instantanés avec copie différée et copie sur écriture). Une fois que vous avez une copie du fichier VHD, vous pouvez l’attacher à une machine virtuelle ou l’utiliser comme image pour joindre des copies du VHD aux machines virtuelles.

##### <a name="powershell"></a>PowerShell

```powershell
# attach a vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -VhdUri <path to vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name newdatadisk -ManagedDiskId <managed disk id> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption attach
$vm | Update-AzVM

# attach a copy of the vhd to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$vm = Add-AzVMDataDisk -VM $vm -Name <disk name> -VhdUri <new path of vhd> -SourceImageUri <path to image vhd> -Caching <caching option> -DiskSizeInGB $null -Lun <lun, for example 0> -CreateOption fromImage
$vm | Update-AzVM

# attach a copy of the managed disk to a vm
$vm = Get-AzVM -ResourceGroupName <resource group name> -Name <vm name>
$diskConfig = New-AzDiskConfig -Location $vm.Location -CreateOption Copy -SourceUri <source managed disk id>
$disk = New-AzDisk -DiskName <disk name> -Disk $diskConfig -ResourceGroupName <resource group name>
$vm = Add-AzVMDataDisk -VM $vm -Caching <caching option> -Lun <lun, for example 0> -CreateOption attach -ManagedDiskId $disk.Id
$vm | Update-AzVM
```

##### <a name="azure-cli"></a>Azure CLI

```azurecli

# attach a vhd to a vm
az vm unmanaged-disk attach --resource-group <resource group name> --vm-name <vm name> --vhd-uri <path to vhd>

# attach a managed disk to a vm
az vm disk attach --resource-group <resource group name> --vm-name <vm name> --disk <managed disk id>

# attach a copy of the vhd to a vm
# this scenario is currently not possible with Azure CLI. A workaround is to manually copy the vhd to the destination.

# attach a copy of a managed disk to a vm
az disk create --name <new disk name> --resource-group <resource group name> --location <location of target virtual machine> --source <source managed disk id>
az vm disk attach --disk <new disk name or managed disk id> --resource-group <resource group name> --vm-name <vm name> --caching <caching option> --lun <lun, for example 0>
```

#### <a name="copying-disks-between-azure-storage-accounts"></a><a name="9789b076-2011-4afa-b2fe-b07a8aba58a1"></a>Copie de disques entre comptes Azure Storage
Il est impossible d’effectuer cette tâche sur le portail Azure. Vous pouvez utiliser les applets de commande Azure PowerShell, Azure CLI ou un navigateur de stockage tiers. Les applets de commande PowerShell ou les commandes CLI peuvent créer et gérer des objets blob. Elles permettent de copier des objets blob de façon asynchrone entre comptes de stockage et régions au sein de l’abonnement Azure.

##### <a name="powershell"></a>PowerShell
Vous pouvez également copier des VHD entre des abonnements. Pour plus d’informations, consultez [cet article][storage-powershell-guide-full-copy-vhd].

Le flux de base de la logique d’applet de commande PS ressemble à ceci :

* Créez un contexte pour le compte de stockage **source** en entrant *New-AzStorageContext* - consultez <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>
* Créez un contexte pour le compte de stockage **cible** en entrant *New-AzStorageContext* - consultez <https://docs.microsoft.com/powershell/module/az.storage/new-AzStoragecontext>
* Démarrer la copie avec

```powershell
Start-AzStorageBlobCopy -SrcBlob <source blob name> -SrcContainer <source container name> -SrcContext <variable containing context of source storage account> -DestBlob <target blob name> -DestContainer <target container name> -DestContext <variable containing context of target storage account>
```

* Vérifier l’état de la copie dans une boucle avec

```powershell
Get-AzStorageBlobCopyState -Blob <target blob name> -Container <target container name> -Context <variable containing context of target storage account>
```

* Joindre le nouveau disque dur virtuel à une machine virtuelle comme décrit ci-dessus.

Pour découvrir des exemples, consultez [cet article][storage-powershell-guide-full-copy-vhd].

##### <a name="azure-cli"></a>Azure CLI
* Démarrer la copie avec

```azurecli
az storage blob copy start --source-blob <source blob name> --source-container <source container name> --source-account-name <source storage account name> --source-account-key <source storage account key> --destination-container <target container name> --destination-blob <target blob name> --account-name <target storage account name> --account-key <target storage account name>
```

* Vérifier l’état si la copie est toujours dans une boucle avec

```azurecli
az storage blob show --name <target blob name> --container <target container name> --account-name <target storage account name> --account-key <target storage account name>
```

* Joindre le nouveau disque dur virtuel à une machine virtuelle comme décrit ci-dessus.

### <a name="disk-handling"></a>Gestion de disque

#### <a name="vmdisk-structure-for-sap-deployments"></a><a name="4efec401-91e0-40c0-8e64-f2dceadff646"></a>Structure de machine virtuelle/disque pour les déploiements SAP

Dans l’idéal, la gestion de la structure d’une machine virtuelle et des disques associés doit être simple. Dans les installations locales, les clients ont développé de nombreuses méthodes pour structurer une installation du serveur.

* Un disque de base qui contient le système d’exploitation et tous les fichiers binaires du système SGBD et/ou SAP. Depuis mars 2015, ce disque peut atteindre une taille de 1 To, au lieu des 127 Go précédents.
* Un ou plusieurs disques contenant le fichier journal SGBD de la base de données SAP et le fichier journal de la zone de stockage temporaire SGBD (si le SGBD les prend en charge). Si les exigences d’E/S par seconde du journal de base de données sont élevées, vous devez entrelacer plusieurs disques afin d’atteindre le volume d’E/S par seconde requis.
* Un certain nombre de disques contenant un ou deux fichiers de base de données de la base de données SAP ainsi que les fichiers de données temporaires SGBD (si le SGBD les prend en charge).

![Configuration de référence de la machine virtuelle IaaS Azure pour SAP][planning-guide-figure-1300]


---
> ![Logo Windows.][Logo_Windows] Windows
>
> Pour de nombreux clients, nous avons remarqué des configurations où les fichiers binaires SAP et SGBD n’étaient pas installés sur le lecteur c:\ où le système d’exploitation a été installé, par exemple. Plusieurs facteurs pouvaient expliquer cela, mais lorsque nous avons examiné la cause racine, il s’est avéré que les lecteurs étaient petits et les mises à niveau du système d’exploitation nécessitaient un espace supplémentaire il y a de cela 10 à 15 ans. Ces deux conditions s’appliquent bien moins souvent de nos jours. Aujourd’hui, le lecteur c:\ peut être mappé sur des machines virtuelles ou des disques de volume important. Afin de conserver une structure simple des déploiements, il est recommandé de suivre le modèle de déploiement suivant pour les systèmes SAP NetWeaver dans Azure
>
> Le fichier d’échange du système d’exploitation Windows doit se trouver sur le lecteur D: (disque non persistant)
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Placez le fichier d’échange Linux dans /mnt/mnt/resource sous Linux, comme décrit dans [cet article][virtual-machines-linux-agent-user-guide]. Le fichier d’échange peut être configuré dans le fichier de configuration de l’agent Linux /etc/waagent.conf. Ajoutez ou modifiez les paramètres suivants :
>
>

```console
ResourceDisk.EnableSwap=y
ResourceDisk.SwapSizeMB=30720
```

Pour activer les modifications, vous devez redémarrer l’agent Linux avec

```console
sudo service waagent restart
```

Pour plus d’informations sur la taille recommandée du fichier d’échange, consultez la note SAP [1597355].

---
Le nombre de disques utilisés pour les fichiers de données SGBD et le type de stockage Azure sur lequel ces disques sont hébergés doivent être déterminés par rapport aux exigences d’E/S par seconde et par la latence requise. Les quotas exacts sont décrits dans [cet article (Linux)][virtual-machines-sizes-linux] et [cet article (Windows)][virtual-machines-sizes-windows].

Les leçons que nous avons tirées de l’expérience des déploiements SAP au cours des 2 dernières années peuvent être résumées comme suit :

* Le trafic d’E/S par seconde vers différents fichiers de données n’est pas toujours le même dans la mesure où les systèmes clients existants peuvent contenir des fichiers de données représentant leur(s) base(s) de données SAP de taille différente. Ainsi, l’utilisation d’une configuration RAID s’est avérée plus efficace que plusieurs disques pour placer des fichiers de données LUN issus de ceux-ci. Dans certaines situations, surtout en ce qui concerne le stockage Azure Standard, la fréquence d’E/S par seconde atteignait le quota d’un seul disque par rapport au journal des transactions SGBD. Dans ce type de scénario, nous vous recommandons d’utiliser le stockage Premium ou d’agréger plusieurs disques de stockage Standard à l’aide d’une frange logicielle.

---
> ![Logo Windows.][Logo_Windows] Windows
>
> * [Meilleures pratiques relatives aux performances de SQL Server dans les machines virtuelles Azure][virtual-machines-sql-server-performance-best-practices]
>
> ![Logo Linux.][Logo_Linux] Linux
>
> * [Configuration d’un RAID logiciel sur Linux][virtual-machines-linux-configure-raid]
> * [Configurer LVM sur une machine virtuelle Linux dans Azure][virtual-machines-linux-configure-lvm]
>
>

---
* Premium Storage affiche des performances en hausse, en particulier pour les écritures du journal des transactions critiques. Pour les scénarios SAP censés stimuler les performances de production, il est recommandé d’utiliser des séries de machines virtuelles pouvant tirer parti du stockage Azure Premium Storage.

N’oubliez pas que le disque qui contient le système d’exploitation et, comme nous le recommandons, les binaires de SAP ainsi que la base de données (machine virtuelle de base) n’est plus limité à 127 Go. Sa taille peut désormais atteindre 1 To. Cette taille devrait être suffisante pour conserver tous les fichiers nécessaires, par exemple, les journaux d’activité de traitements par lots SAP.

Pour plus de suggestions et de détails, en particulier pour les machines virtuelles SGBD, consultez le [Guide de déploiement de SGBD][dbms-guide].

#### <a name="disk-handling"></a>Gestion de disque

Dans la plupart des scénarios, vous devez créer des disques supplémentaires pour déployer la base de données SAP sur la machine virtuelle. Nous avons abordé les éléments à prendre en compte pour un certain nombre de disques dans le chapitre [Structure de machine virtuelle/disque pour les déploiements SAP][planning-guide-5.5.1] dans ce document. Le portail Azure permet d’attacher et de détacher des disques de données une fois qu’une machine virtuelle de base est déployée. Les disques peuvent être attachés/détachés lorsque la machine virtuelle est en cours d’exécution, ainsi que lorsque celle-ci est arrêtée. Quand vous attachez un disque, le portail Azure propose d’attacher un disque vide ou un disque existant qui n’est pas encore attaché à une autre machine virtuelle.

**Remarque** : Les disques peuvent être attachés à une seule machine virtuelle à la fois.

![Attacher/Détacher des disques avec le stockage Azure standard][planning-guide-figure-1400]

Pendant le déploiement d’une nouvelle machine virtuelle, vous pouvez décider si vous souhaitez utiliser la fonctionnalité Disques managés ou placer vos disques sur des comptes de stockage Azure. Si vous souhaitez utiliser un Stockage Premium, nous vous recommandons de recourir à la fonctionnalité Disques managés.

Ensuite, vous devez décider si vous souhaitez créer un disque vide ou si vous souhaitez sélectionner un disque existant qui a été chargé plus tôt et doit désormais être attaché à la machine virtuelle.

**IMPORTANT** : Vous **NE DEVEZ PAS** utiliser la mise en cache de l’hôte avec un stockage Standard Azure. Vous devez conserver les préférences de Cache hôte sur Aucun par défaut. Avec le stockage Azure Premium, vous devez activer la Mise en cache en lecture si les caractéristiques d’E/S se lisent principalement comme du trafic d’E/S standard sur des fichiers de données de base de données. En cas de fichier journal des transactions de base de données, la mise en cache n’est pas recommandée.

---
> ![Logo Windows.][Logo_Windows] Windows
>
> [Attachement d’un disque de données dans le portail Azure][virtual-machines-linux-attach-disk-portal]
>
> Si les disques sont attachés, vous devez vous connecter à la machine virtuelle pour ouvrir le Gestionnaire de disque Windows. Si le montage automatique n’est pas activé comme le recommande le chapitre [Paramétrage du montage automatique pour les disques attachés][planning-guide-5.5.3], le volume nouvellement attaché doit être mis en ligne et initialisé.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Si les disques sont attachés, vous devez vous connecter à la machine virtuelle et initialiser les disques, comme décrit dans [cet article][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux].
>
>

---
Si le nouveau disque est un disque vide, vous devez également le formater. Pour la mise en forme, surtout en ce qui concerne les fichiers journaux et données SGBD, les mêmes recommandations que pour les déploiements sur système nu SGBD s’appliquent.

Un compte de Stockage Azure ne fournit pas de ressources infinies en termes de volume d'E/S, d'IOPS et de volume de données. Généralement, les machines virtuelles SGBD en sont les plus affectées. Il peut être préférable d’utiliser un compte de stockage distinct pour chaque machine virtuelle si vous avez la possibilité de déployer quelques machines virtuelles à un volume d’E/S élevé afin de rester dans la limite du volume du compte de stockage Azure. Dans le cas contraire, vous devez déterminer comment répartir de manière équilibrée ces machines virtuelles entre les différents comptes de stockage sans atteindre la limite de chaque compte de stockage. Pour plus de détails, consultez le [Guide de déploiement de SGBD][dbms-guide]. Vous devez également garder ces limitations à l’esprit pour les machines virtuelles de serveur d’application SAP pures ou d’autres machines virtuelles pouvant potentiellement nécessiter plus de VHD. Ces restrictions ne s’appliquent pas si vous utilisez des disques managés. Si vous envisagez d’utiliser un Stockage Premium, nous vous recommandons de recourir à un disque managé.

La géoréplication des VHD dans un compte de stockage est également un sujet pertinent en ce qui concerne les comptes de stockage. La géo-réplication est activée ou désactivée au niveau du compte de stockage et non au niveau de la machine virtuelle. Si la géo-réplication est activée, les disques durs virtuels au sein du compte de stockage sont répliqués dans un autre centre de données Azure se trouvant dans la même région. Avant de prendre votre décision, considérez la restriction suivante :

La géo-réplication Azure fonctionne localement sur chaque disque dur virtuel d’une machine virtuelle et ne réplique pas les E/S par ordre chronologique sur plusieurs disques durs virtuels d’une machine virtuelle. Par conséquent, le disque dur virtuel qui représente la machine virtuelle de base, ainsi que tout disque dur virtuel supplémentaire attaché à la machine virtuelle, est répliqué indépendamment des autres disques durs virtuels. Ainsi, il n’existe aucune synchronisation des modifications des différents disques durs virtuels. Le fait que les E/S sont répliquées indépendamment de l’ordre dans lequel elles sont écrites signifie que la géo-réplication n’a pas de valeur pour les serveurs de base de données dont les bases de données sont distribuées sur plusieurs disques durs virtuels. Outre le SGBD, il existe peut-être d’autres applications dans lesquelles les processus écrivent ou manipulent des données dans différents disques durs virtuels et où il est important de conserver l’ordre des modifications. Si ce dernier est important, la géo-réplication dans Azure ne doit pas être activée. Si vous souhaitez ou avez besoin de la géo-réplication pour un ensemble de machines virtuelles, mais pas pour un autre ensemble, vous pouvez déjà catégoriser les machines virtuelles et leurs disques durs virtuels correspondants dans différents comptes de stockage dont la géo-réplication est activée ou désactivée.

#### <a name="setting-automount-for-attached-disks"></a><a name="17e0d543-7e8c-4160-a7da-dd7117a1ad9d"></a>Paramétrage du montage automatique pour les disques attachés
---
> ![Logo Windows.][Logo_Windows] Windows
>
> Pour les machines virtuelles créées à partir de vos propres images ou disques, vous devez vérifier et éventuellement définir le paramètre de montage automatique. Ce paramètre permet à la machine virtuelle de monter les lecteurs attachés/montés de nouveau de manière automatique suite à un redémarrage ou un redéploiement dans Azure .
> Le paramètre est défini pour les images fournies par Microsoft dans la Place de marché Azure.
>
> Pour définir le montage automatique, consultez la documentation de l’exécutable de ligne de commande diskpart.exe ici :
>
> * [Options de ligne de commande DiskPart](/previous-versions/windows/it-pro/windows-xp/bb490893(v=technet.10))
> * [Montage automatique](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc753703(v=ws.11))
>
> La fenêtre de ligne de commande Windows doit être ouverte en tant qu’administrateur.
>
> Si les disques sont attachés, vous devez vous connecter à la machine virtuelle pour ouvrir le Gestionnaire de disque Windows. Si le montage automatique n’est pas activé comme le recommande le chapitre [Paramétrage du montage automatique pour les disques attachés][planning-guide-5.5.3], le volume nouvellement attaché doit être mis en ligne et initialisé.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Vous devez initialiser un disque vide nouvellement attaché comme décrit dans [cet article][virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux].
> Vous devez également ajouter de nouveaux disques au /etc/fstab.
>
>

---
### <a name="final-deployment"></a>Déploiement final

Pour plus d’informations sur le déploiement final et ses étapes exactes, en particulier le déploiement de l’extension Azure pour SAP, consultez le [guide de déploiement][deployment-guide].

## <a name="accessing-sap-systems-running-within-azure-vms"></a>Accès aux systèmes SAP s’exécutant dans des machines virtuelles Azure

Les procédures suivantes doivent être appliquées dans le cas de scénarios dans lesquels vous souhaitez vous connecter à ces systèmes SAP via l’Internet public à l’aide de l’interface utilisateur graphique de SAP.

Plus loin dans le document, nous abordons l’autre scénario principal : la connexion aux systèmes SAP dans les déploiements entre sites locaux qui ont une connexion site à site (tunnel VPN) ou une connexion Azure ExpressRoute entre les systèmes locaux et les systèmes Azure.

### <a name="remote-access-to-sap-systems"></a>Accès à distance aux systèmes SAP

Avec Azure Resource Manager, ce modèle n’a plus de point de terminaison par défaut, comme c’était le cas de l’ancien modèle classique. Tous les ports d’une machine virtuelle Azure Resource Manager Azure sont ouverts tant que :

1. Aucun groupe de sécurité réseau n’est défini pour le sous-réseau ou l’interface réseau. Le trafic réseau vers les machines virtuelles Azure peut être sécurisé via ce que l’on appelle « groupes de sécurité réseau ». Pour plus d’informations, voir [Présentation du groupe de sécurité réseau][virtual-networks-nsg]
2. Aucun équilibrage de charge Azure Load Balancer n’est défini pour l’interface réseau

Voir la différence d’architecture entre le modèle classique et l’ARM, comme décrit dans [cet article][virtual-machines-azure-resource-manager-architecture].

#### <a name="configuration-of-the-sap-system-and-sap-gui-connectivity-over-the-internet"></a>Configuration de la connectivité du système SAP et de l’interface graphique utilisateur de SAP sur Internet

Consultez cet article qui décrit en détail ce sujet : .

<https://docs.microsoft.com/archive/blogs/saponsqlserver/sap-gui-connection-closed-when-connecting-to-sap-system-in-azure>

#### <a name="changing-firewall-settings-within-vm"></a>Modification des paramètres de pare-feu au sein de la machine virtuelle

Il peut être nécessaire de configurer le pare-feu sur vos machines virtuelles pour autoriser le trafic entrant vers le système SAP.

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Par défaut, le Pare-feu Windows au sein d’une machine virtuelle Azure déployée est activé. Vous devez à présent autoriser l’ouverture du port SAP, sinon l’interface utilisateur graphique du SAP ne pourra pas se connecter.
> Pour ce faire :
>
> * Ouvrez Panneau de configuration\Système et sécurité\Pare-feu Windows et sélectionnez **Paramètres avancés**.
> * Cliquez avec le bouton droit sur Règles de trafic entrant, puis cliquez sur **Nouvelle règle**.
> * Dans l’Assistant qui s’ouvre, créez une règle **Port**.
> * Dans l’étape suivante de l’Assistant, laissez le paramètre sur TCP et tapez le numéro de port à ouvrir. Étant donné que notre ID d’instance SAP est 00, nous avons choisi 3200. Si votre instance possède un numéro différent, le port que vous avez défini précédemment en fonction du numéro d’instance doit être ouvert.
> * Dans l’étape suivante de l’Assistant, laissez la case **Autoriser la connexion** cochée.
> * Dans l’étape suivante de l’Assistant, vous devez définir si la règle s’applique au réseau Domaine, Privé et Public. Ajustez-la en fonction de vos besoins. Toutefois, si vous vous connectez avec l’interface graphique utilisateur SAP de l’extérieur via le réseau public, vous devez appliquer la règle au réseau public.
> * Dans la dernière étape de l’Assistant, nommez la règle et enregistrez votre travail en appuyant sur **Terminer**.
>
> La règle entre immédiatement en vigueur.
>
> ![Définition des règles de port][planning-guide-figure-1600]
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Les images Linux dans la Place de marché Azure n’activent pas le pare-feu iptables par défaut et la connexion à votre système SAP devrait fonctionner. Si vous avez activé iptables ou un autre pare-feu, consultez la documentation correspondante pour autoriser le trafic TCP entrant sur le port 32xx (où xx est le numéro de votre système SAP).
>
>

---
#### <a name="security-recommendations"></a>Recommandations de sécurité

L’interface graphique utilisateur SAP ne se connecte pas immédiatement aux instances SAP (port 32xx) en cours d’exécution ; elle se connecte d’abord via le port ouvert vers le processus de serveur de messages SAP (port 36xx). Par le passé, le même port était utilisé par le serveur de messages pour la communication interne vers les instances d’application. Pour empêcher les serveurs d’application locaux de communiquer par inadvertance avec un serveur de messages dans Azure, vous pouvez changer les ports de communication interne. Sur les systèmes ayant été clonés à partir de systèmes locaux, par exemple un clone de développement pour les tests de projet, il est recommandé de basculer la communication interne entre le serveur de messages SAP et ses instances d’application sur un autre numéro de port. Cela peut être fait avec le paramètre de profil par défaut :

> rdisp/msserv_internal
>
>

comme décrit dans l’article [Security Settings for the SAP Message Server](https://help.sap.com/saphelp_nwpi71/helpdata/en/47/c56a6938fb2d65e10000000a42189c/content.htm) (Paramètres de sécurité du serveur de messagerie SAP).


### <a name="single-vm-with-sap-netweaver-demotraining-scenario"></a><a name="3e9c3690-da67-421a-bc3f-12c520d99a30"></a>Machine virtuelle unique avec scénario de démonstration/formation SAP NetWeaver

![Exécution de systèmes de démonstration SAP à machine virtuelle unique portant les mêmes noms de machine virtuelle isolés dans des services cloud Azure][planning-guide-figure-1700]

Dans ce scénario, nous implémentons un système de formation et de démonstration classique dans lequel le scénario complet de formation et de démonstration est contenu dans une seule machine virtuelle. Nous partons du principe que le déploiement s’effectue via des modèles d’image de machine virtuelle. Nous supposons également que plusieurs de ces machines virtuelles de démonstration et de formation doivent être déployées avec les machines virtuelles portant le même nom. Ces systèmes de formation ne disposent d’aucune connectivité avec vos ressources locales et s’opposent à un déploiement hybride.

L’hypothèse est que vous avez créé une image de machine virtuelle, comme le décrivent certaines sections du chapitre [Préparation de machines virtuelles avec SAP pour Azure][planning-guide-5.2] dans ce document.

La séquence d’événements pour implémenter le scénario ressemble à ceci :

##### <a name="powershell"></a>PowerShell

* Création d’un groupe de ressources pour chaque paysage de formation et de démonstration

```powershell
$rgName = "SAPERPDemo1"
New-AzResourceGroup -Name $rgName -Location "North Europe"
```

* Création d’un compte de stockage si vous ne souhaitez pas utiliser la fonctionnalité Disques managés

```powershell
$suffix = Get-Random -Minimum 100000 -Maximum 999999
$account = New-AzStorageAccount -ResourceGroupName $rgName -Name "saperpdemo$suffix" -SkuName Standard_LRS -Kind "Storage" -Location "North Europe"
```

* Création d’un réseau virtuel pour chaque paysage de formation et de démonstration pour rendre possible l’utilisation du même nom d’hôte et des mêmes adresses IP. Le réseau virtuel est protégé par un groupe de sécurité réseau qui autorise uniquement le trafic vers le port 3389 afin de rendre possible l’accès au Bureau à distance et activer le port 22 pour SSH.

```powershell
# Create a new Virtual Network
$rdpRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGRDP -Protocol * -SourcePortRange * -DestinationPortRange 3389 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 100
$sshRule = New-AzNetworkSecurityRuleConfig -Name SAPERPDemoNSGSSH -Protocol * -SourcePortRange * -DestinationPortRange 22 -Access Allow -Direction Inbound -SourceAddressPrefix * -DestinationAddressPrefix * -Priority 101
$nsg = New-AzNetworkSecurityGroup -Name SAPERPDemoNSG -ResourceGroupName $rgName -Location  "North Europe" -SecurityRules $rdpRule,$sshRule

$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name Subnet1 -AddressPrefix  10.0.1.0/24 -NetworkSecurityGroup $nsg
$vnet = New-AzVirtualNetwork -Name SAPERPDemoVNet -ResourceGroupName $rgName -Location "North Europe"  -AddressPrefix 10.0.1.0/24 -Subnet $subnetConfig
```

* Création d’une adresse IP publique pouvant être utilisée pour accéder à la machine virtuelle à partir d’Internet

```powershell
# Create a public IP address with a DNS name
$pip = New-AzPublicIpAddress -Name SAPERPDemoPIP -ResourceGroupName $rgName -Location "North Europe" -DomainNameLabel $rgName.ToLower() -AllocationMethod Dynamic
```

* Création d’une interface réseau pour la machine virtuelle

```powershell
# Create a new Network Interface
$nic = New-AzNetworkInterface -Name SAPERPDemoNIC -ResourceGroupName $rgName -Location "North Europe" -Subnet $vnet.Subnets[0] -PublicIpAddress $pip
```

* Création d’une machine virtuelle Pour ce scénario, chaque machine virtuelle a le même nom. Le SID SAP des instances SAP NetWeaver dans ces machines virtuelles sera également le même. Dans un groupe de ressources Azure, le nom de la machine virtuelle doit être unique, mais dans des groupes de ressources Azure différents, vous pouvez exécuter des machines virtuelles portant le même nom. Le compte par défaut Administrateur pour Windows ou « Root » pour Linux ne sont pas valides. Par conséquent, un nouveau nom d’utilisateur administrateur ainsi qu’un nouveau mot de passe doivent être définis. La taille de la machine virtuelle doit également être définie.

```powershell
#####
# Create a new virtual machine with an official image from the Azure Marketplace
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

# select image
$vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer" -Skus "2012-R2-Datacenter" -Version "latest"
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "SUSE" -Offer "SLES-SAP" -Skus "12-SP1" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "RedHat" -Offer "RHEL" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMSourceImage -VM $vmconfig -PublisherName "Oracle" -Offer "Oracle-Linux" -Skus "7.2" -Version "latest"
# $vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$diskName="osfromimage"
$osDiskUri=$account.PrimaryEndpoints.Blob.ToString() + "vhds/" + $diskName  + ".vhd"

$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Windows
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOSDisk -VM $vmconfig -Name $diskName -VhdUri $osDiskUri -CreateOption fromImage -SourceImageUri <path to VHD that contains the OS image> -Linux
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

```powershell
#####
# Create a new virtual machine with a Managed Disk Image
#####
$cred=Get-Credential -Message "Type the name and password of the local administrator account."
$vmconfig = New-AzVMConfig -VMName SAPERPDemo -VMSize Standard_D11

$vmconfig = Add-AzVMNetworkInterface -VM $vmconfig -Id $nic.Id

$vmconfig = Set-AzVMSourceImage -VM $vmconfig -Id <Id of Managed Disk Image>
$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Windows -ComputerName "SAPERPDemo" -Credential $cred
#$vmconfig = Set-AzVMOperatingSystem -VM $vmconfig -Linux -ComputerName "SAPERPDemo" -Credential $cred

$vmconfig = Set-AzVMBootDiagnostics -Disable -VM $vmconfig
$vm = New-AzVM -ResourceGroupName $rgName -Location "North Europe" -VM $vmconfig
```

* Si vous le souhaitez, ajoutez des disques supplémentaires et restaurez le contenu nécessaire. Tous les noms d’objets blob (les URL vers les objets blob) doivent être uniques dans Azure.

```powershell
# Optional: Attach additional VHD data disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
$dataDiskUri = $account.PrimaryEndpoints.Blob.ToString() + "vhds/datadisk.vhd"
Add-AzVMDataDisk -VM $vm -Name datadisk -VhdUri $dataDiskUri -DiskSizeInGB 1023 -CreateOption empty | Update-AzVM

# Optional: Attach additional Managed Disks
$vm = Get-AzVM -ResourceGroupName $rgName -Name SAPERPDemo
Add-AzVMDataDisk -VM $vm -Name datadisk -DiskSizeInGB 1023 -CreateOption empty -Lun 0 | Update-AzVM
```

##### <a name="cli"></a>Interface de ligne de commande

L’exemple de code suivant peut être utilisé sur Linux. Pour Windows, utilisez PowerShell (comme décrit ci-dessus) ou adaptez l’exemple de façon à utiliser %rgName% au lieu de $rgName, et définissez la variable d’environnement à l’aide de la commande Windows *set*.

* Création d’un groupe de ressources pour chaque paysage de formation et de démonstration

```azurecli
rgName=SAPERPDemo1
rgNameLower=saperpdemo1
az group create --name $rgName --location "North Europe"
```

* Création d’un nouveau compte de stockage

```azurecli
az storage account create --resource-group $rgName --location "North Europe" --kind Storage --sku Standard_LRS --name $rgNameLower
```

* Création d’un réseau virtuel pour chaque paysage de formation et de démonstration pour rendre possible l’utilisation du même nom d’hôte et des mêmes adresses IP. Le réseau virtuel est protégé par un groupe de sécurité réseau qui autorise uniquement le trafic vers le port 3389 afin de rendre possible l’accès au Bureau à distance et activer le port 22 pour SSH.

```azurecli
az network nsg create --resource-group $rgName --location "North Europe" --name SAPERPDemoNSG
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGRDP --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 3389 --access Allow --priority 100 --direction Inbound
az network nsg rule create --resource-group $rgName --nsg-name SAPERPDemoNSG --name SAPERPDemoNSGSSH --protocol \* --source-address-prefix \* --source-port-range \* --destination-address-prefix \* --destination-port-range 22 --access Allow --priority 101 --direction Inbound

az network vnet create --resource-group $rgName --name SAPERPDemoVNet --location "North Europe" --address-prefixes 10.0.1.0/24
az network vnet subnet create --resource-group $rgName --vnet-name SAPERPDemoVNet --name Subnet1 --address-prefix 10.0.1.0/24 --network-security-group SAPERPDemoNSG
```

* Création d’une adresse IP publique pouvant être utilisée pour accéder à la machine virtuelle à partir d’Internet

```azurecli
az network public-ip create --resource-group $rgName --name SAPERPDemoPIP --location "North Europe" --dns-name $rgNameLower --allocation-method Dynamic
```

* Création d’une interface réseau pour la machine virtuelle

```azurecli
az network nic create --resource-group $rgName --location "North Europe" --name SAPERPDemoNIC --public-ip-address SAPERPDemoPIP --subnet Subnet1 --vnet-name SAPERPDemoVNet
```

* Création d’une machine virtuelle Pour ce scénario, chaque machine virtuelle a le même nom. Le SID SAP des instances SAP NetWeaver dans ces machines virtuelles sera également le même. Dans un groupe de ressources Azure, le nom de la machine virtuelle doit être unique, mais dans des groupes de ressources Azure différents, vous pouvez exécuter des machines virtuelles portant le même nom. Le compte par défaut Administrateur pour Windows ou « Root » pour Linux ne sont pas valides. Par conséquent, un nouveau nom d’utilisateur administrateur ainsi qu’un nouveau mot de passe doivent être définis. La taille de la machine virtuelle doit également être définie.

```azurecli
#####
# Create virtual machines using storage accounts
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --authentication-type password

#####
# Create virtual machines using Managed Disks
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image SUSE:SLES-SAP:12-SP1:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image RedHat:RHEL:7.2:latest --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --image "Oracle:Oracle-Linux:7.2:latest" --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --authentication-type password
```

```azurecli
#####
# Create a new virtual machine with a VHD that contains the private image that you want to use
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Windows --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --os-type Linux --admin-username <username> --admin-password <password> --size Standard_D11 --use-unmanaged-disk --storage-account $rgNameLower --storage-container-name vhds --os-disk-name os --image <path to image vhd> --authentication-type password

#####
# Create a new virtual machine with a Managed Disk Image
#####
az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id>
#az vm create --resource-group $rgName --location "North Europe" --name SAPERPDemo --nics SAPERPDemoNIC --admin-username <username> --admin-password <password> --size Standard_DS11_v2 --os-disk-name os --image <managed disk image id> --authentication-type password
```

* Si vous le souhaitez, ajoutez des disques supplémentaires et restaurez le contenu nécessaire. Tous les noms d’objets blob (les URL vers les objets blob) doivent être uniques dans Azure.

```azurecli
# Optional: Attach additional VHD data disks
az vm unmanaged-disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --vhd-uri https://$rgNameLower.blob.core.windows.net/vhds/data.vhd  --new

# Optional: Attach additional Managed Disks
az vm disk attach --resource-group $rgName --vm-name SAPERPDemo --size-gb 1023 --disk datadisk --new
```

##### <a name="template"></a>Modèle

Vous pouvez utiliser les exemples de modèles du référentiel Azure-quickstart-templates sur GitHub.

* [Machine virtuelle Linux simple](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux)
* [Machine virtuelle Windows simple](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-windows)
* [Machine virtuelle à partir d’une image](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image)

### <a name="implement-a-set-of-vms-that-communicate-within-azure"></a>Implémenter un ensemble de machines virtuelles qui communiquent au sein d’Azure

Ce scénario non hybride est le scénario classique de formation et de démonstration dans lequel le logiciel représentant la démonstration/formation est réparti sur plusieurs machines virtuelles. Les différents composants installés dans les différentes machines virtuelles doivent communiquer entre eux. Dans ce scénario, aucune communication réseau locale ou scénario entre sites locaux n’est nécessaire.

Ce scénario est une extension de l’installation que décrit le chapitre [Machine virtuelle unique avec scénario de démonstration/formation SAP NetWeaver][planning-guide-7.1] dans ce document. Dans ce cas, des machines virtuelles supplémentaires seront ajoutées à un groupe de ressources existant. Dans l’exemple suivant, le paysage d’entraînement se compose d’une machine virtuelle SAP ASCS/SCS, d’une machine virtuelle exécutant un système SGBD et d’une machine virtuelle d’instance de serveur d’applications SAP.

Avant de créer ce scénario, vous devez définir les paramètres de base comme dans le scénario précédent.

#### <a name="resource-group-and-virtual-machine-naming"></a>Dénomination des groupes de ressources et des machines virtuelles

Tous les noms de groupe de ressources doivent être uniques. Développez votre propre schéma de dénomination de vos ressources, tel que `<rg-name`>-suffixe.

Le nom de la machine virtuelle doit être unique dans le groupe de ressources.

#### <a name="set-up-network-for-communication-between-the-different-vms"></a>Configurer un réseau pour communiquer entre les différentes machines virtuelles

![Ensemble de machines virtuelles au sein d’un réseau virtuel Azure][planning-guide-figure-1900]

Pour éviter des conflits de dénomination avec des clones du même paysage de formation/démonstration, vous devez créer un réseau virtuel Azure pour chaque paysage. La résolution de noms DNS est assurée par Azure, ou vous pouvez configurer votre propre serveur DNS en dehors d’Azure (ne sera pas abordé plus en détail ici). Dans ce scénario, nous ne configurons pas notre propre DNS. La communication via les noms d’hôte sera activée pour toutes les machines virtuelles du réseau virtuel Azure.

Les raisons de la répartition des paysages de formation ou de démonstration en réseaux virtuels et pas seulement en groupes de ressources pourraient être les suivantes :

* Le paysage SAP tel que configuré a besoin de son propre service AD/OpenLDAP et chacun des paysages doit intégrer un serveur de domaine.
* Le paysage SAP tel que configuré possède des composants qui doivent fonctionner avec des adresses IP fixes.

Pour plus de détails sur les réseaux virtuels Azure et leur définition, consultez [cet article][virtual-networks-create-vnet-arm-pportal].

## <a name="deploying-sap-vms-with-corporate-network-connectivity-cross-premises"></a>Déploiement de machines virtuelles SAP avec une connectivité réseau d’entreprise (entre différents locaux)

Vous exécutez un paysage SAP et voulez répartir le déploiement entre un système nu pour les serveurs haut de gamme SGBD, des environnements virtualisés locaux pour les couches Application et des systèmes SAP plus petits configurés sur 2 niveaux et Azure IaaS. Le principe de base est que les systèmes SAP au sein d’un paysage SAP unique doivent communiquer entre eux et avec de nombreux autres composants logiciels déployés dans l’entreprise, indépendamment de leur forme de déploiement. Par ailleurs, aucune différence ne doit être introduite par la forme de déploiement pour la connexion de l’utilisateur final à l’interface utilisateur graphique SAP ou à d’autres interfaces. Ces conditions ne peuvent être satisfaites que si des services Active Directory/OpenLDAP et DNS sont étendus aux systèmes Azure via la connectivité de site à site/multisites ou des connexions privées comme Azure ExpressRoute.



### <a name="scenario-of-an-sap-landscape"></a>Scénario d’un paysage SAP

Le graphique ci-dessous permet de décrire approximativement le scénario intersite ou hybride :

![Connectivité de site à site entre des ressources locales et Azure][planning-guide-figure-2100]

La configuration minimale requise est l’utilisation de protocoles de communication sécurisés tels que SSL/TLS pour l’accès au navigateur ou de connexions VPN pour l’accès du système aux services Azure. Nous supposons que les entreprises gèrent la connexion VPN entre leur réseau d’entreprise et Azure de manière différente. Il se peut que certaines sociétés ouvrent d’emblée tous les ports. D’autres peuvent vouloir spécifier exactement les ports qui doivent être ouverts, etc.

Les ports de communication SAP classiques sont répertoriés dans le tableau ci-dessous. À la base, l’ouverture du port de la passerelle SAP suffit.

<!-- sapms is prefix of a SAP service name and not a spelling error -->

| Service | Nom du port | Exemple `<nn`> = 01 | Plage par défaut (min-max.) | Commentaire |
| --- | --- | --- | --- | --- |
| Répartiteur |sapdp`<nn>` voir * |3201 |3200 - 3299 |Répartiteur SAP, utilisé par l’interface utilisateur graphique SAP pour Windows et Java |
| Serveur de messagerie |sapms`<sid`> voir ** |3600 |sapms gratuit`<anySID`> |sid = SAP-System-ID |
| Passerelle |sapgw`<nn`> voir * |3301 |gratuit |Passerelle SAP, utilisée pour les communications CPIC et RFC |
| Routeur SAP |sapdp99 |3299 |gratuit |Seuls les noms de service de l’instance centrale peuvent être réaffectés dans /etc/services à une valeur arbitraire après l’installation. |

*) nn = Numéro d’instance SAP

\**) sid = SAP-System-ID

Pour plus d’informations sur les ports nécessaires pour les différents produits ou services SAP, consultez <https://scn.sap.com/docs/DOC-17124>.
Ce document vous permet d’ouvrir des ports dédiés sur le périphérique VPN nécessaire pour des scénarios et produits SAP spécifiques.

Lors du déploiement de machines virtuelles dans un tel scénario, les autres mesures de sécurité peuvent consister à créer un [groupe de sécurité réseau][virtual-networks-nsg] pour définir des règles d’accès.



#### <a name="printing-on-a-local-network-printer-from-sap-instance-in-azure"></a>Imprimer sur une imprimante réseau local à partir d’une instance SAP dans Azure

##### <a name="printing-over-tcpip-in-cross-premises-scenario"></a>Impression via TCP/IP dans un scénario intersite

La configuration de vos imprimantes réseau locales TCP/IP sur une machine virtuelle Azure est globalement identique à une configuration de même type dans votre réseau d’entreprise, en supposant qu’une connexion ExpressRoute ou « Tunnel » de site à site soit établie.

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Pour ce faire :
>
> * Certaines imprimantes réseau incluent un assistant de configuration qui facilite leur configuration dans une machine virtuelle Azure. Si aucun assistant n’est fourni avec l’imprimante, la méthode manuelle de configuration de l’imprimante consiste à créer un nouveau port d’imprimante TCP/IP.
> * Ouvrez Panneau de configuration -> Périphériques et imprimantes -> Ajouter une imprimante.
> * Cliquez sur Ajouter une imprimante à l’aide d’une adresse TCP/IP ou d’un nom d’hôte
> * Tapez l’adresse IP de l’imprimante
> * Le port standard de l’imprimante est 9100
> * Si besoin, installez manuellement le pilote d’imprimante qui convient.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> * Comme pour Windows, il suffit de suivre la procédure standard d’installation d’une imprimante réseau.
> * Suivez simplement les guides Linux publics pour [SUSE](https://www.suse.com/documentation/sles-12/book_sle_deployment/data/sec_y2_hw_print.html) ou [Red Hat et Oracle Linux](https://access.redhat.com/documentation/red_hat_enterprise_linux/6/html/deployment_guide/sec-printer_configuration) sur l’ajout d’une imprimante.
>
>

---
![Impression en réseau][planning-guide-figure-2200]

##### <a name="host-based-printer-over-smb-shared-printer-in-cross-premises-scenario"></a>Imprimante basée sur l’hôte via SMB (imprimante partagée) dans un scénario intersite

Les imprimantes basées sur l’hôte ne sont pas compatibles réseau par défaut. Toutefois, une imprimante basée sur l’hôte peut être partagée entre plusieurs ordinateurs sur un réseau tant que l’imprimante est connectée à un ordinateur sous tension. Connectez votre réseau via une connexion de site à site ou ExpressRoute et partagez votre imprimante locale. Le protocole SMB utilise NetBIOS au lieu de DNS comme service de noms. Le nom d’hôte NetBIOS peut être différent du nom d’hôte DNS. Dans une situation standard, les noms d’hôtes NetBIOS et DNS sont identiques. Le domaine DNS est inutile dans l’espace de noms NetBIOS. En conséquence, le nom d’hôte DNS complet comprenant le nom d’hôte DNS et le domaine DNS ne doit pas être utilisé dans l’espace de noms NetBIOS.

Le partage d’imprimante est identifié par le biais d’un nom unique dans le réseau :

* Nom de l’hôte SMB (toujours obligatoire)
* Nom du partage (toujours obligatoire)
* Nom du domaine si le partage d’imprimante n’est pas dans le même domaine que le système SAP.
* En outre, un nom d’utilisateur et un mot de passe peuvent être requis pour accéder au partage d’imprimante.

Procédure :

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Partagez votre imprimante locale.
> Dans la machine virtuelle Azure, ouvrez l’Explorateur Windows et tapez le nom du partage d’imprimante.
> Un assistant installation d’imprimante vous guidera tout au long du processus d’installation.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Voici quelques exemples de documents relatifs à la configuration des imprimantes réseau sous Linux ou comprenant un chapitre concernant l’impression sous Linux. Le fonctionnement dans une machine virtuelle Azure Linux est identique tant que la machine virtuelle fait partie d’un réseau VPN :
>
> * SLES <https://en.opensuse.org/SDB:Printing_via_SMB_(Samba)_Share_or_Windows_Share>
> * RHEL ou Oracle Linux <https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/system_administrators_guide/index#sec-Starting_Print_Settings_Config>
>
>

---
##### <a name="usb-printer-printer-forwarding"></a>Imprimante USB (réacheminement d’imprimante)

La capacité des Services Bureau à distance à fournir aux utilisateurs l’accès à leurs périphériques d’imprimante locale dans une session distante n’est pas disponible dans Azure.

---
> ![Logo Windows.][Logo_Windows] Windows
>
> Pour plus d’informations sur l’impression avec Windows, consultez : <https://technet.microsoft.com/library/jj590748.aspx>.
>
>

---
#### <a name="integration-of-sap-azure-systems-into-correction-and-transport-system-tms-in-cross-premises"></a>Intégration de systèmes Azure SAP au système de transport et correction SAP (TMS) en local

Le système de transport et correction SAP (TMS) doit être configuré pour exporter et importer des demandes de transport entre les systèmes du paysage. Nous partons du principe que les instances de développement d’un système SAP (DEV) se trouvent dans Azure, tandis que l’assurance qualité (QA) et les systèmes de production (PRD) se trouvent en local. En outre, nous supposons qu’il existe un répertoire de transport central.

##### <a name="configuring-the-transport-domain"></a>Configurer le domaine de Transport

Configurez votre domaine de transport sur le système que vous avez désigné en tant que contrôleur de domaine de transport, comme décrit dans [Configuring the Transport Domain Controller](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm)(Configuration du contrôleur de domaine de transport). Un utilisateur système TMSADM sera créé et la destination RFC nécessaire sera générée. Vous pouvez vérifier ces connexions RFC à l’aide de la transaction SM59. La résolution de nom d’hôte doit être activée dans votre domaine de transport.

Procédure :

* Dans notre scénario, nous avons décidé que le système QAS local est le contrôleur de domaine CTS. Appelez le STMS de transaction. La boîte de dialogue du TMS s’affiche. Une boîte de dialogue de configuration du domaine de transport s’affiche. (cette boîte de dialogue apparaît uniquement si vous n’avez pas encore configuré de domaine de transport).
* Assurez-vous que l’utilisateur automatiquement créé TMSADM est autorisé (SM59 -> ABAP Connection -> Details -> Utilities(M) -> Authorization Test) (SM59 -> Connexion ABAP -> TMSADM@E61.DOMAIN_E61 -> Détails -> Utilitaires(M) -> Test d’autorisation. L’écran initial du STMS de transaction doit indiquer que ce système SAP fonctionne désormais comme contrôleur du domaine de transport comme indiqué ici :

![Écran initial du STMS de transaction sur le contrôleur de domaine][planning-guide-figure-2300]

#### <a name="including-sap-systems-in-the-transport-domain"></a>Inclure des systèmes SAP dans le domaine de transport

La séquence d’intégration d’un système SAP dans un domaine de transport se présente comme suit :

* Sur le système de développement dans Azure, accédez au système de transport (Client 000) et appelez le STMS de transaction. Choisissez Other Configuration (Autre configuration) dans la boîte de dialogue et passez à Include System in Domain (Inclure le système dans le domaine). Définissez le contrôleur de domaine en tant qu’hôte cible ([Inclure des systèmes SAP dans le domaine de transport](https://help.sap.com/erp2005_ehp_04/helpdata/en/44/b4a0c17acc11d1899e0000e829fbbd/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)). Le système attend maintenant d’être inclus dans le domaine de transport.
* Pour des raisons de sécurité, vous devez ensuite revenir au contrôleur de domaine pour confirmer votre demande. Choisissez System Overview (Vue d’ensemble du système) et Approve of the waiting system (Approbation du système en attente). Puis, confirmez l’invite pour que la configuration soit distribuée.

Ce système SAP contient à présent les informations nécessaires sur tous les autres systèmes SAP du domaine de transport. Dans le même temps, les données d’adresse du nouveau système SAP sont envoyées à tous les autres systèmes SAP, et le système SAP est entré dans le profil de transport du programme de contrôle de transport. Vérifiez si les RFC et l’accès au répertoire de transport du domaine fonctionnent.

Poursuivez par la configuration de votre système de transport comme vous en avez l’habitude, en procédant de la manière décrite dans la documentation [Change and Transport System](https://help.sap.com/saphelp_nw70ehp3/helpdata/en/48/c4300fca5d581ce10000000a42189c/content.htm?frameset=/en/44/b4a0b47acc11d1899e0000e829fbbd/frameset.htm)(Correction et système de Transport).

Procédure :

* Assurez-vous que votre STMS en local est configuré correctement.
* Assurez-vous que le nom d’hôte du contrôleur de domaine de Transport peut être résolu par votre machine virtuelle sur Azure et vice versa.
* Appelez le STMS de transaction -> Other Configuration -> Include System in Domain (-> Autre configuration -> Inclure le système dans le domaine).
* Confirmez la connexion dans le système TMS local.
* Configurez les itinéraires de transport, les groupes et les couches comme d’habitude.

Dans les scénarios de connexion de site à site, la latence entre les systèmes locaux et Azure peut néanmoins être importante. Si nous suivons la séquence de transport d’objets des systèmes de développement et de test à la production ou si nous envisageons d’appliquer des packages de transport ou de prise en charge aux différents systèmes, vous vous rendez compte qu’en fonction de l’emplacement du répertoire de transport central, certains systèmes subissent une latence élevée en termes de lecture et d’écriture de données dans ce dernier. La situation est semblable aux configurations de paysage SAP où les différents systèmes sont répartis sur différents centres de données très éloignés les uns des autres.

Pour pallier cette latence et permettre aux systèmes d’accélérer la lecture ou l’écriture vers ou depuis le répertoire de transport, vous pouvez configurer deux domaines de transport STMS (un pour les systèmes locaux et un pour les systèmes dans Azure) et lier les domaines de transport. Consultez la documentation qui explique les principes de ce concept dans SAP TMS : <https://help.sap.com/saphelp_me60/helpdata/en/c4/6045377b52253de10000009b38f889/content.htm?frameset=/en/57/38dd924eb711d182bf0000e829fbfe/frameset.htm>.

Procédure :

* Configurer un domaine de transport sur chaque emplacement (local et Azure) à l’aide du STMS de transaction <https://help.sap.com/saphelp_nw70ehp3/helpdata/en/44/b4a0b47acc11d1899e0000e829fbbd/content.htm>
* Associez les domaines avec un lien de domaine et confirmez le lien entre les deux domaines.
  <https://help.sap.com/saphelp_nw73ehp1/helpdata/en/a3/139838280c4f18e10000009b38f8cf/content.htm>
* Distribuez la configuration au système lié.

#### <a name="rfc-traffic-between-sap-instances-located-in-azure-and-on-premises-cross-premises"></a>Trafic RFC entre des instances SAP situées dans Azure et en local (intersite)

Le trafic RFC entre les systèmes locaux et dans Azure doit fonctionner. Pour configurer une connexion, appelez la transaction SM59 dans un système source où vous devez définir une connexion RFC vers le système cible. La configuration est similaire à la configuration standard d’une connexion RFC.

Nous supposons que dans le scénario intersite, les machines virtuelles exécutant des systèmes SAP qui doivent communiquer entre eux se trouvent dans le même domaine. Par conséquent, la configuration d’une connexion RFC entre les systèmes SAP ne diffère pas des étapes d’installation et des entrées des scénarios locaux.

#### <a name="accessing-local-fileshares-from-sap-instances-located-in-azure-or-vice-versa"></a>Accéder à des partages de fichiers locaux à partir d’instances SAP situées dans Azure ou inversement

Les instances SAP dans Azure doivent accéder aux partages de fichiers qui se trouvent localement dans l’entreprise. Par ailleurs, les instances SAP locales doivent accéder aux partages de fichiers qui se trouvent dans Azure. Pour activer les partages de fichiers, vous devez configurer les options d’autorisation et de partage sur le système local. Veillez à ouvrir les ports sur la connexion VPN ou ExpressRoute entre Azure et votre centre de données.

## <a name="supportability"></a>Prise en charge

### <a name="azure-extension-for-sap"></a><a name="6f0a47f3-a289-4090-a053-2521618a28c3"></a>Extension Azure pour SAP

Pour fournir aux instances de SAP Host Agent installées dans des machines virtuelles une partie des informations relatives à l’infrastructure Azure des systèmes SAP critiques, une extension de machine virtuelle Azure pour SAP doit être installée pour les machines virtuelles déployées. Comme les exigences de SAP étaient liées aux applications SAP, Microsoft a décidé de ne pas implémenter de manière générique la fonctionnalité dans Azure, mais de laisser le choix aux clients de déployer l’extension de machine virtuelle et les configurations nécessaires sur leurs machines virtuelles s’exécutant dans Azure. Toutefois, la gestion du cycle de vie et du déploiement de l’extension de machine virtuelle Azure pour SAP est principalement automatisée par Azure.

#### <a name="solution-design"></a>Conception de la solution

La solution développée pour permettre à SAP Host Agent d’obtenir les informations nécessaires repose sur l’architecture du framework de l’agent et des extensions de machine virtuelle Azure. L’idée de l’infrastructure de l’agent et des extensions de machine virtuelle Azure consiste à autoriser l’installation d’applications logicielles disponibles dans la galerie d’extensions de machines virtuelles Azure dans une machine virtuelle. Le principe à l’origine de ce concept est de permettre, dans les cas comme celui de l’extension Azure pour SAP, le déploiement de fonctionnalités spéciales sur une machine virtuelle et la configuration de ces logiciels au moment du déploiement.

L’« agent de machine virtuelle Azure » qui permet une gestion des extensions de machines virtuelles Azure spécifiques au sein de la machine virtuelle est intégré aux machines virtuelles Windows par défaut pendant la création de machines virtuelles dans le portail Azure. Dans le cas de SUSE, Red Hat ou Oracle Linux, l’agent de machine virtuelle fait déjà partie de l’image de la Place de marché Azure. Dans le cas du chargement d’une machine virtuelle Linux d’un système local vers Azure, l’agent de machine virtuelle doit être installé manuellement.

La solution permettant de fournir des informations sur l’infrastructure Azure à SAP Host Agent dans Azure repose sur les modules de base suivants :

![Composants d’extension Microsoft Azure][planning-guide-figure-2400]

Comme indiqué dans le diagramme de blocs ci-dessus, une partie de la solution est hébergée dans l’image de machine virtuelle Azure et la galerie d’extensions Azure, qui est un référentiel à réplication globale géré par des opérations Azure. L’équipe SAP/MS travaillant à l’implémentation Azure de SAP est chargée d’utiliser les opérations Azure pour publier de nouvelles versions de l’extension Azure pour SAP.

Lorsque vous déployez une nouvelle machine virtuelle Windows, l’agent de machine virtuelle Azure est automatiquement ajouté à la machine virtuelle. La fonction de cet agent consiste à coordonner le chargement et la configuration des extensions Azure des machines virtuelles. Pour les machines virtuelles Linux, l’agent de machine virtuelle Azure fait déjà partie de l’image du système d’exploitation de la Place de marché Azure.

Toutefois, il reste une étape à exécuter par le client. Il s’agit de l’activation et de la configuration de la collecte des performances. Le processus relatif à la configuration est automatisé par un script PowerShell ou une commande d’interface de ligne de commande. Le script PowerShell peut être téléchargé à partir du Centre de scripts Microsoft Azure, comme décrit dans le [Guide de déploiement][deployment-guide].

L’architecture globale de l’extension Azure pour SAP ressemble à ceci :

![Extension Azure pour SAP ][planning-guide-figure-2500]

**Pour connaître la procédure exacte et obtenir les étapes détaillées concernant l’utilisation de ces cmdlets PowerShell ou la commande CLI au cours des déploiements, suivez les instructions fournies dans le [Guide de déploiement][deployment-guide].**

### <a name="integration-of-azure-located-sap-instance-into-saprouter"></a>Intégration d’instances SAP situées dans Azure dans SAProuter

Les instances SAP s’exécutant dans Azure doivent également être accessibles à partir de SAProuter.

![Connexion réseau du routeur SAP][planning-guide-figure-2600]

Un programme SAProuter permet la communication TCP/IP entre systèmes participants en l’absence de connexion IP directe. Cela présente l’avantage de ne pas devoir recourir à une connexion de bout en bout entre les partenaires de communication au niveau du réseau. Le programme SAProuter est à l’écoute du port 3299 par défaut.
Pour connecter des instances SAP via un programme SAProuter, vous devez fournir la chaîne et le nom d’hôte SAProuter lors de toute tentative de connexion.

## <a name="sap-netweaver-as-java"></a>SAP NetWeaver AS Java

Jusqu’à présent, ce document a porté sur SAP NetWeaver en général ou sur la pile SAP NetWeaver ABAP. Cette brève section répertorie les considérations spécifiques à la pile SAP Java. L’une des applications les plus importantes exclusivement basées sur SAP NetWeaver Java est le portail d’entreprise SAP. Les autres applications, telles que SAP PI et SAP Solution Manager utilisent toutes deux les piles SAP NetWeaver ABAP et Java. Par conséquent, il est certainement nécessaire de prendre en compte également les aspects spécifiques relatifs à la pile SAP NetWeaver Java.

### <a name="sap-enterprise-portal"></a>Portail d’entreprise SAP

La configuration d’un portail SAP dans une machine virtuelle Azure ne diffère pas d’une installation locale si vous effectuez un déploiement dans des scénarios intersite. Étant donné que le DNS s’effectue en local, les paramètres de port des différentes instances peuvent être considérés comme étant configurés localement. Les recommandations et restrictions décrites jusqu’ici dans ce document s’appliquent en général à une application comme le portail d’entreprise SAP ou la pile SAP NetWeaver Java.

![Portail SAP exposé][planning-guide-figure-2700]

Un scénario de déploiement spécifique par certains clients est l’exposition directe du portail d’entreprise SAP à Internet tandis que l’hôte de machine virtuelle est connecté au réseau d’entreprise via une connexion Tunnel VPN de site à site ou ExpressRoute. Pour ce scénario, vous devez vous assurer que certains ports sont ouverts et ne sont pas bloqués par un pare-feu ou un groupe de sécurité réseau.

L’URI initiale du portail est http(s):`<Portalserver`>:5XX00/irj où le port est formé comme documenté par SAP dans <https://help.sap.com/saphelp_nw70ehp1/helpdata/de/a2/f9d7fed2adc340ab462ae159d19509/frameset.htm>.

![Configuration du point de terminaison][planning-guide-figure-2800]

Si vous voulez personnaliser l’URL et/ou les ports de votre portail d’entreprise SAP, consultez cette documentation :

* [Change Portal URL (Modifier l’URL du portail)](https://wiki.scn.sap.com/wiki/display/EP/Change+Portal+URL)
* [Change Default port numbers, Portal port numbers (Modifier les numéros de port par défaut et les numéros de ports du portail)](https://wiki.scn.sap.com/wiki/display/NWTech/Change+Default++port+numbers%2C+Portal+port+numbers)

## <a name="high-availability-ha-and-disaster-recovery-dr-for-sap-netweaver-running-on-azure-virtual-machines"></a>Haute disponibilité (HA) et récupération d’urgence (DR)pour SAP NetWeaver s’exécutant sur des machines virtuelles Azure

### <a name="definition-of-terminologies"></a>Définition des termes

Le terme **haute disponibilité** est généralement associé à un ensemble de technologies qui réduisent les interruptions des services informatiques en assurant leur continuité par le biais de composants redondants, tolérants aux pannes ou protégés contre le basculement, dans le **même** centre de données. Dans notre cas, au sein d’une région Azure.

La **récupération d’urgence (DR)** vise également à réduire l’interruption des services informatiques et leur récupération, mais entre **différents** centres de données, généralement éloignés de plusieurs centaines de kilomètres les uns des autres. Dans notre cas, généralement entre différentes régions Azure de la même région géopolitique ou tel que défini par vous en tant que client.

### <a name="overview-of-high-availability"></a>Vue d’ensemble de la haute disponibilité

Nous pouvons séparer la discussion concernant la haute disponibilité SAP dans Azure en deux parties :

* La **haute disponibilité de l’infrastructure Azure**, par exemple, celle du calcul (machines virtuelles), du réseau, du stockage, etc., et ses avantages en termes d’augmentation de la disponibilité des applications SAP.
* La **haute disponibilité des applications SAP**, par exemple, celle des composants logiciels SAP :
  * Serveurs d’application SAP
  * Instance SAP ASCS/SCS
  * Serveur de base de données

et comment il peut être combiné avec la haute disponibilité de l’infrastructure Azure.

La haute disponibilité SAP dans Azure présente des différences par rapport à la haute disponibilité SAP dans un environnement physique local ou virtuel. Le document de SAP suivant décrit les configurations de haute disponibilité SAP standard dans des environnements virtualisés sous Windows : <https://scn.sap.com/docs/DOC-44415>. Il n’existe aucune configuration haute disponibilité SAP intégrée pour Linux comparable à celle de Windows. Des informations concernant la haute disponibilité SAP en local pour Linux sont disponibles ici : <https://scn.sap.com/docs/DOC-8541>.

### <a name="azure-infrastructure-high-availability"></a>haute disponibilité de l’infrastructure Azure

Il existe un contrat de niveau de service pour machine virtuelle unique de 99,9 %. Pour avoir une idée de ce à quoi peut ressembler la disponibilité d’une machine virtuelle, vous pouvez calculer le produit des différents contrats SLA Azure disponibles : <https://azure.microsoft.com/support/legal/sla/>.

La base de calcul est de 30 jours par mois ou 43 200 minutes. Par conséquent, le temps d’interruption de 0,05 % correspond à 21,6 minutes. Comme d’habitude, la disponibilité des différents services se multiplie de la façon suivante :

(Service de disponibilité #1/100) * (Service de disponibilité #2/100) * (Service de disponibilité #3/100)

comme ce qui suit :

(99,95/100) * (99,9/100) * (99,9/100) = 0,9975, soit une disponibilité globale de 99,75 %.

#### <a name="virtual-machine-vm-high-availability"></a>Haute disponibilité de la machine virtuelle

Deux types d'événements de plateforme Azure peuvent avoir un effet sur la disponibilité de vos machines virtuelles : la maintenance planifiée et la maintenance non planifiée.

* Les événements de maintenance planifiés sont des mises à jour périodiques effectuées par Microsoft sur la plateforme sous-jacente Azure pour améliorer la fiabilité, les performances et la sécurité de l'infrastructure hébergeant vos machines virtuelles.
* Les événements de maintenance non planifiés ont lieu lorsque l'infrastructure physique ou matérielle sous-jacente de votre machine virtuelle a connu une défaillance. Cela comprend les défaillances du réseau local, du disque local ou au niveau du rack. Lorsqu’une défaillance de ce type est détectée, la plateforme Azure migre automatiquement votre machine virtuelle du serveur physique défectueux vers un serveur physique sain. De tels événements sont rares, mais peuvent entraîner un redémarrage de votre machine virtuelle.

Pour plus d’informations, consultez [Disponibilité des machines virtuelles Windows dans Azure](../../manage-availability.md) et [Disponibilité des machines virtuelles Linux dans Azure](../../manage-availability.md).

#### <a name="azure-storage-redundancy"></a>Redondance d’Azure Storage

Les données de votre compte de stockage Microsoft Azure sont toujours répliquées pour garantir une durabilité et une haute disponibilité, conformément au contrat de niveau de service Stockage Azure, même en cas de défaillances matérielles temporaires.

Dans la mesure où Stockage Azure consiste à conserver trois images des données par défaut, RAID5 ou RAID1 sur plusieurs disques Azure n’est pas nécessaire.

Pour plus d’informations, consultez [Redondance de Stockage Azure](../../../storage/common/storage-redundancy.md).

#### <a name="utilizing-azure-infrastructure-vm-restart-to-achieve-higher-availability-of-sap-applications"></a>Utiliser le redémarrage de la machine virtuelle d’infrastructure Azure pour permettre une plus haute disponibilité des applications SAP

Si vous décidez de ne pas utiliser de fonctionnalités telles que le clustering de basculement Windows Server (WSFC) ou Pacemaker sur Linux (uniquement pris en charge pour SLES versions 12 et ultérieures), le redémarrage de la machine virtuelle Azure permet de protéger un système SAP contre les interruptions de services planifiées ou non de l’infrastructure du serveur physique Azure et de la plateforme Azure sous-jacente globale.

> [!NOTE]
> Il est important de mentionner que le redémarrage de la machine virtuelle Azure protège principalement les machines virtuelles et PAS les applications. Le redémarrage de la machine virtuelle n’offre pas de haute disponibilité pour les applications SAP, mais un certain niveau de disponibilité de l’infrastructure et, par conséquent, indirectement une « plus haute disponibilité » des systèmes SAP. Par ailleurs, il n’existe actuellement aucun contrat de niveau de service (SLA) concernant le temps nécessaire au redémarrage d’une machine virtuelle après une interruption d’hôte planifiée ou non. Par conséquent, cette méthode de haute disponibilité n’est pas adaptée aux composants essentiels d’un système SAP, tel qu’un (A)SCS ou un SGBD (système de gestion de base de données).
>
>

Le stockage est un autre élément important d’infrastructure pour la haute disponibilité. Par exemple, le contrat de niveau de service du Stockage Azure assure une disponibilité de 99,9 %. Si une personne déploie toutes les machines virtuelles et ses disques sur un compte Azure Storage unique, l’indisponibilité potentielle d’Azure Storage entraînera celle de toutes les machines virtuelles placées dans ce compte Azure Storage et également celle de tous les composants SAP s’exécutant sur ces dernières.

Au lieu de placer toutes les machines virtuelles dans un seul compte Azure Storage, vous pouvez également utiliser les comptes de stockage dédiés pour chaque machine virtuelle et ainsi augmenter la disponibilité globale des machines virtuelles et des applications SAP en utilisant plusieurs comptes Azure Storage indépendants.

Les disques managés Azure sont automatiquement placés dans le domaine d’erreur de la machine virtuelle à laquelle ils sont attachés. Si vous placez deux machines virtuelles dans un groupe à haute disponibilité et que vous utilisez la fonctionnalité Disques managés, la plateforme se charge également de distribuer ces derniers dans différents domaines d’erreur. Si vous envisagez d’utiliser un Stockage Premium, nous vous recommandons vivement de recourir à des disques managés également.

Voici un exemple de ce à quoi pourrait ressembler une architecture de système SAP NetWeaver utilisant la haute disponibilité d’infrastructure Azure et des comptes de stockage :

![Diagramme illustrant un système SAP NetWeaver qui utilise la haute disponibilité de l’infrastructure et les comptes de stockage Azure.][planning-guide-figure-2900]

Voici un exemple de ce à quoi pourrait ressembler une architecture de système SAP NetWeaver utilisant la haute disponibilité d’infrastructure Azure et la fonctionnalité Disques managés :

![Utiliser la haute disponibilité d’infrastructure Azure pour permettre une « plus haute » disponibilité de l’application SAP][planning-guide-figure-2901]

Pour les composants SAP stratégiques, nous avons obtenu jusqu’ici ce qui suit :

* La haute disponibilité des serveurs d’applications SAP

  Les instances de serveur d’applications SAP sont des composants redondants. Chaque instance de serveur d’applications SAP est déployée sur sa propre machine virtuelle s’exécutant dans un domaine de mise à niveau et d’erreur différent (voir les chapitres [Domaines d’erreur][planning-guide-3.2.1] et [Domaines de mise à niveau][planning-guide-3.2.2]). Cela est assuré par l’utilisation des groupes à haute disponibilité Azure (voir le chapitre [Groupes à haute disponibilité Azure][planning-guide-3.2.3]). L’indisponibilité planifiée ou non planifiée potentielle d’un domaine de mise à niveau ou d’erreur Azure entraîne l’indisponibilité d’un nombre limité de machines virtuelles avec leurs instances de serveurs d’applications SAP.

  Chaque instance de serveur d’applications SAP est placée dans son propre compte Azure Storage : l’indisponibilité potentielle d’un compte de stockage Azure provoquera l’indisponibilité d’une seule machine virtuelle et de son instance de serveur d’applications SAP. Toutefois, sachez qu’un abonnement Azure est limité quand au nombre de comptes Azure Storage. Pour assurer le démarrage automatique de l’instance (A)SCS après le redémarrage de la machine virtuelle, veillez à définir le paramètre de démarrage automatique dans le profil de démarrage de l’instance (A)SCS décrit dans le chapitre [Utilisation du démarrage automatique pour les instances SAP][planning-guide-11.5].
  Pour plus d’informations, consultez également le chapitre [Haute disponibilité pour les serveurs d’applications SAP][planning-guide-11.4.1].

  Même si vous utilisez des disques managés, ces disques sont également stockés dans un compte de stockage Azure et peuvent être indisponibles en cas d’une panne de stockage.

* *plus haute* disponibilité de l’instance (A)SCS SAP

  Ici, nous utilisons le redémarrage de la machine virtuelle Azure afin de protéger la machine virtuelle avec l’instance (A)SCS SAP installée. En cas d’interruption planifiée ou non de serveurs Azure, les machines virtuelles sont redémarrées sur un autre serveur disponible. Comme nous l’avons mentionné plus tôt, le redémarrage des machines virtuelles Azure protège les machines virtuelles et NON les applications, à savoir dans ce cas l’instance (A)SCS. Ce redémarrage de machine virtuelle permet d’obtenir indirectement une plus haute disponibilité de l’instance SAP (A)SCS. Pour assurer le démarrage automatique de l’instance (A)SCS après le redémarrage de la machine virtuelle, veillez à définir le paramètre de démarrage automatique du profil de démarrage de l’instance (A)SCS décrit dans le chapitre [Utilisation du démarrage automatique pour les instances SAP][planning-guide-11.5]. Cela signifie que l’instance (A)SCS en tant que point de défaillance unique s’exécutant sur une machine virtuelle unique sera un facteur déterminant de la disponibilité de l’ensemble du paysage SAP.

* *plus haute* disponibilité du serveur du SGBD (système de gestion de base de données)

  Ici, comme pour le cas d’utilisation de l’instance (A)SCS SAP, nous utilisons le redémarrage de la machine virtuelle pour protéger la machine virtuelle sur laquelle est installé le logiciel de SGBD (système de gestion de base de données). Ce redémarrage nous permet également d’obtenir une « plus haute disponibilité » du logiciel de SGBD (système de gestion de base de données).
  Le SGBD (système de gestion de base de données) s’exécutant sur une seule machine virtuelle est également un point de défaillance unique, qui est le facteur déterminant de disponibilité de l’ensemble du paysage SAP.

### <a name="sap-application-high-availability-on-azure-iaas"></a>Haute disponibilité de l’application SAP sur Azure IaaS

Pour obtenir une haute disponibilité de l’ensemble du système SAP, nous devons protéger tous les composants critiques du système SAP, par exemple, les serveurs d’applications SAP redondants et les composants uniques (par exemple, le point de défaillance unique) comme l’instance (A)SCS SAP et le SGBD (système de gestion de base de données).

#### <a name="high-availability-for-sap-application-servers"></a><a name="5d9d36f9-9058-435d-8367-5ad05f00de77"></a>Haute disponibilité pour les serveurs d’applications SAP

Pour les instances de boîte de dialogue/serveurs d’applications SAP, vous n’avez pas besoin d’une solution de haute disponibilité spécifique. La haute disponibilité est obtenue grâce à la redondance qui en assure un nombre suffisant dans les différentes machines virtuelles. Ces instances doivent toutes être placées dans le même groupe à haute disponibilité Azure pour éviter que les machines virtuelles soient mises à jour simultanément lors d’une interruption de maintenance planifiée. La fonctionnalité de base, qui repose sur des domaines de mise à niveau et d’erreur différents au sein d’une unité d’échelle Azure a déjà été présentée dans le chapitre [Domaines de mise à niveau][planning-guide-3.2.2]. Les groupes à haute disponibilité Azure ont été présentés dans le chapitre [Groupes à haute disponibilité Azure][planning-guide-3.2.3] de ce document.

Le nombre de domaines de mise à niveau et d’erreur pouvant être utilisés par un groupe à haute disponibilité Azure au sein d’une unité d’échelle Azure est limité. Cela signifie que si vous placez un certain nombre de machines virtuelles dans un groupe à haute disponibilité, tôt ou tard, plusieurs machines virtuelles aboutissent au même domaine de mise à niveau ou d’erreur.

En déployant plusieurs instances de serveurs d’applications SAP dans leurs machines virtuelles dédiées et en supposant que nous avons cinq domaines de mise à niveau, nous obtenons l’image suivante. Le nombre maximal réel de domaines d’erreur et de mise à jour au sein d’un groupe à haute disponibilité peut changer à l’avenir :

![Haute disponibilité des serveurs d’applications SAP dans Azure][planning-guide-figure-3000]


#### <a name="high-availability-for-sap-central-services-on-azure"></a>Haute disponibilité pour SAP Central Services sur Azure

Pour une architecture de haute disponibilité de SAP Central Services sur Azure, consultez l’article [Scénarios et architecture de haute disponibilité pour SAP NetWeaver](./sap-high-availability-architecture-scenarios.md) afin d’avoir une première approche. L’article pointe vers des descriptions plus détaillées pour les systèmes d’exploitation particuliers.

#### <a name="high-availability-for-the-sap-database-instance"></a>Haute disponibilité pour l’instance de base de données SAP

La configuration de la haute disponibilité du SGBD (système de gestion de base de données) SAP classique repose sur deux machines virtuelles de SGBD où la fonctionnalité de haute disponibilité du SGBD sert à répliquer les données issues de l’instance de SGBD active sur l’instance de SGBD passive de la seconde machine virtuelle.

La fonctionnalité de haute disponibilité et de récupération d’urgence pour les SGBD (systèmes de gestion de base de données) en général, ainsi que pour les SGBD spécifiques est décrite dans le [Guide de déploiement de SGBD][dbms-guide].

#### <a name="end-to-end-high-availability-for-the-complete-sap-system"></a>Haute disponibilité de bout en bout pour l’ensemble du système SAP

Voici deux exemples d’architecture de haute disponibilité SAP NetWeaver complète dans Azure (un concernant Windows et un autre pour Linux).

Disques non managés uniquement : De plus, le nombre de machines virtuelles déployées dépasse la limite maximale de comptes de stockage par abonnement. Dans ces cas, les disques durs virtuels doivent être combinés au sein d’un compte de stockage. En général, vous pourriez procéder ainsi en combinant les disques virtuels des machines virtuelles de la couche d’application SAP de différents systèmes SAP.  Nous avons également associé différents disques durs virtuels de plusieurs machines virtuelles de SGBD (système de gestion de base de données) de différents systèmes SAP dans un compte Azure Storage. Tout en gardant à l’esprit les limites d’IOPS des comptes de stockage Azure (<https://azure.microsoft.com/documentation/articles/storage-scalability-targets>)


##### <a name="windows-logologo_windows-ha-on-windows"></a>![Logo Windows.][Logo_Windows] Haute disponibilité sous Windows

![Diagramme illustrant l’architecture de haute disponibilité de l’application SAP NetWeaver avec SQL Server dans Azure IaaS.][planning-guide-figure-3200]

Les constructions Azure suivantes sont utilisées pour le système SAP NetWeaver afin de réduire l’impact par la correction de l’hôte et des problèmes d’infrastructure :

* Le système complet est déployé sur Azure (obligatoire : la couche SGBD, l’instance (A)SCS et la couche d’application complète doivent s’exécuter au même emplacement).
* Le système complet s’exécute au sein d’un abonnement Azure (requis).
* Le système complet s’exécute au sein d’un réseau virtuel Azure (requis).
* La séparation des machines virtuelles d’un système SAP en trois groupes à haute disponibilité est possible même si toutes les machines virtuelles appartiennent au même Réseau virtuel.
* Chaque couche (par exemple SGBD, ASCS, les serveurs d’applications) doit utiliser un groupe à haute disponibilité dédié.
* Toutes les machines virtuelles exécutant des instances de SGBD (système de gestion de base de données) d’un système SAP se trouvent dans un groupe à haute disponibilité. Nous partons du principe qu’il existe plusieurs machines virtuelles exécutant des instances de SGBD (système de gestion de base de données) par système depuis l’utilisation des fonctionnalités de haute disponibilité de SGBD natives, telles que SQL Server AlwaysOn ou Oracle Data Guard.
* Toutes les machines virtuelles exécutant des instances de SGBD (système de gestion de base de données) utilisent leur propre compte de stockage. Les fichiers de données et journaux du SGBD (système de gestion de base de données) sont répliqués à partir d’un compte de stockage vers un autre compte de stockage à l’aide des fonctions de haute disponibilité du SGBD (système de gestion de base de données) qui synchronisent les données. L’indisponibilité d’un compte de stockage entraîne l’indisponibilité d’un nœud de cluster Windows SQL, mais pas du service SQL Server tout entier.
* Toutes les machines virtuelles exécutant l’instance (A)SCS d’un système SAP se trouvent dans un groupe à haute disponibilité. Un cluster de basculement Windows Server est configuré au sein de ces machines virtuelles pour protéger l’instance (A)SCS.
* Toutes les machines virtuelles exécutant des instances (A)SCS utilisent leur propre compte de stockage. Les fichiers d’instance (A)SCS et le dossier global SAP sont répliqués à partir d’un compte de stockage sur un autre compte de stockage à l’aide de la réplication SIOS DataKeeper. L’indisponibilité d’un compte de stockage entraîne l’indisponibilité d’un nœud de cluster Windows du service (A)SCS, mais pas de l’intégralité de ce dernier.
* TOUTES les machines virtuelles représentant la couche du serveur d’applications SAP se trouvent dans un troisième groupe à haute disponibilité.
* TOUTES les machines virtuelles exécutant des serveurs d’applications SAP utilisent leur propre compte de stockage. L’indisponibilité d’un compte de stockage entraîne l’indisponibilité d’un serveur d’applications SAP, là où d’autres serveurs d’applications SAP continuent de s’exécuter.

La figure suivante illustre le même paysage à l’aide de la fonctionnalité Disques managés.

![Architecture de haute disponibilité de l’application SAP NetWeaver avec SQL Server dans Azure IaaS][planning-guide-figure-3201]

##### <a name="linux-logologo_linux-ha-on-linux"></a>![Logo Linux.][Logo_Linux] Haute disponibilité sous Linux

L’architecture de haute disponibilité SAP sous Linux sur Azure est principalement la même que pour Windows comme décrit ci-dessus. Consultez la note SAP [1928533] pour obtenir la liste des solutions de haute disponibilité prises en charge.

### <a name="using-autostart-for-sap-instances"></a><a name="4e165b58-74ca-474f-a7f4-5e695a93204f"></a>Utilisation du démarrage automatique pour les instances SAP

SAP a proposé une fonctionnalité permettant de démarrer automatiquement des instances SAP immédiatement après le démarrage du système d’exploitation au sein de la machine virtuelle. Les étapes exactes ont été documentées dans l’article de base de connaissances SAP [1909114]. Toutefois, SAP recommande de ne plus utiliser ce paramètre, car aucun contrôle n’est exercé sur l’ordre de redémarrage de l’instance, en supposant que plusieurs machines virtuelles ont été affectées ou que plusieurs instances s’exécutent sur une machine virtuelle. Dans le cas d’un scénario Azure classique d’une instance de serveur d’applications SAP sur une machine virtuelle et dans le cas du redémarrage éventuel d’une seule machine virtuelle, le démarrage automatique n’est pas critique et peut être activé en ajoutant ce paramètre :

`Autostart = 1`

Dans le profil de démarrage de l’instance ABAP et/ou Java SAP.

> [!NOTE]
> Le paramètre de démarrage automatique peut avoir également quelques inconvénients. Le paramètre déclenche le démarrage d’une instance ABAP ou Java SAP au démarrage du service Windows/Linux associé de l’instance. Cela est certainement le cas au démarrage du système d’exploitation. Toutefois, les redémarrages des services SAP sont également courants pour la fonctionnalité de gestion du cycle de vie du logiciel SAP, telle que SUM ou d’autres mises à jour et mises à niveau. Ces fonctionnalités n’attendent pas du tout le redémarrage automatique d’une instance. Par conséquent, le paramètre de démarrage automatique doit être désactivé avant d’exécuter ces tâches. Le paramètre de démarrage automatique ne doit pas être utilisé pour les instances SAP en cluster, telles que ASCS/SCS/CI.
>
>

Consultez les informations supplémentaires concernant le démarrage automatique des instances SAP ici :

* [Start/Stop SAP along with your Unix Server Start/Stop (Démarrage/Arrêt de SAP à l’aide de la fonctionnalité correspondante de votre serveur Unix)](https://scn.sap.com/community/unix/blog/2012/08/07/startstop-sap-along-with-your-unix-server-startstop)
* [Starting and Stopping SAP NetWeaver Management Agents (Démarrage et arrêt des agents de gestion SAP NetWeaver)](https://help.sap.com/saphelp_nwpi711/helpdata/en/49/9a15525b20423ee10000000a421938/content.htm)
* [How to enable auto Start of HANA Database (Activation du démarrage automatique de la base de données HANA)](http://sapbasisinfo.com/blog/2016/08/15/enabling-autostart-of-sap-hana-database-on-server-boot-situation/)

### <a name="larger-3-tier-sap-systems"></a>Systèmes SAP à 3 couches plus vastes
Les aspects relatifs à la haute disponibilité des configurations SAP à 3 couches ont déjà été abordés dans les sections précédentes. Mais qu’en est-il des systèmes où la configuration requise du serveur du SGBD (système de gestion de base de données) est trop volumineuse pour se trouver dans Azure, alors que la couche d’application SAP pourrait y être déployée ?

#### <a name="location-of-3-tier-sap-configurations"></a>Emplacement des configurations SAP à 3 couches
Le fractionnement de la couche Application elle-même ou de la couche Application et SGBD (système de gestion de base de données) en local et dans Azure n’est pas pris en charge. Un système SAP est totalement déployé localement OU dans Azure. Il est également impossible que certains serveurs d’applications s’exécutent en local et d’autres dans Azure. Il s’agit du point de départ de la discussion. Le déploiement des composants du SGBD (système de gestion de base de données) d’un système SAP et de la couche du serveur d’applications SAP dans deux régions Azure différentes n’est pas non plus pris en charge. Par exemple, le système SGBD dans la région USA Ouest et la couche Application SAP dans la région USA Centre. La non prise en charge de telles configurations s’explique par la sensibilité de latence de l’architecture de SAP NetWeaver.

Toutefois, au cours de l’année dernière, les partenaires des centres de données ont développé la colocalisation dans les régions Azure. Ces co-emplacements sont souvent proches des centres de données Azure physiques au sein d’une région Azure. La courte distance et la connexion des ressources au sein de la colocalisation via ExpressRoute dans Azure peuvent donner une latence inférieure à 2 millisecondes. Dans ce cas, il est possible de rechercher la couche SGBD (système de gestion de base de données) (y compris le stockage SAN/NAS) dans une colocalisation de ce type, ainsi que la couche Application dans Azure. [Grandes instances HANA](./hana-overview-architecture.md).

### <a name="offline-backup-of-sap-systems"></a>Sauvegarde hors connexion de systèmes SAP
En fonction de la configuration SAP choisie (à 2 ou 3 couches), il peut être nécessaire de procéder à une sauvegarde. du contenu de la machine virtuelle elle-même et de la base de données. Les sauvegardes associées au SGBD (système de gestion de base de données) doivent être effectuées avec des méthodes de base de données. Pour une description détaillée des différentes bases de données, voir le [Guide SGBD (système de gestion de base de données)][dbms-guide]. En revanche, les données SAP peuvent être sauvegardées hors connexion (y compris le contenu de la base de données) comme décrit dans cette section ou en ligne comme indiqué dans la section suivante.

La sauvegarde hors connexion nécessite essentiellement l’arrêt de la machine virtuelle via le portail Azure et une copie du disque de la machine virtuelle de base et de tous les disques attachés à la machine virtuelle. Cela permet de préserver une image dans le temps de la machine virtuelle et des disques associés à celle-ci. Il est recommandé de copier les « sauvegardes » dans un autre compte Azure Storage. Par conséquent, la procédure décrite dans le chapitre [Copie de disques entre comptes Azure Storage][planning-guide-5.4.2] dans ce document s’applique.


Une restauration de cet état consisterait en la suppression de la machine virtuelle de base, ainsi que de ses disques d’origine et des disques montés, la copie des disques enregistrés dans le compte de stockage d’origine ou le groupe de ressources pour les disques managés, puis le redéploiement du système.
Cet article montre comment rédiger le script de ce processus dans PowerShell : <https://www.westerndevs.com/_/azure-snapshots/>

Veillez à installer une nouvelle licence SAP, dans la mesure où la restauration d’une sauvegarde de machine virtuelle selon les modalités ci-dessus entraîne la création d’une clé matérielle.

### <a name="online-backup-of-an-sap-system"></a>Sauvegarde en ligne d’un système SAP

La sauvegarde du SGBD (système de gestion de base de données) est effectuée à l’aide de méthodes de SGBD spécifiques, comme décrit dans le [Guide SGBD (système de gestion de base de données)][dbms-guide].

Il est possible de sauvegarder d’autres machines virtuelles dans le système SAP à l’aide de la fonctionnalité de sauvegarde de machines virtuelles Azure. La sauvegarde de machine virtuelle Azure est une méthode de sauvegarde standard d’une machine virtuelle complète dans Azure. Elle permet de stocker les sauvegardes dans Azure et de restaurer une machine virtuelle.

> [!NOTE]
> Depuis décembre 2015, l’utilisation de la sauvegarde de machine virtuelle NE permet pas de conserver l’ID de machine virtuelle unique utilisé pour la gestion des licences SAP. Cela signifie qu’une restauration à partir d’une sauvegarde de machine virtuelle nécessite l’installation d’une nouvelle clé de licence SAP, dans la mesure où la machine virtuelle restaurée est considérée comme une nouvelle machine virtuelle et non un remplacement de l’ancienne machine sauvegardée.
>
> ![Logo Windows.][Logo_Windows] Windows
>
> En théorie, les machines virtuelles qui exécutent des bases de données peuvent également être sauvegardées de manière cohérente si les systèmes SGBD prennent en charge Windows VSS (Volume Shadow Copy Service <https://msdn.microsoft.com/library/windows/desktop/bb968832(v=vs.85).aspx>) comme le fait SQL Server, par exemple.
> Toutefois, n’oubliez pas qu’une restauration dans le temps des bases de données peut ne pas être possible, selon les sauvegardes de machine virtuelle Azure. Ainsi, il est recommandé d’effectuer des sauvegardes de bases de données avec des systèmes SGBD plutôt que de compter sur la sauvegarde de machines virtuelles Azure.
>
> Pour vous familiariser avec la sauvegarde de machine virtuelle Azure, commencez ici : <https://docs.microsoft.com/azure/backup/backup-azure-vms>.
>
> Il est également possible de combiner Microsoft Data Protection Manager installé sur une machine virtuelle Azure et la sauvegarde Azure pour sauvegarder/restaurer des bases de données. Des informations supplémentaires sont disponibles ici : <https://docs.microsoft.com/azure/backup/backup-azure-dpm-introduction>.
>
> ![Logo Linux.][Logo_Linux] Linux
>
> Il n’existe pas d’équivalent de Windows VSS sous Linux. Par conséquent, seules les sauvegardes cohérentes au niveau des fichiers sont possibles. Les sauvegardes cohérentes au niveau de l’application ne sont pas prises en charge. La sauvegarde du SGBD (système de gestion de base de données) SAP doit être effectuée à l’aide de la fonctionnalité SGBD. Le système de fichiers, qui comprend les données SAP, peut être enregistré, par exemple, à l’aide de tar, comme indiqué ici : <https://help.sap.com/saphelp_nw70ehp2/helpdata/en/d3/c0da3ccbb04d35b186041ba6ac301f/content.htm>
>
>

### <a name="azure-as-dr-site-for-production-sap-landscapes"></a>Azure comme site de récupération d’urgence pour les paysages SAP de production

Depuis le milieu de l’année 2014, des extensions de divers composants relatifs à Hyper-V, System Center et Azure permettent d’utiliser Azure comme site de reprise d’activité après sinistre pour les machines virtuelles basées sur Hyper-V et exécutées localement.

Un billet de blog détaillant comment déployer cette solution est documenté ici : <https://docs.microsoft.com/archive/blogs/saponsqlserver/protecting-sap-solutions-with-azure-site-recovery>.

## <a name="summary-for-high-availability-for-sap-systems"></a>Résumé de la haute disponibilité pour les systèmes SAP

Voici les points clés de la haute disponibilité des systèmes SAP dans Azure :

* À ce stade, le point de défaillance unique de SAP ne peut pas être sécurisé exactement comme il peut l’être dans des déploiements en local. La raison est que les clusters de disques partagés sont encore impossibles à créer dans Azure sans utiliser de logiciels tiers.
* Pour la couche SGBD, vous devez utiliser la fonctionnalité SGBD qui ne repose pas sur la technologie de cluster de disque partagé. Pour plus de détails à ce sujet, voir le [Guide SGBD (système de gestion de base de données)][dbms-guide].
* Pour réduire l’impact des problèmes au sein des domaines d’erreur dans l’infrastructure Azure ou de la maintenance des hôtes, vous devez utiliser des groupes à haute disponibilité Azure :
  * Il est recommandé de disposer d’un groupe à haute disponibilité pour la couche d’applications SAP.
  * Il est recommandé de disposer d’un groupe à haute disponibilité distinct pour la couche du SGBD (système de gestion de base de données) SAP.
  * Il n’est PAS recommandé d’appliquer le même groupe à haute disponibilité pour les machines virtuelles de systèmes SAP différents.
  * Nous vous recommandons d’utiliser la fonctionnalité Disques managés Premium.
* Pour la sauvegarde de la couche SAP SGBD, consultez le [Guide SGBD (système de gestion de base de données)][dbms-guide].
* La sauvegarde des instances de boîte de dialogue SAP n’est pas très utile, dans la mesure où il est généralement plus rapide de redéployer des instances de boîte de dialogue simples.
* La sauvegarde de la machine virtuelle qui contient le répertoire global du système SAP, et tous les profils des différentes instances, est utile et doit être effectuée avec la sauvegarde Windows ou, par exemple, tar sous Linux. Comme il existe des différences entre Windows Server 2008 (R2) et Windows Server 2012 (R2) qui facilitent la sauvegarde à l’aide des versions plus récentes de Windows Server, nous vous recommandons d’exécuter Windows Server 2012 (R2) comme système d’exploitation invité Windows.

## <a name="next-steps"></a>Étapes suivantes
Lisez les articles :

- [Déploiement de machines virtuelles Azure pour SAP NetWeaver](./deployment-guide.md)
- [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](./dbms_guide_general.md)
- [Configurations et opérations de l’infrastructure SAP HANA sur Azure](/- azure/virtual-machines/workloads/sap/hana-vm-operations)
