---
title: Migrer des machines virtuelles VMware vers Azure (sans agent) - PowerShell
description: Découvrez comment exécuter la migration sans agent de machines virtuelles VMware avec Azure Migrate par le biais de PowerShell.
author: rahulg1190
ms.author: rahugup
manager: bsiva
ms.topic: tutorial
ms.date: 10/1/2020
ms.openlocfilehash: ce712736c25c0757f5b654e4442a4f08acdf15e2
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029785"
---
# <a name="migrate-vmware-vms-to-azure-agentless---powershell"></a>Migrer des machines virtuelles VMware vers Azure (sans agent) - PowerShell

Dans cet article, vous allez apprendre à migrer des machines virtuelles VMware découvertes avec la méthode sans agent à l’aide d’Azure PowerShell pour l’outil [Azure Migrate : Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool). 

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Récupérer des machines virtuelles VMware découvertes dans un projet Azure Migrate.
> * Démarrer la réplication des machines virtuelles
> * Mettre à jour les propriétés de réplication des machines virtuelles.
> * Superviser la réplication.
> * Exécuter une migration de test pour vérifier que tout fonctionne comme prévu.
> * Exécuter une migration de machine virtuelle complète

> [!NOTE]
> Les tutoriels vous montrent le chemin de déploiement le plus simple pour un scénario donné afin que vous puissiez configurer rapidement une preuve de concept. Ils utilisent des options par défaut, le cas échéant, et ne montrent pas tous les paramètres et chemins possibles.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce didacticiel, vous devez :

