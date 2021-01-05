---
title: Déploiement SGBD de machines virtuelles SQL Server Azure pour charge de travail SAP | Microsoft Docs
description: Déploiement SGBD de machines virtuelles SQL Server Azure pour charge de travail SAP
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: Azure, SQL Server, SAP, AlwaysOn
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/20/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: ef00f33040d30795ae5374b74d0d7a6e9b6e2156
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957671"
---
# <a name="sql-server-azure-virtual-machines-dbms-deployment-for-sap-netweaver"></a>Déploiement SGBD de machines virtuelles SQL Server Azure pour SAP NetWeaver

[767598]:https://launchpad.support.sap.com/#/notes/767598
[773830]:https://launchpad.support.sap.com/#/notes/773830
[826037]:https://launchpad.support.sap.com/#/notes/826037
[965908]:https://launchpad.support.sap.com/#/notes/965908
[1031096]:https://launchpad.support.sap.com/#/notes/1031096
[1114181]:https://launchpad.support.sap.com/#/notes/1114181
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
[2171857]:https://launchpad.support.sap.com/#/notes/2171857
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

[dbms-guide]:dbms-guide_general.md 
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f 
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b 
[dbms-guide-5.6]:dbms_guide_sqlserver.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d 
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c 
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 
[dbms-guide-900-sap-cache-server-on-premises]:dbms-guide.md#642f746c-e4d4-489d-bf63-73e80177a0a8
[dbms-guide-managed-disks]:dbms-guide.md#f42c6cb5-d563-484d-9667-b07ae51bce29

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

[msdn-set-azurermvmaemextension]:https://msdn.microsoft.com/library/azure/mt670598.aspx

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
[planning-guide-figure-2500]:media/virtual-machines-shared-sap-planning-guide/2500-vm-extension-details.png
[planning-guide-figure-2600]:media/virtual-machines-shared-sap-planning-guide/2600-sap-router-connection.png
[planning-guide-figure-2700]:media/virtual-machines-shared-sap-planning-guide/2700-exposed-sap-portal.png
[planning-guide-figure-2800]:media/virtual-machines-shared-sap-planning-guide/2800-endpoint-config.png
[planning-guide-figure-2900]:media/virtual-machines-shared-sap-planning-guide/2900-azure-ha-sap-ha.png
[planning-guide-figure-300]:media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png
[planning-guide-figure-3000]:media/virtual-machines-shared-sap-planning-guide/3000-sap-ha-on-azure.png
[planning-guide-figure-3200]:media/virtual-machines-shared-sap-planning-guide/3200-sap-ha-with-sql.png
[planning-guide-figure-400]:media/virtual-machines-shared-sap-planning-guide/400-vm-services.png
[planning-guide-figure-600]:media/virtual-machines-shared-sap-planning-guide/600-s2s-details.png
[planning-guide-figure-700]:media/virtual-machines-shared-sap-planning-guide/700-decision-tree-deploy-to-azure.png
[planning-guide-figure-800]:media/virtual-machines-shared-sap-planning-guide/800-portal-vm-overview.png
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd 
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f 

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
[storage-powershell-guide-full-copy-vhd]:../../../storage/common/storage-powershell-guide-full.md#how-to-copy-blobs-from-one-storage-container-to-another
[storage-premium-storage-preview-portal]:../../disks-types.md
[storage-redundancy]:../../../storage/common/storage-redundancy.md
[storage-scalability-targets]:../../../storage/common/scalability-targets-standard-accounts.md
[storage-use-azcopy]:../../../storage/common/storage-use-azcopy.md
[template-201-vm-from-specialized-vhd]:https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-from-specialized-vhd
[templates-101-simple-windows-vm]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-simple-windows-vm
[templates-101-vm-from-user-image]:https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-from-user-image
[virtual-machines-linux-attach-disk-portal]:../../linux/attach-disk-portal.md
[virtual-machines-azure-resource-manager-architecture]:../../../resource-manager-deployment-model.md
[virtual-machines-azurerm-versus-azuresm]:../../../resource-manager-deployment-model.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md 
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md 
[virtual-machines-linux-agent-user-guide]:../../linux/agent-user-guide.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../linux/agent-user-guide.md#command-line-options
[virtual-machines-linux-capture-image]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager]:../../linux/capture-image.md
[virtual-machines-linux-capture-image-resource-manager-capture]:../../linux/capture-image.md#step-2-capture-the-vm
[virtual-machines-linux-configure-raid]:../../linux/configure-raid.md
[virtual-machines-linux-configure-lvm]:../../linux/configure-lvm.md
[virtual-machines-linux-classic-create-upload-vhd-step-1]:../../virtual-machines-linux-classic-create-upload-vhd.md#step-1-prepare-the-image-to-be-uploaded
[virtual-machines-linux-create-upload-vhd-suse]:../../linux/suse-create-upload-vhd.md
[virtual-machines-linux-redhat-create-upload-vhd]:../../linux/redhat-create-upload-vhd.md
[virtual-machines-linux-how-to-attach-disk]:../../linux/add-disk.md
[virtual-machines-linux-how-to-attach-disk-how-to-initialize-a-new-data-disk-in-linux]:../../linux/add-disk.md#connect-to-the-linux-vm-to-mount-the-new-disk
[virtual-machines-linux-tutorial]:../../linux/quick-create-cli.md
[virtual-machines-linux-update-agent]:../../linux/update-agent.md
[virtual-machines-manage-availability-linux]:../../linux/manage-availability.md
[virtual-machines-manage-availability-windows]:../../windows/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:virtual-machines-windows-create-powershell.md
[virtual-machines-sizes-linux]:../../linux/sizes.md
[virtual-machines-sizes-windows]:../../windows/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:../../../azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md
[virtual-machines-sql-server-infrastructure-services]:../../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md
[virtual-machines-sql-server-performance-best-practices]:../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md
[virtual-machines-upload-image-windows-resource-manager]:../../virtual-machines-windows-upload-image.md
[virtual-machines-windows-tutorial]:../../virtual-machines-windows-hero-tutorial.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/eresources/templates/sql-server-2014-alwayson-existing-vnet-and-ad/
[virtual-network-deploy-multinic-arm-cli]:../linux/multiple-nics.md
[virtual-network-deploy-multinic-arm-ps]:../windows/multiple-nics.md
[virtual-network-deploy-multinic-arm-template]:../../../virtual-network/template-samples.md
[virtual-networks-configure-vnet-to-vnet-connection]:../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md
[virtual-networks-create-vnet-arm-pportal]:../../../virtual-network/manage-virtual-network.md#create-a-virtual-network
[virtual-networks-manage-dns-in-vnet]:../../../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md
[virtual-networks-multiple-nics]:../../../virtual-network/virtual-network-deploy-multinic-classic-ps.md
[virtual-networks-nsg]:../../../virtual-network/security-overview.md
[virtual-networks-reserved-private-ip]:../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md
[virtual-networks-static-private-ip-arm-pportal]:../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md
[virtual-networks-udr-overview]:../../../virtual-network/virtual-networks-udr-overview.md
[vpn-gateway-about-vpn-devices]:../../../vpn-gateway/vpn-gateway-about-vpn-devices.md
[vpn-gateway-create-site-to-site-rm-powershell]:../../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md
[vpn-gateway-cross-premises-options]:../../../vpn-gateway/vpn-gateway-plan-design.md
[vpn-gateway-site-to-site-create]:../../../vpn-gateway/vpn-gateway-site-to-site-create.md
[vpn-gateway-vpn-faq]:../../../vpn-gateway/vpn-gateway-vpn-faq.md
[xplat-cli]:../../../cli-install-nodejs.md
[xplat-cli-azure-resource-manager]:../../../xplat-cli-azure-resource-manager.md




