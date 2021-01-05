---
title: Créer un ICF avec partage de fichiers premium
description: Cet article explique comment créer une instance de cluster de basculement(ICF) SQL Server sur des machines virtuelles Azure.
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
ms.openlocfilehash: 2fb9677f0874de1fb715082d58a0e354880e654b
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97358069"
---
# <a name="create-an-fci-with-a-premium-file-share-sql-server-on-azure-vms"></a>Créer un ICF avec un partage de fichiers premium (SQL Server sur les machines virtuelles Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment créer une instance de cluster de basculement (ICF) avec SQL Server sur des machines virtuelles (VM) Azure à l’aide d’un [partage de fichiers premium](../../../storage/files/storage-how-to-create-premium-fileshare.md).

Les partages de fichiers Premium sont des partages de fichiers à faible latence s’appuyant sur des espaces de stockage direct, qui sont entièrement pris en charge pour une utilisation avec une instance de cluster de basculement pour SQL Server 2012 et ultérieur sur Windows Server 2012 et ultérieur. Les partages de fichiers Premium vous offrent une plus grande flexibilité, ce qui vous permet de redimensionner et de mettre à l’échelle un partage de fichiers sans temps d’arrêt.

Pour plus d’informations, consultez une présentation de [ICF avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et les [meilleures pratiques de cluster](hadr-cluster-best-practices.md). 

## <a name="prerequisites"></a>Prérequis

Avant de suivre les instructions décrites dans cet article, vous devez déjà disposer des éléments suivants :

- Un abonnement Azure.
- Un compte qui dispose des autorisations nécessaires pour créer des objets sur les machines virtuelles Azure et dans Active Directory.
- [Deux ou plusieurs machines virtuelles Windows Azure préparées](failover-cluster-instance-prepare-vm.md) dans un [groupe à haute disponibilité](../../../virtual-machines/windows/tutorial-availability-sets.md#create-an-availability-set) ou différentes [zones de disponibilité](../../../virtual-machines/windows/create-portal-availability-zone.md#confirm-zone-for-managed-disk-and-ip-address).
- Un [partage de fichiers Premium](../../../storage/files/storage-how-to-create-premium-fileshare.md) à utiliser comme lecteur en cluster, en fonction du quota de stockage de votre base de données pour vos fichiers de données.
- La dernière version de [PowerShell](/powershell/azure/install-az-ps). 

## <a name="mount-premium-file-share"></a>Monter le partage de fichiers Premium

1. Connectez-vous au [portail Azure](https://portal.azure.com). et accédez à votre compte de stockage.
1. Accédez à **Partages de fichiers** sous **Service de fichiers** et sélectionnez le partage de fichiers premium que vous souhaitez utiliser pour votre stockage SQL.
1. Sélectionnez **Connecter** pour afficher la chaîne de connexion de votre partage de fichiers.
1. Sélectionnez la lettre de lecteur que vous souhaitez utiliser dans la liste déroulante, puis copiez les deux blocs de code dans le Bloc-notes.

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/premium-file-storage-commands.png" alt-text="Copiez les deux commandes PowerShell à partir du portail de connexion au partage de fichiers":::

1. Utilisez le protocole bureau distant (RDP) pour établir une connexion à la machine virtuelle SQL Server à l’aide du compte que votre instance de cluster de basculement SQL Server utilisera pour le compte de service.
1. Ouvrez une console de commande PowerShell d’administration.
1. Exécutez les commandes que vous avez enregistrées précédemment lorsque vous travailliez dans le portail.
1. Accédez au partage avec l’Explorateur de fichiers ou la boîte de dialogue **Exécuter** (touche logo Windows + r). Utilisez le chemin d’accès réseau `\\storageaccountname.file.core.windows.net\filesharename`. Par exemple : `\\sqlvmstorageaccount.file.core.windows.net\sqlpremiumfileshare`

1. Créez au moins un dossier sur le partage de fichiers auquel vous venez de vous connecter afin d’y placer vos fichiers de données SQL.
1. Répétez ces étapes sur chaque machine virtuelle SQL Server qui fera partie du cluster.

  > [!IMPORTANT]
  > - Envisagez d’utiliser un partage de fichiers distinct pour les fichiers de sauvegarde pour préserver les opérations d’entrée/sortie par seconde (IOPS) et d’espace de ce partage pour les fichiers de données et les fichiers journaux. Vous pouvez utiliser un partage de fichiers Premium ou Standard pour les fichiers de sauvegarde.
  > - Si vous utilisez Windows 2012 R2 ou une version antérieure, suivez les mêmes étapes pour monter le partage de fichiers que vous allez utiliser comme témoin de partage de fichiers. 
  > 


## <a name="add-windows-cluster-feature"></a>Ajouter une fonctionnalité de cluster Windows

1. Connectez-vous à la première machine virtuelle avec RDP à l’aide d’un compte de domaine qui est membre du groupe Administrateurs locaux et qui dispose des autorisations pour créer des objets dans Active Directory. Utilisez ce compte pour le reste de la configuration.

1. [Ajoutez le clustering de basculement à chaque machine virtuelle](availability-group-manually-configure-prerequisites-tutorial.md#add-failover-clustering-features-to-both-sql-server-vms).

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

## <a name="validate-cluster"></a>Valider le cluster

Validez le cluster dans l’interface utilisateur ou avec PowerShell.

Pour valider le cluster à l’aide de l’interface utilisateur, procédez comme suit sur l’une des machines virtuelles :

1. Sous **Gestionnaire de serveur**, sélectionnez **Outils**, puis **Gestionnaire du cluster de basculement**.
1. Sous **Gestionnaire du cluster de basculement**, sélectionnez **Action**, puis **Valider la configuration**.
1. Sélectionnez **Suivant**.
1. Sous **Sélectionner des serveurs ou un cluster**, entrez le nom des deux machines virtuelles.
1. Sous **Options de test**, sélectionnez **Exécuter uniquement les tests que je sélectionne**. 
1. Sélectionnez **Suivant**.
1. Sous **Sélection des tests**, sélectionnez tous les tests à l’exception de **Stockage** et **Espaces de stockage direct**, comme illustré ici :

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/cluster-validation.png" alt-text="Sélectionner les tests de validation du cluster":::

1. Sélectionnez **Suivant**.
1. Sous **Confirmation**, sélectionnez **Suivant**.

L’assistant **Valider une configuration** exécute les tests de validation.

Pour valider le cluster avec PowerShell, exécutez le script suivant à partir d’une session PowerShell d’administrateur sur l’une des machines virtuelles :

   ```powershell
   Test-Cluster –Node ("<node1>","<node2>") –Include "Inventory", "Network", "System Configuration"
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

Configurez la solution de quorum qui répond le mieux aux besoins de votre entreprise. Vous pouvez configurer un [Témoin d’Azure Data Box Disk](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum), un [Témoin Cloud](/windows-server/failover-clustering/deploy-cloud-witness) ou un [Assistant de partage de fichiers](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum). Pour plus d’informations, consultez [Quorum avec les machines virtuelles SQL Server](hadr-cluster-best-practices.md#quorum). 


## <a name="test-cluster-failover"></a>Tester le basculement de cluster

Testez le basculement de votre cluster. Dans le **Gestionnaire du cluster de basculement**, cliquez avec le bouton droit sur votre cluster et sélectionnez **Autres actions** > **Déplacer une ressource de cluster principale** > **Sélectionner le nœud** et sélectionnez l’autre nœud du cluster. Déplacez la ressource de cluster principale vers chaque nœud du cluster, puis replacez-la sur le nœud principal. Si vous parvenez à déplacer le cluster vers chaque nœud, vous êtes prêt à installer SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Testez le basculement du cluster en déplaçant la ressource principale sur les autres nœuds":::


## <a name="create-sql-server-fci"></a>Créer l’instance de cluster de basculement SQL Server

Après avoir configuré le cluster de basculement, vous pouvez créer l’instance de cluster de basculement SQL Server.

1. Connectez-vous à la première machine virtuelle avec RDP.

1. Dans le **Gestionnaire du cluster de basculement**, vérifiez que toutes les ressources principales du cluster se trouvent sur la première machine virtuelle. Si nécessaire, déplacez toutes les ressources vers cette machine virtuelle.

1. Recherchez le support d’installation. Si la machine virtuelle utilise l’une des images Azure Marketplace, le support se situe sous `C:\SQLServer_<version number>_Full`. 

1. Sélectionnez **Configuration**.

1. Dans le **Centre d’installation SQL Server**, sélectionnez **Installation**.

1. Sélectionnez **Installation du nouveau cluster de basculement SQL Server**, puis suivez les instructions dans l’assistant pour installer l’ICF SQL Server.

   Les répertoires de données de l’instance de cluster de basculement doivent se trouver sur le partage de fichiers Premium. Entrez le chemin d’accès complet du partage, sous la forme suivante : `\\storageaccountname.file.core.windows.net\filesharename\foldername`. Un avertissement s’affiche, vous informant que vous avez spécifié un serveur de fichiers comme répertoire de données. Cet avertissement est attendu. Vérifiez que le compte d’utilisateur avec lequel vous avez accédé par RPD à la machine virtuelle lorsque vous avez conservé le partage de fichiers est celui que le service SQL Server utilise pour éviter d’éventuelles défaillances.

   :::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/use-file-share-as-data-directories.png" alt-text="Utilisez le partage de fichiers en tant que répertoires de données SQL":::

1. Une fois que vous avez terminé les étapes de l’assistant, le programme d’installation installe une instance de cluster de basculement SQL Server sur le premier nœud.

1. Une fois que le programme d’installation a installé l’instance de cluster de basculement sur le premier nœud, connectez-vous au second nœud avec RDP.

1. Dans le **Centre d’installation SQL Server**, sélectionnez **Installation**.

1. Sélectionnez **Ajouter un nœud à un cluster de basculement SQL Server**. Suivez les instructions de l’Assistant pour installer SQL Server et ajouter le serveur à l’instance de cluster de basculement.

   >[!NOTE]
   >Si vous avez utilisé une image de la galerie Azure Marketplace avec SQL Server, les outils SQL Server ont été inclus avec l’image. Si vous n’avez pas utilisé une de ces images, installez les outils de SQL Server séparément. Pour plus d’informations, consultez la page [Télécharger SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).

1. Répétez ces étapes sur les autres nœuds que vous voulez ajouter à l’instance du cluster de basculement du SQL Server. 

## <a name="register-with-the-sql-vm-rp"></a>S’inscrire auprès de SQL VM RP

Pour gérer votre machine virtuelle SQL Server à partir du portail, inscrivez-la auprès de l’extension SQL IaaS Agent dans le [mode d’administration léger](sql-agent-extension-manually-register-single-vm.md#lightweight-management-mode), actuellement le seul mode pris en charge avec FCI et SQL Server sur les machines virtuelles Azure. 

Inscrire la machine virtuelle SQL Server en mode léger avec PowerShell (le type de licence peut être `PAYG` ou `AHUB`) :

```powershell-interactive
# Get the existing compute VM
$vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
         
# Register SQL VM with 'Lightweight' SQL IaaS agent
New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
   -LicenseType ???? -SqlManagementType LightWeight  
```

## <a name="configure-connectivity"></a>Configurer la connectivité 

Pour acheminer le trafic de manière appropriée vers le nœud principal actuel, configurez l’option de connectivité adaptée à votre environnement. Vous pouvez créer un [équilibreur de charge Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) ou, si vous utilisez SQL Server 2019 CU2+ (ou version ultérieure) et Windows Server 2016 (ou version ultérieure), vous pouvez utiliser la fonctionnalité de [nom de réseau distribué](failover-cluster-instance-distributed-network-name-dnn-configure.md). 

## <a name="limitations"></a>Limites

- Le MSDTC (Microsoft Distributed Transaction Coordinator) n’est pas pris en charge sur Windows Server 2016 et antérieur. 
- Filestream n’est pas pris en charge pour un cluster de basculement avec un partage de fichiers Premium. Pour utiliser le flux de fichier, déployez votre cluster en utilisant des [Espaces de stockage direct](failover-cluster-instance-storage-spaces-direct-manually-configure.md) ou les [Disques partagés Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) à la place.
- Seule l’inscription auprès de l’extension SQL IaaS Agent en [mode d’administration léger](sql-server-iaas-agent-extension-automate-management.md#management-modes) est prise en charge. 

## <a name="next-steps"></a>Étapes suivantes

Si vous ne l’avez pas déjà fait, configurez la connectivité à votre ICF avec un nom de réseau virtuel [et un équilibrage de charge Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) ou [le nom de réseau distribué (DNN)](failover-cluster-instance-distributed-network-name-dnn-configure.md). 


Si les partages de fichiers Premium ne sont pas la solution de stockage ICF appropriée, envisagez de créer votre ICF à l’aide des [Disques partagés Azure](failover-cluster-instance-azure-shared-disks-manually-configure.md) ou [Espaces de stockage direct](failover-cluster-instance-storage-spaces-direct-manually-configure.md) à la place. 

Pour plus d’informations, consultez une présentation de [ICF avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et [meilleures pratiques de configuration de cluster](hadr-cluster-best-practices.md). 

Pour plus d'informations, consultez les pages suivantes : 
- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instances de cluster de basculement SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)
