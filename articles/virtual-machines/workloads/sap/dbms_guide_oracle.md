---
title: Déploiement SGBD de machines virtuelles Oracle Azure pour charge de travail SAP | Microsoft Docs
description: Déploiement SGBD de machines virtuelles Oracle Azure pour charge de travail SAP
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP, Azure, Oracle, Data Guard
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/20/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 3e99b3a8960eb49856e9a016eb054eed41eccde9
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965253"
---
# <a name="azure-virtual-machines-oracle-dbms-deployment-for-sap-workload"></a>Déploiement SGBD Oracle de machines virtuelles Azure pour charge de travail SAP

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
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/resources/templates/sql-server-2014-alwayson-existing-vnet-and-ad/
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


Ce document décrit les différents points à prendre en compte quand vous déployez Oracle Database pour une charge de travail SAP dans Azure IaaS. Avant de lire ce document, nous vous recommandons de lire [Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour une charge de travail SAP](dbms_guide_general.md). Nous vous recommandons aussi de lire d’autres guides de la [documentation de la charge de travail SAP sur Azure](./get-started.md). 

Vous trouverez des informations sur les versions d’Oracle et les versions des systèmes d’exploitation correspondantes qui sont prises en charge pour l’exécution de SAP sur Oracle dans Azure dans la note SAP [2039619].

