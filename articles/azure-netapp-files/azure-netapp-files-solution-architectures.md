---
title: Architectures de solution avec Azure NetApp Files | Microsoft Docs
description: Cet article contient des références aux meilleures pratiques associées aux architectures de solution avec Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 12/15/2020
ms.author: b-juche
ms.openlocfilehash: 2b567eba43cfec9af05ebea262b3f6bd61b00259
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562546"
---
# <a name="solution-architectures-using-azure-netapp-files"></a>Architectures de solution avec Azure NetApp Files
Cet article fournit des références aux meilleures pratiques pouvant vous aider à comprendre les architectures de solution pour l’utilisation d’Azure NetApp Files.  

Le diagramme suivant résume les catégories d’architectures de solution que Azure NetApp Files offre :

![Catégories d’architectures de solution](../media/azure-netapp-files/solution-architecture-categories.png)

## <a name="linux-oss-apps-and-database-solutions"></a>Applications et solutions de base de données OSS Linux

Cette section fournit des références pour les solutions de bases de données et applications OSS Linux. 

### <a name="oracle"></a>Oracle

* [Performances des bases de données Oracle sur des volumes uniques Azure NetApp Files](performance-oracle-single-volumes.md)
* [Oracle on Azure deployment best practice guide - Using Azure NetApp Files](https://www.netapp.com/us/media/tr-4780.pdf)
* [Images de machine virtuelle Oracle et leur déploiement sur Microsoft Azure : options de configuration de stockage partagé](../virtual-machines/workloads/oracle/oracle-vm-solutions.md#shared-storage-configuration-options)
* [Avantages de l’utilisation d’Azure NetApp Files avec Oracle Database](solutions-benefits-azure-netapp-files-oracle-database.md)

### <a name="machine-learning"></a>Machine Learning
*   [Cloudera Machine Learning](https://docs.cloudera.com/machine-learning/cloud/requirements-azure/topics/ml-requirements-azure.html)

## <a name="windows-apps-and-sql-server-solutions"></a>Applications Windows et les solutions de SQL Server

Cette section fournit des références pour les applications Windows et les solutions de SQL Server.

### <a name="file-sharing-and-global-file-caching"></a>Partage de fichiers et mise en cache de fichiers globaux

* [Build Your Own Azure NFS? Wrestling Linux File Shares into Cloud](https://cloud.netapp.com/blog/ma-anf-blg-build-your-own-linux-nfs-file-shares)
* [Déploiement de Global File Cache / Azure NetApp Files](https://youtu.be/91LKb1qsLIM)
* [Conformité cloud pour Azure NetApp Files](https://cloud.netapp.com/hubfs/Cloud%20Compliance%20for%20Azure%20NetApp%20Files%20-%20November%202020.pdf)

### <a name="sql-server"></a>SQL Server

* [Déployer des SQL Server sur SMB avec Azure NetApp Files](https://www.youtube.com/watch?v=x7udfcYbibs)
<!-- * [Deploy SQL Server Always-On Failover Cluster over SMB with Azure NetApp Files](https://www.youtube.com/watch?v=zuNJ5E07e8Q) --> 
<!-- * [Deploy Always-On Availability Groups with Azure NetApp Files](https://www.youtube.com/watch?v=y3VQmzzeyvc) --> 

## <a name="sap-on-azure-solutions"></a>Solutions SAP sur Azure

Cette section fournit des références pour les Solutions SAP sur Azure. 

### <a name="generic-sap-and-sap-netweaver"></a>SAP et SAP Netweavereaver génériques 

* [SAP applications on Microsoft Azure - Using Azure NetApp Files](https://www.netapp.com/us/media/tr-4746.pdf)
* [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](../virtual-machines/workloads/sap/high-availability-guide-suse-netapp-files.md)
* [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur Red Hat Enterprise Linux avec Azure NetApp Files pour les applications SAP](../virtual-machines/workloads/sap/high-availability-guide-rhel-netapp-files.md)
* [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur Windows avec Azure NetApp Files (SMB) pour les applications SAP](../virtual-machines/workloads/sap/high-availability-guide-windows-netapp-files-smb.md)
* [Guide multi-SID de haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur Red Hat Enterprise Linux for SAP Applications](../virtual-machines/workloads/sap/high-availability-guide-rhel-multi-sid.md)

### <a name="sap-hana"></a>SAP HANA 

* [Configurations du stockage des machines virtuelles SAP HANA Azure](../virtual-machines/workloads/sap/hana-vm-operations-storage.md)
* [Haute disponibilité du scale-up SAP HANA avec Azure NetApp Files sur Red Hat Enterprise Linux](../virtual-machines/workloads/sap/sap-hana-high-availability-netapp-files-red-hat.md)
* [Scale-out de SAP HANA avec le nœud de secours sur les machines virtuelles Azure avec Azure NetApp Files sur SUSE Linux Enterprise Server](../virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-suse.md)
* [Scale-out de SAP HANA avec le nœud de secours sur les machines virtuelles Azure avec Azure NetApp Files sur Red Hat Enterprise Linux](../virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-rhel.md)

### <a name="sap-iq-nls"></a>SAP IQ-NLS
*   [Déploiement de la solution de haute disponibilité SAP IQ-NLS avec Azure NetApp Files sur SUSE Linux Enterprise Server](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/deploy-sap-iq-nls-ha-solution-using-azure-netapp-files-on-suse/ba-p/1651172#.X2tDfpNzBh4.linkedin)

### <a name="sap-tech-community-and-blog-posts"></a>Billets de blog et communauté SAP Tech 

* [Azure NetApp Files : sauvegarde SAP HANA en secondes](https://blog.netapp.com/azure-netapp-files-sap-hana-backup-in-seconds/)
* [Azure NetApp Files : restaurer votre base de données HANA à partir d’une sauvegarde d’instantané](https://blog.netapp.com/azure-netapp-files-backup-sap-hana)
* [Azure NetApp Files : déchargement de sauvegardes SAP HANA avec Cloud Sync](https://blog.netapp.com/azure-netapp-files-sap-hana)
* [Accélérez vos copies système SAP HANA à l’aide d’Azure NetApp Files](https://blog.netapp.com/sap-hana-faster-using-azure-netapp-files/)
* [Cloud Volumes ONTAP et Azure NetApp Files : migration du système SAP HANA facilitée](https://blog.netapp.com/cloud-volumes-ontap-and-azure-netapp-files-sap-hana-system-migration-made-easy/)

## <a name="virtual-desktop-infrastructure-solutions"></a>Solutions d’infrastructure de bureau virtuel

Cette section fournit des références pour les solutions d’infrastructure de bureau virtuel.

### <a name="windows-virtual-desktop"></a>Windows Virtual Desktop

* [Avantages d’utiliser Azure NetApp Files avec Windows Virtual Desktop](solutions-windows-virtual-desktop.md)
* [Options de stockage pour conteneurs de profil FSLogix dans Windows Virtual Desktop](../virtual-desktop/store-fslogix-profile.md#azure-platform-details)
* [Créer un conteneur de profils FSLogix pour un pool d’hôtes à l'aide d’Azure NetApp Files](../virtual-desktop/create-fslogix-profile-container.md)
* [Windows Virtual Desktop à l’échelle de l’entreprise](/azure/architecture/example-scenario/wvd/windows-virtual-desktop)
* [Microsoft FSLogix pour l’entreprise – Meilleures pratiques Azure NetApp Files](/azure/architecture/example-scenario/wvd/windows-virtual-desktop-fslogix#azure-netapp-files-best-practices)

## <a name="hpc-solutions"></a>Solutions HPC

Cette section fournit des références pour les solutions HPC (Calcul haute performance). 

### <a name="generic-hpc"></a>HPC générique

* [Azure NetApp Files : Tirer le meilleur parti de votre stockage cloud](https://cloud.netapp.com/hubfs/Resources/ANF%20PERFORMANCE%20TESTING%20IN%20TEMPLATE.pdf)
* [Exécuter des charges de travail MPI avec Azure Batch et Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
* [Cloud Azure Cloud : Environnements HPC CycleCloud sur Azure NetApp Files](/azure/cyclecloud/overview)

### <a name="oil-and-gas"></a>Hydrocarbures

* [Calcul haute performance (HPC) : Hydrocarbures dans Azure](https://techcommunity.microsoft.com/t5/azure-global/high-performance-computing-hpc-oil-and-gas-in-azure/ba-p/824926)
* [Exécuter le logiciel de simulation de réservoir sur Azure](/azure/architecture/example-scenario/infrastructure/reservoir-simulation)

### <a name="electronic-design-automation-eda"></a>Automatisation de la conception électronique (EDA)

* [Avantages de l’utilisation d’Azure NetApp Files pour l’automatisation de la conception électronique](solutions-benefits-azure-netapp-files-electronic-design-automation.md)
* [Azure CycleCloud : Laboratoire EDA HPC avec Azure NetApp Files](https://github.com/Azure/cyclecloud-hands-on-labs/blob/master/EDA/README.md)
* [Azure pour le secteur des semi-conducteurs](https://azurecomcdn.azureedge.net/cvt-f40f39cd9de2d875ab0c198a8d7b186350cf0bca161e80d7896941389685d012/mediahandler/files/resourcefiles/azure-for-the-semiconductor-industry/Azure_for_the_Semiconductor_Industry.pdf)

### <a name="analytics"></a>Analytics

* [Azure NetApp Files : un système de fichiers partagé à utiliser avec la grille SAS sur Microsoft Azure](https://communities.sas.com/t5/Administration-and-Deployment/Azure-NetApp-Files-A-shared-file-system-to-use-with-SAS-Grid-on/m-p/705192)
* [Meilleures pratiques d’utilisation de Microsoft Azure avec SAS®](https://communities.sas.com/t5/Administration-and-Deployment/Best-Practices-for-Using-Microsoft-Azure-with-SAS/m-p/676833#M19680)

## <a name="azure-platform-services-solutions"></a>Déployer des solutions pour les services de plateforme Azure

Cette section fournit des solutions pour les services de plateforme Azure. 

### <a name="azure-kubernetes-services-and-kubernetes"></a>Azure Kubernetes Serviceset Kubernetes

* [Intégrer Azure NetApp Files à Azure Kubernetes Service](../aks/azure-netapp-files.md)
* [Performances exceptionnelles de Kubernetes sur Azure avec Azure NetApp Files](https://cloud.netapp.com/blog/ma-anf-blg-configure-kubernetes-openshift)
* [Trident - Storage Orchestrator pour conteneurs](https://netapp-trident.readthedocs.io/en/stable-v20.04/kubernetes/operations/tasks/backends/anf.html)

### <a name="azure-batch"></a>Azure Batch

* [Exécuter des charges de travail MPI avec Azure Batch et Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
