---
title: Déploiement de machines virtuelles Azure pour SAP NetWeaver | Microsoft Docs
description: Apprenez à déployer des logiciels SAP sur des machines virtuelles Linux dans Azure.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 1c4f1951-3613-4a5a-a0af-36b85750c84e
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/16/2020
ms.author: sedusch
ms.openlocfilehash: d57512d631685f1f8da7dcd22181bf4d4223937f
ms.sourcegitcommit: 02ed9acd4390b86c8432cad29075e2204f6b1bc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97807567"
---
# <a name="azure-virtual-machines-deployment-for-sap-netweaver"></a>Déploiement de machines virtuelles Azure pour SAP NetWeaver

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
[2367194]:https://launchpad.support.sap.com/#/notes/2367194

[azure-cli]:../../../cli-install-nodejs.md
[azure-cli-2]:https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-portal]:https://portal.azure.com
[azure-ps]:/powershell/azure/
[azure-quickstart-templates-github]:https://github.com/Azure/azure-quickstart-templates
[azure-script-ps]:https://go.microsoft.com/fwlink/p/?LinkID=395017
[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#subscription-limits

[dbms-guide]:dbms-guide.md (Déploiement SGBD de machines virtuelles Azure pour SAP)
[dbms-guide-2.1]:dbms-guide.md#c7abf1f0-c927-4a7c-9c1d-c7b5b3b7212f (Mise en cache pour les machines virtuelles et les disques durs virtuels)
[dbms-guide-2.2]:dbms-guide.md#c8e566f9-21b7-4457-9f7f-126036971a91 (RAID logiciel)
[dbms-guide-2.3]:dbms-guide.md#10b041ef-c177-498a-93ed-44b3441ab152 (Stockage Microsoft Azure)
[dbms-guide-2]:dbms-guide.md#65fa79d6-a85f-47ee-890b-22e794f51a64 (Structure d’un déploiement SGBDR)
[dbms-guide-3]:dbms-guide.md#871dfc27-e509-4222-9370-ab1de77021c3 (Haute disponibilité et récupération d’urgence avec les machines virtuelles Azure)
[dbms-guide-5.5.1]:dbms-guide.md#0fef0e79-d3fe-4ae2-85af-73666a6f7268 (SQL Server 2012 SP1 CU4 et versions ultérieures)
[dbms-guide-5.5.2]:dbms-guide.md#f9071eff-9d72-4f47-9da4-1852d782087b (SQL Server 2012 SP1 CU3 et versions antérieures)
[dbms-guide-5.6]:dbms-guide.md#1b353e38-21b3-4310-aeb6-a77e7c8e81c8 (Utilisation d’une image SQL Server à partir d’Azure Marketplace)
[dbms-guide-5.8]:dbms-guide.md#9053f720-6f3b-4483-904d-15dc54141e30 (Résumé – SQL Server général pour SAP sur Azure)
[dbms-guide-5]:dbms-guide.md#3264829e-075e-4d25-966e-a49dad878737 (Éléments spécifiques au SGBDR SQL Server)
[dbms-guide-8.4.1]:dbms-guide.md#b48cfe3b-48e9-4f5b-a783-1d29155bd573 (Configuration du stockage)
[dbms-guide-8.4.2]:dbms-guide.md#23c78d3b-ca5a-4e72-8a24-645d141a3f5d (Sauvegarde et restauration)
[dbms-guide-8.4.3]:dbms-guide.md#77cd2fbb-307e-4cbf-a65f-745553f72d2c (Considérations sur les performances de sauvegarde et de restauration)
[dbms-guide-8.4.4]:dbms-guide.md#f77c1436-9ad8-44fb-a331-8671342de818 (Autres)
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

[deployment-guide]:deployment-guide.md (Déploiement de machines virtuelles Azure pour SAP)
[deployment-guide-2.2]:deployment-guide.md#42ee2bdb-1efc-4ec7-ab31-fe4c22769b94 (Ressources SAP)
[deployment-guide-3.1.2]:deployment-guide.md#3688666f-281f-425b-a312-a77e7db2dfab (Déploiement d’une machine virtuelle à l’aide d’une image personnalisée)
[deployment-guide-3.2]:deployment-guide.md#db477013-9060-4602-9ad4-b0316f8bb281 (Scénario 1 : Déploiement d’une machine virtuelle à partir d’Azure Marketplace pour SAP)
[deployment-guide-3.3]:deployment-guide.md#54a1fc6d-24fd-4feb-9c57-ac588a55dff2 (Scénario 2 : Déploiement d’une machine virtuelle avec une image personnalisée pour SAP)
[deployment-guide-3.4]:deployment-guide.md#a9a60133-a763-4de8-8986-ac0fa33aa8c1 (Scénario 3 : Déplacement d’une machine virtuelle locale à l’aide d’un disque dur virtuel Azure non généralisé avec SAP)
[deployment-guide-3]:deployment-guide.md#b3253ee3-d63b-4d74-a49b-185e76c4088e (Scénarios de déploiement de machines virtuelles pour SAP sur Microsoft Azure)
[deployment-guide-4.1]:deployment-guide.md#604bcec2-8b6e-48d2-a944-61b0f5dee2f7 (Déploiement d’applets de commande Azure PowerShell)
[deployment-guide-4.2]:deployment-guide.md#7ccf6c3e-97ae-4a7a-9c75-e82c37beb18e (Téléchargement et importation des applets de commande PowerShell pertinents pour SAP)
[deployment-guide-4.3]:deployment-guide.md#31d9ecd6-b136-4c73-b61e-da4a29bbc9cc (Joindre une machine virtuelle à un domaine local - Windows uniquement)
[deployment-guide-4.4.2]:deployment-guide.md#6889ff12-eaaf-4f3c-97e1-7c9edc7f7542 (Linux)
[deployment-guide-4.4]:deployment-guide.md#c7cbb0dc-52a4-49db-8e03-83e7edc2927d (Télécharger, installer et activer l’agent de machine virtuelle Azure)
[deployment-guide-4.5.1]:deployment-guide.md#987cf279-d713-4b4c-8143-6b11589bb9d4 (Azure PowerShell)
[deployment-guide-4.5.2]:deployment-guide.md#408f3779-f422-4413-82f8-c57a23b4fc2f (Interface de ligne de commande Azure)
[deployment-guide-4.5]:deployment-guide.md#d98edcd3-f2a1-49f7-b26a-07448ceb60ca (Configurer l’extension Azure pour SAP)
[deployment-guide-configure-new-extension-ps]:deployment-guide.md#2ad55a0d-9937-4943-9dd2-69bc2b5d3de0 (Configuration de la nouvelle extension Azure pour SAP avec Azure PowerShell)
[deployment-guide-configure-new-extension-cli]:deployment-guide.md#c8749c24-fada-42ad-b114-f9aae2dc37da (Configuration de la nouvelle extension Azure pour SAP avec Azure CLI)
[deployment-guide-5.1]:deployment-guide.md#bb61ce92-8c5c-461f-8c53-39f5e5ed91f2 (Vérification de la disponibilité de l’extension Azure pour SAP)
[deployment-guide-5.1-new]:deployment-guide.md#7bf24f59-7347-4c7a-b094-4693e4687ee5 (Vérification de la préparation à la nouvelle extension Azure pour SAP)
[deployment-guide-5.2]:deployment-guide.md#e2d592ff-b4ea-4a53-a91a-e5521edb6cd1 (Contrôle d’intégrité de la configuration de l’extension Azure pour SAP)
[deployment-guide-5.2-new]:deployment-guide.md#464ac96d-7d3c-435d-a5ae-3faf3bfef4b3 (Contrôle d’intégrité de la configuration de la nouvelle extension Azure pour SAP)
[deployment-guide-5.3]:deployment-guide.md#fe25a7da-4e4e-4388-8907-8abc2d33cfd8 (Résolution des problèmes liés à l’extension Azure pour SAP)
[deployment-guide-5.3-new]:deployment-guide.md#b7afb8ef-a64c-495d-bb37-2af96688c530 (Résolution des problèmes liés à la nouvelle extension Azure pour SAP)
[deployment-guide-contact-support]:deployment-guide.md#3ba34cfc-c9bb-4648-9c3c-88e8b9130ca2 (Résolution des problèmes liés à l’extension Azure pour SAP – Contacter le support)
[deployment-guide-run-the-script]:deployment-guide.md#0d2847ad-865d-4a4c-a405-f9b7baaa00c7 (Résolution des problèmes liés à l’extension Azure pour SAP – Exécuter le script d’installation)
[deployment-guide-redeploy-after-sysprep]:deployment-guide.md#2cd61f22-187d-42ed-bb8c-def0c983d756 (Résolution des problèmes liés à l’extension Azure pour SAP – Redéployer après Sysprep)
[deployment-guide-fix-internet-connection]:deployment-guide.md#e92bc57d-80d9-4a2b-a2f4-16713a22ad89 (Résolution des problèmes liés à l’extension Azure pour SAP – Réparer la connexion Internet)


[deployment-guide-configure-monitoring-scenario-1]:deployment-guide.md#ec323ac3-1de9-4c3a-b770-4ff701def65b (Configurer une extension de machine virtuelle)
[deployment-guide-configure-proxy]:deployment-guide.md#baccae00-6f79-4307-ade4-40292ce4e02d (Configurer le proxy)
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
[deployment-guide-troubleshooting-chapter]:deployment-guide.md#564adb4f-5c95-4041-9616-6635e83a810b (Vérifications et résolution des problèmes)

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

[msdn-set-Azvmaemextension]:https://docs.microsoft.com/powershell/module/az.compute/set-azvmaemextension

[planning-guide]:planning-guide.md (Planification et implémentation de machines virtuelles Azure pour SAP)
[planning-guide-1.2]:planning-guide.md#e55d1e22-c2c8-460b-9897-64622a34fdff (Ressources)
[planning-guide-11]:planning-guide.md#7cf991a1-badd-40a9-944e-7baae842a058 (Haute disponibilité et récupération d’urgence pour SAP NetWeaver s’exécutant sur des machines virtuelles Azure)
[planning-guide-11.4.1]:planning-guide.md#5d9d36f9-9058-435d-8367-5ad05f00de77 (Haute disponibilité pour les serveurs d’applications SAP)
[planning-guide-11.5]:planning-guide.md#4e165b58-74ca-474f-a7f4-5e695a93204f (Utilisation du démarrage automatique pour les instances SAP)
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803 (Cloud uniquement : déploiement de machines virtuelles dans Azure sans dépendances du réseau local du client)
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10 (Entre sites - déploiement d’une ou plusieurs machines virtuelles SAP dans Azure entièrement intégrées avec le réseau local)
[planning-guide-3.1]:planning-guide.md#be80d1b9-a463-4845-bd35-f4cebdb5424a (Régions Azure)
[planning-guide-3.2.1]:planning-guide.md#df49dc09-141b-4f34-a4a2-990913b30358 (Domaines d’erreur)
[planning-guide-3.2.2]:planning-guide.md#fc1ac8b2-e54a-487c-8581-d3cc6625e560 (Domaines de mise à niveau)
[planning-guide-3.2.3]:planning-guide.md#18810088-f9be-4c97-958a-27996255c665 (Groupes à haute disponibilité Azure)
[planning-guide-3.2]:planning-guide.md#8d8ad4b8-6093-4b91-ac36-ea56d80dbf77 (Concept des machines virtuelles Microsoft Azure)
[planning-guide-5.1.1]:planning-guide.md#4d175f1b-7353-4137-9d2f-817683c26e53 (Déplacement d’une machine virtuelle locale vers Azure avec un disque non généralisé)
[planning-guide-5.1.2]:planning-guide.md#e18f7839-c0e2-4385-b1e6-4538453a285c (Déploiement d’une machine virtuelle avec une image spécifique du client)
[planning-guide-5.2.1]:planning-guide.md#1b287330-944b-495d-9ea7-94b83aff73ef (Préparation du déplacement d’une machine virtuelle locale vers Azure avec un disque non généralisé)
[planning-guide-5.2.2]:planning-guide.md#57f32b1c-0cba-4e57-ab6e-c39fe22b6ec3 (Préparation du déploiement d’une machine virtuelle avec une image spécifique du client pour SAP)
[planning-guide-5.2]:planning-guide.md#6ffb9f41-a292-40bf-9e70-8204448559e7 (Préparation de machines virtuelles avec SAP pour Azure)
[planning-guide-5.3.1]:planning-guide.md#6e835de8-40b1-4b71-9f18-d45b20959b79 (Différence entre un disque Azure et une image Azure)
[planning-guide-5.3.2]:planning-guide.md#a43e40e6-1acc-4633-9816-8f095d5a7b6a (Chargement d’un disque dur virtuel local vers Azure)
[planning-guide-5.4.2]:planning-guide.md#9789b076-2011-4afa-b2fe-b07a8aba58a1 (Copie de disques entre comptes Azure Storage)
[planning-guide-5.5.1]:planning-guide.md#4efec401-91e0-40c0-8e64-f2dceadff646 (Structure de machine virtuelle/disque dur virtuel pour les déploiements SAP)
[planning-guide-5.5.3]:planning-guide.md#17e0d543-7e8c-4160-a7da-dd7117a1ad9d (Paramétrage du montage automatique pour les disques attachés)
[planning-guide-9.1]:planning-guide.md#6f0a47f3-a289-4090-a053-2521618a28c3 (Solution de supervision Azure pour SAP)
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
[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd (Mise en réseau Microsoft Azure)
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f (Stockage : Stockage Microsoft Azure et disques de données)

[resource-group-authoring-templates]:../../../resource-group-authoring-templates.md
[resource-group-overview]:../../../azure-resource-manager/management/overview.md
[resource-groups-networking]:../../../networking/network-overview.md
[sap-pam]:https://support.sap.com/pam (Tableau de disponibilité des produits SAP)
[sap-templates-2-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-2-tier-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-marketplace-image-md%2Fazuredeploy.json
[sap-templates-2-tier-os-disk]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk%2Fazuredeploy.json
[sap-templates-2-tier-os-disk-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-disk-md%2Fazuredeploy.json
[sap-templates-2-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image%2Fazuredeploy.json
[sap-templates-2-tier-user-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-2-tier-user-image-md%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image%2Fazuredeploy.json
[sap-templates-3-tier-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-md%2Fazuredeploy.json
[sap-templates-3-tier-user-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image%2Fazuredeploy.json
[sap-templates-3-tier-user-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-user-image-md%2Fazuredeploy.json
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
[virtual-machines-Az-versus-azuresm]:virtual-machines-linux-compare-deployment-models.md
[virtual-machines-windows-classic-configure-oracle-data-guard]:../../virtual-machines-windows-classic-configure-oracle-data-guard.md
[virtual-machines-linux-cli-deploy-templates]:../../linux/cli-deploy-templates.md (Déploiement et gestion de machines virtuelles à l’aide des modèles Azure Resource Manager et de l’interface de ligne de commande Azure)
[virtual-machines-deploy-rmtemplates-powershell]:../../virtual-machines-windows-ps-manage.md (Gestion des machines virtuelles à l’aide de modèles Azure Resource Manager et de PowerShell)
[virtual-machines-windows-agent-user-guide]:../../extensions/agent-windows.md
[virtual-machines-linux-agent-user-guide]:../../extensions/agent-linux.md
[virtual-machines-linux-agent-user-guide-command-line-options]:../../extensions/agent-linux.md#command-line-options
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
[virtual-machines-manage-availability]:../../linux/manage-availability.md
[virtual-machines-ps-create-preconfigure-windows-resource-manager-vms]:../../windows/quick-create-powershell.md
[virtual-machines-sizes]:../../linux/sizes.md
[virtual-machines-windows-classic-ps-sql-alwayson-availability-groups]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-alwayson-availability-groups.md
[virtual-machines-windows-classic-ps-sql-int-listener]:./../../windows/sqlclassic/virtual-machines-windows-classic-ps-sql-int-listener.md
[virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions]:../../../azure-sql/virtual-machines/windows/business-continuity-high-availability-disaster-recovery-hadr-overview.md
[virtual-machines-sql-server-infrastructure-services]:../../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md
[virtual-machines-sql-server-performance-best-practices]:../../../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md
[virtual-machines-upload-image-windows-resource-manager]:../../windows/upload-image.md
[virtual-machines-windows-tutorial]:../../windows/quick-create-portal.md
[virtual-machines-workload-template-sql-alwayson]:https://azure.microsoft.com/documentation/templates/sql-server-2014-alwayson-dsc/
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
[qs-configure-powershell-windows-vm]:../../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md
[qs-configure-cli-windows-vm]:../../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md
[howto-assign-access-powershell]:../../../active-directory/managed-identities-azure-resources/howto-assign-access-powershell.md
[howto-assign-access-cli]:../../../active-directory/managed-identities-azure-resources/howto-assign-access-cli.md

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-rm-include.md)]

Machines virtuelles Azure est la solution idéale pour les organisations qui ont besoin de ressources de calcul et de stockage rapidement, sans longs cycles d’acquisition. Vous pouvez utiliser le service Machines virtuelles Azure pour déployer des applications classiques, comme les applications basées sur SAP NetWeaver, dans Azure. Élargissez la fiabilité et la disponibilité d’une application sans ressources locales supplémentaires. Comme le service Machines virtuelles Azure prend en charge la connectivité intersite, vous pouvez intégrer les machines virtuelles Azure dans les domaines locaux, les clouds privés et le paysage SAP de votre organisation.

Dans cet article, nous abordons les étapes permettant de déployer des applications SAP sur des machines virtuelles dans Azure, en proposant notamment des options de déploiement alternatives et des solutions de dépannage. Cet article s’appuie sur les informations de [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver][planning-guide]. Il vient également compléter la documentation d’installation de SAP et les notes SAP, principales ressources pour l’installation et le déploiement des logiciels SAP.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

La configuration d’une machine virtuelle Azure pour le déploiement des logiciels SAP implique plusieurs étapes et diverses ressources. Avant de commencer, vérifiez que vous disposez des prérequis pour l’installation de logiciels SAP sur des machines virtuelles dans Azure.

### <a name="local-computer"></a>Ordinateur local

Pour gérer les machines virtuelles Windows ou Linux, vous pouvez utiliser un script PowerShell et le portail Azure. Dans les deux cas, vous devez disposer d’un ordinateur local exécutant Windows 7 ou une version ultérieure de Windows. Si vous souhaitez gérer uniquement des machines virtuelles Linux et que vous souhaitez utiliser un ordinateur Linux pour cette tâche, vous pouvez utiliser l’interface de ligne de commande Azure.

### <a name="internet-connection"></a>Connexion Internet

Pour télécharger et exécuter les outils et scripts requis pour le déploiement des logiciels SAP, vous devez être connecté à Internet. La machine virtuelle Azure exécutant l’extension Azure pour SAP doit également être connectée à Internet. Si la machine virtuelle Azure fait partie d’un réseau virtuel Azure ou d’un domaine local, assurez-vous que les paramètres de proxy appropriés sont configurés, comme décrit dans [Configurer le proxy][deployment-guide-configure-proxy].

### <a name="microsoft-azure-subscription"></a>Abonnement Microsoft Azure

Vous avez besoin d’un compte Azure actif.

### <a name="topology-and-networking"></a>Topologie et mise en réseau

Vous devez définir la topologie et l’architecture du déploiement SAP dans Azure :

* Comptes de stockage Azure à utiliser
* Réseau virtuel dans lequel déployer le système SAP
* Groupe de ressources dans lequel déployer le système SAP
* Région Azure dans laquelle déployer le système SAP
* Configuration SAP (à deux ou trois niveaux)
* Taille des machines virtuelles et nombre de disques de données supplémentaires à monter sur les machines virtuelles
* Configuration du système de transport et correction SAP (CTS)

Créez et configurez des comptes de stockage Azure (si nécessaire) ou des réseaux virtuels Azure avant de lancer le processus de déploiement de logiciels SAP. Pour plus d’informations sur la création et la configuration de ces ressources, consultez [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver][planning-guide].

### <a name="sap-sizing"></a>Dimensionnement SAP

Pour le dimensionnement SAP, vous devez connaître les informations suivantes :

* Charge de travail SAP projetée, par exemple, en utilisant l’outil SAP Quick Sizer, et nombre de SAPS (SAP Application Performance Standard)
* Consommation de mémoire et de ressources de processeur requise du système SAP
* Opérations d’entrée/sortie (E/S) par seconde requises
* Bande passante réseau requise pour la communication éventuelle entre les différentes machines virtuelles dans Azure
* Bande passante réseau requise entre les ressources locales et les systèmes SAP Azure déployés

### <a name="resource-groups"></a>Groupes de ressources

Dans Azure Resource Manager, vous pouvez utiliser des groupes de ressources pour gérer l’ensemble des ressources d’application de votre abonnement Azure. Pour plus d’informations, consultez [Présentation d’Azure Resource Manager][resource-group-overview].

## <a name="resources"></a>Ressources

### <a name="sap-resources"></a><a name="42ee2bdb-1efc-4ec7-ab31-fe4c22769b94"></a>Ressources SAP

Lorsque vous configurez votre déploiement de logiciels SAP, vous avez besoin des ressources SAP suivantes :

* Note SAP [1928533], qui contient :
  * une liste des tailles de machines virtuelles Azure prises en charge pour le déploiement de logiciels SAP
  * des informations importantes sur la capacité en fonction de la taille des machines virtuelles Azure
  * les logiciels SAP pris en charge et les combinaisons entre système d’exploitation et base de données
  * la version du noyau SAP requise pour Windows et Linux sur Microsoft Azure

* La note SAP [2015553] répertorie les conditions préalables au déploiement de logiciels SAP pris en charge par SAP sur Azure.
* La note SAP [2178632] contient des informations détaillées sur toutes les métriques de surveillance rapportées pour SAP sur Azure.
* La note SAP [1409604] contient la version requise de l’agent hôte SAP pour Windows sur Azure.
* La note SAP [2191498] contient la version requise de l’agent hôte SAP pour Linux sur Azure.
* La note SAP [2243692] contient des informations sur les licences SAP sur Linux dans Azure.
* La note SAP [1984787] contient des informations sur SUSE Linux Enterprise Server 12.
* La note SAP [2002167] contient des informations sur Red Hat Enterprise Linux 7.x.
* La note SAP [2069760] contient des informations générales sur Oracle Linux 7.x.
* La note SAP [1999351] contient des informations complémentaires de résolution des problèmes liés à l’extension Azure pour SAP.
* La note SAP [1597355] contient des informations générales sur l’espace d’échange pour Linux.
* La [page SAP sur Microsoft Azure](https://wiki.scn.sap.com/wiki/x/Pia7Gg) comprend des actualités, ainsi qu’une collection de ressources utiles.
* Le [WIKI de la communauté SAP](https://wiki.scn.sap.com/wiki/display/HOME/SAPonLinuxNotes) contient toutes les notes SAP requises pour Linux.
* Applets de commande PowerShell spécifiques à SAP qui font partie d’[Azure PowerShell][azure-ps].
* Commandes d’interface de ligne de commande Azure spécifiques à SAP faisant partie de l’[interface de ligne de commande Azure][azure-cli].

### <a name="windows-resources"></a><a name="42ee2bdb-1efc-4ec7-ab31-fe4c22769b94"></a>Ressources Windows

Ces articles Microsoft couvrent les déploiements SAP dans Azure :

* [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP NetWeaver (le présent article)][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP NetWeaver][dbms-guide]

## <a name="deployment-scenarios-for-sap-software-on-azure-vms"></a><a name="b3253ee3-d63b-4d74-a49b-185e76c4088e"></a>Scénarios de déploiement de logiciels SAP sur des machines virtuelles Azure

Vous avez plusieurs possibilités pour déployer des machines virtuelles et les disques associés dans Azure. Il est important de comprendre les différences entre les options de déploiement, car les étapes de préparation de vos machines virtuelles pour le déploiement dépendront du type de déploiement que vous choisirez.

### <a name="scenario-1-deploying-a-vm-from-the-azure-marketplace-for-sap"></a><a name="db477013-9060-4602-9ad4-b0316f8bb281"></a>Scénario 1 : Déploiement d’une machine virtuelle à partir d’Azure Marketplace pour SAP

Vous pouvez utiliser une image fournie par Microsoft ou par un tiers dans Azure Marketplace pour déployer votre machine virtuelle. Marketplace propose certaines images standard de Windows Server et de différentes distributions de Linux. Vous pouvez également déployer une image qui inclut les SKU du système de gestion de la base de données (SGBD), par exemple Microsoft SQL Server. Pour plus d’informations sur l’utilisation des images avec les SKU SGBD, consultez [Déploiement SGBD de machines virtuelles Azure pour SAP NetWeaver][dbms-guide].

L’organigramme suivant illustre la séquence d’étapes spécifiques à SAP pour le déploiement d’une machine virtuelle à partir d’Azure Marketplace :

![Organigramme de déploiement de machines virtuelles pour les systèmes SAP à l’aide d’une image de machine virtuelle provenant d’Azure Marketplace][deployment-guide-figure-100]

#### <a name="create-a-virtual-machine-by-using-the-azure-portal"></a>Création d’une machine virtuelle à l’aide du portail Azure

Le moyen le plus simple de créer une machine virtuelle à l’aide d’une image provenant d’Azure Marketplace est d’utiliser le portail Azure.

1.  Atteindre <https://portal.azure.com/#create/hub>.  Ou, dans le menu du portail Azure, sélectionnez **+ Nouveau**.
1.  Sélectionnez **Calcul**, puis choisissez le type de système d’exploitation que vous souhaitez déployer. Par exemple, Windows Server 2012 R2, SUSE Linux Enterprise Server 12 (SLES 12) ou Red Hat Enterprise Linux 7.2 (RHEL 7.2) ou Oracle Linux 7.2. La liste affichée par défaut ne répertorie pas tous les systèmes d’exploitation pris en charge. Pour obtenir une liste complète, sélectionnez **Afficher tout**. Pour plus d’informations sur les systèmes d’exploitation pris en charge pour le déploiement de logiciels SAP, consultez la note SAP [1928533].
1.  Sur la page suivante, passez en revue les conditions générales.
1.  Dans la zone **Sélectionner un modèle de déploiement**, sélectionnez **Resource Manager**.
1.  Sélectionnez **Create** (Créer).

L’assistant vous guide pour configurer les paramètres requis pour créer la machine virtuelle, ainsi que toutes les ressources requises, telles que les interfaces réseau et les comptes de stockage. Voici certains exemples de paramètres :

1. **Paramètres de base**:
   * **Name** : Nom de la ressource (nom de la machine virtuelle).
   * **Type de disque de machine virtuelle** : Sélectionnez le type du disque du système d’exploitation. Si vous souhaitez utiliser un stockage Premium pour vos disques de données, nous recommandons d’utiliser également le stockage Premium pour le disque du système d’exploitation.
   * **Nom utilisateur et mot de passe** ou **Clé publique SSH** : Entrez le nom d’utilisateur et le mot de passe de l’utilisateur créé lors de la configuration. Pour une machine virtuelle Linux, vous pouvez entrer la clé publique Secure Shell (SSH) que vous utilisez pour vous connecter à la machine.
   * **Abonnement**: Sélectionnez l’abonnement à utiliser pour configurer la nouvelle machine virtuelle.
   * **Groupe de ressources** : Nom du groupe de ressources de la machine virtuelle. Vous pouvez saisir soit le nom d’un nouveau groupe de ressources, soit celui d’un groupe de ressources existant.
   * **Emplacement** : Emplacement de déploiement de la nouvelle machine virtuelle. Si vous voulez connecter la machine virtuelle à votre réseau local, veillez à sélectionner l’emplacement du réseau virtuel qui connecte Azure à votre réseau local. Pour plus d’informations, consultez [Mise en réseau Microsoft Azure][planning-guide-microsoft-azure-networking] dans [SAP NetWeaver sur machines virtuelles Azure – Guide de planification et d’implémentation][planning-guide].
1. **Size** :

     Pour obtenir la liste des types de machine virtuelle pris en charge, consultez la note SAP [1928533]. Veillez à sélectionner le type de machine virtuelle approprié si vous voulez utiliser le stockage Premium Azure. Tous les types de machine virtuelle ne prennent pas en charge le stockage Premium. Pour plus d’informations, consultez [Stockage : Stockage Microsoft Azure et disques de données][planning-guide-storage-microsoft-azure-storage-and-data-disks] et [Stockage Azure pour les charges de travail SAP](./planning-guide-storage.md) dans [Planification et implémentation des Machines virtuelles Azure pour SAP NetWeaver][planning-guide].

1. **Paramètres**:
   * **Stockage**
     * **Type de disque** : Sélectionnez le type du disque du système d’exploitation. Si vous souhaitez utiliser un stockage Premium pour vos disques de données, nous recommandons d’utiliser également le stockage Premium pour le disque du système d’exploitation.
     * **Utiliser des disques managés** : Si vous souhaitez utiliser la fonctionnalité Disques managés, sélectionnez Oui. Pour plus d’informations sur la fonctionnalité Disques managés, lisez le chapitre [Disques managés](./planning-guide-storage.md#microsoft-azure-storage-resiliency) du guide de planification.
     * **Compte de stockage** : Sélectionnez un compte de stockage existant ou créez-en un. Notez que tous les types de stockage ne peuvent pas être utilisés pour l’exécution d’applications SAP. Pour plus d’informations sur les types de stockage, consultez [Structure de stockage d’une machine virtuelle pour les déploiements SGBDR](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64).
   * **Réseau**
     * **Réseau virtuel** et **Sous-réseau** : Pour intégrer la machine virtuelle à votre intranet, sélectionnez le réseau virtuel connecté à votre réseau local.
     * **Adresse IP publique** : Sélectionnez l’adresse IP publique que vous voulez utiliser ou entrez les paramètres nécessaires pour en créer une. Vous pouvez utiliser une adresse IP publique pour accéder à votre machine virtuelle via Internet. Assurez-vous également de créer un groupe de sécurité réseau pour sécuriser l’accès à votre machine virtuelle.
     * **Groupe de sécurité réseau** : Pour plus d’informations, consultez [Contrôler le flux de trafic réseau avec les groupes de sécurité réseau][virtual-networks-nsg].
   * **Extensions** : Vous pouvez installer des extensions de machine virtuelle en les ajoutant au déploiement. Il n’est pas nécessaire d’ajouter des extensions dans cette étape. Les extensions nécessaires à la prise en charge SAP sont installées ultérieurement. Consultez le chapitre [Configurer l’extension Azure pour SAP][deployment-guide-4.5] de ce guide.
   * **Haute disponibilité** : Sélectionnez un groupe à haute disponibilité ou entrez les paramètres permettant d’en créer un. Pour plus d’informations, consultez [Groupes à haute disponibilité Azure][planning-guide-3.2.3].
   * **Surveillance**
     * **Diagnostics de démarrage** : Vous pouvez sélectionner **Désactiver** pour le diagnostic de démarrage.
     * **Diagnostics du système d’exploitation invité** : Vous pouvez sélectionner **Désactiver** pour les diagnostics d’analyse.

1. **Résumé**:

   Passez en revue vos sélections, puis sélectionnez **OK**.

Votre machine virtuelle est déployée dans le groupe de ressources que vous avez sélectionné.

#### <a name="create-a-virtual-machine-by-using-a-template"></a>Création d’une machine virtuelle à l’aide d’un modèle

Vous pouvez créer une machine virtuelle à l’aide de l’un des modèles SAP publiés dans le [dépôt GitHub azure-quickstart-templates][azure-quickstart-templates-github]. Vous pouvez également créer une machine virtuelle manuellement à l’aide du [portail Azure][virtual-machines-windows-tutorial], de [PowerShell][virtual-machines-ps-create-preconfigure-windows-resource-manager-vms] ou d’[Azure CLI][virtual-machines-linux-tutorial].

* [**Modèle de configuration à deux niveaux (une seule machine virtuelle)** (sap-2-tier-user-disk)][sap-templates-2-tier-marketplace-image]

  Pour créer un système à deux niveaux avec une seule machine virtuelle, utilisez ce modèle.
* [**Modèle de configuration à deux niveaux (une seule machine virtuelle) - Disques managés** (sap-2-tier-marketplace-image-md)][sap-templates-2-tier-marketplace-image-md]

  Pour créer un système à deux niveaux avec une seule machine virtuelle et la fonctionnalité Disques managés, utilisez ce modèle.
* [**Modèle de configuration à trois niveaux (plusieurs machines virtuelles)** (sap-3-tier-marketplace-image)][sap-templates-3-tier-marketplace-image]

  Pour créer un système à trois niveaux avec plusieurs machines virtuelles, utilisez ce modèle.
* [**Modèle de configuration à trois niveaux (plusieurs machines virtuelles) - Disques managés** (sap-3-tier-marketplace-image-md)][sap-templates-3-tier-marketplace-image-md]

  Pour créer un système à trois niveaux avec plusieurs machines virtuelles et des disques managés, utilisez ce modèle.

Dans le portail Azure, entrez les paramètres suivants pour le modèle :

1. **Paramètres de base**:
   * **Abonnement**: L’abonnement à utiliser pour déployer le modèle.
   * **Groupe de ressources** : Le groupe de ressources à utiliser pour déployer le modèle. Vous pouvez créer un groupe de ressources ou sélectionner un groupe de ressources existant dans l’abonnement.
   * **Emplacement** : Emplacement où déployer le modèle. Si vous avez sélectionné un groupe de ressources existant, l’emplacement du groupe de ressources est utilisé.

1. **Paramètres**:
   * **ID du système SAP** : Identifiant du système SAP (SID).
   * **Type de système d’exploitation** : Le système d’exploitation que vous souhaitez déployer, par exemple, Windows Server 2012 R2, SUSE Linux Enterprise Server 12 (SLES 12) ou Red Hat Enterprise Linux 7.2 (RHEL 7.2) ou Oracle Linux 7.2.

     La liste affichée ne répertorie pas tous les systèmes d’exploitation pris en charge. Pour plus d’informations sur les systèmes d’exploitation pris en charge pour le déploiement de logiciels SAP, consultez la note SAP [1928533].
   * **Taille du système SAP** : Taille du système SAP.

     Le nombre de SAP fournis par le nouveau système. Si vous ne savez pas combien de SAP sont requis par le système, demandez à votre partenaire technologique SAP ou un intégrateur système.
   * **Disponibilité du système** (modèle à trois niveaux uniquement) : Disponibilité du système.

     Sélectionnez la haute disponibilité (**HA**) si la configuration est adaptée à une installation haute disponibilité. Deux serveurs de base de données et deux serveurs pour ABAP SAP Central Services (ASC) sont créés.
   * **Type de stockage** (modèle à deux niveaux uniquement) : Type de stockage à utiliser.

     Pour les grands systèmes, nous vous recommandons l’utilisation du stockage Premium Azure. Pour plus d’informations sur les types de stockage, consultez ces ressources :
      * [Utilisation du stockage SSD Azure Premium pour l’instance de SGBD SAP][2367194]
      * [Structure de stockage d’une machine virtuelle pour les déploiements SGBDR](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64)
      * [Stockage Premium : Stockage hautes performances pour les charges de travail de machine virtuelle Azure][storage-premium-storage-preview-portal]
      * [Introduction à Microsoft Azure Storage][storage-introduction]
   * **Nom d’utilisateur** et **Mot de passe administrateur** : Un nom d’utilisateur et un mot de passe.
     Un nouvel utilisateur est créé pour la connexion à la machine virtuelle.
   * **Sous-réseau nouveau ou existant** : Détermine si un réseau virtuel et un sous-réseau sont créés ou si un sous-réseau existant est utilisé. Si vous disposez déjà d’un réseau virtuel connecté à votre réseau local, sélectionnez **Existant**.
   * **ID de sous-réseau** : Si vous voulez déployer la machine virtuelle dans un réseau virtuel existant où vous avez défini un sous-réseau auquel la machine virtuelle doit être attribuée, nommez l’ID de ce sous-réseau spécifique. L’ID se présente généralement comme suit : /subscriptions/&lt;id d’abonnement>/resourceGroups/&lt;nom du groupe de ressources>/providers/Microsoft.Network/virtualNetworks/&lt;nom du réseau virtuel>/subnets/&lt;nom du sous-réseau>

1. **Conditions générales** :  
    Lisez et acceptez les conditions juridiques.

1. Sélectionnez **Achat**.

L’agent de la machine virtuelle Azure est déployé par défaut lorsque vous utilisez une image provenant d’Azure Marketplace.

#### <a name="configure-proxy-settings"></a>Configuration des paramètres de proxy

Selon la configuration de votre réseau local, vous devrez peut-être configurer le proxy sur votre machine virtuelle. Si votre machine virtuelle est connectée à votre réseau local via VPN ou ExpressRoute, la machine virtuelle ne sera peut-être pas en mesure d’accéder à Internet et de télécharger les extensions nécessaires ou de collecter les informations d’infrastructure Azure pour l’agent hôte SAP par le biais de l’extension SAP pour Azure. Pour plus d’informations, consultez [Configurer le proxy][deployment-guide-configure-proxy].

#### <a name="join-a-domain-windows-only"></a>Joindre un domaine (Windows uniquement)

Si votre déploiement Azure est connecté à une instance Active Directory ou DNS locale via une connexion VPN de site à site Azure ou via ExpressRoute (connexion *intersite* dans [SAP NetWeaver sur machines virtuelles Azure – Guide de planification et d’implémentation][planning-guide]), la machine virtuelle doit être jointe à un domaine local. Pour plus d’informations sur les éléments à prendre en compte pour cette tâche, consultez [Joindre une machine virtuelle à un domaine local (Windows uniquement)][deployment-guide-4.3].

#### <a name="configure-vm-extension"></a><a name="ec323ac3-1de9-4c3a-b770-4ff701def65b"></a>Configurer une extension de machine virtuelle

Pour être sûr que SAP est compatible avec votre environnement, configurez l’extension Azure pour SAP, comme décrit dans [Configurer l’extension Azure pour SAP][deployment-guide-4.5]. Consultez la configuration requise de SAP pour connaître les versions minimales requises pour le noyau SAP et l’agent hôte SAP dans les ressources listées dans [Ressources SAP][deployment-guide-2.2].

#### <a name="vm-extension-for-sap-check"></a>Vérification d’extension de machine virtuelle pour SAP

Vérifiez si l’extension de machine virtuelle pour SAP fonctionne, selon les modalités décrites dans [Vérifications et résolution des problèmes][deployment-guide-troubleshooting-chapter].

#### <a name="post-deployment-steps"></a>Étapes de post-déploiement

Après avoir créé et déployé la machine virtuelle, vous devez installer les composants logiciels requis dans la machine virtuelle. En raison de la séquence de déploiement/d’installation de logiciels dans ce type de déploiement de machine virtuelle, le logiciel à installer doit déjà être disponible, soit dans Azure, soit sur une autre machine virtuelle, ou en tant que disque qui peut être lié. Sinon, envisagez un scénario intersite dans lequel la connectivité aux ressources locales (partages d’installation) est acquise.

Une fois que vous avez déployé votre machine virtuelle dans Azure, utilisez les mêmes instructions et outils pour installer les logiciels SAP sur votre machine virtuelle comme vous le feriez dans un environnement local. Pour installer les logiciels SAP sur une machine virtuelle Azure, SAP et Microsoft recommandent de charger et stocker le support d’installation SAP sur des disques durs virtuels Azure ou sur la fonctionnalité Disques managés, ou de créer une machine virtuelle Azure faisant office de serveur de fichiers contenant tous les supports d’installation SAP nécessaires.

### <a name="scenario-2-deploying-a-vm-with-a-custom-image-for-sap"></a><a name="54a1fc6d-24fd-4feb-9c57-ac588a55dff2"></a>Scénario 2 : Déploiement d’une machine virtuelle avec une image personnalisée pour SAP

Dans la mesure où les différentes versions d’un système d’exploitation ou d’un SGBD ont des exigences différentes en matière de correctif, les images disponibles dans Azure Marketplace ne répondront peut-être pas à vos besoins. Peut-être préférerez-vous créer une machine virtuelle à l’aide de votre propre image de machine virtuelle du système d’exploitation/SGBD, que vous pourrez de nouveau déployer ultérieurement.
Différentes étapes permettent de créer une image privée pour Linux et pour Windows.

---
> ![Logo Windows][Logo_Windows] Windows
>
> Pour préparer une image Windows pouvant être utilisée pour déployer plusieurs machines virtuelles, les paramètres Windows (comme le nom d’hôte et le SID Windows) doivent être abstraits ou généralisés sur la machine virtuelle locale. Pour ce faire, vous pouvez utiliser la commande [sysprep](/previous-versions/windows/it-pro/windows-8.1-and-8/hh825084(v=win.10)).
>
> ![Logo Linux][Logo_Linux] Linux
>
> Pour préparer une image Linux pouvant être utilisée pour déployer plusieurs machines virtuelles, certains paramètres Linux doivent être abstraits ou généralisés sur la machine virtuelle locale. Pour ce faire, vous pouvez utiliser la commande `waagent -deprovision`. Pour plus d’informations, consultez [Capturer une machine virtuelle Linux exécutée sur Azure][virtual-machines-linux-capture-image] et le [Guide de l’utilisateur de l’agent Linux Azure][virtual-machines-linux-agent-user-guide-command-line-options].
>
>

---
Vous pouvez préparer et créer une image personnalisée, puis l’utiliser pour créer plusieurs nouvelles machines virtuelles. Cette opération est décrite dans [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver][planning-guide]. Configurez le contenu de votre base de données en utilisant le gestionnaire de déploiement de logiciels SAP pour installer un nouveau système SAP (restaure une sauvegarde de base de données à partir d’un disque attaché à la machine virtuelle) ou restaurer directement une sauvegarde de base de données à partir du stockage Azure si votre SGBD le permet. Pour plus d’informations, consultez [Déploiement SGBD de machines virtuelles Azure pour SAP NetWeaver][dbms-guide]. Si vous avez déjà installé un système SAP sur votre machine virtuelle locale (en particulier pour les systèmes à deux niveaux), vous pouvez adapter les paramètres du système SAP après le déploiement de la machine virtuelle Azure à l’aide de la procédure de renommage système prise en charge par le gestionnaire de déploiement de logiciels SAP (Note SAP [1619720]). Sinon, vous pouvez installer les logiciels SAP après avoir déployé la machine virtuelle Azure.

L’organigramme suivant illustre la séquence d’étapes spécifiques à SAP pour le déploiement d’une machine virtuelle à partir d’une image personnalisée :

![Organigramme de déploiement de machines virtuelles pour les systèmes SAP à l’aide d’une image de machine virtuelle provenant du Marketplace privé][deployment-guide-figure-300]

#### <a name="create-a-virtual-machine-by-using-the-azure-portal"></a>Création d’une machine virtuelle à l’aide du portail Azure

Le moyen le plus simple de créer une machine virtuelle à partir d’une image de disque managé est d’utiliser le portail Azure. Pour plus d’informations sur la création d’une image de disque managé, consultez [Capturer une image managée d’une machine virtuelle généralisée dans Azure](../../windows/capture-image-resource.md).

1.  Atteindre <https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Compute%2Fimages>. Vous pouvez également accéder au menu du portail Azure et sélectionner **Images**.
1.  Sélectionnez l’image de disque managé que vous souhaitez déployer, puis cliquez sur **Créer une machine virtuelle**.

L’assistant vous guide pour configurer les paramètres requis pour créer la machine virtuelle, ainsi que toutes les ressources requises, telles que les interfaces réseau et les comptes de stockage. Voici certains exemples de paramètres :

1. **Paramètres de base**:
   * **Name** : Nom de la ressource (nom de la machine virtuelle).
   * **Type de disque de machine virtuelle** : Sélectionnez le type du disque du système d’exploitation. Si vous souhaitez utiliser un stockage Premium pour vos disques de données, nous recommandons d’utiliser également le stockage Premium pour le disque du système d’exploitation.
   * **Nom utilisateur et mot de passe** ou **Clé publique SSH** : Entrez le nom d’utilisateur et le mot de passe de l’utilisateur créé lors de la configuration. Pour une machine virtuelle Linux, vous pouvez entrer la clé publique Secure Shell (SSH) que vous utilisez pour vous connecter à la machine.
   * **Abonnement**: Sélectionnez l’abonnement à utiliser pour configurer la nouvelle machine virtuelle.
   * **Groupe de ressources** : Nom du groupe de ressources de la machine virtuelle. Vous pouvez saisir soit le nom d’un nouveau groupe de ressources, soit celui d’un groupe de ressources existant.
   * **Emplacement** : Emplacement de déploiement de la nouvelle machine virtuelle. Si vous voulez connecter la machine virtuelle à votre réseau local, veillez à sélectionner l’emplacement du réseau virtuel qui connecte Azure à votre réseau local. Pour plus d’informations, consultez [Mise en réseau Microsoft Azure][planning-guide-microsoft-azure-networking] dans [SAP NetWeaver sur machines virtuelles Azure – Guide de planification et d’implémentation][planning-guide].
1. **Size** :

     Pour obtenir la liste des types de machine virtuelle pris en charge, consultez la note SAP [1928533]. Veillez à sélectionner le type de machine virtuelle approprié si vous voulez utiliser le stockage Premium Azure. Tous les types de machine virtuelle ne prennent pas en charge le stockage Premium. Pour plus d’informations, consultez [Stockage : Stockage Microsoft Azure et disques de données][planning-guide-storage-microsoft-azure-storage-and-data-disks] et [Stockage Azure pour les charges de travail SAP](./planning-guide-storage.md) dans [Planification et implémentation des Machines virtuelles Azure pour SAP NetWeaver][planning-guide].

1. **Paramètres**:
   * **Stockage**
     * **Type de disque** : Sélectionnez le type du disque du système d’exploitation. Si vous souhaitez utiliser un stockage Premium pour vos disques de données, nous recommandons d’utiliser également le stockage Premium pour le disque du système d’exploitation.
     * **Utiliser des disques managés** : Si vous souhaitez utiliser la fonctionnalité Disques managés, sélectionnez Oui. Pour plus d’informations sur la fonctionnalité Disques managés, lisez le chapitre [Disques managés](./planning-guide-storage.md#microsoft-azure-storage-resiliency) du guide de planification.
   * **Réseau**
     * **Réseau virtuel** et **Sous-réseau** : Pour intégrer la machine virtuelle à votre intranet, sélectionnez le réseau virtuel connecté à votre réseau local.
     * **Adresse IP publique** : Sélectionnez l’adresse IP publique que vous voulez utiliser ou entrez les paramètres nécessaires pour en créer une. Vous pouvez utiliser une adresse IP publique pour accéder à votre machine virtuelle via Internet. Assurez-vous également de créer un groupe de sécurité réseau pour sécuriser l’accès à votre machine virtuelle.
     * **Groupe de sécurité réseau** : Pour plus d’informations, consultez [Contrôler le flux de trafic réseau avec les groupes de sécurité réseau][virtual-networks-nsg].
   * **Extensions** : Vous pouvez installer des extensions de machine virtuelle en les ajoutant au déploiement. Il n’est pas nécessaire d’ajouter l’extension dans cette étape. Les extensions nécessaires à la prise en charge SAP sont installées ultérieurement. Consultez le chapitre [Configurer l’extension Azure pour SAP][deployment-guide-4.5] de ce guide.
   * **Haute disponibilité** : Sélectionnez un groupe à haute disponibilité ou entrez les paramètres permettant d’en créer un. Pour plus d’informations, consultez [Groupes à haute disponibilité Azure][planning-guide-3.2.3].
   * **Surveillance**
     * **Diagnostics de démarrage** : Vous pouvez sélectionner **Désactiver** pour le diagnostic de démarrage.
     * **Diagnostics du système d’exploitation invité** : Vous pouvez sélectionner **Désactiver** pour les diagnostics d’analyse.

1. **Résumé**:

   Passez en revue vos sélections, puis sélectionnez **OK**.

Votre machine virtuelle est déployée dans le groupe de ressources que vous avez sélectionné.

#### <a name="create-a-virtual-machine-by-using-a-template"></a>Création d’une machine virtuelle à l’aide d’un modèle

Pour créer un déploiement à l’aide d’une image privée de système d’exploitation à partir du portail Azure, utilisez l’un des modèles SAP suivants. Ces modèles sont publiés dans le [dépôt GitHub azure-quickstart-templates][azure-quickstart-templates-github]. Vous pouvez aussi créer une machine virtuelle manuellement à l’aide de [PowerShell][virtual-machines-upload-image-windows-resource-manager].

* [**Modèle de configuration à deux niveaux (une seule machine virtuelle)** (sap-2-tier-user-image)][sap-templates-2-tier-user-image]

  Pour créer un système à deux niveaux avec une seule machine virtuelle, utilisez ce modèle.
* [**Modèle de configuration à deux niveaux (une seule machine virtuelle) - Image de disque managé** (sap-2-tier-user-image-md)][sap-templates-2-tier-user-image-md]

  Pour créer un système à deux niveaux avec une seule machine virtuelle et une image de disque managé, utilisez ce modèle.
* [**Modèle de configuration à trois niveaux (plusieurs machines virtuelles)** (sap-3-tier-user-image)][sap-templates-3-tier-user-image]

  Pour créer un système à trois niveaux avec plusieurs machines virtuelles ou votre propre image de système d’exploitation, utilisez ce modèle.
* [**Modèle de configuration à trois niveaux (plusieurs machines virtuelles) - Image de disque managé** (sap-3-tier-user-image-md)][sap-templates-3-tier-user-image-md]

  Pour créer un système à trois niveaux avec plusieurs machines virtuelles ou votre propre image de système d’exploitation ou image de disque managé, utilisez ce modèle.

Dans le portail Azure, entrez les paramètres suivants pour le modèle :

1. **Paramètres de base**:
   * **Abonnement**: L’abonnement à utiliser pour déployer le modèle.
   * **Groupe de ressources** : Le groupe de ressources à utiliser pour déployer le modèle. Vous pouvez créer un groupe de ressources ou sélectionner un groupe de ressources existant dans l’abonnement.
   * **Emplacement** : Emplacement où déployer le modèle. Si vous avez sélectionné un groupe de ressources existant, l’emplacement du groupe de ressources est utilisé.
1. **Paramètres**:
   * **ID du système SAP** : ID du système SAP.
   * **Type de système d’exploitation** : Type de système d’exploitation à déployer (Windows ou Linux).
   * **Taille du système SAP** : Taille du système SAP.

     Le nombre de SAP fournis par le nouveau système. Si vous ne savez pas combien de SAP sont requis par le système, demandez à votre partenaire technologique SAP ou un intégrateur système.
   * **Disponibilité du système** (modèle à trois niveaux uniquement) : Disponibilité du système.

     Sélectionnez la haute disponibilité (**HA**) si la configuration est adaptée à une installation haute disponibilité. Deux serveurs de base de données et deux serveurs pour l’ASCS sont créés.
   * **Type de stockage** (modèle à deux niveaux uniquement) : Type de stockage à utiliser.

     Pour les grands systèmes, nous vous recommandons l’utilisation du stockage Premium Azure. Pour plus d’informations sur les types de stockage, consultez les ressources suivantes :
      * [Utilisation du stockage SSD Azure Premium pour l’instance de SGBD SAP][2367194]
      * [Structure de stockage d’une machine virtuelle pour les déploiements SGBDR](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64)
      * [Stockage Premium : Stockage hautes performances pour les charges de travail de machine virtuelle Azure][storage-premium-storage-preview-portal]
      * [Introduction à Microsoft Azure Storage][storage-introduction]
   * **URI du disque dur virtuel de l’image utilisateur** : (modèle d’image de disque non managé) : URI du disque dur virtuel de l’image privée du système d’exploitation, par exemple https://&lt;nom du compte&gt;.blob.core.windows.net/vhds/userimage.vhd.
   * **Compte de stockage de l’image utilisateur** : (modèle d’image de disque non managé) : Nom du compte de stockage où est stockée l’image privée du système d’exploitation, par exemple &lt;nom du compte&gt; dans https://&lt;nom du compte&gt;.blob.core.windows.net/vhds/userimage.vhd.
   * **userImageId** (modèle d’image de disque non managé) : ID de l’image de disque managé à utiliser
   * **Nom d’utilisateur** et **Mot de passe administrateur** : Nom d’utilisateur et mot de passe.

     Un nouvel utilisateur est créé pour la connexion à la machine virtuelle.
   * **Sous-réseau nouveau ou existant** : Détermine si un réseau virtuel et un sous-réseau sont créés ou si un sous-réseau existant est utilisé. Si vous disposez déjà d’un réseau virtuel connecté à votre réseau local, sélectionnez **Existant**.
   * **ID de sous-réseau** : Si vous voulez déployer la machine virtuelle dans un réseau virtuel existant où vous avez défini un sous-réseau auquel la machine virtuelle doit être attribuée, nommez l’ID de ce sous-réseau spécifique. L’ID se présente généralement comme suit : /subscriptions/&lt;id d’abonnement>/resourceGroups/&lt;nom du groupe de ressources>/providers/Microsoft.Network/virtualNetworks/&lt;nom du réseau virtuel>/subnets/&lt;nom du sous-réseau>

1. **Conditions générales** :  
    Lisez et acceptez les conditions juridiques.

1. Sélectionnez **Achat**.

#### <a name="install-the-vm-agent-linux-only"></a>Installer l’agent de machine virtuelle (Linux uniquement)

Pour utiliser les modèles décrits dans la section précédente, l’agent Linux doit déjà être installé sur l’image de l’utilisateur. Dans le cas contraire, le déploiement échoue. Téléchargez et installez l’agent de machine virtuelle sur l’image utilisateur, comme décrit dans [Télécharger, installer et activer l’agent de machine virtuelle Azure][deployment-guide-4.4]. Si vous n’utilisez pas les modèles, vous pouvez également installer l’agent de machine virtuelle plus tard.

#### <a name="join-a-domain-windows-only"></a>Joindre un domaine (Windows uniquement)

Si votre déploiement Azure est connecté à une instance Active Directory ou DNS locale via une connexion VPN de site à site Azure ou via Azure ExpressRoute (connexion *intersite* dans [SAP NetWeaver sur machines virtuelles Azure – Guide de planification et d’implémentation][planning-guide]), la machine virtuelle doit être jointe à un domaine local. Pour plus d’informations sur les éléments à prendre en compte pour cette étape, consultez [Joindre une machine virtuelle à un domaine local (Windows uniquement)][deployment-guide-4.3].

#### <a name="configure-proxy-settings"></a>Configuration des paramètres de proxy

Selon la configuration de votre réseau local, vous devrez peut-être configurer le proxy sur votre machine virtuelle. Si votre machine virtuelle est connectée à votre réseau local via VPN ou ExpressRoute, la machine virtuelle ne sera peut-être pas en mesure d’accéder à Internet et de télécharger les extensions nécessaires ou de collecter les informations d’infrastructure Azure pour l’agent hôte SAP par le biais de l’extension SAP pour Azure, consultez [Configurer le proxy][deployment-guide-configure-proxy].

#### <a name="configure-azure-vm-extension-for-sap"></a>Configurer l’extension de machine virtuelle Azure pour SAP

Pour être sûr que SAP est compatible avec votre environnement, configurez l’extension Azure pour SAP, comme décrit dans [Configurer l’extension Azure pour SAP][deployment-guide-4.5]. Consultez la configuration requise de SAP pour connaître les versions minimales requises pour le noyau SAP et l’agent hôte SAP dans les ressources listées dans [Ressources SAP][deployment-guide-2.2].

#### <a name="sap-vm-extension-check"></a>Vérification extension machine virtuelle SAP

Vérifiez si l’extension de machine virtuelle pour SAP fonctionne, selon les modalités décrites dans [Vérifications et résolution des problèmes][deployment-guide-troubleshooting-chapter].


### <a name="scenario-3-moving-an-on-premises-vm-by-using-a-non-generalized-azure-vhd-with-sap"></a><a name="a9a60133-a763-4de8-8986-ac0fa33aa8c1"></a>Scénario 3 : Déplacement d’une machine virtuelle locale à l’aide d’un disque dur virtuel Azure non généralisé avec SAP

Dans ce scénario, vous souhaitez déplacer un système SAP spécifique d’un environnement local vers Azure. Pour ce faire, vous pouvez charger le disque dur virtuel qui contient le système d’exploitation, les fichiers binaires SAP et les éventuels fichiers binaires du SGBD, ainsi que les disques durs virtuels contenant les fichiers de données et les fichiers journaux du SGBD vers Azure. Contrairement au scénario décrit dans [Scénario 2 : Déploiement d’une machine virtuelle avec une image personnalisée pour SAP][deployment-guide-3.3], dans le cas présent, vous conservez le nom d’hôte, le SID SAP et les comptes d’utilisateur SAP dans la machine virtuelle Azure, dans la mesure où ils ont été configurés dans l’environnement local. Il est inutile de généraliser le système d’exploitation. Ce scénario s’applique généralement aux scénarios intersites où une partie du paysage SAP s’exécute en local et une autre partie sur Azure.

Dans ce scénario, l’agent de machine virtuelle **n’est pas** installé automatiquement lors du déploiement. Dans la mesure où l’agent de machine virtuelle et l’extension Azure pour SAP sont nécessaires pour l’exécution de SAP NetWeaver sur Azure, vous devez télécharger, installer et activer ces deux composants manuellement après avoir créé la machine virtuelle.

Pour plus d’informations sur l’agent de machine virtuelle Azure, consultez les ressources suivantes.

---
> ![Logo Windows][Logo_Windows] Windows
>
> [Vue d’ensemble d’agent de machine virtuelle Azure][virtual-machines-windows-agent-user-guide]
>
> ![Logo Linux][Logo_Linux] Linux
>
> [Guide d’utilisateur de l’agent Linux Azure][virtual-machines-linux-agent-user-guide]
>
>

---

L’organigramme suivant présente la séquence d’étapes permettant de déplacer une machine virtuelle locale à l’aide d’un disque dur virtuel Azure non généralisé :

![Organigramme d’un déploiement de machine virtuelle pour les systèmes SAP à l’aide d’un disque de machine virtuelle][deployment-guide-figure-400]

Si le disque est déjà chargé et défini dans Azure (consultez [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver][planning-guide]), effectuez les tâches décrites dans les sections suivantes.

#### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Pour créer un déploiement à l’aide d’un disque privé de système d’exploitation par l’intermédiaire du portail Azure, utilisez le modèle SAP publié dans le [dépôt GitHub azure-quickstart-templates][azure-quickstart-templates-github]. Vous pouvez également créer une machine virtuelle manuellement à l’aide de PowerShell.

* [**Modèle de configuration à deux niveaux (une seule machine virtuelle)** (sap-2-tier-user-disk)][sap-templates-2-tier-os-disk]

  Pour créer un système à deux niveaux avec une seule machine virtuelle, utilisez ce modèle.
* [**Modèle de configuration à deux niveaux (une seule machine virtuelle) - Disque managé** (sap-2-tier-user-disk-md)][sap-templates-2-tier-os-disk-md]

  Pour créer un système à deux niveaux avec une seule machine virtuelle et un disque managé, utilisez ce modèle.

Dans le portail Azure, entrez les paramètres suivants pour le modèle :

1. **Paramètres de base**:
   * **Abonnement**: L’abonnement à utiliser pour déployer le modèle.
   * **Groupe de ressources** : Le groupe de ressources à utiliser pour déployer le modèle. Vous pouvez créer un groupe de ressources ou sélectionner un groupe de ressources existant dans l’abonnement.
   * **Emplacement** : Emplacement où déployer le modèle. Si vous avez sélectionné un groupe de ressources existant, l’emplacement du groupe de ressources est utilisé.
1. **Paramètres**:
   * **ID du système SAP** : ID du système SAP.
   * **Type de système d’exploitation** : Type de système d’exploitation à déployer (Windows ou Linux).
   * **Taille du système SAP** : Taille du système SAP.

     Le nombre de SAP fournis par le nouveau système. Si vous ne savez pas combien de SAP sont requis par le système, demandez à votre partenaire technologique SAP ou un intégrateur système.
   * **Type de stockage** (modèle à deux niveaux uniquement) : Type de stockage à utiliser.

     Pour les grands systèmes, nous vous recommandons l’utilisation du stockage Premium Azure. Pour plus d’informations sur les types de stockage, consultez les ressources suivantes :
      * [Utilisation du stockage SSD Azure Premium pour l’instance de SGBD SAP][2367194]
      * [Structure de stockage d’une machine virtuelle pour les déploiements SGBDR](./dbms_guide_general.md#65fa79d6-a85f-47ee-890b-22e794f51a64)
      * [Stockage Premium : Stockage hautes performances pour les charges de travail de machine virtuelle Azure][storage-premium-storage-preview-portal]
      * [Introduction à Microsoft Azure Storage][storage-introduction]
   * **URI du disque dur virtuel du disque du système d’exploitation** (modèle de disque non managé uniquement) : URI du disque privé du système d’exploitation, par exemple https://&lt;nom du compte&gt;.blob.core.windows.net/vhds/osdisk.vhd.
   * **ID du disque managé du disque de système d’exploitation** (modèle de disque managé uniquement) : ID du disque managé du disque de système d’exploitation, /subscriptions/92d102f7-81a5-4df7-9877-54987ba97dd9/resourceGroups/group/providers/Microsoft.Compute/disks/WIN.
   * **Sous-réseau nouveau ou existant** : Détermine s’il faut créer un réseau virtuel et un sous-réseau, ou utiliser un sous-réseau existant. Si vous disposez déjà d’un réseau virtuel connecté à votre réseau local, sélectionnez **Existant**.
   * **ID de sous-réseau** : Si vous voulez déployer la machine virtuelle dans un réseau virtuel existant où vous avez défini un sous-réseau auquel la machine virtuelle doit être attribuée, nommez l’ID de ce sous-réseau spécifique. L’ID se présente généralement comme suit : /subscriptions/&lt;id d’abonnement>/resourceGroups/&lt;nom du groupe de ressources>/providers/Microsoft.Network/virtualNetworks/&lt;nom du réseau virtuel>/subnets/&lt;nom du sous-réseau>

1. **Conditions générales** :  
    Lisez et acceptez les conditions juridiques.

1. Sélectionnez **Achat**.

#### <a name="install-the-vm-agent"></a>Installer l'agent de machine virtuelle

Pour utiliser les modèles décrits dans la section précédente, l’agent de machine virtuelle doit être installé sur le disque du système d’exploitation. Dans le cas contraire, le déploiement échoue. Téléchargez et installez l’agent de machine virtuelle sur la machine virtuelle, comme décrit dans [Télécharger, installer et activer l’agent de machine virtuelle Azure][deployment-guide-4.4].

Si vous n’utilisez pas les modèles décrits dans la section précédente, vous pouvez également installer l’agent de machine virtuelle plus tard.

#### <a name="join-a-domain-windows-only"></a>Joindre un domaine (Windows uniquement)

Si votre déploiement Azure est connecté à une instance Active Directory ou DNS locale via une connexion VPN de site à site Azure ou via ExpressRoute (connexion *intersite* dans [SAP NetWeaver sur machines virtuelles Azure – Guide de planification et d’implémentation][planning-guide]), la machine virtuelle doit être jointe à un domaine local. Pour plus d’informations sur les éléments à prendre en compte pour cette tâche, consultez [Joindre une machine virtuelle à un domaine local (Windows uniquement)][deployment-guide-4.3].

#### <a name="configure-proxy-settings"></a>Configuration des paramètres de proxy

Selon la configuration de votre réseau local, vous devrez peut-être configurer le proxy sur votre machine virtuelle. Si votre machine virtuelle est connectée à votre réseau local via VPN ou ExpressRoute, la machine virtuelle ne sera peut-être pas en mesure d’accéder à Internet et de télécharger les extensions nécessaires ou de collecter les informations d’infrastructure Azure pour l’agent hôte SAP par le biais de l’extension SAP pour Azure, consultez [Configurer le proxy][deployment-guide-configure-proxy].

#### <a name="configure-azure-vm-extension-for-sap"></a>Configurer l’extension de machine virtuelle Azure pour SAP

Pour être sûr que SAP est compatible avec votre environnement, configurez l’extension Azure pour SAP, comme décrit dans [Configurer l’extension Azure pour SAP][deployment-guide-4.5]. Consultez la configuration requise de SAP pour connaître les versions minimales requises pour le noyau SAP et l’agent hôte SAP dans les ressources listées dans [Ressources SAP][deployment-guide-2.2].

#### <a name="sap-vm-check"></a>Vérification machine virtuelle SAP

Vérifiez si l’extension de machine virtuelle pour SAP fonctionne, selon les modalités décrites dans [Vérifications et résolution des problèmes][deployment-guide-troubleshooting-chapter].

## <a name="update-the-configuration-of-azure-extension-for-sap"></a>Mettre à jour la configuration de l’extension Azure pour SAP

Mettez à jour la configuration de l’extension Azure pour SAP dans les scénarios suivants :
* L’équipe Microsoft/SAP a étendu les capacités de l’extension de machine virtuelle et a décidé d’ajouter ou de supprimer des compteurs.
* Microsoft introduit une nouvelle version de l’infrastructure Azure sous-jacente qui génère les données, et l’extension Azure pour SAP doit s’adapter à ces modifications.
* Vous montez des disques de données supplémentaires sur votre machine virtuelle Azure ou vous supprimez un disque de données. Dans ce scénario, mettez à jour la collection de données liées au stockage. L’ajout ou la suppression de points de terminaison, ou l’affectation d’adresses IP à une machine virtuelle n’affecte pas la configuration de l’extension.
* Vous modifiez la taille de votre machine virtuelle Azure, par exemple en passant de la taille A5 à une autre taille de machine virtuelle.
* Vous ajoutez de nouvelles interfaces de réseau à votre machine virtuelle Azure.

Pour mettre à jour les paramètres, mettez à jour la configuration de l’extension Azure pour SAP en suivant les étapes décrites dans [Configurer l’extension Azure pour SAP][deployment-guide-4.5].

## <a name="detailed-tasks-for-sap-software-deployment"></a>Détail des tâches pour le déploiement de logiciels SAP

Cette section détaille les étapes permettant d’effectuer des tâches spécifiques lors du processus de configuration et de déploiement.

### <a name="deploy-azure-powershell-cmdlets"></a><a name="604bcec2-8b6e-48d2-a944-61b0f5dee2f7"></a>Déploiement d’applets de commande Azure PowerShell

Suivez la procédure décrite dans l’article [Installation du module Azure PowerShell](/powershell/azure/install-az-ps).

Recherchez régulièrement les mises à jour des applets de commande PowerShell, qui sont généralement publiées une fois par mois. Suivez la procédure décrite dans [cet](/powershell/azure/install-az-ps#update-the-azure-powershell-module) article. Sauf indication contraire dans les notes SAP [1928533] ou [2015553], nous vous recommandons d’utiliser la dernière version des applets de commande Azure PowerShell.

Pour vérifier la version des applets de commande Azure PowerShell installées sur votre ordinateur, exécutez cette commande PowerShell :

```powershell
(Get-Module Az.Compute).Version
```

### <a name="deploy-azure-cli"></a><a name="1ded9453-1330-442a-86ea-e0fd8ae8cab3"></a>Déploiement de l’interface de ligne de commande Azure

Suivez la procédure décrite dans l’article [Installation d’Azure CLI](/cli/azure/install-azure-cli).

Recherchez régulièrement les mises à jour de l’interface de ligne de commande Azure, qui sont généralement publiées une fois par mois.

Pour vérifier la version de l’interface de ligne de commande Azure installée sur votre ordinateur, exécutez cette commande :

```console
az --version
```

### <a name="join-a-vm-to-an-on-premises-domain-windows-only"></a><a name="31d9ecd6-b136-4c73-b61e-da4a29bbc9cc"></a>Joindre une machine virtuelle à un domaine local (Windows uniquement)

Si vous déployez des machines virtuelles SAP dans le cadre d’un scénario intersite où les instances Active Directory et DNS locales sont étendues dans Azure, les machines virtuelles doivent être jointes à un domaine local. Les étapes détaillées de connexion d’une machine virtuelle à un domaine local et les logiciels supplémentaires requis pour les membres d’un domaine local dépendent du client. Généralement, pour joindre une machine virtuelle à un domaine local, vous devez installer des logiciels supplémentaires, tels que des logiciels anti-programme malveillant et des logiciels de sauvegarde ou d’analyse.

Dans ce scénario, vous devez également vous assurer que si les paramètres du proxy Internet sont forcés lorsqu’une machine virtuelle rejoint un domaine dans votre environnement, les paramètres de proxy du compte système local Windows (S-1-5-18) de la machine virtuelle invitée sont identiques. Le plus simple est d’imposer le proxy avec une stratégie de groupe de domaine qui s’applique aux systèmes du domaine.

### <a name="download-install-and-enable-the-azure-vm-agent"></a><a name="c7cbb0dc-52a4-49db-8e03-83e7edc2927d"></a>Télécharger, installer et activer l’agent de machine virtuelle Azure

Pour les machines virtuelles qui sont déployées à partir d’une image du système d’exploitation non généralisée (par exemple, une image qui ne provient pas de l’outil de préparation du système Windows, ou sysprep), vous devez télécharger, installer et activer manuellement l’agent de machine virtuelle Azure.

Si vous déployez une machine virtuelle à partir d’Azure Marketplace, cette étape n’est pas nécessaire. Les images provenant d’Azure Marketplace disposent déjà de l’agent de machine virtuelle Azure.

#### <a name="windows"></a><a name="b2db5c9a-a076-42c6-9835-16945868e866"></a>Windows

1. Téléchargez l’agent de machine virtuelle Azure :
   1.  Téléchargez le [package d’installation de l’agent de machine virtuelle Azure](https://go.microsoft.com/fwlink/?LinkId=394789).
   1.  Stockez le package MSI de l’agent de machine virtuelle localement sur un serveur ou un ordinateur personnel.
1. Installez l’agent de machine virtuelle Azure :
   1.  Connectez-vous à la machine virtuelle Azure déployée à l’aide du protocole Remote Desktop Protocol (RDP).
   1.  Ouvrez une fenêtre de l’explorateur Windows sur la machine virtuelle et sélectionnez un répertoire cible pour le fichier MSI de l’agent de machine virtuelle.
   1.  Faites glisser le fichier MSI du programme d’installation de l’agent de machine virtuelle Azure de votre ordinateur/serveur local dans le répertoire cible de l’agent de machine virtuelle dans la machine virtuelle.
   1.  Double-cliquez sur le fichier MSI dans la machine virtuelle.
1. Pour les machines virtuelles associées à des domaines locaux, veillez à ce que les éventuels paramètres de proxy Internet s’appliquent également au compte système local Windows (S-1-5-18) de la machine virtuelle, comme décrit dans [Configurer le proxy][deployment-guide-configure-proxy]. L’agent de machine virtuelle s’exécute dans ce contexte et doit être en mesure de se connecter à Azure.

Aucune interaction de l’utilisateur n’est requise pour mettre à jour l’agent de machine virtuelle Azure. L’agent de machine virtuelle est automatiquement mis à jour, sans nécessiter de redémarrage de la machine virtuelle.

#### <a name="linux"></a><a name="6889ff12-eaaf-4f3c-97e1-7c9edc7f7542"></a>Linux

Utilisez les commandes suivantes pour installer l’agent de machine virtuelle pour Linux :

* **SUSE Linux Enterprise Server (SLES)**

  ```console
  sudo zypper install WALinuxAgent
  ```

* **Red Hat Enterprise Linux (RHEL) ou Oracle Linux**

  ```console
  sudo yum install WALinuxAgent
  ```

Si l’agent est déjà installé, pour mettre à jour l’agent Linux Azure, suivez les étapes décrites dans [Mettre à jour l’agent Linux Azure sur une machine virtuelle vers la dernière version à partir de GitHub][virtual-machines-linux-update-agent].

### <a name="configure-the-proxy"></a><a name="baccae00-6f79-4307-ade4-40292ce4e02d"></a>Configurer le proxy

Les étapes à suivre pour configurer le proxy sous Windows sont différentes de celles utilisées pour configurer le proxy sous Linux.

#### <a name="windows"></a>Windows

Les paramètres de proxy doivent être correctement configurés pour que le compte système local puisse accéder à Internet. Si vos paramètres de proxy ne sont pas définis par la stratégie de groupe, vous pouvez les configurer pour le compte système local.

1. Accédez à **Démarrer**, entrez **gpedit.msc**, puis appuyez sur **Entrée**.
1. Sélectionnez **Configuration ordinateur** > **Modèles d’administration** > **Composants Windows** > **Internet Explorer**. Assurez-vous que le paramètre **Paramètres machine du serveur proxy (plutôt que les paramètres individualisés)** est désactivé ou non configuré.
1. Dans le **panneau de configuration**, accédez à **Centre réseau et partage** > **Options Internet**.
1. Dans l’onglet **Connexions**, sélectionnez le bouton **Paramètres LAN**.
1. Décochez la case **Détecter automatiquement les paramètres de connexion**.
1. Cochez la case **Utiliser un serveur proxy pour votre réseau local** et entrez l’adresse et le port du proxy.
1. Sélectionnez le bouton **Advanced (Avancé)** .
1. Dans la zone **Exceptions**, entrez l’adresse IP **168.63.129.16**. Sélectionnez **OK**.

#### <a name="linux"></a>Linux

Configurez le proxy approprié dans le fichier de configuration de l’agent invité Microsoft Azure, qui se trouve dans \\etc\\waagent.conf.

Définissez les paramètres suivants :

1. **Hôte proxy HTTP**. Par exemple, définissez-le sur **proxy.corp.local**.

   ```console
   HttpProxy.Host=<proxy host>

   ```

1. **Port proxy HTTP**. Par exemple, définissez-le sur **80**.

   ```console
   HttpProxy.Port=<port of the proxy host>

   ```

1. Redémarrez l’agent.

   ```console
   sudo service waagent restart
   ```

Les paramètres de proxy dans \\etc\\waagent.conf s’appliquent également aux extensions de machine virtuelle requises. Si vous voulez utiliser les référentiels Azure, assurez-vous que le trafic vers ces référentiels ne passe pas par l’intranet local. Si vous avez créé des itinéraires définis par les utilisateurs pour activer le tunneling forcé, veillez à ajouter un itinéraire qui achemine le trafic vers les référentiels directement vers Internet, et non par le biais de votre connexion VPN de site à site.

* **SLES**

  Vous devez également ajouter des itinéraires pour les adresses IP répertoriées dans \\etc\\regionserverclnt.cfg. La figure suivante montre un exemple :

  ![Tunneling forcé][deployment-guide-figure-50]


* **RHEL**

  Vous devez également ajouter des itinéraires pour les adresses IP des hôtes répertoriés dans \\etc\\yum.repos.d\\rhui-load-balancers. Pour obtenir un exemple, consultez la figure précédente.

* **Oracle Linux**

  Il n’existe aucun référentiel pour Oracle Linux sur Azure. Vous devez configurer vos propres référentiels pour Oracle Linux ou utiliser des référentiels publics.

Pour plus d’informations sur les routes définies par l’utilisateur, consultez [Routes définies par l’utilisateur et transfert IP][virtual-networks-udr-overview].

### <a name="configure-the-azure-extension-for-sap"></a><a name="d98edcd3-f2a1-49f7-b26a-07448ceb60ca"></a>Configurer l’extension Azure pour SAP

> [!NOTE]
> Instruction générale de support : Pour tout besoin de support concernant l’extension Azure pour SAP, ouvrez un incident auprès de SAP sur le composant BC-OP-NT-AZR pour Windows ou BC-OP-LNX-AZR.
> Des ingénieurs de support Microsoft dédiés travaillent dans le système de support SAP pour aider nos clients communs.

Une fois que la machine virtuelle est préparée comme décrit dans [Scénarios de déploiement de machines virtuelles pour SAP sur Azure][deployment-guide-3], l’agent de machine virtuelle Azure est installé sur la machine virtuelle. L’étape suivante consiste à déployer l’extension Azure pour SAP, qui est disponible dans le référentiel d’extensions Azure dans les centres de données mondiaux d’Azure. Pour plus d’informations, consultez [SAP NetWeaver sur machines virtuelles Azure – Guide de planification et d’implémentation][planning-guide-9.1].

Nous sommes en train de publier une nouvelle version de l’extension Azure pour SAP. La nouvelle extension utilise l’identité affectée par le système de la machine virtuelle pour obtenir des informations sur les disques attachés, les interfaces réseau et la machine virtuelle proprement dite. Pour que ces ressources soient accessibles, il faut que l’identité système de la machine virtuelle dispose de l’autorisation Lecteur sur la machine virtuelle, le disque du système d’exploitation, les disques de données et les interfaces réseau. Nous vous recommandons actuellement de n’installer la nouvelle extension que dans les cas de figure suivants :

1. Vous souhaitez installer l’extension avec Terraform, les modèles Azure Resource Manager ou tout moyen autre qu’Azure CLI ou Azure PowerShell.
1. Vous souhaitez installer l’extension sur SUSE SLES 15 ou une version ultérieure.
1. Le support Microsoft ou SAP vous demande d’installer la nouvelle extension.
1. Vous souhaitez utiliser le Disque Ultra ou les Disques managés standard Azure.

Dans ces cas de figure, suivez les étapes décrites dans le chapitre [Configuration de la nouvelle extension Azure pour SAP avec Azure PowerShell][deployment-guide-configure-new-extension-ps] pour Azure PowerShell ou [Configuration de la nouvelle extension Azure pour SAP avec Azure CLI][deployment-guide-configure-new-extension-cli] pour Azure CLI.

Suivez [Azure PowerShell][deployment-guide-4.5.1] ou [Azure CLI][deployment-guide-4.5.2] pour installer et configurer la version standard de l’extension Azure pour SAP.

#### <a name="azure-powershell-for-linux-and-windows-vms"></a><a name="987cf279-d713-4b4c-8143-6b11589bb9d4"></a>Azure PowerShell pour les machines virtuelles Linux et Windows

Pour installer l’extension Azure pour SAP à l’aide de PowerShell :

1. Vérifiez que vous avez installé la dernière version de l’applet de commande Azure PowerShell. Pour plus d’informations, consultez [Déploiement d’applets de commande Azure PowerShell][deployment-guide-4.1].  
1. Exécutez l’applet de commande PowerShell suivant.
    Pour afficher la liste des environnements disponibles, exécutez la cmdlet `Get-AzEnvironment` . Si vous voulez utiliser la version globale d’Azure, votre environnement est **AzureCloud**. Pour Azure China 21Vianet, sélectionnez **AzureChinaCloud**.

    ```powershell
    $env = Get-AzEnvironment -Name <name of the environment>
    Connect-AzAccount -Environment $env
    Set-AzContext -SubscriptionName <subscription name>

    Set-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
    ```

Une fois que vous avez entré les données de votre compte, le script déploie les extensions requises et active les fonctionnalités nécessaires. Ceci peut prendre plusieurs minutes.
Pour plus d’informations sur `Set-AzVMAEMExtension`, consultez [Set-AzVMAEMExtension][msdn-set-Azvmaemextension].

![Exécution réussie de la cmdlet Azure spécifique à SAP Set-AzVMAEMExtension][deployment-guide-figure-900]

La configuration `Set-AzVMAEMExtension` effectue toutes les étapes de configuration de collecte de données d’hôte pour SAP.

La sortie du script comprend les informations suivantes :

* La confirmation que la collecte de données a été configurée pour le disque du système d’exploitation et tous les autres disques de données.
* Les deux messages suivants confirment la configuration des métriques de stockage pour un compte de stockage spécifique.
* Une des lignes de sortie fournit l’état de la mise à jour réelle de la configuration de l’extension de machine virtuelle pour SAP.
* Une autre ligne de sortie confirme que la configuration a été déployée ou mise à jour.
* La dernière ligne de sortie est informative. Elle répertorie les options permettant de tester la configuration de l’extension de machine virtuelle pour SAP.
* Pour vérifier que toutes les étapes de configuration de l’extension de machine virtuelle Azure pour SAP ont été exécutées correctement et que l’infrastructure Azure fournit les données nécessaires, vérifiez la disponibilité de l’extension Azure pour SAP, comme décrit dans [Vérification de la disponibilité de l’extension Azure pour SAP][deployment-guide-5.1].
* Attendez 15 à 30 minutes pour que les diagnostics Azure collectent les données pertinentes.

#### <a name="azure-cli-for-linux-vms"></a><a name="408f3779-f422-4413-82f8-c57a23b4fc2f"></a>Interface de ligne de commande Azure pour machines virtuelles Linux

Pour installer l’extension Azure pour SAP à l’aide d’Azure CLI :

   1. Installez l’interface de ligne de commande Azure Classic comme décrit dans [Installer l’interface de ligne de commande Azure Classic][azure-cli].
   1. Connectez-vous à votre compte Azure :

      ```console
      azure login
      ```

   1. Basculez en mode Azure Resource Manager :

      ```console
      azure config mode arm
      ```

   1. Activez l’extension Azure pour SAP :

      ```console
      azure vm enable-aem <resource-group-name> <vm-name>
      ```

1. Installer avec Azure CLI 2.0

   1. Installez Azure CLI 2.0, comme décrit dans [Installer Azure CLI 2.0][azure-cli-2].
   1. Connectez-vous à votre compte Azure :

      ```azurecli
      az login
      ```

   1. Installer l’extension Azure CLI AEM
  
      ```azurecli
      az extension add --name aem
      ```
  
   1. Installez l’extension avec
  
      ```azurecli
      az vm aem set -g <resource-group-name> -n <vm name>
      ```

1. Vérifiez que l’extension Azure pour SAP est active sur la machine virtuelle Azure Linux. Vérifiez si le fichier \\var\\lib\\AzureEnhancedMonitor\\PerfCounters existe. S’il existe, à l’invite de commandes, exécutez cette commande pour afficher les informations collectées par l’extension Azure pour SAP :

   ```console
   cat /var/lib/AzureEnhancedMonitor/PerfCounters
   ```

   Le résultat se présente ainsi :

   ```output
   ...
   2;cpu;Current Hw Frequency;;0;2194.659;MHz;60;1444036656;saplnxmon;
   2;cpu;Max Hw Frequency;;0;2194.659;MHz;0;1444036656;saplnxmon;
   ...
   ```

#### <a name="configure-the-new-azure-extension-for-sap-with-azure-powershell"></a><a name="2ad55a0d-9937-4943-9dd2-69bc2b5d3de0"></a>Configuration de la nouvelle extension Azure pour SAP avec Azure PowerShell

La nouvelle extension de machine virtuelle pour SAP utilise une identité managée affectée à la machine virtuelle pour accéder aux données de surveillance et de configuration de la machine virtuelle. Pour installer la nouvelle extension à l’aide de PowerShell, vous devez d’abord attribuer à la machine virtuelle une identité de ce type, à laquelle vous accordez l’accès à toutes les ressources utilisées par cette machine virtuelle, par exemple les disques et les interfaces réseau.

> [!NOTE]
> Des privilèges de propriétaire sur le groupe de ressources ou certaines ressources (machine virtuelle, disques de données, etc.) sont nécessaires pour suivre les étapes ci-dessous.

1. Veillez à utiliser la version 7.21 PL 47 ou une version ultérieure de l’agent hôte SAP.
1. Désinstallez la version actuelle de l’extension de machine virtuelle pour SAP. L’installation des deux versions de l’extension de machine virtuelle pour SAP sur la même machine virtuelle n’est pas prise en charge.
1. Vérifiez que vous avez installé la dernière version de la cmdlet Azure PowerShell (au minimum, la version 4.3.0). Pour plus d’informations, consultez [Déploiement d’applets de commande Azure PowerShell][deployment-guide-4.1].
1. Exécutez l’applet de commande PowerShell suivant.
    Pour afficher la liste des environnements disponibles, exécutez la cmdlet `Get-AzEnvironment` . Si vous voulez utiliser la version globale d’Azure, votre environnement est **AzureCloud**. Pour Azure China 21Vianet, sélectionnez **AzureChinaCloud**.

    ```powershell
    $env = Get-AzEnvironment -Name <name of the environment>
    Connect-AzAccount -Environment $env
    Set-AzContext -SubscriptionName <subscription name>

    Set-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name> -InstallNewExtension
    ```

#### <a name="configure-the-new-azure-extension-for-sap-with-azure-cli"></a><a name="c8749c24-fada-42ad-b114-f9aae2dc37da"></a>Configuration de la nouvelle extension Azure pour SAP avec Azure CLI

La nouvelle extension de machine virtuelle pour SAP utilise une identité managée affectée à la machine virtuelle pour accéder aux données de surveillance et de configuration de la machine virtuelle. Pour installer la nouvelle extension à l’aide d’Azure CLI, vous devez d’abord attribuer à la machine virtuelle une identité de ce type, à laquelle vous accordez l’accès à toutes les ressources utilisées par cette machine virtuelle, par exemple les disques et les interfaces réseau.

> [!NOTE]
> Des privilèges de propriétaire sur le groupe de ressources ou certaines ressources (machine virtuelle, disques de données, etc.) sont nécessaires pour suivre les étapes ci-dessous.

1. Veillez à utiliser la version 7.21 PL 47 ou une version ultérieure de l’agent hôte SAP.
1. Désinstallez la version actuelle de l’extension de machine virtuelle pour SAP. L’installation des deux versions de l’extension de machine virtuelle pour SAP sur la même machine virtuelle n’est pas prise en charge.
1. Installez Azure CLI 2.0, comme décrit dans [Installer Azure CLI 2.0][azure-cli-2].

1. Connectez-vous à votre compte Azure :

   ```azurecli
   az login
   ```

1. Suivez la procédure décrite dans l’article [Configuration des identités managées pour les ressources Azure sur une machine virtuelle à l’aide d’Azure CLI][qs-configure-cli-windows-vm] pour activer une identité managée affectée par le système à la machine virtuelle. Les identités managées affectées par l’utilisateur ne sont pas prises en charge par l’extension de machine virtuelle pour SAP. Toutefois, vous pouvez activer à la fois une identité affectée par le système et une identité affectée par l’utilisateur.

   Exemple :
   ```azurecli
   az vm identity assign -g <resource-group-name> -n <vm name>
   ```

1. Attribuez à l’identité managée l’accès au groupe de ressources de la machine virtuelle ou à toutes les interfaces réseau, aux disques managés et à la machine virtuelle proprement dite, selon les modalités décrites dans [Attribution de l’accès à une ressource à une identité managée à l’aide d’Azure CLI][howto-assign-access-cli].

    Exemple :

    ```azurecli
    # Azure CLI on Linux
    spID=$(az resource show -g <resource-group-name> -n <vm name> --query identity.principalId --out tsv --resource-type Microsoft.Compute/virtualMachines)
    rgId=$(az group show -g <resource-group-name> --query id --out tsv)
    az role assignment create --assignee $spID --role 'Reader' --scope $rgId

    # Azure CLI on Windows/PowerShell
    $spID=az resource show -g <resource-group-name> -n <vm name> --query identity.principalId --out tsv --resource-type Microsoft.Compute/virtualMachines
    $rgId=az group show -g <resource-group-name> --query id --out tsv
    az role assignment create --assignee $spID --role 'Reader' --scope $rgId
    ```

1. Exécutez la commande Azure CLI suivante afin d’installer l’extension Azure pour SAP.
    L’extension n’est actuellement prise en charge que dans AzureCloud. Azure China 21Vianet, Azure Government et les autres environnements spéciaux ne sont pas encore pris en charge.

    ```azurecli
    # Azure CLI on Linux
    ## For Linux machines
    az vm extension set --publisher Microsoft.AzureCAT.AzureEnhancedMonitoring --name MonitorX64Linux --version 1.0 -g <resource-group-name> --vm-name <vm name> --settings '{"system":"SAP"}'

    ## For Windows machines
    az vm extension set --publisher Microsoft.AzureCAT.AzureEnhancedMonitoring --name MonitorX64Windows --version 1.0 -g <resource-group-name> --vm-name <vm name> --settings '{"system":"SAP"}'

    # Azure CLI on Windows/PowerShell
    ## For Linux machines
    az vm extension set --publisher Microsoft.AzureCAT.AzureEnhancedMonitoring --name MonitorX64Linux --version 1.0 -g <resource-group-name> --vm-name <vm name> --settings '{\"system\":\"SAP\"}'

    ## For Windows machines
    az vm extension set --publisher Microsoft.AzureCAT.AzureEnhancedMonitoring --name MonitorX64Windows --version 1.0 -g <resource-group-name> --vm-name <vm name> --settings '{\"system\":\"SAP\"}'
    ```

## <a name="checks-and-troubleshooting"></a><a name="564adb4f-5c95-4041-9616-6635e83a810b"></a>Vérifications et résolution des problèmes

Une fois que vous avez déployé votre machine virtuelle Azure et configuré l’extension Azure pour SAP appropriée, vérifiez si tous les composants de l’extension fonctionnent correctement.

Vérifiez la disponibilité de l’extension Azure pour SAP, comme décrit dans [Vérification de la disponibilité de l’extension Azure pour SAP][deployment-guide-5.1]. Si tous les résultats de la vérification de disponibilité sont positifs et que tous les compteurs de performances ont le statut OK, l’extension Azure pour SAP a été correctement configurée. Vous pouvez poursuivre l’installation de l’agent hôte SAP, comme décrit dans les notes SAP sous [Ressources SAP][deployment-guide-2.2]. Si la vérification de disponibilité signale des compteurs manquants, effectuez un contrôle d’intégrité de l’extension Azure pour SAP, comme décrit dans [Contrôle d’intégrité de la configuration de l’extension Azure pour SAP][deployment-guide-5.2]. Pour obtenir davantage d’options de résolution des problèmes, consultez [Résolution des problèmes de l’extension Azure pour SAP][deployment-guide-5.3].

### <a name="readiness-check-for-the-azure-extension-for-sap"></a><a name="bb61ce92-8c5c-461f-8c53-39f5e5ed91f2"></a>Vérification de la disponibilité de l’extension Azure pour SAP

> [!NOTE]
> L’extension de machine virtuelle se présente sous deux versions. Ce chapitre traite de l’extension par défaut. Si vous avez installé la nouvelle extension de machine virtuelle, consultez le chapitre [Vérification de préparation à la nouvelle extension Azure pour SAP][deployment-guide-5.1-new].

Cette vérification vous assure que tous les indicateurs de performance affichés au sein de votre application SAP sont fournis par l’extension Azure pour SAP sous-jacente.

#### <a name="run-the-readiness-check-on-a-windows-vm"></a>Vérifier la disponibilité d’une machine virtuelle Windows

1. Connectez-vous à la machine virtuelle Azure (l’utilisation d’un compte Administrateur n’est pas nécessaire).
1. Ouvrez une fenêtre d’invite de commandes.
1. À l’invite de commandes, remplacez le référentiel par le dossier d’installation de l’extension Azure pour SAP : C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\&lt;version>\\drop

   La *version* figurant dans le chemin d’accès à l’extension peut varier. Si vous voyez des dossiers pour plusieurs versions de l’extension dans le dossier d’installation, vérifiez la configuration du service Windows AzureEnhancedMonitoring, puis accédez au dossier indiqué en tant que *Chemin d’accès des fichiers exécutables*.

   ![Propriétés du service exécutant l’extension Azure pour SAP][deployment-guide-figure-1000]

1. À l’invite de commandes, exécutez **azperflib.exe** sans aucun paramètre.

   > [!NOTE]
   > Le fichier exécutable azperflib.exe fonctionne en boucle et met à jour les compteurs collectés toutes les 60 secondes. Pour mettre fin à la boucle, fermez la fenêtre d’invite de commandes.
   >
   >

Si l’extension Azure pour SAP n’est pas installée ou que le service AzureEnhancedMonitoring n’est pas en cours d’exécution, cela signifie que l’extension n’a pas été configurée correctement. Pour plus d’informations sur la façon de déployer l’extension, consultez [Résolution des problèmes de l’extension Azure pour SAP][deployment-guide-5.3].

> [!NOTE]
> Le fichier Azperflib.exe est un composant qui ne peut pas être utilisé pour des besoins propres. Ce composant fournit des données d’infrastructure Azure relatives à la machine virtuelle pour l’agent hôte SAP exclusivement.
> 

##### <a name="check-the-output-of-azperflibexe"></a>Vérifiez la sortie renvoyée par azperflib.exe

La sortie de azperflib.exe indique tous les compteurs de performances Azure remplis pour SAP. En bas de la liste des compteurs collectés, vous trouverez un résumé et un indicateur d’intégrité, qui indiquent l’état de l’extension Azure pour SAP.

![Sortie du contrôle d’intégrité effectué avec azperflib.exe indiquant l’absence de problèmes][deployment-guide-figure-1100]
<a name="figure-11"></a>

Vérifiez le résultat renvoyé pour la sortie **Nombre total de compteurs**, qui est indiqué comme étant vide, et pour **État d’intégrité**, comme indiqué à la figure précédente.

Interprétez les valeurs obtenues comme suit :

| Résultats du fichier exécutable azperflib.exe | État d’intégrité de l’extension Azure pour SAP |
| --- | --- |
| **Appels de l’API - non disponibles** | Les compteurs qui ne sont pas disponibles peuvent ne pas être concernés par la configuration de la machine virtuelle ou constituent des erreurs. Voir **État d’intégrité**. |
| **Nombre total de compteurs : vide** |Les deux compteurs suivants de stockage Azure peuvent être vides : <ul><li>Stockage Lecture Op Latence Serveur msec</li><li>Stockage Lecture Op Latence E2E msec</li></ul>Tous les autres compteurs doivent contenir des valeurs. |
| **État d’intégrité** |Uniquement OK si l’état renvoyé est **OK**. |
| **Diagnostics** |Informations détaillées sur l’état d’intégrité. |

Si la valeur **État d’intégrité** n’est pas **OK**, suivez les instructions de [Contrôle d’intégrité de la configuration de l’extension Azure pour SAP][deployment-guide-5.2].

#### <a name="run-the-readiness-check-on-a-linux-vm"></a>Vérifier la disponibilité d’une machine virtuelle Linux

1. Connectez-vous à la machine virtuelle Azure à l’aide de SSH.

1. Vérifiez la sortie de l’extension Azure pour SAP.

   a.  Exécutez `more /var/lib/AzureEnhancedMonitor/PerfCounters`

   **Résultat attendu** : Renvoie la liste des compteurs de performances. Ce fichier ne doit pas être vide.

   b. Exécutez `cat /var/lib/AzureEnhancedMonitor/PerfCounters | grep Error`

   **Résultat attendu** : Renvoie une ligne où l’erreur est **none**, par exemple **3;config;Error;;0;0;none;0;1456416792;tst-servercs;**

   c. Exécutez `more /var/lib/AzureEnhancedMonitor/LatestErrorRecord`

   **Résultat attendu** : Revient vide ou n’existe pas.

Si la vérification précédente a échoué, exécutez ces vérifications supplémentaires :

1. Assurez-vous que waagent est installé et démarré.

   a.  Exécutez `sudo ls -al /var/lib/waagent/`

     **Résultat attendu** : Répertorie le contenu du répertoire waagent.

   b.  Exécutez `ps -ax | grep waagent`

   **Résultat attendu** : Affiche une entrée similaire à : `python /usr/sbin/waagent -daemon`

1. Assurez-vous que l’extension Azure pour SAP est installée et en cours d’exécution.

   a.  Exécutez `sudo sh -c 'ls -al /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-*/'`

   **Résultat attendu** : Répertorie le contenu de l’annuaire de l’extension Azure pour SAP.

   b. Exécutez `ps -ax | grep AzureEnhanced`

   **Résultat attendu** : Affiche une entrée similaire à : `python /var/lib/waagent/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux-2.0.0.2/handler.py daemon`

1. Installez l’agent hôte SAP comme décrit dans la note SAP [1031096] et vérifiez le résultat de `saposcol`.

   a.  Exécutez `/usr/sap/hostctrl/exe/saposcol -d`

   b.  Exécutez `dump ccm`

   c.  Vérifiez si la métrique **Virtualization_Configuration\Enhanced Monitoring Access** a la valeur **true**.

Si vous avez déjà installé un serveur d’applications ABAP SAP NetWeaver, ouvrez la transaction ST06 et regardez si l’analyse est activée.

Si l’une de ces vérifications échoue et pour plus d’informations sur la façon de redéployer l’extension, consultez [Résolution des problèmes de l’extension Azure pour SAP][deployment-guide-5.3].

### <a name="readiness-check-for-the-new-azure-extension-for-sap"></a><a name="7bf24f59-7347-4c7a-b094-4693e4687ee5"></a>Vérification de la préparation à la nouvelle extension Azure pour SAP

> [!NOTE]
> L’extension de machine virtuelle se présente sous deux versions. Ce chapitre traite de la nouvelle extension. Si vous avez installé l’extension de machine virtuelle par défaut, consultez le chapitre [Vérification de préparation à l’extension Azure pour SAP][deployment-guide-5.1].

Cette vérification vous assure que tous les indicateurs de performance affichés au sein de votre application SAP sont fournis par l’extension Azure pour SAP sous-jacente.

#### <a name="run-the-readiness-check-on-a-windows-vm"></a>Vérifier la disponibilité d’une machine virtuelle Windows

1. Connectez-vous à la machine virtuelle Azure (l’utilisation d’un compte Administrateur n’est pas nécessaire).
1. Ouvrez un navigateur web et accédez à http://127.0.0.1:11812/azure4sap/metrics.
1. Le navigateur affiche ou télécharge un fichier XML contenant les données d’analyse de votre machine virtuelle. Si ce n’est pas le cas, vérifiez que l’extension Azure pour SAP est installée.

##### <a name="check-the-content-of-the-xml-file"></a>Vérification du contenu du fichier XML

Le fichier XML accessible à l’adresse http://127.0.0.1:11812/azure4sap/metrics contient tous les compteurs de performances Azure remplis pour SAP. Il comporte également un résumé et un indicateur d’intégrité de l’état de l’extension Azure pour SAP.

Vérifiez la valeur de la description de l’élément **Description de l’intégrité du fournisseur**. Si la valeur n’est pas **OK**, suivez les instructions de [Contrôle d’intégrité de la configuration de la nouvelle extension Azure pour SAP][deployment-guide-5.2-new].

#### <a name="run-the-readiness-check-on-a-linux-vm"></a>Vérifier la disponibilité d’une machine virtuelle Linux

1. Connectez-vous à la machine virtuelle Azure à l’aide de SSH.

1. Vérifiez la sortie de la commande suivante :

    ```console
    curl http://127.0.0.1:11812/azure4sap/metrics
    ```
    
   **Résultat attendu** : Retourne un document XML contenant les informations d’analyse de la machine virtuelle, de ses disques et de ses interfaces réseau.

Si la vérification précédente a échoué, exécutez ces vérifications supplémentaires :

1. Assurez-vous que waagent est installé et démarré.

   a.  Exécutez `sudo ls -al /var/lib/waagent/`

     **Résultat attendu** : Répertorie le contenu du répertoire waagent.

   b.  Exécutez `ps -ax | grep waagent`

   **Résultat attendu** : Affiche une entrée similaire à : `python /usr/sbin/waagent -daemon`

1. Assurez-vous que l’extension Azure pour SAP est installée et en cours d’exécution.

   a.  Exécutez `sudo sh -c 'ls -al /var/lib/waagent/Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Linux-*/'`

   **Résultat attendu** : Répertorie le contenu de l’annuaire de l’extension Azure pour SAP.

   b. Exécutez `ps -ax | grep AzureEnhanced`

   **Résultat attendu** : Affiche une entrée similaire à : `/var/lib/waagent/Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Linux-1.0.0.82/AzureEnhancedMonitoring -monitor`

1. Installez l’agent hôte SAP comme décrit dans la note SAP [1031096] et vérifiez le résultat de `saposcol`.

   a.  Exécutez `/usr/sap/hostctrl/exe/saposcol -d`

   b.  Exécutez `dump ccm`

   c.  Vérifiez si la métrique **Virtualization_Configuration\Enhanced Monitoring Access** a la valeur **true**.

Si vous avez déjà installé un serveur d’applications ABAP SAP NetWeaver, ouvrez la transaction ST06 et regardez si l’analyse est activée.

Si l’une de ces vérifications échoue ou pour savoir comment redéployer l’extension, consultez [Résolution des problèmes de la nouvelle extension Azure pour SAP][deployment-guide-5.3-new].

### <a name="health-check-for-the-azure-extension-for-sap-configuration"></a><a name="e2d592ff-b4ea-4a53-a91a-e5521edb6cd1"></a>Contrôle d’intégrité de la configuration de l’extension Azure pour SAP

> [!NOTE]
> L’extension de machine virtuelle se présente sous deux versions. Ce chapitre traite de l’extension par défaut. Si vous avez installé la nouvelle extension de machine virtuelle, consultez le chapitre [Contrôle d’intégrité en vue de la configuration de la nouvelle extension Azure pour SAP][deployment-guide-5.2-new].

Si certaines des données d’infrastructure ne sont pas fournies correctement, comme indiqué par le test décrit dans [Vérification de la disponibilité de l’extension Azure pour SAP][deployment-guide-5.1], exécutez l’applet de commande `Test-AzVMAEMExtension` pour tester si l’infrastructure Azure et l’extension Azure pour SAP sont correctement configurées.

1. Assurez-vous d’avoir installé la dernière version de l’applet de commande Azure PowerShell, comme décrit dans le chapitre [Déploiement d’applets de commande Azure PowerShell][deployment-guide-4.1].
1. Exécutez l’applet de commande PowerShell suivant. Pour afficher la liste des environnements disponibles, exécutez l’applet de commande `Get-AzEnvironment`. Pour utiliser la version globale d’Azure, sélectionnez l’environnement **AzureCloud**. Pour Azure China 21Vianet, sélectionnez **AzureChinaCloud**.

   ```powershell
   $env = Get-AzEnvironment -Name <name of the environment>
   Connect-AzAccount -Environment $env
   Set-AzContext -SubscriptionName <subscription name>
   Test-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
   ```

1. Le script teste la configuration de la machine virtuelle sélectionnée.

   ![Sortie de test réussi de l’extension Azure pour SAP][deployment-guide-figure-1300]

Assurez-vous que le résultat de chaque vérification d’intégrité est **OK**. Si certaines vérifications indiquent un état autre que **OK**, exécutez l’applet de commande update, comme décrit dans [Configurer l’extension Azure pour SAP][deployment-guide-4.5]. Veuillez patienter 15 minutes, puis réexécutez les vérifications décrites dans [Vérification de la disponibilité de l’extension Azure pour SAP][deployment-guide-5.1] et [Contrôle d’intégrité de la configuration de l’extension Azure pour SAP][deployment-guide-5.2]. Si les vérifications indiquent encore un problème avec certains ou tous les compteurs, consultez [Résolution des problèmes de l’extension Azure pour SAP][deployment-guide-5.3].

> [!Note]
> Vous pouvez recevoir des avertissements si vous utilisez des disques Azure standard managés. Les tests ne retournent pas « OK » et des avertissements s’affichent. Il s’agit d’un comportement normal et attendu pour ce type de disque. Voir aussi [Résolution des problèmes liés à l’extension Azure pour SAP][deployment-guide-5.3]
> 

### <a name="health-check-for-the-new-azure-extension-for-sap-configuration"></a><a name="464ac96d-7d3c-435d-a5ae-3faf3bfef4b3"></a>Contrôle d’intégrité de la configuration de la nouvelle extension Azure pour SAP

> [!NOTE]
> L’extension de machine virtuelle se présente sous deux versions. Ce chapitre traite de la nouvelle extension. Si vous avez installé l’extension de machine virtuelle par défaut, consultez le chapitre [Contrôle d’intégrité en vue de la configuration de l’extension Azure pour SAP][deployment-guide-5.2].

Si certaines des données d’infrastructure ne sont pas délivrées correctement, conformément aux indications du test décrit dans [Vérification de la préparation à l’extension Azure pour SAP][deployment-guide-5.1-new], exécutez la cmdlet `Get-AzVMExtension` afin de vérifier si l’extension Azure pour SAP est installée. La cmdlet `Test-AzVMAEMExtension` ne prend pas encore en charge la nouvelle extension. Nous mettrons à jour cet article quand ce sera le cas.

1. Assurez-vous d’avoir installé la dernière version de l’applet de commande Azure PowerShell, comme décrit dans le chapitre [Déploiement d’applets de commande Azure PowerShell][deployment-guide-4.1].
1. Exécutez l’applet de commande PowerShell suivant. Pour afficher la liste des environnements disponibles, exécutez l’applet de commande `Get-AzEnvironment`. Pour utiliser la version globale d’Azure, sélectionnez l’environnement **AzureCloud**. Pour Azure China 21Vianet, sélectionnez **AzureChinaCloud**.

   ```powershell
   $env = Get-AzEnvironment -Name <name of the environment>
   Connect-AzAccount -Environment $env
   Set-AzContext -SubscriptionName <subscription name>
   Test-AzVMAEMExtension -ResourceGroupName <resource group name> -VMName <virtual machine name>
   ```

1. La cmdlet teste la configuration de l’extension de machine virtuelle pour SAP sur la machine virtuelle sélectionnée.

### <a name="troubleshooting-azure-extension-for-sap"></a><a name="fe25a7da-4e4e-4388-8907-8abc2d33cfd8"></a>Résolution des problèmes liés à l’extension Azure pour SAP

> [!NOTE]
> L’extension de machine virtuelle se présente sous deux versions. Ce chapitre traite de l’extension par défaut. Si vous avez installé la nouvelle extension de machine virtuelle, consultez le chapitre [Résolution des problèmes de la nouvelle extension Azure pour SAP][deployment-guide-5.3-new].

#### <a name="windows-logologo_windows-azure-performance-counters-do-not-show-up-at-all"></a>![Logo Windows][Logo_Windows] Les compteurs de performances Azure ne s’affichent pas

Le service Windows AzureEnhancedMonitoring recueille les indicateurs de performance dans Azure. Si le service n’a pas été installé correctement ou s’il n’est pas exécuté sur votre machine virtuelle, aucun indicateur de performance ne peut être collecté.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>Le répertoire d’installation de l’extension Azure pour SAP est vide

###### <a name="issue"></a>Problème

Le répertoire d’installation C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\&lt;version>\\drop est vide.

###### <a name="solution"></a>Solution

L’extension n’est pas installée. Déterminez s’il s’agit d’un problème de proxy (comme décrit précédemment). Vous devrez peut-être redémarrer la machine ou réexécuter le script de configuration `Set-AzVMAEMExtension`.

##### <a name="service-for-azure-extension-for-sap-does-not-exist"></a>Le service pour l’extension Azure pour SAP n’existe pas

###### <a name="issue"></a>Problème

Le service Windows AzureEnhancedMonitoring n’existe pas.

La sortie de azperflib.exe génère une erreur :

![L’exécution d’azperflib.exe indique que le service de l’extension Azure pour SAP n’est pas en cours d’exécution][deployment-guide-figure-1400]
<a name="figure-14"></a>

###### <a name="solution"></a>Solution

Si le service n’existe pas, cela signifie que l’extension Azure pour SAP n’a pas été installée correctement. Redéployez l’extension en suivant la procédure décrite pour votre scénario de déploiement dans [Scénarios de déploiement de machines virtuelles pour SAP dans Azure][deployment-guide-3].

Une fois l’extension déployée, vérifiez de nouveau après une heure si les compteurs de performances Azure sont fournis dans la machine virtuelle Azure.

##### <a name="service-for-azure-extension-for-sap-exists-but-fails-to-start"></a>Le service pour l’extension Azure pour SAP existe, mais ne parvient pas à démarrer

###### <a name="issue"></a>Problème

Le service Windows AzureEnhancedMonitoring existe et est activé, mais ne parvient pas à démarrer. Pour plus d’informations, consultez le journal des événements de l’application.

###### <a name="solution"></a>Solution

La configuration est incorrecte. Redémarrez l’extension Azure pour SAP, comme décrit dans [Configurer l’extension Azure pour SAP][deployment-guide-4.5].

#### <a name="windows-logologo_windows-some-azure-performance-counters-are-missing"></a>![Logo Windows][Logo_Windows] Certains compteurs de performances Azure sont manquants

Le service Windows AzureEnhancedMonitoring recueille les indicateurs de performance dans Azure. Le service obtient des données provenant de plusieurs sources. Certaines données de configuration sont collectées localement, et certains indicateurs de performance sont lus à partir des diagnostics Azure. Les compteurs de stockage sont utilisés à partir de votre enregistrement au niveau de l’abonnement de stockage.

Si la résolution des problèmes à l’aide de la note SAP [1999351] ne résout pas le problème, réexécutez le script de configuration `Set-AzVMAEMExtension`. Vous devrez peut-être attendre une heure, car il est possible que les compteurs d’analyse ou de diagnostics de stockage ne soient pas créés immédiatement après avoir été activés. Si le problème persiste, ouvrez un message de support client SAP sur le composant BC-OP-NT-AZR pour Windows ou BC-OP-LNX-AZR pour une machine virtuelle Linux.

#### <a name="linux-logologo_linux-azure-performance-counters-do-not-show-up-at-all"></a>![Logo Linux][Logo_Linux] Les compteurs de performances Azure ne s’affichent pas

Les indicateurs de performance dans Azure sont collectés par un démon. Si le démon ne fonctionne pas, aucun indicateur de performance ne peut être collecté.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>Le répertoire d’installation de l’extension Azure pour SAP est vide

###### <a name="issue"></a>Problème

Le répertoire \\var\\lib\\waagent\\ ne contient pas de sous-répertoire pour l’extension Azure pour SAP.

###### <a name="solution"></a>Solution

L’extension n’est pas installée. Déterminez s’il s’agit d’un problème de proxy (comme décrit précédemment). Vous devrez peut-être redémarrer la machine et/ou réexécuter le script de configuration `Set-AzVMAEMExtension`.

##### <a name="the-execution-of-set-azvmaemextension-and-test-azvmaemextension-show-warning-messages-stating-that-standard-managed-disks-are-not-supported"></a>L’exécution de Set-AzVMAEMExtension et Test-AzVMAEMExtension affiche des messages d’avertissement indiquant que les disques managés standard ne sont pas pris en charge

###### <a name="issue"></a>Problème

Lorsque vous exécutez Set-AzVMAEMExtension ou Test-AzVMAEMExtension, des messages comme ceux qui suivent s’affichent :

<pre><code>
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
WARNING: [WARN] Standard Managed Disks are not supported. Extension will be installed but no disk metrics will be available.
</code></pre>

En exécutant azperfli.exe comme décrit plus haut, vous pouvez obtenir un résultat indiquant un état défectueux. 

###### <a name="solution"></a>Solution

Les messages s’affichent parce que les disques managés standard ne fournissent pas les API utilisées par l’extension de SAP pour Azure pour vérifier les statistiques des comptes de stockage Azure standard. Vous n’avez pas à vous en soucier. Nous avons introduit la collecte des données des comptes de stockage sur disque standard pour éviter les limitations trop fréquentes des entrées et sorties. Les disques managés évitent ces limitations en réduisant le nombre de disques dans un compte de stockage. Par conséquent, ces données ne sont pas capitales dans ce contexte.


#### <a name="linux-logologo_linux-some-azure-performance-counters-are-missing"></a>![Logo Linux][Logo_Linux] Certains compteurs de performances Azure sont manquants

Les indicateurs de performance sur Azure sont collectés par un démon, qui obtient des données de plusieurs sources. Certaines données de configuration sont collectées localement, et certains indicateurs de performance sont lus à partir des diagnostics Azure. Les compteurs de stockage proviennent des journaux d’activité de votre abonnement de stockage.

Pour obtenir une liste complète et à jour des problèmes connus, consultez la note SAP [1999351], qui contient des informations de dépannage supplémentaires pour l’extension Azure pour SAP.

Si le recours à la note SAP [1999351] ne résout pas le problème, réexécutez le script de configuration `Set-AzVMAEMExtension` comme décrit dans [Configurer l’extension Azure pour SAP][deployment-guide-4.5]. Vous devrez peut-être attendre une heure, car il est possible que les compteurs d’analyse ou de diagnostics de stockage ne soient pas créés immédiatement après avoir été activés. Si le problème persiste, ouvrez un message de support client SAP sur le composant BC-OP-NT-AZR pour Windows ou BC-OP-LNX-AZR pour une machine virtuelle Linux.

### <a name="troubleshooting-the-new-azure-extension-for-sap"></a><a name="b7afb8ef-a64c-495d-bb37-2af96688c530"></a>Résolution des problèmes liés à la nouvelle extension Azure pour SAP

> [!NOTE]
> L’extension de machine virtuelle se présente sous deux versions. Ce chapitre traite de la nouvelle extension. Si vous avez installé l’extension de machine virtuelle par défaut, consultez le chapitre [Résolution des problèmes de l’extension Azure pour SAP][deployment-guide-5.3].

#### <a name="windows-logologo_windows-azure-performance-counters-do-not-show-up-at-all"></a>![Logo Windows][Logo_Windows] Les compteurs de performances Azure ne s’affichent pas

Le processus AzureEnhancedMonitoring recueille les indicateurs de performance dans Azure. S’il ne s’exécute pas sur votre machine virtuelle, aucun indicateur de performance ne peut être collecté.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>Le répertoire d’installation de l’extension Azure pour SAP est vide

###### <a name="issue"></a>Problème

Le répertoire d’installation C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Windows\\&lt;version> est vide.

###### <a name="solution"></a>Solution

L’extension n’est pas installée. Déterminez s’il s’agit d’un problème de proxy (comme décrit précédemment). Vous devrez peut-être redémarrer l’ordinateur ou réinstaller l’extension de machine virtuelle.

#### <a name="windows-logologo_windows-some-azure-performance-counters-are-missing"></a>![Logo Windows][Logo_Windows] Certains compteurs de performances Azure sont manquants

Le processus Windows AzureEnhancedMonitoring recueille les indicateurs de performance dans Azure. Il récupère des données provenant de plusieurs sources. Certaines données de configuration sont collectées localement, certains indicateurs de performance sont lus à partir d’Azure Monitor.

Si la note SAP [1999351] ne permet pas de résoudre le problème, ouvrez un message de support client SAP sur le composant BC-OP-NT-AZR pour Windows ou BC-OP-LNX-AZR pour une machine virtuelle Linux. Joignez le fichier journal C:\\Packages\\Plugins\\Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Windows\\&lt;version>\\logapp.txt à l’incident.

#### <a name="linux-logologo_linux-azure-performance-counters-do-not-show-up-at-all"></a>![Logo Linux][Logo_Linux] Les compteurs de performances Azure ne s’affichent pas

Les indicateurs de performance dans Azure sont collectés par un démon. Si le démon ne fonctionne pas, aucun indicateur de performance ne peut être collecté.

##### <a name="the-installation-directory-of-the-azure-extension-for-sap-is-empty"></a>Le répertoire d’installation de l’extension Azure pour SAP est vide

###### <a name="issue"></a>Problème

Le répertoire \\var\\lib\\waagent\\ ne contient pas de sous-répertoire pour l’extension Azure pour SAP.

###### <a name="solution"></a>Solution

L’extension n’est pas installée. Déterminez s’il s’agit d’un problème de proxy (comme décrit précédemment). Vous devrez peut-être redémarrer l’ordinateur ou réinstaller l’extension de machine virtuelle.

#### <a name="linux-logologo_linux-some-azure-performance-counters-are-missing"></a>![Logo Linux][Logo_Linux] Certains compteurs de performances Azure sont manquants

Les indicateurs de performance sur Azure sont collectés par un démon, qui obtient des données de plusieurs sources. Certaines données de configuration sont collectées localement, certains indicateurs de performance sont lus à partir d’Azure Monitor.

Pour obtenir une liste complète et à jour des problèmes connus, consultez la note SAP [1999351], qui contient des informations de dépannage supplémentaires pour l’extension Azure pour SAP.

Si la note SAP [1999351] ne permet pas de résoudre le problème, réinstallez l’extension selon les modalités décrites dans [Configuration de l’extension Azure pour SAP][deployment-guide-4.5]. Si le problème persiste, ouvrez un message de support client SAP sur le composant BC-OP-NT-AZR pour Windows ou BC-OP-LNX-AZR pour une machine virtuelle Linux. Joignez le fichier journal /var/lib/waagent/Microsoft.AzureCAT.AzureEnhancedMonitoring.MonitorX64Linux-&lt;version>/logapp.txt à l’incident.

## <a name="azure-extension-error-codes"></a>Codes d’erreur de l’extension Azure

| ID d'erreur | Description de l'erreur | Solution |
|---|---|---|
| <a name="cfg_018"></a>cfg/018 | Configuration de l’application manquante. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_019"></a>cfg/019 | Aucun ID de déploiement dans la configuration de l’application. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_020"></a>cfg/020 | Aucun RoleInstanceId dans la configuration de l’application. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_022"></a>cfg/022 | Aucun RoleInstanceId dans la configuration de l’application. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_031"></a>cfg/031 | Impossible de lire la configuration Azure. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_021"></a>cfg/021 | Fichier de configuration de l’application manquant. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_015"></a>cfg/015 | Aucune taille de machine virtuelle dans la configuration de l’application. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_016"></a>cfg/016 | Échec du compteur GlobalMemoryStatusEx. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_023"></a>cfg/023 | Échec du compteur MaxHwFrequency. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_024"></a>cfg/024 | Échec des compteurs NIC. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_025"></a>cfg/025 | Échec du compteur de mappages de disque. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_026"></a>cfg/026 | Échec du compteur du nom du processeur. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_027"></a>cfg/027 | Échec du compteur de mappages de disque. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_038"></a>cfg/038 | Indicateur « Type de disque » manquant dans le fichier de configuration de l’extension config.xml. « Type de disque », ainsi que d’autres compteurs, a été introduit dans la version 2.2.0.68 le 16/12/2015. Si vous avez déployé l’extension avant le 16/12/2015, elle utilise l’ancien fichier de configuration. L’infrastructure d’extension Azure met à niveau l’extension automatiquement, mais le fichier config.xml reste inchangé. Pour mettre à jour la configuration, téléchargez et exécutez le dernier script d’installation PowerShell. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_039"></a>cfg/039 | Aucune mise en cache du disque. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_036"></a>cfg/036 | Aucun débit SLA sur le disque. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_037"></a>cfg/037 | Aucune IOPS SLA sur le disque. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_028"></a>cfg/028 | Échec du compteur de mappages de disque. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_029"></a>cfg/029 | Échec du compteur des dernières modifications matérielles. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_030"></a>cfg/030 | Échec des compteurs NIC. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_017"></a>cfg/017 | Modification du SID Windows due à Sysprep sur la machine virtuelle. | [Redéployer après Sysprep][deployment-guide-redeploy-after-sysprep] |
| <a name="str_007"></a>str/007 | Échec de l’accès à Storage Analytics. <br /><br />Dans la mesure où le remplissage des données Storage Analytics sur une nouvelle machine virtuelle peut prendre une demi-heure, il est possible que l’erreur disparaisse au bout d’un certain temps. Si elle persiste, réexécutez le script d’installation. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="str_010"></a>str/010 | Aucun compteur Storage Analytics. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="str_009"></a>str/009 | Échec des journaux Storage Analytics. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="wad_004"></a>wad/004 | Configuration WAD incorrecte. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="wad_002"></a>wad/002 | Format WAD inattendu. | [Contacter le support][deployment-guide-contact-support] |
| <a name="wad_001"></a>wad/001 | Compteurs WAD introuvables. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="wad_040"></a>wad/040 | Compteurs WAD obsolètes. | [Contacter le support][deployment-guide-contact-support] |
| <a name="wad_003"></a>wad/003 | Impossible de lire la table WAD. Il n’y a pas de connexion à la table WAD. Plusieurs causes possibles :<br /><br /> 1) Configuration obsolète <br />2) Aucune connexion réseau à Azure <br />3) Problèmes d’installation WAD | [Exécuter le script d’installation][deployment-guide-run-the-script]<br />[Réparer la connexion Internet][deployment-guide-fix-internet-connection]<br />[Contacter le support][deployment-guide-contact-support] |
| <a name="prf_011"></a>prf/011 | Échec des métriques NIC Perfmon. | [Contacter le support][deployment-guide-contact-support] |
| <a name="prf_012"></a>prf/012 | Échec des métriques de disque Perfmon. | [Contacter le support][deployment-guide-contact-support] |
| <a name="prf_013"></a>prf/013 | Échec de certaines métriques Perfmon. | [Contacter le support][deployment-guide-contact-support] |
| <a name="prf_014"></a>prf/014 | Échec de la création de compteur par Perfmon. | [Contacter le support][deployment-guide-contact-support] |
| <a name="cfg_035"></a>cfg/035 | Aucun fournisseur de métriques configuré. | [Contacter le support][deployment-guide-contact-support] |
| <a name="str_006"></a>str/006 | Configuration Storage Analytics incorrecte. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="str_032"></a>str/032 | Échec des métriques Storage Analytics. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="cfg_033"></a>cfg/033 | Échec de l’un des fournisseurs de métriques. | [Exécuter le script d’installation][deployment-guide-run-the-script] |
| <a name="str_034"></a>str/034 | Échec du thread du fournisseur. | [Contacter le support][deployment-guide-contact-support] |

### <a name="detailed-guidelines-on-solutions-provided"></a>Instructions détaillées sur les solutions apportées

#### <a name="run-the-setup-script"></a><a name="0d2847ad-865d-4a4c-a405-f9b7baaa00c7"></a>Exécuter le script d’installation

Suivez les étapes décrites dans le chapitre [Configuration de l’extension Azure pour SAP][deployment-guide-4.5] de ce guide pour réinstaller l’extension. Notez que le provisionnement de certains compteurs peut prendre 30 minutes.

Si les erreurs persistent, [contactez le support][deployment-guide-contact-support].

#### <a name="contact-support"></a><a name="3ba34cfc-c9bb-4648-9c3c-88e8b9130ca2"></a>Contacter le support

Il s’est produit une erreur inattendue ou il n’existe aucune solution connue. Collectez le fichier AzureEnhancedMonitoring_service.log situé dans le dossier C:\Packages\Plugins\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\\<version\>\drop (Windows) ou /var/log/azure/Microsoft.OSTCExtensions.AzureEnhancedMonitorForLinux (Linux) et contactez le support SAP pour obtenir de l’aide.

#### <a name="redeploy-after-sysprep"></a><a name="2cd61f22-187d-42ed-bb8c-def0c983d756">Redéployer après Sysprep</a>

Si vous envisagez de créer une image de système d’exploitation généralisée et préparée avec Sysprep (qui peut inclure des logiciels SAP), nous vous recommandons de ne pas y inclure l’extension Azure pour SAP. Installez l’extension Azure pour SAP après le déploiement de la nouvelle instance de l’image généralisée du système d’exploitation.

Toutefois, si votre image de système d’exploitation généralisée et préparée avec Sysprep contient déjà l’extension Azure pour SAP, vous pouvez appliquer la solution de contournement suivante pour reconfigurer l’extension, sur l’instance de machine virtuelle qui vient d’être déployée :

* Sur l’instance de machine virtuelle qui vient d’être déployée, supprimez le contenu des dossiers suivants :  
  C:\Packages\Plugins\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\\<version\>\RuntimeSettings C:\Packages\Plugins\Microsoft.AzureCAT.AzureEnhancedMonitoring.AzureCATExtensionHandler\\\<version\>\Status

* Suivez les étapes décrites dans le chapitre [Configuration de l’extension Azure pour SAP][deployment-guide-4.5] de ce guide pour réinstaller l’extension.

#### <a name="fix-internet-connection"></a><a name="e92bc57d-80d9-4a2b-a2f4-16713a22ad89"></a>Réparer la connexion Internet

La machine virtuelle Microsoft Azure exécutant l’extension Azure pour SAP a besoin d’un accès à Internet. Si elle fait partie d’un réseau virtuel Azure ou d’un domaine local, vérifiez que les paramètres de proxy nécessaires sont définis. Ces paramètres doivent également être valides pour que le compte LocalSystem puisse accéder à Internet. Suivez le chapitre [Configuration du proxy][deployment-guide-configure-proxy] de ce guide.

En outre, si vous devez définir une adresse IP statique pour votre machine virtuelle Azure, ne la définissez pas manuellement dans la machine virtuelle Azure, mais avec [Azure PowerShell](../../../virtual-network/virtual-networks-static-private-ip-arm-ps.md), [Azure CLI](../../../virtual-network/virtual-networks-static-private-ip-arm-cli.md) ou le [Portail Azure](../../../virtual-network/virtual-networks-static-private-ip-arm-pportal.md). L’adresse IP statique est propagée via le service DHCP Azure.

Il n’est pas possible de définir manuellement une adresse IP statique à l’intérieur de la machine virtuelle Azure, car cela pourrait poser problème avec l’extension Azure pour SAP.
