---
title: Haute disponibilité multi-SID de SAP ASCS/SCS avec WSFC et disque partagé Azure | Microsoft Docs
description: Haute disponibilité multi-SID pour une instance SAP ASCS/SCS avec WSFC et disque partagé Azure
services: virtual-machines-windows,virtual-network,storage
documentationcenter: saponazure
author: rdeltcheva
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: cbf18abe-41cb-44f7-bdec-966f32c89325
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 08/12/2020
ms.author: radeltch
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4dfbffcaedb6c544a34e347633d5adc173fab33e
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97655983"
---
# <a name="sap-ascsscs-instance-multi-sid-high-availability-with-windows-server-failover-clustering-and-azure-shared-disk"></a>Haute disponibilité multi-SID pour une instance SAP ASCS/SCS avec clustering de basculement Windows Server et disque partagé Azure

> ![Système d’exploitation Windows][Logo_Windows] Windows
>

Cet article explique comment passer d'une installation ASCS/SCS unique à une configuration SAP multi-SID en installant des instances SAP ASCS/SCS en cluster supplémentaires dans un cluster de basculement Windows Server (WSFC) existant avec disque partagé Azure. Une fois ce processus terminé, vous aurez configuré un cluster multi-SID SAP.

## <a name="prerequisites-and-limitations"></a>Conditions préalables et limitations

Vous pouvez actuellement utiliser des disques SSD Premium Azure en tant que disque partagé Azure pour l'instance SAP ASCS/SCS. Les limitations suivantes s'appliquent :