1. [Suivre le tutoriel sur la découverte](tutorial-discover-vmware.md) afin de configurer Azure et VMware pour la migration.
2. Nous vous recommandons de suivre le deuxième tutoriel afin d’[évaluer les machines virtuelles VMware](./tutorial-assess-vmware-azure-vm.md) avant de les migrer vers Azure.
3. Vous devez disposer du module Azure PowerShell `Az`. Si vous devez installer ou mettre à niveau Azure PowerShell, suivez le [guide d’installation et de configuration d’Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="install-azure-migrate-powershell-module"></a>Installer le module PowerShell Azure Migrate

Le module PowerShell Azure Migrate est disponible en préversion. Vous devez installer le module PowerShell à l’aide de la commande suivante. 

```azurepowershell
Install-Module -Name Az.Migrate 
```

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>Se connecter à un abonnement Microsoft Azure

Connectez-vous à votre abonnement Azure avec la cmdlet `Connect-AzAccount`.

```azurepowershell
Connect-AzAccount
```

Sélectionnez votre abonnement Azure. Utilisez la cmdlet `Get-AzSubscription` pour obtenir la liste des abonnements Azure auxquels vous avez accès. Sélectionnez l’abonnement Azure contenant le projet Azure Migrate que vous voulez utiliser à l’aide de l’applet de commande `Set-AzContext`.

```azurepowershell
Set-AzContext -SubscriptionId "xxxx-xxxx-xxxx-xxxx"
```

## <a name="retrieve-the-azure-migrate-project"></a>Récupérer le projet Azure Migrate

Un projet de Azure Migrate est utilisé pour stocker les métadonnées de découverte, d’évaluation et de migration collectées à partir de l’environnement que vous évaluez ou migrez.
Dans un projet, vous pouvez suivre les ressources découvertes, orchestrer les évaluations et effectuer des migrations.

Dans le cadre des conditions préalables, vous devoir avoir déjà créé un projet Azure Migrate. Utilisez l’applet de commande `Get-AzMigrateProject` pour récupérer les détails d’un projet Azure Migrate. Vous devez spécifier le nom du projet Azure Migrate (`Name`) et le nom du groupe de ressources de ce projet (`ResourceGroupName`).

```azurepowershell
# Get resource group of the Azure Migrate project
$ResourceGroup = Get-AzResourceGroup -Name "MyResourceGroup"

# Get details of the Azure Migrate project
$MigrateProject = Get-AzMigrateProject -Name "MyMigrateProject" -ResourceGroupName $ResourceGroup.ResourceGroupName

# View Azure Migrate project details
$MigrateProject | ConvertTo-JSON
```

## <a name="retrieve-discovered-vms-in-an-azure-migrate-project"></a>Récupérer des machines virtuelles découvertes dans un projet Azure Migrate

Azure Migrate utilise une [appliance Azure Migrate légère](migrate-appliance-architecture.md). Dans le cadre des conditions préalables, vous devez avoir déployé l’appliance Azure Migrate comme machine virtuelle VMware.

Pour récupérer une machine virtuelle VMware spécifique dans un projet Azure Migrate, spécifiez le nom de ce dernier (`ProjectName`), son groupe de ressources (`ResourceGroupName`) et le nom de la machine virtuelle (`DisplayName`). 

> [!NOTE]
> **La valeur du paramètre de nom de machine virtuelle (`DisplayName`) respecte la casse**.

```azurepowershell
# Get a specific VMware VM in an Azure Migrate project
$DiscoveredServer = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -DisplayName "MyTestVM"

# View discovered server details
$DiscoveredServer | ConvertTo-JSON
```
Dans le cadre de ce tutoriel, nous allons migrer cette machine virtuelle.

Vous pouvez également récupérer toutes les machines virtuelles VMware d’un projet Azure Migrate à l’aide des paramètres `ProjectName` et `ResourceGroupName`.

```azurepowershell
# Get all VMware VMs in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName 
```
Si vous avez plusieurs appliances dans un projet Azure Migrate, vous pouvez utiliser les paramètres `ProjectName`, `ResourceGroupName` et `ApplianceName` pour récupérer toutes les machines virtuelles découvertes à l’aide d’une appliance Azure Migrate spécifique. 

```azurepowershell
# Get all VMware VMs discovered by an Azure Migrate Appliance in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -ApplianceName "MyMigrateAppliance" |Format-Table DisplayName, Name, Type

```

## <a name="initialize-replication-infrastructure"></a>Initialiser l’infrastructure de réplication

[Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool) s’appuie sur plusieurs ressources Azure pour la migration de machines virtuelles. L’outil Migration de serveur provisionne les ressources suivantes dans le même groupe de ressources que le projet.

- **Service Bus** : Migration de serveur utilise Service Bus pour envoyer des messages d’orchestration de réplication à l’appliance.
- **Compte de stockage de la passerelle** : Server Migration utilise le compte de stockage de la passerelle pour stocker des informations d’état relatives aux machines virtuelles en cours de réplication.
- **Compte de stockage de journal** : l’appliance Azure Migrate charge les journaux de réplication pour les machines virtuelles vers un compte de stockage de journal. Azure Migrate applique les informations de réplication aux disques managés de réplica.
- **Coffre de clés** : l’appliance Azure Migrate utilise le coffre de clés afin de gérer les chaînes de connexion pour Service Bus, et des clés d’accès pour les comptes de stockage utilisés dans la réplication.

Avant de répliquer la première machine virtuelle dans le projet Azure Migrate, exécutez le script suivant pour provisionner l’infrastructure de réplication. Ce script provisionne et configure les ressources mentionnées ci-dessus afin que vous puissiez commencer à migrer vos machines virtuelles VMware.

> [!NOTE]
> Un projet Azure Migrate prend en charge les migrations vers une seule région Azure. Une fois que vous avez exécuté ce script, vous ne pouvez pas changer la région cible vers laquelle vous voulez migrer vos machines virtuelles VMware.
> Vous devez exécuter le script `Initialize-AzMigrateReplicationInfrastructure` si vous configurez une nouvelle appliance dans votre projet Azure Migrate. 

Dans cet article, nous allons initialiser l’infrastructure de réplication afin de pouvoir migrer nos machines virtuelles vers la région `Central US`. Vous pouvez [télécharger le fichier](https://github.com/Azure/azure-docs-powershell-samples/tree/master/azure-migrate/migrate-at-scale-vmware-agentles) à partir du dépôt GitHub ou l’exécuter à l’aide de l’extrait de code suivant. 

```azurepowershell
# Download the script from Azure Migrate GitHub repository 
Invoke-WebRequest https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-migrate/migrate-at-scale-vmware-agentles/Initialize-AzMigrateReplicationInfrastructure.ps1 -OutFile .\AzMigrateReplicationinfrastructure.ps1

# Run the script for initializing replication infrastructure for the current Migrate project
.\AzMigrateReplicationInfrastructure.ps1 -ResourceGroupName $ResourceGroup.ResourceGroupName -ProjectName $MigrateProject.Name -Scenario agentlessVMware -TargetRegion "CentralUS" 
```


## <a name="replicate-vms"></a>Répliquer des machines virtuelles

Une fois la découverte et l’initialisation de l’infrastructure de réplication terminées, vous pouvez commencer la réplication de machines virtuelles VMware sur Azure. Vous pouvez exécuter jusqu’à 300 réplications simultanément.

Vous pouvez spécifier les propriétés de réplication comme suit.

- **Abonnement et groupe de ressources cibles** : spécifiez l’abonnement et le groupe de ressources vers lesquels la machine virtuelle doit être migrée en fournissant l’ID du groupe de ressources à l’aide du paramètre `TargetResourceGroupId`. 
- **Réseau virtuel et sous-réseau cibles** : spécifiez l’ID du réseau virtuel Azure et le nom du sous-réseau vers lesquels la machine virtuelle doit être migrée en utilisant les paramètres `TargetNetworkId` et `TargetSubnetName`, respectivement. 
- **Nom de la machine virtuelle cible** : spécifiez le nom de la machine virtuelle Azure à créer à l’aide du paramètre `TargetVMName`.
- **Taille de machine virtuelle cible** : spécifiez la taille de machine virtuelle Azure à utiliser pour la machine virtuelle de réplication à l’aide du paramètre `TargetVMSize`. Par exemple, pour migrer une machine virtuelle vers la machine virtuelle D2_v2 dans Azure, spécifiez « Standard_D2_v2 » comme valeur de `TargetVMSize`.  
- **Licence** : Si vous souhaitez utiliser Azure Hybrid Benefit pour vos ordinateurs Windows Server qui couverts par des abonnements Software Assurance ou Windows Server actifs, spécifiez « WindowsServer » comme valeur du paramètre `LicenseType`. Sinon, spécifiez « NoLicenseType » comme valeur du paramètre `LicenseType`.
- **Disque de système d’exploitation** : spécifiez l’identificateur unique du disque qui contient le chargeur de démarrage et le programme d’installation du système d’exploitation. L’ID de disque à utiliser est la propriété UUID (identificateur unique) pour le disque récupéré à l’aide de l’applet de commande `Get-AzMigrateServer`.
- **Type de disque** : spécifiez la valeur du paramètre `DiskType` comme suit.
    - Pour utiliser des disques managés Premium, spécifiez « Premium_LRS » comme valeur du paramètre `DiskType`. 
    - Pour utiliser des disques SSD standard, spécifiez « StandardSSD_LRS » comme valeur du paramètre `DiskType`. 
    - Pour utiliser des disques HHD standard, spécifiez « Standard_LRS » comme valeur du paramètre `DiskType`. 
- **Redondance d’infrastructure** : spécifiez l’option de redondance d’infrastructure comme suit. 
    - Zone de disponibilité pour épingler la machine migrée à une Zone de disponibilité spécifique dans la région. Utilisez cette option pour distribuer les serveurs qui forment une couche Application à plusieurs nœuds entre des Zones de disponibilité. Cette option est disponible uniquement si la région cible sélectionnée pour la migration prend en charge la fonctionnalité Zones de disponibilité. Pour utiliser des zones de disponibilité, spécifiez la valeur de zone de disponibilité souhaitée pour le paramètre `TargetAvailabilityZone`.
    - Groupe à haute disponibilité pour placer la machine migrée dans un groupe à haute disponibilité. Pour utiliser cette option, le groupe de ressources cible qui a été sélectionné doit comporter un ou plusieurs groupes à haute disponibilité. Pour utiliser un groupe à haute disponibilité, spécifiez son ID pour le paramètre `TargetAvailabilitySet`. 

### <a name="replicate-vms-with-all-disks"></a>Répliquer des machines virtuelles avec tous les disques
Dans ce tutoriel, nous allons répliquer tous les disques de la machine virtuelle découverte et définir un nouveau nom pour celle-ci dans Azure. Nous spécifions le premier disque du serveur découvert comme disque de système d’exploitation et nous migrons tous les disques sous forme de disque HDD Standard. Le disque du système d’exploitation est le disque qui contient le chargeur de démarrage et le programme d’installation du système d’exploitation. L’applet de commande retourne une tâche qui peut être suivie pour superviser l’état de l’opération. 

```azurepowershell
# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name "MyTargetResourceGroup"

# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name "MyVirtualNetwork"

# Start replication for a discovered VM in an Azure Migrate project
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType "NoLicenseType" -OSDiskID $DiscoveredServer.Disk[0].Uuid -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -DiskType "Standard_LRS" -TargetVMName "MyMigratedTestVM" -TargetVMSize "Standard_DS2_v2"

# Track job status to check for completion
while (($MigrateJob.State -eq "InProgress") -or ($MigrateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $MigrateJob.State
```

### <a name="replicate-vms-with-select-disks"></a>Répliquer des machines virtuelles avec une sélection de disques
Vous pouvez également répliquer de manière sélective les disques de la machine virtuelle découverte en utilisant l’applet de commande `New-AzMigrateDiskMapping` et en fournissant le résultat comme entrée du paramètre `DiskToInclude` dans l’applet de commande `New-AzMigrateServerReplication`. Vous pouvez également utiliser l’applet de commande `New-AzMigrateDiskMapping` afin de spécifier différents types de disques cibles pour chaque disque individuel à répliquer. 

Spécifiez des valeurs pour les paramètres suivants de l’applet de commande `New-AzMigrateDiskMapping`.

- **DiskId** : spécifiez l’identificateur unique du disque à migrer. L’ID de disque à utiliser est la propriété UUID (identificateur unique) pour le disque récupéré à l’aide de l’applet de commande `Get-AzMigrateServer`.  
- **IsOSDisk** : spécifiez « true » si le disque à migrer est le disque du système d’exploitation de la machine virtuelle ; sinon, « false ».
- **DiskType** : spécifiez le type de disque à utiliser dans Azure. 

Dans l’exemple suivant, nous allons répliquer uniquement deux disques de la machine virtuelle découverte. Nous allons spécifier le disque du système d’exploitation et utiliser différents types de disques pour chaque disque à répliquer. L’applet de commande retourne une tâche qui peut être suivie pour superviser l’état de l’opération. 

```azurepowershell
# View disk details of the discovered server
$DiscoveredServer.Disk | ConvertTo-JSON

# Create a new disk mapping for the disks to be replicated
$DisksToReplicate = @()
$OSDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[0].Uuid -DiskType "StandardSSD_LRS" -IsOSDisk "true"
$DataDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[1].Uuid -DiskType "Premium_LRS" -IsOSDisk "false"

$DisksToReplicate += $OSDisk
$DisksToReplicate += $DataDisk 

# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name "MyTargetResourceGroup"

# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name "MyVirtualNetwork"

# Start replication for the VM
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType "NoLicenseType" -DiskToInclude $DisksToReplicate -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -TargetVMName "MyMigratedTestVM" -TargetVMSize "Standard_DS2_v2"

# Track job status to check for completion
while (($MigrateJob.State -eq "InProgress") -or ($MigrateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $MigrateJob.State
```

## <a name="monitor-replication"></a>Superviser la réplication 

La réplication se déroule comme suit :

- Une fois le travail Démarrer la réplication terminé, les machines commencent leur réplication initiale vers Azure.
- Lors de la réplication initiale, une capture instantanée de machine virtuelle est créée. Les données de disque de la capture instantanée sont répliquées sur des disques managés de réplica dans Azure.
- Au terme de la réplication initiale, la réplication différentielle commence. Les modifications incrémentielles apportées aux disques locaux sont répliquées régulièrement sur les disques de réplica dans Azure.

Suivez l’état de la réplication à l’aide de l’applet de commande `Get-AzMigrateServerReplication`. 

> [!NOTE]
> L’ID de la machine virtuelle découverte et l’ID de la machine virtuelle de réplication sont deux identificateurs uniques différents. Ces deux identificateurs peuvent être utilisés pour récupérer les détails d’un serveur de réplication.  

### <a name="monitor-replication-using-discovered-vm-identifier"></a>Superviser la réplication à l’aide d’un identificateur de machine virtuelle découverte
```azurepowershell
# Retrieve the replicating VM details by using the discovered VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -DiscoveredMachineId $DiscoveredServer.ID
```

### <a name="monitor-replication-using-replicating-vm-identifier"></a>Superviser la réplication à l’aide d’un identificateur de machine virtuelle de réplication

```azurepowershell
# List all replicating VMs in an Azure Migrate project and filter the result for selecting the replication VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName | where MachineName -eq $DiscoveredServer.DisplayName

# Retrieve replicating VM details using replicating VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id 
```

Vous pouvez suivre les propriétés « MigrationState » (État de la migration) et « MigrationStateDescription » (Description de l’état de la migration) dans la sortie. 
- Pour la réplication initiale, les valeurs des propriétés d’état de la migration et de description de l’état de la migration sont, respectivement, « InitialSeedingInProgress » (Amorçage initial en cours) et « Initial replication » (Réplication initiale). 
- Pendant une réplication différentielle, les valeurs des propriétés d’état de la migration et de description de l’état de la migration sont, respectivement, « Replicating » (Réplication en cours) et « Ready to migrate » (Prêt pour la migration).
- Une fois la migration effectuée, les valeurs des propriétés d’état de la migration et de description de l’état de la migration sont, respectivement, « Migration succeeded » (Migration réussie) et « Migrated » (Migré).

```Output
AllowedOperation            : {DisableMigration, TestMigrate, Migrate}
CurrentJobId                : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationJobs/None
CurrentJobName              : None
CurrentJobStartTime         : 1/1/1753 1:01:01 AM
EventCorrelationId          : 9d435c55-4660-41a5-a8ed-dd74213d85fa
Health                      : Normal
HealthError                 : {}
Id                          : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationFabrics/xxx/replicationProtectionContainers/xxx/
                              replicationMigrationItems/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-
                              39b2-1e14-f90584522703
LastTestMigrationStatus     :
LastTestMigrationTime       :
Location                    :
MachineName                 : MyTestVM
MigrationState              : InitialSeedingInProgress
MigrationStateDescription   : Initial replication
Name                        : 10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-39b2-1e14-f90584522703
PolicyFriendlyName          : xxx
PolicyId                    : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationPolicies/xxx
ProviderSpecificDetail      : Microsoft.Azure.PowerShell.Cmdlets.Migrate.Models.Api20180110.VMwareCbtMigrationDetails
TestMigrateState            : None
TestMigrateStateDescription : None
Type                        : Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems
```

Pour plus d’informations sur la progression d’une réplication, exécutez l’applet de commande suivante.

```azurepowershell
$replicatingserver.ProviderSpecificDetail | convertto-json
```
Vous pouvez suivre la progression de la réplication initiale à l’aide des propriétés « Initial Seeding Progress Percentage » (Pourcentage de progression de l’amorçage initial) dans la sortie.

```output
    "DataMoverRunAsAccountId": "/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.OffAzure/VMwareSites/xxx/runasaccounts/xxx",
    "FirmwareType":  "BIOS",
    "InitialSeedingProgressPercentage": 20,
    "InstanceType":  "VMwareCbt",
    "LastRecoveryPointReceived":  "\/Date(1601733591427)\/",
    "LicenseType":  "NoLicenseType",
    "MigrationProgressPercentage":  null,
    "MigrationRecoveryPointId":  null,
    "OSType":  "Windows",
    "PerformAutoResync":  "true",
```

La réplication se déroule comme suit :

- Une fois le travail Démarrer la réplication terminé, les machines commencent leur réplication initiale vers Azure.
- Lors de la réplication initiale, une capture instantanée de machine virtuelle est créée. Les données de disque de la capture instantanée sont répliquées sur des disques managés de réplica dans Azure.
- Au terme de la réplication initiale, la réplication différentielle commence. Les modifications incrémentielles apportées aux disques locaux sont répliquées régulièrement sur les disques de réplica dans Azure.

## <a name="retrieve-the-status-of-a-job"></a>Récupérer l’état d’une tâche

Vous pouvez superviser l’état d’une tâche à l’aide de l’applet de commande `Get-AzMigrateJob`. 

```azurepowershell
# Retrieve the updated status for a job
$job = Get-AzMigrateJob -InputObject $job
```

## <a name="update-properties-of-a-replicating-vm"></a>Mettre à jour les propriétés d’une machine virtuelle de réplication

L’outil [Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-migration-tool) vous permet de changer des propriétés cibles, comme le nom, la taille, le groupe de ressources, la configuration de la carte réseau, etc., pour une machine virtuelle de réplication. L’applet de commande retourne une tâche qui peut être suivie pour superviser l’état de l’opération. 

```azurepowershell
# Retrieve the replicating VM details by using the discovered VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -DiscoveredMachineId $DiscoveredServer.ID

# View NIC details of the replicating server
Write-Output $ReplicatingServer.ProviderSpecificDetail.VMNic 
```
Les propriétés suivantes peuvent être mises à jour pour une machine virtuelle.

- **Nom de la machine virtuelle** : spécifiez le nom de la machine virtuelle Azure à créer à l’aide du paramètre `TargetVMName`.
- **Taille de machine virtuelle** : spécifiez la taille de machine virtuelle Azure à utiliser pour la machine virtuelle de réplication à l’aide du paramètre `TargetVMSize`. Par exemple, pour migrer une machine virtuelle vers la machine virtuelle D2_v2 dans Azure, spécifiez « Standard_D2_v2 » comme valeur de `TargetVMSize`.  
- **Réseau virtuel** : spécifiez l’ID du réseau virtuel Azure vers lequel la machine virtuelle doit être migrée à l’aide du paramètre `TargetNetworkId`. 
- **Groupe de ressources** : spécifiez l’ID du groupe de ressources vers lequel la machine virtuelle doit être migrée à l’aide du paramètre `TargetResourceGroupId`.
- **Interface réseau** : la configuration de la carte réseau peut être spécifiée à l’aide de l’applet de commande `New-AzMigrateNicMapping`. Une entrée pour le paramètre `NicToUpdate` de l’applet de commande `Set-AzMigrateServerReplication` est ensuite passée à l’objet. 

    - **Changer l’allocation d’adresses IP** : pour spécifier une adresse IP statique pour une carte réseau, fournissez l’adresse IPv4 à utiliser comme adresse IP statique pour la machine virtuelle à l’aide du paramètre `TargetNicIP`. Pour attribuer une adresse IP de façon dynamiquement pour une carte réseau, indiquez « auto » comme valeur pour le paramètre `TargetNicIP`.
    - Utilisez les valeurs « Primary », « Secondary » ou « DoNotCreate » pour le paramètre `TargetNicSelectionType` afin de spécifier si la carte réseau doit être principale, secondaire ou qu’elle ne doit pas être créée sur la machine virtuelle migrée. Une seule carte réseau peut être spécifiée comme carte réseau principale pour la machine virtuelle. 
    - Pour créer une carte réseau principale, vous devez également spécifier les autres cartes réseau qui doivent être secondaires ou qui ne doivent pas être créées sur la machine virtuelle migrée.  
    - Pour changer le sous-réseau de la carte réseau, spécifiez le nom du sous-réseau à l’aide du paramètre `TargetNicSubnet`.

 - **Zone de disponibilité** : pour utiliser des zones de disponibilité, spécifiez la valeur de zone de disponibilité souhaitée pour le paramètre `TargetAvailabilityZone`.
 - **Groupe à haute disponibilité** : pour utiliser un groupe à haute disponibilité, spécifiez son ID pour le paramètre `TargetAvailabilitySet`.

Dans l’exemple suivant, nous allons mettre à jour la configuration de la carte réseau en définissant la première carte réseau comme carte principale et en lui affectant une adresse IP statique. Nous allons ignorer la deuxième carte réseau pour la migration, et mettre à jour le nom et la taille de la machine virtuelle cible. 

```azurepowershell
# Specify the NIC properties to be updated for a replicating VM. 
$NicMapping = @()
$NicMapping1 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[0].NicId -TargetNicIP "xxx.xxx.xxx.xxx" -TargetNicSelectionType "Primary"
$NicMapping2 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[1].NicId -TargetNicSelectionType "DoNotCreate"

$NicMapping += $NicMapping1
$NicMapping += $NicMapping2

# Update the name, size and NIC configuration of a replicating server
$UpdateJob = Set-AzMigrateServerReplication -InputObject $ReplicatingServer -TargetVMSize "Standard_DS13_v2" -TargetVMName "MyMigratedVM" -NicToUpdate $NicMapping

# Track job status to check for completion
while (($UpdateJob.State -eq "InProgress") -or ($UpdateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $UpdateJob = Get-AzMigrateJob -InputObject $UpdateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $UpdateJob.State
```

Vous pouvez également lister tous les serveurs de réplication d’un projet Azure Migrate, puis utiliser l’identificateur de machine virtuelle de réplication pour mettre à jour les propriétés de la machine virtuelle.

```azurepowershell
# List all replicating VMs in an Azure Migrate project and filter the result for selecting the replication VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName | where MachineName -eq $DiscoveredServer.DisplayName

# Retrieve replicating VM details using replicating VM identifier
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id 
```


## <a name="run-a-test-migration"></a>Exécuter un test de migration

Quand la réplication différentielle commence, vous pouvez exécuter une migration de test pour les machines virtuelles avant d’exécuter une migration complète vers Azure. Nous vous recommandons vivement d’effectuer une migration de test au moins une fois pour chaque machine avant sa migration. L’applet de commande retourne une tâche qui peut être suivie pour superviser l’état de l’opération. 

- L’exécution d’une migration de test permet de vérifier que la migration fonctionnera comme prévu. Une migration de test n’a pas d’impact sur la machine locale, qui reste opérationnelle et qui continue la réplication. 
- Une migration de test simule la migration en créant une machine virtuelle Azure à l’aide de données répliquées (il s’agit en général d’une migration vers un réseau virtuel hors production dans votre abonnement Azure).
- Vous pouvez utiliser la machine virtuelle Azure de test répliquée pour valider la migration, effectuer des tests d’applications et résoudre les éventuels problèmes avant la migration complète.

Sélectionnez le réseau virtuel Azure à utiliser pour le test en spécifiant l’ID du réseau virtuel à l’aide du paramètre `TestNetworkID`.

```azurepowershell
# Retrieve the Azure virtual network created for testing 
$TestVirtualNetwork = Get-AzVirtualNetwork -Name MyTestVirtualNetwork

# Start test migration for a replicating server
$TestMigrationJob = Start-AzMigrateTestMigration -InputObject $ReplicatingServer -TestNetworkID $TestVirtualNetwork.Id

# Track job status to check for completion
while (($TestMigrationJob.State -eq "InProgress") -or ($TestMigrationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $TestMigrationJob = Get-AzMigrateJob -InputObject $TestMigrationJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TestMigrationJob.State
```

Une fois le test terminé, nettoyez la migration de test à l’aide de l’applet de commande `Start-AzMigrateTestMigrationCleanup`. L’applet de commande retourne une tâche qui peut être suivie pour superviser l’état de l’opération. 

```azurepowershell
# Clean-up test migration for a replicating server
$CleanupTestMigrationJob = Start-AzMigrateTestMigrationCleanup -InputObject $ReplicatingServer

# Track job status to check for completion
while (($CleanupTestMigrationJob.State -eq "InProgress") -or ($CleanupTestMigrationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $CleanupTestMigrationJob = Get-AzMigrateJob -InputObject $CleanupTestMigrationJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $CleanupTestMigrationJob.State
```

## <a name="migrate-vms"></a>Migrer des machines virtuelles

Après avoir vérifié que la migration de test fonctionne comme prévu, vous pouvez migrer le serveur de réplication à l’aide de l’applet de commande suivante. L’applet de commande retourne une tâche qui peut être suivie pour superviser l’état de l’opération. 

Si vous ne voulez pas arrêter le serveur source, n’utilisez pas le paramètre `TurnOffSourceServer`.

```azurepowershell
# Start migration for a replicating server and turn off source server as part of migration
$MigrateJob = Start-AzMigrateServerMigration -InputObject $ReplicatingServer -TurnOffSourceServer 

# Track job status to check for completion
while (($MigrateJob.State -eq "InProgress") -or ($MigrateJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $MigrateJob.State
```

## <a name="complete-the-migration"></a>Effectuer la migration

1. Une fois la migration terminée, arrêtez la réplication de la machine locale et nettoyez les informations d’état de réplication de la machine virtuelle à l’aide de l’applet de commande suivante. L’applet de commande retourne une tâche qui peut être suivie pour superviser l’état de l’opération. 

```azurepowershell
# Stop replication for a migrated server
$StopReplicationJob = Remove-AzMigrateServerReplication -InputObject $ReplicatingServer 

# Track job status to check for completion
while (($StopReplicationJob.State -eq "InProgress") -or ($StopReplicationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $StopReplicationJob = Get-AzMigrateJob -InputObject $StopReplicationJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $StopReplicationJob.State
```

2. Installez l’agent [Linux](../virtual-machines/extensions/agent-linux.md) sur les machines migrées si la machine exécute le système d’exploitation Linux. Nous installons automatiquement l’agent de machine virtuelle pour les machines virtuelles Windows pendant la migration.
3. Effectuez les éventuels ajustements post-migration de l’application, comme la mise à jour des chaînes de connexion de base de données et les configurations du serveur web.
4. Effectuez les tests finaux de réception de l’application et de la migration sur l’application migrée qui s’exécute maintenant dans Azure.
5. Réduisez le trafic vers l’instance de machine virtuelle Azure migrée.
6. Supprimez les machines virtuelles locales de votre inventaire des machines virtuelles locales.
7. Supprimez les machines virtuelles locales des sauvegardes locales.
8. Mettez à jour la documentation interne en y mentionnant le nouvel emplacement et la nouvelle adresse IP des machines virtuelles Azure. 

## <a name="post-migration-best-practices"></a>Bonnes pratiques après la migration

- Pour une meilleure résilience :
    - Sécurisez les données en sauvegardant les machines virtuelles Azure avec le service Sauvegarde Azure. [Plus d’informations](../backup/quick-backup-vm-portal.md)
    - Conservez les charges de travail en cours d’exécution et disponibles en continu en répliquant des machines virtuelles Azure vers une région secondaire avec Site Recovery. [Plus d’informations](../site-recovery/azure-to-azure-tutorial-enable-replication.md)
- Pour renforcer la sécurité :
    - Verrouillez et limitez l’accès du trafic entrant avec l’[administration juste-à-temps d’Azure Security Center](../security-center/security-center-just-in-time.md).
    - Limitez le trafic réseau vers les points de terminaison de gestion avec des [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md).
    - Déployez [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md) pour sécuriser les disques, et protégez les données contre le vol et les accès non autorisés.
    - Découvrez plus d’informations sur la [sécurisation des ressources IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) et visitez [Azure Security Center](https://azure.microsoft.com/services/security-center/).
- Pour la surveillance et la gestion :
-  Envisagez de déployer [Azure Cost Management](../cost-management-billing/cloudyn/overview.md) pour surveiller l’utilisation et les coûts des ressources.