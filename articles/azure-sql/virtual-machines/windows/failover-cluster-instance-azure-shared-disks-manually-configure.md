---
title: Créer une instance FCI avec des disques partagés Azure
description: Utiliser des disques partagés Azure pour créer une instance de cluster de basculement(FCI) avec SQL Server sur des machines virtuelles Azure.
services: virtual-machines
documentationCenter: na
author: MashaMSFT
editor: monicar
tags: azure-service-management
ms.service: virtual-machines-sql
ms.custom: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/26/2020
ms.author: mathoma
ms.openlocfilehash: becf9f8c7f6a967ed63cfd3040de90de76e32fff
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607267"
---
# <a name="create-an-fci-with-azure-shared-disks-sql-server-on-azure-vms"></a>Créer une instance FCI avec des disques partagés Azure (SQL Server sur les machines virtuelles Azure)
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Cet article explique comment créer une instance de cluster de basculement (FCI) à l’aide de disques partagés Azure avec SQL Server sur des machines virtuelles Azure. 

Pour plus d’informations, consultez une présentation de [l’instance FCI avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et [les meilleures pratiques de cluster](hadr-cluster-best-practices.md). 

## <a name="prerequisites"></a>Prérequis 

Avant de suivre les instructions décrites dans cet article, vous devez déjà disposer des éléments suivants :

- Un abonnement Azure. Démarrer [gratuitement](https://azure.microsoft.com/free/). 
- [Au moins deux machines virtuelles Windows Azure](failover-cluster-instance-prepare-vm.md). Les [groupes à haute disponibilité](../../../virtual-machines/windows/tutorial-availability-sets.md) et les [groupes de placement de proximité](../../../virtual-machines/windows/co-location.md#proximity-placement-groups) (PPG) sont pris en charge pour SSD Premium et les [zones de disponibilité](../../../virtual-machines/windows/create-portal-availability-zone.md#confirm-zone-for-managed-disk-and-ip-address) sont prises en charge pour Disques Ultra. Si vous utilisez un groupe PPG, tous les nœuds doivent exister dans le même groupe.
- Un compte qui dispose des autorisations nécessaires pour créer des objets sur les machines virtuelles Azure et dans Active Directory.
- La dernière version de [PowerShell](/powershell/azure/install-az-ps). 

## <a name="add-azure-shared-disk"></a>Ajouter un disque partagé Azure
Déployez un disque SSD Premium managé avec la fonctionnalité de disque partagé activée. Définissez `maxShares` sur **aligner sur le nombre de nœuds de cluster** pour rendre le disque partageable sur tous les nœuds d’instance de cluster de basculement. 

Ajoutez un disque partagé Azure en procédant comme suit : 

1. Enregistrez le script suivant en tant que *SharedDiskConfig.json* : 

   ```JSON
   { 
     "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
       "dataDiskName": {
         "type": "string",
         "defaultValue": "mySharedDisk"
       },
       "dataDiskSizeGB": {
         "type": "int",
         "defaultValue": 1024
       },
       "maxShares": {
         "type": "int",
         "defaultValue": 2
       }
     },
     "resources": [
       {
         "type": "Microsoft.Compute/disks",
         "name": "[parameters('dataDiskName')]",
         "location": "[resourceGroup().location]",
         "apiVersion": "2019-07-01",
         "sku": {
           "name": "Premium_LRS"
         },
         "properties": {
           "creationData": {
             "createOption": "Empty"
           },
           "diskSizeGB": "[parameters('dataDiskSizeGB')]",
           "maxShares": "[parameters('maxShares')]"
         }
       }
     ] 
   }
   ```

2. Exécutez *SharedDiskConfig.json* à l’aide de PowerShell : 

   ```powershell
   $rgName = < specify your resource group name>
       $location = 'westcentralus'
       New-AzResourceGroupDeployment -ResourceGroupName $rgName `
   -TemplateFile "SharedDiskConfig.json"
   ```

3. Pour chaque machine virtuelle, initialisez les disques partagés attachés en tant que table de partition GUID (GPT) et formatez-les en tant que NTFS (New Technology File System) en exécutant la commande suivante : 

    ```powershell
    $resourceGroup = "<your resource group name>"
    $location = "<region of your shared disk>"
    $ppgName = "<your proximity placement groups name>"
    $vm = Get-AzVM -ResourceGroupName "<your resource group name>" `
        -Name "<your VM node name>"
    $dataDisk = Get-AzDisk -ResourceGroupName $resourceGroup `
        -DiskName "<your shared disk name>"
    $vm = Add-AzVMDataDisk -VM $vm -Name "<your shared disk name>" `
        -CreateOption Attach -ManagedDiskId $dataDisk.Id `
        -Lun <available LUN - check disk setting of the VM>
    Update-AzVm -VM $vm -ResourceGroupName $resourceGroup
    ```

## <a name="create-failover-cluster"></a>Créer un cluster de basculement

Pour créer le cluster de basculement, vous avez besoin des éléments suivants :

- Les noms des machines virtuelles qui deviennent les nœuds du cluster.
- Un nom pour le cluster de basculement.
- Une adresse IP pour le cluster de basculement. Vous pouvez spécifier une adresse IP qui n’est pas utilisée sur le même réseau virtuel et sous-réseau Azure que les nœuds du cluster.

# <a name="windows-server-2012-2016"></a>[Windows Server 2012-2016](#tab/windows2012)

Le script PowerShell suivant crée un cluster de basculement. Mettez à jour le script avec les noms des nœuds (les noms des machines virtuelles) et une adresse IP disponible à partir du réseau virtuel Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage
```   

# <a name="windows-server-2019"></a>[Windows Server 2019](#tab/windows2019)

Le script PowerShell suivant crée un cluster de basculement. Mettez à jour le script avec les noms des nœuds (les noms des machines virtuelles) et une adresse IP disponible à partir du réseau virtuel Azure.

```powershell
New-Cluster -Name <FailoverCluster-Name> -Node ("<node1>","<node2>") –StaticAddress <n.n.n.n> -NoStorage -ManagementPointNetworkType Singleton 
```

Pour plus d’informations, consultez [Cluster de basculement : Objet réseau en cluster](https://blogs.windows.com/windowsexperience/2018/08/14/announcing-windows-server-2019-insider-preview-build-17733/#W0YAxO8BfwBRbkzG.97).

---

## <a name="configure-quorum"></a>Configurer un quorum

Configurez la solution de quorum qui répond le mieux aux besoins de votre entreprise. Vous pouvez configurer un [Témoin de disque](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum), un [Témoin de cloud](/windows-server/failover-clustering/deploy-cloud-witness) ou un [Témoin de partage de fichiers](/windows-server/failover-clustering/manage-cluster-quorum#configure-the-cluster-quorum). Pour plus d’informations, consultez [Quorum avec les machines virtuelles SQL Server](hadr-cluster-best-practices.md#quorum). 

## <a name="validate-cluster"></a>Valider le cluster
Validez le cluster dans l’interface utilisateur ou avec PowerShell.

Pour valider le cluster à l’aide de l’interface utilisateur, procédez comme suit sur l’une des machines virtuelles :

1. Sous **Gestionnaire de serveur**, sélectionnez **Outils**, puis **Gestionnaire du cluster de basculement**.
1. Sous **Gestionnaire du cluster de basculement**, sélectionnez **Action**, puis **Valider la configuration**.
1. Sélectionnez **Suivant**.
1. Sous **Sélectionner des serveurs ou un cluster**, entrez le nom des deux machines virtuelles.
1. Sous **Options de test**, sélectionnez **Exécuter uniquement les tests que je sélectionne**. 
1. Sélectionnez **Suivant**.
1. Sous **Sélection des tests**, sélectionnez tous les tests *à l’exception* **Stockage**

## <a name="test-cluster-failover"></a>Tester le basculement de cluster

Testez le basculement de votre cluster. Dans le **Gestionnaire du cluster de basculement**, cliquez avec le bouton droit sur votre cluster et sélectionnez **Autres actions** > **Déplacer une ressource de cluster principale** > **Sélectionner le nœud** et sélectionnez l’autre nœud du cluster. Déplacez la ressource de cluster principale vers chaque nœud du cluster, puis replacez-la sur le nœud principal. Si vous parvenez à déplacer le cluster vers chaque nœud, vous êtes prêt à installer SQL Server.  

:::image type="content" source="media/failover-cluster-instance-premium-file-share-manually-configure/test-cluster-failover.png" alt-text="Testez le basculement du cluster en déplaçant la ressource principale sur les autres nœuds":::

## <a name="create-sql-server-fci"></a>Créer l’instance de cluster de basculement SQL Server

Après avoir configuré le cluster de basculement et tous les composants du cluster, notamment le stockage, vous pouvez créer l’instance de cluster de basculement SQL Server.

1. Se connecte au premier ordinateur virtuel à l’aide du protocole RDP (Remote Desktop Protocol).

1. Dans le **Gestionnaire du cluster de basculement**, vérifiez que toutes les ressources principales du cluster se trouvent sur la première machine virtuelle. Si nécessaire, déplacez toutes les ressources vers cette machine virtuelle.

1. Recherchez le support d’installation. Si la machine virtuelle utilise l’une des images Azure Marketplace, le support se situe sous `C:\SQLServer_<version number>_Full`. 

1. Sélectionnez **Configuration**.

1. Dans le **Centre d’installation SQL Server**, sélectionnez **Installation**.

1. Sélectionnez **Installation d’un nouveau cluster de basculement SQL Server**. Suivez les instructions de l’Assistant pour installer l’instance de cluster de basculement SQL Server.

Les répertoires de données de l’instance de cluster de basculement doivent se trouver sur les disques partagés Azure. 

1. Une fois que vous avez terminé les instructions de l’assistant, le programme d’installation installe une instance de cluster de basculement SQL Server sur le premier nœud.

1. Une fois que le programme d’installation a installé l’instance de cluster de basculement sur le premier nœud, connectez-vous au second nœud avec RDP.

1. Dans le **Centre d’installation SQL Server**, sélectionnez **Installation**.

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

- Seule l’inscription auprès de l’extension SQL IaaS Agent en [mode d’administration léger](sql-server-iaas-agent-extension-automate-management.md#management-modes) est prise en charge.

## <a name="next-steps"></a>Étapes suivantes

Si vous ne l’avez pas déjà fait, configurez la connectivité à votre instance FCI avec un [nom de réseau virtuel et un équilibrage de charge Azure](failover-cluster-instance-vnn-azure-load-balancer-configure.md) ou [le nom de réseau distribué (DNN)](failover-cluster-instance-distributed-network-name-dnn-configure.md). 

Si les disques partagés Azure ne sont pas la solution de stockage FCI appropriée pour vous, envisagez de créer votre instance FCI à l’aide de [partages de fichiers Premium](failover-cluster-instance-premium-file-share-manually-configure.md) ou d’[espaces de stockage direct](failover-cluster-instance-storage-spaces-direct-manually-configure.md) à la place. 

Pour plus d’informations, consultez une présentation de [Instance FCI avec SQL Server sur les machines virtuelles Azure](failover-cluster-instance-overview.md) et [Meilleures pratiques de configuration de cluster](hadr-cluster-best-practices.md).

Pour plus d'informations, consultez les pages suivantes : 
- [Technologies de cluster Windows](/windows-server/failover-clustering/failover-clustering-overview)   
- [Instances de cluster de basculement SQL Server](/sql/sql-server/failover-clusters/windows/always-on-failover-cluster-instances-sql-server)