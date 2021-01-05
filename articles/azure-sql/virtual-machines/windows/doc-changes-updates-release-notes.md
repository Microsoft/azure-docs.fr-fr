---
title: Modifications apportées à la documentation concernant SQL Server sur des machines virtuelles Azure
description: Découvrez les nouvelles fonctionnalités et améliorations pour les différentes versions de SQL Server sur Machines virtuelles Microsoft Azure.
services: virtual-machines-windows
author: MashaMSFT
ms.author: mathoma
tags: azure-service-management
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.service: virtual-machines-sql
ms.topic: reference
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 10/15/2020
ms.openlocfilehash: ff4e6e0451b57046fb8f07f5a1051235e1f6d0f5
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325721"
---
# <a name="documentation-changes-for-sql-server-on-azure-virtual-machines"></a>Modifications apportées à la documentation concernant SQL Server sur des machines virtuelles Azure
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Azure permet de déployer une machine virtuelle avec une image de SQL Server intégrée. Cet article résume les modifications apportées à la documentation en lien avec les nouvelles fonctionnalités et les améliorations introduites dans les mises en production récentes de [SQL Server sur des machines virtuelles Azure](https://azure.microsoft.com/services/virtual-machines/sql-server/). 

## <a name="october-2020"></a>Octobre 2020

| Modifications | Détails |
| --- | --- |
| **DNN pour AG** | Vous pouvez maintenant configurer un [écouteur DNN (Distributed Network Name)](availability-group-distributed-network-name-dnn-listener-configure.md) pour SQL Server 2019 CU8 et versions ultérieures afin de remplacer l’[écouteur VNN](availability-group-overview.md#connectivity) traditionnel, en annulant la nécessité d’un Azure Load Balancer.   | 

## <a name="september-2020"></a>Septembre 2020

| Modifications | Détails |
| --- | --- |
| **Inscription automatique auprès de l’extension** | Vous pouvez maintenant activer la fonctionnalité [Inscription automatique](sql-agent-extension-automatic-registration-all-vms.md) pour inscrire automatiquement toutes les machines virtuelles SQL Server déjà déployées sur votre abonnement auprès de [l’extension SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md). Cela s’applique à toutes les machines virtuelles existantes et inscrira automatiquement toutes les machines virtuelles SQL Server ajoutées à l’avenir.   | 


## <a name="august-2020"></a>Août 2020

| Modifications | Détails |
| --- | --- |
| **Configurer la disponibilité générale dans le portail** | Il est désormais possible de [configurer votre groupe de disponibilité via le portail Azure](availability-group-azure-portal-configure.md). Cette fonctionnalité est actuellement en préversion et en cours de déploiement. Par conséquent, si la région de votre choix n’est pas disponible, nous vous invitions à revérifier ultérieurement. | 


## <a name="july-2020"></a>Juillet 2020


| Modifications | Détails |
| --- | --- |
| **Migration du journal vers un disque Ultra** | Découvrez comment [migrer votre fichier journal vers un disque Ultra](storage-migrate-to-ultradisk.md) pour bénéficier de performances élevées et d’une faible latence. | 
| **Créer un groupe de disponibilité avec Azure PowerShell** | Il est désormais possible de simplifier la création d’un groupe de disponibilité à l’aide d’[Azure PowerShell](availability-group-az-commandline-configure.md), ainsi que d’Azure CLI. | 


## <a name="june-2020"></a>Juin 2020

| Modifications | Détails |
| --- | --- |
| **Nom de réseau distribué (DNN)** | SQL Server 2019 sur Windows Server 2016+ propose en préversion la prise en charge de l’acheminement du trafic vers votre instance de cluster de basculement (FCI) en utilisant un [nom de réseau distribué](./failover-cluster-instance-distributed-network-name-dnn-configure.md) au lieu d’utiliser Azure Load Balancer. Cette prise en charge simplifie et rationalise la connexion à votre solution de haute disponibilité dans Azure. | 
| **FCI avec des disques partagés Azure** | Vous pouvez désormais déployer votre [instance de cluster de basculement (FCI)](failover-cluster-instance-overview.md) à l’aide de [disques partagés Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md). |
| **Documents FCI réorganisés** | La documentation relative aux [instances de cluster de basculement avec SQL Server sur des machines virtuelles Azure](failover-cluster-instance-overview.md) a été réécrite et réorganisée pour plus de clarté. Nous avons séparé une partie du contenu de configuration, comme les [meilleures pratiques en matière de configuration de cluster](hadr-cluster-best-practices.md), la façon de préparer une [machine virtuelle pour une FCI SQL Server](failover-cluster-instance-prepare-vm.md) et la manière de configurer [Azure Load Balancer](./availability-group-vnn-azure-load-balancer-configure.md). | 
| &nbsp; | &nbsp; |


## <a name="may-2020"></a>Mai 2020 

| Modifications | Détails |
| --- | --- |
| **Famille Azure SQL** | SQL Server sur Machines virtuelles Microsoft Azure fait désormais partie intégrante de la [famille de produits Azure SQL](../../azure-sql-iaas-vs-paas-what-is-overview.md). Découvrez notre [nouvelle apparence](../index.yml) ! Rien n’a changé dans le produit, mais la documentation vise à faciliter le choix du produit Azure SQL. | 


## <a name="january-2020"></a>Janvier 2020

| Modifications | Détails |
| --- | --- |
| **Prise en charge d’Azure Government** | Il est désormais possible d’inscrire des machines virtuelles SQL Server auprès de l’extension SQL IaaS Agent pour les machines virtuelles hébergées dans le cloud [Azure Government](https://azure.microsoft.com/global-infrastructure/government/). | 
| &nbsp; | &nbsp; |

## <a name="2019"></a>2019

|Modifications | Détails |
 --- | --- |
| **Réplica de récupération d’urgence gratuit dans Azure** | Vous pouvez héberger une [instance passive gratuite](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure) pour la récupération d’urgence dans Azure pour votre instance SQL Server locale si vous disposez de [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default?rtc=1&activetab=software-assurance-default-pivot:primaryr3). | 
| **Inscription en bloc auprès de l’extension SQL IaaS** | Vous pouvez désormais [inscrire en bloc](sql-agent-extension-manually-register-vms-bulk.md) des machines virtuelles SQL Server auprès de [l’extension SQL IaaS Agent](sql-server-iaas-agent-extension-automate-management.md). | 
|**Configuration du stockage à performances optimisées** | Vous pouvez désormais [personnaliser totalement votre configuration de stockage](storage-configuration.md#new-vms) lors de la création d’une machine virtuelle SQL Server. |
|**Partage de fichiers Premium pour instance de cluster de basculement** | Vous pouvez maintenant créer une instance de cluster de basculement à l’aide d’un [partage de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md) plutôt qu’avec la méthode d’origine d’[Espaces de stockage direct](failover-cluster-instance-storage-spaces-direct-manually-configure.md). 
| **Azure Dedicated Host** | Vous pouvez exécuter votre machine virtuelle SQL Server sur [Azure Dedicated Host](dedicated-host.md). | 
| **Migration d’une machine virtuelle SQL Server vers une autre région** | Utilisez Azure Site Recovery pour [migrer votre machine virtuelle SQL Server d’une région à une autre](move-sql-vm-different-region.md). |
|  **Nouveaux modes d’installation de SQL IaaS** | Il est désormais possible d’installer l’extension IaaS SQL Server en [mode allégé](sql-server-iaas-agent-extension-automate-management.md) pour éviter de redémarrer le service SQL Server.  |
| **Modification de l’édition de SQL Server** | Vous pouvez désormais changer la [propriété d’édition](change-sql-server-edition.md) de votre machine virtuelle SQL Server. |
| **Modifications apportées à l’extension SQL IaaS Agent** | Vous pouvez [inscrire votre machine virtuelle SQL Server auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md) en utilisant les nouveaux modes IaaS SQL. Cette fonctionnalité inclut des images [Windows Server 2008](sql-server-iaas-agent-extension-automate-management.md#management-modes).|
| **Images BYOL (apportez votre propre licence) à l’aide d’Azure Hybrid Benefit** | Les images BYOL (apportez votre propre licence) déployées à partir de la Place de marché Azure peuvent désormais changer leur [type de licence avec paiement à l'utilisation](licensing-model-azure-hybrid-benefit-ahb-change.md#remarks).| 
| **Nouvelle gestion des machines virtuelles SQL Server dans le portail Azure** | Il y a désormais une façon de gérer votre machine virtuelle SQL Server dans le portail Azure. Pour plus d’informations, consultez [Gérer des machines virtuelles SQL Server dans le portail Azure](manage-sql-vm-portal.md).  | 
| **Support étendu pour SQL Server 2008 et 2008 R2** | [Étendez la prise en charge](sql-server-2008-extend-end-of-support.md) pour SQL Server 2008 et SQL Server 2008 R2 en migrant *tel quel* dans une machine virtuelle Azure. | 
| **Prise en charge de l’image personnalisée** | Vous pouvez maintenant installer l’[extension IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md#installation) pour les images du système d’exploitation et SQL Server personnalisées, qui offre des fonctionnalités limitées de [flexibilité de licence](licensing-model-azure-hybrid-benefit-ahb-change.md). Lorsque vous inscrivez votre image personnalisée auprès de l’extension SQL IaaS Agent, spécifiez le type de licence « AHUB ». Sinon, l’inscription échoue. | 
| **Prise en charge d’instance nommée** | Vous pouvez maintenant utiliser l’[extension IaaS SQL Server](sql-server-iaas-agent-extension-automate-management.md#installation) avec une instance nommée si l’instance par défaut a été correctement désinstallée. | 
| **Amélioration du portail** | L’expérience du portail Azure pour le déploiement d’une machine virtuelle SQL Server a été revisitée pour en améliorer son usage. Pour plus d’informations, consultez le court [Démarrage rapide](sql-vm-create-portal-quickstart.md) et un [Guide pratique](create-sql-vm-portal.md) plus détaillé pour déployer d’une machine virtuelle SQL Server.|
| **Amélioration du portail** | Il est désormais possible de changer le modèle de licence d’une machine virtuelle SQL Server de « Paiement à l’utilisation » en « BYOL » (apportez votre propre licence) en utilisant le [portail Azure](licensing-model-azure-hybrid-benefit-ahb-change.md#change-license-model).|
| **Simplification du déploiement d’un groupe de disponibilité sur une machine virtuelle SQL Server par le biais d’Azure CLI** | Il est désormais plus facile que jamais de déployer un groupe de disponibilité sur une machine virtuelle SQL Server dans Azure. Vous pouvez utiliser [Azure CLI](/cli/azure/sql/vm?view=azure-cli-2018-03-01-hybrid&preserve-view=true) pour créer le cluster de basculement Windows, l’équilibreur de charge interne et les écouteurs de groupe de disponibilité, le tout à partir de la ligne de commande. Pour plus d’informations, consultez [Utiliser l’interface de ligne de commande Azure pour configurer un groupe de disponibilité Always On pour SQL Server sur une machine virtuelle Azure](./availability-group-az-commandline-configure.md). | 
| &nbsp; | &nbsp; |

## <a name="2018"></a>2018 

 Modifications | Détails |
| --- | --- |
|  **Nouveau fournisseur de ressources pour un cluster SQL Server** | Un nouveau fournisseur de ressources (Microsoft.SqlVirtualMachine/SqlVirtualMachineGroups) définit les métadonnées du cluster de basculement Windows. Joindre une machine virtuelle SQL Server à *SqlVirtualMachineGroups* amorce le cluster de basculement Windows Server (WSFC) et joint la machine virtuelle au cluster.  |
| **Configuration automatisée du déploiement d’un groupe de disponibilité avec des modèles de démarrage rapide Azure** |Il est désormais possible de créer le cluster de basculement Windows, d’y joindre des machines virtuelles SQL Server, de créer l’écouteur et de configurer l’équilibreur de charge interne à l’aide de deux modèles de démarrage rapide Azure. Pour plus d’informations, consultez [Utiliser un modèle de démarrage rapide Azure pour configurer un groupe de disponibilité Always On pour SQL Server sur une machine virtuelle Azure](availability-group-quickstart-template-configure.md). | 
| **Inscription automatique auprès de l’extension SQL IaaS Agent** | Les machines virtuelles SQL Server déployées après la fin du mois sont automatiquement inscrites auprès de la nouvelle extension SQL IaaS Agent. Les machines virtuelles SQL Server déployées dans les mois précédents nécessitent quand même une inscription manuelle. Pour plus d’informations, consultez [Inscrire une machine virtuelle SQL Server dans Azure auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md).|
|**Nouvelle extension SQL IaaS Agent** |  Un nouveau fournisseur de ressources pour les machines virtuelles SQL Server (Microsoft.SqlVirtualMachine) permet de mieux gérer vos machines virtuelles SQL Server. Pour plus d’informations sur l’inscription de vos machines virtuelles, consultez [Inscrire une machine virtuelle SQL Server dans Azure auprès de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md). |
|**Changer de modèle de licence** | Vous pouvez maintenant passer du modèle de paiement à l’utilisation au modèle BYOL (apportez votre propre licence) pour votre machine virtuelle SQL Server à l’aide de l’interface de ligne de commande Azure CLI ou de PowerShell. Pour plus d'informations, consultez [Guide pratique pour changer le modèle de licence d'une machine virtuelle SQL Server](licensing-model-azure-hybrid-benefit-ahb-change.md). | 
| &nbsp; | &nbsp; |

## <a name="additional-resources"></a>Ressources supplémentaires

**Machines virtuelles Windows** :

* [Vue d’ensemble de SQL Server sur une machine virtuelle Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Approvisionner SQL Server sur une machine virtuelle Windows](create-sql-vm-portal.md)
* [Migrer d’une base de données vers SQL Server sur une machine virtuelle Azure](migrate-to-vm-from-sql-server.md)
* [Haute disponibilité et récupération d’urgence pour SQL Server sur des machines virtuelles Azure](business-continuity-high-availability-disaster-recovery-hadr-overview.md)
* [Meilleures pratiques relatives aux performances de SQL Server sur les machines virtuelles Azure](performance-guidelines-best-practices.md)
* [Modèles d'application et stratégies de développement pour SQL Server sur des machines virtuelles Azure](application-patterns-development-strategies.md)

**Machines virtuelles Linux** :

* [Vue d’ensemble de SQL Server sur une machine virtuelle Linux](../linux/sql-server-on-linux-vm-what-is-iaas-overview.md)
* [Approvisionner SQL Server une machine virtuelle Linux](../linux/sql-vm-create-portal-quickstart.md)
* [Forum Aux Questions (Linux)](../linux/frequently-asked-questions-faq.md)
* [Documentation relative à SQL Server sur Linux](/sql/linux/sql-server-linux-overview)