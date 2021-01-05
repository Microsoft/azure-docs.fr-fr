---
title: Créer une FCI avec Espaces de stockage direct
description: Utiliser Espaces de stockage direct pour créer une instance de cluster de basculement (FCI) avec SQL Server sur des machines virtuelles Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.custom: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/18/2020
ms.author: mathoma
ms.openlocfilehash: 6ed5e11a8492314e99b9f105d259fa910dcdb77d
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97357804"
---
# <a name="create-an-fci-with-storage-spaces-direct-sql-server-on-azure-vms"></a>Créer une FCI avec Espaces de stockage direct (SQL Server sur machines virtuelles Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment créer une instance de cluster de basculement (FCI) à l’aide d’[Espaces de stockage direct](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) avec SQL Server sur des machines virtuelles Azure. Espaces de stockage direct agit comme un réseau de zone de stockage virtuel (VSAN) basé sur logiciel qui synchronise le stockage (disques de données) entre les nœuds (machines virtuelles Azure) dans un cluster Windows. 

Pour plus d’informations, consultez une présentation de [ICF avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et les [meilleures pratiques de cluster](hadr-cluster-best-practices.md). 


## <a name="overview"></a>Vue d’ensemble 

La technologie [Espaces de stockage direct (S2D)](/windows-server/storage/storage-spaces/storage-spaces-direct-overview) prend en charge deux types d’architectures : convergée et hyperconvergée. Une infrastructure hyperconvergée place le stockage sur les mêmes serveurs que ceux qui hébergent l’application en cluster, de sorte que le stockage se trouve sur chaque nœud FCI SQL Server. 

Le diagramme suivant montre la solution complète, laquelle utilise des espaces de stockage direct hyperconvergés avec SQL Server sur des machines virtuelles Azure : 

![Diagramme de la solution complète, utilisant des espaces de stockage direct hyperconvergés](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/00-sql-fci-s2d-complete-solution.png)

Le diagramme précédent montre les ressources suivantes dans le même groupe de ressources :

- Deux machines virtuelles dans un cluster de basculement Windows Server. Lorsqu’une machine virtuelle se trouve dans un cluster de basculement, elle est également désignée comme *nœud* ou *nœud de cluster*.
- Chaque machine virtuelle possède au moins deux disques de données.
- La technologie Espaces de stockage direct synchronise les données sur le disque de données et présente le stockage synchronisé en tant que pool de stockage.
- Le pool de stockage présente un volume partagé de cluster (CSV) au cluster de basculement.
- Le rôle du cluster de l’instance de cluster de basculement SQL Server utilise le volume partagé de cluster pour les lecteurs de données.
- Un équilibrage de charge Azure pour contenir l’adresse IP de l’instance de cluster de basculement SQL Server.
- Un groupe à haute disponibilité Azure contient toutes les ressources.

   > [!NOTE]
   > Vous pouvez créer la solution complète dans Azure à partir d’un modèle. Un exemple de modèle est disponible sur la page [Modèles de démarrage rapide Azure (en anglais)](https://github.com/MSBrett/azure-quickstart-templates/tree/master/sql-server-2016-fci-existing-vnet-and-ad) de GitHub. Cet exemple n’est pas conçu ni testé pour une charge de travail spécifique. Vous pouvez exécuter le modèle pour créer une instance de cluster de basculement SQL Server avec le stockage Espaces de stockage direct connecté à votre domaine. Vous pouvez évaluer le modèle et le modifier selon vos besoins.


## <a name="prerequisites"></a>Prérequis

Avant de suivre les instructions décrites dans cet article, vous devez déjà disposer des éléments suivants :

- Un abonnement Azure. Démarrer [gratuitement](https://azure.microsoft.com/free/). 
- [Deux machines virtuelles Windows Azure préparées ou plus](failover-cluster-instance-prepare-vm.md) dans un [groupe à haute disponibilité](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set).
- Un compte qui dispose des autorisations nécessaires pour créer des objets sur les machines virtuelles Azure et dans Active Directory.
- La dernière version de [PowerShell](/powershell/azure/install-az-ps). 


## <a name="add-the-windows-cluster-feature"></a>Ajouter la fonctionnalité de cluster Windows

1. Connectez-vous à la première machine virtuelle à l’aide de Remote Desktop Protocol (RDP) avec un compte de domaine qui est l’un des administrateurs locaux et qui dispose des autorisations pour créer des objets dans Active Directory. Utilisez ce compte pour le reste de la configuration.

1. Ajoutez le clustering de basculement à chaque machine virtuelle.

   Pour installer le clustering de basculement à partir de l’interface utilisateur, exécutez les étapes suivantes sur les deux machines virtuelles :

   1. Dans le **Gestionnaire de serveur**, sélectionnez **Gérer**, puis **Ajouter des rôles et fonctionnalités**.
   1. Dans l’assistant **Ajouter des rôles et des fonctionnalités**, sélectionnez **Suivant** jusqu’à ce que vous atteigniez la page **Sélectionner les fonctionnalités**.
   1. Dans **Sélectionner les fonctionnalités**, sélectionnez **Clustering de basculement**. Incluez toutes les fonctionnalités et les outils de gestion requis. 
   1. Sélectionnez **Ajouter des fonctionnalités**.
   1. Sélectionnez **Suivant**, puis **Terminer** pour installer les fonctionnalités.

   Pour installer le clustering de basculement avec PowerShell, exécutez le script suivant à partir d’une session PowerShell d’administrateur sur l’une des machines virtuelles :

   ```powershell
   $nodes = ("<node1>","<node2>")
   Invoke-Command  $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools}
   ```

Pour plus d’informations sur les étapes suivantes, consultez les instructions de la section « Étape 3 : Configurer les espaces de stockage direct » de la rubrique [Solution hyperconvergée utilisant Espaces de stockage direct dans Windows Server 2016](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-3-configure-storage-spaces-direct).


## <a name="validate-the-cluster"></a>Valider le cluster

Validez le cluster dans l’interface utilisateur ou avec PowerShell.

Pour valider le cluster à l’aide de l’interface utilisateur, procédez comme suit sur l’une des machines virtuelles :

1. Sous **Gestionnaire de serveur**, sélectionnez **Outils**, puis **Gestionnaire du cluster de basculement**.
1. Sous **Gestionnaire du cluster de basculement**, sélectionnez **Action**, puis **Valider la configuration**.
1. Sélectionnez **Suivant**.
1. Sous **Sélectionner des serveurs ou un cluster**, entrez le nom des deux machines virtuelles.
1. Sous **Options de test**, sélectionnez **Exécuter uniquement les tests que je sélectionne**. 
1. Sélectionnez **Suivant**.
1. Sous **Sélection des tests**, sélectionnez tous les tests à l’exception de **Stockage**, comme illustré ici :

   ![Sélectionner les tests de validation du cluster](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/10-validate-cluster-test.png)

1. Sélectionnez **Suivant**.
1. Sous **Confirmation**, sélectionnez **Suivant**.

    L’assistant **Valider une configuration** exécute les tests de validation.

Pour valider le cluster avec PowerShell, exécutez le script suivant à partir d’une session PowerShell d’administrateur sur l’une des machines virtuelles :

   ```powershell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Storage Spaces Direct", "Inventory", "Network", "System Configuration"
   ```

Après avoir validé le cluster, créez le cluster de basculement.


## <a name="create-failover-cluster"></a>Créer un cluster de basculement

Pour créer le cluster de basculement, vous avez besoin des éléments suivants :

- Les noms des machines virtuelles qui deviennent les nœuds du cluster.
- Un nom pour le cluster de basculement.
- Une adresse IP pour le cluster de basculement. Vous pouvez spécifier une adresse IP qui n’est pas utilisée sur le même réseau virtuel et sous-réseau Azure que les nœuds du cluster.


# <a name="windows-server-2012---2016"></a>[Windows Server 2012 - 2016](#tab/windows2012)

Le script PowerShell suivant crée un cluster de basculement pour Windows Server 2012 à Windows Server 2016. Mettez à jour le script avec les noms des nœuds (les noms des machines virtuelles) et une adresse IP disponible à partir du réseau virtuel Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage
```   

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows2019)

Le script PowerShell suivant crée un cluster de basculement pour Windows Server 2019.  Mettez à jour le script avec les noms des nœuds (les noms des machines virtuelles) et une adresse IP disponible à partir du réseau virtuel Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage -ManagementPointNetworkType Singleton 
```

Pour plus d’informations, consultez [Cluster de basculement : Objet réseau en cluster](https://blogs.windows.com/windowsexperience/2018/08/14/announcing-windows-server-2019-insider-preview-build-17733/#W0YAxO8BfwBRbkzG.97).

---


## <a name="configure-quorum"></a>Configurer un quorum

Configurez la solution de quorum qui répond le mieux aux besoins de votre entreprise. Vous pouvez configurer un [Témoin de disque](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum), un [Témoin de cloud](/windows-server/failover-clustering/deploy-cloud-witness) ou un [Témoin de partage de fichiers](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum). Pour plus d’informations, consultez [Quorum avec les machines virtuelles SQL Server](hadr-cluster-best-practices.md#quorum). 

## <a name="add-storage"></a>Ajouter du stockage

Les disques Espaces de stockage direct doivent être vides. Ils ne peuvent pas contenir de partitions ou d’autres données. Pour nettoyer les disques, suivez les instructions de la rubrique [Déployer des espaces de stockage direct](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-31-clean-drives).

1. [Activer les espaces de stockage direct](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-35-enable-storage-spaces-direct).

   Le script PowerShell suivant active les Espaces de stockage direct :  

   ```powershell
   Enable-ClusterS2D
   ```

   Dans le **Gestionnaire du cluster de basculement**, vous pouvez maintenant voir le pool de stockage.

1. [Créez un volume](/windows-server/storage/storage-spaces/deploy-storage-spaces-direct#step-36-create-volumes).

   La technologie Espaces de stockage direct crée automatiquement un pool de stockage lorsque vous l’activez. Vous êtes maintenant prêt à créer un volume. La cmdlet PowerShell `New-Volume` automatise le processus de création du volume. Ce processus comprend la mise en forme, l’ajout du volume au cluster et la création d’un CSV. Cet exemple crée un volume partagé de cluster de 800 gigaoctets (Go) :

   ```powershell
   New-Volume -StoragePoolFriendlyName S2D* -FriendlyName VDisk01 -FileSystem CSVFS_REFS -Size 800GB
   ```   

   Une fois que vous avez exécuté la commande précédente, un volume de 800 Go est monté en tant que ressource de cluster. Le volume se trouve sous `C:\ClusterStorage\Volume1\`.

   Cette capture d’écran montre un CSV avec Espaces de stockage direct :

   ![Capture d’écran d’un volume partagé de cluster avec Espaces de stockage direct](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/15-cluster-shared-volume.png)



## <a name="test-cluster-failover"></a>Tester le basculement de cluster

Testez le basculement de votre cluster. Dans le **Gestionnaire du cluster de basculement**, cliquez avec le bouton droit sur votre cluster et sélectionnez **Autres actions** > **Déplacer une ressource de cluster principale** > **Sélectionner le nœud** et sélectionnez l’autre nœud du cluster. Déplacez la ressource de cluster principale vers chaque nœud du cluster, puis replacez-la sur le nœud principal. Si vous parvenez à déplacer le cluster vers chaque nœud, vous êtes prêt à installer SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Testez le basculement du cluster en déplaçant la ressource principale sur les autres nœuds":::

## <a name="create-sql-server-fci"></a>Créer l’instance de cluster de basculement SQL Server

Après avoir configuré le cluster de basculement et tous les composants du cluster, notamment le stockage, vous pouvez créer l’instance de cluster de basculement SQL Server.

1. Connectez-vous à la première machine virtuelle avec RDP.

1. Dans **Gestionnaire du cluster de basculement**, vérifiez que toutes les ressource de cluster principales se trouvent sur la première machine virtuelle. Si nécessaire, déplacez toutes les ressources vers cette machine virtuelle.

1. Recherchez le support d’installation. Si la machine virtuelle utilise l’une des images Azure Marketplace, le support se situe sous `C:\SQLServer_<version number>_Full`. Sélectionnez **Configuration**.

1. Dans le **Centre d’installation SQL Server**, sélectionnez **Installation**.

1. Sélectionnez **Installation d’un nouveau cluster de basculement SQL Server**. Suivez les instructions de l’Assistant pour installer l’instance de cluster de basculement SQL Server.

   Les répertoires de données de l’instance de cluster de basculement doivent se trouver sur le stockage en cluster. Avec Espaces de stockage direct, il ne s’agit pas d’un disque partagé, mais d’un point de montage vers un volume sur chaque serveur. La technologie Espaces de stockage direct synchronise le volume entre les deux nœuds. Le volume est présenté au cluster en tant que CSV. Utilisez le point de montage du volume partagé de cluster pour les répertoires de données.

   ![Répertoires de données](./media/failover-cluster-instance-storage-spaces-direct-manually-configure/20-data-dicrectories.png)

1. Une fois que l’Assistant a terminé, le programme d’installation installe une FCI SQL Server sur le premier nœud.

1. Une fois que le programme d’installation a installé l’instance de cluster de basculement sur le premier nœud, connectez-vous au second nœud avec RDP.

1. Ouvrez le **Centre d’installation SQL Server**. Sélectionnez **Installation**.

1. Sélectionnez **Ajouter un nœud à un cluster de basculement SQL Server**. Suivez les instructions de l’Assistant pour installer SQL Server et ajouter le serveur à l’instance de cluster de basculement.

   >[!NOTE]
   >Si vous avez utilisé une image de la galerie de la Place de marché Azure avec SQL Server, les outils SQL Server ont été inclus avec l’image. Si vous n’avez pas utilisé une de ces images, installez les outils de SQL Server séparément. Pour plus d’informations, consultez la page [Télécharger SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).
   >


## <a name="register-with-the-sql-vm-rp"></a>S’inscrire auprès de SQL VM RP

Pour gérer votre machine virtuelle SQL Server à partir du portail, inscrivez-la auprès de l’extension SQL IaaS Agent dans le [mode d’administration léger](sql-agent-extension-manually-register-single-vm.md#lightweight-management-mode), actuellement le seul mode pris en charge avec FCI et SQL Server sur les machines virtuelles Azure. 


Inscrire une machine virtuelle SQL Server en mode léger avec PowerShell :  

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
         
# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
   -LicenseType PAYG -SqlManagementType LightWeight  
```

## <a name="configure-connectivity"></a>Configurer la connectivité 

Pour acheminer le trafic de manière appropriée vers le nœud principal actuel, configurez l’option de connectivité adaptée à votre environnement. Vous pouvez créer un [équilibreur de charge Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) ou, si vous utilisez SQL Server 2019 CU2+ (ou version ultérieure) et Windows Server 2016 (ou version ultérieure), vous pouvez utiliser la fonctionnalité de [nom de réseau distribué](failover-cluster-instance-distributed-network-name-dnn-configure.md). 

## <a name="limitations"></a>Limites

- Les machines virtuelles Azure prennent en charge Microsoft Distributed Transaction Coordinator (MSDTC) sur Windows Server 2019 avec un stockage sur des CSV et un [équilibreur de charge standard](../../../load-balancer/load-balancer-overview.md).
- Les disques qui ont été joints en tant que disques au format NTFS peuvent être utilisés avec Espaces de stockage direct uniquement si l’option d’éligibilité du disque est décochée ou désactivée lorsque le stockage est ajouté au cluster. 
- Seule l’inscription auprès de l’extension SQL IaaS Agent en [mode d’administration léger](sql-server-iaas-agent-extension-automate-management.md#management-modes) est prise en charge.

## <a name="next-steps"></a>Étapes suivantes

Si vous ne l’avez pas déjà fait, configurez la connectivité à votre instance FCI avec un [nom de réseau virtuel et un équilibrage de charge Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) ou [le nom de réseau distribué (DNN)](failover-cluster-instance-distributed-network-name-dnn-configure.md). 

Si Espaces de stockage direct n’est pas la solution de stockage FCI appropriée, envisagez de créer votre FCI à l’aide de [disques partagés Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) ou de [partages de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md) à la place. 

Pour plus d’informations, consultez une présentation de [Instance FCI avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et [Meilleures pratiques de configuration de cluster](hadr-cluster-best-practices.md). 

Pour plus d'informations, consultez les pages suivantes : 
- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instances de cluster de basculement SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)