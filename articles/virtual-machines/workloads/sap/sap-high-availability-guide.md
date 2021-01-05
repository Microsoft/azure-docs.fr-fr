---
title: Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver
description: Dans cet article, nous allons parler des machines virtuelles Azure haute disponibilité pour SAP NetWeaver.
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
ms.date: 05/05/2017
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8e0baeb7eddb1d74a8d7708b04391134d2e188b2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005671"
---
# <a name="high-availability-azure-virtual-machines-for-sap-netweaver"></a>Machines virtuelles Azure haute disponibilité pour SAP NetWeaver

[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md

[dbms-guide]:dbms-guide.md

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



Machines virtuelles Azure est la solution idéale pour les organisations qui ont besoin de ressources de calcul, de stockage et réseau rapidement, sans longs cycles d’acquisition. Vous pouvez utiliser le service Machines virtuelles Azure pour déployer des applications classiques, comme les applications basées sur SAP NetWeaver ABAP, Java et ABAP+Java. Élargissez la fiabilité et la disponibilité sans ressources locales supplémentaires. Comme le service Machines virtuelles Azure prend en charge la connectivité intersite, vous pouvez intégrer les machines virtuelles Azure dans les domaines locaux, les clouds privés et le paysage SAP de votre organisation.

Cet article décrit les étapes à suivre pour déployer des systèmes SAP à haute disponibilité dans Azure à l’aide du modèle de déploiement Azure Resource Manager. Nous vous guidons à travers les tâches principales suivantes :

* Les guides d’installation et les notes SAP adéquats sont disponibles dans la section [Ressources][sap-ha-guide-2]. Cet article vient compléter la documentation d’installation de SAP et les notes SAP, principales ressources qui peuvent vous aider à installer et à déployer des logiciels SAP sur des plateformes spécifiques.
* Découvrir les différences entre le modèle de déploiement Azure Resource Manager et le modèle de déploiement Azure Classic.
* Découvrir les modes de quorum du clustering de basculement Windows Server, pour pouvoir sélectionner le modèle adapté à votre déploiement Azure.
* Découvrir le stockage partagé en cluster de basculement Windows Server dans les services Azure.
* Découvrir comment protéger les composants à point de défaillance unique tels qu’Advanced Business Application Programming (ABAP) SAP Central Services (ASCS)/SAP Central Services (SCS) et les systèmes de gestion de bases de données (SGBD), ainsi que les composants redondants comme les serveurs d’applications SAP, dans Azure.
* Suivre un exemple pas à pas d’installation et de configuration d’un système SAP à haute disponibilité dans un cluster de clustering de basculement Windows Server dans Azure en utilisant Azure Resource Manager.
* Découvrir les étapes supplémentaires requises pour utiliser le clustering de basculement Windows Server dans Azure, mais qui ne sont pas nécessaires dans le cadre d’un déploiement local.

Pour simplifier le déploiement et la configuration, dans cet article, nous utilisons les modèles Resource Manager SAP à trois niveaux et haute disponibilité. Les modèles permettent d’automatiser le déploiement de toute l’infrastructure dont vous avez besoin pour un système SAP à haute disponibilité. L’infrastructure prend également en charge le dimensionnement SAP Application Performance Standard (SAPS) de votre système SAP.

## <a name="prerequisites"></a><a name="217c5479-5595-4cd8-870d-15ab00d4f84c"></a> Conditions préalables
Avant de commencer, assurez-vous que vous remplissez les conditions préalables décrites dans les sections suivantes. Veillez également à consulter toutes les ressources répertoriées dans la section [Ressources][sap-ha-guide-2].

Dans cet article, nous utilisons des modèles Azure Resource Manager pour [SAP NetWeaver à trois niveaux à l’aide de la fonctionnalité Disques managés](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md/). Pour une vue d’ensemble utile des modèles Azure Resource Manager SAP, consultez [cet article](/archive/blogs/saponsqlserver/azure-quickstart-templates-for-sap).

## <a name="resources"></a><a name="42b8f600-7ba3-4606-b8a5-53c4f026da08"></a> Ressources
Ces articles couvrent les déploiements SAP dans Azure :

* [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver][planning-guide]
* [Déploiement de machines virtuelles Azure pour SAP NetWeaver][deployment-guide]
* [Déploiement SGBD de machines virtuelles Azure pour SAP NetWeaver][dbms-guide]
* [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver (ce guide)][sap-ha-guide]

> [!NOTE]
> Dans la mesure du possible, nous vous fournissons un lien vers le guide d’installation de SAP concerné (consultez les [Guides d’installation de SAP][sap-installation-guides]). Pour plus d’informations sur les conditions préalables à l’installation et la procédure d’installation, il est recommandé de lire attentivement les guides d’installation de SAP NetWeaver. Cet article traite uniquement des tâches spécifiques pour les systèmes basés sur SAP NetWeaver que vous pouvez utiliser avec le service Machines virtuelles Azure.
>
>

Les notes SAP suivantes sont associées à la question de SAP dans Azure :

| Numéro de la note | Intitulé |
| --- | --- |
| [1928533] |Applications SAP sur Azure : Produits et tailles pris en charge |
| [2015553] |SAP sur Microsoft Azure : prérequis pour le support |
| [1999351] |Surveillance Azure améliorée pour SAP |
| [2178632] |Métriques de surveillance clés pour SAP sur Microsoft Azure |
| [1999351] |Virtualisation sur Windows : supervision améliorée |
| [2243692] |Utilisation du stockage SSD Azure Premium pour l’instance de SGBD SAP |

Prenez connaissance des [limites des abonnements Azure][azure-resource-manager/management/azure-subscription-service-limits-subscription], y compris des limites par défaut générales et des limites maximales.

## <a name="high-availability-sap-with-azure-resource-manager-vs-the-azure-classic-deployment-model"></a><a name="42156640c6-01cf-45a9-b225-4baa678b24f1"></a>Système SAP à haute disponibilité avec modèle de déploiement Azure Resource Manager ou Classic
Les modèles de déploiement Azure Resource Manager et Azure Classic présentent les différences suivantes :

- Groupes de ressources
- Équilibrage de charge interne Azure dépendant du groupe de ressources Azure
- Prise en charge des scénarios SAP multi-SID

### <a name="resource-groups"></a><a name="f76af273-1993-4d83-b12d-65deeae23686"></a> Groupes de ressources
Dans Azure Resource Manager, vous pouvez utiliser des groupes de ressources pour gérer l’ensemble des ressources d’application de votre abonnement Azure. Dans une approche intégrée, toutes les ressources d’un groupe de ressources présentent le même cycle de vie. Par exemple, toutes les ressources sont créées en même temps et supprimées au même moment. En savoir plus sur les [groupes de ressources](../../../azure-resource-manager/management/overview.md#resource-groups).

### <a name="azure-internal-load-balancer-dependency-on-the-azure-resource-group"></a><a name="3e85fbe0-84b1-4892-87af-d9b65ff91860"></a> Équilibrage de charge interne Azure dépendant du groupe de ressources Azure

Dans le modèle de déploiement Azure Classic, il existe une dépendance entre l’équilibreur de charge interne Azure (service Azure Load Balancer) et le service cloud. Chaque équilibreur de charge interne a besoin d’un service cloud.

Dans le Gestionnaire de ressources Azure, toutes les ressources Azure doivent être placées dans un groupe de ressources Azure. Ceci est également valable pour Azure Load Balancer. Toutefois, il est inutile d’avoir un groupe de ressources Azure par équilibreur de charge Azure, par exemple, un groupe de ressources Azure peut contenir plusieurs équilibreurs de charge Azure. L’environnement est plus simple et plus flexible. 

### <a name="support-for-sap-multi-sid-scenarios"></a>Prise en charge des scénarios SAP multi-SID

Dans Azure Resource Manager, vous pouvez installer plusieurs instances d’identificateur système SAP (SID) ASCS/SCS dans un cluster. Il est possible d’utiliser des instances multi-SID en raison de la prise en charge de plusieurs adresses IP pour chaque équilibreur de charge interne Azure.

Pour utiliser le modèle de déploiement Azure Classic, suivez les procédures décrites dans [SAP NetWeaver dans Azure : Clustering d’instances SAP ASCS/SCS à l’aide d’un cluster de basculement Windows Server dans Azure avec SIOS DataKeeper](https://go.microsoft.com/fwlink/?LinkId=613056).

> [!IMPORTANT]
> Nous vous recommandons vivement d’utiliser le modèle de déploiement Azure Resource Manager pour vos installations SAP. Celui-ci offre de nombreux avantages par rapport au modèle de déploiement Classic. En savoir plus sur les [modèles de déploiement][virtual-machines-azure-resource-manager-architecture-benefits-arm] Azure.   
>
>

## <a name="windows-server-failover-clustering"></a><a name="8ecf3ba0-67c0-4495-9c14-feec1a2255b7"></a> Clustering de basculement Windows Server
Le clustering de basculement Windows Server constitue la base d’une installation de SGBD et de SAP ASCS/SCS à haute disponibilité dans Windows.

Un cluster de basculement est un groupe de 1 + n serveurs indépendants (nœuds) qui fonctionnent ensemble pour accroître la disponibilité des applications et des services. En cas d’échec d’un nœud, le clustering de basculement Windows Server calcule le nombre d’échecs qui peuvent se produire tout en maintenant l’intégrité du cluster afin d’être en mesure de fournir les applications et les services. Différents modes de quorum sont disponibles pour obtenir un clustering de basculement.

### <a name="quorum-modes"></a><a name="1a3c5408-b168-46d6-99f5-4219ad1b1ff2"></a> Modes de quorum
Lorsque vous utilisez le clustering de basculement Windows Server, vous pouvez choisir parmi quatre modes de quorum :

* **Nœud majoritaire** : chaque nœud du cluster peut voter. Le cluster fonctionne uniquement avec une majorité de voix, c’est-à-dire plus de la moitié. Nous recommandons cette option pour les clusters qui présentent un nombre impair de nœuds. Par exemple, si trois nœuds d’un cluster à sept nœuds échouent, celui-ci obtient toujours la majorité et continue à s’exécuter.  
* **Nœud et disque majoritaires** : chaque nœud et un disque désigné dans l’espace de stockage en cluster (disque témoin) peuvent voter lorsqu’ils sont disponibles et en communication. Le cluster fonctionne uniquement avec une majorité de voix, c’est-à-dire plus de la moitié. Ce mode est pertinent dans un environnement de cluster comprenant un nombre pair de nœuds. Si la moitié des nœuds et le disque sont en ligne, le cluster conserve son état d’intégrité.
* **Nœud et partage de fichiers majoritaires** : chaque nœud et un partage de fichiers désigné (témoin de partage de fichiers) créé par l’administrateur peuvent voter, que les nœuds et le partage de fichiers soient ou non disponibles et en communication. Le cluster fonctionne uniquement avec une majorité de voix, c’est-à-dire plus de la moitié. Ce mode est pertinent dans un environnement de cluster comprenant un nombre pair de nœuds. Il est similaire au mode Nœud et disque majoritaires, mais il utilise un témoin de partage de fichiers au lieu d’un disque témoin. Ce mode est facile à implémenter, mais si le partage de fichiers lui-même n’est pas hautement disponible, il risque de devenir un point de défaillance unique.
* **Sans majorité : Disque uniquement**. un quorum est atteint pour le cluster si un nœud est disponible et en communication avec un disque spécifique dans l’espace de stockage en cluster. Seuls les nœuds qui sont également en communication avec ce disque peuvent rejoindre le cluster. Nous vous déconseillons d’utiliser ce mode.


## <a name="windows-server-failover-clustering-on-premises"></a><a name="fdfee875-6e66-483a-a343-14bbaee33275"></a> Clustering de basculement Windows Server local
La figure 1 illustre un cluster de deux nœuds. Si la connexion réseau entre les nœuds échoue et les deux nœuds restent opérationnels, un disque ou un partage de fichiers quorum détermine quel nœud continuera de fournir les applications et services du cluster. Le nœud qui a accès au disque ou au partage de fichiers quorum est celui qui garantit que les services restent disponibles.

Comme un cluster à deux nœuds est utilisé dans cet exemple, nous faisons appel au mode de quorum Nœud et partage de fichiers majoritaires. Le mode Nœud et disque majoritaires est également une option valide. Dans un environnement de production, nous vous recommandons d’utiliser un disque quorum. Vous pouvez utiliser une technologie de système de stockage et de réseau pour le rendre hautement disponible.

![Figure 1 : Exemple de configuration de clustering de basculement Windows Server pour SAP ASCS/SCS dans Azure][sap-ha-guide-figure-1000]

_**Figure 1 :** Exemple de configuration de clustering de basculement Windows Server pour SAP ASCS/SCS dans Azure_

### <a name="shared-storage"></a><a name="be21cf3e-fb01-402b-9955-54fbecf66592"></a> Stockage partagé
La figure 1 illustre également un cluster de stockage partagé à deux nœuds. Dans un cluster de stockage partagé local, tous les nœuds du cluster détectent le stockage partagé. Un mécanisme de verrouillage protège les données contre toute altération. Tous les nœuds peuvent détecter si un autre nœud échoue. Si un nœud échoue, le nœud restant s’approprie les ressources de stockage et garantit la disponibilité des services.

> [!NOTE]
> Avec certaines applications de SGBD, telles que SQL Server, vous n’avez pas besoin de disques partagés pour atteindre une haute disponibilité. SQL Server Always On assure la réplication des données et des fichiers de journaux du SGBD à partir du disque local d’un nœud du cluster vers le disque local d’un autre nœud du cluster. Dans ce cas, la configuration du cluster Windows ne nécessite pas de disque partagé.
>
>

### <a name="networking-and-name-resolution"></a><a name="ff7a9a06-2bc5-4b20-860a-46cdb44669cd"></a> Mise en réseau et résolution de noms
Les ordinateurs clients atteignent le cluster via une adresse IP virtuelle et un nom d’hôte virtuel fournis par le serveur DNS. Les nœuds locaux et le serveur DNS peuvent gérer plusieurs adresses IP.

Dans une installation type, vous utilisez deux ou plusieurs connexions réseau :

* Une connexion dédiée au stockage
* Une connexion de réseau interne au cluster pour la pulsation
* Un réseau public utilisé par les clients pour se connecter au cluster

## <a name="windows-server-failover-clustering-in-azure"></a><a name="2ddba413-a7f5-4e4e-9a51-87908879c10a"></a> Clustering de basculement Windows Server dans Azure
Par rapport aux déploiements complets ou de cloud privé, le service Machines virtuelles Azure requiert des étapes supplémentaires pour configurer le clustering de basculement Windows Server. Lorsque vous créez un disque de cluster partagé, vous devez définir plusieurs adresses IP et noms d’hôtes virtuels pour l’instance SAP ASCS/SCS.

Cet article présente les concepts clés et les étapes supplémentaires requises lors de la création d’un cluster de services centraux SAP à haute disponibilité dans Azure. Nous vous expliquons comment configurer l’outil tiers SIOS DataKeeper et l’équilibrage de charge interne Azure. Vous pouvez utiliser ces outils pour créer un cluster de basculement Windows avec un témoin de partage de fichiers dans Azure.

![Figure 2 : Configuration du clustering de basculement Windows Server dans Azure sans disque partagé][sap-ha-guide-figure-1001]

_**Figure 2 :** Configuration du clustering de basculement Windows Server dans Azure sans disque partagé_

### <a name="shared-disk-in-azure-with-sios-datakeeper"></a><a name="1a464091-922b-48d7-9d08-7cecf757f341"></a> Disque partagé dans Azure avec SIOS DataKeeper
Vous avez besoin d’un stockage partagé en cluster pour bénéficier d’une instance SAP ASCS/SCS à haute disponibilité. Au mois de septembre 2016, Azure n’offre pas de stockage partagé pour créer un cluster de stockage partagé. Vous pouvez utiliser le logiciel tiers SIOS DataKeeper Cluster Edition pour créer un stockage en miroir qui simule un stockage partagé en cluster. La solution SIOS assure la réplication synchrone des données en temps réel. Voici comment créer une ressource de disque partagé pour un cluster :

1. Attachez un disque supplémentaire à chaque machine virtuelle se trouvant dans une configuration de cluster Windows.
2. Exécutez SIOS DataKeeper Cluster Edition sur les deux nœuds de machine virtuelle.
3. Configurez SIOS DataKeeper Cluster Edition de sorte qu’il mette en miroir le contenu du volume attaché au disque supplémentaire de la machine virtuelle source vers le volume attaché au disque supplémentaire de la machine virtuelle cible. SIOS DataKeeper extrait les volumes locaux source et cible, puis les présente au clustering de basculement Windows Server comme un seul disque partagé.

Vous trouverez plus d’informations sur SIOS DataKeeper [ici](https://us.sios.com/products/datakeeper-cluster/).

![Figure 3 : Configuration du clustering de basculement Windows Server dans Azure avec SIOS DataKeeper][sap-ha-guide-figure-1002]

_**Figure 3 :** Configuration du clustering de basculement Windows Server dans Azure avec SIOS DataKeeper_

> [!NOTE]
> Avec certains SGBD, tels que SQL Server, vous n’avez pas besoin de disques partagés pour atteindre une haute disponibilité. SQL Server Always On assure la réplication des données et des fichiers de journaux du SGBD à partir du disque local d’un nœud du cluster vers le disque local d’un autre nœud du cluster. Dans ce cas, la configuration du cluster Windows ne nécessite pas de disque partagé.
>
>

### <a name="name-resolution-in-azure"></a><a name="44641e18-a94e-431f-95ff-303ab65e0bcb"></a> Résolution de noms dans Azure
La plateforme cloud Azure ne permet pas de configurer des adresses IP virtuelles telles que des adresses IP flottantes. Vous avez besoin d’une autre solution pour configurer une adresse IP virtuelle afin d’atteindre la ressource de cluster dans le cloud.
Azure offre un équilibrage de charge interne dans le cadre du service Azure Load Balancer. Avec l’équilibrage de charge interne, les clients atteignent le cluster via son adresse IP virtuelle.
Vous devez déployer l’équilibrage de charge interne dans le groupe de ressources qui contient les nœuds de cluster. Ensuite, configurez toutes les règles de réacheminement de port nécessaires avec les ports de sondage de l’équilibrage de charge interne.
Les clients peuvent se connecter avec le nom d’hôte virtuel. Le serveur DNS résout l’adresse IP du cluster et l’équilibrage de charge interne gère le réacheminement de port vers le nœud actif du cluster.

## <a name="sap-netweaver-high-availability-in-azure-infrastructure-as-a-service-iaas"></a><a name="2e3fec50-241e-441b-8708-0b1864f66dfa"></a> Haute disponibilité pour SAP NetWeaver dans Azure IaaS (Infrastructure as a Service)
Pour atteindre une haute disponibilité des applications SAP, par exemple pour les composants logiciels SAP, vous devez protéger les composants suivants :

* Instance de serveur d’applications SAP
* Instance SAP ASCS/SCS
* Serveur SGBD

Pour plus d’informations sur la protection des composants SAP dans des scénarios de haute disponibilité, consultez [Planification et implémentation de machines virtuelles Azure pour SAP NetWeaver][planning-guide-11].

### <a name="high-availability-sap-application-server"></a><a name="93faa747-907e-440a-b00a-1ae0a89b1c0e"></a> Serveur d’applications SAP à haute disponibilité
En règle générale, vous n’avez pas besoin d’une solution à haute disponibilité pour le serveur d’applications et les instances de dialogue SAP. La haute disponibilité s’obtient par redondance, et vous configurez plusieurs instances de dialogue sur différentes instances de machines virtuelles Azure. Vous devez avoir au moins deux instances d’applications SAP installées dans deux instances de machines virtuelles Azure.

![Figure 4 : Serveur d’applications SAP à haute disponibilité][sap-ha-guide-figure-2000]

_**Figure 4 :** Serveur d’applications SAP à haute disponibilité_

Vous devez placer toutes les machines virtuelles qui hébergent des instances de serveur d’applications SAP dans le même groupe à haute disponibilité Azure. Un groupe à haute disponibilité Azure garantit que :

* Toutes les machines virtuelles font partie du même domaine de mise à niveau. Un domaine de mise à niveau permet par exemple d’éviter que les machines virtuelles soient mises à jour en même temps lors d’une interruption de maintenance planifiée.
* Toutes les machines virtuelles font partie du même domaine d’erreur. Un domaine d’erreur permet par exemple de s’assurer que les machines virtuelles soient déployées de sorte qu’aucun point de défaillance unique n’affecte la disponibilité de l’ensemble des machines virtuelles.

Découvrez comment [Gérer la disponibilité des machines virtuelles][../manage-availability.md].

Disque non managé uniquement : Comme le compte de stockage Azure constitue un point de défaillance unique potentiel, il est important de disposer d’au moins deux comptes de stockage Azure sur lesquels au moins deux machines virtuelles sont distribuées. Dans une configuration idéale, les disques de chaque machine virtuelle exécutant une instance de dialogue SAP sont déployés dans un compte de stockage différent.

### <a name="high-availability-sap-ascsscs-instance"></a><a name="f559c285-ee68-4eec-add1-f60fe7b978db"></a> Instance SAP ASCS/SCS à haute disponibilité
La figure 5 est un exemple d’instance SAP ASCS/SCS à haute disponibilité.

![Figure 5 : Instance SAP ASCS/SCS à haute disponibilité][sap-ha-guide-figure-2001]

_**Figure 5 :** Instance SAP ASCS/SCS à haute disponibilité_

#### <a name="sap-ascsscs-instance-high-availability-with-windows-server-failover-clustering-in-azure"></a><a name="b5b1fd0b-1db4-4d49-9162-de07a0132a51"></a> Haute disponibilité de l’instance SAP ASCS/SCS avec le clustering de basculement Windows Server dans Azure
Par rapport aux déploiements complets ou de cloud privé, le service Machines virtuelles Azure requiert des étapes supplémentaires pour configurer le clustering de basculement Windows Server. Afin de créer un cluster de basculement Windows, vous avez besoin d’un disque de cluster partagé, de plusieurs adresses IP et noms d’hôtes virtuels, et d’un équilibrage de charge interne Azure pour le clustering d’une instance SAP ASCS/SCS. Nous approfondirons ce point plus loin dans cet article.

![Figure 6 : Configuration du clustering de basculement Windows Server pour SAP ASCS/SCS dans Azure à l’aide de SIOS DataKeeper][sap-ha-guide-figure-1002]

_**Figure 6 :** Configuration du clustering de basculement Windows Server pour une instance SAP ASCS/SCS dans Azure avec SIOS DataKeeper_

### <a name="high-availability-dbms-instance"></a><a name="ddd878a0-9c2f-4b8e-8968-26ce60be1027"></a> Instance SGBD à haute disponibilité
Le SGBD constitue également un point de défaillance unique d’un système SAP. Vous devez le protéger à l’aide d’une solution à haute disponibilité. La figure 7 montre une solution à haute disponibilité SQL Server Always On dans Azure, avec le clustering de basculement Windows Server et l’équilibrage de charge interne Azure. SQL Server Always On réplique les fichiers journaux et les données du SGBD à l’aide de sa propre réplication de SGBD. Dans ce cas, vous n’avez pas besoin de disques partagés en cluster, ce qui simplifie l’ensemble de la configuration.

![Figure 7 : Exemple de SGBD SAP à haute disponibilité avec SQL Server Always On][sap-ha-guide-figure-2003]

_**Figure 7 :** Exemple de SGBD SAP à haute disponibilité avec SQL Server Always On_

Pour plus d’informations sur le clustering de SQL Server dans Azure à l’aide du modèle de déploiement Azure Resource Manager, consultez les articles suivants :

* [Configuration manuelle d’un groupe de disponibilité Always On dans une machine virtuelle Azure à l’aide de Resource Manager][virtual-machines-windows-portal-sql-alwayson-availability-groups-manual]
* [Configurer un équilibrage de charge interne Azure pour un groupe de disponibilité Always On dans Azure][virtual-machines-windows-portal-sql-alwayson-int-listener]

## <a name="end-to-end-high-availability-deployment-scenarios"></a><a name="045252ed-0277-4fc8-8f46-c5a29694a816"></a> Scénarios de déploiement à haute disponibilité de bout en bout

### <a name="deployment-scenario-using-architectural-template-1"></a>Scénario de déploiement à l’aide du modèle architectural n°1

La figure 8 illustre une architecture SAP NetWeaver à haute disponibilité dans Azure pour **un** système SAP. Ce scénario est défini comme suit :

- Un cluster dédié est utilisé pour l’instance SAP ASCS/SCS.
- Un cluster dédié est utilisé pour l’instance SGBD.
- Des instances de serveurs d’applications SAP sont déployées dans des machines virtuelles dédiées.

![Figure 8 : Modèle d’architecture SAP à haute disponibilité n°1, avec un cluster dédié à ASCS/SCS et SGBD][sap-ha-guide-figure-2004]

_**Figure 8 :** Modèle d’architecture SAP à haute disponibilité n°1, clusters dédiés à ASCS/SCS et SGBD_

### <a name="deployment-scenario-using-architectural-template-2"></a>Scénario de déploiement à l’aide du modèle architectural n°2

La figure 9 illustre une architecture SAP NetWeaver à haute disponibilité dans Azure pour **un** système SAP. Ce scénario est défini comme suit :

- Un cluster dédié est utilisé **à la fois** pour l’instance SAP ASCS/SCS et SGBD.
- Des instances de serveurs d’applications SAP sont déployées dans leurs propres machines virtuelles dédiées.

![Figure 9 : Modèle d’architecture SAP à haute disponibilité n°2, avec un cluster dédié à l’instance ASCS/SCS et un cluster dédié à SGBD][sap-ha-guide-figure-2005]

_**Figure 9 :** Modèle d’architecture SAP à haute disponibilité n°2, avec un cluster dédié à l’instance ASCS/SCS et un cluster dédié à SGBD_

### <a name="deployment-scenario-using-architectural-template-3"></a>Scénario de déploiement à l’aide du modèle architectural n°3

La figure 10 montre une architecture SAP NetWeaver à haute disponibilité dans Azure pour **deux** systèmes SAP, avec &lt;SID1&gt; et &lt;SID2&gt;. Ce scénario est défini comme suit :

- Un cluster dédié est utilisé **à la fois** à l’instance SAP ASCS/SCS SID1 *et* à l’instance SAP ASCS/SCS SID2 (un cluster).
- Un cluster dédié est utilisé pour SGBD SID1, et un autre cluster dédié est utilisé pour SGBD SID2 (deux clusters).
- Les instances de serveurs d’applications SAP du système SAP SID1 utilisent leurs propres machines virtuelles dédiées.
- Les instances de serveurs d’applications SAP du système SAP SID2 utilisent leurs propres machines virtuelles dédiées.

![Figure 10 : Modèle d’architecture SAP à haute disponibilité n°3, avec un cluster dédié aux différentes instances ASCS/SCS][sap-ha-guide-figure-6003]

_**Figure 10 :** Modèle d’architecture SAP à haute disponibilité n°3, avec un cluster dédié aux différentes instances ASCS/SCS_

## <a name="prepare-the-infrastructure"></a><a name="78092dbe-165b-454c-92f5-4972bdbef9bf"></a>réparer l’infrastructure

### <a name="prepare-the-infrastructure-for-architectural-template-1"></a>Préparer l’infrastructure du modèle d’architecture n°1
Les modèles Azure Resource Manager pour SAP simplifient le déploiement des ressources requises.

Les modèles à trois niveaux dans Azure Resource Manager prennent également en charge les scénarios de haute disponibilité, comme dans le modèle d’architecture nº1, qui présente deux clusters. Chaque cluster constitue un point de défaillance pour l’instance SAP ASCS/SCS et l’instance SGBD.

Voici où vous pouvez obtenir des modèles Azure Resource Manager pour l’exemple de scénario décrit dans cet article :

* [Image de Place de marché Azure](https://github.com/Azure/azure-quickstart-templates/)  
* [Image de Place de marché Azure au moyen de la fonctionnalité Disques managés](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-md)  
* [Image personnalisée](https://github.com/Azure/azure-quickstart-templates/)
* [Image personnalisée à l’aide de la fonctionnalité Disques managés](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-md)

Pour préparer l’infrastructure du modèle d’architecture n°1 :

- Dans le portail Azure, panneau **Paramètres**, boîte de dialogue **SYSTEMAVAILABILITY** (disponibilité du système), sélectionnez **HA** (Haute disponibilité).

  ![Figure 11 : Définir les paramètres Azure Resource Manager de haute disponibilité SAP][sap-ha-guide-figure-3000]

_**Figure 11 :** Définir les paramètres Azure Resource Manager de haute disponibilité SAP_


  Les modèles créent :

  * **Des machines virtuelles** :
    * Des machines virtuelles de serveur d’applications SAP : <*SIDSystèmeSAP*>-di-<*Numéro*>
    * Des machines virtuelles de cluster ASCS/SCS : <*SIDSystèmeSAP*>-ascs-<*Numéro*>
    * Cluster SGBD : <*SIDSystèmeSAP*>-db-<*Numéro*>

  * **Des cartes réseau pour toutes les machines virtuelles, avec une adresse IP associée** :
    * <*SIDSystèmeSAP*&gt;-nic-di-&lt;*Numéro*>
    * <*SIDSystèmeSAP*&gt;-nic-ascs-&lt;*Numéro*>
    * <*SIDSystèmeSAP*&gt;-nic-db-&lt;*Numéro*>

  * **Comptes de stockage Azure (disques non managés uniquement)**

  * **Des groupes de disponibilité** pour :
    * Les machines virtuelles de serveur d’applications SAP : <*SIDSystèmeSAP*>-avset-di
    * Les machines virtuelles de cluster SAP ASCS/SCS : <*SIDSystèmeSAP*>-avset-ascs
    * Les machines virtuelles de cluster SGBD : <*SIDSystèmeSAP*>-avset-db

  * **Un équilibrage de charge interne Azure** :
    * Avec tous les ports pour l’instance ASCS/SCS et l’adresse IP <*SIDSystèmeSAP*>-lb-ascs
    * Avec tous les ports pour l’instance SGBD et l’adresse IP <*SIDSystèmeSAP*>-lb-db

  * **Un groupe de sécurité réseau** : &lt;*SIDSystèmeSAP*&gt;-nsg-ascs-0  
    * Avec un port RDP (Remote Desktop Protocol) ouvert vers la machine virtuelle <*SAPSystemSID*>-ascs-0

> [!NOTE]
> Toutes les adresses IP des cartes réseau et des équilibrages de charge internes Azure sont **dynamiques** par défaut. Définissez-les en tant qu’adresses IP **statiques**. La marche à suivre est décrite plus loin dans cet article.
>
>

### <a name="deploy-virtual-machines-with-corporate-network-connectivity-cross-premises-to-use-in-production"></a><a name="c87a8d3f-b1dc-4d2f-b23c-da4b72977489"></a> Déployer des machines virtuelles avec connectivité réseau d’entreprise (intersite) pour une utilisation en production
Pour les systèmes SAP de production, déployez les machines virtuelles Azure avec une connectivité de réseau d’entreprise à l’aide d’un VPN de site à site Azure ou d’Azure ExpressRoute.

> [!NOTE]
> Vous pouvez utiliser votre instance de réseau virtuel Azure. Le réseau virtuel et le sous-réseau ont déjà été créés et préparés.
>
>

1. Dans le portail Azure, panneau **Paramètres**, boîte de dialogue **NEWOREXISTINGSUBNET** (sous-réseau nouveau ou existant), sélectionnez **existing** (existant).
2. Dans la boîte de dialogue **SUBNETID** (ID de sous-réseau), vous devez ajouter la chaîne complète de l’ID du sous-réseau Azure préparé sur lequel vous envisagez de déployer vos machines virtuelles Azure.
3. Pour obtenir la liste de tous les sous-réseaux du réseau Azure, exécutez cette commande PowerShell :

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets
   ```

   Le champ **ID** indique l’ID de sous-réseau (**SUBNETID**).
4. Pour obtenir la liste de toutes les valeurs **SUBNETID** (ID de sous-réseau), exécutez cette commande PowerShell :

   ```powershell
   (Get-AzVirtualNetwork -Name <azureVnetName>  -ResourceGroupName <ResourceGroupOfVNET>).Subnets.Id
   ```

   Le **SUBNETID** se présente ainsi :

   ```
   /subscriptions/<SubscriptionId>/resourceGroups/<VPNName>/providers/Microsoft.Network/virtualNetworks/azureVnet/subnets/<SubnetName>
   ```

### <a name="deploy-cloud-only-sap-instances-for-test-and-demo"></a><a name="7fe9af0e-3cce-495b-a5ec-dcb4d8e0a310"></a> Déployer des instances SAP dans le cloud uniquement à des fins de test et de démonstration
Vous pouvez déployer votre système SAP à haute disponibilité dans un modèle de déploiement cloud uniquement. Ce type de déploiement est surtout utile pour les démonstrations et les tests. Il ne convient pas à une utilisation en production.

- Dans le portail Azure, panneau **Paramètres**, boîte de dialogue **NEWOREXISTINGSUBNET** (sous-réseau nouveau ou existant), sélectionnez **new** (nouveau). Laissez le champ **SUBNETID** vide.

  Le modèle Azure Resource Manager SAP crée automatiquement le réseau virtuel et le sous-réseau Azure.

> [!NOTE]
> Vous devez également déployer au moins une machine virtuelle dédiée pour Active Directory et le service DNS dans la même instance de réseau virtuel Azure. Le modèle ne crée pas ces machines virtuelles.
>
>


### <a name="prepare-the-infrastructure-for-architectural-template-2"></a>Préparer l’infrastructure du modèle d’architecture n°2

Vous pouvez utiliser ce modèle Azure Resource Manager pour SAP afin de simplifier le déploiement des ressources d’infrastructure requises pour le modèle d’architecture SAP n°2.

Voici où vous pouvez obtenir des modèles Azure Resource Manager pour ce scénario de déploiement :

* [Image de Place de marché Azure](https://github.com/Azure/azure-quickstart-templates/)  
* [Image de Place de marché Azure au moyen de la fonctionnalité Disques managés](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-marketplace-image-converged-md)  
* [Image personnalisée](https://github.com/Azure/azure-quickstart-templates/)
* [Image personnalisée à l’aide de la fonctionnalité Disques managés](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-3-tier-user-image-converged-md)


### <a name="prepare-the-infrastructure-for-architectural-template-3"></a>Préparer l’infrastructure du modèle d’architecture n°3

Vous pouvez préparer l’infrastructure et configurer SAP pour **multi-SID**. Par exemple, vous pouvez ajouter une instance SAP ASCS/SCS supplémentaire à une configuration de cluster *existante*. Pour plus d’informations, consultez [Configurer une instance SAP ASCS/SCS supplémentaire dans une configuration de cluster existante pour créer une configuration SAP multi-SID dans Azure Resource Manager][sap-ha-multi-sid-guide].

Pour créer un cluster multi-SID, vous pouvez utiliser les [modèles de démarrage rapide multi-SID sur GitHub](https://github.com/Azure/azure-quickstart-templates).
Pour créer un nouveau cluster multi-SID, vous devez déployer les trois modèles suivants :

* [Modèle ASCS/SCS](#ASCS-SCS-template)
* [Modèle de base de données](#database-template)
* [Modèle de serveurs d’applications](#application-servers-template)

Les sections suivantes fournissent plus de détails sur les modèles et les paramètres que vous devez fournir dans les modèles.

#### <a name="ascsscs-template"></a><a name="ASCS-SCS-template"></a> Modèle ASCS/SCS

Le modèle ASCS/SCS déploie deux machines virtuelles que vous pouvez utiliser pour créer un cluster de basculement Windows hébergeant plusieurs instances ASCS/SCS.

Pour définir le modèle multi-SID ASCS/SCS, dans le [modèle multi-SID ASCS/SCS][sap-templates-3-tier-multisid-xscs-marketplace-image] ou le [modèle multi-SID ASCS/SCS avec Disques managés][sap-templates-3-tier-multisid-xscs-marketplace-image-md], entrez des valeurs pour les paramètres suivants :

  - **Préfixe de ressource**.  Définissez le préfixe de ressource servant à attribuer un préfixe à toutes les ressources créées lors du déploiement. Comme les ressources n’appartiennent pas à un seul système SAP, le préfixe de la ressource n’est pas le SID d’un système SAP.  Le préfixe doit contenir entre **trois et six caractères**.
  - **Type de pile**. Sélectionnez le type de pile du système SAP. Selon le type de pile, Azure Load Balancer a une adresse IP privée (ABAP ou Java uniquement) ou deux adresses IP privées (ABAP + Java) par système SAP.
  -  **Type de système d’exploitation**. Sélectionnez le système d’exploitation des machines virtuelles.
  -  **Nombre de systèmes SAP**. Sélectionnez le nombre de systèmes SAP que vous souhaitez installer dans ce cluster.
  -  **Disponibilité du système**. Sélectionnez la haute disponibilité **(HA)** .
  -  **Nom d’utilisateur et mot de passe d’administrateur**. Créez un nouvel utilisateur pouvant être utilisé pour ouvrir une session sur la machine.
  -  **Sous-réseau nouveau ou existant**. Déterminez s’il faut créer un réseau virtuel et un sous-réseau, ou utiliser un sous-réseau existant. Si vous disposez déjà d’un réseau virtuel connecté à votre réseau local, sélectionnez **existant**.
  -  **ID du sous-réseau**. Si vous voulez déployer la machine virtuelle dans un réseau virtuel existant où vous avez défini un sous-réseau auquel la machine virtuelle doit être attribuée, nommez l’ID de ce sous-réseau spécifique. L’ID se présente généralement comme suit : /subscriptions/<*id_abonnement*>/resourceGroups/<*nom_groupe_ressources*>/providers/Microsoft.Network/virtualNetworks/<*nom_réseau_virtuel*>/subnets/<*nom_sous-réseau*>

Le modèle déploie une instance d’équilibrage de charge Azure qui prend en charge plusieurs systèmes SAP.

- Les instances ASCS sont configurées pour les numéros 00, 10, 20...
- Les instances SCS sont configurées pour les numéros 01, 11, 21...
- Les instances ASCS Enqueue Replication Server (ERS) (Linux uniquement) sont configurées pour les numéros 02, 12, 22...
- Les instances SCS ERS (Linux uniquement) sont configurées pour les numéros 03, 13, 23...

L’équilibrage de charge contient 1 adresse IP virtuelle (2 pour Linux), 1 adresse IP virtuelle ASCS/SCS et 1 adresse IP virtuelle pour ERS (Linux uniquement).

La liste suivante contient toutes les règles d’équilibrage de charge (où x représente le numéro du système SAP, par exemple, 1, 2, 3...) :
- Ports spécifiques de Windows pour chaque système SAP : 445, 5985
- Ports ASCS (numéro d’instance x0) : 32x0, 36x0, 39x0, 81x0, 5x013, 5x014, 5x016
- Ports ASCS (numéro d’instance x1) : 32x1, 33x1, 39x1, 81x1, 5x113, 5x114, 5x116
- Ports ASCS ERS sur Linux (numéro d’instance x2) : 33x2, 5x213, 5x214, 5x216
- Ports SCS ERS sur Linux (numéro d’instance x3) : 33x3, 5x313, 5x314, 5x316

L’équilibrage de charge est configuré pour utiliser les ports de sondage suivants (x étant le numéro du système SAP, par exemple 1, 2, 3...) :
- Port de sondage de l’équilibreur de charge interne ASCS/SCS : 620x0
- Port de sondage de l’équilibreur de charge interne ERS (Linux uniquement) : 621x2

#### <a name="database-template"></a><a name="database-template"></a> Modèle de base de données

Le modèle de base de données déploie une ou deux machines virtuelles que vous pouvez utiliser pour installer le système de gestion de base de données relationnelle (SGBDR) pour un système SAP. Par exemple, si vous déployez un modèle ASCS/SCS pour cinq systèmes SAP, vous devez déployer ce modèle cinq fois.

Pour définir le modèle multi-SID de bases de données, dans le [modèle multi-SID de bases de données][sap-templates-3-tier-multisid-db-marketplace-image] ou le [modèle multi-SID de bases de données avec Disques managés][sap-templates-3-tier-multisid-db-marketplace-image-md], entrez des valeurs pour les paramètres suivants :

- **ID du système SAP**. Entrez l’ID du système SAP que vous souhaitez installer. Cet ID sera utilisé comme préfixe pour les ressources déployées.
- **Type de système d’exploitation**. Sélectionnez le système d’exploitation des machines virtuelles.
- **Type de base de données**. Sélectionnez le type de base de données que vous souhaitez installer sur le cluster. Sélectionnez **SQL** si vous souhaitez installer Microsoft SQL Server. Sélectionnez **HANA** si vous prévoyez d’installer SAP HANA sur les machines virtuelles. Veillez à sélectionner le type de système d’exploitation correct : **Windows** pour SQL et une distribution Linux pour HANA. Azure Load Balancer connecté aux machines virtuelles sera configuré pour prendre en charge le type de base de données sélectionné :
  * **SQL**. L’équilibrage de charge utilise le port 1433. Veillez à utiliser ce port pour la configuration de SQL Server Always On.
  * **HANA**. L’équilibrage de charge utilisera les ports 35015 et 35017. Veillez à installer SAP HANA avec l’instance numéro **50**.
  L’équilibrage de charge utilise le port de sondage 62550.
- **Taille du système SAP**. Définissez le nombre de SAP fournis par le nouveau système. Si vous ne savez pas combien de SAP sont requis par le système, demandez à votre partenaire technologique SAP ou un intégrateur système.
- **Disponibilité du système**. Sélectionnez la haute disponibilité **(HA)** .
- **Nom d’utilisateur et mot de passe d’administrateur**. Créez un nouvel utilisateur pouvant être utilisé pour ouvrir une session sur la machine.
- **ID du sous-réseau**. Entrez l’ID du sous-réseau que vous avez utilisé lors du déploiement du modèle ASCS/SCS ou l’ID du sous-réseau créé dans le cadre du déploiement du modèle ASCS/SCS.

#### <a name="application-servers-template"></a><a name="application-servers-template"></a> Modèle de serveurs d’applications

Le modèle de serveurs d’applications déploie plusieurs machines virtuelles utilisables comme des instances de serveurs d’applications SAP pour un système SAP. Par exemple, si vous déployez un modèle ASCS/SCS pour cinq systèmes SAP, vous devez déployer ce modèle cinq fois.

Pour définir le modèle multi-SID des serveurs d’application, dans le [modèle multi-SID des serveurs d’application][sap-templates-3-tier-multisid-apps-marketplace-image] ou le [modèle multi-SID des serveurs d’application avec Disques managés][sap-templates-3-tier-multisid-apps-marketplace-image-md], entrez les valeurs des paramètres suivants :

  -  **ID du système SAP**. Entrez l’ID du système SAP que vous souhaitez installer. Cet ID sera utilisé comme préfixe pour les ressources déployées.
  -  **Type de système d’exploitation**. Sélectionnez le système d’exploitation des machines virtuelles.
  -  **Taille du système SAP**. Le nombre de SAP fournis par le nouveau système. Si vous ne savez pas combien de SAP sont requis par le système, demandez à votre partenaire technologique SAP ou un intégrateur système.
  -  **Disponibilité du système**. Sélectionnez la haute disponibilité **(HA)** .
  -  **Nom d’utilisateur et mot de passe d’administrateur**. Créez un nouvel utilisateur pouvant être utilisé pour ouvrir une session sur la machine.
  -  **ID du sous-réseau**. Entrez l’ID du sous-réseau que vous avez utilisé lors du déploiement du modèle ASCS/SCS ou l’ID du sous-réseau créé dans le cadre du déploiement du modèle ASCS/SCS.


### <a name="azure-virtual-network"></a><a name="47d5300a-a830-41d4-83dd-1a0d1ffdbe6a"></a> Réseau virtuel Azure
Dans notre exemple, l’espace d’adressage du réseau virtuel Azure est 10.0.0.0/16. Il existe un sous-réseau appelé **Subnet** (Sous-réseau), avec la plage d’adresses 10.0.0.0/24. L’ensemble des machines virtuelles et des équilibrages de charge internes sont déployés au sein de ce réseau virtuel.

> [!IMPORTANT]
> N’apportez aucune modification aux paramètres réseau dans le système d’exploitation invité (adresses IP, serveurs DNS, sous-réseau, etc.). Configurez tous vos paramètres réseau dans Azure. Ils sont propagés par le service DHCP.
>
>

### <a name="dns-ip-addresses"></a><a name="b22d7b3b-4343-40ff-a319-097e13f62f9e"></a>Adresses IP DNS

Pour définir les adresses IP DNS requises, procédez comme suit.

1. Dans le panneau **Serveurs DNS** du portail Azure, assurez-vous que l’option **Serveurs DNS** de votre réseau virtuel est définie sur **DNS personnalisé**.
2. Sélectionnez les paramètres en fonction de votre type de réseau. Pour plus d’informations, consultez les ressources suivantes :
   * Ajoutez les adresses IP des serveurs DNS locaux.  
   Vous pouvez étendre les serveurs DNS locaux aux machines virtuelles exécutées dans Azure. Dans ce scénario, vous pouvez ajouter l’adresse IP des machines virtuelles Azure sur lesquelles vous exécutez le service DNS.
   * Pour les déploiements isolés de machines virtuelles dans Azure : Déployez une machine virtuelle supplémentaire dans l’instance de réseau virtuel jouant le rôle de serveur DNS. Ajoutez l’adresse IP des machines virtuelles Azure que vous avez configurées pour exécuter le service DNS.

   ![Figure 12 : Configurer des serveurs DNS pour le réseau virtuel Azure][sap-ha-guide-figure-3001]

   _**Figure 12 :** Configurer des serveurs DNS pour le réseau virtuel Azure_

   > [!NOTE]
   > Si vous modifiez l’adresse IP des serveurs DNS, vous devez redémarrer les machines virtuelles Azure afin d’appliquer la modification et de propager les nouveaux serveurs DNS.
   >
   >

Dans notre exemple, le service DNS est installé et configuré sur les machines virtuelles Windows suivantes :

| Rôle de la machine virtuelle | Nom d’hôte de la machine virtuelle | Nom de la carte réseau | Adresse IP statique |
| --- | --- | --- | --- |
| Premier serveur DNS |domcontr-0 |pr1-nic-domcontr-0 |10.0.0.10 |
| Deuxième serveur DNS |domcontr-1 |pr1-nic-domcontr-1 |10.0.0.11 |

### <a name="host-names-and-static-ip-addresses-for-the-sap-ascsscs-clustered-instance-and-dbms-clustered-instance"></a><a name="9fbd43c0-5850-4965-9726-2a921d85d73f"></a> Noms d’hôtes et adresses IP statiques de l’instance en cluster SAP ASCS/SCS et l’instance en cluster SGBD

Pour un déploiement local, vous avez besoin des noms d’hôtes et adresses IP réservés suivants :

| Rôle du nom d’hôte virtuel | Nom d’hôte virtuel | Adresse IP statique virtuelle |
| --- | --- | --- |
| Nom d’hôte virtuel du premier cluster SAP ASCS/SCS (pour la gestion du cluster) |pr1-ascs-vir |10.0.0.42 |
| Nom d’hôte virtuel de l’instance SAP ASCS/SCS |pr1-ascs-sap |10.0.0.43 |
| Nom d’hôte virtuel du deuxième cluster SAP SGBD (gestion du cluster) |pr1-dbms-vir |10.0.0.32 |

Lorsque vous créez le cluster, créez les noms d’hôtes virtuels **pr1-ASC-vir** et **pr1-SGBD-vir**, ainsi que les adresses IP associées qui gèrent le cluster lui-même. Pour plus d’informations sur la procédure à suivre, consultez [Collecter les nœuds de cluster dans une configuration de cluster][sap-ha-guide-8.12.1].

Vous pouvez créer manuellement les deux autres noms d’hôtes virtuels, **pr1 ASC sap** et **pr1 SGBD sap**, ainsi que les adresses IP associées, sur le serveur DNS. L’instance SAP ASCS/SCS en cluster et l’instance SGBD en cluster utilisent ces ressources. Pour plus d’informations sur la procédure à suivre, consultez [Créer un nom d’hôte virtuel pour une instance SAP ASCS/SCS en cluster][sap-ha-guide-9.1.1].

### <a name="set-static-ip-addresses-for-the-sap-virtual-machines"></a><a name="84c019fe-8c58-4dac-9e54-173efd4b2c30"></a>Définir des adresses IP statiques pour les machines virtuelles SAP
Après avoir déployé les machines virtuelles à utiliser dans votre cluster, vous devez définir des adresses IP statiques pour l’ensemble des machines virtuelles. Effectuez cette opération dans la configuration du réseau virtuel Azure et non dans le système d’exploitation invité.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** > **Carte réseau** > **Paramètres** > **Adresse IP**.
2. Sur le panneau **Adresses IP**, sous **Affectation**, sélectionnez **Statique**. Dans la zone **Adresse IP**, entrez l’adresse IP à utiliser.

   > [!NOTE]
   > Si vous modifiez l’adresse IP de la carte réseau, vous devez redémarrer les machines virtuelles Azure pour que la modification prenne effet.  
   >
   >

   ![Figure 13 : Définir des adresses IP statiques pour la carte réseau de chaque machine virtuelle][sap-ha-guide-figure-3002]

   _**Figure 13 :** Définir des adresses IP statiques pour la carte réseau de chaque machine virtuelle_

   Répétez cette étape pour toutes les interfaces réseau, c’est-à-dire pour toutes les machines virtuelles, y compris celles que vous souhaitez utiliser pour votre service Active Directory/DNS.

Dans notre exemple, nous avons les machines virtuelles et les adresses IP statiques suivantes :

| Rôle de la machine virtuelle | Nom d’hôte de la machine virtuelle | Nom de la carte réseau | Adresse IP statique |
| --- | --- | --- | --- |
| Première instance du serveur d’applications SAP |pr1-di-0 |pr1-nic-di-0 |10.0.0.50 |
| Deuxième instance du serveur d’applications SAP |pr1-di-1 |pr1-nic-di-1 |10.0.0.51 |
| ... |... |... |... |
| Dernière instance du serveur d’applications SAP serveur d’applications SAP |pr1-di-5 |pr1-nic-di-5 |10.0.0.55 |
| Premier nœud de cluster pour l’instance ASCS/SCS |pr1-ascs-0 |pr1-nic-ascs-0 |10.0.0.40 |
| Deuxième nœud de cluster pour l’instance ASCS/SCS |pr1-ascs-1 |pr1-nic-ascs-1 |10.0.0.41 |
| Premier nœud de cluster pour l’instance SGBD |pr1-db-0 |pr1-nic-db-0 |10.0.0.30 |
| Deuxième nœud de cluster pour l’instance SGBD |pr1-db-1 |pr1-nic-db-1 |10.0.0.31 |

### <a name="set-a-static-ip-address-for-the-azure-internal-load-balancer"></a><a name="7a8f3e9b-0624-4051-9e41-b73fff816a9e"></a> Définir une adresse IP statique pour l’équilibrage de charge interne Azure

Le modèle Azure Resource Manager SAP crée un équilibrage de charge interne Azure utilisé pour le cluster de l’instance SAP ASCS/SCS et le cluster SGBD.

> [!IMPORTANT]
> L’adresse IP du nom d’hôte virtuel de l’instance SAP ASCS/SCS est identique à celle de l’équilibrage de charge interne SAP ASCS/SCS : **pr1-lb-ascs**.
> L’adresse IP du nom virtuel du SGBD est identique à celle de l’équilibrage de charge interne du SGBD SAP : **pr1-lb-dbms**.
>
>

Pour définir une adresse IP statique pour l’équilibrage de charge interne Azure :

1. Le déploiement initial définit l’adresse IP de l’équilibrage de charge interne sur **Dynamique**. Dans le portail Azure, sur le panneau **Adresses IP**, sous **Affectation**, sélectionnez **Statique**.
2. Définissez l’adresse IP de l’équilibrage de charge interne **pr1-lb-ascs** comme l’adresse IP du nom d’hôte virtuel de l’instance SAP ASCS/SCS.
3. Définissez l’adresse IP de l’équilibrage de charge interne **pr1-lb-dbms** sur l’adresse IP du nom d’hôte virtuel de l’instance SGBD.

   ![Figure 14 : Définir une adresse IP statique pour l’équilibreur de charge interne de l’instance SAP ASCS/SCS][sap-ha-guide-figure-3003]

   _**Figure 14 :** Définir une adresse IP statique pour l’équilibreur de charge interne de l’instance SAP ASCS/SCS_

Dans notre exemple, nous avons deux équilibrages de charge internes Azure qui présentent les adresses IP statiques suivantes :

| Rôle de l’équilibrage de charge interne Azure | Nom de l’équilibrage de charge interne Azure | Adresse IP statique |
| --- | --- | --- |
| Équilibrage de charge interne de l’instance SAP ASCS/SCS |pr1-lb-ascs |10.0.0.43 |
| Équilibrage de charge interne du SGBD SAP |pr1-lb-dbms |10.0.0.33 |


### <a name="default-ascsscs-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="f19bd997-154d-4583-a46e-7f5a69d0153c"></a> Règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibrage de charge interne Azure

Le modèle Azure Resource Manager SAP crée les ports dont vous avez besoin :
* Une instance ABAP ASCS, avec le numéro d’instance par défaut **00**
* Une instance Java SCS, avec le numéro d’instance par défaut **01**

Lorsque vous installez votre instance SAP ASCS/SCS, vous devez utiliser le numéro d’instance par défaut **00** pour votre instance ABAP ASCS et le numéro d’instance par défaut **01** pour votre instance Java SCS.

Ensuite, créez les points de terminaison d’équilibrage de charge internes requis pour les ports SAP NetWeaver.

Pour créer les points de terminaison d’équilibrage de charge interne requis, créez tout d’abord ces points pour les ports SAP NetWeaver ABAP ASC :

| Service/nom de la règle d’équilibrage de charge | Numéros de ports par défaut | Ports concrets pour (instance ASCS avec numéro d’instance 00) (ERS avec 10) |
| --- | --- | --- |
| Serveur de file d’attente / *lbrule3200* |32<*NuméroInstance*> |3200 |
| Serveur de messages ABAP / *lbrule3600* |36<*NuméroInstance*> |3600 |
| Message ABAP interne / *lbrule3900* |39<*NuméroInstance*> |3900 |
| Serveur de messages HTTP / *Lbrule8100* |81<*NuméroInstance*> |8100 |
| Service de démarrage SAP ASCS HTTP / *Lbrule50013* |5<*NuméroInstance*>13 |50013 |
| Service de démarrage SAP ASCS HTTPS / *Lbrule50014* |5<*NuméroInstance*>14 |50014 |
| Réplication de la file d’attente / *Lbrule50016* |5<*NuméroInstance*>16 |50016 |
| Service de démarrage SAP ERS HTTP / *Lbrule51013* |5<*NuméroInstance*>13 |51013 |
| Service de démarrage SAP ERS HTTP / *Lbrule51014* |5<*NuméroInstance*>14 |51014 |
| WinRM *Lbrule5985* | |5985 |
| Partage de fichiers *Lbrule445* | |445 |

_**Tableau 1 :** Numéro de port des instances SAP NetWeaver ABAP ASCS_

Ensuite, créez ces points de terminaison d’équilibrage de charge pour les ports SAP NetWeaver Java SCS :

| Service/nom de la règle d’équilibrage de charge | Numéros de ports par défaut | Ports concrets pour (instance SCS avec numéro d’instance 01) (ERS avec 11) |
| --- | --- | --- |
| Serveur de file d’attente / *lbrule3201* |32<*NuméroInstance*> |3201 |
| Serveur de passerelle / *lbrule3301* |33<*NuméroInstance*> |3301 |
| Serveur de messages Java / *lbrule3900* |39<*NuméroInstance*> |3901 |
| Serveur de messages HTTP / *Lbrule8101* |81<*NuméroInstance*> |8101 |
| Service de démarrage SAP SCS HTTP / *Lbrule50113* |5<*NuméroInstance*>13 |50113 |
| Service de démarrage SAP SCS HTTPS / *Lbrule50114* |5<*NuméroInstance*>14 |50114 |
| Réplication de la file d’attente / *Lbrule50116* |5<*NuméroInstance*>16 |50116 |
| Service de démarrage SAP ERS HTTP / *Lbrule51113* |5<*NuméroInstance*>13 |51113 |
| Service de démarrage SAP ERS HTTP / *Lbrule51114* |5<*NuméroInstance*>14 |51114 |
| WinRM *Lbrule5985* | |5985 |
| Partage de fichiers *Lbrule445* | |445 |

_**Table 2 :** Numéro de port des instances SAP NetWeaver Java SCS_

![Figure 15 : Règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibreur de charge interne Azure][sap-ha-guide-figure-3004]

_**Figure 15 :** Règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibreur de charge interne Azure_

Définissez l’adresse IP de l’équilibrage de charge **pr1-lb-dbms** sur l’adresse IP du nom d’hôte virtuel de l’instance SGBD.

### <a name="change-the-ascsscs-default-load-balancing-rules-for-the-azure-internal-load-balancer"></a><a name="fe0bd8b5-2b43-45e3-8295-80bee5415716"></a> Modifier les règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibrage de charge interne Azure

Si vous souhaitez utiliser d’autres numéros pour les instances SAP ASCS ou SCS, vous devez modifier les noms et valeurs par défaut de ces ports.

1. Dans le portail Azure, sélectionnez **<*SID*-lb-ascs load balancer** > **Règles d’équilibrage de charge**.
2. Pour toutes les règles d’équilibrage de charge qui appartiennent à l’instance SAP ASCS ou SCS, modifiez les valeurs suivantes :

   * Nom
   * Port
   * Port principal

   Par exemple, si vous souhaitez remplacer le numéro d’instance ASC par défaut 00 par 31, vous devez modifier tous les ports répertoriés dans le tableau 1.

   Voici un exemple de mise à jour pour le port *lbrule3200*.

   ![Figure 16 : Changer les règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibreur de charge interne Azure][sap-ha-guide-figure-3005]

   _**Figure 16 :** Changer les règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibreur de charge interne Azure_

### <a name="add-windows-virtual-machines-to-the-domain"></a><a name="e69e9a34-4601-47a3-a41c-d2e11c626c0c"></a> Ajouter des machines virtuelles Windows au domaine

Après avoir affecté une adresse IP statique aux machines virtuelles, ajoutez les machines virtuelles au domaine.

![Figure 17 : Ajouter une machine virtuelle à un domaine][sap-ha-guide-figure-3006]

_**Figure 17 :** Ajouter une machine virtuelle à un domaine_

### <a name="add-registry-entries-on-both-cluster-nodes-of-the-sap-ascsscs-instance"></a><a name="661035b2-4d0f-4d31-86f8-dc0a50d78158"></a> Ajouter des entrées de Registre aux deux nœuds de cluster de l’instance SAP ASCS/SCS

Azure Load Balancer offre un équilibrage de charge interne qui ferme les connexions lorsqu’elles sont inactives pendant un laps de temps défini (délai d’inactivité). Les processus de travail SAP dans les instances de dialogue ouvrent des connexions au processus de mise en file d’attente SAP dès que la première requête de mise en file d’attente/suppression de la file d’attente doit être envoyée. Ces connexions restent généralement établies jusqu’à ce que le processus de travail ou de fil d’attente redémarre. Toutefois, si la connexion est inactive pendant un laps de temps donné, l’équilibrage de charge interne Azure ferme les connexions. Ce n’est pas un problème, car le processus de travail SAP rétablit la connexion au processus de file d’attente si elle n’existe plus. Ces activités sont documentées dans les traces de développement des processus SAP, mais elles créent une quantité importante de contenu supplémentaire dans les traces. Il est conseillé de modifier les paramètres TCP/IP `KeepAliveTime` et `KeepAliveInterval` sur les deux nœuds de cluster. Associez ces modifications des paramètres TCP/IP et les paramètres de profil SAP décrits plus loin dans cet article.

Pour ajouter des entrées de registre aux deux nœuds du cluster de l’instance SAP ASCS/SCS, ajoutez tout d’abord ces entrées de registre Windows aux deux nœuds de cluster Windows pour SAP ASCS/SCS :

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Nom de la variable |`KeepAliveTime` |
| Type de variable |REG_DWORD (décimal) |
| Valeur |120 000 |
| Lien vers la documentation |[https://technet.microsoft.com/library/cc957549.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957549(v=technet.10)) |

_**Tableau 3 :** Changer le premier paramètre TCP/IP_

Puis, ajoutez ces entrées de registre Windows aux deux nœuds de cluster Windows pour l’instance SAP ASCS/SCS :

| Path | HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters |
| --- | --- |
| Nom de la variable |`KeepAliveInterval` |
| Type de variable |REG_DWORD (décimal) |
| Valeur |120 000 |
| Lien vers la documentation |[https://technet.microsoft.com/library/cc957548.aspx](/previous-versions/windows/it-pro/windows-2000-server/cc957548(v=technet.10)) |

_**Tableau 4 :** Changer le deuxième paramètre TCP/IP_

**Pour appliquer les modifications, redémarrez les deux nœuds de cluster**.

### <a name="set-up-a-windows-server-failover-clustering-cluster-for-an-sap-ascsscs-instance"></a><a name="0d67f090-7928-43e0-8772-5ccbf8f59aab"></a> Configurer un cluster de clustering de basculement Windows Server pour une instance SAP ASCS/SCS

La configuration d’un cluster de basculement Windows Server pour une instance SAP ASCS/SCS implique les tâches suivantes :

- Collecter les nœuds de cluster dans une configuration de cluster
- Configurer un témoin de partage de fichiers de cluster

#### <a name="collect-the-cluster-nodes-in-a-cluster-configuration"></a><a name="5eecb071-c703-4ccc-ba6d-fe9c6ded9d79"></a> Collecter les nœuds de cluster dans une configuration de cluster

1. Dans l’Assistant Ajout de rôles et de fonctionnalités, ajoutez un clustering de basculement aux deux nœuds de cluster.
2. Configurez le cluster de basculement à l’aide du Gestionnaire du cluster de basculement. Dans le Gestionnaire du cluster de basculement, sélectionnez **Créer un cluster**, puis ajoutez uniquement le nom du premier cluster, nœud A. N’ajoutez pas encore le second nœud ; vous le ferez à une étape ultérieure.

   ![Figure 18 : Ajouter le nom de serveur ou de machine virtuelle du premier nœud de cluster][sap-ha-guide-figure-3007]

   _**Figure 18 :** Ajouter le nom de serveur ou de machine virtuelle du premier nœud de cluster_

3. Entrez le nom de réseau (nom d’hôte virtuel) du cluster.

   ![Figure 19 : Entrer le nom de cluster][sap-ha-guide-figure-3008]

   _**Figure 19 :** Entrer le nom de cluster_

4. Une fois le cluster créé, exécutez un test de validation du cluster.

   ![Figure 20 : Exécuter la vérification de validation de cluster][sap-ha-guide-figure-3009]

   _**Figure 20 :** Exécuter la vérification de validation de cluster_

   Vous pouvez ignorer les avertissements concernant des disques à ce stade de la procédure. Vous ajouterez un témoin de partage de fichiers et les disques partagés SIOS ultérieurement. À ce stade, le fait de ne pas disposer d’un disque quorum n’est pas un problème.

   ![Figure 21 : Aucun disque quorum][sap-ha-guide-figure-3010]

   _**Figure 21 :** Aucun disque quorum_

   ![Figure 22 : La ressource principale du cluster nécessite une nouvelle adresse IP][sap-ha-guide-figure-3011]

   _**Figure 22 :** La ressource principale du cluster nécessite une nouvelle adresse IP_

5. Modifiez l’adresse IP du service principal du cluster. Le cluster ne peut pas démarrer tant que vous n’avez pas modifié l’adresse IP du service principal du cluster car l’adresse IP du serveur pointe vers un des nœuds de machine virtuelle. Effectuez cette opération dans la page **Propriétés** de la ressource IP du service principal du cluster.

   Par exemple, vous devez affecter une adresse IP (dans notre exemple, **10.0.0.42**) au nom d’hôte virtuel du cluster **pr1-ascs-vir**.

   ![Figure 23 : Dans la boîte de dialogue Propriétés, changer l’adresse IP][sap-ha-guide-figure-3012]

   _**Figure 23 :** Dans la boîte de dialogue **Propriétés**, changer l’adresse IP_

   ![Figure 24 : Attribuer l’adresse IP réservée pour le cluster][sap-ha-guide-figure-3013]

   _**Figure 24 :** Attribuer l’adresse IP réservée pour le cluster_

6. Mettez en ligne le nom de l’hôte virtuel du cluster.

   ![Figure 25 : Le service principal du cluster est opérationnel avec une adresse IP correcte][sap-ha-guide-figure-3014]

   _**Figure 25 :** Le service principal du cluster est opérationnel avec une adresse IP correcte_

7. Ajoutez le deuxième nœud de cluster.

   Maintenant que le service principal du cluster est opérationnel, vous pouvez ajouter le deuxième nœud du cluster.

   ![Figure 26 : Ajouter le deuxième nœud de cluster][sap-ha-guide-figure-3015]

   _**Figure 26 :** Ajouter le deuxième nœud de cluster_

8. Entrez un nom pour le second hôte du nœud de cluster.

   ![Figure 27 : Entrer le nom d’hôte du deuxième nœud de cluster][sap-ha-guide-figure-3016]

   _**Figure 27 :** Entrer le nom d’hôte du deuxième nœud de cluster_

   > [!IMPORTANT]
   > Assurez-vous que la case à cocher **Ajouter la totalité du stockage disponible au cluster** n’est **PAS** sélectionnée.  
   >
   >

   ![Figure 28 : Ne pas cocher la case][sap-ha-guide-figure-3017]

   _**Figure 28 :** Ne **pas** cocher la case_

   Vous pouvez ignorer les avertissements concernant le quorum et les disques. Vous définirez le quorum et partagerez le disque ultérieurement, comme indiqué dans [Installer SIOS DataKeeper Cluster Edition pour le disque de partage en cluster SAP ASCS/SCS][sap-ha-guide-8.12.3].

   ![Figure 29 : Ignorer les avertissements concernant le disque quorum][sap-ha-guide-figure-3018]

   _**Figure 29 :** Ignorer les avertissements concernant le disque quorum_


#### <a name="configure-a-cluster-file-share-witness"></a><a name="e49a4529-50c9-4dcf-bde7-15a0c21d21ca"></a> Configurer un témoin de partage de fichiers de cluster

La configuration d’un témoin de partage de fichiers de cluster implique les tâches suivantes :

- Création d'un partage de fichiers
- Définition du quorum de témoin de partage de fichiers dans le Gestionnaire du cluster de basculement

##### <a name="create-a-file-share"></a><a name="06260b30-d697-4c4d-b1c9-d22c0bd64855"></a> Créer un partage de fichiers

1. Sélectionnez un témoin de partage de fichiers plutôt qu’un disque quorum. SIOS DataKeeper prend en charge cette option.

   Dans les exemples de cet article, le témoin de partage de fichiers se trouve sur le serveur Active Directory/DNS exécuté dans Azure. Le témoin de partage de fichiers porte le nom **domcontr-0**. Comme vous avez configuré une connexion VPN à Azure (via un VPN de site à site ou Azure ExpressRoute), votre service Active Directory/DNS est local et n’est pas adapté à l’exécution d’un témoin de partage de fichiers.

   > [!NOTE]
   > Si votre service Active Directory/DNS s’exécute uniquement en local, ne configurez pas votre témoin de partage de fichiers sur le système d’exploitation Windows du service Active Directory/DNS exécuté localement. La latence du réseau entre les nœuds de cluster exécutés dans Azure et le service Active Directory/DNS local peut être trop importante et entraîner des problèmes de connectivité. Veillez à configurer le témoin de partage de fichiers sur une machine virtuelle Azure proche du nœud de cluster.  
   >
   >

   Au moins 1 024 Mo d’espace libre sont nécessaires pour le disque quorum. Le disque quorum a besoin d’au moins 2 048 Mo d’espace libre.

2. Ajoutez l’objet nom de cluster.

   ![Figure 30 : Attribuer les autorisations sur le partage pour l’objet nom de cluster][sap-ha-guide-figure-3019]

   _**Figure 30 :** Attribuer les autorisations sur le partage pour l’objet nom de cluster_

   Assurez-vous que les autorisations incluent la possibilité de modifier des données du partage pour l’objet nom de cluster (dans notre exemple, **pr1-ascs-vir$** ).

3. Pour ajouter l’objet nom de cluster à la liste, sélectionnez **Ajouter**. Modifiez le filtre de manière à rechercher les objets ordinateurs, en plus de ceux indiqués dans la figure 31.

   ![Figure 31 : Changer les types d’objet pour inclure les ordinateurs][sap-ha-guide-figure-3020]

   _**Figure 31 :** Changer les types d’objet pour inclure les ordinateurs_

   ![Figure 32 : Cocher la case Ordinateurs][sap-ha-guide-figure-3021]

   _**Figure 32 :** Cocher la case **Ordinateurs**_

4. Entrez l’objet nom de cluster, comme indiqué dans la figure 31. L’enregistrement étant déjà créé, vous pouvez modifier les autorisations, comme indiqué dans la figure 30.

5. Sélectionnez l’onglet **Sécurité** du partage, puis définissez des autorisations plus détaillées pour l’objet nom de cluster.

   ![Figure 33 : Définir les attributs de sécurité de l’objet nom de cluster sur le quorum de partage de fichiers][sap-ha-guide-figure-3022]

   _**Figure 33 :** Définir les attributs de sécurité de l’objet nom de cluster sur le quorum de partage de fichiers_

##### <a name="set-the-file-share-witness-quorum-in-failover-cluster-manager"></a><a name="4c08c387-78a0-46b1-9d27-b497b08cac3d"></a> Définir le quorum de témoin de partage de fichiers dans le Gestionnaire du cluster de basculement

1. Ouvrez l’Assistant Configuration de quorum du cluster.

   ![Figure 34 : Démarrer l’Assistant Configuration du paramètre de quorum du cluster][sap-ha-guide-figure-3023]

   _**Figure 34 :** Démarrer l’Assistant Configuration du paramètre de quorum du cluster_

2. Sur la page **Sélectionner la configuration du quorum** , sélectionnez **Sélectionner le témoin de quorum**.

   ![Figure 35 : Configurations de quorum sélectionnables][sap-ha-guide-figure-3024]

   _**Figure 35 :** Configurations de quorum sélectionnables_

3. Sur la page **Sélectionner le témoin de quorum**, sélectionnez **Configurer un témoin de partage de fichiers**.

   ![Figure 36 : Sélectionner le témoin de partage de fichiers][sap-ha-guide-figure-3025]

   _**Figure 36 :** Sélectionner le témoin de partage de fichiers_

4. Entrez le chemin d’accès UNC du partage de fichiers (dans notre exemple, \\domcontr-0\FSW). Pour afficher une liste des modifications possibles, sélectionnez **Suivant**.

   ![Figure 37 : Définir l’emplacement du partage de fichiers témoin][sap-ha-guide-figure-3026]

   _**Figure 37 :** Définir l’emplacement du partage de fichiers témoin_

5. Apportez les modifications souhaitées, puis sélectionnez **Suivant**. Vous devez reconfigurer correctement le cluster, comme indiqué dans la figure 38.  

   ![Figure 38 : Confirmation de la reconfiguration du cluster][sap-ha-guide-figure-3027]

   _**Figure 38 :** Confirmation de la reconfiguration du cluster_

Après avoir installé le cluster de basculement Windows avec succès, des modifications doivent être apportées à certains seuils pour adapter la détection de basculement aux conditions dans Azure. Les paramètres à modifier sont décrits dans ce blog : https://blogs.msdn.microsoft.com/clustering/2012/11/21/tuning-failover-cluster-network-thresholds/ . En supposant que les deux machines virtuelles qui génèrent la configuration du cluster Windows pour ASCS/SCS se trouvent sur le même sous-réseau, les paramètres suivants doivent être modifiés avec ces valeurs :
- SameSubNetDelay = 2
- SameSubNetThreshold = 15

Ces paramètres ont été testés avec nos clients et ont offert un bon compromis pour être suffisamment résilients d’un côté. D’autre part, ces paramètres proposent un basculement suffisamment rapide dans les conditions d’erreur réelles en cas de problème sur le nœud/la machine virtuelle ou les logiciels SAP. 

### <a name="install-sios-datakeeper-cluster-edition-for-the-sap-ascsscs-cluster-share-disk"></a><a name="5c8e5482-841e-45e1-a89d-a05c0907c868"></a> Installer SIOS DataKeeper Cluster Edition pour le disque de partage en cluster SAP ASCS/SCS

Vous disposez maintenant d’une configuration fonctionnelle de clustering de basculement Windows Server dans Azure. Cependant, pour installer une instance SAP ASCS/SCS, vous avez besoin d’une ressource de disque partagé. Il n’est pas possible de créer les ressources de disque partagé requises dans Azure. SIOS DataKeeper Cluster Edition est une solution tierce que vous pouvez utiliser pour créer des ressources de disque partagé.

L’installation de SIOS DataKeeper Cluster Edition pour le disque de partage en cluster SAP ASCS/SCS implique les tâches suivantes :

- Ajout de .NET Framework 3.5
- Installation de SIOS DataKeeper
- Configuration de SIOS DataKeeper

#### <a name="add-the-net-framework-35"></a><a name="1c2788c3-3648-4e82-9e0d-e058e475e2a3"></a> Ajouter .NET Framework 3.5
Microsoft .NET Framework 3.5 n’est pas automatiquement activé ou installé dans Windows Server 2012 R2. Comme SIOS DataKeeper requiert .NET Framework sur tous les nœuds sur lequel vous installez DataKeeper, vous devez installer .NET Framework 3.5 sur le système d’exploitation invité de toutes les machines virtuelles du cluster.

Pour ajouter .NET Framework 3.5, deux options s’offrent à vous :

- Utilisez l’Assistant Ajout de rôles et de fonctionnalités, comme indiqué dans la figure 39.

  ![Figure 39 : Installer .NET Framework 3.5 à l’aide de l’Assistant Ajout de rôles et de fonctionnalités][sap-ha-guide-figure-3028]

  _**Figure 39 :** Installer .NET Framework 3.5 à l’aide de l’Assistant Ajout de rôles et de fonctionnalités_

  ![Figure 40 : Barre de progression de l’installation de .NET Framework 3.5 affichée dans l’Assistant Ajout de rôles et de fonctionnalités][sap-ha-guide-figure-3029]

  _**Figure 40 :** Barre de progression de l’installation de .NET Framework 3.5 affichée dans l’Assistant Ajout de rôles et de fonctionnalités_

- Utilisez l’outil de ligne de commande dism.exe. Pour ce type d’installation, vous devez accéder au répertoire SxS du média d’installation Windows. Dans une invite de commandes avec élévation de privilèges, tapez :

  ```
  Dism /online /enable-feature /featurename:NetFx3 /All /Source:installation_media_drive:\sources\sxs /LimitAccess
  ```

#### <a name="install-sios-datakeeper"></a><a name="dd41d5a2-8083-415b-9878-839652812102"></a> Installer SIOS DataKeeper

Installez SIOS DataKeeper Cluster Edition sur chaque nœud du cluster. Pour créer un stockage partagé virtuel avec SIOS DataKeeper, créez un miroir synchronisé, puis simulez le stockage partagé en cluster.

Avant d’installer le logiciel SIOS, créez l’utilisateur de domaine **DataKeeperSvc**.

> [!NOTE]
> Ajoutez l’utilisateur **DataKeeperSvc** au groupe **Administrateur local** sur les deux nœuds de cluster.
>
>

Pour installer SIOS DataKeeper :

1. Installez le logiciel SIOS sur les deux nœuds de cluster.

   ![Programme d’installation de SIOS][sap-ha-guide-figure-3030]

   ![Figure 41 : Première page de l’installation de SIOS DataKeeper][sap-ha-guide-figure-3031]

   _**Figure 41 :** Première page de l’installation de SIOS DataKeeper_

2. Dans la boîte de dialogue affichée dans la figure 42, sélectionnez **Oui**.

   ![Figure 42 : DataKeeper vous indique qu’un service va être désactivé][sap-ha-guide-figure-3032]

   _**Figure 42 :** DataKeeper vous indique qu’un service va être désactivé_

3. Dans la boîte de dialogue affichée dans la figure 43, nous vous recommandons de sélectionner **Compte de domaine ou de serveur**.

   ![Figure 43 : Sélection de l’utilisateur de SIOS DataKeeper][sap-ha-guide-figure-3033]

   _**Figure 43 :** Sélection de l’utilisateur de SIOS DataKeeper_

4. Entrez le nom et le mot de passe de l’utilisateur de compte de domaine que vous avez créés pour SIOS DataKeeper.

   ![Figure 44 : Entrer le nom et le mot de passe de l’utilisateur du domaine pour l’installation de SIOS DataKeeper][sap-ha-guide-figure-3034]

   _**Figure 44 :** Entrer le nom et le mot de passe de l’utilisateur du domaine pour l’installation de SIOS DataKeeper_

5. Installez la clé de licence de votre instance SIOS DataKeeper, comme indiqué dans la figure 45.

   ![Figure 45 : Entrer votre clé de licence SIOS DataKeeper][sap-ha-guide-figure-3035]

   _**Figure 45 :** Entrer votre clé de licence SIOS DataKeeper_

6. À l’invite, redémarrez la machine virtuelle.

#### <a name="set-up-sios-datakeeper"></a><a name="d9c1fc8e-8710-4dff-bec2-1f535db7b006"></a> Configurer SIOS DataKeeper

Après avoir installé SIOS DataKeeper sur les deux nœuds, vous devez commencer la configuration. L’objectif de la configuration est de mettre en place une réplication synchrone des données entre les disques supplémentaires attachés à chacune des machines virtuelles.

1. Démarrez l’outil de configuration et de gestion de DataKeeper, puis sélectionnez **Connect Server** (Connexion au serveur) (Cette option est entourée en rouge dans la figure 46.)

   ![Figure 46 : Outil de configuration et de gestion de SIOS DataKeeper][sap-ha-guide-figure-3036]

   _**Figure 46 :** Outil de configuration et de gestion de SIOS DataKeeper_

2. Insérez le nom ou l’adresse TCP/IP du premier nœud auquel l’outil de gestion et de configuration doit se connecter, puis du second nœud.

   ![Figure 47 : Insérer le nom ou l’adresse TCP/IP du premier nœud auquel l’outil de gestion et de configuration doit se connecter, puis du deuxième nœud][sap-ha-guide-figure-3037]

   _**Figure 47 :** Insérer le nom ou l’adresse TCP/IP du premier nœud auquel l’outil de gestion et de configuration doit se connecter, puis du deuxième nœud_

3. Créez le travail de réplication entre les deux nœuds.

   ![Figure 48 : Créer un travail de réplication][sap-ha-guide-figure-3038]

   _**Figure 48 :** Créer un travail de réplication_

   Un assistant vous guide à travers le processus de création d’un travail de réplication.
4. Définissez le nom, l’adresse TCP/IP et le volume de disque du nœud source.

   ![Figure 49 : Définir le nom du travail de réplication][sap-ha-guide-figure-3039]

   _**Figure 49 :** Définir le nom du travail de réplication_

   ![Figure 50 : Définir les données de base du nœud qui doit être le nœud source actuel][sap-ha-guide-figure-3040]

   _**Figure 50 :** Définir les données de base du nœud qui doit être le nœud source actuel_

5. Définissez le nom, l’adresse TCP/IP et le volume de disque du nœud cible.

   ![Figure 51 : Définir les données de base du nœud qui doit être le nœud cible actuel][sap-ha-guide-figure-3041]

   _**Figure 51 :** Définir les données de base du nœud qui doit être le nœud cible actuel_

6. Définissez les algorithmes de compression. Dans notre exemple, nous vous recommandons de compresser le flux de réplication. Dans les situations de resynchronisation notamment, la compression du flux de réplication accélère considérablement la resynchronisation. Notez que la compression utilise les ressources de processeur et de RAM d’une machine virtuelle. Plus le taux de compression augmente, plus le volume de ressources de processeur utilisées est important. Vous pouvez également ajuster ce paramètre ultérieurement.

7. Vous devez également vérifier si la réplication se fait de façon asynchrone ou synchrone. *Lorsque vous protégez des configurations SAP ASCS/SCS, vous devez utiliser la réplication synchrone*.  

   ![Figure 52 : Définir les détails de la réplication][sap-ha-guide-figure-3042]

   _**Figure 52 :** Définir les détails de la réplication_

8. Définissez si le volume répliqué par le travail de réplication doit être représenté auprès d’une configuration de cluster de clustering de basculement Windows Server en tant que disque partagé. Pour la configuration SAP ASCS/SCS, sélectionnez **Oui** pour que le cluster Windows voie le volume répliqué en tant que disque partagé utilisable comme volume de cluster.

   ![Figure 53 : Sélectionner Oui pour définir le volume répliqué comme volume de cluster][sap-ha-guide-figure-3043]

   _**Figure 53 :** Sélectionner **Oui** pour définir le volume répliqué comme volume de cluster_

   Une fois le volume créé, l’outil de configuration et de gestion de DataKeeper indique que le travail de réplication est actif.

   ![Figure 54 : La mise en miroir synchrone de DataKeeper pour le disque de partage SAP ASCS/SCS est active][sap-ha-guide-figure-3044]

   _**Figure 54 :** La mise en miroir synchrone de DataKeeper pour le disque de partage SAP ASCS/SCS est active_

   Le Gestionnaire du cluster de basculement affiche maintenant le disque en tant que disque DataKeeper, comme indiqué dans la figure 55.

   ![Figure 55 : Le Gestionnaire du cluster de basculement montre le disque répliqué par DataKeeper][sap-ha-guide-figure-3045]

   _**Figure 55 :** Le Gestionnaire du cluster de basculement montre le disque répliqué par DataKeeper_

## <a name="install-the-sap-netweaver-system"></a><a name="a06f0b49-8a7a-42bf-8b0d-c12026c5746b"></a> Installer le système SAP NetWeaver

Nous ne décrirons pas l’installation du SGBD, car celle-ci varie en fonction du SGBD utilisé. Toutefois, nous partons du principe que la haute disponibilité du SGBD est assurée par les fonctionnalités prises en charge par les différents fournisseurs de SGBD pour Azure, par exemple Always On ou la mise en miroir de base de données pour SQL Server, et Oracle Data Guard pour bases de données Oracle. Dans le scénario utilisé dans cet article, nous n’avons pas ajouté de protection supplémentaire au SGBD.

Il n’existe pas de considérations particulières lorsque différents services de SGBD interagissent avec ce type de configuration SAP ASCS/SCS en cluster dans Azure.

> [!NOTE]
> Les procédures d’installation de systèmes SAP NetWeaver ABAP, Java et ABAP+Java sont presque identiques. La différence la plus notable est qu’un système SAP ABAP comprend une instance ASCS, le système SAP Java une instance SCS, et le système SAP ABAP+Java une instance ASCS et une instance SCS exécutées dans le même groupe de cluster de basculement Microsoft. Toute différence d’installation pour chaque pile d’installation de SAP NetWeaver est mentionnée explicitement. Le reste de la procédure est identique.  
>
>

### <a name="install-sap-with-a-high-availability-ascsscs-instance"></a><a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a> Installer SAP avec une instance ASCS/SCS à haute disponibilité

> [!IMPORTANT]
> Veillez à ne pas placer votre fichier d’échange sur des volumes DataKeeper mis en miroir. DataKeeper ne prend pas en charge les volumes mis en miroir. Vous pouvez laisser votre fichier d’échange sur le lecteur temporaire D d’une machine virtuelle (valeur par défaut). S’il ne s’y trouve pas déjà, déplacez le fichier de la page Windows sur le lecteur D: de votre machine virtuelle Azure.
>
>

L’installation de SAP avec une instance ASCS/SCS à haute disponibilité implique les tâches suivantes :

- Création d’un nom d’hôte virtuel pour l’instance SAP ASCS/SCS en cluster
- Installation du premier nœud de cluster SAP
- Modification du profil SAP de l’instance ASCS/SCS
- Ajout d'un port de sondage
- Ouverture du port de sondage du pare-feu Windows

#### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a><a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a> Créer un nom d’hôte virtuel pour l’instance SAP ASCS/SCS en cluster

1. Dans le Gestionnaire DNS Windows, créez une entrée DNS pour le nom d’hôte virtuel de l’instance ASC/SCS.

   > [!IMPORTANT]
   > L’adresse IP que vous affectez au nom d’hôte virtuel de l’instance ASC/SCS doit être identique à celle que vous avez affectée à l’équilibrage de charge Azure Load Balancer ( **<*SID*>-lb-ascs**).  
   >
   >

   L’adresse IP du nom d’hôte SAP ASCS/SCS virtuel (**pr1-ascs-sap**) est identique à celle de l’équilibrage de charge Azure Load Balancer (**pr1-lb-ascs**).

   ![Figure 56 : Définir l’entrée DNS pour le nom virtuel et l’adresse TCP/IP du cluster SAP ASCS/SCS][sap-ha-guide-figure-3046]

   _**Figure 56 :** Définir l’entrée DNS pour le nom virtuel et l’adresse TCP/IP du cluster SAP ASCS/SCS_

2. Pour définir l’adresse IP affectée au nom d’hôte virtuel, sélectionnez **Gestionnaire DNS** > **Domaine**.

   ![Figure 57 : Nouveau nom virtuel et nouvelle adresse TCP/IP de la configuration du cluster SAP ASCS/SCS][sap-ha-guide-figure-3047]

   _**Figure 57 :** Nouveau nom virtuel et nouvelle adresse TCP/IP de la configuration du cluster SAP ASCS/SCS_

#### <a name="install-the-sap-first-cluster-node"></a><a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> Installer le premier nœud de cluster SAP

1. Exécutez l’option du premier nœud de cluster sur le nœud A du cluster, par exemple l’hôte **pr1-ascs-0**.
2. Pour conserver les ports par défaut pour l’équilibrage de charge interne Azure, sélectionnez :

   * **Système ABAP** : Numéro d’instance **ASCS** **00**
   * **Système Java** : Numéro d’instance **SCS** **01**
   * **Système ABAP+Java** : Numéro d’instance **ASCS** **00** et numéro d’instance **SCS** **01**

   Pour utiliser des numéros autres que 00 et 01 respectivement pour les instances ASCS ABAP et SCS Java, vous devez d’abord modifier les règles d’équilibrage de charge interne Azure par défaut, comme indiqué dans [Modifier les règles d’équilibrage de charge ASCS/SCS par défaut pour l’équilibrage de charge interne Azure][sap-ha-guide-8.9].

Les quelques étapes suivantes ne sont pas décrites dans la documentation d’installation de SAP standard.

> [!NOTE]
> La documentation d’installation de SAP explique comment installer le premier nœud de cluster ASC/SCS.
>
>

#### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a><a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> Modifier le profil SAP de l’instance ASCS/SCS

Vous devez ajouter un nouveau paramètre de profil. Ce paramètre de profil empêche les connexions entre les processus de travail SAP et le serveur de mise en file d’attente de se fermer lorsqu’elles sont inactives pendant trop longtemps. Nous avons mentionné ce scénario problématique dans la section [Ajouter des entrées de registre aux deux nœuds de cluster de l’instance SAP ASCS/SCS][sap-ha-guide-8.11]. Dans cette section, nous avons également apporté deux modifications à certains paramètres de connexion TCP/IP de base. Dans un deuxième temps, vous devez configurer le serveur de mise en file d’attente pour qu’il envoie un signal `keep_alive` afin que les connexions n’atteignent pas le seuil d’inactivité de l’équilibrage de charge interne Azure.

Pour modifier le profil SAP de l’instance ASCS/SCS :

1. Ajoutez ce paramètre de profil au profil de l’instance SAP ASCS/SCS :

   ```
   enque/encni/set_so_keepalive = true
   ```
   Dans notre exemple, le chemin est :

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_ASCS00_pr1-ascs-sap`

   Par exemple, au profil de l’instance SAP SCS et au chemin correspondant :

   `<ShareDisk>:\usr\sap\PR1\SYS\profile\PR1_SCS01_pr1-ascs-sap`

2. Pour appliquer les modifications, redémarrez l’instance SAP ASCS/SCS.

#### <a name="add-a-probe-port"></a><a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> Ajouter un port de sondage

Utilisez la fonctionnalité de sondage de l’équilibrage de charge interne pour que la configuration de cluster entière fonctionne avec Azure Load Balancer. Généralement, l’équilibrage de charge interne Azure répartit équitablement la charge de travail entrante entre les machines virtuelles participantes. Toutefois, cela ne fonctionne pas dans certaines configurations de cluster, car une seule instance est active. L’autre instance est passive et ne peut accepter aucune partie de la charge de travail. La fonctionnalité de sondage est utile lorsque l’équilibrage de charge interne Azure affecte du travail uniquement à une instance active. Avec la fonctionnalité de sondage, l’équilibrage de charge interne peut détecter les instances actives, puis cibler uniquement ces instances avec la charge de travail.

Pour ajouter un port de sondage :

1. Vérifiez le paramètre **ProbePort** actuel en exécutant la commande PowerShell suivante. Exécutez-la depuis l’une des machines virtuelles de la configuration de cluster.

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter
   ```

2. Définissez un port de sondage. Le numéro de port de sondage par défaut est **0**. Dans notre exemple, nous utilisons le port de sondage **62000**.

   ![Figure 58 : Le port de sondage de la configuration de cluster est défini sur 0 par défaut][sap-ha-guide-figure-3048]

   _**Figure 58 :** Le port de sondage de la configuration de cluster est 0 par défaut_

   Le numéro de port est défini dans les modèles Azure Resource Manager SAP. Vous pouvez affecter le numéro de port dans PowerShell.

   Pour définir une nouvelle valeur ProbePort pour la ressource de cluster **SAP <*SID*> IP**, exécutez le script PowerShell suivant. Mettez à jour les variables PowerShell de votre environnement. Après l’exécution du script, vous serez invité à redémarrer le groupe de clusters SAP pour activer les modifications.

   ```powershell
   $SAPSID = "PR1"      # SAP <SID>
   $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

   Clear-Host
   $SAPClusterRoleName = "SAP $SAPSID"
   $SAPIPresourceName = "SAP $SAPSID IP"
   $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
   $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
   $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
   $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
   $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
   $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
   $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value

   $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }

   Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter

   Write-Host
   Write-Host "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." -ForegroundColor Cyan
   Write-Host
   Write-Host "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." -ForegroundColor Cyan
   Write-Host

   $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}

   Write-Host

   $ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"

   if($ActivateChanges -eq "yes"){
   Write-Host
   Write-Host "Activating changes..." -ForegroundColor Cyan

   Write-Host
   write-host "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..." -ForegroundColor Cyan
   Stop-ClusterResource -Name $SAPIPresourceName
   sleep 5

   Write-Host "Starting SAP cluster role '$SAPClusterRoleName' ..." -ForegroundColor Cyan
   Start-ClusterGroup -Name $SAPClusterRoleName

   Write-Host "New ProbePort parameter is active." -ForegroundColor Green
   Write-Host

   Write-Host "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" -ForegroundColor Cyan
   Write-Host
   Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
   }else
   {
   Write-Host "Changes are not activated."
   }
   ```

   Après avoir mis en ligne le rôle de cluster **SAP <*SID*>** , vérifiez que **ProbePort** est défini sur la nouvelle valeur.

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPNetworkIPClusterName = "SAP $SAPSID IP"
   Get-ClusterResource $SAPNetworkIPClusterName | Get-ClusterParameter

   ```

   ![Figure 59 : Sonder le port de cluster après avoir défini la nouvelle valeur][sap-ha-guide-figure-3049]

   _**Figure 59 :** Sonder le port de cluster après avoir défini la nouvelle valeur_

#### <a name="open-the-windows-firewall-probe-port"></a><a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a> Ouvrir le port de sondage du pare-feu Windows

Vous devez ouvrir un port de sondage du pare-feu Windows sur les deux nœuds de cluster. Utilisez le script suivant pour ouvrir un port de sondage du pare-feu Windows. Mettez à jour les variables PowerShell de votre environnement.

  ```powershell
  $ProbePort = 62000   # ProbePort of the Azure Internal Load Balancer

  New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

**ProbePort** est réglé sur **62000**. Vous pouvez maintenant accéder au partage de fichiers **\\\ascsha-clsap\sapmnt** depuis d’autres hôtes, par exemple **ascsha-dbas**.

### <a name="install-the-database-instance"></a><a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> Installer l’instance de base de données

Pour installer l’instance de base de données, suivez la procédure décrite dans la documentation d’installation de SAP.

### <a name="install-the-second-cluster-node"></a><a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> Installer le deuxième nœud de cluster

Pour installer le deuxième cluster, suivez les étapes du guide d’installation de SAP.

### <a name="change-the-start-type-of-the-sap-ers-windows-service-instance"></a><a name="094bc895-31d4-4471-91cc-1513b64e406a"></a> Modifier le type de démarrage de l’instance de service Windows SAP ERS

Choisir le type de démarrage du service Windows SAP ERS **Automatique (début différé)** sur les deux nœuds du cluster.

![Figure 60 : Définir le type de service de l’instance SAP ERS sur Automatique (début différé)][sap-ha-guide-figure-3050]

_**Figure 60 :** Définir le type de service de l’instance SAP ERS sur Automatique (début différé)_

### <a name="install-the-sap-primary-application-server"></a><a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> Installer le serveur d’applications principal SAP

Installez l’instance de serveur d’application principal (PAS) <*SID*>-di-0 sur la machine virtuelle désignée pour héberger le PAS. Il n’existe aucune dépendance sur des paramètres spécifiques à Azure ou DataKeeper.

### <a name="install-the-sap-additional-application-server"></a><a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a> Installer le serveur d’applications supplémentaire SAP

Installez un serveur d’applications supplémentaire SAP (AAS) sur toutes les machines virtuelles désignées pour héberger une instance de serveur d’applications SAP. Par exemple, sur <*SID*>-di-1 à <*SID*>-di-&lt;n&gt;.

> [!NOTE]
> Cette étape conclut l’installation d’un système SAP NetWeaver de haute disponibilité. Procédez maintenant à un test de basculement.
>


## <a name="test-the-sap-ascsscs-instance-failover-and-sios-replication"></a><a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> Tester le basculement et la réplication SIOS de l’instance SAP ASCS/SCS
Vous pouvez facilement tester et surveiller le basculement et la réplication de disque SIOS d’une instance SAP ASCS/SCS à l’aide du Gestionnaire du cluster de basculement et de l’outil de configuration et de gestion de SIOS DataKeeper.

### <a name="sap-ascsscs-instance-is-running-on-cluster-node-a"></a><a name="65fdef0f-9f94-41f9-b314-ea45bbfea445"></a> L’instance SAP ASCS/SCS s’exécute sur le nœud de cluster A

Le groupe de clusters **SAP PR1** s’exécute sur le nœud de cluster A, par exemple sur **pr1-ascs-0**. Affectez le lecteur de disque partagé S, qui fait partie du groupe de clusters **SAP PR1** et que l’instance ASCS/SCS utilise, au nœud de cluster A.

![Figure 61 : Gestionnaire du cluster de basculement : Le groupe de cluster SAP <SID> s’exécute sur le nœud de cluster A][sap-ha-guide-figure-5000]

_**Figure 61 :** Gestionnaire du cluster de basculement : Le groupe de cluster SAP <*SID*> s’exécute sur le nœud de cluster A_

Dans l’outil de configuration et de gestion de SIOS DataKeeper, vous pouvez constater que les données du disque partagé sont répliquées de manière synchrone à partir du lecteur du volume source S sur le nœud de cluster A vers le lecteur du volume cible S sur le nœud de cluster B. Par exemple, il est répliqué de **pr1-ascs-0 [10.0.0.40]** vers **pr1-ascs-1 [10.0.0.41]** .

![Figure 62 : Dans SIOS DataKeeper, répliquer le volume local du nœud de cluster A sur le nœud de cluster B][sap-ha-guide-figure-5001]

_**Figure 62 :** Dans SIOS DataKeeper, répliquer le volume local du nœud de cluster A sur le nœud de cluster B_

### <a name="failover-from-node-a-to-node-b"></a><a name="5e959fa9-8fcd-49e5-a12c-37f6ba07b916"></a> Basculement du nœud A au nœud B

1. Choisissez l’une des options suivantes pour lancer le basculement du groupe de cluster SAP <*SID*> du nœud de cluster A au nœud de cluster B :
   - Utiliser le Gestionnaire du cluster de basculement  
   - Utiliser l’applet de commande PowerShell de cluster de basculement

   ```powershell
   $SAPSID = "PR1"     # SAP <SID>

   $SAPClusterGroup = "SAP $SAPSID"
   Move-ClusterGroup -Name $SAPClusterGroup

   ```
2. Redémarrer le nœud de cluster A dans le système d’exploitation invité Windows (cela lance le basculement automatique du groupe de cluster SAP <*SID*> du nœud A au nœud B).  
3. Redémarrer le nœud de cluster A depuis le portail Azure (cela lance le basculement automatique du groupe de cluster SAP <*SID*> du nœud A au nœud B).  
4. Redémarrer le nœud de cluster A à l’aide d’Azure PowerShell (cela lance le basculement automatique du groupe de cluster SAP <*SID*> du nœud A au nœud B).

   Après le basculement, le groupe de clusters SAP <*SID*> s’exécute sur le nœud de cluster B. Par exemple, il s’exécute sur **pr1-ascs-1**.

   ![Figure 63 : Dans le Gestionnaire du cluster de basculement, le groupe de cluster SAP <SID> s’exécute sur le nœud de cluster B][sap-ha-guide-figure-5002]

   _**Figure 63** : Dans le Gestionnaire du cluster de basculement, le groupe de cluster SAP <*SID*> s’exécute sur le nœud de cluster B_

   Le disque partagé est maintenant monté sur le nœud de cluster B. SIOS DataKeeper réplique les données du lecteur du volume source S sur le nœud de cluster B vers le lecteur du volume cible sur le nœud de cluster A. Par exemple, il est répliqué de **pr1-ascs-1 [10.0.0.41]** vers **pr1-ascs-0 [10.0.0.40]** .

   ![Figure 64 : SIOS DataKeeper réplique le volume local du nœud de cluster B sur le nœud de cluster A][sap-ha-guide-figure-5003]

   _**Figure 64 :** SIOS DataKeeper réplique le volume local du nœud de cluster B sur le nœud de cluster A_
