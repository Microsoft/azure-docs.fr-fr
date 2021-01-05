---
title: Préparer des machines virtuelles Azure pour une instance FCI
description: Préparer vos machines virtuelles Azure pour les utiliser avec une instance de cluster de basculement (FCI) et SQL Server.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/02/2020
ms.author: mathoma
ms.openlocfilehash: 52d6bc97245423a4add392ab05634d21bcf83a0d
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358008"
---
# <a name="prepare-virtual-machines-for-an-fci-sql-server-on-azure-vms"></a>Préparer des machines virtuelles pour une instance FCI (SQL Server sur des machines virtuelles Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment préparer des machines virtuelles Azure pour les utiliser avec une instance de cluster de basculement SQL Server (FCI). Les paramètres de configuration varient en fonction de la solution de stockage FCI. Par conséquent, vérifiez que vous avez choisi la configuration adaptée à votre environnement et à votre entreprise. 

Pour plus d’informations, consultez une présentation de [l’instance FCI avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et [les meilleures pratiques de cluster](hadr-cluster-best-practices.md). 

## <a name="prerequisites"></a>Prérequis 

- Un abonnement Microsoft Azure. Démarrer [gratuitement](https://azure.microsoft.com/free/). 
- Un domaine Windows sur des machines virtuelles Azure ou un centre de données local étendu à Azure par jumelage de réseaux virtuels.
- Un compte qui dispose des autorisations nécessaires pour créer des objets sur les machines virtuelles Azure et dans Active Directory.
- Un réseau virtuel Azure et un sous-réseau avec suffisamment d’espace d’adressage IP pour ces composants :
   - Les deux machines virtuelles.
   - Les adresses IP du cluster de basculement Windows
   - Une adresse IP pour chaque instance de cluster de basculement
- Un DNS configuré sur le réseau Azure, pointant vers les contrôleurs de domaine.

## <a name="choose-an-fci-storage-option"></a>Choisir une option de stockage FCI

Les paramètres de configuration de votre machine virtuelle varient en fonction de l’option de stockage que vous prévoyez d’utiliser pour votre instance de cluster de basculement SQL Server. Avant de préparer la machine virtuelle, passez en revue les [options de stockage FCI disponibles](failover-cluster-instance-overview.md#storage) et choisissez l’option la mieux adaptée aux besoins de votre environnement et de votre entreprise. Sélectionnez ensuite avec soin les options de configuration de machine virtuelle appropriées dans cet article en fonction du stockage sélectionné. 

## <a name="configure-vm-availability"></a>Configurer la disponibilité des machines virtuelles 

La fonctionnalité de cluster de basculement nécessite que les machines virtuelles soient placées dans un [groupe à haute disponibilité](../../../virtual-machines/linux/tutorial-availability-sets.md) ou dans une [zone de disponibilité](../../../availability-zones/az-overview.md#availability-zones). Si vous choisissez des groupes à haute disponibilité, vous pouvez utiliser des [groupes de placement de proximité](../../../virtual-machines/windows/co-location.md#proximity-placement-groups) pour localiser les machines virtuelles plus près. En fait, les groupes de placement de proximité sont une condition préalable à l’utilisation des disques partagés Azure. 

Sélectionnez avec soin l’option de disponibilité de la machine virtuelle qui correspond à la configuration souhaitée pour votre cluster : 

- **Disques partagés Azure** : l’option de disponibilité varie selon que vous utilisez des disques SSD Premium ou un disque Ultra :
   - Disque SSD Premium : [groupe à haute disponibilité](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set) dans différents domaines d’erreur/de mise à jour pour des disques SSD Premium placés à l’intérieur d’un [groupe de placement de proximité](../../../virtual-machines/windows/proximity-placement-groups-portal.md).
   - Disque Ultra : [zone de disponibilité](../../../virtual-machines/windows/create-portal-availability-zone.md#confirm-zone-for-managed-disk-and-ip-address) mais les machines virtuelles doivent être placées dans la même zone de disponibilité, ce qui réduit la disponibilité du cluster à 99,9 %. 
- **Partages de fichiers Premium** : [Groupe à haute disponibilité](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set) ou [zone de disponibilité](../../../virtual-machines/windows/create-portal-availability-zone.md#confirm-zone-for-managed-disk-and-ip-address).
- **Espaces de stockage direct** : [Groupe à haute disponibilité](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set).

> [!IMPORTANT]
> Vous ne pouvez pas définir ou modifier le groupe à haute disponibilité après avoir créé une machine virtuelle.

## <a name="create-the-virtual-machines"></a>Créer les machines virtuelles

Une fois que vous avez configuré la disponibilité de votre machine virtuelle, vous êtes prêt à créer vos machines virtuelles. Vous pouvez choisir d’utiliser une image de la place de marché Azure sur laquelle SQL Server est déjà installé ou non. Toutefois, si vous choisissez une image pour SQL Server sur des machines virtuelles Azure, vous devez désinstaller SQL Server à partir de la machine virtuelle avant de configurer l’instance de cluster de basculement. 

### <a name="considerations"></a>Considérations

Sur un cluster de basculement invité de machine virtuelle Azure, nous recommandons une seule carte réseau par serveur (nœud de cluster) et un seul sous-réseau. Les réseaux Azure intègrent une redondance physique, ce qui rend inutiles les cartes réseau et les sous-réseaux supplémentaires sur un cluster invité de machine virtuelle IaaS Azure. Même si le rapport de validation de cluster émet un avertissement stipulant que les nœuds sont uniquement accessibles sur un seul réseau, vous pouvez ignorer ce dernier en toute sécurité sur les clusters de basculement invités de machine virtuelle IaaS Azure.

Placez les deux machines virtuelles :

- Dans le même groupe de ressources Azure que celui où se trouve le groupe à haute disponibilité si vous utilisez des groupes à haute disponibilité.
- Sur le même réseau virtuel que votre contrôleur de domaine.
- Sur un sous-réseau avec un espace d’adressage IP suffisant pour les deux machines virtuelles et toutes les instances de cluster de basculement que vous pourriez utiliser sur le cluster.
- Dans le groupe à haute disponibilité ou la zone de disponibilité Azure.

Vous pouvez créer une machine virtuelle Azure à l’aide d’une image [avec](sql-vm-create-portal-quickstart.md) ou [sans que ](../../../virtual-machines/windows/quick-create-portal.md) SQL Server soit préinstallé. Si vous choisissez l’image SQL Server, vous devez désinstaller manuellement l’instance SQL Server avant d’installer l’instance de cluster de basculement. 


## <a name="uninstall-sql-server"></a>Désinstaller SQL Server

Dans le cadre du processus de création de l’instance FCI, vous allez installer SQL Server en tant qu’instance en cluster dans le cluster de basculement. *Si vous avez déployé une machine virtuelle avec une image de la place de marché Azure sans SQL Server, vous pouvez sauter cette étape.* Si vous avez déployé une image avec SQL Server préinstallé, vous devez désinscrire la machine virtuelle SQL Server de l’extension SQL IaaS Agent, puis désinstaller SQL Server. 

### <a name="unregister-from-the-sql-iaas-agent-extension"></a>Désinscrire de l’extension SQL IaaS Agent

Les images de machine virtuelle SQL Server qui proviennent de Place de marché Azure sont automatiquement inscrites auprès de l’extension SQL IaaS Agent. Avant de désinstaller l’instance SQL préinstallée, vous devez d’abord [désinscrire chaque machine virtuelle SQL Server de l’extension SQL IaaS Agent](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension). 

### <a name="uninstall-sql-server"></a>Désinstaller SQL Server

Une fois que vous avez désinscrit SQL Server de l’extension, vous pouvez le désinstaller. Procédez comme suit sur chaque machine virtuelle : 

1. Connectez-vous à la machine virtuelle à l’aide du protocole RDP (Remote Desktop Protocol).

   Lors de votre première connexion à une machine virtuelle avec le protocole RDP, une invite vous demande si vous souhaitez autoriser que cet ordinateur soit détecté sur le réseau. Sélectionnez **Oui**.

1. Si vous utilisez l’une des images de machine virtuelle basée sur SQL Server, supprimez l’instance SQL Server :

   1. Dans **Programmes et fonctionnalités**, cliquez avec le bouton droit sur **Microsoft SQL Server 201_ (64 bits)** et sélectionnez **Désinstaller/Modifier**.
   1. Sélectionnez **Supprimer**.
   1. Sélectionnez l’instance par défaut.
   1. Supprimer toutes les fonctionnalités sous **Services Moteur de base de données**. Ne supprimez rien sous **Fonctionnalités partagées**. Vous devez voir quelque chose de similaire à la capture d’écran suivante :

      ![Sélectionner les caractéristiques](./media/failover-cluster-instance-prepare-vm/03-remove-features.png)

   1. Sélectionnez **Suivant**, puis **Supprimer**.
   1. Une fois l’instance supprimée, redémarrez la machine virtuelle. 

## <a name="open-the-firewall"></a>Ouvrir le pare-feu 

Sur chaque machine virtuelle, ouvrez le port TCP du pare-feu Windows utilisé par SQL Server. Par défaut, il s'agit du port 1433. Toutefois, vous pouvez modifier le port SQL Server sur un déploiement de machine virtuelle Azure. Pour ce faire, ouvrez le port que SQL Server utilise dans votre environnement. Ce port s’ouvre automatiquement sur les images SQL Server déployées à partir de la place de marché Azure. 

Si vous utilisez un [équilibreur de charge](failover-cluster-instance-vnn-azure-load-balancer-configure.md), vous devez également ouvrir le port utilisé par la sonde d’intégrité. Par défaut, ce port est le port 59999. Mais il peut s’agir de n’importe quel port TCP que vous spécifiez lorsque vous créez l’équilibreur de charge. 

Ce tableau détaille les ports que vous devrez peut-être ouvrir, en fonction de la configuration de votre FCI : 

   | Objectif | Port | Notes
   | ------ | ------ | ------
   | SQL Server | TCP 1433 | Port normal pour les instances par défaut de SQL Server. Si vous avez utilisé une image de la galerie, ce port s’ouvre automatiquement. </br> </br> **Utilisé par** : toutes les configurations de l’instance FCI. |
   | Sonde d’intégrité | TCP 59999 | Tout port TCP ouvert. Configurez la [sonde d’intégrité](failover-cluster-instance-vnn-azure-load-balancer-configure.md#configure-health-probe) de l’équilibreur de charge et le cluster pour qu’ils utilisent ce port. </br> </br> **Utilisé par** : Instance FCI avec l’équilibreur de charge. |
   | Partage de fichiers | UDP 445 | Port utilisé par le service de partage de fichiers. </br> </br> **Utilisé par** : Instance FCI avec partage de fichiers Premium. |

## <a name="join-the-domain"></a>Joindre le domaine

Vous devez également joindre vos machines virtuelles au domaine. Pour ce faire, vous pouvez utiliser un [modèle de démarrage rapide](../../../active-directory-domain-services/join-windows-vm-template.md#join-an-existing-windows-server-vm-to-a-managed-domain). 

## <a name="review-storage-configuration"></a>Passer en revue la configuration de stockage

Les machines virtuelles créées à partir de la place de marché Azure sont dotées d’un stockage attaché. Si vous envisagez de configurer votre stockage FCI à l’aide de partages de fichiers Premium ou de disques partagés Azure, vous pouvez supprimer le stockage attaché pour faire des économies, car le stockage local n’est pas utilisé pour l’instance de cluster de basculement. Toutefois, il est possible d’utiliser le stockage attaché pour les solutions FCI avec espaces de stockage direct. Par conséquent, sa suppression peut ne pas être utile dans ce cas. Passez en revue votre solution de stockage FCI pour déterminer si la suppression du stockage attaché est optimale pour faire des économies. 


## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez préparé votre environnement de machine virtuelle, vous êtes prêt à configurer votre instance de cluster de basculement. 

Choisissez l’un des guides suivants pour configurer l’environnement FCI adapté à votre entreprise : 
- [Configurer FCI avec des disques partagés Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md)
- [Configurer FCI avec un partage de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md)
- [Configurer FCI avec des espaces de stockage direct](failover-cluster-instance-storage-spaces-direct-manually-configure.md)

Pour plus d’informations, consultez une présentation de [FCI avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et [Configurations HADR prise en charge](hadr-cluster-best-practices.md). 

Si vous souhaitez en savoir plus, veuillez consulter : 
- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instances de cluster de basculement SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