Ce document décrit les différents domaines à prendre en compte lors du déploiement de SQL Server pour une charge de travail SAP dans Azure IaaS. Condition préalable à ce document, vous devez avoir lu le document [Éléments à prendre en compte pour le déploiement SGBD de machines virtuelles Azure pour charge de travail SAP](./dbms_guide_general.md) et d’autres guides de la [documentation sur la charge de travail SAP sur Azure](./get-started.md). 



> [!IMPORTANT]
> Ce document est consacré à la version de Windows sur SQL Server. Le protocole SAP ne prend en charge la version Linux de SQL Server avec aucun logiciel SAP. Le document ne traite pas ici de Microsoft Azure SQL Database, qui est une offre Platform as a Service (PaaS) de la plateforme Microsoft Azure. Ce document porte sur l’exécution du produit SQL Server pour les déploiements locaux dans Azure Virtual Machines, en tirant parti des capacités d’Infrastructure as a Service (IaaS) d’Azure. Les capacités et fonctionnalités de base de données de ces deux offres sont différentes et ne doivent pas être confondues. Voir aussi : <https://azure.microsoft.com/services/sql-database/>
> 
>

En général, vous devez penser à utiliser les versions de SQL Server les plus récentes pour exécuter une charge de travail SAP dans Azure IaaS. Les dernières versions de SQL Server offrent une meilleure intégration à certains des services et fonctionnalités Azure. Et elles comportent des modifications qui optimisent les opérations dans une infrastructure Azure IaaS.

