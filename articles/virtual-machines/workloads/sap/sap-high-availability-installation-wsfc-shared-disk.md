---
title: Installer la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un disque partagé pour une instance SAP ASCS/SCS dans Azure | Microsoft Docs
description: Découvrez comment installer la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un disque partagé pour une instance SAP ASCS/SCS.
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: 6209bcb3-5b20-4845-aa10-1475c576659f
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/12/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 1324afb38560afeb43b5be9191b6e2e7afc5c81d
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654895"
---
# <a name="install-sap-netweaver-ha-on-a-windows-failover-cluster-and-shared-disk-for-an-sap-ascsscs-instance-in-azure"></a>Installer la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un disque partagé pour une instance SAP ASCS/SCS dans Azure

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
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-ha-guide-8.9]:high-availability-guide.md#fe0bd8b5-2b43-45e3-8295-80bee5415716
[sap-ha-guide-8.11]:high-availability-guide.md#661035b2-4d0f-4d31-86f8-dc0a50d78158
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md

[planning-guide]:planning-guide.md
[planning-guide-11]:planning-guide.md
[planning-guide-2.1]:planning-guide.md#1625df66-4cc6-4d60-9202-de8a0b77f803
[planning-guide-2.2]:planning-guide.md#f5b3b18c-302c-4bd8-9ab2-c388f1ab3d10

[planning-guide-microsoft-azure-networking]:planning-guide.md#61678387-8868-435d-9f8c-450b2424f5bd
[planning-guide-storage-microsoft-azure-storage-and-data-disks]:planning-guide.md#a72afa26-4bf4-4a25-8cf7-855d6032157f