Pour obtenir des informations d’ordre général sur l’exécution de SAP Business Suite sur Oracle, consultez [SAP sur Oracle](https://www.sap.com/community/topic/oracle.html).
Les logiciels Oracle sont pris en charge par Oracle pour s’exécuter dans Microsoft Azure. Pour plus d’informations sur la prise en charge générale de Windows Hyper-V et d’Azure, consultez le [Forum aux questions sur Oracle et Microsoft Azure](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html). 

## <a name="sap-notes-relevant-for-oracle-sap-and-azure"></a>Notes SAP utiles pour Oracle, SAP et Azure 

Les notes SAP suivantes concernent SAP sur Azure.

| Numéro de la note | Intitulé |
| --- | --- |
| [1928533] |Applications SAP sur Azure : Produits et types de machines virtuelles pris en charge |
| [2015553] |SAP sur Microsoft Azure : Prérequis pour la prise en charge |
| [1999351] |Résolution des problèmes de la surveillance Azure améliorée pour SAP |
| [2178632] |Métriques de surveillance clés pour SAP sur Microsoft Azure |
| [2191498] |SAP sur Linux avec Azure : Surveillance améliorée |
| [2039619] |Applications SAP sur Microsoft Azure avec la base de données Oracle : Produits et versions pris en charge |
| [2243692] |Linux sur machine virtuelle Microsoft Azure (IaaS) : problèmes de licence SAP |
| [2069760] |Installation et mise à niveau de SAP pour Oracle Linux 7.x |
| [1597355] |Recommandations relatives à l’espace d’échange pour Linux |
| [2171857] |Oracle Database 12C - Prise en charge du système de fichiers dans Linux |
| [1114181] |Oracle Database 11g - Prise en charge du système de fichiers dans Linux |

Les configurations et les fonctionnalités exactes prises en charge par Oracle et SAP sur Azure sont documentées dans la note SAP [#2039619](https://launchpad.support.sap.com/#/notes/2039619).

Windows et Oracle Linux sont les seuls systèmes d’exploitation pris en charge par Oracle et SAP sur Azure. Les distributions Linux RHEL et SLES largement utilisées ne sont pas prises en charge pour le déploiement de composants Oracle dans Azure. Les composants Oracle incluent le client Oracle Database, utilisé par les applications SAP pour se connecter au SGBD Oracle. 

Les exceptions, d’après la note SAP [#2039619](https://launchpad.support.sap.com/#/notes/2039619), sont les composants SAP qui n’utilisent pas le client Oracle Database. Ces composants SAP sont les empileurs autonomes, serveurs de messages, services de réplication d’empilement, WebDispatcher et passerelles SAP.  

Même si vous exécutez vos instances du SGBD Oracle et d’applications SAP sur Oracle Linux, vous pouvez exécuter vos services centraux SAP sur SLES ou RHEL, et les protéger avec un cluster Pacemaker. Pacemaker en tant qu’infrastructure à haute disponibilité n’est pas pris en charge sur Oracle Linux.

## <a name="specifics-for-oracle-database-on-windows"></a>Spécificités d’Oracle Database sur Windows

### <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms-on-windows"></a>Instructions de configuration d’Oracle pour les installations de SAP dans des machines virtuelles Azure sur Windows

Conformément au manuel d’installation de SAP, aucun fichier lié à Oracle ne doit être installé ou placé sur le disque de système d’exploitation de la machine virtuelle (lecteur c:). Les machines virtuelles de différentes tailles peuvent prendre en charge un nombre variable de disques attachés. Les types de machines virtuelles plus petites peuvent prendre en charge un plus petit nombre de disques attachés. 

Si vous avez des machines virtuelles plus petites et que vous atteindriez la limite du nombre de disques que vous pouvez attacher à la machine virtuelle, vous pouvez installer/localiser Oracle home, stage, `saptrace`, `saparch`, `sapbackup`, `sapcheck` ou `sapreorg` sur le disque du système d’exploitation. Ces éléments des composants du SGBD Oracle n’ont pas un débit d’E/S trop élevé. Cela signifie que le disque du système d’exploitation peut faire face aux exigences en termes d’E/S. La taille par défaut du disque du système d’exploitation doit être de 127 Go. 

Les fichiers Oracle Database et les fichiers journaux de phase de restauration par progression doivent être stockés sur des disques de données distincts. Il existe une exception pour l’espace disque logique temporaire d’Oracle. `Tempfiles` peut être créé sur D:/ (lecteur non persistant). Le lecteur D:\ non persistant offre également un débit et une latence d’E/S améliorés (à l’exception des machines virtuelles de la série A). 

Pour déterminer la quantité appropriée d’espace pour les `tempfiles`, vous pouvez vérifier les tailles des `tempfiles` sur des systèmes existants.

### <a name="storage-configuration"></a>Configuration du stockage
Une seule instance d’Oracle utilisant des disques au format NTFS est prise en charge. Tous les fichiers de base de données doivent être stockés sur le système de fichiers NTFS sur la fonctionnalité Disques managés (recommandé) ou sur des disques durs virtuels. Ces disques sont montés sur la machine virtuelle Azure et sont basés sur le [stockage d’objets blob de pages Azure](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) ou sur [Azure Disques managés](../../managed-disks-overview.md). 

Consultez l’article [Types de stockage Azure pour une charge de travail SAP](./planning-guide-storage.md) pour obtenir plus de détails sur les types de stockage de bloc Azure spécifiques adaptés à la charge de travail SGBD.

Nous recommandons fortement l’utilisation d’[Azure Disques managés](../../managed-disks-overview.md). Nous recommandons également d’utiliser [le stockage Premium Azure ou un disque Azure Ultra](../../disks-types.md) pour vos déploiements d’Oracle Database.

Les lecteurs réseau ou les partages distants comme les services de fichiers Azure ne sont pas pris en charge pour les fichiers Oracle Database. Pour plus d'informations, consultez les pages suivantes :

- [Présentation de Microsoft Azure File Service](/archive/blogs/windowsazurestorage/introducing-microsoft-azure-file-service)

- [Conservation des connexions vers les fichiers Microsoft Azure](/archive/blogs/windowsazurestorage/persisting-connections-to-microsoft-azure-files)


Si vous utilisez des disques basés sur le stockage d’objets blob de pages Azure ou sur la fonctionnalité Disques managés, les instructions figurant dans [Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md) s’appliquent également aux déploiements avec Oracle Database.

Il existe des quotas sur le débit d’IOPS pour les disques Azure. Ce concept est expliqué dans [Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md). Les quotas exacts dépendent du type de machine virtuelle que vous utilisez. Pour obtenir la liste des types de machines virtuelles et des quotas correspondants, consultez [Tailles des machines virtuelles Windows dans Azure][virtual-machines-sizes-windows].

Pour connaître les types de machines virtuelles Azure pris en charge, consultez la note SAP [1928533].

La configuration minimale est la suivante : 

| Composant | Disque | Mise en cache | Pool de stockage |
| --- | ---| --- | --- |
| \oracle\<SID>\origlogaA & mirrlogB | Disque Premium ou Ultra | None | Inutile |
| \oracle\<SID>\origlogaB & mirrlogA | Disque Premium ou Ultra | None | Inutile |
| \oracle\<SID>\sapdata1...n | Disque Premium ou Ultra | Lecture seule | Peut être utilisé pour Premium |
| \oracle\<SID>\oraarch | Standard | None | Inutile |
| Oracle Home, `saptrace`, ... | Disque du système d’exploitation (Premium) | | Inutile |


La sélection des disques pour l’hébergement des journaux de phase de restauration par progression en ligne doit être motivée par les exigences d’IOPS. Il est possible de stocker tous les sapdata1...n (espaces disque logiques) sur un même disque monté tant que la taille, les IOPS et le débit répondent aux exigences. 

La configuration des performances est la suivante :

| Composant | Disque | Mise en cache | Pool de stockage |
| --- | ---| --- | --- |
| \oracle\<SID>\origlogaA | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium  |
| \oracle\<SID>\origlogaB | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| \oracle\<SID>\mirrlogAB | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| \oracle\<SID>\mirrlogBA | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| \oracle\<SID>\sapdata1...n | Disque Premium ou Ultra | Lecture seule | Recommandé pour Premium  |
| \oracle\SID\sapdata(n+1)* | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| \oracle\<SID>\oraarch* | Disque Premium ou Ultra | None | Inutile |
| Oracle Home, `saptrace`, ... | Disque du système d’exploitation (Premium) | Inutile |

*(n+1) : hébergement des espaces disque logiques SYSTEM, TEMP et UNDO. Le modèle d’E/S des espaces disques logiques System et Undo est différent des autres espaces disques logiques qui hébergent des données d’application. « Aucune mise en cache » est la meilleure option pour les performances des espaces disques logiques System et Undo.

*oraarch : un pool de stockage n’est pas nécessaire du point de vue des performances. Il peut être utilisé pour avoir davantage d’espace.

Si plus d’IOPS sont nécessaires dans le cas du stockage Premium Azure, nous recommandons d’utiliser des pools de stockage Windows (disponibles seulement dans Windows Server 2012 et versions ultérieures) afin de créer un seul grand périphérique logique sur plusieurs disques montés. Cette approche simplifie les tâches d’administration pour gérer l’espace disque et évite les tâches de distribution manuelles des fichiers sur plusieurs disques montés.


#### <a name="write-accelerator"></a>Accélérateur des écritures
Pour des machines virtuelles Azure de la série M, la latence d’écriture dans les journaux de phase de restauration par progression en ligne peut être réduite considérablement par rapport au stockage Premium Azure. Activez l’Accélérateur d’écriture Azure pour les disques (VHD) basés sur le Stockage Azure Premium qui sont utilisés pour les fichiers journaux de phase de restauration par progression en ligne. Pour plus d’informations, consultez [Accélérateur d’écriture](../../how-to-enable-write-accelerator.md). Ou utilisez un disque Azure Ultra pour le volume du journal de phase de restauration par progression en ligne.


### <a name="backuprestore"></a>Sauvegarde/restauration
Pour la fonctionnalité de sauvegarde/restauration, les outils SAP BR*Tools for Oracle sont pris en charge de la même façon que sur les systèmes d’exploitation Windows Server standard. Oracle Recovery Manager (RMAN) est également pris en charge pour les sauvegardes sur disque et les restaurations à partir du disque.

Vous pouvez aussi utiliser Sauvegarde Azure pour effectuer une sauvegarde de machine virtuelle cohérente du point de vue des applications. L’article [Planification de votre infrastructure de sauvegarde de machines virtuelles dans Azure](../../../backup/backup-azure-vms-introduction.md) explique comment Sauvegarde Azure utilise la fonctionnalité Windows VSS pour effectuer une sauvegarde cohérente du point de vue des applications. Les versions du SGBD Oracle prises en charge sur Azure par SAP peuvent tirer parti de la fonctionnalité VSS pour les sauvegardes. Pour plus d’informations, consultez la documentation Oracle [Basic Concepts of Database Backup and Recovery with VSS](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/ntqrf/basic-concepts-of-database-backup-and-recovery-with-vss.html#GUID-C085101B-237F-4773-A2BF-1C8FD040C701).


### <a name="high-availability"></a>Haute disponibilité
Oracle Data Guard est pris en charge à des fins de haute disponibilité et de récupération d’urgence. Pour bénéficier d’un basculement automatique dans Data Guard, vous devez utiliser FSFA (Fast-Start Failover). L’Observateur (FSFA) déclenche le basculement. Si vous n’utilisez pas FSFA, vous pouvez seulement utiliser une configuration de basculement manuel.

Pour plus d’informations sur la reprise d’activité pour les bases de données Oracle dans Azure, consultez [Récupération d’urgence pour une base de données Oracle Database 12c dans votre environnement Azure](../oracle/oracle-disaster-recovery.md).

### <a name="accelerated-networking"></a>Mise en réseau accélérée
Pour les déploiements d’Oracle sur Windows, nous recommandons fortement l’accélération réseau, comme décrit dans [Accélération réseau Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/). Consultez aussi les recommandations dans [Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md). 
### <a name="other"></a>Autres
[Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md) décrit d’autres concepts importants liés aux déploiements de machines virtuelles avec Oracle Database, notamment les groupes à haute disponibilité et la surveillance de SAP.

## <a name="specifics-for-oracle-database-on-oracle-linux"></a>Spécificités d’Oracle Database sur Oracle Linux
Les logiciels Oracle sont pris en charge par Oracle pour s’exécuter dans Microsoft Azure avec Oracle Linux comme système d’exploitation invité. Pour plus d’informations sur la prise en charge générale de Windows Hyper-V et d’Azure, consultez le [Forum aux questions sur Azure et Oracle](https://www.oracle.com/technetwork/topics/cloud/faq-1963009.html). 

Le scénario spécifique d’applications SAP exploitant des bases de données Oracle est également pris en charge. Les détails sont présentés dans la partie suivante du document.

### <a name="oracle-version-support"></a>Prise en charge des versions d’Oracle
Pour plus d’informations sur les versions d’Oracle et les versions de systèmes d’exploitation correspondantes prises en charge pour l’exécution de SAP sur Oracle dans des machines virtuelles Azure, consultez la note SAP [2039619].

Pour obtenir des informations d’ordre général sur l’exécution de SAP Business Suite sur Oracle, consultez la [page de la communauté SAP sur Oracle](https://www.sap.com/community/topic/oracle.html).

### <a name="oracle-configuration-guidelines-for-sap-installations-in-azure-vms-on-linux"></a>Instructions de configuration d’Oracle pour les installations de SAP dans des machines virtuelles Azure sur Linux

Conformément aux guides d’installation SAP, aucun fichier lié à Oracle ne doit être installé ou placé dans des pilotes système pour un disque de démarrage d’une machine virtuelle. Les différentes tailles de machines virtuelles prennent en charge un nombre variable de disques attachés. Les types de machines virtuelles plus petites peuvent prendre en charge un plus petit nombre de disques attachés. 

Dans ce cas, nous vous recommandons d’installer/localiser Oracle home, stage, `saptrace`, `saparch`, `sapbackup`, `sapcheck` ou `sapreorg` sur le disque de démarrage. Ces éléments des composants du SGBD Oracle n’ont pas un débit d’E/S élevé. Cela signifie que le disque du système d’exploitation peut faire face aux exigences en termes d’E/S. La taille par défaut du disque du système d’exploitation est de 30 Go. Vous pouvez étendre le disque de démarrage en utilisant le portail Azure, PowerShell ou Azure CLI. Une fois le disque de démarrage étendu, vous pouvez ajouter une partition supplémentaire pour les fichiers binaires d’Oracle.


### <a name="storage-configuration"></a>Configuration du stockage

Les systèmes de fichiers ext4, xfs ou Oracle ASM sont pris en charge pour les fichiers d’Oracle Database sur Azure. Tous les fichiers de base de données doivent être stockés sur ces systèmes de fichiers basés sur les disques durs virtuels ou la fonctionnalité Disques managés. Ces disques sont montés sur la machine virtuelle Azure et sont basés sur le [stockage d’objets blob de pages Azure](/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs) ou sur [Azure Disques managés](../../managed-disks-overview.md).

Pour les noyaux Oracle Linux UEK, UEK version 4 ou ultérieure est nécessaire afin de prendre en charge les [disques SSD Premium Azure](../../premium-storage-performance.md#disk-caching).

Consultez l’article [Types de stockage Azure pour une charge de travail SAP](./planning-guide-storage.md) pour obtenir plus de détails sur les types de stockage de bloc Azure spécifiques adaptés à la charge de travail SGBD.

Nous vous recommandons vivement d’utiliser des [disques managés Azure](../../managed-disks-overview.md). Nous vous recommandons également d'utiliser des [disques SSD Premium Azure](../../disks-types.md) pour vos déploiements Oracle Database.

Les lecteurs réseau ou les partages distants comme les services de fichiers Azure ne sont pas pris en charge pour les fichiers Oracle Database. Pour plus d’informations, consultez les rubriques suivantes : 

- [Présentation de Microsoft Azure File Service](/archive/blogs/windowsazurestorage/introducing-microsoft-azure-file-service)

- [Conservation des connexions vers les fichiers Microsoft Azure](/archive/blogs/windowsazurestorage/persisting-connections-to-microsoft-azure-files)

Si vous utilisez des disques basés sur le stockage d’objets blob de pages Azure ou la fonctionnalité Disques managés, les instructions figurant dans [Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md) s’appliquent également aux déploiements avec Oracle Database.

Il existe des quotas sur le débit d’IOPS pour les disques Azure. Ce concept est expliqué dans [Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md). Les quotas exacts dépendent du type de machine virtuelle qui est utilisé. Pour obtenir la liste des types de machines virtuelles et des quotas correspondants, consultez [Tailles des machines virtuelles Linux dans Azure][virtual-machines-sizes-linux].

Pour connaître les types de machines virtuelles Azure pris en charge, consultez la note SAP [1928533].

Configuration minimale :

| Composant | Disque | Mise en cache | Agrégation par bandes* |
| --- | ---| --- | --- |
| /oracle/\<SID>/origlogaA & mirrlogB | Disque Premium ou Ultra | None | Inutile |
| /oracle/\<SID>/origlogaB & mirrlogA | Disque Premium ou Ultra | None | Inutile |
| /oracle/\<SID>/sapdata1...n | Disque Premium ou Ultra | Lecture seule | Peut être utilisé pour Premium |
| /oracle/\<SID>/oraarch | Standard | None | Inutile |
| Oracle Home, `saptrace`, ... | Disque du système d’exploitation (Premium) | | Inutile |

*Agrégation par bandes : Volume LVM agrégé par bandes ou MDADM à l’aide de RAID0

La sélection de disque pour l’hébergement des journaux d’activité de phase de restauration par progression en ligne d’Oracle doit être motivée par les exigences d’IOPS. Il est possible de stocker tous les sapdata1...n (espaces disque logiques) sur un même disque monté tant que le volume, les IOPS et le débit répondent aux exigences. 

Configuration des performances :

| Composant | Disque | Mise en cache | Agrégation par bandes* |
| --- | ---| --- | --- |
| /oracle/\<SID>/origlogaA | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium  |
| /oracle/\<SID>/origlogaB | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| /oracle/\<SID>/mirrlogAB | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| /oracle/\<SID>/mirrlogBA | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| /oracle/\<SID>/sapdata1...n | Disque Premium ou Ultra | Lecture seule | Recommandé pour Premium  |
| /oracle/\<SID>/sapdata(n+1)* | Disque Premium ou Ultra | Aucun | Peut être utilisé pour Premium |
| /oracle/\<SID>/oraarch* | Disque Premium ou Ultra | None | Inutile |
| Oracle Home, `saptrace`, ... | Disque du système d’exploitation (Premium) | Inutile |

*Agrégation par bandes : Volume LVM agrégé par bandes ou MDADM à l’aide de RAID0

*(n+1) : hébergement des espaces disque logiques SYSTEM, TEMP et UNDO : Le modèle d’E/S des espaces disques logiques System et Undo est différent des autres espaces disques logiques qui hébergent des données d’application. « Aucune mise en cache » est la meilleure option pour les performances des espaces disques logiques System et Undo.

*oraarch : un pool de stockage n’est pas nécessaire du point de vue des performances.


Si vous avez besoin de plus d’IOPS en cas d’utilsiation du stockage Premium Azure, nous recommandons d’utiliser LVM (Logical Volume Manager) ou MDADM pour créer un seul grand volume logique sur plusieurs disques montés. Pour plus d’informations, consultez [Facteurs à prendre en compte pour le déploiement de SGBD sur des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md) pour des instructions et des liens sur la façon de tirer parti de LVM ou de MDADM. Cette approche simplifie les tâches d’administration pour gérer l’espace disque et évite les tâches de distribution manuelles des fichiers sur plusieurs disques montés.


#### <a name="write-accelerator"></a>Accélérateur des écritures
Pour des machines virtuelles Azure de la série M, quand vous utilisez l’Accélérateur d’écriture Azure, la latence d’écriture dans les journaux de phase de restauration par progression en ligne peut être réduite considérablement lors de l’utilisation du stockage Premium Azure. Activez l’Accélérateur d’écriture Azure pour les disques (VHD) basés sur le Stockage Azure Premium qui sont utilisés pour les fichiers journaux de phase de restauration par progression en ligne. Pour plus d’informations, consultez [Accélérateur d’écriture](../../how-to-enable-write-accelerator.md). Ou utilisez un disque Azure Ultra pour le volume du journal de phase de restauration par progression en ligne.


### <a name="backuprestore"></a>Sauvegarde/restauration
Pour la fonctionnalité de sauvegarde/restauration, les outils SAP BR*Tools for Oracle sont pris en charge de la même façon que sur les systèmes nus et Hyper-V. Oracle Recovery Manager (RMAN) est également pris en charge pour les sauvegardes sur disque et les restaurations à partir du disque.

Pour plus d’informations sur l’utilisation des services Sauvegarde Azure et Récupération Azure pour sauvegarder et récupérer des bases de données Oracle, consultez [Sauvegarder et récupérer une base de données de la base de données Oracle Database 12c sur une machine virtuelle Linux Azure](../oracle/oracle-overview.md).

### <a name="high-availability"></a>Haute disponibilité
Oracle Data Guard est pris en charge à des fins de haute disponibilité et de récupération d’urgence. Pour bénéficier d’un basculement automatique dans Data Guard, vous devez utiliser FSFA (Fast-Start Failover). La fonctionnalité d’Observateur (FSFA) déclenche le basculement. Si vous n’utilisez pas FSFA, vous pouvez seulement utiliser une configuration de basculement manuel. Pour plus d’informations, consultez [Implémenter Oracle Data Guard sur une machine virtuelle Azure Linux](../oracle/configure-oracle-dataguard.md).


Les aspects de la récupération d'urgence pour les bases de données Oracle dans Azure sont présentés dans l’article [Récupération d'urgence d’une base de données Oracle Database 12c dans un environnement Azure](../oracle/oracle-disaster-recovery.md).

### <a name="accelerated-networking"></a>Mise en réseau accélérée
La prise en charge de la mise en réseau accélérée Azure dans Oracle Linux est fournie avec oracle Linux 7 Update 5 (Oracle Linux 7.5). Si vous ne parvenez pas à mettre à niveau vers la dernière version Oracle Linux 7.5, vous pouvez utiliser le noyau RHCK (RedHat Compatible Kernel) à la place du noyau Oracle UEK. 

L’utilisation du noyau RHEL dans Oracle Linux est prise en charge selon la Note SAP [#1565179](https://launchpad.support.sap.com/#/notes/1565179). Pour l’Accélération réseau Azure, la version minimale du noyau RHCKL doit être 3.10.0-862.13.1.el7. Si vous utilisez le noyau UEK dans Oracle Linux conjointement avec l’[Accélération réseau Azure](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/), vous devez utiliser le noyau Oracle UEK version 5.

Si vous ne déployez pas des machines virtuelles à partir d’une image qui n’est pas basée sur la Place de marché Azure, vous devez copier des fichiers de configuration supplémentaires sur la machine virtuelle en exécutant le code suivant : 
<pre><code># Copy settings from GitHub to the correct place in the VM
sudo curl -so /etc/udev/rules.d/68-azure-sriov-nm-unmanaged.rules https://raw.githubusercontent.com/LIS/lis-next/master/hv-rhel7.x/hv/tools/68-azure-sriov-nm-unmanaged.rules 
</code></pre>


## <a name="next-steps"></a>Étapes suivantes
Lire l’article 

- [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](dbms_guide_general.md)