-  Le [disque Ultra Azure](../../disks-types.md#ultra-disk) n'est pas pris en charge en tant que disque partagé Azure pour les charges de travail SAP. Il est actuellement impossible de placer des machines virtuelles Azure dans un groupe à haute disponibilité à l'aide d'un disque Ultra Azure.
-  Le [disque partagé Azure](../../disks-shared.md) avec disques SSD Premium n'est pris en charge qu'avec les machines virtuelles d'un groupe à haute disponibilité. Il n'est pas pris en charge dans le cadre d'un déploiement de Zones de disponibilité. 
-  La valeur [maxShares](../../disks-shared-enable.md?tabs=azure-cli#disk-sizes) du disque partagé Azure détermine combien de nœuds de cluster peuvent utiliser le disque partagé. En règle générale, pour une instance SAP ASCS/SCS, vous configurez deux nœuds dans le cluster de basculement Windows. Par conséquent, la valeur de `maxShares` doit être définie sur deux.
-  Toutes les machines virtuelles en cluster SAP ASCS/SCS doivent être déployées dans le même [groupe de placement de proximité Azure](../../windows/proximity-placement-groups.md).   
   Bien qu'il soit possible de déployer des machines virtuelles en cluster Windows dans un groupe à haute disponibilité avec un disque partagé Azure sans groupe de placement de proximité, le groupe de placement de proximité garantira une proximité physique étroite des disques partagés Azure et des machines virtuelles en cluster, réduisant ainsi la latence entre les machines virtuelles et la couche de stockage.    

Pour plus de détails sur les limitations du disque partagé Azure, lisez attentivement la section [Limitations](../../disks-shared.md#limitations) de la documentation consacrée au disque partagé Azure.  

> [!IMPORTANT]
> Lors du déploiement d'un cluster de basculement Windows SAP ASCS/SCS avec disque partagé Azure, sachez que votre déploiement fonctionnera avec un seul disque partagé au sein d'un cluster de stockage. Votre instance SAP ASCS/SCS sera impactée en cas de problème au niveau du cluster de stockage dans lequel le disque partagé Azure est déployé.  

> [!IMPORTANT]
> La configuration doit répondre aux conditions suivantes :
> * Chaque SID de système de gestion de base de données (SGBD) a son propre cluster WSFC dédié.  
> * Les serveurs d’applications SAP appartenant au système SAP SID utilisent leurs propres machines virtuelles.  
> * La combinaison d'Enqueue Replication Server 1 et d'Enqueue Replication Server 2 dans le même cluster n'est pas prise en charge.  


## <a name="supported-os-versions"></a>Versions du système d'exploitation prises en charge

Windows Server 2016 et Windows Server 2019 sont tous deux pris en charge (utilisez les images les plus récentes du centre de données).

Nous vous recommandons vivement d'utiliser **Windows Server 2019 Datacenter**, car :
- Le service de cluster de basculement Windows 2019 est compatible avec Azure.
- L'intégration et la sensibilisation à la maintenance de l'hôte Azure sont renforcées, et l'expérience est améliorée grâce à la surveillance des événements de planification Azure.
- Il est possible d'utiliser le nom du réseau distribué (il s'agit de l'option par défaut). Il n'est donc pas nécessaire d'avoir une adresse IP dédiée pour le nom réseau du cluster. En outre, il n'est pas nécessaire de configurer cette adresse IP sur l'équilibreur de charge interne Azure. 

## <a name="architecture"></a>Architecture

Le serveur ERS1 (Enqueue Replication Server 1) et le serveur ERS2 (Enqueue Replication Server 2) sont pris en charge dans la configuration multi-SID.  Il n'est pas possible de combiner les serveurs ERS1 et ERS2 dans le même cluster. 

1. Le premier exemple illustre deux SID SAP, tous deux dotés de l'architecture ERS1 où :

   - SAP SID1 est déployé sur un disque partagé, avec ERS1. L'instance ERS est installée sur l'hôte local et sur le lecteur local.
     SAP SID1 possède sa propre adresse IP (virtuelle) (SID1 (A)SCS IP1), qui est configurée sur l'équilibreur de charge interne Azure.

   - SAP SID2 est déployé sur un disque partagé, avec ERS1. L'instance ERS est installée sur l'hôte local et sur le lecteur local.
     SAP SID2 possède sa propre adresse IP (virtuelle) (SID2 (A)SCS IP2), qui est aussi configurée sur l'équilibreur de charge interne Azure.

![Instance SAP ASCS/SCS à haute disponibilité - deux instances avec configuration ERS1][sap-ha-guide-figure-6007]

2. Le deuxième exemple illustre deux SID SAP, tous deux dotés de l'architecture ERS2 où : 

   - SAP SID1 avec ERS2 est également mis en cluster et déployé sur le lecteur local.  
     SAP SID1 possède sa propre adresse IP (virtuelle) (SID1 (A)SCS IP1), qui est configurée sur l'équilibreur de charge interne Azure.
     SAP ERS2, utilisé par le système SAP SID1, possède sa propre adresse IP (virtuelle) (SID1 ERS2 IP2), qui est configurée sur l'équilibreur de charge interne Azure.

   - SAP SID2 avec ERS2 est également mis en cluster et déployé sur le lecteur local.  
     SAP SID2 possède sa propre adresse IP (virtuelle) (SID2 (A)SCS IP3), qui est configurée sur l'équilibreur de charge interne Azure.
     SAP ERS2, utilisé par le système SAP SID2, possède sa propre adresse IP (virtuelle) (SID2 ERS2 IP4), qui est configurée sur l'équilibreur de charge interne Azure.

   Nous avons ici un total de quatre adresses IP virtuelles :  
   - SID1 (A)SCS IP1
   - SID2 ERS2   IP2
   - SID2 (A)SCS IP3
   - SID2 ERS2   IP4

![Instance SAP ASCS/SCS à haute disponibilité - deux instances avec configuration ERS1 et ERS2][sap-ha-guide-figure-6008]

## <a name="infrastructure-preparation"></a>Préparation de l'infrastructure

Nous allons installer un nouveau SID SAP **PR2**, en plus de l'instance SAP **PR1** ASCS/SCS **en cluster existante**.  

### <a name="host-names-and-ip-addresses"></a>Noms d’hôtes et adresses IP

| Rôle du nom d'hôte | Nom de l’hôte | Adresse IP statique | Groupe à haute disponibilité | Groupe de placement de proximité |
| --- | --- | --- |---| ---|
| 1er nœud de cluster du cluster ASCS/SCS |pr1-ascs-10 |10.0.0.4 |pr1-ascs-avset |PR1PPG |
| 2e nœud de cluster du cluster ASCS/SCS |pr1-ascs-11 |10.0.0.5 |pr1-ascs-avset |PR1PPG |
| Nom réseau du cluster | pr1clust |10.0.0.42 (**uniquement** pour le cluster Win 2016) | n/a | n/a |
| Nom réseau du cluster **SID1** ASCS | pr1-ascscl |10.0.0.43 | n/a | n/a |
| Nom réseau du cluster **SID1** ERS (**uniquement** pour ERS2) | pr1-erscl |10.0.0.44 | n/a | n/a |
| Nom réseau du cluster **SID2** ASCS | pr2-ascscl |10.0.0.45 | n/a | n/a |
| Nom réseau du cluster **SID2** ERS (**uniquement** pour ERS2) | pr1-erscl |10.0.0.46 | n/a | n/a |

### <a name="create-azure-internal-load-balancer"></a>Créer l'équilibreur de charge interne Azure

SAP ASCS, SAP SCS et la nouvelle instance SAP ERS2 utilisent un nom d'hôte virtuel et des adresses IP virtuelles. Sur Azure, un [équilibreur de charge](../../../load-balancer/load-balancer-overview.md) est nécessaire pour utiliser une adresse IP virtuelle. Nous vous recommandons vivement d'utiliser [Standard Load Balancer](../../../load-balancer/quickstart-load-balancer-standard-public-portal.md). 

Vous devrez ajouter la configuration à l'équilibreur de charge existant pour la deuxième instance SAP SID ASCS/SCS/ERS **PR2**. La configuration du premier SID SAP **PR1** doit déjà être en place.  

**(A)SCS PR2 [numéro d'instance 02]**
- Configuration du frontend
    - Adresse IP ASCS/SCS statique **10.0.0.45**
- Configuration du backend  
    Déjà en place - les machines virtuelles ont déjà été ajoutées au pool back-end, lors de la configuration du SID SAP **PR1**
- Port de la sonde
    - Port 620 **nr** [**62002**] Conserver l'option par défaut pour Protocole (TCP), Intervalle (5), Seuil de défaillance sur le plan de l'intégrité (2)
- Règles d’équilibrage de charge
    - Si vous utilisez Standard Load Balancer, sélectionnez Ports haute disponibilité
    - Si vous utilisez Basic Load Balancer, créez des règles d’équilibrage de charge pour les ports suivants
        - 32 **nr** TCP [**3202**]
        - 36 **nr** TCP [**3602**]
        - 39 **nr** TCP [**3902**]
        - 81 **nr** TCP [**8102**]
        - 5 **nr** 13 TCP [**50213**]
        - 5 **nr** 14 TCP [**50214**]
        - 5 **nr** 16 TCP [**50216**]
        - Associer à l'adresse IP front-end ASCS **PR2**, à la sonde d'intégrité et au pool back-end existant.  

    - Assurez-vous que le délai d'inactivité (en minutes) est défini sur la valeur maximale de 30 et que l'adresse IP flottante (retour direct du serveur) est activée.

**ERS2 PR2 [numéro d'instance 12]** 

Comme le serveur ERS2 (Enqueue Replication Server 2) est également en cluster, l'adresse IP virtuelle ERS2 doit aussi être configurée sur Azure ILB en plus de l'adresse IP SAP ASCS/SCS ci-dessus. Cette section ne s'applique que si l'architecture ERS2 (Enqueue Replication Server 2) est utilisée pour **PR2**.  
- Nouvelle configuration du front-end
    - Adresse IP statique SAP ERS2 **10.0.0.46**

- Configuration du backend  
  Les machines virtuelles ont déjà été ajoutées au pool back-end ILB.  

- Nouveau port de sonde
    - Port 621 **nr** [**62112**] Conserver l'option par défaut pour Protocole (TCP), Intervalle (5), Seuil de défaillance sur le plan de l'intégrité (2)

- Nouvelles règles d'équilibrage de charge
    - Si vous utilisez Standard Load Balancer, sélectionnez Ports haute disponibilité
    - Si vous utilisez Basic Load Balancer, créez des règles d’équilibrage de charge pour les ports suivants
        - 32 **nr** TCP [**3212**]
        - 33 **nr** TCP [**3312**]
        - 5 **nr** 13 TCP [**51212**]
        - 5 **nr** 14 TCP [**51212**]
        - 5 **nr** 16 TCP [**51212**]
        - Associer à l'adresse IP front-end ERS2 **PR2**, à la sonde d'intégrité et au pool back-end existant.  

    - Assurez-vous que le délai d'inactivité (en minutes) est défini sur la valeur maximale, par exemple 30, et que l'adresse IP flottante (retour direct du serveur) est activée.


### <a name="create-and-attach-second-azure-shared-disk"></a>Créer et attacher un deuxième disque partagé Azure

Exécutez cette commande sur l'un des nœuds du cluster. Vous devez ajuster les valeurs de votre groupe de ressources, de votre région Azure, du SID SAP, etc.  

```powershell
    $ResourceGroupName = "MyResourceGroup"
    $location = "MyRegion"
    $SAPSID = "PR2"
    $DiskSizeInGB = 512
    $DiskName = "$($SAPSID)ASCSSharedDisk"
    $NumberOfWindowsClusterNodes = 2
    $diskConfig = New-AzDiskConfig -Location $location -SkuName Premium_LRS  -CreateOption Empty  -DiskSizeGB $DiskSizeInGB -MaxSharesCount $NumberOfWindowsClusterNodes
    
    $dataDisk = New-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName -Disk $diskConfig
    ##################################
    ## Attach the disk to cluster VMs
    ##################################
    # ASCS Cluster VM1
    $ASCSClusterVM1 = "pr1-ascs-10"
    # ASCS Cluster VM2
    $ASCSClusterVM2 = "pr1-ascs-11"
    # next free LUN number
    $LUNNumber = 1
    # Add the Azure Shared Disk to Cluster Node 1
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM1 
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun $LUNNumber
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
    # Add the Azure Shared Disk to Cluster Node 2
    $vm = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $ASCSClusterVM2
    $vm = Add-AzVMDataDisk -VM $vm -Name $DiskName -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun $LUNNumber
    Update-AzVm -VM $vm -ResourceGroupName $ResourceGroupName -Verbose
   ```
### <a name="format-the-shared-disk-with-powershell"></a>Formater le disque partagé avec PowerShell
1. Procurez-vous le numéro du disque. Exécutez les commandes PowerShell sur l'un des nœuds du cluster :

   ```powershell
    Get-Disk | Where-Object PartitionStyle -Eq "RAW"  | Format-Table -AutoSize 
    # Example output
    # Number Friendly Name     Serial Number HealthStatus OperationalStatus Total Size Partition Style
    # ------ -------------     ------------- ------------ ----------------- ---------- ---------------
    # 3      Msft Virtual Disk               Healthy      Online                512 GB RAW            

   ```
2. Formatez le disque. Dans cet exemple, il s'agit du disque numéro 3. 

   ```powershell
    # Format SAP ASCS Disk number '3', with drive letter 'S'
    $SAPSID = "PR2"
    $DiskNumber = 3
    $DriveLetter = "S"
    $DiskLabel = "$SAPSID" + "SAP"
    
    Get-Disk -Number $DiskNumber | Where-Object PartitionStyle -Eq "RAW" | Initialize-Disk -PartitionStyle GPT -PassThru |  New-Partition -DriveLetter $DriveLetter -UseMaximumSize | Format-Volume  -FileSystem ReFS -NewFileSystemLabel $DiskLabel -Force -Verbose
    # Example outout
    # DriveLetter FileSystemLabel FileSystem DriveType HealthStatus OperationalStatus SizeRemaining      Size
    # ----------- --------------- ---------- --------- ------------ ----------------- -------------      ----
    # S           PR2SAP          ReFS       Fixed     Healthy      OK                    504.98 GB 511.81 GB
   ```

3. Vérifiez que le disque est maintenant visible en tant que disque de cluster.  
   ```powershell
    # List all disks
    Get-ClusterAvailableDisk -All
    # Example output
    # Cluster    : pr1clust
    # Id         : c469b5ad-d089-4d8f-ae4c-d834cbbde1a2
    # Name       : Cluster Disk 2
    # Number     : 3
    # Size       : 549755813888
    # Partitions : {\\?\GLOBALROOT\Device\Harddisk3\Partition2\}
   ```

4. Enregistrez le disque dans le cluster.  
   ```powershell
     # Add the disk to cluster 
    Get-ClusterAvailableDisk -All | Add-ClusterDisk
    # Example output 
    # Name           State  OwnerGroup        ResourceType 
    # ----           -----  ----------        ------------ 
    # Cluster Disk 2 Online Available Storage Physical Disk
   ```

## <a name="create-a-virtual-host-name-for-the-clustered-sap-ascsscs-instance"></a>Créer un nom d’hôte virtuel pour l’instance SAP ASCS/SCS en cluster

1. Créez une entrée DNS pour le nom d'hôte virtuel de la nouvelle instance SAP ASCS/SCS dans le gestionnaire DNS Windows.  
   L'adresse IP que vous attribuez au nom d'hôte virtuel dans le DNS doit être identique à celle que vous avez attribuée dans Azure Load Balancer.  

   ![Définir l'entrée DNS pour le nom virtuel et l'adresse IP du cluster SAP ASCS/SCS][sap-ha-guide-figure-6009]
 
   _Définir l'entrée DNS pour le nom virtuel et l'adresse IP du cluster SAP ASCS/SCS_

2. Si vous utilisez SAP Enqueue Replication Server 2, qui est également une instance en cluster, vous devez aussi réserver dans le DNS un nom d'hôte virtuel pour ERS2. 
   L'adresse IP que vous attribuez au nom d'hôte virtuel pour ERS2 dans le DNS doit être identique à celle que vous avez attribuée dans Azure Load Balancer.  

   ![_Définir l'entrée DNS pour le nom virtuel et l'adresse IP du cluster SAP ERS2][sap-ha-guide-figure-6010]
 
   _Définir l'entrée DNS pour le nom virtuel et l'adresse IP du cluster SAP ERS2_

3. Pour définir l’adresse IP affectée au nom d’hôte virtuel, sélectionnez **Gestionnaire DNS** > **Domaine**.

   ![Nouveau nom virtuel et nouvelle adresse IP de la configuration de cluster SAP ASCS/SCS et ERS2][sap-ha-guide-figure-6011]

   _Nouveau nom virtuel et nouvelle adresse TCP/IP de la configuration de cluster SAP ASCS/SCS et ERS2_

## <a name="sap-installation"></a>Installation de SAP 

### <a name="install-the-sap-first-cluster-node"></a>Installer le premier nœud de cluster SAP

Suivez la procédure d'installation décrite par SAP. Assurez-vous que l'option de démarrage de l'installation « Premier nœud de cluster » est activée, et choisissez « Disque partagé de cluster » comme option de configuration.  
Choisissez le disque partagé nouvellement créé.  

### <a name="modify-the-sap-profile-of-the-ascsscs-instance"></a>Modifier le profil SAP de l'instance ASCS/SCS

Si vous exécutez Enqueue Replication Server 1, ajoutez le paramètre de profil SAP `enque/encni/set_so_keepalive`, comme décrit ci-dessous. Ce paramètre de profil empêche les connexions entre les processus de travail SAP et le serveur de mise en file d’attente de se fermer lorsqu’elles sont inactives pendant trop longtemps. Le paramètre SAP n'est pas nécessaire pour ERS2. 

1. Si vous utilisez ERS1, ajoutez ce paramètre de profil au profil de l'instance SAP ASCS/SCS.

   ```
   enque/encni/set_so_keepalive = true
   ```
   
   Pour ERS1 et ERS2, assurez-vous que les paramètres de système d'exploitation `keepalive` sont définis comme décrit dans la note SAP [1410736](https://launchpad.support.sap.com/#/notes/1410736).   

2. Pour appliquer les changements de paramètres du profil SAP, redémarrez l'instance SAP ASCS/SCS.

### <a name="configure-probe-port-on-the-cluster-resource"></a>Configurer le port de la sonde sur la ressource de cluster

Utilisez la fonctionnalité de sondage de l’équilibrage de charge interne pour que la configuration de cluster entière fonctionne avec Azure Load Balancer. Généralement, l’équilibrage de charge interne Azure répartit équitablement la charge de travail entrante entre les machines virtuelles participantes.

Toutefois, cela ne fonctionne pas dans certaines configurations de cluster, car une seule instance est active. L’autre instance est passive et ne peut accepter aucune partie de la charge de travail. La fonctionnalité de sondage est utile lorsque l'équilibreur de charge interne Azure détecte quelle instance est active, et ne cible que l'instance active.  

> [!IMPORTANT]
> Dans cet exemple de configuration, le paramètre **ProbePort** est défini sur 620 **Nr**. Pour l'instance SAP ASCS portant le numéro **02**, il est défini sur 620 **02**.
> Vous devrez ajuster la configuration pour qu'elle corresponde à vos numéros d'instance SAP et à votre SID SAP.

Pour ajouter un port de sonde, exécutez ce module PowerShell sur l'une des machines virtuelles du cluster :

- Dans le cas d'une instance SAP ASC/SCS portant le numéro d'instance **02** 
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID PR2 -ProbePort 62002
   ```

- Si vous utilisez ERS2, avec le numéro d'instance **12**, qui est en cluster. Il n'est pas nécessaire de configurer le port de sonde pour ERS1, car il n'est pas en cluster.  
   ```powershell
   Set-AzureLoadBalancerHealthCheckProbePortOnSAPClusterIPResource -SAPSID PR2 -ProbePort 62012 -IsSAPERSClusteredInstance $True
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
    # To activate the changes you need to manually restart 'SAP AB1' cluster group.
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
    
                #Write-Host "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" -ForegroundColor Cyan
                Write-Output "Current configuration parameters for SAP IP cluster resource '$SAPIPresourceName' are:" 
    
                Get-ClusterResource -Name $SAPIPresourceName | Get-ClusterParameter
    
                Write-Output " "
                Write-Output "Current probe port property of the SAP cluster resource '$SAPIPresourceName' is '$OldProbePort'." 
                Write-Output " "
                Write-Output "Setting the new probe port property of the SAP cluster resource '$SAPIPresourceName' to '$ProbePort' ..." 
                Write-Output " "
    
                $var | Set-ClusterParameter -Multiple @{"Address"=$IPAddress;"ProbePort"=$ProbePort;"Subnetmask"=$SubnetMask;"Network"=$NetworkName;"OverrideAddressMatch"=$OverrideAddressMatch;"EnableDhcp"=$EnableDhcp}
    
                Write-Output " "
    
                #$ActivateChanges = Read-Host "Do you want to take restart SAP cluster role '$SAPClusterRoleName', to activate the changes (yes/no)?"
    
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

### <a name="continue-with-the-sap-installation"></a>Poursuivre l'installation de SAP

1. Installez l'instance de base de données en suivant la procédure décrite dans le guide d'installation de SAP.  
2. Installez SAP sur le deuxième nœud de cluster en suivant les étapes décrites dans le guide d'installation de SAP.  
3. Installez l'instance du serveur d'application principal (PAS) SAP sur la machine virtuelle que vous avez désignée pour héberger le serveur PAS.   
   Suivez la procédure décrite dans le guide d'installation de SAP. Il n’y a aucune dépendance vis-à-vis d’Azure.
4. Installez des serveurs d'applications SAP supplémentaires sur les machines virtuelles désignées pour héberger des instances de SAP.  
   Suivez la procédure décrite dans le guide d'installation de SAP. Il n’y a aucune dépendance vis-à-vis d’Azure. 

## <a name="test-the-sap-ascsscs-instance-failover"></a>Testez le basculement de l’instance SAP ASCS/SCS
Pour les tests de basculement décrits, nous supposons que SAP ASCS est actif sur le nœud A.  

1. Vérifiez que le système SAP peut correctement basculer du nœud A vers le nœud B. Dans cet exemple, le test est effectué pour le SID SAP **PR2**.  
   Assurez-vous que chacun des SID SAP peut correctement se déplacer vers l'autre nœud du cluster.   
   Choisissez l’une des options suivantes pour lancer le basculement du groupe de clusters \<SID\> SAP du nœud de cluster A au nœud de cluster B :
    - Gestionnaire du cluster de basculement  
    - Applet de commande PowerShell de cluster de basculement

    ```powershell
    $SAPSID = "PR2"     # SAP <SID>
 
    $SAPClusterGroup = "SAP $SAPSID"
    Move-ClusterGroup -Name $SAPClusterGroup

    ```
2. Redémarrez le nœud de cluster A dans le système d’exploitation invité Windows. Cela lance le basculement automatique du groupe de clusters \<SID\> SAP du nœud A vers le nœud B.  
3. Redémarrez le nœud de cluster A à partir du portail Azure. Cela lance le basculement automatique du groupe de clusters \<SID\> SAP du nœud A vers le nœud B.  
4. Redémarrez le nœud de cluster A à l’aide d’Azure PowerShell. Cela lance le basculement automatique du groupe de clusters \<SID\> SAP du nœud A vers le nœud B.

## <a name="next-steps"></a>Étapes suivantes

* [Préparer l’infrastructure Azure pour la haute disponibilité SAP à l’aide d’un cluster de basculement Windows et d’un disque partagé pour une instance SAP ASCS/SCS][sap-high-availability-infrastructure-wsfc-shared-disk]
* [Installer la haute disponibilité SAP NetWeaver sur un cluster de basculement Windows et un disque partagé pour une instance SAP ASCS/SCS][sap-high-availability-installation-wsfc-shared-disk]


[1928533]:https://launchpad.support.sap.com/#/notes/1928533
[1999351]:https://launchpad.support.sap.com/#/notes/1999351
[2015553]:https://launchpad.support.sap.com/#/notes/2015553
[2178632]:https://launchpad.support.sap.com/#/notes/2178632
[2243692]:https://launchpad.support.sap.com/#/notes/2243692
[1869038]:https://launchpad.support.sap.com/#/notes/1869038
[2287140]:https://launchpad.support.sap.com/#/notes/2287140
[2492395]:https://launchpad.support.sap.com/#/notes/2492395

[sap-installation-guides]:http://service.sap.com/instguides

[azure-resource-manager/management/azure-subscription-service-limits]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[azure-resource-manager/management/azure-subscription-service-limits-subscription]:../../../azure-resource-manager/management/azure-subscription-service-limits.md
[networking-limits-azure-resource-manager]:../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits
[load-balancer-multivip-overview]:../../../load-balancer/load-balancer-multivip-overview.md


[sap-net-weaver-ports]:https://help.sap.com/viewer/ports
[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-hana-ha]:sap-hana-high-availability.md
[sap-suse-ascs-ha]:high-availability-guide-suse.md
[sap-net-weaver-ports-ascs-scs-ports]:sap-high-availability-infrastructure-wsfc-shared-disk.md#fe0bd8b5-2b43-45e3-8295-80bee5415716

[dbms-guide]:../../virtual-machines-windows-sap-dbms-guide.md

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

[sap-high-availability-architecture-scenarios]:sap-high-availability-architecture-scenarios.md
[sap-high-availability-guide-wsfc-shared-disk]:sap-high-availability-guide-wsfc-shared-disk.md
[sap-high-availability-guide-wsfc-file-share]:sap-high-availability-guide-wsfc-file-share.md
[sap-ascs-high-availability-multi-sid-wsfc]:sap-ascs-high-availability-multi-sid-wsfc.md
[sap-high-availability-infrastructure-wsfc-shared-disk]:sap-high-availability-infrastructure-wsfc-shared-disk.md
[sap-high-availability-infrastructure-wsfc-file-share]:sap-high-availability-infrastructure-wsfc-file-share.md

[sap-high-availability-installation-wsfc-shared-disk]:sap-high-availability-installation-wsfc-shared-disk.md
[sap-high-availability-installation-wsfc-shared-disk-install-ascs]:sap-high-availability-installation-wsfc-shared-disk.md#31c6bd4f-51df-4057-9fdf-3fcbc619c170
[sap-high-availability-installation-wsfc-shared-disk-modify-ascs-profile]:sap-high-availability-installation-wsfc-shared-disk.md#e4caaab2-e90f-4f2c-bc84-2cd2e12a9556
[sap-high-availability-installation-wsfc-shared-disk-add-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#10822f4f-32e7-4871-b63a-9b86c76ce761
[sap-high-availability-installation-wsfc-shared-disk-win-firewall-probe-port]:sap-high-availability-installation-wsfc-shared-disk.md#4498c707-86c0-4cde-9c69-058a7ab8c3ac
[sap-high-availability-installation-wsfc-shared-disk-change-ers-service-startup-type]:sap-high-availability-installation-wsfc-shared-disk.md#094bc895-31d4-4471-91cc-1513b64e406a
[sap-high-availability-installation-wsfc-shared-disk-test-ascs-failover-and-sios-repl]:sap-high-availability-installation-wsfc-shared-disk.md#18aa2b9d-92d2-4c0e-8ddd-5acaabda99e9


[sap-high-availability-installation-wsfc-file-share]:sap-high-availability-installation-wsfc-file-share.md
[sap-high-availability-infrastructure-wsfc-shared-disk-install-sios]:sap-high-availability-infrastructure-wsfc-shared-disk.md#5c8e5482-841e-45e1-a89d-a05c0907c868

[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png

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

[sap-ha-guide-figure-6001]:media/virtual-machines-shared-sap-high-availability-guide/6001-sap-multi-sid-ascs-scs-sid1.png
[sap-ha-guide-figure-6002]:media/virtual-machines-shared-sap-high-availability-guide/6002-sap-multi-sid-ascs-scs.png
[sap-ha-guide-figure-6003]:media/virtual-machines-shared-sap-high-availability-guide/6003-sap-multi-sid-full-landscape.png
[sap-ha-guide-figure-6004]:media/virtual-machines-shared-sap-high-availability-guide/6004-sap-multi-sid-dns.png
[sap-ha-guide-figure-6005]:media/virtual-machines-shared-sap-high-availability-guide/6005-sap-multi-sid-azure-portal.png
[sap-ha-guide-figure-6006]:media/virtual-machines-shared-sap-high-availability-guide/6006-sap-multi-sid-sios-replication.png

[sap-ha-guide-figure-6007]:media/virtual-machines-shared-sap-high-availability-guide/6007-sap-multi-sid-ascs-azure-shared-disk-sid1.png
[sap-ha-guide-figure-6008]:media/virtual-machines-shared-sap-high-availability-guide/6008-sap-multi-sid-ascs-azure-shared-disk-sid2.png
[sap-ha-guide-figure-6009]:media/virtual-machines-shared-sap-high-availability-guide/6009-sap-multi-sid-ascs-azure-shared-disk-dns1.png
[sap-ha-guide-figure-6010]:media/virtual-machines-shared-sap-high-availability-guide/6010-sap-multi-sid-ascs-azure-shared-disk-dns2.png
[sap-ha-guide-figure-6011]:media/virtual-machines-shared-sap-high-availability-guide/6011-sap-multi-sid-ascs-azure-shared-disk-dns3.png

[sap-ha-guide-figure-8001]:./media/virtual-machines-shared-sap-high-availability-guide/8001.png
[sap-ha-guide-figure-8002]:./media/virtual-machines-shared-sap-high-availability-guide/8002.png
[sap-ha-guide-figure-8003]:./media/virtual-machines-shared-sap-high-availability-guide/8003.png
[sap-ha-guide-figure-8004]:./media/virtual-machines-shared-sap-high-availability-guide/8004.png
[sap-ha-guide-figure-8005]:./media/virtual-machines-shared-sap-high-availability-guide/8005.png
[sap-ha-guide-figure-8006]:./media/virtual-machines-shared-sap-high-availability-guide/8006.png
[sap-ha-guide-figure-8007]:./media/virtual-machines-shared-sap-high-availability-guide/8007.png
[sap-ha-guide-figure-8008]:./media/virtual-machines-shared-sap-high-availability-guide/8008.png
[sap-ha-guide-figure-8009]:./media/virtual-machines-shared-sap-high-availability-guide/8009.png
[sap-ha-guide-figure-8010]:./media/virtual-machines-shared-sap-high-availability-guide/8010.png
[sap-ha-guide-figure-8011]:./media/virtual-machines-shared-sap-high-availability-guide/8011.png
[sap-ha-guide-figure-8012]:./media/virtual-machines-shared-sap-high-availability-guide/8012.png
[sap-ha-guide-figure-8013]:./media/virtual-machines-shared-sap-high-availability-guide/8013.png
[sap-ha-guide-figure-8014]:./media/virtual-machines-shared-sap-high-availability-guide/8014.png
[sap-ha-guide-figure-8015]:./media/virtual-machines-shared-sap-high-availability-guide/8015.png
[sap-ha-guide-figure-8016]:./media/virtual-machines-shared-sap-high-availability-guide/8016.png
[sap-ha-guide-figure-8017]:./media/virtual-machines-shared-sap-high-availability-guide/8017.png
[sap-ha-guide-figure-8018]:./media/virtual-machines-shared-sap-high-availability-guide/8018.png
[sap-ha-guide-figure-8019]:./media/virtual-machines-shared-sap-high-availability-guide/8019.png
[sap-ha-guide-figure-8020]:./media/virtual-machines-shared-sap-high-availability-guide/8020.png
[sap-ha-guide-figure-8021]:./media/virtual-machines-shared-sap-high-availability-guide/8021.png
[sap-ha-guide-figure-8022]:./media/virtual-machines-shared-sap-high-availability-guide/8022.png
[sap-ha-guide-figure-8023]:./media/virtual-machines-shared-sap-high-availability-guide/8023.png
[sap-ha-guide-figure-8024]:./media/virtual-machines-shared-sap-high-availability-guide/8024.png
[sap-ha-guide-figure-8025]:./media/virtual-machines-shared-sap-high-availability-guide/8025.png


[sap-templates-3-tier-multisid-xscs-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs%2Fazuredeploy.json
[sap-templates-3-tier-multisid-xscs-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-xscs-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db%2Fazuredeploy.json
[sap-templates-3-tier-multisid-db-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-db-md%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps%2Fazuredeploy.json
[sap-templates-3-tier-multisid-apps-marketplace-image-md]:https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fsap-3-tier-marketplace-image-multi-sid-apps-md%2Fazuredeploy.json

[virtual-machines-azure-resource-manager-architecture-benefits-arm]:../../../azure-resource-manager/management/overview.md#the-benefits-of-using-resource-manager

[virtual-machines-manage-availability]:../../virtual-machines-windows-manage-availability.md