[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md

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
[sap-ha-guide-figure-3046-ers2]:./media/virtual-machines-shared-sap-high-availability-guide/3046-dns-entry-sap-ers2-virtual-name-ip.png
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

Cet article explique comment installer et configurer un système SAP à haute disponibilité dans Azure à l’aide d’un cluster de basculement Windows Server et d’un disque partagé de cluster pour le clustering d’une instance SAP ASCS/SCS. Comme décrit dans [Guide d’architecture : Clustering d’une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster][sap-high-availability-guide-wsfc-shared-disk], il existe deux possibilités pour un *disque partagé de cluster* :

- [Disques partagés Azure](../../disks-shared.md)
- Utilisation de [SIOS DataKeeper Cluster Edition](https://us.sios.com/products/datakeeper-cluster/) pour créer un stockage en miroir, qui simule le disque partagé en cluster 

## <a name="prerequisites"></a>Prérequis

Avant de commencer l’installation, consultez ces documents :

* [Guide d'architecture : Mettre en cluster une instance SAP ASCS/SCS sur un cluster de basculement Windows à l’aide d’un disque partagé de cluster][sap-high-availability-guide-wsfc-shared-disk]

* [Préparer l’infrastructure Azure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows et d’un disque partagé pour une instance SAP ASCS/SCS][sap-high-availability-infrastructure-wsfc-shared-disk]

Nous ne décrivons pas l’installation du système SGBD dans cet article, car celle-ci varie en fonction du système utilisé. Nous partons du principe que la haute disponibilité du SGBD est assurée par les fonctionnalités prises en charge par les différents fournisseurs de SGBD pour Azure, par exemple AlwaysOn ou la mise en miroir de base de données pour SQL Server, et Oracle Data Guard pour les bases de données Oracle. Les scénarios de haute disponibilité pour le SGBD ne sont pas traités dans cet article.

Il n’existe pas de considérations particulières lorsque différents services de SGBD interagissent avec une configuration SAP ASCS ou SCS en cluster dans Azure.

> [!NOTE]
> Les procédures d’installation de systèmes SAP NetWeaver ABAP, Java et ABAP+Java sont presque identiques. La différence la plus notable est qu’un système SAP ABAP comprend une instance ASCS, le système SAP Java une instance SCS, et le système SAP ABAP+Java une instance ASCS et une instance SCS exécutées dans le même groupe de cluster de basculement Microsoft. Toute différence d’installation pour chaque pile d’installation de SAP NetWeaver est mentionnée explicitement. Vous pouvez considérer que les autres étapes sont identiques.  

## <a name="install-sap-with-a-high-availability-ascsscs-instance"></a><a name="31c6bd4f-51df-4057-9fdf-3fcbc619c170"></a>Installer SAP avec une instance ASCS/SCS à haute disponibilité

> [!IMPORTANT]
> Si vous utilisez SIOS pour présenter un disque partagé, ne placez pas votre fichier d’échange sur les volumes mis en miroir SIOS DataKeeper. Vous pouvez laisser votre fichier d’échange sur le lecteur temporaire D d’une machine virtuelle (valeur par défaut). S’il ne s’y trouve pas déjà, déplacez le fichier d’échange Windows sur le lecteur D de votre machine virtuelle Azure.  

L’installation de SAP avec une instance ASCS/SCS à haute disponibilité implique les tâches suivantes :

* Créer un nom d’hôte virtuel pour l’instance SAP ASCS/SCS en cluster.
* Installer SAP sur le premier nœud de cluster.
* Modifier le profil SAP de l’instance ASCS/SCS.
* Ajouter un port de sondage.
* Ouvrir le port de sondage du Pare-feu Windows.

### <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a><a name="a97ad604-9094-44fe-a364-f89cb39bf097"></a>Créer un nom d’hôte virtuel pour l’instance SAP ASCS/SCS en cluster

1. Dans le Gestionnaire DNS Windows, créez une entrée DNS pour le nom d’hôte virtuel de l’instance ASC/SCS.

   > [!IMPORTANT]
   > L’adresse IP que vous affectez au nom d’hôte virtuel de l’instance ASC/SCS doit être identique à celle que vous avez affectée à Azure Load Balancer.  

   ![Figure 1 : Définir l’entrée DNS pour le nom virtuel et l’adresse TCP/IP du cluster SAP ASCS/SCS][sap-ha-guide-figure-3046]

   _Définir l’entrée DNS pour le nom virtuel et l’adresse TCP/IP du cluster SAP ASCS/SCS_

2. Si vous utilisez le nouveau serveur SAP Enqueue Replication Server 2, qui est également une instance en cluster, vous devez aussi réserver dans le DNS un nom d’hôte virtuel pour ERS2. 

   > [!IMPORTANT]
   > L’adresse IP que vous affectez au nom d’hôte virtuel de l’instance ERS2 doit être la deuxième adresse IP que vous avez affectée à Azure Load Balancer.    

   ![Figure 1A : Définir l’entrée DNS pour le nom virtuel et l’adresse TCP/IP du cluster SAP ASCS/SCS][sap-ha-guide-figure-3046-ers2]

   _Définir l’entrée DNS pour le nom virtuel et l’adresse TCP/IP du cluster SAP ERS2_

3. Pour définir l’adresse IP affectée au nom d’hôte virtuel, sélectionnez **Gestionnaire DNS** > **Domaine**.

   ![Figure 2 : Nouveau nom virtuel et nouvelle adresse TCP/IP de la configuration du cluster SAP ASCS/SCS][sap-ha-guide-figure-3047]

   _Nouveau nom virtuel et nouvelle adresse TCP/IP de la configuration du cluster SAP ASCS/SCS_

### <a name="install-the-sap-first-cluster-node"></a><a name="eb5af918-b42f-4803-bb50-eff41f84b0b0"></a> Installer le premier nœud de cluster SAP

1. Exécutez l’option du premier nœud de cluster sur le nœud A. Sélectionnez :

   * **Système ABAP** : Numéro d’instance **ASCS** **00**
   * **Système Java** : Numéro d’instance **SCS** **01**
   * **Système ABAP+Java** : Numéro d’instance **ASCS** **00** et numéro d’instance **SCS** **01**

   > [!IMPORTANT]
   > N’oubliez pas que la configuration dans les règles d’équilibrage de charge de l’équilibreur de charge interne Azure (si vous utilisez la référence SKU De base) et les numéros d’instance SAP sélectionnés doivent correspondre.

2. Suivez la procédure d’installation décrite par SAP. Dans l’option de démarrage de l’installation « Premier nœud de cluster », veillez à choisir « Disque partagé de cluster » comme option de configuration.

> [!TIP]
> La documentation d’installation de SAP explique comment installer le premier nœud de cluster ASC/SCS.

### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a><a name="e4caaab2-e90f-4f2c-bc84-2cd2e12a9556"></a> Modifier le profil SAP de l’instance ASCS/SCS

Si vous avez Enqueue Replication Server 1, ajoutez le paramètre de profil SAP `enque/encni/set_so_keepalive`, comme décrit ci-dessous. Ce paramètre de profil empêche les connexions entre les processus de travail SAP et le serveur de mise en file d’attente de se fermer lorsqu’elles sont inactives pendant trop longtemps. Le paramètre SAP n’est pas nécessaire pour ERS2. 

1. Si vous utilisez ERS1, ajoutez ce paramètre de profil au profil de l’instance SAP ASCS/SCS.

   ```
   enque/encni/set_so_keepalive = true
   ```

   Pour ERS1 et ERS2, assurez-vous que les paramètres de système d’exploitation `keepalive` sont définis comme décrit dans la note SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).   

2. Pour appliquer les changements de paramètres du profil SAP, redémarrez l’instance SAP ASCS/SCS.

### <a name="add-a-probe-port"></a><a name="10822f4f-32e7-4871-b63a-9b86c76ce761"></a> Ajouter un port de sondage

Utilisez la fonctionnalité de sondage de l’équilibrage de charge interne pour que la configuration de cluster entière fonctionne avec Azure Load Balancer. Généralement, l’équilibrage de charge interne Azure répartit équitablement la charge de travail entrante entre les machines virtuelles participantes.

Toutefois, cela ne fonctionne pas dans certaines configurations de cluster, car une seule instance est active. L’autre instance est passive et ne peut accepter aucune partie de la charge de travail. La fonctionnalité de sondage est utile quand l’équilibreur de charge interne Azure détecte quelle instance est active, et ne cible que l’instance active.  

> [!IMPORTANT]
> Dans cet exemple de configuration, le paramètre **ProbePort** est défini sur 620 **Nr**. Pour l’instance SAP ASCS portant le numéro **00**, il est défini sur 620 **00**. Vous devrez ajuster la configuration pour qu’elle corresponde à vos numéros d’instance SAP et à votre SID SAP.

Pour ajouter un port de sonde, exécutez ce module PowerShell sur l’une des machines virtuelles du cluster :

- Dans le cas d’une instance SAP ASC/SCS 
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID SID -ProbePort 62000
   ```

- Si vous utilisez ERS2, qui est en cluster. Il n’est pas nécessaire de configurer le port de sonde pour ERS1, car il n’est pas en cluster.  
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID SID -ProbePort 62001 -IsSAPERSClusteredInstance $True
   ```

 Le code de la fonction `Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource` peut être semblable au suivant :
   ```powershell
    function Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource {

    <#
    .SYNOPSIS 
    Set-AzureLoadBalancerHealthProbePortOnSAPClusterIPResource will set a new Azure Load Balancer Health Probe Port on 'SAP $SAPSID IP' cluster resource.

    .DESCRIPTION
    Set-AzureLoadBalancerHealthProbePortOnSAPClusterIPResource will set a new Azure Load Balancer Health Probe Port on 'SAP $SAPSID IP' cluster resource.
    It will also restart SAP Cluster group (default behavior), to activate the changes. 

    You need to run it on one of the SAP ASCS/SCS Windows cluster nodes.

    Expectation is that SAP group is installed with official SWPM installation tool, which will set default expected naming convention for:
    - SAP Cluster Group:               'SAP $SAPSID'
    - SAP Cluster IP Address Resource: 'SAP $SAPSID IP' 

    .PARAMETER SAPSID 
    SAP SID - 3 characters staring with letter.

    .PARAMETER ProbePort 
    Azure Load Balancer Health Check Probe Port.

    .PARAMETER RestartSAPClusterGroup 
    Optional parameter. Default value is '$True', so SAP cluster group will be restarted to activate the changes.

    .PARAMETER IsSAPERSClusteredInstance 
    Optional parameter.Default value is '$False'.
    If set to $True , then handle clsutered new SAP ERS2 instance.

    .EXAMPLE 
    # Set probe port to 62000, on SAP cluster resource 'SAP AB1 IP', and restart the SAP cluster group 'SAP AB1', to activate the changes.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 

    .EXAMPLE 
    # Set probe port to 62000, on SAP cluster resource 'SAP AB1 IP'. SAP cluster group 'SAP AB1' IS NOT restarted, therefore changes are NOT active.
    # To activate the changes you need to manualy restart 'SAP AB1' cluster group.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 -RestartSAPClusterGroup $False

    .EXAMPLE 
    # Set probe port to 62001, on SAP cluster resource 'SAP AB1 ERS IP'. SAP cluster group 'SAP AB1 ERS' IS restarted, to activate the changes.
    Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID AB1 -ProbePort 62000 -IsSAPERSClusteredInstance $True

    #> 

        [CmdletBinding()]
        param(

            [Parameter(Mandatory=$True)]
            [ValidateNotNullOrEmpty()]  
            [ValidateLength(3,3)]      
            [string]$SAPSID,

            [Parameter(Mandatory=$True)]
            [ValidateNotNullOrEmpty()]        
            [int] $ProbePort,

            [Parameter(Mandatory=$False)] 
            [bool] $RestartSAPClusterGroup = $True,

            [Parameter(Mandatory=$False)] 
            [bool] $IsSAPERSClusteredInstance = $False
        )

        BEGIN{}

        PROCESS{
            try{                                      

                if($IsSAPERSClusteredInstance){
                    #Handle clustered SAP ERS Instance
                    $SAPClusterRoleName = "SAP $SAPSID ERS"
                    $SAPIPresourceName = "SAP $SAPSID ERS IP"            
                }else{
                    #Handle clustered SAP ASCS/SCS Instance
                    $SAPClusterRoleName = "SAP $SAPSID"
                    $SAPIPresourceName = "SAP $SAPSID IP"
                }

                $SAPIPResourceClusterParameters =  Get-ClusterResource $SAPIPresourceName | Get-ClusterParameter
                $IPAddress = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Address" }).Value
                $NetworkName = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "Network" }).Value
                $SubnetMask = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "SubnetMask" }).Value
                $OverrideAddressMatch = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "OverrideAddressMatch" }).Value
                $EnableDhcp = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "EnableDhcp" }).Value
                $OldProbePort = ($SAPIPResourceClusterParameters | Where-Object {$_.Name -eq "ProbePort" }).Value

                $var = Get-ClusterResource | Where-Object {  $_.name -eq $SAPIPresourceName  }
                Write-Output "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" 

                Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter

                Write-Output " "
                Write-Output "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." 
                Write-Output " "
                Write-Output "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." 
                Write-Output " "

                $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}

                Write-Output " "

                if($RestartSAPClusterGroup){
                    Write-Output ""
                    Write-Output "Activating changes..." 

                    Write-Output " "
                    Write-Output "Taking SAP cluster IP resource '$SAPIPresourceName' offline ..."
                    Stop-ClusterResource -Name $SAPIPresourceName
                    sleep 5

                    Write-Output "Starting SAP cluster role '$SAPClusterRoleName' ..."
                    Start-ClusterGroup -Name $SAPClusterRoleName

                    Write-Output "New ProbePort parameter is active." 
                    Write-Output " "

                    Write-Output "New configuration parameters for SAP IP cluster resource '$SAPIPresourceName':" 
                    Write-Output " " 
                    Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
                }else
                {
                    Write-Output "SAP cluster role '$SAPClusterRoleName' is not restarted, therefore changes are not activated."
                }
            }
            catch{
               Write-Error  $_.Exception.Message
           }
        }
        END {}
    }

   ```

### <a name="open-the-windows-firewall-probe-port"></a><a name="4498c707-86c0-4cde-9c69-058a7ab8c3ac"></a> Ouvrir le port de sondage du pare-feu Windows

Ouvrez un port de sondage du pare-feu Windows sur les deux nœuds de cluster. Utilisez le script suivant pour ouvrir un port de sondage du pare-feu Windows. Mettez à jour les variables PowerShell de votre environnement.  
Si vous utilisez ERS2, vous devez également ouvrir le port du pare-feu pour le port de sonde ERS2.  

  ```powershell
    $ProbePort = 62000   # ProbePort of the Azure internal load balancer
    New-NetFirewallRule -Name AzureProbePort -DisplayName "Rule for Azure Probe Port" -Direction Inbound -Action Allow -Protocol TCP -LocalPort $ProbePort
  ```

## <a name="install-the-database-instance"></a><a name="85d78414-b21d-4097-92b6-34d8bcb724b7"></a> Installer l’instance de base de données

Pour installer l’instance de base de données, suivez la procédure décrite dans la documentation d’installation de SAP.

## <a name="install-the-second-cluster-node"></a><a name="8a276e16-f507-4071-b829-cdc0a4d36748"></a> Installer le deuxième nœud de cluster

Pour installer le deuxième cluster, suivez les étapes décrites dans le guide d’installation de SAP.

## <a name="install-the-sap-primary-application-server"></a><a name="2477e58f-c5a7-4a5d-9ae3-7b91022cafb5"></a> Installer le serveur d’applications principal SAP

Installez l’instance de serveur d’application principal (PAS) \<SID\>-di-0 sur la machine virtuelle désignée pour héberger le PAS. Il n’y a aucune dépendance vis-à-vis d’Azure. Si vous utilisez SIOS, il n’y a aucun paramètre propre à DataKeeper.

## <a name="install-the-sap-additional-application-server"></a><a name="0ba4a6c1-cc37-4bcf-a8dc-025de4263772"></a> Installer le serveur d’applications supplémentaire SAP

Installez un serveur d’applications supplémentaire SAP (AAS) sur toutes les machines virtuelles désignées pour héberger une instance de serveur d’applications SAP. 

## <a name="test-the-sap-ascsscs-instance-failover"></a><a name="18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9"></a> Tester le basculement de l’instance SAP ASCS/SCS

Pour les tests de basculement décrits, nous supposons que SAP ASCS est actif sur le nœud A.  

1. Vérifiez que le système SAP peut effectuer un basculement du nœud A vers le nœud B. Choisissez l’une des options suivantes pour lancer un basculement du groupe de clusters \<SID\> SAP du nœud de cluster A au nœud de cluster B :
    - Gestionnaire du cluster de basculement  
    - Applet de commande PowerShell de cluster de basculement

    ```powershell
    $SAPSID = "PR1"     # SAP <SID>

    $SAPClusterGroup = "SAP $SAPSID"
    Move-ClusterGroup -Name $SAPClusterGroup

    ```

2. Redémarrez le nœud de cluster A dans le système d’exploitation invité Windows. Cela lance le basculement automatique du groupe de clusters \<SID\> SAP du nœud A vers le nœud B.  
3. Redémarrez le nœud de cluster A à partir du portail Azure. Cela lance le basculement automatique du groupe de clusters \<SID\> SAP du nœud A vers le nœud B.  
4. Redémarrez le nœud de cluster A à l’aide d’Azure PowerShell. Cela lance le basculement automatique du groupe de clusters \<SID\> SAP du nœud A vers le nœud B.

5. Vérification
   - Après le basculement, vérifiez que le groupe de cluster \<SID\> SAP s’exécute sur le nœud de cluster B. 

      ![Figure 8 : Dans le Gestionnaire du cluster de basculement, le groupe de cluster SAP \<SID\> s’exécute sur le nœud de cluster B][sap-ha-guide-figure-5002]

      _Dans le Gestionnaire du cluster de basculement, le groupe de cluster \<SID\> SAP s’exécute sur le nœud de cluster B_

   - Après le basculement, vérifiez que le disque partagé est maintenant monté sur le nœud de cluster B. 
   - Après le basculement, si vous utilisez SIOS, vérifiez que SIOS DataKeeper réplique les données à partir du lecteur du volume source S sur le nœud de cluster B vers le lecteur du volume cible S sur le nœud de cluster A. 

      ![Figure 9 : SIOS DataKeeper réplique le volume local du nœud de cluster B sur le nœud de cluster A][sap-ha-guide-figure-5003]

      _SIOS DataKeeper réplique le volume local du nœud de cluster B sur le nœud de cluster A_