Avant de continuer, nous vous recommandons de consulter l’article [Qu’est-ce que SQL Server sur les machines virtuelles Azure (Windows) ?][https://docs.microsoft.com/azure/azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview ]

Dans les sections suivantes, des parties de la documentation à laquelle le lien ci-dessus renvoie sont regroupées et mentionnées. Les particularités concernant SAP sont également indiquées et certains concepts décrits plus en détail. Cependant, nous vous recommandons vivement d’examiner la documentation ci-dessus avant de lire la documentation propre à SQL Server.

Avant de continuer, il y a certaines informations spécifiques sur SQL Server dans IaaS que vous devez connaître :

* **Prise en charge des versions SQL** : Pour les clients SAP, les versions SQL Server 2008 R2 et ultérieures sont prises en charge sur les machines virtuelles Microsoft Azure. Les éditions antérieures ne sont pas prises en charge. Pour plus d’informations, voir cette [déclaration officielle](https://support.microsoft.com/kb/956893) générale. En règle générale, SQL Server 2008 l’est également. Cependant, en raison de fonctionnalités significatives pour SAP introduites avec SQL Server 2008 R2, SQL Server 2008 R2 est la version minimale requise pour SAP. En général, vous devez penser à utiliser les versions de SQL Server les plus récentes pour exécuter une charge de travail SAP dans Azure IaaS. Les dernières versions de SQL Server offrent une meilleure intégration à certains des services et fonctionnalités Azure. Et elles comportent des modifications qui optimisent les opérations dans une infrastructure Azure IaaS. Cet article est donc limité à SQL Server 2016 et SQL Server 2017.
* **Performances SQL** : Les machines virtuelles hébergées par Microsoft Azure fonctionnent bien par rapport aux autres offres de virtualisation cloud publiques, mais les résultats peuvent varier au cas par cas. Consultez l’article [Meilleures pratiques relatives aux performances de SQL Server dans les machines virtuelles Azure](../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md).
* **Utilisation d’images de la Place de marché Azure** : La méthode la plus rapide pour déployer une nouvelle machine virtuelle Microsoft Azure est d’utiliser une image de la Place de marché Microsoft Azure. En effet, cette plateforme propose des images qui contiennent les versions les plus récentes de SQL Server. Les images hébergeant déjà SQL Server ne peuvent pas être directement utilisées pour les applications SAP NetWeaver. En effet, le classement par défaut installé au sein de ces images correspond à celui de SQL Server, et non au classement requis par les systèmes SAP NetWeaver. Pour pouvoir utiliser ces images, suivez la procédure décrite dans le chapitre [Utilisation d'images SQL Server issues de la Place de marché Microsoft Azure][dbms-guide-5.6]. 


## <a name="recommendations-on-vmvhd-structure-for-sap-related-sql-server-deployments"></a>Recommandations portant sur la structure des machines virtuelles/disques VHD pour les déploiements de SQL Server associés à SAP
Conformément à la description générale, le système d’exploitation, les exécutables SQL Server et, dans le cas de systèmes SAP à deux niveaux, les exécutables SAP doivent se situer ou être installés sur des disques Azure distincts. Bien souvent, la plupart des bases de données système SQL Server ne sont pas pleinement exploitées par les charges de travail SAP NetWeaver. Néanmoins, les bases de données système de SQL Server (master, msdb et modèle) doivent être, avec les autres répertoires SQL Server, sur un disque Azure distinct. La base de données tempdb de SQL Server doit se trouver sur le lecteur D:\ non persistant ou sur un disque distinct.


* Avec tous les types de machines virtuelles certifiées SAP (voir la note SAP [1928533]), à l’exception des machines virtuelles de la série A, les données de la base de données tempdb et les fichiers journaux peuvent être placés sur le lecteur D:\ non persistant. 
* Pour les versions antérieures de SQL Server, où SQL Server installe tempdb avec un fichier de données par défaut, il est recommandé d’utiliser plusieurs fichiers de données tempdb. N’oubliez pas que les volumes de lecteur D:\ sont différents selon le type de machine virtuelle. Pour connaître les tailles exactes du lecteur D:\ des différentes machines virtuelles, consultez l’article [Tailles des machines virtuelles Windows dans Azure](../../sizes.md).

Ces configurations permettent à la base de données tempdb de consommer davantage d’espace et, plus important, davantage d’IOPS et de bande passante de stockage que celui que peut proposer le lecteur système. Le lecteur D:\ non persistant offre également un débit et une latence d’E/S améliorés (à l’exception des machines virtuelles de la série A). Pour déterminer la taille de base de données tempdb appropriée, vous pouvez consulter les tailles des bases de données tempdb sur les systèmes existants. 

>[!NOTE]
> Si vous placez un fichier journal et les fichiers de données tempdb dans un dossier sur le lecteur D:\ que vous avez créé, vous devez vous assurer que le dossier existe après un redémarrage de la machine virtuelle. Étant donné que le lecteur D:\ vient d’être initialisé après le redémarrage d’une machine virtuelle, toutes les structures de répertoires et de fichiers sont effacées. La possibilité de recréer des structures de répertoires ultérieurement sur lecteur D:\ avant le démarrage du service SQL Server est documentée dans [cet article](https://cloudblogs.microsoft.com/sqlserver/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions/).

Voici un exemple de configuration de machine virtuelle qui exécute SQL Server avec une base de données SAP, et dans laquelle les données et le fichier journal de tempdb sont placés sur le lecteur D:\ :

![Diagramme de configuration de disque de machine virtuelle simple pour SQL Server](./media/dbms_sqlserver_deployment_guide/Simple_disk_structure.PNG)

Le diagramme ci-dessus affiche un cas simple. Comme l’évoque l’article [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md), le type, le nombre et la taille des disques du stockage Azure dépendent de différents facteurs. Cependant, en général, nous recommandons :

- D’utiliser un grand volume, qui contient les fichiers de données de SQL Server. La raison motivant cette configuration est que dans un environnement réel, il existe de nombreuses bases de données SAP avec des fichiers de base de données de taille différente avec diverses charges de travail d’E/S.
- D’utiliser le lecteur D:\ pour tempdb tant que les performances sont suffisantes. Si la charge de travail globale est limitée en performances par tempdb qui se trouve sur le lecteur D:\, vous devez peut-être songer à déplacer tempdb vers des disques Stockage Premium Azure ou Ultra distincts, comme recommandé dans [cet article](../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md).


### <a name="special-for-m-series-vms"></a>Remarque relative aux machines virtuelles de la série M
Pour une machine virtuelle Azure de la série M, la latence d’écriture dans les journaux de transactions peut être réduite par des facteurs, comparée aux performances de stockage Premium Azure, lors de l’utilisation de l’Accélérateur des écritures Azure. Par conséquent, vous devez déployer l’Accélérateur des écritures Azure pour les disques durs virtuels qui constituent le volume du journal des transactions SQL Server. Les détails peuvent être consultés dans le document [Accélérateur des écritures](../../how-to-enable-write-accelerator.md).
  

### <a name="formatting-the-disks"></a>Formatage des disques
Dans le cas de SQL Server, la taille du bloc NTFS des disques contenant des données et des fichiers journaux SQL Server doit être de 64 ko. Il est inutile de mettre en forme le lecteur D:\. En effet, ce lecteur est déjà mis en forme.

Pour vous assurer que la restauration ou la création de bases de données n’initialise pas les fichiers de données en supprimant le contenu des fichiers, vous devez vous assurer que le contexte utilisateur dans lequel le service SQL Server s’exécute dispose de l’autorisation adéquate. En général, les utilisateurs du groupe Administrateurs Windows disposent des autorisations requises. Si le service SQL Server est exécuté dans le contexte utilisateur d’un administrateur autre que Windows, vous devez affecter à cet utilisateur le droit **Effectuer les tâches de maintenance de volume**.  Pour plus d’informations, consultez cet article de la Base de connaissances Microsoft : <https://support.microsoft.com/kb/2574695>

### <a name="impact-of-database-compression"></a>Impact de la compression de base de données
Dans les configurations pour lesquelles la bande passante d’E/S peut devenir un facteur de limitation, toutes les mesures qui réduisent le nombre d’E/S par seconde peuvent contribuer à étirer la charge de travail exécutable dans un scénario IaaS comme Azure. Par conséquent, si vous ne l’avez pas encore fait, SAP et Microsoft recommandent l’application de la compression de page SQL Server avant le chargement de bases de données SAP existantes dans Azure.

Nous recommandons d’effectuer une compression de base de données avant le chargement sur Azure pour deux raisons :

* La quantité de données à charger est moins importante.
* En supposant que l’on peut utiliser en local un matériel plus performant avec plus d’UC, une bande passante d’E/S supérieure ou une latence d’E/S inférieure, la durée d’exécution de la compression est plus courte.
* Des bases de données plus petites peuvent permettre de diminuer les coûts liés à l’allocation de disque.

La compression de bases de données fonctionne aussi bien dans une machine virtuelle Azure qu’en local. Pour plus d’informations sur la compression des bases de données SQL Server pour SAP NetWeaver, consultez l’article [Improved SAP compression tool MSSCOMPRESS](/archive/blogs/saponsqlserver/improved-sap-compression-tool-msscompress) (Outil de compression SAP amélioré, MSSCOMPRESS). 

## <a name="sql-server-2014-and-more-recent---storing-database-files-directly-on-azure-blob-storage"></a>SQL Server 2014 et ultérieur : Stockage des fichiers de base de données directement dans le Stockage Blob Azure
Les versions SQL Server 2014 et ultérieures permettent de stocker les fichiers de base de données directement dans le stockage d’objets blob Azure, sans qu’il soit nécessaire d’utiliser un disque VHD pour « envelopper » ces fichiers. Lorsque vous utilisez le stockage Azure Standard ou des types de machines virtuelles plus petits, ce type de déploiement permet de dépasser les limites d’IOPS imposées par le nombre limité de disques pouvant être montés sur certains types de machines virtuelles de taille plus réduite. Ce mode de déploiement fonctionne dans le cas des bases de données utilisateur, mais pas pour les bases de données système de SQL Server. Il peut également être appliqué aux fichiers journaux et données de SQL Server. Si vous souhaitez déployer une base de données SQL Server SAP de cette manière plutôt que de recourir à des disques VHD, tenez compte des remarques suivantes :

* Le compte de stockage utilisé doit se trouver dans la même région Azure que celui qui permet de déployer la machine virtuelle sur laquelle SQL Server s’exécute.
* Les considérations relatives à la répartition de disques VHD sur différents comptes de stockage Azure qui ont été abordées précédemment portent également sur cette méthode de déploiement. Cela signifie que les opérations d’E/S sont concernées par les limites du compte Azure Storage.
* À la place de la prise en compte du quota d’E/S de stockage des machines virtuelles, c’est le trafic sur les objets blob de stockage représentant les fichiers journaux et de données SQL Server, qui est pris en compte dans la bande passante réseau de du type de machine virtuelle spécifique. Pour connaître la bande passante réseau et de stockage d’un type particulier de machine virtuelle, consultez l’article [Tailles des machines virtuelles Windows dans Azure](../../sizes.md).
* Quand vous poussez des E/S de fichier au-delà du quota de réseau, vous abandonnez la plupart du quota de stockage et, par conséquent, n’utilisez que partiellement la bande passante de la machine virtuelle.
* Les objectifs de performances de débit d’IOPS et d’E/S du stockage Premium Azure pour les différentes tailles de disques ne s’appliquent plus, même si les objets blob que vous avez créés sont situés dans le stockage Premium Azure. Les objectifs sont documentés dans l’article [Stockage Premium hautes performances et disques managés pour les machines virtuelles](../../disks-types.md#premium-ssd). Conséquence du placement des fichiers journaux et des fichiers de données SQL Server directement dans les objets blob qui sont stockés dans le stockage Premium Azure, les caractéristiques de performances peuvent être différentes par rapport aux disques durs virtuels situés dans le stockage Premium Azure.
* La mise en cache basée sur l’hôte, disponible pour les disques de stockage Premium Azure, n’est pas disponible quand vous placez les fichiers de données SQL Server directement sur des objets blob Azure.
* Sur des machines virtuelles de la série M, l’Accélérateur des écritures Azure ne peut pas être utilisé pour assurer la prise en charge des écritures en moins d’une milliseconde dans le fichier journal de transactions SQL Server. 

Vous trouverez des détails sur cette fonctionnalité dans l’article [Fichiers de données SQL Server dans Microsoft Azure](/sql/relational-databases/databases/sql-server-data-files-in-microsoft-azure).

Pour les systèmes de production, il est recommandé d’éviter cette configuration et de placer plutôt les fichiers de données et les fichiers journaux SQL Server sur des disques durs virtuels de stockage Premium Azure au lieu de les placer directement dans les objets blob Azure.


## <a name="sql-server-2014-buffer-pool-extension"></a>Fonctionnalité d’extension du pool de mémoires tampons de SQL Server 2014
SQL Server 2014 propose une nouvelle fonctionnalité appelée [extension du pool de mémoires tampons](/sql/database-engine/configure-windows/buffer-pool-extension). Cette fonctionnalité permet d’étendre le pool de mémoires tampons de SQL Server conservé en mémoire grâce à un cache de deuxième niveau pris en charge par les disques SSD locaux d’un serveur ou d’une machine virtuelle. L’extension du pool de mémoires tampons permet de conserver une plus grande plage de travail des données « en mémoire ». Par rapport à l’accès au stockage Azure standard, l’accès à l’extension du pool de mémoires tampons stocké sur les disques SSD locaux d’une machine virtuelle est beaucoup plus rapide. Si vous comparez l’extension du pool de mémoires tampons au cache de lecture du stockage Premium Azure, comme recommandé pour les fichiers de données SQL Server, aucun avantage significatif n’est attendu pour l’extension du pool de mémoires tampons. En effet, les deux caches (le cache de lecture du stockage Premium et la fonction d’extension du pool de mémoires tampons SQL Server) utilisent les disques locaux du nœud de calcul Azure.

L’expérience acquise dans l’intervalle avec l’extension du pool de mémoires tampons SQL Server avec la charge de travail SAP est mixte et ne permet pas encore de préconiser des recommandations claires sur son utilisation dans tous les cas. Il est idéal que la plage de travail requise par l’application SAP s’intègre à la mémoire principale. Avec Azure qui propose désormais des machines virtuelles avec jusqu’à 4 To de mémoire, il doit être réalisable de conserver la plage de travail en mémoire. L’utilisation de l’extension du pool de mémoires tampons est donc limitée à quelques rares cas et ne doit pas être généralisée.  

## <a name="backuprecovery-considerations-for-sql-server"></a>Considérations relatives à la sauvegarde/restauration pour SQL Server
Lors du déploiement de SQL Server dans Azure, votre méthodologie de sauvegarde doit être passée en revue. Même si le système n’est pas un système de production, la base de données SAP hébergée par SQL Server doit être sauvegardée régulièrement. Comme Azure Storage conserve trois images, la sauvegarde joue désormais un rôle moins important en matière de compensation des pannes du stockage. La raison principale du maintien d’un plan de sauvegarde et de récupération approprié réside davantage dans le fait que vous pouvez compenser les erreurs logiques/manuelles en fournissant des fonctionnalités de récupération jusqu’à une date et heure. Par conséquent, l’objectif est soit d’utiliser les sauvegardes pour restaurer la base de données à un moment donné, soit d’utiliser les sauvegardes dans Azure pour amorcer un autre système en copiant la base de données existante. 

Pour examiner les différentes possibilités de sauvegarde de SQL Server dans Azure, consultez l’article [Sauvegarde et restauration de SQL Server dans les machines virtuelles Azure](../../../azure-sql/virtual-machines/windows/backup-restore.md). Cet article présente plusieurs possibilités.

### <a name="manual-backups"></a>Sauvegardes manuelles
Vous disposez de plusieurs solutions pour effectuer des sauvegardes « manuelles » :

1. Vous pouvez effectuer des sauvegardes SQL Server classiques sur des disques Azure attachés. Grâce à cette méthode, vous disposez de sauvegardes rapidement pour les actualisations de systèmes et développez de nouveaux systèmes, copies de systèmes SAP existants.
2.  SQL Server 2012 CU4 et les versions ultérieures permettent de sauvegarder les bases de données vers une URL de stockage Azure.
3.  Sauvegardes sous forme d’instantanés de fichier pour les fichiers de base de données dans le stockage Blob Azure. Cette méthode fonctionne uniquement si vos fichiers journaux et fichiers de données SQL Server sont situés dans le stockage blob Azure.

La première méthode est bien connue et appliquée également dans de nombreux cas dans le monde réel. Néanmoins, avec cette méthode, il vous incombe de trouver une solution pour l’emplacement de sauvegarde à plus long terme. Puisque vous ne souhaitez pas conserver vos sauvegardes pendant 30 jours ou plus dans le stockage Azure attaché localement, vous devez utiliser les services de sauvegarde Azure ou un autre outil de sauvegarde/récupération tiers qui inclut la gestion de la rétention de vos sauvegardes ainsi que la gestion de leur accès. Vous pouvez également créer un serveur de fichiers volumineux dans Azure à l’aide des espaces de stockage Windows.

La seconde méthode est décrite de façon plus détaillée dans l’article [Sauvegarde SQL Server vers une URL](../../../azure-sql/virtual-machines/windows/backup-restore.md). Cette fonctionnalité varie en fonction des versions de SQL Server. En conséquence, vous devez consulter la documentation correspondant à votre version SQL Server. Il est important de noter que cet article répertorie un grand nombre de restrictions. Vous pouvez effectuer la sauvegarde dans :

- Un seul objet blob de pages Azure, ce qui limite la taille de la sauvegarde à 1 000 Go. Cette restriction limite également le débit que vous pouvez obtenir.
- Plusieurs objets blob de blocs Azure (jusqu’à 64), ce qui permet d’obtenir une taille théorique de sauvegarde de 12 To. Toutefois, les tests réalisés avec des bases de données de clients ont révélé que la taille de sauvegarde maximale peut être inférieure à sa limite théorique. Dans ce cas, vous êtes chargé de gérer la rétention des sauvegardes ainsi que leur accès.


### <a name="automated-backup-for-sql-server"></a>Sauvegarde automatisée de SQL Server
La sauvegarde automatisée fournit un service de sauvegarde automatique pour les éditions SQL Server Standard et Entreprise s’exécutant sur une machine virtuelle Windows dans Azure. Ce service est fourni par [l’extension SQL Server IaaS Agent](../../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md), qui est automatiquement installée sur les images de machine virtuelle Windows SQL Server dans le portail Azure. Si vous déployez vos propres images de système d’exploitation avec SQL Server installé, vous devez installer les extensions de machine virtuelle séparément. Les étapes nécessaires sont documentées dans cet [article](../../../azure-sql/virtual-machines/windows/sql-server-iaas-agent-extension-automate-management.md).

Vous trouverez plus d’informations sur les potentiels de cette méthode dans les articles suivants :

- SQL Server 2014 : [Sauvegarde automatisée pour les machines virtuelles SQL Server 2014 (Resource Manager)](../../../azure-sql/virtual-machines/windows/automated-backup-sql-2014.md)
- SQL Server 2016/2017 : [Sauvegarde automatisée v2 pour les machines virtuelles Azure (Resource Manager)](../../../azure-sql/virtual-machines/windows/automated-backup.md)

En consultant la documentation, vous pouvez constater que la fonctionnalité a été améliorée avec les versions plus récentes de SQL Server. Vous trouverez des détails supplémentaires sur les sauvegardes automatisées de SQL Server dans l’article [Sauvegarde managée SQL Server sur Microsoft Azure](/sql/relational-databases/backup-restore/sql-server-managed-backup-to-microsoft-azure). En théorie, la taille de sauvegarde est limitée à 12 To.  Les sauvegardes automatisées peuvent être une bonne solution pour les tailles de sauvegarde atteignant jusqu’à 12 To. Étant donné que plusieurs objets blob sont écrits en parallèle, vous pouvez vous attendre à un débit supérieur à 100 Mo/s. 
 

### <a name="azure-backup-for-sql-server-vms"></a>Sauvegarde Azure pour machines virtuelles SQL Server
Cette nouvelle méthode de sauvegarde SQL Server est proposée depuis juin 2018, en tant que préversion publique, par les services de sauvegarde Azure. La méthode de sauvegarde SQL Server est la même que celle qui est utilisée par d’autres outils tiers, à savoir l’interface VSS/VDI de SQL Server pour transmettre en continu les sauvegardes vers un emplacement cible. Dans ce cas, l’emplacement cible est le coffre Azure Recovery Service.

Une description plus que détaillée de cette méthode de sauvegarde, qui ajoute de nombreux avantages tels que les configurations de sauvegarde centrales, la supervision et l’administration est disponible [ici](../../../backup/backup-azure-sql-database.md). 


### <a name="third-party-backup-solutions"></a>Solutions de sauvegarde tierces
Pour un certain nombre de clients SAP, il n’y avait aucune possibilité de recommencer et d’introduire de toutes nouvelles solutions de sauvegarde pour leur environnement SAP qui s’exécutait sur Azure. En conséquence, les solutions de sauvegarde existantes devaient être utilisées et étendues dans Azure. L’extension des solutions de sauvegarde existantes dans Azure fonctionnait généralement bien avec la plupart des principaux fournisseurs de cet espace. 


## <a name="using-a-sql-server-image-out-of-the-microsoft-azure-marketplace"></a><a name="1b353e38-21b3-4310-aeb6-a77e7c8e81c8"></a>Utilisation d’une image SQL Server issue de la Place de marché Microsoft Azure
Dans la Place de marché Azure, Microsoft propose des machines virtuelles qui contiennent déjà des versions de SQL Server. Pour les clients SAP qui requièrent des licences pour SQL Server et Windows, l’utilisation de ces images peut être l’occasion de répondre aux besoins en termes de licences, en configurant des machines virtuelles déjà dotées de SQL Server. Pour pouvoir utiliser ces images pour SAP, vous devez tenir compte des considérations suivantes :

* Les versions de SQL Server autres que les versions d’évaluation nécessitent des frais d’acquisition plus élevés que les machines virtuelles « Windows uniquement » qui sont déployées à partir de la Place de marché Azure. Pour comparer les prix, consultez les articles <https://azure.microsoft.com/pricing/details/virtual-machines/windows/> et <https://azure.microsoft.com/pricing/details/virtual-machines/sql-server-enterprise/>. 
* Vous pouvez uniquement utiliser les versions de SQL Server qui sont prises en charge par SAP.
* Le classement de l’instance SQL Server qui est installée dans les machines virtuelles proposées par la Place de marché ne correspond pas à celui que nécessite SAP NetWeaver pour l’instance SQL Server. Toutefois, vous pouvez modifier ce classement, en suivant les instructions de la section suivante.

### <a name="changing-the-sql-server-collation-of-a-microsoft-windowssql-server-vm"></a>Modification du classement SQL Server d’une machine virtuelle Microsoft Windows/SQL Server
Étant donné que les images SQL Server figurant dans la Place de marché Microsoft Azure ne sont pas configurées pour utiliser le classement exigé par les applications SAP NetWeaver, elles doivent être modifiées immédiatement après le déploiement. Pour SQL Server, cette modification de classement peut être exécutée en suivant la procédure ci-après dès que la machine virtuelle est déployée et qu’un administrateur peut se connecter à cette dernière :

* Ouvrez une fenêtre de commande Windows en tant qu’administrateur.
* Remplacez le répertoire par celui-ci : C:\Program Files\Microsoft SQL Server\110\Setup Bootstrap\SQLServer2012.
* Exécutez la commande : Setup.exe /QUIET /ACTION=REBUILDDATABASE /INSTANCENAME=MSSQLSERVER /SQLSYSADMINACCOUNTS=`<local_admin_account_name`> /SQLCOLLATION=SQL_Latin1_General_Cp850_BIN2   
  * `<local_admin_account_name`&gt; correspond au compte qui a été défini en tant que compte Administrateur lors du déploiement de la machine virtuelle pour la première fois, via la galerie.

Le processus doit prendre quelques minutes seulement. Pour vérifier que le résultat attendu a été obtenu, effectuez les étapes suivantes :

* Ouvrez SQL Server Management Studio.
* Ouvrez une fenêtre Requête.
* Exécutez la commande sp_helpsort dans la base de données MASTER de SQL Server.

Le résultat doit être similaire à ce qui suit :

```output
Latin1-General, binary code point comparison sort for Unicode Data, SQL Server Sort Order 40 on Code Page 850 for non-Unicode Data
```

Si le résultat est différent, INTERROMPEZ immédiatement le déploiement de SAP et cherchez à savoir pourquoi la commande n’a pas fonctionné comme prévu. Le système ne prend **PAS** en charge le déploiement d’applications SAP NetWeaver sur une instance SQL Server avec des pages de codes SQL autres que celle indiquée ci-dessus.

## <a name="sql-server-high-availability-for-sap-in-azure"></a>Solutions SQL Server haute disponibilité pour SAP dans Azure
En utilisant SQL Server dans les déploiements IaaS Azure pour SAP, plusieurs méthodes sont à votre disposition pour déployer la couche SGBD à haute disponibilité. Comme mentionné déjà dans l’article [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md), Azure fournit différents contrats de niveau de service (SLA) de durée de bon fonctionnement pour une seule machine virtuelle et une paire de machines virtuelles déployées dans un groupe à haute disponibilité Azure. Il est supposé que vous recherchez le contrat SLA de temps d’activité pour vos déploiements de production qui nécessitent le déploiement dans des groupes à haute disponibilité Azure. Dans ce cas, vous devez déployer au moins deux machines virtuelles dans un tel groupe à haute disponibilité. Une machine virtuelle exécute l’instance SQL Server active. L’autre machine virtuelle exécute l’instance passive.

### <a name="sql-server-clustering-using-windows-scale-out-file-server-or-azure-shared-disk"></a>Clustering de SQL Server à l’aide du serveur de fichiers avec Scale-out de Windows ou d’un disque partagé Azure
Avec Windows Server 2016, Microsoft a introduit les [espaces de stockage direct](/windows-server/storage/storage-spaces/storage-spaces-direct-overview). La prise en charge du clustering FCI de SQL Server dépend du déploiement des espaces de stockage direct. Azure propose également des [disques partagés Azure](../../disks-shared-enable.md?tabs=azure-cli) qui peuvent être utilisés pour le clustering Windows. Pour la charge de travail SAP, nous ne prenons pas en charge ces options de haute disponibilité. 

### <a name="sql-server-log-shipping"></a>Copie des journaux de transaction SQL Server
L’une des méthodes permettant d’assurer la haute disponibilité est la copie des journaux de transaction SQL Server. Si les machines virtuelles prenant part à la configuration haute disponibilité disposent de la fonctionnalité de résolution de noms, il n’y a aucune difficulté. L’installation dans Azure ne diffère pas d’une installation effectuée en local. Pour en savoir plus sur la configuration de la copie des journaux de transaction et connaître les principes correspondants, consultez l’article [À propos de la copie des journaux de transaction (SQL Server)](/sql/database-engine/log-shipping/about-log-shipping-sql-server).

La fonctionnalité de copie des journaux de transaction de SQL Server a été rarement utilisée dans Azure pour atteindre la haute disponibilité dans une région Azure. Toutefois, dans les scénarios suivants, les clients SAP utilisaient la copie des journaux de transaction avec succès conjointement avec Azure :

- Scénarios de récupération d’urgence d’une région Azure dans une autre région Azure.
- Configuration de la reprise d’activité après sinistre à partir d’un environnement local dans une région Azure.
- Scénarios de basculement à partir d’un environnement local vers Azure. Dans ces cas, la copie des journaux de transaction est utilisée pour synchroniser le nouveau déploiement de SGBD dans Azure avec le système de production en cours en local. Au moment du basculement, la production est arrêtée, et il convient de vérifier que les dernières sauvegardes des journaux de transactions ont été transférées vers le déploiement SGBD Azure. Le déploiement SGBD Azure est ouvert pour la production.  



### <a name="database-mirroring"></a>Mise en miroir de bases de données
La mise en miroir de bases de données prise en charge par SAP (voir Note de SAP [965908]) s’appuie sur la définition d’un partenaire de basculement dans la chaîne de connexion SAP. Dans le cas de déploiements entre différents locaux, nous partons du principe que les deux machines virtuelles se trouvent au sein du même domaine, et que, au sein du contexte utilisateur dans lequel les deux instances SQL Server s’exécutent, ces dernières sont également des utilisateurs du domaine et disposent de privilèges adéquats dans les deux instances SQL Server concernées. Par conséquent, la procédure d’installation de la fonctionnalité de mise en miroir de bases de données dans Azure ne diffère pas de celle d’une configuration ou installation locale classique.

Pour les déploiements sur cloud uniquement, la méthode la plus simple consiste à configurer un autre domaine dans Azure, afin que les machines virtuelles SGBD (système de gestion de base de données), ainsi que les machines virtuelles SAP dédiées, dans l’idéal, se trouvent au sein d’un même domaine.

Si cela n’est pas possible, vous pouvez recourir aux certificats associés aux points de terminaison de la mise en miroir de bases de données, comme indiqué ici : <https://docs.microsoft.com/sql/database-engine/database-mirroring/use-certificates-for-a-database-mirroring-endpoint-transact-sql>

Un tutoriel permettant de configurer la mise en miroir de bases de données dans Azure est disponible ici : <https://docs.microsoft.com/sql/database-engine/database-mirroring/database-mirroring-sql-server> 

### <a name="sql-server-always-on"></a>SQL Server AlwaysOn
Étant donné que la fonction Always On est prise en charge pour les systèmes SAP locaux (voir la note SAP [1772688]), sa prise en charge est assurée avec SAP dans Azure. Certains éléments doivent être pris en compte pour le déploiement de l’écouteur de groupe de disponibilité SQL Server (à ne pas confondre avec le groupe à haute disponibilité Azure), car à ce stade Azure ne permet pas de créer un objet AD/DNS (il le permet toutefois localement). Par conséquent, certaines étapes d’installation différentes sont nécessaires pour surmonter le comportement spécifique d’Azure.

Lors de l’utilisation de l’écouteur de groupe de disponibilité, tenez compte des considérations suivantes :

* L’utilisation de l’écouteur de groupe de disponibilité n’est possible que sur un système Windows Server 2012 ou version ultérieure, utilisé en tant que SE invité de la machine virtuelle. Pour Windows Server 2012, veillez à appliquer ce correctif : <https://support.microsoft.com/kb/2854082> 
* Ce correctif n’existe pas pour Windows Server 2008 R2. La fonction AlwaysOn doit être utilisée de la même manière que la fonctionnalité de mise en miroir de bases de données, via la spécification d’un partenaire de basculement dans la chaîne de connexion (grâce au paramètre SAP default.pfl dbs/mss/server : voir la note SAP [965908]).
* Lorsque vous utilisez un écouteur de groupe de disponibilité, les machines virtuelles de base de données doivent être connectées à un équilibreur de charge dédié. Pour éviter qu'Azure n'attribue de nouvelles adresses IP lorsque les deux machines virtuelles sont arrêtées en même temps, il convient d'attribuer des adresses IP statiques aux interfaces réseau de ces machines au sein de la configuration Always On (la procédure de définition d'une adresse IP statique est décrite dans [cet][virtual-networks-reserved-private-ip] article).
* La création d’une configuration de cluster WSFC requiert certaines étapes spécifiques lorsque ce cluster doit se voir affecter une adresse IP spécifique, car la fonctionnalité actuelle d’Azure affecte au nom du cluster la même adresse IP que celle du nœud sur lequel le cluster est créé. Ce comportement signifie que l’attribution d’une adresse IP différente au cluster doit faire l’objet d’une étape manuelle.
* L’écouteur de groupe de disponibilité va être créé dans Azure avec les points de terminaison TCP/IP qui sont affectés aux machines virtuelles exécutant les réplicas principaux et secondaires du groupe de disponibilité.
* Il peut être nécessaire de sécuriser ces points de terminaison avec des ACL.

Documentation détaillée sur le déploiement de la solution Always On avec SQL Server sur des machines virtuelles Azure :

- [Présentation des groupes de disponibilité SQL Server Always On sur des machines virtuelles Azure](../../../azure-sql/virtual-machines/windows/availability-group-overview.md)
- [Configurer un groupe de disponibilité Always On sur des machines virtuelles Azure dans des régions différentes](../../../azure-sql/virtual-machines/windows/availability-group-manually-configure-multiple-regions.md)
- [Configurer un équilibreur de charge pour un groupe de disponibilité Always On dans Azure](../../../azure-sql/virtual-machines/windows/availability-group-load-balancer-portal-configure.md)

>[!NOTE]
> Si vous configurez l’équilibreur de charge Azure pour l’adresse IP virtuelle de l’écouteur de groupe de disponibilité, assurez-vous que l’élément DirectServerReturn est configuré. La configuration de cette option permet de réduire la latence d’opération complète du réseau entre la couche d’application SAP et la couche SGBD. 

SQL Server Always On est la fonctionnalité de récupération d’urgence et haute disponibilité la plus couramment utilisée dans Azure pour les déploiements de charge de travail SAP. La plupart des clients utilisent Always On pour la haute disponibilité au sein d’une seule région Azure. Si le déploiement est limité à deux nœuds uniquement, vous avez deux possibilités pour la connectivité :

- Utilisation de l’écouteur de groupe de disponibilité. Avec l’écouteur de groupe de disponibilité, vous devez déployer un équilibreur de charge Azure. Il s’agit de la méthode de déploiement par défaut. Les applications SAP sont configurées pour se connecter dans l’écouteur de groupe de disponibilité et non dans un seul nœud.
- Utilisation des paramètres de connectivité de la mise en miroir de bases de données SQL Server. Dans ce cas, vous devez configurer la connectivité des applications SAP d’une manière où les deux noms de nœud sont désignés. Les détails précis de cette configuration côté SAP sont documentés dans la note SAP [n°965908](https://launchpad.support.sap.com/#/notes/965908). Si vous utilisez cette option, vous n’avez pas besoin de configurer un écouteur de groupe de disponibilité ni d’équilibreur de charge Azure pour la haute disponibilité de SQL Server. En conséquence, le temps de réponse du réseau entre la couche d’application SAP et la couche SGBD est inférieur, car le trafic entrant dans l’instance SQL Server n’est pas acheminé via l’équilibreur de charge Azure. Cependant, souvenez-vous que cette option fonctionne uniquement si vous limitez votre groupe de disponibilité pour couvrir deux instances. 

Très peu de clients tirent parti de la fonctionnalité SQL Server Always On pour garantir une récupération d’urgence supplémentaire entre les régions Azure. Plusieurs clients utilisent également la possibilité d’effectuer des sauvegardes à partir d’un réplica secondaire. 

## <a name="sql-server-transparent-data-encryption"></a>SQL Server Transparent Data Encryption
Plusieurs clients utilisent SQL Server [Transparent Data Encryption (TDE)](/sql/relational-databases/security/encryption/transparent-data-encryption) lorsqu’ils déploient leurs bases de données SQL Server SAP dans Azure. La fonctionnalité SQL Server TDE est entièrement prise en charge par SAP (voir la note SAP [n°1380493](https://launchpad.support.sap.com/#/notes/1380493)). 

### <a name="applying-sql-server-tde"></a>Application de SQL Server TDE
Dans les cas où vous effectuez une migration hétérogène à partir d’un autre système SGBD, qui s’exécute localement, vers Windows/SQL Server exécuté dans Azure, vous devez créer votre base de données cible vide dans SQL Server à l’avance. À l’étape suivante, vous devez appliquer la fonctionnalité SQL Server TDE tout en utilisant votre système de production en local. La raison pour laquelle vous souhaitez effectuer cette procédure dans cette séquence est que le processus de chiffrement de la base de données vide peut prendre beaucoup de temps. Les processus d’importation SAP importent ensuite les données dans la base de données chiffrée pendant la phase de temps d’arrêt. Le traitement de l’importation dans une base de données chiffrée a un impact sur le temps plus faible que le chiffrement de la base de données après la phase d’exportation au cours du temps d’arrêt. Des expériences ont été négatives lors de la tentative d’application du chiffrement TDE avec charge de travail SAP exécutée sur la base de données. Il est donc recommandé de traiter le déploiement de TDE en tant qu’activité devant être effectuée sans charge de travail SAP sur la base de données particulière.

Dans les cas où vous déplacez des bases de données SQL Server SAP d’un environnement local vers Azure, il est recommandé de vérifier sur quelle infrastructure vous pouvez obtenir le plus rapidement le chiffrement appliqué. Pour ce faire, gardez à l’esprit les points suivants :

- Vous ne pouvez pas définir le nombre de threads utilisés pour appliquer le chiffrement de données à la base de données. Le nombre de threads dépend principalement du nombre de volumes de disque sur lesquels les fichiers journaux et les fichiers de données SQL Server sont distribués. Cela signifie que plus les volumes sont distincts (lettres de lecteur), plus les threads sont engagés en parallèle pour effectuer le chiffrement. Une telle configuration est en légère contradiction avec la suggestion de configuration de disque indiquée plus haut, préconisant la création d’un ou de quelques espaces de stockage pour les fichiers de base de données SQL Server sur des machines virtuelles Azure. Une configuration comprenant un petit nombre de volumes conduirait à un petit nombre de threads exécutant le chiffrement. Un chiffrement à un seul thread lit les étendues de 64 ko, les chiffre, puis écrit un enregistrement dans le fichier journal de transactions, ce qui indique que l’étendue a été chiffrée. Par conséquent, la charge sur le journal des transactions est modérée.
- Dans les versions antérieures de SQL Server, la compression de sauvegarde n’était plus efficace lorsque vous aviez chiffré votre base de données SQL Server. Ce comportement pouvait se transformer en problème si vous envisagiez de chiffrer votre base de données SQL Server en local, puis de copier une sauvegarde dans Azure pour restaurer la base de données dans Azure. La compression de sauvegarde de SQL Server permet généralement d’obtenir un taux de compression de facteur 4.
- SQL Server 2016 introduit de nouvelles fonctionnalités qui permettent de compresser des bases de données chiffrées de manière efficace. Pour plus d’informations, consultez [ce blog](/archive/blogs/sqlcat/sqlsweet16-episode-1-backup-compression-for-tde-enabled-databases).
 
Lors de l’application du chiffrement TDE avec pas ou peu de charge de travail SAP uniquement, vous devez procéder à des tests dans votre configuration spécifique pour déterminer s’il est plus approprié d’appliquer le chiffrement TDE à votre base de données SAP en local ou de le faire dans Azure. Dans Azure, vous avez certainement plus de flexibilité en termes d’infrastructure de surapprovisionnement et réduisez l’infrastructure après l’application du chiffrement TDE.

### <a name="using-azure-key-vault"></a>Utilisation d’Azure Key Vault
Azure propose le service [Key Vault](https://azure.microsoft.com/services/key-vault/) pour stocker les clés de chiffrement. Parallèlement, SQL Server propose un connecteur pour tirer parti d’Azure Key Vault comme magasin pour les certificats TDE.

Vous trouverez plus d’informations sur l’utilisation d’Azure Key Vault pour SQL Server TDE dans les articles suivants :

- [Gestion de clés extensible à l’aide d’Azure Key Vault (SQL Server)](/sql/relational-databases/security/encryption/extensible-key-management-using-azure-key-vault-sql-server)
- [Gestion de clés extensible de SQL Server TDE avec Azure Key Vault - Étapes de configuration](/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault)
- [Résolution des problèmes et maintenance du connecteur SQL Server](/sql/relational-databases/security/encryption/sql-server-connector-maintenance-troubleshooting)
- [More Questions From Customers About SQL Server Transparent Data Encryption – TDE + Azure Key Vault](/archive/blogs/saponsqlserver/more-questions-from-customers-about-sql-server-transparent-data-encryption-tde-azure-key-vault) (Autres questions de clients sur SQL Server TDE et Azure Key Vault)


>[!IMPORTANT]
>En utilisant SQL Server TDE, notamment avec Azure Key Vault, il est recommandé d’utiliser les derniers correctifs de SQL Server 2014, SQL Server 2016 et SQL Server 2017. En effet, des optimisations et correctifs ont été appliqués au code suite aux commentaires de clients. À titre d’exemple, consultez l’article de la Base de connaissances [KBA n°4058175](https://support.microsoft.com/help/4058175/tde-enabled-backup-and-restore-slow-if-encryption-key-is-stored-in-ekm).
>  

## <a name="general-sql-server-for-sap-on-azure-summary"></a><a name="9053f720-6f3b-4483-904d-15dc54141e30"></a>Résumé – SQL Server général pour SAP sur Azure
Ce guide offre de nombreuses recommandations. Nous vous invitons à les parcourir plusieurs fois avant de planifier votre déploiement Azure. Cependant, de manière générale, vous devez suivre les principales recommandations générales propres à la fonction SGBD sur Azure :

1. Utilisez la dernière version du système SGBD, comme SQL Server 2017, qui présente les avantages les plus intéressants dans Azure. 
2. Planifiez avec soin votre paysage de système SAP dans Azure, afin de trouver l’équilibre entre la disposition des fichiers de données et les restrictions d’Azure :
   * Évitez d’utiliser un trop grand nombre de disques. Cependant, vous devez en configurer suffisamment pour atteindre le nombre d’E/S par seconde nécessaire.
   * Si vous n'utilisez pas la fonctionnalité Disques managés, n'oubliez pas que les E/S par seconde sont limitées pour chaque compte Stockage Azure, et que les comptes de stockage sont limités au sein de chaque abonnement Azure ([plus d'informations disponibles ici][azure-resource-manager/management/azure-subscription-service-limits]). 
   * N’effectuez une agrégation par bandes que si vous devez obtenir un débit supérieur.
3. N’installez pas vos logiciels et ne placez pas les fichiers nécessitant une persistance sur le lecteur D:\, car il n’est pas permanent. Les données placées sur ce lecteur ne sont pas conservées après le redémarrage de Windows.
4. N’utilisez pas la mise en cache de disque Azure pour le stockage Azure standard.
5. N’utilisez pas un compte de stockage Standard Azure géorépliqué.  Utilisez des systèmes localement redondants pour les charges de travail SGBD (système de gestion de base de données).
6. Utilisez la solution de haute disponibilité/récupération d’urgence de votre fournisseur SGBD (système de gestion de base de données) pour répliquer les données des bases de données.
7. Utilisez toujours la fonction de résolution de noms ; ne vous fiez pas aux adresses IP.
8. À l’aide de SQL Server TDE, appliquez les derniers correctifs de SQL Server.
9. Utilisez la fonctionnalité de compression de base de données la plus élevée possible. Il s’agit de la compression de page pour SQL Server.
10. Veillez à recourir à des images SQL Server de Microsoft Azure Marketplace. Si vous utilisez le serveur SQL numéro un, vous devez modifier le classement de l’instance avant d’installer un système SAP NetWeaver sur ce serveur.
11. Installez et configurez la surveillance d'hôte SAP pour Azure comme décrit dans le [Guide de déploiement][deployment-guide].


## <a name="next-steps"></a>Étapes suivantes
Lire l’article 

- [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md)